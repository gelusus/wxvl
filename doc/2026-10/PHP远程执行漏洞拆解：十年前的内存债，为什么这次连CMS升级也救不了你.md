#  PHP远程执行漏洞拆解：十年前的内存债，为什么这次连CMS升级也救不了你  
原创 davluman
                    davluman  鹭信网安攻防实验室   2026-10-22 10:23  
  
> 从 PHP 5.4 到 8.5.11，十几年间几乎所有版本都在影响范围内；CVSS v4.0 9.2，补丁公开之前就已经在野利用。  
  
  
PHP2Shell 是 PHP 引擎层的未授权 RCE 加进程内存泄露组合漏洞，它证明了一件事：应用代码是干净的，不再等于服务器是安全的  
。  
  
官方安全公告：https://www.php.net/archive/2026.php#2026-10-21-5  
  
CNCERT 风险提示：https://www.cert.org.cn/  
# 前言  
  
2026 年 10 月 21 日 14:00 UTC，PHP 官方发布了 8.5.12、8.4.27、8.3.36、8.2.35 四个版本。这个时间比原计划提前了一天，原因是漏洞细节在 10 月 20 日已经泄露。  
  
比"提前发布"更值得记住的是另一件事：Cloudflare 在当天 07:43 UTC 就观测到了攻击尝试，  
比补丁公开早了六个多小时  
。直接起因是 Chainguard 在 07:41 UTC 单方面破坏协同披露禁运，发布了内含修复的镜像——任何人逆向那个镜像，都能在补丁公开前两分钟定位到漏洞。奇安信 CERT 随后确认已发现在野利用，荷兰的 Tweakers、众议院、NCSC 官网在 21 日选择了预防性下线。  
  
这次的代号叫 PHP2Shell，对应 CVE-2026-109618 和 CVE-2026-109619，两枚漏洞的 CVSS v4.0 评分都是   
9.2（Critical）  
。  
  
很多团队的第一反应是"我 WordPress 是最新版"“我框架一直在跟”。这次这两件事  
都不起作用  
。  
# 这次为什么不一样：攻击面在应用代码之下  
  
先说结论：漏洞在 PHP 引擎里，不在你写的任何一行业务代码、任何一个 CMS、任何一个框架里。  
  
Wiz Research 在技术分析里给出了一个很重的判断：即使是跑在受影响版本上的一个纯 “Hello World” 应用，也依然可被利用。上游给出的受影响范围是   
PHP 5.4.0 至 8.5.11  
，也就是 2012 年以来发布的几乎所有版本。  
  
第二件事更麻烦：  
攻击发生在应用代码执行之前  
。PHP 在请求启动阶段就要解析 RFC1867 multipart 请求体，这时框架中间件、登录校验、应用层安全插件都还没加载。你在应用层写多少鉴权检查都没用，攻击者根本没走到那一步。  
  
已公开的验证结果里，被打通的目标包括 WordPress、Drupal、Joomla、Magento、MediaWiki、phpBB、Laravel、Symfony、WHMCS、phpMyAdmin、Roundcube、Nextcloud，以及最朴素的 Apache mod_php 裸环境。  
  
这个清单看起来像"又一个 CMS 漏洞通报"，其实正好相反：它说明被打穿的不是这些应用，而是它们共同踩着的那一层地板。  
# 拆根因：三次"没来得及清理"  
  
两枚 CVE 底下其实是四个 bug，它们共享同一个模式：一次致命错误（典型是内存耗尽 OOM）打断了 PHP 的原生执行流程，让清理逻辑没跑完；worker 进程活了下来，却带着指向已释放内存的悬空状态；下一个请求又复用了这块内存。  
  
打断机制值得记住：memory_limit  
 触顶后，PHP 走 zend_mm_safe_error()  
 → zend_error_noreturn()  
 → zend_bailout()  
，而 zend_bailout()  
 本质是一次 longjmp  
，直接从任意深的 C 调用栈跳回请求顶层——中间那些"构造函数没跑完、链表没挂上、全局指针没复位"的步骤，全部被跳过。  
  
这是典型的   
UAF（use-after-free，释放后重用）  
。  
## CVE-2026-109618：上传进度反序列化器  
  
PHP 的 Session 模块需要在 multipart 解析阶段解码 session 数据，而这时正常的模块初始化还没完成。解码器会把请求级的记账数据存进 BG(unserialize)  
 这个全局槽位。  
  
如果此时发生内存分配失败，请求堆被销毁，但反序列化器的全局状态仍然"发布在外"。下一个落到同一个 worker 上的请求，就会把攻击者回收重写过的内存当成活着的记账数据来用。  
  
