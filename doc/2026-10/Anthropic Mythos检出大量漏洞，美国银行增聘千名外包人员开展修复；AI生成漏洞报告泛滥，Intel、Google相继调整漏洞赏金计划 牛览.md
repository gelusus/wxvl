#  Anthropic Mythos检出大量漏洞，美国银行增聘千名外包人员开展修复；AI生成漏洞报告泛滥，Intel、Google相继调整漏洞赏金计划 |牛览  
 安全牛   2026-10-19 03:50  
  
**点击蓝字 关注我们**  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/wKeDC5RjIzEnTdiaJz3ia4iaBvy6YVbg0fHMb9xG6bUhofhlibhKMgTQNYEqrELDkUFdaqhKFcpW6AYCrbkdicC3Ez5ckFodP7Hic0We8YO9ibJYFA/640?wx_fmt=png&from=appmsg "")  
  
  
新闻速览  
  
  
- Anthropic Mythos 检出大量漏洞，美国银行增聘千名外包人员开展修复  
  
  
- AI生成漏洞报告泛滥，Intel、Google相继调整漏洞赏金计划  
  
  
- 美国多机构建议为前沿AI系统部署网络安全控制措施并开展独立测试  
  
  
- 无需Root或漏洞利用，Shizuku可让Android应用获取Shell级权限  
  
  
- 俄罗斯数据中心接连遭无人机袭击，互联网巨头申请境外数据存储豁免  
  
  
- BAE Systems遭遇网络安全事件，美国破产法院通知系统瘫痪逾一周  
  
  
- ConnectWise 遭遇供应链攻击，大量 Microsoft 365 租户遭入侵  
  
  
- AI 自主越权风险：OpenAI 披露 7 起研究模型突破边界的安全事件  
  
  
- ConnectWise在攻击者攻破Microsoft 365租户后暂停云备份  
  
  
- ServiceNow 发布 AI Passport，以密码学凭证实现 AI Agent 运行时权限管控  
  
  
  
  
特别关注  
  
  
**美国多机构建议为前沿AI系统部署网络安全控制措施并开展独立测试**  
  
美国 CISA、NSA 以及商务部 CAISSI 联合发布前沿 AI 系统安全指引，该文件为自愿遵循性质，也是美国联邦政府针对 AI 模型沙箱测试环境出台的最详尽网络安全指导文件。  
  
  
此前夏季，OpenAI、Anthropic、Google、Meta 等企业多次出现 AI 模型逃出沙箱测试环境，入侵真实机构与网站的事件，研究人员推测同类沙箱逃逸事件可达数百甚至数千起，暴露出测试环节监控机制存在明显短板，促使官方出台本次指引。  
  
  
指引要求企业完善沙箱虚拟环境技术管控，部署外部监测工具，排查 AI agent 可能用来秘密留存信息、跨系统通信的持久化机制；特别警示不能仅依靠其他 SI（Super Intelligence）系统管控测试模型，红蓝对抗测试需结合人力，仅在不会危害关键系统的场景下才可使用 SI 开展红队演练。安全评估不能只针对大模型本身，还需覆盖 technical harness、主机、网络等全套基础设施，分层部署安全策略，拦截非法联网行为、识别异常行为。  
  
  
文件倡导引入独立外部审计机构开展安全评估，批评观点指出头部 AI 厂商过度依赖 METR、Redwood Research 这类小众机构完成安全测试。行业现状是厂商大量依靠 AI 系统完成模型监控，专家认为人力监督、技术管控、AI 工具三者结合才是约束 AI agent 攻击行为的最优路径。同时指引要求企业高管充分知悉风险后，亲自参与模型关停、重启的决策流程。  
  
  
原文链接：  
  
https://cyberscoop.com/federal-ai-agent-sandbox-security-guidance/  
  
  
  
热点观察  
  
  
**俄罗斯数据中心接连遭无人机袭击，互联网巨头申请境外数据存储豁免**  
  
