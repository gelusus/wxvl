#  关于 “史诗级漏洞”PHP2Shell 你应该了解的事实  
创宇安全智脑
                    创宇安全智脑  创宇安全智脑   2026-10-22 10:11  
  
近日，**PHP Core 远程代码执行漏洞**  
（CVE-2026-109618 / CVE-2026-109619）的安全态势严峻，知道创宇积极防御实验室已完成相关应急响应。  
  
该漏洞位于 PHP 解释器内核，攻击者无需任何权限即可发起远程代码执行（RCE）攻击，被 PHP 项目安全工程师称为「十多年来 PHP 面临的最严重安全事件」。  
  
目前，知道创宇全线安全产品已完成该漏洞的检测规则及威胁情报升级，支持对相关资产与攻击行为的快速排查与拦截。  
  
以下为本次漏洞的详细解读、实测验证及产品支持情况。  
  
01  
漏洞概述与影响范围  
  
CVE-2026-109618 / CVE-2026-109619（又称   
PHP2Shell）是存在于 **PHP 核心（PHP Core）组件**  
中的两个远程代码执行漏洞，根源均为 **PHP 请求生命周期边界上的内存状态管理缺陷**  
，与上层应用代码无关。  
  
▎两个漏洞的技术成因  
  
**CVE-2026-109618（RCE via OOM corruption）：**  
PHP 在处理内存耗尽（OOM）错误时，会通过 longjmp()  
 直接跳出当前上下文。在部分场景下，请求相关的内存尚未完成初始化就已处于可访问状态，后续 request shutdown 阶段的清理流程会把未初始化数据误判为有效数据，造成内存破坏，最终实现代码执行。  
  
**CVE-2026-109619（Stale global state UAF）：**  
PHP 工作进程在请求结束后，error、session、object、extension 等全局状态未被彻底重置，其中部分指针仍指向已释放的请求内存。攻击者可在同一 worker 上先后构造「生产者」与「消费者」两个请求，后者触发对残留状态的访问，形成跨请求的释放后重用（UAF），进而导致敏感内存泄露并最终实现远程代码执行。  
  
<table><thead><tr><th data-colwidth="170" style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">项目</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">详情</span></section></th></tr></thead><tbody><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">漏洞类型</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">远程代码执行（RCE）</span></section></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">CVSS 评分</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><strong style="color: rgb(192, 57, 43);"><span leaf="">9.2（Critical） <span textstyle="" style="color: rgb(0, 0, 0);">CVSS 4.0</span></span></strong></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">影响组件</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP Core（解释器内核，非扩展/框架层）</span></section></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">触发方式</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">特制 multipart POST 请求，无需身份认证</span></section></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">CNVD 编号</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">CNVD-2026-41674 / CNVD-2026-41675</span></section></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">POC 状态</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><strong style="color: rgb(192, 57, 43);"><span leaf="">已公开</span></strong></td></tr><tr><td data-colwidth="170" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">在野利用</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><strong style="color: rgb(192, 57, 43);"><span leaf="">已发现</span></strong></td></tr></tbody></table>  
  
**受影响版本与修复版本**  
  
<table><thead><tr><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">分支</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">受影响版本</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP 8.5</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">&lt; 8.5.12</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><strong style="color: rgb(27, 126, 58);"><span leaf="">8.5.12</span></strong></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">PHP 8.4</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">&lt; 8.4.27</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><strong style="color: rgb(27, 126, 58);"><span leaf="">8.4.27</span></strong></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP 8.3</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">&lt; 8.3.36</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><strong style="color: rgb(27, 126, 58);"><span leaf="">8.3.36</span></strong></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">PHP 8.2</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">&lt; 8.2.35</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><strong style="color: rgb(27, 126, 58);"><span leaf="">8.2.35</span></strong></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP 8.1 及更早</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">全部版本</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><strong style="color: rgb(192, 57, 43);"><span leaf="">EOL，无官方补丁</span></strong></td></tr></tbody></table>  
  
⚠️ 注意：PHP 8.1 已于 2024 年 11 月结束支持，PHP 7.x / 5.x 更早已 EOL，官方不会为这些版本提供修复。仍在运行旧版本的环境需通过发行版补丁、源码回填或迁移解决。  
  
影响范围  
  
P  
HP 作为全球范围内广泛使用的服务端  
脚本语言，支撑着庞大的 Web 业务生态。据 W3Techs 统计，**全球约 70%–75% 的网站使用 PHP**  
，WordPress、Drupal、Joomla、MediaWiki、Magento、Laravel、Symfony、phpMyAdmin、Nextcloud、WHMCS 等主流应用均依赖 PHP 运行。  
  
