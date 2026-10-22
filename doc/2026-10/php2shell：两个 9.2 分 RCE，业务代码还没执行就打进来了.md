#  php2shell：两个 9.2 分 RCE，业务代码还没执行就打进来了  
原创 奋斗的小浪
                        奋斗的小浪  船山信安   2026-10-22 03:33  
  
漏洞挖掘 · VULNHUNT  
  
php2shell：两个 9.2 分 RCE，业务代码还没执行就打进来了  
  
10 月 21 日，PHP 官方一次性发了两个 Critical：CVE-2026-109618 和 CVE-2026-109619，CVSS 4.0 都是 9.2，代号 php2shell。  
  
它吓人的地方不是分数，是位置——漏洞在 PHP 核心的 Zend 引擎里，在扩展初始化之前、在任何一行业务代码被编译之前就能触发。你代码写得多干净，跟这事一点关系都没有。  
  
本文为漏洞情报整理与防守加固建议。文中不含可利用的攻击载荷、复现脚本或堆布局方法，所有验证动作均应在自有隔离环境中进行。  
  
01  
先说结论：这是什么级别的洞  
  
两个漏洞，同一个 9.2 分，同一个前提：**无需认证、无需交互、无需会话**  
。确认的影响是完整的远程代码执行，权限就是 web worker 那个账号。  
<table><tbody><tr style="background:#fdf2f3;"><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf=""><br/></span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">CVE-2026-109618</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">CVE-2026-109619</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">官方命名</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">RCE via OOM corruption</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">RCE via stale global state</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">CVSS 4.0</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">9.2 Critical</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">9.2 Critical</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">根因</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">内存耗尽时的提前退出</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">跨请求残留的全局状态</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">已确认</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">mod_php 完整 RCE 已验证</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">mod_php 确认；session 路径 mod_php 与 FPM 均验证</span></section></td></tr></tbody></table>  
时间线很短：10 月 20 日 UTC 约 15:00，两个 CVE 编号开始在论坛和社交流传；10 月 21 日，PHP 官方发版并公开两份公告。从编号流出到补丁落地，不到一天。  
  
02  
它坏在哪：longjmp() 这一个动作  
  
两个洞共用一条根因。PHP 遇到致命条件——  
memory_limit  
 耗尽、执行超时、客户端断连——并不沿调用栈把错误往上抛，而是 raise 一个 fatal，用   
longjmp()  
 直接放弃当前上下文。  
  
问题就在这个「放弃」：一个 C 函数正建到一半的数据结构被就地丢掉，析构不跑，账不回滚，也没有任何地方记录「这个结构没建完」。然后 PHP 照常走清理流程——而清理流程假设每个结构都是完整一致的。  
  
PHP 把一个半截的结构，当成完整的去清理。两个 CVE 描述的就是这个假设在哪两处崩掉。  
  
03  
618：把请求推到 OOM 的那条路  
  
请求内存在一个窗口期里「已经可以访问，但还没初始化完」。这时任何一次分配失败，PHP 都没能记账这次提前退出，请求内存停在无效状态，紧随其后的清理会去读这些字节、甚至按它做分支。  
  
官方列了三种能让请求耗光内存、表面却又没超   
post_max_size  
 的办法：不带可用 Content-Length 的 multipart 请求能绕过初始检查；单个 multipart 段塞不限数量的 header，PHP 内部表示它们用的内存远大于线路体积；空文件控件不计入   
max_file_uploads  
，但元数据数组照样建。  
<table><tbody><tr style="background:#fdf2f3;"><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">部署形态</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">处境</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">说明</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">Apache mod_php</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">最危险</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">无条件调用请求关闭流程，哪怕初始化已失败</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">PHP-FPM</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">源码分析支持受影响</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">启动失败时子进程退出，官方称暂无完整利用演示</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">FrankenPHP</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">很可能受影响</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">与 mod_php 同构</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">LSAPI / LiteSpeed</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">条件性</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">模型接近 FPM</span></section></td></tr></tbody></table>  
已确认：**在默认输入与内存限制、常规多 worker 的 prefork 配置下，mod_php 上的完整 RCE 已通过 HTTP 验证**  
。worker 更多只会增加尝试次数，不会消除条件。  
  
