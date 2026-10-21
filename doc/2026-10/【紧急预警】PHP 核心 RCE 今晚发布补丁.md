#  【紧急预警】PHP 核心 RCE 今晚发布补丁  
原创 观未安全
                    观未安全  观未安全   2026-10-21 06:57  
  
![VisFuture](https://mmbiz.qpic.cn/mmbiz_png/ZlAYZ95xsqAupdzh2AFibGMSezlicKLo3IKk9TngrupZndAIcqsiaZeWGgmuZnIfe6SkIiapiac5xUCyxUd9dXfx5XicJSgX6Q6xkeKnvMbzK6DDY/640?wx_fmt=png&from=appmsg "")  
  
威 胁 情 报  
  
# 【紧急预警】PHP 核心 RCE 今晚发布补丁  
  
两个 CVSS 9.2 核心层漏洞 · 官方今日 22:00 发布更新，请立即排查 PHP 资产  
  
TL;DR  
- PHP 官方提前发布安全预告：今天（10-21）**北京时间 22:00（UTC 14:00）**  
发布紧急安全更新，修复 **两个 CVSS v4.0 9.2（Critical）的远程代码执行漏洞**  
。  
  
- 漏洞位于 **PHP 解释器核心层**  
，Apache mod_php 与 PHP-FPM 两种部署模式均可能受影响；流传编号 **CVE-2026-109618 / CVE-2026-109619**  
（以官方发布为准）。  
  
- 当前 **PoC 未公开、未见在野利用**  
，技术细节今晚随补丁公布；**PHP 8.1 及更早 EOL 版本无官方补丁**  
。  
  
- 处置：今晚前盘点全部 PHP 资产（CLI / Cron / 面板托管都算）；补丁后升级 8.2+ 并**重启进程**  
，容器需重建镜像。  
  
  
## 0x1 事件概述  
  
PHP 官方于昨日（10-20）发布**罕见的提前安全预告**  
：若漏洞细节提前公开或监测到在野利用，将**提前发布补丁**  
——官方把"提前发布"写进公告，本身就是风险等级极高的信号。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZlAYZ95xsqDcxLEb4MpRPTyYLOiat4QPanc9Og6KJRk4AkLvuosUJTaVHfoAzZdA8m3uqHqbH6tWWeg8BeTmmdYGA894ZRq55Vh1JMuK5K1g/640?wx_fmt=png&from=appmsg "")  
  
"It is scheduled for Wednesday, 21 October 2026 at 14:00 UTC. It addresses **two critical vulnerabilities**  
, both rated **CVSS v4.0 9.2**  
. … Full details, affected versions, and the fixed versions will be published with the release."  
## 0x2 漏洞概况  
  
1.**位于解释器/引擎核心层**  
，与 SAPI 运行架构无关——mod_php 与 PHP-FPM 均可能受影响，不存在"用 FPM 就豁免"；  
  
2.  
 泄露的利用演示自称覆盖 **PHP 5.4.0 及以上**  
，演示环境为最新的 PHP 8.5.4；对 WordPress、Laravel、Drupal 等 9 款主流应用演示命令执行（www-data 权限）；  
  
3.  
 流传编号 **CVE-2026-109618 / CVE-2026-109619**  
（编号以官方发布为准）；当前 PoC 未公开、未见在野利用；  
  
4.  
 官方仅对受支持分支（PHP 8.2+）提供修复，**PHP 8.1 及更早 EOL 版本无官方补丁**  
，需提前准备升级或隔离。  
## 0x3 处置指引  
  
▍补丁发布前（今晚 22:00 前）  
  
1.**盘点所有 PHP 资产**  
：不止 Web 站点，CLI 脚本、Cron 任务、队列 Worker、后台面板都要逐一确认版本；  
  
2.**公网暴露的 PHP 服务优先处理**  
；EOL 版本提前制定升级/隔离方案；  
  
3.  
 宝塔、Plesk 等面板环境，**确认站点实际走的 PHP handler**  
——系统 PHP 与面板托管 PHP 是两套，别漏。  
  
▍补丁发布后（数小时内完成）  
  
1.  
 对照官方公告的**受影响版本与触发条件**  
，核实自身是否存在攻击面；  
  
2.  
 从官方渠道升级到 8.2+ 修复版本，确认回滚方案；  
  
3.**升级后必须重启进程**  
：mod_php 重启 Apache，PHP-FPM 重启进程池，常驻 CLI 逐一起——不要只看 php -v  
；  
  
4.  
 容器环境需**重建镜像**  
并滚动更新，仅重启容器不生效。  
## 0x4 误区与传言  
  
1.disable_functions  
 / open_basedir**无效**  
：漏洞在解释器原生层，脚本层限制管不到；  
  
2.  
 封禁 SSH 端口 **无效**  
：攻击入口是 80/443 Web 端口；  
  
3.  
 别只依赖常规 WAF：新漏洞暂无成熟通用规则，WAF 仅能兜底。  
  
**① 流传信息要分清**  
：官方预告（两个 9.2、今晚 22:00 发布）= 事实；"影响 PHP 5 到 8 全版本""已被攻破" = 泄露演示与传言，**未获官方确认**  
——一切以今晚官方公告为准。  
  
## 参考  
  
· PHP 官方安全预告（2026-10-20）：php.net/archive/2026.php  
  
· watchTowr FAQ：watchtowr.com/intelligence/php-cve-2026-109618-faq  
  
· sansec 分析：sansec.io/research/php2shell  
  
· Huntress：huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities  
  
本文基于 PHP 官方公告与公开情报整理，补丁发布前的漏洞细节与受影响版本以官方公告为准。本文仅供安全防御参考。  
  
  
  
