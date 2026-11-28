#  四个请求，从游客到 Shell：Cacti 未授权 RCE 深度分析（QVD-2026-117631·附PoC）  
原创 赤霄攻防实验室
                    赤霄攻防实验室  赤霄攻防实验室   2026-11-28 00:51  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8acicV9z0hbXn2Odg6uoUOHIweyFnNGj7upDpKmZEQZKqMa97W1C1qib6LferrBhiaUicphch0SQNJQpbTAyycia6Dx0F8Bw98x1RgcA/640?wx_fmt=png "")  
  
▲ 本机靶场：未登录状态下直接打开 Cacti 的「用户资料」页，guest 会话已就位  
  
最新版 Cacti（1.2.31）存在一条**不需要账号、不需要密码、四个 HTTP 请求就走到命令执行**  
的链路，11 月 26 日由研究者 ek0ms savi0r 公开，命名为 4HTTP2SHELL；奇安信 CERT 已收录为 **QVD-2026-117631**  
并标记已验证。截至本文写作时，官方没有发布修复版本，修复 PR 还停在「已开启、未合并」的状态。本号在 Cacti 1.2.31 原生部署的靶场上，用非破坏方式把这条链完整走了一遍，并把验证脚本整理成了三态判定的 PoC，文末可取。  
  
漏洞概要  
  
 编号：QVD-2026-117631（暂无 CVE）  
  
 类型：二阶 SQL 注入 → 轮询器命令执行（未授权 RCE）  
  
 影响：Cacti Core 1.2.31（当前最新版），**guest 账户处于启用状态**  
时可达（guest 默认关闭，需管理员手动开启）  
  
 前置：无需凭据、无需用户交互；目标需有至少一个 guest 可见且带脚本型数据源的图表（默认安装的「本机 Linux」设备即满足）  
  
 状态：研究者 2026-11-26 公开细节与 PoC；官方修复 PR #9002 于 11-27 开启、未合并，无 release  
  
 缓解：立即禁用 guest 账户；管理面不要裸露公网   
## TL;DR  
  
🔹**是什么：**  
Cacti 的「用户设置」接口允许 guest 把一个「本应是数字」的设置写成任意字符串，这个字符串随后被当成 SQL 的 LIMIT 片段拼进查询。  
  
⚠️**影响谁：**  
所有开着 guest 账户的 Cacti 实例。guest 默认关闭，但监控大屏、NOC 墙、运维共享看板往往为了省事把它打开。  
  
🛠**怎么修：**  
官方尚无版本；立即禁用 guest（Console → Configuration → Users），并把 Cacti 管理面收口到内网或 VPN 之后。  
  
💡**看点：**  
请求侧的整数校验其实存在，但攻击者只要不传那个参数，程序就会从数据库读「默认值」——而默认值通道绕过了校验。这是二阶注入的教科书案例。  
## guest 不是「匿名游客」，是「有账号的人」  
  
要理解这条链为什么成立，得先理解 Cacti 的 guest 机制。很多开源系统的「游客」只是一个标记，但 Cacti 的 guest 是一个**真实存在于用户表里的账户**  
：有 user_id、有自己的设置行、能登录、能改自己的偏好。它默认是关闭的，要管理员在认证设置里手动指定才会生效；而一旦启用，guest 默认能看系统里所有图表。管理员启用它的本意，是让没有账号的同事也能看监控图——于是在「用户资料」页，guest 和普通用户一样，可以保存自己的显示偏好。  
  
问题就出在「保存偏好」这个动作上。本篇在靶场上做的第一件事，就是用全新 Cookie 直接打开用户资料页：页面正常返回，CSRF token 也在——这意味着后面的每一个写操作，攻击者都能合法带上 token，不会被 CSRF 防御拦住。整条链路里，**没有任何一步需要绕过认证**  
，因为 guest 本来就是「已登录」。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8acREUVrJDL6aRhZhTI8BLfhVLS6zQcNGaNtykH7ic6tPYQagciaBXiaycZ61iaboxaIhSuEe5PLCfs5Vp9hibQwNfa3BPnLQ3wd4nvI/640?wx_fmt=png "")  
  
▲ 4HTTP2SHELL 的四个请求：前三步把 payload 从「用户设置」搬进轮询任务表，第四步让轮询器把它当命令执行  
## 四个失效点，缺一个都打不通  
  
把 1.2.31 的源码逐层拆开，这条链实际上是四个各自「看起来没问题」的决定叠出来的。单独看每一个，都像是可以接受的工程妥协；连起来看，就是一条从游客到 shell 的直通线。  
### 第一处：写入时不验类型  
  
