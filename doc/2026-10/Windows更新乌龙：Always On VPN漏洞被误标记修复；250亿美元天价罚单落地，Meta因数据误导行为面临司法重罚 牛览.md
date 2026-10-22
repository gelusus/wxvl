#  Windows更新乌龙：Always On VPN漏洞被误标记修复；250亿美元天价罚单落地，Meta因数据误导行为面临司法重罚| 牛览  
 安全牛   2026-10-22 03:52  
  
**点击蓝字 关注我们**  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/wKeDC5RjIzGAPhiaQYFrHvY2WzCO1l4z3WLJyRHKv3oAVUJuIJYZ6V0pDDXawKEayhON8o4OFjt5sx2BpuC4HxkKxYia8vqYupgibseHzibwy9Q/640?wx_fmt=png&from=appmsg "")  
  
  
新闻速览  
  
  
- Windows 更新乌龙：Always On VPN 漏洞被误标记修复，企业需采用临时规避方案  
  
  
- 绿盟科技2026 TechWorld智慧安全大会在京召开  
  
  
- CISA 联合多机构预警：SafePay 勒索软件已攻击超 600 家关键基础设施机构  
  
  
- 诱饵式防御：Context Bombs 误导 AI 攻击代理构建新型 AI 安全防护手段  
  
  
- Moody’s 警示：AI 驱动网络威胁，薄弱防护将抬升企业信贷成本  
  
  
- Chrome 引入 Gemini 服务器端检测，新增ClickFix 攻击剪贴板防护能力  
  
  
- 250 亿美元天价罚单落地，Meta 因数据误导行为面临司法重罚并拟上诉  
  
  
- Blackstone 高管：AI 基建债务风险被夸大，与次贷危机风险存在本质区别  
  
  
- 日本 VPS 平台被入侵，最高 26739 台客户服务器存在失陷风险  
  
  
- Arrakis Security 报告：76% AI 代理长期休眠，但凭据与触发机制仍保持有效  
  
  
- 英国CMA拟要求Apple开放iPhone NFC支付，降低接入费用  
  
  
  
  
特别关注  
  
  
**Windows 更新乌龙：Always On VPN 漏洞被误标记修复，企业需采用临时规避方案**  
  
2026 年 10 月 21 日消息，Microsoft 向 IT 管理员发布服务警报，承认 Windows11 十月累积更新并未修复 Always On VPN 的连接故障，此前错误将该漏洞状态标记为已修复，现已改回 “已缓解”，正式补丁将在未来数周推出。  
  
  
该问题源于 2026 年 9 月安全更新，影响 Windows11 24H2 及更高版本设备。当 Always On VPN 启用 IKEv2、SSTP 自动协议选择与故障转移功能时，首次连接失败后，VPN 会卡在 “正在连接” 状态，或出现 “指定端口已在使用” 报错，无法建立企业加密隧道。Always On VPN 是替代 DirectAccess 的企业远程访问组件，面向域加入、Microsoft Entra ID 加入设备，在后台自动建立内网隧道。  
  
  
官方给出临时处置方案，管理员需将 VPN 配置从协议自动选择，修改为仅使用 SSTP 或者仅使用 IKEv2 单一协议，根据业务环境选型。9 月补丁还引发 Hyper-V、远程桌面、USB 音频等多项故障，Microsoft 曾发布紧急带外更新处理部分问题。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/microsoft/microsoft-didnt-actually-fix-vpn-issues-caused-by-windows-updates/  
  
  
  
热点观察  
  
  
**CISA 联合多机构预警：SafePay 勒索软件已攻击超 600 家关键基础设施机构**  
  
美国 CISA 联合 FBI、美国特勤局、德英两国执法机构发布联合报告，自 2024 年 8 月起，SafePay 勒索软件团伙已入侵全球超过 600 家关键基础设施机构，受害范围覆盖北美、南美、欧洲、澳大利亚多地。  
  
  
SafePay 诞生于 2024 年 8 月，不属于 RaaS（勒索即服务）模式，为独立运作团伙，现已填补 BlackCat（ALPHV）、LockBit 退场所留下的威胁缺口，是当前最活跃的勒索团伙之一。该团伙采用双重勒索策略，加密系统前窃取敏感数据，以泄露数据要挟受害者支付赎金，同时会删除卷影副本、重置 NAS 备份设备，破坏受害方恢复能力。  
  
  
攻击者初始入侵手段包括凭证泄露、暴力破解、利用 VPN（重点针对 Mullvad 的 IP）、钓鱼账号，以及借助自动化脚本利用互联网暴露设备的 CVEs 漏洞。报告附带 IOCs 与 TTPs 威胁情报。  
  
  
机构提示，支付赎金无法保证文件恢复，建议限制管理员权限阻断横向移动，部署 MFA、落地 ICAM 身份访问管理策略，及时修补 VPN、防火墙等边缘设备的高危漏洞，并建议受害者向执法部门上报安全事件。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/security/cisa-and-fbi-safepay-ransomware-has-hit-over-600-critical-infrastructure-organizations/  
  
  
**Blackstone 高管：AI 基建债务风险被夸大，与次贷危机风险存在本质区别**  
  
