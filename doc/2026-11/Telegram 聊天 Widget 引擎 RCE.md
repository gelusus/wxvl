#  Telegram 聊天 Widget 引擎 RCE  
 Ots安全   2026-11-13 05:51  
  
     
  
2026 年 11 月 11 日，一个名叫 davtur19/tg-widget-rce 的仓库在 GitHub 公开。仓库标题写着 Telegram Widget RCE，副标题是 Telegram 聊天 Widget 引擎里的原生代码执行。点进去，目录里摆着 bot、builders、payloads、scripts、vulns 这些名字，README 里还挂着 Sony 手机、iPhone 6s、iPhone 17e 的真实设备 RCE 证明。  
  
这是一份已经打完的补丁说明书，不是正在飞的子弹。作者在前言就写明，仓库里所有发现如今都是 n-day：它们曾经活在旧版 Telegram 构建上，当前版本已经修复，本仓库是技术研究记录。那些设备证明指向的是过去时间点的旧构建。  
  
正因为这份记录把攻击面、引擎内部和交付方式都摊开了，它反而比一份干瘪的 CVE 更有防守价值。本文只做一件事：从防御视角拆开这个聊天 Widget 引擎到底哪里失守，以及如何确认自己是否已经站在修复一侧。本文不转载任何武器化利用代码、投递机器人、载荷或利用链，只引用仓库 README 中公开的概念性触发片段来说明原理。  
  
### 官方与仓库怎么描述这个洞  
  
仓库 README 的自述是：一条 sendWidget 消息携带 HTML 与 JavaScript，当聊天处于打开状态时，应用用 litehtml 与 QuickJS-ng 渲染它，Widget 在消息布局阶段就执行，不需要用户点击。也就是说，攻击者只要能让一条 widget 消息进入受害者的聊天界面并显示出来，代码就会自动跑。  
  
这不是单个 CVE 编号能概括的事件。作者把问题拆成了一组编号：BUG-1 到 BUG-6 是他在 QuickJS-ng 与引擎里挖到的 0day，NDAY-1 与 NDAY-2 是上游 QuickJS-ng 已修但被旧版 Telegram 漏带过来的已知缺陷，QJS-DOS 与 LH-DOS 是引擎层的拒绝服务类问题。仓库自身没有向 NVD 申请统一编号，它是一份研究者视角的归档。  
  
README 的漏洞表里，最重的一处是 BUG-1 解析器 0day，标注为作用域标记 ARG_SCOPE_END 错位，CVSS 给到 9.6 与 8.8 两个口径。其余几处分布在 5.3 到 8.8 之间。把影响版本按族列出，能够暴露的主机集中在旧构建：  
<table><thead><tr style="box-sizing: border-box;"><th data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">编号</span></section></th><th data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">类型</span></section></th><th data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">CVSS 口径</span></section></th><th style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">受影响旧构建</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-1 解析器 0day</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">RCE</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">9.6 / 8.8</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 72889、72929、73029、73039；iOS 13.2；tdesktop 7.4.0；macOS 13.1</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-2 直接 eval 闭包变量截断</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">RCE</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">8.1</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 73029、73039</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-3 <a class="wx_topic_link" topic-id="mwej1e5g-80dqve" style="color: #576B95 !important;" data-topic="1" data-recommend="">#1865</a> 捕获变量截断</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">内存安全</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">5.8</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 73029、73039</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-4 Map.clear UAF</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">内存安全</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">6.5</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 73029、73039</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-5 FR.unregister UAF</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">内存安全</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">6.5</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 73029、73039</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">BUG-6 reset_weak_ref 中止</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">稳定性</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">6.5</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 73029、73039</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">NDAY-1 <a class="wx_topic_link" topic-id="mwej1e5g-9ypzwb" style="color: #576B95 !important;" data-topic="1" data-recommend="">#1827</a> 陈旧正则字节码</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">RCE</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">8.8</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 13.2.0；iOS 13.2；tdesktop 7.4.0</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="154" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">NDAY-2 <a class="wx_topic_link" topic-id="mwej1e5g-rjzk44" style="color: #576B95 !important;" data-topic="1" data-recommend="">#1865</a> var_ref_count 截断</span></section></td><td data-colwidth="109" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">RCE</span></section></td><td data-colwidth="108" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">8.8</span></section></td><td style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Android 13.2.0</span></section></td></tr></tbody></table>## 攻击面：聊天 Widget 引擎与 sendWidget 投递  
  
