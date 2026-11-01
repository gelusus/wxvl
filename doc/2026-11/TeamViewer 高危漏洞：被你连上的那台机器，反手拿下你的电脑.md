#  TeamViewer 高危漏洞：被你连上的那台机器，反手拿下你的电脑  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-11-01 01:04  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6P80pxQiaZMMFxt0rZYuoYz202iaFDqn69MhJVEveQF1hnSwCvLueiapQHQysibibH1xzAZVvEtadyOungQuHZtTmrbaibj1TXFSpx7s/640?from=appmsg "")  
> **导语**  
：远程协助最讽刺的一幕：你主动连上去救人的那台电脑，反手把你拿下了。TeamViewer 一个整数溢出漏洞把这件事做实了——恶意被控端发一帧屏幕数据，IT 支持人员的机器就被拿下。  
  
  
![你连上去救的那台机器，反手拿下了你](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6PibQ9RMEb7m8e0ARaLXNBErI9w7zw46uYiaiaK0gIVFIM5OiaxHPd2OlHCAPsBYkZYkl6OHlgjhgxBfZMJIicS6qtzZ67sicZ3RMX1U/640?from=appmsg "你连上去救的那台机器，反手拿下了你")  
## 一、这次角色反了  
  
远程协助的安全模型一直建立在“我信任我连过去的那台机器”上：被控端只负责吐画面，键盘鼠标从我这头回去。这次漏洞把这层信任掀了个底朝天——猎人是 Host，猎物是 Viewer。  
  
Remedio（雷梅迪奥）研究员 Omri Dar 在 10 月 29 日公开了完整技术分析。一句话概括：Host 在会话建立时告诉 Viewer “我的屏幕多大”，Viewer 拿这几个数字去分配解码缓冲区，而这个乘法是在 32 位里做的，溢出了。  
  
![Remedio 官方示意图：远程协助的攻击方向反转](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6POGITNTTDw3dakXNpsBTn1IpLKgflVEmqjwYtKFldr4y9JcYToYh1iaRscc5Wzib2ccqXibLXWoibXMLBMYz34aaYsKtiaT59TftNU/640?from=appmsg "Remedio 官方示意图：远程协助的攻击方向反转")  
## 二、一个 32 位乘法把缓冲区算小了  
  
Viewer 会收到一条 DisplayParams 消息，带着宽、高和色深。构造函数拿它算分配：  
```
stride = (bpp_bits * (width + 0x10)) >> 3;plane.byteSize = (uint32)(stride * (height + 0x10));buf = operator new[]((ulong)(stride * (height + 0x10)));
```  
  
乘积在 32 位里回绕，之后才扩成 64 位交给分配器。Dar 举的例子是宽高各 0x8000（32768）、bpp 32：stride 是 131136，预期要 4,299,162,624 字节，落到 32 位回绕后实际只分到 4,195,328 字节，约 4MB。可这个缓冲区对外仍宣称自己高 32,784 行。  
  
RLE 解码器按 base + y * stride 写像素，第 32 行起全部越界，直接踩进堆上相邻的对象。  
## 三、护栏读的是另一个数字  
  
越界写之所以能一路走到底，是因为边界检查压根不看实际分配：  
```
if ((D_height < y2) || (D_width < x2)) return "invalid rectangle";
```  
  
D_width、D_height 是声明尺寸，没截断。于是 (0, 0, 32768, 32768) 这种覆盖全屏的矩形轻松通过检查。分配算的是一个数，护栏验的是另一个数——这就是整个 bug。  
  
更要命的是攻击者能挑缓冲区大小。换成 width = 0xFFFFFFF5、height = 1、bpp = 8，分配会缩到约 0x440 字节，落进小块堆，正好便于堆塑形。这不是单纯的远程崩溃，而是一个落点由攻击者挑选的线性堆溢出原语。  
## 四、渲染路径是攻击者说了算  
  
CBitmapBuffer 只在没有硬件视频解码器时才启用，否则帧走 AV1 或 H.264。而编解码器的协商由 Host 主导——Host 拒绝协商视频编解码器，就把 Viewer 逼上 bitmap 这条路，主动把脆弱表面送到对方面前。  
  
剩下的就是等。屏幕更新以压缩矩形持续到达，Viewer 的解码循环在整个会话期间一直挂着，随时准备解下一帧。信任边界就摆在链路上：矩形、压缩载荷、DisplayParams 的几何参数，全是远端可控字节，接收路径上没有任何上限校验。  
## 五、从崩溃到代码执行  
  
崩掉很容易，定级高低取决于同样的写能不能在完全加固的机器上拿到执行。Dar 的答案是能。  
  