这一步已经把"能让进程崩溃"升级成了"能操纵内存"。  
## 顺带的内存泄露：PHPSESSID 响应 Cookie  
  
同一个 CVE 底下还挂着一条独立的泄露路径，危害被很多人低估了。  
  
早启动阶段，upload-progress 处理会把 session id 发布到 PS(id)  
。OOM 中止请求后请求堆被销毁，PS(id)  
 可能仍指向那块已释放的分配。之后同一个 worker 处理下一个 multipart 请求时，PHP 会把这条陈旧指针当成活着的字符串，把回收后的堆内存   
URL 编码进 PHPSESSID  
 响应 Cookie 回吐给攻击者  
。  
  
关键点有三个：  
  
  
• 泄露的是 worker 进程内存，里面可能有  
其他请求  
的口令、session token、API key，甚至 TLS 私钥，性质非常接近 Heartbleed。  
  
• 应用即使从不调用 session_start()  
 也受影响，因为这条路径在应用初始化之前就跑完了。  
  
• 泄露不需要 RCE 成功，  
单独存在  
。也就是说"我没被打穿"并不代表"我没被读过内存"。  
  
## CVE-2026-109619：致命错误回溯  
  
这一枚的主路径更隐蔽。PHP 会把最近一次致命错误的回溯留在 EG(last_fatal_error_backtrace)  
，OOM 销毁请求堆时没有 undefine 这个 executor 全局；下一次请求如果在 executor 重新初始化之前发出诊断信息，就会顺着悬空指针走。  
  
同一个 CVE 底下还有一条 OPcache 跨 worker 的类型混淆，原理同类——发布共享状态与推进进程内状态之间存在时间窗，已经在 WordPress 上做成完整 RCE 链，本文不展开。  
## 为什么 mod_php 比 PHP-FPM 更急  
  
官方在 Apache mod_php  
 上验证了完整的 HTTP 远程代码执行，用的是默认的输入与内存限制，以及常见的多工作进程 prefork 配置。CVE-2026-109619 的 RCE 也在 mod_php  
 和 FPM 上分别验证过，FPM 路径依赖具体应用行为。  
  
简单说：  
跑 mod_php 的优先处理，别等排期  
。  
# 第一步：先确认"真正在处理请求的那个 PHP"  
  
这是最常见也最致命的失误：在跳板机上敲一下 php -v  
，看到是 8.3.x 就觉得没事，而真正对外服务的是另一个 FPM 池、另一个容器镜像，或者宿主机上的 Apache 模块。  
  
CLI 的版本和配置，证明不了任何服务端环境的事。  
```
# 1) 列出所有 PHP 可执行文件与版本（不要只敲一次 php -v）for p in $(ls /usr/bin/php* /usr/local/bin/php* /opt/*/bin/php 2>/dev/null); do  printf '%-40s ' "$p"; "$p" -v 2>/dev/null | head -1done# 2) 核对 CLI 读的是哪份配置（Web 侧另算）php --ini# 3) Debian/Ubuntu：直接查已安装包版本dpkg -l | grep -E '^ii\s+php' | awk '{print $2, $3}'# 4) 发行版回溯补丁：不相信版本号，直接搜 CVE 号zcat /usr/share/doc/php*/changelog.Debian.gz 2>/dev/null | grep -i 'CVE-2026-1096'rpm -q --changelog php-fpm 2>/dev/null | grep -i 'CVE-2026-1096'# 5) 容器环境：镜像里的 PHP 与宿主机完全无关，逐个查docker ps --format '{{.Names}}\t{{.Image}}' | while read -r n i; do  v=$(docker exec "$n" php -r 'echo PHP_VERSION;' 2>/dev/null)  echo "$n  $i  PHP=${v:-unknown}"done
```  
  
三个必须补上的盲区：  
  
  
•   
面板与一体机  
：cPanel、Plesk、宝塔类面板、网络设备管理后台、Windows 上的 PHP 环境，都自带一份 PHP。  
  
•   
共享主机  
：你控制不了 PHP 版本，直接问服务商是否已打补丁，别自己猜。  
  