04  
619：跨请求活下来的陈旧状态  
  
这是典型的释放后使用。worker 里的全局状态——错误、会话、对象、扩展——指向某个请求拥有的内存，本该在请求内存释放前被重置。但一次 fatal 会把操作在「执行匹配的重置」之前丢下；或者一条非致命诊断在状态已被清理后又把它重新填上。请求堆释放了，全局字段还指着它。  
  
官方把它描述成**生产者 + 消费者**  
模型，并强调两个请求必须由**同一个 worker**  
处理——多 worker 和回收会降低单次成功率，但消除不了条件。两类已确认的「消费者」：  
  
1  
初始化前的 INI 诊断。一条 warning、notice 或 deprecation 就够，**不需要致命、不需要显示、不需要记进日志**  
。改   
error_reporting  
、加   
@  
、关   
display_errors  
 都挡不住——全局状态在 PHP 决定要不要报告之前就被访问了。  
  
2  
Session upload progress。它能在常规会话初始化之前触达会话处理，拿到上个被中断请求留下的状态。**默认开启，且不需要应用调用 session_start()**  
。  
  
其中 PHP 8.5 的   
fatal_error_backtraces  
 是最强的已确认案例，也是 mod_php 上被确认利用的基础。  
  
05  
为什么特别难防：它站的位置  
  
PHP 解析请求数据的时机在扩展初始化之前、在任何一行应用代码被编译之前。所以利用跟服务器上跑什么完全无关——**维护得一丝不苟的应用，和年久失修的应用，暴露程度一模一样**  
。  
  
而且它非常安静。CrowdStrike 的验证结论：一个匿名 POST 就能让 mod_php 的 worker 拉起 shell 执行指定命令，过程中**worker 不崩溃，服务端返回标准 HTTP 200，应用日志记不下任何异常**  
。  
  
这意味着「看崩溃、看 500、看异常日志」这套常规检测思路，在这个洞上基本是瞎的。  
  
06  
影响范围：谁在里面  
<table><tbody><tr style="background:#fdf2f3;"><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">分支</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">受影响</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">修复版本</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.5.x</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">&lt; 8.5.12</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.5.12</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.4.x</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">&lt; 8.4.27</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.4.27</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.3.x</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">&lt; 8.3.36</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.3.36</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.2.x</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">&lt; 8.2.35</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.2.35</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">8.1 及更早</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">官方 EOL，无官方补丁</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">升级到受支持分支</span></section></td></tr></tbody></table>  
几个必须单独记的数字：PHP 8.1 及更早已停止官方更新（官方为 7.4 / 8.0 / 8.1 提供了最小化源码 backport）；PHP 8.2 安全支持 2026 年 12 月 31 日结束；Cisco Talos 基于开源情报估计，**约 50% 的 PHP 系统不会收到补丁**  
。  
  
Talos 判断这两个洞影响 5.4 以来的所有 PHP 构建。应用层被点名的有 WordPress、Drupal、Magento、Shopware、WooCommerce、Laravel，以及 cPanel、Plesk 托管的站点；部署形态上 PHP-FPM、mod_php、php-cgi、FrankenPHP、LiteSpeed 全在风险面里。  
  
关于在野利用，各家口径目前不一致，如实摆：Talos 截至 10 月 21 日 14:00 UTC 称尚未观测到；加拿大网安中心称已有公开 PoC、暂未发现活跃利用；也有厂商称已观测到利用尝试。**结论只有一个——按已经有人在打来处置。**  
  
07  
第一步：30 秒确认你中没中  
  
最容易踩的坑：**CLI 里的 PHP 版本，不等于 web 侧实际跑的版本**  
。  
php -v  
 看到的可能是另一个二进制。  
  
# 1) CLI 版本只作参考，不能当结论  
  
php -v  
  
   
  
# 2) 看 web 侧实际加载的 PHP  
  
php-fpm8.3 -v  
  
apache2ctl -M | grep php  
  
   
  
# 3) 最可靠：临时 phpinfo，看完立刻删掉  
  
