#  一夜拔网线：php2shell 双 RCE 全分析  
原创 Attacker安全
                    Attacker安全  Attacker安全   2026-10-22 09:02  
  
10 月 21 日晚上，欧洲的运维们大概没怎么睡觉。  
  
芬兰政府把所有对外网络服务关了。荷兰更狠，国家网络安全中心把自己的官网都撤了，DigiD——那个一千多万人日常用的电子身份系统——直接预防性下线，鹿特丹港、荷兰足协、五大政党的网站和多家医院跟着关。法国内政部通知各省政府网站暂时下线，数据保护机构 CNIL 也撤了。德国这边，Charité 医院整个网站关停，柏林市门户 berlin.de 部分功能禁用，柏林自由大学限制 IT 服务，柏林警察的在线报案门户打不开，巴伐利亚、下萨克森好几个州的政府和警察网站同样不可用。比利时外交部、英国预算责任办公室、维基媒体基金会、任天堂，都动了。  
  
据路透社和德新社的报道，名单里还有丹麦。日本那边，财务省、pixiv、Niconico 也进了紧急维护。  
  
这些网站有一个共同点：**没有被黑，是自己关的。**  
> 起因是 PHP 内核的两个漏洞：CVE-2026-109618 和 CVE-2026-109619，安全圈起了个外号叫 php2shell。CVSS v4.0 双双 9.2，无需登录、无需交互、默认配置，一个精心构造的请求就可能拿到服务器的 shell。  
  
  
我在态势感知平台上见过不少高危告警，但能让好几个国家的政府在同一天主动拔网线的洞，这些年只有 Log4Shell 够排面。而 php2shell 阴险的地方，比 Log4Shell 还多一层。这篇文章把三件事讲清楚：漏洞到底怎么回事，欧美为什么选择关站而不是硬扛，以及这事跟每个跑 PHP 的人有什么关系。  
  
## 一、先说结论：这是个什么级别的洞  
  
<table><tbody><tr style="background: #1B4B8F;color: #fff;"><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf="">项目</span></section></th><th style="padding: 8px 12px;border: 1px solid #1B4B8F;text-align: left;"><section><span leaf="">内容</span></section></th></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">CVE 编号</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">CVE-2026-109618、CVE-2026-109619</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">漏洞类型</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">远程代码执行（RCE），内存破坏类</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">严重程度</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">CVSS v4.0 9.2（Critical），watchTowr 直接打了满分</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">受影响组件</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">PHP 语言核心（php-src），不是任何框架、插件或 CMS</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">影响版本</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">PHP 5.4 起全部版本（Cloudflare 说法），直至各分支修复版之前的全部版本</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">修复版本</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">PHP 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35；7.4、8.0、8.1 有最小源码补丁</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">披露时间</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">2026-10-21，细节提前一天泄露</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">利用条件</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">网络可达、无需认证、无需交互、默认配置即可</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">暴露面</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">所有处理 Web 请求的 PHP：PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed</span></section></td></tr></tbody></table>  
一句话概括官方 advisory 的口径：**所有对外提供服务的 PHP 安装都在射程之内**  
，其中 Apache mod_php 风险最高——已有验证完整的利用程序，未授权请求可以在任何应用代码运行之前就触发攻击。翻译成人话：你的 WordPress 装了多少安全插件、框架中间件写了多少登录校验，在这条攻击链面前全部失效。等它们加载起来，仗已经打完了。  
  
## 二、漏洞分析：根子在 PHP 的错误处理机制  
  
  
这次出问题的不是某个插件的代码质量，是 PHP 处理"出事了怎么办"的方式本身。两个漏洞，同一个根因：**bailout**  
。  
  
什么是 bailout：一场没有收尾的紧急撤离  
  
PHP 遇到致命错误或者内存耗尽（OOM）的时候，不会沿着函数调用栈一层一层正常返回，而是用一个叫 longjmp()  
 的机制直接跳出去，社区管这个动作叫 bailout。打个比方：厨师炒菜炒到一半被紧急叫走，火没关，半成品还留在灶台上。下一个人不知情，端起那口锅接着用。bailout 跳过的，正是"收拾现场"这一段。  
  
