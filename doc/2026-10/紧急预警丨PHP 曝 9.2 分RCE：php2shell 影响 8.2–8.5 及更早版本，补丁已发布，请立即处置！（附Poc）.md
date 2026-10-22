#  紧急预警丨PHP 曝 9.2 分RCE：php2shell 影响 8.2–8.5 及更早版本，补丁已发布，请立即处置！（附Poc）  
原创 蓝星安全
                    蓝星安全  蓝星安全   2026-10-22 03:22  
  
         
**点击上方****蓝星安全****关注我**  
  
****  
  
**免责声明：本公众号分享的任何资料仅限用于安全学习，严禁用于其他用途，请严格遵守中华人民共和国法律法规，对因不遵守国家法律法规而产生的任何后果，均由个人自行承担，本公众号不承担任何责任！**  
  
****  
**获取资料，请扫码下方二维码加入知识星球**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmxg12rU33pwcICNRkiaof5YSUAGfWPApU7M1BfdsTTOyvREw0gKD42g4U8eefG4n0XuYETEtbxSIN2drACnNVz3UeicCcldM5AA/640?wx_fmt=png&from=appmsg "")  
  
     
 2026 年 10 月 21 日，PHP 官方发布紧急安全更新，修复两个 **CVSS v4.0 9.2 分**  
的预认证远程代码执行漏洞。攻击者不需要任何账号，只需构造普通的 HTTP multipart 请求，就有机会在**应用代码执行之前**  
拿下服务器。这两个漏洞被社区统称为 **php2shell**  
，编号为 **CVE-2026-109618**  
 与 **CVE-2026-109619**  
。目前官方补丁、技术公告与公开复现仓库都已出现。  
  
    蓝星安全内部漏洞库已收录该漏洞POC和EXP，加入蓝星安全知识星球即可解锁（已收录7000+漏洞）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcm5KOye7uw51sKpucLw7PZwvHoMRmgQ5ylYiaHlINFjZsfPE0tmQWh7ZpQWXOqIfYy5A7CLDhfoWQNppECbvCdMg6X1cQicAAhz4/640?wx_fmt=png&from=appmsg "")  
## 一、事件速览  
<table><thead><tr style="corner-shape: superellipse(1.5);"><th data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">项目</span></span></section></th><th style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">内容</span></span></section></th></tr></thead><tbody><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">漏洞编号</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">CVE-2026-109618（OOM 内存破坏）、CVE-2026-109619（全局状态残留）</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">漏洞类型</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">Use-After-Free 或内存破坏，可导致远程代码执行</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">风险评分</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">CVSS v4.0 </span></span><strong style="corner-shape: superellipse(1.5);"><span leaf=""><span textstyle="" style="font-size: 16px;">9.2（Critical）</span></span></strong><span leaf=""><span textstyle="" style="font-size: 16px;">，两个漏洞均为 9.2</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">认证要求</span></span></section></td><td style="corner-shape: superellipse(1.5);"><strong style="corner-shape: superellipse(1.5);"><span leaf=""><span textstyle="" style="font-size: 16px;">无需认证</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 16px;">（Pre-Auth）</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">影响版本</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">PHP 8.2 低于 8.2.35、8.3 低于 8.3.36、8.4 低于 8.4.27、8.5 低于 8.5.12；8.1 及以下已停止维护</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">修复版本</span></span></section></td><td style="corner-shape: superellipse(1.5);"><strong style="corner-shape: superellipse(1.5);"><span leaf=""><span textstyle="" style="font-size: 16px;">8.2.35 / 8.3.36 / 8.4.27 / 8.5.12</span></span></strong></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">触发入口</span></span></section></td><td style="corner-shape: superellipse(1.5);"><code style="corner-shape: superellipse(1.5);text-autospace: no-autospace;"><span leaf=""><span textstyle="" style="font-size: 16px;">session.upload_progress</span></span></code><section><span leaf=""><span textstyle="" style="font-size: 16px;">，随 multipart 请求在请求初始化阶段触发</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">高危场景</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">Apache + </span></span><code style="corner-shape: superellipse(1.5);text-autospace: no-autospace;"><span leaf=""><span textstyle="" style="font-size: 16px;">mod_php</span></span></code><span leaf=""><span textstyle="" style="font-size: 16px;">（已演示预认证 RCE）；PHP-FPM 在特定条件下同样可被利用</span></span></section></td></tr><tr style="corner-shape: superellipse(1.5);"><td data-colwidth="111" style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">当前状态</span></span></section></td><td style="corner-shape: superellipse(1.5);"><section><span leaf=""><span textstyle="" style="font-size: 16px;">官方已修复；技术细节公开；复现仓库与 PoC 已扩散</span></span></section></td></tr></tbody></table>  
**处置优先级：P0。互联网暴露的 PHP 服务，建议 24 小时内完成升级并重启 worker。**  
## 二、两个漏洞，同一条攻击链  
  
