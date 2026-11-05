#  SonicWall 提醒注意将不再修复的 GMS 软件漏洞  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-11-05 09:32  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**SonicWall****提醒客户称，两个漏洞（****CVE-2026-109593****和****CVE-2026-109594****）影响已停止支持的全球管理系统（****GMS****），但不会发布补丁。**  
  
全球管理系统  
 (GMS)   
可帮助  
IT  
管理员通过单一界面部署和管理数千台  
SonicWall  
安全设备，但  
SonicWall  
已正式用  
Network Security Manager (NSM)   
管理平台替代该产品。这两个漏洞由  
Pentest-Tools  
安全研究人员发现并报告，影响运行  
SonicWall GMS 9.5.2  
及更早版本的系统。  
  
CVE-2026-109593  
可使具有网络访问权限的未认证攻击者通过读取内部文件窃取敏感信息，影响运行  
GMS 9.5.2  
及更早版本的易受攻击实例。具有高权限的威胁行动者可利用  
CVE-2026-109594  
通过低复杂度  
SQL  
注入攻击访问或修改  
GMS  
数据库中存储的信息。  
  
本周二，  
SonicWall  
敦促客户升级到  
NMS  
，因为受影响系统已达到生命周期终止，且  
SonicWall  
不会发布安全补丁。  
SonicWall  
周二警告称：  
“  
受影响产品已达到生命周期终止。因此，不会针对这些漏洞发布修复措施或安全更新。使用该产品的组织机构应了解相关安全风险，并强烈建议升级到  
SonicWall NSM  
。  
”  
  
攻击者经常在野利用  
SonicWall  
漏洞，自今年年初以来已在零日攻击中利用了多个漏洞。  
2  
月，  
SonicWall  
要求客户修复两个  
SMA1000 0day  
漏洞（  
CVE-2026-15409  
和  
CVE-2026-15410  
），并在  
7  
月要求修复另外两个（  
CVE-2026-15409  
和  
CVE-2026-15410  
）。它还修复了另外三个已遭  
 0day   
状态利用的  
SMA1000  
漏洞（  
CVE-2026-83548  
、  
CVE-2026-83549  
和  
CVE-2026-102255  
。过去四年，美国网络安全和基础设施安全局  
 (CISA)   
已将  
19  
个  
SonicWall  
漏洞纳入已遭利用  
KEV  
列表，其中  
13  
个还被勒索软件团伙滥用。  
  
SonicWall  
通过由超过  
17000  
家合作伙伴组成的网络，为  
215  
个国家和地区超过  
50  
万客户提供服务，包括中小企业、大型企业和政府机构。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[SonicWall：注意SMA1000网关中的满分 SSRF 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527276&idx=1&sn=7776586a7fd71f15af5a7d30a2d0dc00&scene=21#wechat_redirect)  
  
  
[SonicWall 提醒注意已遭利用的 SMA1000 0day 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527014&idx=1&sn=2bf129add6d324acc41c93a2c5631a2d&scene=21#wechat_redirect)  
  
  
[SonicWall NetExtender 多个漏洞可用于以root身份写入任意文件](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526978&idx=2&sn=43af008fe7c471a0e023d7056f53e66d&scene=21#wechat_redirect)  
  
  
[SonicWall：立即修复已遭0day攻击利用的 SMA1000 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526630&idx=1&sn=3f9411edee6fd9805cd5ac7a7991a3c2&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/sonicwall-warns-of-unpatched-flaws-in-end-of-life-gms-software/  
  
  
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
  
