#  Zoom Workplace 严重漏洞可导致未认证远程提权  
Ravie Lakshmanan
                    Ravie Lakshmanan  代码卫士   2026-11-12 08:50  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Zoom****发布软件更新，修复影响****Zoom Workplace for Windows****的两个漏洞，其中一个可能导致受影响系统提权后果。**  
  
CVE-2026-108186  
（  
CVSS   
评分：  
9.6  
），是不正确的输入验证漏洞，可能导致未经身份验证的用户通过网络访问进行提权。  
CVE-2026-108187  
（  
CVSS   
评分：  
8.1  
）是源验证漏洞，可能导致未经身份验证的用户通过网络访问进行提权。  
  
成功利用其中任一缺陷，都可能使远程攻击者在无需任何特殊权限或访问权的情况下，实现提升权限。  
  
CVE-2026-108187   
影响  
 7.1.5   
版本之前的  
 Zoom Workplace for Windows  
，而  
 CVE-2026-108186   
影响从  
 7.1.0   
起至  
 7.1.8   
之前的版本。  
  
今年  
 7   
月早些时候，  
Zoom   
还修复了另一个影响  
 Zoom Workplace for Windows   
的不正确输入验证漏洞（  
CVE-2026-53412  
，  
CVSS   
评分：  
9.8  
），它可能使未经身份验证的用户接管账户。  
  
尽管 Zoom 未提及这些漏洞已遭在野利用，但建议用户应用最新补丁以缓解潜在威胁。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Zoom：注意这个严重的账户接管漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526636&idx=1&sn=7a16b14bc318e77d8eaa96cd6c7f7025&scene=21#wechat_redirect)  
  
  
[黑客滥用Zoom远程控制特性盗取密币](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522834&idx=2&sn=45d71524957ea78c0fdafa0a1236e4bc&scene=21#wechat_redirect)  
  
  
[Zoom 修复 Whiteboard 中的XSS漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247515145&idx=2&sn=ddda6fbf6a4687d99b355b93d446357f&scene=21#wechat_redirect)  
  
  
[微软CVSS 满分 Azure AI Foundry 漏洞，可导致未授权提权](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527201&idx=2&sn=0aebf0bab0fe5a789b9ddb1f114173b3&scene=21#wechat_redirect)  
  
  
[cPanel：LiteSpeed Web Server 企业版严重漏洞可导致提权](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527155&idx=2&sn=731309593b43d749b4d952223f7fc491&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/11/critical-zoom-workplace-flaw-could.html  
  
  
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
  
