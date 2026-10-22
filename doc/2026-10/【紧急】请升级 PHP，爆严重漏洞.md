#  【紧急】请升级 PHP，爆严重漏洞  
原创 胡明
                    胡明  黑白之道   2026-10-21 23:12  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6PImc9J0surGneNW7PqDhPdtPxzO1xERzF7OaplPiaMXibRhsIVlxlwCAeYgAt35ibZeeiaqVnjfgicic0M8rzyK3yo0ErUK7qZo2DOk/640?from=appmsg "")  
> **导语**  
：**这是 PHP 史上最严重的一批漏洞，没有之一。**  
 2026 年 10 月 21 日 14:00 UTC，PHP 团队紧急发布 8.5.12、8.4.27、8.3.36、8.2.35 四个安全版本，一次性修掉两个 CVSS v4.0 **9.2 顶格**  
的远程代码执行漏洞，统称 PHP2Shell。要命的地方有四个：**不需要登录、不需要装 WordPress、不需要特殊配置、漏洞长在 PHP 内核里**  
——一个 Hello World 应用照样被拿下，WordPress、Drupal、Joomla 的未认证 RCE 已全部演示打通。更麻烦的是 PHP 进程内存可以直接被读走，里面的密码、令牌、证书原始内容直接落到攻击者手里，**而且不需要先拿到代码执行，所以专盯恶意代码的检测工具反而看不见它。****是今天必须处理的事。**  
  
> **官方披露声明**  
（Wiz 研究员 Sagi Tzadik 于披露当日发布）  
> Today we disclose PHP2Shell - remotely exploitable memory corruptions in PHP  
> WordPress pre-auth RCE: ✅Drupal pre-auth RCE: ✅Joomla pre-auth RCE: ✅Remote memory disclosure of any PHP application: ✅  
> 翻译过来就是：PHP2Shell 是一组可远程利用的内存破坏漏洞。三个主流 CMS 的**未认证远程执行全部打通**  
，而且**任意 PHP 应用都能被远程泄露内存**  
。四个勾，一个都不是"仅影响特定配置"。  
  
## 零、先回答一个问题：这到底有多严重  
  
用红队的标准过一遍，PHP2Shell 在几乎每一个维度上都踩在最坏的一档：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">维度</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">评级</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVSS v4.0</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">9.2 / 9.2</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">两个都是 Critical 顶格</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">要不要登录</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">完全不要</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">mod_php 上已验证的攻击链在应用代码执行前就完成</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">要不要装特定应用</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">不要</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Hello World 应用照样中招</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">要不要特殊配置</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">基本不要</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">session.upload_progress.enabled</span></code><section><span leaf=""> 出厂默认开启，multipart 是最常见的请求类型</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">漏洞在哪</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">PHP 内核</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">换框架、换 CMS、代码写得再安全都挡不住</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">爆炸半径</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">全球约 70% 的网站</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">WordPress / Drupal / Joomla / MediaWiki / Magento 全在内</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">野外是否已被利用</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">是</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">watchTowr 蜜罐、Cloudflare 均报告攻击流量</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">隐蔽性</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">中 → 极高</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">RCE 会留痕；内存泄露路径不留 webshell、不起子进程</span></section></td></tr></tbody></table>  
**历史坐标**  
：CISA 已知被利用漏洞（KEV）目录里已经有三个 PHP 自身的漏洞，其中 CVE-2024-4577（PHP-CGI）和 CVE-2019-11043（PHP-FPM）都被标记为"已用于勒索软件活动"。PHP 内核级 RCE 不是理论风险，它有前科。  
  
**为什么这次比一般的严重漏洞还要更急一层**  
1. **暴露窗口极短**  
。10 月 20 日 CVE 编号就在社交媒体上传开了，10 月 21 日 14:00 UTC 才出补丁。这中间技术细节已经公开流传，攻击者的准备时间被压缩到了以小时计。  
  
1. **补丁动的是解释器核心行为**  
。官方改的是 OOM 的处理路径和 shutdown 阶段的全局状态重置，不是某个函数的一行判断。这种改动级别本身就说明问题的深度。  
  
1. **打补丁不等于结束**  
。第三方独立实验室在已打补丁的 8.5.12 上，仍测出三条通向影响的存活路径。后续版本和厂商通告要持续跟。  
  
1. **这次漏洞的研究和 exploit 开发有前沿 AI 深度参与**  
（第三节会讲）。攻击方的 exploit 迭代速度可能快过以往任何一轮漏洞。  
  
一句话对比：Log4Shell 的杀伤力在于 log4j 几乎渗透进了每一台 Java 服务器；PHP2Shell 的杀伤力在于 **PHP 本身就是承载全球约七成网站的那个运行时**  
。而这一次，连"换个安全框架躲一躲"这条 Java 时代常用的躲法都不成立。  
## 一、一句话结论：这不是你代码的问题，是 PHP 内核的问题  
  
再说一遍这个判断，因为它决定了后面所有的处置优先级。  
  