要理解这批漏洞，先要搞清楚 Telegram 的聊天 Widget 引擎是什么，以及它为什么危险。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0H46EnI2AvkDhz9ibX58giakUjn5RVuW68PqWp66KP9gl4ZErUyFgX1WW9TaK5fxicqsSzAHL26UGFaPH1A1Yr6kgLO7E0sx2qhz0/640?wx_fmt=png&from=appmsg "")  
  
Widget 引擎由两层组成。底层是 litehtml，一个轻量 HTML 渲染器，负责把 Widget 消息里的 HTML 画出来；上层是 QuickJS-ng，一个嵌入式 JavaScript 引擎，负责执行 Widget 消息里携带的脚本。两者叠加，让一条聊天消息不再只是文本或图片，而是一段能在客户端本地运行的迷你应用。  
  
危险来自触发条件。仓库指出，sendWidget 消息在聊天窗口打开、消息进入布局阶段时就会渲染并执行，不需要接收者去点击、长按或打开任何附件。这与传统钓鱼有本质区别：传统钓鱼要等受害者主动打开文件或链接，而 Widget 的执行发生在消息被显示的瞬间。  
  
投递通道也值得防守方留意。Telegram 的 Bot API 提供 sendWidget 接口，标准域名与测试域名都能发送这类消息。一个被允许发送 Widget 的机器人，就能把携带脚本的消息推给目标对话。仓库还提到 answerWidgetQuery 等配套接口，构成一套完整的 widget 交互回路。这意味着攻击面不只在客户端代码，也在账号与机器人权限的边界上。  
  
防守方应当把 Widget 引擎视为一个隐式的本地代码执行环境，哪怕它名义上跑在受限上下文里。下面这处 0day 恰好说明了，名义上的限制并不一定牢靠。  
## 核心 0day：解析器作用域标记错位  
  
BUG-1 是这份归档里标为 9.6 与 8.8 的那一处，也是被设备证明最多的发现。它的根因在 QuickJS-ng 的解析器与编译器，涉及 js_parse_for_in_of、OP_eval、add_closure_variables 这几个内部函数。  
  
机理可以拆成一条短链。一个带有严格模式、带一个空解构参数、且参数初始化器里写了直接 eval 的函数，会在作用域处理上留下一个本该被清掉的标志。严格模式会跳过 add_arguments_arg() 这一步，于是参数作用域结束标记 ARG_SCOPE_END（值为 -2）被原样保留下来。编译器随后把这个标记当操作数发射出去：它算出 first + 1 = -1，当作 16 位操作数写成 0xFFFF；运行时再减 1，就把 65534 当成了一个变量索引。  
  
仓库 README 给出了一段概念性触发片段，仅用于说明编译器如何产生这个越界索引，不含任何投递、堆喷或利用载荷：  
```
"use strict";
function f([] = eval("1")) {}
f();
```  
  
这段代码的破坏力在于越界读写的位置。按作者描述，越界读发生在 b->vardefs 之后约 1.31 MB 处，越界写则是向一个存活分配之外的 8 字节闭包记录写入。经过堆塑形，这个写入可以伪造出 ArrayBuffer 的 byteLength 字段，而这正是后续构筑原生代码执行原语的支点。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0EgggictzKpCQogWQxRDeYdVLVDG65EyU80YRY5kVto7gXar5yON5Lia2bnicxBLqjVgFaHj2BqOib4L6s3KiabAgQfeNxmxicKAtMCI/640?wx_fmt=png&from=appmsg "")  
  
值得注意的克制是，作者明确说这个 bug 不需要触发内存耗尽就能成立，并且它存在于 QuickJS-ng v0.17.0 与本次归档用到的 Telegram 13.x 构建中。解析器检查与解释器防护都已经进入当前构建。换句话说，这处 0day 的生命周期已经随着版本更新结束。  
  
