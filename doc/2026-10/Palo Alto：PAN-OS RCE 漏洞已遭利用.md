#  Palo Alto：PAN-OS RCE 漏洞已遭利用  
Lawrence Abrams
                    Lawrence Abrams  代码卫士   2026-10-15 06:54  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Palo Alto Networks****公司发布安全更新，修复了****PAN-OS RCE****高危****0day****漏洞****CVE-2026-0322****（****CVSS****评分****8.6****）。该漏洞已被用于攻击防火墙。**  
  
该漏洞影响  
 PAN-OS   
管理接口，可导致经过身份认证的且具有网络访问权限的攻击者以  
 root   
身份在易受攻击设备上执行代码。  
Palo Alto Networks   
公司在本周三发布安全公告提到，已发现数量有限的攻击活动。  
  
该漏洞影响  
PA  
系列和  
VM  
系列防火墙以及  
 Panorama   
虚拟设备和  
M  
系列设备。  
Palo Alto   
公司提到，  
Cloud NGFW   
和  
 Prisma Access   
不受影响。攻击者需要对  
PAN-OS   
管理接口拥有现有凭据和网络访问权限才能实施利用。因此，该公司建议仅限受信任内部  
IP  
地址访问该管理接口。  
  
该公司已为受影响的  
 PAN-OS 10.2  
、  
11.1  
、  
11.2  
、  
12.1  
和  
12.2  
版本发布修复方案，  
PAN-OS 11.2.16  
的修复方案预计在当地时间今天发布。运行老旧、不受支持版本的客户必须升级到受支持版本。  
  
Palo Alto   
公司表示，虽然尚不存在已知的应变措施，但订阅威胁防御服务的客户可启用签名，从而为已知的攻击模式提供受限的防护措施，“虽然可通过当前的威胁  
ID  
识别出已知的攻击模式，但可能存在当下无法检测到的变体。如果该  
CVE  
漏洞和暴露配置要求影响环境，则建议升级至不受影响版本。”  
  
该漏洞由该公司的高级首席产品安全研究员  
 Suraj Siddharudh   
内部发现。该公司还共享了可能在利用该漏洞的三个  
IP  
地址。该公司尚未透露利用开始的时间、攻击的幕后黑手以及攻击者攻陷防火墙还是通过防火墙访问内网。美国网络安全和基础设施安全局  
 (CISA)   
已将该漏洞纳入  
KEV  
必修清单，并要求联邦民事机构在  
10  
月  
17  
日前缓解该漏洞。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Palo Alto 提醒注意严重的 PAN-OS RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525932&idx=2&sn=a1f8acec7865ec3eec445777c4ad6251&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks User-ID 凭据代理漏洞以明文形式暴露密码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524006&idx=2&sn=7d834750d24e4c7125f06fe6c19d570e&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks 修复多个提权漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247523281&idx=2&sn=c4ed265127f77866bb9d4f397428f552&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks 修复PAN-OS 中的认证绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522232&idx=2&sn=3bc7a4466c3c33ff643ca604524fa401&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks 修复退市 Migration Tool中的高危漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522035&idx=2&sn=71c00b294647fadae4e56ffc500f1300&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/palo-alto-networks-warns-of-pan-os-rce-flaw-exploited-in-attacks/  
  
  
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
  
