#  高危预警｜PHP 曝出未授权远程代码执行漏洞，全球四分之三网站受影响  
值得信赖的
                    值得信赖的  默安科技   2026-10-22 08:41  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/PRUwRKvusicPXQhp9NVSkXZZN8WZYye6Dfacb5bbPNt9PkOGMzlTsHgTPicPZQW4PyxTgjRS4ib2lSqiaO9IXKATXQ/640?wx_fmt=gif "")  
  
  
**0****1**  
  
  
**漏洞速览**  
  
  
  
<table><thead><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">项目</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">内容</span></span></span></td></tr></thead><tbody><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">漏洞名称</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">PHP 远程代码执行漏洞</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">漏洞编号</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">CVE-2026-109618、CVE-2026-109619</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">公开时间</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">2026 年 10 月 21 日</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">漏洞类型</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">释放后重用（UAF）导致远程代码执行</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">影响版本</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">PHP &lt; 8.2.35、PHP &lt; 8.3.36、PHP &lt; 8.4.27、PHP &lt; 8.5.12</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">利用条件</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">无需认证、无需用户交互、无需应用层漏洞</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">漏洞状态</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">PoC 已公开、EXP 已公开、技术细节已公开</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">官方定级</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">Critical（严重）</span></span></span></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">影响量级</span></span></span></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;font-size: 15px;color: rgb(87, 107, 149);letter-spacing: normal;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><span style="color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">千万级</span></span></span></td></tr></tbody></table>  
  
PHP 是全球使用最广泛的服务器端脚本语言之一。据 W3Techs 统计，全球约四分之三的网站使用 PHP，WordPress、Drupal、Joomla、MediaWiki、Magento、phpMyAdmin、Nextcloud、Laravel、Symfony、WHMCS 等主流应用均依赖 PHP 运行。这意味着，本次漏洞的影响面远不止 PHP 本身，而是覆盖了互联网上大量 Web 应用与建站系统。  
  
  
**0****2**  
  
  
**漏洞原理：一次“内存耗尽”，撕开请求生命周期的边界**  
  
  
  
  
这两个漏洞都源于 PHP 请求生命周期边界上的状态管理缺陷，但触发路径各有不同：  
  
  
**· CVE-2026-109618：**  
PHP 在内存耗尽（OOM）时使用 longjmp() 提前退出当前上下文。在某些场景下，请求内存在尚未完成初始化时就已经对后续流程可见；一旦此时发生分配失败，PHP 没有正确处理这次提前退出，导致请求内存在 request shutdown 阶段被错误消费。  
  
  
**· CVE-2026-109619：**  
Worker 的全局状态在请求结束后没有被正确重置，后续请求在完成正常初始化之前，就可能访问到已经释放的请求内存。  
  
  
攻击者无需任何认证，只需构造恶意 HTTP 请求，即可触发释放后重用（UAF），进而实现内存泄露与远程代码执行。更值得警惕的是，攻击发生在应用代码执行之前：恶意请求在 PHP 自动解析请求体、注册请求变量的阶段，就已经可以影响请求关闭流程。  
  
  
研究团队还披露了不同部署形态下的差异：  
  
  
**· Apache mod_php：**  
即使请求启动失败，也会无条件执行请求关闭流程，因此可直接消费异常状态，是本次确认的完整利用路径；  
  
  
**· PHP-FPM：**  
请求启动失败时会退出 Worker，利用条件更复杂，但源码分析表明其仍具备可利用性，研究团队已演示在 Nginx + PHP-FPM 环境下的利用；  
  
  
**· FrankenPHP、LiteSpeed LSAPI：**  
与上述模型类似，存在受影响可能。  
  
  
值得一提的是，这并非一个“突然出现”的新问题。其技术脉络最早可追溯到 2004 年 PHP 内存限制相关的远程漏洞，2020 年 PHP Bug [#78876]()  
 再次触及类似问题，2024 年 OSTIF 与 Quarkslab 的安全审计也发现过相关缺陷但未能证明可利用性。2026 年，Calif Research 联合 OpenAI、Wiz 在“Patch the Planet”项目中重新审计 PHP，最终打通了从 OOM 到 RCE 的完整利用链，并于 10 月 21 日随官方补丁一同公开。  
  
  
**0****3**  
  
  
**影响范围：不止 PHP，更是整个 Web 生态**  
  
  
  
  
**· 受影响版本：**  
  
PHP < 8.2.35  
  
PHP < 8.3.36  
  
PHP < 8.4.27  
  
PHP < 8.5.12  
  
  
研究团队指出，最早的受影响代码路径可追溯至 PHP 5.4.0，实际受影响范围可能比官方分支列表更广。PHP 8.1 及更早版本已停止官方支持，不再提供补丁。  
  
  
**· 受影响的部署形态：**  
  
Apache mod_php  
  
PHP-FPM（含 Nginx、Apache mod_proxy_fcgi 等组合）  
  
FrankenPHP  
  
LiteSpeed LSAPI  
  
Docker 官方 PHP-FPM 镜像及各类集成环境（宝塔、phpStudy、XAMPP 等）  
  
  
**· 受影响的典型应用：**  
  
研究团队已确认可在以下应用中实现未授权远程代码执行：  
WordPress、Drupal、Joomla、MediaWiki、Roundcube、Magento、phpMyAdmin、Nextcloud、Laravel、Symfony、WHMCS。  
  
  
**0****4**  
  
  
**默安科技应对方案：从“看见风险”到“验证风险”**  
  
  
  
  
面对这类影响面广、利用门槛低的高危漏洞，企业需要的不仅是“知道有漏洞”，更是快速回答三个问题：  
  
  
1. 1. 我到底有多少资产受影响？  
  
1. 2. 这些资产是否真的可以被利用？  
  
1. 3. 修复之后，风险是否真正关闭？  
  
1.   
1. 默安科技以“巡哨 + AI 红队智能体”的组合，给出从资产发现、漏洞检测到利用验证、修复复测的完整闭环。  
  
1.   
  
  
**巡哨：已支持该漏洞的扫描检测**  
  
  
  
巡哨（智能资产风险监控系统） 是默安科技推出的新一代智能化资产与漏洞风险监控系统，以攻击者视角持续梳理企业互联网暴露面，主被动结合发现内外网资产，覆盖 2000+ 应用漏洞策略及海量第三方组件、合规漏洞信息。  
  
  
针对本次 PHP 高危漏洞，巡哨已在第一时间完成漏洞插件更新，支持对 CVE-2026-109618、CVE-2026-109619 的扫描检测。企业可通过巡哨：  
  
  
· 快速定位暴露在互联网上的 PHP 资产与相关 Web 应用；  
  
· 识别受影响版本与部署形态，输出受影响资产清单；  
  
· 结合资产重要性、业务价值与漏洞风险，给出修复优先级；  
  
· 通过 AI 智能体引擎中的漏洞分析 Agent 对扫描结果进行二次研判，自动剔除无效告警与误报，让安全团队聚焦真实风险；  
  
· 依托 7×24 小时漏洞响应机制与高危漏洞插件 24 小时内更新能力，持续跟踪漏洞态势与新增暴露资产。  
  
  
对于重保、攻防演练等关键时期，巡哨能够帮助企业在最短时间内完成“摸清家底—定位风险—排序处置”的第一步。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Gianlj61NfoxERGD5tnDvGqnkylH5cj6kXJr9dmtlAhZCqiaBk7ohvJLc6oH93ZcgCE52kGPXo7QXpScv8xicIMTZ6l2v7JP6fq1m8IsNwW8pk/640?wx_fmt=png&from=appmsg "")  
  
  
**AI 红队智能体：已支持该漏洞的利用验证**  
  
  
  
