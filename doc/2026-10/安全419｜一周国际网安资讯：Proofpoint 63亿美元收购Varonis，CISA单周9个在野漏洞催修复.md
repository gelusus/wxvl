#  安全419｜一周国际网安资讯：Proofpoint 63亿美元收购Varonis，CISA单周9个在野漏洞催修复  
原创 安全419
                    安全419  安全419   2026-10-20 09:42  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwNB1SjIThfjSEBgCCMBicZ4vpTM8K98Y6hHVAANuLaxJC0Mq6hkoDhjHWxANt9DCvPdRwiajXxU9IaxRQLreuojNlgU6NSLiaznzE/640?wx_fmt=gif&from=appmsg "")  
  
**一周热点导读**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwM9buPUThFVXoBvS3icTjSDjPicu3q5EGCzGkLTMKtI04KJQTqDp2F7Lnc8LWuQzxXWUPMEiaZaWXPBcaecZjg7UDovM6s0ojHFv8/640?wx_fmt=gif&from=appmsg "")  
  
  
  
上周（2026年10月14日至10月20日）全球网络安全领域两线并进。产业侧，Proofpoint于10月15日宣布以63亿美元全现金收购数据安全公司Varonis，成为2026年网安行业最大并购案之一；以色列AI治理初创Pluto Security出隐身即获4200万美元，应用安全厂商Harmonic Security完成5000万美元B轮，Crunchbase报告显示Q3全球网安投资达68亿美元。漏洞侧，CISA KEV目录单周新增9个在野利用漏洞，覆盖Cisco Secure Email Gateway、Palo Alto PAN-OS、BeyondTrust Remote Support、Fortinet、AhsayCBS等8家厂商，联邦机构修复截止日期密集落在10月17日至19日。事件侧，美国马里兰州Anne Arundel County确认Rhysida泄露2.6TB数据波及59万居民；巴西爱因斯坦医院、日本熊本福田医院、意大利Alma Center医疗中心一周内接连被攻；FBI紧急预警Kali365钓鱼平台专门窃取Microsoft 365 OAuth令牌、完全绕过多因素认证。以下为安全419整理的重点资讯。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9Nf1wzzcfwMHYtvaiacAvpNdEfD0dVfyxnIz3zOejHrAftYCtNqtz8C6NmUveiag8aUmp8gDp5Z4tjic95Z39WI83ibbI7ngCYMyHwCFx9FqER0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwNrVaoencoj5wqVHRRibfGnsOusmVGbPIzVR6mCVbr6nBicRDqPdficVv4GuMwT2JFibwh1CLZG1GZa62O3SIotzSvsH1aQ7JqRvia0/640?wx_fmt=gif&from=appmsg "")  
  
**一、企业并购与投融资**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwPHyCDgPaEeq6LprCIsNLMM0S1HoYvFj0jibJLibzUep61SYy4EwyDYmKmzb7DPzUsgBtl6USNUV7mnX1MzlqMxhQaCK4rutWUeI/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**Proofpoint宣布63亿美元全现金收购Varonis**  
  
美国邮件和身份安全厂商Proofpoint于10月15日宣布，已与数据安全公司Varonis Systems签署最终收购协议，以约63亿美元全现金交易完成收购。这是2026年网络安全行业最大并购案之一，合并后公司将把Proofpoint在邮件安全、身份防护和AI代理安全方面的能力，与Varonis在数据安全态势管理（DSPM）、敏感数据发现和权限分析方面的能力整合为一体。Proofpoint在公告中强调，AI时代最大的安全问题已经从"守住边界"转向"保护数据本身"——当员工通过AI工具和AI代理自由访问企业数据时，传统的边界防护失效，必须知道敏感数据在哪里、谁能访问、是否异常。这笔交易是Proofpoint继此前收购Hornetsecurity、扩展Microsoft 365防护之后的又一次大型整合，也是Varonis在多次寻求战略选择后的最终归宿。市场普遍认为，这一交易将推动数据安全与邮件安全厂商进一步合并，形成覆盖"员工身份—AI代理—数据流动"全链路的新安全平台。对国内安全厂商而言，Proofpoint-Varonis的整合方向提示：未来安全平台的竞争不再是单一产品功能比拼，而是身份、邮件、数据、AI代理四条数据线的横向打通能力。  
  
  
**Harmonic Security完成5000万美元B轮融资**  
  
