#  一个上传请求打穿 PHP 引擎：php2shell 双漏洞拆解  
原创 Qsec
                    Qsec  Qsec   2026-10-21 16:14  
  
# 一个上传请求打穿 PHP 引擎：php2shell 双漏洞拆解  
  
2026 年 10 月 21 日，PHP 基金会发布了一次计划之外的紧急安全更新，一口气修复两个 CVSS v4.0 评分 9.2 的严重级漏洞。两个漏洞都位于 PHP 引擎内核，都能在无需任何认证的情况下实现远程代码执行。按原计划，细节本应在 22 日随官方公告一起发布。10 月 20 日上午，一个声称 PHP 将有重大披露的帖子出现在 Hacker News，随即被删除，但截图已经在社交网络扩散。PHP 基金会当天拍板：发版提前一天。  
  
这两个漏洞很快被安全圈冠以 php2shell 的代号。watchTowr 的蜜罐网络在补丁发布当天就观测到了利用尝试；Cloudflare 在泄露后看到真实攻击流量，攻击者已经开始针对 WAF 规则变异载荷。  
  
这篇文章把两个漏洞讲清楚：成因是什么，利用链怎么走，影响有多大。先说结论：漏洞不在你的代码里，在 PHP 引擎的请求善后逻辑里。只要服务器在用 PHP 响应 Web 请求，你就是潜在目标，修复只能靠升级 PHP 本身。  
  
口径说明：本文为防御视角的机制拆解，不含可操作的攻击细节。利用可行性区分两类口径：官方安全公告确认的，和安全厂商研究验证的，分开标注。  
## 一、地基：PHP 是怎么跑网站的  
  
理解这两个漏洞，先要理解 PHP 处理请求的模型。  
  
Nginx 或 Apache 收到请求后交给 PHP 解释器，主流部署有两种。一种是 php-fpm，独立进程池，每个 worker 进程串行处理成百上千个请求；另一种是 mod_php，解释器直接嵌进 Apache 子进程，同样一个进程服务多个请求。关键在于：worker 进程处理完一个请求后并不销毁，而是留在内存里等下一个请求。这省掉了进程启动的开销，代价是所有请求共享同一块进程内存。  
  
每个请求结束时要走一遍 shutdown：释放这个请求申请的内存，把全局状态复位，然后回到空闲循环。一切顺利时没有问题。但如果一个请求非正常死亡呢？比如执行超时，比如内存耗尽触发 fatal error。致命错误的处理路径和正常退出不同，中断可能发生在引擎内部任意两行代码之间，那些只有不中断才成立的内部约定，在这一刻全部失效。  
  
两个漏洞的舞台就是这里：一个请求死得不清爽，在 worker 进程里留下不该留下的东西，然后被下一个毫无防备的请求踩中。  
  
