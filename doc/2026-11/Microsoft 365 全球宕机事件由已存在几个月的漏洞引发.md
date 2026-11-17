#  Microsoft 365 全球宕机事件由已存在几个月的漏洞引发  
Lawrence Abrams
                    Lawrence Abrams  代码卫士   2026-11-17 08:51  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**微软表示，****7****月引入的一个软件缺陷导致了上周全球性的****Microsoft 365****宕机事件，引发连锁故障，使其超过****80%****的网络网关服务器无响应。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
11  
月  
12  
日持续近  
24  
小时的宕机扰乱了全球  
Microsoft 365  
服务，包括  
Outlook  
、  
Exchange Online  
、  
Teams  
和  
Copilot  
，导致用户无法访问电子邮件，并造成邮件投递严重延迟。在  
Microsoft 365  
管理中心分享的初步事故报告中，该公司表示，此次宕机由三个问题导致：一个此前未被发现的软件缺陷、  
Microsoft Teams  
故障转移后流量增加，以及服务器容量减少。  
  
该事件始于美国东部时间  
11  
月  
12  
日周四上午  
11  
：  
20  
，持续至次日上午  
11  
：  
10  
，微软确认服务已完全恢复。据微软称，该软件缺陷是  
7  
月通过为  
Microsoft 365  
网络网关服务器开发的一个更新引入的。虽然该更新已部署三个多月且未出现问题，但它包含一个缺陷，在某些流量条件下可能使服务器过载，导致其停止响应。微软表示，其对该更新的测试未能发现该缺陷，因为测试所用硬件的性能低于生产服务器。“我们正常的负载测试未能发现这一潜在问题，因为实验室中使用的流量组合和  
SKU  
不同，  
”  
微软在初步报告中解释道，“这种争用问题在核心数更多、硬件更新的机器上更为严重；而我们的负载测试往往侧重于核心数较少的较弱硬件。”该缺陷最终在  
Microsoft Teams  
于宕机前约  
15  
小时自动切换到  
Exchange Online  
使用的网络基础设施后触发，增加了这些服务器上的流量。  
  
Microsoft Teams  
当天早些时候还经历了另一次宕机，但目前尚不清楚这两起事件之间是否存在关联。更糟糕的是，微软表示，由于一个无关问题，网关近  
14%  
的服务器容量已经离线。虽然基础设施有足够容量应对这一减少，但流量增加与容量减少的结合触发了该软件缺陷。随着服务器停止响应，微软的系统将用户重定向到其它可用服务器，这些服务器也因额外流量而不堪重负并同样失效。“在不到两小时内，这一此前未知的缺陷波及我们的整个服务器群，使其无响应。最低谷时，超过  
80%  
的服务器无响应，  
”  
微软解释道。  
  
微软表示，旨在防止问题在区域间蔓延的防护机制也未能遏制连锁宕机，使中断扩散至全球。虽然  
Teams  
、  
OneDrive  
和  
SharePoint  
等其他  
Microsoft 365  
服务可以切换到替代基础设施并更早恢复，但  
Exchange Online  
无法进行同样的切换，导致用户无法访问其邮箱。微软表示，恢复工作困难重重，因为重启受影响的服务器会导致它们很快再次被积压的流量压垮。在确定该缺陷后，微软工程师开发了修复程序并部署到受影响的服务器上，使服务在周五全天逐步稳定。微软确认，截至美国东部时间  
11  
月  
13  
日上午  
11  
：  
10  
，对客户的活跃影响已经结束。该公司表示，此后已增加容量、改进监控，并添加了防护机制，以防止流量重定向触发类似的连锁故障。  
  
微软负责  
Copilot  
、代理和平台事务的执行副总裁  
Charles Lamanna  
在  
X  
社交平台发布的一份单独声明中，为此次中断致歉，“我们对给所有客户造成的影响深表歉意，并认识到这对他们正常业务和运营的影响。”  
  
微软表示，调查仍在继续，最终事故报告预计于本周晚些时候发布。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[安全公司Tenable插件更新有bug，全球Nessus 代理皆宕机](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247521996&idx=1&sn=96013ce25046ff02b2d52d5d4a7a3423&scene=21#wechat_redirect)  
  
  
[CrowdStrike 宕机后，微软拟让EDR厂商在内核模式外”运行](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247520848&idx=3&sn=4a3ec7b673c6f5a5650610d98d80b076&scene=21#wechat_redirect)  
  
  
[技术提供商遭供应链勒索攻击，逾60家信用社服务宕机](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247518271&idx=1&sn=06e13d43d03ff0da28c0310447755248&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/microsoft/microsoft-says-months-old-bug-caused-global-microsoft-365-outage/  
  
  
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
  
