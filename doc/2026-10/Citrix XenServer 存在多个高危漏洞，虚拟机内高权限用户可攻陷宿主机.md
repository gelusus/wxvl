#  Citrix XenServer 存在多个高危漏洞，虚拟机内高权限用户可攻陷宿主机  
Abinaya
                    Abinaya  代码卫士   2026-10-16 09:16  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Citrix****披露了四个****XenServer****漏洞，其中三个可能让虚拟机内拥有提升权限的恶意用户入侵或使底层主机崩溃。**  
  
10  
月  
13  
日，  
Citrix  
发布了涵盖  
 XenServer 8.4   
和  
 XenServer 9  
高危漏洞的公告  
 CTX697170  
，包括影响主机的漏洞  
CVE-2026-79610  
、  
CVE-2026-102700   
和  
 CVE-2026-102701  
。第四个问题  
 CVE-2026-79611   
的影响范围较窄，它可能允许客户虚拟机内的非特权用户使该客户虚拟机崩溃。  
  
利用这三个影响主机的漏洞需要客户虚拟机内拥有特权的用户实施，因此该威胁位于虚拟环境内部，而非面向互联网服务的未经身份验证攻击。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfV3tvbSUMwoyJiaZ39ZJoMGxqQjK32TBzbHT6rh4CF9ibuXVpSI1yVeGLOicv6ia6nozKiczbtF8zOkJASR32sDcUgh5jr27ibJpcveQ/640?wx_fmt=gif&from=appmsg "")  
  
**多个 Citrix XenServer 漏洞**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfW1fFCiafz3pyu78lbKj8lq6g2uvsyjPUSPWeu22UOibnqBU5NRzmH5FZZyLP0rBdYS2gh4QFZfpyDuHphy8YT53Bl3oxa1uwQIc/640?wx_fmt=gif&from=appmsg "")  
  
  
  
公告将这些漏洞归为可能导致主机被入侵或主机崩溃的问题，但并未分开说明这些漏洞各自的后果。在评估暴露情况时，这种区分很重要。仅影响客户虚拟机的崩溃影响的是攻击者所操作的虚拟机。主机级问题则会触及支持虚拟环境的系统，从而扩大潜在影响。然而，所提供的公告并未确定每个漏洞都允许代码执行，也未确定攻击者能够自动访问其他客户虚拟机。  
  
CVE-2026-102700   
仅影响  
 XenServer 8.4  
。其余三个漏洞影响  
 XenServer 8.4   
和  
 XenServer 9  
。因此，管理员需要检查已安装的版本，再决定哪些编号适用于系统。  
  
此前已有涉及客户机访问和主机稳定性的提醒。例如影响  
Citrix XenServer   
和  
 Hypervisor   
的漏洞允许恶意客户机管理员使主机崩溃或失去响应。该较早的问题涉及不同的编号和受影响版本，与本次更新不同。另有关于  
 XenServer VM Tools for Windows   
漏洞的报道，描述了可能提升攻击者在  
 Windows 客户机  
虚拟机内权限的缺陷。这些问题影响的是客户机驱动程序，而不是最新公告中描述的主机级影响。  
  
安全团队应以  
 CTX697170   
作为此次披露的参考，并在规划修复前查阅最新的供应商指南。  
Citrix   
在公告中建议用户及时应用已发布的安全更新。该公司所提供的  
 XenServer   
摘录没有指明修复版本、补丁包名称或重启要求，因此不应对这些细节进行假设。所提供的材料也未包含漏洞利用说明、技术根因细节，或确认存在主动利用的声明。对于评估漏洞修复紧迫性的读者来说，已确认的担忧是所述可能性，即拥有特权的客户机  
账户可能导致主机被入侵或中断，此外还有一个单独的、仅影响客户虚拟机的拒绝服务风险。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Citrix：这个严重的 NetScaler SAML 漏洞可导致 RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527286&idx=2&sn=3083d12c9537dcc1cdaf2f329457dc89&scene=21#wechat_redirect)  
  
  
[六天，研究员利用AI模型构建 Chrome 完整利用链，获赏$25万](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527392&idx=2&sn=324df0a8612a62b14cb09f554273fef6&scene=21#wechat_redirect)  
  
  
[Citrix NetScaler 存在两个严重漏洞，无需凭证即可绕过认证机制](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526935&idx=1&sn=07147e7ec3cbfb9ffe750a7853798e1e&scene=21#wechat_redirect)  
  
  
[Citrix 修复可导致文件读取和拒绝服务的多个漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526489&idx=1&sn=70f1d92bbf820f6f31eaeaed6636eb22&scene=21#wechat_redirect)  
  
  
[Citrix：尽快修复这两个 NetScaler 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525554&idx=2&sn=1cd600c5708dc44ab8e2421ef606e780&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/citrix-xenserver-vulnerabilities/  
  
  
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
  
