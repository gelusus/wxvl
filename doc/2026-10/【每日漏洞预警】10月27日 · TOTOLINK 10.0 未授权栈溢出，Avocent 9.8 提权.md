#  【每日漏洞预警】10月27日 · TOTOLINK 10.0 未授权栈溢出，Avocent 9.8 提权  
原创 chicken
                    chicken  爱坤sec   2026-10-27 01:22  
  
每日漏洞预警  
  
10 月 27 日  
  
3 个高危漏洞 · 3 个未授权可打 · 滚动 24 小时  
  
3  
  
高危漏洞  
  
3  
  
未授权可打  
  
24h  
  
统计窗口  
  
今日概览  
  
过去 24 小时 NVD 新增 755 个 CVE  
，其中 CVSS ≥ 9.0 的 26 条  
；按本号主榜口径（≥9.5 / 未授权 / 外网可直接访问的 RCE  
）筛完只剩 3 组  
：TOTOLINK A950RG 的 10.0 未授权栈溢出  
（同批还有 3 条 root 命令注入）、Vertiv Avocent ACS8000 的 9.8 wsserver 未授权命令注入  
、Fortinet FortiSandbox 的 9.8 未授权认证绕过  
。被排除的高分条目：zip-rs 10.0 与 OpenHands SDK 9.8（纯库 / 容器内端口，不是公网暴露面）、Mautic 7 的 9.9 任意文件上传（要求已登录的 campaign 导入权限）、danmu_api 9.8 与 NexoPOS 9.8 及 ArcGIS 9.8（改配置 / 目录穿越 / SQL 注入，不属未授权 RCE）、Burrito 9.9、Consul-k8s 9.9（内网编排面、需凭据），以及 WolfVision Cynap 的 9.8（厂商 9 月 21 日  
已公开披露，属旧洞，不重复收录）。  
  
   
  
01  
CVE-2026-116508 / CVE-2026-116504 / CVE-2026-116506 / CVE-2026-116505  
  
TOTOLINK A950RG（AC1200 双频千兆路由，固件 V4.1.2cu.5268_B20221104）  
  
未授权栈溢出 → 直取 root，同批 3 条命令注入  
  
CVSS 10.0 · CRITICAL  
  
   
  
A950RG 的 web_cste/cgi-bin/infostat.cgi  
 是一个独立 MIPS CGI，被 lighttpd 用 cgi.assign = ("" => "")  
 全量映射，代码里没有任何 session 校验  
，也不调用 cstecgi.cgi 的鉴权门——只要能连到 80 端口就能打。漏洞点在 main  
：它把 strtol(getenv("CONTENT_LENGTH"))  
 的结果 +1 直接当成 fread  
 的元素个数，写进一个 4096 字节  
的栈数组，既不比对 sizeof 也没有上限。这个帧总共 8272 字节  
，v11  
 后面紧跟第二个 4100 字节缓冲，再往后就是保存的寄存器与返回地址——距 v11 起点 8200 字节  
就是 main 的返回地址。更致命的是运行环境：目标非 PIE  
、编译未开栈保护  
、ASLR 关闭  
，30 KB 的请求允许量意味着返回地址之后还有 25 KB 完全可控的数据，直接写 ROP 链即可，不需要爆破。NVD 给的向量是 AV:N/AC:L/PR:N/UI:N/S:C  
，满分 10.0  
（提交者自评 9.8）。同一次提交里还有三条同设备的问题：CVE-2026-116504（9.9）  
云升级路径 setCloudUpgradeFW  
 把 FileName  
 拼进 "rm -rf %s"  
 交给 CsteSystem，错误分支（文件不存在或小于 1000 字节）必走，所以不需要真固件镜像；CVE-2026-116506（9.9）  
是二阶注入，Web 处理器把参数写进 MIB，bin/sysconf  
 再把 MIB 值插进二十多个 shell 模板（route/brctl/reload/udhcpc…）以 root 执行；CVE-2026-116505（9.1）  
则从 /var/lib/misc/udhcpd.leases  
 读 MAC 拼进 system()  
，要靠 DHCP 侧先落一条畸形租约。三条都需要先登录（PR:L），但配合上面的未授权溢出就是「先拿 shell、再持久化」的完整链路。  
  
  
📦 影响版本  
  
固件 V4.1.2cu.5268_B20221104（A950RG，C8188R / IP04308 平台）；更早版本未逐一验证，按同架构推断同样受影响  
  
✅ 修复版本  
  
厂商未公布修复版本，也未发布补丁固件；只能隔离管理面或替换设备  
  
  
🎯 利用状态：  
 ⚠️ 提交材料（含寄存器偏移、ROP 布局与请求样例）已在 gist 公开，VulDB 已收录；暂无独立 EXP 仓库  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：13 个（fofa 全网测绘，2026-10-27）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
02  
CVE-2026-65184  
  
Vertiv Avocent ACS8000（串口控制台服务器 / wsserver 终端桥接）  
  
未授权命令注入：WebSocket 字段直进 system()/popen()，以 root 执行  
  
CVSS 9.8 · CRITICAL  
  
   
  
ACS8000 是所有被管设备的“总闸”，而它自己的 Web 面出了个不需要凭据的口子。wsserver  
（/usr/bin/wsserver）由 /etc/init.d/wstermd  
 用 start-stop-daemon -S  
 启动，没有 --chuid  
，所以它就是 root  
；它把 libwebsockets 上下文绑在环回 2000 端口  
，然后由 Apache 在 /etc/apache2/httpd.conf  
 里用一条裸的反代把 /termviewer  
 暴露到网络。对比一下就看出问题：同级的 /plugin/avtrans  
