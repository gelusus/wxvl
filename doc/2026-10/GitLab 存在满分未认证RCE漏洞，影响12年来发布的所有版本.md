#  GitLab 存在满分未认证RCE漏洞，影响12年来发布的所有版本  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-10-22 07:38  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**10****月****21****日****GitLab****修复了一个允许攻击者无需登录就在自管理服务器上运行代码的漏洞（****CVE-2026-110264****，****CVSS****满分****10****），该漏洞影响自****2014****年发布的****7.0****以来的所有版本。**  
  
由于9天前即10月12日的更新只覆盖19.2及更高版本，因此即使管理员之前已经打过补丁，这次修复仍然重要，因为本次发布的是高一个补丁版本的修复方案，因此当时更新过的实例仍然在受影响范围内。  
  
GitLab表示，在某些条件下，由于编译特殊构造的正则表达式时发生整数溢出，该漏洞可能允许未认证用户在服务器上运行任意代码。不过，GitLab尚未说明这些条件是什么。GitLab 表示并未发现漏洞遭利用的迹象，截至10月22日，该漏洞也未列入美国CISA已知被利用漏洞目录。  
  
仅自管理安装需要采取行动。GitLab.com已经运行修复版本，使用内部版本19.3.5-internal0和19.4.2-internal0或更高版本的GitLab Dedicated租户也已获得修复方案。GitLab一次性修复了七个版本。  
  
受影响版本以及修复版本如下：  
  
- 19.5.1 之前的19.5版本，升级至19.5.1  
  
- 19.4.3 之前的19.4版本，升级至19.4.3  
  
- 19.3.6之前的19.3版本，升级至 19.3.6  
  
- 19.2.9 之前的19.2版本，升级至19.2.9  
  
- 19.1.9之前的19.1版本，升级至 19.1.9  
  
- 19.0.10之前的19.0版本，升级至 19.0.10  
  
- 7.0至18.11，且低于18.11.13的版本，升级至18.11.13  
  
  
  
其中四个版本超出了GitLab通常提供补丁的范围。该公司策略覆盖当前版本和前两个月度版本，即截至今天的版本19.5、19.4和19.3。GitLab表示在标准发布版本之外，将此次修复回移植到18.11.13、19.0.10、19.1.9和19.2.9，而通常这样做适用于严重问题。  
  
该漏洞影响范围如此久远的原因在于GitLab的失败之处。GitLab将Ruby内置引擎Onigmo 的正则表达式编译器打包在自己的软件包中，并在自己的构建中打补丁。该引擎中的弱点存在于GitLab交付它的所有地方，也就是自7.0以来的每个版本。  
  
GitLab还发布了两条检测规则，帮助自管理管理员查找利用尝试，名称分别为“Onigmo native worker crash”和“Org mode render error burst”。它们属于该公司开源TLDR框架的一部分，该框架以Sigma格式针对GitLab自身应用日志编写每条检测规则。  
  
GitLab尚未说明攻击者到达编译器需要哪些前置条件，因此管理员还不能通过配置排除自身风险；修复是唯一确定的保护措施，另外GitLab的公告也未提供临时缓解措施。  
  
这是昵称为joaxcar的研究人员在一个月内报告的第四个GitLab正则表达式引擎代码执行漏洞，也是第一个影响19.2之前版本的漏洞。10月12日，GitLab修复了CVE-2026-97229，它也是正则表达式编译器中一个未认证的10.0溢出漏洞，影响19.2.8、19.3.5和19.4.2版本。9月23日，它又修复了两个漏洞，CVE-2026-93577和CVE-2026-89078，二者均评为9.9，但需要登录账户才能实施利用。前三个漏洞止于19.2版本；但本文所述漏洞则并非如此。  
  
GitLab 在10月21日的发布共修复11个安全问题，一个严重，其余为高危、中危或低危。其中另外两个涉及认证。CVE-2026-15723评为高危8.2，可能允许已登录用户接管另一个账户，但仅当管理员同时启用了SAML和LDAP自动链接时才会触发。  
  
GitLab的OAuth和OpenID Connect端点中存在一个缓存投毒漏洞CVE-2026-112212，评为高危7.5。在使用共享缓存的设置中，可能允许攻击者将某些登录流程重定向到他们控制的服务器。  
  
升级会带来提前规划的成本。19.4.3和19.3.6版本包含在升级期间运行的数据库变更，在单节点服务器上这意味着在变更完成期间会停机。GitLab表示，多节点设置可以使用零停机流程应用补丁而无需停机。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[GitLab 紧急修复严重漏洞，可在服务器上实现任意代码执行](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527348&idx=1&sn=e77de684fc69ef2772a6284af052c198&scene=21#wechat_redirect)  
  
  
[GitLab 邮件地址可用于供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527227&idx=2&sn=ebe8eb1c6273dd854494ce61cdcf3e57&scene=21#wechat_redirect)  
  
  
[GitLab：速修复这个满分路径遍历漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527116&idx=1&sn=a5bb9e6f5885a23ace296375f350354b&scene=21#wechat_redirect)  
  
  
[GitLab GraphQL 严重漏洞可使未认证攻击者删除公开项目](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526883&idx=1&sn=80604cef5784651f30751cec579a2172&scene=21#wechat_redirect)  
  
  
[GitLab 公开仓库暴露超过1.7万份机密信息](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524561&idx=2&sn=7ed0ade9d82089d6e02e5c90e36d90a4&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/critical-gitlab-flaw-cvss-100-could.html  
  
  
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
  
