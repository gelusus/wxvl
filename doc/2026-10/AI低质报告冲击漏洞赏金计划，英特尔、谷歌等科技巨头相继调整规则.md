#  AI低质报告冲击漏洞赏金计划，英特尔、谷歌等科技巨头相继调整规则  
 FreeBuf   2026-10-15 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX0ayXrb9wyWibiatCj0C3sniaicib20LRcYmPsico7NShhI7lUt4q57DDOiaTOgibkDsrjaQ6Sibmq528rqTDzbicwnNb7jPCQqoCoibVmGcw/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0GJdrpticA7Uh68gIjSsQ9ia9Ymog4EVDYe2UyyiaNH1WLxR5swIPS83CNwWzzRdFIzxTI4sxnC98kLfNtcy28fibPdEYicyLC5tMA/640?wx_fmt=png&from=appmsg "")  
  
  
生成式AI工具大量涌入漏洞提交流程，产出大量看似真实可信的低质无效内容，科技巨头开始重新评估传统付费漏洞赏金计划的投入产出比。在外包审核团队不堪重负、内部AI自动化审计逐渐兴起的背景下，英特尔、谷歌等行业重量级企业正推动漏洞披露规则全面重构。  
  
  
这些最初的零散调整，可能预示着全行业的方向转型。今年早些时候，Curl创始人Daniel Stenberg因不堪海量LLM生成的误报困扰，直接关闭了该项目的漏洞赏金计划。  
  
  
9月中旬，有消息称英特尔已悄悄暂停了运营多年的漏洞赏金计划的现金奖励。数周后的10月1日，谷歌停止接收开源软件漏洞奖励计划（Open-Source Software Vulnerability Reward Program, OSS VRP）的漏洞报告，明确给出的理由是自动化生成的无效报告大量涌入，已经到了无法承受的地步。  
  
  
这些举措共同标志着漏洞管理领域迎来转折点。安全团队正面临极不对称的资源消耗：提交者几乎零成本就能批量生成AI报告，安全团队却要投入数百个成本高昂的工程工时，手动审核并排查每份报告的有效性。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1IiaKK2FJVLHQ4JdW8hzcU1ltcwALfVd0XrjjZ22s4WibK77k87pzxsQGZDNOiav0n21ibk2Q1kvgG2hq7VHiaoWe18smbk1IrwZVw/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
漏洞初审承压  
  
无效报告消耗大量审核资源  
  
  
英特尔突然暂停付费漏洞赏金，是这一行业运行环境变化下最受关注的标志性事件。  
  
  
英特尔在给TechTarget的声明中表示："为了评估漏洞赏金计划如何更好地支撑英特尔整体产品安全工作，我们正在开展相关评审，目前已暂停该计划，重启时间另行通知。英特尔将继续履行已有的漏洞赏金承诺，同时通过漏洞披露计划持续接收安全报告。"  
  
  
对于这一决定是否因AI工具用于漏洞挖掘的情况增多导致，英特尔没有回应后续追问。不过，英特尔赏金计划的托管平台HackerOne首席执行官Kara Sprague证实，全行业企业漏洞赏金计划的提交量都在急剧攀升。她表示："现在很多报告乍看可信度很高，但提交者根本没有针对目标做过测试，或是内容完全不在计划受理范围内——生成这样一份报告现在只需要几分钟。但验证报告需要投入的精力丝毫没有减少：必须复现漏洞、确认影响范围，再判断是否需要修复。这是目前所有漏洞赏金计划都面临的核心瓶颈，也是我们平台搭建核心能力所要解决的问题。"  
  
  
行业数据充分体现了当前漏洞披露环节的瓶颈有多严重。根据谷歌近期发布的报告，2026年1月至7月，月度漏洞披露数量翻倍，1月为5045个，7月升至10477个，8月进一步增长至10740个。  
  
  
Part  
02  
  
重心转向内部审计  
  
企业重构安全预算分配  
  
  
Curl项目维护团队的经历已经证明，提交量大幅上涨会占用分析师和研究人员的大量核心时间，而这些人员本就需要逐一审核每份报告。因此，行业的重心正在转移：一方面要求提交内容必须满足明确标准，另一方面减少审核无效误报所花费的时间。  
  
  
Daniel Stenberg在博客文章中回顾Curl漏洞赏金计划停摆的过程时指出，ChatGPT等生成式AI工具的普及，与漏洞提交队列中报告有效率的暴跌存在明确关联。他写道："回头看，我们可以确定漏洞赏金计划的问题从2024年下半年就开始缓慢显现，2025年情况急剧恶化。"  
  
  
英特尔方面证实，其漏洞赏金计划只是暂停，并非彻底终止。Luta Security首席执行官Katie Moussouris认为，英特尔暂停漏洞现金奖励主要有两方面原因：一是AI生成的低质报告泛滥，二是英特尔内部团队已经开始部署AI工具，能更高效地在内部挖掘漏洞。  
  
  
这些高关注度的暂停事件，让外界开始质疑众包安全模式的长期可行性。不过专家认为，当前的动荡并不意味着漏洞赏金模式会彻底消失，而是企业安全预算的一次永久性调整。  
  
  
Moussouris在接受TechTarget采访时表示，漏洞赏金的奖励标准不会回到此前的水平。企业仍然欢迎漏洞披露，但整体格局取决于AI能以多快的速度实现内部测试和补丁制作的自动化。她表示："相比此前分配给各类外部测试的预算和时间，企业在安全内置类软件、内部使用AI发现并修复漏洞上的投入占比会更高。对大多数组织而言，AI工具、Agentic红队演练、漏洞赏金计划带来的负担过重，很难在大量重复报告和误报中筛选出有价值的内容。"  
  
  
Part  
03  
  
行业发展面临抉择  
  
暂停赏金或流失优质研究员  
  
  
Kara Sprague预计，会有更多漏洞赏金计划调整规则，部分计划可能暂停发放奖励。她表示："很多计划运营方面临的问题十分常见：报告提交量持续上涨，总得有人筛选出值得工程团队处理的内容。但这件事还有另一面：现在我们发现的有效漏洞数量也比以往任何时候都多。暂停计划或许能减少报告总量，但也会错过那些能发现真实问题的研究人员提交的成果。"  
  
  
她强调，暂停奖励是一个艰难的选择——计划暂停后，平台可能依然会收到同样多的未审核原始报告，同时却失去了有能力发现复杂漏洞的资深研究人员的支持。她表示："如果核心问题是海量报告的分拣效率，我们就需要提升这方面的能力。这些研究人员的发现依然值得重视。"  
  
  
英特尔暂停赏金计划的影响将波及整个行业：不管是希望通过提交漏洞获得报酬的研究人员，还是负责接收报告、验证漏洞、开发补丁的企业，都会受到影响。  
  
  
随着头部科技企业纷纷收缩漏洞赏金计划的部分投入，接下来的问题是其他企业是否会跟进，以及整个行业能否适应新的环境：AI降低了生成潜在漏洞报告的门槛，但验证漏洞的有效性依然是一项巨大挑战。  
  
  
参考来源：  
  
Bug bounty reckoning: AI slop forces enterprise strategy shift  
  
https://www.techtarget.com/cybersecurity/news/366652109/Bug-bounty-reckoning-AI-slop-forces-enterprise-strategy-shift  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
