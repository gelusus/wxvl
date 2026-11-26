#  Upload2Shell：从普通文件上传到服务器命令执行——Ghostscript 沙箱逃逸漏洞技术分析  
原创 极客零零七
                    极客零零七  极客零零七   2026-11-26 03:37  
  
**2026 年 11 月 17 日，安全研究机构 pwn.ai 公开了一项名为****Upload2Shell**  
 的漏洞研究。研究人员发现，攻击者可以通过普通的文件上传、邮件附件预览甚至网络打印功能，将恶意构造的文档送入Ghostscript 解释器（一个用于读取、执行并渲染**PostScript和PDF**  
页面描述语言文件的开源引擎，常用于文档格式转换、打印渲染以及 PDF 的合并、拆分与压缩），利用其 Type 1 字体加载器中的内存安全漏洞突破 -dSAFER  
 沙箱限制，最终在服务器上执行系统命令。  
  
该漏洞编号为 **CVE-2026-125396**  
，CVSS 4.0 评分为 8.5。Ghostscript 上游已于 2026 年 9 月 8 日发布 10.08.0 版本修复。研究人员已在多个常见应用的特定配置中验证攻击链，包括 WordPress、Roundcube 和 Debian CUPS 打印服务。目前公开资料尚未证实该漏洞已被用于真实攻击。  
### 一、漏洞概况  
  
传统的文件上传漏洞往往依赖 Web 服务器错误地执行了上传的文件，例如将恶意 PHP 脚本保存到可执行目录，再通过 HTTP 请求触发。  
  
Upload2Shell 的攻击逻辑完全不同。  
  
攻击者不需要让服务器把上传文件当成 PHP、Python 或 Shell 脚本执行，也不需要获得上传目录的执行权限。只要应用程序在处理文件时调用了存在漏洞的 Ghostscript，恶意文档就可能在转换过程中触发内存破坏。  
  
这意味着，即使应用满足以下条件，也不能仅凭这些条件判定上传处理安全：  
- 上传目录不可执行  
  
- 文件被重命名为随机名称  
  
- Web 服务器禁止直接访问原始上传文件  
  
- 应用只允许图片或 PDF  
  
- 文件转换时启用了 Ghostscript 的 -dSAFER  
 模式。  
  
这些控制措施各有价值，但它们不能替代对实际文件内容、下游解释器和运行环境的安全验证。  
### 二、攻击面如何形成？  
  
Ghostscript 是一个用于解释 PostScript 和 PDF 等页面描述格式的开源软件，广泛出现在 Linux 文档转换、打印、图片处理及预览服务中。  
  
开发人员可能从未在业务代码中直接调用过 Ghostscript，却依然通过依赖库使用了它。典型调用路径如下：  
  
