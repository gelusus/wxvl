#  F5紧急修复多个严重的 BIG-IP RCE 漏洞  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-11-23 07:53  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**美国网络安全公司****F5****提醒客户修复五个****BIG-IP****漏洞，它们可被用于触发拒绝服务或获得远程代码执行权限。这五个漏洞的编号为****CVE-2026-121805****、****CVE-2026-120292****、****CVE-2026-120285****、****CVE-2026-116953****和****CVE-2026-123637****，均为基于栈的缓冲区溢出缺陷，影响该公司的****BIG-IP****访问策略管理器（****APM****）集中访问管理代理解决方案以及****BIG-IP iRules****可编程脚本功能。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
所有这五个漏洞均被评为严重级别，可被未经身份验证的威胁行动者用于在配置存在漏洞的系统上终止流量管理微内核（  
TMM  
）并远程执行任意代码。  
  
F5   
公司还针对一个高危  
BIG-IP APM SAML  
漏洞（  
CVE-2026-126822  
）发布了工程热修复补丁。该漏洞源于整数下溢（回绕或环绕）缺陷，可被远程未经身份验证的威胁行动者滥用于拒绝服务攻击。该公司表示，  
 “TMM  
进程重启期间流量会中断。该漏洞允许远程未经身份验证的攻击者在  
BIG-IP  
系统上造成拒绝服务，并可能执行  
RCE  
。不存在控制平面暴露情况；只是数据平面问题。  
”  
  
F5  
公司表示，目前没有缓解措施，并建议安全团队安装周三发布的带外热修复补丁，修复受影响设备。  
  
互联网威胁监测机构  
Shadowserver  
目前追踪到超过  
177000  
个  
BIG-IP  
实例和超过  
14600  
个带有  
BIG-IP APM  
指纹的  
IP  
地址暴露在互联网上。然而，目前没有信息表明其中有多少是蜜罐或已修复这些漏洞。  
  
尽管该公司尚未将这些漏洞中的任何一个标记为已在攻击中被利用，但管理员应尽快将系统升级到最新版本，因为网络犯罪和国家支持的威胁组织经常在野外滥用  
F5  
漏洞。  
  
近年来，攻击者针对  
F5  
安全漏洞入侵企业网络、绘制内部服务器地图、劫持设备、部署数据擦除恶意软件，并从全球受害者处窃取敏感文件。自  
2021  
年  
11  
月以来，美国网络安全和基础设施安全局（  
CISA  
）已将七个  
F5  
漏洞标记为正在被积极利用，其中四个还被勒索软件团伙使用。今年  
9  
月，  
F5  
公司还修复了一个严重的  
BIG-IP APM  
漏洞（  
CVE-2026-94127  
），它已在零日远程代码执行攻击中被积极利用。一年前，  
F5  
还披露，受国家支持的黑客组织于  
2025  
年  
8  
月入侵其系统，窃取源代码和未披露的  
BIG-IP  
安全漏洞信息。  
  
F5  
是一家财富  
500  
强科技公司，为超过  
23000  
家客户提供网络安全、应用交付网络和其它服务，客户包括  
80%  
的财富全球  
500  
强企业和  
48  
家财富  
50  
强企业。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[F5 紧急修复多个 NGINX 高危漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527447&idx=1&sn=ef5ab66d9cc81c27ef07fed449bf016f&scene=21#wechat_redirect)  
  
  
[CISA要求三天内修复这个严重的 F5 BIG-IP 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525571&idx=1&sn=3596dc720ddc3cfdb3245a4b7597f249&scene=21#wechat_redirect)  
  
  
[F5 BIG-IP 源代码和 0day 漏洞数据遭泄露，补丁已发布](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524194&idx=1&sn=ab0b13f163b9168171ef5effd17379be&scene=21#wechat_redirect)  
  
  
[F5 BIG-IP 修复高危提权漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247521146&idx=1&sn=9082baa037a6e59fedb1cd685658ae1c&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/f5-releases-emergency-patches-for-critical-big-ip-rce-flaws/  
  
  
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
  
