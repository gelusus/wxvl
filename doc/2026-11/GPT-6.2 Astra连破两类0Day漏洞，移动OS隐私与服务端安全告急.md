#  GPT-6.2 Astra连破两类0Day漏洞，移动OS隐私与服务端安全告急  
 FreeBuf   2026-11-23 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX20I0DfLOe3Xz6VmXyBiben75zhBlqmDxF08px4lhH7y0qIqicP02BCL5X8NDvXSpHb6pB3WxibWXNPMmdq4cTPm0ovcnFmk0CXWc/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX38hicascVzGl1NMXjpblEbr1icHE76g6eLaibRJbf5DWRiatCSkV3Nfycgx0GdY68OyZJV0nf1zkS1BetDSpX9rWoMBPm0QpnBWKo/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
两类漏洞暂未披露受影响产品细节  
  
  
根据OpenAI最新发布的系统卡，其旗下GPT-6.2 Astra已发现并成功利用两类0Day漏洞，分别影响一款主流移动操作系统和一款被广泛部署的网络服务。  
  
  
本次发现的漏洞分为两类：一类是权限绕过漏洞，可导致隐私数据泄露；另一类是RCE漏洞，低权限角色即可触发。这些漏洞来自安全厂商Irregular的测试发现，并非基于针对真实用户的攻击报告。  
  
  
OpenAI在11月17日发布的系统卡中说明，目前Irregular正负责推进漏洞的负责任披露流程。公告未提及受影响产品名称、存在漏洞的版本、对应的CVE编号，也未确认补丁的具体发布时间。  
  
  
其中影响移动操作系统的漏洞，可允许恶意应用在未获得所需权限的情况下访问隐私数据，说明该系统的访问控制机制存在缺陷。目前公开的信息未说明受泄露的数据类型，也未解释恶意应用具体绕过了哪些权限检查环节。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1nBFuBJIWKtNJJOVFib5L7eAq6GgD91eNzVkrvSMhNnjMhc7Z93AoMa00zNPDjIB8UuKCssT7d6LK4TczI2nEAZXajwaEu5yQQ/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
漏洞未形成移动利用链  
  
新模型整体表现未超越前代  
  
  
本次发现的RCE漏洞存在于一款独立的网络服务中。Astra从低权限角色出发完成了对该服务的利用，证明仅需有限权限即可触发代码执行。本次报告未证实漏洞支持未授权访问、管理员权限获取，也未证实两类漏洞可串联形成针对移动设备的完整利用链。  
  
  
明确区分两类漏洞的影响范围十分重要。移动OS漏洞的核心影响是受保护信息被未授权访问，网络服务漏洞的核心影响是攻击者可执行任意代码。两类漏洞的细节描述，都无法支撑“Astra可远程接管手机、或入侵该未具名操作系统下所有设备”的结论。  
  
  
Irregular通过FrontierCyber与CyScenarioBench两款基准测试工具对Astra开展测试：前者用于衡量真实系统环境下的offensive skills，后者覆盖多阶段安全运营场景。  
  
  
测试结果显示，Astra在217道FrontierCyber挑战中解决了68道，而GPT-6 Astra的解题数为77道。也就是说，这款新模型虽然发现了严重的安全漏洞，但在整套测试集上的整体表现并未超越前代。  
  
  
两款模型均未能成功攻破经过全面加固的目标，FrontierCyber中难度最高的15道挑战均未被任何一款模型解决。在CyScenarioBench测试中，两款模型都至少一次完成了11道挑战中的9道；其中Astra的总成功率为63%，前代模型为61%。  
  
  
OpenAI单独开展的测试显示，模型在为已知漏洞开发利用代码方面的进展更为明显。Astra在ExploitBench Internal Port测试集上的平均任意代码执行成功率达到53.5%，而GPT-6 Astra的成功率为31.5%。  
  
  
这项评估使用的是2026年6月至8月间披露的V8和Chrome漏洞，并非新发现的0Day漏洞。明确这一区别有助于读者客观评判测试结果。发现未知漏洞、将已披露漏洞转化为可运行的利用代码，这两类相关skills属于完全不同的任务。此前关于Astra开展0Day研究的报道，已经为这种不断提升的能力提供了背景参考。  
  
  
Part  
03  
  
评估未启用生产防护  
  
暂无漏洞在野利用证据  
  
  
OpenAI表示，GPT-6.2 Astra与Sol均达到其设定的关键网络安全能力阈值。公司针对高风险辅助功能设置了更严格的限制与监控机制，对更高权限的访问流程增设了额外审批环节。本次能力评估是在未启用生产环境安全防护的条件下开展的，因此测试结果不能代表普通客户正常访问时的模型能力。  
  
  
此前关于OpenAI安全隔离测试的相关报道，也印证了除模型规则外，隔离管控的重要性。这类AI系统可以帮助研究人员发现安全弱点，但必须对其可调用的工具和访问权限设置严格的边界。  
  
  
目前由于缺少受影响产品名称和补丁细节，用户暂时无法开展针对性的暴露面排查。当前可以确认的是，模型仅在评估环境下成功完成了0Day漏洞研究，并无证据表明相关漏洞已被广泛利用。后续的漏洞披露信息将明确哪些用户需要安装修复补丁，以及厂商需要以何种优先级响应处置。  
  
****  
  
参考来源：  
  
OpenAI’s GPT-6.2 Astra Discovers 0-Day Vulnerabilities Enabling Mobile OS Security Bypass and RCE  
  
https://cybersecuritynews.com/openai-gpt-6-2-astra-0-day-vulnerabilities/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
