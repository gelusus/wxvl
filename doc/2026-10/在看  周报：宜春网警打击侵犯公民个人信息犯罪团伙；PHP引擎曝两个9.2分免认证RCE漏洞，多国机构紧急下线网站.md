#  在看 | 周报：宜春网警打击侵犯公民个人信息犯罪团伙；PHP引擎曝两个9.2分免认证RCE漏洞，多国机构紧急下线网站  
原创 管窥蠡测
                    管窥蠡测  安在   2026-10-24 11:43  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5eH7xATwT39x3nndZanlbZOk0K8ibhGOS3mskN8Bwx0bJ9KgpRsYSJ106ib1991xicajm3DeAcXdpEVl2icku02jBg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8C1NLS8ickpf8AS4j9iaCh4cKjtonIiaubP4yCic8hDDCODKJcSsyGd6ck64eUJsEuAtUolNtkogulKLDEu9M7H8GIaa2ibiakQZI1Gt4RKCfUYrE/640?wx_fmt=png&from=appmsg "")  
  
**[导读]**  
  
国内网警破获千万元级侵犯公民个人信息团伙，多地开展网络安全执法约谈；AI 智能体被用于批量发起网络攻击，多起平台 API 密钥泄露事件敲响数据防护警钟。同时大批高危漏洞集中爆发，PHP、GitLab、Node.js 等多款主流组件出现免认证 RCE 等严重缺陷，已出现在野武器化利用。海外多起大规模数据泄露、天价处罚与供应链攻击事件频发，企业需重视漏洞补丁、密钥轮换与 AI 工具接入风险。  
  
  
  
****  
**数据安全及个人信息保护**  
  
  
  
  
  