这两个 CVE 根因不同，但都涉及 PHP 的**请求生命周期管理**  
。  
### CVE-2026-109618：OOM 时的内存破坏  
  
PHP 用 memory_limit  
 限制单请求内存，超限时通过 longjmp()  
 中止请求。问题在于：如果内存耗尽发生在内部请求结构**只初始化了一半**  
的时候，提前退出会把这些结构留在非法状态；后续清理逻辑却把它们当作合法数据继续释放、析构，攻击者就可以借此控制内存。  
  
公开复现中，攻击者用大量表单字段制造 OOM，使 session  
 相关指针悬空，再在下一个请求里重新占用这段内存。  
### CVE-2026-109619：跨请求的“脏状态”  
  
某个请求持有的内部状态，在内存释放后仍被常驻 worker 引用。下一个请求如果落到**同一个 worker**  
，就可能在这份“脏状态”被正常初始化覆盖之前访问到它，形成 Use-After-Free。  
  
已确认的路径包括 session.upload_progress  
；在 PHP 8.5 上还包括致命错误回溯状态。致命错误、执行超时、初始化前产生的警告或非法 INI 指令，都可能成为攻击链中的一环。  
### 为什么叫“预认证”  
  
session.upload_progress  
 的 multipart 回调（rfc1867）发生在 **RINIT 之前**  
，也就是在 PHP 脚本被编译执行之前。攻击流量只需要一个 multipart POST，目标页面哪怕只有一行 <?php echo "OK"; ?>  
，也可能受到影响。  
  
这也是官方把 **mod_php 列为最高风险**  
的原因：一个未认证请求就能在应用代码运行前触发漏洞。  
  
需要说明的是，公开  
复现链路是在 php:8.4.26-apache  
 官方镜像上跑通的，演示条件包括较小的 memory_limit  
、display_errors=1  
、单 worker prefork 等。这不代表所有 PHP 站点都能被同一条链“一键打穿”，但漏洞本身不需要认证，攻击面足够大，因此不能据此降低处置优先级。  
## 三、影响范围：不要只看版本号  
  
官方明确：**所有处理 Web 请求的 PHP 运行时都受影响**  
，包括：  
- Apache + mod_php  
（风险最高）  
  
- PHP-FPM（已验证 session 上传进度路径可利用）  
  
- php-cgi、FrankenPHP  
  
- LiteSpeed / LSAPI（可能需要应用侧条件）  
  
- 容器镜像、虚拟机模板、托管面板、一体机、第三方产品中**自带的 PHP**  
  
几个容易漏掉的点：  
1. **只升级 WordPress / Drupal / Joomla 没用**  
。漏洞在 PHP 运行时，不在应用；应用升级了、PHP 没升级，依然存在风险。  
  
1. **CLI 版本不等于 Web 版本**  
。php -v  
 看到的是命令行解释器，Web 服务可能加载的是另一个版本或另一个 SAPI。  
  
1. **发行版会回移补丁**  
。版本号没变，不代表没修；需要查看厂商公告、软件包 changelog 与构建号，确认**两个 CVE 都已修复**  
。  
  
1. **PHP 8.1 及以下已停止维护**  
。官方破例提供了 7.4 / 8.0 / 8.1 的最小源码回移，但这不等于恢复支持；能升级就升级。  
  
1. **PHP 8.2 将于 2026 年 12 月 31 日 EOL**  
。这次修完，也建议把升级到 8.3/8.4 排进计划。  
  
公开测绘数据可以说明暴露面有多大：VirusTotal 识别到约 **510 万个**  
 IP 运行 PHP，其中约 **380 万**  
当前至少有一个开放端口；在愿意暴露版本号的约 110 万台中，只有约 **27.2 万**  
运行在受支持的 8.2–8.5 版本，约 **75.8 万**  
仍停留在 5.4–8.1 的 EOL 分支。注意，这是“互联网 PHP 资产面”，不等于“确认可利用主机”，但足以说明排查工作量。  
## 四、修复：升级、重启、验证  
  
**唯一完整的修复方式，是升级到修复版本。**  
  
PHP 8.5.x → 8.5.12  
  
PHP 8.4.x → 8.4.27  
  
PHP 8.3.x → 8.3.36  
  
PHP 8.2.x → 8.2.35  
  
操作要点：  
- 优先使用**发行版/厂商的安全包**  
；rpms.remirepo.net  
 在发布当天不会同步，依赖它的环境要提前切换方案或从源码构建。  
  
- 升级后**必须重启所有 PHP worker**  
（php-fpm、Apache、FrankenPHP 等）。只替换磁盘文件、不重启，旧 worker 仍可能继续在漏洞状态下服务。  
  
- 容器环境要**重建镜像并滚动重启**  
，不要只在运行中的容器里替换文件。  
  
- 升级后逐项确认：Web 实际加载的 PHP 版本、SAPI、补丁包版本、worker 启动时间。  
  