根据ZoomEye网络空间搜索引擎关键字  
app="PHP"  
对潜在可能目标进行搜索，共得到  
110,987,753条  
搜索结果。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Ou5agVlRCticmaFYWynzWcVmnxmkKkbHbCYicwRfmHIZsU43uKWDSicziaaM6lSic0lMw7U5DILlmRhQ8kT3Czta1BlzyKOYJoPNGLo9h9Fmic4ZU/640?wx_fmt=png&from=appmsg "")  
  
**相关讨论热点**  
  
<table><thead><tr><th data-colwidth="139" style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;text-align: left;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">来源</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;text-align: left;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">发布情况</span></section></th></tr></thead><tbody><tr><td data-colwidth="139" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP 官方</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">2026-10-21 14:00 UTC 发布安全公告与四个修复版本，原定次日协调披露，因提前泄露被迫抢跑</span></section></td></tr><tr><td data-colwidth="139" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">CNVD</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">发布《关于对 PHP2Shell 漏洞开展修复的风险提示》，编号 CNVD-2026-41674 / 41675</span></section></td></tr><tr><td data-colwidth="139" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">CISA / CERT-EU</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">同日发布紧急告警，强烈敦促各组织立即修补</span></section></td></tr><tr><td data-colwidth="139" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">Cloudflare/watchTowr / Unit 42</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">确认复现漏洞，并报告补丁上线前已出现真实攻击尝试</span></section></td></tr><tr><td data-colwidth="139" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">海外媒体</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">路透社报道荷兰、芬兰、法国等多国政府网站预防性下线；日本多家游戏平台紧急维护</span></section></td></tr></tbody></table>  
  
社区讨论态势：漏洞细节与演示视频在 Hacker News 提前泄露，GitHub 上已出现可直接运行的利用程序，X（Twitter）相关话题热度持续攀升。  
  
02  
实际利用难度与验证情况  
  
为评估该漏洞对企业实际业务的真实威胁，**知道创宇安全研究团队**  
在隔离环境中对   
PHP2Shell漏洞进行了复现与验证。  
  
**实际利用难度分析**  
  
<table><thead><tr><th data-colwidth="115" style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">维度</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">利用条件</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">无需登录、无需用户交互、无需应用层漏洞；但需同时满足：SAPI 为 mod_php（Apache prefork）、display_errors=On、session.save_handler=files、session.upload_progress.enabled=On（默认开）、memory_limit 足够小（使 12MB 的 multipart 请求在解析阶段即 OOM），且 PHP 版本/构建在符号偏移表与堆布局表内（实测 8.3.x / 8.4.x；8.2.x 实测不可用）</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">利用复杂度</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><strong style="color: rgb(192, 57, 43);"><span leaf="">中</span></strong><section style="text-align: left;"><span leaf="">需先用 OOM 请求换 fresh worker 的 A/B 双请求链，再打三段：stage1 用 Set-Cookie 泄漏 Zend MM chunk 基址、stage2 泄漏 libphp 基址、stage3 借 upload_progress 伪造引用把 pDestructor 改写为 zif_system；依赖离线提取的符号偏移与现场标定的堆布局（换版本需重新标定），单条命令受 167 字节载体限制</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">前置配置</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">session.upload_progress 自 PHP 5.4 起默认开启，入口条件容易满足；但 display_errors=On 是硬条件——生产环境常见 Off，此时只能拿到堆基址、打不到 RCE，利用工具会识别并直接退出，不会硬打</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">稳定性</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">强依赖对端环境：单 worker 或 pin 住 N-1 个 worker 才稳定；多 worker 不 pin 实测命中率 0；同一镜像不同 docroot 配置下 2/2 与 6/11 都出现过，失败原样重跑通常能中</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">影响与权限</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">命令以 Web 进程身份执行（通常 www-data），无自动提权、无持久化；每阶段都会在日志里留下 &#34;Input variables exceeded 1000&#34;、&#34;Allowed memory size ... exhausted&#34;、worker abort 等明显痕迹</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">风险最高场景</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">Apache + mod_php 部署，且 display_errors=On、memory_limit 较小、PHP 版本命中偏移表。此类环境下任意 PHP 端点都可触发——入口在请求解析阶段（session.upload_progress 自动启动 session），不依赖应用代码，实测无业务逻辑的 index.php 也能打通</span></section></td></tr><tr><td data-colwidth="115" style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">不适用场景</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">PHP-FPM（OOM 后 worker 被回收，悬空指针不在同一进程，链断，按结构判断不可用）；8.2.x（A 请求后 B 不再下发 Set-Cookie，泄漏通道拿不到）；display_errors=Off；版本/构建不在表内</span></section></td></tr></tbody></table>  
  
