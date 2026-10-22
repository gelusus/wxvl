#  紧急预警！PHP最新超危漏洞可绕过所有上层应用防护，请立即排查！  
 金盾检测股份   2026-10-22 09:36  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/rgCG6icPVe41wY02Jskfp7Vtm7YxrRxw2OVicaUSZ05qIM1gVuCNr5cWGct96M0GrB2zRhnyq3Qd3kDCaa2eybpaxyRux2HYftY3AlbtRQfmk/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**一分钟结论**  
  
只要你的服务器**对外提供Web服务且运行PHP**  
，版本  
低于  
**8.5.12 / 8.4.27 / 8.3.36 / 8.2.35**  
，今天就应视为**已被暴露在攻击面**  
。攻击者  
不需要账号、不需要登录、不需要任何交互  
，一个HTTP请求即可在PHP进程权限下执行任意代码。  
  
正确处置只有三步**：查版本→升级→重启所有PHP进程**  
。缺最后一步，补丁等于没打。  
  
  
**一、**  
  
**漏洞速览**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/rgCG6icPVe41LmCHviaQvic0yia949yM8uPVSwRkBRp1ZicEqhVoZCZsgO1sN86NWufPcST5Rke3tK7lea53lbZE8aDBU8J70FpuprmqFLlqDMia0/640?wx_fmt=gif&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe43Eo5xymAHI41XrxkBRwF9dvUia11nWy1U5lWc0Zaafem4iaUE70hQicKaFR9iaJ3eW2dRvhsibmibsso3TQxH4bnuicoON3GWuNwXh7Q/640?wx_fmt=png&from=appmsg "")  
  
**关键认知：**  
这不是WordPress、Discuz、ThinkPHP 或某个插件的问题，而是**PHP语言解释器自身**  
的问题。你把CMS升到最新版、把所有插件删干净、把代码审计做到零漏洞，  
只要底下跑的PHP是旧版本，风险依然存在  
——甚至在应用代码开始执行之前，攻击就已经完成了。  
  
  
**二、**  
  
**影响范围**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/rgCG6icPVe41f4icUITnQ7wLYKdpMGKiavseSuWwUYCjosMrbUPriaiahSeC4A6av9gRticfdibz3Re5lyWNjZNB5NWfktp5uXL1kO0TiaL9XFxicJWg/640?wx_fmt=gif&from=appmsg "")  
  
  
**受影响版本：**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rgCG6icPVe43HuEia8hUvTUauGfo4jIq8QAKaxFMR4yv5JhCMILcNTIqibVjWRsr6fvpmWUGOkUjYm69SAeWj1WI1ov5SVX6dsyic0U4dyBCbKI/640?wx_fmt=png&from=appmsg "")  
  
  
官方明确指出：**所有处理 Web 请求的 PHP 部署方式均受影响**  
，包括PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed。  
  
其中**Apache + mod_php**  
风险最高——已演示的攻击链可在应用代码执行之前触发；FrankenPHP机制类似mod_php；LiteSpeed 的LSAPI行为类似FPM。  
  
Wiz的研究进一步表明，**近十余年的PHP版本（可追溯至PHP5.4）至少命中其中一个缺陷**  
。而据W3Techs 10月21日数据，在可识别服务端语言的网站中约  
69.3%使用PHP  
。  
  
**易被忽略的暴露面：**  
除公网门户网站外，请一并排查——内网管理系统、运维后台、OA/门户/教务/医务HIS前端、报表系统、监控面板（如 Zabbix 前端）、堡垒机外挂页面、NAS/打印设备/各类一体化设备内置的PHP、宝塔/小皮等面板所管理的站点，以及镜像里“顺手”装上的PHP。很多单位不知道自己某个业务系统的底座是PHP。  
  
  
**三、**  
  
漏洞原理  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe40Bs0FeNGoWZQQ5jUY86LoYOqXWlfwiauprRib9VYjUSxuup1LJnHQpxmMHYOUkZ3bPBdhiaVm0W7JzSlEbPl9ukKx06ls4dmc9Zs/640?wx_fmt=gif&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe43Ox1YyvxO8am0GUhUrRtsMX8KTJPBheEhXnmLJY1HkPopNFCY9TJdlaY7ib0mKo6vBaERXBQJRj5EjNeP4zqOBc2gialPvR5dto/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe41CH90uIhNop8epxNdpwMgePBGLVic0tTOh0CCyxfLzbJGsp229qf1sLfRtQmuHicfVqQr6hcDuqA5YgdBlamQiblcia1QwhMDBCMY/640?wx_fmt=png&from=appmsg "")  
  
