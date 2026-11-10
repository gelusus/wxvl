#  SonicWall：注意已遭利用的 SSL-VPN MFA 绕过漏洞  
Lawrence Abrams
                    Lawrence Abrams  代码卫士   2026-11-09 05:51  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**SonicWall****已发布安全更新，修复****SonicOS****防火墙中的七个漏洞，其中包括一个已在攻击中被利用的****SSL-VPN****身份验证绕过漏洞（****CVE-2026-19868****，****CVSS 7.1****）。**  
  
该漏洞可导致拥有备用身份验证方法有效凭据的攻击者绕过  
 SAML   
身份验证控制，可能包括  
 SAML   
强制实施的多因素身份验证（  
MFA  
）。该漏洞仅影响配置为使用  
 SAML   
身份验证的  
 SonicOS SSL-VPN   
服务。  
  
SonicWall   
公司在上周五发布的一份安全公告中提醒称，已调查了一份关于该漏洞在攻击中被利用的报告，并敦促客户安装最新的安全更新，降低相关风险。该漏洞由  
The Electronic Office   
公司的研究员  
 Ryan Harris   
发现并报送。  
  
虽然  
 SonicWall   
公司未分享有关所报告利用行为的细节，但美国网络安全和基础设施安全局  
 (CISA)   
已将  
 CVE-2026-19868   
添加到已知被利用漏洞  
 (KEV)   
目录中，确认该漏洞已遭在野利用。  
CISA   
要求联邦民用行政分支机构在  
 2026   
年  
 11   
月  
 20   
日前保护受影响系统。  
  
该公告还修复了  
 SonicOS   
中的另外六个漏洞，包括两个严重性评分为  
 8.8   
的管理界面漏洞，可允许未经身份验证的攻击者在特定条件下执行未经授权的操作。其余漏洞包括  
 SSL-VPN   
和  
 TACACS+   
服务中的预身份验证缓冲区溢出，可能导致拒绝服务，以及影响虚拟设备的弱密码哈希漏洞。这些漏洞影响  
 SonicOS Gen7   
版本  
 7.3.3-7015   
及更早版本和  
 Gen8   
版本  
 8.2.2-8015   
及更早版本。  
SonicWall   
已在版本  
 7.3.4-7013   
和  
 8.2.3-8015   
中修复了这些问题，并表示没有可用的应变措施。  
  
在该提醒发布之前，已出现一系列针对  
 SonicWall SMA1000   
安全远程访问设备的攻击。  
10   
月，安全研究人员检测到针对  
 CVE-2026-102255   
的利用尝试，该满分漏洞可导致未经身份验证的攻击者访问内部功能并执行未经授权的操作。在此之前，攻击者在  
9   
月的攻击链利用了两个  
 SMA1000 0day   
漏洞实现远程代码执行，  
7   
月则利用了另外两个  
 0day   
漏洞在易受攻击的设备上部署恶意软件。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[SonicWall 提醒注意将不再修复的 GMS 软件漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527692&idx=1&sn=a8348dd483d5beeb4f78c1ae81030651&scene=21#wechat_redirect)  
  
  
[SonicWall：注意SMA1000网关中的满分 SSRF 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527276&idx=1&sn=7776586a7fd71f15af5a7d30a2d0dc00&scene=21#wechat_redirect)  
  
  
[SonicWall 提醒注意已遭利用的 SMA1000 0day 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527014&idx=1&sn=2bf129add6d324acc41c93a2c5631a2d&scene=21#wechat_redirect)  
  
  
[SonicWall NetExtender 多个漏洞可用于以root身份写入任意文件](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526978&idx=2&sn=43af008fe7c471a0e023d7056f53e66d&scene=21#wechat_redirect)  
  
  
[SonicWall：立即修复已遭0day攻击利用的 SMA1000 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526630&idx=1&sn=3f9411edee6fd9805cd5ac7a7991a3c2&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/sonicwall-warns-of-new-ssl-vpn-mfa-bypass-flaw-exploited-in-attacks/  
  
  
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
  
