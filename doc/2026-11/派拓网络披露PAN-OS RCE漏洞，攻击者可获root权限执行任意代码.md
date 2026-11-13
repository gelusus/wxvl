#  派拓网络披露PAN-OS RCE漏洞，攻击者可获root权限执行任意代码  
 FreeBuf   2026-11-13 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1xcib38aibCIWpHeSdFB0iaqBev0O5qV3F7WulZAiciaMsyVoB4knPom7I9cZPnhjhCD9fbote61h6aGCKoGNqj78B1GPDEENoD5Pk/640?wx_fmt=gif "")  
  
  
![PAN-OS RCE漏洞](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1DBfmH6hIh1whqF8RnRDTIWYHiaico2cJ1nGnLIgPKF8PMFiciafgwHo8IzutKJhtGibvIz4IZwhNFCsnq5VJNvzDAGhwTwKzEept0/640?wx_fmt=png "")  
  
  
派拓网络（Palo Alto Networks）披露，Panorama设备运行的PAN-OS系统存在一处远程代码执行（RCE）漏洞，漏洞编号为CVE-2026-0330。拥有网络访问权限的认证管理员可利用该漏洞，以root权限执行任意代码。  
  
  
2026年11月11日，派拓网络发布安全公告，敦促用户尽快为受影响的Panorama部署安装补丁。Cloud NGFW、Prisma Access以及独立部署的PAN-OS防火墙均不受该漏洞影响。  
  
  
派拓网络表示，目前尚未发现该漏洞遭恶意利用的案例，漏洞由内部研究员Daniel Voce发现并上报。官方安全公告将该漏洞归类为CWE-94，属于代码注入类缺陷。  
  
  
攻击者成功利用漏洞后，可突破普通管理员的权限边界，无需受常规管理权限限制，直接以root权限运行代码。根据已发布的漏洞评估，利用该漏洞需要高权限账号，但无需用户交互。  
  
  
受影响的Panorama设备无需特殊配置即可存在漏洞风险。但利用该漏洞需要同时满足管理员认证、网络访问两个前提条件。该漏洞不支持未授权利用，互联网上的任意攻击者无法直接对可触达的系统发起攻击。  
  
  
Part  
01  
  
漏洞评级为中危  
  
  
派拓网络将该漏洞评为中危，CVSS威胁评分为6.1，基础评分为8.6。厂商将补丁修复优先级列为中等，但提醒用户，如果设备允许来自外网地址的管理访问，面临的风险将达到最高水平。  
  
  
该漏洞与此前披露的PAN-OS root权限代码执行漏洞CVE-2026-0322相互独立。此前的CVE-2026-0322已出现被利用的案例，但这些攻击不能作为攻击者已在利用本次新披露漏洞的证据。  
  
  
Part  
02  
  
多版本PAN-OS已发布补丁  
  
  
目前Panorama平台的PAN-OS 12.2、12.1、11.2、11.1、10.2版本均已推出修复补丁。各分支最新维护线的修复版本包括12.2.4-h2、12.1.10-h4、11.2.16-h2、11.1.16-h5、10.2.18-h14。管理员可参照公告中的升级指引，适配当前所使用的版本完成升级。  
  
  
更早的维护分支也已推出对应修复补丁。PAN-OS 12.1分支的修复版本包括12.1.4-h14、12.1.7-h12；PAN-OS 11.2分支的修复版本包括11.2.4-h24、11.2.7-h24、11.2.10-h18、11.2.13-h5。更旧的11.1、10.2维护版本也有单独的热修复补丁可用。  
  
  
使用已停止支持版本的用户，需升级至仍在维护周期内、已发布修复的版本。目前没有已知的临时规避方案可以修复该漏洞的根本问题，派拓网络建议用户将管理访问权限限制在受信任的内部IP地址范围内。  
  
  
Part  
03  
  
建议限制管理接口访问  
  
  
派拓网络的管理安全规范明确提醒，不要将防火墙或Panorama的管理接口暴露在互联网或其他不可信网络中。  
  
  
用户可将管理接口的访问权限限制为专属跳板机，进一步缩小攻击暴露面。此前已有多起PAN-OS管理接口暴露引发安全事件的报道，足以说明这一访问边界防护的重要性。  
  
  
需要注意的是，访问限制措施仅能降低风险，无法替代安全补丁的修复作用。  
  
  
参考来源：  
  
Palo Alto PAN-OS RCE Flaw Allows Attackers to Execute Arbitrary Code With Root Privileges  
  
https://cybersecuritynews.com/palo-alto-networks-pan-os-rce-vulnerability/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
  
