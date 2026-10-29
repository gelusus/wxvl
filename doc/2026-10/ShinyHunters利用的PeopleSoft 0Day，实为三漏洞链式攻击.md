#  ShinyHunters利用的PeopleSoft 0Day，实为三漏洞链式攻击  
 FreeBuf   2026-10-29 11:03  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX165xR5nXXBYZNKcOtIKicpquEqX1HKyNQRrKZIydFwwicibseN6hbXtdYoPfzBHMTpu1r2LdXhslia1HoQibm6FxXibVF9sjDRCIPyo/640?wx_fmt=gif "")  
  
  
![Oracle PeopleSoft 0Day](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX2pBCdym2YsG2NVSVAp1QtbLlP1iamIoucjRX7PBsdGlsicB6nEZcLvLxHM1anZ98rsfBB2n8ctPvS1goqJWdN5RCrNaOoUKVZF0/640?wx_fmt=png "")  
  
  
Part  
01  
  
0Day补丁前已被利用  
  
  
本周，披露该漏洞的研究人员表示，ShinyHunters自5月起开始利用的Oracle PeopleSoft高危漏洞并非单个漏洞，而是由三个漏洞链式组合而成。  
  
  
今年6月10日，Oracle通过带外安全公告，将三个漏洞统一打包在CVE-2026-35273编号下发布修复补丁。该漏洞影响PeopleSoft Enterprise PeopleTools 8.61和8.62版本，CVSS评分为9.8，未授权攻击者可通过网络直接接管服务器，目前已被纳入CISA的已知被利用漏洞目录。漏洞根源是一处服务器端请求伪造（SSRF）缺陷：由于PeopleSoft默认信任本机环回地址，攻击者可通过该SSRF访问内部管理接口，无需登录即可实现远程代码执行。  
  
  
上述漏洞的技术拆解来自零日计划（Zero Day Initiative, ZDI）的Bobby Gould，其所在团队负责向Oracle上报该漏洞。ZDI在分析报告中指出，团队共上报了位于PeopleSoft两个组件中的三个独立漏洞，目前正与Oracle协调，为每个漏洞分配独立CVE编号。截至本周，6月发布的这一CVE编号仍是相关漏洞的唯一公开标识。  
  
  
在补丁发布前，攻击者就已经开始在野利用该漏洞。Mandiant与谷歌威胁情报组（GTIG）将早期攻击活动归因于其追踪编号为UNC6240的威胁组织——也就是更广为人知的ShinyHunters。该组织在5月27日至6月9日期间利用该漏洞发起攻击，主要目标为高校，由于攻击发生在补丁发布之前，整个利用阶段该漏洞都属于0Day漏洞。在9月的后续追踪中，Mandiant发现攻击者发起了规模更大的新一轮攻击，目标范围从教育行业扩展到科技、IT服务、医疗、农业、交通、政府领域，全球范围内数十套系统受到影响。  
  
  
Part  
02  
  
SSRF突破信任边界  
  
  
ZDI的分析明确了这一补丁修复的具体漏洞逻辑，整个漏洞链涉及PeopleSoft的两个组件，二者均作为独立Web应用运行。其中集成网关（PSIGW）负责与外部系统交换消息，环境管理中心（PSEMHUB）则是内部管理接口，设计上仅响应可信主机的请求。  
  
  
漏洞链的第一环是PSIGW中存在的SSRF缺陷。攻击者向PSIGW的监听连接器发送请求，可同时控制请求转发的目标地址和发送的完整payload，处理程序不会校验目标地址是否合法。PSEMHUB通过allowedhosts.properties文件中的列表校验请求来源是否可信，该文件默认将本机环回地址127.0.0.1加入可信列表。攻击者将SSRF的目标指向127.0.0.1时，请求看起来来自服务器本机，可直接通过信任校验，从外部访问原本被隔离的内部接口。  
  
  
![PeopleSoft漏洞链](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0YuBrjQnauWOyzfiaVicXUOsu30bFmUDiaMoSjA8qm5KJTHTD9GgjuNREEia17MOJVXX2Svuj3zWIaJrD05sC32kv9mKMqzwZsLms/640?wx_fmt=png "")  
  
  
通过上述方式获取内部接口访问权限后，攻击者可通过两种路径实现代码执行。第一种路径滥用PSEMHUB的日志写入功能：攻击者可控制写入的文件内容和磁盘存储路径，将Webshell写入应用目录，以PeopleSoft服务账户权限执行命令。第二种路径是写入恶意XML文件，待服务下次启动时，PSEMHUB会通过Java的XMLDecoder解析该文件，直接执行攻击者代码，无需植入Webshell，这种方式更难被常规检测机制发现。  
  
  
ZDI指出，这一漏洞链的核心设计问题在于，默认允许环回地址访问的机制，导致SSRF漏洞出现时信任校验机制完全失效，这一点足以推翻部分防御者"防火墙规则已覆盖风险"的判断。在9月的攻击浪潮中，Mandiant发现ShinyHunters通过对单个字符进行URL编码，将请求路径改为/%50SEMHUB/，绕过了拦截/PSEMHUB/路径的Web应用防火墙（WAF）规则。多数WAF规则仅匹配明文路径，而PeopleSoft服务器会自动对URL进行解码并正常返回页面内容。  
  
  
Part  
03  
  
安装补丁为首要方案  
  
  
以下修复方案和防护指引来自Oracle和Mandiant，ZDI的分析仅聚焦漏洞技术原理，不涉及响应处置建议。  
  
  
及时安装补丁。部署Oracle针对CVE-2026-35273发布的安全公告修复补丁，使用处于支持周期内的PeopleTools版本。Mandiant明确指出，基于路径的WAF拦截规则不能替代补丁修复。  
  
  
若暂时无法安装补丁，可按照Oracle公告的建议，在多服务器部署环境中禁用环境管理中心服务，或在单服务器部署环境中移除PSEMHUB应用。限制这些管理端点访问不会影响正常用户会话。  
  
  
若疑似已遭入侵，需将存在Webshell的主机视为已失陷主机，妥善留存证据，并轮换PeopleSoft层级可访问的所有凭证。ShinyHunters通常会先窃取数据再索要赎金，受影响机构需做好应对勒索的准备。  
  
  
需要注意的是，目前官方并未明确安装补丁是否会清除攻击者已获取的访问权限。Oracle的安全公告未提及这一问题，Mandiant提出的轮换凭证、排查Webshell等指引，默认补丁无法清除已有的入侵痕迹。补丁仅能封堵入侵入口，无法自动清除已经进入系统的攻击者。  
  
  
参考来源：  
  
ShinyHunters-Exploited Oracle PeopleSoft Zero-Day Turns Out to Be Three Chained Flaws  
  
https://cybersecuritynews.com/shinyhunters-oracle-peoplesoft-zero-day/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
