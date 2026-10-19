#  APT 组织利用 BeyondTrust 漏洞作为零日漏洞  
爱拍照的老李
                    爱拍照的老李  爱拍照的老李   2026-10-19 01:00  
  
**导****读**  
  
  
  
Armory Castle 利用 CVE-2026-102764 进行初始访问，部署了自定义工具，并删除了日志和痕迹。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/PaFY6wibdwyJ70KbccrW1rXW6dSIoZSA98Kpe7Wib0qNIJG5pseiags5pqHmsdWNtcf9fJFIN1gEGAMkI57Abhtl0GZcJID51Gg7ib8Xm23rsts/640?wx_fmt=jpeg&from=appmsg "")  
  
  
被利用的漏洞编号为 CVE-2026-102764（通用漏洞评分系统分值为9.3），该漏洞被描述为对消息来源的验证不当，可能让攻击者对远程支持（RS）设备执行文件操作。  
  
  
据 BeyondTrust 称，26.2.3 及之前所有版本的 RS 均受此影响。24.3.1 至 26.2.3 版本的 BT26-05-RS 补丁已包含该漏洞的修复程序。24.3.1 之前版本的用户应先升级至较新版本，再应用该补丁。  
  
![https://storage.googleapis.com/gweb-cloudblog-publish/images/Figure_1_ARMORY_CASTLE_BeyondTrust_comprom.max-1400x1400.jpg](https://mmbiz.qpic.cn/mmbiz_jpg/PaFY6wibdwyLoSSqibH6F3goWtyd0liaApR6ic4mDQ0MgdLibcM2EfDRWVz0uiaq79BcC1MhqYgjgewWqatIcYNTjpn4ua8226QyeLIlVl7y0XCCo/640?wx_fmt=jpeg&from=appmsg "")  
  
  
BeyondTrust于9月28日发现了这一安全缺陷，也就是针对某安全设备提供商的漏洞利用行为公开后的第二天。不过，首次漏洞利用发生在8月29日，目标为一台由BeyondTrust托管的RS设备和一台自主托管的RS设备。  
  
  
该公司表示其在9月30日前就制作出了补丁，但花了不少时间对其进行测试并解决额外的攻击途径，最终在10月8日推送了修复程序。  
  
  
“截至2026年10月9日，针对所有远程支持SaaS客户已应用修复此漏洞的补丁。”BeyondTrust 表示，并指出自托管客户若未开启自动更新，应手动应用该修复程序。  
  
  
根据Mandiant和谷歌威胁情报集团（GTIG）的消息，CVE-2026-102764 已被威胁组织 Armory Castle 用作零日漏洞进行利用，该组织也被追踪为 UNC5221 和 Warp Panda。  
  
  
Armory Castle  
是一个由国家支持的间谍组织，专注于长期情报收集，以针对软件即服务（SaaS）提供商、科技公司、政府机构、法律服务机构和业务流程外包商而闻名。  
  
  
Mandiant 与 GTIG 观察到，该高级持续性威胁（APT）以 BeyondTrust 设备为目标获取初始访问权限，部署了 Sawfly 网页后门和 Bareclaw 隧道工具与该设备进行交互，实现权限提升，并在已被入侵的环境中横向移动。  
  
  
Sawfly是一款轻量级的PHP网页后门，具备双向加密的命令与控制（C&C）通信功能，是威胁组织在目标环境中建立初始立足点的主要工具。  
  
  
部署 webshell 后，Armory Castle  
获得了设备主机的根级控制权，获取了凭证，获得了对站点服务账户的无密码根权限，并系统性地删除了系统、网络服务器和应用级日志。  
  
  
随后，攻击者利用被攻陷的管理员凭据转向RS应用程序，修改了账户权限以获得对远程系统的控制权，并开始针对内部终端。  
  
  
“威胁组织利用设备的内置支持工具请求并获得了完整的文件系统访问权限。借助原生的 BeyondTrust 远程支持文件传输服务，直接从用户的桌面环境下载并窃取了有助于网络侦察的敏感内部文档，例如软件清单。”Mandiant 和 GTIG 解释道。  
  
  
此外，有观察到攻击者使用了自定义SOCKS5代理隧道工具Bareclaw，该工具支持数据加密、向上游代理进行流量链式传输以及身份验证等功能。  
  
  
“GTIG 此前曾报告，Armory Castle组织利用边缘设备中的多个漏洞，并部署了一套全面的自定义工具来实现其目标，这些工具包括被动后门、网页后门以及边缘设备被木马化的组件。该组织早在 2023 年 10 月就已开始活动，在其行动期间，针对了多个国家和多个行业领域。”Mandiant 与 GTIG 指出。  
  
  
该威胁组织此前被发现利用 Citrix NetScaler（CVE-2023-4966，又称 CitrixBleed）、Ivanti Connect Secure（CVE-2023-46805、CVE-2024-21887 和 CVE-2025-0282）以及戴尔虚拟机 RecoverPoint（CVE-2026-22769）中的零日漏洞进行攻击。  
  
  
技术报告：  
  
https://cloud.google.com/blog/topics/threat-intelligence/armory-castle-exploits-beyondtrust  
  
  
报告链接：  
  
https://www.securityweek.com/chinese-apt-exploited-beyondtrust-vulnerability-as-zero-day/  
  
![](https://mmbiz.qpic.cn/mmbiz_svg/McYMgia19V0WHlibFPFtGclHY120OMhgwDUwJeU5D8KY3nARGC1mBpGMlExuV3bibicibJqMzAHnDDlNa5SZaUeib46xSzdeKIzoJA/640?wx_fmt=svg "")  
  
**今日安全资讯速递**  
  
  
  
**APT事件**  
  
  
Advanced Persistent Threat  
  
1. APT 组织利用 BeyondTrust 漏洞作为零日漏洞  
  
Armory Castle 利用 CVE-2026-102764 进行初始访问，部署了自定义工具，并删除了日志和痕迹。  
  
🔗  
https://www.securityweek.com/chinese-apt-exploited-beyondtrust-vulnerability-as-zero-day/  
  
  
2. AI 辅助黑客入侵政府 Linux 服务器以部署后门并窃取凭据  
  
黑客利用 AI 辅助脚本在 2025 年 10 月至 2026 年 4 月期间观察到的一次行动中，攻破了政府 Linux 服务器，收集管理员凭据并建立了持久化后门。研究人员尚未确定该活动是受国家支持还是以经济利益为动机。  
  
🔗  
https://gbhackers.com/linux-servers-compromised/  
  
  
3. 曹县IT人员利用被盗的美国身份赚取300万美元  
  
FBI 因一起利用虚假 IT 工人针对 70 家美国公司并赚取 300 万美元的计划，突袭了德克萨斯州的 40 个家庭。  
  
🔗  
https://securityaffairs.com/201193/intelligence/north-korean-it-workers-used-stolen-us-identities-to-earn-3-million.html  
  
  
4  
. SideCopy黑客组织利用ReverseRAT恶意软件针对印度机构实施数据窃取攻击  
  
印度有关部门警告称，SideCopy 黑客组织正针对印度各类机构（包括学术机构）发动鱼叉式网络钓鱼攻击，此类攻击会在 Windows 系统上部署 ReverseRAT 恶意软件。  
  
🔗  
https://gbhackers.com/sidecopy-hackers-target-indian-organizations/  
  
  
  
**一般威胁事件**  
  
  
General Threat Incidents  
  
1. 黑客通过 ConnectWise 供应链攻击入侵 Microsoft 365 租户  
  
在黑客获取应用身份信息并入侵微软 365 租户后，ConnectWise 已关闭客户对其云备份服务（前身为 SkyKick 云备份）的访问权限。  
  
🔗  
https://www.bleepingcomputer.com/news/security/hackers-breach-microsoft-365-tenants-via-connectwise-supply-chain-attack/  
  
  
2. GhostAction黑客在94个GitHub仓库的伪造字体文件中隐藏窃取凭据的恶意软件  
  
GhostAction 攻击者已将其 GitHub 凭证窃取活动升级至第二阶段，该阶段会将恶意 JavaScript 伪装成字体文件，并通过 Visual Studio Code 任务针对开发者工作站发动攻击。  
  
🔗  
https://gbhackers.com/ghostaction-hackers-hide-credential-stealing-malware/  
  
  
3  
. 新型Aoki信息窃取程序瞄准人工智能编码助手凭证  
  
赛门铁克的一份新报告警告称，新版The Aoki（青木）信息窃取软件正针对流行人工智能编码助手使用的凭证和会话文件。该窃取程序还能从多款人工智能编码辅助工具中获取API密钥和令牌，包括Anthropic的Claude、OpenAI的Codex、SpaceX的Cursor以及谷歌的Gemini。  
  
🔗  
https://www.infosecurity-magazine.com/news/aoki-infostealer-ai-coding/  
  
  
4  
. 安卓恶意软件在劫持手机并窃取敏感数据的同时暴露了自身  
  
一款名为“远控”的Android远程访问木马，其防护薄弱的加载程序暴露的远不止隐藏的有效负载：它还会暴露每个版本中嵌入的操作员账户标识符。  
  
🔗  
https://gbhackers.com/android-remote-access-trojan/  
  
  
5  
. 研究人员发现数千个隐藏在非活动页面背后的钓鱼域名  
  
Check Point 研究人员发现一套大规模、多区域的钓鱼基础设施，该设施会将活跃的钓鱼活动隐藏在看似未被使用的域名背后，这些域名针对多个地区的机构和用户展开攻击。  
  
🔗  
https://blog.checkpoint.com/security/researchers-uncover-thousands-of-phishing-domains-hiding-behind-inactive-pages/  
  
  
6  
. 新型GhostBat安卓恶意软件瞄准60款银行及UPI应用以窃取密码  
  
一种新的GhostBat RAT变体针对印度使用的60款安卓金融应用程序包展开攻击，该恶意软件结合了基于辅助功能的PIN码捕获技术与透明覆盖层，可截获用户在合法的Google Pay和MobiKwik应用界面内的操作交互。  
  
🔗  
https://gbhackers.com/ghostbat-android-malware/  
  
  
7  
. 新型ROMULUS恶意软件平台利用六台服务器投放多种恶意负载  
  
ROMULUS是一个跨六台指定服务器运行的共享ClickFix投递平台，其确认的路由会分发被木马植入的Stardock软件包与LegionLoader加载器。  
  
🔗  
https://gbhackers.com/romulus-malware-platform/  
  
  
8  
. OpenCode 安全漏洞导致其控制台用户的第三方 API 密钥泄露  
  
OpenCode 披露了一起影响其 1.4% 控制台用户的安全漏洞，并警告称受影响工作区中保存的第三方 API 密钥可能已被泄露。在 10 月 17 日的一份声明中，该公司将此次事件与大规模蒸馏攻击关联起来，并表示正与 Anthropic、OpenAI 和谷歌合作解决该问题。  
  
🔗  
https://cybersecuritynews.com/opencode-security-breach/  
  
  
**漏洞事件**  
  
  
Vulnerability Incidents  
  
1. Drupal 修复关键远程代码执行漏洞，警告漏洞利用代码或很快出现  
  
Drupal 已为两个高危漏洞发布补丁，这些漏洞可能让攻击者在运行任何受支持版本的内容管理系统的网站上远程执行代码。  
  
🔗  
https://www.bleepingcomputer.com/news/security/drupal-patches-critical-rce-flaws-warns-exploits-may-emerge-soon/  
  
  
2. 严重 WordPress 漏洞允许攻击者在无需认证的情况下执行远程代码  
  
根据Seqrite研究员Amruta Wagh的分析，两个 WordPress Core 漏洞可以串联起来，在无需身份验证的情况下执行远程代码。  
  
🔗  
https://gbhackers.com/critical-wordpress-flaws/  
  
  
3. Next.js 发布带外补丁 修复缓存和 SVG 处理中的两个关键远程代码执行漏洞  
  
Next.js 于 10 月 14 日发布了一项带外安全更新，修复了两个可导致未授权远程代码执行的严重漏洞，以及一个高严重性的拒绝服务漏洞。一处代码执行漏洞存在于 React 服务器组件中，另一处则存在于 librsvg——Next.js 通过 sharp 包使用的开源库，用于处理 SVG 图像。  
  
🔗  
https://thehackernews.com/2026/10/nextjs-ships-out-of-band-patch-for-two.html  
  
  
4  
. 美国CISA 将 Fortinet 多款产品和 BeyondTrust 远程支持漏洞添加到其已知被利用漏洞目录中  
  
美国网络安全与基础设施安全局（CISA）将飞塔多款产品及BeyondTrust远程支持的漏洞加入其已知被利用漏洞目录。  
  
🔗  
https://securityaffairs.com/201178/security/u-s-cisa-adds-fortinet-multiple-products-and-beyondtrust-remote-support-flaws-to-its-known-exploited-vulnerabilities-catalog.html  
  
  
5  
. 可追溯至37年的OpenBSD内核漏洞可实现本地权限提升至root权限  
  
一段新披露的 OpenBSD 内核漏洞利用链结合了 共享内存记账漏洞和一个单独的越界写入漏洞，以获取 root 权限。这个最古老的漏洞可追溯到约37年前源自原始BSD源码的共享内存附加计数器。  
  
🔗  
https://gbhackers.com/openbsd-kernel-flaw/  
  
  
6  
. 更新 Chrome：谷歌修复 37 个安全漏洞，含 7 个高危漏洞  
  
桌面版更新将 Chrome 升级至 155.0.8059.79/80 版本（适用于 Windows 和 Mac 系统）以及 155.0.8059.79 版本（适用于 Linux 系统）。此次更新包含 37 项安全修复，其中 7 项被评为严重级别。  
  
🔗  
https://www.malwarebytes.com/blog/bugs/2026/10/update-chrome-google-fixes-37-security-flaws-including-seven-critical-bugs  
  
  
7  
. 黑客利用 SiteGuard WAF 漏洞以管理员身份执行命令  
  
黑客正在利用 SiteGuard 服务器版 Web 应用防火墙（WAF）中的操作系统命令注入漏洞，得以在受影响的设备上以 root 或系统权限执行命令。漏洞编号为 CVE-2026-106931，影响基于主机的 WAF 实例中的统计功能，该功能默认处于启用状态。  
  
🔗  
https://www.bleepingcomputer.com/news/security/hackers-exploit-siteguard-waf-flaw-to-run-commands-as-system-root/  
  
  
8  
. 黑客利用 Collabora Online 严重漏洞对 Nextcloud 服务器植入后门  
  
黑客正在利用 Nextcloud 专用的 Collabora Online 内置 CODE Server 应用程序中的一个严重漏洞，有用户报告在被入侵的服务器上发现了网页后门。该漏洞编号为 CVE-2026-110447，允许远程攻击者无需登录即可执行操作系统命令。  
  
🔗  
https://cybersecuritynews.com/hackers-exploit-critical-collabora-online-flaw/  
  
****  
扫码关注  
  
爱拍照的老李  
  
**讲述普通人能听懂的安全故事**  
  
  