默安 AI 红队智能体是 7×24 小时全自主运行的 AI 智能红队，采用调度层、执行层、知识层三层架构，能够在授权范围内连续完成情报收集、资产发现、漏洞比对、验证利用、权限获取的全流程。  
  
  
针对本次 PHP 漏洞，  
**默安 AI 红队智能体已完成利用能力适配，支持对该漏洞进行真实可利用性验证。**  
与仅停留在“版本匹配”层面的扫描不同，AI 红队智能体可以在授权演练环境中实际发起验证，回答“这个漏洞到底能不能被打穿”：  
  
  
· 自动识别目标 PHP 版本、SAPI 类型与部署形态，判断适用的利用路径；  
  
· 对确认受影响的目标执行无害化利用验证，获取可复现的证据链；  
  
· 记录完整攻击路径、影响范围与验证过程，任务看板全程可视、可审计；  
  
· 修复完成后再次验证，确认风险真正闭环，而不是“版本号看起来对了”。  
  
  
在实战中，这套能力尤其适用于三类场景：  
  
  
**· 漏洞应急：**  
PoC 公开后，快速确认自有资产是否真的可被利用，避免“全部升级”或“全部忽略”的粗放决策；  
  
**· 重保与攻防演练前：**  
以攻击者视角检验边界资产，提前发现最危险的突破口；  
  
**· 修复复测：**  
补丁上线、服务重启后，验证旧 Worker 是否真正退出、漏洞是否彻底关闭。  
  
  
AI 红队智能体全程遵循授权演练边界，只计数、不拉全量、无破坏性操作，主机驻留采用可快速重建模式，确保零残留、可逆、可审计。  
  
  
**组合价值：发现、验证、复测一体化**  
  
  
<table><thead><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">阶段</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">能力</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">输出</span></span></p></td></tr></thead><tbody><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">资产发现</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">巡哨主被动测绘</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">互联网暴露面与 PHP 资产清单</span></span></p></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">漏洞检测</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">巡哨漏洞插件 + AI 漏洞分析 Agent</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">受影响资产、版本、风险等级、修复优先级</span></span></p></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">利用验证</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">AI 红队智能体</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">真实可利用性结论、攻击路径与证据链</span></span></p></td></tr><tr style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: #e5e5e5;"><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">修复复测</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">AI 红队智能体 + 巡哨</span></span></p></td><td style="box-sizing: border-box;border: 1px solid rgb(223, 223, 223);text-align: left;line-height: 1.75;font-size: 14px;color: rgb(63, 63, 63);word-break: keep-all;padding: 5px 10px;font-family: -apple-system-font, BlinkMacSystemFont, Arial, sans-serif;"><p style="margin-top: 0px;margin-bottom: 0px;display: block;"><span style="box-sizing: border-box;border-width: 0px;border-style: solid;border-color: rgb(229, 229, 229);text-align: left;line-height: 1.75;letter-spacing: normal;color: rgb(51, 51, 51);font-size: 15px;"><span leaf="">修复有效性确认与风险闭环报告</span></span></p></td></tr></tbody></table>  
  
