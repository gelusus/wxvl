#  Volt Typhoon 在四次零日漏洞利用活动中使用隐蔽 Silent Train 网络  
爱拍照的老李
                    爱拍照的老李  爱拍照的老李   2026-10-29 01:38  
  
**导****读**  
  
  
  
Black Lotus Labs 在一份报告中表示，Silent Train 最早于 2023 年底被识别，它利用由被攻陷的 SOHO、IoT 和企业基础设施组成的动态链路。  
  
  
被称为“Volt Typhoon”的网络间谍组织，其活动被归因于一个现已被捣毁的、代号为“Silent Train”隐蔽接入网络，该网络被用于零日漏洞利用和战略性接入。  
  
  
“Silent Train”于2023年末首次被发现，它利用被攻陷的小型办公室/家庭办公室、物联网以及企业基础设施的轮换链路，Black Lotus Labs 在一份报告中表示，“该组织采用反取证技术、分区化操作以及事后清理手段，以最大限度降低被发现和溯源的可能性。”  
  
  
其主要目的是助力漏洞利用、凭据访问、数据窃取等关键攻击阶段，并维持对受害者环境的持续访问。  
  
  
该行为主要有两大突出原因：  
- 它挑出了那些能够为下游客户、敏感网络或战略项目提供规模化接入的实体。  
- 该攻击目标为终端管理平台、网络管理系统、软件定义广域网编排器以及在企业环境中发挥关键作用的控制平面技术。  
在电信运营商、政府、国防承包商、卫星和关键任务通信服务、大学、IT公司以及供水、能源、交通、海事、铁路、航空和应急服务等关键基础设施领域，已有数百名受害者被确认身份。  
  
  
Volt Typhoon (别名 Bronze Silhouette, Insidious Taurus, UNC3236, Vanguard Panda,Voltzite) 是一个威胁组织代号，据报告称，该组织以针对关键基础设施网络为目标而闻名，尤其涉及通信、能源、交通以及供水和废水处理系统领域。  
  
  
该组织至少自 2021 年起就已活跃，此前曾被发现利用 Versa Director 中的一个文件上传漏洞作为零日漏洞（CVE-2024-39717），来投放一款名为 VersaMem 的定制化网页后门，该后门旨在窃取凭证并允许访问下游客户系统。  
  
  
Black Lotus Labs 的最新研究结果显示，该安全漏洞是威胁组织通过“Silent Train”发起的四场已确认零日攻击活动之一。  
  
  
其余三个漏洞如下所示：  
- CVE-2025-8875 与 CVE-2025-8876 - N-able N-central 中存在的一组不安全反序列化与命令注入漏洞。  
  
- CVE-2026-20127 - 思科 Catalyst SD-WAN 控制器存在身份验证绕过漏洞  
  
- CVE-2026-16812 - Arista VeloCloud Orchestrator 中的操作系统命令注入漏洞  
  
