#  PHP 中的高危远程代码执行漏洞  
原创 卡巴斯基
                    卡巴斯基  卡巴斯基威胁情报   2026-10-23 14:01  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5DBHibIELyXAOy6cGria8MQUgichvYnFzH71wU4qbc5xkTwty6Hjrxn0j6CrZjYEsdhLF3QSMgtHfCdZHEGBQxOu96iaB4u2PLGtD1BhxMMJR28/640?wx_fmt=png&from=appmsg "")  
  
更新 2026年10月22日：新增卡巴斯基托管检测与响应服务检测上述恶意活动的相关说明。  
  
更新 2026年10月22日：新增卡巴斯基漏洞管理识别存在漏洞的PHP版本的相关说明。  
  
  
  
  
**漏洞概述**  
  
  
2026年10月20日，PHP安全团队发布一则预通知，预告即将推出紧急版本，修复两个高危漏洞（两项漏洞的CVSS评分均为9.2分）。相关补丁已于世界协调时间2026年10月21日周三14:00发布。  
  
由于漏洞相关信息在原定保密期限解禁前泄露至公众，本次版本发布被紧急提前。有用户在黑客新闻（Hacker News）发帖，称PHP存在两个远程代码执行（RCE）漏洞，据称从PHP 5.4.0起的所有版本均受影响。  
  
涉事漏洞编号为  
CVE-2026-109618和  
CVE-2026-109619。该帖子发布后不久，研究人员证实了漏洞真实存在。伴随漏洞细节泄露，一款非官方命名为PHP2Shell的漏洞利用程序在网上流传。泄露的演示视频显示，该利用程序可在运行WordPress、MediaWiki、phpBB、Magento、Joomla、Laravel、Symfony、Drupal以及WHMCS的网页服务器上生效，这些平台支撑着大量现代网站。本文发布时，该演示视频的真实性以及漏洞利用程序的确切行为尚未经过独立验证。  
  
据发布概念验证（PoC）的研究人员介绍，这些漏洞和PHP内核中multipart/form-data MIME类型的处理逻辑、会话上传进度机制（session.upload_progress）相关，属于释放后重用（use-after-free）与内存损坏类漏洞。这意味着无法通过修改应用代码完成修复，唯一可行的解决方式是更新服务器上的PHP程序。该攻击无需身份验证：攻击者只要能够访问PHP应用所在网络，无需账号认证，就可以以PHP进程的身份执行任意代码。PHP官方在10月21日发布的安全公告也证实了这一点。  
  
PHP 5、6、7、8系列版本均受这些漏洞影响。截至本文发布，8.2至8.5分支仍在官方支持周期内。PHP 8.1、8.0、7.x、5.x等旧版本早已停止维护；但鉴于漏洞危害等级极高，官方也为7.4、8.0和8.1版本提供了反向移植补丁。  
  
Sansec安全专家预估，补丁发布后的15分钟内就可能出现大规模攻击利用。我们强烈建议负责对外提供访问的PHP网页服务器管理员，**在补丁就绪前执行以下操作**  
：  
  
  
- 对基础设施中所有基于PHP的系统开展盘点（包括网页服务器、控制面板、第三方软件等）。  
  
- 禁用上传进度功能：在PHP-FPM池配置中通过php_admin_flag将session.upload_progress.enabled设置为Off。  
  
- 在nginx和Apache服务器上拦截多部分请求。  
  
- 考虑将联网的PHP应用临时隔离，直至官方修复程序发布。  
  
- 做好准备，以便补丁一经发布可立即安装。  
  
- 制定迁移方案，将运行不受支持版本（8.1及更早版本）的服务器迁移至最新PHP分支。  
  
  
  
  
  
  
**卡巴斯基产品检测**  
  
  
卡巴斯基解决方案可在攻击的多个阶段检测到上述恶意行为，可能的检测场景如下。当有人尝试利用这些远程代码执行（RCE）漏洞时，卡巴斯基终端检测与响应系统会及时向用户告警，提示在Linux和Windows设备的Web服务器环境中出现可疑操作。  
该行为可通过以下规则进行检测：  
- creating_windows_shell_from_a_web_servers_process;  
  
- possible_web_shell_creating_via_exploit;  
  
- spawning_process_from_webserver_more_filters_linux;  
  
- webserver_interpreter_execution_linux;  
  
- executable_file_in_www_folder.  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/5DBHibIELyXCck7fcNjZXgmffdicLE2xaicB9iaJSANp6FIahkWk2WDGWaefYq6ib4N99ncb8TwV4IOUgKLpTwibtyMXwrvkxkVJY61GzeHmiaqZsA/640?wx_fmt=png&from=appmsg "")  
  
**卡巴斯基检测到PHP2Shell漏洞利用行为**  
  
  
卡巴斯基反定向攻击平台（KATA）检测到网络流量中存在针对PHP2Shell漏洞的利用行为。该安全方案通过如下判定标识识别此类活动：Exploit.CVE-2026-109618.HTTP.C&C 以及 Exploit.CVE-2026-109619.HTTP.C&C。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/5DBHibIELyXCiavIibiaLEHJeI6ZIfcSlfVWUThx8GbIFFyXaHhVePoODbhiakTduXhkduwqaNiaMYM1q2pYVOoHk0kBM0tZ1CEIJ3w2wrzJylF4Q/640?wx_fmt=png&from=appmsg "")  
  
**由KATA检测到PHP2Shell漏洞利用行为**  
  
  
卡巴斯基终端安全（现以卡巴斯基Next形式提供）还可通过行为检测组件检测恶意的利用后行为，例如PDM:Exploit.Linux.Generic以及PDM:Exploit.Win32.Generic。  
  
我们的卡巴斯基漏洞管理解决方案依靠漏洞扫描组件识别存在漏洞软件版本。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/5DBHibIELyXCNk9JlO3mWME6mutR2gcjrNibJCI2vUTAv8ibic1LmsvG0gFbPlaLhMYl1bPPwibYtvmWDEjv0DAB2SNjXFpIncnlCdH718mPbpjo/640?wx_fmt=png&from=appmsg "")  
  
**卡巴斯基漏洞管理中PHP漏洞的检测**  
  
  
卡巴斯基托管检测与响应（MDR）服务通过监控远程代码执行（RCE）的典型行为，检测针对上述漏洞的利用行为：  
  
• 由网页服务器生成可疑进程；  
  
• 在Web应用目录中创建可疑可执行文件；  
  
• 在Web应用目录中创建可疑脚本；  
  
• 在Web应用目录中创建带有少见后缀的文件；  
  
• 以网页服务器身份执行侦察命令；  
  
• 以网页服务器身份执行持久化驻留命令；  
  
• 在网页服务器上投放并运行Web木马。  
  
此外，托管检测与响应服务还可检测其他和远程代码执行无直接关联的恶意行为。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/5DBHibIELyXDiaeOsqsibz9LmrlwZ7AbepnbUCv8rAWof34StjoHUpIO93vdYE6aZKDhL8micFt5huGxNh4jM3UPBnt4EtgvvRSicjMo3LiaXaJn8/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5DBHibIELyXDJwCFPQFEAibwYC43R3emQ3AXwIuFojqmaVVfcApEwnUxFiaH3Vm1enxADLUJ6jozNeTr05iataS1tAXAnARGknzCk6WT9qFsfL4/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
