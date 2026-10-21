#  PHP 双洞预警：两个 9.2 分 RCE 细节已泄露，补丁今晚 22 点发布  
原创 Kratos
                    Kratos  Kratos Sec   2026-10-21 11:00  
  
PHP 安全团队上周一反常态发了一条预告：两个严重漏洞，双双 CVSS v4.0 9.2，紧急安全 release 定在北京时间 10 月 21 日（今天）22:00。按惯例这种事都是补丁和公告同时落地，这次却提前 24 小时打招呼，让所有用户"现在就开始准备"——因为这个洞的细节在前一天已经泄露了。原定周四的披露计划被打乱，watchTowr 确认漏洞真实且已复现其中一个，一张名为 PHP2Shell 的利用工具截图声称能打 5.4.0 起的所有 PHP 版本。截稿时补丁还有几个小时才发布，在野利用尚未出现——这几个小时就是全部的准备窗口。  
## 0x0 概述  
  
<table><thead><tr><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">项目</span></section></th><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CVE 编号</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CVE-2026-109618 / CVE-2026-109619</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">漏洞类型</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">远程代码执行（RCE，PHP 核心层面）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">评分</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">两个均为 CVSS v4.0 9.2（官方评定）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">影响范围</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">未经官方确认；泄露方与安全厂商称 PHP 5.4+ 的 Web 服务器全部在射程</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">补丁发布</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">2026-10-21 14:00 UTC（北京时间今晚 22:00）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">预期修复版本</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">8.5.12 / 8.4.27 / 8.3.36 / 8.2.35（社区消息，以 php.net 官方公告为准）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">在野利用</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">截稿未确认（Huntress 称其客户环境未见异常）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">官方维护范围</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">仅 8.2–8.5；≤8.1 为 EOL，无官方补丁</span></section></td></tr></tbody></table>  
  
两个细节先划重点：第一，这是 PHP **语言本体**  
的洞，不是某个框架或 CMS 的洞，WordPress、Drupal、Laravel、Magento、内网老系统——所有跑在 PHP 上的东西共用这一个底层；第二，官方原文写明保留"提前发布"的权利（细节公开或出现利用即触发），所以你看到这篇文章时补丁可能已经落地，版本号以 php.net 为准。  
## 0x1 事件时间线：一次被打乱的协调披露  
- **10 月 20 日**  
：php.net 首页挂出"Upcoming Critical PHP Security Release"预告，罕见地提前一天通知全网准备升级；  
  
- **同日**  
：Hacker News 出现帖子，声称两个漏洞即将公开披露。帖子本身很快被删，但截图已在社交网络传开；  
  
- **watchTowr 随即确认**  
：漏洞真实存在，并成功复现了 CVE-2026-109618；  
  
- **泄露的工具截图**  
：一个名为 PHP2Shell 的利用工具（署名 Calif.IO），声称对 5.4.0 起的每一个 PHP 发行版有效；  
  
- **原定披露时间是周四**  
，泄密后 PHP 安全团队把 release 提前到周三 14:00 UTC；  
  
- **CERT-EU 发布 2026-018 公告**  
，SlowMist 等威胁情报渠道跟进转发，国内多家预警平台已开始推送。  
  
这是典型的"零日倒计时"局面：补丁 diff 公开的那一刻，攻击者同时拿到漏洞原理。Sansec 的预判很不客气——**发布后 15 分钟内就会出现大规模攻击**  
。历史上这类 PHP 核心洞的武器化速度（参照 CVE-2024-4577 当天出现全网扫描）证明这不是危言耸听。  
## 0x2 确认的与未确认的  
  
预警文章最忌把传闻当结论，分开列。  
  
**已确认：**  
- 两个漏洞编号、CVSS v4.0 9.2 评分、今天 14:00 UTC 的发布时间（php.net 官方原文）；  
  
- watchTowr 独立复现了 CVE-2026-109618；  
  
- 漏洞位于 PHP 本体，利用发生在应用代码执行**之前**  
——Sansec 明确提醒：WAF 规则和应用层防护（包括他们自家 Shield）不能作为依赖，修复只能在基础设施层完成，即升级 PHP 本身。  
  
**未确认，但值得知道：**  
- **影响版本"5.4+"是泄露方说法**  
，官方原文写的是"受影响版本随补丁一同公布"。Sansec 独立给出了"所有 PHP 5.4 及以上的 Web 服务器"的口径，两个来源一致，但严格说还不是官方数字；  
  
- **根因未公开。**  
 Sansec 页面的元数据关键词里出现了 session.upload_progress.enabled  
、session.use_strict_mode  
、php-fpm  
——如果属实，指向 session 上传进度和 FPM 相关的路径，但这只是页面关键词，不是分析结论，别据此做定点缓解，等 diff；  
  
- **修复版本号**  
 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 来自社区讨论和 git 动向，最终以官方公告为准。  
  
## 0x3 真正的坏消息：EOL 版本没有补丁  
  
PHP 官方只维护 8.2–8.5 四个分支。这意味着：  
- **8.1 及更早版本（含整个 7.x、5.x）不会有官方修复**  
。而泄露工具声称的打击面恰恰是"5.4.0 起的全部版本"——如果属实，EOL 存量就是这个事件里最大的暴露面。现网跑 7.4、5.6 的系统比很多人想象的多，内网 OA、老电商、十年没动的营销页都在其列；  
  
