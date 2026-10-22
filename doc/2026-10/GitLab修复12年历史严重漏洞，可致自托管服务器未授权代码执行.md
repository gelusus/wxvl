#  GitLab修复12年历史严重漏洞，可致自托管服务器未授权代码执行  
 FreeBuf   2026-10-22 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3QiawRecWIAFNyia8fY45ofAriakCFVgzrH974gn71WuItdHu3SficC49sunu6TtIV1gkcRJKCINDZjLe8xb3cGKQHMzb4Qjn8KWM/640?wx_fmt=gif "")  
  
  
![GitLab修复12年历史严重漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX3sZxclVnJib1ic99ibnKae0y0UWXhg8Z7hh5HLNibNpu3hwSSPOJ9bmJHyUjm2KfeRXwhoZnPqPrgVdqjeibG8AJZQQlLb9xsAIEib4/640?wx_fmt=png "")  
  
  
GitLab已发布紧急安全更新，修复一处最高严重等级的漏洞。攻击者无需登录，即可在GitLab自托管服务器上执行任意代码。  
  
  
该漏洞编号为CVE-2026-110264，CVSS评分达满分10.0，同时影响GitLab社区版（CE）和企业版（EE）。官方于2026年10月21日正式推送对应补丁。  
  
  
存在漏洞的代码最早可追溯至2014年发布的GitLab 7.0，至今已存在约12年。但这并不代表攻击者已经利用该漏洞长达12年。GitLab在安全公告中说明了相关风险，但未收到已发生在野攻击的确认报告，也未公开漏洞利用代码。  
  
  
Part  
01  
  
漏洞攻击无需权限与交互  
  
  
根据GitLab官方安全公告，该漏洞成因是整数溢出，触发场景为服务器编译攻击者特制的正则表达式。正则表达式是用于匹配文本的规则字符串。在特定条件下，服务器处理攻击者构造的恶意正则规则时，攻击者即可在服务器上执行任意代码。  
  
  
GitLab的CVSS评估显示，该漏洞可通过网络发起，攻击复杂度低。攻击者无需持有账号权限，也不需要诱导用户交互即可完成攻击。  
  
  
漏洞对数据保密性、完整性和服务可用性的影响均为最高等级。不过GitLab目前尚未披露触发漏洞代码所需的全部前置条件。  
  
  
GitLab确认，该漏洞由安全研究员joaxcar通过HackerOne漏洞赏金计划上报。版本更新说明中提到了Ruby正则表达式相关补丁，进一步指明了受影响的软件层级，但未公开完整攻击路径。管理员不要默认该漏洞仅能通过特定项目或功能触发，除非GitLab官方明确确认存在这类访问限制。  
  
  
Part  
02  
  
漏洞覆盖12年版本分支  
  
  
该漏洞影响从7.0版本开始的所有GitLab CE和EE版本。GitLab已在18.11.13、19.0.10、19.1.9、19.2.9、19.3.6、19.4.3、19.5.1版本中完成修复，上述分支的更早版本仍存在风险。除标准补丁覆盖的分支外，GitLab还为4个旧版本分支额外推送了修复。  
  
  
使用自托管部署的用户需立即升级。GitLab.com已运行修复后的版本，无需用户操作。  
  
  
根据公告，运行19.3.5-internal0、19.4.2-internal0及更高版本的GitLab Dedicated实例已预装修复补丁，租户无需采取任何操作。GitLab同时说明，若公告未明确排除某类部署形态，则所有部署类型均受漏洞影响。  
  
  
注意不要将该漏洞与10月12日修补的另一处GitLab代码执行漏洞混淆。那处独立漏洞编号为CVE-2026-97229，同样涉及正则表达式处理逻辑。该漏洞影响的版本范围与本次漏洞不同，安装此前的补丁无法起到修复作用。  
  
  
GitLab为自托管环境发布了两份检测指南，分别名为"Onigmo原生工作进程崩溃"和"Org模式渲染错误突增"。这些指南可帮助安全团队识别可能的漏洞利用尝试。发布检测指南不代表已发生在野攻击，公告也未将这两类异常信号作为系统已被入侵的判定依据。  
  
  
Part  
03  
  
本次补丁共修复12个安全漏洞  
  
  
本次补丁版本共修复12个安全漏洞。其中另一处高危漏洞编号为CVE-2026-15723，CVSS评分8.2：当同时启用OmniAuth SAML与LDAP自动关联功能时，已认证攻击者无需受害者操作即可接管现有绑定LDAP的账号。  
  
  
编号为CVE-2026-112212的漏洞评分为7.5，攻击者可通过投毒OAuth与OpenID Connect发现响应，将部分认证流程重定向至其控制的服务器。  
  
  
本次更新是GitLab近期一系列安全补丁的最新一环，此前官方还修复过AI网关的远程命令执行漏洞。这些漏洞涉及不同组件，触发攻击的条件也各不相同。准确核查版本比默认"一次更新覆盖所有产品问题"更可靠。  
  
  
管理员在部署更新前需仔细查阅GitLab的官方升级指引。公告提醒，数据库迁移可能影响升级流程：单节点部署升级期间会出现服务中断，符合条件的多节点集群可使用零停机升级流程。  
  
  
19.3.6与19.4.3版本包含常规数据库迁移。GitLab通常会在补丁发布90天后公开漏洞的完整技术细节，团队不要为了等完整分析而推迟升级。  
  
  
本次GitLab补丁推送的同时，Node.js与PHP也发布了安全更新。这进一步增加了运行Web应用与开发服务的团队的补丁运维工作量。据Cyber Security News报道，Node.js本次发布的安全更新共修复26.x、24.x、22.x三个分支的10个漏洞，修复后的版本号分别为26.12.1、24.22.1、22.23.4。PHP则针对PHP2Shell发布紧急补丁，修复两处严重内存处理漏洞，对应编号为CVE-2026-109618、CVE-2026-109619。  
  
  
参考来源：  
  
12-Year-Old GitLab Vulnerability Allows Remote Code Execution on Self-Managed Server  
  
https://cybersecuritynews.com/gitlab-critical-vulnerability-remote-code-execution/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