2026 年 10 月 21 日，在 TechCrunch Disrupt 大会上，私募巨头 Blackstone 高管 Jas Khaira 针对市场热议的 AI 基建债务风险公开发声，认为市场对于 AI 算力扩建带来的债务危机担忧被过度放大。  
  
  
当前行业大量举债建设 AI 数据中心，部分算力企业债务规模高企，CoreWeave 股价出现约 20% 回撤，Firmus 也被迫取消 IPO 计划，叠加高利率环境，不少投资者担忧 AI 建设热潮会诱发新一轮信贷风险。  
  
  
Jas Khaira 指出，AI 基础设施债务和 2008 年次贷危机的劣质银行信贷并不相同。这类债务底层有大型科技企业长期算力采购合同作为支撑，资产背靠高信用等级客户，具备稳定现金流基础，风险可控。  
  
  
Blackstone 管理资产规模达 1.3 万亿美元，是公有云之外全球规模靠前的算力资产持有方，持续向数据中心、算力服务商提供大额融资，包括向 CoreWeave 等企业输出资金支持。同时他也承认，利率上行确实给 AI 基建行业带来现实压力，但 AI 模型对算力的刚性需求仍在持续上涨，足以支撑相关资产的商业逻辑。  
  
  
原文链接：  
  
https://techcrunch.com/2026/10/21/blackstones-jas-khaira-says-fears-over-ai-buildout-debt-are-overblown/  
  
  
**Arrakis Security 报告：76% AI 代理长期休眠，但凭据与触发机制仍保持有效**  
  
2026 年 10 月 21 日，Arrakis Security 发布《2026 Autonomous Agents Threat Report》，基于 12 周内数千条企业环境 AI 代理匿名生产遥测数据，揭示企业 AI 代理存在突出权限治理风险。  
  
  
报告显示，76% 的 AI 代理已有 60 天未执行任务，但凭据依旧有效、触发条件仍可生效；44% 代理闲置时长达到 180 天；24% 休眠代理可被外部无签名 webhook 直接启动。与人员离职会立刻回收权限不同，闲置 AI 代理普遍缺少负责人，权限无人清理。  
  
  
报告还披露多项风险数据，70% 代理能够访问云存储等敏感系统，且几乎没有部署经过验证的数据防泄露（DLP）能力；82% 代理至少存在一项安全暴露点，其中 46% 存在多项风险；22% 代理可向第三方模型提交内容并执行返回结果，20% 代理可跨不同权限代理传递任务。该报告指出，企业普遍缺少对 AI 代理生命周期的安全管控。  
  
  
原文链接：  
  
https://securityonline.info/arrakis-security-report-finds-76-of-ai-agents-are-dormant-but-keep-their-credentials-and-connections/  
  
  
  
安全事件  
  
  
**250 亿美元天价罚单落地，Meta 因数据误导行为面临司法重罚并拟上诉**  
  
2026 年 10 月 21 日，美国新墨西哥州法院公布判决文书，Meta 因剑桥分析（Cambridge Analytica）相关隐私违规，被裁定缴纳 250 亿美元民事罚款，逾期未缴将按 15% 年利率计收利息。  
  
  
该案源自 2018 年曝光的丑闻，Facebook 曾向剑桥分析泄露多达 8700 万用户数据，该机构利用数据构建选民心理画像，用于 2016 年美国总统大选定向广告投放。今年 9 月，陪审团认定 Meta 违反该州不公平行为法，违规次数达 4300 万次，单次违规最高处罚 5000 美元，最初判罚金额高达 2200 亿美元，法官 Francis Mathew 最终将罚金下调至 250 亿美元。  
  
  
判决文书指出，以 Mark Zuckerberg、Sheryl Sandberg 为代表的企业高层制定策略对外误导公众，内部已经确认无法兑现数据管控承诺，却继续对外宣传用户数据保护、第三方数据治理等相关承诺，承诺开展的全面调查也因记录管理、第三方开发者审核缺陷沦为形式。  
  
  
Meta 首席法务官 C.J. Mahoney 表示该裁决近乎让企业无法在该州开展业务，称诉讼实质是对网络言论征税，公司计划提起上诉。此前 Meta 已因剑桥分析事件，经历多轮和解与处罚。  
  
  
原文链接：  
  
