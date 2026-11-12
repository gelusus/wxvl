#  访谈｜安恒信息研究院院长王欣：PHP2Shell9.2分高危漏洞敲响警钟，堆人堆工具的时代过去了  
 信息安全研究   2026-11-12 07:00  
  
**点击蓝字 关注我们**  
  
  
  
  
**【编者按】**  
  
近期，PHP爆出代号“PHP2Shell”的高危远程代码执行漏洞，CVSS评分高达9.2分。该漏洞由AI辅助安全研究发现，披露后引发多国网络安全机构预警，同时有威胁情报显示，攻击组织已开始大范围侦察。  
  
AI一方面让安全研究者更高效地发现底层漏洞，另一方面也降低了攻击者的攻击门槛，攻防不对称矛盾更加突出。传统依赖安全工具、靠人力扩张的安全服务模式，越来越难以适配快速演化的安全风险。围绕AI驱动安全能力变革，数字员工、人机协同正从概念走向落地。安恒信息推出X安全数字员工体系，尝试把安全专家经验转化为可标准化、规模化交付的AI安全能力。  
  
为此，《信息安全研究》专访**安恒信息高级副总裁王欣**  
，结合PHP2Shell漏洞事件，剖析AI时代网络安全面临的现实挑战，解读安全数字员工的技术逻辑、落地价值，探讨网络安全行业从“交付工具”走向“交付专业安全能力”的路径。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/buQvkZfsTH8RIv16PuvEpqKAyrsgtPAeibvDcUdQeW5bBayFF2NH1rb5EdCRA8c0Dr0QHoRZsib8mOMqFKGicS8h0FMznf1qKpTWOiapGHbibVLw/640?wx_fmt=jpeg "")  
  
**王  欣**  
  
  
  
安恒信息高级副总裁、中央研究院院长，深耕网络安全十余年。主导研发恒脑安全大模型及智能体平台，落地杭州亚运会、哈尔滨亚冬会等重大活动重保，获吴文俊人工智能科技进步奖等国家级荣誉；输送原创漏洞上千项，持发明专利 32 项，主编《AI 驱动安全》。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHicERnYdcPwAvKa6qAIPl9ZWPLDrPrX3IZwGp3pfOoZODSLH2mb4bFNh29Fnt2Uro6CYcxtc3D51F7VOmibDusiczdjx5APSAuwHA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTHibBuSweKuhJgxzhjDDVf0nmpjnVSibWvU5P7xxhPAbwzqFJRiaRjgk3fEEvAZchSPuFSqeCbKUAqgal2KsH0GeBJWEyoiaLYicnuoc/640?wx_fmt=png&from=appmsg "undefined")  
  
王院长，您好。PHP作为广泛使用的底层服务端语言，这次核心环境爆出高危漏洞，您怎么看它带来的现实风险？  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH9iadfFFdaYYZgEM8J585cTd2ougVOB0HCKFhw1ibQlcXnVzlQfyfb7kCUeoTozWRK2UUU3dBasiaxXSPLj9prGlDGNH9MrkuhwbA/640?wx_fmt=png&from=appmsg "undefined")  
  
PHP  
大量应用在企业官网、电商平台和各类业务系统中。这次漏洞不是某个插件、某套应用的问题，而是PHP核心运行环境层面的缺陷，风险覆盖面很广。漏洞披露后，欧盟网络安全局以及加拿大、荷兰等多国网络安全机构迅速发布预警，威胁情报也观测到攻击组织已经启动大范围资产侦察，重点瞄准政府、金融、制造、物流等行业。  
  
这件事有一个很关键的信号：重大漏洞公开后，留给企业排查风险的窗口期正在被压缩。攻击者借助自动化工具快速全网扫描、筛选目标，防守方却要面对数量庞大、环境各异的业务资产，时间压力非常大。更值得行业重视的是，本次漏洞的研究发现环节，AI模型发挥了重要作用，OpenAI和云安全厂商Wiz都参与了漏洞挖掘与报告。这说明AI已经深度进入复杂底层软件漏洞研究，攻防两边都在获得AI带来的能力增益。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHibJzTDNApdgwnVNcs4DlwwxanmtrVDPnElfDdrap94JFeaYry8fD3NKicXhnJ0O9J5tMLXRbf7wCUC4jfUsbo6ia63sicKjd9r1Fc/640?wx_fmt=png&from=appmsg "undefined")  
  
