#  PHP 曝两个"满分级"漏洞，多国政府网站连夜下线  
NERCIS
                    NERCIS  信息安全国家工程研究中心   2026-10-23 09:15  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/jGCFB5hoMibIsSciajVCLCCOh6mZeL2jZWrpoBkIUu8k1nwPvXGzyHKk6aG1nBRf4gzLic9A1PYs3MfokADeWNxMRDpgLahsSLJ0v8JkUf03p4/640?wx_fmt=gif&from=appmsg "")  
  
据路透社 10 月 21 日报道，一种广泛使用的编程语言被曝光高危漏洞后，欧洲多国及其他地区的政府网站当天纷纷下线，世界各地的网站运营方紧急行动，保护各自系统。出事的是 PHP。这门诞生于 20 世纪 90 年代初的服务器端脚本语言，支撑着全球约七成网站。10 月 21 日，PHP 开发团队紧急发布四个修复版本——8.5.12、8.4.27、8.3.36 和8.2.35，并敦促所有用户"立即更新"。  
  
这次的两处漏洞编号为 CVE-2026-109618 和 CVE-2026-109619，被合称为"PHP2Shell"，CVSS v4.0 评分均为 9.2，属"严重"级别。攻击者无需任何身份验证，即可通过构造特定的HTTP 请求触发内存破坏，在服务器上远程执行代码，直接接管服务器、窃取业务数据。  
  
PHP 官方提示，所有对外提供 Web 服务的 PHP 部署都存在风险，其中 Apache mod_php配置下风险最高——攻击可在应用代码执行之前就被触发。值得注意的是漏洞的发现方式。  
  
网络安全公司 Calif 称，该公司借助大语言模型于 8 月 1日发现相关问题；谷歌旗下网络安全企业 Wiz 也在独立报告中表示，其借助自研工具 WizAtlas，以及 Gemini 4 Argon、GPT-5.6 Sol 等模型发现了这些漏洞。OpenAI 表示，过去两个月其牵头"Patch the Planet"计划，联合维护者、研究机构和安全服务商协同推进漏洞验证、补丁开发和防护部署。据 Wiz 披露，至少其中一个漏洞影响过去十年发布的每一个PHP 版本，部分问题代码可追溯到 PHP 5.4。由于部分黑客已经在利用这些漏洞，很多机构选择把网站下线作为预防措施。荷兰国家网络安全中心率先下线自家网站，众议院、鹿特丹港及多家医院网站随后跟进；芬兰暂停政府面向公众的网络服务；法国内政部将各省政府网站置于"预防性维护"状态；德国联邦 IT服务机构 ITZBund 关闭多个网站；英国牛津市议会网站以"影响底层技术的全球性问题"为由临时关闭，英国国家网络安全中心（NCSC）发出警报；日本首相高市早苗当晚在社交平台呼吁各方"今晚就采取行动"；澳大利亚信号局深夜发布警告，部分政府网站在修复期间保持不可访问状态。  
  
补丁发布并不意味着风险解除。网络安全研究与事件响应机构 TLP:Black 联合创始人科斯廷·拉尤说，因为补丁现在已经公开，人工智能工具可以毫不费力地对其进行逆向工程，黑客会抢在系统完成更新之前发起攻击，人工智能正在驱动一场"漏洞利用竞赛"。他判断，下一步是互联网上所有存在漏洞的网络服务器遭到大规模入侵。攻击确实抢跑了。Cloudflare 在格林尼治时间 21 日 7 时 43 分就监测到首批攻击，比补丁发布早了数小时；该公司首席技术官 Dane Knecht 表示，已为全部客户设置拦截规则，但这只是临时措施。美国国家安全局（NSA）建议，无法立即更新的机构应将系统下线，直至完成修复；美国网络安全和基础设施安全局（CISA）也敦促相关机构尽快安装补丁。  
  
在国内，腾讯云安全中心、阿里云安全团队、奇安信 CERT 等同步发布风险通告，提醒相关企业尽快开展资产自查，升级至对应 PHP 安全版本。计算机取证专家 Hal Pomeranz 提醒，大量无人维护的老旧网站可能永远不会更新，隐患将长期存在。OpenAI 则表示，PHP 漏洞只是其发现的多个重大漏洞之一，预计类似发现将大幅增加。  
  
来源：路透社、新华社、澎湃新闻、财联社、凤凰网等公开报道，  
关键信息基础设施安全保护联盟  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/jGCFB5hoMibLTMZwT1kCPRpKdP2nibsJqhfTiblKgOLEmXNc7AMbXX9zPW6n69qmT0yf7pS5LicXPUmkaEAArOwkgn9rB7ciaJiaE8YlSpQrehNCE/640?wx_fmt=gif&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/jGCFB5hoMibJibsQ7omu61vK4m0et959g0jjgQyiaMkqRGzs1tbQO8cWbLDj9J6w6K8uUmgwmC772XyArQWQYUWc2DDahHIToM1jdezbEm6084/640?wx_fmt=gif&from=appmsg "")  
  