https://therecord.media/meta-new-mexico-penalty-cambridge-analytica-privacy-case  
  
  
**日本 VPS 平台被入侵，最高 26739 台客户服务器存在失陷风险**  
  
2026 年 10 月 21 日消息，日本 GMO Internet 集团旗下 VPS 服务 Onamae Desktop Cloud 遭到黑客入侵，最多 26739 台客户服务器被植入恶意程序，平台已启动全服务关停处置。  
  
  
该平台主打面向 MetaTrader 4（MT4）外汇自动化交易场景的云主机服务。企业于 10 月 20 日确认发生第三方未授权访问，黑客在部分租户服务器环境部署恶意软件。为遏制威胁扩散，当地时间 10 月 21 日凌晨 4 时，服务商开始逐步下线全部套餐的服务器，目前尚未公布服务恢复时间。  
  
  
官方提示，受影响服务器内存储数据存在泄露风险，但暂未核实到数据外泄、滥用的实际案例；信用卡等支付信息存储于独立基础设施，未遭受波及。平台提醒用户尽快轮换服务器内保存的密码等密钥，同时警惕攻击者借本次事件发起钓鱼邮件诈骗。GMO Internet 将持续开展调查并对外更新事件进展。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/security/japans-onamae-desktop-cloud-hacked-up-to-27-000-servers-affected/  
  
  
**英国CMA拟要求Apple开放iPhone NFC支付，降低接入费用**  
  
2026 年 10 月 21 日，英国竞争与市场管理局（CMA）发布监管提案，拟强制苹果开放 iPhone 非接触支付技术，改善英国金融科技市场竞争环境。英国非接触交易占全部支付的 39%，金融科技行业带动 36 万余个岗位，但受苹果严苛收费与规则限制，当地至今没有第三方独立数字钱包落地。  
  
  
提案包含两项核心诉求，一是要求苹果支持 Host Card Emulation（HCE）这类国际通用非接触框架，实现技术层面的对等；二是确立合理定价原则，大幅下调开发者接入费用，保障第三方钱包具备商业生存空间。本次公众意见征询将持续至 2026 年 11 月 17 日。  
  
  
对比欧盟市场，欧盟依据《数字市场法案》，苹果已向开发者免费开放 NFC；而英国当前仅开放苹果自有 Secure Element 相关 API。苹果对此表示，封闭架构的出发点是保障用户隐私与交易安全，强制开放会削弱消费者防护能力。若该提案最终落地，或将对全球苹果设备 NFC 生态带来连锁影响。  
  
  
原文链接：  
  
https://securityonline.info/uk-cma-iphone-contactless-payments/  
  
  
  
安全攻防  
  
  
**诱饵式防御：Context Bombs 误导 AI 攻击代理构建新型 AI 安全防护手段**  
  
Tracebit 公司推出名为 Context Bombs（上下文炸弹）的防御方案，借鉴地雷阻滞敌方部队的思路，将经过构造的隐藏提示词部署在受保护目标环境中。当恶意 AI 代理爬取目标资源时，会自动读取这些提示，干扰攻击者原本的恶意意图，实现误导、困住攻击代理的防护效果。  
  
  
该研究共开发 9 项全新上下文欺骗技术。测试中，这套防护可完全阻断 GPT 5.6 Cyber、Kimi K3、Qwen 3.8 Max、DeepSeek V4 Pro 的攻击；对前沿大模型，GPT-6 Astra 攻击成功率下降 54%，单轮攻击成本接近翻四倍；Claude Opus 5 攻击成功率降幅达 91%。  
  
  
效果最优的是伪造渗透测试报告，它将真实攻击路径标记为诱捕标记，把虚假资源伪装成已被攻陷的高价值目标，令 AI 将真实系统判定为陷阱而主动规避，目标达成率下降 58%。其余手段包含无限循环 Web 服务、多层加密虚假密码、无限延展数据库等欺骗载体，测试中 GLM-5.3 曾爬取 825 页陷入圈套。将全部 9 种技术组合部署后，国内多款大模型均无法完成攻击目标。  
  
  
即便未能阻止攻击，上下文炸弹也可将单次成功攻击成本抬高 23 倍。成本上涨会削弱攻击者的行动意愿，同时异常的代理行为更容易触发安全告警，为防御方争取响应时间。该技术属于反向提示注入防御，内部预置的欺骗提示可覆盖外部攻击者注入的恶意指令，以此对抗 AI 代理带来的新型安全威胁。  
  
  
原文链接：  
  
https://www.securityweek.com/context-bombs-cut-ai-attack-success-by-misleading-agents-with-fake-targets/  
  
  
  
产业动态  
  
  
**绿盟科技2026 TechWorld智慧安全大会在京召开**  
  
