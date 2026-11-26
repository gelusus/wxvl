#  InfraTrust 提醒注意已遭利用的网络设备供应链漏洞  
Lawrence Abrams
                    Lawrence Abrams  代码卫士   2026-11-26 09:24  
  
   
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
   
聚焦源代码安全，网罗国内外最新资讯！  
   
  
编译：代码卫士  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oBANLWYScMRSylJK2k7H6mNqiaS2G6WRaeeK34cLHE6pe9VeOIHYiboAnKB0TMoayZCxFpHMLljzTnz9DnNuFiaqQ/640?wx_fmt=png "")  
  
  
专栏·供应链安全  
  
  
数字化时代，软件无处不在。软件如同社会中的“虚拟人”，已经成为支撑社会正常运转的最基本元素之一，软件的安全性问题也正在成为当今社会的根本性、基础性问题。  
  
  
随着软件产业的快速发展，软件供应链也越发复杂多元，复杂的软件供应链会引入一系列的安全问题，导致信息系统的整体安全防护难度越来越大。近年来，针对软件供应链的安全攻击事件一直呈快速增长态势，造成的危害也越来越严重。  
  
  
为此，我们推出“供应链安全”栏目。本栏目汇聚供应链安全资讯，分析供应链安全风险，提供缓解建议，为供应链安全保驾护航。  
  
  
注：以往发布的部分供应链安全相关内容，请见文末“推荐阅读”部分。  
  
**Eclypsium****公司发布的****11****月****InfraTrust****报告警告称，上个月关于已遭利用漏洞的大多数基础设施安全公告与产品所绑定的第三方软件缺陷有关，而非与供应商自身开发的代码有关。**  
  
11   
月  
 InfraTrust Pulse   
报告追踪了  
 23   
家基础设施供应商在  
 10   
月  
 15   
日至  
 11   
月  
 19   
日期间发布的  
 373   
份新安全公告，涵盖  
 3880   
个漏洞。在这些公告中，  
98   
份被评为严重级别，  
38   
份包含列入  
 CISA   
已知被利用漏洞（  
KEV  
）目录的漏洞。  
  
然而，在这  
 38   
份公告中，有  
 33   
份修复的是捆绑软件组件中的已被利用漏洞，主要涉及  
 PHP  
、  
Node.js   
和  
 OpenSSH  
。这些漏洞给企业带来了额外风险，因为即使软件开发人员发布了安全修复方案，组织机构也必须等待基础设施供应商将其纳入产品中。报告写道：“对已被利用漏洞的暴露程度取决于每个供应商在上游修复发布后重建产品的速度。”  
  
例如上个月  
 InfraTrust   
报告中重点提到的被统称为  
 “PHP2Shell”   
的两个  
 PHP   
漏洞  
CVE-2026-109618   
和  
 CVE-2026-109619  
，目前正被用于针对  
 Palo Alto Networks   
和  
 Check Point   
防火墙的攻击活动。在这两起案例中，威胁行动者组合利用  
 PHP   
漏洞与其它缺陷，获取受感染设备的  
 root   
访问权限；  
Check Point   
报告称  
VMware ESXi   
主机上被部署了数据擦除恶意软件。  
  
截至  
 11   
月  
 20   
日，  
Fortinet  
、  
Juniper  
、  
Lenovo  
、  
HPE  
、  
Netgear  
、  
HP   
和  
 Siemens   
受影响产品的  
 PHP2Shell   
修复方案要么部分可用，要么尚未发布。  
11   
月  
 19   
日又发布了  
 18   
份  
 PHP   
安全公告，进一步增加了基础设施供应商及其客户的补丁修复难度。  
  
由于管理员需要处理越来越多的基础设施漏洞，  
InfraTrust   
建议本月优先立即修补以下漏洞：  
  
- Cisco Expressway  
：严重漏洞  
CVE-2026-76466  
于  
 11   
月  
 20   
日被纳入  
 CISA   
的  
 KEV   
目录。  
11   
月  
 23   
日，  
CISA   
将该漏洞标记为已被积极利用，此前俄罗斯网络安全公司  
 F6   
发现黑客利用该漏洞获取  
 root   
权限。  
  
- Palo Alto Networks   
和  
 Check Point  
：  
PHP2Shell   
漏洞以及  
 Check Point   
网关中的  
 CVE-2026-123039  
。  
Check Point   
警告称，自  
 10   