PHP2Shell 是两个漏洞的合集，都在 PHP 解释器核心，不在任何框架或应用里：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE 编号</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">漏洞名称</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVSS v4.0</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">官方 GHSA</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-109618</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">OOM 内存破坏远程执行</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9.2</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">GHSA-w8hw-w9fx-j79w</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-109619</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">陈旧全局状态远程执行（UAF）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9.2</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">GHSA-6wmf-4rv5-p3rp</span></section></td></tr></tbody></table>  
影响范围：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">PHP 分支</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">受影响版本</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">首个修复版本</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.5</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">&lt; 8.5.12</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">8.5.12</span></strong></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.4</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">&lt; 8.4.27</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">8.4.27</span></strong></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.3</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">&lt; 8.3.36</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">8.3.36</span></strong></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.2</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">&lt; 8.2.35</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">8.2.35</span></strong></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.1 / 8.0 / 7.4</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">全部</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">仅提供最小源码补丁，无正式发行版</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5.4 – 7.3</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">全部</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">已 EOL，PHP 官方不再出补丁</span></section></td></tr></tbody></table>  
官方 FAQ 里的原话是：受影响的发布版本可以追溯到 **PHP 5.4**  
，修复前几乎每一个 PHP 版本都至少受其中一个问题影响。  
  
SAPI 层面，官方把所有对外提供 Web 服务的形态都列进了风险清单：PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed。但官方明确说风险最高的是 **Apache mod_php**  
——因为它有一个已验证的攻击链，可以在任何一行应用代码执行之前就完成触发。PHP-FPM 也必须打补丁，它的启动阶段会杀掉子进程挡掉一条特定路径，但会话生命周期的远程执行在 FPM 上同样验证成立，官方说法是"FPM 的 RCE 通过应用层代码路径确认"。  
  
![PHP2Shell 漏洞影响范围](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6O6Xyicps2IsgV8F5z1yOMf9mcj9rNsOd7qcBuIenR3Pe2VJTcTrhsHM6p9v6qe2QgbGgaNs3kEvRgf2ib9mVuDpQv2iaW9iaXyk9Y/640?from=appmsg "PHP2Shell 漏洞影响范围")  
## 二、时间线：从编号泄露到野外利用，一共四天  
  
这个事件的节奏非常紧，值得完整记一遍：  
- **2026 年 10 月 20 日**  
，CVE 编号先于官方公告出现在 Hacker News 和社交媒体上。原本定在 10 月 22 日的协调披露，因为编号泄露提前。同日 PHP 安全团队发布预告，宣布 10 月 21 日 14:00 UTC 出补丁，并说明"若细节公开或出现利用证据，会更早发布"。CERT-EU（欧盟计算机应急响应团队）发布 2026-018 号公告。  
  
- **2026 年 10 月 21 日 14:00 UTC**  
，PHP 官方安全公告和四个修复版本同步落地，官方专题站 **www.php2shell.com**  
 上线。同日 Wiz、OpenAI、Calif 三方公开技术博客，Qualys、Tenable、Corelight、Akamai、Imperva、Cloudflare、CrowdStrike、Unit 42、Cisco Talos 等厂商陆续放出分析与检测规则。Cloudflare 已经给包括免费版在内的所有客户下发了针对已知利用向量的 WAF 规则，CTO 公开表态"正在看到早期攻击出现"。  
  
- **2026 年 10 月 21 日当天**  
，watchTowr 表示其全球蜜罐网络已观察到针对两个漏洞的利用尝试，并称自己复现了漏洞。  
  
- **2026 年 10 月 22 日（本文发布日）**  
，厂商遥测仍有分歧：watchTowr 和 Cloudflare 报告看到攻击流量，Huntress 表示截至当日美东时间上午尚未在自己客户群中观察到。两个 CVE 目前均未进入 CISA 已知被利用漏洞目录（KEV）。  
  
有一个细节值得单独拎出来：泄露阶段流传的 CVE 编号配的评分是 10.0，PHP 官方后来确认为 **9.2**  
。泄露阶段还伴随大量假 PoC，JFrog 就发现有人借机投放恶作剧仓库，表面上是 Base64 加密的"payload"，实际执行只是把受害者壁纸换成不适宜内容。PHP 安全团队也确认，他们在发布时检查过泄露后流传的 PoC，"在发布和公开披露时没有发现功能可用的 exploit"。  
  
边界也要说清楚：PHP 官方自己在公告里留了一句很重的话——**没有观察到日志里有异常，不代表你安全**  
。历史日志可能压根没开 deprecation 级别的记录，而失败也可能发生在 PHP 写下日志之前。  
## 三、官方怎么定义 PHP2Shell，以及 AI 在这次事件里的角色  
  
php2shell.com  
 是这次事件的官方联合研究主页。署名很清楚：  
- **Wiz Research**  
 —— Sagi Tzadik、Shir Tamari、Nir Ohfeld、Tomer Schwartz  
  
- **OpenAI Codex Security**  
 —— Amy Burnett  
  
- **Calif**  
 —— Ryan A.  
  
- **PHP 团队**  
负责响应与修复，Google 的 Gemini 团队参与了发现、协调和补丁验证  
  
