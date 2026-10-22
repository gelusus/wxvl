#  两个 CVSS v4.0 9.2 严重漏洞、无需认证即可远程执行代码  
 晟晖科技   2026-10-22 03:10  
  
10月21日，PHP 官方提前发布紧急安全补丁，修复两个 CVSS v4.0 评分 9.2 的远程代码执行漏洞。攻击者无需认证即可远程执行任意代码，影响范围覆盖 PHP 5.4 以来的绝大多数在网版本。补丁发布不足 24 小时，已有多家厂商报告观测到在野利用尝试。  
  
  
##   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wFG4yuGVVyJCgHUsFaCEKYwgT7wNDNksGGAR2gJK9UtJRgqyhfAt8rGsyr5f7vwREykH3uwYAq5ibib6mulr8SBmWL89JgHKd5TcwKkKgnWoU/640?wx_fmt=gif&from=appmsg "")  
  
**事件速览**  
  
  
此次披露的两个漏洞为 CVE-2026-109618 和 CVE-2026-109619，官方评级均为 CVSS v4.0 9.2（Critical，严重级别）。业界以"php2shell"指代该组漏洞。漏洞位于 PHP 解释器核心，构造特制请求即可触发，无需身份认证。  
  
**0****1**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wFG4yuGVVyIJqn8auXWOxzDhMypluBYvuFhqAB2sTiaCPJqNCarlPPJsJiajJ3ZIib0oDQON3K0WYzL3xVnwhUzbz9QVlfanP9P1nuAPQUbuOo/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞编号**  
：CVE-2026-109618、CVE-2026-109619（GHSA-w8hw-w9fx-j79w、GHSA-6wmf-4rv5-p3rp）  
  
**危险等级**  
：严重（CVSS v4.0 9.2）  
  
**攻击条件**  
：无需认证，远程可利用  
  
**披露时间**  
：2026年10月21日（早于原定计划，因细节提前外泄）  
  
  
**发现经过**  
：由 Calif、OpenAI 与 Wiz 于今年 8 月先后独立发现并协调披露——Calif 与 OpenAI 于 8 月 11 日各自独立报送，Wiz 后续补充了更多攻击路径。三方均表示，证明这些缺陷的可利用性需要极复杂的步骤链，处在人力研究者能力的边缘，是前沿 AI 模型把这道门槛压了下来。  
  
  
**成因简述**  
：两处漏洞均源于内存管理缺陷。CVE-2026-109618 属内存耗尽（OOM）处理中的内存破坏——PHP 使用 longjmp() 提前退出上下文，导致请求数据结构尚未完全初始化即被清理逻辑当作有效数据处理，进而形成可控的内存破坏；攻击者可构造包含大量请求头、嵌套字段名或空文件控件的 multipart 请求，在不超出 post_max_size 的前提下耗尽 memory_limit。  
  
CVE-2026-109619 属跨请求的陈旧全局状态问题——工作进程保留指向前一请求内存的全局状态且未完全重置，处理后续请求时形成释放后重用（UAF）。  
  
官方已确认session上传进度（session.upload_progress）是其中一条可利用路径。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wFG4yuGVVyJFJRcGb1HpBl31LoqBpSr19t0AYCeic5I2ka6Pn4HjoLEh2ps81yVLLnPhAjxKfIlMRbnAhPRK2F2fdBhicnhkI95DNsEU2zoEo/640?wx_fmt=gif&from=appmsg "")  
  
**影响范围**  
  
  
PHP 是目前全球使用最广泛的服务端脚本语言之一，据行业统计，全球约七成网站的服务端运行在 PHP 之上。WordPress、Drupal、Joomla、Magento 等主流 CMS 均以其为底层，国内大量政务、教育、医疗、企业网站亦基于 PHP 构建。  
<table><tbody><tr><td data-colwidth="149.141px" width="149.141px" style="word-break: break-all;box-sizing: border-box;"><p><span style="font-size:15px;"><strong><span leaf="">项目</span></strong></span></p></td><td data-colwidth="312.859px" width="312.859px" style="box-sizing: border-box;"><p><span style="font-size:15px;"><strong><span leaf="">内容</span></strong></span></p></td></tr><tr><td style="word-break: break-all;"><p><span style="font-size:15px;"><span leaf="">受影响版本</span></span></p></td><td><p><span style="font-size:15px;"><span leaf="">PHP 5.4 起至 8.5.11（含 8.5 / 8.4 / 8.3 / 8.2 各分支）</span></span></p></td></tr><tr><td><p><span style="font-size:15px;"><span leaf="">官方修复版本</span></span></p></td><td><p><span style="font-size:15px;"><span leaf="">PHP 8.5.12、8.4.27、8.3.36、8.2.35</span></span></p></td></tr><tr><td style="word-break: break-all;"><p><span style="font-size:15px;"><span leaf="">无正式版本分支</span></span></p></td><td><p><span style="font-size:15px;"><span leaf="">PHP 8.1 及更早版本已停止安全支持（含仍被大量使用的 PHP 7.x、8.0、8.1），官方不发布正式修复版本，仅提供 7.4 / 8.0 / 8.1 的最小化源码回移植，需按过渡措施对待，并安排隔离或迁移</span></span></p></td></tr><tr><td style="word-break: break-all;"><p><span style="font-size:15px;"><span leaf="">受影响部署方式</span></span></p></td><td><p><span style="font-size:15px;"><span leaf="">PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed 等；其中 Apache mod_php 风险最高</span></span></p></td></tr><tr><td style="word-break: break-all;"><p><span style="font-size:15px;"><span leaf="">是否存在完整缓解</span></span></p></td><td><p><span style="font-size:15px;"><span leaf="">官方明确表示无完整缓解措施，升级是唯一安全选择</span></span></p></td></tr></tbody></table>  
需要特别说明：漏洞在请求进入业务代码之前即可触发。即便是已打全补丁的 WordPress、Laravel 等应用，只要底层 PHP 未升级，依然暴露在风险中——升级框架无用，必须升级 PHP 本身。  
  
  
关于在野利用：补丁发布前后，Cloudflare 表示在补丁落地前即已看到早期攻击；watchTowr 在其全球蜜罐网络中观测到利用尝试，但暂未见到成功案例；Patchstack 的遥测则以探测流量为主。各机构对当下规模的判断并不一致，但对趋势的判断一致——公开 PoC 已开始流传，攻击窗口期正在打开。需注意此类热点漏洞的公开 PoC 仓库中常混有投毒样本，来源不明的工具不要直接运行。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wFG4yuGVVyKBuUj6SIhjkv7Dk8eWh0hdh5VjV4G4sZt9EunQ11KNFk6LZqLeh9EF3MjHhbc0pIlDDwdmauLvGXBxVjwXKaK0tcVlsdz0IVM/640?wx_fmt=gif&from=appmsg "")  
  
