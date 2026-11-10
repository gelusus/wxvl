#  谷歌修复Chrome在野0Day漏洞，可远程执行恶意代码  
 FreeBuf   2026-11-10 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3yD4libzqiavBRl14iakh1UM1IZFjPt1icYJZl2fLuwv2rv8m5m1j9bwe61CPZeDIG8D0aR8tludg0Z4Jt9LCFvPicoiajkpUXvJQaU/640?wx_fmt=gif "")  
  
  
![Chrome在野0Day修复](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1MtLWgJicVpyBBAu2yn7pnTZOic5MDKbUUsWeicwMymLtZgBldOPqcWiaE1RhWXpcGOxtgaicB99mJDLRVzwGDSHxLCvAOQrKJllHg/640?wx_fmt=png "")  
  
  
Part  
01  
  
0Day已遭在野利用  
  
  
谷歌已修复一款Chrome 0Day漏洞，目前攻击者已在利用该漏洞。该漏洞编号为CVE-2026-125350，远程攻击者可利用其在浏览器中执行恶意代码。  
  
  
本次稳定通道更新后，Windows和Mac平台Chrome版本将升级至157.0.8103.50/.51，Linux平台升级至157.0.8103.50。更新将在未来数天至数周内逐步推送给所有用户。Chrome安全团队表示，在绝大多数用户完成修复前，漏洞细节将不予公开。  
  
  
CVE-2026-125350是Chrome XML组件中的高危释放后重用漏洞。谷歌确认，目前已存在该漏洞的在野利用代码。一名匿名研究人员于2026年11月6日上报该漏洞，仅3天后补丁就正式发布。  
  
  
谷歌尚未披露利用该漏洞的攻击主体，也未公布受攻击的用户规模。释放后重用漏洞的成因是程序在内存被释放后仍继续调用该段内存。攻击者通常可通过构造恶意网页，利用这类漏洞造成浏览器崩溃，或执行任意代码。  
  
  
Part  
02  
  
同批修复8个漏洞  
  
  
本次更新共包含9个安全修复，其中7个评级为高危。谷歌公布的修复列表包括：WebGL组件未初始化资源漏洞（CVE-2026-125345）、FileSystem组件授权不当漏洞（CVE-2026-125347）、合成组件缓冲区溢出漏洞（CVE-2026-125343）、GPU组件越界写入漏洞（CVE-2026-125348）、V8引擎类型混淆漏洞（CVE-2026-125344）。  
  
  
XML组件存在的另一处释放后重用漏洞（CVE-2026-125346）由Neonix Security的Arni Hardarson上报。本次更新还修复了2个中危漏洞，分别为CVE-2026-125349和CVE-2026-125342，均属于GPU组件的未初始化资源漏洞，后者由Vo Duc Thang上报。其余漏洞大多由谷歌内部团队发现。待绝大多数用户完成更新后，谷歌将公开完整的问题追踪列表。  
  
  
按CVE编号、组件、类型与上报日期梳理：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3Y1RUEnlLvtNV45329oAY6OiaG7xCcA5ic7cdfz5xYDm0WUd9WRvcicJHtasBia6XtQhKmxpfn8Xr1d0qXR3XLDIUjxSLY2Z5ON7s/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
03  
  
漏洞利用门槛极低  
  
  
浏览器日常会处理大量不可信内容，因此XML等组件的内存漏洞一直是攻击者的首选目标。受害者只需打开恶意链接就可能中招。攻击者可通过钓鱼邮件、广告或被入侵的网站传播这类恶意链接。  
  
  
今年6月谷歌修复另一款Chrome 0Day漏洞时，攻击模式完全一致，当时的漏洞同样允许攻击者通过构造网页实现远程代码执行。本次补丁也同步推送给企业用户，扩展稳定通道的156.0.8078.96版本已包含该0Day漏洞的修复。  
  
  
本次补丁发布距Chrome 157正式上线仅过去数天。此前的Chrome 157版本共修复213个安全问题，其中7个为严重级别。谷歌表示，多数安全漏洞可被AddressSanitizer、MemorySanitizer、控制流完整性防护机制、libFuzzer等工具提前捕获。这些工具可在版本推送至稳定通道前发现内存错误，但本次这类0Day漏洞说明，仍有漏网之鱼。  
  
  
Chrome会在后台自动下载更新，但补丁需要重启浏览器才能生效。用户可打开菜单，依次选择"帮助"-"关于Google Chrome"，待更新下载完成后点击"重启"即可。Linux用户需通过系统包管理器完成更新。使用Chromium内核浏览器的用户需留意对应厂商发布的补丁，这类浏览器大多与Chrome共用相同的引擎代码。  
  
  
参考来源：  
  
Google Chrome 0-Day Vulnerability in the Wild to Execute Malicious Code Remotely  
  
https://cybersecuritynews.com/google-chrome-0-day-exploited/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
  
