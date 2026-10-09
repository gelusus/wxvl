#  Atlassian高危漏洞遭在野利用，细节公开数小时即现攻击  
 FreeBuf   2026-10-09 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1DwcwvHXl0ZZgpBsiaTHvhzkiaehVBSoIMicj5nI3Ctpiaice9LT1Qho8ywERyqN1HUt4K15yibDcHBEt9ibHOz1fWjoce0E0q2OxodA/640?wx_fmt=gif "")  
  
  
![Atlassian logo](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icBE3OpK1IX3xp7FsnRibia0vruY0Tnsd9C5CdgzDS26G0ANQv4A4AELMNqwHyOcibEHoBjsdNHJleJAicznZb0ltmKZNjVX8iaLoiaYgib619QA22E/640?wx_fmt=jpeg "")  
  
  
Part  
01  
  
漏洞影响Atlassian数据中心产品  
  
  
威胁行为者已开始利用CVE-2026-21589（CVSS评分9.3），这是Atlassian数据中心产品中的严重任意文件访问漏洞。在特定条件下，攻击者可利用该漏洞访问敏感文件。受影响产品包括Bitbucket、Confluence、Jira Service Management、Jira Software、Bamboo、Crowd、Crucible和Fisheye。  
  
  
Atlassian在安全公告中表示：“该任意文件访问漏洞可导致未授权攻击者访问受影响版本Web应用根目录下的特定文件。攻击者需要提前知晓目标文件的准确名称与路径才能完成利用，无法通过该漏洞枚举或列出目录内容。在部分配置场景下，攻击者可访问到敏感文件，因此漏洞严重程度极高。”  
  
  
Previdian的遥测数据已检测到15次利用尝试，攻击来源为日本和美国的3个IP地址。  
  
  
Part  
02  
  
漏洞为路径遍历缺陷  
  
双冒号可被识别为路径分隔符  
  
  
10月6日，watchTowr Labs对比存在漏洞和已修复的Atlassian安装包后，发布了CVE-2026-21589的技术分析报告。  
  
  
研究人员发现这是一处路径遍历漏洞，程序会将双冒号（::）序列识别为路径分隔符，攻击者可借此读取应用Web根目录下的文件。  
  
  
watchTowr Labs在分析报告中写道：“Java应用的路径遍历漏洞中，分号（;）、点（.）和斜杠（/）是非常常见的利用字符，但这次我们发现了一个特殊点：双冒号（::）也能被匹配识别。这类语法在同类漏洞中并不常见，但我们排查代码库后，发现了十分熟悉的利用路径。”  
  
  
watchTowr表示，漏洞出在Atlassian的Web资源处理逻辑中。程序会将..::..::..::..::WEB-INF::web.xml这类字符串转换为../../../../WEB-INF/web.xml，攻击者可借此穿越目录，访问本无权读取的文件。  
  
  
在一套集成Jira的Crowd部署环境中，研究人员成功读取到crowd.properties文件，获取到应用凭证。随后研究人员利用这些凭证创建用户，并将其加入jira-administrators管理员组。用户可通过配置Crowd IP白名单阻断这条攻击路径。  
  
  
Part  
03  
  
漏洞公开两小时即遭利用  
  
  
据watchTowr披露，其发布漏洞技术分析约两小时后，旗下蜜罐网络就检测到了相关利用尝试。  
  
  
Previdian也证实该漏洞已出现活跃利用。Previdian创始人Ryan Dewhurst在LinkedIn上写道：“目前我们的蜜罐网络已经开始捕获到相关利用活动。”  
  
  
该漏洞允许未授权攻击者发送单个请求就读取Web根目录下的敏感文件，可能导致令牌、凭证、密钥及其他认证数据泄露。据称，watchTowr发布更多漏洞技术细节两小时后，针对Previdian蜜罐网络的利用活动就已出现。  
  
  
Atlassian表示，所有版本号低于以下修复版本的产品均受影响：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0mhhzI9RfdM0kDdjKhXH1v51B2dtICZyqSF3eIialbRQM0l0TIDDRcfxsWKW5UIIzoZ8rOmnhcKMSabwyav52pNgAyDmibwDtM0/640?wx_fmt=png&from=appmsg "")  
  
  
作为临时缓解措施，Atlassian建议用户将受影响实例从公网下线。用户也可以配置Web应用防火墙（WAF）规则拦截恶意请求。  
  
  
针对Confluence、Jira Service Management、Jira Software、Bamboo和Crowd，用户可通过Tomcat的RewriteValve配置拦截相关请求。Bitbucket用户可在urlrewrite.xml文件中新增规则进行防护。  
  
  
watchTowr已在GitHub上发布免费检测工具，帮助用户排查服务器是否存在该漏洞。  
  
  
参考来源：  
  
Atlassian Vulnerability Comes Under Attack Hours After Details Go Public  
  
https://securityaffairs.com/200591/security/atlassian-vulnerability-comes-under-attack-hours-after-details-go-public.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