此外，Volt Typhoon 还被怀疑在 Quest KACE 的一个可能的零日漏洞利用活动有关。据估计，此次活动影响超过30个受害者。  
  
  
“Silent Train”围绕被入侵的家庭办公、物联网和企业设备的快速轮换链构建，这些设备会被按需利用，针对精心挑选的目标发起攻击，每次操作后都会被清理，并持续进行更新替换。”Black Lotus Labs 表示。  
  
  
“该模型让 Volt Typhoon 能够以极高的操作安全水平开展渗透利用和后渗透利用行动，同时最大限度降低被检测、归因和干扰的可能性。”  
  
  
这些攻击的一个显著特征是，它们反复针对那些能够为美国带来战略价值的大型企业环境打开广泛可见性、控制权和访问权限大门的系统。  
  
  
有趣的是，秘密访问能力在2023年11月左右开始成型，就在针对KV僵尸网络的第一轮破坏行动开始前几周。KV僵尸网络是一个物联网僵尸网络，与包括“Volt Typhoon ”在内的多个威胁组织存在关联。  
  
  
据称，在僵尸网络于2024年1月下旬发起第二次干扰之后，“Silent Train”的运营活动大幅激增，与此同时，其转而采用由各类设备类型组成且分布于不同地区的临时基础设施。  
  
  
“Black Lotus Labs”表示：“Volt Typhoon不再持续使用固定基础设施，转而采用由一次性受损设备链组成的模式，可根据需要重新搭建。该组织快速搭建并拆解临时基础设施，构建出一套具备高韧性、安全性且隔离性强的架构，以此保护关键访问权限。”  
  
  
被称为“分层多跳框架”的“Silent Train”，其设计目的是让“Volt Typhoon”的攻击者在恶意流量抵达目标设备前，通过多台设备进行转发，从而向受害者和防御者隐藏流量的源头。  
  
  
该链路中的每一环都承担着特定功能：  
- 虚拟专用服务器（VPS）充当临时暂存区，让威胁行为者在每次会话后配置并销毁这些服务器。这些会话均从位于美国境外的VPS基础设施发起。  
- 分布在不同国家、不同提供商和不同硬件上的受损非美国小型办公室/家庭办公室（SOHO）设备和物联网（IoT）设备，这些设备会隐藏攻击的起始点。  
- 位于美国的受感染设备会在连接到国内受害者之前充当出口节点，有助于规避地理围栏和IP信誉黑名单。  
至少有27类设备被利用并纳入该隐蔽网络，涵盖从小型办公室/家庭办公室（SOHO）和物联网（IoT）设备（如TP-Link、Intelbras、Ruckus Wireless、SATO、NETGEAR、Zyxel和Cambium）到企业级平台（如Fortinet、Cisco、PRTG、Mirth Connect和CyberPanel）。  
  
  
“Silent Train”以其运营纪律性脱颖而出，该组织专注于短时效、自管理的会话，而非持久型中继或命令与控制（C2）基础设施。Black Lotus Labs 表示。“为搭建这些会话，其攻击者利用小型办公室/家庭办公室（SOHO）、物联网（IoT）及其他中继设备上暴露的网络接口，随后仅部署必要工具，通过受感染节点实现链式安全外壳协议（SSH）访问。”  
  
  
“这包括临时SSH服务、密钥、配置文件以及‘就地取用’工具。在行动会话结束时，他们清除了所有痕迹，并拆除了随之而来的行动链路。‘Silent Train’行动会话平均持续约四小时，部分会话仅持续数分钟。”  
  
  
另一个值得注意的方面是，在“Silent Train”行动中，特定设备类型被指定部署在特定位置。例如，经发现，Intelbras调制解调器仅作为中继节点出现，而Netgear  
Orbi 设备和Fortinet  
FortiGate 设备则充当了美国境内的中介节点和出口节点，负责与受害者进行交互。  
  
  
到2025年初，Silent Train 已从一个完全临时的中继网络演变为混合架构，其企业级设备的留存时间平均为一到三个月，而非每次任务后就更换。  
  
  
该平台后续迭代出了反向隧道变种方案，这类方案借助 Chisel、FRP 等开源代理，将受感染设备的流量转发至上游基础设施。  
  
  
技术报告：  
  
https://www.lumen.com/blog/en-us/silent-train-volt-typhoon-operational-framework  
  
  
新闻链接：  
  
https://thehackernews.com/2026/10/volt-typhoon-used-covert-silent-train.html  
  