用户资料的保存逻辑走的是一个通用的「更新用户设置」函数，它的职责只有一件事：确认你改的设置名是存在的，然后原样写进数据库。至于「这个值应该是整数」——它默认上游已经校验过了。可上游是 HTTP 请求，而 guest 恰好有权调用这个接口。于是 preview_graphs_per_page 这个字段，guest 想写什么就写什么。有趣的是，这一层用了预处理语句，写入是安全的——**存进去的是字符串，不是 SQL**  
，这也是为什么很多基于「写入点」的扫描器会放过它。  
### 第二处：校验存在，但默认值不走校验  
  
图表预览页对每页显示数量是有校验的：请求参数 graphs 必须通过整数过滤器，不过滤掉就丢弃。看起来没问题？但这个过滤函数只在**请求里带了 graphs 参数**  
时才生效。攻击者只要不传这个参数，程序就会走另一条路——读取用户设置表里的「默认值」，而默认值是直接从数据库取出来就用的，**不经过任何过滤器**  
。这正是二阶注入的经典形态：脏数据从写入口进库，在读出口被当成可信值拼接。  
### 第三处：LIMIT 子句的字符串拼接 + 多语句  
  
读出来的默认值最终被拼进「每页图表」查询的 LIMIT 片段。这里没有再做一次整数强转，PDO 连接又允许多语句执行，于是攻击者在默认值里塞的分号后面，可以跟任意第二条 SQL。研究者选择的目标是轮询任务表 poller_item：这是 Cacti 定时采集数据用的表，里面有一列 arg1，对脚本型数据源而言，它存的就是**要在服务器上执行的命令**  
。  
### 第四处：轮询器无条件信任库里的命令  
  
轮询器读到脚本型任务后，把 arg1 原样交给进程执行函数，没有任何白名单或路径限制。这个设计本身有历史原因——Cacti 的脚本路径由管理员定义，默认情况下是可信的。但当一张本应只有管理员能写的表，被 guest 通过 SQL 注入改写时，「库里的命令是可信的」这个前提就塌了。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8adle8pp8GGSwIP2mGThlKm3VDOJh0lcCSnRsudaPm9j2iac6jPqyWK5ica6xyydwQA5icxI2MaFldktdicHYj9dEPcnNTtnQQ3HVZ4/640?wx_fmt=png "")  
  
▲ 1.2.31 源码：写入端只查设置名；预览页的默认值直接来自用户设置表，未经整数过滤  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8adL8MPU2ASdDichicmKpTYicMIr0eJrayPJC0gP27WzeBF56jFXBHbFjxc7ruXYq3fJuqwqic47ia7f0eTof9cDiaibIhbJibMgcg4X0Y0/640?wx_fmt=png "")  
  
▲ LIMIT 片段字符串拼接，与轮询器把任务字段交给进程执行的那一行  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8adGoUXuhrcX5FoYsIf7XiahcmkVMLWW1IPJM4S6ytjavX3AsQAV3ec9gp6X4MRVsmKiaHRsCRLRicxsiaryicQhv9a6hpTff6IY7NOE/640?wx_fmt=png "")  
  
▲ 左：四个失效点在源码中的实际行为；右：官方修复与部署侧对策  
## 靶场复现：四个请求，一个哨兵文件  
  
本篇在 Cacti 1.2.31 上做了原生部署（PHP 8.1 + MySQL 8），用默认安装自带的「本机 Linux」设备作为数据源，启用 guest 账户，其余一切保持默认。复现脚本按研究者的四步链实现，但把 payload 换成了**非破坏版本**  
：命令只在 Cacti 的 resource 目录下写一个随机命名的一行哨兵文件，然后通过 HTTP 把它读回来——读得到，就证明命令确实以 Web 用户身份执行了。  
  
结果是干净的：**VULNERABLE**  
，哨兵可读回；数据库里轮询任务表的 arg1 已被改写成 payload；用户设置表里，那段以分号开头的字符串原样躺在 preview_graphs_per_page 字段下。整个过程 PHP 日志里只多了一条「遇到非数值」的 warning——这是 LIMIT 片段被拿去做分页计算时触发的，也是这条链在日志里留下的**唯一痕迹**  
。验证完成后，脚本通过同一通道删除哨兵并把任务表恢复原值。  
  
一个值得记下的细节：payload 里不能用子查询去引用正在更新的那张表，MySQL 会直接拒绝——研究者公开的版本用的是常量，本篇起初改成了子查询，首跑就撞上了这个错误。这也是读别人的 PoC 时容易忽略的地方：能跑通的版本里，每一个看似随意的选择都可能是踩过坑的。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8af6alwk0nQRgUVwmyoGjibtlawKWicw3F9NibW4D45VGku6DGqz5JQHP9sc0G5FXzN0ZgxuNicQ5ZjC2nqo9mpOKMWIbIrj3YMU8v0/640?wx_fmt=png "")  
  
