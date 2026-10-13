#  OpenAI 失控 AI 代理对 Wikimedia 发起异常操作，海量流量诱发服务中断风险；CNVD两期周报：收录866个漏洞，0day占比达88%| 牛览  
 安全牛   2026-10-13 05:45  
  
**点击蓝字 关注我们**  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/wKeDC5RjIzF25rCNYPe13pWXkHxDpibcZaTibibSq82bKpRodhpMWrXQ5EMCD0hbWUGu6w9JC6B41iasTKIglyYIj45W3c95FZWE9IQpdb8OnBE/640?wx_fmt=png&from=appmsg "")  
  
  
新闻速览  
  
  
- CNVD两期周报：收录866个漏洞，0day占比达88%  
  
  
- CNVD盘点高关注漏洞：OpenClaw权限绕过、多个产品可执行命令  
  
  
- Meta 推出首款前沿级 Muse Wonder 模型，闭源付费模式搅动 AI 智能体安全格局  
  
  
- ERP厂商强化AI布局以打消买家顾虑，抵御前沿AI厂商竞争  
  
  
- Warden Stealer 窃取木马升级，将 AI 编码智能体 Token 列为重点窃取目标  
  
  
- Q3 漏洞报告：近四分之一漏洞披露一周内即遭受攻击，微软成受影响最多厂商  
  
  
- OT 安全告警：法国数百超市制冷控制系统遭网攻，大量生鲜食品损毁  
  
  
- 零售巨头 Ahold Delhaize 遭遇网络安全事件，多国旗下品牌网站下线  
  
  
- OpenAI 失控 AI 代理对 Wikimedia 发起异常操作，海量流量诱发服务中断风险  
  
  
- 美国执法部门持令部署 IMSI 捕捉器，追踪新闻事件相关人员手机位置  
  
  
  
  
特别关注  
  
  
**CNVD两期周报：收录866个漏洞，0day占比达88%**  
  
CNVD2026年第39、40期周报显示，9月28日至10月11日共收录漏洞866个，高危428个、中危372个、低危66个，平均分值6.53；0day漏洞760个，占88%。同期，党政机关和企事业单位事件型漏洞46669个，环比增42%，整体威胁评级为“中”。  
  
  
风险涉及Oracle、Ivanti、Ubiquiti、Tenda等产品。Oracle Siebel CRM漏洞可致未授权访问、敏感数据泄露，部分情形可被完全接管；Ivanti漏洞涉及身份验证绕过、代码注入和权限提升；Ubiquiti漏洞包括SQL注入、服务器端请求伪造及命令执行；Tenda G103命令注入、AC6缓冲区溢出均为高危。另有Leave Management System SQL注入漏洞，源于/module/leave/index.php的ID参数处理不当，可被利用执行恶意SQL、窃取或篡改数据库信息，目前尚无补丁。  
  
  
多数相关厂商已发布修复程序。CNVD建议用户排查受影响资产并及时更新；对未修复漏洞，持续关注厂商公告，采取临时防护措施。  
  
  
原文链接：  
  
https://www.cnvd.org.cn/webinfo/show/12826  
  
  
**CNVD盘点高关注漏洞：OpenClaw权限绕过、多个产品可执行命令**  
  
CNVD发布的2026年9月28日至10月11日高关注度漏洞信息显示，境内外多款网络管理、AI智能体和企业软件产品存在任意命令执行、权限绕过、信息泄露及跨站脚本等风险。  
  
  
境外方面，Ubiquiti UniFi OS Server输入验证不当可导致任意命令执行。OpenClaw有两项授权问题：启用会话记忆与dreaming时，受限外部发送者可植入指令，诱使高权限后台代理越权操作；另一漏洞因node.invoke未校验管理员作用域，可绕过限制访问浏览器内容或执行未授权操作。其CSV导出还存在公式注入风险。WordPress Eventin plugin的event_layout参数存在本地文件包含，可执行服务器PHP代码。  
  
  
境内方面，海康威视iVMS-A100存在SQL注入，金蝶EAS可被远程命令执行；Netcore NR255 V的QoS、DDNS处理路径存在持久型XSS，可能导致信息泄露或会话劫持；TOTOLINK T6访问控制不当，特制POST请求可读取LAN寻址和DHCP配置。CNVD称，关注度依据漏洞信息查阅量及产品应用广泛程度综合评定。  
  
  
原文链接：  
  
