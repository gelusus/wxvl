#  【漏洞预警】PHP再现远程代码执行漏洞，腾讯云安全已支持检测与防护  
 云鼎实验室   2026-10-21 23:49  
  
## 概述  
  
PHP 是一种开源的通用脚本语言，主要用于快速开发动态网页，可运行于大多数服务器和操作系统，在 Web 应用中使用广泛。  
  
近日，腾讯云安全中心监测到，PHP 官方于 2026 年 10 月 21 日披露两个远程代码执行漏洞，编号为 **CVE-2026-109618**  
和 **CVE-2026-109619，**  
代号“php2shell”。未经身份验证的远程攻击者可通过发送特制的 HTTP 请求触发内存破坏或释放后使用，最终以 Web 工作进程账户身份在服务器上执行任意代码，可能造成敏感数据泄露、恶意程序植入等危害。PHP 官方将两个漏洞均评为严重（CVSS v4 9.2）。  
  
官方已发布修复版本。建议使用 PHP 8.2 至 8.5 分支的用户尽快开展自查并升级；暂时无法升级的，可先采取下文的临时缓解措施。  
## 漏洞详情  
<table><tbody><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">漏洞名称</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 远程代码执行漏洞（php2shell）</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">漏洞编号</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">CVE-2026-109618、CVE-2026-109619</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">危害等级</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><span style="display: inline-block;background: rgb(215, 58, 73);color: rgb(255, 255, 255);font-size: 12px;font-weight: 600;padding: 2px 8px;border-radius: 3px;"><span leaf="">高危</span></span></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">漏洞类型</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">远程代码执行（内存破坏 / 释放后使用）</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">利用条件</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">无需身份验证、无需用户交互；利用复杂度较高，成功率受 PHP 构建、扩展、前端行为与内存分配器布局影响</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">影响范围</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 8.2.x &lt; 8.2.35、8.3.x &lt; 8.3.36、8.4.x &lt; 8.4.27、8.5.x &lt; 8.5.12</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;background: rgb(246, 248, 250);font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">修复版本</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">8.2.35、8.3.36、8.4.27、8.5.12</span></section></td></tr></tbody></table>## 漏洞原理  
### CVE-2026-109618：内存耗尽处理缺陷  
  
PHP 在发生内存耗尽（OOM）错误时，使用 longjmp()  
提前退出当前上下文。部分上下文中，请求内存在完成初始化之前就已处于可访问状态，如果此时任一内存分配失败，PHP 未能正确处理这次提前退出，请求内存会停留在无效状态。  
  
攻击者可构造包含大量头部、嵌套字段名或空文件控件的 multipart 请求，在不超出 post_max_size  
限制的前提下耗尽 memory_limit  
设定的内存上限，从而触发 OOM。在 Apache mod_php 这类无条件执行请求关闭流程的 SAPI 中，关闭流程会处理上述无效状态，导致内存破坏和远程代码执行。mod_php 环境下，这一过程发生在请求启动阶段对请求体的自动解析中，早于任何业务代码执行。  
### CVE-2026-109619：跨请求的陈旧全局状态  
  
PHP 工作进程会保留错误、会话、对象和扩展等全局状态，这些状态指向某一请求所拥有的内存，必须在该请求内存释放前完成重置。当致命错误（包括内存耗尽）中断了某个操作、使其未执行对应的重置，或者警告等诊断信息在关闭阶段重新填充了已清空的错误状态时，请求内存释放后，全局状态仍指向这块内存。  
  
下一个请求中，部分输入、配置、会话和错误处理操作会在正常初始化之前运行，访问到这些未被重置的陈旧状态，形成释放后使用。攻击者可在同一工作进程先后处理的两个请求中分别充当“生产者”和“消费者”，配合构造的堆布局，实现未经身份验证的释放后使用、内存泄露乃至远程代码执行。  
## 受影响的运行方式  
  
PHP 官方对两个漏洞在不同运行方式（SAPI）下的验证情况如下：  
<table><thead><tr style="background: rgb(246, 248, 250);"><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">运行方式</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">CVE-2026-109618</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">CVE-2026-109619</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">Apache mod_php</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">已验证完整远程代码执行（默认输入与内存限制、常规 prefork 多进程配置）</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">已验证远程代码执行（致命错误回溯路径、会话生命周期路径）</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP-FPM</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">请求启动失败时子进程直接退出，可避开启动阶段的利用路径；正常启动后仍可能触发。源码分析认定受影响，尚无完整利用演示</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">已验证远程代码执行（会话生命周期路径）</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">FrankenPHP</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">与 mod_php 类似，无条件执行请求关闭流程，很可能受到与具体业务无关的攻击</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">官方未单独说明</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">LiteSpeed（LSAPI）</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">行为模型与 PHP-FPM 相近，针对具体应用的利用很可能存在，尚未验证</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">官方未单独说明</span></section></td></tr></tbody></table>  
采用 Nginx + PHP-FPM 架构的业务同样受影响，CVE-2026-109619 已在 FPM 上验证可远程执行代码。  
## 影响版本  
<table><thead><tr style="background: rgb(246, 248, 250);"><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">分支</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">受影响版本</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">安全版本</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 8.2</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&lt; 8.2.35</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&gt;= 8.2.35</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 8.3</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&lt; 8.3.36</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&gt;= 8.3.36</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 8.4</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&lt; 8.4.27</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&gt;= 8.4.27</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">PHP 8.5</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&lt; 8.5.12</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">&gt;= 8.5.12</span></section></td></tr></tbody></table>  
可通过以下命令查看版本。命令行与 Web 运行环境可能使用不同的 PHP，请以实际处理 Web 请求的版本为准：  
```
php -vphp-fpm -v
```  
  