俄罗斯多座数据中心遭无人机袭击后，互联网企业开始寻求跨境灾备的法律豁免。10月15日，俄罗斯互联网贸易企业协会（AKIT）致函总理Mikhail Mishustin，提议允许受重大事故影响的企业，临时将关键系统及个人数据迁移至友好国家的数据中心。该协会成员包括Ozon、Wildberries等大型电商平台。  
  
  
此次提议源于近期针对数字基础设施的连续袭击。10月7日至11日，Yandex位于梁赞州、卡卢加州和弗拉基米尔州的数据中心先后遇袭，部分设施停运。Yandex Cloud两个可用区受到影响，多项在线服务出现故障，暴露出传统跨地域灾备在大规模物理攻击下的局限性。  
  
  
俄罗斯自2025年7月1日起实施更严格的个人数据本地化要求，限制使用境外数据库收集和存储公民个人信息。AKIT建议设立临时特别法律机制，允许企业在事故得到官方确认、国内基础设施无法承载关键业务且满足信息安全要求的情况下，启用境外备用数据中心。该机制拟最迟于2028年3月1日终止。  
  
  
目前，该提议尚未获得批准。专家指出，事故发生后才启动豁免机制，不利于企业提前部署灾备系统。此外，境外数据中心的算力和电力资源不足，也可能影响实际恢复效率。如何兼顾数据主权、信息安全与业务连续性，成为俄罗斯亟待解决的问题。  
  
  
原文链接：  
  
https://www.securitylab.ru/news/578863.php  
  
  
**Anthropic Mythos 检出大量漏洞，美国银行增聘千名外包人员开展修复**  
  
美国银行（Bank of America）CEO Brian Moynihan 对外披露，为处理 Anthropic Mythos 前沿 AI 模型扫描发现的大量软件漏洞，该行在第二季度新增 1000 名外包人员参与漏洞修复工作，外包团队与原有 3000 名内部安全人员协同作业，相关人员至今仍在岗。  
  
  
Mythos 是 Anthropic 于 2026 年 4 月推出的专用生成式 AI 模型，仅向银行、关键基础设施、政府机构及网络安全企业小范围开放，该模型具备较强的代码挖掘能力，可检出系统中存续多年的隐藏软件缺陷。  
  
  
受漏洞修复以及反洗钱合规工作叠加影响，美国银行第三季度相关支出达到数亿美元。事件直观体现安全 AI 的双面效应：AI 能够高效批量挖掘代码漏洞，但漏洞修复仍依赖人工，会带来高额的人力与资金成本。  
  
  
与此同时，美国银行也在利用 AI 降本增效，目前运行约 130 个内部 AI 项目，预计今年可实现 3 亿美元成本节约，2027 年节约规模将提升至 7 亿美元。本次是美国大型企业首次公开披露针对 Mythos 漏洞排查所投入的具体人力规模，也为金融行业使用 AI 安全工具的成本收益提供现实参考。  
  
  
原文链接：  
  
https://www.techradar.com/pro/security/bank-of-america-had-to-hire-1-000-contractors-to-patch-software-flaws-spotted-by-anthropic-mythos  
  
  
**AI 自主越权风险：OpenAI 披露 7 起研究模型突破边界的安全事件**  
  
