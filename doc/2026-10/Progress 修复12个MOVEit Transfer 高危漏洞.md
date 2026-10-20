#  Progress 修复12个MOVEit Transfer 高危漏洞  
Do Son
                    Do Son  代码卫士   2026-10-20 08:08  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Progress Software****公司于****2026****年****10****月****14****日披露了****MOVEit Transfer****中的****12****个高危漏洞。这些漏洞可在托管文件传输环境中导致身份验证绕过、****SQL****注入、跨站脚本和本地权限提升后果。因此，系统管理员必须立即升级，防止未经授权的系统访问。**  
  
MOVEit Transfer   
是一款企业托管文件传输  
 (MFT)   
解决方案，供全球数千家组织用来传输敏感数据。虽然公告中未说明确切的受影响安装数量，但  
 MFT   
服务器始终是勒索软件和数据勒索团伙的主要目标。  
  
该安全公告提到，成功利用这些  
 MOVEit Transfer   
漏洞可使攻击者绕过登录控制、提升权限或操纵数据库查询。目前尚无公开的概念验证，也未确认存在在野积极利用。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVOGdjUnEkpic9u9kPtVz4dibeRtbsF6u2NaNywPzWcP65lJrIjcqFctNy2hv1M4E9etm8MhFBGSnZUBQgdC9aLvzR7lpfkmj540/640?wx_fmt=gif&from=appmsg "")  
  
**攻击原理**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfUT7q8vmu3ng5gjBt2IsnaVKrkAMGX7ofduuuAhY4ACyXPiaNLwI8aRfP9g1LNyN46bV1noyy3APyAMsseCD5bbFQOvGtk2MEcA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
这  
12  
个缺陷涉及多个核心组件，包括  
 LDAP  
、  
OIDC  
、  
SFTP  
、  
Ad Hoc  
、自定义报表和客户端证书模块。  
  
**（****1****）身份验证绕过与访问控制缺陷**  
  
首先，有两个漏洞的  
 CVSS v3.1   
基础评分最高，为  
 8.8  
。  
CVE-2026-89067 (CWE-288)   
允许低权限远程攻击者使用替代路径或通道绕过  
 Secure Messaging   
中的身份验证。同时，  
CVE-2026-87093  (CWE-732)   
源于关键服务目录上的权限分配不当，允许本地用户提升权限。  
  
其次，  
CVE-2026-89068  
（  
CVSS 8.1  
）在  
 LDAP   
模块中遗漏了关键的身份验证步骤（  
CWE-304  
），允许未经身份验证的网络攻击者入侵服务器。此外，  
CVE-2026-89070  
（  
CVSS 7.5  
）允许通过  
 OIDC   
模块中的捕获重放攻击绕过身份验证。在  
 SFTP   
模块中，  
CVE-2026-89077  
（  
CVSS 7.5  
）引入了不正确的身份验证算法实现，而  
 CVE-2026-89072  
（  
CVSS 7.5  
）导致客户端证书模块中的授权不正确。  
  
**（****2****）注入与会话管理缺陷**  
  
其他几个缺陷针对输入验证和数据存储。具体而言，  
CVE-2026-89069  
（  
CVSS 8.0  
）是  
 Ad Hoc   
模块中的跨站脚本（  
XSS  
）缺陷。类似地，  
CVE-2026-89071  
（  
CVSS 7.5  
）可导致  
 LDAP   
注入，  
CVE-2026-89076  
（  
CVSS 7.2  
）允许高权限攻击者在自定义报表模块中执行  
 SQL   
注入攻击。最后，  
Progress   
修复了会话过期不足（  
CVE-2026-89073  
）、  
Unicode   
编码处理不当（  
CVE-2026-89074  
）以及敏感数据存储不安全（  
CVE-2026-89075  
）的问题。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfX3vj9u0ed9wPAvXTDch8T13JUwT2dibSEwX2DXm4PibONe9uEIwQ36mmRsZIlxTFnrzrcoTlq7ibqok63JOPcVKAtPvKY2NrrMkM/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVKLmJpib5sKVDya0x1iaibtz2Y2ytZUmyJheibvGyh9EiczvRI3M1giaagowTW48EVzLrmKHLXkDk1fVuuU8MibDwRGvEficXxibp0bajU/640?wx_fmt=gif&from=appmsg "")  
  
  
  
所有  
12  
个  
 MOVEit Transfer   
漏洞影响  
 2025.1.6   
之前的版本，以及  
 2026.0.0   
至  
 2026.0.4   
之前的版本。  
2025.1.6   
和  
 2026.0.4   
版本不受影响。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUChvjrgnMicnoeErAJZL5fiaeW11KIibOJLqxmj8mFbevT75LpUyRqgGF8ib3y8bLKEPgElJ3UiahP6u3Vta91OP57L7NoeL4k1tpc/640?wx_fmt=gif&from=appmsg "")  
  
**补丁与缓解步骤**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUNz9SGXKNLWjZ7lNibLQRLIicawGh26yRlgoYlRMEgy8wDtxR7VXMia8nicMQPFvAiaibiaMREIUaDMnUpNxtxV6miaxc4ibA935VyhmdM/640?wx_fmt=gif&from=appmsg "")  
  
  
  
Progress Software   
在  
 MOVEit Transfer 2025.1.6   
和  
 2026.0.4   
版本中修复了全部  
12  
个漏洞。管理员应立即从  
 Progress Community   
门户下载并应用相应的修复版本。此外，安全团队应审计本地服务器目录权限，并仅限可信网络访问管理控制台。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Progress：存在可信外部威胁，ShareFile 管理员需立即关闭服务器](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526606&idx=2&sn=92be8299dea05a32e51baa1cad82b509&scene=21#wechat_redirect)  
  
  
[Progress 提醒注意严重的 MOVEit 自动化认证绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525932&idx=1&sn=cf1d9580501b94ce494712fa2e09e95c&scene=21#wechat_redirect)  
  
  
[Progress ShareFile 漏洞可用于发动预认证 RCE 攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525643&idx=2&sn=46f21b64114594cdb5db7473129e09a8&scene=21#wechat_redirect)  
  
  
[MOVEit Transfer 软件中存在高危的认证不当漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247519884&idx=1&sn=6407cbe53c48d873acacc653dceafd9b&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://securityonline.info/progress-patches-moveit-transfer-vulnerabilities/  
  
  
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
  