**为什么难修：**  
漏洞位于PHP几乎每个请求都会经过的内存管理与错误处理路径上。修复不能只堵某一条已知利用链，必须从底层消除缺陷，同时不能破坏现有应用兼容性、不能引入新的拒绝服务风险，还要协调大量第三方扩展同步修改。PHP基金会坦承，2024年Quarkslab外部审计期间就已发现相关隐患，但因“无法证明可远程利用”、且熟悉内核的开发者有限，修复工作多次停滞，最终由前沿AI模型在2026年夏季补上了从“**存在缺陷”到“可实际利用”的关键一环**  
。  
  
  
**四、**  
  
**实际危害**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe43B2rnTAHZhaLX9YCqxmMACeDqV7uiccF0IqgX80gtAfaYbNhMC4kz3nz2ynt4AIrr6r1ibH2vIzMQSj01UgnujiabHswFqib658wA/640?wx_fmt=gif&from=appmsg "")  
  
  
**01****未授权接管服务器：**  
以 Web 进程权限（通常为 www-data / apache / nginx）执行任意命令，无需任何凭据。  
  
**02**  
 **拖库与凭据外泄：**  
读取站点配置文件拿到数据库账号口令，进而导出业务数据；或直接从进程内存捞取密钥、令牌、支付证书。  
  
**03****页面被挂马/篡改：**  
批量植入暗链、博彩跳转、钓鱼页面，直接损害单位对外形象与公信力。  
  
**04****沦为内网跳板：**  
以DMZ区Web服务器为支点向内网横向，这是护网/重保演习中最常见的失分路径。  
  
**05****合规风险：**  
一旦因未及时修补已知高危漏洞导致数据泄露，在《网络安全法》《数据安全法》及等保测评中均属明确的管理缺失项。  
  
  
**五、**  
  
**从合规角度看**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/rgCG6icPVe40eAB1ib6qY0HESKSCj7ePHBciaXshNnfEicQLDEVQrHN1ffZccnmdsJzNTyYqEz5tb2LsBTvicoOERk7GbRurEO9dkUEThXn4ibs2E/640?wx_fmt=gif&from=appmsg "")  
  
  
等保2.0的  
“入侵防范”与“漏洞和风险管理”  
条款，要求对已知高危漏洞及时修补并留存记录。本次漏洞的特殊之处在于：它不在业务系统里，而在平台层——这意味着很多单位在做资产梳理时根本不会把它列进台账，**测评时也最容易被抽查到“未修补的高危 CVE”**  
。  
  
**借这次机会把流程补上：**  
  
**①补齐中间件与运行时台账：**  
PHP / Tomcat / Nginx / Redis 等平台组件的版本、部署位置、责任人，与业务系统绑定。  
  
**②建立漏洞响应SLA：**  
严重级 24 小时内处置、高危 72 小时内处置，处置动作（升级 / 缓解 / 接受风险）书面留痕。  
  
**③把订阅官方安全公告变成日常动作：**  
PHP、厂商LTS、发行版安全公告、云厂商镜像公告，四条线都要有人盯。  
  
**④护网/重保期间**  
，此类“全网通杀型”漏洞是首要攻击面，建议在值守预案中预设  
应急下线与流量封禁  
的授权流程。  
  
  
六、  
  
**第一步：10分钟自查**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/rgCG6icPVe40E3wJIqfh5Jib2AFdCDGTqvdXm5ics4TDhvluP5kibqXVp2ShvqSS9hQia2UEicasDTFnDicibAsLkLVmfVjiaiahvpVLhbS5adx6k0gmk/640?wx_fmt=gif&from=appmsg "")  
  
  
**1. 单机查看版本与运行方式**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe42iaoIHMm4ficd5BldBA2EW1xCN4LDUe1Bu5scgaUWLVbDVowonPib6IpsPCoF9lwwqJlQSunW2PEmv6ATzibgQk2XExFmYpyAUT2U/640?wx_fmt=png&from=appmsg "")  
  
重点看  
Server API  
：出现  
Apache 2.0 Handler  
即 mod_php，属最高危配置，请优先处置。  
  
  
**2. 批量盘点（Linux）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rgCG6icPVe42Al8hbBzuM5icITyM2WVP52kxZibHFfmGIQ1ue3kq9U7rxlGO7CTibFknAmlusdpcicpeaZvLiaISwxywbGqfTK1wWUvNEyKQgWrzY/640?wx_fmt=png&from=appmsg "")  
  
  
**3. 判断是否需要处置**  
  
版本号  
低于  
 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 中对应分支的 →   
**需要处置**  
。  
**注意：**  
通过apt / yum安装的发行版包，版本号常显示为  
8.1.2-1ubuntu2.14  
之类，看似"没升级"，实际厂商可能已回溯修复。请以**发行版安全公告**  
（Ubuntu USN、Debian DSA、RedHat RHSA、SUSE、阿里云/腾讯云镜像站公告）为准，或直接比对包内 PHP 的完整版本号。  
  
  
**七、**  
  