披露当日，Wiz 研究员 Sagi Tzadik 用四个勾做了个极简战报：WordPress 未认证 RCE ✅、Drupal 未认证 RCE ✅、Joomla 未认证 RCE ✅、任意 PHP 应用远程内存泄露 ✅。四个勾全部实打实，没有一条标注"仅在特定配置下成立"。这也是这篇文章开头那段披露声明的出处。  
  
两件容易被忽略的事：  
  
**第一，名字的来历。**  
 "php2shell" 最初是他们用来演示 WordPress、Drupal、Joomla 未认证远程执行的那个**组合 exploit 脚本的名字**  
，后来被用来指代整类 PHP 内核漏洞研究。所以你看到叫 php2shell 的东西，要么是那个演示脚本，要么是这套漏洞的总称。  
  
**第二，这次挖洞和写 exploit 的过程里，AI 是主力之一。**  
 官方 FAQ 里直接写了：Wiz 在发现、exploit 开发和补丁验证三个阶段都用了 Wiz Atlas、Gemini 4 Argon 和 GPT-5.6-Sol，Gemini 4 在 exploit 开发和补丁验证上贡献尤其大。Calif 和 Wiz 也都公开表示研究过程中用了 AI 模型。这是个新信号——**"模型写出可用的 RCE exploit"这件事已经从演示变成了生产**  
，而且是被有顶级能力的团队用在了真实漏洞上。  
  
两个 CVE 各自演示到了什么程度，官方也分开列了：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">攻击者能达到什么</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">研究团队实际演示了什么</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-109618</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">以 PHP web worker 身份无认证执行代码</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">在 WordPress、Drupal、Joomla 上演示未认证 RCE</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-109619</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">触发 use-after-free，泄露 worker 内存或执行代码</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">在 WordPress、Drupal 上演示未认证 RCE；演示了与应用无关的远程内存泄露</span></section></td></tr></tbody></table>## 四、CVE-2026-109618：OOM 把指针留在半路  
  
这个漏洞的根因在 PHP 处理内存耗尽的方式上。  
  
PHP 原本用 longjmp()  
 处理 OOM——直接跳出当前执行上下文终止请求。问题在于，某些环节里请求内存只是"临时可达"，还没完成初始化。只要在这个窗口里分配失败，longjmp()  
 就会把控制权强行抽走，留下一个没配对清理的指针。等请求 shutdown 流程跑起来，它会把这个半成品当成有效数据来读、来分支判断。**失败的那次分配和这块坏内存甚至可以毫无关系**  
，只要发生在这个窗口里就行。  
  
触发 OOM 的路径全是 multipart 解析，官方点名的有四条：  
1. multipart 请求如果没有可用的 Content-Length，能绕过 post_max_size  
 的初始检查，继续喂解码后的 body 数据。  
  
1. 同一个 multipart part 里的 header 数量不受限制。PHP 内部给每个 header 预留固定的 256 字节 SMART_STR_START_SIZE  
，线上 6 个字节的 header 到内存里膨胀成三百多字节——这就是后面会讲到的 **52 倍内存放大**  
。  
  
1. 空的 file 控件不计入 max_file_uploads  
，却照样生成完整的文件元数据数组。  
  
1. 嵌套字段名会把数组构造的工作量成倍放大。  
  
**为什么 mod_php 最惨**  
：Apache 的 mod_php 在请求启动失败后，仍然会无条件调用请求 shutdown 流程。而请求启动阶段的自动 body 解析本身就可能耗尽 memory_limit  
，在这个状态下退出，shutdown 就会去处理这块无效状态。整个过程发生在应用代码开始执行之前。FPM 则是启动失败直接退出子进程，这条特定序列活不下来。  
  
补丁怎么改的？把 OOM 路径上的 longjmp()  
 拿掉（php-src 提交 31c0ea7d7ff  
），换成延迟触发的软 OOM——到下一个 VM 安全点才优雅终止；内存消耗超过第二个更高阈值时用 _exit()  
 硬终止。这样清理逻辑总能跑完，指针在内存释放前一定被清掉，PHP 限制内存的能力也保留。  
## 五、CVE-2026-109619：上一个请求的遗物，被下一个请求捡起来用  
  
这个是教科书级的 use-after-free，但触发路径设计得很巧。  
  
PHP worker 有一批全局状态：错误状态、会话状态、对象状态、扩展状态。这些状态被 worker 长期持有，但**指向的内存属于某一个具体请求**  
。正常情况下，请求内存释放前这些全局字段必须先重置。问题是有几条路径能在初始化完成前就碰到它们：  
  
**生产者**  
：把请求私有状态写进全局可访问位置的操作。触发点包括致命错误（解析失败、未捕获异常、require  
 失败、执行超时）、OOM 中断，以及一个更隐蔽的情况——非致命的 warning 在 shutdown 后期又把已经清空的错误状态重新填了回去。  
  
**消费者**  
：在下一个请求正常初始化之前，读取、释放或基于这个悬垂状态分支的操作。官方确认了两个：  
- **初始化前的 INI 诊断**  
。Apache 的 .htaccess  
 和 php_value  
 / php_flag  
 / php_admin_*  
 是在 php_request_startup()**之前**  