CVE-2026-109618：内存耗尽留下的半成品  
  
PHP 在准备请求数据的过程中内存耗尽，被打断时，请求的内部数据只初始化了一半。随后的清理阶段，却把这份半成品当成完整数据来用。关键在于时序：这条路径在**请求解析阶段**  
就能触发，早于任何应用代码。官方在 Apache mod_php 上、默认的输入与内存限制下，验证了完整 RCE。PHP-FPM 经源码分析同样受影响，但截至 advisory 发布时没有演示完整利用。  
  
CVE-2026-109619：跨请求的野指针  
  
worker 进程里有一批全局字段，记录错误状态、会话状态等，它们指向当前请求的内存。bailout 之后这块内存被释放了，指针却还留在那里。同一个 worker 处理的下一个请求，可能在正常初始化把它重置掉之前，就用上了这份残留状态——典型的释放后重用（UAF）。补丁的变更记录只有一行：Avoid state corruption via bailout。做法是两件事：把 OOM 错误暂存起来，等虚拟机走到下一个安全点再抛出；同时在请求结束时清理残留的全局状态。  
  
为什么"同一个 worker"这么关键  
  
PHP-FPM 和 mod_php 的 worker 是长驻进程，一个 worker 要处理成百上千个请求，残留状态因此能跨请求存活。这也解释了官方的提醒：多 worker 部署和 worker 回收会降低单次请求的成功率，但不会消除这个条件——攻击者只需要多发几次。已确认的路径之一涉及 session.upload_progress（会话上传进度）机制，而这个功能默认是开着的。  
  
为什么改配置救不了自己  
  
PHP 8.6 发布经理的原话是：ini settings won't save you。没有任何 php.ini 指令能关掉这两条脆弱路径，disable_functions  
 拦不住，安全模式早没了。唯一完整的修复，是更新 PHP 这个二进制本身。  
  
官方的临时缓解（明确说了"不完整"，只降低特定路径暴露面）：  
<table><tbody><tr style="background: #1B4B8F;color: #fff;"><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf=""><span textstyle="" style="color: #000000;">措施</span></span></section></th><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf=""><span textstyle="" style="color: #000000;">针对</span></span></section></th><th style="padding: 8px 12px;border: 1px solid #1B4B8F;text-align: left;"><section><span leaf=""><span textstyle="" style="color: #000000;">说明</span></span></section></th></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">session.upload_progress.enabled = 0</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">619</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">必须在请求解析之前生效的配置层设置，代码里 ini_set() 或 .user.ini 已经太晚</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">memory_limit = -1</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">618</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">用 FPM pool 或 Apache php_admin_value 设，确保应用改不回来；不要用 RLIMIT_AS / RLIMIT_DATA 替代，反而会触发脆弱路径</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">压低 post_max_size</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">618</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">不上传的业务设 8M，上传的设 32M，保持 upload_max_filesize 小于 post_max_size</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">fatal_error_backtraces = 0</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">619</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">仅 PHP 8.5 上有效，只覆盖已确认的 backtrace 路径</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">代理层限制请求体</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">通用</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">限制解码后请求大小、字段数、multipart 分段数、头部数与嵌套深度，覆盖没有 Content-Length 的请求</span></section></td></tr></tbody></table>  
注意一个反直觉的点：关掉 memory_limit 只是消除了那个可被攻击者预测和控制的 PHP 内存上限失败点，底层分配失败仍可能到达脆弱路径。所以官方话说得很死——**不存在完整的缓解措施，更新是唯一安全的选择。**  
  
最阴的地方：日志是干净的  
  
CrowdStrike 有一段描述，值得每个靠日志排查的人抄在墙上：  
> 没有认证需要绕过，没有会话需要窃取，没有脆弱端点需要发现。Worker 进程一切正常，返回 HTTP 200，应用日志记录不到任何异常——因为你的应用代码根本没被执行。服务器端唯一的痕迹，是错误日志里一条"内存限制致命错误"，而这条日志平时早就被当成噪音忽略掉了。  
  
  
也就是说：服务器被拿了，Web 日志干干净净，告警平台鸦雀无声。等发现的时候，WebShell 可能已经躺了很久，横向移动可能已经完成。做态势感知的都知道，这种"应用层无痕"的攻击，最后靠的是进程行为和网络流量被抓出来，不是靠翻访问日志。  
  