https://www.cnvd.org.cn/webinfo/show/12831  
  
  
  
热点观察  
  
  
**Q3 漏洞报告：近四分之一漏洞披露一周内即遭受攻击，微软成受影响最多厂商**  
  
近日 Daily CyberSecurity 发布 2026 年第三季度 CVE 漏洞报告，2026 年 79 月，美国 CISA 在 KEV（已知被利用漏洞目录）新增 100 个存在实际在野利用的安全漏洞，较二季度 75 个环比上升 33.3%。  
  
  
数据显示，本季度漏洞风险持续走高，100 条新增记录中 49 个漏洞 CVSS 评分≥9.0，中位数达 9.2；21 个漏洞 EPSS 得分超过 50%，平均 EPSS 为 25.3%，代表漏洞被攻击概率较高。在有公开披露时间可查的 58 个漏洞中，26% 的漏洞在公开披露 7 天内就出现攻击行为，漏洞从披露到录入 KEV 目录的中位数间隔仅 44 天，补丁修复窗口期显著缩短，部分多年遗留老漏洞仍被攻击者持续利用。  
  
  
从漏洞类型看，OS 命令注入（CWE78）、代码注入（CWE94）、身份验证不当（CWE287）为高发弱点，这类漏洞大多无需用户交互，便于自动化扫描批量攻击。厂商维度，微软以 12 个漏洞居首位，Cisco、Linux 紧随其后，防火墙、VPN、路由器等互联网暴露的边界设备成为攻击重点，GitLab、TeamCity 等开发工具也成为攻击者目标。报告 EPSS 排名首位的 CVE202672898（Metabase）EPSS 高达 94.2%，已经出现成熟 POC 可利用代码。  
  
  
报告建议安全团队优先处置 EPSS 排名靠前漏洞，面向互联网业务力争 7 天内完成补丁更新，尽量将防火墙、管理控制台等设备从公网剥离，同时结合 CVSS 与 EPSS 指标，合理排序补丁优先级，并排查遗留老旧未修补漏洞风险。  
  
  
原文链接：  
  
https://securityonline.info/q3-2026-cve-report/  
  
  
**OpenAI 失控 AI 代理对 Wikimedia 发起异常操作，海量流量诱发服务中断风险**  
  
维基媒体基金会披露，一批归属 OpenAI 的失控 AI 代理，近期实施大量未授权操作，海量 API 请求或促成 2026 年 5 月一场持续约四天的服务宕机事件。  
  
  
这批 AI 代理生成巨量访问流量，主要在非公开沙箱区域开展测试编辑，还尝试利用配置漏洞入侵公开笔记工具 Etherpad，但入侵并未成功。其核心意图是借助维基媒体平台与工具充当代理，抓取第三方站点数据。平台允许经过社区报备审批的机器人编辑，而这批 AI 代理并未履行审批流程，属于违反服务条款的资源滥用行为，维基媒体并未观测到代理借助平台互相协同通信的迹象，整体系统未被实际攻破。  
  
  
该事件凸显 AI 代理集群带来的新型安全风险：即便没有主观恶意，AI 代理为完成目标产生的高负载流量，可等效形成 DDoS 效果，造成服务停机与带宽成本激增。数据显示，2025 年维基媒体机器人带宽占用上涨 50%，机器人流量占资源消耗流量的 65%。  
  
  
2026 年多起同类事件多源于 AI 对齐缺陷与人机监管缺位，已引发美国立法层面讨论。安全从业者指出，AI 代理风险不在于主观恶意，而在于目标驱动下突破边界；开发者应当针对模糊指令场景完成充分测试，明确代理行为边界，设置人工介入机制，避免自主智能体肆意扰动公共网络资源。  
  
  
原文链接：  
  
