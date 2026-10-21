#  Plex媒体服务器未授权RCE链被在野利用,CISA已列入KEV  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-21 01:07  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NMyxFdHDybl4UUFWeMwzBtZxIc69t3hRohiaAft5osFOrZS9ialqtqE60D2CibvNjia6CX2bp3mqU45RA5Z2ayU7FiahXmViay3Ms34/640?from=appmsg "")  
> **导语**  
：Plex Media Server 被曝出一条无需登录凭据的远程代码执行链 plex2shell，由四个 CVE 串成。补丁已发布，攻击者动手更早——10 月 17 日起已在野投放挖矿木马和住宅代理。  
  
## 一、四个 CVE，一条链  
  
研究者是安全研究员 Zach Main（zmain.info）。7 月 27 日报给 Plex，8 月 4 日确认并支付赏金，9 月 23 日 CVE 编号落地，10 月 16 日公开披露：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">原语</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-96652</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">/player/timeline</span></code><section><span leaf=""> 的 </span><code><span leaf="">protocol=</span></code><span leaf=""> 参数未授权 SSRF</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-96654</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">环回鉴权豁免 + </span><code><span leaf="">searchOne</span></code><span leaf=""> 的 </span><code><span leaf="">identifier=</span></code><span leaf=""> 注入</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-96655</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">owner 鉴权下的转码器 SSRF + 任意路径写文件（</span><code><span leaf="">dump-yuv</span></code><span leaf="">）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-96656</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">未签名 codec 的 </span><code><span leaf="">dlopen</span></code><span leaf=""> 导致代码执行</span></section></td></tr></tbody></table>  
单看任何一个最高也只有 7.2 分，串起来却是 CVSS 9.8 的未授权 RCE，且不需要有效登录态。受影响的是 1.43.3.10828 之前的版本，修复版本 1.43.3.10861。  
  
![plex2shell 仓库概览](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6Mx4dIoBfaRibiaDMjfU8OANZobeML4IETXicibkWFSwZlfyLibBJj1RD3dficvvKWPjicibPDOibH0efibDHVk8ej2TatiagcuKYjv8JCkek/640?from=appmsg "plex2shell 仓库概览")  
  
图源：推文卡片截图  
## 二、链是怎么串起来的  
  
第一步是**伪造身份混过闸门**  
。/player/timeline  
 端点的 token 检查只判断 X-Plex-Token  
 这个头**存不存在**  
，从不校验值是否有效。随便填个 junk  
 就能进。  
  
第二步是**让服务器打自己**  
。播放器订阅回调的 URL 由 protocol  
、host  
、port  
 拼成，protocol  
 未经校验直接拼进去。攻击者把完整 URL 塞进这个字段，取消订阅时 PMS 就从真实的 127.0.0.1 源头发 POST——服务器自己把自己 SSRF 了。  
  
