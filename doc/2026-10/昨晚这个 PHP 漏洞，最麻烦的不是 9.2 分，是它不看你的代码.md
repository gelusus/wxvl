#  昨晚这个 PHP 漏洞，最麻烦的不是 9.2 分，是它不看你的代码  
原创 simeon的文章
                    simeon的文章  小兵搞安全   2026-10-22 02:12  
  
昨晚十点（北京时间 10 月 21 日 22:00，UTC 14:00），PHP 官方发布了紧急安全更新：两个 CVSS v4.0 均为 9.2 分的未认证远程代码执行漏洞，编号 CVE-2026-109618 和 CVE-2026-109619，研究圈给这对组合起了个名字，叫   
**php2shell**  
（PHP 官方公告原文见文末来源）。  
  
一晚上过去，你的时间线上大概已经刷过五六篇通告了。CVE 编号、影响版本、升级命令，每篇都差不多。所以这篇不重复通告，讲几个通告里没有的判断。这些判断，决定了你的服务器是"打了补丁就完事"，还是"这三周天天悬着"。  
## 1.先把事实摆清楚  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n21" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">项目</span></span></span></th><th data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n22" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">内容</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n24" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞编号</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n25" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVE-2026-109618（OOM 期间请求内存未初始化，清理阶段被当作有效数据读取）/ CVE-2026-109619（陈旧全局状态在内存释放后仍被同一 worker 的后续请求复用）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n27" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">官方定性</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n28" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">均为</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未认证远程代码执行</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，CVSS v4.0 </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">9.2</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，无需登录、无需交互</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n30" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">打到什么程度</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n31" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">官方确认影响是&#34;</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">以 Web 服务账号权限执行任意代码</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">&#34;，附带堆信息泄露；不是&#34;能读个文件&#34;级别</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n33" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">影响</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n34" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">所有处理 Web 请求的 PHP：PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed；</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">官方认为 mod_php 风险最高</span></span></strong></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n36" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">修复版本</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n37" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8.5.12 / 8.4.27 / 8.3.36 / 8.2.35</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（2026 年 10 月 21 日 14:00 UTC 发布）；PHP 8.6.0 RC4 也已包含修复</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n39" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8.1 及更老</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n40" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">官方不再发版，但为 </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">7.4 / 8.0 / 8.1 提供了最小化源码回溯补丁</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">；发行版与 LTS 厂商可能已 backport</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n42" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">官方态度</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n43" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">原话：&#34;</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">不存在完整的缓解措施，升级是唯一安全的选择</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">&#34;</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="142" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n45" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 72.0739px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">现状</span></span></span></td><td data-colwidth="423" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n46" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 838.134px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">细节与可用 PoC 在 10 月 20 日</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">提前泄露</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（原定 22 日揭露）；有监测厂商记录到补丁公开前五个半小时的攻击请求</span></span></span></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/mmbiz_png/dMqqFicgqDGBTjUjt7WeMfQztkn8p3IfjV36CBYH7icYN7y69oWoicVh41M7wXXzUxF6mcI4dHvWz3rmiaI9E4FVVobUcnicibWIMxCr64uHm4OPA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dMqqFicgqDGDFSCIVvPibiby2y8wrJoGZqxQLAeRl5SeialRSs6gYVtXz9XiaR0zslYC1nmLYzkBbFR4wxIrsSTRfCnBCxQ9Lbp5osHNf7sbdiaYM/640?wx_fmt=png&from=appmsg "")  
  
  
时间线值得记一笔，因为它是这次事件性质的来源：  
- **8 月 27 日**  
：多个研究团队的报告送到 PHP 安全团队；  
  
- **9 月 25 日**  
：修复 PR 开出，最后一次提交在   
**10 月 18 日**  
；  
  
- **10 月 20 日**  
：细节与可用 PoC 在 Hacker News 泄露，官方当天发预通告，把发布提前一天；  
  
- **10 月 21 日 08:36 UTC**  
：有厂商蜜罐收到第一个 php2shell 请求——  
**比补丁公开早五个半小时**  
；  
  
- **10 月 21 日 14:00 UTC**  
：四个修复版本发布。  
  
**计划内的协调披露，变成了计划外的抢跑。这三周的窗口期就是这么来的。**  
## （一）判断一：它在你的业务代码之前就结束了  
  
