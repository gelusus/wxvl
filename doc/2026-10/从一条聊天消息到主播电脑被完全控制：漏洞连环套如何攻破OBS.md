#  从一条聊天消息到主播电脑被完全控制：漏洞连环套如何攻破OBS  
 Z2O安全攻防   2026-10-04 12:36  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table># 一个存在漏洞的聊天窗口叠加层、一个禁用了沙盒的Chromium渲染器，以及一个已在野外被利用的V8引擎漏洞，三者结合足以将观众可控制的文本转化为本地代码执行，且OBS自身保持默认设置。我曾发现一款Twitch聊天窗口叠加层 (chat overlay)，它将在OBS的浏览器源 (Browser Source) 中将观众消息直接渲染为原始HTML。这能让观众在OBS内嵌的Chromium浏览器中执行JavaScript。而当时的最新版OBS自带的Chromium构建版本禁用了其正常的沙盒，并且其V8引擎版本仍然存在 CVE-2024-7971 漏洞——该漏洞已在野外被利用。将这些拼凑起来，一条消息始于Twitch聊天，终于对主播机器的完全控制。概念验证 (PoC)：OBS 32.2.2上实现的远程代码执行  
## 一张截图引发的调查  
  
我的一个朋友为OBS制作了一款显示Twitch聊天框的软件并在社交平台发布了截图。如果你对直播设置不熟悉，简单来说，一个聊天窗口叠加层本质上就是OBS通过浏览器源 (Browser Source) 在直播画面上渲染的一个微小网页。它可能用来拉取实时聊天消息、提醒、打赏 (Donation) 等信息。  
  
截图恰好展示了一部分代码，而其中一行代码立刻引起了我的警觉：聊天消息**未经任何清理**  
就直接作为HTML插入到页面中。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZh3FTtwiagK8FMutviasvhLpb3pnxWwpq2qSfl8AsRia75Yg3rCvnmotHLDRv1aQgrbkicC52U3uto7EBRHUESDb7niaUZ8nSZxNqag/640?wx_fmt=png&from=appmsg "")  
**引发整个调查的那条推文**  
  
这是一个**经典的跨站脚本 (Cross-Site Scripting, XSS)**  
 漏洞。你可能已经见过类似场景：观众控制消息内容，叠加层将其视为HTML而非普通文本处理，攻击者控制的内容即可在页面内执行代码。相信不少人看到这里已经摇了摇头，理由很充分。  
  
这让我想起了一个关于通过OBS的WebSocket接口进行攻击的旧视频。其思路是利用聊天XSS作为切入点，然后与OBS的本地WebSocket服务器通信，触发诸如切换场景或停止直播等动作。  
  
这条路今天已经不那么有趣了，因为OBS的WebSocket服务器默认**是禁用的**  
，启用时需要密码（系统会自动生成一个）。  
  
我想要更强大的效果：**一条Twitch消息、最新的OBS、默认配置、无需主播任何交互**  
，最终在机器上实现代码执行。  
  
WebSocket这条路达不到这个目标。但**浏览器**  
或许可以。  
## OBS内部藏着一个完整的浏览器  
  
OBS的浏览器源由**Chromium**  
通过**CEF**  
 (Chromium Embedded Framework， Chromium嵌入式框架) 驱动。它们用于创建聊天框、提醒、打赏插件、动画以及自定义叠加层。这个浏览器组件也用于支撑浏览器侧边栏 (Browser Docks) 和服务集成。  
  
所以，这个小小的聊天布局实际上运行在OBS内嵌的一个**完整的Chromium浏览器**  
中。  
  
有了XSS漏洞，观众控制的JavaScript现在就在主播机器内的Chromium中执行了。  
  
仅仅这样，还不足以在Windows上实现代码执行。浏览器设有专门的安全边界，以防止网页内容变成对宿主系统的控制。  
  
但OBS的浏览器**缺失了一个关键边界**  
。  
## Chromium沙盒被禁用了  
  
Chrome通常使用**沙盒 (sandbox)**  
 来隔离渲染器进程。即使攻击者利用内存破坏漏洞在渲染器内实现了原生代码执行，他们通常仍需突破沙盒才能触及主机系统。  
  
OBS在初始化其内嵌的CEF浏览器时使用了以下设置：  
```
CefString(&settings.log_file) = log_path_abs;settings.windowless_rendering_enabled = true;settings.no_sandbox = true; // <- 关键行：禁用沙盒！uint32_t obs_ver = obs_get_version();uint32_t obs_maj = obs_ver >> 24;
```  
  
该设置直接存在于当前的 **obs-browser源代码**  
 中。  
  