现在，一边是AI辅助研究者加速漏洞挖掘、前置识别  
风险  
，另一边攻击者通过AI拉低攻击门槛，此消彼长下，攻防不对称是否会被进一步放大？  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH8R2N3lBBfclB5UmZsPEdnznstyQmprckjtnWADWiczkste1VIsTHTUGNhXSWgCLov7qtwceReIGBXkGqoyf4Aia619ic3hY0McYc/640?wx_fmt=png&from=appmsg "undefined")  
  
的确如此，AI正在改变网络安全攻防的基本逻辑。从漏洞发现看，过去复杂底层漏洞高度依赖顶尖安全专家的经验积累，现在借助AI的代码理解、推理能力，漏洞研究效率被大幅放大，复杂缺陷的发现速度明显提升。  
  
从攻防成本看，攻击者利用AI可以快速完成漏洞分析、目标筛选、攻击脚本生成，攻击门槛持续降低；防御侧则要守护海量异构业务系统，风险排查、验证、处置链条很长。  
  
更深一层的问题在安全能力供给。今天企业手里并不缺少安全检测工具，但从拿到漏洞情报、确认自身资产是否受影响，到真正落地处置，大量工作仍然依赖稀缺的安全专家和人工流程。工具可以不断采购，专家人力却很难同步规模化扩张。  
  
所以，AI时代的安全挑战，不只是把检测速度做快。真正要回答的问题是：怎么把专家经验、技术能力和处置流程，转变成可以自主运行、持续输出结果的安全生产力。说到底，AI时代拼的不再是谁工具多、人手足，而是谁能把专业能力变成持续运转的生产力。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH8jx3s2oCBvWFVI4DQ0SBxgpBzvttduibYr1lNSfiafdActUbOD2hn2fvzfkP8FGLUEmKf1g4pcyZ5cfZbqZ2khLCia93yyO6l57c/640?wx_fmt=png&from=appmsg "undefined")  
  
针对您刚才说的这些痛点，安恒信息推出了X安全数字员工，全球首发渗透测试数字员工“安小龙”。这套体系怎么运转？名字里的“X”，又意味着什么？  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH9zOZFXRvXFq8icVEMxaMYibC7kp3sNEnerkQv3X7d2pgED5ANo5OnRPz8W0f7s3X8wOEAQniaGnMM1Uk6acpAyceZy8yfljgibqfw/640?wx_fmt=png&from=appmsg "undefined")  
  
很多朋友会问“X”是什么意思。说实话，它没有一个固定的标准答案，我更愿意把它看作一个开放的指向——既代表岗位可以不断延伸，也暗合数量没有上限的意思。回到这套体系本身，它要解决的核心问题很明确：把资深专家的经验，变成能自主干活的能力。  
  
安恒X安全数字员工，说白了就是AI时代下一代安全服务的交付方式。我们先把资深安全专家沉淀下来的专业知识、实战经验和标准化工作方法，转化成可以独立开展工作的数字员工。数字员工不是要替代人，而是和人类安全专家协同完成各类专业安全任务，把安全服务推向更智能、更自主的方向。  
  
和传统模式最大的差别在于：传统模式是“人操作工具”，服务交付高度靠人力堆；X安全数字员工具备自主理解任务、规划执行路径、落地开展专业工作、输出完整成果的能力，能做到7×24小时不间断运行。它和我们现有的专家团队、安全服务体系是互补协同的关系，推动安全服务从过去“交付工具、投入人力”，升级到“自主执行、持续交付”，让高质量专业安全能力更容易获取、持续可用。  
  
目前我们已经落地四位成熟数字员工：安小龙负责渗透测试，安小虎负责代码审计，安小风负责安全运营，安小云负责安全托管。支持灵活聘用、按需扩容，适配大中小不同客户的差异化需求。  
  
