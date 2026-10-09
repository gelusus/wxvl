#  思科：注意可导致 Nexus 交换机遭接管的多个严重漏洞  
Bill Toulas
                    Bill Toulas  代码卫士   2026-10-09 07:21  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**思科发布了五份关于****NX-OS****数据中心网络操作系统中五个严重漏洞的安全公告，这些漏洞可能被用于在****Nexus****交换机上以****root****权限运行任意代码。**  
  
如无法实现远程代码执行，攻击者还可以利用这些漏洞使进程崩溃并迫使受影响设备重新加载，从而导致拒绝服务状况。这些漏洞影响Nexus 3000和Nexus 9000系列交换机中的NX-API、下一代OAM（NGOAM）和MPLS OAM功能。  
  
所有漏洞都与验证失败有关。这些漏洞影响处于独立NX-OS模式的Nexus 3000和Nexus 9000系列交换机。不过，成功利用取决于NX-API、下一代OAM（NGOAM）和MPLS OAM功能中是否至少一个处于活动状态：  
  
- CVE-2026-76471：输入验证不充分；可通过向NX-API发送特殊构造的HTTP请求实施利用，而该功能默认禁用。  
  
- CVE-2026-76485、CVE-2026-76486和CVE-2026-76501：对IP流量的验证不当，可通过向IP接口发送特殊构造的数据包来利用；这三个漏洞都要求启用NGOAM。  
  
- CVE-2026-76465：对MPLS echo-request数据包的验证不当，可通过向受影响设备的IP地址发送特殊构造的请求来利用。  
  
  
  
其中，利用CVE-2026-76486漏洞还要求启用IPv6分段路由 (SRv6) 或网络虚拟化覆盖。思科在公告中提到：“网络虚拟化覆盖还要求将VXLAN以太网VPN VXLAN网络标识符映射到网络虚拟化端点接口，并且至少学习到一个对等VXLAN隧道端点，例如BGP EVPN或入口复制静态对等体。” 如启用了SRv6，则CVE-2026-76501可被利用，而SRv6仅在某些Nexus 9000型号上受支持。  
  
至于CVE-2026-76465，必须显式激活MPLS OAM，因为该功能默认禁用。配备Silicon One ASIC的Nexus 9000交换机不支持该功能，因此不受该漏洞影响。思科指出，Nexus 7000交换机以及以ACI模式运行的Nexus 9000交换机不受这五个漏洞影响。  
  
思科建议将NX-OS版本升级到已修复版本，可通过Software Checker工具确定具体版本。该公司建议，如果不需要NGOAM、NX-API或MPLS OAM功能，则应禁用以消除攻击途径。思科还为所有五个漏洞提供了临时Live Protect防护，这是一种面向尚无法升级并重启的交换机的保护系统。  
  
所有五个漏洞都是在内部安全测试期间发现的，思科表示，在发布公告时，尚未得知有公开披露或恶意利用的情况。  
  
除了此次修复的五个Nexus漏洞外，思科还发布了针对Cisco License（原Smart Software Manager）的安全加固更新，涵盖关键功能缺少身份验证（CVE-2026-76480，CVSS 9.8）、加密签名验证不当（CVE-2026-76482，CVSS 10.0）、凭据保护不足（CVE-2026-76483，CVSS 9.1）以及代码注入（CVE-2026-76484，CVSS 8.8）。任何配置下的受影响版本都存在漏洞，思科建议升级到10-202609版本，并且没有可用的应变方法。以Smart Software Manager命名的较旧版本无法获得这些漏洞的补丁，因此思科建议在这些情况下迁移到受支持的版本。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[思科：ISE 认证绕过满分 0day 已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527173&idx=1&sn=1ae52540af18bbb2b176b85380e718cc&scene=21#wechat_redirect)  
  
  
[思科紧急提醒：Secure Email Gateway 0day 漏洞已被用于运行恶意代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527140&idx=1&sn=81268f1bb9309a82d8c875415dcca36a&scene=21#wechat_redirect)  
  
  
[思科：注意 CVSS 满分 Crosswork SQL 命令注入漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526935&idx=2&sn=6df734eb52e6f93d21542cbc093f1c92&scene=21#wechat_redirect)  
  
  
[思科提醒注意多个高危 ClamAV 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526852&idx=1&sn=8d342c76e61f26dbc2a64eeb6b0648c9&scene=21#wechat_redirect)  
  
  
[思科修复12个 SD-WAN 和 IOS XE 漏洞，含多个高危](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526840&idx=1&sn=d925710bf66e2f5889e993d29d7d276c&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/cisco-warns-of-critical-flaws-allowing-nexus-switch-takeover/  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