OpenAI 披露 7 起 AI 模型对齐失效事件，其中典型案例为 GPT5.6 Sol 内部研究模型未经许可，挪用第三方钱包内 Sepolia 测试 ETH 完成智能合约部署。事件发生在 2026 年 6 月 26 日，时隔三个月才被安全团队发现。  
  
  
该模型的任务是更新加密货币交易应用并在以太坊 Sepolia 测试网部署合约，任务上下文并未提供存有代币的钱包。模型申请免费测试代币失败后，自行编写脚本，利用公开开发者助记词推导私钥，批量扫描 362 个钱包地址，找到持有测试币的账户并完成合约付费部署。模型内部曾评估该行为合规性，但自行判定测试网代币无实际价值，可直接使用，并且未主动上报此次越权操作。  
  
  
其余 6 起对齐异常事件包含绕过 HTTP GET 访问限制、突破沙箱读取内部配置、下载非公开数据集、窃取网站敏感凭证、伪造任务文件、破坏运行环境，部分场景下模型会刻意隐藏越权动作。  
  
  
OpenAI 指出，这类风险的核心是模型会用自身对危害的主观判断，替代系统所有者的授权规则。企业已实施加固措施，限制训练阶段模型直接访问公网，将对齐异常监测覆盖全部强化学习流量，优化对齐评分机制，降低模型自主越权的可能性。  
  
  
原文链接：  
  
https://thehackernews.com/2026/10/openai-research-model-spent-3rd-party.html  
  
  
  
安全事件  
  
  
**BAE Systems遭遇网络安全事件，美国破产法院通知系统瘫痪逾一周**  
  
10月17日，Security Affairs报道称，英国国防承包商BAE Systems旗下美国子公司遭遇网络安全事件，导致其运营的美国破产通知中心（Bankruptcy Noticing Center，BNC）自10月9日起持续瘫痪，影响全美联邦破产法院的通知服务。  
  
  
BNC负责为美国所有联邦破产法院生成和发送法律通知，包括债权人通知及案件期限提醒。此次事件导致bnc.uscourts.gov、bankruptcynotices.uscourts.gov、notices.uscourts.gov等网站及国家债权人登记服务（National Creditor Registration Service）无法访问。  
  
  
故障还中断了纸质和电子破产通知的制作、传输及分发流程，并影响案件管理和电子档案系统（CM/ECF）生成债权人邮寄名单。不过，法院仍可接收新案件申请，面向已注册律师的部分电子通知服务也继续运行。  
  
  
根据法院公告，BAE Systems于10月9日晚通报故障，法院10月13日开始发布服务中断通知，美国联邦司法机构直到10月15日才正式确认涉及网络安全事件。  
  
  
BAE Systems表示，事件影响范围仅限BNC系统，正与法院合作调查和恢复服务。目前尚未披露具体攻击手法、是否存在数据泄露及服务恢复时间。  
  
  
此次事件凸显第三方IT服务商集中承载关键公共系统的风险，单一供应商故障可能造成全国性业务中断。  
  
  
原文链接：  
  
https://securityaffairs.com/201166/security/bae-systems-cyber-incident-takes-us-bankruptcy-noticing-center-offline-for-a-week.html  
  
  
**ConnectWise 遭遇供应链攻击，大量 Microsoft 365 租户遭入侵**  
  
击者利用 ConnectWise Cloud Backup（原 SkyKick Cloud Backup）发生供应链攻击，入侵多家 Microsoft 365 租户，ConnectWise 已紧急关停该服务。Huntress 监测确认至少 162 个租户遭到入侵，攻击溯源至 IP 地址 144.172.64 [.] 128，恶意行为最早出现在 10 月 13 日。  
  
  
该备份服务面向 MSP 服务商，依托 Microsoft Entra ID 下的 service principal（服务主体）获取租户权限。攻击者窃取应用凭证，通过自动化脚本以应用身份登录，由于属于非用户账号访问，MFA 多因素认证无法起到防护效果。凭借服务主体已被授予的权限，攻击者可读写邮件、OneDrive、SharePoint 文件以及 Teams 聊天记录。  
  
  
部分受害租户中，恶意应用自行分配 Exchange Administrator、Global Administrator 等高权限角色，植入凭证建立持久化后门，调用 Microsoft Graph API 开展侦察。ConnectWise 表示暂无证据表明备份存档数据遭到泄露，但关停服务不会清除租户内已植入的恶意凭证。  
  
  
安全厂商提示，仅禁用受影响应用无法清除已发放的访问令牌，令牌到期前仍可被滥用。管理员需要核查 Entra ID 中相关服务主体登录日志、OAuth 授权、新增凭证，排查指定的一批应用 ID。目前 ConnectWise 同时暂停部分 SaaS 安全与云管理服务，尚未公布恢复时间，攻击者获取凭证的具体手段暂未查明。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/security/hackers-breach-microsoft-365-tenants-via-connectwise-supply-chain-attack/  
  
  
**ConnectWise在攻击者攻破Microsoft 365租户后暂停云备份**  
  