![uploadshell-20261125211130.png](https://mmbiz.qpic.cn/sz_mmbiz_png/L7VicJKsiaibFDaSyeTUFRaN3WwBXBib5gA7uRZGsicIX5n3zdqKWHcz6sWD7ibbczZz4L6cMllgibBA91yBCDx9j4GtpSHIS1D9ym7Q4jy6ibQlJfU/640?from=appmsg "")  
#### 2.1 文件扩展名与真实格式之间的信任错位  
  
假设某 Web 应用只允许用户上传 .jpg  
 文件。  
  
攻击者提交一个名为 quarterly-results.jpg  
 的文件，并声明 MIME 类型为 image/jpeg  
，但文件内部实际包含 PostScript 指令。  
  
如果应用仅依据扩展名或客户端提供的MIME 类型判断文件合法性，而下游 ImageMagick 根据实际内容或格式探测选择了解释器，就可能出现以下情况：  
```
文件名：quarterly-results.jpg
声明类型：image/jpeg
实际内容：PostScript

应用层：认为这是图片
转换层：识别或委派给 PostScript 处理器
解释器：Ghostscript 执行文档处理
```  
  
这里需要区分两个概念。  
  
**格式混淆本身不等于远程代码执行。**  
 它解决的是攻击者如何将危险内容送入原本不应该暴露的解释器。  
  
真正导致 RCE 的，是 Ghostscript 内部另一个独立的安全缺陷。  
  
在研究人员公开的 WordPress 演示中，应用通过 Imagick 读取上传文件的第一页：  
```
$imagick->readImage($input . '[0]');
```  
  
虽然上传文件名以 .pdf  
 结尾，内部却是 PostScript 内容。ImageMagick 最终将其交给 Ghostscript 处理。  
  
这不单纯是文件扩展名绕过问题，而是由多个组件对文件类型和信任边界的理解不一致。  
#### 2.2 为什么 PostScript 比普通图片更危险？  
  
JPEG 和 PNG 主要是数据格式，而 PostScript 本质上是一门具有过程式执行能力的页面描述语言。  
  
PostScript 支持操作数栈、字典、过程、条件判断和循环等编程结构。例如：  
```
%!PS
/Helvetica findfont
24 scalefont
setfont
72 720 moveto
(Hello World) show
showpage
```  
  
这段代码描述如何选择字体、设置字号、移动绘图位置并输出文字。  
  
因此，Ghostscript 处理 PostScript 文档时，并非简单地读取像素，而是在解释并执行文档中的操作。  
  
这正是 -dSAFER  
 沙箱存在的原因：允许文档完成必要的渲染操作，同时限制其访问宿主文件和执行外部命令的能力。  
  
一旦解释器内部的安全控制被绕过，原本用于绘制页面的文档执行环境，就可能跨越到宿主操作系统。  
### 三、漏洞根因分析  
  
Upload2Shell 最值得分析的部分，是攻击者如何从 PostScript 语言级别的执行能力，逐步获得对 Ghostscript 原生内存的修改能力。  
  
根据 pwn.ai 公开的技术报告，这条漏洞利用链包含三个关键阶段：  
1. 利用字体加载过程中的异常处理，获得原本受保护的字典写入能力。  
  
1. 破坏 Ghostscript 的局部与全局对象生命周期约束，制造 Use-After-Free。  
  
1. 将悬空引用转化为原生内存写入能力，修改解释器的安全控制状态。  
  
这是一次跨越语言解释器内部多个安全边界的组合攻击。  
#### 3.1 第一阶段：受保护操作符.forceput的权限泄漏  
  
Ghostscript 的部分字体加载逻辑由 PostScript 实现，而不是全部由 C 语言实现。  
  
研究人员定位到 Resource/Init/gs_fonts.ps  
 中的内部辅助过程 .putgstringcopy  
。  
  
该过程需要操作受保护的字体字典，因此拥有普通文档代码不应该获得的写入权限。其关键逻辑可以简化为：  
```
dup length string copy
.forceput setglobal
```  
  
其中：  
- dup  
 复制栈顶对象；  
  
- length  
 获取对象长度；  
  
- string  
 分配字符串；  
  
- copy  
 将内容复制到目标；  
  
- .forceput  
 执行特殊的受保护写入；  
  
- setglobal  
 恢复内存分配模式。  
  
问题出在执行 .forceput  
 之前，程序没有充分验证操作数类型。  
  
研究人员构造了一个异常场景，使 copy  
 遇到不符合预期的对象并触发 typecheck  
 错误。通过 PostScript 的错误处理机制，攻击者能够影响恢复执行时的操作数栈。  
  
其结果是：一个原本由内部字体加载代码使用的特权写入操作，可能作用于攻击者选择的对象。  
  
这里的安全问题是**受信任代码在发生异常之后，继续使用了已经受到不可信执行上下文影响的状态**  
。  
  
**这是一种典型的权限混淆。**  
 受保护的内部函数本来拥有合法权限，但调用过程没有完整维持权限使用所依赖的状态不变量。  
#### 3.2 第二阶段：局部对象与全局对象的生命周期冲突  
  
Ghostscript 的 PostScript 内存模型区分局部对象与全局对象。  
  
为了理解漏洞，可以将其简化为两个生命周期不同的内存区域：  
- Local VM：对象可能随着 restore  
 操作被回收。  
  
- Global VM：部分对象能够在相应的局部内存恢复之后继续存在。  
  
正常情况下，解释器需要防止一个长期存活的对象持有已经被回收的短生命周期对象的有效引用。否则就会产生悬空引用（Dangling Reference）。  
  
研究人员发现，.forceput  
 在内部执行某些字典写入时，会临时调整字典的内存空间属性。这使得原本不应成立的跨生命周期引用关系成为可能。  
  
![uploadshell-20261125211146.png](https://mmbiz.qpic.cn/sz_mmbiz_png/L7VicJKsiaibFCRzfK8TkIPEzgRic73AWnewopWO5bKwzY2oPiaKrASeCfDabjdKVUNR4qF7Y1iaZMicatb4OzzYUFlgSGiaGa1D5uUCOBPJlY80cn8/640?from=appmsg "")  
  
在 restore  
 之后，局部对象已经失效，但全局字典仍然保存着原来的引用。  
  
如果内存分配器随后将这块空间重新分配给另一个对象，原来的引用就可能访问或修改新对象的数据。  
  
这正是 **Use-After-Free**  
 的核心机制：程序仍然使用一个逻辑上已经失效的引用，而该引用指向的物理内存已经有了新的用途。  
#### 3.3 第三阶段：从 UAF 到原生内存控制  
  
并不是所有 UAF 都能稳定转化为任意代码执行。  
  
攻击者通常还需要解决几个问题：如何让已释放的内存被重新使用，如何识别新对象的位置，以及如何将有限的内存读写能力扩展为更强的控制能力。  
  
pwn.ai的研究利用了 Ghostscript 内部对象的表示方式。  
  
研究人员在其测试的 x86-64 构建中观察到，一个 Ghostscript ref  
 对象占用 16 字节，包含类型及属性信息，以及值或指针等字段。  
  
通过控制释放后的内存复用，并观察不同对象之间的别名关系，研究人员证明，悬空引用能够影响仍在使用的解释器对象。  
  
这意味着漏洞的影响不再局限于 PostScript 字典中的普通数据，而可能触及解释器用于表示内部对象、设备及操作符的原生内存。  
#### 3.4 第四阶段：为什么修改内存能够绕过-dSAFER？  
  
-dSAFER  
 并不是 Linux 内核提供的隔离机制，而是 Ghostscript 解释器内部实现的一套访问控制。  
  
它通过检查文档操作是否被允许，限制对文件路径、外部程序和其他敏感资源的访问。  
  
这意味着解释器的安全性依赖于一个基本假设：  
负责执行安全检查的解释器自身，其内部状态没有被不可信文档破坏。  
  
Upload2Shell 恰好打破了这个假设。  
  
研究人员通过 UAF 获得的原生内存访问能力，定位到解释器内部与路径访问控制有关的状态，并在演示中改变了其行为。  
  
当负责执行权限检查的状态被破坏时，-dSAFER  
 即使在启动参数中仍然启用，也可能无法继续提供预期的保护。  
  
这里必须区分两种沙箱逃逸：  
<table><thead><tr><th data-colwidth="243" style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);background-color: rgba(255, 140, 0, 0.08);color: rgb(10, 8, 4);font-weight: 600;text-align: left;vertical-align: middle;"><section><span leaf="">类型</span></section></th><th style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);background-color: rgba(255, 140, 0, 0.08);color: rgb(10, 8, 4);font-weight: 600;text-align: left;vertical-align: middle;"><section><span leaf="">含义</span></section></th></tr></thead><tbody><tr><td data-colwidth="243" style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">Ghostscript </span><code style="font-family: Menlo-Regular, Menlo, Monaco, Consolas, &#34;Courier New&#34;, monospace;display: inline;color: rgb(199, 90, 0);background-color: rgba(255, 140, 0, 0.07);padding: 0.1em 0.4em;border-radius: 3px;font-size: 0.88em;"><span leaf="">-dSAFER</span></code><span leaf=""> 逃逸</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">绕过解释器内部的文档访问限制</span></section></td></tr><tr style="background-color: rgba(250, 248, 245, 0.8);"><td data-colwidth="243" style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">容器或内核沙箱逃逸</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">突破容器、命名空间、seccomp 或内核隔离边界</span></section></td></tr></tbody></table>  
Upload2Shell 直接证明的是第一种。  
  
研究人员实现的命令执行，使用的是文件渲染进程本身的操作系统身份。它并不意味着自动获得 root 权限，也不意味着已经突破 Docker 容器边界。  
  
如果渲染进程是 www-data  
，攻击者获得的初始执行上下文通常也是 www-data  
；如果渲染进程处于容器内部，命令通常首先在容器内执行。  
  
这一区分对真实风险评估至关重要。  
### 四、真实应用中的攻击链  
  
Upload2Shell 的现实威胁在于，Ghostscript 并非一个孤立运行的解释器，而是广泛存在于现代应用的文件处理依赖链中。  
  
研究人员在多个应用的特定配置下验证了远程命令执行：  
<table><thead><tr><th style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);background-color: rgba(255, 140, 0, 0.08);color: rgb(10, 8, 4);font-weight: 600;text-align: left;vertical-align: middle;"><section><span leaf="">应用</span></section></th><th style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);background-color: rgba(255, 140, 0, 0.08);color: rgb(10, 8, 4);font-weight: 600;text-align: left;vertical-align: middle;"><section><span leaf="">攻击入口</span></section></th><th style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);background-color: rgba(255, 140, 0, 0.08);color: rgb(10, 8, 4);font-weight: 600;text-align: left;vertical-align: middle;"><section><span leaf="">触发条件</span></section></th></tr></thead><tbody><tr><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">Roundcube</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">邮件附件</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">收件人查看邮件并触发附件预览</span></section></td></tr><tr style="background-color: rgba(250, 248, 245, 0.8);"><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">WordPress</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">媒体文件上传</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">具有媒体上传权限</span></section></td></tr><tr><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">Forminator</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">公共表单</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">特定文件上传配置</span></section></td></tr><tr style="background-color: rgba(250, 248, 245, 0.8);"><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">Essential Addons</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">注册组件</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">特定公开上传配置</span></section></td></tr><tr><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">CUPS</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">IPP 打印任务</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">可向打印队列提交任务</span></section></td></tr><tr style="background-color: rgba(250, 248, 245, 0.8);"><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">Mayan EDMS</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">邮件文档导入</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">启用相应的导入功能</span></section></td></tr><tr><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">MediaWiki / DokuWiki</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">文件上传</span></section></td><td style="padding: 0.6em 1em;font-size: 0.9em;border: 1px solid rgb(232, 229, 224);text-align: left;vertical-align: middle;"><section><span leaf="">特定缩略图处理配置</span></section></td></tr></tbody></table>  
这些应用虽然具有不同的业务功能，却共享着相似的底层处理逻辑：  
  
