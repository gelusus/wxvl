#  UEFI 严重漏洞潜伏16年，可绕过 Secure Boot 在网络启动服务器上执行代码  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-10-29 22:59  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**多数硬件厂商用来构建****UEFI****固件的开源代码****EDK II****中存在一个潜伏****16****年的漏洞（****CVE-2026-97631****，****CVSS****评分****9.5****）。无需密码或已有机器的访问权限，攻击者就能在操作系统加载前在机器上运行代码，不过仅限于机器进行网络启动的短暂窗口内。**  
  
UEFI固件是计算机开机后、操作系统之前最先运行的软件。攻击者若能在这样的网络启动期间用构造的网络消息触达机器，就可以使启动挂起，或者在针对确切固件构建量身定制工作后，在Secure Boot和其他启动防御加载之前运行自己的代码。  
  
Red Hat和SUSE都提供受影响代码，二者将该漏洞评为攻击者从相邻网络位置利用，而不是跨越开放互联网利用。EDK II背后的项目TianoCore于10月22日发布公告提到，已有代码修复方案，但尚无EDK II稳定版本包含该修复，公告仍未列出大多数用户无法安装的已修复版本。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXyMCBfSAamN0wwWbzTDT5E2mtHSqO9hX4ibJNVHxDF1vyHCUPydTg2H439RQFZFDYzOX71traxCRTtP4lswxck6bneDT6CDUpo/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞后果**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUREIxROlurYu8QAB4I3aaVuCN5Jiczcic49Dq4OSc1HFnslSYZvS6yeC45kUQQwyYZNp2Ld9VNEc2RybfuhfBTgXjk5aSxgfWSY/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该漏洞是EDK II网络启动代码 (PXE) 中的缓冲区溢出。在PXE启动期间，机器从网络而不是本地磁盘加载启动镜像。  
  
作为其中一部分，PXE驱动监听ICMP错误消息和网络设备回送的小型状态消息，并将每条消息复制到固定的504字节内存区域。该复制中的大小检查在IPv4和IPv6两个版本的代码中都有错误，因此超过504字节的消息会溢出该区域，并用攻击者选择的数据覆盖附近内存。攻击者只需在机器正处于网络启动过程中时，向其地址发送一条构造的错误消息就能触发该漏洞，无需密码或已有机器访问权限。  
  
该漏洞造成的简单结果是崩溃：启动挂起，机器无法启动。让代码运行则困难得多，可靠性也低得多。公告称，知道确切固件构建的攻击者可植入“伪造事件”，使固件运行攻击者自己的代码，因此利用程序必须针对特定构建来构建。当代码运行时，它在固件内执行，发生在Secure Boot和其他启动时检查验证任何东西之前，而这正是导致漏洞严重性的原因所在。以这种方式植入的代码只运行一次，自身不会在重启后存活。但这一次运行仍足以安装其他每次启动都会运行的代码。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXWM3IkPibmOaRJRPqRSiaEzGC9TxaGibHIm6SBMeNBjTusQX2uksT9xJ26XeVcpddrhQLDJkeFWgjYTUtsZO4uvS1WBqVj4wRXb0/640?wx_fmt=gif&from=appmsg "")  
  
**受影响群体**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUco65F5ibhWORoarRdNv6LH9LEHDibzJcDAV8yTaRrUPf2gZbNg6vA3icYxoW43tOl5xnOJN20bFy42xIIS1S9sBzFH0fJ8mG2Lg/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该漏洞存在于EDK II至edk2-stable202608（最新稳定版本，日期为8月21日）及包括该版本在内的版本中。EDK II是几乎所有UEFI固件厂商都基于其构建的参考代码，包括独立BIOS厂商AMI、Insyde和Phoenix，以及微软的加固分支Project Mu。  
  
本文作者通过追踪项目历史发现，有缺陷的复制循环自2010年11月添加的一次提交起就存在于代码中。PXE网络启动在EDK II标准构建中默认开启。该漏洞不影响只从本地磁盘启动的家庭或办公用户。它适用于机器通过网络启动的组织机构，主要是服务器和数据中心设备。  
  
目前尚不清楚有多少已出货产品带有易受攻击代码。英特尔公司表示其“当前受安全支持的商业产品”均未受影响，并且其“参考代码不包含受影响的EDK2功能”，不过OEM板卡厂商或系统厂商可能添加。Arm表示，该漏洞“不直接适用于Arm的产品组合”。两家公司自身的固件都基于EDK II。  
  
