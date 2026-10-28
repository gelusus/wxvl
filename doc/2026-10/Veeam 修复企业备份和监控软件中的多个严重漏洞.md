#  Veeam 修复企业备份和监控软件中的多个严重漏洞  
Do Son
                    Do Son  代码卫士   2026-10-28 06:50  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Veeam****于****2026****年****10****月****27****日发布两份安全公告，修复企业备份和监控软件中的五个****Veeam****漏洞（****1****个严重、****2****个高危、****2****个中危）。其中最严重的漏洞为****CVE-2026-108142****（****CVSS v4.0****：****9.1****），可导致未经身份验证的攻击者绕过****CDP Coordinator Service****上的授权。系统管理员必须立即应用最新软件更新，保护备份基础设施。这些漏洞可能导致未经授权的服务访问、****MFA****绕过和任意文件写入，目前该公司并未发现这些漏洞的公开****PoC****或遭利用的迹象。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
Veeam Data Platform  
解决方案为全球超过  
55  
万家组织保护数据。虽然公告中未说明暴露的版本  
13  
的确切部署数量，但企业备份服务器仍然是勒索软件攻击者的首要目标。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWbqN7SB3g4cEugBuKbuASibhuFZicjwBk9nSzgI7gQjBcTBEepEpdFUhJoIwLB7SsJ6VLB313Z1vE3Fw6wGwkUicDPI7HcxvOicTM/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞简述**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWibUfUv1NK3ZnEGCuJibqkp1uiaQxO7BXr72sgicFAIzHcs26PIr3FPesXQzt5j5qryaRziaw8O997L2fMWSE9khg0MrINtVunjpIE/640?wx_fmt=gif&from=appmsg "")  
  
  
  
所有五个漏洞均通过  
HackerOne  
漏洞赏金平台报告，影响身份验证、文件恢复和本地代理权限。  
  
**（****1****）授权与****MFA****绕过**  
  
根据官方  
Veeam KB4948  
安全公告，严重漏洞  
CVE-2026-108142  
（  
CVSS 9.1  
）允许未经身份验证的攻击者在  
Windows  
和  
Veeam Software Appliance  
部署中绕过  
Continuous Data Protection  
（  
CDP  
）  
Coordinator Service  
的授权检查。同时，  
Veeam ONE KB4951  
安全公告详细说明了  
CVE-2026-107671  
（  
CVSS 8.5  
），这一高危漏洞允许拥有有效管理员凭据的攻击者在完成  
MFA  
验证步骤之前全局禁用多因素认证。  
  
**（****2****）权限提升与路径遍历**  
  
CVE-2026-105477  
（  
CVSS 7.3  
）允许在由  
Veeam Backup & Replication  
服务器管理的  
Veeam Agent for Linux  
中进行本地权限提升。  
CVE-2026-108140  
和  
CVE-2026-108141  
（均为  
CVSS 5.9  
）允许控制已备份  
NAS  
共享的经过身份验证的  
Restore Operator  
通过相对路径遍历向备份服务器写入任意文件。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfUcpscNY3X0fd9tJUd6XaFx3Peo5wYTWztdvW7EnibGJ8HTyrB0FSGgaQo7L2tcrSFoo7yicTCRpARYdqFr222NfywFA3IQegxzM/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWnDEqhCVDJu9ibaReriajpkuyE8nktKsZOgiatF6LI4EUpy2joCsCnWaPWLFqbjm0kO65qazjd1pvCKTSgJ6Zp8yd5rd3eoQuiblA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
这些  
Veeam  
漏洞仅影响版本  
13  
发行版；较旧的  
12.x  
和  
6.x  
分支不受影响。具体而言：  
  
- CVE-2026-108142  
、  
CVE-2026-108140  
和  
CVE-2026-108141  
影响  
Veeam Backup & Replication 13.1.1.18  
及更早的  
13.1  
版本，也影响  
13.0.2.29  
及更早的  
13.0  
版本。  
  
- CVE-2026-105477  
影响  
Veeam Agent for Linux 13.1.1.4  
及更早的  
13.1  
版本。  
  
- CVE-2026-107671  
影响  
Veeam ONE 13.1.0.7233  
及更早的  
13.1  
版本。  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUnR9tRnLIQ1TpPT5viajaJ6jcUIV4ibdMgI6MSWUyh9CvtnkibYX86tvKJfPqQqz0Bvbo1U1icMnQRpGUPaykn1ya7vebK2kFEDyc/640?wx_fmt=gif&from=appmsg "")  
  
**补丁与缓解步骤**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXV3ic5eiaHpR0aLZ74yLOdEcfnDPH40bzlYgAJJbibib2t5er5qLY0bFefDhsSLvWToAiaOxNNvVcGoQfBxKZDOCf9DK2n7znX4GTE/640?wx_fmt=gif&from=appmsg "")  
  
  
  
管理员必须将  
Veeam Backup & Replication  
升级至版本  
13.1.2  
（内部版本  
13.1.2.57  
）或  
13.0.3  
（内部版本  
13.0.3.63  
）。此外，应将  
Veeam Agent for Linux  
更新至  
13.1.2  
（内部版本  
13.1.2.11  
），还必须立即将  
Veeam ONE  
修补至  
13.1 P1  
（内部版本  
13.1.0.7940  
）。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Veeam 新漏洞导致Backup 服务器易受 RCE 攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526255&idx=2&sn=e65c7331cf40704380d18555dad266f7&scene=21#wechat_redirect)  
  
  
[Veeam 修复位于备份服务器上的多个 RCE 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525415&idx=2&sn=259129929dcb8d74602ea462298ce874&scene=21#wechat_redirect)  
  
  
[Veeam 修复备份服务器中的RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524822&idx=2&sn=bf38cc5485bf41c1eb57971488d1f356&scene=21#wechat_redirect)  
  
  
[HPE 修复八个 CVSS 满分RCE和认证绕过 AOS 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527512&idx=2&sn=9bebc29568e6fc01cdf9b66bbd3e3bd8&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://securityonline.info/veeam-vulnerabilities-backup-replication-veeam-one/  
  
  
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
  
