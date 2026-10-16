#  谷歌跳过 Gemini3.5，Gemini4 Argon 正式发布强化网络安全防御能力；CNCERT周报：后门网站与新增漏洞双升，仿冒攻击仍锁定银行政务| 牛览  
 安全牛   2026-10-16 06:28  
  
**点击蓝字 关注我们**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/wKeDC5RjIzEEg9ibH6bBNfLtqIOibE7icSPK5m4MFW3rS8SwX7RmE7IA3ZicK70V2J4dgYBkptkpqjX8qowjFMmvKJBDZRibhYrxtNGSe6nUyyiaM/640?wx_fmt=png&from=appmsg "")  
  
  
新闻速览  
  
  
- CNCERT周报：后门网站与新增漏洞双升，仿冒攻击仍锁定银行政务  
  
  
- CNNVD最新周报：新增2550个安全漏洞，超高危漏洞达1210个  
  
  
- 谷歌跳过 Gemini3.5，Gemini4 Argon 正式发布强化网络安全防御能力  
  
  
- 哈佛调研：男性总在不懂装懂地吹嘘自己懂AI  
  
  
- CISA 呼吁 AI 实验室公开测试网络 IP 与 ASN，以识别逃逸的恶意 AI 智能体  
  
  
- OpenAI 发生内部窃听事件：员工借助聊天机器人策划部署录音设备  
  
  
- 新型 Android 木马 Hirudo 剖析：以替换应用方式绕过本地安全校验  
  
  
- NGINX 发布新版本修复 5 处漏洞，含潜在 RCE 与 TLS 证书校验绕过缺陷  
  
  
- 微软联合多方瓦解 CoGUI 钓鱼套件生态，打掉 YAM Store 与 Iron Man 犯罪平台  
  
  
- Agentic AI 重塑软件采购逻辑：席位与用量混合定价考验 CIO 预算管理  
  
  
  
  
特别关注  
  
  
**CNCERT周报：后门网站与新增漏洞双升，仿冒攻击仍锁定银行政务**  
  
国家互联网应急中心（CNCERT）发布2026年第41期周报，统计周期为10月5日至10月11日。本周互联网网络安全态势整体评价为良，我国互联网基础设施运行平稳，全国或省级层面未发生造成重大影响的基础设施安全事件；主要威胁仍来自软件高危漏洞、恶意代码传播和网站攻击。  
  
  
监测显示，境内计算机恶意程序传播次数环比下降31.6%，Malware/Linux.Mozi仍居传播次数首位，Trojan/Linux.Winnti、Trojan/Win32.Siscrat等家族紧随其后，提示僵尸网络与木马类风险具备持续运营特征。网站侧呈现分化：境内被植入后门网站总数环比上升24.2%，域名类型以.COM及其他后缀为主；针对境内网站的仿冒页面数量环比下降16.4%，但行业分布高度集中，银行最多，政府公益、证券基金次之，说明钓鱼攻击正向高价值业务场景聚焦。漏洞侧，新增信息安全漏洞数量环比上升6.7%，其中高危漏洞占比最高，中危、低危依次减少，优先级处置压力仍集中在可被远程利用或影响面广的组件。  
  
  
事件处置方面，CNCERT本周协调云服务商、域名注册服务机构、应用商店、省级分中心及国际合作组织共处理网络安全事件401起，其中跨境事件375起，跨境占比约93.5%；并会同境内外域名注册机构、境外CERT重点处理17起仿冒投诉。对安全运营团队而言，建议将Linux服务器与物联网设备纳入Mozi类家族重点监测，同步推进高危漏洞补丁验证、Web后门清理与仿冒域名快速下架联动。  
  
  
原文链接：  
  
https://mp.weixin.qq.com/s/k1D1iXG8lVBFxkNWQsaebg  
  
  
**CNNVD最新周报：新增2550个安全漏洞，超高危漏洞达1210个**  
  
