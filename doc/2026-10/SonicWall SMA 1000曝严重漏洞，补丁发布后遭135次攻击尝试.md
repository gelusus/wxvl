#  SonicWall SMA 1000曝严重漏洞，补丁发布后遭135次攻击尝试  
 FreeBuf   2026-10-13 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX0qzuTh5mhuzdZecFlLlz0nmag7oE56GWwfrqQzWu5gCTib98nrEDOmWgkM5Xpj3GH6MqWxsTuW6pRqRzXLoY5DMqjWyn0aJO2o/640?wx_fmt=gif "")  
  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX10XdPJictGJJNGkhS1iafHBiaMnoKTMiaVeypCjQjKJhic69nstbuttmEZxAfZsskaD4tq7VmJogiaFwSoe1FeVxrDphEfiaowD4hwpE/640?wx_fmt=png "")  
  
  
Part  
01  
  
补丁发布后现135次利用尝试  
  
  
据观测，SonicWall Secure Mobile Access（SMA）1000设备近期披露的一处严重安全漏洞，在公开后数日内就遭到威胁行为者利用。  
  
  
该漏洞编号为CVE-2026-102255，CVSS评分10.0，属于预认证服务器端请求伪造（SSRF）漏洞。远程攻击者可借此操控设备代其发送请求，访问设备内部功能接口。攻击者无需经过授权，即可执行各类敏感操作。  
  
  
2026年10月6日，SonicWall为该漏洞发布补丁，对应修复版本为12.4.3-03670（平台热修复补丁）和12.5.0-03082（平台热修复补丁）。  
  
  
补丁发布后，该漏洞很快出现活跃利用尝试。威胁情报厂商Defused Cyber和Previdian于2026年10月8日首次发现攻击迹象。  
  
  
Previdian的数据显示，截至目前共观测到135次攻击尝试，来自2个位于美国的独立IP地址。SonicWall尚未更新安全公告，说明该漏洞已遭在野利用。  
  
  
Part  
02  
  
SMA 1000此前已有0Day被利用  
  
  
CVE-2026-102255是SonicWall SMA 1000设备近期最新一个遭在野利用的漏洞。2026年6月，同产品的另外两处漏洞（CVE-2026-15409，CVSS评分10.0；CVE-2026-15410，CVSS评分7.2）就曾被名为UTA0533的威胁行为者作为0Day漏洞利用，部署Java Webshell和名为KNUCKLEBALL的恶意软件。  
  
  
Rapid7也观测到，威胁行为者正利用这些漏洞攻击面向互联网的设备，将其作为隐蔽的初始访问入口。攻击者可在设备操作系统上执行命令，建立长期持久化访问。  
  
  
该公司指出：“攻击者在设备上建立立足点后，会系统性窃取高价值凭证、活跃会话数据库，以及基于时间的一次性密码（TOTP）多因素认证（MFA）种子配置。凭借这些窃取到的资源，威胁行为者会快速开展横向移动，从已失陷的设备直接切入企业内部网络。”  
  
  
参考来源：  
  
Critical SonicWall SMA 1000 Flaw Targeted in 135 Exploitation Attempts After Patch  
  
https://thehackernews.com/2026/10/critical-sonicwall-sma-1000-flaw.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