10月21日，2026 TechWorld智慧安全大会在北京盛大召开。大会以“驭智・HARNESS——以智慧安全重构攻防新秩序”为主题，汇聚国家级科研机构负责人、高校学者、产业专家以及技术精英，聚焦AI驱动的网络安全范式变革，沿着“AI赋能安全”、“AI自身安全”双主线展开深度研讨，共论智能时代下发展与安全共生之道。  
  
  
AI技术正在重新定义安全的边界，把握变革的主动权，需要安全理念与技术能力同步演进。会上，绿盟科技重磅发布智慧安全4.0技术理念，并揭牌成立绿盟科技上海研究院，以理念创新把握前进方向，以前沿研究积蓄长远力量，推动安全能力与技术演进同频共振、与业务韧性深度适配，为发展新质生产力、迈向高质量发展筑起坚实的安全屏障。  
  
  
原文链接：https://mp.weixin.qq.com/s/I6SdeyIQwtfYTU9B2ERqyw  
  
  
**Moody’s 警示：AI 驱动网络威胁，薄弱防护将抬升企业信贷成本**  
  
Moody’s Ratings 发布报告指出，随着进攻性 AI 降低攻击门槛，网络防御能力薄弱的企业将面临更高借贷成本，这也为 CISO 向董事会争取安全预算提供金融层面的有力依据。  
  
  
受算力成本下降、具备前沿攻击能力的开源 AI 模型推动，自动化、持续性网络攻击持续增多。AI 辅助漏洞挖掘大幅压缩漏洞利用窗口期，企业补丁积压问题加剧。Moody’s 对约 9500 家受评机构分析显示，60% 企业网络存在至少一处未修复的已知在野利用漏洞，漏洞修复中位耗时接近三个月，但对抗 AI 威胁需要对外暴露资产在数天甚至数小时内完成修复。  
  
  
行业专家建议安全团队转向 VulnOps 漏洞运维模式，将漏洞管理转为 DevOps 式持续流程。对于老旧系统无法快速打补丁的企业，可采用网络分段、访问限制等补偿控制手段。专家同时提醒，企业需做好同时应对多起 AI 驱动安全事件的预案，开展红蓝对抗与桌面推演，提升整体业务韧性。  
  
  
安全负责人建议改变汇报思路，减少堆砌漏洞、补丁等技术指标，向董事会重点阐述业务营收系统的风险、业务中断时长、现存韧性短板，配套可量化风险指标，从财务损失视角传递网络安全风险。  
  
  
原文链接：  
  
https://www.techtarget.com/cybersecurity/news/366652364/AI-will-drive-up-credit-cost-of-weak-cyberdefenses-says-Moodys  
  
  
  
新品发布  
  
  
**Chrome 引入 Gemini 服务器端检测，新增 ClickFix 攻击剪贴板防护能力**  
  
2026 年 10 月 21 日，Google 为 Chrome 浏览器推出基于 Gemini 的新型诈骗检测能力，面向开启 “增强保护（Enhanced Protection）” 模式的用户，同时新增针对 ClickFix 攻击的剪贴板告警功能。  
  
  
ClickFix 属于社会工程攻击，诱骗用户复制并在 PowerShell、命令提示符中执行恶意指令，以此下发恶意程序，传统网页防护较难拦截该类威胁。  
  
  
本次更新将 Gemini 由本地模型迁移至算力更强的服务器端，模型会分析网页品牌信息、行为意图，结合域名信誉等安全信号综合判定诈骗风险。为保护隐私，浏览器仅在页面出现高风险诈骗特征时才上传网页文本，相关内容临时处理，不会留存或用于模型训练。  
  
  
Chrome 新增 “可疑网站” 提示，针对存在欺骗特征、但尚未达到高危红页警告置信度的站点进行预警，以此扩大风险覆盖范围。当用户复制疑似 ClickFix 恶意命令时，浏览器会剔除剪贴板内个人身份信息后提交分析，确认危险后向用户弹出拦截提醒。  
  
  
Google 表示，开启增强保护后，用户遭遇钓鱼、社会工程类诈骗的安全防护能力是标准保护模式的十倍。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/google/google-chrome-adds-gemini-powered-scam-detection-warns-users-of-clickfix-attacks/  
  
  
  
  
  
  
**联系我们**  
  
合作电话：18610811242  
  
合作微信：aqniu001  
  
联系邮箱：bd@aqniu.com  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wKeDC5RjIzGztHz1hVJd4nu8LMib9a1F7L2MRmibMzjGAHlZXD23rVVicDjot3icwhp9Uqmpb3k7ibnQf53DO7cmHm7ydvGd8M3RQEMero5hNFgI/640?wx_fmt=gif&from=appmsg "")  
  
  
  
