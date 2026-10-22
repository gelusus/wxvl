#  php2shell最新进展：PHP Core 远程代码执行漏洞详解  
原创 森林之家zbs
                    森林之家zbs  allby森林之家   2026-10-22 04:35  
  
10 月 21 日，PHP 一口气发布四个安全版本：8.5.12、8.4.27、8.3.36、8.2.35，修复 CVE-2026-109618 和 CVE-2026-109619——这两个 CVSS v4.0 都是 9.2 的未认证远程代码执行漏洞，也就是这两天刷屏的 php2shell。  
  
网上什么说法都有：有人说“一个 PoC 通杀全网”，有人说“它危害没你想的那么大”，还有人（我本人）说[补丁没出POC先到了？PHP最新高危漏洞+免杀马完整攻击链](https://mp.weixin.qq.com/s?__biz=Mzg3NjY2NTYzOQ==&mid=2247486305&idx=1&sn=d2db36984f78880efa702a98b21304c9&scene=21#wechat_redirect)  
  
  
在这众说纷纭之下，只有时间才会告诉大众真相……  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/2wf8V0A7uBf8hLC98a6tFBWG7ibzxuWrxSCXL2T6iaZ1MHJRetfeNxRObicHibcab87JewUicLU1tsAiajkXibAE9F1oPicGfOyQPrpG6pSPXEicYcGU/640?wx_fmt=jpeg "")  
  
所以今天我们继续跟进php2shell的最新进展，把它的漏洞原理、真实危害边界和处置动作一次讲清楚。  
## 01 最新进展  
  
官方补丁已发布，微步判定为高风险漏洞。  
  
补丁已出  
：8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 四个分支的修复版本已发布，覆盖当前所有官方维护的分支。但补丁上线后战况已经很紧：社区监测显示补丁发布约 15 分钟内，就有大规模自动扫描开始全网探测；AWS 蜜罐在公开披露前就已捕获到利用尝试。  
  
**最新动态**  
：已有多个国家政府网站被逼停服，安全公司 watchTowr 甚至给出满分 10.0 的评价。这个漏洞自 2012 年发布的 PHP 5.4 起就存在，隐藏超过十年。  
  
**但到目前为止，网上还没有公开的、准确可复现的 PoC。**  
流传的大多是验证崩溃的探测包，不是稳定的命令执行链——这一点后面细说。  
  
8.1 及更早版本官方不再发版，但给了仅供自行编译的最小 backport 分支（PHP 7.4 / 8.0 / 8.1 各有对应 CVE backport 分支）。  
  
**风险定性**  
：微步情报局综合处置优先级判定为"高风险"，利用条件评估是"远程、无权限、无交互、无需绕过安全机制"，已有 PoC 流出，暂未捕获在野利用。  
  
**影响面**  
：PHP 核心层漏洞，主流 CMS 和框架（WordPress、Laravel、Drupal 等）只要解释器没打补丁都在范围内——和你的业务代码用不用 session 无关。  
## 02 这两个漏洞是什么？  
  
这两个洞是同源的，得配合起来看。  
  
**CVE-2026-109618：异常退出留下脏状态。**  
  
PHP 用 memory_limit 限制单请求内存，内存耗尽时走一条"紧急退出"路径（zend_bailout，本质是 longjmp）。这一跳，直接从当前 C 栈跳回请求顶层，中间没跑完的构造、没挂上的链表、没复位的全局指针，全被跳过。  
  
**CVE-2026-109619：脏状态被下一个请求用上。**  
  
mod_php 和 php-fpm 的 worker 是长驻进程，session 模块的全局变量是进程级的。上一个请求异常结束时这些全局没被清成 NULL，下一个请求就把它当有效对象去析构——悬空指针，Use After Free。  
  
**连接点：session.upload_progress。**  
  
这个特性从 PHP 5.4 起默认开启，multipart 解析回调在业务脚本执行之前就跑了。所以哪怕目标页面只有一行 <?php echo "OK"; ?>  
，不需要 session_start()，不需要任何上传逻辑，一样会被卷进来。  
  
换句话说：这不是"传一个文件就执行代码"，而是"构造请求把 PHP 自己的状态搅乱"。  
  
**拼成 RCE 的过程**  
：请求 A 用带 PHP_SESSION_UPLOAD_PROGRESS 的 multipart POST 把内存撑爆，触发 OOM 紧急退出，留下脏的 session 全局；请求 B 打到同一个 worker，复用被释放的内存块，把析构链改写成调用 system()。  
## 03 为什么没有"通杀 PoC"？  
  
这是全网吵得最凶的地方。  
  
结论其实很简单：**漏洞触发面是通用的，稳定 RCE 只能针对精确构建做定点利用。**  
  
原因是 php2shell 是堆 UAF，不是"把 payload 交给某个业务函数执行"那种应用层 RCE。  
  
普通 RCE 的稳定原语是字符串、对象图，换个版本常常还能打。php2shell 的稳定原语是悬空指针和堆块尺寸——被释放对象多大、下一次分配会不会重用同一块、伪造对象的字段偏移、Zend MM 堆布局、libphp.so 符号偏移、ASLR 泄漏结果、PHP 与 Apache 初始化顺序，全部要提前知道。  
  