## 三、攻击面：数字摆在这里  
  
  
PHP 驱动着能识别出服务端语言的网站里约七成。WordPress、Drupal、Joomla、Magento、Laravel、Symfony、MediaWiki、phpBB，以及国内各类 CMS、论坛、电商和自建业务系统，全部跑在它上面。间接受影响的应用名单可以列一屏。  
  
VulnCheck 对公网暴露 PHP 主机的测绘：  
<table><tbody><tr style="background: #1B4B8F;color: #fff;"><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf="">版本区间</span></section></th><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf="">规模</span></section></th><th style="padding: 8px 12px;border: 1px solid #1B4B8F;text-align: left;"><section><span leaf="">处境</span></section></th></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">PHP 3.x – 7.3</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">约 70 万台</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">早已 EOL，永远等不到补丁</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">PHP 7.4 / 8.0 / 8.1</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">约 40 万台</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">刚停止官方支持，只有最小源码补丁</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">PHP 8.2 – 8.5</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">约 40 万台</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">受支持分支，有正式修复版本</span></section></td></tr></tbody></table>  
另外，安全公司 Mallory 统计，补丁发布前一天，互联网上约有 **88 万台服务器**  
还在对外显示 PHP 8.1 或更老的版本号。对它们来说，"理论上有漏洞、实际上没人会打"的那份安全感，这次没有了。  
  
## 四、混乱的 48 小时：从泄露到关站  
  
  
这个漏洞的时间线，值得完整看一遍，因为它几乎集齐了漏洞应急管理能踩的所有坑。  
  
**它一点都不新。**  
首个发现它的安全公司 Calif 在技术文章里，把源头追溯到 2004 年——安全研究员 Stefan Esser 就发现过 PHP 在内存耗尽时的同类问题。2020 年有人提交过相关 bug 报告，几乎没人理会。2024 年一次第三方资助的 PHP 安全审计又碰到了相关问题，但因为没能证明它真的可以被利用，最后没有修。这个漏洞在 PHP 里躺了二十多年，不止一次被人摸到边。  
  
原因不是没人想到，是太费劲。独立作者 Nick Rogers 的说法：把这类漏洞做成能用的攻击，需要"数周枯燥的手工调试"，过去没有人愿意为它付出这个成本。他给这种状态起了个名字：**security by exhaustion——靠攻击者累到放弃来保证安全。**  
  
改变这件事的是 AI。今年 6 月，OpenAI 启动 "Patch the Planet" 计划，出钱出模型，请安全公司审计那些被全世界依赖的开源软件。Calif 是参与者之一，8 月 1 日开始带着 AI 审计 PHP：AI 梳理内核代码、追踪执行路径，第一个漏洞的关键缺陷就是 AI agent 在内核里发现的。8 月 11 日 Calif 把漏洞报给 PHP，随后让 AI 把第一个漏洞的模式总结成一份可复用的技能文件——第二个漏洞就是靠这份文件找到的。AI 找到一个漏洞之后，可以把找到它的方法打包，再去找下一个。  
  
几乎同一时间，另一家安全公司 Wiz 也撞上了同一组漏洞：8 月 19 日，Wiz 的研究员借助自家工具加 Gemini 4 Argon 和 GPT-5.6 Sol，独立发现了第一个漏洞；第二天在 Drupal 上验证 RCE；第三天在 WordPress 上验证成功。Wiz 的结论是：一名研究员一周完成了过去需要世界顶尖团队数月的工作。  
  