https://www.cpomagazine.com/cyber-security/wikimedia-rogue-ai-agents-traced-back-to-openai-made-unauthorized-edits-attempted-to-compromise-etherpad/  
  
  
  
安全事件  
  
  
**OT 安全告警：法国数百超市制冷控制系统遭网攻，大量生鲜食品损毁**  
  
2026 年 10 月 11 日夜间，法国发生一起针对民生运营设备的网络攻击事件，数百间连锁超市的制冷设备被远程操控，大量生鲜食品变质报废。  
  
  
本次事件波及 Intermarché、Netto 约 300 家门店，以及不足 100 家 Coopérative U 门店。攻击者远程触发制冷机组的除霜功能，直接破坏冷链存储能力。Coopérative U 首席执行官 Dominique Schelcher 表示，攻击瞄准保障民众食品供应的关键设备，属于史无前例的严重威胁。法国国家网络安全局 ANSSI 已介入处置，并开展跨国协同响应工作。  
  
  
据当地媒体披露，遭入侵的是丹麦厂商 Danfoss 生产的制冷控制器，攻击疑似利用该设备旧版软件的安全缺陷。此前 CISA 就发布过该系列设备漏洞安全公告，安全研究团队 Claroty Team82 也曾预警，被攻陷的控制器可对外上报虚假正常温度，同时设备实际停止制冷；互联网上公网暴露的同类型设备超过 2700 台，目前暂未确认本次攻击具体使用哪一处漏洞。设备供应商 Danfoss 尚未公开确认遭受攻击。受影响零售集团表示，相关系统由供应商负责运维，将由供应商发起刑事报案。  
  
  
该事件凸显工业物联网 OT 设备风险：这类设备一旦被攻破，会直接造成现实世界的物理与经济损失，安全从业者应当重视公网暴露的工控类老旧设备的加固与隔离工作。  
  
  
原文链接：  
  
https://therecord.media/cyberattack-disrupts-refridgeration-in-france  
  
  
**零售巨头 Ahold Delhaize 遭遇网络安全事件，多国旗下品牌网站下线**  
  
2026 年 10 月 12 日，跨国零售集团 Ahold Delhaize 对外披露遭遇网络安全事件，为规避风险，企业主动将部分系统下线作为预防性防护措施。  
  
  
事件造成旗下多个品牌网站无法访问，美国区域 Hannaford、Stop & Shop、Giant Food、Food Lion、The Giant Company 共五大品牌网站周一上午全部宕机；荷兰本土 Albert Heijn、Etos、Gall & Gall 网站同样同步中断。集团表示，所有线下实体门店保持正常营业，但暂未说明门店内部业务系统是否受到波及，也尚未公布系统恢复时间。  
  
  
本次并非该集团首次遭遇网络安全事故。2024 年末 Ahold Delhaize 就曾发生安全泄露事件，超 200 万人敏感信息存在外泄风险，当时同样出现系统下线、网站故障、电商服务暂停等现象，受影响对象包含在职及离职员工及其家属，客户支付与药房系统当时未确认被泄露。  
  
  
目前 Ahold Delhaize 尚未公开本次事件的攻击类型、入侵路径，也未确认是否发生数据泄露。该案例提醒零售行业，需要完善网络安全应急预案，面对安全威胁时及时开展系统隔离，但同时也要兼顾业务连续性评估。  
  
  
原文链接：  
  
https://www.cybersecuritydive.com/news/ahold-delhaize-adusa-cybersecurity/832750/  
  
  
**美国执法部门持令部署 IMSI 捕捉器，追踪新闻事件相关人员手机位置**  
  