**第二步：分环境升级**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe437lZkd6yXBrb4blCMXViasn03YO6qVD7gYIiaYibW0xLf9mmic1qElSDjOp6E4dTp94bULvGlicNhBjIy1HDpic4lfRVDxMQaGkyPq0/640?wx_fmt=gif&from=appmsg "")  
  
  
**升级铁律：改完必须重启所有PHP进程。**  
PHP是常驻worker加载引擎的，只替换文件而不重启，等于继续跑着有洞的旧引擎——这是本次处置最容易被漏掉、也最容易导致“打了补丁仍然被打”的一步。  
  
**源码编译安装**  
  
从php.net下载 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 源码，按原编译参数重编（php -i | grep "Configure Command" 可查原参数），然后重启FPM / Web服务。Windows用户在同页下载对应 TS / NTS二进制包替换，重启IIS或Apache服务。  
  
**Debian / Ubuntu**  
  
apt update   
  
apt install --only-upgrade php8.3-fpm php8.3-cli php8.3-common   
  
systemctl restart php8.3-fpm && systemctl restart apache2 nginx  
  
**RHEL / CentOS / Rocky / AlmaLinux**  
  
dnf update 'php*' -y   
  
systemctl restart php-fpm httpd nginx  
  
若使用社区维护的 Remi 仓库，官方提示该仓库在发布当日可能无法同步更新，建议改用发行版官方包或自行编译。  
  
**Docker / 容器化环境**  
  
# Dockerfile:改为已修复版本并重新构建  
  
FROM php:8.3.36-fpm  
  
docker build -t your-app:php-patched .   
  
docker compose up -d --force-recreate  
  
注意：仅重启容器不解决问题，必须重建镜像。K8s环境需滚动更新Pod，并确认无旧 ReplicaSet残留。  
  
**宝塔面板 / 小皮（phpstudy）等集成环境**  
  
宝塔：**软件商店 → 已安装 → 对应 PHP 版本 → 更新（或切换到已修复版本）→ 重载配置。**  
若面板未推送新版，请勿硬等，直接从源码编译对应版本后挂载，或临时把站点迁到已修复的 PHP 版本。  
  
小皮：在版本管理中安装并切换至 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35，然后重启Apache / Nginx。  
  
**PHP 8.1 及更早（已 EOL）**  
  
官方已不再提供更新。三条路：①联系发行版/商业LTS厂商确认是否提供回溯补丁；②使用官方提供的 7.4 / 8.0 / 8.1 最小源码回溯补丁自行编译；③**升级到受支持分支（推荐）**  
。顺带提醒：PHP 8.2将于2026年12月31日停止支持，年底前应把迁移计划一并排上。  
  
**升级后必须验证：**  
重新执行  
 php -v   
与   
php -i | grep "Server API"  
，确认版本号已变；用临时页面  
 <?php echo PHP_VERSION  
; 从外网实际访问一次（验证完立即删除探针）；确认FPM池、CLI、cron任务、队列消费者使用的都是新版本——**很多单位只升了CLI，忘了FPM，或者反过来**  
。  
  
  
**八、**  
  
**第三步（仅当无法立即升级）：临时缓解**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe42njRtPV4AhvaYRGwfpERsezOLzibZgQuOPguZ2r3wwDsRGPbaI4faQQoIIiay9aphkibvsDHUfEuMTTshTBFhHHVgp1bBOnlHmHg/640?wx_fmt=gif&from=appmsg "")  
  
  
**官方原话：不存在能够完全消除风险的临时方案。**  
以下措施只能降低特定攻击路径的暴露面，**不能让系统变安全**  
，也不能替代补丁。WAF同理——只能拦已知特征，拦不住改写的载荷。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe40AN7yic8ePsqX4bPtpIkbjoG5SSgwWNrpiaks9UPT0eExFvEvnVWMUEg6NGN1SBsDicKNvNAROASQLibrzRNRKGH4y5nEgVnicGMCw/640?wx_fmt=png&from=appmsg "")  
  
**Nginx侧限制请求体示例：**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rgCG6icPVe41oibgibOLPr0xMIicNUBfzXJoeRjnUrY8ykvic7m7VicvXXACPAgGcBJibb5Uux1YibCEauGF5Ejo8JKfIJzvWAd4AynKrpYqWck55icc/640?wx_fmt=png&from=appmsg "")  
  
Apache 侧可用 LimitRequestBody 8388608。**注意：**  
memory_limit=-1会让PHP失去自我保护，务必同步配置容器内存上限与worker重启策略，避免真实OOM打满整机形成拒绝服务。  
  
  
**九、**  
  
