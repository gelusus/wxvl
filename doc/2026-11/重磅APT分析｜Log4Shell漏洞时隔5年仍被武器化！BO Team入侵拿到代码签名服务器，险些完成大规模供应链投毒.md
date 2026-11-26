#  重磅APT分析｜Log4Shell漏洞时隔5年仍被武器化！BO Team入侵拿到代码签名服务器，险些完成大规模供应链投毒  
原创 AI紫队安全研究
                        AI紫队安全研究  AI紫队安全研究   2026-11-26 03:58  
  
**大家好，我是AI紫队安全研究。建议大家把公众号“AI紫队安全研究”设为星标，否则可能就无法及时看到啦！因为公众号只对常读和星标的公众号才能大图推送。操作方法：先点击上面的“AI紫队安全研究”，然后点击右上角的【...】,然后点击【设为星标】即可。**  
  
**关注视频号 “AI紫队安全研究” 不定期周五晚上10点直播。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/E3ZvvAXyiaibmYfEa8AGAMVQmqn4yqg9ciawyAoyq1SicPlINXXkGlP6jwh3nNsCsQWonntA8PazoYKgLd8s7Q2YVZlyZKr8ReXQcKicBzJGs7uo/640?wx_fmt=png&from=appmsg "")  
  
导语  
  
卡巴斯基ICS‑CERT披露BO Team（别名Black Owl黑猫头鹰 / Hoody Hyena）的完整入侵事件。攻击者利用早已公开的Log4Shell（CVE‑2021‑44228）漏洞，入侵VMware vCenter虚拟化管理平台，在受害者内网潜伏长达数月。  
  
  
攻击者一路横向渗透，拿下Hypervisor、GitLab代码仓库、Ansible自动化运维平台，最终触达软件代码签名服务器。一旦得手，黑客就可以使用企业合法数字证书给恶意程序签名，软件下游全部客户都会被波及，酿成大规模供应链灾难。好在企业及时发现处置，攻击者最终没能完成最终投毒环节。  
  
警示：很多企业以为老漏洞威胁已经过去，但是国家级威胁组织依旧持续利用未修补历史高危漏洞打开入侵大门。  
  
  
一、团伙背景：BO Team（Black Owl）  
  
BO Team自至少2023年开始活跃，公开研判为乌克兰关联威胁组织。该团伙擅长间谍窃密与黑客行动主义，武器库包含Cobalt Strike、Mythic以及自研BrockenDoor、ZeronetKit后门。  
  
该团伙有两大作战特征：  
  
1. 优先大量使用LotL就地取材原生系统工具，尽量少投放自定义恶意样本，降低被安全产品检测概率；  
  
2. 目标重点瞄准对手软件、军工相关企业，意图破坏其业务能力；同时会运营Telegram频道对外公布攻击成果。  
  
本次入侵事件中攻击者没有执行数据擦除、没有公开宣称战果，核心目标是长期潜伏，伺机发动供应链攻击。  
  
  
二、完整攻击时间线与攻击链路  
  
事件发现时间：2025年6月，受害软件企业向卡巴斯基求助，追溯发现入侵已经持续数月之久。  
  
  
1. 初始立足点：暴露的SSH终端服务器  
  
攻击者首先扫描到一台公网暴露SSH服务的Linux终端服务器完成初始接入，对内网开展大规模探测扫描，枚举SMB、Kerberos、WinRM、LDAP、SSH、Telnet等各类内网服务。  
  
  
2. 突破虚拟化核心：Log4Shell攻击VMware vCenter 7.0.3  
  
攻击者手动尝试登录vCenter网页界面失败，转而直接利用Log4Shell漏洞拿到vCenter服务器root最高权限。通过SSH登入底层Hypervisor虚拟化宿主机，拿下虚拟化集群控制权。  
  
  
3. 特殊休眠式后门部署（pgcache）  
  
在宿主机`/etc/cron.hourly/`写入名为`pgcache`的每小时执行脚本，伪装成PostgreSQL数据库组件。  
  
后门访问免费动态DNS服务`nofrillsdns.com`；  
  