接下来是彻底失控的一段：  
<table><tbody><tr style="background: #1B4B8F;color: #fff;"><th style="padding: 8px 12px;border: 1px solid rgb(27, 75, 143);text-align: left;"><section><span leaf="">时间（UTC）</span></section></th><th style="padding: 8px 12px;border: 1px solid #1B4B8F;text-align: left;"><section><span leaf="">事件</span></section></th></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-17</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">AWS 威胁情报团队在蜜罐 MadPot 上检测到一次利用尝试，攻击源经商业匿名网络转发。此时距预定披露日还有 5 天，攻击者怎么提前拿到利用方法的，至今成谜</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-20</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">Hacker News 和地下论坛 BreachForums 出现消息，watchTowr 几小时内确认漏洞为真并复现成功；当晚 Calif 误发演示视频，攻击代码在多个真实目标上跑通，副本疯传</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-20 晚</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">PHP 官方被迫把补丁发布提前 24 小时</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-21 02:00</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">AWS 检测到第一批 PoC 利用尝试，6 小时内多个匿名 IP 蜂拥而至</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-21 早</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">提前拿到补丁的发行商 Chainguard 单方面打破保密协议，抢先发布修复镜像——补丁本身就是一份&#34;漏洞地图&#34;，对比新旧二进制就能反推原理</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-21 08:36</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">安全厂商 ELLIO 的传感器捕获到首个攻击请求，距补丁发布不到 1 小时</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-21 14:00</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">PHP 发布 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35</span></section></td></tr><tr><td style="padding: 8px 12px;border: 1px solid #E3E9F4;background: #EAF1FB;font-weight: 700;color: #1B4B8F;"><section><span leaf="">10-21 当天</span></section></td><td style="padding: 8px 12px;border: 1px solid #E3E9F4;"><section><span leaf="">Cloudflare 观测到数百万次攻击尝试（其 CTO 在补丁发布前近 6 小时就公开表示&#34;早期攻击正在出现&#34;）；WebOps 平台 Pantheon 单日拦截超 6 万次探测；VulnCheck 发现攻击者直接用 php2shell-scan/109618 当 User-Agent 扫描，连掩饰都懒得做</span></section></td></tr></tbody></table>  
到底有没有打成的，各方说法还不一致：亚马逊称漏洞已在野被广泛利用，heise 发稿时则还没有成功攻击的确认报告。但这种"没确认"，很多时候只是"还没发现"。  
  
## 五、欧美为什么选择"拔网线"  
  
  
漏洞出来后，美国国安局（NSA）、英国国家网络安全中心（NCSC）、欧盟 CERT-EU 给出的建议几乎一致：**来不及打补丁的系统，先下线。**  
德国联邦信息安全局（BSI）同样如此建议，CISA 表示认同。于是就有了开头那一幕。  
  
站在运维角度，这笔账并不难算。  
  
**第一，补丁覆盖不到全部。**  
官方只为 8.2 到 8.5 四个受支持分支发布了正式修复，7.4、8.0、8.1 只有最小源码补丁，5.x 和 7.3 什么都没有。那 88 万台老版本服务器，升级意味着动业务，不动就是裸奔。Remi 仓库当天也没有更新，用它的用户被建议切发行版包或自行编译。  
  
**第二，利用门槛已经归零。**  
PoC 公开、演示视频流传、扫描器 UA 印着漏洞编号、补丁发布 1 小时内攻击就到。这不是"可能有攻击者盯上"，是"全世界都能扫"。Cloudflare 给所有套餐包括免费版都部署了 WAF 规则，同时观察到攻击者不断变形载荷，在实测防护边界在哪里——权宜之计，扛不了多久。  
  
**第三，中了看不出来。**  
前面说过，日志是干净的。对于一个政务门户或者医院网站，你甚至无法确认自己有没有被拿下过。在这种不确定性面前，"先关几天"是唯一能给出确定性答案的动作。  
  
关站的损失是业务停几小时；不关的损失是整台服务器，加上以服务器为跳板的内网横向，加上数据泄露，加上勒索软件。Charité 是欧洲最大的大学医院之一，这个选择并不难做。后续也有务实的一面：Debian、Fedora、Ubuntu、Red Hat、Gentoo、Slackware 的更新包陆续到位，德国北部部分网站已经恢复上线。先拔线，再修复，修复完再回来——这套动作本身就是应急响应该有的样子。  
  
