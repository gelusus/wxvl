#  安全预警 | PHP远程代码执行漏洞（PHP2Shell）  
灰度安全
                    灰度安全  灰度安全   2026-10-23 09:08  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77hEFAb1ib7BEEN6iaXdRhaltz1JaYxb2icqWpoia0thiaYs0r8KQkFvLG6s1DJBHKqTv4S8WJic6LAzKibiaYkJxw8xZsN1tia8rOwibdCzI/640?wx_fmt=png&from=appmsg "")  
  
01  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77iag70vTK4gZku4zgTnxMY1rpUnWwSJVOiaEAlXGfobV73bNibvd3zdMnPra8ibhr1x7FNZuV4LU0lDVQU2KJBdHkcbRibbjpPFic1ms/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞概述**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77hluFGDic4bKVL3nlJtjwrGRAW4uH6JYsgbdkOkO2Cexm9WrsG6p8KlGvbbKvUBaibQQF6u91S4CQhClGOhqGCse9Ch5gvR7GtCQ/640?wx_fmt=png&from=appmsg "")  
  
  
2026 年 10 月 21 日，全球应用最为广泛的服务器端编程语言 PHP 被披露存在两处严重安全漏洞（CVE-2026-109618、CVE-2026-109619），安全社区统称为 “PHP2Shell”。两处漏洞 CVSS v4.0 评分均为 9.2，等级严重（Critical）。攻击者在特定条件下无需身份认证，即可通过构造的 HTTP 请求触发漏洞实现远程代码执行（RCE），以 Web 进程权限接管服务器，或读取内存中残留的口令、密钥等敏感数据。  
  
PHP 官方指出，所有对外提供 Web 服务的 PHP 部署均受影响，其中 Apache mod_php 风险最高，已验证的利用可在任何应用层代码执行之前触发。因漏洞细节在官方协调披露前遭提前泄露，PHP 官方提前于 10 月 21 日发布应急安全版本；补丁发布前后已有安全厂商观测到针对性利用与规模化扫描。鉴于 PHP 的覆盖面与在野利用态势，天弓实验室评估其具备大规模被利用的现实风险，建议相关单位立即排查并升级。  
  
  
02  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77hIuQXriacswbVVnMGicKtoW2I41HrCKhMzJG1Lfw6HibbtiaNZSEf7Zd7eXOhyMHPJMWX5A2uB4ZgYibck9OHwMJdO3iaWiaJibGj9kRQ/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞详情**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77ia11BmJhA65CPdZYBZSIgo8YcgoVjY9EO3qEjzWcNkekt2wyjWVBRf0yakLqEC8dCYfb9oyF6KZ8Qpia4PRPaBAe5StzQJDgSRs/640?wx_fmt=png&from=appmsg "")  
  
  
**CVE-2026-109618：基于 OOM 破坏的远程代码执行（RCE via OOM corruption）**  
  
PHP 在发生内存耗尽（Out-Of-Memory，OOM）错误时使用 longjmp() 退出当前上下文并中止请求处理。在某些上下文中，请求所占用的内存会在被完全初始化之前就临时可达；一旦此时任意一次内存分配失败，PHP 无法正确处理这次提前退出，使请求内存处于非法状态。随后的请求关闭流程会读取或基于这块非法内存进行分支判断。攻击者可通过构造 multipart 请求（例如不含有效 Content-Length、单个 part 内塞入超量 header、利用空文件控件绕过上传数量限制等）稳定触发 OOM 条件，最终导致远程代码执行。  
  
在 Apache mod_php 下，请求关闭流程会无条件执行，即便请求启动已失败——整个利用过程发生在任何应用层代码执行之前。相关弱点编号为 CWE-416（释放后使用）、CWE-755、CWE-824、CWE-1325 等。  
  
  
CVSS 向量：CVSS:4.0/AV:N/AC:H/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
  
**CVE-2026-109619：基于残留全局状态的远程代码执行（RCE via stale global state）**  
  
PHP 的错误、会话、对象、扩展等全局状态由工作进程持有，但其指向的是某个请求所拥有的内存。该状态必须在请求内存被释放之前完成重置。某些输入、配置、会话与错误处理操作会在下一个请求正常初始化之前运行，它们会误用上一个请求未被重置的残留状态，造成释放后使用（UAF）。通过精心构造的堆布局，攻击者可将其提升为完整的远程代码执行。  
  
触发需同时具备 “生产者” 与 “消费者” 两个条件：生产者是将某请求所拥有状态存入全局可访问位置的 PHP 操作；消费者是在后续请求（由同一工作进程处理）正常初始化之前，读取、释放或基于该残留状态分支的操作。已确认的两类消费者为：请求初始化前的 INI 诊断，以及正常初始化前的会话上传进度（session upload progress）处理。会话上传进度功能默认开启且无需应用调用 session_start()，因此成为重点利用面。  
  
  
CVSS 向量：CVSS:4.0/AV:N/AC:H/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
  