应用的。一条 auto_detect_line_endings=1  
、一个非默认的 filter.default  
、显式配的 mbstring.*  
，或者一个格式错误的 memory_limit  
，都可能触发 warning，进而消费掉陈旧状态。改 error_reporting  
、关 display_errors  
、加 @  
 抑制符，**统统没用**  
——全局状态是在 PHP 决定要不要报这个 warning 之前就被访问的。  
  
- **会话上传进度**  
。session.upload_progress  
 默认开启，可以在正常会话初始化之前就摸到会话和解码器状态，不需要应用调用 session_start()  
。PHP 8.5 的 fatal_error_backtraces  
 保存回溯状态时漏掉重置，是官方确认的最强生产路径，也是 mod_php 上已验证 RCE 的基础。  
  
要打中，必须生产和消费落在**同一个 worker**  
 上。多 worker、worker 回收会降低单次成功率，但不会让条件消失。  
### 攻击链怎么串起来的  
  
JFrog 把完整链条还原得比较清楚：  
1. multipart POST 会被 PHP 处理成上传进度跟踪，磁盘上落一个会话文件；  
  
1. 会话 ID 用户可控（普通 POST 字段 PHPSESSID  
），会话 key 的后缀 PHP_SESSION_UPLOAD_PROGRESS  
 用户可控，上传的文件名**原样存储**  
——相当于能把任意内容写进会话文件；  
  
1. 会话文件是序列化格式，PHP 打开会话时在**任何脚本执行之前**  
反序列化它；  
  
1. 解码器不校验这些字节是不是 PHP 自己写的，用户影响的内容被当成活的会话数据解码；  
  
1. 反序列化过程中 PHP 构造 zval（存动态值的结构），并在一个记账结构里登记引用和析构。这个记账结构假设"每次初始化都在同一个请求里配对清理"；  
  
1. 第一个请求出错跳过了清理，全局仍指向已释放的记账内存；  
  
1. **同一个 worker**  
 处理的第二个 multipart 上传重新占住这块内存，解码攻击者控制的字节，会话回写时调用假对象的序列化函数，控制流被交出去。  
  
![PHP2Shell 攻击链示意图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6OW7sg4TicwI2hayS5FCEM4nZQa2t9iasFSma0p3uWtj4Ae6Y55JGVtFicQ5C2Y67KEMZwbHssx8T1icZ2jhsY2jchzibVvaqudq6Xc/640?from=appmsg "PHP2Shell 攻击链示意图")  
## 六、影响面：七成网站都在射程内，而且不挑应用  
  
官方专题站的第一句话就是：**PHP 支撑着大约 70% 的网站**  
，包括 WordPress、Drupal 和维基百科用的 MediaWiki。第三方统计（StatCounter）给的口径是 69.3%。  
  
这次演示的目标是 WordPress、Drupal、Joomla，用的是测试时的最新容器镜像。但官方 FAQ 特意加了一句我觉得每个防守方都应该划重点的话：  
> 这三个应用只是用来演示未认证远程执行的**代表性目标。利用并不要求你装了这三者中的任何一个。**  
 漏洞代码在 PHP 本身，跑在受影响 PHP 运行时上的一个简单 "Hello World" 应用也会中招。  
  
  
再具体一点的规模估算：安全研究员 Silas Cutler 估计超过 **130 万台**  
互联网暴露主机可能受影响；Shodan 上 Henk Poley 的检索结果是超过 **190 万台**  
跑着 PHP 的暴露主机。Calif 另有一份报告提到对 WordPress、Drupal、Joomla、Magento 具体版本的验证。  
  
这两个数字之间这么大的差，本身就说明一件事：**你的资产清单大概率是错的**  
。很多主机跑着 PHP，但没人登记过。远程桌面面板、虚拟主机、遗留的营销站、内部工具的后台，都可能是暴露面。  
### 比 RCE 更难抓的是内存泄露  
  
官方把内存泄露单独拎出来强调了一遍，因为它有个很讨厌的性质：**不需要先拿到代码执行，就能把服务器内存里的敏感数据读走**  
。PHP 进程内存里可能有其他请求、其他用户、甚至服务器自己的密码、认证令牌、API Key 和证书。  
  
官方原话是这么评价的：**这种数据窃取不依赖恶意代码执行，因此针对"恶意代码执行"的检测工具更容易漏掉它。**  
 这一条对防守方的现实意义是——如果你只盯着有没有 webshell、有没有异常子进程，那可能已经在被读了还不自知。  
## 七、扫描工具：php2shell-scanner（非破坏性）  
  
**地址**  
：https://github.com/JoasASantos/php2shell-scanner  
  
这是目前最适合做资产盘点的开源扫描器：纯 Python 标准库，不装任何 pip 依赖，默认不破坏目标，只做版本比对判断。这个定位是对的——你现在要做的是先知道"我有多少台在风险面上"，不是去打未知目标的洞。  
```
git clone https://github.com/JoasASantos/php2shell-scanner.gitcd php2shell-scanner
```  
  
