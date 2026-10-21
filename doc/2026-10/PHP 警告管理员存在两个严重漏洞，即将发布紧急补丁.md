#  PHP 警告管理员存在两个严重漏洞，即将发布紧急补丁  
Z0安全
                    Z0安全  Z0安全   2026-10-21 02:23  
  
   
  
## 内容摘要  
  
PHP 团队将于 2026 年 10 月 21 日 14:00（UTC，北京时间今晚 22:00）发布安全更新，修复两个 CVSS v4.0 评分 9.2 的严重漏洞。泄露信息将它们与 **CVE-2026-109618**  
 和 **CVE-2026-109619**  
 关联，二者被描述为 PHP 核心中的远程代码执行漏洞。补丁发布前，一段名为 PHP2Shell 的演示视频已在 X 平台流出，视频中对 WordPress、Magento、Drupal 等 9 款主流应用实现了命令执行回显（www-data 权限）。管理员应立即完成 PHP 资产盘点，在更新发布的第一时间打补丁。  
## 影响  
  
PHP 支撑着 WordPress、Drupal、Magento 以及绝大多数共享主机，一个核心层漏洞可能波及互联网的极大一部分。Mallory 的统计显示，约 **150 万台**  
对外暴露且会报告 PHP 版本的主机中，**59% 运行 PHP 8.1 或更早的分支**  
——这些分支已经不再获得官方修复，即使补丁发布，它们也等不到修复包。  
  
此前德国安全团队将本次风险与 Log4Shell 做了对比：若漏洞可免认证通过普通 HTTP 请求触发且影响常规部署配置，实际影响面可能超过 Log4Shell——Log4Shell 的战场在 Java 生态的企业内网，而 PHP 的大量部署直接裸露在公网上，其中相当比例是多年无人维护的旧安装。  
## 泄露促成了提前预警  
  
10 月 20 日，一篇匿名的 Hacker News 帖子率先声称存在两个 CVSS 10.0 的 PHP 漏洞。随后 Kevin Beaumont 在 Mastodon 上确认："有人在补丁发布之前泄露了 PHP 中的 CVE-2026-109618 和 CVE-2026-109619。"watchTowr 同日在 X 上表示这些漏洞属实。  
  
同一天，PHP 官方发布安全更新提前通知："鼓励所有 PHP 用户立即做好准备，以便在更新可用时第一时间升级。"通知同时保留提前发布的权利——若出现积极利用的证据，发布时间将提前。官方主动把"提前放货"写进公告，本身就是一个信号。  
## PHP2Shell：补丁未发  
  
预告发出当晚，X 平台流出一段约 3 分钟的利用演示视频 PHP2Shell。操作流程：输入 HTTP 目标地址 + 要执行的命令，直接返回操作系统回显 uid=33(www-data)  
。  
  
演示覆盖的目标：**WordPress、MediaWiki、phpBB、Magento、Joomla、Laravel、Symfony、Drupal、WHMCS**  
。  
  
三个关键技术信号：  
  
<table><thead><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">观察点</span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">结论</span></section></td></tr></thead><tbody><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">Apache mod_php 与 PHP-FPM 双路径均利用成功</span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">漏洞大概率位于解释器/引擎层，与 SAPI 架构无关，&#34;我们是 FPM&#34;不构成豁免</span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">演示横幅声称 </span><code style="text-align: left;line-height: 1.75;font-family: -apple-system-font, BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB, Microsoft YaHei UI, Microsoft YaHei, Arial, sans-serif;font-size: 90%;color: rgb(221, 17, 68);background: rgba(27, 31, 35, 0.05);padding: 3px 5px;border-radius: 4px;"><span leaf="">PHP &gt;= 5.4.0 RCE</span></code><span leaf="">，演示环境为 Ubuntu 26.04 + PHP 8.5.4</span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">版本范围仅为演示方单方面声明，官方未确认；8.5.4 正是被打的环境</span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">回显为 www-data 权限</span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 24px;"><span leaf="">非.root，但已等于整站沦陷：可读数据库凭据、写 WebShell、作为横向移动起点</span></section></td></tr></tbody></table>  
  
注意：PHP2Shell 演示与官方预告的两个 CVE 之间的对应关系，目前没有任何官方渠道确认。  
## 攻击原理  
  
目前尚无公开技术细节。PHP 官方表示"完整细节、受影响版本及修复版本将随更新一同公布"。泄露信息将其描述为远程代码执行漏洞，但 PHP 的通知本身并未指明漏洞类型。  
## 利用状态  
  
尚未确认在野利用或公开 PoC。PHP 保留提前发布更新的权利，"若出现在野积极利用的证据"即会触发。但鉴于 PHP2Shell 演示视频已经流出，技术细节大概率在补丁发布后数小时内被完整还原，攻击者的武器化速度会远快于往年。  
## 受影响版本  
  
PHP 尚未指明受影响的版本。泄露信息声称自 PHP 5 以来的每个版本均受影响，该说法未获证实。不可据此对任何操作系统或 SAPI 给出"安全"结论。  
## 补丁与缓解措施  
  
**发布前立即执行：**  
- • 清点每一个基于 PHP 的应用及其版本——包括 CLI、Cron 任务、常驻队列 Worker、Web 实际加载的 handler，四者版本经常不一致  
  
- • Plesk 用户注意双轨：系统 PHP 与 Plesk 托管 PHP 是两套独立安装，站点与 handler 对应关系用 plesk bin php_handler --get-usage  
 查询  
  
- • 记录每台机器 PHP 的包来源（发行版自带 / Plesk / 三方仓库 / 容器 / 自编译），明确补丁获取路径与回滚方式  
  
- • 为运行 PHP 8.1 或更早版本的服务器制定升级计划——这些分支已 EOL，本次不会有修复包  
  
**发布后数小时内完成：**  
- • 将官方公告的受影响版本、触发前提与实际安装、handler、扩展清单对照核实，确认攻击路径可达的实例  
  
- • 从指定来源获取修复包，确认回滚路径后升级  
  
- • **重启所有相关进程**  
：mod_php 存在于 Apache 子进程、PHP-FPM 为独立 worker 池、CLI 常驻进程自成一体——二进制更新后旧进程内存中仍是老代码，php -v  
 显示新版本号不等于修复完成，需核对进程启动时间或 phpinfo 中的 build 日期  
  
- • 容器环境：重启容器不等于更新，镜像未更换则副本仍运行旧代码，滚动更新须确认所有副本切换至新镜像  
  
**无效缓解，提前排除：**  
- • disable_functions  
：仅拦截 PHP 函数调用层，解释器层漏洞在进入函数表前已取得执行流  
  
- • open_basedir  
：约束的是脚本层文件访问，管不住失控的原生代码  
  
- • 禁用 SSH：RCE 入口在 80/443，拦截的是运维自身  
  
- • WAF 自动防御：新利用手法无现成规则，不可作为依赖项  
  
鉴于技术细节很可能在更新发布后随之公开，各团队应在**数小时内、而非数天内**  
完成修补。补丁发布前若机器已有被入侵迹象，打补丁只是换锁——上传目录后门要清理，数据库凭据、API key、面板密码全部轮换。  
## 参考链接  
- • PHP 官方安全更新预告：https://www.php.net/archive/2026.php[#2026]()  
-10-20-1  
  
- • w  
  
  
  
  
   
  
  
