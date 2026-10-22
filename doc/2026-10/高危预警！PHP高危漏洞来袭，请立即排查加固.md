#  高危预警！PHP高危漏洞来袭，请立即排查加固  
 骇极安全   2026-10-22 10:01  
  
![](https://mmecoa.qpic.cn/mmecoa_jpg/KEvyRkMibLKib0FESNXiaztnthDtAcLa26OfwQS0uq2OAJ864hHjAd1b6Zso7JHLYAdFdaBV58PCG9IMH09y9icaNQ1QAN7s65tcPtGPgLVQoicw/640?wx_fmt=jpeg&from=appmsg "")  
  
10 月 21 日，  
 PHP 语言被曝存在两个可导致远程代码执行的高危漏洞（CVE-2026-109618、CVE-2026-109619），安全社区将其统一命名为 PHP2Shell。  
  
PHP 官方已于北京时间当日晚间发布 8.5.12、8.4.27、8.3.36 和 8.2.35 四个维护版本完成修复，并建议所有用户尽快升级。此缺陷波及多个主流 CMS 及政府网站、企业业务系统、电商等各类互联网平台，在互联网应用基础设施中影响范围极大，CVSS v4.0 评分为 9.2（严重）。  
  
从根因看，两处缺陷均位于  
 PHP 请求生命周期的状态管理边界：内存耗尽后，尚未完成初始化或已被释放的请求内存仍会在请求关闭或后续请求中被错误消费，形成堆内存释放后重用（UAF）与信息泄露。未经身份验证的攻击者只需构造恶意 HTTP 请求即可触发，进而以 Web Worker权限执行任意命令。PHP 在政府网站、企业业务系统及互联网平台中部署极为广泛，加之大量已停止维护的历史版本仍在运行，本次漏洞的暴露面与修复难度均较为突出。目前漏洞技术细节与 PoC 已公开，并已出现在野利用迹象，建议受影响用户立即排查资产  
、完成升级，并在升级后重启全部  
 PHP 服务进程。  
  
值得关注的是，此次漏洞涉及  
PHP底层内存处理机制，影响多个主流版本及常见Web运行环境，大量已停止官方维护的历史版本进一步增加了修复难度。同时，AI技术在此次漏洞发现与修复协调过程中发挥了重要作用，也反映出AI正在加速改变网络安全攻防态势。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/KEvyRkMibLKibOaa8tX1tJNmoBPdrHJBYzIB8fhKX3UKUCXQLgVVkQfpt62lQuNOkEQGe8rOy5iaGServnvRjbLe7XQRrJV2u7JPJibZFzNcDLc/640?wx_fmt=png&from=appmsg "")  
# 漏洞详情  
## 影响组件  
  
PHP 是一种广泛使用的开源服务器端脚本语言，主要运行于 Web 服务器端。W3Techs 统计显示，全球约四分之三的网站使用 PHP 作为服务器端编程语言，WordPress、Magento、phpMyAdmin、Nextcloud、Laravel等主流应用均依赖 PHP  
。  
## 漏洞描述  
  
两个漏洞的根因都位于  
 PHP 请求生命周期的状态管理边界——当请求因内存耗尽或致命错误被提前中止时，本应随请求一同销毁的内部状态（请求内存、全局状态、会话数据）没有被正确清理，却被后续流程或同一 worker 进程的下一个请求当作有效数据继续消费。未经身份验证的攻击者只需发送一个构造过的 HTTP 请求，就可能以 Web Worker 的权限执行任意命令。  
## 利用条件  
  
1.目标使用受影响版本 PHP，运行于`mod_php`或`php  
‑  
fpm`模式；  
  
2.PHP 配置开启`memory_limit`限制（绝大多数生产环境默认开启）；  
  
3.攻击者可以向目标 PHP 服务发送 HTTP 请求，无需账号密码、无需身份认证，不需要登录后台；  
  