被破坏的 CBitmapBuffer 是个活对象，Viewer 之后会通过它自身第一个字段里的指针去调用，覆写这个指针就能把调用重定向。CFG 和 DEP 正是为了拦这个，但 PoC 两个都绕过去了，把调用路由到 WinExec，跑起攻击者选定的程序。  
  
整条链路跑在正式发行版二进制上，客户端是最新的、系统防护是开着的。PoC 演示里，攻击端跑着 VIEWER2SHELL，受害端屏幕上直接弹出了计算器。  
  
![Remedio 公开 PoC 实测：恶意主机反向拿下支持人员机器](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6MCgv9hwicjeYM9SUI0icLmQDWqenyXRjrMDzdUIibFNiclYzMuOJicdibh0YDGwZbsZaLqJkfwRV2U5CeVG42pXx1oCmasDG3oN3Nz0/640?from=appmsg "Remedio 公开 PoC 实测：恶意主机反向拿下支持人员机器")  
## 六、这条链为什么刺得深  
  
触发成本等于产品正常工作流。你不需要骗人点任何额外的东西，IT 支持人员的日常就是接受一个远程协助请求，CVSS 里的 UI:R 说的正是这件事。  
  
目标选取还有方向性：天天被叫去连陌生机器的就是他们，而这类岗位的机器上通常挂着域凭据、工单系统、VPN 客户端和内部运维工具，一次会话就能把整个内网的口子撕开。  
  
覆盖面也压不下去。TeamViewer 官方称装机量超过 25 亿台，Windows、Linux、macOS 全平台加安卓端都在射程内。  
## 七、影响面与处置  
  
漏洞编号 CVE-2026-111585，CWE-190 整数溢出，CVSS 3.1 评分 8.8（高危），向量 AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H。  
  
受影响的是 Full Client（Windows、Linux、macOS）、Windows Portable 和安卓 Remote Control，15.83 之前全部中招。要注意，9 月发布的 15.82 安全更新并没有修掉它。  
  
处置只有一条：升到 15.83 及以上，厂商没给任何临时绕过方案。同一批公告 TV-2026-1011 还修了另外两个漏洞，其中 CVE-2026-92374 是 macOS 特权助手的 TOCTOU 加符号链接处理不当导致的本地提权；同日的 TV-2026-1012 又修了远程管理插件的六个漏洞，跑 Host 的也得跟着升。  
  
目前厂商称未发现在野利用，漏洞也还没进 CISA 的 KEV 目录，Remedio 没有公开利用代码。  
## 八、时间线  
- 2026-08-14：报告给 TeamViewer 产品安全团队  
  
- 2026-10-22：厂商接受，定级 8.8，分配 CVE-2026-111585  
  
- 2026-10-27：15.83 发布修复，公告 TV-2026-1011  
  
- 2026-10-29：Remedio 公开技术分析与演示视频  
  
- 2026-10-30：The Hacker News（黑客新闻）报道  
  
## 九、给防守方的三条硬规矩  
  
立刻升级 15.83 以上，Host 也要升。别把“接受一次远程协助”当成安全边界，支持人员的机器要分层隔离，域凭据不进远程会话。告警侧盯住异常的会话发起方：从低信誉资产发起、批量接受、深夜时段突然活跃的账号，都是可回溯的特征。  
  
这个 bug 的本质就是一个 32 位的乘法错误。凡是“一处算尺寸、另一处查边界”的地方，都该问一句——这两个数会不会对不上。分配和护栏，永远不该读两个不同的数字。  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。文中技术细节引自 Remedio Research、The Hacker News 与 TeamViewer 官方公告。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OIia0LX2xWhtwIrj9UUn7fJGuhYwAG0E2QTdPeI2T46fd2EEBicvp6WlNicMIz89MC5SQiaanwZ27pu9zWAoniaibQvTibh9mc6y32b8/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PLSpVzNM3fN3yn6edqA3bCPC7bssb09ficTxicPYxBueEuj4O5Rcr5aZmIyXccoVhQcFgV43wvATVpHWDuibomLzaGp28ibKnXTD4/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6MKLtzTicXRKfzib4gMfpxEyiaAOj0DEwq3qyRpJVzPLP2ibXUlCIZdZmTGXkfqhFWhnoyB2b2nh0ibjGtNRZKC9Ox0oeQWiauPwnArk/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6Mvs3F9Hiavh6A17ibGIq4gVUMticcryCicyFfHgokO6a8YhvSnOiaibGzA5W4rhcFibAbSzOyLlG9qja40a2ewb23myPLQdjl5InxbxI/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
