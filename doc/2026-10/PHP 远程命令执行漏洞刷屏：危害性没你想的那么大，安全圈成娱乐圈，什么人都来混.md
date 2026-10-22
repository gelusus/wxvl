#  PHP 远程命令执行漏洞刷屏：危害性没你想的那么大，安全圈成娱乐圈，什么人都来混  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-22 01:19  
  
QUOTE  
  
漏洞是通用的，  
   
PoC 不是通用的  
   
。这两件事，被太多人混成了一件。  
  
—— 网络安全透视镜  
  
10 月 21 日，PHP 一次性发布四个安全版本，连同两个 CVSS 9.2 的高危漏洞一起把 PHP2Shell 推上了热搜。官方定性很直接：  
未认证远程代码执行  
。  
  
于是各种「一个 poc 通杀」的截图开始流传。但真正在实验室把命令执行跑通的人会发现，事情远比想象中复杂：  
触发漏洞很容易，稳定拿 shell 却极其苛刻  
。  
  
在供应链安全漏洞管理中，新的漏洞出现，相关方做升级修复本是正常现象  
  
推特(x)上高市早苗不知道抽什么风，政治圈混不下去了？也开始混安全圈，发了一条推特(x), 这事一下子就火了。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPrVcasYJtqUULTFglyf44moRyRhJ5mwZNxbMFWpicA16Nwibia4YPbHw6rGCH7mFQqicR5RBZj647Sz6vVic6F2iaabTibmibpTDFL1q80/640?wx_fmt=png&from=appmsg "")  
  
  
这篇就把完整利用条件一条条说明白，顺便说清楚这个漏洞真正的危害边界在哪里。  
  
  
本文看点  
  
01  
  
两个 CVE 怎么配合  
  
02  
  
完整利用条件清单  
  
03  
  
为什么没有通用 PoC  
  
01  
  
VULNERABILITY  
### 漏洞是什么：两个 CVE 各管一段  
  
  
PHP 8.5.12、8.4.27、8.3.36、8.2.35 已发布，修复了   
CVE-2026-109618  
 与   
CVE-2026-109619  
，两个 CVSS v4.0 评分都是 9.2。这两个漏洞是同源的，需要配合起来看。  
  
CVE-2026-109618 管的是「异常退出留下脏状态」。PHP 用 `memory_limit` 限制单请求内存，内存耗尽时走 `zend_mm_safe_error()` → `zend_error_noreturn()` → `zend_bailout()` 这条路。`zend_bailout()` 本质上是一次 `longjmp`，直接从当前任意 C 调用栈跳回请求顶层，中间那些还没做完的清理步骤就被跳过了——构造函数没跑完、链表没挂上、全局指针没复位。  
  
CVE-2026-109619 管的是「脏状态被下一个请求用上」。`mod_php` 和 `php-fpm` 的 worker 都是长驻进程，session 模块的 `PS(id)`、`PS(http_session_vars)` 是进程级全局变量。如果上一个请求是异常结束的，这些全局变量没被复位成 `NULL`，下一个请求就会把它当成有效对象去析构：  
  
.  
.  
.  
c  
  
zval_ptr_dtor(PS(http_session_vars))  
  
→ 访问上一请求留下的悬空指针  
  
→ Use After Free  
  
关键的连接点是 `session.upload_progress`。这个特性从 PHP 5.4 起就  
默认开启  
，multipart 解析回调会在你的应用脚本还没执行之前就跑起来。所以目标页面哪怕只有一行 ``，不需要 `session_start()`，不需要任何上传逻辑，一样会被卷进来。  
  
换句话说：这不是「上传一个文件就执行代码」，而是「构造一个请求把 PHP 自己的状态搅乱」。  
  
  
02  
  
EXPLOIT CONDITIONS  
### 完整利用条件：触发容易，拿 shell 很难  
  
  
利用条件必须拆成两层看。触发层很通用，拿命令执行层一点都不通用。  
  
第一层：能触发漏洞的条件  
  
这一层其实相当通用，满足下面几条就能把漏洞打出来：  
  
1  
  
PHP 没升到修复版本。  
  
  
2  
  
session.upload_progress.enabled = On  
（默认就是开，必须在   
php.ini  
 或   
php_admin_value  
 里关掉，  
ini_set()  
 来不及生效）。  
  
  
3  
  
memory_limit  
 是有限值。默认 128M 也能触发，实验环境用 8M 提高稳定性。  
  
  
4  
  
post_max_size  
 大于用来撑爆内存的那个字段。  
  
  
5  
  
攻击者能向任意   
.php  
 页面发 HTTP POST。  
  
  
6  
  