4.无特殊客户端条件，普通 HTTP 请求即可完成攻击，支持公网、内网环境触发。  
## 影响版本  
<table><tbody><tr style="height:20.75pt;"><td data-colwidth="187" width="187" style="height: 20.75pt;"><section><span leaf="">PHP</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">分支</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">受影响版本</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">✅</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">已修复版本</span></section></td></tr><tr style="height:18.25pt;"><td data-colwidth="187" width="187" style="height: 18.25pt;"><section><span leaf="">PHP 8.2</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">低于</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">8.2.35</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">8.2.35</span></section></td></tr><tr style="height:18.25pt;"><td data-colwidth="187" width="187" style="height: 18.25pt;"><section><span leaf="">PHP 8.3</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">低于</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">8.3.36</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">8.3.36</span></section></td></tr><tr style="height:18.25pt;"><td data-colwidth="187" width="187" style="height: 18.25pt;"><section><span leaf="">PHP 8.4</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">低于</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">8.4.27</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">8.4.27</span></section></td></tr><tr style="height:18.25pt;"><td data-colwidth="187" width="187" style="height: 18.25pt;"><section><span leaf="">PHP 8.5</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">低于</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">8.5.12</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">8.5.12</span></section></td></tr><tr style="height:53.25pt;"><td data-colwidth="187" width="187" style="height: 53.25pt;"><section><span leaf="">PHP 8.1</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">及以下</span><span style="mso-spacerun:yes;"><span leaf=""> </span></span><span leaf="">/ 5.x</span></section></td><td data-colwidth="198" width="198"><section><span leaf="">全部受影响（已停止官方维护）</span></section></td><td data-colwidth="212" width="212"><section><span leaf="">需联系厂商移植补丁或升级</span></section></td></tr></tbody></table>##   
  
只要你的版本低于上表的  
"已修复版本"，就是在受影响范围内。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/KEvyRkMibLK8KiakkNw4J8J8JKL0JM9ziaWKiapQQVKCw2vnAuOBquJzTQATaYsxA5Dib0TPqZtS3ibYB97WqibwDNWRGevOuypqGA7IMaCQyQMz3o/640?wx_fmt=png&from=appmsg "")  
## 修复方式  
  
官方已发布安全补丁，请及时更新至最新版本：  
  
PHP 8.5.12  
  
PHP 8.4.27  
  
PHP 8.3.36  
  
PHP 8.2.35  
  
下载地址：  
  
https://www.php.net/downloads.php  
  
1. 升级后必须重启 Apache、PHP-FPM 及所有常驻 worker，确保旧进程不再处理请求。  
  
2. 容器环境应重新构建镜像，避免沿用包含旧 PHP 的 layer；Kubernetes 环境应滚动重启所有 Pod，并确认没有旧 ReplicaSet 继续服务。  
  
3. 发行版仓库（Ubuntu、Debian、RHEL、Alpine 等）和集成环境（宝塔、phpStudy、XAMPP、Docker 官方镜像等）可能存在同步延迟，请以 `php -v`、`php-fpm -v` 的实际输出为准，不要仅核对包管理器版本号。  
  
4. PHP 8.1 及更早版本已停止支持，官方不再提供补丁，建议迁移至 PHP 8.2 或更高版本；若使用发行版长期支持分支，请以厂商 backport 公告为准。  
  
5. 共享主机和虚拟主机用户应联系服务商确认 PHP 版本。  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/KEvyRkMibLKibbs17VeUqNAXqokQo37QicvcFDWehicx5Cp1e8cE8TBibADMFcIhCfZbK70uBLeSWxdrhq3nckAyJQs4yBXOFUZGxF3ezcttqsts/640?wx_fmt=png&from=appmsg "")  
# 安全建议  
  
最近漏洞披露节奏快，高危  
 RCE 漏洞利用链传播速度极快，很多攻击者会在补丁放出后快速开发利用工具，建议各单位建立常态化资产台账，重点跟踪 PHP、Nginx、CMS 这类高频组件安全公告。  
  
1.资产排查  
：对内网、公网全部  
 PHP 业务资产进行版本梳理，优先处置公网暴露业务；可执行php -v、phpfpm -v快速核对真实运行版本；  
  
2.升级与验证  
：升级前做好服务器快照、业务备份，升级完成重启服务，做业务功能回归测试；  
  
3.入侵自查  
：完成升级后，建议开展安全自查：检查  
 Web 目录新增可疑 PHP 脚本 / WebShell、异常定时任务、新增系统账号、SSH 异常公钥、对外恶意外联；  
  
4.持续监控  
：  
Web 访问日志、系统安全审计日志保留足够时长，配置告警规则，及时发现异常攻击行为；  
  
5.老旧版本处置  
：对于已经停止维护  
 PHP 8.1 及更早环境，尽快规划版本迁移，如暂时无法迁移，评估使用发行版厂商后向补丁或者隔离网络访问，收缩访问来源。  
  
  
