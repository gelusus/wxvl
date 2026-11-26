#  VMware vCenter 修复严重的认证绕过漏洞  
Ravie Lakshmanan
                    Ravie Lakshmanan  代码卫士   2026-11-25 06:27  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**博通修复了影响****VMware ESX****、****vCenter****、****Workstation****、****Fusion****和****Avi Load Balancer****中的****23****个漏洞，其中一个严重漏洞可滥用于认证绕过。该漏洞编号为****CVE-2026-123903****，****CVSS****评分****9.8****，位于****VMware Directory Service (“vmdir”)****中。**  
  
博通在今天发布的安全公告中提到，“拥有  
 vCenter   
网络访问权限的恶意人员可利用该漏洞绕过认证并获得对系统的未授权访问权限。”该漏洞由奇安信  
 sglab  
以及其它公司研究员分别独立发现并报送。  
  
另外，博通还修复了位于  
 vCenter   
中的四个高危漏洞，其中一个任意文件写漏洞（  
CVE-2026-123904  
，  
CVSS  
评分  
7.5  
）可在处理某些文档时在设备上实现代码执行。余下三个漏洞与  
 VMware Directory Service   
中的拒绝服务有关，编号为  
CVE-2026-123905  
、  
CVE-2026-123906  
和  
CVE-2026-123907  
（  
CVSS  
评分  
7.5  
）。  
  
这些漏洞已在如下版本中修复：  
  
- VMware Cloud Foundation and VMware vSphere Foundation 9.1.x.x –   
在  
9.1.2  
版本中修复  
  
- VMware Cloud Foundation and VMware vSphere Foundation 9.0.x.x –   
在  
9.0.2.0300  
版本中修复  
  
- VMware vCenter 8.0 –   
在  
 8.0 U3m  
版本中修复  
  
- VMware vCenter 7.0  
（如签订了扩展支持合同，则联系博通支持部门）  
  
- VMware Cloud Foundation 5.x –   
在  
5.2.5  
版本中修复  
  
- VMware Telco Cloud Platform –   
在  
 vCenter 8.0 U3m (TCI: 3.0, TCP: 3.0, 4.x, 5.0.x, 5.1.x, 5.2)   
版本中修复  
  
- VMware Telco Cloud Infrastructure –   
在  
 vCenter 8.0 U3m (TCI: 3.0, TCP: 3.0, 4.x, 5.0.x, 5.1.x, 5.2)   
版本中修复  
  
  
  
几天前，博通曾为影响  
 VMware Directory Service   
的另外一个高危漏洞  
CVE-2026-123890  
（  
CVSS  
评分  
9.8  
）发布安全更新，据悉该漏洞已遭利用。  
VMware   
提到，  
CVE-2026-123903  
是另外一个漏洞，与  
CVE-2026-123890  
不同。  
  
虽然博通并未提到今天新发布的漏洞是否已遭在野利用，但客户应尽快安全更新，获得最佳防护措施。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[VMware Workstation 和 Fusion 严重漏洞可导致 VM 管理员执行主机代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527035&idx=1&sn=413d9de04171f8d76780daba7016dad3&scene=21#wechat_redirect)  
  
  
[博通修复三个严重的VMware 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526757&idx=1&sn=c4b049923f35e7a191a5591945eaf80b&scene=21#wechat_redirect)  
  
  
[CISA：VMware Aria Operations RCE漏洞已遭利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525306&idx=2&sn=bd95e3c5ab9c19e0a8d50c704895e37c&scene=21#wechat_redirect)  
  
  
[Vmware 修复 Pwn2Own 柏林大赛上遭利用的四个 ESXi 0day漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247523602&idx=1&sn=3ec9fd59b332276a13eb21d88cdd9217&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/11/vmware-vcenter-hit-by-another-critical.html  
  
  
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
  