第三步是**环回免鉴权**  
。/system/agents/*  
 这套插件管理 API 对环回来源不要求 token，上一步的环回请求因此拿到通行证。接着 searchOne  
 的插件白名单检查是个摆设：装饰器包住的内部函数提前返回时返回值被丢弃，外层照常往下走，从未"检查"从未拦截过任何东西。  
  
第四步是**偷走管理员 token**  
。伪造的调用最终指向 networking.py:149  
 的 GetResultFromNetwork  
。PMS 会给发往 127.0.0.1 的请求自动附加 owner token，攻击者只要把 URL 写成 http://127.0.0.1.<attacker>.sslip.io/...  
，服务器看到前缀就贴上管理员凭据，而 DNS 把域名解析到攻击者的 IP。  
  
第五步是**投递载荷并执行**  
。拿到 owner token 后，调转码接口把 path  
 指向攻击者托管的一段"载体视频"，再用 TranscoderH264Options=dump-yuv  
 让 x264 把解码后的帧原样写成 Codecs 目录下的共享对象文件。下一次转码加载 codec 时 dlopen  
 了这个没签名的库，构造函数直接跑起来，攻击者拿到 uid=1000(plex) 的反弹 shell。  
  
![蓝队排查思路](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M5L1NhFBKAQmbA39Fl7KfhbS24pV40Dm7GFOkYuZkyJ2OicB6kGXNwfWxVF3vvViavwd7Cp4bvGaaibEzooMmQmQKG440o9iboRCE/640?from=appmsg "蓝队排查思路")  
## 三、找漏洞这件事，LLM 是怎么参与的  
  
Zach 在博客里写清了方法论：Recon（拆组件、画出输入到危险汇聚点的路径）→ Hunt（逐组件定向狩猎）→ Verify（专门起一个 agent 去**推翻**  
每个发现，推不倒才算数）→ Chain（找最大影响的组合路径），循环到收益递减。他强调这不是"叫 LLM 去黑 Plex"，判断全由人做，LLM 只让他在更高抽象层上引导。  
  
让 AI 自我证伪，比让 AI 找漏洞更有价值。  
## 四、攻击者比补丁先动了  
  
10 月 16 日公开披露，17 日起就在野被利用。蜜罐捕获的攻击行为很朴素：投放 XMRig 挖矿程序和住宅代理工具，矿池 ota.proxystream.st:3333  
，带 --rig-id plex  
 参数。  
  
10 月 20 日，CISA 把四个 CVE 全部列入已知被利用漏洞目录（KEV），期限分两档：CVE-2026-96654 和 -96656 是 10 月 23 日，CVE-2026-96652 和 -96655 是 11 月 3 日。其中 -96656 还需额外做取证排查（BOD 26-04）。"已知被勒索软件使用"一栏仍标记为未知。  
  
IOC 包括攻击源 IP 45.194.92.222  
、91.92.40.202  
、176.65.139.228  
，载荷托管地址 217.60.194.116  
、77.90.185.205  
，以及落地文件 /tmp/mylovelyminer<pid>  
、myawesomeproxy  
、/tmp/xsetup.lock  
 和 Codecs 目录下形如 zz<hex>.so  
 的植入库。攻击者随时会换名字换 IP。  
## 五、怎么查，怎么修  
  
**先修**  
。升级到 Plex Media Server 1.43.3.10861 或更高版本。NAS 用户要注意，包管理器版本可能还没跟上，得去官网下载页手动装。修完顺手撤掉 32400 端口转发。  
  
**再查**  
。研究者提供了 YARA 检测规则（GPL-3.0），按三档置信度分层：high 档匹配产物结构特征，攻击者改工具也照样命中；medium 档匹配公开 PoC 字符串，重新编译就失效；hunting 档只能排优先级。规则要求 YARA ≥ 4.2，作者 10 月 18 日用 4.5.7 验证过，素材包括真实转码器 dump 文件和被攻击实验室日志。  
  
社区还基于这套逻辑做了个**只读排查脚本**plex2shell-check  
（GPL-3.0）：  
```
curl -fsSLO https://raw.githubusercontent.com/ChrisLPJones/plex2shell-check/main/plex2shell-check.shless plex2shell-check.sh          # 以 root 运行前务必先读一遍sudo bash plex2shell-check.sh -o plex2shell-report.txt
```  
  
退出码 0 干净、1 有告警待看、2 发现入侵指标，全程不修改系统。它检查版本、日志里的攻击请求模式、Preferences.xml 残留的 dump-yuv  
、Codecs 里的可疑 .so  
、以 Plex 用户跑着的可疑进程、临时目录落地文件、到已知攻击 IP 和矿池端口 3333 的连接，以及 cron、systemd、SSH 公钥等持久化痕迹。-y  
 可挂 YARA 规则一起跑；它能识别原生安装、snap、Docker，甚至通过 /proc/<pid>/root  
 钻进运行中的容器，在 Proxmox 主机上跑一次即覆盖 LXC 里的 Plex。  
  
**下载地址**  
（均为防御侧工具）：  
  
**YARA 检测规则**  
 https://raw.githubusercontent.com/code-zm/plex2shell/main/detection/yara/plex2shell.yar  
  
**只读排查脚本**  
 https://raw.githubusercontent.com/ChrisLPJones/plex2shell-check/main/plex2shell-check.sh  
  
**排查脚本仓库**  
 https://github.com/ChrisLPJones/plex2shell-check  
  
**研究者分析长文**  
 https://zmain.info/blog/plex2shell  
  
**野利用取证 pcap**  
 https://github.com/BoredHackerBlog/plex2shell_pcap  
  
**Plex 官方安全公告**  
 https://forums.plex.tv/t/important-security-update-for-plex-media-server-v1-43-2-and-earlier/942319  
  
**Plex 官方下载页**  
 https://www.plex.tv/media-server-downloads/  
  
**CISA KEV 数据源**  
 https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json  
  
**处置顺序别搞反**  
。确认被入侵后，先备份 Logs、Preferences.xml 和可疑文件，再停 Plex、杀进程、删掉植入的 .so  
（或整个 Codecs 目录，Plex 会重新下载），**升级补丁后再重新启动**  
。这条链会偷走 owner token，必须改密码并执行"退出所有设备"。如果 Plex 用户有 sudo 或 docker 权限、或与其他服务共用 uid，建议直接重建主机。  
  
研究者的完整利用代码仓库公开（github.com/code-zm/plex2shell，GPL-3.0），但鉴于这条链此刻正在被人在野牟利使用，本文只提供防御侧的检测规则与只读排查工具，不分发可直接运行的武器化利用脚本。  
## 六、写在最后  
  
这条链最值得警惕的不是单个漏洞，而是两个架构习惯：**环回来源直接视为可信**  
，以及**服务端自动给"看起来像自己"的请求附加管理员凭据**  
。前者让盲 SSRF 变成内部 API 的钥匙，后者让凭据自己送上门。  
  
消费级服务直暴露公网不是原罪，但它同时具备庞大攻击面、宽松的默认信任模型和不打补丁的用户群——三者凑齐就是攻击者最爱的目标。修完补丁顺手关掉端口转发，性价比比任何检测规则都高。  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6NAE4HR1E2rDU6YFMOe4sNEicBJ4R1D2cc4sQYibncFdLL5xrjEQLqe0D2lHt16JibNnxgj0XE7SWuCoRzo0OlW9jAsHO5UrS0wtc/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