应用安全测试初创公司Harmonic Security于10月16日宣布完成5000万美元B轮融资。Harmonic Security此前已经在2023年底由Ten Eleven Ventures领投850万美元种子轮，并在2024年10月由西门子旗下投资部门next47领投1750万美元A轮。公司累计融资规模已超过7500万美元。Harmonic Security的产品定位是帮助企业在软件开发生命周期中持续发现和验证安全漏洞，结合自动化测试和人工渗透测试，服务对象以金融、科技和大型企业为主。本轮融资反映出在AI辅助代码生成普及后，企业应用发布速度加快、漏洞数量同步上升，传统的"上线前人工渗透测试"模式已经跟不上CI/CD节奏，自动化持续测试平台的需求快速增长。Harmonic Security的融资节奏与本周Proofpoint收购Varonis、Pluto Security出隐身形成呼应——应用安全、AI安全、数据安全三个赛道在本周同时获得大额资本注入，显示2026年下半年网安投资正在从通用EDR/XDR转向更细分的工作流安全。对企业用户而言，应用安全测试预算应从"年度渗透测试"转向"持续自动化测试+按需人工验证"的组合模式。  
  
  
**以色列初创Pluto Security获4200万美元融资**  
  
以色列AI安全初创公司Pluto Security于10月13日正式退出隐身模式，并在10月16日披露已完成4200万美元融资，其中包括850万美元种子轮（由TLV Partners领投）和3400万美元后续轮。公司定位是帮助企业发现和治理员工"自建AI工具"产生的影子AI风险——当员工在工作中自行使用未批准的ChatGPT、Claude、Copilot或自建的AI代理时，会把企业敏感数据上传到未经批准的模型，产生数据泄露和合规风险。Pluto Security的平台通过网络流量分析、API审计和云端日志，自动发现员工在使用哪些AI工具、上传了什么类型的数据、是否触发DLP策略，并提供统一的治理和管控界面。随着企业AI部署从"官方批准的少数工具"扩散到"员工自建的数百个代理"，影子AI管理正在成为新的安全品类。Pluto Security的融资规模在出隐身阶段即达到4200万美元，显示资本市场对AI治理赛道的高预期。这与Rein Security（AI代理运行时防护）、Hadrian（AI代理渗透测试）本周的融资形成"AI安全一周三连投"，提示企业在部署AI代理时必须同步规划AI治理与运行时安全能力。  
  
  
**Crunchbase：2026年Q3网安投资68亿美元**  
  
Crunchbase News于10月15日发布2026年第三季度网络安全投资报告，Q3全球网安风投总额达到68亿美元，主要流向大额融资轮次和AI国防类初创公司。报告指出，与前几个季度相比，Q3的投资活动更加集中——早期小额种子轮数量下降，而B轮以后的大额融资和战略并购（如本周Proofpoint收购Varonis）占比上升。报告特别提到Aura公司通过合并和品牌整合快速做大，反映出"平台化整合"正在取代"单点产品"成为市场主流。报告认为，AI正在重塑网安投资逻辑：一方面AI被攻击者用于自动化漏洞挖掘和钓鱼个性化，另一方面AI也被防御方用于SOC自动化、威胁狩猎和漏洞验证，两类需求都在催生新的初创公司。这一数据与本周Proofpoint 63亿美元收购、Harmonic 5000万美元B轮、Pluto 4200万美元出隐身等事件形成印证——2026年下半年网安投资并未遇冷，而是从"广泛撒网"转向"押注平台型和AI原生型公司"。对国内初创公司而言，跨境融资窗口仍在，但估值逻辑已经从"产品叙事"转向"客户留存+收入增速+AI能力"三个硬指标。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwMgFqHJJzdtTwp02AETBkibia9icJ5j9Nlj711edCedF9ibxuRKsxg34rcNVs4a2JudA2BHN9FrmVPJq9J1KJp3h2zbs3VOEKWZCd0/640?wx_fmt=gif&from=appmsg "")  
  