国家信息安全漏洞库（CNNVD）发布2026年第40期《信息安全漏洞周报》。数据显示，10月5日至11日期间，CNNVD新增收录安全漏洞2550个，较前一周有所上升，其中超危漏洞313个、高危漏洞897个、中危漏洞1211个、低危漏洞129个。  
  
  
从厂商分布看，Google新增漏洞298个，位居首位，Linux基金会和WordPress基金会分别涉及220个和176个。从漏洞类型看，授权问题最多，共490个，占比19.22%；资源管理错误和跨站脚本漏洞分别为240个和208个。  
  
  
本期重点披露多项高风险漏洞。Google Chrome 155.0.8059.39之前版本存在内存释放后重用漏洞，攻击者可通过特制HTML页面在沙箱外执行任意代码。IBM Langflow OSS 1.0.0至1.12.2版本存在代码注入漏洞，可导致任意代码执行。此外，LMCache和Obot AI Obot分别存在身份验证缺失和凭证保护不足问题，可能造成敏感信息泄露及API密钥滥用。  
  
  
修复方面，本周已有1938个漏洞发布补丁，整体修复率为76.00%，其中超危漏洞修复率仅64.22%。建议相关单位优先排查高风险资产，及时升级受影响组件并落实补丁修复。  
  
  
原文链接：  
  
https://www.cnnvd.org.cn/group1/M00/03/2C/rBBl8WrPKoaAKy_IAAT1JLp2GFY923.pdf  
  
  
  
热点观察  
  
  
**哈佛调研：男性总在不懂装懂地吹嘘自己懂AI**  
  
哈佛大学 Shorenstein 中心的调研，研究对 5400 余名美国受访者开展问卷调查，分析公众对人工智能行业的真实认知水平与主观自信度差异。  
  
  
调研问卷罗列一批 AI 行业人物，其中包含 OpenAI 的 Sam Altman、前 CTO Mira Murati 两位真实人物，同时混入 Susan Milner、Craig Dalton 两个完全虚构的人名。数据显示，73% 男性受访者表示认识 Sam Altman，53% 男性称自己认识 Mira Murati，同时也声称认识两名虚构人物；女性群体中，表示认识虚构人物的比例为 40%，仅 38% 女性识别出真实从业者 Mira Murati。  
  
  
在 AI 事实问答环节，男女受访者答题错误率接近，但女性更愿意选择 “不知道” 作答，男性更少承认知识盲区，更倾向猜测作答。研究人员 Laura Manley 指出，男性群体对 AI 普遍表现更高乐观度，部分演变为过度自信。  
  
  
该现象对网络安全领域具备现实警示意义，过度自信、认知虚高容易造成技术误判，在 AI 安全评估、威胁研判场景中，可能带来决策偏差风险。  
  
  
原文链接：  
  
https://www.wired.com/story/men-cant-stop-faking-how-much-they-know-about-ai/[#intcid]()  
=a9ca26b0-ae7a-461f-9012-6f615e7f33a0  
  
  
**CISA 呼吁 AI 实验室公开测试网络 IP 与 ASN，以识别逃逸的恶意 AI 智能体**  
  
10 月 13 日，在美国 Rubrik 公共部门峰会上，美国网络安全和基础设施安全局（CISA）高级技术顾问 Jonathan "Jono" Spring 提出倡议，呼吁 AI 测试实验室对外公开测试网络的 IP 地址与自治系统编号（ASN），帮助网络防御人员识别逃逸出测试环境的 rogue agents（恶意智能体），在其损害关键基础设施前完成处置。  
  
  
该构想延续网络安全领域长期奉行的 “睦邻协作” 实践，外部安全团队监测到异常流量后可及时通知 AI 实验室关停风险行为。但行业分析师观点分化，Forrester 的 Jeff Pollard 认为该思路具备现实基础，不过自愿模式很难普及，需监管或客户提出强制要求。Gartner 分析师 Dennis Xu 提出质疑，AI 实验室大量使用公有云，IP 常归属云服务商，难以精准溯源；未来大量 AI 智能体广泛部署后，IP 与 ASN 追踪模式或将失效，建议转向监控智能体思维链等逻辑层信号。  
  
  
Rubrik 则参考 EICAR 测试文件的思路，提议开发专属检测特征，搭建全球预警响应体系。Spring 同时建议公共机构将 AI 智能体当作攻击方开展安全测评，落实风险驱动补丁策略，高风险漏洞需在三日内完成修复。  
  
  
原文链接：  
  
https://www.techtarget.com/cybersecurity/news/366651845/CISA-to-AI-labs-Share-test-network-IPs-to-flag-rogue-agents  
  
  
  
安全事件  
  
  
**OpenAI 发生内部窃听事件：员工借助聊天机器人策划部署录音设备**  
  