- 共享托管、多租户环境要确认**每一个池、每一个版本**  
都升级，而不是只升级默认池。  
  
## 五、临时缓解：只能降低风险，不能替代补丁  
  
如果确实无法立刻升级，官方给出了一些**按 CVE 区分**  
的缓解措施。注意：它们只阻断特定路径，**没有任何一条能让未修复的 PHP 变安全**  
。  
### 针对 CVE-2026-109618（OOM 内存破坏）  
- 以管理员强制方式设置 memory_limit=-1  
（如 FPM pool 的 php_admin_value  
、Apache 配置），并确认没有 pool、目录或应用把它改回有限值。代价是失去单请求内存上限，需配套容器/cgroup 限额。  
  
- 调低 post_max_size  
：不需要上传的业务设 8M  
，需要上传的设 32M  
，并保持 upload_max_filesize  
 小于它。  
  
- 在 Web 服务器/代理层限制**解码后**  
的请求体大小、字段数、multipart part 数、header 数与嵌套深度，且要覆盖**没有有效 Content-Length**  
 的请求。  
  
- 用容器/cgroup 内存限制作为可用性边界，策略上只终止受影响的 worker；**不要**  
用 RLIMIT_AS  
 / RLIMIT_DATA  
 代替，它们可能反过来触发漏洞路径。  
  
### 针对 CVE-2026-109619（状态残留）  
- 设置 session.upload_progress.enabled=0  
，并**重启所有 worker**  
。必须写在 php.ini  
、FPM pool 或服务端配置里，在请求解析前生效；ini_set()  
 和 .user.ini  
 不算。  
  
- PHP 8.5 上设置 fatal_error_backtraces=0  
，同样重启 worker。它只阻断已确认的致命错误回溯路径。  
  
- 清理会在请求初始化前产生诊断信息的非法/弃用 INI 指令；用 error_reporting  
、display_errors  
 或 @  
 隐藏报错**没有用**  
。  
  
- 修复攻击者可达的致命错误与执行超时，包括回调、上传/导入处理、析构与清理阶段的问题。  
  
另外两点必须说清楚：  
- **WAF 和登录认证不是修复**  
。漏洞可能在应用处理请求之前触发，WAF 规则最多提高门槛。  
  
- 如果业务暂时无法下线也无法升级，**先减少互联网暴露面**  
（VPN、IP 白名单、临时摘流），比继续暴露并等待补丁更稳妥。  
  
## 六、排查建议：重点看这几类痕迹  
  
官方公告没有给出通用的流量 IOC，但下列迹象值得重点排查：  
- 异常的 multipart POST：字段数量巨大、part/header 异常多、嵌套很深、请求体大小与 Content-Length  
 不匹配或缺失；  
  
- 请求中出现 PHP_SESSION_UPLOAD_PROGRESS  
，尤其是发往本不接受上传的接口；  
  
- PHP/Apache worker 异常崩溃、频繁重启、OOM、OOM killer 活动，并与可疑 HTTP 流量时间相关；  
  
- PHP 致命错误、执行超时、启动期诊断、扩展清理阶段告警；  
  
- Web 进程拉起异常子进程、web 目录被写入陌生文件、计划任务被改、异常外连。  
  
发现疑似痕迹时，先**保留 Web/PHP/应用/主机日志与内存转储**  
，再隔离主机；不要直接重启了事，重启会破坏关键现场。  
## 七、时间线  
- **10 月 20 日**  
：PHP 官方预告紧急安全版本，两个漏洞均为 CVSS 9.2；细节在发布前数小时已开始泄露。  
  
- **10 月 21 日 11:11 UTC**  
：公开复现仓库上线，给出纯 HTTP 绕过 ASLR 的完整利用链（PHP 8.4.26 + Apache prefork + mod_php）。  
  
- **10 月 21 日 13:30 UTC**  
：PHP 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 发布，官方公告与 GHSA 同步公开。  
  
- **10 月 21 日—22 日**  
：CERT-EU、CIRCL 等发布预警；PoC、复现仓库与检测规则快速扩散。  
  
**利用代码比补丁更早公开**  
，这是本次事件最危险的信号之一。  
## 八、给运维/安全的行动清单  
- 盘点所有 Web 侧 PHP 运行时（含容器、面板、一体机、托管服务）  
  
- 确认实际 SAPI 与版本，区分 CLI 与 Web  
  
- 升级到 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12，或厂商等效补丁  
  
- 重启全部 PHP worker 与 Web 服务，确认新进程生效  
  
- 无法升级的，按 CVE 分别落实缓解，并减少公网暴露  
  
- 检查 10 月 20 日以来的异常 multipart 流量、OOM 与 worker 崩溃  
  
- 排查 web 目录与计划任务，确认无 webshell 落地  
  
- 把 PHP 8.2 的 EOL 升级排期提上日程  
  
##   
  
