#  西门子：注意未修复的八个 SIMATIC S7-1500 Web Server 漏洞  
Do Son
                    Do Son  代码卫士   2026-10-14 08:33  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**2026****年****10****月****13****日，西门子披露了****S7-1500 PLC****网页服务器中的八个漏洞。其中三个被评为严重级别，****CVSS 4.0****评分为****9.4****。目前尚无补丁，因此西门子敦促客户立即采取缓解措施。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVDBv3sHiaSRiacA367VGgstJeqfabvic0c8IB78JofyLY0XrHRdjtEicCspsNtQ20eryBnxKyvc52bHhcicNBwvhnIQricWQT1xbLiaY/640?wx_fmt=gif&from=appmsg "")  
  
**值得关注的漏洞**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXBtbUKBkvrP3rcPLLWNnb4SRhbob5YGlGQBXJtC4eukKffOrWibc9gzgz91NUEodJ2nH8KGibzP5YhkpOLgrdIeRPibbeQ9blE7c/640?wx_fmt=gif&from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/t5z0xV2OYfXjcDYcTATmjBLggILkwtjOxU8vOVfRbhXO4HSnHPZs7YFK9NKzhJIUnGqTqTfA9Tib2pW8CyfttvFple9xbPG1Nic6j2Cd02BVY/640?wx_fmt=png&from=appmsg "")  
  
  
SIMATIC S7-1500控制器运行着许多行业的生产线。西门子表示，这些CPU服务于“全球的制造业、食品饮料和化工行业”。因此，被攻陷的控制器会话可能影响物理过程，而不仅仅是数据。  
  
根据西门子安全公告SSA-591731，这些缺陷“可能允许攻击者在授权用户的浏览器会话中执行恶意代码，并在该用户授权范围内执行PLC操作”。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUE3NZluWC8bqTJiaUjicSjOicmV4YZF1SF1nk4EIu3xicTH0z82y26BfCxoBianfnEGWnvZVj5G49XDibEDvYiaia7XUUW9Mdo20ncOFA/640?wx_fmt=gif&from=appmsg "")  
  
**攻击如何运作**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXwvYicxXU118UnGhXWZOcQIoaJSkt8U0ovxZCBvCbvwgGvYPjklGF8q115JYiclPN3MhRYPXTROWI6iaY7jCZian1QWxau7PSPvyE/640?wx_fmt=gif&from=appmsg "")  
  
  
  
所有八个漏洞都位于网页服务器的Trace页面。该页面在显示导入的跟踪文件之前，未能正确清理其中的数据，从而造成了跨站脚本风险。  
  
攻击者无法单独触发该漏洞，而是必须诱骗授权用户导入恶意跟踪文件。该用户还必须拥有“读取诊断”功能权限。如果诱骗成功，攻击者控制的JavaScript会在受害者浏览器中运行。由此，它可以“通过网页服务器在该用户授权范围内执行PLC操作”。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXGoXUytvR8fLdbbeNiaeMmvnicrFic9ebaA0NAiaiaK2oVI4icxiarFNdRHxHPzoLickr1yjaDEDeLn9uj8ku96288ubB1wPTl3cMFXDI/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfU7h9jndW2Q2Sd8ueXcsrnsBYLZV6L6qLVDs3ug9x5oQFnewzThITvsSYTQKRG6MtzxytuVWRZp7J7ZUgheFQGss4nhWGbkyuw/640?wx_fmt=gif&from=appmsg "")  
  
  
  
西门子并未发现这些漏洞遭在野利用，也并未确认这些漏洞的公开 PoC 利用代码。西门子列出了受影响产品的所有版本，每个产品都受所有八个CVE 漏洞影响。受影响的系列包括：  
  
- SIMATIC S7-1500 CPU，包括F、T、TF、MFP和ODK变体  
  
- 相关的ET 200SP和ET 200pro CPU，以及SIPLUS版本  
  
- SIMATIC Drive Controller CPU 1504D TF和CPU 1507D TF  
  
- SIMATIC ET 200SP Open Controller CPU 1515SP PC  
  
  
  
  
受影响的其它产品可参见西门子发布的安全公告。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWLEibLxfNw7Sr3akWXmefpp2KMZkC5JicDhBNgiblK8khRGHLby5r3we2lboYOh8iabTCG6bTlJdoZNZ7ia1wynMlHP4icMlCK2emz8/640?wx_fmt=gif&from=appmsg "")  
  
**补丁和缓解步骤**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVPHITRgKyPKJPxOozuLIJeJOY3vJO7Yc3CPlkgUhCluTDX7P95LhQ3WcJ4ZAqkyaZD2u9AQeD5IXE5OK34piamRa1OAXMicJicEA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
西门子在公告中指出：“西门子正在准备修复方案，并针对尚未提供修复方案的产品推荐具体缓解措施。”在修复方案发布之前，西门子建议采取以下步骤：  
  
- “如果受影响的系统不需要网页服务器，请禁用。”  
  
- 将80/tcp和443/tcp端口的访问限制为仅受信任的IP地址。  
  
- “仅上传受信任的跟踪文件。”  
  
- 确保只有合格人员才能访问网页服务器。  
  
  
  
操作人员还应将PLC断网，并将OT网络与办公系统隔离。最后，培训工程师谨慎对待来自未知来源的跟踪文件。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[西门子爱立信施耐德电气等：欧盟《网络安全弹性法案(CRA)》或破坏供应链](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247518086&idx=1&sn=f13ff428d3f9ed6d4ea180de7154309a&scene=21#wechat_redirect)  
  
  
[工控补丁星期二：西门子、施耐德电气等修复50个漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247517016&idx=2&sn=8edf5db583691ce70b6b4910729e5c5b&scene=21#wechat_redirect)  
  
  
[严重的西门子 RTU 漏洞可导致电网不稳定](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247516420&idx=1&sn=89d69a30e805ed5fabd9a6d3e57a5388&scene=21#wechat_redirect)  
  
  
[西门子许可管理器中存在两个严重漏洞，可用于攻击ICS网络](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247515512&idx=1&sn=7d39a4d7d874a7b1c041b7d631be5e96&scene=21#wechat_redirect)  
  
  
[CISA提醒注意西门子、通用数字和康泰克工控系统中的漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247515346&idx=2&sn=9c64d78059c7b3ee275ab9039c5b3544&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://securityonline.info/siemens-s7-1500-vulnerabilities/  
  
  
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
  