•   
Remi 仓库（rpms.remirepo.net  
）  
：RHEL 系很常用，但 PHP Foundation 明确说过这个志愿者仓库在 10 月 21 日当天不提供更新。用它供包的，当天要么换发行版包，要么自己编译。  
  
  
目标版本对照：  
<table><thead><tr><th style="background: rgb(30, 41, 59);color: white;padding: 10px 12px;text-align: left;"><section><span leaf="">分支</span></section></th><th style="background: rgb(30, 41, 59);color: white;padding: 10px 12px;text-align: left;"><section><span leaf="">修复版本</span></section></th><th style="background: rgb(30, 41, 59);color: white;padding: 10px 12px;text-align: left;"><section><span leaf="">处置优先级</span></section></th></tr></thead><tbody><tr><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">PHP 8.5</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">8.5.12</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">高</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">PHP 8.4</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">8.4.27</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">高</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">PHP 8.3</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">8.3.36</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">高</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">PHP 8.2</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">8.2.35</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">最高（2026-12-31 即 EOL，迁移要排进同一张计划表）</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">PHP 8.1 及更早</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">无官方补丁</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid #e2e8f0;"><section><span leaf="">迁移或隔离；PHP 团队另给了 7.4 / 8.0 / 8.1 的最小源码回移补丁</span></section></td></tr></tbody></table># 第二步：升级之后，别忘替换 worker  
  
升级 PHP 只是装上了新二进制。旧工作进程还在内存里跑着旧代码，谁在服务请求就是谁说了算。  
  
顺序是：装包，然后  
重启或完整滚动替换所有 PHP 工作进程  
，再在实际 Web 环境确认版本与配置，最后查关键业务路径和错误日志。容器里重启一个旧镜像等于什么都没做，必须用打过补丁的基础镜像重建再滚动替换。验证用的临时页用完即删，别把 phpinfo()  
 长期挂在公网。  
  
还有一个容易被忘掉的动作：  
如果这台机器暴露过生产凭据，而它又在补丁窗口期被扫过，请把泄露路径当成已经发生过  
。 内存泄露不要求 RCE 成功，所以轮换 API key、数据库口令、TLS 私钥的优先级不比打补丁低。  
  
（凭据轮换这条是本文的判断，不在官方公告清单里——官方谈的是漏洞面，不是事故响应。但内存泄露的真实暴露面摆在那儿，这一步省不得。）  
# 第三步：升不了级时的应急配置  
  
官方口径很清楚：  
没有任何配置项能完整替代补丁  
。补丁本身的做法是：OOM 改为延迟到安全点处理、触及硬阈值直接退出进程，session 全局在 multipart 起始事件就复位成 NULL  
——等于把整条脆弱路径拆掉了，配置做不到这件事。所以下面这些只是争取时间的临时措施，只能阻断部分已知路径。  
  
下面每条我都标了出处：  
官方公告  
的按公告原话写，  
安全厂商  
的建议单独标注，属于  
本文判断  
的也明说——免得你照着执行时把优先级搞错。  
```
  /etc/php/8.3/fpm/php.ini 或 conf.d 下的 drop-in  关键：必须在请求解析之前生效，改完重启所有 worker  阻断已确认的早期会话相关路径（所有受影响分支）session.upload_progress.enabled = 0  仅 PHP 8.5：阻断已确认的致命错误回溯路径fatal_error_backtraces = 0  降低攻击者指定任意会话 ID 的风险  注意：这一条是安全厂商建议，不在官方公告的缓解清单里session.use_strict_mode = 1  官方建议：移除攻击者可控的 PHP 层分配失败点  官方原话：底层分配失败仍可能到达脆弱路径 —— 所以必须同时配容器/cgroup 边界  代价是失去内存上限，还要确认没有池/目录/应用配置把有限值改回来memory_limit = -1  外部资源边界（不上传业务用 8M，需要上传时官方建议 32M）post_max_size = 8Mupload_max_filesize = 2M
```  
  
FPM 读的就是同一份 php.ini  
（除非启动时用 -c  
 指定了别的路径）。但  
池配置里的 php_admin_value  
 / php_admin_flag  
 优先级高于 php.ini，会把它覆盖掉  
——所以光改 php.ini 不够，每个池都要单独核对：  
```
; /etc/php/8.3/fpm/pool.d/www.confphp_admin_flag[session.upload_progress.enabled] = offphp_admin_value[memory_limit] = -1php_admin_value[post_max_size] = 8M
```  
  
改完按官方要求  
重启每一个 worker  
（systemctl restart php8.3-fpm  
，版本号换成你自己的），然后  
在实际由 FPM 服务的请求里  
确认配置生效，不要在命令行自证。  
  