![](https://mmbiz.qpic.cn/mmbiz_svg/McYMgia19V0WHlibFPFtGclHY120OMhgwDUwJeU5D8KY3nARGC1mBpGMlExuV3bibicibJqMzAHnDDlNa5SZaUeib46xSzdeKIzoJA/640?wx_fmt=svg "")  
  
**今日安全资讯速递**  
  
  
  
**APT事件**  
  
  
Advanced Persistent Threat  
  
1. Volt Typhoon 在四次零日漏洞利用活动中使用了隐蔽  
Silent Train 网络  
  
Black Lotus Labs 在一份报告中表示，Silent Train 最早于 2023 年底被识别，它利用由被攻陷的 SOHO、IoT 和企业基础设施组成的动态链路。  
  
🔗  
https://thehackernews.com/2026/10/volt-typhoon-used-covert-silent-train.html  
  
  
2  
. 意大利议会多个邮箱遭遇网络攻击  
  
意大利议会遭遇网络攻击，这一事件凸显混合战争的风险，也表明有必要提高透明度、强化防御能力并落实问责机制。  
  
🔗  
https://securityaffairs.com/201867/hacking/italys-parliament-under-cyber-pressure-why-the-attack-on-the-chamber-matters.html  
  
  
3  
. 调查贝尔格莱德特朗普大厦项目的塞尔维亚检察官遭间谍软件攻击  
  
塞尔维亚一名教授及其重要支持者成为 Pegasus 间谍软件的攻击目标。调查还发现，曾调查帮助特朗普集团在贝尔格莱德修建大厦相关阴谋的塞尔维亚检察官也遭此针对。  
  
🔗  
https://citizenlab.ca/serbian-prosecutor-who-investigated-belgrade-trump-tower-project-targeted-with-spyware/  
  
  
4. 摩尔多瓦调查重大政府网络袭击是否存在外国势力介入  
  
摩尔多瓦总理周三表示，该国正在调查外国国家行为体是否参与了针对其政府系统的重大网络攻击，此次攻击泄露了敏感个人信息，并导致数十项公共服务陷入瘫痪。  
  
🔗  
https://therecord.media/moldova-investigates-possible-foreign-involvement-cyberattacks  
  
  
  
**一般威胁事件**  
  
  
General Threat Incidents  
  
1. 美国政府称黑客正在利用关键的 PHP 网页漏洞  
  
CISA证实，黑客正积极利用广泛使用的PHP网络软件中的一对严重安全漏洞，并已下令美国政府内部的民用联邦机构在未来三天内修复其系统。  
  
🔗  
https://techcrunch.com/2026/10/27/us-government-says-hackers-are-now-exploiting-critical-php-web-bugs/  
  
  
2. 黑客劫持 Convertr 分析脚本以窃取支付卡数据  
  
一个信用卡盗刷团伙入侵了电商平台 Convertr 使用的一款共享分析脚本，在顾客结账期间窃取他们的支付数据。幕后黑手为一个巴西犯罪团伙，该团伙至少自2024年7月起便已活跃，主要针对运行Magento、OpenCart和WooCommerce平台的网店实施恶意软件感染。  
  
🔗  
https://www.bleepingcomputer.com/news/security/hackers-hijack-convertr-analytics-script-to-steal-payment-card-data/  
  
  
3  
. CyberStrikeAI 利用44个  
 AI Agent   
针对日本目标自动化黑客攻击行动  
  
网络安全研究人员披露了三起针对日本机构的网络攻击细节，在这些攻击中，人工智能（AI）无论是作为主导方还是协助人类操作员，都是关键要素。  
  
🔗  
https://thehackernews.com/2026/10/cyberstrikeai-uses-44-ai-agents-to.html  
  
  
4  
. Warp 披露数据泄露事件，AI 代理日志与用户遥测数据遭泄露  
  
在该项目基础设施对第三方供应商发起的一次攻击中，属于 Warp 智能体开发环境用户的智能体决策日志和遥测数据被泄露。  
  
🔗  
https://www.bleepingcomputer.com/news/security/warp-discloses-breach-exposing-ai-agent-logs-user-telemetry-data/  
  
  
5  
. 仅靠软件物料清单（SBOM）无法揭示供应链攻击的范围  
  
近期发生的 Shai-Hulud 攻击表明，防御方需要对生产代码和受影响的服务进行实时监控，以应对供应链入侵事件。  
  
🔗  
https://www.darkreading.com/cyber-risk/sboms-alone-scope-supply-chain-attack  
  
  
6  
. 攻击者利用Teams外部聊天功能在15分钟内攻破企业  
  
北极狼（Arctic Wolf）的最新研究发现，威胁行为者正利用微软Teams的外部聊天创建功能，将员工作为目标，以此实现对企业组织的初步入侵。研究人员表示，他们已发现400多起该功能被滥用的事件，恶意行为者采用的IT支持人员冒充手段“高度一致”。  
  
🔗  
https://www.infosecurity-magazine.com/news/exploit-teams-chat-breach/  
  
  
7  
. GhostAction黑客攻击浪潮渐歇，但攻击者仍潜伏在GitHub平台  
  
GhostAction是一场持续进行的攻击活动，攻击者通过入侵开发者账户，向代码仓库中注入GitHub Actions工作流文件，以此将目标对准GitHub平台。  
  
🔗  
https://www.darkreading.com/application-security/ghostaction-hacking-wave-fizzles-attackers-github  
  
  
8  
. 黑客窃取了数千名国土安全部员工的人事数据  
  
美国国土安全部发生了一起此前未公开的数据泄露事件，一名黑客窃取了大量数据，导致该部门数千名员工的个人可识别信息被发布到了网上。  
  
🔗  
https://techcrunch.com/2026/10/28/hacker-stole-personnel-data-about-thousands-of-homeland-security-employees-including-cbp-and-ice-agents/  
  
  
9  
. 攻击者将恶意软件隐藏在经过签名的 ChatGPT 和 Claude 二进制文件背后  
  
攻击者利用托管在GitHub上的伪造ChatGPT和Claude桌面应用程序分发恶意软件链，该恶意软件链滥用合法签名的二进制文件以规避安全防护措施。  
  
🔗  
https://www.infosecurity-magazine.com/news/attackers-hide-malware-behind/  
  
  
10  
. Sony 的 ISP 子公司数据泄露导致 250 万条客户记录被曝光  
  
索尼网络通信公司已确认发生数据泄露事件，约250万条包含现有及过往客户个人信息的记录遭到泄露。  
  
🔗  
https://www.bleepingcomputer.com/news/security/sony-isp-subsidiary-data-breach-exposes-25-million-customer-records/  
  
  
1  
1  
. 日本航空警告发生数据泄露，该国正应对多次网络事件  
  
日本航空公司表示，其一家IT供应商遭遇数据泄露，导致9万多名客户的个人信息被泄露。这使得近期针对日本企业的一系列网络攻击事件进一步升级，而这些攻击已促使日本政府近期采取行动。  
  
🔗  
https://therecord.media/japan-airlines-warns-of-breach-country-grapples-with-incidents  
  
  
1  
2. 新墨西哥大学健康系统遭勒索软件攻击，患者护理与手术受影响  
  
新墨西哥大学健康科学中心披露了一起网络攻击事件，该事件导致系统中断，并扰乱了患者护理和学术活动。  
  
🔗  
https://www.bleepingcomputer.com/news/security/unm-health-ransomware-attack-disrupts-patient-care-surgeries/  
  
  
**漏洞事件**  
  
  
Vulnerability Incidents  
  
1. PHP2Shell 攻击攻陷 25 台服务器，入侵美国政府关联网络  
  
2026年10月25日观察到的这次活动中，攻击者结合 PHP2Shell 和一个内核权限提升漏洞来获取 root 权限。  
  
🔗  
https://thehackernews.com/2026/10/php2shell-attack-roots-25-servers.html  
  
  
2. CVE-2026-111497: 原生 Brotli 解码器 RCE 与内存泄露  
  
AWS 已关注到CVE-2026-111497，这是原生 Brotli 解压缩库中存在的一个缓冲区溢出漏洞，该漏洞在 Node.js 2026 年 10 月的安全更新中得到了首次修复。  
  
🔗  
https://aws.amazon.com/security/security-bulletins/rss/2026-155-aws/  
  
  
3. Tailscale 警告存在可导致 root 权限提升的 SSH 漏洞  
  
Tailscale 安全团队正着手修复近十二个安全漏洞，这些漏洞可能被利用来窃取凭证、导致服务器崩溃、获取 root 权限等。  
  
🔗  
https://www.bleepingcomputer.com/news/security/tailscale-warns-of-serious-ssh-flaws-allowing-root-privilege-escalation/  
  
  
4. 苹果为旧款iPhone回溯修复CoreGraphics零日漏洞补丁  
  
苹果已将上月发布的安全补丁向后移植到旧款 iPhone 和 iPad 中，以修复在针对 iOS 设备的“极高技术水平”定向攻击中被利用的零日漏洞。  
  
🔗  
https://www.bleepingcomputer.com/news/apple/apple-backports-coregraphics-zero-day-patches-to-older-iphones/  
  
  
5. OffSec 在发现关键漏洞后推出免费的 PHP2Shell 实验环境  
  
全新的实操实验室让安全从业者能够亲身体验PHP2Shell漏洞利用，与此同时，人工智能辅助的研究正在重塑漏洞发现的方式。  
  
🔗  
https://hackread.com/offsec-launches-free-php2shell-lab-following-critical-vulnerability-discovery/  
  
  
6. 澳大利亚网络情报主管：随着 AI 加速黑客攻击，预计将出现更多破坏  
  
澳大利亚网络安全机构警告称，人工智能将催生更多破坏性攻击，帮助黑客更快发现漏洞并以更大规模加以利用。  
  
🔗  
https://securityaffairs.com/201859/ai/australias-cyber-spy-chief-expect-more-disruption-as-ai-accelerates-hacking.html  
  
****  
扫码关注  
  
爱拍照的老李  
  
**讲述普通人能听懂的安全故事**  
  
  
