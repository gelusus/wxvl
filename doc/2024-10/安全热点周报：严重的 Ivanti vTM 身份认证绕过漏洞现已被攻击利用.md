#  安全热点周报：严重的 Ivanti vTM 身份认证绕过漏洞现已被攻击利用   
 奇安信 CERT   2024-09-30 17:00  
  
<table><tbody style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;"><tr bgless="lighten" bglessp="20%" data-bglessp="40%" data-bgless="lighten" style="-webkit-tap-highlight-color: transparent;outline: 0px;border-bottom: 4px solid rgb(68, 117, 241);visibility: visible;"><th align="center" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;border-width: 0px;border-style: none;border-color: initial;background-color: rgb(254, 254, 254);font-size: 20px;line-height: 1.2;visibility: visible;"><span style="-webkit-tap-highlight-color: transparent;outline: 0px;color: rgb(68, 117, 241);visibility: visible;"><strong style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;"><span style="-webkit-tap-highlight-color: transparent;outline: 0px;font-size: 17px;visibility: visible;">安全资讯导视 </span></strong></span></th></tr><tr data-bcless="lighten" data-bclessp="40%" style="-webkit-tap-highlight-color: transparent;outline: 0px;border-bottom: 1px solid rgb(180, 184, 175);visibility: visible;"><td align="center" valign="middle" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;border-width: 0px;border-style: none;border-color: initial;font-size: 14px;visibility: visible;"><p style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;">• 《工业和信息化领域数据安全合规指引》公开征求意见</p></td></tr><tr data-bglessp="40%" data-bgless="lighten" data-bcless="lighten" data-bclessp="40%" style="-webkit-tap-highlight-color: transparent;outline: 0px;border-bottom: 1px solid rgb(180, 184, 175);visibility: visible;"><td align="center" valign="middle" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;border-width: 0px;border-style: none;border-color: initial;font-size: 14px;visibility: visible;"><p style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;">• 国家安全部发文披露“台独”网军“匿名者64”</p></td></tr><tr data-bcless="lighten" data-bclessp="40%" style="-webkit-tap-highlight-color: transparent;outline: 0px;border-bottom: 1px solid rgb(180, 184, 175);visibility: visible;"><td align="center" valign="middle" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;border-width: 0px;border-style: none;border-color: initial;font-size: 14px;visibility: visible;"><p style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;">• 英国主要火车站紧急关停公共WiFi：因被黑后传播恐怖主义信息</p></td></tr></tbody></table>  
  
**PART****0****1**  
  
  
**漏洞情报**  
  
  
**1.cups-browsed远程代码执行漏洞安全风险通告**  
  
  
9月27日，奇安信CERT监测到官方修复cups-browsed远程代码执行漏洞(CVE-2024-47176)，该漏洞是由于cups-browsed服务在处理网络打印任务时，会绑定到UDP端口631的INADDR_ANY地址，从而信任来自任何来源的数据包。这会导致未经身份验证的远程攻击者可以利用该漏洞发送特制的数据包，触发恶意请求到攻击者控制的URL，从而在目标系统上执行任意命令。利用此漏洞需要启用cups-browsed服务（Ubuntu Desktop环境下默认启用），且需要受害者主动使用该恶意IPP服务器配置的打印机设备进行打印操作。目前该漏洞技术细节和POC已公开，鉴于该漏洞影响范围较大，建议客户尽快做好自查及防护。  
  
  
**PART****0****2**  
  
  
**新增在野利用**  
  
  
**1.****Ivanti vTM身份认证绕过漏洞(CVE-2024-7593)**  
  
  
9月24日，CISA 已将一个关键的 Ivanti 安全漏洞标记为攻击中积极利用的漏洞，该漏洞可让黑客在易受攻击的 Virtual Traffic Manager （vTM） 设备上创建流氓管理员用户。此身份验证绕过漏洞被跟踪为 CVE-2024-7593，是由于身份验证算法的实现不正确导致的，该算法允许未经身份验证的远程攻击者绕过 Internet 公开的 vTM 管理面板上的身份验证。  
  
Ivanti 在全球拥有 7,000 多家合作伙伴，其产品被 40,000 多家公司用于系统和 IT 资产管理。Ivanti vTM 是一款基于软件的应用程序交付控制器 （ADC），可为托管关键业务服务提供负载平衡和流量管理。“成功利用可能导致绕过身份验证并创建管理员用户，”Ivanti 在发布安全更新时警告说。  
  
虽然该公司表示，PoC代码在 8 月 13 日发布 CVE-2024-7593 补丁时已经可用，但它尚未更新安全公告以确认主动利用。但是，它建议检查通过 GUI 或公开可用的漏洞利用代码添加的新“user1”或“user2”管理员用户的审核日志输出，以查找泄露证据。  
  