**二、漏洞预警**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwP3JGX1NewCuJbeFwGSyeSQiaAheJG8v8zDGEaOS37Cs38ScwNWDW2kTPhCUpaw0Ln1YHsg3WOS2G5Wew0COWaiaXDnfNicbgmR00/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**CISA KEV新增4个在野利用漏洞**  
  
CISA于10月14日将4个确认遭在野利用的漏洞加入已知被利用漏洞（KEV）目录，要求联邦民事机构在10月17日前完成修复。本次新增包括：Cisco Secure Email Gateway中的路径遍历漏洞CVE-2026-76519——该产品负责读取企业所有入站邮件，一旦被攻破，攻击者可直接在邮件流中注入恶意附件；Palo Alto Networks PAN-OS中的一个安全绕过漏洞；以及备份管理控制台AhsayCBS中的两个漏洞，两者可链式组合利用。  
  
  
**CISA将零日漏洞CVE-2026-102764加入KEV**  
  
CISA于10月16日将BeyondTrust Remote Support中的严重零日漏洞CVE-2026-102764加入KEV目录，要求美国联邦机构在10月19日前完成修复并开展取证排查。据安全厂商Rescana分析，该漏洞自2026年8月起即被威胁行为者在野外利用，BeyondTrust随后发布了补丁。BeyondTrust Remote Support是企业广泛部署的远程支持和特权访问管理（PAM）平台，IT和运维人员用它远程连接员工电脑和服务器，一旦该平台本身被攻破，攻击者即获得"合法"的远程管理通道，可以绕过终端检测和网络分段直接进入任何受支持资产。CISA在指令中特别要求机构在修复后检查Remote Support管理日志，确认8月以来是否存在异常远程会话或未授权账号创建。这一事件再次敲响PAM平台安全的警钟——企业把最高权限集中到PAM后，PAM本身成为攻击方的首选目标，PAM供应商的安全补丁响应速度直接决定企业风险敞口。建议所有使用BeyondTrust的企业在本周内完成升级，并审计近90天的远程会话记录，确认是否有未授权访问。  
  
  
**CISA将Fortinet CVE-2025-61624加入KEV**  
  
CISA于10月16日将Fortinet产品中的CVE-2025-61624加入KEV目录，要求联邦机构在10月19日前完成修复。历史经验表明，Fortinet产品的严重漏洞一旦公开，通常在数天到数周内就会被勒索软件团伙和初始访问经纪人（IAB）大规模扫描利用，成为入侵企业网络的首选入口。CISA同时记录到，初始访问经纪人已将多个Fortinet漏洞的访问权限在地下市场转售，接入后通常会进一步部署勒索软件  
  
  
**微软修补Office Word远程代码执行漏洞**  
  
微软于10月15日发布安全更新，修补Microsoft Office Word中的远程代码执行漏洞CVE-2026-108426。该漏洞允许攻击者通过构造恶意Word文档，在用户打开文档时以当前用户权限执行任意代码。微软建议企业管理员通过Microsoft Update或Office部署工具在本周内完成更新，并对员工进行"不要打开来自陌生发件人的Office附件"的专项提醒。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwNCe5NRFoQvfHsT6ufiaicjB1Qa0wvynTiaCIUfkUe741Ap8Fevee6k470iaVvtYFfocRzAXptku9nt542icfTPFAHvcawyYUAVLYWw/640?wx_fmt=gif&from=appmsg "")  
  
**三、安全事件**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwN2TPT0nwGr6ibIcNHA66wATUyxMOQJNkcpyssDw1W2gmqLvLaXjz5XCdHarjIXv3JgNJjJYKAN0I4q09BvPD1xibib4Yq0W3emeY/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**巴西爱因斯坦医院确认患者数据库被未授权访问**  
  
