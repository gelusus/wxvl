#  Citrix：这个严重的 NetScaler SAML 漏洞可导致 RCE  
Do Son
                    Do Son  代码卫士   2026-10-09 07:21  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Citrix****发布了一份针对****NetScaler ADC****和****NetScaler Gateway****中内存溢出漏洞****CVE-2026-107406****（****CVSS v4.0****评分****9.5****）的严重公告。该漏洞在配置用于****SAML****的设备上可能导致远程代码执行或拒绝服务。****Citrix****敦促客户尽快升级。**  
  
  
NetScaler  
设备位于网络边缘，为许多组织机构处理远程访问，在过去的多轮攻击活动中也经常成为目标。根据  
Citrix  
的客户指南，  
Citrix  
目前  
“  
不清楚该漏洞存在任何未被缓解的利用  
”  
。  
  
Citrix  
将该漏洞描述为内存溢出。它仅在  
“  
特定配置条件  
”  
下适用，即当设备充当  
SAML  
服务提供者或  
SAML  
身份提供者时。  
Citrix  
未分享更多技术细节。不过，版本列表显示了一种分化：在最新受影响的版本中，仅  
IdP  
角色暴露；较旧版本在任一角色下都存在风险；未配置  
SAML  
的设备完全不受影响。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfV01hSXP4OpNejvJEOrFF9oJKQVwSkt44fCZTG8wuxbDxIHzSg9rbABJrIAqkuyR6aNLdhQtqyLSUNHWiaUydPYbO3KQSPxKwxw/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfUJAh5PjEZeyziaHfZP9JA3xqwpMgjTMgCUqD0Ih4L2nRIg0hct5pS57x6jUhPAdyXkGKGiaX5K1Z70WiakXiayYQcg3WPfSAmbSdM/640?wx_fmt=gif&from=appmsg "")  
  
  
  
暴露情况取决于版本和  
SAML  
角色：  
  
- 仅  
SAML IdP  
：  
14.1-73.37  
至  
14.1-73.41  
，以及  
13.1-64.23  
至  
13.1-64.28  
，加上匹配的  
FIPS  
和  
NDcPP  
版本；  
  
- SAML SP  
或  
IdP  
：早于  
14.1-73.37  
和早于  
13.1-64.23  
的版本，加上匹配的  
FIPS  
和  
NDcPP  
版本；  
  
- 使用  
NetScaler  
的  
Secure Private Access  
混合部署也受到影响。  
Citrix  
管理的云服务和  
Adaptive Authentication  
会自动更新，不受影响。  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWL5KdVaeo7JPNj6ue5K4VE8JhAAhewuRuoyYYk690WichVRN50yjXrzQLbWH3wWZF03tdKApBXl60oBf8ia4LcdH6ymNwXa6r1c/640?wx_fmt=gif&from=appmsg "")  
  
**补丁和缓解步骤**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUNmLzT3eqnFvCic59aXfniakYq0xibLoUUrPdTItNumkrV9RHj4riaXH8yBdED9gck24xQtF6gksksJN9x8uTcSjLVeLYF291gFI4/640?wx_fmt=gif&from=appmsg "")  
  
  
  
首先，检查  
Citrix NetScaler  
漏洞是否适用。在配置中查找  
“add authentication samlAction”  
（  
SP  
）或  
“add authentication samlIdPProfile”  
（  
IdP  
）。如果其中任何一项出现在受影响的版本上，应升级到  
Citrix  
安全公告中列出的已修复版本。鉴于  
NetScaler  
的历史，应将此  
Citrix NetScaler  
漏洞视为紧急事项。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Citrix 两个 NetScaler RCE 0day 已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527239&idx=1&sn=3b030d59dfddeecbee84c4bc01d53401&scene=21#wechat_redirect)  
  
  
[Citrix NetScaler 存在两个严重漏洞，无需凭证即可绕过认证机制](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526935&idx=1&sn=07147e7ec3cbfb9ffe750a7853798e1e&scene=21#wechat_redirect)  
  
  
[Citrix 修复可导致文件读取和拒绝服务的多个漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526489&idx=1&sn=70f1d92bbf820f6f31eaeaed6636eb22&scene=21#wechat_redirect)  
  
  
[Citrix：尽快修复这两个 NetScaler 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525554&idx=2&sn=1cd600c5708dc44ab8e2421ef606e780&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://securityonline.info/citrix-netscaler-vulnerability-cve-2026-107406/  
  
  
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
  