部分发行版的 FPM 命令带版本号后缀，如 php-fpm8.2 -v  
等形式。  
## 处置建议  
### 官方修复方案  
  
官方已发布漏洞补丁及修复版本，请评估业务是否受影响后，升级至安全版本。PHP 官方明确，升级到包含补丁的版本是唯一完整的修复方式。针对 CVE-2026-109618 的补丁移除了 OOM 场景下的 longjmp()  
调用，改为在下一个虚拟机安全点触发延迟的软性 OOM；内存占用若继续升至更高阈值，则通过 _exit()  
立即终止进程。  
  
官方下载地址：https://www.php.net/downloads.php  
  
升级前请做好数据备份，避免出现意外；升级后需重启 Web 服务与 PHP 工作进程，使新版本生效。  
### 临时缓解措施  
  
暂时无法升级的用户，可参考 PHP 官方给出的以下措施降低风险。这些措施不能替代升级，配置修改后均需重启全部 PHP 工作进程。  
  
**针对 CVE-2026-109618：**  
- 在管理层面设置并强制 memory_limit=-1，去掉可预测的单请求内存上限，这是官方给出的首要配置缓解。修改后需确认没有任何进程池、目录或应用配置重新设置了有限的内存上限。该措施并不完整，若 mmap() 等底层分配失败，PHP 仍可能进入存在漏洞的 longjmp() 路径。  
  
- 保持 post_max_size 的保守取值，并在前端（如反向代理、Web 应用防火墙）独立限制解码后的请求体大小。  
  
- 调低 max_input_nesting_level，并限制输入变量数量、multipart 分段数、每个分段的头部数量和上传文件数量。  
  
- 业务不使用 multipart 表单时，在前端直接拦截此类请求，不让其到达 PHP。  
  
- 可用容器或 cgroup 内存限制（memory.max）替代 PHP 自身的内存上限，但无法保证避免分配失败；若整个容器或 cgroup 被终止，主进程和全部 worker 会一并退出，除非每个 worker 都有独立的 cgroup。  
  
- 不要使用 RLIMIT_AS 或 RLIMIT_DATA 限制内存，达到上限后 mmap() 会失败，反而会触发存在漏洞的 longjmp() 路径。  
  
**针对 CVE-2026-109619（官方说明该漏洞不存在完整的通用配置缓解）：**  
- 设置 session.upload_progress.enabled=0，阻断官方演示的会话生命周期利用路径。业务不需要上传进度功能时，即使已升级也建议保持关闭。  
  
- PHP 8.5 用户设置 fatal_error_backtraces=0，去除官方已确认利用案例中使用的状态来源。  
  
- 减少攻击者可触达路径上的致命错误，包括上传、导入、错误页面、析构、序列化、输出处理、插件和扩展等环节，以及请求初始化之前生效的配置项所产生的警告与诊断信息。  
  
- 将 PHP 执行超时视同致命错误，避免攻击者可控的请求在受影响操作中可预测地触发“Maximum execution time exceeded”。仅调整超时时长只能消除这一种中断方式，无法缓解其他情况。  
  
- 性能允许时，可设置 pm.max_requests=1，让每个 FPM 工作进程只处理一个请求，切断跨请求的状态残留。  
  
## 腾讯云安全产品支持  
- 腾讯T-Sec 主机安全已支持该漏洞检测  
  
- 腾讯T-Sec Web应用防火墙已支持防护  
  
- 腾讯T-Sec 云防火墙已支持防护  
  
## 参考链接  
- 腾讯云安全公告：https://cloud.tencent.com/announce/detail/2525  
  
- PHP 官方公告（CVE-2026-109618）：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
- PHP 官方公告（CVE-2026-109619）：https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
- PHP 官方下载：https://www.php.net/downloads.php  
  
**-END-**  
  
**——关注云鼎实验室，获取更多安全情报——**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/VL7Qr6N3skia608BI7oAPyEaz4C6QibUnEeHTGICC2fJCfeu6AxgZk7QvqhbrOZrVs0ClYcQjqp0PFgETUTosTdGxSaH0Cg32eCk2OlVaXfrI/640?wx_fmt=png&from=appmsg "")  
  
本文参与腾讯云自媒体同步曝光计划，分享自微信公众号“云鼎实验室”。如有侵权请联系 cloudcommunity@tencent.com 删除。  
  
  