DNS记录为`127.0.0.1本地回环地址时，后门保持休眠，完全不执行恶意动作；  
  
黑客只需要远程修改DNS解析记录，后门立刻被激活，下载GSocket远控工具建立C2通道。  
  
这种休眠设计极大增加日常检测难度，平时没有任何外联异常行为。  
  
  
4. 内网横向扩散，逐步打通全业务域  
  
利用被攻陷Hypervisor的证书凭证，横向接管第二套虚拟化集群，部署同款休眠后门；  
  
读取虚拟机内部密钥，入侵GitLab代码仓库，窃取源代码与私有密钥；  
  
利用GitLab密钥接管Ansible自动化运维平台；  
  
在虚拟主机内存抓取凭证，拿到域管理员账号，攻陷域控服务器，导出ntds.dit域数据库，dump全部域账号哈希；  
  
篡改`/usr/bin/nxnode‑login`程序，拦截登录账号密码，写入共享内存文件窃取凭据；  
  
大量使用系统自带工具：tasklist、findstr、comsvcs.dll、regedit，甚至绕过检测不使用Mimikatz，规避安全设备特征库；  
  
使用时间戳篡改timestomp、删除事件日志等手段抹除入侵痕迹。  
  
  
5. 抵达最高价值目标：代码签名服务器  
  
攻击者成功访问企业用于商业软件编译签名的核心服务器。  
  
两种可怕后果：  
  
① 在软件编译打包流程注入恶意代码，使用企业合法证书签名，分发到下游客户；  
  
② 直接盗取代码签名证书，后续对外给任意恶意程序盖章，让木马获得系统信任。  
  
  
6. 入侵被阻断之后，黑客发起二次钓鱼反扑  
  
企业应急处置切断攻击者全部访问权限。BO Team并未放弃，发起钓鱼邮件反扑：  
  
邮件主题“账户已与邮件服务器断开连接”，邮件绕过SPF、DKIM校验（攻击者已经拿到受害企业邮件服务器私钥），套取员工账号密码。该轮钓鱼被安全防护全部拦截。  
  
  
三、本次攻击暴露出3个高危认知误区  
  
误区1：老漏洞不用紧张，Log4Shell这类好几年前的漏洞没有威胁  
  
现实：只要还有设备没打补丁，国家级APT依旧会把老漏洞当做核心突破口，本次攻击就是真实案例。vCenter是企业虚拟化心脏，一旦沦陷，成百上千台虚拟机全部暴露风险。  
  
  
误区2：只有陌生exe、恶意二进制才需要警惕  
  
BO Team尽量不释放独有恶意样本，优先全盘复用系统自带命令行工具、脚本，大量LotL行为。传统靠病毒样本哈希查杀的防御思路很难捕捉。  
  
  
误区3：代码签名服务器、CI/CD开发环境只需要内网隔离就万事大吉  
  
攻击者可以从虚拟化层向下突破虚拟机，虚拟化管理平台失守，再严格的内网隔离都可能被绕开。虚拟化平台本身就是最高风险面。  
  
  
四、运维&安全团队威胁狩猎和加固清单  
  
🔍威胁狩猎重点告警  
  
1. Linux侧：检查`/etc/cron.hourly/`目录下异常脚本`pgcache`；监控访问`nofrillsdns.com`域名的主机；  
  
2. VMware vCenter与Hypervisor：监控vCenter出现Log4Shell漏洞利用行为；监控hypervisor异常每小时定时任务、未知shell脚本；  
  
3. 行为告警：  
  
    系统使用`comsvcs.dll`导出lsass内存；  
  
    无业务背景导出ntds.dit域数据库；  
  
    修改系统二进制文件（nxnode‑login等登录组件）拦截凭据；  
  
    大量删除Windows事件日志、文件时间戳篡改timestomp行为；  
  
4. 虚拟化层审计：Hypervisor直接向内网虚拟机发起SSH、命令行操作行为。  
  
  
🛡️安全加固建议  
  
1. 高危组件补丁闭环  
  
vCenter、Log4j组件必须及时更新补丁；定期扫描虚拟化平台，不允许生产环境遗留Log4Shell这类历史高危漏洞。vCenter禁止直接暴露公网，使用VPN+IP白名单访问。  
  
  
2. 虚拟化平台本身要做最高等级防护  
  
很多企业把vCenter、Hypervisor当成网络设备，忽视主机安全基线。虚拟化管理服务器是整个内网制高点，部署EDR、开启完整日志审计，禁止普通业务人员访问hypervisor。  
  
  
3. 开发CI/CD、代码签名服务器专项防护  
  
代码签名服务器尽可能物理隔离，最小化访问来源；  
  
签名证书密钥做硬件HSM硬件密钥存储，避免服务器被入侵直接盗取证书私钥；  
  
监控编译、签名服务器一切外来登录、脚本执行行为。  
  
  
4. 防御LotL就地取材攻击  
  
不要只查杀恶意文件，开启行为检测，监控系统原生工具的反常使用：lsass内存导出、域数据库导出、日志批量清除。  
  
  
5. DNS异常监控  
  
企业内网服务器访问各类免费动态DNS服务（nofrillsdns.com等）视为高风险告警，很多休眠后门依赖动态DNS机制触发。  
  
  
🚨应急提示  
  
如果确认遭遇同类入侵：仅仅删除脚本远远不够。攻击者已经拿到大量域账号、各类密钥。必须轮换全部域管理员、运维账号密码，全面排查虚拟化集群、CI/CD、签名服务器，评估供应链被污染风险。  
  
  
五、安全思考  
  
BO Team这起事件带给行业非常现实的警示：  
  
高危漏洞不会随着时间自动消失，只要网络中还有未修补资产，它就永远是APT的突破口。  
  
  
虚拟化平台作为企业IT底座，一旦失守，上层所有虚拟机、业务系统全部面临被横向渗透的风险。  
  
而代码签名、CI/CD流水线是供应链攻击的皇冠上的宝石。攻击者的目标不一定是偷数据，而是篡改信任链，利用合法证书把恶意软件输送给成千上万下游用户。  
  
对于软件开发商，虚拟化基础设施、开发构建、签名环境，三者同等重要，缺一不可。  
  
  
文末互动  
  
你们环境的vCenter是否做过漏洞基线核查？如何防护LotL就地取材式APT攻击？欢迎评论区交流。  
  
   
  
**加入知识星球，可获取权益**  
  
一、"全球高级持续威胁：网络世界的隐形战争"，总共26章，为你带来体系化认识APT，欢迎感兴趣的朋友入圈交流。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/E3ZvvAXyiaiblj6Qa1c5j4iaSxNtaWyMmOrsJ7WJafnTfxff3PA2nhkdQL7AyqtkzhPaoCicbu2FWhIAe1y02o5icTMZiaiaD1T4WXgf5TRsAVyEFU/640?wx_fmt=png&from=appmsg "")  
![](https://mmbiz.qpic.cn/mmbiz_png/E3ZvvAXyiaibm0l6wIBoUfic1Rxr77k9bUlBJeO2gkADWstEJ1u2JGkNGd6Td2RFTWbUh4PWaibl2jEpIAZnNsBUjCX8D6Xrlmuw44kpnsx1H34/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/E3ZvvAXyiaibnOts86xJKqAicF3fEIc4dnBIEm1bCBvX9PhLYRgIIpzQRnfnkanibo4N4ogOicxz4HEc3rFqIBscWYQdYZpL8Ucu7kbX0aRicnZjk/640?wx_fmt=png&from=appmsg "")  
  
二、为什么加入？    
  
职场瓶颈期找不到突破方向？安全项目落地缺成熟方案？面对APT攻击、勒索病毒不知如何构建防御体系？    
  
  
三、在这里，你能获得的不只是资料包，而是直接对接行业专家的「私人顾问服务」   
  
✅ 职业发展「精准导航」  
  
 1v1简历优化：针对安全岗（渗透测试/安全运营/合规等）拆解JD，突出核心竞争力；    
  
 晋升避坑指南：从工程师到安全负责人，分享晋升路径，避开「技术强但管理弱」的晋升陷阱；  
  
 技能栈规划：根据你的基础（应届生/3年经验/资深专家）定制学习路线，比如从0到1学SOC安全建设、APT威胁狩猎。  
  
✅ 安全方案「对症开方」  
  
 实战方案库：含医疗/制造业/等行业的勒索防御、数据安全合规、供应链安全加固方案（附落地工具清单+成本测算）；  
  
 架构设计咨询：小到EDR选型，大到零信任体系搭建，提供「预算效果」平衡的最优解（已帮10+企业节省40%防护成本）。  
  
✅ 圈子资源「直接对接」  
  
 大厂安全负责人拆解真实案例（如某支付公司攻防对抗的实战复盘）；    
  
  
四、适合谁？    
  
 想突破职业天花板的安全工程师/架构师；    
  
 需快速落地安全项目的企业负责人；    
  
 关注行业动态的安全爱好者或IT从业人员。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/E3ZvvAXyiaibkJsTBMez9zJVBx2GkJZX37f7O4FrIibRh5t4A452yETKicDN4YVqlC8IFp7j3rb1FtERwaHNkNFWq93j1mMPnGemzXIv4NGaUSU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/E3ZvvAXyiaibkfA9XDdSgS2UFvFl6eje0BXEeKlZScMVtCNVBSqD7DzicMw2yPB4iahzUA3H97RvicGicibqricFoEQQey8l2qRVdeUHoYRcRDMbl8Q/640?wx_fmt=png&from=appmsg "")  
  
**喜欢文章的朋友动动发财手点赞、转发、赞赏，你的每一次认可，都是我继续前进的动力。**  
  
  
  
