#  Barracuda 修复两个漏洞，可绕过授权和获取 root 权限  
Ravie Lakshmanan
                    Ravie Lakshmanan  代码卫士   2026-11-05 09:32  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Barracuda****披露称，如组合利用两个漏洞，可能导致攻击者绕过授权控制并获得对易受攻击设备的****root****访问权限。**  
  
这两个漏洞分别是预认证授权绕过漏洞和命令注入漏洞：CVE-2026-116606（CVSS评分9.3）和CVE-2026-111560（CVSS评分8.6）。  
  
CVE-2026-116606仅影响Email Security Gateway，而CVE-2026-111560影响以下Barracuda产品——  
  
- Email Security Gateway  
  
- Load Balancer Application Delivery Controller  
  
- Message Archiver  
  
- WAF Control Center  
  
- Web Application Firewall  
  
- Web Security Gateway  
  
  
  
Barracuda表示：“成功利用需要攻击者能够访问受影响产品的管理界面。如组合利用，可能允许攻击者绕过授权控制并获得设备的root访问权限。”  
  
该表示，如果管理界面未暴露在互联网上，则受该攻击路径影响较小，并敦促客户遵循以下步骤保护自身安全：  
  
- 将管理界面的访问限制在可信内部网络  
  
- 使用适当的网络访问控制来限制暴露面  
  
  
  
该公司提到，产品客户应确认其系统运行的是最新固件版本，并已获得所有安全补丁。如设备上的固件落后多个版本，则可能需要执行多次更新。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[黑客早在2022年10月就利用0day 攻击 Barracuda ESG 设备](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247516628&idx=2&sn=eb96b9d65915efbbd1f1b4c2d593f8ee&scene=21#wechat_redirect)  
  
  
[速修复Next.js中严重的授权绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522568&idx=1&sn=6c208cae9bd5b7c538ed45a6a6cd2308&scene=21#wechat_redirect)  
  
  
[Check Point 紧急修复严重漏洞，可使远程未认证攻击者获得 root 权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527165&idx=1&sn=065c0d69dff53c546fce09eee69ae140&scene=21#wechat_redirect)  
  
  
[SolarWinds 修复15个严重漏洞，可使攻击者获得 root 权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526708&idx=1&sn=88286beec0ba9cb55a929f29a593a557&scene=21#wechat_redirect)  
  
  
[Bad Epoll：Linux 内核新漏洞，导致低权限用户获得 root 权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526510&idx=1&sn=5d8dd588bd5df97068ed04c486fd6458&scene=21#wechat_redirect)  
  
  
[Dirty Frag 可被用于获得Linux所有主流发行版本的 root 权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525959&idx=2&sn=da57a54e174c45c1b0df30db368b79b8&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/11/barracuda-flaws-could-be-chained-to.html  
  
  
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
  