外部文件 → 业务应用 → 文件转换组件 → Ghostscript → 沙箱逃逸 → 命令执行。  
### 五、防御和风险识别  
  
针对 Ghostscript 沙箱逃逸漏洞，风险识别不应局限于组件版本检查，还应关注 ImageMagick 等间接依赖，以及文件上传、附件预览和文档转换过程中解释器的实际可达性。  
  
防御方面，应及时安装安全补丁，限制非必要的 PS/EPS 格式解析，并通过独立容器、最小权限和系统级访问控制隔离文档处理进程。同时，监控异常子进程、敏感文件访问及网络连接。  
  
**核心原则：不应将解释器自身的沙箱视为最终安全边界，而应通过纵深防御限制漏洞利用后的影响范围。**  
### 总结  
  
这条攻击链说明了一个更普遍的问题：  
  
**沙箱的安全性不仅取决于访问控制规则是否严格，还取决于执行这些规则的运行时本身是否能够抵抗不可信输入。**  
  
对于应用开发者，安全审计不能停留在上传扩展名、MIME 类型和文件保存路径；对于安全研究人员，需要沿着解码器、委派机制和解释器继续追踪；对于企业安全团队，则必须将文件处理 Worker、运行时依赖和操作系统级隔离纳入同一套风险管理体系。  
  
