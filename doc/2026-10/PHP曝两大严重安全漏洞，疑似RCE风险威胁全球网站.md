#  PHP曝两大严重安全漏洞，疑似RCE风险威胁全球网站  
 FreeBuf   2026-10-21 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3tEDTXlFRo7CSmib8ZISZj9JxDUKltlYicDcP6NFu8oT0GTHJdXiaGupK8yCp1guXr05KcicGzHt8o4ua2f2ZicDLneC4feiaaHglVU/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1fmGwp8I0A273AriaEYz4RVCRWvicvCTEhmva0jwNASKQ3U9pE23yvBTM4sS2L47icjfCLKqm1rVefVcZqjNSpzSnZmqQWmtRxd8/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
PHP两大高危漏洞  
  
  
PHP安全团队已向用户预警两个严重安全漏洞，敦促管理员为即将发布的安全更新做好准备。两个漏洞的CVSS v4.0评分均为9.2。更新计划于2026年10月21日周三UTC时间14:00正式发布，留给网站所有者和托管服务商的准备窗口十分有限。  
  
  
PHP在10月20日发布的官方安全公告中表示，补丁发布同步会公开完整技术细节、受影响版本范围以及修复版本。如果任一漏洞细节被提前公开，或出现漏洞被主动利用的证据，团队可能提前推送更新。  
  
  
公开报道显示，本次即将修复的两个漏洞对应编号为CVE-2026-109618和CVE-2026-109619。安全厂商watchTowr指出，二者均为已上报的远程代码执行漏洞，攻击者可利用漏洞在受影响服务器上运行任意代码。不过PHP发布的提前预警并未提及具体CVE编号，也未说明漏洞的具体原理。  
  
  
CERT-EU发布的公告也提及这两个漏洞编号，预警称漏洞可能允许攻击者无需认证即可执行远程代码。该机构的评估部分参考了泄露信息，因此管理员需要将这类非官方报道与PHP直接确认的有限事实明确区分开。  
  
  
有报道称漏洞影响范围覆盖PHP 5及以上所有版本，但PHP官方尚未证实这一宽泛说法。目前PHP尚未确认默认安装配置是否受影响、触发漏洞需要哪些配置条件，也未确认特定操作系统是否面临更高风险。  
  
  
部分早期网传帖子称这两个漏洞的最高严重等级评分为10.0，但PHP官方公布的两个漏洞CVSS v4.0评分均为9.2。在更多细节披露、明确底层风险和攻击条件之前，所有相关报道应以官方评分为准。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0VibMaD4Fuq69NicmnDU5sz2BkENZz1M7GrlIndf9Z2p7bic8ic6vd7ibo4cIE0leabyoEIiazJdAczsUtDUnzoXaGqf1prRic6H594g/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
漏洞利用情况与风险  
  
  
截至watchTowr发布相关更新时，尚未发现这些漏洞存在公开在野利用。PHP提及可能提前发布更新只是预防性措施，并不代表已经出现实际攻击。网上流传的泄露漏洞演示、利用代码等内容，不能作为真实目标已遭攻击的证据。  
  
  
过往案例表明管理员需尽快做好准备。据Cybersecurity News记录，此前另一项独立PHP漏洞CVE-2024-4577披露后24小时内，研究人员就检测到了主动利用尝试。这类历史事件可以解释本次补丁部署的紧迫性，但不能证明本次新披露的漏洞已遭到利用。  
  
  
2026年初，该媒体也曾报道过PHP SOAP扩展相关漏洞。这类漏洞涉及不同组件，对应独立的修复补丁。管理员切勿认为安装此前PHP漏洞的补丁，就能修复本次即将推送补丁对应的两个漏洞。  
  
  
Part  
03  
  
CERT-EU建议立即梳理PHP资产  
  
  
CERT-EU建议，各机构立即梳理自身资产中的PHP应用，做好准备在修复版本发布后第一时间为受影响系统打补丁。如果无法立刻完成补丁部署，应考虑将面向互联网的应用从公网隔离，直到补丁安装完成。采取隔离措施前需充分评估对服务可用性和业务需求的影响。  
  
  
各团队要利用这段准备窗口，确认每一处PHP安装的运维负责人，明确生产系统的更新落地流程。补丁正式发布后，管理员需先核对官方公布的受影响版本列表，及时部署对应修复补丁，完成后验证当前运行的PHP版本是否正确。在官方补丁发布前，不能仅凭自身配置与网上流传的受影响条件不符，就断定业务部署安全无风险。  
  
  
参考来源：  
  
PHP Warns of Two Critical 9.2 Security Vulnerabilities, Urges Users to Prepare for Patches  
  
https://cybersecuritynews.com/php-warns-of-two-critical-9-2-security-vulnerabilities/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