Ivanti 还建议管理员通过将 vTM 管理界面绑定到内部网络或私有 IP 地址来限制对 vTM 管理界面的访问，以阻止潜在的攻击尝试并减少攻击面。  
  
最近几个月，Ivanti 的几个漏洞被利用为零日漏洞，以该公司的 VPN 设备以及 ICS、IPS 和 ZTA 网关为目标。该公司本月早些时候还警告说，黑客还将两个最近修复的云服务设备 （CSA） 漏洞链接在持续的攻击中。Ivanti 在 9 月表示，为了应对这些攻击，它已经增强了内部扫描和测试能力，目前正在努力改进其负责任的披露流程，以更快地解决潜在的安全问题。  
  
  
参考链接：  
  
https://www.bleepingcomputer.com/news/security/critical-ivanti-vtm-auth-bypass-bug-now-exploited-in-attacks/  
  
**PART****0****3**  
  
  
**安全事件**  
  
  
**1.英国主要火车站紧急关停公共WiFi：因被黑后传播恐怖主义信息**  
  
  
9月26日英国卫报消息，由于发生“网络安全事件”，英国境内19个主要火车站宣布暂停提供WiFi服务，其中伦敦有10个车站受到影响。有乘客在皮卡迪利站连接WiFi时，被引导到一个标题为“我们爱你，欧洲”的网页。该网页包含了反伊斯兰的信息以及关于英国和欧洲几起恐怖袭击的详细内容。英国铁路网公司（Network Rail）的发言人表示，“我们正在应对影响铁路网公司管理的车站公共WiFi的网络安全事件。由于WiFi服务由第三方提供，调查期间该服务已被暂停。”为该公司提供WiFi服务的Telent公司表示，WiFi登录页面遭遇“未经授权的更改”，该更改是通过该网页供应商Global Reach公司的“合法管理员账号”进行的。稍晚时候，英国交通警察局表示，一名受雇于Global Reach的男子已因涉嫌违反《1990年计算机滥用法》被捕。  
  
  
原文链接：  
  
https://www.theguardian.com/uk-news/2024/sep/26/wifi-suspended-big-uk-train-stations-cybersecurity-incident  
  
  
**2.美国汇款巨头速汇金遭网络攻击，支付服务中断约5天**  
  
  
9月24日BleepingComputer消息，全球第二大汇款公司速汇金（MoneyGram）23日发布公告称，自9月20日以来，该公司由于遭受网络攻击导致系统故障和支付服务中断。公告中指出：“速汇金近期发现了一个影响我们部分系统的网络安全问题。我们在发现问题后，立即展开调查，并采取了保护措施应对此次事件，其中包括主动下线部分系统，这对网络连接产生了影响。”该公司此前一直声称这是一次“网络故障”，直到23日才披露是一次“网络安全事件”，但并未披露攻击细节，媒体认为该事件的特征与勒索软件攻击极为类似。25日下午，速汇金在X平台上称，许多代理合作伙伴的汇款和收款服务已经恢复，待处理的交易正在陆续完成，公司将继续恢复剩余的其他服务，包括官方网站。  
  
  
原文链接：  
  
https://www.bleepingcomputer.com/news/security/moneygram-confirms-a-cyberattack-is-behind-dayslong-outage/  
  
  
**3.背调公司发生超大规模数据泄漏，一亿美国人隐私信息暴露**  
  
  
9月23日Cybernews消息，网络安全研究机构Cybernews发现，美国背景调查和公共记录服务公司MC2 Data发生了大规模数据泄露事件，由于数据库没有设置密码保护，暴露了该公司2.2TB的敏感数据。这些数据中包含超过1亿美国公民的个人信息，涉及姓名、电子邮箱、电话号码、家庭住址、加密的密码、部分支付信息、房产记录、法律记录、就业经历、家庭亲戚及邻居信息等，严重威胁了个人隐私与信息安全。这是Facebook（2019年5亿）和Linkedin（2021年7亿）之后，近年来美国发生的最严重的数据泄漏事件。MC2 Data旗下运营的网站包括PrivateRecords.net、PrivateReports、PeopleSearcher、ThePeopleSearchers以及PeopleSearchUSA。这些网站通过汇集公开数据，如犯罪记录、就业历史、家庭信息和联系方式，提供背景调查服务，广泛应用于雇主、房东等作出决策时的参考。  
  
  
原文链接：  
  