六种扫描模式：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">模式</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">做什么</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">LOCAL</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">本机 </span><code><span leaf="">php -r</span></code><span leaf=""> 读运行时的精确版本、SAPI、</span><code><span leaf="">memory_limit</span></code><span leaf="">、</span><code><span leaf="">post_max_size</span></code></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">REMOTE</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">通过 </span><code><span leaf="">X-Powered-By</span></code><span leaf=""> / </span><code><span leaf="">Server</span></code><span leaf=""> 头和暴露的 </span><code><span leaf="">phpinfo()</span></code><span leaf=""> 指纹定版本</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">PRESENCE</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">只判断有没有 PHP，不定版本（PHPSESSID、</span><code><span leaf="">.php</span></code><span leaf=""> 引用、跳转）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">BANNER</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">裸 socket 稳健抓 banner：HEAD/GET/OPTIONS 加畸形探测，http/https 都覆盖，错误页也算</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">RECON</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">通过 </span><strong><span leaf="">crt.sh</span></strong><span leaf=""> 做子域名枚举，</span><strong><span leaf="">subfinder</span></strong><span leaf=""> 作为补充和合并，然后扇出扫描可解析主机</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">PROBE</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">大 multipart 请求观察 worker 是否不稳定，</span><strong><span leaf="">可能造成 DoS，需要显式开启</span></strong></section></td></tr></tbody></table>  
常用命令：  
```
# 本机运行时自检python3 php2shell_scan.py --local# 单目标，顺带找暴露的 phpinfo()python3 php2shell_scan.py https://example.com --phpinfo-paths# 子域名侦察 + 10 线程扫描 + JSON 报告python3 php2shell_scan.py example.com --recon --threads 10 -o out.json# 只做子域名枚举，不扫描python3 php2shell_scan.py example.com --recon-only# 目标清单 + 强制稳健 banner 抓取python3 php2shell_scan.py -f targets.txt --banner# 压测探针（仅限授权目标，有 DoS 风险）python3 php2shell_scan.py https://example.com --probe --probe-count 5
```  
  
几个实战里特别值得用上的参数：  
- --deep-version  
 —— banner 拿不到版本时的兜底：PHP easter eggs（?=PHPE9568F34-...  
）和版本泄露文件（composer.json  
、.env  
）。.env  
 里的 PHP_VERSION  
 是精确值，而 "php": ">=8.2"  
 这种约束只能缩小分支。  
  
- --origin-scan  
 —— **CDN 会把源站藏起来，读 header 只会读到边缘节点**  
。这个参数会解析 apex、www  
、常见非代理前缀（origin.*  
、cpanel.*  
、mail.*  
）和侦察出的子域名，剔除 Cloudflare / Imperva / Sucuri 的边缘 IP 段（IPv4 和 IPv6 都剔），然后带真实 Host 头和 SNI 直接请求剩下的**源站 IP**  
，从源站读 Server  
 / X-Powered-By  
 / 错误页。配 --shodan-key  
 或 --censys-id  
 加 --censys-secret  
（或环境变量 SHODAN_API_KEY  
、CENSYS_API_ID  
、CENSYS_API_SECRET  
）能拿到 DNS 从不暴露的 IP。  
  
- 判定结论分五档：VULNERABLE  
 / PATCHED  
 / VULNERABLE_EOL  
（低于 8.2，已 EOL 无上游补丁，去查发行版或 OSV 的回补）/ UNKNOWN  
 / SUSPECTED  
。  
  
- 置信度分三档：HIGH  
 来自暴露的 phpinfo 或 PHP 错误页；MEDIUM  
 来自 HTTP banner；LOW  
 只是推断、在代理后面、或发行版可能已回补补丁（会提示你去核对 changelog）。  
  
被动情报这块做得比同类工具细：它会根据 SAPI 和 worker 模型判断——常驻 worker（fpm  
、mod_php  
）会复用请求状态，直接踩 CVE-2026-109619 的雷区，纯 cgi  
 风险低得多；看到 CDN / WAF / 代理会降低置信度，因为 banner 可能是边缘而不是源站；看到发行版构建标记（ubuntu  
、+deb  
、.el8  
）会提示回补可能性。  
  
工具本身有明确授权声明，只对自己拥有或明确获授权的资产运行，--probe  
 模式可能打挂脆弱的 worker。  
## 八、验证工具：php-cve-2026-10961x-lab-poc（实验室复现包）  
  
**地址**  
：https://github.com/dinosn/php-cve-2026-10961x-lab-poc  
  
**先说清楚这个工具是干什么用的：它在隔离实验室里复现漏洞，证明"这条链子确实能走通"，它不是用来打别人服务器的。**  
 仓库自己在安全属性一节写得很明确：专用 Docker 容器跑在内网、不发布任何宿主端口、镜像按 digest 固定、PoC 构造器不接受任意命令、执行 payload 限制为 id  
 / printf  
 这类只往容器内 /tmp  
 写东西的命令、每次会话强制跑负例和受控 abort 对照、宿主机 gdb 只允许挂到指定容器的唯一 worker。  
  
