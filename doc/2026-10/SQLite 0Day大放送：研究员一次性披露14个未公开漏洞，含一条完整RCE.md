#  SQLite 0Day大放送：研究员一次性披露14个未公开漏洞，含一条完整RCE  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-11 01:06  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NQjNU9KuVdShib8ddhPcfAJU10XXMctSKl8icCxDCKBQP7iccKJbae5Z6iaEsR1HtzUmsa8SKUxia8wv7nibVib5gkjAibKib8e0jQ8cRs/640?from=appmsg "")  
> **导语**  
：安全研究员 abraxas_null（Loopback Lab）近日一次性公开了 14 个未披露的 SQLite 0Day 漏洞，覆盖 RCE、本地利用、拒绝服务三类，影响 SQLite 3.53.4 GA 版本。SQLite 作为全球部署最广泛的数据库引擎之一，这一波集中披露给依赖它的产品带来了密集的安全补丁压力。  
  
  
![SQLite 0Day封面图](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PcLia1UcdSkRNHNgVianhmWbr6IlFQ7qLVPVj7qic33ia73R1FAYfhZsy9FGWnsqlJjpibxfTvc6R1R8cGicsCd1jUk6ArHyRe1AGq4/640?from=appmsg "SQLite 0Day封面图")  
## 一、事件背景  
  
SQLite 一直被官方称为「全球使用最广泛的数据库引擎」，从浏览器、移动 App、操作系统到嵌入式设备，几乎所有能跑代码的环境里都能看到它的身影。也正因为这种「无处不在」的部署特性，SQLite 的每一个安全漏洞都可能波及成千上万的上下游产品。  
  
这次集中披露来自安全研究员 abraxas_null，他在推文中列出了 14 个未公开漏洞，并附上 14 个对应的 GitHub 仓库，每个仓库都包含漏洞说明、复现步骤和 PoC 代码。从仓库命名可以看出，受影响版本集中在 2026 年 7 月 24 日发布的 SQLite 3.53.4 GA，源码指纹 Hwaci 指代的就是同一份编译源。  
## 二、14 个 0Day 整体分类  
  
abraxas 把这 14 个漏洞按影响类型分成了三类：  
  
![14个SQLite 0Day分类图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MPhufI5QPt2IujfzjJSvjpibneeaFoxfH3Nj6AwQsRD2tRicMI9TtY12p5VuxZibPjPYf3icujPy1w33f0Jom7nzibK1n9EaubY6sY/640?from=appmsg "14个SQLite 0Day分类图")  
- **1 条 RCE**  
：sqlite-rsync-ssh（远程代码执行）  
  
- **8 条 本地利用**  
：包含 CLI 命令注入、UAF、堆越界写入、权限绕过、信息泄露  
  
- **5 条 DoS**  
：解析器嵌套深度、UNION 链清理、PutVarint 溢出等触发的崩溃  
  
### 唯一一条 RCE：sqlite-rsync-ssh  
  
这条 RCE 出现在 SQLite 自带的 sqlite3_rsync  
 工具中。sqlite3_rsync  
 的功能是把本地 SQLite 数据库与远程主机同步，远程目标通过 SSH 完成。当用户在命令行传入精心构造的 filename 参数时，参数里的 shell 元字符会被传到 SSH 命令行，被远端的 shell 当作命令执行。  
  
由于这是命令行注入而不是 SQL 注入，攻击者只要能影响目标 filename 的来源（比如让脚本自动处理外部输入），就能以调用 sqlite3_rsync  
 的用户身份在远端机器上执行任意命令。对管理员备份、CI 同步脚本等场景，这是一条可以直接利用的远程路径。abraxas 在仓库 sqlite-rsync-ssh 中给出了完整复现脚本，仅绑定 loopback 进行授权验证。  
### 八条本地利用  
  
8 条本地利用可以再分四类：  
- **CLI 命令注入 2 条**  
：sqlite-cli-go（go 与「/」终止符后跟 dot-command）和 sqlite-cli-recover（recover 子命令把 hostile schema 导出为 dot-command）。两者都能让用户在打开构造好的数据库后，被 .  
 开头的 dot-command 当作 shell 命令执行。  
  
- **权限绕过 2 条**  
：sqlite-fj-using-auth（FULL JOIN USING 跳过 sqlite3AuthRead  
）和 sqlite-xfer-auth（INSERT SELECT 时源表 READ 权限被跳过）。这两条对基于 SQLite 的多用户应用是直接威胁。  
  
- **UAF / 堆越界 3 条**  
：sqlite-fts3-snippet、sqlite-rtree-rowid0（UAF），sqlite-session-ncol（堆越界写入）。这些通常先表现为崩溃，但配合 ASan 或用户态堆分配器可被改造成更危险的路径。  
  