官方公告里有一句话，是这次事件最该被引用的：mod_php 的风险最高，因为  
**已演示的攻击可以在任何应用代码运行之前触发**  
。  
  
翻译一下：你花预算做的代码审计、登录加固、参数过滤、业务逻辑层的鉴权——这些应用层的东西，在这个漏洞面前全部不参与。请求到 PHP 引擎那一层就结束了，你的业务代码还没开始跑。所以很多单位自查时说"我的代码是外包写的、我用的开源系统没这漏洞"——  
**问错了对象。这次坏的不是代码，是发动机。**  
  
这个"抢在业务代码之前"的结论，官方是在   
**mod_php**  
 上完整演示的；FPM 上前置程度弱一些，具体分层放在判断三。  
  
但另一头也得说清，否则容易走到"什么都没用"的极端：官方给出的临时措施里，明确包括在 Web 服务器或代理层限制请求体大小、字段数、multipart 分片数、header 数量和嵌套深度——也就是说，  
**网络层对"请求形状"的限制仍然有效**  
。Cloudflare 也针对已知攻击向量上了 WAF 规则，但同时说明那只是权宜之计，覆盖不到走别处的流量。  
  
准确的说法是：  
**应用层的"逻辑型"防护全部失效，网络层的"形状型"限制还能减压。**  
这两句话差一个字，处置动作差很多。  
## （二）判断二：两个洞的开关是反的，很多人第一反应会调错  
  
这是本篇最想讲的一点，因为我核对时发现，第一批中文报道里就有把两个洞的缓解措施混着写的。  
  
**109619（陈旧全局状态）**  
：走的是"上一次请求留下的状态，被同一 worker 的下一次请求读到"。已确认的读取路径有两条，一条是 PHP 8.5 上的致命错误回溯，一条是  
**会话上传进度**  
。  
  
这个"会话上传进度"值得单独说：它从 PHP 5.4（2011 年）起默认开启，作用是给上传页面显示进度条。而官方公告里自己写了一句很实在的话——  
**这个功能"很少被使用和测试"**  
。一个开了十四年、很少有人用、连官方都承认很少被测的默认开关，昨天成了入口之一。  
  
所以 109619 的临时措施是：  
session.upload_progress.enabled=0  
，PHP 8.5 再加   
fatal_error_backtraces=0  
，  
**改完必须重启所有 worker**  
（不能靠   
ini_set()  
 或   
.user.ini  
，那些生效太晚）。  
  
**109618（OOM 路径）**  
：它的开关方向是  
**反的**  
。这个洞的触发前提是"请求能在你的内存限额内把内存耗尽"，所以官方临时措施的第一条是——  
> **把 memory_limit 设为 -1。**  
  
  
不是调小，是  
**放开**  
。而且要确保没有任何池、目录或应用的设置又把它设回有限值。  
  
我知道这条反直觉到什么程度：绝大多数安全加固清单里写的都是"限制内存、限制上传大小"，而这次内存要放开。所以你会看到两种人：一种是照着老经验把   
memory_limit  
 从 256M 调到 64M，觉得更安全了，其实正好把可利用窗口留得更宽；另一种是看到官方这条直接照做。  
**同一份通告，两种读法，处置效果相反。**  
  
post_max_size  
 那条才是收紧（无上传 8M / 有上传 32M），而且官方特意提醒：  
**只限总大小不够**  
，还得限字段数、分片数、header 数，否则请求体不超限也能把内存吃掉。  
  
一句话记法：  
**老经验防的是"太大"，这次要防的是"刚好卡在限额上"。**  
## （三）判断三：9.2 分不等于到处都是 9.2 分  
  
这条是通告不会写的，但办案的人会先问。  
  
CVSS 9.2 是官方给的分数，可它的向量里写着  
**攻击复杂度为高（AC:H）**  
，还带一个"攻击前提条件存在（AT:P）"——意思是这个洞要打穿，得凑齐一些部署条件。具体分层是这样的：  
- **Apache mod_php**  
：官方确认在默认输入与内存限制、常规 prefork 配置下  
**完整验证过 RCE**  
；  
  
- **PHP-FPM**  
：源码分析判定"易感"，但官方明确说  
**没有完整的 FPM 利用被演示**  
（FPM 在请求启动失败时直接杀掉子进程，恰好躲掉了一条路径）；  
  
- **FrankenPHP**  
：机制与 mod_php 类似，官方判定"很可能同样易感"；  
  