如果只想先切断入口流量，可以在 Web 层拦掉 multipart，因为目前已知的触发路径几乎都是 multipart/form-data  
：  
```
# 仅作应急，会打断正常文件上传；打上补丁后请移除if ($http_content_type ~* "multipart/form-data") {    return 403;}# 更激进：只放行 GET/HEAD，代价是用户无法登录和下单，二选一即可# if ($request_method !~ ^(GET|HEAD)$) {#     return 503;# }
```  
  
放完规则记得 nginx -t && systemctl reload nginx  
，并  
下一笔真实订单  
确认业务没被打断。  
  
官方还列了两条我在很多二次传播的通报里没见到。  
  
一是  
修掉能被攻击者触发的致命错误和执行超时  
——包括回调函数、上传/导入处理、清理阶段的失败。它们都会让请求走异常退出路径，而异常退出正是这两个 CVE 的触发前提。  
  
二是把限制做在   
web server 或代理层  
，而不只是 PHP 层：解码后的请求体大小、输入字段数、multipart 段数、请求头数量、嵌套深度。官方特别强调，  
这些限制必须覆盖没有可用 Content-Length  
 的请求  
（也就是 chunked 传输），否则等于没设。  
  
还有一条不在官方清单里、但工程上最省事的缓解：把 FPM 的 pm.max_requests  
 调成 1，让每个 worker 处理完一个请求就退出。这直接打掉"两次请求命中同一个 worker"这个前提。代价是性能，而且  
它只降低单次尝试的成功率，不是修复  
，打完补丁记得改回来。  
# 四个必须纠正的错误处置动作  
  
这几件事在网上被反复传播，但官方逐条说过无效。  
  
  
•   
关闭错误显示没用  
。 那条配置路径的问题发生在 PHP 决定"是否报告、显示或记录这条诊断"之前。display_errors=Off  
、降低 error_reporting  
、关 log_errors  
、用 @  
 抑制，全部无法修复。  
  
•   
用 .user.ini  
 关会话路径没用  
。 .user.ini  
 在正常初始化之后才应用，而漏洞路径在初始化之前。必须走 php.ini  
、FPM 池配置或等价的服务器启动配置。  
  
•   
用 RLIMIT_AS  
 / RLIMIT_DATA  
 兜底反而更糟  
。 它们可能直接制造底层内存分配失败，正好重新踩回那条有问题的处理路径。要限资源就用容器或 cgroup，并尽量让超限只终止受影响的那个 worker。  
  
•   
升级 CMS / 框架 / 插件无效  
。 WordPress 官方已明确说明：这次  
不需要、也不会  
发布 WordPress 自身的更新，需要更新的是 PHP。装应用层安全插件同理，攻击在插件运行之前就结束了。  
  
# 检测与狩猎：日志里可能什么都没有  
  
这一点要提前跟团队讲清楚：漏洞在应用下方执行，  
应用日志缺少上下文是正常的  
，不是"没事"。  
  
真正值得看的信号是这些：  
  
  
• Fatal error: Allowed memory size of ... exhausted in Unknown on line 0  
。重点是 Unknown on line 0  
：它说明错误发生在用户脚本开始之前，是这类攻击最硬的指纹，比"内存耗尽"本身精确得多。  
  
• zend_mm_heap corrupted  
，或 PHP worker 频繁收到 SIGABRT / SIGSEGV。  
  
• 异常 multipart 请求之后紧跟上述错误，或同一来源连续 HTTP 500。  
  
•   
响应  
里出现畸形、异常大的 Set-Cookie: PHPSESSID=  
，这是内存泄露回吐的直接痕迹。  
  
• Failed to read session data  
、非法 session ID、session 文件读取异常。  
  
• PHP worker 派生了子进程、shell、脚本解释器，或 curl  
 / wget  
 这类下载工具。  
  
• 应用运行正常，但服务器上出现无法解释的新增文件写入。  
  
  
这里有一条判读纪律：  
看到 worker 崩溃就宣布 RCE，是这类分析里最常见的误判  
，崩溃只能证明状态被搅乱，证明不了命令执行。但反过来同样成立：  
没看到崩溃不代表没中招  
，因为内存泄露那条链不需要 RCE 成功。两个方向都不能关掉排查。  
  
巡检脚本可以先跑起来：  
```
# 1) worker 是否在派生奇怪子进程（高危窗口期高频采样）for i in $(seq 1 30); do  ps -eo pid,ppid,comm,args --sort=ppid \    | awk '$3 ~ /^(sh|bash|dash|perl|python3?|curl|wget|nc|ncat)$/ {print}'  sleep 2done | sort -u# 2) worker 重启次数 / OOM 是否异常增长systemctl show php8.3-fpm -p NRestartsdmesg -T | grep -iE 'oom|killed process' | tail -50# 3) Web 日志里 5xx 的聚集情况（按你的日志格式调整）grep -E '" 5[0-9]{2} ' /var/log/nginx/access.log | tail -100# 4) 是否存在异常大的 PHPSESSID 响应（需打开响应头日志）grep -i 'set-cookie: PHPSESSID' /var/log/nginx/error.log \  | awk '{print length($0)}' | sort -rn | head -20
```  
  