XSS目前给我们的仍然是JavaScript。但如果这个JavaScript能够利用V8漏洞转变为渲染器内的原生代码执行，那么**已经没有任何Chromium沙盒需要突破了**  
。  
  
于是我开始检查OBS内置的Chromium版本。结果只能说，并非我所希望的。  
## 此时，CVE-2024-7971登场  
  
撰写本文时，我测试的最新OBS版本使用的是 **Chromium 127.0.6533.120**  
 和 **V8 12.7.224.18**  
。  
  
这很有趣，因为 **CVE-2024-7971**  
 （一个V8类型混淆漏洞）影响的是 128.0.6613.84  
 之前的Chromium版本。谷歌于2024年8月21日在Chrome 128版本中修复了它。  
  
这不是一个理论上的浏览器漏洞。微软曾记录该漏洞被其追踪为**Citrine Sleet（柠檬黄雪貂）**  
 的朝鲜威胁组织所利用，并且CISA (美国网络安全和基础设施安全局) 已将其添加到“已知被利用的漏洞”目录中。  
  
在微软观察到的攻击中，利用V8漏洞赋予了攻击者在Chrome沙盒渲染器内的代码执行能力，但他们**仍然需要另一个漏洞来逃逸沙盒**  
。  
  
而在OBS内部，这层屏障**早已被禁用**  
。  
  
拼图至此完整：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZhcosXYPf0zhjdIB4VcdZISBxLq00u1iaoibJr20h45sQdQNMIRxoFBKyicmiaYPvdib8nEKeHvmOJ7P2icBKDicS7tictrPpnMFzl0nc0/640?wx_fmt=png&from=appmsg "")  
攻击链图示  
## 构建完整的漏洞利用链  
  
没有公开的概念验证 (PoC) 针对我测试的确切CEF构建版本，所以我构建了自己的利用程序。开发过程中，我启用了Chromium远程调试，使用开发者工具协议 (DevTools Protocol) 来检查渲染器和调试漏洞利用。  
  
在高层次上，V8漏洞赋予了页面本不应拥有的内存访问权限。利用程序进一步发展此能力，获得更广泛的进程内存访问，最终实现原生代码执行。我刻意不在这篇文章中透露漏洞利用的内部细节。  
  
最终的结果则更容易解释：观众发送一条恶意Twitch聊天消息 → 有漏洞的叠加层将其转变为JavaScript执行 → V8漏洞利用代码将JavaScript转变为原生代码执行 → 攻击者最终获得在主播机器上运行任意代码的能力。  
## “默认配置”意味着什么？  
  
这**并不**  
意味着一台新安装的OBS可以被Twitch聊天中的任何人远程利用。在我的演示中，**零点击**  
的入口点是那个存在漏洞的叠加层：主播必须正在使用一个将观众控制的内容渲染为HTML而没有进行恰当清理的浏览器源。  
  
然而，一旦该页面被加载，为了实现剩余的漏洞链，我**并没有**  
削弱OBS。没有设置WebSocket，没有管理员权限，用户没有更改任何沙盒选项，也无需主播点击任何东西。  
  
Twitch叠加层也只是到达浏览器的一种方式。更普遍地说，任何加载到OBS浏览器源或浏览器侧边栏的、攻击者控制的页面，都有可能直接从浏览器利用阶段开始攻击。只是**聊天XSS使得这条特定的攻击链从观众的视角看是远程且零点击的**  
。  
  
因此，有趣的部分并非XSS本身，而是**攻击者控制的网页内容正在何处运行**  
。  
## 为什么这很重要  
  
直播设置中充斥着网页内容。聊天框、打赏提醒、新关注者通知以及自定义插件都是网页，而其大部分数据来自互联网上的陌生人。  
  
**如果聊天消息是文本，就将其作为文本来渲染。如果你确实需要HTML，务必进行恰当的消毒清理。**  
## OBS正在采取什么措施  
  
OBS团队确认两项修复措施已在推进中。  
  
**第一项是升级内嵌的浏览器。**  
 主要的阻碍是新的Chrome Runtime，直到最近它还不支持OBS所依赖的那种离屏渲染 (Off-Screen Rendering)。Chromium 127于2024年7月达到稳定版本，这意味着OBS 32.2.2中的引擎现在大约落后了两年。  
  
这项升级已在路上：一个将 obs-browser  
 迁移至CEF 128+的拉取请求 (Pull Request) 目前正在审阅中，目标是将其纳入OBS Studio 33.0里程碑。一旦合并，它将修复本文中用到的特定V8漏洞。但OBS是一个主要由志愿者维护的开源项目，让这项升级走到这一步需要数月的测试和兼容性工作。  
  