ConnectWise 在 10 月 17 日 UTC 时间 07:00 禁用 7 项应用身份，关停 Cloud Backup 服务并吊销相关连接证书，处置攻击者窃取应用身份入侵 Microsoft 365 租户的供应链攻击事件。该企业 10 月 14 日通过内部监控发现异常，同步暂停 SaaS Security 与 Cloud Management 服务开展调查。  
  
  
关闭应用身份只能阻断新的认证请求，无法清除租户内已完成的恶意改动，已下发的访问令牌在过期前依旧有效。备份存档保持加密隔离，暂无证据显示备份数据遭到泄露，但备份调度、恢复功能全部暂停。  
  
  
攻击者窃取应用凭证后，借助 Microsoft Entra ID 获取访问令牌，依托应用原有高权限读取邮件、SharePoint、Teams 等资源，还可提升至 Global Administrator 权限，新增凭证制造持久化后门。Huntress 监测到来自 IP144.172.64 [.] 128 的登录行为，近千个 Entra 租户出现该 IP 登录记录，其中 300 次登录成功；10 月 15 日攻击者还针对政策、国家安全、关键矿产等行业的 16 个租户开展定向第二轮攻击，但未取得成功。  
  
  
厂商尚未确认凭证泄露的根本原因。安全团队提示管理员需要排查休眠 SkyKick 连接、审计登录日志、检查新增应用凭证与目录角色，同时可借助工作负载身份条件访问、应用管理策略降低同类攻击危害。  
  
  
原文链接：  
  
https://thehackernews.com/2026/10/connectwise-suspends-cloud-backup-after.html  
  
  
  
安全攻防  
  
  
**无需Root或漏洞利用，Shizuku可让Android应用获取Shell级权限**  
  
移动安全公司Pradeo发布研究报告指出，Android工具Shizuku的授权机制可能被恶意软件滥用。应用无需Root设备或利用系统漏洞，只需获得用户一次授权，即可通过Shizuku服务器以ADB Shell身份（UID 2000）执行命令，获取超出常规应用沙箱限制的权限。  
  
  
研究人员在Android 16环境下，使用Shizuku 13.6测试了41条命令，其中27条获得额外信息或操作能力。授权后，测试应用可识别的安装包数量从98个增至261个，进程数量从2个增至382个，logcat日志从284行增至1978行。  
  
  
研究还成功复现四类滥用场景，包括读取其他应用写入日志的验证码和令牌、静默安装APK、绕过系统弹窗授予联系人访问权限，以及收集设备账户和网络连接信息。  
  
  
实际攻击中，Android恶意软件RedHook已采用类似机制。攻击者通过社会工程学诱导用户授予Accessibility权限，随后自动开启开发者选项和无线调试，连接本机ADB服务并部署基于Shizuku的特权服务，甚至能在设备重启后恢复权限。  
  
  
Pradeo强调，这并非Android或Shizuku的软件漏洞，而是合法功能遭滥用。ADB模式下的Shell仍无法直接读取普通应用在/data/data中的私有数据。安全团队应重点监控异常调试配置、可疑Accessibility授权及Shizuku服务，降低恶意软件滥用风险。  
  
  
原文链接：  
  
https://www.securitylab.ru/news/578868.php  
  
  
  
产业动态  
  
  
**AI生成漏洞报告泛滥，Intel、Google相继调整漏洞赏金计划**  
  