如果你们用云 WAF，三家的状态并不一样：  
Cloudflare 已经在托管规则集里默认启用了 PHP2Shell 防护，免费版也含，但付费用户要确认托管规则集是拦截模式而不是仅记录  
；  
AWS 和谷歌云给的是可直接套用的规则模板与配置指引  
（AWS WAF 自定义规则、Cloud Armor 自建规则），不是内置开关。判定核心都是"POST 加带 PHPSESSID  
 的 Cookie 或参数，再加带 PHP_SESSION_UPLOAD_PROGRESS  
 字段"这个组合。自建规则  
务必先跑观察模式  
：PHP 应用对 multipart 和 query 参数的结构差异极大，直接上线拦很可能把正常上传和下单一起拦掉。  
  
还要提醒一句：  
WAF 只能降低命中率，不能作为修复  
。Cloudflare 已经观察到攻击者在变形 payload 绕过规则。这次攻击本身就是用大 payload 制造 OOM，所以要同时确保超过检查长度的请求体不会不经检查就到达后端。  
# 结语：真正的分水岭  
  
技术上看，PHP2Shell 就是两枚 CVSS 9.2 的内存安全漏洞，范围广、优先级高，处置路径清晰。  
  
真正值得记住的是它的来龙去脉。这批代码从 PHP 5.4 就存在，但"一直没人知道"并不准确：2024 年 Quarkslab 做 PHP 核心安全审计时就碰到过这个 OOM 问题的迹象，当时也理论性地认为可能可利用，只是没人找到具体向量。  
问题不是没被发现，而是两年里没有人能把它证明成可利用  
。  
  
Wiz Research 的结论是：靠 Wiz Atlas、Gemini 4 Argon、GPT-5.6 Sol，一个研究员在一周内完成了过去需要世界级团队数月才能完成的事。也就是说，  
"这条链太复杂，应该没人能稳定利用"这个假设，从此不能再作为一台 PHP 5.6 机器继续跑下去的理由  
。  
  
三条现在就能做的动作：  
  
  
•   
今天之内  
盘出所有对外提供 HTTP 服务的 PHP 实例，输出"实例 → SAPI → 版本 → 包来源 → 镜像 → 负责人"表，含面板、设备、容器和共享主机。  
  
•   
24 小时内  
按 mod_php → FPM → 容器的顺序升级并替换 worker；升不了级的用上面的应急配置兜底，并把"临时状态"写进台账、附上截止日期。  
  
•   
本周之内  
把 PHP 的 EOL 日期加进资产台账字段，设定提前 6 个月触发的迁移提醒。PHP 8.2 在 2026-12-31 就到点。  
> 本文只讨论漏洞机理与防御处置，不提供利用代码。所有缓解措施以 PHP 官方安全公告与 CNCERT 风险提示为准。  
  
<table><tbody><tr><td style="background:#041536;padding:32px 34px 24px;color:#fff;font-family:Microsoft YaHei,sans-serif;"><p style="text-align:center;font-size:34px;color:#46c8ff;font-weight:700;letter-spacing:1px;margin-bottom:30px;"><span leaf="">鹭信网安攻防实验室</span></p><p style="text-indent:2em;margin-bottom:22px;"><span leaf="">鹭信网安攻防实验室正式成立于2025年9月，是厦门信网安检测旗下以互联网攻防技术研究为目标的安全团队，&#34;鹭&#34; 取自厦门鹭岛别称，彰显本土属地底色；护航八闽福地，后跟信网安字号，一脉相承品牌体系；</span></p><p style="text-indent:2em;margin-bottom:22px;"><span leaf="">团队主要负责出战国内攻防赛事，福建省内各攻防演习，曾获国家级赛事网鼎杯实网演练三等奖，陇剑杯以及福建省内各攻防演练优异成绩，作为厦门信网安检测技术有限公司的公司整体攻防能力展现，研究方向覆盖网络攻防、系统安全、Web安全、物联网/工控安全/AI安全等多个领域。</span></p><p><span leaf="">厦门信网安检测技术有限公司 · 攻防技术团队</span></p></td></tr></tbody></table>  