两次请求命中同一个 worker。  
  
  
打出来之后的现象是：第一个请求返回 `Fatal error: Allowed memory size of 8388608 bytes exhausted in Unknown on line 0`。注意 `Unknown on line 0`——  
错误发生在用户脚本开始之前  
。第二个请求可能看到非法 `PHPSESSID`、`Failed to read session data`、worker 退出、`zend_mm_heap corrupted` 或者干脆 SIGSEGV。  
  
注意  
  
这些现象能证明漏洞被触发了，但  
证明不了命令执行  
。看到 worker 崩溃就宣布 RCE，是这类分析里最常见的误判。  
  
第二层：能稳定拿到 RCE 的条件  
  
这一层就一点都不通用了。要把 UAF 变成 `system()` 调用，下面这些条件缺一不可，而且每一项都对具体构建高度敏感：  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">条件</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">实验室取值</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">不满足会怎样</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">精确镜像</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">php:8.4.26-apache-bookworm</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">堆喷尺寸全部失效</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">SAPI</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">mod_php + mpm_prefork</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">FPM 得另写一条利用链</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Worker 数</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无法稳定完成 A→B 同进程利用</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">display_errors</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">On</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Cookie 泄漏链失效</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">max_input_vars</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">默认 1000</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">超限 warning 的触发时机变了</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">memory_limit</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">8M</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">OOM 分配点漂移，对象落点改变</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">session.use_strict_mode</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Off</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">带 \x00 的 PHPSESSID 逼不出换 ID</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">libphp.so 符号偏移</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">zif_system = 0x45a3b0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">阶段 3 可能调错函数</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">堆布局常数</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">HTDATA = +0x690d8、filler = 841</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">重占失败，通常只是崩溃</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">用户态地址范围</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">本机映射约 0x79...</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">泄漏解析器会判定未泄漏</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">命令长度</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">有效命令不超过 14 字节</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">得换别的长度控制方式</span></section></td></tr></tbody></table>  
  
还有一个经常被忽略的：RCE 需要  
两类  
地址泄漏。一类是 Zend MM 的 chunk 基址（约 2 MiB 对齐），通过构造超长 `zend_string` 让 `php_url_encode(PS(id))` 越界读，从 `Set-Cookie: PHPSESSID=` 里捞出来；另一类是 `libphp.so` 的基址，靠读 `zval_ptr_dtor` 的地址减去目标构建的文件偏移得到。任一没拿到，伪造出来的 `HashTable` 就只是指向随机内存，表现出来是 worker 崩溃，不是命令执行。  
  
复现环境长什么样  
  
一个能跑通的靶场，PHP 配置是这些：  
  
.  
.  
.  
ini  
  
display_errors = On  
  
memory_limit = 8M  
  
post_max_size = 64M  
  
upload_max_filesize = 64M  
  
max_input_vars = 1000  
  
session.save_path = "/tmp"  
  
session.use_strict_mode = 0  
  
session.upload_progress.enabled = On  
  
Apache 侧要单 worker：  
  
.  
.  
.  
apache  
  
KeepAlive Off  
  
  
StartServers 1  
  
MinSpareServers 1  
  
MaxSpareServers 1  
  
MaxRequestWorkers 1  
  
ServerLimit 1  
  
  
请求 A 是一个带 `PHP_SESSION_UPLOAD_PROGRESS` 字段的 multipart POST，再加一个约 12 MiB 的字段把内存撑爆，目标是 ``。请求 B 用 841 个 48 字节字段加 79 字节伪造块去做堆重占，把伪造的 `ZSTR_LEN` 设成 `0x1000`，靠越界读从响应里的 `Set-Cookie` 泄漏出 chunk 基址。最后一个阶段用 15 字节 carrier 改写悬空 reference 的 `val.value`，再塞进携带伪造 `HashTable` 的字段，让析构链走成：  
  
.  
.  
.  
c  
  
php_session_track_init()  
  
→ zval_ptr_dtor(PS(http_session_vars))  
  
→ zend_reference_destroy(R0)  
  
→ zend_array_destroy(fake HT)  
  
→ zend_hash_destroy(fake HT)  
  
→ pDestructor(&bucket->val) = zif_system  
  
→ system("id>/out/p1")  
  
注意那个 fake HashTable 是依赖地址和堆布局的二进制结构，不是一个能抽象成「固定 Raw HTTP 包」的东西。直接把 stage3 的 body 复制到别的机器上，多数情况下只会得到一次纯崩溃。  
  
  
03  
  
WHY NO UNIVERSAL POC  
### 为什么没有一个包通杀  
  
  
这是最容易被误解的地方。PHP2Shell 是  
堆 UAF  
，不是「把 payload 交给某个业务函数执行」那种 RCE。  
  