OpenAI 工程师 Abdulrahman Alfozan 涉嫌在公司高管会议室部署隐蔽录音设备，该会议室用于讨论 AI 研发、战略规划等高敏感信息，涉案人员已遭逮捕并被公司解雇。  
  
  
法庭文件显示，该嫌疑人借助 AI 聊天机器人策划窃听行动，除使用 OpenAI 内部工具查询会议室日程，还使用与其配偶关联的聊天机器人账号，咨询录音设备组装、隐蔽部署、加密存储等技术问题，相关提问记录包含窃听设备的硬件标识信息。录音设备被卫生纸包裹，藏匿于会议室电视后方，安全团队还在嫌疑人包内查获另一台未投入使用的录音装置。  
  
  
OpenAI 利用自有 AI 模型完成溯源锁定嫌疑人，随即移交 FBI 调查。嫌疑人面临拦截口头通信的刑事指控，若罪名成立最高可判处五年监禁。公司称暂未发现其窃取模型权重、源代码与客户数据的证据。  
  
  
该案件凸显新的内部威胁场景：AI 工具可辅助攻击者策划物理窃听，对企业内部安全管控、AI 会话日志审计提出新挑战。  
  
  
原文链接：  
  
https://www.wired.com/story/an-openai-employee-allegedly-used-a-chatbot-to-help-plant-a-recording-device-at-the-office/  
  
  
**NGINX 发布新版本修复 5 处漏洞，含潜在 RCE 与 TLS 证书校验绕过缺陷**  
  
10 月 14 日，NGINX 发布 1.30.6 稳定版与 1.31.7 主线版，修复 5 个安全漏洞，目前暂无公开在野利用记录。  
  
  
其中 3 个为内存类漏洞（CVE-2026-15176、CVE-2026-93479、CVE-2026-97205），CVSS v4.0 评分为 9.2 分，可造成工作进程崩溃，极端条件下存在远程代码执行（RCE）风险，触发均依赖特定配置，分别涉及 alias+rewrite、正则 map、aio 与 SSI 组合场景。  
  
  
CVE-2026-97629 为 TLS 会话票据密钥旋转缺陷，启用共享 TLS 会话缓存且未手动加载密钥时可出现会话伪造，存在非确定性绕过客户端证书校验风险，CVSS v4.0 评分 8.3。  
  
  
CVE-2026-95101 属于 HTTP/3 解析漏洞，默认配置下可向后端注入额外请求字节，CVSS v4.0 评分 6.9。  
  
  
全部漏洞均需要开启对应指令才会受影响，管理员可通过nginx -T检查运行配置。官方强烈建议升级版本；无法升级可采用文档给出临时规避方案。同期 njs 1.0.2 也修复两处 XML 处理中危漏洞。  
  
  
原文链接：  
  
https://thehackernews.com/2026/10/nginx-fixes-5-flaws-including-possible.html  
  
  
**微软联合多方瓦解 CoGUI 钓鱼套件生态，打掉 YAM Store 与 Iron Man 犯罪平台**  
  
10 月 14 日，微软联合 Proofpoint、HealthISAC 及多国机构，依据美国法院裁定开展联合行动，瓦解 CoGUI 钓鱼套件、YAM Store 黑市、Iron Man 钓鱼平台组成的跨国黑产链条。  
  
  
调查由日本警方线索发起，该黑产自 2026 年 1 月起，产出超 10 亿条诈骗消息，波及全球 75000 家机构，非法获利达数千万美元。YAM Store 倒卖劫持的过期域名、伪造 Microsoft 365 与 Google Workspace 账号；攻击者搭配 CoGUI、Iron Man 实施钓鱼，Iron Man 内置 2300 余套仿银行、物流、政府的钓鱼模板，覆盖 150 个国家。  
  
  
CoGUI、CoAceV、CoFish_ws 属于同源工具家族，具备沙箱逃逸、欺骗自动化检测的能力，共享大量域名基础设施。本次行动查封恶意域名，封禁 20000 余个关联账号，拦截超 10 万个欺诈域名。  
  
  
安全团队提示，该套件可绕过传统 URL 过滤，企业应部署抗钓鱼多因素认证，关注自身过期域名归属风险。  
  
  
原文链接：  
  
https://securityonline.info/cogui-phishing-kit-takedown/  
  
  
  
安全攻防  
  
  
**新型 Android 木马 Hirudo 剖析：以替换应用方式绕过本地安全校验**  
  