![PHP worker 复用与跨请求 UAF 的形成过程](https://mmbiz.qpic.cn/mmbiz_png/icVSvicxhfMd46Ss6FGrCWThAicgrucMq6IA7tvPTwS79mGbKZVchZfVC9scUbpFBEehEpZyIvAE3nsGicchHDa3r98dcaYRJibXsPpaGCr7Letk/640?from=appmsg "")  
  
PHP worker 复用与跨请求 UAF 的形成过程  
  
▲ worker 复用与跨请求污染：请求 A 非正常死亡留下的悬垂状态，被同一 worker 的请求 B 在初始化之前踩中（依官方安全公告绘制）  
## 二、CVE-2026-109618：内存耗尽时的「跳伞」  
  
先看第一个漏洞。官方安全公告的描述很直白：PHP 处理内存耗尽的方式是 longjmp，从出错点直接跳到预设的安全位置，放弃当前请求。问题在于，某些内部上下文里，请求内存处于一个窗口期，它已经可以被访问，但还没初始化完成。如果分配失败恰好发生在这个窗口里，longjmp 的提前退出没有被正确记账，这块内存就停留在无效状态。  
  
这不是推测，是能直接读出来的代码。修复前（标签 php-8.4.26  
），内存分配器里的超限处理长这样（Zend/zend_alloc.c  
，节选有删节）：  
```
static ZEND_COLD ZEND_NORETURN voidzend_mm_safe_error(zend_mm_heap *heap,constchar*format,size_t limit,size_t size){	heap->overflow =1;	zend_try {zend_error_noreturn(E_ERROR,			format, limit, size);} zend_catch {}zend_end_try();	heap->overflow =0;zend_bailout();exit(1);}
```  
  
各处分配超限时调用它，报出那句每个 PHP 开发者都见过的 Allowed memory size of %zu bytes exhausted  
。关键在倒数第二行：zend_bailout()  
。这个函数的定义在 Zend/zend.c  
 里，最后一行暴露了本质（原文完整节选）：  
```
ZEND_API ZEND_COLD ZEND_NORETURN void_zend_bailout(constchar*filename,uint32_t lineno){if(!EG(bailout)){zend_output_debug_string(1,"%s(%d) : Bailed out without a bailout address!", filename, lineno);exit(-1);}gc_protect(1);CG(unclean_shutdown)=1;CG(active_class_entry)=NULL;CG(in_compilation)=0;CG(memoize_mode)=0;EG(current_execute_data)=NULL;LONGJMP(*EG(bailout), FAILURE);}
```  
  
LONGJMP  
。一次跨函数的远距离跳转，从内存分配器的深处直接跳回请求入口附近，中间所有正在执行的初始化、清理、状态复位代码全部作废。  
  
更麻烦的是，导致 OOM 的那次分配，和最终被污染的那块内存，可以毫无关系。请求 shutdown 阶段照常运行，把无效内存当有效数据来读、来分支，错误在这里才兑现成实际危害。  
  
攻击者要做的第一件事是制造 OOM。听起来需要发一个巨大的请求把内存打爆，实际上不需要。官方公告列出了四种在限额内撑爆 memory_limit 的 multipart 解析技巧：请求不带可用的 Content-Length，可以绕过 post_max_size 的前置检查；multipart 的单个 part 可以携带几乎不限量的头部，而头部在 PHP 内部的表示形式远大于它在网络上的字节数，一个限额内的请求就能耗尽 memory_limit；空文件控件不计入 max_file_uploads 限制，却照样创建文件元数据数组；嵌套的字段名会成倍放大数组操作。  
  
也就是说，触发漏洞的是一个看起来完全正常的文件上传请求。  
  
不同部署方式的风险不同。官方口径下，mod_php 最危险：Apache 会无条件执行请求清理，哪怕请求初始化就已经失败，攻击发生在任何应用代码运行之前。在默认配置的 mod_php 上，未经认证的远程代码执行已经被确认。php-fpm 的启动失败会直接杀掉子进程，正常启动后理论上仍可达，但官方明确表示没有演示出完整的 FPM 利用链。FrankenPHP 的处境类似 mod_php，大概率受影响；LiteSpeed 的 LSAPI 建模接近 FPM，未经证实。  
  
补丁的修法说明了问题之深。核心修复集中在官方仓库的一个提交（506e87fa7d  
，"Avoid state corruption via bailout"，提交说明明确写着 Fix 这两个 CVE），思路是：彻底废除 OOM 时跳 longjmp 的路，改成「软 OOM」加「硬 OOM」两级机制。  
  
软 OOM 不再当场跳走，只把错误信息挂到执行全局上、把状态标记为 PENDING，然后拉起虚拟机中断信号（Zend/zend_alloc.c  
，修复后，节选有删节）：  
```
static ZEND_COLD voidzend_mm_delayed_verror(constchar*format, va_list args){if(ZEND_HAS_OOM()){return;}/* ... 记录脚本文件名与行号，有删节 ... */EG(delayed_oom).message =zend_string_init(buf,strlen(buf),/* persistent */ true);EG(delayed_oom).filename = script_filename;EG(delayed_oom).lineno = script_lineno;EG(delayed_oom).state = ZEND_OOM_STATE_PENDING;zend_atomic_bool_store_ex(&EG(vm_interrupt), true);}
```  
  
虚拟机只在自己代码的安全点上检查这个中断，让当前操作体面地收尾。那如果内存真的彻底耗尽、连收尾都做不了呢？源码里新加的注释把设计意图写得明明白白：  
```
/* Returns FAILURE if allocation should fail (in case the heap is fallible).  * When the soft limit is to be exceeded, a delayed OOM error is queued.  * When the hard limit is to be exceeded, the function does not return.  */
```  
  
软限制超了就排队等安全点处理；硬限制超了，函数根本不会返回，直接结束进程（修复后 Zend/zend_alloc.c  
，节选有删节）：  
```
static ZEND_COLD ZEND_NORETURN voidzend_mm_hard_limit_error(zend_mm_heap *heap,size_t add_size){/* ... 定位脚本位置，有删节 ... */fprintf(stderr,"Hard memory limit of %zu bytes exhausted (tried to allocate %zu bytes) in %s on line %d\n",			heap->hard_limit, add_size,ZSTR_VAL(script_filename), script_lineno);_exit(137);}
```  
  
_exit(137)  
。宁可让 worker 进程死透，由进程池重新拉一个干净的，也不留半初始化的尸体给下一个请求。  
## 三、CVE-2026-109619：全局状态里的「幽灵指针」  
  
第二个漏洞的成因更隐蔽。worker 进程的全局状态里，有一批指针指向请求期的内存：错误处理状态、会话状态、对象状态、扩展状态。规矩是，这些状态必须在请求内存释放之前复位。致命错误会打断这个流程，操作半途而废，复位没有执行。更刁钻的是，一些非致命的诊断信息还可能在清理完成之后，把错误状态重新填上。于是内存被释放了，全局指针还指着那里。  
  
下一个请求到来时，在正常初始化流程跑完之前，某段代码读了这个悬垂指针。这就是 use-after-free。官方公告的原话是：精心构造的堆布局可以产生完整的远程代码执行。  
  
这个漏洞的利用被官方拆解成生产者和消费者两端。生产者负责制造悬垂指针：解析或编译阶段的致命错误、未捕获的错误、require 失败、执行超时、状态发布之后的内存耗尽。消费者负责在错误的时机读到它，官方确认了两条路径。  
  
第一条是 INI 设置诊断。PHP 在应用配置时会发出警告，比如非法的 memory_limit 值、无效的 date.timezone、被 open_basedir 拒绝的路径。Apache 的 .htaccess 里一行 php_value，或者前端向 FPM 转发的 PHP_VALUE 头，都能触发它。把 error_reporting 关掉、用 @ 抑制、关掉日志，全都防不住，因为问题不在错误显示，在错误状态本身。  
  
第二条是会话上传进度（session upload progress）。这个特性在 php.ini 里默认开启，甚至不需要应用代码调用过 session_start。它也是 FPM 上已验证的利用路径，尤其在 multipart 处理中发生 OOM 之后。  
  
听起来像一个冷门特性，但它的问题恰恰写在代码里。修复前（php-8.4.26  
，ext/session/session.c  
，节选有删节），multipart POST 处理过程中每次刷新上传进度都会走到这里：  
```
staticvoidphp_session_rfc1867_update(php_session_rfc1867_progress *progress,int force_update){/* ... 频率检查，有删节 ... */	progress->next_update =Z_LVAL_P(progress->post_bytes_processed)+ progress->update_step;php_session_initialize();PS(session_status)= php_session_active;IF_SESSION_VARS(){		zval *sess_var =Z_REFVAL(PS(http_session_vars));SEPARATE_ARRAY(sess_var);		progress->cancel_upload |=php_check_cancel_upload(progress);Z_TRY_ADDREF(progress->data);zend_hash_update(Z_ARRVAL_P(sess_var), progress->key.s,&progress->data);}php_session_flush(1);}
```  
  
注意时序：php_session_initialize()  
 是在解析 multipart 请求体的过程中被调用的，此时正常的请求初始化流程（RINIT）还没走完，session 模块却已经开始解码、改写、回写会话数据。这正是「消费者在错误的时机读到状态」的那个时机。修复时，PHP 开发者在同一个位置上方留下了一段注释，把问题说得比任何外部分析都清楚（506e87fa7d  
 提交原文）：  
```
/* The upload progress handler (php_session_rfc1867_update()) decodes and  * re-encodes the session during request startup, before RINIT, when no user  * code may run. Objects are then kept as __PHP_Incomplete_Class: no  * autoloading, no unserialize_callback_func, no __wakeup() or __unserialize(),  * and they are written back unchanged. */
```  
  
翻译过来：上传进度处理器在请求启动阶段、RINIT 之前解码并重新编码会话，而这个阶段不允许任何用户代码运行。修复后，解码出的对象一律保持为不完整的类，不触发自动加载、不触发魔术方法，写回时不做任何改变。这条利用路径上的可乘之机，被一块一块砌死了。  
  
这个漏洞有一个天然的限制：生产者和消费者必须命中同一个 worker。worker 池越大、进程回收越频繁，成功率越低。不过攻击者可以用更多的请求次数来补偿。  
## 四、利用链：从一个上传请求到服务器上的 shell  
  
把两个漏洞拼起来，利用链是这样的（研究者已验证口径，防御视角）：  
  
第一步，投递。攻击者发送一个构造好的 multipart/form-data 请求，看起来就是一次普通的表单提交或文件上传。  
  
第二步，撑爆内存。利用头部放大或无 Content-Length 绕过等技巧，让请求在 post_max_size 限额之内耗尽 memory_limit。  
  
第三步，中断。OOM 触发 longjmp 跳出，或者致命错误打断清理流程，请求死在半路，脏状态留在 worker 进程里。  
  
第四步，兑现。同一个 worker 的清理阶段或下一个请求踩到悬垂数据，构成跨请求的 use-after-free。  
  
第五步，布局。利用链串联多个缺陷原语，并适配目标的内存布局，攻击者通过多次请求调整堆形态。  
  
第六步，落地。以 Web worker 账户（通常是 www-data）的身份执行任意代码。  
  
![php2shell 利用链六步示意](https://mmbiz.qpic.cn/mmbiz_png/icVSvicxhfMd44nMflO4ZTBicsCibicBLv1WjYnKr5pLgJcJiaz8Qse3clemWHATcL9So9xlwwnvFjGRAyLb4bhcY0kgqhAyGiaGP3o05Ly6xS6Dt8/640?from=appmsg "")  
  
php2shell 利用链六步示意  
  
▲ 利用链全景：步骤 2 至 3 是 CVE-2026-109618 的触发面，步骤 3 至 4 构成 CVE-2026-109619 的生产者与消费者；底部是不执行代码的内存泄露变体（依官方安全公告与 Sansec 研究绘制）  
  
Sansec 在研究中还确认了一个更安静的变体：被释放的 worker 内存会随 PHPSESSID cookie 返回给攻击者，一块 cookie 里可能躺着其他用户的密码、会话令牌、API 密钥，甚至是 TLS 私钥。这是 Heartbleed 式的信息泄露，不执行代码，直接读内存。  
  
因为漏洞在引擎层，任何 PHP 应用都是合格的投递载体。Sansec 已在 Magento、WordPress、Drupal、Joomla、MediaWiki、Roundcube、phpMyAdmin、Nextcloud、Laravel、Symfony、WHMCS 上验证了无需认证的远程代码执行。这份清单几乎覆盖了开源 Web 世界的半壁江山。  
## 五、影响：为什么说这一次几乎是「所有人」  
  
先看版本。官方修复覆盖四个仍在维护的分支：8.2.35、8.3.36、8.4.27、8.5.12。8.1 及更早版本没有官方更新，其中 7.4、8.0、8.1 提供了最小化的源码后向移植，需要发行版和运维自行处理；PHP 5 和 7 的分支则永远等不到官方补丁。另外别忘了，8.2 分支本身将在 2026 年 12 月 31 日停止维护。  
  
存量有多大？W3Techs 10 月 21 日的统计显示，在能识别服务端语言的网站里，69.3% 使用 PHP。而用 PHP 的站点里，27.0% 还在 7.x，7.7% 在 5.x。加起来，三分之一的 PHP 站点跑在拿不到官方补丁的版本上。  
  
![修复矩阵与 PHP 站点版本分布](https://mmbiz.qpic.cn/mmbiz_png/icVSvicxhfMd67zziaVNJ9dib2VjG0DtwVwvoNEOwj83mPcuTrFLxlj9HBgLkW74p7lCX7HURyyhbRoVdvFIm1IN3JgOS5JtndeK69icuCevHhv0/640?from=appmsg "")  
  
修复矩阵与 PHP 站点版本分布  
  
▲ 左：三个版本的修复档位，只有 8.2 至 8.5 拿得到官方安装包，而 8.2 分支年底就停止维护；右：W3Techs 统计的存量分布，34.8% 的 PHP 站点运行在无官方完整补丁的版本上（2026-10-21 数据）  
  
再看利用进展。watchTowr 的蜜罐网络在补丁发布当天观测到利用尝试；Cloudflare 披露在泄露之后看到了真实攻击流量，包括针对 WAF 规则的载荷变异，其 WAF 默认防护在 21 日凌晨 3:02 UTC 就已生效，比官方披露早了约 9 小时；Huntress 截至当天上午未在其客户群观测到利用。截至发稿，没有公开的漏洞利用代码，没有确认的 IOC，漏洞也不在 CISA KEV 清单上。但所有人都清楚，补丁一旦发布，代码差异就是最好的利用说明书。这场修复竞速的窗口，就是现在。  
  
CVSS 评分 9.2 而不是满分 10，原因写在向量里：攻击复杂度高（AC:H），且存在攻击前提条件（AT:P）。这不是点一下就中的漏洞，利用可靠性随构建、扩展、前端行为和内存布局浮动，可能需要多次尝试。但对攻击者来说，互联网上 PHP 站点的绝对数量保证了目标池的深度。  
## 六、现在该做什么  
  
升级是唯一完整的修复：8.5.12、8.4.27、8.3.36、8.2.35，对号入座。清点资产时注意角落：cPanel、Plesk 这类控制面板自带的 PHP，发行版打包的 PHP，还有厂商一体包里捆绑的运行时。Sansec 实测 Remi 第三方仓库在发版当天没有修复包，这类环境需要换发行版包或源码编译。  
  
升级完成前的临时缓解（官方公告口径，须在 FPM pool 或 Apache 配置里设置，ini_set 和 .user.ini 无效）：  
- •  
针对 CVE-2026-109618：在管理面设置 memory_limit=-1（不完全，底层的 mmap 失败仍可能走进漏洞路径），收紧 post_max_size，并在前端限制请求体大小；  
  
- •  
针对 CVE-2026-109619：关闭 session.upload_progress；PHP 8.5 上设置 fatal_error_backtraces=0；可选 pm.max_requests=1，让 worker 高频重生；  
  
- •  
特别注意：不要用 RLIMIT_AS 或 RLIMIT_DATA，它们引发的 mmap 失败恰好会触发漏洞路径；cgroup memory.max 可用，前提是确保只杀掉受影响的 worker。  
  
WAF 可以买时间，不能替代升级。Cloudflare 新上了请求加响应联动的 full transaction detection 检测和七条异常执行规则，但其官方口径很明确：修应用代码和插件没有用，必须更新 PHP 运行时本身。  
  
监测侧看三个信号：伴随内存超限报错或 HTTP 500 的异常 multipart 请求；过大或畸形的 PHPSESSID cookie；worker 进程异常换血。主机层面，worker 进程拉起 shell、解释器或下载工具，即为高危。  
## 七、一件事值得记住：AI 进了漏洞挖掘的核心圈  
  
这次披露的协同方有四家：Calif.io、Wiz、OpenAI、Quarkslab。官方安全公告的致谢名单里，能看到 Wiz 与 OpenAI 研究者的署名。据 Sansec 整理的时间线，Calif 与 OpenAI 合作，从 8 月 1 日开始寻找 RCE 路径，8 月 11 日上报 PHP；Wiz 在 8 月 24 日独立上报，并称一名研究员借助 AI 约一周就完成了利用链。三支研究团队都使用了前沿 AI 模型。  
  
PHP 是互联网存量最大的服务端运行时，内存安全漏洞的挖掘与利用长期是少数顶尖专家的领域。当 AI 把利用链的开发周期压缩到一周，这次事件的警示就不只针对 PHP 运维。上个月，一个自主 AI Agent 用两个零日漏洞打穿了荷兰的漏洞披露研究院 DIVD；这一次，AI 出现在漏洞发现的更上游。武器化的门槛在下移，而防御侧的补丁节奏仍然取决于人类工程师的响应速度。  
  
这次 PHP 团队从上报到发布补丁用了七十余天，泄露后又把发版提前了一天。这个速度在协同披露里不算慢，但面对被 AI 加速的攻击研究，恐怕会越来越不够用。  
## 参考资料  
- •  
PHP: October 2026 Security Releases（php.net 官方公告，2026-10-21）  
  
- •  
CVE-2026-109618 / GHSA-w8hw-w9fx-j79w 与   
CVE-2026-109619 / GHSA-6wmf-4rv5-p3rp（php-src 官方安全公告）  
  
- •  
php-src 提交 506e87fa7d "Avoid state corruption via bailout"（两个 CVE 的核心修复；本文代码块均节选自官方仓库：修复前代码取自标签 php-8.4.26  
，修复后代码取自该提交，删节处已逐处标注）  
  
- •  
php2shell: Critical PHP RCE（Sansec，2026-10-21）  
  
- •  
PHP Remote Code Execution FAQ: CVE-2026-109618（watchTowr，2026-10-21）  
  
- •  
php2shell, Leaked Critical PHP Vulnerabilities（Huntress，2026-10-21）  
  
- •  
Cloudflare deploys protections against php2shell（Cloudflare，2026-10-21）  
  
- •  
PHP Usage Statistics（W3Techs）（2026-10-21 读取）  
  
  
  