、/plugin/fileTransfer  
 都要求校验过的客户端证书，/appliance/*  
 与 REST 虚主机直接 Require all denied  
，只有 /termviewer  
 既没有 <Location> Require  
、也没有 AuthType / SSLVerifyClient。Apache 的默认 deny 管的是文件系统，管不到一条被代理的 WebSocket，于是这个路径未授权可达  
。进去之后，守护进程把客户端 WebSocket 里的字段直接拼成 shell 命令交给 system()  
 和 popen()  
，不引号、不转义、不做字符集校验；公开密钥会话路径还不做 appliance-session 检查  
。结果就是：只要能连到设备的 Web 端口，就能以 root 执行任意命令，顺带拿到它背后的每一条串口会话和被管设备。这条是 runZero 的 HD Moore 在固件分析 + 虚拟化目标（ACS8000 固件 2.32.4）上发现的，2026-10-26 公开披露  
；披露者自己注明“尚未在真机上完整验证”。  
  
  
📦 影响版本  
  
ACS8000 固件（验证版本 2.32.4，推测同系列 2.x 均受影响）；只要 /termviewer 可达即成立  
  
✅ 修复版本  
  
厂商目前未公布修复版本（2026-08-27 已通知 Vertiv，2026-10-26 公开披露）  
  
  
🎯 利用状态：  
 ⚠️ 公开分析已发布（含固件路径与反代配置证据），暂无公开 EXP 代码  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：27 个（fofa 全网测绘，2026-10-27）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
03  
CVE-2026-107656  
  
Fortinet FortiSandbox / FortiSandbox PaaS（沙箱设备 Web 管理面）  
  
未授权认证绕过（CWE-288）→ 以设备权限执行任意代码或命令  
  
CVSS 9.8 · CRITICAL  
  
   
  
FortiSandbox 的 GUI 存在“用替代路径绕开认证”  
的问题：构造过的 HTTP 请求可以跳过登录直接进入管理接口，厂商把它归到 CWE-288 Authentication Bypass Using an Alternate Path or Channel  
，影响面写的是「执行未授权的代码或命令」（Execute unauthorized code or commands）。这类设备通常同时挂着沙箱投递入口（样本上传）和管理 GUI，在不少单位里它的 Web 面是对公网开放  
的，所以这条的门槛实际上只有一个 URL。评分上两家略有差异：Fortinet 自己的公告给 CVSSv3 9.6  
，NVD 记的是 9.8  
，本号统一按 v3.1 的 9.8 标注。公告号 FG-IR-26-185  
，2026-10-13  
 首次发布，NVD 在 10-26 才收录；披露方是 Fortinet 产品安全团队（内部发现）与 TrendAI Research 的 Jason McFadyen。受影响的 4.2 系列在公告里没有修复版本，PaaS 4.4 也是「迁移到修复版本」，还在跑这两个线的要按“换版本”而不是“打补丁”来排期。  
  
  
📦 影响版本  
  
FortiSandbox 5.0.0–5.0.6、4.4.0–4.4.9、4.2 全版本；FortiSandbox PaaS 5.0.0–5.0.6、4.4 全版本、4.2 全版本（5.2 不受影响）  
  
✅ 修复版本  
  
5.0.7 / 4.4.10 及以上（PaaS 同为 5.0.7）；4.2 系列与 PaaS 4.4 在公告里没有补丁，只能迁移到修复版本  
  
  
🎯 利用状态：  
 ⚠️ 厂商公告（FG-IR-26-185）与修复版本已公开，暂无公开 PoC / EXP 仓库  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：196 个（fofa 全网测绘，2026-10-27）  
  
参考 / PoC  
```
```  
  
  
   
  
处置建议  
  
1TOTOLINK A950RG：  
厂商至今没有发布修复固件，唯一有效处置是把管理面从公网撤下来  
（关掉 WAN 侧 80/443 的远程管理、只留内网 + VLAN）。这台机器是 2022 年的固件，infostat.cgi 不做任何鉴权、程序非 PIE 且编译时没开栈保护、运行时 ASLR 还是关的，一旦能被访问到就是稳定拿 shell，别指望“改个弱口令”能挡住。  
  
2FortiSandbox：  
升到 5.0.7 / 4.4.10  
 及以上；PaaS 4.4 全版本没有修复版本，按公告迁移。升不了先把 GUI（HTTPS 管理口）收紧到运维网段，别让沙箱设备的 Web 面挂在公网——它通常还兼着文件投递入口。  
  
3Avocent ACS8000：  
厂商目前没有公布修复版本  
。立刻限制 80/443，特别是 /termviewer  
 这个反代路径（其余 /plugin/* 都要求客户端证书，唯独它没有）；串口控制台服务器是所有被管设备的“总闸”，被拿下的代价远大于一台普通主机，建议放到带认证的跳板之后。  
  
4今日未入主榜但同样要看：  
Mautic 7 的 campaign 导入任意文件上传（9.9，需 campaign 导入权限，但落到 web 根就能执行）；danmu_api < 1.21.7（9.8，默认 TOKEN 未改 + ADMIN_TOKEN 未设时可未授权改环境变量、劫持代理造成 SSRF）；NexoPOS 6.2.4 的 api/update 目录穿越（9.8）；Esri ArcGIS Server ≤ 12.1 的 SQL 注入（9.8）；OpenHands SDK 的后端容器把 sandbox agent 绑 0.0.0.0 且默认无鉴权（9.8），自建部署要确认端口没有对外发布。  
  
⚠️ 安全提示  
  
本文内容仅用于安全研究与防御加固学习，所有信息来自厂商公告与公开漏洞库。请勿对未授权目标实施攻击，否则后果自负。  
  
— EOF —  
  