**综合危害**  
- 未授权远程代码执行：以 Web 工作进程权限在服务器上执行任意代码或命令，进而植入 WebShell、横向移动。  
  
- 内存数据泄露：可读取其他请求遗留在内存中的口令、令牌、密钥等敏感信息，部分路径影响类似 Heartbleed。  
  
- 拒绝服务：可触发工作进程崩溃 / 频繁重启，影响业务可用性。  
  
  
  
  
**BAS验证向量**  
  
灰度安全先知BAS已添加该漏洞验证向量，提示广大用户及时进行设备知识库升级。  
  
  
03  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77g7cia9mJbJSTSxe6MJ2kQAiarCQxsqy5vPUHDyXNtaKqzaxN9Fue9tqAS4ibkT0ye3LIuMpJEwFpYfF5ELoiajBK9FMiaX8oMfnkww/640?wx_fmt=png&from=appmsg "")  
  
  
**影响范围**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77giaplgJCJX9ESNp9ib1uJvw4mIJt8Gv11XoCQspfOF6W02c6D8tOMP6o7apqXXr5LQwjFKY2Y8MNYy1iayklKkr2reK86OlZtB98/640?wx_fmt=png&from=appmsg "")  
  
  
公开报告显示，自 PHP 5 以来的各版本均可能受影响。所有对外提供 Web 服务的 PHP 部署方式均在受影响范围内，包括 PHP-FPM、Apache mod_php、php-cgi、FrankenPHP 与 LiteSpeed，其中 mod_php 风险最高。  
  
  
**受影响版本**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77iaAtsFiafvDPNIVzFg5nzEQcuCGQGP6RrMzia7cEbUPumU62jVd1mImBRNraxLTGgm0Ulnl7xNB5WuwWn7sicOIepRpZoAc1A3oJM/640?wx_fmt=png&from=appmsg "")  
  
说明：PHP 8.1 及更早版本已不再获得 PHP 官方更新。鉴于本次漏洞的严重性，官方额外为 PHP 7.4、8.0、8.1 提供了最小化源码 backport，供自行编译或无 LTS 厂商支持的用户使用。使用 rpms.remirepo.net 的用户应改用发行版软件包或从源码编译以便及时修补。  
  
  
04  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77gDyWtuRgn26rK8LnOJYkUOiczCreBtEHTlIEjvknm35UkHDl0vtuVdAa19zhQNIwFo8YJczDiacbQKYKG0ic6Dbs09AIA65MNFr0/640?wx_fmt=png&from=appmsg "")  
  
  
**自查方法**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77gaLXdF28icZZ9jgD79FbvM3zIqlw65YRHbE4W9IX62EZOmlmxgvb2vrbia6qiaHibAfyTMPFydwCpfecM4vTWjjbJzdNHpoZNAdn4/640?wx_fmt=png&from=appmsg "")  
  
  
请在服务器侧（而非仅命令行）确认 Web 服务实际加载的 PHP 版本与配置，命令行 php -v 与 Web 实际使用的版本可能不同：  
1. 清点所有 PHP 运行时：包括容器、控制面板、CMS（WordPress、Drupal、Joomla、Magento 等）及自带 PHP 的厂商设备。  
  
1. 核对版本：执行 php -v、php-fpm -v 或检查 Apache 模块；通过响应头（expose_php 开启时的 X-Powered-By）从外部粗略判断，但无该头不代表安全。  
  
1. 核对会话上传进度配置：检查 Web 服务实际加载的 php.ini / pool 文件中 session.upload_progress.enabled（默认值为 1）。  
  
1. 标记停止维护分支：PHP 8.1 及更早版本需 backport、升级或隔离处置。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77hEnIg5m57ibqDykKZ1vFPO35YpSmIWI0ic7M6K7mar2YyPzq86V47ibp45LicfPicsJBjbmL0RfRDJkRDppTVyib0SVByOA4fEw0CDI/640?wx_fmt=png&from=appmsg "")  
  
  
05  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77gZwcTTnq6ibOVax0XsqzXXicRdVwibu4yO5Y8r1zibvwPWQyZG7icHOBKKVKBESp7zAbL4ZI4apiaV6X1pTKfLjEsfy4phmicGXRkt6Q/640?wx_fmt=png&from=appmsg "")  
  
  
**处置建议**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77hRMQLfxiaCJjp61xn5lHE12ibag1PC6sVicJiclickhVcIm8OQgTdaVLPczCssEgicicTZ4wOsFf0j28kz7fruxM5ia9NTu8W2Apwic4eY/640?wx_fmt=png&from=appmsg "")  
  
  
**升级修复（唯一完整方案）**  
  
PHP 官方明确指出，升级到含补丁的版本是唯一完整的缓解措施。请尽快升级至以下安全版本：  
- PHP 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
- 优先修补对外暴露的系统；升级后务必重启 PHP-FPM / Apache 并重建容器镜像，确保运行中的工作进程加载新代码。  
  