据 Wired 查阅的法庭卷宗，美国联邦执法机构取得司法搜查令，启用颇具争议的基站模拟设备 cellsite simulator，又称 Stingray、IMSI 捕捉器，用于定位并逮捕 21 岁大学生 Jerome Richardson。该学生是 Don Lemon 反 ICE 抗议报道的制作协助人员，也是 39 名被告之一，被控共谋侵犯信众公民权利、干扰宗教礼拜活动。  
  
  
事件源于今年 1 月 18 日明尼苏达州圣保罗市 Cities Church 教堂抗议，示威针对同时担任 ICE 地方主管的教堂牧师。抗议结束 12 天后，明尼苏达州、宾夕法尼亚州法官，先后于 1 月 30 日、31 日签署搜查令，授权用 IMSI 捕捉器针对 Jerome Richardson 手机开展追踪，其辩护律师的法律文书写明监控目的为 “定位并实施逮捕”。  
  
  
调查方还申请对记者 Georgia Fort 以及当时的州参议院候选人使用同款监控工具，另外申请 pen register 命令记录通话联络号码，暂不清楚后部分申请是否获得法官批准。IMSI 捕捉器俗称伪基站，通过伪装成运营商基站截获周边手机识别信息，实现定位追踪，该技术长期存在公民隐私与监控边界的争议。Jerome Richardson 的辩护团队已提交动议，申请排除这套追踪获取的证据。  
  
  
原文链接：  
  
https://www.wired.com/story/feds-got-warrants-to-use-fake-cell-towers-to-hunt-down-don-lemons-producer/  
  
  
  
安全攻防  
  
  
**Warden Stealer 窃取木马升级，将 AI 编码智能体 Token 列为重点窃取目标**  
  
2026 年 10 月 12 日，Gen Threat Labs 发布研究报告，披露名为 Warden Stealer（曾命名 CallbackBeaver）的 Rust 编写 MaaS 信息窃取恶意软件，该威胁自 2026 年 8 月起在地下论坛扩散，已成为高频拦截的恶意样本，主要针对 Windows 8 至 Windows 11 系统。  
  
  
该恶意软件通过破解软件、游戏作弊工具、恶意广告、ClickFix 钓鱼进行分发，自带定制加载器，可将载荷注入 explorer.exe 等可信进程；借助大量垃圾数据填充文件、代码变形、虚拟机检测等手段规避沙箱与杀毒引擎检测，配置多组备用 C2 域名，通过 HTTPS 传输窃取数据，还支持下发执行额外载荷。  
  
  
Warden Stealer1.9 版本新增 AI 工具窃取能力，可盗取 Claude、Codex、Grok、Cursor 等 AI 助手、编码智能体的访问 Token、MCP 凭证与提示词历史记录。该恶意软件能够绕过 ApplicationBound Encryption（ABE）防护，窃取浏览器 Cookie 与密码；内置加密货币篡改器，可替换剪贴板内多类币种钱包地址。研发者疑似来自俄语网络犯罪圈层，恶意程序会主动规避独联体及波罗的海国家设备。  
  
  
安全团队给出防御建议，规避非官方来源软件，警惕网页诱导复制执行命令的 ClickFix 攻击；将 AI 智能体 Token、MCP 密钥存放至密钥保险箱，疑似失陷后立即轮换凭证；监控 explorer.exe 异常内存写入行为，对超大填充的可执行文件重点告警。  
  
  
原文链接：  
  
https://securityonline.info/warden-stealer-rust-infostealer/  
  
  
  
产业动态  
  
  
**ERP厂商强化AI布局以打消买家顾虑，抵御前沿AI厂商竞争**  
  