## 六、这事跟每个跑 PHP 的人有什么关系  
  
  
**"存在感太低所以安全"的时代结束了。**  
security by exhaustion 保护了互联网底层大量的老代码：理论上有漏洞，但真要打起来太费劲，所以一直放着。AI 把找漏洞、验证漏洞、把攻击适配到不同环境的成本，从"顶尖团队几个月"降到"一个人一周"甚至"几个小时"。过去那些"太累了不值得"的漏洞，现在全都值得做。Rogers 的说法是，AI 第一次把网络攻击的边际成本推向了零。  
  
**防守窗口从两个月压缩到了几小时。**  
这次原本有两个多月的协调披露期，一次提前泄露，压缩成了几个小时。而修补仍然要一台服务器一台服务器地做。这个不对称，未来只会更严重。  
  
**攻击者手里的 AI，和防守方用的是同一代模型。**  
Calif 特别说明，最初发现漏洞时没有使用特殊工具，也没有使用普通人拿不到的模型，用的是 GPT-5.6 Sol 这样更便宜的模型。防守方用公开模型能找到它，攻击方同样可以。watchTowr 在诱饵服务器上观察到的攻击尝试，用的工具部分就是借助 AI 写出来的。  
  
落到实操，给还在跑 PHP 的团队几条可以直接抄的作业：  
- **清点，而且要比你以为的更细。**  
全量梳理对外提供服务的 PHP 主机、容器与镜像，建立版本台账，优先处置直连互联网的。特别留意隐藏实例：一键面板环境里 PHP 版本常和业务实际用的不一致，Docker 的 php-fpm 镜像、虚拟主机、子站点、第三方 CMS 配套系统，都容易漏。荷兰政府就是全量排查之后才决定"先下线再修复"的。记住：更新 WordPress 插件、跑一次 composer update，都不会更新服务器上的 PHP 运行时，这是两件事。  
  
- **升级到 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35，然后重启每一个 worker。**  
旧代码在进程里一直有效，不重启等于没打补丁。一台机器上并存多个 PHP 版本时（面板环境很常见），逐个确认、逐个升级，用 php -v 或 phpinfo 逐实例核对，别只看包管理器的返回值。先在测试环境验证，留好回滚。  
  
- **升不了的，先按第二节的临时措施压暴露面**  
，并尽快排期。托管环境（虚拟主机、共享主机、云平台）找服务商确认 PHP 运行时是否已修补——Laravel Cloud 已自动修补，Forge 需手动升级，Vapor 自动重新部署。  
  
- **监控埋点换个思路。**  
这种应用层无痕的攻击，重点盯：PHP worker 异常退出或重启、Web 进程拉起的异常子进程、HTTP 500 尖峰、异常大的 PHPSESSID cookie、非预期文件写入、可疑外联、错误日志里的"内存限制致命错误"。对边界侧来说，防火墙和 EDR 的进程/流量告警，比 Web 访问日志更值得先看。  
  
- **日志留存。**  
保留 Web 服务器与 PHP 日志。日志里没有相关条目，不等于这套环境是安全的——这句话这次要反过来读：正因为痕迹可能只有一行被当成噪音的报错，留存和基线比对才有意义。  
  
- **把运行时版本纳入常态化资产台账。**  
框架和插件有扫描器盯着，PHP 本身往往没人管，而它恰恰是这次出问题的地方。  
  
## 写在最后  
  
  
PHP 基金会的 Volker Dusch 把这个洞和 2014 年的心脏出血（Heartbleed）相提并论，说这是十多年来最严重的 PHP 安全问题。watchTowr 给了满分。这些评价都不夸张。  
  
但更值得记住的是另一件事：这是第一次，一个由 AI 挖出来的漏洞，让好几个国家的政府在同一天关掉了自己的网站。二十多年没被打穿的漏洞，AI 几周就打通了；过去靠"太累了"保护着的互联网底层，这层保护正在消失。下一个 php2shell 出现的时候，不会有人提前通知你。你的服务器升没升，取决于你现在有没有去查那台你可能都不知道存在的 PHP。  
  
  