▲ 本机靶场上的 PoC 运行输出：指纹探测 → 前置条件判定 → 链路执行 → 哨兵读回 → 自动清理  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8adtZRayAchjib90eBQ4z7xN0Htzurx4rBtb0pfXvS97Oc6729vTw5I6YOc33kJXicleVfOV1jK96pCraKJfiac73FYcpPrl8jfZsI/640?wx_fmt=png "")  
  
▲ 研究者公开的 PoC 说明（Sad-Girls-0day-Club 仓库）：四个请求与每一步的落点  
## 修复 PR 在做什么，以及一个被遗漏的信号  
  
官方修复 PR #9002 的思路是对的：不再信任传入的 LIMIT 片段，而是新增一个专门的构造函数——页码必须通过整数过滤且不小于 1，行数至少为 1，然后由代码自己算出「偏移量,行数」这两个整数。换句话说，**从「清洗输入」改成「生成输出」**  
，攻击者控制的字符串根本没有机会进入 SQL。PR 还顺手把 graphs 参数改成了强类型过滤，并附带了单元测试和 semgrep 规则。  
  
本篇在靶场上把这个修复思路模拟了一遍（在 LIMIT 拼接处加整数强转），再用同一个 PoC 重跑：链路照常执行，但 arg1 不再被改写、哨兵没有落地——判定从 VULNERABLE 变为 UNKNOWN。修复方向有效，剩下的只是合并与发版的时间。  
  
还有一个信号值得写出来。翻 1.2.31 的测试目录时，会发现一个名为「私有 advisory 证据回归」的测试文件，里面记录了八个 GHSA 编号对应的修复断言——导入签名校验、登录重定向、ORDER BY 白名单、脚本路径转义等等。这说明 Cacti 团队在 1.2.31 之前已经通过私有披露流程修了一批洞，并且为每个修复留了回归测试。但这十条断言里，**没有一条涉及用户设置或 LIMIT 片段**  
。那条链不在私有披露的清单上，于是走到了公开 0day 的路上。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8aeKHtCO7RiaBCx70Kw0gamR23vkIrXU8632ich3WUtJKibdh4oF2R63xibgyHDKicB3dBlA4ZVJZywWHCVMnwkGgdYRCQWbDvCO5Cx8/640?wx_fmt=png "")  
  
▲ 修复 PR #9002 的核心：由函数生成 LIMIT 片段，页码与行数都必须是正整数  
## 现在该做什么  
  
官方版本没有出来之前，最有效的一步是**禁用 guest 账户**  
——Console → Configuration → Users，把 guest 设为禁用。这一步直接掐断了链路的第一环，因为没有 guest 会话就没有 CSRF token，后面的写操作全部失效。如果你的看板确实需要匿名访问，退而求其次的做法是把 Cacti 放到内网或 VPN 之后，并在反向代理上限制用户资料页的访问。  
  
排查方面，可以看三处：用户设置表里 preview_graphs_per_page 字段有没有非数字值；轮询任务表里脚本型任务的命令字段有没有近期被改过的痕迹；PHP 日志里「遇到非数值」的 warning 是否集中在图表预览页。这三处任何一处命中，都值得当作入侵痕迹认真对待。  
  
归根结底，这条链里没有所谓的「奇技淫巧」——每一环都是常规操作，连起来却是一个未授权 RCE。它更像一道给所有开发者的提醒：当你决定信任数据库里的某个值时，最好先确认这个值是谁写进去的。  
## 参考  
  
▸**研究者 PoC 与说明（Sad-Girls-0day-Club）**  
  
https://github.com/ekomsSavior/Sad-Girls-0day-Club/blob/main/4http2shell/README.md  
  
▸**The Hacker News 报道**  
  
https://thehackernews.com/2026/11/unpatched-cacti-zero-day-enables-rce-in.html  
  
▸**SecurityOnline 技术综述**  
  
https://securityonline.info/cacti-zero-day-rce-4http2root-poc/  
  
▸**官方修复 PR #9002**  
  
https://github.com/Cacti/cacti/pull/9002  
  
▸**Cacti 1.2.31 源码（lib/html_graph.php、lib/auth.php、cmd_realtime.php）**  
  
https://github.com/Cacti/cacti/tree/1.2.31  
  
▸**本篇 PoC（poc.py，三态判定 + 哨兵验证 + 自动清理）**  
  
https://github.com/Rubby2001/CVE-POCS/tree/main/QVD-2026-117631  
  
我是赤霄攻防实验室，每晚挑一个最值得讲的漏洞，从源码拆到实测。这篇的 PoC 和复现记录都在 GitHub 上，觉得有用点个「在看」，让还在开着 guest 的 Cacti 今晚就把这个账户关掉。  
  
免责声明：本文仅用于技术研究与授权安全测试。文中提到的漏洞利用方式请勿用于未授权目标，由此产生的一切后果与本号无关。文中 PoC 仅在目标 web 根目录写入并清理一个哨兵文件、默认指向本机，使用者需自行承担合规责任。  
  
— 完 —  
  