拿这次PHP2Shell事件来说，依托零日雷达的漏洞情报体系，渗透测试数字员工安小龙快速集成专项检测技能，在客户授权环境下自动完成资产摸排、漏洞风险验证并输出检测报告，帮助企业尽快确认哪些资产真正暴露风险，优先处置重点资产，缩短从拿到漏洞情报到完成风险处置的链路。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH9SibZWmb7pcct4H0qlG0XNtbCXZH3pTM4ScmnlsR60mlPekgAJQJ41ZiafFwC3VXM6ANFdEso1Kd29Vt6l34icpwTAZmgpe3BhVs/640?wx_fmt=png&from=appmsg "undefined")  
  
很多人会简单理解，安全数字员工就是AI替代安全工程师。在您看来，它的价值应该怎么衡量？  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTHicRoBPuEfjqjx3icqjiaCeLWsics5GoJw5MaibRwnb6bgIASjWFmYFLbwc1kUeZOnXDtJMf9mh1ctSicddf7hyiaiak62Sc19dVx9YlNI/640?wx_fmt=png&from=appmsg "undefined")  
  
我不建议简单用“一个数字员工等于多少个人工”去对标。它的核心价值，是改变安全服务产能和人力投入之间的绑定关系。看它的价值，不是看它顶替了几个人力，而是看它帮客户落地了多少真实的风险处置。  
  
传统安全服务，交付规模受专家人数、工作时长、项目周期约束。数字员工可以并行处理多项任务，持续不间断工作，把大量标准化、重复度高的安全任务规模化交付。评判价值，要看单位安全投入能完成多少专业任务、覆盖多少业务资产、落地多少真实风险处置成果。  
  
还要强调一句：AI不会取代安全专家。重大安全事件研判、复杂业务影响评估、关键安全决策，依然离不开人的专业判断。数字员工承担的是高频、重复、标准化的工作，把安全专家解放出来，投入到更复杂的风险分析和决策中。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9ibzzkSWINnlbW6RDicm2PuxBlgVc5l1R0LuES0fIOjnYpjwzLkwiac4VO1xzXEwxC1ickso7CwlslbFk5EBpJxQ7JcsDo64P1QYY/640?wx_fmt=png&from=appmsg "undefined")  
  
这套模式对大型企业和中小企业，能解决他们什么问题？  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHiceC7xdaSH53Nd0DRmakiaFlEI5CMicNHusNCamrllvicBYDfPuIG2mX0RoEsUTmLv9pJwVhGUpwh3BxIEjayZ79h9SgB5wY6URGQ/640?wx_fmt=png&from=appmsg "undefined")  
  
企业安全长期有一个现实矛盾：外部风险不断增长，但内部专业安全力量很难同步增长。两类企业的痛点差异很大。  
  
大型政企客户大多已经部署全套安全产品，也组建了内部安全团队，但普遍面临资产分散、告警海量、多系统协同繁琐的问题。X安全数字员工可以承接资产排查、漏洞批量验证、风险梳理这类高频工作，减少人工重复劳动，释放现有安全团队精力，让已采购的安全体系发挥更大效能。  
  
中小企业的痛点则是“用不起、配不齐完整安全团队”。不少中小企业同样会遭遇漏洞攻击、数据泄露风险，即便采购了安全硬件，缺少运维人员，设备能力也很难落地。借助数字员工加安全托管模式，中小企业可以获得持续的风险监测、漏洞管理和安全运营支持。我们也提供租赁这类灵活交付模式，企业按需采购对应安全岗位能力，不用承担搭建整套安全团队的一次性成本。  
  
行业的风向也在变：客户要的不再只是一堆安全工具，而是安全工作实实在在运转起来，风险能被持续发现、验证、处置，从一次性项目式服务走向持续性安全能力。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTHicz8mZtauU0CIE2x7icvprFOOiboXdczcRAbe85TyKqJtSJX4jZRQJ25MGsQuCwia4JgBfoXniciaviaV63IYw95V7YnZomTveFJvwL0/640?wx_fmt=png&from=appmsg "undefined")  
  