巴西圣保罗顶级私立医院Einstein Hospital Israelita于10月8日发布通告，并在10月15日经多家媒体报道确认，医院在8月4日发生的网络安全事件中，部分患者数据库遭到未授权访问。事件已由巴西联邦警察（Polícia Federal）立案调查，并通报巴西国家数据保护局（ANPD）。医院在通告中表示，已经隔离受影响系统并启动外部取证，但未披露受影响患者人数。这是2026年巴西医疗行业第二起重大数据泄露，与意大利Alma Center医疗中心、西班牙Sant Pau医院被勒索在同一周，形成"全球医疗行业一周三连击"的局面。  
  
  
**日本熊本福田医院预约系统遭网络攻击**  
  
日本熊本县福田医院（Fukuda Hospital）于10月14日通报，该院产科和妇科预约系统因一起"大规模网络攻击"而中断。医院于当日收到可能存在未授权访问的通知，虽然门诊诊疗本身尚未受影响，但预约管理系统被迫暂停，导致部分就诊安排延误。医院正在与外部安全厂商合作调查事件范围，并确认是否有患者个人信息被访问。  
  
  
**意大利Alma Center医疗中心被SpaceBears公开点名**  
  
勒索追踪机构Yazoul Security于10月15日记录到，勒索软件组织SpaceBears将意大利Alma Center Servizi Medicali S.r.l.（阿尔玛中心医疗服务公司）列入受害者名单，标注为"严重"级别。SpaceBears是2025年开始活跃的双重勒索团伙，主要攻击欧洲医疗、制造和服务业企业，通常在攻击后先窃取数据、再加密系统，并在公开网站上释放部分文件作为施压手段。与Alma Center同日被记录的还有西班牙巴塞罗那的Hospital de la Santa Creu i Sant Pau（圣保罗医院），被勒索组织thegentlemen列入名单。  
  
  
**Nightspire勒索组织攻击美国公立学区**  
  
勒索追踪网站Ransomware.live于10月15日记录到，勒索软件组织Nightspire将美国一所公立学区列入受害者名单。记录显示攻击发生日期为9月14日，10月15日被公开。Nightspire是2026年活跃的新型勒索团伙，以教育行业为主要目标之一。美国K-12学区长期是勒索攻击的重灾区——学校普遍IT预算紧张、系统老旧，且因业务连续性压力往往选择支付赎金。学区数据包含学生姓名、出生日期、家庭住址、家长联系方式和特殊教育记录，一旦泄露会导致儿童身份盗用风险。建议所有学区立即审查远程桌面和VPN访问日志，启用MFA，并将备份系统与生产网络物理隔离，避免9月入侵后10月才被公开的"延迟发现"情况。国内教育行业虽然勒索攻击密度低于美国，但学生信息保护合规要求逐年提高，同样需要把教育系统作为重点保护对象。  
  
  
**微软披露JadePuffer代理式勒索后续**  
  
威胁情报机构本周（10月15日前后）报告，曾主导首个已知代理式勒索软件JadePuffer的网络犯罪团伙，再次利用窃取的Azure云身份对企业云存储等资源发动破坏性攻击。JadePuffer是2025年底出现的新型勒索软件，区别于传统文件加密型勒索，它利用云环境的原生API删除和破坏对象存储、数据库和虚拟机，造成业务中断但不留下传统加密痕迹。攻击者通过钓鱼或凭证填充获得Azure AD用户凭据后，使用合法的Azure CLI和PowerShell命令删除快照、清空存储账户，由于操作本身是"合法云API调用"，传统的防勒索软件检测无法识别。微软建议企业：对所有高权限云身份启用PIM（特权身份管理）和JIT临时提权；启用Azure Monitor审计日志，对批量删除和异常API调用告警；为关键存储账户启用软删除和版本化，并将备份与生产订阅隔离。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwMSMoX18VrtPkZzWVeDYianDdzrunEDjO0MwGJ1Xey74EqC1sBS9vjHL3atO93D3VU7y4ktToI2Lp8x2vJNPicfmCxbJVERib73IA/640?wx_fmt=gif&from=appmsg "")  
  