**先看它的实测结果清单**  
（在 PHP 8.5.11 上）：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">发现</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">结论</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">四个普通 HTTP 请求完成自包含 RCE，in-band </span><code><span leaf="">PS(id)</span></code><span leaf=""> → cookie 内存泄露，两条基址来自同一个 512 字节 PHPSESSID 泄露；12/12 命中，uid=33（www-data），跨 8 轮 ASLR</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">已证明自包含 RCE</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">重复 6 字节 multipart header 造成 </span><strong><span leaf="">52 倍内存放大</span></strong><span leaf="">，绕过 </span><code><span leaf="">post_max_size</span></code><span leaf="">；128M 限制下 2.57 MB 的 body 就能 OOM，7.4 到 8.5.11 四个版本全部复现</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">已证明</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">三条利用路径在 8.5.12 安全补丁后仍然存活（</span><code><span leaf="">var_destroy</span></code><span leaf=""> 重入 UAF、序列化处理器不匹配、上传进度注入）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">ASAN 下已证明</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">FPM 陈旧 </span><code><span leaf="">BG(unserialize)</span></code><span leaf="">：worker 在会话解码 OOM 后存活，陈旧状态被下一个请求的 rfc1867 窗口消费，堆形态触发 SIGSEGV</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">机制 + DoS 已证明，RCE 未证</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Content-Type</span></code><section><span leaf=""> 分隔符绕过</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">已证明</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">两条 CVE 链在打了补丁的 8.5.12 上完全失效（GPG 验证过的同配置源码）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">已证明（负例对照）</span></section></td></tr></tbody></table>  
**52 倍放大这块值得单独拎出来说**  
，因为它直接击穿了很多人的心理防线：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><code><span leaf="">memory_limit</span></code></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">触发 OOM 的最小 body</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">放大倍数</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8M</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">153 KB</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">54.8x</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">32M</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">636 KB</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">52.8x</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">128M</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2.57 MB</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">52.3x</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">256M</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5.14 MB</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">52.2x</span></section></td></tr></tbody></table>  
所有 body 都塞得进默认的 post_max_size=8M  
。也就是说"我把上传限制调小了就安全了"这个假设，在未打补丁的版本上不成立。分块传输还能额外绕过 Content-Length 的预检查。8.5.12 的补丁同时关了这两条路（未知 header 过滤 + 流式 post_max_size  
 强制）。  
  
**WAF 签名绕过这条，是给防守方看的**  
：Content-Type: multipart/form-data, boundary=X  
（逗号分隔）PHP 是接受的——SAPI.c  
 在 ;  
、,  
 和空格处都截断。所以**所有锚定在 Content-Type: multipart/form-data; 上的公开签名，全部可以被绕过**  
。如果你现在的 WAF 规则是照着某篇分析抄的，现在就去改。官方 FAQ 也明确回过"WAF、CDN 或主机商的规则能保护我吗"这个问题，答案是"取决于供应商和利用路径，**不要把防护规则当成升级 PHP 的替代品**  
"。  
  
快速开始：  
```
sudo ./scripts/preflight.shsudo ./scripts/reproduce.sh
```  
  
脚本会校验环境、生成 5,767,505 字节的固定 fixture、校准一个安全的 type-0x7 槽位，然后跑"新 worker 负例 → abort → marker"三段流程，每段都有独立的 marker 字节、worker 转换、健康和 trace 检查，最后写出完整的 SHA-256 清单。判定文件在 runs/DATE/VERDICT.json  
，不实际开火也能复验：  
```
python3 scripts/verify_run.py runs/DATE
```  
  
它的"证明边界"一节我建议防守方也读一遍——这个仓库明确说了它**没有**  
证明的事：对加固后的生产部署做单发攻击（多 worker prefork 的可靠性、display_errors=Off  
、高 memory_limit  
 都是实打实的屏障）、FPM 陈旧状态原语能达成 RCE（目前只有崩溃）、以及明确测试版本之外的 PHP（测过的是 7.4.33 / 8.1.34 / 8.4.26 / 8.5.11 / 8.5.12 原版和 3 个 LiteSpeed 构建）。  
  
顺带提醒：现在 GitHub 上和这两个 CVE 相关的仓库鱼龙混杂，有人在拿漏洞做恶作剧投放。别见 CVE 就 clone 就跑。  
## 九、暂时升不了级？这些是止血，不是治病  
  
**PHP 官方说得很清楚：升级是唯一完整的修复手段。**  
 下面每一条都只是降低特定攻击路径的暴露面，没有一条能让未打补丁的安装变安全。官方还专门加了一句：这些配置变更**可能影响应用功能、性能或可用性**  
，务必先在测试环境验证。  
  
针对 CVE-2026-109618：  
- **强制 memory_limit=-1**  
，并且要用应用覆盖不了的方式，比如 FPM pool 或 Apache 配置里的 php_admin_value  
。改完重启所有 worker，并确认没有任何 pool 配置、目录配置或应用代码把有限限制又设回来。 但官方专题站在这条上给了个更重的警告：设成 -1  
 等于把内存耗尽的处理权交给操作系统，内存耗尽的 worker 可能被直接终止，正在处理的请求会被打断，服务器稳定性和可用性都会受影响。如果决定这么做，用 **Linux cgroup v2 的 memory.max**  