**紧急应对建议**  
  
  
**立即升级（最高优先级）**  
  
排查所有对外提供服务的 PHP 环境，执行 php -v 确认版本与部署方式（PHP-FPM / mod_php / php-cgi）。按分支升级至 8.5.12、8.4.27、8.3.36 或 8.2.35，不走常规维护窗口，按紧急变更处理，并重启每一个 PHP 工作进程。容器镜像、共享主机、第三方托管环境常被遗漏，需一并核查。  
  
  
**临时缓解（补丁无法立即落地时）**  
- 若业务不使用上传进度功能，在 php.ini、.user.ini 或 .htaccess  
  
中设置 session.upload_progress.enabled=Off，并重载 PHP-FPM 或 Web 服务。该措施可切断已验证的 session 生命周期路径，但不能替代官方补丁。  
  
- 在 WAF、反向代理或网关层收紧对 multipart/form-data 请求的检查。  
  
重点保护不需要文件上传的业务端点；可考虑拦截请求体含 PHP_SESSION_UPLOAD_PROGRESS 关键字的请求，但需注意会影响所有使用上传进度的业务。  
  
- 对 PHP 8.1 及更早的停止支持版本，采取下线、网络隔离、VPN 或白名单访问等措施，直至完成迁移；已有官方最小化源码回移植的分支（7.4 / 8.0 / 8.1）可作为过渡，但须按临时措施对待并锁定迁移时间表。  
  
**排查与监测**  
  
检查 Web 服务与 PHP 日志中的异常访问记录、工作进程异常退出或崩溃日志、PHP 进程派生的异常子进程与异常文件写入，确认是否已遭入侵。重点检索近期含上传进度字段的 multipart POST 请求，以及工作进程返回 5xx 后紧跟的异常请求。同时持续关注 PHP 官方安全公告与 CNVD 后续更新。  
  
  
**云环境注意**  
  
谷歌云已回应：Google Cloud 云平台本身不受这两个漏洞影响，但在其上运行 PHP 的用户仍可能面临风险，并敦促客户升级至 PHP 8.2 或更高版本。云厂商负责的是平台层，云上自建 PHP 环境、托管镜像与容器基础镜像仍需自行核查。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4ia5UrsIrkGrdzLz4x5Nbv7F8m1CQmrcTyM2C93TQoITGibV8TL7dJYbPS0I7SWC4mjCXAzUTJegqQ/640 "")  
  
  
一个值得重视的信号是：漏洞细节先于补丁外泄，AI 又把可利用性验证的门槛压低——两者叠加，留给防守方的反应窗口正在被压缩。对这类位于解释器运行时本身、无法靠移除依赖规避的漏洞，收敛暴露面与快速升级仍是目前最可靠的控制手段。  
  
## 参考来源  
- PHP 官方安全公告（php.net/archive/2026.php）  
  
- GitHub Security Advisory：GHSA-w8hw-w9fx-j79w、GHSA-6wmf-4rv5-p3rp  
  
- 国家信息安全漏洞共享平台（CNVD）已收录 PHP Core php2shell 远程代码执行漏洞相关公告  
  
- watchTowr、Cloudflare、Patchstack 等在野利用与遥测报告  
  
![](https://mmbiz.qpic.cn/mmbiz_png/JaFvPvvA2J1l9248vhzo8jicBPJaBT4ZZvqlXPF8rockoI0fkvClGaibs9yhsutSd0djZfstFIZaKKnHHbGEhVwg/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/wFG4yuGVVyIXRNsk6IC2jLyDAsGwuPeHjBhMTUFoXVia4GfLYj95Hxll0CTDNQicSso2ZKCA77kQgsVKjc8RDT6kItdEBD7GoNzbVXGYGfqaI/640?wx_fmt=jpeg&from=appmsg "undefined")  
  
**全时全域  安全无忧**  
  
  