**四、攻防技术与监管动态**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwNUYplHzRsXsTqNERP8mj42MUgZ1Na03icLicM91W9Nicaic2vMJMIUtKSGqwEstiaDI6U0J8mnGoVN821XOZMRiaahEaQXWff5VJexc/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**FBI紧急预警Kali365钓鱼平台**  
  
美国FBI互联网犯罪投诉中心（IC3）本周就名为Kali365的钓鱼即服务平台发布紧急预警，该平台专门针对Microsoft 365用户，核心目标是窃取OAuth访问令牌和刷新令牌，能够在完全不触碰用户密码、不触发短信验证码的前提下绕过多因素认证（MFA）防线。Kali365并非传统凭证填充工具，而是一个完整的钓鱼基础设施：攻击者向企业员工发送伪装成"共享文档""待审批事项"的邮件，引导用户访问仿真的Microsoft登录页，用户完成正常登录后，恶意应用会请求OAuth授权，用户一旦同意，攻击者即获得长期有效的刷新令牌，可以持续访问用户邮件和OneDrive文件而无需再次登录。FBI建议企业在Azure AD中审查所有OAuth应用授权，撤销可疑应用；对高风险用户启用条件访问策略，限制陌生设备和位置登录；并对员工进行OAuth授权场景的专项培训——"点击同意"也是一种身份验证行为。  
  
  
**微软披露钓鱼活动滥用合法签名RMM软件M360**  
  
微软安全响应中心本周持续跟踪到多起钓鱼活动，攻击者分发经过伪装的、由合法数字签名的MSP360远程监控与管理（RMM）软件安装包，将其作为进入企业网络后的持久化访问通道。攻击链为：攻击者先通过钓鱼邮件或凭证填充获得员工Office 365凭据，然后发送伪装成"IT支持远程协助"的邮件，诱导员工下载安装被篡改的MSP360包，安装后攻击者即可远程控制员工电脑、横向移动并部署勒索软件。微软建议企业对所有RMM软件实施安装白名单管控，监控异常的MSP360/Chat/ScreenConnect等工具的出站连接，并在条件访问策略中限制新设备登录。这一攻击模式与本周Kali365 OAuth令牌攻击形成"两条入口"——一个通过OAuth应用授权绕MFA，一个通过合法RMM软件绕EDR，企业需要同时防御。  
  
  
**FBI在11月中期选举前双重预警**  
  
美国FBI互联网犯罪投诉中心（IC3）于10月15日同日发布两份选举安全预警：一份警告选举工作人员面临的威胁，另一份提醒选民警惕选举相关犯罪。FBI警告，选举工作人员——包括地方选举办公室职员、投票站志愿者和计票人员——可能面临来自国内外行为者的骚扰、胁迫和网络攻击，目的是干扰选举正常进行或制造对选举公正性的怀疑。针对选民的预警则覆盖选举诈骗（假冒选举官员索取个人信息）、虚假投票点信息钓鱼、以及通过电话和短信散布的选举谣言。FBI建议地方选举办公室立即审查选举系统访问日志、启用MFA、对投票机和计票系统实施物理隔离，并与当地FBI外勤办公室建立联络通道。  
  
  
**IBTimes综述：日本进入网络攻击紧急状态**  
  
IBTimes UK于10月14日发表综述，标题为"日本进入网络攻击紧急状态：勒索浪潮冲击主要品牌并暴露数百万记录"，系统梳理了过去两周日本发生的多起重大网络安全事件。综述指出，软银集团旗下IDC Frontier云服务勒索（495家企业和地方政府客户受影响）、日水物流云数据中心勒索、熊本福田医院预约系统中断、Advantest半导体测试设备巨头员工PII泄露等事件，在时间上高度集中，构成日本近年来最严重的一波勒索浪潮。受影响行业从云服务、物流、医疗到半导体制造，覆盖日本经济多个关键节点。日本经济产业省和厚生劳动省已召开紧急会议，要求关键基础设施运营者加强云租户权限管理和离线备份。综述同时指出，日本企业过去依赖"集团内垂直整合"的IT治理模式，但在云服务和外包趋势下，租户管理员凭据成为新的薄弱点。这一事件对亚洲其他国家的企业也是警示——云环境并不天然安全，第三方云服务商被攻破时，所有租户同时暴露。国内企业在选择云服务商时，应把云服务商自身的安全事件响应能力和数据隔离能力作为核心评估指标，而不仅仅是价格和功能。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwO2lEBiboUdpznkiboyiaGcZxibdVG8I4jFPv61XKZRvlzzxR0WuicmPiabwvtxYyibywXW65wBdaNibt8fDaP47UwyJmibdy0EaHkJK9ibU/640?wx_fmt=gif&from=appmsg "")  
  