（或等价的操作系统机制）把资源消耗圈在 PHP 进程范围内。另外在多线程部署里，终止一个进程可能连带打断其他线程上正在跑的请求。这条**也不覆盖**  
应用特有的大额分配路径和非内存相关的触发方式。  
  
- **绝对不要用 RLIMIT_AS 或 RLIMIT_DATA**  
。PHP 公告明确警告它们会触发同一条脆弱的失败路径，等于亲手把攻击者想要的东西递过去。  
  
- post_max_size  
 保守设置：不需要文件上传的应用设 **8M**  
，需要上传的设 **32M**  
，并保证 upload_max_filesize  
 低于 post_max_size  
。  
  
- max_input_nesting_level  
 能调低就调低，但复杂表单可能需要更高的值，要测。  
  
- 在 web server 或代理层独立强制**解码后**  
的请求大小、输入字段数、multipart part 数、每个 part 的 header 数和嵌套深度。**这些限制必须覆盖没有有效 Content-Length 的请求。**  
 如果应用根本不用 multipart form data，在前置层直接拦掉。  
  
针对 CVE-2026-109619：  
- **session.upload_progress.enabled=0**  
，写进 php.ini  
、FPM pool 或服务器配置，**必须在请求解析前生效**  
。不要依赖应用层的 ini_set()  
，也不要用 FPM 的 .user.ini  
——后者是在正常初始化之后才应用的。改完重启所有 worker。补一句：就算打了补丁，如果用不到这个功能，也建议关掉。  
  
- **fatal_error_backtraces = 0**  
（在支持的版本上，通常是 PHP 8.5），管理级配置，重启所有 worker。  
  
- 清掉所有会在请求初始化前产生诊断的每请求无效或废弃设置。用 error_reporting  
、display_errors  
 或 @  
 压制消息**不解决底层访问问题**  
。  
  
- 修掉攻击者能触发的致命错误和执行超时，包括回调里、上传处理里、导入流程里、错误页、析构、序列化、输出缓冲、插件和扩展清理路径里的失败。  
  
- PHP-FPM 如果能接受性能损失，可以考虑降低 pm.max_requests  
，让 worker 更频繁地回收，减少被污染状态跨请求存活的窗口。有性能成本，也不覆盖不依赖多请求的机制。  
  
另外两条纯粹的减损措施：  
- **生产环境设 display_errors = 0**  
 并重启 worker，防止 PHP 错误输出直接暴露给远程用户。这只是减少一条信息泄露路径，不修复漏洞本身。  
  
- **在兼容的前提下用 SELinux 或 AppArmor 策略**  
限制 worker 的能力和程序执行范围。这能约束影响面，但别把它当成能挡住所有利用或内存泄露的东西。  
  
兜底手段：CERT-EU 建议实在升不动的组织，**先把面向公网的应用隔离或下线**  
，直到修复完成并验证。按 Qualys 的说法，给每个待修系统指定责任人和截止时间，别让它在待办里躺着。---  
## 十、被人打过了吗？官方给的调查线索  
  
这是这次事件里最实用的一节。官方在 FAQ 里直接给了排查线索：  
- **异常巨大或畸形的 PHPSESSID 响应 cookie**  
；  
  
- **异常的 multipart 请求，紧接着出现内存限制错误，或者重复的 HTTP 500 响应**  
；  
  
- **无法解释的 PHP worker 崩溃或重启**  
。  
  
但官方加的限定条件比线索本身更重要：  
> 这些信号可以指引调查方向，但**它们本身不能证明系统被利用了，它们的缺失也不能证明系统没受影响**  
。有些攻击可以在服务器上不执行任何恶意代码的情况下就泄露敏感内存；而且因为脆弱的路径跑在应用层之下，应用日志里可能几乎没有任何上下文。  
  
  
所以结论还是那句：**不管有没有观察到可疑活动，都要打补丁。**  
## 十一、日志审计与网络检测  
### 开源检测规则  
  
Corelight Labs 在协调披露当天开源了 Zeek 和 Suricata 的检测包：**https://github.com/corelight/php2shell-detect**  
  
Zeek 包不做特征匹配，走行为分析，一共九类 notice：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">Notice</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">告诉分析员什么</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">PHP_Crash_Sequence</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">有限时间窗内，同一会话关联到多条可疑 HTTP 事务</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Multipart_Header_Flood</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">multipart 请求里有大量重复的已知 MIME header 模式</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Large_Form_Key</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">表单字段名异常长</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Serialized_Integer_Overflow</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">序列化整数 token 长度异常</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Serialized_Integer_Collisions</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">请求体命中已知序列化整数键模式</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Serialized_String_Collisions</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">请求体命中已知序列化字符串键模式</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Serialized_Duplicate_Array</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">请求体命中已知序列化数组条目的重复</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Serialized_Object_Repetition</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">请求体命中已知序列化对象的重复</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Potentially_Affected_PHP_Version</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">HTTP 响应头暴露出落在受影响区间的 PHP 版本</span></section></td></tr></tbody></table>  
最后一条特别实用：不管有没有可疑请求，光靠解析网络流量里的 Server  
 和 X-Powered-By  
 头就能做出**版本清单**  