月下旬以来，该漏洞一直被积极利用于获取  
 root   
权限并在易受攻击的设备上建立持久化基础。  
  
- Arista VeloCloud Orchestrator  
：  
CVE-2026-114625  
是  
CVSS   
评分为  
 9.9   
的严重漏洞，已在  
 0day   
攻击中被利用。  
  
- Cisco Secure Firewall ASA   
和  
 FTD  
：  
CVE-2026-76418  
影响配置为远程访问  
 VPN   
的设备，已在攻击中被利用，导致设备崩溃。  
  
- SonicWall SonicOS  
：  
CVE-2026-19868  
允许在配置了单点登录的  
 VPN   
上绕过多重身份验证。  
SonicWall   
警告称，已收到该漏洞在  
0day  
攻击中被利用的报告。  
  
  
  
管理员还应优先安装  
 Fortinet   
的  
 10   
月加固更新，它修复了通过  
 AI   
模型发现的众多漏洞。随着  
 AI   
使漏洞的发现和利用变得更加容易，  
InfraTrust   
警告称，组织机构必须采取额外措施在等待供应商发布安全更新的同时保护其设备。这些措施包括从互联网移除管理接口，在内部对其进行分段以限制横向移动，并调查先前暴露的设备是否存在即使安装安全更新后仍可能残留的后门。  
  