[1、宜春网警收网一个侵犯公民个人信息犯罪团伙，涉案1400余万元](https://mp.weixin.qq.com/s?__biz=MzU0MTA3OTU5Ng==&mid=2247584443&idx=1&sn=140895fbce5508696e90f3245b1c0743&scene=21#wechat_redirect)  
  
  
  
江西宜春公安网安部门对藏匿于写字楼内的侵犯公民个人信息犯罪团伙开展集中收网行动，端掉以罗某宇等人为首的团伙，当场抓获涉案人员23名，查扣电脑、手机等电子设备70余台。经查，该团伙非法获取大量公民个人信息，借助短信通道批量发送虚假银行贷款营销短信，为下游非正规贷款机构引流牟利，涉案金额1400余万元。23名涉案人员均被依法采取刑事强制措施，案件正在进一步侦办中。  
  
  
[2、Meta因违反新墨西哥州消费者保护法律被判罚250亿美元](https://mp.weixin.qq.com/s?__biz=MzIyNjUxOTQ0MQ==&mid=2247587950&idx=1&sn=a4abfab2c869caec1d215474c2e6e88a&scene=21#wechat_redirect)  
  
  
  
美国新墨西哥州第一司法地区法院裁定，社交媒体巨头Meta Platforms因违反该州消费者保护法律，须向州政府支付250亿美元（约合人民币1780亿元）民事罚款。陪审团认定Meta旗下Facebook长期在用户数据隐私、第三方数据访问及有害内容管理等方面作出虚假或误导性陈述，累计违法超过4389万次；法院认为按法律计算处罚金额原本可达约2195亿美元，综合考量后将罚款降至250亿美元，未清偿罚款按每年15%利率计息。  
  
  
[3、OpenCode Console遗留系统失陷，第三方大模型API密钥存泄露风险](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=3&sn=f20efa338f57605e69383fafbd726697&scene=21#wechat_redirect)  
  
  
  
AI编程辅助平台OpenCode证实，其托管服务OpenCode Console一套遗留系统遭黑客非法侵入，用户在控制台保存的OpenAI、Anthropic等第三方大模型供应商API密钥可能已泄露，已有开发者账户被盗刷数百至上千美元。OpenCode官方紧急向受影响的1.4%用户发送预警邮件，勒令其立即注销并轮换凭据。分析指出，API Key属持票人凭证，不校验调用方IP或设备指纹，黑客提取密钥后可绕过平台直连各厂商接口发起计费，控制台配额随之失效。  
  
  
[4、外包厂商Conduent因泄露6220万人数据，拟支付5.33亿元和解金](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516763&idx=1&sn=9d9dbbf83d424e23078496666730e65a&scene=21#wechat_redirect)  
  
  
  
美国业务外包厂商Conduent就一起数据泄露集体诉讼提出总额8000万美元（约合人民币5.33亿元）的和解方案，方案尚需法院批准。调查显示，未经授权的第三方入侵其网络并长期访问系统；取证后发现约6220万人的个人信息和受保护健康信息被窃取，涉及姓名、出生日期、医疗信息、医保信息及社会安全号码等。合并诉讼共提出16项指控，被告否认相关主张。  
  
  
[5、日经披露云服务遭未授权访问，超183万条个人信息记录泄露](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=2&sn=733eeac4e94a65c0f26e2056b11c597a&scene=21#wechat_redirect)  
  
  
  
日本传媒集团日经披露一起数据泄露事件，影响超180万条记录。攻击者目标是日经用于系统开发的云服务，黑客利用窃取的账号凭证获取访问权限。公告显示，约183万条个人信息泄露，其中约15万条包含姓名、邮箱地址等身份信息，剩余约168万条仅含无法识别身份的信息，泄露信息中均不含信用卡信息。日经封禁所有失窃凭证，未发现数据被滥用、系统宕机或篡改等进一步损害，并已向日本个人信息保护委员会上报、公开致歉。  
  
  
  
  
**AI安全**  
  
  
  
  
  
[1、AI Agent集群攻击致全球440台PaperCut服务器遭入侵](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
  
黑客利用数百个AI Agent对暴露在公网的PaperCut打印管理服务器发起大规模攻击，至少440台服务器遭入侵，波及48个国家的395家机构。攻击者借助OpenAI Codex、DeepSeek等AI模型自动开发、测试并优化漏洞利用代码，组合利用两个漏洞实现远程代码执行，随后收集系统凭据并向企业内网渗透。监测显示攻击全程高度自动化，官方建议及时修补漏洞并开展入侵排查。  
  
  
[2、孤狼黑客借AI智能体攻破上百家企业，含三家上市公司](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516749&idx=1&sn=9dbafca59e0c1859741353938cdb0b9d&scene=21#wechat_redirect)  
  
  
  
网络安全研究人员发现，一名孤狼黑客入侵100多家企业网站，其中包括美国农机制造商Deere、家电企业SharkNinja及发动机制造商Cummins等上市公司；该黑客几乎未亲自动手，而是指挥AI智能体自主运行，持续探测目标漏洞并实施入侵。NorForge研究员Snorre Fagerland在扫描意外暴露于公网的Cairn开源AI平台实例时发现了这名黑客。该黑客给出不到100个字的攻击策略指令，AI智能体已尝试入侵1200多个网站；Fagerland估计攻击混合使用DeepSeek与Claude Code，消耗模型令牌费用约5000美元，平均每次入侵成本约50美元。  
  
  
[3、印度CERT-In就多起AI智能体未授权访问事件发布紧急预警](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516738&idx=1&sn=5651b72f452d82baf0b293f3893adbfc&scene=21#wechat_redirect)  
  
  
  
印度计算机应急响应小组（CERT-In）发布“紧急”警告，部分测试阶段的自主AI智能体未经授权访问了印度多家组织的在用系统和机密数据，并称犯罪分子正利用公开AI工具加快攻击速度。公告介绍两类事件：一类中，AI模型及智能体超出任务范围，利用配置中暴露的凭据读取含个人信息与机密内容的业务记录，受影响组织未能自行发现，直到模型提供商披露后才曝光；另一类中，犯罪分子利用公开AI安全工具扫描目标、识别漏洞，针对印度实体发起攻击。  
  
  
[4、OpenCode披露遗留系统漏洞，1.4%控制台用户第三方API密钥存泄露风险](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349155&idx=2&sn=0a0815857a62c50d55dd32733ef18874&scene=21#wechat_redirect)  
  
  
  
AI编程平台OpenCode披露一起安全漏洞事件，1.4%的控制台用户受影响，受影响工作区内存储的第三方API密钥可能已遭泄露。公司称已向受影响用户发送邮件通知，未收到邮件者未受波及。通知显示，攻击者未授权访问一套原本准备下线的遗留系统，工作区中OpenAI、Anthropic等服务商密钥可能泄露；通知要求收件人在服务商控制台吊销更换密钥并更新应用，且仅称密钥存在泄露可能，并未确认所有密钥均被窃取或滥用。  
  
  
  
**网络安全**  
  
  
  
  
  
[1、程序员编写抢号软件致医院挂号系统瘫痪](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=1&sn=62452dec7ffbb0c77a86ee1e051297e4&scene=21#wechat_redirect)  
  
  
  
10月6日报道，一名程序员为帮女友挂上热门专家号，编写了模拟点击抢号的软件，每秒可点击千余次。二人随后收费帮他人抢号，共获利12900元。该软件与同类抢号工具同时抢号，导致武汉某医院挂号系统瘫痪、无法正常预约，医院随即报警。二人被抓获后，法院认定其构成破坏计算机信息系统罪，判处程序员有期徒刑十二个月，其女友系从犯，获刑十个月、缓刑一年。  
  
  
[2、鄂尔多斯两家企业因主机遭远控及高危漏洞被网信部门约谈整改](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516769&idx=1&sn=5380f484b019e9789c2b8037a8970533&scene=21#wechat_redirect)  
  
  
  
鄂尔多斯市委网信办检查发现，属地两家企业的信息系统存在未授权访问等高危漏洞，主机遭到远程控制并造成危害网络安全的后果。网信部门依据相关法律法规对企业负责人及运维人员开展执法约谈，要求企业立即排查整改风险隐患，建立常态化安全防护与应急处置机制，按期完成整改并报送复核材料。  
  
  
[3、京东JSRC大促重保期间，全面暂停外部白帽子测试与漏洞收录](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=1&sn=64985dae9e27b64b33a95e031a7e8784&scene=21#wechat_redirect)  
  
  
  
京东安全应急响应中心（JSRC）公告，在大促重保周期内全面暂停授权外部白帽子测试京东全部产品与业务系统，并暂停收录漏洞报告。叫停源于多起AI自动化渗透工具与智能体失控：测试者将脚本未经隔离接入生产接口，发送模糊测试载荷冲击网关与数据库连接池、干扰正常交易，大模型生成的低质报告也挤占审核资源。公告严禁AI工具直连生产环境与违规拖库；继续测试者将被封号、取消奖励，造成系统异常或数据泄露的将被追责。  
  
  
[4、PHP引擎曝两个9.2分免认证RCE漏洞，多国机构紧急下线网站](https://mp.weixin.qq.com/s?__biz=MzU5ODgzNTExOQ==&mid=2247658273&idx=3&sn=d47dfca560b06ff59fd11ad269da9c24&scene=21#wechat_redirect)  
  
  
  
PHP官方发布四个安全版本，修复两个未经身份验证的远程代码执行漏洞CVE-2026-109618与CVE-2026-109619，合称PHP2Shell，评分均为9.2；攻击者无需账号密码，发送构造请求即可执行代码并读取内存中的密码、令牌与API密钥。漏洞源于引擎内存安全缺陷，存在于PHP 5.4起各版本，由Calif在OpenAI“Patch the Planet”计划下借助GPT-5.6 Sol找到入口。事件引发多国应急响应，日本首相高市早苗发帖呼吁紧急应对，韩国KISA、欧盟ENISA及国内云厂商安全团队与奇安信CERT分别发布处置建议。  
  
  
[5、美国依据PAIPA制裁五家涉Google商业秘密案的中国企业](https://mp.weixin.qq.com/s?__biz=MzIyNjUxOTQ0MQ==&mid=2247587932&idx=1&sn=02e0d6fcf0fa31647229098f49e3eda9&scene=21#wechat_redirect)  
  
  
  
美国国务院依据《保护美国知识产权法》（PAIPA），以涉嫌参与或受益于商业秘密被窃取为由，制裁奇绩创坛及上海至算科技等5家中国企业；财政部OFAC将其列入非SDN菜单式制裁清单，实施资产冻结、融资与金融交易限制及投资禁令。制裁关联前Google工程师丁林葳案，美方指控其将含Google商业秘密的文件上传个人云端账户；法院裁定其窃取商业秘密罪成立。受制裁方包括丁林葳创立的上海至算科技，以及注资持股的奇绩创坛及关联创投中心。  
  
  
[6、昆明公积金系统因存储集群节点故障瘫痪停办线上线下业务](https://mp.weixin.qq.com/s?__biz=MzI4OTc4MzI5OA==&mid=2247902429&idx=1&sn=9ec9b4616c8bc0c2e7c92ef0c5f37e36&scene=21#wechat_redirect)  
  
  
  
昆明市住房公积金管理中心发布通告，称因存储系统集群节点故障、数据库运行异常，公积金系统无法正常办理业务，正组织技术力量紧急抢修。受此影响，该中心及省级职工住房资金管理中心所有线下柜面窗口、线上服务渠道的公积金业务同步暂停；停办期间12329热线及窗口咨询照常，贷款还款划扣同步暂停，系统恢复后统一补扣，期间不记逾期、不计罚息、不影响个人征信，恢复安排待另行通知。  
  
  
[7、Linux内核曝9.3分高危漏洞，可越狱虚拟机攻击宿主机](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=2&sn=870c96014815465341569810ecc94d94&scene=21#wechat_redirect)  
  
  
  
Linux内核被曝存在一个CVSS评分高达9.3分的严重漏洞。该漏洞源于网络栈对IPv4校验和元数据处理不当，攻击者可借此使内核将原本20字节的IPv4头误判为60字节，进而读取数据包之外的宿主机内存，实现进一步写入与代码执行。在KVM等虚拟化环境中，恶意客户机可利用该漏洞突破隔离边界、攻击宿主机，对云厂商、多租户沙箱及虚拟化平台风险尤为突出，相关用户应尽快升级内核至已修复版本。  
  
  
[8、黑客利用Collabora漏洞为Nextcloud服务器植入后门](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=3&sn=83fe8f1d0aa273a7e5c0ca6d4a83d64e&scene=21#wechat_redirect)  
  
  
  
威胁行为者正利用Nextcloud内置Collabora Online服务器应用中的严重命令执行漏洞发起攻击。该漏洞无需身份验证或用户交互，攻击者仅需向proxy.php端点发送恶意请求，即可触发操作系统命令注入，为服务器植入PHP webshell，进而搜索密码数据库、密码管理器工件和加密货币钱包。官方已发布修复版本，厂商提醒仅禁用该应用并不足以防御，管理员应尽快升级并排查入侵痕迹。  
  
  
[9、OpenBSD内核37年历史漏洞可提权至root](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=2&sn=23f8b93cfd061160e8aba3844107efa7&scene=21#wechat_redirect)  
  
  
  
安全公司披露OpenBSD内核存在一组可串联利用的本地提权漏洞，其中核心漏洞可追溯至37年前的BSD原始代码。漏洞源于共享内存挂载计数器采用16位有符号整数，本地用户反复派生子进程可令其溢出回绕，触发整数溢出与竞态条件，进而造成内核释放后使用，最终实现root权限获取。利用代码已公开，官方已发布补丁完成修复，目前未观测到相关漏洞被真实攻击活跃利用。  
  
  
[10、FoxIO披露HTTP/3协议栈QPACK内存放大DoS漏洞，七成主流协议栈受影响](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079445&idx=1&sn=faa0a86fd9d2ee022af099b2286cf0d2&scene=21#wechat_redirect)  
  
  
  
安全研究机构FoxIO披露HTTP/3协议栈一组严重拒绝服务漏洞并公开概念验证代码；未认证攻击者仅凭单条UDP连接发送少量特制压缩请求头，即可使服务端内存暴增至最高4000倍，耗尽物理内存并触发OOM崩溃，主流协议栈中70%存在缺陷。HTTP/3基于UDP上的QUIC并采用QPACK压缩请求响应头，诸多协议栈存在“先膨胀后检查”缺陷，未在解压过程中即时限制内存阈值，攻击者使极小载荷膨胀为数十万个请求头对象。  
  
  
[11、Node.js修复内置Brotli解压越界写RCE，22.x/24.x/26.x全系受影响](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079445&idx=2&sn=2cf06c774d54c3072b9333380e0d8051&scene=21#wechat_redirect)  
  
  
  
Node.js官方发布安全更新，一次性修复10处漏洞，最严重缺陷源于内置Brotli解压库：攻击者构造畸形压缩数据即可在解压中触发堆内存越界写入并导致远程代码执行，活跃分支22.x、24.x、26.x全部受影响，停更的20.x及旧版本永久暴露。缺陷位于底层绑定的Google Brotli C语言解压库，应用调用node:zlib解压数据流或使用fetch()接收br响应时都会触发。官方修复版本为26.12.1、24.22.1与22.23.4，20.x及更早版本不提供补丁。  
  
  
[12、GhostAction供应链攻击转向钓鱼劫持OAuth令牌，94个GitHub仓库遭假字体投毒](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=2&sn=5347b52ee72fe6b67a3af750295fed29&scene=21#wechat_redirect)  
  
  
  
开源供应链攻击“GhostAction”出现演变，GitHub拦截恶意Actions工作流后，攻击者改用反向代理钓鱼静默劫持开发者OAuth令牌，并向94个开源仓库植入二阶段恶意载荷；其滥用VS Code自动运行任务机制，将窃密程序伪装为字体文件fa-solid-400.woff2，开发者克隆打开后，云凭证、包发布令牌及大模型API Key即被窃取。攻击者注册仿冒域名my-github[.]com投递伪造告警邮件，反向代理截获密码与2FA验证码，后台自动完成OAuth设备流绑定。  
  
  
[13、GitLab曝CVSS 10.0免认证RCE漏洞CVE-2026-110264，旧版私有化实例均受影响](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079445&idx=3&sn=aefb0161b5fcc45e5ee5a8c61db25522&scene=21#wechat_redirect)  
  
  
  
GitLab官方紧急发布跨版本补丁通告，披露CVSS 10.0关键漏洞CVE-2026-110264；未授权攻击者无需登录，仅凭特制正则表达式即可触发整数溢出并达成任意代码执行，漏洞波及GitLab 7.0以来所有版本，自建私有化部署实例全部在射程内。漏洞位于内置打包的Ruby正则引擎Onigmo，编译特制正则时发生整数溢出，引发内存越界覆盖，攻击者据此劫持进程控制流并执行任意系统指令。官方为七个旧分支回溯发布补丁，云端租户已自动热修复；官方上线Sigma规则监控异常崩溃，升级补丁是唯一途径。  
  
  
[14、Plex Media Server四漏洞组合利用链Plex2Shell遭武器化，植入挖矿与代理后门](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=3&sn=c59fa44ef0a5791429f5ba4c7b36e7e4&scene=21#wechat_redirect)  
  
  
  
开源多媒体服务器Plex Media Server曝出“Plex2Shell”利用链，在野攻击者已将其武器化，对公网未打补丁服务器展开扫描与远程代码执行；攻陷后立即植入XMRig门罗币挖矿程序与隐蔽Linux代理后门，主机被纳入僵尸网络。利用链由4个中高危缺陷串联，无需认证仅凭外部HTTP请求即可提权执行，依次经无认证SSRF、参数注入窃取管理员令牌、转码通道SSRF与路径穿越四个环节落地恶意动态链接库并完成执行，对应CVE-2026-96652、96654、96655、96656。  
  
  
[15、多个勒索团伙利用Palo Alto GlobalProtect身份验证绕过漏洞入侵11个行业组织](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516757&idx=1&sn=f9f6348a423a0db56bbc2348f9f5914e&scene=21#wechat_redirect)  
  
  
  
派拓网络研究人员警告，旗下GlobalProtect产品漏洞披露后勒索团伙正加大利用；漏洞CVE-2026-0257为身份验证绕过漏洞，攻击者可未经授权建立VPN连接，严重性评分7.8。ReliaQuest发现Qilin、Settra等团伙利用该漏洞建立未授权VPN会话进入内网获取初始访问；攻击与合法会话行为难以区分、容易逃过检测，攻击者可窃取凭证、横向移动、窃取数据乃至部署勒索软件。  
  
  
**RECOMMEND**  
  
  
推荐阅读  
# ● 在看 | 周报：邮储分行因网络安全等问题被罚二百余万；深圳警方破获非法破解无人机飞控系统牟利案  
  
# ●在看 | 周报：福建连江农商银行因数据安全等问题被“双罚”574万元；字节跳动通报114名员工被辞退，8人涉罪移交司法  
#   
#   
  
  
[●](https://mp.weixin.qq.com/s?__biz=MzU5ODgzNTExOQ==&mid=2247657804&idx=1&sn=3a2b1f7772c2bd92e01e3a4087d71b38&scene=21#wechat_redirect)  
  
在看 | 周报：福建连江农商行因数据安全等7项违规，被罚574万元；OpenAI Agent再曝失控：攻击RubyGems代码托管平台  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5eH7xATwT3ibRpa5yOEg8zZ5voaDbvKA75C5LJDKwGFzKfWoxFGdDfCOUYRAiaH27KC1O6npvh4UbX3ZWliblsiatg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
  
扫码加入诸子云知识星球。  
  
  
  
**END**  
  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzU5ODgzNTExOQ==&mid=2247657060&idx=1&sn=9fdcb0cd7084523a3ea7fb24ce6e2c31&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzU5ODgzNTExOQ==&mid=2247652102&idx=1&sn=8af8808f9055f99fa71020922d80058c&scene=21#wechat_redirect)  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5eH7xATwT38j3Ndib8YhjyiaBQhdzUe1AArbhGtTxVSkCv3A7f6fo6n1dmFJEGEtMuDxGyeR81CndSR08OKTGQaw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5eH7xATwT38HPkvxLkOy5rLCeVBtj8H9SUbVPNZbibc4N2knPCDFjTKduRLhiaAZVQShUa2IZqsBShI2GG2dpqBg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
  
点击这里阅读原文  
  