**本周安全建议**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/9Nf1wzzcfwOQT661mLR2J7f5TEoDZ8K8kRI5PJE2aQia2w5FxtUr9so9Sic8eH4icwGPea0RO9SEdqkzoJ6ibkrWSe34ISWkXIJwJhj8aica3SXo/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**1. 紧急补丁：**  
CISA KEV本周新增9个漏洞，优先处理互联网暴露设备——Cisco Secure Email Gateway、Palo Alto PAN-OS、BeyondTrust Remote Support、Fortinet、AhsayCBS，按厂商SOP本周内完成升级。  
  
  
**2. OAuth治理：**  
在Azure AD中审查所有第三方OAuth应用授权，撤销可疑应用；对高风险用户启用条件访问，限制陌生设备和位置登录；培训员工"点击同意"也是一种授权行为。  
  
  
**3. RMM管控：**  
对所有远程监控管理软件（MSP360、ScreenConnect、BeyondTrust、TeamViewer等）实施安装白名单，监控异常出站连接，避免合法签名软件被武器化。  
  
  
**4. 云备份重检：**  
针对JadePuffer类"删除云资源"攻击，启用关键存储账户软删除和版本化，备份与生产订阅物理隔离，对高权限云身份启用PIM/JIT临时提权。  
  
  
**5. 医疗与地方政府：**  
本周医疗行业一周四起事件、美国地方政府2.6TB泄露，提示医疗和政务系统应专项检查远程访问日志、MFA覆盖率和离线备份。  
  
  
**6. AI安全投入：**  
Proofpoint收购Varonis、Pluto 4200万美元出隐身、Harmonic 5000万美元B轮，提示企业在AI代理规模化部署前，同步规划影子AI治理和AI代理运行时安全能力。  
  
  
**免责声明**  
  
本周报内容由安全419编辑部基于公开资讯整理汇总，旨在为网络安全从业者提供参考信息，不代表安全419的立场和观点，我们已尽力确保信息的准确性和完整性，但不对信息的及时性、准确性、完整性做出保证，同时也不构成任何安全建议、法律意见或投资推荐。依据以上内容做出任何决策前，都应独立进行进一步核实和研究，对于因使用以上内容而导致的任何损失或损害，安全419概不负责。  
  
  
  
END  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/9Nf1wzzcfwNF15ibIbAdSRdZWcfMWoExdN7k4PNPia8qTVwnjU8gEHLh7KicbN3LezLB43B2iaxcLwwSJFaPnCUht9znnkJZnUmaGcyLZ6qm6Ho/640?wx_fmt=gif&from=appmsg "")  
  
  
✦  
  
**推荐阅读**  
  
✦  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzUyMDQ4OTkyMg==&mid=2247555850&idx=1&sn=89fae134bc13b7c6fb0af8060dc19300&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzUyMDQ4OTkyMg==&mid=2247555803&idx=1&sn=a9121ddf1b66f0321117bd09f2be6416&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzUyMDQ4OTkyMg==&mid=2247555780&idx=1&sn=ec05d366630e8a3e29ea321ead5861f8&scene=21#wechat_redirect)  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/9Nf1wzzcfwNpMOIiak0dVxwD2xOYyz90ptzRRaroibHkicQlzujEWBrASZMz9AS2pP7Az7e0qJNQTBKfUxKAS8HIkias2baM2DyAibLTPP4mIqZ4/640?wx_fmt=jpeg&from=appmsg "")  
  
**粉丝福利群开放啦**  
  
加安全419好友进群  
  
红包/书籍/礼品等不定期派送  
  