Linux发行版也提供该代码。Red Hat将Red Hat Enterprise Linux 8、9和10中的edk2包列为受影响且尚未修复，SUSE和Debian也以同样方式标记其对应软件包。AMD、Phoenix Technologies、AMI、Microsoft和Amazon在发本文发布前并未回应，TianoCore也未回应，因此其产品状态仍不清楚。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXicdXcVFdLPIXAO7cmLCK80EPxv6onMymDOBLDj7Df7Rs5cQlABlSAiaOXFYPRxUJvPD4qXZpr3hLnoXNJny8Y5cNJYqZ5VnosY/640?wx_fmt=gif&from=appmsg "")  
  
**修复方案**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVRgCudZ1OCIZU2RW6A2icRDRD1LiceZp1MQV1kEcDh1NHpIDw8NQmibEBBhCGI5Pawh9zmtIt3xEyqIdbs6DKPmxctZNd5ddscgY/640?wx_fmt=gif&from=appmsg "")  
  
  
  
修复方案已合并进EDK II源代码。它重写了有缺陷的复制逻辑，使超大消息在504字节处被截断而不是溢出，并于10月15日进入主开发分支。  
  
但目前尚无EDK II稳定版本包含该修复。本文作者于10月28日检查了项目的发布标签，发现最新的edk2-stable202608早于该修复，公告仍将修复版本列为“当前未修复”。微软的Project Mu已将修复拉入其发布分支，并于10月26日生成带标签构建。  
  
即使修复已在源代码中公开，服务器运营者也要等到其固件厂商发布基于该修复构建的BIOS或UEFI更新后才能安装。固件安全公司Eclypsium的首席研究员Jesse Michael向新闻媒体Ars Technica表示，这一步至少很直接：与BootHole和BlackLotus等Secure Boot绕过不同，该漏洞“可以通过基本代码修复来解决”，不需要证书或密钥吊销。公告并未列出应变措施。由于该漏洞只能在网络启动期间触达，不需要PXE的运营者可以禁用它以减少暴露。Ars还建议监控启动时网络流量和日志，以寻找攻击尝试迹象。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfX1UsncjmO4GHw1mSiatLHbO69zRwVaF96UMBL8ACiaIwnpHuhuFyttAwLD0YvRNwLwqfJxAjI02dK1n4tbZX4pr5ziacyvbr5tUo/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞披露**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUzRVlF79VHagawcdXSiabfExkJYT0KyztyoX1nm6q3Gx1StUZmZRTtEBwUe7ovwCagd3vDsw21sjjUgDrqCBR9HgRnEHS7vnJk/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该漏洞由Anthropic公司的研究员Alex Matrosov和谷歌公司的Dan Holmes-Mitra发现。原计划于2027年1月进行协调披露。但在9月28日一份由AMI工程师签署的EDK II拉取请求公开以CVE编号描述了该漏洞，并包含修复方案。发现者看到该帖子后，TianoCore于10月22日发布公告，比计划日期提前数周。媒体Ars Technica指出，补丁公开后，熟练攻击者可研究它以确定漏洞如何被触发，并构建可用的利用程序。  
  
目前尚无迹象表明该漏洞已被用于攻击。截至10月27日更新，该漏洞未被列入CISA的已知被利用漏洞目录，也没有收到已遭利用的报告。Ars Technica在向多位固件研究者提问后报道称，该漏洞本身也不是进入网络的方式。现实攻击者是内部人员，或已在本地网络获得低权限立足点并希望进一步深入的人员。  
  
Metasploit创建者、安全公司Dragos的首席研究官HD Moore评论称：“意图入侵附近端点或服务器的攻击者，可以利用这个漏洞，在启动过程仍能访问敏感组件（如端点TPM）的时刻劫持程序流。在攻击者拥有短期网络访问的情况下，这可用于破坏磁盘加密（在盗窃场景中甚至更容易）。”  
  
该漏洞并非EDK II网络启动代码中的第一个严重漏洞。2024年1月，研究者披露了被称为 “PixieFail” 的EDK II同一区域的一组九个漏洞，它们也可在PXE启动期间触达。报道称，PixieFail之后启动的、旨在发现该代码中更多漏洞的模糊测试项目并未发现本文所述漏洞。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[UEFI新漏洞可用于在多家厂商主板上执行预引导攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524734&idx=2&sn=7ccd5eea40d0b6a7a1e2dc6c3f41c681&scene=21#wechat_redirect)  
  
  
[MSI UEFI 签名密钥遭泄漏 恐引发“灾难性”供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247516496&idx=2&sn=05ab156deeadfbc7ffedcd43bddc9323&scene=21#wechat_redirect)  
  
  
[华硕修复可禁用安全启动程序的UEFI漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247514814&idx=2&sn=d77760825e29cde63586a63c502a43c9&scene=21#wechat_redirect)  
  
  
[Check Point 紧急修复严重漏洞，可使远程未认证攻击者获得 root 权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527165&idx=1&sn=065c0d69dff53c546fce09eee69ae140&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/critical-16-year-old-uefi-flaw-can-run.html  
  
  
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
  