回到PHP2Shell事件，这次事件也预示网络安全竞争逻辑正在改变。您怎么看行业未来的竞争走向？  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9lyNVdsI7ibjuwpwsFg7LwVY7dSrALRKghicOkGfZrBlpplXtPWvqhYYZ47dnkPeCO3fBpNQRqnTNNqBZjgX9TUPDYOh22ByUuM/640?wx_fmt=png&from=appmsg "undefined")  
  
PHP2Shell带给行业的启示，已经不止于一次漏洞的应急修复。漏洞挖掘、攻击侦察、风险扫描，攻防两端都在走向智能化、自动化。  
  
对企业来说，衡量安全能力的标尺正在变化：不再只看采购了多少安全产品，更要看能不能快速识别真实风险、调动专业能力、推动风险真正解决。  
  
对安全厂商来说，竞争也跳出单一产品功能比拼，变成安全研究、AI执行能力、整套服务交付体系的综合较量。从漏洞情报研究，到专家技术沉淀，再到数字员工自主执行任务，整套链条打通，才可以把专业安全能力变成可持续、可规模化输出的安全生产力。  
  
未来安全竞争的关键，不只是谁手握更先进的工具，而是谁能用更高效率、更低门槛，把专业安全能力交付给不同规模的企业。这也是我们做X安全数字员工体系背后，真正想带给行业的变化——让专业安全能力真正下沉，按需可用、持续在线。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9s8pnbdTYcic7sliaZodLoiafErAUbgXgJa6XroYXiakU49JALMSBvt6ibPtsJMyNcibPAPNQXQ9Ua5VUINoFr2sfKlXOrROgSicsFbE/640?wx_fmt=png&from=appmsg "undefined")  
  
站在当下，对于正在面对PHP2Shell这类高危漏洞威胁的企业，您有什么建议？  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHibhHe8XgIatE5fic3DVX5x8fkxicjRsWtIqhUzMBUOD2MPxBQ0AomnFGeD31OCgdqFHCN04cqPIaINFqwtJ1F4clfahTcrBL4nHI/640?wx_fmt=png&from=appmsg "undefined")  
  
第一，尽快完成资产梳理，确认环境PHP版本，优先完成官方补丁升级；第二，不要只停留在收到漏洞预警，要落地验证自身资产是否真实存在风险，区分风险优先级；第三，重大漏洞层出不穷会是常态，企业要思考如何建立常态化的漏洞处置能力，而不是漏洞爆出才临时应急。AI数字员工这类新形态能力，可以作为企业现有安全体系的重要补充。  
  
**后记**  
    
After the Interview  
  
  
**···**  
  
当 AI 同时作用于攻防两端，网络安全已进入生产力比拼阶段。PHP2Shell 漏洞折射出行业缩影：漏洞披露、侦察扫描、攻防对抗节奏不断加快，传统人力扩张模式难以为继。以安恒信息X安全数字员工为代表，行业正尝试将专家能力标准化、规模化交付，探索人机协同的新型安全服务模式。  
  
安全行业能否真正完成从 "卖工具" 到 "交付可落地专业安全能力" 的转型，答案尚待时间检验。![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHib6ibZ2SxLTPmAcichS6lZXws5DksiaLKRHzhRJ4Q8wfDPCdqkhZ9IztHMglrWERicB8U6raslBNEhuCBwiblK3ZcKGxzy2YlYtNCGI/640?wx_fmt=png&from=appmsg "")  
****  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9CA3UYlmtLhrM4ecLjPHb4Q7GJyFNMaUcTSmO545mAbgSYNX5keDm6jHyicLGYsP7wyMlkxiaBAbS2eYSUOjwl6a1puqWHicibGpQ/640?wx_fmt=png&from=appmsg "")  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WicaKuMLeYl3l23eSpOn1tBIbdC0Z6pRUzaRLw6Xdg6H5JggcVxceibhpPE7GLBwib6bIDMiaKsouqKQaJQWc2cF5Q/640?wx_fmt=jpeg&from=appmsg#imgIndex=15 "")  
  
  