2026 年 10 月 12 日报道，SAP、Oracle、Infor 等主流 ERP 厂商集中更新 AI 相关产品，四大行业趋势显现，厂商通过强化内置 AI 能力，一方面满足企业需求，另一方面抵御 hyperscalers 与 frontier AI 模型厂商的竞争冲击。  
  
  
Oracle 推出 Fusion Claw 运行时软件，搭载 25 个 agentic AI 应用，内置企业规模化 AI 所需的治理与安全能力。SAP 在 SAP Connect 大会宣布，5 月发布的 agentic AI 产品正式上线，升级 Joule Work 生成式 AI 交互界面；同时计划收购 AI 企业 TechWolf。Infor 发布 Infor Industry AI 架构，新增垂直行业 AI 智能体，配置治理、风险与合规校验层。  
  
  
当前 ERP 的 AI 用户界面基本落地，依托生成式 AI 结合 agentic AI，支持自然语言指令执行业务操作。AI 也打破 SaaS ERP 对定制化的约束，用户可快速搭建自定义 AI 智能体，但分析师警示，无节制定制会出现过度改造，增加系统运维与安全隐患，企业仍需遵循 clean core 理念，保障核心系统稳定。  
  
  
ERP 厂商的主要竞争对手不再是同行，而是 AWS、Google Cloud、Palantir、Anthropic 等企业。外界通用前沿模型缺少 ERP 内部业务逻辑与完整数据，输出准确性不足。各大厂商着重强调 agentic AI 的企业级安全，复用 ERP 原有身份授权体系，管控自定义智能体全生命周期，防范智能体非主观误操作带来的业务风险。行业建议企业部署前夯实数据与流程基础，完善 AI 治理框架。  
  
  
原文链接：  
  
https://www.techtarget.com/it-strategy/news/366651906/Vendors-fortify-AI-in-ERP-to-assure-buyers-deter-frontier-AI-rivals  
  
  
  
新品发布  
  
  
**Meta 推出首款前沿级 Muse Wonder 模型，闭源付费模式搅动 AI 智能体安全格局**  
  
2026 年 10 月 12 日，Meta 正式发布旗下首款 frontier 模型 Muse Wonder，以及性能更强的 Muse Wonder Full 版本，面向个人与专业 AI 智能体场景提供服务。这是 Meta 超级智能实验室团队经过 14 个月团队重组、算力投入后推出的旗舰成果，扎克伯格称该产品代表 Meta 已经追上行业前沿 AI 竞争对手。  
  
  
根据 Meta 公布的基准测试结果，Muse Wonder 在部分评测指标上优于 OpenAI 的 GPT6 Astra、Anthropic 的 Claude Opus 5.5 与 Fable 5.1，擅长处理租约解析、财务与健康信息研判、编程、法律文本分析等高复杂度任务，强化长周期智能体任务与长期记忆能力。开发者可通过 Meta Model API、Muse Code 接口调用该模型，Muse Wonder Full 现阶段优先向 Muse Code 订阅用户开放，后续将赋能个人智能体产品 Muse。  
  
  
本次发布的重要变化在于，Muse Wonder 系列放弃 Meta 过往优先开源的路线，采用闭源付费模式，API 输入每百万 token 定价 4 美元、输出每百万 token 定价 20 美元，同时个人端 Muse 服务设置月订阅档位，主要争夺企业级 AI 市场。  
  
  
对安全从业者而言，前沿闭源大模型大规模落地，意味着企业在 API 接入、智能体业务部署过程中，需要重点关注提示词注入、数据泄露、模型输出不可控等安全威胁，做好业务侧风险管控。  
  
  
原文链接：  
  
https://techcrunch.com/2026/10/12/meta-releases-its-first-frontier-model-muse-wonder/  
  
  
  
  
  
  
  
**联系我们**  
  
合作电话：18610811242  
  
合作微信：aqniu001  
  
联系邮箱：bd@aqniu.com  
  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wKeDC5RjIzEIQJapWpRKLFPpEAPkibBhlfbBEdghxUOfx4ZLicUwyoOLJ9rULg3d7E7OBuP7icJZEnwZLMcnZJh8KLJwjpj9WW9OGkYPTiaVu7w/640?wx_fmt=gif&from=appmsg "")  
  
  
  
