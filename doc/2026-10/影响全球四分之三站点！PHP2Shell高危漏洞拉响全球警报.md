#  影响全球四分之三站点！PHP2Shell高危漏洞拉响全球警报  
网络安全研究院
                    网络安全研究院  亚信安全   2026-10-22 03:56  
  
10 月 21 日，亚信网络安全研究院监测到 PHP 语言被曝存在两个可导致远程代码执行的高危漏洞（CVE-2026-109618、CVE-2026-109619），安全社区将其统一命名为 PHP2Shell。  
  
  
**PHP 官方已于北京时间当日晚间发布 8.5.12、8.4.27、8.3.36 和 8.2.35 四个维护版本完成修复**  
，  
并建议所有用户尽快升级。此缺陷波及多个主流 CMS 及政府网站、企业业务系统、电商等各类互联网平台，在互联网应用基础设施中影响范围极大，**CVSS v4.0 评分为 9.2（严重）**  
。  
  
  
从根因看，两处缺陷均位于 PHP 请求生命周期的状态管理边界：内存耗尽后，尚未完成初始化或已被释放的请求内存仍会在请求关闭或后续请求中被错误消费，形成堆内存释放后重用（UAF）与信息泄露。未经身份验证的攻击者只需构造恶意 HTTP 请求即可触发，进而以 Web Worker权限执行任意命令。PHP 在政府网站、企业业务系统及互联网平台中部署极为广泛，加之大量已停止维护的历史版本仍在运行，本次漏洞的暴露面与修复难度均较为突出。目前漏洞技术细节与 PoC 已公开，并已出现在野利用迹象，建议受影响用户立即排查资产、完成升级，并在升级后重启全部 PHP 服务进程。  
  
  
**漏洞概述**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/YUnWCyLjbul3kp9VAHlmmtibS2gnKiaUZnjqBtuUcoWUR5gDeeD89CQfQzgYEK9hb9EMMNPWlMj1zUUHibswkLzrqR0xUlG9VL6qrbagWDZF30/640?wx_fmt=png "")  
  
  
综合处置优先级为**紧急**  
。漏洞无需认证、远程可直接触发，影响多个PHP版本及CMS、政府网站、企业业务系统、电商等各类互联网平台。目前漏洞技术细节与 PoC 已公开，并已出现在野利用迹象，建议受影响用户立即排查资产、完成升级，并在升级后重启全部 PHP 服务进程。  
  
  
**漏洞复现**  
  
  
目前，亚信网络安全研究院已成功复现该漏洞，截图如下：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/YUnWCyLjbumllbID8bQDS9Lco9T6Kvw6WQsDFKVOFaC8HiczoP5TdcJRs3YAHpY6V9TuXZL9hdrEdPnObSzuXh7cNgpgEXON7fTXnIqiccicrE/640?wx_fmt=jpeg "")  
  
  
**漏洞详情**  
  
  
PHP 是全球使用最广泛的服务端脚本语言之一，支持 Apache mod_php、PHP-FPM、FrankenPHP 等多种部署方式。据 W3Techs 统计，全球约四分之三的站点由 PHP 驱动，广泛应用于 WordPress、Magento、Shopware、WooCommerce、Drupal、Joomla 等主流 CMS 及各类电商、政务系统等互联网平台，在互联网应用基础设施中占有重要地位。  
  
  
**漏洞细节**  
  
**CVE-2026-109618：**  
PHP 在处理内存耗尽（OOM）错误时，会通过 longjmp() 提前退出当前上下文，请求内存在尚未完成初始化时仍可被访问。攻击者可构造包含大量header、嵌套字段名或空文件控件的 multipart/form-data 请求，在不超过 post_max_size 限制的情况下耗尽 memory_limit 并触发 OOM。对于 Apache mod_php 等无条件调用请求关闭流程的 SAPI，关闭阶段会处理这些无效状态，最终造成内存破坏与远程代码执行。  
  
  
**CVE-2026-109619：**  
在内存错误发生后，PHP 工作进程仍会保留 error、session、object、extension 等全局状态，其中部分指针指向已释放的请求内存。若请求结束时未能彻底重置这些状态，后续请求在完成初始化之前就可能访问到残留数据，形成跨请求的释放后重用（UAF）。攻击者可在同一 worker 上先后构造两个请求，分别充当“生产者”与“消费者”：前者留下未清理的状态，后者触发对残留状态的访问，最终在无需认证的情况下造成敏感内存泄露，并可进一步实现远程代码执行。  
  
  
**影响范围**  
  
所有对外提供 Web 请求处理能力的 PHP 安装均受影响，与上层应用框架无关。各 PHP 分支的受影响版本与修复版本如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/YUnWCyLjbukLjkiaQLO2cdTLGBhrEHrX5ZoGpf8HwNlFkBkEDzN9DbvHWD1s0nEwHyuREOW6ZKRu8F4Micm4Dag8ic1jias4ZbJUK4FRKPuLsAY/640?wx_fmt=png "")  
  
  
  
  
  
  
  
  
  
  
**其他受影响组件：**  
以 PHP 为运行时的应用、框架与建站系统可能间接受影响，包括 WordPress、MediaWiki、phpBB、Magento、Laravel、Symfony、Drupal 等；部署形态方面，mod_php、PHP-FPM（含 Docker PHP-FPM 镜像）等均需纳入排查。  
  
  
**修复建议**  
  
  
**官方补丁**  
  