从“扫到漏洞”到“证明可利用”，再到“确认已修复”，企业可以用一套组合能力完成完整闭环，把有限的安全资源投入到真正高风险、真正可利用的资产上。  
  
  
**0****5**  
  
  
**处置建议**  
  
  
  
  
**·立即升级 PHP 至安全版本：**  
  
PHP 8.5.12  
  
PHP 8.4.27  
  
PHP 8.3.36  
  
PHP 8.2.35  
  
下载地址：https://www.php.net/downloads.php  
  
  
**· 升级后必须重启所有常驻进程：**  
  
仅替换二进制文件而不重启进程，不能消除风险。请务必重启 Apache、PHP-FPM 及所有常驻 Worker，确保旧进程不再处理请求。  
  
  
**· 容器与 Kubernetes 环境需重建：**  
  
容器环境应重新构建镜像，避免沿用包含旧 PHP 的 Layer；Kubernetes 环境应滚动重启所有 Pod，并确认没有旧 ReplicaSet 继续提供服务。  
  
  
**· 注意发行版与集成环境的同步延迟：**  
  
Ubuntu、Debian、RHEL、Alpine 等发行版仓库，以及宝塔、phpStudy、XAMPP、Docker 官方镜像等集成环境，可能存在补丁同步延迟。请以 php -v、php-fpm -v 的实际输出为准，不要仅核对包管理器版本号。  
  
  
**·  PHP 8.1 及更早版本需尽快迁移：**  
  
PHP 8.1 及更早版本已停止支持，官方不再提供补丁。建议迁移至 PHP 8.2 或更高版本；若使用发行版长期支持分支，请以厂商 backport 公告为准。  
  
  
**· 无法立即升级时的临时缓解措施：**  
  
· 对 PHP-FPM，可临时关闭 Session 上传进度：session.upload_progress.enabled=0，并设置 memory_limit=-1；  
  
· 对 mod_php，可临时设置 memory_limit=-1；  
  
· 保持 post_max_size 保守配置，并在前端独立限制解码后的请求体大小；  
  
· 降低 max_input_nesting_level，限制输入变量、multipart 分段、每段 Header 数量与上传文件数量；  
  
· 若业务不使用 multipart 表单，可在前端直接阻断此类请求。  
  
  
**特别提醒：不要使用 RLIMIT_AS 或 RLIMIT_DATA 作为缓解措施。**  
 这些限制会导致 mmap() 失败，反而触发 CVE-2026-109618 所依赖的 longjmp() 路径。  
  
  
需要强调的是，以上均为临时缓解手段，  
**升级到包含补丁的 PHP 版本是唯一完整的修复方式。**  
  
  
**· 共享主机与多租户环境需重点排查：**  
  
共享主机、虚拟主机用户应联系服务商确认 PHP 版本；多租户环境需确认所有租户实例均已完成升级，避免单点遗漏。  
  
  
**06**  
  
  
总结  
  
  
  
  
对企业而言，当前最重要的三件事是：**尽快摸清 PHP 资产、****尽快完成升级重启、****尽快验证修复效果。**  
  
  
默安科技“巡哨 + AI 红队智能体”已做好能力准备：  
  
  
**· 巡哨：**  
支持 CVE-2026-109618、CVE-2026-109619 扫描检测，快速定位受影响资产；  
  
**· AI 红队智能体：**  
支持该漏洞的利用验证，在授权范围内证明风险是否真实可利用。  
  
  
如需进行资产排查、漏洞验证或应急支撑，欢迎联系默安科技安全团队。  
  
  
参考链接：  
  
1. PHP 官方 2026 年 10 月安全公告  
  
2. GitHub Security Advisory：CVE-2026-109618（RCE via OOM corruption）  
  
3. GitHub Security Advisory：CVE-2026-109619（RCE via stale global state）  
  
4. Calif Research：php2shell 技术分析与完整时间线  
  
5. CISA 安全公告（2026-10-21）  
  
6. CERT-EU 安全公告 2026-018  
  
7. Rapid7：CVE-2026-109618 与 CVE-2026-109619 漏洞分析  
  
8. Sansec：PHP2Shell 分析与缓解建议  
  
9. PHP Foundation 官方说明  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PRUwRKvusicMtGCo8BKXNic4OSw52pibHc7q6Xfo674pm4jBtG6PPhPhFsoo8gOufRBTuXayugM3suOVu5icscy9Rw/640?wx_fmt=png&from=appmsg "")  
  