https://cybernews.com/security/us-mc2-background-check-data-leak/  
  
  
**4.国家安全部发文披露“台独”网军“匿名者64”**  
  
  
9月23日国家安全部消息，国家安全部公众号发布文章《起底“台独”网军“匿名者64”》称，今年以来，一个名为“匿名者64”的黑客组织，针对中国大陆和港澳地区，频繁开展网络攻击，试图获取有关门户网站、户外电子屏幕、网络电视等控制权限，进而非法上传、插播诋毁大陆政治制度和大政方针的内容，颠倒黑白，散布谣言。对此，国家安全机关立即行动，采取有效措施，处置危害隐患，消除不良影响。深入调查确认，“匿名者64”组织并非普通黑客，而是由“台独”势力豢养的一支网军。目前，国家安全机关锁定了实施相关网络攻击活动的台湾人员身份信息，其中包括台湾资通电军现役人员罗俊铭、洪莉棋、廖韦纶。国家安全机关已依法对三人立案侦查。  
  
  
原文链接：  
  
https://mp.weixin.qq.com/s/uYOStO0FOZxaHQ0dOVuDhA  
  
  
**PART****0****4**  
  
  
**政策法规**  
  
  
**1.《工业和信息化领域数据安全合规指引》公开征求意见**  
  
  
9月29日，中国钢铁工业协会、中国有色金属工业协会、中国石油和化学工业联合会、中国电子信息行业联合会、中国通信标准化协会、中国通信企业协会等十七家行业组织共同编制《工业和信息化领域数据安全合规指引（征求意见稿）》，现面向成员单位公开征求意见。该文件共9个章节，包括概述、数据分类分级、数据安全管理体系、数据全生命周期保护、数据安全风险监测预警&报告&处置、数据安全事件应急处置、数据安全风险评估、数据出境、数据交易。该文件聚焦数据处理者在履行数据安全保护义务过程中的难点问题，明确数据安全合规依据，提供实务指引，指导数据处理者开展数据安全合规管理，以提升数据安全保护能力。  
  
  
原文链接：  
  
https://ccsaweb.eos-beijing-7.cmecloud.cn/ccsa/20240929/b22edba6e82b452fb564f1353138fb76/b22edba6e82b452fb564f1353138fb76.pdf  
  
  
**2.国家网信办《终端设备直连卫星服务管理规定》公开征求意见**  
  
  
9月27日，国家互联网信息办公室会同有关部门起草了《终端设备直连卫星服务管理规定（征求意见稿）》，现向社会公开征求意见。该文件共5章41条，包括总则、发展与促进、设备设施与服务管理、监督管理和法律责任、附则。该文件要求，终端设备直连卫星服务有关的卫星通信系统、关口站、地球站、配套通信平台等基础设施建设活动，应当符合网络安全、数据安全和国防建设相关法律法规；终端设备直连卫星服务提供者应当做到与国家安全、网络安全需求同步规划、同步建设、同步运行，履行网络安全、数据安全和个人信息保护义务，落实网络安全等级保护制度、通信网络安全防护制度、数据分类分级保护制度，采取必要措施保障数据和个人信息安全，防范违法犯罪活动，保障卫星通信系统安全稳定运行；终端设备直连卫星服务提供者应当履行电信网络诈骗风险防控义务，建立反电信网络诈骗内部控制机制和安全责任制度，开展终端设备直连卫星服务涉诈风险安全评估。  
  
  
原文链接：  
  
https://www.cac.gov.cn/2024-09/27/c_1729036112375138.htm  
  
  
**3.国家数据局《关于促进数据产业高质量发展的指导意见》公开征求意见**  
  
  
9月27日，国家数据局会同有关部门研究起草了《关于促进数据产业高质量发展的指导意见》，现向社会公开征求意见。该文件共9章，包括总体要求、加强数据产业规划布局、培育多元经营主体、加快数据技术创新、提高数据资源开发利用水平、繁荣数据流通交易市场、强化基础设施支撑、提高数据领域动态安全保障能力、优化产业发展环境。该文件在安全保障部分共两方面。一是创新数据安全产品服务。推动基础设施安全、数据安全、应用安全协同发展，加强身份认证、数据加密、安全传输、合规检测等技术创新，培育壮大适应数据流通特征和人工智能应用的安全服务业态。支持企业创新数据分类分级、隐私保护、安全监测、应急处置等数据安全产品和服务。二是加强动态数据安全保障。扩大数据空间、区块链、隐私计算等可信流通技术及模式应用范围，增强数据可信、可控、可计量开发利用能力。建立健全数据安全风险识别、监测预警、应急处置等相关规范，落实数据流通利用全过程相关主体的安全责任。健全数据分类分级标准，加强对涉及国家安全、商业秘密、个人隐私等数据的保护。  
  
  
原文链接：  
  