普通应用层 RCE 的稳定原语是字符串、对象图、业务逻辑，跨版本换个 payload 常常还能用。PHP2Shell 的稳定原语是悬空指针和堆块尺寸——被释放对象的尺寸、下一次分配会不会重用同一个 bin、伪造对象字段的偏移、Zend MM 的堆布局、`libphp.so` 的符号偏移、ASLR 泄漏的结果、PHP 与 Apache 的初始化顺序，全都要提前知道。  
  
更扎心的是，同一个 PHP 版本也不通用。同样是 8.4.26，两个 Debian 大版本的偏移是这样的：  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">镜像</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">zif_system</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">zval_ptr_dtor</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Debian 13 Trixie</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">0x465370</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">0x65eb60</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Debian 12 Bookworm</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">0x45a3b0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">0x64eb60</span></section></td></tr></tbody></table>  
  
而堆喷参数 `841`、`406`、`+0x690d8` 是按 Bookworm 标定的，直接搬到 Trixie 上，5 次尝试全部 `stage1 HEAP_LEAK_FAILED`。更有意思的是，即使在同一 Bookworm 镜像上，把脚本支持的指针窗口从 `0x7f...` 放宽到 `0x55...–0x80...`，  
第一次尝试就直接 RCE 成功  
。  
  
「漏洞触发面是通用的，稳定 RCE 只能针对精确构建做定点利用。」  
  
能通用的是验证包，不能通用的是 RCE 包  
  
  
04  
  
REMEDIATION  
### 修复与缓解建议  
  
  
首选是升级到   
8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
，或者装发行版厂商的 backport。发行版的 backport 可能保持原版本号，所以别只看版本字符串，去 changelog 里确认有没有那两个 GHSA 编号。升级完必须重启所有 PHP worker、FPM pool 和 Apache/mod_php 进程。  
  
官方补丁的核心改动是：软 OOM 改成写 `EG(delayed_oom)` 并在 VM 安全点执行 `zend_throw_unwind_exit()`，硬 OOM 走新的 `memory_hard_limit_percent`，超过就 `_exit(137)` 直接退进程，不再在损坏状态下继续 shutdown；session 侧则在 `MULTIPART_EVENT_START` 时就立刻 `php_rinit_session_globals()` 复位 `PS(id)` 和 `PS(http_session_vars)`，解析阶段反序列化只生成 `__PHP_Incomplete_Class`，不再继续执行不安全回调。  
  
暂时升不动的话，缓解手段按优先级排：  
  
1  
  
session.upload_progress.enabled = 0  
，这条能断掉已展示的 session 链。  
  
  
2  
  
反向代理直接丢弃   
multipart/form-data  
 请求（不上传文件的应用适用）。  
  
  
3  
  
缩短 worker 生命周期，FPM 侧   
pm.max_requests = 1  
。  
  
  
4  
  
限制请求体大小，  
post_max_size  
 收紧。  
  
  
5  
  
放宽内存限制   
memory_limit = -1  
，减少 OOM 触发路径。  
  
  
别搞反了  
  
不要  
用   
RLIMIT_AS  
 /   
RLIMIT_DATA  
 去卡内存。那会导致   
mmap  
 失败，反而把进程推进那条脆弱的 OOM/bailout 路径。  
  
  
05  
  
DETECTION  
### 排查指标与应急处置  
  
  
如果线上出现下面这些现象，建议按「可能已失陷」处理：  
  
1  
  
zend_mm_heap corrupted  
  
  
2  
  
Allowed memory size ... exhausted in Unknown on line 0  
  
  
3  
  
异常超长的   
Set-Cookie: PHPSESSID=  
  
  
4  
  
worker 频繁 SIGABRT 或 SIGSEGV  
  
  
5  
  
session 文件读取异常、非法 session ID  
  
  
处置动作：轮换密钥凭据、排查 webshell、查 Apache/PHP worker 日志和反向代理访问日志、确认所有 worker 都已重启、核对 changelog 里的 backport 是否真的包含两个 GHSA。  
  
  
∞  
  
CONCLUSION  
### 结论  
  
  
PHP2Shell 的漏洞面是内核级的，默认配置就能触发 UAF 和状态损坏，这部分确实该重视；但要把它变成稳定 RCE，需要  
同 worker  
、  
精确的堆布局  
、  
精确的 libphp.so 偏移  
、  
成功的地址泄漏  
，以及对特定 INI 和 ELF 构建的预先标定。  
  
最可靠的动作始终是那一条：升级，然后重启。  
  
END  
  
我是 网络安全透视镜，专注网络与信息安全领域。  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