- **LiteSpeed（LSAPI）**  
：行为接近 FPM，应用相关的利用"可能但未验证"。  
  
再往外一层，公开的第三方分析（研究者 Jackalsyn 那篇流传较广）还提出：109618 可能存在结构性限制，109619 要稳定复现堆破坏，需要 Apache prefork 加上 glibc 2.39 这个特定版本。这类分析值得看，但要记住它是  
**单点复现结论**  
，不是官方口径。  
  
**这些都不是"可以等"的理由**  
，官方和各国 CERT 的口径仍然是立即升级——因为细节已经公开，研究者的起跑线被提前了。但这些分层解释了一个你一定会在接下来两周看到的现象：  
**有人拖了几天没打补丁，什么也没发生，于是觉得没那么严重。**  
  
那不是判断，那是运气。你没法知道自己的部署刚好落在哪一层，而攻击者只要猜中一次。  
## （四）判断四：你很可能打不上这个补丁  
  
通告说"立即升级"，听起来一分钟的事。按我这些年的经验，至少有五种情况会让升级卡住：  
  
**一、版本停在 8.1 以下。**  
 官方不再为这些分支发版，但这次破例给 7.4 / 8.0 / 8.1 出了  
**最小化源码回溯补丁**  
——意思是这三条线只能自己从源码编译，或者等发行版与 LTS 厂商的包。老 MIS、老 CMS 还在 7.x 上跑的，升级要动代码兼容性，不是换个包的事。  
  
**二、包源当天没更新。**  
 这次有个很具体的坑：依赖   
rpms.remirepo.net  
 这个第三方仓库的用户，发布当天那个仓库没有更新包，官方公告里专门点名让这批人改用发行版官方包或自己编译。  
**"我 yum update 过了"和"我升到了修复版本"是两件事。**  
  
**三、以为升级了，其实没生效。**  
 发行版、宝塔/cPanel 这类面板、部分商业产品的修复包  
**版本号可能不变**  
，只变构建号；更常见的是装了新包但   
**worker 进程没重启**  
，内存里跑的还是旧代码。官方在两条缓解措施里都专门写了"重启所有 worker"，原因就在这。  
  
**四、一台机器多套 PHP。**  
 站点、计划任务、命令行 CLI、容器镜像，各有一份。修了网站那份，漏了容器里的和 crontab 用的。容器要  
**重建镜像**  
，  
docker restart  
 不解决。  
  
**五、供应商产品里捆绑的 PHP。**  
 一体机、运维平台、第三方 appliance、监控系统，内部带着   
php.exe  
/  
php-cgi  
，厂商不发补丁你动不了它。CERT-EU 和多个国家 CERT 都在提醒这件事——影响面不止网站。  
  
所以升级这个动作的真实难度，不在"敲升级命令"，在  
**先有一份版本台账**  
。如果你现在答不出"我环境里到底有几份 PHP、各是什么版本、跑在哪些机器上"——那你的问题比这个漏洞大。  
## 2.今天该做的五件事  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n104" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">顺序</span></span></span></th><th data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n105" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">动作</span></span></span></th><th data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n106" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">要点</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n108" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n109" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">升级到修复版本</span></span></span></td><td data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n110" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8.5.12 / 8.4.27 / 8.3.36 / 8.2.35；公网暴露的系统</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">今天</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">做，内网排本周；注意 8.2 将于 2026 年 12 月 31 日终止支持</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n112" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">2</span></span></span></td><td data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n113" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">重启 PHP 进程</span></span></strong></span></span></td><td data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n114" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">升级不重启等于没升级；确认 worker 真正加载了新版本</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n116" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">3</span></span></span></td><td data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n117" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">盘点所有 PHP 实例</span></span></span></td><td data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n118" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">网站、CLI、容器镜像、面板、供应商产品、监控与一体机，建版本台账</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n120" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">4</span></span></span></td><td data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n121" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">实在升不动的，按洞分别上临时配置</span></span></span></td><td data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n122" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">两个洞的开关相反</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">：109619 → 关 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">session.upload_progress</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（8.5 再关 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">fatal_error_backtraces</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">）；109618 → </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">memory_limit=-1</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">放开</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，同时收紧 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">post_max_size</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 与请求形状限制。官方明说这只是减压，不是修复</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="59" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n124" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 22.1686px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">5</span></span></span></td><td data-colwidth="163" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n125" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 118.513px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">回查 10 月 20 日以来的日志</span></span></span></td><td data-colwidth="332" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n126" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 742.936px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">这是第五件事，也是下面要展开的事</span></span></span></td></tr></tbody></table>  
## （一）第五件事展开说：这三周，日志会替谁说话  
  