**第二项是启用CEF沙盒**  
，该功能也正在作为同一更新的一部分进行测试。最初禁用沙盒是因为它会破坏某些服务集成的身份验证，但团队猜测这些问题现在可能已得到解决。**如果沙盒能被重新启用，仅利用浏览器漏洞将不再足以单独实现攻击**  
：攻击者还需要第二个漏洞来逃逸沙盒。  
  
对于一个其全部职责就是渲染可能不受信任的网页内容的组件来说，**这两层防护都至关重要**  
。将浏览器引擎保持在接近当前安全版本的水平，与为其加上沙盒保护同样重要。  
  
在这些更改发布之前，我们每个人可以掌控的部分是**加载到这些插件中的内容**  
。任何放入浏览器源的东西都应被视为不受信任的输入，特别是**任何插件都不应将观众的内容渲染为HTML**  
。这是主播或叠加层作者今天就可以做的修复。  
  
  
内部SRC专项学习圈 ，今年最后的  
半价  
优惠券来了，低于一个低危漏洞的价格，只需  
45  
￥  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/JnmoqeNZZwRd8n7EjV7X34hicvuiaLibY3mbs4LRDicLKgPePGjfWZVENgYDSkboPZNH2Ckd3XJsk6Fcu0iaZe1MSvyoLomQ57p6pvTqRw3qz1DE/640?wx_fmt=png&from=appmsg "")  
  
圈子专注于更新src相关：  
  
```
1、维护更新src专项漏洞知识库，包含原理、挖掘技巧、实战案例
2、分享src优质视频课程
3、分享src挖掘技巧tips
4、小群一起挖洞
```  
  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuaRqDOYRFjU73rIsVy2ISg41LkR0ezBlmjJY4Lwgg8mr1A5efwqe0yGE9KTQwLPJTe9zyv3wgYnhA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuY813zmiaXibeTuHFXd8WtJAOXg868PqXyjsACp9LhuEeyfB2kTZVOt5Pz48txg7ueRUvDdeefTNKdg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1 "")  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/h8P1KUHOKuZDDDv3NsbJDuSicLzBbwVDCPFgbmiaJ4ibf4LRgafQDdYodOgakdpbU1H6XfFQCL81VTudGBv2WniaDA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=2 "null")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/JnmoqeNZZwQxuVTqXKZ8JoibGsCu9g4OpSeIiaQ7sPlScekuyU258338AMJVG6aavrDNYe1MjA1GjuLIWTTvBxdicGicibtXQDlr9AwLr7jUjkwE/640?wx_fmt=png&from=appmsg "")  
  
图片  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuYx6e5OYqRUhe5nHp6uuOTahgbr35OD8B1WCHW2uGMetuDzTPJiaHibhWhMm8UQ5iboDmNKqrRfjIrXQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=5 "")  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuadANlnTubvh6Abe7UZLdQWr5g7s0TNF4tBZqNbdewPNswTDOfvN6PkggCqz8j3mib6Vf3z4ia83asg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=6 "")  
  
图片  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuaRqDOYRFjU73rIsVy2ISg4Bd1oBmTkA5xlNwZM5fLghYeibMBttWrf57h8sU7xDyTe5udCNicuHo8w/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=7 "")  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuYx6e5OYqRUhe5nHp6uuOTaTWxLibDHdqdx6IahjVWr6ficJWskIMjdrbYaLGBIVsbONxbb5ibDS5trQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/JnmoqeNZZwSgBovs69l864sgqthfrOHLNc9IGeyian7ibDiaT4GBQ7joWRyqQEEkjPQClbFus1KGMo7Dv6drmJ3NqiafU2dxOl8YOWsNHByG9Ow/640?wx_fmt=png&from=appmsg "")  
  
图片  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuadANlnTubvh6Abe7UZLdQWWIDTric5u0Q03o25wLLgNBwFd6t4ud64ACo8icCdQRzrEGezUzIKSvEA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=12 "")  
  
![Image](https://mmbiz.qpic.cn/sz_mmbiz_png/h8P1KUHOKuadANlnTubvh6Abe7UZLdQWXytl9Ioah3X7tw7EMlWV96wWXEHFEM4m6NwlvvkcmEcPqcxcE9MQDg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=14 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/JnmoqeNZZwQkS7KobQKXicYntibjUCVzIPN8gyrIEQE6O1pgiaZeRhyYPDHxxLgd0Ca2EvBZrWAhORVEmGSWzl1chVbbqp0orf2C6zRm59Skys/640?wx_fmt=jpeg "")  
  
  
  