- **8.2 分支本身将于 2026 年 12 月 31 日 EOL**  
，这大概率是它最后几次安全更新，还在 8.2 的应该把升级 8.3+ 排上日程，而不是补完这个洞就完事；  
  
- 用 8.1 以下版本的，今晚之后的选择只有三条：找发行版厂商要 backport（RHEL/Ubuntu LTS 通常会对 PHP 打补丁，注意区分"发行版维护"和"上游维护"）、业务上 WAF/网络隔离顶住暴露面、或者干脆趁机把版本升了。  
  
托管环境（云虚拟主机、Managed WordPress、Pantheon 这类平台）用户自己动不了 PHP，现在就去问服务商"今晚的 PHP 安全更新你们几点推"——这类平台通常会在几小时内自动滚动，但你要确认，不要默认。  
## 0x4 发布前的准备清单  
  
到今晚 22:00 之前，按优先级做四件事。  
  
**第一，盘点资产。**  
 把手里所有 PHP 暴露面列出来：公网 Web 站点用的 PHP 版本、php-fpm/php-cgi/mod_php 的部署方式、内网系统。特别容易漏的是**内嵌在商业软件里的 PHP**  
——Huntress 专门提醒了 php.exe / php-cgi.exe 被打进安装包的情况，cPanel/Plesk 面板、某些 Windows 管理工具、出厂固件里都可能有。这种你能做的只有盯厂商通告。用 httpd -v  
、php -v  
、php-fpm -i | grep version  
 或者直接看响应头里的 X-Powered-By  
 快速摸底。  
  
**第二，预演升级路径。**  
 确认每个系统能不能升、怎么升：apt/yum 源是否就绪、面板（宝塔、lnmp 一键包等）什么时候出更新包、重启 php-fpm 的维护窗口怎么安排。目标是补丁一出、命令回车、十分钟收工，而不是今晚现查文档。  
  
**第三，EOL 版本今晚的预案。**  
 升级不了的，今晚开始把暴露面收窄：公网入口限流或挂 VPN、把非必要的 PHP 站点直接下线。要诚实：没有补丁就没有真正的缓解，这些只是拖时间。  
  
**第四，把监控先架好。**  
 检测特征在 0x5，现在就把日志和进程监控的告警通道调通，别等明天。  
## 0x5 发布后的 24 小时  
  
**升级动作本身：**  
 22:00 后盯 php.net 和 php-src 的 release 页，版本号确认后立即更新四个在维护分支（8.2/8.3/8.4/8.5），重启 php-fpm 或 Web 服务生效。别忘了容器镜像（php:x.y-fpm 这类 tag 会重新构建，重新 pull + 重部署）、CI 环境和开发机。  
  
**检测特征（来自 Huntress 的建议，发布后它们会更新 Sigma/YARA 规则）：**  
- 进程链异常：php-fpm  
 / php-cgi  
 / httpd  
 / apache2  
 的 worker 派生 sh  
、bash  
、cmd.exe  
、powershell.exe  
，或拉起 curl  
、wget  
、nc  
、certutil.exe  
 等下载工具——正常 PHP 业务几乎不会有这条进程链；  
  
- 命令行内容特征：/dev/tcp/  
、fsockopen  
、mkfifo  
，以及 whoami  
、id  
、uname -a  
、ipconfig  
 这类踩点命令；  
  
- 文件系统：Web 根目录新出现的文件（webshell 落地）、新增 cron 计划任务、新账号。  
  
**中招了怎么办：**  
 隔离主机、保留 PHP 和 Web 日志用于溯源、全盘查 webshell 和持久化（cron、SSH key、新增账户），然后从干净镜像重建——PHP 核心 RCE 的利用痕迹在内存里，别指望"清个马"就完事。  
## 0x6 冷静一下  
  
满分预警 ≠ 满分灾难，几个客观限制：  
- **在野利用目前为零**  
，泄露的只是一个工具截图，不是可运行的 PoC 仓库。武器化需要等 diff 公开后的分析工作，这通常要几小时到几天；  
  
- **9.2 是 CVSS v4 基础分**  
，实际风险取决于暴露面：公网直面、EOL 版本、电商类系统（Sansec 点名 Magento/Shopware/WooCommerce）风险最高；内网纯 8.x 且升级通道顺畅的，按计划处理即可；  
  
- **"通打 5.4 起所有版本"是工具截图的一面之词**  
，最终以官方公布的影响范围为准，不排除实际利用有 SAPI、操作系统或配置前提——CVE-2024-4577 当年就只打 Windows CGI 这一种部署。  
  
需要注意的是：** PHP 装机量的太大，今晚之后全网扫描是一定会来的**。你要做的不是判断"我会不会被选中"，而是确保扫描器路过你门口时，版本号已经不是今晚之前的那个。  
  
**参考**  
- https://www.php.net/index.php[#2026]()  
-10-20-1（官方预告原文）  
  
- https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities  
  
**免责声明**  
：本文仅用于安全研究与学习，未经授权不得用于任何非法渗透测试活动。  
  
  