从今天算起，细节公开、攻击已经出现，而补丁覆盖到全行业至少要几周。  
**这三周是一个天然的"窗口期"**  
——所有在这期间出的安全事件，都会被问同一个问题：你们是哪天补的？  
  
我在公安办了十年案，这种"补丁窗口期"里的事故，最后定性就取决于两样东西：  
**你什么时候打的补丁，和补丁之前的那段时间里，日志能不能说清楚发生了什么。**  
  
补得早，出过事也叫"已按规范处置"；补得晚还没日志，性质就完全不一样了。所以第五件事别省：  
- 把 10 月 20 日（细节泄露那天）以来的 Web 日志、PHP-FPM 日志过一遍，重点看   
**PHP 进程的异常子进程、非预期文件写入、可疑外联**  
；  
  
- 升级重启这件事本身，记下准确时间，  
**留档**  
——它是你将来证明"已尽责"的证据；  
  
- 公网还没来得及补的系统，日志先单独归档一份到攻击者够不到的地方，别等出事时发现日志可以被人删。  
  
顺带一句：官方在缓解建议里也提到，可以用   
pm.max_requests=1  
 这种"每个 worker 只处理一个请求"的极端方式压缩状态残留的空间——它不是给生产用的，但它说明了一件事：  
**这次的问题本质是"状态在请求之间活得太久"。**  
而日志，恰恰是唯一不能被这种状态污染的东西。  
  
这个判断我在另一篇讲日志的文章里写过，这里再说一遍：  
**一个正常运转的系统不会突然停止记录。**  
这三周，请替你的服务器把"记录"这件事盯紧。  
  
部队 8 年，公安网安 10 年，阿里安全 P8。20 余年一线攻防，11 本安全书作者。  
  
这篇是应急判断，不是通告合集——通告网上有的是，判断只有这里有。系列会继续从攻、守、判三个角度写下去。  
  
回复「PHP」领本次应急的自查清单（版本盘点 + 两个洞分别对应的配置核验 + 日志回查，可打印发给运维）。  
  
## 参考来源  
1. **PHP 官方安全公告**  
：《Security advisory: Critical remote code execution vulnerabilities, update now》，2026-10-21 ——   
https://www.php.net/archive/2026.php  
  
1. **GHSA-w8hw-w9fx-j79w**  
（CVE-2026-109618，RCE via OOM corruption）——   
https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
1. **GHSA-6wmf-4rv5-p3rp**  
（CVE-2026-109619，RCE via stale global state）——   
https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
1. **CERT-EU 安全公告 2026-018**  
 及配套操作信息（NCSC-NL 维护，TLP:CLEAR）——   
https://github.com/NCSC-NL/php2shell  
  
1. **Tenable**  
：《php2shell (CVE-2026-109618, CVE-2026-109619): Frequently asked questions》，2026-10-20 ——   
https://www.tenable.com/blog/php2shell-cve-2026-109618-cve-2026-109619-frequently-asked-questions-about-critical-php-remote  
  
1. **BleepingComputer**  
：《PHP warns admins of two critical flaws, emergency patches coming tomorrow》，2026-10-20 ——   
https://www.bleepingcomputer.com/news/security/php-warns-admins-of-two-critical-flaws-emergency-patches-coming-tomorrow/  
  
1. **Huntress**  
：《Rapid Response: php2shell, Leaked Critical PHP Vulnerabilities》 ——   
https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities  
  
1. **ELLIO**  
：《PHP2Shell: what we observed and what to do》（补丁前攻击时间点与完整时间线）——   
https://ellio.tech/en/blog/php2shell-what-we-observed-and-what-to-do  
  
1. **Aviatrix**  
：《php2shell: Two Critical PHP RCEs, a Pre-Patch Leak...》（汇总 Jackalsyn 等第三方技术分析）——   
https://aviatrix.ai/threat-research-center/research-insights/php2shell-two-critical-php-rces-a-pre-patch-leak-and-the-php-you-didnt-know-you-had/  
  