文件上传功能的安全边界，并不止于文件成功保存的那一刻。它一直延伸到最后一个读取、解析和执行该文件内容的组件。  
  
  
关注「极客零零七」，每周实战攻防干货。  
> 回复「提权」获取 Windows + Linux 提权速查表 · 回复「AD 攻击」获取 AD 域攻击手册  
  
  
  
  
参考链接  
  
https://pwn.ai/upload2shell/research.html  
  
https://thehackernews.com/2026/11/researchers-demonstrate-ghostscript.htm  
  
https://www.cve.org/CVERecord?id=CVE-2026-125396  
  
https://ghostscript.com  
  
  
推荐阅读  
  
[AI 脚本横扫防线：美国零售商遭遇 AI 辅助横向渗透的技术剖析](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486732&idx=1&sn=b6abe7331f1e2852e4aa825b70e93489&scene=21#wechat_redirect)  
  
  
[从 PHP2Shell 到 ESXi 擦除：一次Check Point 网关失陷的完整攻击链拆解](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486719&idx=1&sn=08fe69b907c466d0c56cd7023938d26f&scene=21#wechat_redirect)  
  
  
[给 AI 黑客注入“冷静”，它真的更会挖洞了？](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486672&idx=1&sn=662f99e609543c53ecc90eec74552725&scene=21#wechat_redirect)  
  
  
[AWS IAM 架构体系与攻防漏洞深度剖析](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486659&idx=1&sn=e919d29da91dc4d0e6323a9b8e07d0f9&scene=21#wechat_redirect)  
  
  
[OWASP 大模型与生成式 AI 应用安全 Top 10（2026版）](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486645&idx=1&sn=ff369708c98d6c76f39e5e9a7fcd3097&scene=21#wechat_redirect)  
  
  
[告别只会搜索的聊天机器人：如何用 754 个专业 Playbook，把 AI 变成你的高级安全分析师？](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486571&idx=1&sn=69551a5433f22eac17fe671722033c93&scene=21#wechat_redirect)  
  
  
[[工具·书·人] HTB × HackerOne 联手推出 AI CTF：大模型漏洞赏金，下一个黄金赛道](https://mp.weixin.qq.com/s?__biz=Mzk2NDgwNjA2NA==&mid=2247486271&idx=1&sn=9cb654874991ba016c3513922d067c13&scene=21#wechat_redirect)  
  
  
  
