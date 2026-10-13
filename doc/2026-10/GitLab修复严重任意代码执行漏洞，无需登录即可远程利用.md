#  GitLab修复严重任意代码执行漏洞，无需登录即可远程利用  
 FreeBuf   2026-10-13 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1icOtoA6cibmFPv6uQp8PNAZmkudQ9VsUAbGWayZVRicicGwYicibHbmoLRLXeLVBlNQUiboXjfWq5krOhYMvkuXaDd98KRvyic60UyQs/640?wx_fmt=gif "")  
  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0r1he6h7muSIQyI7d3NRPaFnl9bqAuXQlXkOcfmKpw1icsSIJZPoWzZ5EJiamjibDvlwkMh9ziaQViblgy4QZZdicaqoXfLU83cTHH4/640?wx_fmt=png "")  
  
  
Part  
01  
  
未授权远程执行  
  
  
GitLab已发布紧急安全更新，修复一处严重漏洞。攻击者无需登录，就可以在受影响服务器上执行任意代码。  
  
  
该漏洞编号为CVE-2026-97229，CVSS评分达最高值10.0，同时影响GitLab社区版（CE）与企业版（EE）。GitLab已于2026年10月12日发布相关补丁。  
  
  
漏洞根源是GitLab CI/CD正则表达式编译器存在整数溢出问题。由于正则表达式校验逻辑存在缺陷，未授权攻击者可在CI/CD配置中植入恶意构造的表达式，触发代码执行。正则表达式是用于匹配或过滤数据的文本模式，服务器一旦处理这类恶意构造的正则模式，自身就会遭到入侵。  
  
  
根据GitLab发布的安全公告，该漏洞的利用需要满足特定条件。目前公开的公告内容未披露完整攻击链路，也未明确说明利用所需的具体前提。  
  
  
但从CVSS评分向量来看，该漏洞可通过网络远程利用，攻击复杂度低。利用过程无需任何权限，也不需要用户交互，对系统机密性、完整性、可用性的影响评级均为高危。  
  
  
该漏洞属于服务端代码执行，不同于仅在访问者浏览器中运行的恶意JavaScript。对于在受影响GitLab实例上托管开发项目的企业来说，这类漏洞的危害尤为严重。  
  
  
攻击者成功利用漏洞后，会威胁服务器的数据安全与正常运行。目前公告未提及该漏洞已出现真实在野利用案例。  
  
  
Part  
02  
  
漏洞发现与影响  
  
  
GitLab公布的致谢信息显示，该漏洞由安全研究员joaxcar通过HackerOne漏洞赏金计划上报。今年9月，GitLab刚修复过GitLab CI/CD模块的多处严重正则表达式漏洞。  
  
  
此前修复的漏洞需要认证后才能利用，本次新披露的漏洞则无需账号就可以发起攻击。公告未说明两类漏洞是否存在相同的根因。  
  
  
受影响版本包括GitLab CE和EE：19.2系列早于19.2.8的版本、19.3系列早于19.3.4的版本，以及19.4系列早于19.4.2的版本。使用自托管实例的管理员需立即将实例升级到对应修复版本。GitLab.com已运行修复后的版本，GitLab Dedicated客户无需采取任何操作。  
  
  
Part  
03  
  
补丁更新与注意事项  
  
  
本次更新还修复了其他安全问题，包括跨站脚本漏洞、授权失效问题，以及GitLab Duo Agent Platform中一处与提示词相关的问题。此前GitLab发布的CI/CD授权漏洞安全更新早已表明，管理员应当完整查阅补丁公告，而非仅关注最受关注的头号漏洞。  
  
  
GitLab提醒，19.3.4版本存在一个已知Bug：启用Ona集成时，合并请求页面会出现异常。该Bug计划在19.3.5版本中修复。  
  
  
管理员在升级过程中需要留意该已知问题，但不能因此忽略本次紧急安全补丁的部署。GitLab在公告中附带的实例安全指南提供了更全面的防护建议。  
  
  
参考来源：  
  
Critical GitLab Vulnerability Enables Arbitrary Code Execution Attacks on Servers  
  
https://cybersecuritynews.com/gitlab-vulnerability-code-execution/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
