#  Anthropic紧急断网，Claude利用注入漏洞越界入侵真实网站  
 FreeBuf   2026-10-11 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX2uUSWaFbD0COB5L6P6T6SyyFXlwkPicoVFpODKGRI1jyRWfuOD0qLHtibBJ6CPUCKEObcCfDERpDcF80kyK2ibicUtaYhEOCVblIM/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX2icrhmDGN5BKibhQ7ofaUcrp6yHRGpuRh9pV1j3LXEKA8x4P5btpiastojKP98wn65cwxUdFetVuFw3gwJNUkiaIfIgFmUdScHrJY/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
Anthropic切断内部测试联网权限  
  
  
Anthropic于周五宣布，近期发现旗下AI模型出现对齐失效行为、针对真实网站发起操作后，将切断所有内部评估环境的实时互联网访问权限。  
  
  
在Claude的内部评估和使用过程中，Anthropic共发现四类非预期模型行为，具体包括：  
  
- Claude Mythos Preview 利用未具名第三方软件中的SQL注入或命令注入漏洞，在某大学服务器上执行命令。出现该行为的原因，要么是模型自带工具被人为限制，要么是所需的外部服务不可用，导致模型转而调用第三方站点上托管的其他工具完成任务。  
  
  
- Claude Haiku 4.5 及一款非前沿研究模型在未获授权的情况下，向真实网站提交了包含敏感信息的表单。这类情况出现在指令模糊，或环境配置错误导致agent无法使用测试表单的场景中。  
  
  
- Claude Mythos 5 绕过访问限制，获取受令牌或付费门槛保护的数据，例如识别照片中的地点、调取国家机构公开的可用数据。  
  
  
- Claude 使用URL缩短服务，绕过其fetch工具的访问限制。  
  
  
出于保护涉事机构安全的考虑，Anthropic不会公开相关机构名称，以免暴露其系统中可能存在的漏洞。按照公司的评估，这些事件对现实世界造成的影响极小。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0PiceQM34Nx57xDOuPoXThhcKZibNTa65rnLXaribegebibkuGVCPkfsnEXIVsep4W10eZNoS4iaV2QATeErbfGchDmHicpkfYMVFvE/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
模型越界访问政府站点  
  
  
Anthropic透露，部分事件的目标是美国联邦、州、地方层级政府机构运营的网站。其中一起属于上述第二类行为的事件中，Claude Haiku 4.5访问了某警局运营的悬案线索提交页面，页面内容涉及一起未破凶杀案。  
  
  
尽管测试人员明确指示模型不得输入个人数据、创建账号、发起购买或提交任何破坏性内容，但模型未将表单提交纳入约束范围，最终提交了一条虚假凶杀案线索，内容如下：  
  
  
“我可能掌握与本案相关的信息。我记得案发时段在[页面提及的街道]附近见过符合描述的人员。如果该信息有价值，请与我联系。”  
  
  
后续核实显示，这起事件的涉事方为美国费城警察局（PPD），这条虚假线索于2026年7月18日通过PhillyUnsolvedMurders.com提交。但Anthropic直到2026年9月28日才发现该事件，随后于2026年10月7日通知费城警察局。  
  
  
虽然系统最终将这条线索识别为垃圾信息，但费城警察局对事件的处理过程提出了批评。警方认为，Anthropic需要进一步完善安全防护，避免类似行为在市政府毫不知情的情况下影响公共系统。尤其是事件发生两个多月后才被发现并通报，这样的延迟难以接受。  
  
  
Part  
03  
  
企业扩大断网覆盖范围  
  
  
这些非预期行为是在Anthropic自2026年7月启动的日志审查工作中陆续发现的。  
  
  
今年7月，Anthropic曾披露三起模型越界事件。由于测试环境存在配置问题，Claude在网络安全评估过程中实施了未授权操作，并入侵三家外部机构的系统。  
  
  
上月，Anthropic又披露了发生在2026年1月的第四起事件，涉事模型为早期版本Claude Opus 4.6，该模型“因无法中止任务，入侵了第三方系统”。  
  
  
Anthropic表示：“尽管这些行为的影响极小，且我们此前已切断部分高风险、网络安全相关评估环境的实时联网权限，但现在我们决定将这一措施扩展到所有内部评估场景，直到确认我们的安全与监控措施能够可靠拦截这类行为。”  
  
  
这一最新发现促使这家AI巨头启动更深度的扫描，重点覆盖Claude可访问互联网的环境。Anthropic表示，随着调查推进，预计还会发现新的非预期行为案例。  
  
  
Part  
04  
  
AI安全风险引发全行业关注  
  
  
近几个月AI安全担忧持续升温，此次事件正是在这一背景下发生。2026年7月，失控的OpenAI Agent突破测试环境入侵Hugging Face的事件曝光后，陆续有多起相关网络事件浮出水面。  
  
  
随着AI模型厂商推出的模型能力不断增强，其安全实践也面临越来越严格的审视。全行业纷纷发出警告，称模型发展速度已经超过安全护栏的迭代速度。业内同时呼吁放缓AI开发节奏，出台更多监管措施。  
  
  
本周早些时候，英国信息专员办公室（ICO）表示，亚马逊、Anthropic、苹果、Cohere、DeepSeek、谷歌、Meta、微软、OpenAI、Stability AI等10家头部基础模型开发商，已经或承诺调整其数据保护政策。  
  
  
这些调整主要涉及三个方面：向公众提供更加清晰透明的数据处理说明，完善用户行使个人信息相关权利的保障机制，以及对现有安全措施开展更加严格的评估。  
  
  
英国信息专员办公室技术监管主管Richard Nevinson强调，人工智能虽然具有巨大的社会应用潜力，但这种潜力必须建立在公众信任和信息透明的基础上。随着AI系统能够独立完成越来越多操作，企业更需要确保个人数据得到充分保护。  
  
  
尤其对于具备自主行动能力的AI Agent，技术能力的提升不能成为企业忽视合规义务的理由。公众有权了解自己的个人信息如何被收集、使用和保护，而这也是AI技术获得长期信任的重要前提。  
  
  
参考来源：  
  
REA Tool Connects Claude Code and Cursor to Ghidra and IDA Pro to Reverse Engineer Anything  
  
https://cybersecuritynews.com/reverse-engineer-anything-tool/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
