#  突发！PHP 两大严重 RCE 漏洞已被利用，官方紧急补丁发布  
原创 didiplus
                    didiplus  攻城狮成长日记   2026-10-22 04:43  
  
安全警报  
预计阅读   
10  
 分钟 · 共 3970 字  
  
突发！PHP 两大严重 RCE 漏洞已被利用，官方紧急补丁发布  
  
CVE-2026-109618 / CVE-2026-109619 · CVSS 9.2  
  
[#PHP]()  
  
[#RCE漏洞]()  
  
[#紧急补丁]()  
  
[#运维]()  
  
  
READING PATH  
  
阅读路线  
  
7 个章节  
  
01  
  
漏洞消息提前泄露，PHP 官方紧急应对  
  
-  
  
02  
  
这两项漏洞到底有多危险？  
  
-  
  
03  
  
最需要警惕的是：你可能正在间接使用 PHP  
  
-  
  
04  
  
官方补丁已经发布！哪些版本需要升级？  
  
-  
  
05  
  
网站管理员现在应该怎么做？这份排查清单请收好  
  
-  
  
06  
  
补丁发布之后，还要不要检查攻击日志？  
  
-  
  
07  
  
最后提醒：别让旧版本成为服务器的长期隐患  
  
如果你的网站正在使用 PHP，现在请先放下手头的工作，检查一下服务器上的 PHP 版本。  
  
紧急  
两项严重 RCE 漏洞，已有实际利用尝试  
  
CVSS v4.0 评分 9.2 · 严重  
  
[#CVE]()  
-2026-109618  
[#CVE]()  
-2026-109619  
  
PHP 核心曝出两项严重远程代码执行（RCE）漏洞，编号   
CVE-2026-109618  
 与   
CVE-2026-109619  
，CVSS v4.0 评分均为 **9.2 分**  
。漏洞信息在原定披露时间前已提前泄露，安全研究机构 watchTowr 已观察到实际利用尝试。  PHP 官方已于 **10 月 21 日**  
发布安全更新。风险不只是"可能存在漏洞"，而是需要运维人员立即行动。  
  
![179263474253a3.png](https://mmbiz.qpic.cn/sz_mmbiz_png/kztGyHFwmfw5hf75BJ6nkhmgUC1WmKRbGaBx2uEIjib56NO4sZWxSMkiap4iaJa5v4RxkOEmgNQjhQ7Pvib4ZCSvVdNrEjWwneHp9pbfkN7ez7c/640?wx_fmt=png&from=appmsg "")  
  
CHAPTER 01  
  
**01****漏洞消息提前泄露，PHP 官方紧急应对**  
TIMELINE  
  
先回顾一下这次事件的时间线。  
  
10 月 20 日  
  
预告发布  
  
PHP 安全团队提前发布更新预告，计划 10 月 21 日 14:00 UTC 修复  
  
  
披露前  
  
信息泄露  
  
漏洞信息在正式披露前已在网络提前泄露  
  
  
随后  
  
watchTowr 确认  
  
安全研究公司确认漏洞真实存在并成功复现  
  
  
10 月 21 日  
  
官方补丁  
  
PHP 官方正式发布安全更新，修复两项漏洞  
  
  
10 月 22 日  
  
立即排查  
  
网站管理员开展修复排查  
  
这类事件最令人担忧的地方在于：漏洞信息提前曝光，并不意味着所有网站管理员都能及时获知风险。攻击者可能立即研究利用方式，而普通维护人员可能还在等待官方公告，甚至不知道自己的服务器已处于风险之中。  
  
原定的补丁发布计划已经结束，现在真正需要关注的是：  
你的服务器是否已经完成升级  
。  
  
![179263474253a3.png](https://mmbiz.qpic.cn/mmbiz_png/kztGyHFwmfwCZJiceIGk1w8mHKvUqFSnc7GibUoiaTnoS3H08lz4mTmoVGRAHb5QibGb1BoJbdibtOwbwHUabHCTI6AKgAEVFZjOhDqpnnOckbtA/640?wx_fmt=png&from=appmsg "")  
  
CHAPTER 02  
  
**02****这两项漏洞到底有多危险？**  
RCE EXPLORED  
  
先明确一个概念：什么是远程代码执行漏洞？  
  
简单来说，远程代码执行漏洞意味着攻击者在满足特定条件时，可以通过网络向存在漏洞的系统发起请求，进而让目标服务器执行攻击者控制的代码。  
  
如果攻击成功，后果可能包括：  
  
后果 01  
读取服务器上的敏感文件和业务数据  
  
后果 02  
窃取数据库连接信息、应用密钥等凭据  
  
后果 03  
植入恶意程序或后门  
  
后果 04  
篡改网站内容，破坏正常业务  
  
后果 05  
利用服务器权限进一步攻击内部系统  
  
具体影响还取决于服务器配置、运行账户权限、网络隔离和其他安全措施。这次漏洞尤其值得重视的原因，是其攻击可能  
不需要经过传统的用户登录认证  
。  
  
根据 Google Cloud 技术说明，两项漏洞涉及 PHP 请求处理过程中的内存异常和内部状态管理问题：  
  
**CVE-2026-109618**  
  
与内存耗尽处理过程中的异常状态有关。当请求数据尚未完整初始化时，后续清理逻辑可能错误地处理相关数据。  
  
**CVE-2026-109619**  
  
与已释放内存对应的内部状态被后续请求继续使用有关，属于典型的   
Use-After-Free  
 问题。  
  
这类底层运行时漏洞发生在 PHP 核心运行环境中，不能仅靠更新 WordPress 插件或修改业务代码就认定风险消除。  
  
漏洞评分 9.2 反映严重程度，不代表所有服务器都会以相同方式被攻击，也不代表攻击成功后必然获得最高系统权限。  
  
![179263474253a3.png](https://mmbiz.qpic.cn/mmbiz_png/kztGyHFwmfysQ2rdvpxrpp6WufCSPFnsYqu82PIewIWX2sOjOrHx3kxCnV6icOyaCMK3ZrHPy4GVzvlTMTibjUtvQHwxMicqOg2g1H6LWoxxLE/640?wx_fmt=png&from=appmsg "")  
  
CHAPTER 03  
  
**03****最需要警惕的是：你可能正在间接使用 PHP**  
HIDDEN DEPENDENCY  
  
很多人看到 PHP 漏洞，第一反应是："我又不开发 PHP，应该跟我没关系吧？"  
  
事实上，这种想法并不安全。PHP 广泛应用于网站和 Web 应用的服务端。即使你没有亲自编写 PHP 代码，正在使用的网站平台、业务系统或托管服务，也可能依赖 PHP 运行。  
  
依赖 01  
WordPress 等内容管理系统  
  
依赖 02  
Magento 等电商平台  
  
依赖 03  
使用 PHP 开发的企业管理系统  
  
依赖 04  
论坛、博客和其他建站程序  
  
依赖 05  
PHP-FPM、Apache mod_php 等方式运行的 Web 服务  
  
依赖 06  
容器镜像中内置的 PHP 运行环境  
  
这里还有一个容易被忽视的问题：**应用本身没有更新，不代表底层 PHP 运行环境也得到了修复；反过来，应用升级了，也不一定意味着 PHP 已经升级。**  
  
例如网站使用 WordPress，管理员可能已把 WordPress 核心更新到最新版，却仍让服务器运行旧版 PHP。漏洞存在于 PHP 运行环境时，应用程序更新并不能替代 PHP 安全补丁。  
  
同样，Docker 容器、云主机、虚拟主机和托管平台也需分别确认。只检查自己电脑上的 PHP 版本，并不能证明线上服务器已经安全。  
  
![179263474253a3.png](https://mmbiz.qpic.cn/mmbiz_png/kztGyHFwmfwgCfb585FGaMlwfgZtzUI0XuMEJjnaLJW3awwyagUocb9xMRibUHInb9ZcXBiagHnkmfYNLO5QtrIPr92fmYfGoVibVoxKHPB24E/640?wx_fmt=png&from=appmsg "")  
  
CHAPTER 04  
  
**04****官方补丁已经发布！哪些版本需要升级？**  
PATCH MATRIX  
  
这是本文最重要的部分。根据 PHP 官方公告，已为当前受支持的主要分支发布修复版本：  
  
<table><thead><tr><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;font-weight:700;color:rgb(51,65,85);"><section><span leaf="">PHP 分支</span></section></td><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;font-weight:700;color:rgb(51,65,85);"><section><span leaf="">修复版本</span></section></td></tr></thead><tbody><tr><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">PHP 8.5</span></section></td><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">8.5.12</span></section></td></tr><tr><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">PHP 8.4</span></section></td><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">8.4.27</span></section></td></tr><tr><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">PHP 8.3</span></section></td><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">8.3.36</span></section></td></tr><tr><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">PHP 8.2</span></section></td><td valign="top" align="left" style="vertical-align:top;border:0px;padding:0px;text-align:left;font-size:13px;color:rgb(51,65,85);"><section><span leaf="">8.2.35</span></section></td></tr></tbody></table>  
  
如果你的服务器运行上述分支，请核对实际安装版本，尽快升级至对应修复版本或该分支更新的安全版本。潜在影响范围涉及 PHP 5.4 至 PHP 8.5.11。  
  
漏洞影响范围较广，不代表每一个历史版本都能获得官方修复。  
  
PHP 8.1 及更早版本已结束官方维护，PHP 项目没有为这些旧分支发布本次对应的官方修复版本。如果业务仍依赖 PHP 7.x、PHP 8.0 或 PHP 8.1，不能简单地认为"升级到旧分支最新版本"就一定安全。  
  
OLD VERSION  
  
旧分支应对策略  
  
1  
  
评估迁移  
  
评估升级到受支持分支的可行性  
  
2  
  
核查回补  
  
暂时无法迁移时，核实操作系统发行版或商业维护方是否提供安全回补，确认补丁覆盖相关问题  
  
3  
  
核对状态  
  
不要仅凭版本字符串判断，部分 Linux 发行版会把安全修复回补到较旧上游版本号中，需同时核对软件包安全公告和实际补丁状态  
  
官方公告：https://www.php.net/archive/2026.php  
  
![179263474253a3.png](https://mmbiz.qpic.cn/sz_mmbiz_png/kztGyHFwmfyRL0PiadIibDld8rVRFA7geuQsKGAxnPe2lbTALZicFLcJ9Thf2YeB3J2pDRfYhnWM3cqRSLQJAHEgtsZjcbAiaZnicmbZRktNOvRc/640?wx_fmt=png&from=appmsg "")  
  
CHAPTER 05  
  
**05****网站管理员现在应该怎么做？这份排查清单请收好**  
ACTION CHECKLIST  
  
如果你负责网站运维、服务器管理或企业信息系统，建议按以下顺序开展排查。  
  
01  
  
第一步：盘点所有 PHP 运行环境  
  
不要只检查主站。建议同时核对：正式环境和测试环境、云主机/物理服务器/虚拟机、Docker 容器及其基础镜像、Apache/PHP-FPM/php-cgi 等运行方式、虚拟主机和第三方托管平台、内部业务系统和长期未维护的旧项目。  
  
在能够执行命令的服务器上，可以先运行：  
  
bash  
  
php -v  
  
查看当前命令行 PHP 的版本。但需要注意，命令行 PHP 与网站实际使用的 PHP 可能不是同一个版本。例如命令行使用 PHP 8.3，而网站通过 PHP-FPM 运行另一个版本。因此还应检查 Web 服务配置、PHP-FPM 进程、容器镜像以及托管平台提供的运行时信息。  
  
02  
  
第二步：优先修复公网暴露的服务器  
  
如果资源有限，不妨先按风险排序：  
  
1.  
直接面向互联网、承载真实业务的 PHP 服务器。  
  
2.  
使用 Apache mod_php 等高风险配置的系统。  
  
3.  
运行旧版本 PHP、长期未维护的站点。  
  
4.  
内网业务系统和测试环境。  
  
这并不意味着内网系统可以忽略，而是应优先处置攻击面更大的资产。  
  
03  
  
第三步：升级后验证，而不是只看安装过程  
  
在维护窗口内完成升级，并确认 Web 服务实际加载的是修复后的 PHP 运行环境。建议执行以下检查：  
  
●  
确认安装的软件包或运行时版本。  
  
●  
检查 PHP-FPM、Apache 等相关服务状态。  
  
●  
通过业务测试确认网站功能正常。  
  
●  
检查错误日志和异常请求。  
  
●  
确认容器重新部署后没有继续使用旧镜像。  
  
●  
确认负载均衡后的所有后端实例均已完成更新。  
  
尤其是容器化部署，不要只更新正在运行的容器而忽略镜像构建流程，否则后续重新部署时旧镜像可能再次把漏洞带回来。  
  
04  
  
第四步：暂时无法修复怎么办  
  
对于无法立即升级的旧系统，应尽可能降低暴露风险。可以考虑：暂时限制公网访问、通过防火墙/访问控制列表/VPN 限制访问来源、在业务允许情况下隔离高风险服务、联系云服务商或托管平台确认修复计划、为迁移和升级制定明确时间表。  
  
需要说明，WAF 规则、关闭某个功能或调整 Web 配置，不一定能完整阻断这两项漏洞的所有利用路径。PHP 官方已明确指出，没有能够完全替代安全更新的通用缓解措施。  
  
CHAPTER 06  
  
**06****补丁发布之后，还要不要检查攻击日志？**  
POST-PATCH AUDIT  
  
答案是：需要。  
  
补丁解决的是后续漏洞利用风险，但不能自动证明此前没有发生过入侵。watchTowr 后续分析报告称，已观察到针对相关漏洞的实际利用尝试。安全团队应结合自己的资产暴露情况，评估是否存在被攻击的可能。  
  
如果服务器在补丁发布前长期暴露于公网，或使用了受影响的运行环境，建议进一步检查：  
  
●  
Web 访问日志中异常的请求模式。  
  
●  
PHP、Apache、Nginx 和 PHP-FPM 的错误日志。  
  
●  
异常进程、可疑命令执行和非预期的网络连接。  
  
●  
网站目录中新增或被修改的可疑文件。  
  
●  
新增账户、异常计划任务和启动项。  
  
●  
云平台审计记录以及数据库访问日志。  
  
不要因为升级成功、网站可以正常打开，就直接认定此前没有受到攻击。如果发现明确的入侵迹象，应保留日志和相关证据，隔离受影响系统，并按照应急响应流程调查。对于已泄露的数据库凭据、API 密钥和其他秘密信息，还应评估轮换的必要性。  
  
同样需要避免过度解读：安全机构观察到攻击尝试，不代表每一台存在漏洞的服务器都已经被攻破。具体风险仍需结合日志、系统状态和业务环境判断。  
  
CHAPTER 07  
  
**07****最后提醒：别让旧版本成为服务器的长期隐患**  
FINAL REMINDER  
  
这次 PHP 安全事件再次提醒我们，网站安全不仅仅是安装一个防火墙，或者定期更新几个应用插件。真正有效的安全管理，至少需要做到三件事：  
  
SECURE MGMT  
  
有效安全管理的三件事  
  
1  
  
知道运行了什么  
  
服务器、容器、应用和第三方托管环境都应纳入资产清单  
  
2  
  
知道漏洞在哪修  
  
PHP 核心漏洞需核对并修复 PHP 运行环境，而不是只更新业务应用  
  
3  
  
知道修复后验证什么  
  
版本确认、服务验证、日志审计和必要的应急响应，一个都不能少  
  
目前 PHP 官方已发布针对 CVE-2026-109618 和 CVE-2026-109619 的修复版本。对于仍在运行受影响环境的管理员来说，现在最重要的不是继续等待消息，而是尽快完成资产排查、补丁升级和风险验证。  
  
如果你正在维护 PHP 网站，建议今天就检查一遍。  
  
  
  
转发给负责服务器和网站运维的同事  
  
转发提醒  
  
你的服务器 PHP 版本排查了吗？  
  
评论区聊聊修复进度，关注我持续分享安全干货  
  
·  
 点赞   
·  
  
喜欢就点个赞吧  
  
  
  
·  
 转发   
·  
  
分享给更多朋友  
  
  
  
·  
 推荐   
·  
  
推荐给身边的人  
  
  
### 参考资料  
  
1.  
PHP 官方安全公告[1]  
  
2.  
watchTowr 漏洞分析[2]  
  
3.  
Google Cloud 安全分析[3]  
  
4.  
BleepingComputer 漏洞修复报道[4]  
  
## 参考资料  
  
[1]  
PHP 官方安全公告  
：https://www.php.net/archive/2026.php  
  
[2]  
watchTowr 漏洞分析  
：https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/  
  
[3]  
Google Cloud 安全分析  
：https://cloud.google.com/blog/products/identity-security/responding-to-core-php-vulnerabilities-cve-2026-109618-cve-2026-109619  
  
[4]  
BleepingComputer 漏洞修复报道  
：https://www.bleepingcomputer.com/news/security/php-ships-emergency-patches-for-critical-php2shell-flaws/  
  
  