把机理讲清楚的目的，是让防守方明白这类引擎漏洞的共性：它们往往不是某个显眼的函数写错了，而是编译器在特殊语法组合下吐出了错误的操作数，运行时又信任了这个操作数。检测与防护的抓手，因此要从引擎层的安全检查是否到位入手，而不是盯着单条消息的内容特征。  
## 漏洞家族总览：0day、n-day 与拒绝服务  
  
把仓库里的发现按性质归拢，能看出作者的挖掘是系统性的，而不是单点突破。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0HVUIsZPujZiaSWJP9fcicreEhNJMCy6ribR9bWvID2bM3kS1kUSoUI6TVRBnIBcpFAibbNB0VvgYtiaMmIfB2SuU7O4KBuX1yrviarE/640?wx_fmt=png&from=appmsg "")  
  
0day 一组共有六处。BUG-1 是解析器 0day，评分居首；BUG-2 是直接 eval 导致的闭包变量截断，会在越界指针读与 JSValue 读写上出问题；BUG-3 是 [#1865]()  
 捕获变量截断；BUG-4 与 BUG-5 分别是 Map.clear 与 FinalizationRegistry.unregister 的释放后使用；BUG-6 是 reset_weak_ref 触发的中止。后面五处都被作者标为在当前构建里已修或已缓解。  
  
n-day 一组有两处，性质更值得防守方重视。NDAY-1 对应上游 QuickJS-ng 的 [#1827]()  
，是一处陈旧正则字节码问题：JS_RegExpDelete 在强转 lastIndex 之前先快照了 re->bytecode，而 lastIndex 对象的 valueOf() 可以在删除路径上重编译并释放字节码，于是 lre_exec 跑在已释放的程序上。作者指出，这处上游已修的缺陷之所以还能被利用，是因为旧版 Telegram 分发的源码没有把修复带回。NDAY-2 对应 [#1865]()  
，uint16_t 的 var_ref_count 在大量重复参数加解析期直接 eval 下发生截断，导致越界索引到帧外约 524 KB。  
  
拒绝服务一类覆盖 QuickJS-ng 与 litehtml 两侧，包括 SVG 路径无限增长、渐变分配耗尽、嵌套表格与嵌套弹性布局造成的界面线程卡死、Ton 区块链对象字典爆炸等。它们单独不构成代码执行，但能稳定让应用崩溃或卡顿，属于低门槛的骚扰面。  
  
一个对防守方重要的判断是：这些 n-day 的可利用性，并不取决于漏洞本身是否修好，而取决于你手上的构建有没有把上游修复带回。这正是下一节要谈的修复状态分歧。  
## 影响边界与隔离缓解  
  
仓库的 VULNS.md 里有一处容易被误读的细节，值得单独拎出来讲清楚：不同构建把 Widget 引擎放在不同的进程里，直接影响漏洞的破坏半径。  
  
在较早的 Android 构建（如 72929、72889）上，Widget 引擎运行在应用主进程 UID 10346 内，解析器 0day 的 primitives 最终能转化为应用层级的代码执行。这也是设备证明里那些 Sony 手机结果对应的运行环境。  
  
从 beta 72979 起，Widget 引擎被搬进了隔离进程，运行在 UID 99017（SELinux 上下文 u:r:isolated_app:s0）；后续 72989 沿用同一隔离服务，UID 变为 99024。作者明确记录，在这种隔离模型下，解析器 primitive 仍能命中，但写入被阻断，应用数据写与本地回连被拦截。也就是说，同样一处引擎 bug，在新构建里被进程隔离削成了受限的代码执行或拒绝服务，而不是应用层失守。  
  
更深一层，仓库提到构建里装了 Seccomp 审计模式（installSeccomp(0)），三十条拒绝项，默认放行，被拦的系统调用返回错误而非真正阻断；真正的闸门是隔离 UID 加 SELinux 的 isolated_app 策略。这说明平台侧已经用多层机制在压缩 Widget 引擎的破坏半径，而不是只靠修单个 bug。  
  
防守方应当把这条信息当作一个校准器：评估自己环境风险时，不能只看漏洞存在与否，还要看引擎跑在哪个进程、有没有隔离与系统调用过滤。同样的 0day，在不同构建上的实际危害可以差出好几个量级。  
## 参考  
- https://github.com/davtur19/tg-widget-rce  
  
