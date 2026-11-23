#  SonicWall：因AI攻击风险，将不再修复 Gen 6 防火墙漏洞  
Ravie Lakshmanan
                    Ravie Lakshmanan  代码卫士   2026-11-23 07:53  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**SonicWall****公司敦促所有仍在使用第六代防火墙的客户立即淘汰并更换这些设备，停止将其暴露在互联网边缘，并称此举是出于应对****AI****辅助攻击带来的安全风险。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
SonicWall  
公司表示：  
“  
目前攻击者利用自动化和  
AI  
，比以往更快地发现并利用不受支持的边缘设备。我们预计面向互联网的第六代防火墙将成为积极攻击的目标。请将任何仍在使用的第六代防火墙视为高风险暴露，并迁移到运行  
SonicOS 8  
的受支持的第八代防火墙。  
”  
  
运行  
SonicOS 6.x  
的受影响产品包括：  
  
- TZ350  
、  
TZ350W  
、  
TZ500  
、  
TZ500W  
、  
SOHO 250  
、  
SOHO 250W  
  
- NSA 4600  
、  
NSA 5600  
、  
NSA 6600  
  
- NSa 4650  
、  
NSa 5650  
、  
NSa 6650  
  
- TZ400  
、  
TZ400W  
、  
TZ600  
、  
TZ600P  
、  
NSA 3600  
  
- NSa 3650  
  
- NSa 2650  
  
- SOHO  
、  
SOHO W  
、  
TZ300P  
、  
NSa 9250  
、  
NSa 9450  
、  
NSa 9650  
  
- NSv 10  
至  
NSv 1600  
（第六代虚拟型号）  
  
- SuperMassive 9200  
、  
9400  
、  
9600  
、  
9800  
  
- NSsp 12800  
、  
NSsp 12400  
  
- TZ300  
、  
TZ300W  
  
- NSA 2600  
  
  
  
近年来，威胁行动者越来越多地针对边缘设备作为获取初始访问权限的切入点，与此同时，从漏洞披露到确认利用之间的窗口已大幅缩短，部分原因是威胁行为者利用自动化和  
AI  
扫描互联网上的不受支持设备、分析补丁和漏洞披露公告，并更快地构建可用的漏洞利用程序。  
  
根据  
Google Mandiant  
于  
2026  
年  
3  
月发布的  
M-Trends  
报告，漏洞利用的平均时间已转为负值，预计为  
-7  
天，这意味着利用行为平均在补丁可用之前就已开始，而  
2018  
年这一数字为  
63  
天，  
2024  
年为  
-1  
天。  
  
谷歌表示：  
“  
这一趋势还因威胁行动者对安全设备和网络基础设施日益增长的兴趣而加剧。由于这些资产对数据入口和出口点提供关键检查，因此为获得访问权限的人员提供了重要的可见性和权限。  
”  
  
Cisco Talos  
发布  
2025  
年度回顾报告指出，当年发现的  
CVE  
中有  
23%  
直接影响网络设备，如  
VPN  
设备、下一代防火墙（  
NGFW  
）、负载均衡器、路由器等。  
Talos  
表示：  
“  
由于这些系统位于企业环境的外围，一旦被攻陷可能导致直接访问关键网络。这与  
CISA  
已知被利用漏洞（  
KEV  
）目录中的现实趋势一致，其中利用行为始终偏向边缘设备，因为它们面向互联网、补丁更新往往滞后，并且一旦被攻陷就能提供直接的作战杠杆。  
”  
  
由于第六代产品不再受到活跃支持，也不再接收补丁（即使是严重漏洞），建议用户立即按照以下步骤操作：  
  
- 清点生产环境中的每一台第六代防火墙并将其淘汰。  
  
- 优先处理暴露在互联网上或提供远程访问的设备。  
  
- 将其替换为运行  
SonicOS 8  
的第八代防火墙。  
  
- 联系授权  
SonicWall  
合作伙伴规划迁移。  
  
  
  
在防火墙更换之前，建议采取以下措施以尽量减少暴露：  
  
- 禁用  
WAN  
管理（  
WAN  
接口上的  
HTTPS  
、  
SSH  
和  
SNMP  
）。  
  
- 禁用  
SSL VPN  
，或将其限制为已知源地址。  
  
- 对所有剩余的远程访问和管理账户强制执行多因素认证，并重置本地密码。  
  
- 禁用未使用的服务和接口。  
  
- 密切监控日志，留意异常登录、配置更改和流量。  
  
  
  
SonicWall  
公司表示：  
“  
不受支持的防火墙每在线一天，风险就增加一分。一次入侵就能触达一切。被攻陷的外围防火墙为攻击者提供了凭据窃取、横向移动和部署勒索软件的立足点。  
”  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[SonicWall：注意已遭利用的 SSL-VPN MFA 绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527704&idx=1&sn=b5908433e9c0092a8355eb0aef85977f&scene=21#wechat_redirect)  
  
  
[SonicWall 提醒注意将不再修复的 GMS 软件漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527692&idx=1&sn=a8348dd483d5beeb4f78c1ae81030651&scene=21#wechat_redirect)  
  
  
[SonicWall：注意SMA1000网关中的满分 SSRF 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527276&idx=1&sn=7776586a7fd71f15af5a7d30a2d0dc00&scene=21#wechat_redirect)  
  
  
[SonicWall 提醒注意已遭利用的 SMA1000 0day 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527014&idx=1&sn=2bf129add6d324acc41c93a2c5631a2d&scene=21#wechat_redirect)  
  
  
[SonicWall NetExtender 多个漏洞可用于以root身份写入任意文件](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526978&idx=2&sn=43af008fe7c471a0e023d7056f53e66d&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/11/sonicwall-warns-gen-6-firewalls-wont.html  
  
  
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
  