https://mp.weixin.qq.com/s/KgbREe03PKX-6ZGvcxUXxw  
  
  
**4.国家数据局《关于促进企业数据资源开发利用的意见》公开征求意见**  
  
  
9月27日，国家数据局会同有关部门研究起草了《关于促进企业数据资源开发利用的意见》，现向社会公开征求意见。该文件共7章16条，包括总体要求、健全企业数据权益实现机制、培育企业数字化竞争力、赋能产业转型升级、服务经济社会高质量发展、营造开放透明可预期的发展环境、保障措施。该文件要求，落实国家数据分类分级保护制度要求，在防范实质性风险前提下，鼓励企业针对不同敏感级别的数据和数据处理场景，采取差异化的数据安全与合规管理措施，优化对同类型数据处理行为的内部合规审批流程。鼓励企业采用数据空间、区块链、隐私计算、匿名化等技术模式，促进数据安全流动和开发利用。  
  
  
原文链接：  
  
https://mp.weixin.qq.com/s/_a0us8No4vJGLbi-s5kwnA  
  
  
**5.欧盟发布《数据治理法案》指导文件，进一步明确非个人数据跨境流动规则**  
  
  
9月24日，在《数据治理法案》推出一周年后，欧盟委员会发布了一份关于实施《数据治理法案》的指导文件。该文件明确指出，《数据治理法案》的适用范围包括个人数据和非个人数据。对于个人数据，《通用数据保护条例》和《电子隐私条例》将同时适用。该文件重申，《数据治理法案》不会修改《通用数据保护条例》，也不涉及《通用数据保护条例》中监管机构的作用。该文件还指出，在非个人数据跨境流动规则方面，《数据治理法案》仅为非个人数据的跨境转移设定规则，个人数据的跨境转移仍受《通用数据保护条例》第五章规定的约束。  
  
  
原文链接：  
  
https://digital-strategy.ec.europa.eu/en/library/new-practical-guide-data-governance-act  
  
  
**6.美国拟禁用中国、俄罗斯智能网联汽车软硬件**  
  
  
9月23日，美国商务部工业与安全局发布《保障信息和通信技术及服务供应链：网联汽车》的拟议规则通知，并于9月26日正式公布了文本，面向公众征求意见。该文件提出了一项规则，旨在解决由于某些外国对手（如中国和俄罗斯）设计、开发、制造或供应的信息和通信技术及服务用于网联汽车而带来的国家安全风险。该文件重点关注集成到车辆连接系统（VCS）中的硬件和软件（例如蓝牙、卫星、蜂窝和Wi-Fi模块）以及集成到自动驾驶系统（ADS）中的软件。这些关键系统通过特定的硬件和软件，可以在网联汽车中实现外部连接和自动驾驶功能。该文件将禁止2027年款的车辆进口VCS、ADS软件，以及2030年款的车辆进口硬件。小规模生产者可能会获得豁免，以防止行业混乱。  
  
  
原文链接：  
  
https://www.federalregister.gov/documents/2024/09/26/2024-21903/securing-the-information-and-communications-technology-and-services-supply-chain-connected-vehicles  
  
  
**往期精彩推荐**  
  
  
[【已复现】cups-browsed 远程代码执行漏洞(CVE-2024-47176)安全风险通告第二次更新](https://mp.weixin.qq.com/s?__biz=MzU5NDgxODU1MQ==&mid=2247502217&idx=1&sn=58b66c5fd01549ef7633e6a8ae104491&chksm=fe79ed11c90e64070b4bb5460a15a15b98547f02a41cfa58d6fda022edf4c224827d9e4ca2b2&token=78969541&lang=zh_CN&scene=21#wechat_redirect)  
[cups-browsed 远程代码执行漏洞(CVE-2024-47176)安全风险通告](https://mp.weixin.qq.com/s?__biz=MzU5NDgxODU1MQ==&mid=2247502207&idx=1&sn=cfc9b1847e536079d77a15b62c233f71&chksm=fe79ede7c90e64f152e40ebd2eb9ea5c25f60156788a4a3ba17b585916170400fcc6cb8739d3&token=78969541&lang=zh_CN&scene=21#wechat_redirect)  
  
[安全热点周报：时隔一周，Ivanti 又公开一云服务设备漏洞正面临在野利用](https://mp.weixin.qq.com/s?__biz=MzU5NDgxODU1MQ==&mid=2247502185&idx=1&sn=4c979dfd0e63af2bb96c91360d9e23d9&chksm=fe79edf1c90e64e7751ef785fcb7e26cf09d317648d2b36002ac1b42ec41a60b323d60527343&token=78969541&lang=zh_CN&scene=21#wechat_redirect)  
  
  
  
  
本期周报内容由安全内参&虎符智库&奇安信CERT联合出品！  
  
  
  
  
  
  
  
  
