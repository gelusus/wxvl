#  CISA确认Outlook 0Day遭在野利用，可泄露用户认证数据  
 FreeBuf   2026-11-13 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX0o6Ov0Diaqnicthk0j1TqNfEwCfg6eibeb2ug13iaKEJPse42xh7Pn2SqibCmJSBibyeFyX8vj9jqA9tOic5JmsaGrWtEkEFRTTswqpE/640?wx_fmt=gif "")  
  
  
![Outlook 0Day漏洞](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX20FO50jaeLPMdHibJmcRQcDEkegTzlicms9yicDRFgJnY3I1wIpibkaaYFLpxUIw1aKRBibhgUzdEnXicKG2hZW8t2ZGdRpA2AuA5DI/640?wx_fmt=png "")  
  
  
Part  
01  
  
特制消息文件可触发漏洞  
  
  
CISA证实，微软Outlook存在一个编号为CVE-2026-124194的0Day漏洞，目前已遭在野利用。攻击者可利用该漏洞窃取用户的认证数据。  
  
  
根据微软发布的安全公告，攻击者可发送特制的消息文件，诱导受害者在Outlook中打开。此时Outlook不会弹出常规的外部内容警告，会直接连接到攻击者控制的网络地址。  
  
  
该连接会泄露受害者的NTLMv2哈希值，攻击者可利用这类认证数据仿冒用户身份。微软将该漏洞归类为欺骗类漏洞，而非远程代码执行漏洞。经评估，该漏洞会对机密性造成严重影响，但不会直接影响数据完整性或服务可用性。  
  
  
该漏洞CVSS评分为6.5，未经身份验证的攻击者即可利用它实施网络欺骗。微软目前已针对该漏洞发布安全更新。  
  
  
该漏洞属于CWE-356范畴，这类缺陷的核心特征是软件执行潜在危险操作前，未获得用户的明确许可。本次漏洞中，Outlook连接外部网络地址前缺失警告提示，相当于绕过了一道安全检查机制。  
  
  
Part  
02  
  
漏洞并非0-Click攻击  
  
  
值得注意的是，连接阶段无用户交互提示，并不意味着这是0-Click攻击。微软明确指出，受害者必须手动打开恶意文件才会触发漏洞，Outlook预览窗格并非该漏洞的攻击载体。这一特征可以帮助防御方将其与此前无需用户操作的Outlook漏洞区分开。  
  
  
Cyber Security News此前曾报道俄罗斯黑客利用CVE-2023-23397攻击Outlook，以及CISA发布的MonikerLink漏洞警告。这些报道可以作为Outlook漏洞利用的背景参考，但其中的攻击方法不能直接套用到本次新漏洞上。  
  
  
Part  
03  
  
CISA将漏洞纳入KEV目录  
  
  
本次漏洞影响的产品包括32位和64位版本的Microsoft 365 Apps for Enterprise、Office LTSC 2021、Office LTSC 2024。其中Office LTSC 2021的修复版本号为16.0.14334.21048，Office LTSC 2024的修复版本号为16.0.17932.21140。Microsoft 365管理员可参考公告中链接的Office安全发布指南，确定自身部署环境对应的正确更新版本。  
  
  
CISA已于2026年11月10日将该漏洞加入KEV目录，要求各机构在11月24日前完成修复。各机构需落实厂商提供的缓解措施，排查自身互联网暴露面。如果暂时没有可用修复方案，应立即停用受影响产品。  
  
  
管理员应排查所有受影响的Office安装，部署对应补丁。修复完成后需核实实际安装的版本号，不能仅依赖更新部署报告判断修复结果。各团队在确定补丁优先级时，不能因为该漏洞被标记为中危级别，就忽视其已被在野利用的事实。  
  
  
目前微软已检测到针对该漏洞的利用活动，但已公开的公告中并未披露攻击者身份、受害者范围、攻击活动规模或IOC。这些信息缺口应明确告知相关方，不能直接用以往Outlook攻击的细节来填充。  
  
  
参考来源：  
  
CISA Confirms Active Exploitation of Outlook User Interaction 0-Day Vulnerability  
  
https://cybersecuritynews.com/active-exploitation-of-outlook-0-day-vulnerability/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
  