更扎心的是：同一个 PHP 8.4.26，换一个 Debian 大版本，偏移量就全变了。  
```
```  
  
堆喷参数按 Bookworm 标定，直接搬到 Trixie 上，5 次尝试全部失败。  
  
所以：**能通用的是验证包，不能通用的是 RCE 包。**  
网上流传的"一个 poc 打全网"，大概率只是触发崩溃的验证包，不是真正拿 shell 的。  
## 04 利用条件  
  
触发漏洞容易，但拿shell却很难。  
  
我们可以把洞拆成两层看，就会发现想要复现漏洞的条件一层比一层苛刻：  
  
**第一层：能触发漏洞（相对通用）**  
  
1. PHP 没升到修复版本；  
  
2. session.upload_progress.enabled = On（默认开，必须在 php.ini 里关，ini_set 来不及）；  
  
3. memory_limit 是有限值（默认 128M 也能触发，实验环境常用 8M 提稳定性）；  
  
4. post_max_size 大于撑爆内存的那个字段；  
  
5. 攻击者能向任意 .php 页面发 HTTP POST；  
  
6. 两次请求命中同一个 worker（单 worker 时 100%）。  
  
打出来的现象：第一个请求返回 Allowed memory size ... exhausted in Unknown on line 0  
——注意 "Unknown on line 0"，错误发生在用户脚本开始之前。第二个请求可能出现非法 PHPSESSID、session 读取失败、worker 退出、zend_mm_heap corrupted，或干脆 SIGSEGV。  
  
划重点：**看到 worker 崩溃就宣布 RCE，是这类分析最常见的误判。**  
这些现象只能证明漏洞被触发了，证明不了命令执行。  
  
**第二层：能稳定 RCE（极不通用）**  
  
要把 UAF 变成 system() 调用，下面条件缺一不可，每一条都对具体构建高度敏感：  
- 精确镜像（实验环境 php:8.4.26-apache-bookworm）；  
  
- SAPI = mod_php + mpm_prefork（FPM 得另写一条链）；  
  
- worker 数 = 1（否则无法稳定完成 A→B 同进程利用）；  
  
- display_errors 等一堆 INI 精确匹配；  
  
- 两次地址泄漏：Zend MM chunk 基址（从超长 Set-Cookie 里捞）+ libphp.so 基址（读 zval_ptr_dtor 地址反推）；  
  
- 有效命令不超过十几个字节。  
  
任一条件不满足，多数结局是"worker 崩溃"，不是"命令执行"。  
  
![CVE-2026-109618/109619修复版本矩阵](https://mmbiz.qpic.cn/sz_mmbiz_jpg/2wf8V0A7uBcxICUSwLavyAb3ONwH494b20EylUcTB0aE48xAY6PticKAySl5gu3kFxGiclQ2sicAleHjC6ibv9fyO28Dvw3r4gyq9KXGibwdCWas/640?wx_fmt=jpeg "")  
## 05 修复与缓解  
  
**修复最佳首选手段，肯定是——升级。**  
  
升到 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35，或用发行版厂商的 backport。**注意：backport 可能保持原版本号，别只看版本字符串，去 changelog 确认有没有那两个 GHSA 编号。**  
  
升级完必须重启所有 PHP worker、FPM pool、Apache/mod_php 进程——二进制更新了，旧进程内存里还是老代码。  
  
**暂时升级不了话，我也搜集到了其他缓解手段（按优先级排序）：**  
  
1. session.upload_progress.enabled = 0（断掉 session 利用链）；  
  
2. 反向代理直接丢弃 multipart/form-data 请求（不上传文件的应用适用）；  
  
3. 缩短 worker 生命周期，FPM 侧 pm.max_requests = 1；  
  
4. 收紧 post_max_size；  
  
5. 放宽 memory_limit = -1（减少 OOM 触发路径，注意会增加内存耗尽风险）。  
  
**提醒！一个别搞反的坑**  
：不要用 RLIMIT_AS / RLIMIT_DATA 去卡内存——那会导致 mmap 失败，反而把进程推进那条脆弱的 OOM/bailout 路径。  
  
  
出现下面现象，按"可能已失陷"处理：  
  
1. zend_mm_heap corrupted；  
  
2. Allowed memory size ... exhausted in Unknown on line 0；  
  
3. 异常超长的 Set-Cookie: PHPSESSID=；  
  
4. worker 频繁 SIGABRT / SIGSEGV；  
  
5. session 文件读取异常、非法 session ID。  
  
处置动作：轮换密钥凭据、排查 WebShell、查 Apache/PHP worker 日志和反向代理访问日志、确认所有 worker 都重启、核对 changelog 里的 backport 是否真的包含两个 GHSA。  
  
**漏洞面是内核级的，默认配置就能触发，这部分该重视；但稳定 RCE 需要同 worker、精确堆布局、精确符号偏移和地址泄漏——所以现在修复还来得及，你的网站大概率还只是"可能被触发漏洞"。**  
  
【免责声明】本文基于公开安全情报与技术报告整理，仅供安全研究与防御认知，不提供任何可利用的完整攻击代码。  
  
  