**验证情况说明**  
  
知道创宇安全研究团队针对可复现的部署形态进行了模拟攻击实测：![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ou5agVlRCt9Iv2h8fgxYgibhJO8SibxXzYEb0Ylkpw45qRBysuHY0NOvicUItqDr7796GicvdALvDkYFibh4r4Zpz6pmlTib1kTkCT0aUvfB36AvA/640?wx_fmt=png&from=appmsg "")  
  
  
**环境准备：在**  
本地隔离环境启动官方镜像 php:8.4.26-apache（Debian trixie、Apache 2.4.69、mod_php prefork 4 worker，linux/amd64），配置为 display_errors=On、memory_limit=8M、max_input_vars=1000、session.upload_progress.enabled=1 。  
  
**POC 验证：**  
先对目标做指纹确认（版本号、display_errors 状态、版本是否在偏移表与堆布局表内），再自动探测 prefork 饱和点并 pin 住 3 个 worker 留 1 个干活，随后打三段链。实测 stage1 泄漏 chunk=0x7ffffc400000、stage2 泄漏 libphp=0x7ffffe013000，stage3 成功执行命令并取得 uid=33(www-data)，退出码 0；符号偏移与离线表完全一致（zif_system=0x465370、zval_ptr_dtor=0x65eb60），堆布局 stage1_n=841 首次即命中。  
  
**验证结论：**  
该漏洞并非在所有默认配置下都高可利用——它要求 mod_php + display_errors=On + 较小 memory_limit + 版本命中；满足这些条件时可稳定拿到 Web 进程命令执行，而在最常见的生产配置（display_errors=Off、PHP-FPM）下不可利用。建议企业优先排查：SAPI 是否为 mod_php、display_errors 是否开启、memory_limit 设置、PHP 版本是否在受影响范围，以及 Apache worker 数（多 worker 需要额外规避手段才能稳定利用，但不应据此放松）。  
  
声明：本文内容基于现阶段已获取的公开信息、技术资料及实验验证结果进行分析与整理。受研究范围、实验环境及现有技术认知等因素限制，相关分析可能存在不完整或偏差之处，不排除存在其他利用方式或尚未发现的利用路径。  
（此处插入漏洞复现与验证过程截  
  
03  
知道创宇全线产品已支持该漏洞检测与防护  
  
为了帮助客户快速感知风险、排查资产并阻断攻击，我们的产品矩阵已在第一时间更新了针对   
PHP2Shell 漏洞的检测与防御规则：  
  
实时阻断**创宇盾**  
  
**能力支持：创宇盾用户无需升级，该漏洞默认即可拦截。**  
  
资产排查**ZoomEye攻击面管理平台**  
  
**能力支持：**  
支持对企业互联网暴露资产的 PHP 版本进行精准识别与风险标记。****  
  
**检测状态：**  
已更新检测规则，客户可下发风险监测任务直接一键发起针对CVE-2026-109618/109619  
漏洞  
的全网资产盘点，输出受影响资产清单。  
  
无损检测**ScanV**  
  
**能力支持：**  
支持通过无害化插件精准探测目标服务是否存在  
CVE-2026-109618/109619  
漏洞，不影响业务运行。****  
  
**规则版本：**  
插件库已完成更新，客户登录平台即可使用。  
  
04  
已捕获的威胁情报  
  
根据我们创宇智脑全球威胁情报的持续监测：  
  
**野外利用态势：**  
自 2026 年 10 月 20 日漏洞信息提前泄露起，创宇智脑已监测到多起利用该漏洞进行的野外扫描与攻击行为，补丁发布后攻击强度进一步上升。Cloudflare、watchTowr、Unit 42 等机构均确认在补丁上线前数小时已检测到真实攻击。  
  
**攻击源分布：**  
近24小时，我们通过创宇智脑威胁情报网络共捕获到92个针对  
PHP2Shell  
漏洞发起攻击的真实高价值攻击源，其中境外攻击源 9 个，累计拦截 69 次，覆盖瑞士、荷兰、菲律宾、加拿大、马来西亚、法国、孟加拉、日本等 8 个国家。其中瑞士苏黎世单一 IP（107.189.14.87）拦截 56 次，占比约 81%，为本次攻击的主要来源，疑似云主机被控后用作扫描跳板；其余攻击源分散于欧洲、东南亚、北美及南亚地区，单点拦截量较低，呈现多点试探特征。  
  
