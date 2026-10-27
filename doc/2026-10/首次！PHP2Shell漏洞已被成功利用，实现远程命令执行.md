#  首次！PHP2Shell漏洞已被成功利用，实现远程命令执行  
安全内参编译
                    安全内参编译  安全内参   2026-10-27 08:29  
  
**关注我们**  
  
  
**带你读懂网络安全**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/wT9KAyOic0NBIMb8Z4103h2SCrO3JqHfeVMjeIlPQMZzqqmYMsabIvvMuelIxvicpzkVRibM21PutiaRgK5LORpWBxtFXgAkEWCHbuicjblWAaYU/640?wx_fmt=jpeg&from=appmsg "")  
  
  
上周披露的PHP2Shell漏洞让全球进入应急响应状态，此前已经出现大量攻击利用，但一直未有造成严重影响的案例；  
  
  
日前有研究员监测到可能是首个实现远程命令执行的攻击请求，代表该漏洞的稳定利用有了较大进步，各组织应尽快排查和修复受影响资产。  
  
  
前情回顾·  
全球漏洞利用态势  
- [CNCERT：关于PHP存在远程代码执行漏洞的安全公告](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516763&idx=2&sn=6d5b7b6719e344ae5575811ba0decc7f&scene=21#wechat_redirect)  
  
  
- [VPN漏洞遭勒索软件大规模利用，Palo Alto警告多家客户被入侵](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247516757&idx=1&sn=f9f6348a423a0db56bbc2348f9f5914e&scene=21#wechat_redirect)  
  
  
- [苹果漏洞利用工具遭公开，数亿台iPhone随时可被静默入侵窃密](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247515729&idx=1&sn=e89ad98e20e9e813c713f2faa4fdd570&scene=21#wechat_redirect)  
  
  
- [2025年至少90个零日漏洞遭滥用，企业软件及设备占比近一半](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247515644&idx=1&sn=5225e463062df00e90439d4927d5cfc1&scene=21#wechat_redirect)  
  
  
安全内参10月27日消息，近期引发轩然大波的PHP2Shell漏洞现已确认遭到成功利用。该系列漏洞包括两个远程代码执行（RCE）漏洞（CVE-2026-109618、CVE-2026-109619）。  
  
  
首次检测到成功的RCE利用  
  
  
  
AI搜索厂商Perplexity检测与响应负责人Adel Karimi通过蜜罐捕获了相关恶意活动。记录显示，攻击者已成功在一套运行PHP 8.4.26的Apache mod_php环境中实现远程代码执行。  
  
Karimi表示：“目前已有确凿证据证实攻击者实现了远程代码执行，并成功获取可用的内存泄露数据。这并不只是导致程序崩溃、随后出现可疑进程那么简单。攻击返回的数据中有一份解码后的Cookie转储文件，大小达800 KiB。根据公开漏洞利用代码中的偏移量，从这份数据中可以提取出一个函数地址。这个地址随后出现在攻击者执行的载荷中。”  
  
他补充说：“这些证据证实，这一攻击链在单工作进程mod_php环境中确实可行。该环境被刻意设置为对攻击者有利。但这并不能说明，该漏洞利用方式在其他工作进程数量、不同PHP构建版本或不同应用环境中同样可靠。”  
  
Expel威胁研究员Marcus Hutchins也证实了Karimi的发现。此前，Hutchins一直在跟踪一系列有关PHP2Shell漏洞被成功利用的报告。  
  
![Adel Ka PHP2Shell 推文](https://mmbiz.qpic.cn/sz_mmbiz_png/wT9KAyOic0NBB8274kL38eEHkEibheibb4zMWA9MzULSg75eiadq27KiauPf69YSwO5ucRjXH7uCp19rqfwxF8xFoNxb0XDDphzC8mb7K1oHqGRM/640?wx_fmt=png&from=appmsg "")  
  
Hutchins表示：“这是迄今为止我所见到的、最有力的PHP2Shell在野利用证据。相关分析深入拆解了一次成功攻击的技术细节。这次攻击针对的是运行PHP 8.4.26的Apache mod_php环境。”  
  
  
PHP2Shell漏洞引发全球应急响应  
  
  
  
上周五，美国网络安全和基础设施安全局（CISA）发布了一条SIGMA规则，用于检测针对PHP2Shell远程代码执行漏洞的利用尝试。但该机构尚未将这两个漏洞列入已遭在野利用的漏洞名单。  
  
PHP安全团队于10月21日修复了PHP2Shell漏洞。就在前一天，该团队提醒管理员提前做好准备，以便安全更新发布后立即安装。  
  
在补丁正式发布前，PHP2Shell漏洞的相关信息已在网上泄露，包括CVE编号，以及提供给部分关键基础设施运营方的缓解措施建议。此外，一些概念验证（PoC）漏洞利用代码也已公开。不过，PHP安全团队表示，这些代码都无法真正奏效。  
  
与此同时，Cloudflare、Hadrian、watchTowr、Previdian和Patchstack等多家网络安全公司在漏洞公开披露前就发现了相关利用尝试，但都认定这些攻击没有成功。  
  
亚马逊威胁情报团队的安全研究人员也发现，攻击者早在安全补丁发布前四天就已开始针对PHP2Shell漏洞展开攻击。然而，据Dataminr旗下威胁研究与网络工程（TRACE）团队介绍，在Karimi公布相关发现之前，这些恶意活动“仅限于扫描、识别目标系统特征，以及使用无法奏效的漏洞利用代码，尚无已知系统遭到入侵”。  
  
上周五，PHP安全团队还公布了另一个远程代码执行漏洞的缓解措施。该漏洞编号为CVE-2026-115560，目前尚无可用补丁，只有在目标系统的配置与应用程序代码存在“不寻常组合”时才能被利用。  
  
在安全补丁发布之前，PHP开发团队建议管理员将`session.upload_progress.enabled`设置为`0`，以阻止潜在攻击。在PHP2Shell安全更新发布前，该团队也曾建议采取这一缓解措施。  
  
PHP是一种广泛用于Web开发的编程语言。根据VirusTotal近期的估算，全球约70%的网站使用PHP。暴露在互联网上的PHP主机超过510万台，其中约380万台至少有一个端口开放，可能受到攻击。  
  
  
**参考资料：bleepingcomputer.com**  
  
  
**推荐阅读**  
- [网安智库平台长期招聘兼职研究员](http://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247499450&idx=2&sn=2da3ca2e0b4d4f9f56ea7f7579afc378&chksm=ebfab99adc8d308c3ba6e7a74bd41beadf39f1b0e38a39f7235db4c305c06caa49ff63a0cc1d&scene=21#wechat_redirect)  
  
  
- [欢迎加入“安全内参热点讨论群”](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247501251&idx=1&sn=8b6ebecbe80c1c72317948494f87b489&chksm=ebfa82e3dc8d0bf595d039e75b446e14ab96bf63cf8ffc5d553b58248dde3424fb18e6947440&token=525430415&lang=zh_CN&scene=21#wechat_redirect)  
  
  
  
  
  
  
  
点击下方卡片关注我们，  
  
带你一起读懂网络安全 ↓  
  
  
  
  