- **信息泄露 1 条**  
：sqlite-wal-pagesize（WAL/DB page-size 不一致导致未初始化页尾被读出），对多租户或共享同一进程的应用意味着跨租户内存泄露。  
  
### 五条 DoS  
  
DoS 类集中在解析器边界检查缺失：  
- sqlite-collate-depth：COLLATE 链跳过 SQLITE_LIMIT_EXPR_DEPTH  
。  
  
- sqlite-cte-nest：嵌套 CTE 跳过 nNestSel  
 视图深度限制。  
  
- sqlite-fts5-rowidmerge：FTS5 detail=none 前缀合并欠分配。  
  
- sqlite-fts5-securedel：FTS5 secure-delete 写 PutVarint 时 footer 越界。  
  
- sqlite-values-union：VALUES UNION ALL fallback 清理递归溢出 C 栈。  
  
任何一条在自动化批量任务中触发都意味着服务中断。  
### PoC 仓库一览  
  
abraxas 在 GitHub 上为每个漏洞都建立了独立仓库，统一命名 sqlite-<short-name>  
，每个仓库都包含受影响版本的源码指纹、复现步骤、触发 witness 和 PoC 脚本。除了上面点名的 sqlite-rsync-ssh 之外，其余 13 个仓库地址如下：  
- CLI 命令注入：sqlite-cli-go、sqlite-cli-recover  
  
- 权限绕过：sqlite-fj-using-auth、sqlite-xfer-auth  
  
- UAF / 堆越界：sqlite-fts3-snippet、sqlite-rtree-rowid0、sqlite-session-ncol  
  
- 信息泄露：sqlite-wal-pagesize  
  
- DoS / 崩溃：sqlite-collate-depth、sqlite-cte-nest、sqlite-fts5-rowidmerge、sqlite-fts5-securedel、sqlite-values-union  
  
需要特别说明的是，abraxas 在每个仓库 README 里都明确写有「**Target only loopback. Do not point this script at the internet**  
」的授权边界——PoC 仅供本地复现，不应用于未授权系统。  
## 三、修复状态  
  
从仓库描述可以看到，修复 commit 都已经进入 SQLite 主干（trunk），最晚的几个修复指向 2026-10-04 之后的提交。但 3.53 分支截至目前还没有发布新的 GA 版本，下游厂商拿到的「稳定版」仍然是带漏洞的 3.53.4。  
  
这是一个比较微妙的发布节奏问题：trunk 修了但分支没发，对不能跟随 trunk 编译的发行版（Debian stable、RHEL、各大手机厂商 BSP 等）来说，它们手里的 3.53.4 仍然全员躺枪。  
## 四、影响范围  
  
SQLite 是全球部署量最大的数据库引擎之一，这一批 14 个 0Day 几乎覆盖所有把 SQLite 3.53.4 编进二进制的产品：  
- **桌面浏览器**  
：Chromium 内核浏览器都内置了 SQLite。  
  
- **操作系统**  
：macOS、iOS、各 Linux 发行版的包管理器都用 SQLite。  
  
- **嵌入式 / IoT**  
：几乎所有带本地存储的设备。  
  
- **第三方软件**  
：Office 类工具、设计软件、各种本地数据库工具等。  
  
需要注意的是，**CLI 命令注入、RCE 类问题只在用户主动打开构造数据库或调用 sqlite3_rsync 时触发**  
，但权限绕过类问题对嵌入式数据库的多租户场景是直接可被远程触发的。  
## 五、防御建议  
  
对下游厂商和使用方：  
- 等待 SQLite 3.53.x 下一个 GA 版本发布，并尽快升级。  
  
- 如果能跟随 trunk 编译，使用 2026-10-04 之后的 commit。  
  
- 在升级前的窗口期，**限制 sqlite3_rsync 处理不可信 filename**  
，不要把它放在自动跑通的脚本里。  
  
- 对 CLI 用户，**不要用 sqlite3 打开来源不明的数据库文件**  
，尤其是 dot 命令会被执行的环境。  
  
- 应用层不要把 SQLite 当多用户权限隔离的最终防线，权限绕过类问题提示了「用数据库做权限检查」这种模式的风险。  
  
## 六、总结  
  
14 个 0Day 一次性集中披露，单独看每一个并不算「炸裂」，但合在一起构成了对 SQLite 3.53.4 这一个 GA 版本的密集漏洞矩阵。这给依赖 SQLite 的产品厂商提了个醒：当一个无版本号快速演进的引擎成为基础设施时，安全补丁节奏本身就是产品安全的一部分。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OUurSm4blvLL9J8JTRlnRmrmZSu1z6c08qdE2o9t0ghRdyrsMHaL5bWciboQJWGeiaw1OLzL6DPeAx9F7cPxoZ3eJK68uNxibmkQ/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