**第四步：确认是否已被打过**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe41qCT2plpxzEH6pSosB77UZiaS8LC0huQ11xqRcngl83g7cxARmgG1pycc7tWJVPby6UPsBhtCjiauJDlMartaWaoegFhaDysibOQ/640?wx_fmt=gif&from=appmsg "")  
  
  
打补丁**不能**  
修复已经发生的入侵。  
公网可达且未修补的PHP主机，在补丁前就应默认视为"疑似失陷"，先保全证据再处置。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rgCG6icPVe43z6ofT3icZ9RrCcHwCSpyjgrJkcf2icoJPicBz9rSI7S1vdfbRx2Qzu9ficQ49tUWN1PIntFQXeMstMicZUfXuEicAWDjcb2icbxs8Qs/640?wx_fmt=png&from=appmsg "")  
  
需要说明：PHP官方未公布专用IOC。因此**行为特征比特征值更可靠**  
——异常崩溃、OOM报错、陌生子进程这三条组合出现，就应启动应急响应。  
  
  
**十、**  
  
**金盾检测建议**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/rgCG6icPVe42T4tkIp0smicHYkUkQygd8jZu2gQvKSvESIr0hx99SOuEvdOTOXJu48icsVQWiaEjM6uwlcyOzobkWaic4bMsSoL8BzqJbD0xLb3Q/640?wx_fmt=gif&from=appmsg "")  
  
  
**第一，立即排查 PHP 资产，优先处理互联网暴露系统。**  
确认对外提供服务的 PHP 版本是否落在上述受影响区间。对于 Apache mod_php 部署的实例，应作为最高优先级处置，因为其攻击路径最短、利用条件最成熟。  
  
**第二，升级到对应分支的安全版本，不要等待。**  
仅有升级是完整的修复方案。尚未升级的期间，任何缓解措施都只能降低特定攻击路径的暴露面，不能使系统恢复到安全状态。  
  
**第三，若因客观原因无法立即升级，可临时采取官方建议的缓解配置（仅降低风险，不能替代更新）：**  
  
·    
对 CVE-2026-109618：在不可被应用覆盖的层级设置 memory_limit=-1（如 FPM pool 的 php_admin_value 或 Apache 配置）；同时设置较低的 post_max_size（不需要上传的场景设为 8M，需要上传的场景设为 32M），并确保 upload_max_filesize 小于 post_max_size。  
  
·  
对CVE-2026-109619：可考虑关闭 session.upload_progress.enabled。  
  
**第四，完成更新后重启 PHP 进程。**  
状态残留类漏洞（CVE-2026-109619）的修复依赖于worker进程不再携带旧状态，重启是确保修复生效的必要步骤。  
  
  
  
**写在最后**  
  
**底座的风险会穿透所有上层防护，**  
资产梳理必须做到“运行时”这一层。PHP作为中间件/运行时组件，在很多单位的资产台账里根本不存在——但它恰恰是这次漏洞的所在。  
  
对防守方而言，判断标准不应是“有没有公开 EXP”，而应是“这个洞能不能被利用”——而我们永远比攻击者更晚知道答案。  
  
这次事件真正值得记住的是：代码写得再干净、WAF买得再贵、等保过了三级，只要运行时是旧的，攻击者就可以绕过这一切，在应用代码之前完成入侵。  
  
今天请完成三件事：查版本、升级、重启所有PHP进程。  
  
  
参考资料  
  
· PHP 官方安全公告（2026-10-21）：php.net —— Critical remote code execution vulnerabilities, update now  
  
· PHP 安全公告：CVE-2026-109618 / GHSA-w8hw-w9fx-j79w  
  
· PHP 安全公告：CVE-2026-109619 / GHSA-6wmf-4rv5-p3rp  
  
· CERT-EU Security Advisory 2026-018  
  
· 瑞士联邦网络安全局（BACS / NCSC）漏洞预警  
  
· Wiz Research 技术分析：PHP2Shell  
  
· W3Techs 服务端编程语言使用统计（2026-10-21）  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rgCG6icPVe431xDrZKyF9IQE2gpiaIdH8UYrNMNJP2NffUQwRdODfbNLicFibAstdBnNdGdLtDGgCicAAzvPbeeeMK5ds7LQuXKic6OWXibcrf0fvE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rgCG6icPVe41s1je8sVLeBvO7mHuw92uOBWe9FptAUYUrMoDz1um2d49Nb0lxj37eLrrCHd0ibMBWLORibvc0mVn6wfBJWQu4WRBS8eP8OLibnU/640?wx_fmt=png&from=appmsg "")  
  
  