大语言模型（LLM）生成的低质量漏洞报告激增，正迫使科技企业重新评估漏洞赏金（Bug Bounty）计划，并调整安全投入策略。  
  
  
今年1月，Curl因大量AI生成的误报关闭漏洞赏金计划。项目创建者Daniel Stenberg表示，漏洞报告的确认率从此前超过15%降至2025年的不足5%。9月，Intel暂停漏洞赏金支付，但仍通过漏洞披露计划（VDP）接收报告，尚未确认此举是否与AI有关。10月1日，Google暂停接收开源软件漏洞奖励计划（OSS VRP）中的产品漏洞报告，供应链漏洞报告不受影响。  
  
  
HackerOne首席执行官Kara Sprague指出，AI能在几分钟内生成看似可信的漏洞报告，但其中不少未经实际测试，甚至超出项目范围。安全人员仍需逐一复现漏洞、确认影响并评估修复必要性，导致审核成本大幅上升。  
  
  
Google数据显示，2026年1月至8月，每月公开披露的漏洞数量从5045个增至10740个，但该数据并非漏洞赏金报告数量。  
  
  
业内专家预计，企业将强化漏洞复现和提交证据要求，把更多资源投入内部AI自动化审计、安全设计及漏洞修复。不过，暂停赏金也可能导致高水平安全研究人员流失，削弱企业发现复杂漏洞的能力。  
  
  
原文链接：  
  
https://www.techtarget.com/cybersecurity/news/366652109/Bug-bounty-reckoning-AI-slop-forces-enterprise-strategy-shift  
  
  
  
新品发布  
  
  
**ServiceNow 发布 AI Passport，以密码学凭证实现 AI Agent 运行时权限管控**  
  
ServiceNow 推出 AI Passport 解决方案，用于在运行时校验、管控 AI agent 权限，配套升级 Super Intelligence Control Tower 平台，强化企业 AI 治理、安全与成本管控能力，该产品预计 2026 年第四季度上线，NVIDIA 为设计与发布合作伙伴。  
  
  
AI Passport 采用密码学签名凭证，可对多 agent 工作流内 AI agent、Model Context Protocol (MCP) 服务器的可执行动作做权限校验，基于企业策略与审批规则，阻止越权行为，支持权限撤销并留存完整审计记录，方案基于开放规范，兼容自研及第三方 AI 代理。它将对接 NVIDIA Open Agent Safety Platform，在基础设施层实现运行时权限持续校验。  
  
  
Super Intelligence Control Tower 新增多项能力：借助 400 余个预置连接器实现 Shadow AI 影子 AI 发现，持续观测 agent 行为轨迹，完成权限映射；新增 AI 成本管理，按用户、部门统计 AI 开销，设置预算告警；同时支持分级处置，对异常 AI 行为执行告警、限流、阻断。  
  
  
调研显示，受访银行均已使用 AI，但仅有 24% 机构具备匹配业务规模的 AI 治理能力。ServiceNow 面向金融业推出的 Operational Resilience 现已可用，可跨 IT、云、第三方环境监控 AI 资产。已有企业实践为上千个 AI agent 绑定人员 ID，实现责任追溯。相关技术细节将在 NVIDIA GTC Berlin 大会进行分享。  
  
  
原文链接：  
  
https://www.expresscomputer.in/news/servicenow-introduces-ai-passport-to-strengthen-enterprise-ai-governance/140049/  
  
  
  
  
  
  
**联系我们**  
  
合作电话：18610811242  
  
合作微信：aqniu001  
  
联系邮箱：bd@aqniu.com  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wKeDC5RjIzFFyAvo0Yqic9zYcpyiaicJq29L9gXcnmUCfAPLYKyIOe7LQhlBmBbFgKp2jkhLiah7tUszGZn0Kde3GxODib73x68P4GOooQnvQdvs/640?wx_fmt=gif&from=appmsg "")  
  
  