<table><thead><tr><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;text-align: left;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">攻击源 IP</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;text-align: left;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">归属地</span></section></th><th style="padding: 10px 8px;background: rgb(192, 57, 43);color: rgb(255, 255, 255);font-size: 13px;font-weight: bold;text-align: left;border: 1px solid rgb(192, 57, 43);"><section style="text-align: left;"><span leaf="">拦截次数</span></section></th></tr></thead><tbody><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">107.189.14.87</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">瑞士苏黎世</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">56</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">34.158.136.105</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">荷兰格罗宁根</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">3</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">175.158.224.187</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">菲律宾马卡蒂市</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">2</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">158.69.113.75</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">加拿大蒙特利尔</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">2</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">14.192.215.60</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">马来西亚吉隆坡</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">2</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">89.93.181.106</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">法国</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">1</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">2407:9803:92c0:30e4:d898:ac81:3c6:6c0a</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">菲律宾马卡蒂市</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">1</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">104.28.208.84</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">孟加拉达卡</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(253, 246, 245);"><section style="text-align: left;"><span leaf="">1</span></section></td></tr><tr><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">103.151.173.97</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">日本东京</span></section></td><td style="padding: 10px 8px;font-size: 13px;line-height: 1.7;color: rgb(31, 45, 61);border: 1px solid rgb(232, 213, 210);background: rgb(255, 255, 255);"><section style="text-align: left;"><span leaf="">1</span></section></td></tr></tbody></table>  
  
注：  
相关 ioc 已经通过创宇智脑平台 实时联动同步到创宇盾，创宇云蜜罐，威胁情报网关等安全产品和服务 支持实时检测分析处置和关联分析。  
  
**常见攻击特征：**  
攻击者通常发送特制 multipart/form-data POST 请求，请求体中包含异常的 PHP_SESSION_UPLOAD_PROGRESS  
 字段与超大 Content-Length，并伴随畸形分片边界；部分 Payload 会在短时间内对同一 worker 连发两次请求以完成「生产者—消费者」链路。成功利用还会在日志中留下 "Input variables exceeded 1000"、"Allowed memory size ... exhausted"、worker abort 等特征痕迹，可作为排查依据。  
  
**紧急处置与修复建议**  
  
方案一：官方补丁升级（强烈推荐）  
  
请尽快将 PHP 升级至对应分支的安全版本：**8.5.12 / 8.4.27 / 8.3.36 / 8.2.35**  
 或更高版本。  
  
官方公告：· php.net/archive/2026.php#2026-10-21-5· GHSA-w8hw-w9fx-j79w（CVE-2026-109618）· GHSA-6wmf-4rv5-p3rp（CVE-2026-109619）  
  
**关键动作：**  
升级后必须**重启全部 PHP 进程**  
（PHP-FPM / Apache / CLI 常驻进程），否则残留 worker 仍带漏洞。  
  
方案二：临时缓解措施（无法立即升级时）  
  
以下配置必须写入 **php.ini 或 FPM 池配置**  
，在请求解析前生效（ini_set()  
 与 .user.ini  
 均来不及）：  
```
```  
  
**切勿使用**  
 RLIMIT_AS / RLIMIT_DATA 作为缓解手段——这类限制会导致 mmap() 失败，反而触发 CVE-2026-109618 所依赖的 longjmp() 路径。  
  
**安全排查与监控**  
  
1. **资产盘点：通过ZoomEye攻击面管理平台**  
梳理全部 PHP 实例及版本，覆盖容器镜像、主机面板、PHP-FPM、Apache mod_php、CLI 脚本及常驻进程，重点关注 PHP 8.1 及更早的 EOL 环境。  
  
2.   
边界防护：  
将运行在互联网侧的相关网站或或业务系统全面接入创宇盾云防御系统，即时检测阻断相应漏洞攻击，后续再适时完成 PHP 版本升级。  
  
3. **流量回溯：**  
检索相关 IOC，回溯近 7 天访问日志中是否存在异常 multipart POST 请求与可疑 session.upload_progress 字段。  
  
4. **主机检查：**  
检查 Web 目录是否存在异常 PHP 文件、进程是否存在异常外联、系统是否存在新增计划任务或 SSH 公钥。  
  
5. **共享主机用户：**  
请主动联系服务商确认 PHP 版本，多租户环境需确认所有租户实例均已升级。  
  
