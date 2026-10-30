#  思科提醒注意已遭在野利用的 ASA 高危漏洞  
Do Son
                    Do Son  代码卫士   2026-10-29 22:59  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**思科披露了一个已遭在野利用的****Cisco Secure Firewall****高危漏洞（****CVE-2026-76418****，****CVSS v3****评分****8.6****）。该漏洞影响****Cisco Secure Firewall Adaptive Security Appliance****（****ASA****）和****Threat Defense****（****FTD****）软件，可导致未认证攻击者通过****SSL VPN****服务触发设备意外重载。因此，网络管理员必须立即应用最新软件更新或配置缓解措施。**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUBdOetsb9iaoNR8CVNJnEN1UVKwIxTmiahVvKGdoEEV7rLhqITbajhg3Ylaz98cibQsI7vYxJcO9AtP5llONGcROT9Zh5ewg3kKs/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞简介**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXFVfxgPzhueQrTe6cJvpDRgRibm0ZxJiaJQNuHSx8NHBFP0VyOdaReQvcgbFCb3dMMjgX3qonKorPSsLVUbuVFLSG7ibswdyR5xw/640?wx_fmt=gif&from=appmsg "")  
  
  
  
思科  
ASA  
和  
FTD  
设备是全球数万家企业网络的核心边界防火墙和  
VPN  
网关。不过，该公告未说明受影响的精确部署数量。该漏洞是由研究员  
Eric Fernald  
在处理思科  
 TAC  
支持案例时发现的。  
  
思科在官方安全公告中提到，在  
10  
月了解到该漏洞已遭活跃利用，尽管尚无公开的概念验证利用代码发布，但在野活跃利用威胁企业远程访问的可用性。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXWpKgFFz9Iow980kv7kL0zngc8SyMFgibdBuibUicrkVWaQAqWrfFaBjujUcoM9Q8LoYF3KuXotmsaUcNNcO5sfPmjW8aus9Bay0/640?wx_fmt=gif&from=appmsg "")  
  
**攻击如何运作**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVgrTWqIrAPv5nBP8MSNPkyuHv6aVicIOsxh2rgBWGtmacMT3Z1nqLhiaNbj9iaOoeBjQHjBG5nNTLNOcVaibbI2m0NJTTibDe4YEVA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该漏洞位于远程访问  
SSL VPN  
服务中，前提是隧道组上启用了辅助认证。思科在公告中表示：  
“  
该漏洞是由于在隧道组上配置辅助认证时出现逻辑错误。攻击者可通过在连接  
SSL  
远程访问  
VPN  
时发送构造请求来利用该漏洞。  
”  
  
**认证前设备重载**  
  
公告提到：“成功  
VPN  
登录并非触达受影响处理路径所必需。  
”  
因此，未认证远程攻击者可发送构造的连接请求，迫使防火墙重载。这会造成持续性拒绝服务（  
DoS  
）状况。如果至少一个接口上启用了  
webvpn  
关键字，设备就会暴露。此外，必须在任何隧道组或  
DefaultWEBVPNGroup  
上配置  
secondary-authentication-server-group  
。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVqC937ZKDW30zkFfwh0co0nKZU5GcaJicKyfajibKSCZ5Cm8lqX7uibZwCAgHzu6TTnWCGRRDSxUaicFDqXPPgKcxj20QgMKEy8Ag/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVkb2x1KZBJ4Zl6eSCJJT78xwyQXkaC9jwbMpjYl1jJZGjmibOP3TEdD9dcnKTia7EpMbtgSsmhHjFhLkfTHibyw37nicvosP3UzmA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该思科  
Secure Firewall  
漏洞影响  
Cisco ASA  
软件  
9.16  
至  
9.24  
版本。它也影响配置了  
SSL VPN  
辅助认证的  
Cisco FTD  
软件  
7.0  
至  
10.1  
版本。  
Cloud-delivered Firewall  
（  
CDFW  
）、  
cdFMC  
、  
Secure Access  
（  
CSA  
）和  
Secure FMC Software  
不受影响。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXjEyxhYHwCxeIXjlrXiczyKI30MFNjLzzM4aFx8qaIlKKqWAhRcxg5hhgL3F6uanfQTSkeibvQbkbib3G1H1D7vPPfrT5c0zbnpA/640?wx_fmt=gif&from=appmsg "")  
  
**补丁与缓解步骤**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXqCUdzq2HQ2bGZZE0NSNTpKvxhsbzomWYsIDwibsmLDJ0ibJzuqCWnpznwLaMibBFXQkgFSnSMsibbJMEL1VhN8uYntr2zl3HPN1Q/640?wx_fmt=gif&from=appmsg "")  
  
  
  
Cisco  
在  
ASA  
版本  
9.16.4.104  
、  
9.18.4.96  
、  
9.20.4.50  
、  
9.22.3.200  
、  
9.23.1.48  
、  
9.24.1.27  
和  
9.24.10.1  
中修复了该漏洞，也在  
FTD  
版本  
7.0.11  
、  
7.2.13  
、  
7.4.9  
、  
7.6.7  
、  
7.7.14  
、  
10.0.3  
和  
10.1.1  
中予以修复。  
  
目前虽然没有完整的应变措施，但思科提到  
“  
存在一种缓解措施，即在任何用户配置的隧道组列表或默认隧道组  
DefaultWEBVPNGroup  
上禁用辅助认证。  
”  
如果  
DefaultWEBVPNGroup  
不使用辅助认证，管理员可以运行  
no tunnel-group-list enable  
。否则，可以运行  
no secondary-authentication-server-group  
将其移除，直到完成修复。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[思科修复 Expressway 严重漏洞，可用于 RCE 攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527504&idx=1&sn=8e7175cc4854eb2e71a497ff5bbcf412&scene=21#wechat_redirect)  
  
  
[思科：注意可导致 Nexus 交换机遭接管的多个严重漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527286&idx=1&sn=eb70fbb49754e65564698f039b45ba32&scene=21#wechat_redirect)  
  
  
[思科：ISE 认证绕过满分 0day 已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527173&idx=1&sn=1ae52540af18bbb2b176b85380e718cc&scene=21#wechat_redirect)  
  
  
[思科紧急提醒：Secure Email Gateway 0day 漏洞已被用于运行恶意代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527140&idx=1&sn=81268f1bb9309a82d8c875415dcca36a&scene=21#wechat_redirect)  
  
  
[思科：注意 CVSS 满分 Crosswork SQL 命令注入漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526935&idx=2&sn=6df734eb52e6f93d21542cbc093f1c92&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://securityonline.info/cisco-secure-firewall-vulnerability-exploited-ssl-vpn/  
  
  
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
  