PHP 安全团队已发布安全版本修复上述漏洞，具体通告链接：https://www.php.net/archive/2026.php#2026-10-21-5https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79whttps://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
  
**请及时更新升级至最新版本： PHP 8.5.12、8.4.27、8.3.36 或 8.2.35。**  
  
  
**说明：**  
部分 Linux 发行版可能以原版本号携带回移补丁，请以发行版 changelog 和安全公告为准。升级后需重启 PHP-FPM、Apache 及全部常驻 worker，否则旧进程仍运行旧代码。升级后应复核 memory_limit、post_max_size 等配置。新版以“软 OOM + 硬 OOM _exit()”替代 longjmp()，并新增硬内存上限，相关配置需相应调整。  
  
  
**已停止维护的分支：**  
PHP 8.1、8.0、7.4 及更早版本，以及 PHP 7.3、5.x 等历史版本，官方均已停止支持且不再发布修复版本，相关环境应优先安排升级；短期内无法升级的，应实施网络隔离并限制来源 IP。  
  
  
**临时缓解措施**  
  
**官方："没有完整的缓解措施，升级是唯一安全选择。"**  
  
  
- **CVE-2026-109618：**  
管理端强制设置 memory_limit=-1；post_max_size 设置为 8M（无上传业务）或 32M；在前端限制请求体大小、字段数、part 数、header 数与嵌套深度；不要使用 RLIMIT_AS 或 RLIMIT_DATA 进行限制。  
  
- **CVE-2026-109619：**  
设置 session.upload_progress.enabled=0，并重启全部 worker；PHP 8.5 环境同时设置 fatal_error_backtraces=0。  
  
- **其他措施：**  
可参考关闭 opcache.enable（第三方建议，非完整缓解）；WAF 虚拟补丁仅能争取处置时间；建议监控 php-fpm、php-cgi、apache2 进程是否异常生成 sh、bash、curl、wget 或 /dev/tcp/ 子进程。  
  
- **EOL 环境：**  
无法升级的 8.1 及更早环境应实施网络隔离，限制来源 IP，并优先安排迁移。  
  
  
  
  
亚信解决方案  
  
  
**亚信已支持该漏洞的检测**  
  
目前亚信安全规则研发团队已第一时间新增了流量(怒狮)检测规则，支持CVE-2026-109618/CVE-2026-109619漏洞的检测，请及时更新TDA产品的特征库到最新版本nsd$3000.293。规则编号：106110452，规则名称：PHP2Shell 远程代码执行漏洞(CVE-2026-109618/CVE-2026-109619)。  
  
  
**更新方式如下：**  
  
TDA产品在线更新方法：登录系统-》系统管理-》系统升级-》特征码更新。  
  
  
**亚信星海EASM已同步支持PHP漏洞从互联网侧的暴露面监测**  
  
目前，星海外部攻击面管理平台从互联网侧持续发现使用 PHP 的对外资产并识别版本分布，建议已部署星海EASM的客户及时更新监测策略，登录平台即可查看受影响资产清单。  
  
  
**总结**  
  
  
本次披露的 CVE-2026-109618 与 CVE-2026-109619 是无需认证、可远程触发的严重级别代码执行漏洞，在互联网应用基础设施中影响范围极大。PHP官方已正式发布更新补丁，建议各相关单位及用户立即完成资产盘点与升级，升级后重启全部 PHP 进程，EOL 环境应同步推进源码回填或迁移，并在过渡期落实临时缓解与监控。  
  
  
**往期推荐**  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633497&idx=1&sn=f3949a64ebbec0e5b28a60abf43c29ed&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633452&idx=1&sn=6750e2d034f2a571d34433bc8d7bae3f&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633415&idx=1&sn=a594dab138e6f21d36153f1295d3c752&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633330&idx=1&sn=a8dfe939b299b22b8dcbcc4b8c651864&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633340&idx=1&sn=34b40a3c44c85b193c75c566d24bd2f8&scene=21#wechat_redirect)  
  
  
了解亚信安全，请点击  
**“阅读原文”**  
  
  
**求点赞**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoG8KmicxOYyR9Em8f5BFRia2jia66l1HibEyCKXqUq6bGLUCj7uDtZS58pg/640?wx_fmt=gif "")  
  
  
**求分享**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoHFSQRwicdBWfDiaNibTtUyQ2lPiaicDV5pZaUgZTzY3TJQ3ZbmR1Tj5iciaYg/640?wx_fmt=gif "")  
  
  
**求喜欢**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoAV9dDSmbcAxOs8iaCgcpDEBjGNObDFpgTXjZSyjr9DzTgMblPrUYILA/640?wx_fmt=gif "")  
  
  