echo '&lt;?php phpinfo(); ?&gt;' &gt; /var/www/html/_v.php  
  
curl -s http://127.0.0.1/_v.php | grep -iE 'PHP Version|Server API|memory_limit|upload_progress'  
  
rm -f /var/www/html/_v.php  
  
看三样：  
PHP Version  
 是否低于修复版本、  
Server API  
 是哪一种（FPM-FastCGI / Apache 2.0 Handler 即 mod_php）、  
memory_limit  
 与   
session.upload_progress.enabled  
 的现值。  
  
别漏存量盲区：容器镜像、内置 PHP 的第三方设备（运维面板、监控设备）、CI 上的固定版本镜像。它们通常不在常规漏扫清单里，却是这次最难清的一块。  
  
08  
第二步：隔离环境里验证补丁真的生效  
  
这一段是**验证修复**  
，不是验证攻击。找一台与生产同版本、同 SAPI 的测试机，全程断外网。  
  
1  
① 记录基线：PHP Version、Server API、  
memory_limit  
、  
session.upload_progress.enabled  
，截图留存。  
  
2  
② 升级到对应修复版本（8.5.12 / 8.4.27 / 8.3.36 / 8.2.35）。  
  
3  
③ 重启。**这是最容易漏的一步**  
——不重启，worker 里跑的还是旧二进制，版本号看着变了其实没换。  
  
4  
④ 回 web 侧再确认一次版本号，确认 Server API 没变、扩展都还在。  
  
5  
⑤ 确认三个配置落地：  
memory_limit  
、  
session.upload_progress.enabled  
、PHP 8.5 上的   
fatal_error_backtraces  
。  
  
6  
⑥ 确认所有 worker 都是新进程，无旧进程残留。  
  
7  
⑦ 跑业务冒烟，重点看   
memory_limit=-1  
 之后的内存增长。  
  
# 升级后确认 worker 全部换新（FPM 为例）  
  
systemctl restart php8.3-fpm apache2  
  
ps -eo pid,lstart,cmd | grep -E 'php-fpm|apache2' | grep -v grep  
  
   
  
# 确认没有旧版本二进制残留  
  
ls -la /usr/sbin/php-fpm* /usr/lib/apache2/modules/libphp* 2>/dev/null  
  
   
  
# 确认配置真生效（看 web 侧，不是 ini 文件）  
  
php-fpm8.3 -i | grep -iE 'memory_limit|upload_progress|fatal_error_backtraces'  
  
官方明确提醒：**只有升级是完整缓解**  
。下面的临时措施，官方原话是「不能使安装变得安全」，只能降低特定攻击路径的暴露。  
  
09  
第三步：暂时升不了，把这几道闸拉上  
<table><tbody><tr style="background:#fdf2f3;"><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">动作</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">针对</span></section></td><td style="padding: 10px 12px;border: 1px solid rgb(246, 217, 220);font-weight: 700;color: rgb(192, 21, 47);"><section><span leaf="">是否完整缓解</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><span style="font-family:monospace;"><span leaf="">memory_limit=-1</span></span><section><span leaf="">（php_admin_value 强制）</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">618</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">否，仅移除可控的 OOM 触发点</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><span style="font-family:monospace;"><span leaf="">post_max_size</span></span><section><span leaf=""> 保守 + 前端限制解码后大小</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">618</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">否</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #f6d9dc;"><span style="font-family:monospace;"><span leaf="">session.upload_progress.enabled=0</span></span><section><span leaf=""> 并重启 worker</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">619</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">否，仅挡已确认的会话路径</span></section></td></tr><tr style="background:#fdf2f3;"><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">PHP 8.5 上 </span><span style="font-family:monospace;"><span leaf="">fatal_error_backtraces=0</span></span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">619</span></section></td><td style="padding:10px 12px;border:1px solid #f6d9dc;"><section><span leaf="">否，仅挡已确认的 backtrace 路径</span></section></td></tr></tbody></table>  
四个细节，做错等于没做：  
  
1  
memory_limit=-1  
 必须用   
php_admin_value  
（FPM pool 或 Apache 配置）写死，并逐个检查 pool、目录、应用有没有把有限值加回来。  
ini_set()  
 和   