ThreatFabric 披露一款名为 Hirudo 的新型 Android 银行木马，该恶意程序不采用叠加虚假登录界面的传统手段，而是卸载正版银行应用，植入篡改后的版本，从内部瓦解银行客户端安全防护，实施金融欺诈。  
  
  
该木马监控 42 款金融应用，已为印尼 7 家银行的 8 款 App 制作专属替换包，攻击范围覆盖印尼、中国、印度、墨西哥、摩洛哥、泰国用户。恶意载荷伪装成购物、短剧、影音类投放器（dropper）分发，借助 WebView 展示正常内容迷惑用户，诱导用户授予无障碍、通知权限，通过伪造更新弹窗完成攻击流程。  
  
  
篡改后的应用保留原包名与版本号，仅新增一行代码，加载 YAHFA、Dobby 钩子框架。触发后伪造证书、无障碍服务列表返回结果，解除截屏限制，并通过攻击者服务器中继 Play Integrity 校验结果，令客户端本地安全信号全部不可信。  
  
  
攻击者借助 Amazon CloudFront 下发恶意程序，其后台曾未设身份认证，泄露大量被盗账号、设备、收入风险档案等数据。研究提示，银行不可仅依赖设备本地校验结果，必须强化服务端风险判定，本地返回的签名、完整性校验等信息仅可作为参考依据。  
  
  
原文链接：  
  
https://securityaffairs.com/200970/malware/hirudo-android-trojan-replaces-banking-apps-to-steal-credentials.html  
  
  
  
产业动态  
  
  
**Agentic AI 重塑软件采购逻辑：席位与用量混合定价考验 CIO 预算管理**  
  
2026 年 10 月 15 日 TechTarget 发布分析，Agentic AI 正在颠覆企业传统软件成本核算模式。过去企业主要按照员工数量采购席位许可，如今厂商普遍采用席位、积分、动作次数、用量计费相叠加的混合定价模式。  
  
  
微软将 Copilot 固定用户订阅与高级 Agent 任务按量计费相结合，Agent 365 按用户而非 Agent 授权；Salesforce、SAP、Anthropic 也陆续推出类似混合方案。Gartner 指出，席位并未消失，而是和执行能力解耦，席位更多代表人类管控、编排与审批权限，Agent 执行工作量单独计费。  
  
  
专家提示，不应单纯对比单位标价，要评估完成业务工作流的实际综合成本。相同 Agent 在理想流程与复杂企业真实环境中开销差异巨大，重试、工具调用、人工介入都会抬高成本。  
  
  
预算预测不能再仅参考员工规模，还需要纳入工作流数量、执行频率、异常报错率等变量。建议 IT 部门管控平台与风险策略，各业务单元承担自身 Agent 产生的可变消耗成本。  
  
  
原文链接：  
  
https://www.techtarget.com/it-strategy/news/366652139/AI-agents-are-changing-the-math-on-software-spending  
  
  
  
新品发布  
  
  
**谷歌跳过 Gemini3.5，Gemini4 Argon 正式发布强化网络安全防御能力**  
  
继 Meta 发布 Muse Wonder 模型后，谷歌正式推出新一代基础大模型 Gemini4 Argon，该产品直接跳过原定 Gemini3.5 Pro 版本完成迭代升级。  
  
  
该模型早期仅对 Fairwind 计划参与国政府、具备高级网络安全防御需求的可信合作伙伴开放，现已接入 Gemini API 与智能体开发平台 Antigravity，普通用户可通过每月 20 美元 Google AI Pro、每月 100 美元 Ultra 订阅使用该能力。  
  
  
在能力层面，Gemini4 Argon 擅长处理跨学科复杂任务，覆盖金融分析、大规模软件工程、代码生成、网络安全防御等场景，在 Intelligence Index 基准评测中性能对标 OpenAI 的 GPT-6 Astra。API 限时促销至 11 月 30 日，输入 token 每百万 2 美元、输出每百万 10 美元，约为 GPT-6 Astra 六成成本；促销结束后恢复标准定价，输入每百万 4 美元、输出每百万 20 美元。  
  
  
消费端体验得到优化，模型可边思考边输出结果，减少冗余话术。谷歌还将 Antigravity 智能体编码平台内置进 macOS 端 Gemini App，直接对标 GitHub Copilot。行业分析认为，激进定价与开发者工具链整合，是谷歌抢夺企业及开发者生态的核心策略  
  
  
原文链接：  
  
https://securityonline.info/google-gemini-4-argon/  
  
  
  
  
  
  
**联系我们**  
  
合作电话：18610811242  
  
合作微信：aqniu001  
  
联系邮箱：bd@aqniu.com  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wKeDC5RjIzHJCln6OrTpFNWl3agJog1dJuvXEOEcAfaCzUG5w0Jibibu2APmaicR4bdvhEl4PKxdibmXeIaB6cbPR9LLN4BBxp50777IPiboPicpY/640?wx_fmt=gif&from=appmsg "")  
  
  