- Linux 发行版与 LTS 厂商已提前获得补丁，请通过官方软件源获取更新包。  
  
- 官方下载：https://www.php.net/downloads.php  
  
  
  
  
**临时缓解（无法立即升级时，均非完整修复）**  
  
以下措施仅能降低特定攻击路径的暴露面，不能使系统安全，任何未打补丁的主机都应视为已暴露：  
1. 针对 CVE-2026-109618：通过 php_admin_value 设置并强制 memory_limit=-1；收紧 post_max_size（无上传场景设为 8M，有上传设为 32M），并在 Web 服务器 / 代理 / WAF 侧限制解码后请求大小、字段数、part 数、header 数与嵌套深度；无需 multipart 时在前端直接拦截。切勿使用 RLIMIT_AS / RLIMIT_DATA。  
  
1. 针对 CVE-2026-109619：在 php.ini / FPM pool / 服务器配置中设置 session.upload_progress.enabled=0 并重启全部工作进程（勿用 ini_set() 或 .user.ini）；PHP 8.5 另需设置 fatal_error_backtraces=0；清理在请求初始化前会产生诊断的无效 / 过时 INI 设置；消除攻击者可达的致命错误与执行超时。  
  
1. 对无法及时修补的系统，应将其从公网隔离、置于 WAF/VPN 之后或直接下线。Cloudflare 等已为已知利用向量下发了 WAF 规则。  
  
  
  
  
06  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77j85QlwB30oc9M2JTrYLeRwjtMicicSicMMDIgdneDWb85CAmDMV5fSWBsicZ4K3FTlZOiaQiaQVicMt3A3qG9mPCibibnNTFrbWhSnoUdk/640?wx_fmt=png&from=appmsg "")  
  
  
**事件时间线**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77jewo6mt4ich6ibKf93XgbnT39MYRgqhYxMaVKlpT20I5ePvnuHEoxufsgtSuxDrqS5YrXQqSH5EondQdaKTtMBzpjPA9Ks7PXEs/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77gn4DmzguDoiczUiahCmP1Xk55gNqSric3vBE5nnnya6msG6SyhpgJpxTMJMHXnXW3VOwOUXc7LRISnpicZiaeZyXGsbufyTcqvCw3Y/640?wx_fmt=png&from=appmsg "")  
  
  
07  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WgoiaaDMW77h2NDATdxkLic3FoCQYJeGbUMNzDVWoqtyRcVkiagZp7ZjRSDmbP7dgicchvWFa2Qic9N26fE6ViaLpEkiazJrf9MuqQbzqrFhe8f95Q/640?wx_fmt=png&from=appmsg "")  
  
  
**参考链接**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WgoiaaDMW77hjTCibT6BVuxHm8fOmR40JcF38ibdw3SiciakibpeA2QWe5nhV1SGhRBTehP2pM6eCxE930SYsSuaEJ6ISCNsCMLRLIkfytUyichLPE/640?wx_fmt=png&from=appmsg "")  
  
  
[1] PHP 官方安全公告：https://www.php.net/archive/2026.php#2026-10-21-5  
  
[2] GitHub GHSA（CVE-2026-109618）：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
[3] GitHub GHSA（CVE-2026-109619）：https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
[4] PHP 下载页面：https://www.php.net/downloads.php  
  
[5] CERT-EU Security Advisory 2026-018：https://cert.europa.eu/publications/security-advisories/2026-018/  
  
[6] PHP 应急升级指引（参考）：https://jackalsyn.com/posts/php-up-2026.html  
  
  
  
**从风险暴露到主动防御——灰度安全 AI 实战化安全风险运营**  
  
PHP2Shell 等高危漏洞的出现，再次凸显了企业面临的现实挑战：漏洞修复需要争分夺秒，但仅完成补丁升级并不意味着风险彻底消除，企业还需要确认暴露资产是否全面识别、安全设备是否有效l拦截、潜在攻击路径是否得到控制。  
  
北京灰度科技有限公司（灰度安全）依托天弓实验室深厚的攻防研究积淀，以持续威胁暴露面管理为核心，深度融合大模型、智能体与自主攻防技术，打造集攻击面发现、安全防护有效性验证、数据安全验证、AI 深度渗透与智能安全运营于一体的 AI 实战化安全运营平台。平台以 AI 驱动安全能力持续进化，通过智能感知、动态推理、自主探索与多智能体协同，实现对新型威胁的快速分析、潜在攻击路径的深度挖掘、安全防护效果的自动化验证及风险处置建议的智能生成，推动安全运营从被动响应向主动防御、从单点检测向体系化验证、从人工驱动向智能协同升级，助力企业全面掌握风险暴露态势、精准验证防护能力、持续提升实战化安全防御水平。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Jtyo8XW8JbtEnRUqcU6YV8LadCqpLP7wlOficBxIQFbXtY6dCadNT3QCjT9jyS7rNkcZGSsyTNG4Uwb8FvPJZng/640?wx_fmt=png&from=appmsg "")  
  
  