该报告认为，“打补丁是基本要求，但在补丁可用之前和漏洞被利用之后所做的，关乎生死存亡。  
”  
  
  
  
  
  
  
  
  
  
  
  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[在线阅读版：《2025中国软件供应链安全分析报告》全文](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247523516&idx=1&sn=0b6fc53ba92e7b5135395b67fff6a822&scene=21#wechat_redirect)  
  
  
[Hugging Face 平台上的热门AI模型可遭劫持](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527770&idx=2&sn=13ee682ea2278091f063ed4818bf00d2&scene=21#wechat_redirect)  
  
  
[Optimizely 供应链攻击波及全球178家组织机构](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527757&idx=1&sn=7c3c4ff77b2e926f70ab7041e36d0622&scene=21#wechat_redirect)  
  
  
[因商业合作伙伴营销平台失陷 全球能源巨头 ENGIE 客户数据被盗](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527548&idx=2&sn=a11df00a8850e51d2b68138ebe61be1c&scene=21#wechat_redirect)  
  
  
[OVHcloud CLI GitHub 发布遭劫持 窃取开发人员机密](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527478&idx=2&sn=06311a4828fcae0248947498dfb5dcf0&scene=21#wechat_redirect)  
  
  
[恶意 GitHub Actions 工作流被植入数万个仓库，窃取凭据](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527332&idx=1&sn=b20cb150f2648c226a53601d04cec2db&scene=21#wechat_redirect)  
  
  
[GitLab 邮件地址可用于供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527227&idx=2&sn=ebe8eb1c6273dd854494ce61cdcf3e57&scene=21#wechat_redirect)  
  
  
[Brevo 供应链攻击将 ClickFix 脚本注入客户网站](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527173&idx=2&sn=925116097a06747f1ebf65484314fe58&scene=21#wechat_redirect)  
  
  
[Anthropic、谷歌和OpenAI 的编程代理严重漏洞可致RCE和供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526840&idx=2&sn=45556dffc35484aa67d144e24667f622&scene=21#wechat_redirect)  
  
  
[GitHub 内部仓库疑遭未授权访问，TeamPCP 据称正在出售 GitHub 内部源代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526060&idx=1&sn=63894334faf0814e075ab85697c75a66&scene=21#wechat_redirect)  
  
  
[TeamPCP再发动供应链攻击；数百个恶意包被上传，RubyGems 暂停新账号注册](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525995&idx=3&sn=e59f7d088b3f4113b18c149ac6e505c3&scene=21#wechat_redirect)  
  
  
[“冬虫夏草”供应链漏洞影响数千家组织机构的代码仓库](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526367&idx=2&sn=873a39cf55d45ffab906c2a6ab004056&scene=21#wechat_redirect)  
  
  
[多家网络安全公司受 Klue 供应链攻击影响](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526263&idx=2&sn=c680ad4854702c179a3070ca44d72c78&scene=21#wechat_redirect)  
  
  
[GitHub 推出 npm 安全变更，对抗供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526263&idx=2&sn=c680ad4854702c179a3070ca44d72c78&scene=21#wechat_redirect)  
  
  
[最新软件供应链事件概览：Red Hat npm 包遭劫持；投毒 Claude Code；OpenAI Codex 认证令牌被盗](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526103&idx=1&sn=5bb0348b6f36ac8d144547cea211d8bd&scene=21#wechat_redirect)  
  
  
[TrapDoor 供应链攻击通过 npm、PyPI 和 CratesIO 传播凭据窃取恶意软件](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526103&idx=1&sn=5bb0348b6f36ac8d144547cea211d8bd&scene=21#wechat_redirect)  
  
  
[自动化供应链攻击6小时内攻陷5561个 GitHub 仓库](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526078&idx=2&sn=23c01cd3ffaa8a2a7421ffb9fe242d2a&scene=21#wechat_redirect)  
  
  
[GitHub 被黑或因员工安装 Nx Console 恶意扩展引发，更多详情待调查](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526078&idx=2&sn=23c01cd3ffaa8a2a7421ffb9fe242d2a&scene=21#wechat_redirect)  
  
  
[GitHub 内部仓库疑遭未授权访问，TeamPCP 据称正在出售 GitHub 内部源代码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526060&idx=1&sn=63894334faf0814e075ab85697c75a66&scene=21#wechat_redirect)  
  
  
[奇安信Qcode Agents重磅升级，正式解锁操作系统级漏洞挖掘能力](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526078&idx=2&sn=23c01cd3ffaa8a2a7421ffb9fe242d2a&scene=21#wechat_redirect)  
  
  
[Grafana 令牌被盗，GitHub 环境可遭访问且代码库被下载](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526043&idx=2&sn=ef8599cf70e02716369d0205be9be468&scene=21#wechat_redirect)  
  
  
[TeamPCP再发动供应链攻击；数百个恶意包被上传，RubyGems 暂停新账号注册](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525995&idx=3&sn=e59f7d088b3f4113b18c149ac6e505c3&scene=21#wechat_redirect)  
  
  
[Checkmarx 再遭攻击，Jenkins AST 插件受陷](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525972&idx=1&sn=b93bcffc7c3ad4c106fbd39a4ee2218e&scene=21#wechat_redirect)  
  
  
[Go 流行库 fsnotify 的维护人员访问权限变更，拉响供应链攻击警报](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525972&idx=2&sn=26ec27a2c831c25b913ce2dfb5658469&scene=21#wechat_redirect)  
  
  
[Gemini CLI 严重漏洞可触发 RCE 攻击和软件供应链风险](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525887&idx=1&sn=294cc8c49080c6239db19c1f8525457e&scene=21#wechat_redirect)  
  
  
[自传播供应链蠕虫劫持 npm 包，窃取开发人员令牌](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525844&idx=2&sn=3f396c2336c086719e62350cd61cd2bb&scene=21#wechat_redirect)  
  
  
[Axios 严重漏洞可导致 RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525768&idx=2&sn=b8967ced3022f4f88a311a652e635650&scene=21#wechat_redirect)  
  
  
[Trivy供应链攻击触发CanisterWorm 在47个 npm 包中自传播](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525520&idx=2&sn=b3d4dddc586c4b0aa8cefb09c0344cb8&scene=21#wechat_redirect)  
  
  
[热门包管理器中存在多个漏洞，JavaScript 生态系统易受供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524984&idx=1&sn=19aef4ce8e288278782458e430a710d8&scene=21#wechat_redirect)  
  
  
[开源自托管平台 Coolify 修复11个严重漏洞，可导致服务器遭完全攻陷](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524828&idx=2&sn=21af241f60f1452013815133745e9a72&scene=21#wechat_redirect)  
  
  
[得不到就毁掉：第二轮Sha1-Hulud供应链攻击已发起，影响2.5万+仓库](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524487&idx=1&sn=f170d3131122071dec6e419c6cff562c&scene=21#wechat_redirect)  
  
  
[vLLM 高危漏洞可导致RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524481&idx=3&sn=6d0b161f8add2f6c1ee65e60ef6955d8&scene=21#wechat_redirect)  
  
  
[开源AI框架 Ray 的0day已用于攻陷服务器和劫持资源](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247519162&idx=1&sn=3872fcc82018e2c561d9e4e7574f0c8e&scene=21#wechat_redirect)  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/infratrust-warns-of-exploited-supply-chain-flaws-in-network-appliances/  
  
  
**本文由奇安信编译，不代表****奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
” 吧~  
  
  