，先把该排期打补丁的主机捞出来。  
### 日志怎么查  
  
隔离审计期间，在启动级配置里开 log_errors=On  
、配一个能用的 error_log  
、error_reporting=E_ALL  
，然后在 Apache、PHP-FPM、反向代理和服务日志里找这些：  
- 在目标脚本执行**之前**  
反复出现的 warning、deprecation 和核心错误；  
  
- 扩展或服务器清理阶段产生的 warning 和其他诊断；  
  
- fatal、解析失败、未捕获错误消息；  
  
- Maximum execution time exceeded  
；  
  
- 只在攻击者能控制的路由、内容类型、文件、回调实现或插件选项上出现的失败。  
  
修掉观察到的错误能降低暴露面，但日志干净**不等于**  
安全。  
## 十二、升级完成不等于修复完成  
  
装完包只是第一步，官方专题站和 Qualys 都给了验证清单：  
1. **找齐所有 PHP 运行时**  
——并行的多个分支、容器镜像、主机控制面板自带的包、捆绑在其他软件里的 PHP。php -v  
 输出的版本很可能不是真正在服务 Web 请求的那个。官方说得很直白：**更新应用不会更新它底下的 PHP 运行时。**  
  
1. **更新正确的包、二进制或镜像**  
，然后**重启 Apache、PHP-FPM 和所有长驻 PHP worker**  
。发行版包要看完整包版本对照厂商公告，因为回补补丁可能保留旧的上游版本号；容器部署要拿打过补丁的镜像，或者用打了补丁的依赖重新构建。Linux 发行版和 LTS 厂商提前拿到了补丁，所以以自己厂商的公告为准。  
  
1. **确认没有 worker 还跑在旧构建上。**  
 换句话说，框架代码更新不解决问题——漏洞在 PHP 运行时里，修复需要的是新的 PHP 构建：重建镜像、重新跑一次 setup-php  
，或者更新发行版包。  
  
1. **评估新硬内存限制的运行影响。**  
 补丁引入了一个新的硬内存上限，触发时进程会被直接终止，**shutdown 处理器不会执行**  
。依赖 shutdown 做清理的应用必须专门验一遍。官方建议升级后重新审视 memory_limit  
、请求大小限制和预期内存用量，把升级前临时设的 memory_limit=-1  
 按 PHP 最终建议调回来，并考虑 expose_php = 0  
 减少版本泄露（这是可选加固，不修复漏洞）。  
  
1. **复查应急配置**  
，撤掉临时的上传限制和 worker 回收设置。  
  
## 十三、现在就去升级  
  
再说一遍，因为这是唯一重要的事：  
  
**PHP 8.5.12、8.4.27、8.3.36、8.2.35。今天就升。**  
- 跑 8.1、8.0、7.4 的，官方只给了最小源码补丁，要么找发行版或 LTS 供应商拿回补版本，要么直接升到受支持分支。  
  
- 跑 5.4 到 7.3 的，PHP 官方明确不再出补丁，**只能升级**  
。  
  
- 用 rpms.remirepo.net  
 这种志愿者维护仓库的，官方直接喊话让你切回发行版包或源码构建，那个仓库当天不会有更新。  
  
- 找完资产、打完补丁、验证完运行时的实际版本之前，这件事不算完。  
  
- 暂时升不动的，先按第九节做配置止血，同时按 CERT-EU 的建议把公网暴露面隔离掉。每一台待修的系统都要有责任人和截止时间。  
  
漏洞长在 PHP 内核里，你的应用写得再干净也挡不住，一个 Hello World 也照样中招；配置改得再细也只是把一扇窗关上，补丁才是把整面墙换掉。现在是 10 月 22 日，野外已经有利用尝试在跑，而且这次连内存泄露路径都不需要先拿到代码执行。每多拖一小时，风险就多累积一小时。  
  
**素材出处**  
- 官方专题站（四方联合研究主页）：https://www.php2shell.com/  
  
- PHP 官方安全公告（2026-10-21）：https://www.php.net/  
  
- CVE-2026-109618 官方 GHSA：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
- CVE-2026-109619 官方 GHSA：https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
- 扫描工具：https://github.com/JoasASantos/php2shell-scanner  
  
- 实验室复现包：https://github.com/dinosn/php-cve-2026-10961x-lab-poc  
  
- 开源检测规则：https://github.com/corelight/php2shell-detect  
  
- Wiz 技术分析：https://www.wiz.io/blog/php2shell-php-rce  
  
- Calif 技术分析：https://calif.io/php2shell/  
  
- OpenAI 协调披露：https://openai.com/index/coordinated-disclosure-of-critical-vulnerabilities  
  
- PHP Foundation 公告：https://thephp.foundation/blog/2026/10/21/todays-php-security-release-and-how-were-strengthening-php-security/  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OZEV0U2eT60qsRA4Keox5O1JhOjbuueXNYYfJvX7Z5Yy0dibgZecvLCP2mHn5U18K5YGY4ubicAO5xwKhpqmFwUO7JgcSNHhELg/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