.user.ini  
 都不算数。  
  
2  
session.upload_progress.enabled=0  
 必须在请求解析之前生效的配置层写，写完**重启每一个 worker**  
。官方还补了一句大实话：即使打完补丁，不需要这个功能就该关——它很少被使用，也很少被测试。  
  
3**绝对不要用 RLIMIT_AS 或 RLIMIT_DATA**  
 代替内存限制。它们会让   
mmap()  
 失败，正好把请求推进那条有漏洞的 longjmp 路径。  
  
4  
容器 / cgroup 的   
memory.max  
 只能当可用性边界，不是缓解——它可能把 master 和所有 worker 一起杀掉，除非每个 worker 单独成组。  
  
前端也别空着：限制解码后的请求大小、输入字段数、multipart 段数、每段内 header 数、嵌套深度，且**必须覆盖没有可用 Content-Length 的请求**  
——那正是绕开 post_max_size 检查的那条路。  
  
10  
打完之后：怎么查有没有被摸过  
  
先接受一个事实：这个洞**不留崩溃、不留 500、应用日志可能干干净净**  
。排查要换角度。  
  
1  
访问日志里有没有异常 multipart 请求，尤其是紧跟内存限制类错误的那些。  
  
2  
HTTP 500 有没有出现不明原因的尖峰。  
  
3  
有没有异常偏大的   
PHPSESSID  
 cookie。  
  
4  
worker 有没有不明原因的重启。  
  
5  
网络侧：Talos 已发布 SNORT 规则 67359、67360、67375，社区规则集可用。  
  
官方给的审计配置是   
log_errors=On  
、可用的   
error_log  
、  
error_reporting=E_ALL  
，重点查四类：目标脚本运行前重复出现的 warning / deprecation；扩展或服务器清理阶段产生的诊断；fatal、parse、uncaught error；  
Maximum execution time exceeded  
。  
  
最后一句必须写进去：**日志里没有错误，不等于安全**  
。历史日志可能没采集到相关级别，E_ALL 也只覆盖你实际跑到的路由，而且故障有可能发生在 PHP 写下日志之前。  
  
11  
这次真正值得记住的  
  
一是发现方式。PHP 官方公告的致谢名单里，报告方包含 OpenAI 的安全研究人员；据媒体报道，Google 称 Gemini 系列模型参与了漏洞发现、利用开发、检测与补丁验证（媒体转述，未经官方公告确认）。Talos 点了题：AI 正在改变漏洞的披露节奏。  
  
二是补丁节奏。从编号在社交流传到官方发版，不到 24 小时。以前「补丁 bake in 几个月再上线」的做法，在这个速度面前撑不住。  
  
三是传导。Talos 把它和 Log4j 相提并论，不是技术相似，而是**核心组件的洞会顺着生态一层层传导**  
——应用、面板、设备、容器镜像，每层都要等自己的 vendor 出补丁，相当一部分永远等不到。  
  
核心组件的漏洞，考验的从来不是你代码写得好不好，而是你多久能把所有资产盘点清楚。  
  
参考来源  
  
PHP 官方安全公告 GHSA-w8hw-w9fx-j79w — CVE-2026-109618（RCE via OOM corruption）  
  
PHP 官方安全公告 GHSA-6wmf-4rv5-p3rp — CVE-2026-109619（RCE via stale global state）  
  
PHP 官网 2026-10-21 安全公告 — 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
Cisco Talos《Two critical RCEs in PHP core》（2026-10-21，SNORT SID 67359/67360/67375）  
  
加拿大网络安全中心 AL26-026（2026-10-21）  
  
CrowdStrike《Protects Customers from Threats Delivered via PHP2Shell》（2026-10-21）  
  
NetSPI 漏洞通告 — PHP2Shell Unauthenticated RCE（2026-10-21）  
  
腾讯云安全中心 PHP php2shell 远程代码执行漏洞风险通告（2026-10-21）  
  
本文为技术事实陈述与防护建议，全部案例基于授权测试与公开研究；不提供针对未修复系统的完整攻击链，不构成未授权测试的建议。  
  
