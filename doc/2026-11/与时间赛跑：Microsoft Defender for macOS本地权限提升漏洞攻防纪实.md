#  与时间赛跑：Microsoft Defender for macOS本地权限提升漏洞攻防纪实  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-11-16 03:08  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZgQQrBo3KOX4vXyaibds86WmVy4JksWlicdKEx1BXLpibcOjfalnDsRoAJgUibsDnIE0KoKC0YRbLOXuFno2nIbcU9Pic9UPooFFWyQ/640?wx_fmt=png&from=appmsg "")  
  
Microsoft Defender（微软安全卫士）作为一款终端防护产品，与市面上诸多EDR（终端检测与响应）和AV（防病毒）产品一样，因其运行于系统最高权限且内置大量内核级组件，自然而然成了漏洞猎人们眼中的"香饽饽"。它还有一个显著特点扩大了其攻击面：跨平台支持——同时能在Windows、macOS和Linux三大系统上运行，每个系统都有各自一套截然不同的操作系统API（应用程序编程接口）和安全控制机制。  
  
2026年初，两起针对Microsoft Defender for Windows的本地权限提升(本地授权升级)漏洞被公开披露并已在真实攻击中被利用。其中CVE-2026-33825代号"BlueHammer"，利用的是竞争条件漏洞；CVE-2026-41091则利用的是微软卫士检测与修复流程中对符号链接验证的缺失。虽然这些漏洞都已获得微软的补丁修复，但经过对根本原因的深入分析，我发现这些漏洞虽然在Windows上可能已被修复，但在macOS平台，由于系统安全与文件操作API的工作方式存在微妙差异，导致这些漏洞的变种依然存在被利用的可能。  
  
微软后续为这两个macOS漏洞分配了编号：**CVE-2026-50657**  
（任意文件读取/信息泄露）和**CVE-2026-50658**  
（TOCTOU竞争条件/权限提升）。我在2026年的BlueHat Asia安全大会上介绍了这项研究成果，本文即为此研究的完整技术报告。  
  
尽管这些漏洞在获得初始利用原语后相对直接，但我认为这是一个绝佳机会，值得深入探讨macOS特有的安全机制，并对比分析与Windows同类机制的差异所在。跨平台漏洞变体分析往往容易被忽视，却往往能揭示出重要的安全机制差异——某漏洞可能在某个系统中已修复，但在另一系统中却依然可被利用。  
## macOS特有的进程间通信安全机制  
  
本文探讨的漏洞之所以独特性地存在于macOS平台，原因有二：一是微软卫士在macOS上的架构实现与Windows版本截然不同；二是macOS操作系统自身在处理进程身份标识、授权机制和链接方式上也有其独特之处。  
  
特权进程间通信（IPC）一直是安全漏洞的"重灾区"。攻击模式通常千篇一律：一个低权限的用户应用程序通过某种通信管道向一个高权限的监听器发送请求，后者代其执行高权限操作。问题往往出在监听器缺少或存在缺陷的身份认证/授权验证环节，对特权操作的输入数据验证不够充分，或者是对操作系统授权API的配置不当。我曾在关于[Zscaler客户端连接器（CVE-2023-41973）]的漏洞分析中深入探讨过这一模式。操作系统厂商通常会提供一些机制来帮助开发者规避这些"陷阱"，因此了解macOS在这个领域提供了哪些防护手段很有必要。  
## 代码签名、开发者ID与公证机制  
  
macOS运行的每一个可执行文件都必须经过代码签名。那些在App Store以外分发的第三方软件，通常使用由苹果颁发给开发者**团队ID**  
（微软的团队ID为UBF8T346G9  
）的**开发者ID**  
证书进行签名，并会经过苹果的**公证**  
流程。  
  
macOS的**代码签名服务API**  
（包括SecStaticCodeCreateWithPath  
、SecCodeCopyGuestWithAttributes  
、SecStaticCodeCheckValidity  
等函数）允许用户空间程序用一种小型"谓词语言"来验证另一个文件或正在运行的进程是否满足**代码签名要求**  
。例如可以要求签名链必须能追溯到苹果官方（anchor apple  
），或经过苹果认证（如开发者ID，即anchor apple generic  
），或者精准匹配特定开发者（如identifier "com.microsoft.wdav" and certificate leaf[subject.OU] = "UBF8T346G9"  
）。这套机制类似于Windows平台上的WinVerifyTrust  
 API和Authenticode策略体系。  
  
代码签名机制使得macOS上的高权限守护进程能够验证与其通信的IPC对端进程是否确实是来自厂商的正规签名程序。但也正因如此，后面我们将看到，一旦这种验证机制出现纰漏，后果将是致命的。  
## 系统完整性保护与加固运行时  
  
系统完整性保护（SIP）是macOS默认开启、由内核强制执行的强制性访问控制机制：它严格禁止修改受保护的系统位置，并阻止对受保护进程的代码注入或系统调用拦截。SIP的存在意味着攻击者不能简单地通过向某个微软卫士的签名助手进程注入恶意代码来"借用"其身份进行IPC通信。当SIP启用时，针对苹果签名或已加固二进制文件的task_for_pid  
（进程注入）操作会失败，同时DYLD_*  
系列环境变量（相当于Windows的DLL注入）的注入也会被拦截。因此，为了成功伪装成受信任进程进行IPC通信，攻击者必须在目标程序中找到其他可利用的缺陷。  
## 授权服务  
  
微软卫士在macOS上的IPC机制建立在苹果的授权服务框架之上，以此执行需要特权的操作。由于这套API在开发者群体中存在不少误解，有必要准确说明其设计目的和工作原理。  
  
程序首先需要获取一个AuthorizationRef  
授权引用，然后调用：  
```
OSStatus AuthorizationCopyRights(AuthorizationRef authorization,                                 const AuthorizationRights *rights,                                 const AuthorizationEnvironment *environment,                                 AuthorizationFlags flags,                                 AuthorizationRights **authorizedRights);
```  
  
通过此函数，程序可以使用字符串键值（例如com.microsoft.mdatp.controllogging  
）来申请一个或多个**权限**  
。该调用会查询系统的**授权策略数据库**  
，其中每个权限都映射到一条相应的**规则**  
。系统内置的规则包括allow  
（允许）、deny  
（拒绝）、is-root  
（必须是root用户）、authenticate-admin  
（管理员认证）和authenticate-session-owner  
（会话所有者认证）等。如果请求的权限符合对应规则要求，函数将返回errAuthorizationSuccess  
成功状态。Csaba Fitzl在其[深入剖析macOS授权机制]的技术文章中对此有详尽分析。  
  
在我所分析的软件版本中，微软卫士在授权策略数据库中注册了13个自定义权限。通过读取具体的生效规则，可以了解其具体要求。例如执行命令：  
```
security authorizationdb read com.microsoft.mdatp.readlogs
```  
  
得到的规则中，rule  
数组包含is-root**或者**authenticate-admin-30  
。换句话说，只有当发起授权请求的进程本身已经拥有root权限，**或者**  
有管理员在弹出对话框中完成认证（认证结果缓存30秒），才会授予该权限。  
  
关键在于，这套授权机制是**"自我限制型"**的——借用苹果官方的术语来说，它是一种"可选"功能。AuthorizationCopyRights提供的保护效果完全取决于代码是否主动调用它并根据返回结果采取相应行动。它**  
并非**操作系统自动维护的强制执行边界。  
## 对端进程身份验证与审计令牌  
  
macOS提供了多种IPC通信机制：Mach消息、**XPC**  
和**NSXPC**  
（现代推荐的高级抽象），以及传统的BSD套接字（包括AF_UNIX  
域套接字）。微软卫士的wdavdaemon  
进程使用的就是AF_UNIX  
流式套接字。  
  
对于任何特权监听器而言，一个核心安全问题是：**对端到底是什么进程？它有权执行此操作吗？**  
 一旦判断错误，往往就是本地权限提升的开始。在macOS上，程序可以通过以下几种方式来确认对端身份：  
- **getsockopt(client, SOL_LOCAL, LOCAL_PEERPID, …)**  
：在客户端发起connect()  
连接的时刻获取对端进程的PID（进程ID）。XNU（macOS内核）会在这一时刻记录PID，之后永不更新。  
  
- **proc_pidpath(pid, …)**  
：获取某个PID对应的**当前**  
可执行文件的绝对文件系统路径。这个函数回答的是："进程N**此刻**  
运行的是什么二进制文件？"如果进程使用execve()  
函数重新执行了新程序，这个返回值就会改变。  
  
- **getsockopt(client, SOL_LOCAL, LOCAL_PEERTOKEN, …)**  
，对于XPC来说是xpc_connection_get_audit_token  
：获取对端进程的**审计令牌**  
。这是一个不透明的数据结构，除PID外还包含pidversion  
版本计数器（每当进程执行execve()  
时该值就会递增）。审计令牌还附带了用于查询权限的signing_id  
。它是唯一既能绑定到特定进程，又在该进程重新执行后会失效的对端凭证。Scott Knight对此有[一篇优秀的审计令牌解析文章]。  
  
一种常见的易受攻击模式是仅依赖前两种方式：获取对端的PID后，仅根据该PID检查可执行文件路径是否在白名单内。这种作法在"获取PID的时刻"和"检查路径的时刻"之间留下了一个竞争窗口。这种"PID重利用"技术最早由Samuel Groß在2018年的WarCon大会上通过其演讲[不要相信PID！]中提出。  
## 微软卫士的攻击面分析  
## 进程模型  
  
在一台运行着微软卫士for macOS的系统上，相关的关键进程包括：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">进程名称</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">权限级别</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">主要功能</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">wdavdaemon</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">root</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">核心权限代理。负责管理两个IPC套接字、防护引擎、文件隔离和诊断功能。</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">wdavdaemon_enterprise</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">root</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">EDR企业功能工作进程。</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">wdavdaemon_unprivileged</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">_mdatp</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">文件扫描工作进程；与root守护进程通信。</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">Microsoft Defender Helper</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">普通用户</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">图形界面/系统托盘助手程序。</span></section></td></tr></tbody></table>  
  
拥有root权限的wdavdaemon  
进程使用微软证书签名（团队IDUBF8T346G9  
），以系统LaunchDaemon服务形式运行。  
## IPC套接字  
  
wdavdaemon  
开放了两个AF_UNIX  
流式套接字：  
- /var/run/wdavdaemon.sock  
：**公开**  
套接字。  
  
- /var/run/priv_wdavdaemon.sock  
：**私有**  
套接字。  
  
这两个套接字的创建权限都是**0666**（srw-rw-rw-）——由root所有但全局可读写。这意味着**  
任何本地进程都能连接上它们**。"私有"套接字仅仅是设计意图上的私有：它负责处理更敏感的操作码（如威胁处置、配置更改等），但从访问权限而言，它与公开套接字一样易于连接。  
  
![两个IPC套接字都是全局可写的srw-rw-rw-](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZh8HmcxpYrvnrV0XcRK3ibACFaTkkmSDnW5LP0kRPBdKcBTyvUqqUnQCwRWRon0kQPpGiccQOav6nZYlUnB37ias1TRbTYNCMblP4/640?wx_fmt=png&from=appmsg "")  
  
两个IPC套接字都是全局可写的srw-rw-rw-  
  
两个套接字共享相同的基于SOCK_STREAM  
的报文结构：  
```
frame_header (16字节，大端序)  u32  seq            ; 序列号，回复报文会原样回显  u8   reserved[8]    ; 固定填充0  u32  body_len       ; 后续报文体的长度body (body_len字节)  u32  opcode  (小端序)  ...  payload        ; 操作码特定的数据载荷
```  
  
本文重点分析其中两个关键操作码：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">操作码</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">功能名称</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">主要作用</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">0x05</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">handle_threat</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">隔离/还原受感染文件</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">0x0f</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><code><span leaf="">get_diagnostic_files</span></code></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">生成全局可读的诊断文件压缩包</span></section></td></tr></tbody></table>  
  
绝大多数特权操作码都会在libwdavdaemon_core.dylib  
库中统一收敛到对AuthorizationCopyRights  
的一次调用（重新验证com.microsoft.mdatp.*  
权限）。如果报文体中没有提供有效的32字节AuthorizationExternalForm  
授权数据，带有权限校验的操作码会统一返回"认证失败"响应，具体的处理函数根本不会执行。这是一道合理的**授权同意**  
关卡，但它仅能确认管理员已经批准操作，**并不能**  
验证调用者是否真的是合法的微软卫士客户端程序。调用者身份验证的任务，被完全委托给了独立的对端连接验证环节。  
## 对端连接验证机制  
  
在处理任何具体请求之前，守护进程首先会验证连接过来的对端身份：  
1. 调用getsockopt(SOL_LOCAL, LOCAL_PEERPID)  
获取对端连接时的PID。  
  
1. 调用proc_pidpath(pid, …)  
获取该PID当前对应的可执行文件路径。  
  
1. 将该路径与运行时构建的、以"/Applications/Microsoft Defender.app"  
开头的白名单进行严格的**前缀匹配检查**  
。这种匹配会严格遵循目录边界，所以像/Applications/Microsoft Defender.appEVIL  
这样的路径**无法**  
通过验证。  
  
1. 如果路径匹配成功，再通过代码签名服务接口（anchor apple  
 / anchor apple generic  
）进行签名验证。不过，对于非应用程序包安装的程序，这一步永远不会被执行，因为前缀检查会首先将其拒绝。  
  
1. 验证失败，则立即调用shutdown(SHUT_RDWR)  
 + close()  
断开连接。从客户端角度看，就像是握手过程中突然遇到了连接中断。  
  
对于普通的uid-501用户进程，第三步的前缀匹配检查会无条件拒绝。然而请注意，第1步和第2步分别使用了两种**不同**  
的对端标识信息：一个是"冻结"在连接时刻的PID，另一个是"实时"查询到的进程路径。这种实现恰恰创造了我们后面将要利用的时间差窗口。  
## 绕过对端验证的竞赛攻击  
  
这种时间差竞赛攻击实施起来相当直接：  
- **LOCAL_PEERPID**  
 的获取时机固定在客户端建立连接的瞬间，之后该值就固定不变了。  
  
- **proc_pidpath**  
 返回的是PID当前对应的可执行文件路径，这个路径我们可以通过调用execve()  
函数来修改。  
  
严格来说，本文采用的并不是那种被回收重复使用的PID恰好落在一个不同进程上的"PID回收重用"攻击，而是**同一个**  
进程通过execve()  
重新执行不同程序，改变了其PID对应的路径。这种操作属于"PID信任家族"内的变种。  
  
关于execve()  
函数有一个关键特性：内核会在**返回用户空间之前、新程序第一条指令执行之前**  
更新进程记录的二进制映像路径。被执行的新程序甚至不需要有实际功能，不需要执行任何IPC操作，甚至不需要继续存在下去——它只要能在守护进程调用proc_pidpath  
时**成功完成PID路径更新**  
就算完成任务了。  
  
因此，攻击脚本会创建（fork）大量子进程，每个子进程按照这种时序竞速执行：  
1. s = socket(AF_UNIX, SOCK_STREAM)  
 - 创建UNIX域流套接字  
  
1. connect("/var/run/wdavdaemon.sock")  
 - 连接守护进程：此刻LOCAL_PEERPID  
被"冻结"为该子进程的PID  
  
1. send([16B header][u32 LE 0x0f])  
 - 发送操作请求入队  
  
1. execve("/Applications/Microsoft Defender.app/Contents/MacOS/wdavdaemon_unprivileged.app/Contents/MacOS/wdavdaemon_unprivileged", argv=["wdavdaemon"], …)  
 - 执行微软签名的合法二进制文件：内核将PID路径更新为白名单内的、微软签名的程序路径（新程序可能会因为单例检查而立即退出，但这并不影响攻击目的）  
  
与此同时，守护进程在accept()  
接受连接后执行：  
1. getsockopt(LOCAL_PEERPID, &pid)  
 - 获取连接PID  
  
1. proc_pidpath(pid, path)  
 - 获取该PID**此刻**  
对应的路径  
  
1. prefix-match(path, "/Applications/Microsoft Defender.app")  
 - 执行前缀匹配检查  
  
1. 如果匹配成功 → 执行请求处理函数；如果匹配失败 → 立即shutdown(fd)  
断开连接  
  
一旦守护进程执行第2步的时刻**晚于**  
子进程执行第4步的时刻，proc_pidpath  
返回的路径就会是微软应用包路径，校验就会成功通过。尽管实际发起连接的进程原本是攻击者的恶意脚本（在我的PoC中是一个Perl脚本），但到守护进程读取路径时，那个子进程已经通过execve()  
重新执行了白名单内的微软签名二进制文件，同时满足了路径前缀匹配和代码签名验证。此时，这个被攻击者控制的子进程就能任意调用wdavdaemon  
的所有IPC功能。  
## 实现本地权限提升  
  
绕过了对端身份验证后，我们就能在任意套接字上调用所有的操作码，将微软卫士自身的特权处理函数转变为攻击的"工具"。  
  
有两个操作码可以实现权限提升：0x0f get_diagnostic_files  
能提供root权限的任意文件读取能力；0x05 handle_threat::restore  
则能实现root权限的任意文件写入操作。  
## 任意文件读取漏洞（CVE-2026-50657）  
  
与其他高权限操作码不同，0x0f get_diagnostic_files  
操作**完全绕过**  
了AuthorizationCopyRights  
授权检查环节。只需要发送一个不包含授权信息、仅包含[u32 LE 0x0f]  
操作码的报文，就能直接获得完整的响应结果：  
```
result<optional<boost::filesystem::path>>  成功状态path = "/Library/Application Support/Microsoft/Defender/wdavdiag/<UUID>.zip"
```  
  
每次调用都会让root守护进程在/Library/Application Support/Microsoft/Defender/wdavdiag/  
目录下以UUID命名新建一个诊断压缩文件，而这个目录的权限设置是**全局可读可搜索**  
（0755 root:wheel  
）。  
  
![wdavdiag目录全局可读并保存生成的诊断压缩包](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZgOt1oCXz0jOJsibII7iboNkx6cmQdu0lPV01O9iaDfUaVsLfO6J0lpPxajRHrsrkGkNyib8DUCs5sHIzhFG64PFXDuwrKic1qeGaC8/640?wx_fmt=png&from=appmsg "")  
  
wdavdiag目录全局可读并保存生成的诊断压缩包  
  
详细检查生成的ZIP文件内容发现，诊断收集器会打包**$HOME/Library/Logs/Microsoft/Defender/  
目录下的所有常规文件**。这是首次启动微软卫士时自动创建的用户专属日志目录。文件收集函数使用守护进程的标准文件打开流程打开每个要收集的日志文件。由于守护进程运行在root权限，所有文件的open(O_RDONLY)  
（只读打开）操作都会成功。  
  
![解压诊断压缩包可见收集到的所有卫士日志文件](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZhITSkUSXnzh1XfHWtvcfyuT3A0piaOl44XAzEZOibns4gBwT26Dvc4eAh924vBTw6OpaMibQypntaIZjy9yia9NZWKUoRibrgP8F18/640?wx_fmt=png&from=appmsg "")  
  
解压诊断压缩包可见收集到的所有卫士日志文件  
### 使用硬链接实现攻击重定向  
  
因为$HOME/Library/Logs/Microsoft/Defender/  
目录的所有者就是当前用户（uid 501，权限0755  
），所以用户可以在其中自由创建目录条目，包括**硬链接**  
。这个漏洞之所以能实现任意文件读取，关键在于POSIX的link(2)  
系统调用的一个特性：创建硬链接**不需要**  
有源文件的读取权限。它只需要源文件的路径组件可被搜索（即有访问路径的权限），且目标目录对创建者可写。  
  
因此，作为uid 501的普通用户可以执行：  
```
link("/etc/kcpassword",     "/Users/eugenelim/Library/Logs/Microsoft/Defender/microsoft_defender_app_logs.log");
```  
  
这个操作是完全合法的，即使目标文件/etc/kcpassword  
只有root用户才能访问。通过此操作在当前用户可写的目录下创建的硬链接条目，实际指向的是root用户所有的文件系统节点（inode）。  
  
为什么不使用符号链接而用硬链接呢？这是因为当诊断收集器遍历日志目录时，它会检查每个目录条目，并使用Boost文件系统库的boost::filesystem::status()  
函数进行筛选，只处理类型为**常规文件**  
（regular_file）的条目（返回的file_type  
枚举值）。被过滤掉的类型包括：directory_file  
（目录）、symlink_file  
（符号链接）、block_file  
（块设备）、character_file  
（字符设备）、fifo_file  
（管道）、socket_file  
（套接字）、reparse_file  
（重解析文件）和type_unknown  
（未知类型）。所以，如果创建的是普通符号链接，它会被直接过滤掉，永远不会走到真正的open()  
打开文件环节。  
  
![微软卫士只收集类型为常规文件的条目](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZhbIXxL0UvBvfRm7Fk5noWGBib3DQAWKobCicRL0FOnzt1eQAVsTOUNQtJ95uERMicjDZRibzOQWmhdLyh8CkvAoY5BepaZ8jxCRNo/640?wx_fmt=png&from=appmsg "")  
  
微软卫士只收集类型为常规文件的条目  
  
相比之下，**硬链接**  
在文件系统层面并不被视为特殊的文件类型：它同样被归为普通的常规文件类型（S_IFREG  
，uid=0），能够顺利通过"常规文件"过滤器。当守护进程以root权限打开该硬链接时，实际上打开的是目标系统文件，从而将敏感文件的字节内容打包到用户可读取的诊断压缩包中。  
  
这个漏洞可以看作是Windows上CVE-2026-41091漏洞在macOS平台的"变种"，但仅靠修复符号链接问题是无法防范这种攻击的。Windows系统自2015年起就要求创建硬链接需要对目标文件拥有FILE_WRITE_ATTRIBUTES  
权限（James Forshaw在[在坚硬与硬链接之间]一文中详细记录过这一点），然而macOS的硬链接创建却完全不需要目标文件的任何访问权限。这个安全假设上的重大差异，正是Windows和macOS平台安全机制的重要区别之一。  
  
一旦获得了root权限的任意文件读取能力，进一步的权限提升就不再困难。值得关注的有价值目标文件包括：/var/db/dslocal/nodes/Default/users/*.plist  
（存储ShadowHashData  
本地账户密码哈希，相当于macOS的SAM数据库）、/etc/kcpassword  
（经过XOR混淆处理的自动登录密码）、TCC.db  
（macOS透明性、可控性和隐私数据库）、其他用户的~/Library  
文件夹中的敏感数据、SSH密钥、以及浏览器cookie存储文件等。  
## 任意文件写入漏洞（CVE-2026-50658）  
  
第二个漏洞瞄准的是微软卫士的**私有**  
套接字，具体操作码是handle_threat::restore  
（操作码0x05  
，子操作restore  
，还原被隔离文件），这项功能允许已授权的管理员将被隔离文件释放到指定的恢复位置。  
### 可选跳过授权验证的"后门"  
  
还原请求将AuthorizationExternalForm  
格式的授权数据编码为std::optional<array<u8,32>>  
（标准库可选类型，包含32字节数组）。其编码格式是：一个表示"是否存在"的标志字节，后面**仅当标志为"存在"时**  
才跟着长度值和实际字节数据：  
```
body (还原操作码0x05):  u32 LE opcode      = 5  u32 LE pad         = 0  u64 LE tid_len     + threat_id            ; ASCII格式的UUID，36字节  u32 LE sub_action  = 1                     ; restore子操作  u8  has_dest_path  = 1  u64 LE dest_len    + dest_path             ; 守护进程会以root权限写入此处  u8  has_auth_blob  = 0   <-- 绕过点    ; 标志为0，后面无授权数据字节
```  
  
当has_auth_blob  
标志字节设为0时，数据解析器会读取到一个**缺失的可选值**  
，对应的处理函数会**完全跳过授权验证流程**  
。整个过程既不调用AuthorizationCreateFromExternalForm  
转译授权数据，也不调用AuthorizationCopyRights  
验证授权。  
  
![当has_auth_blob=0时，处理函数直接跳过了授权检查环节](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZhxkaJZm6rA0snaCJaib8yYXFlycIA0F6ibQbibme3AaHmSrTlgHia3Q73ve2yGnr7cibdPWFK22QbmGRcDl4Z5LbiaZkFh8fhMdPSIA/640?wx_fmt=png&from=appmsg "")  
  
当has_auth_blob=0时，处理函数直接跳过了授权检查环节  
  
这与"提供无效授权数据"的处理路径不同：如果一个32字节全零的授权数据包（has_auth_blob = 1  
，内容全零）会被AuthorizationCreateFromExternalForm  
函数正确**拒绝**  
。而这个漏洞的关键在于，**缺失**  
的可选值会让程序流程直接"落空"到未经授权的处理路径上。  
  
此时剩下的唯一防护就是对端验证，但私有套接字使用的恰恰是与公开套接字相同的getsockopt(LOCAL_PEERPID) → proc_pidpath → SecStaticCodeCreateWithPath  
验证模式，因此同样容易受到相同的PID重用竞赛攻击。如此一来，非root权限的普通用户也能将被隔离的内容恢复到任意的系统位置。  
### 准备攻击用的"隔离文件"  
  
因为还原处理函数只能还原**先前已被隔离**  
的文件，攻击者需要先让微软卫士隔离一段自己控制的文件内容。通常的做法是：先在/tmp/<文件名>  
中写入一段EICAR测试字符串（标准的防病毒软件测试文件），微软卫士的实时保护引擎就会将其隔离，并**保留原始的基本文件名**  
作为隔离元数据的一部分。  
  
被隔离的文件最终存储在/Library/Application Support/Microsoft/Defender/quarantine/  
目录下，而且这些文件的**所有者仍是用户自身**  
。存储的内容只是简单的XOR(0x25)  
编码处理后的数据。这使得攻击者能够轻易地将隔离的EICAR文件替换为任意恶意内容，为后续还原操作做好准备。攻击脚本随后触发restore  
操作并指定攻击者控制的dest_path  
目标路径，微软卫士会解码并写入攻击者预先准备的内容到<目标路径>/<原始基本文件名>  
位置。这实质上实现了root权限的任意文件写入。  
  
但这里有一个重要的限制需要注意：还原操作流程只能写入**尚不存在的**  
目标路径。  
  
![还原处理函数拒绝覆盖已存在的目标文件](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZh4QNOx2b7KqFica5gQ5QdoE5bq2gF0Sx5XXuftEsupkvibwg7OYD0RJzWR1Va3HrlibuQjSr1KoUuxMP8S6XvpeTxAPruHPIWqibI/640?wx_fmt=png&from=appmsg "")  
  
还原处理函数拒绝覆盖已存在的目标文件  
  
处理函数会先检查目标路径的状态；如果目标已存在，理论上可以进行覆盖并创建备份的操作模式，但对于还原这个子操作，相关标志位被硬编码为**不覆盖**  
模式，遇到已存在的文件会直接拒绝并返回EEXIST  
错误。所以这个漏洞实际上是一个"任意文件**创建**  
"原语，而非"覆盖"原语。  
## 从文件写入到代码执行的跨越  
  
这一部分说起来有点令人尴尬——文件仅能创建不能覆盖的限制困扰了我好一阵子，因为通常我知道的从文件写入漏洞提升到代码执行，大多依赖于覆盖或追加到现有关键文件。  
### 尝试一：利用系统启动守护进程  
  
系统启动守护进程（LaunchDaemons，位于/Library/LaunchDaemons/  
目录）是macOS的"服务程序"，类似于Windows的Scheduled Tasks计划任务或系统服务。创建新的启动守护进程不需要覆盖现有文件，看起来是个理想方案。然而问题在于，以root权限运行的启动守护进程脚本必须由root  
用户所有，而我们的还原处理函数在最后一步会通过chown()  
函数恢复文件的**原始所有者**  
：  
```
1. open(<dest>/<basename>.tmp, O_CREAT|O_EXCL)        ; 以root权限创建临时文件2. write(解码后的原始内容)3. rename(<dest>/<basename>.tmp, <dest>/<basename>)   ; 重命名为最终文件4. chown(<dest>/<basename>, source_uid, source_gid)   ; **通过路径名恢复原始所有者**
```  
  
注意第4步的chown()  
是**通过路径名**  
调用，而非通过文件描述符调用。在第3步和第4步之间，最终文件已经存在且所有者为root，但如果<dest>  
是一个**符号链接**  
，并且在那个短暂的时间窗口内将其重定向到另一个文件（诱饵文件），那么第4步的chown()  
就会落到不同的文件上，而我们真正想要保留root所有权的目标文件就能维持root所有者身份。这个时间窗口比之前的PID重用竞争要短暂得多，但理论上仍有获胜可能。  
  
然而，另一个难题出现了：非特权用户无法通过launchctl bootstrap  
命令加载系统域的plist配置文件。这意味着，即使成功获得了root所有的启动守护进程文件，攻击者也无法主动触发它的执行，必须等到系统重启。  
### 尝试二：利用微软卫士自带的安装监视器守护进程  
  
既然我自己无法触发启动守护进程，也许微软卫士自身会帮我触发。产品附带了多个安装监视器守护进程，专门监控文件追踪和DLP子系统，当指定的文件出现在/Library/Application Support/Microsoft/{DLP, Defender/dlp_processor, Defender/tracer}/install/  
监控目录下时，就会自动运行一个root权限的脚本：  
```
cp  "<app-bundle>/.../com.microsoft.wdav.tracedaemon.plist"  /Library/LaunchDaemons/launchctl bootstrap system  /Library/LaunchDaemons/com.microsoft.wdav.tracedaemon.plist
```  
  
乍一看，这是个无须系统重启的确定性攻击路径。但这条路同样走不通：我们的写入原语只能**创建**  
文件，而cp  
命令的源文件是应用软件包内预先存在的、root所有的plist文件，还原操作会拒绝覆盖它。因此，脚本复制到启动目录的始终是微软卫士自身的plist文件，而不是我的恶意plist。如果把恶意文件放在cp  
命令的**目标位置**  
，也会被root用户的cp  
命令在执行bootstrap  
之前覆盖掉；而且我也没有权限锁定目标文件，因为设置文件不可变标志需要拥有该文件的所有权，而launchd  
启动服务又要求plist文件必须由root所有。这条路径的每个变种都要求覆盖或重写已存在的文件，而这正是我们的原语无法做到的。  
### 尝试三：回归Unix传统，巧用cron  
  
macOS系统中保留了相当多源自传统Unix的组件，其中之一就是大家都熟悉的**cron定时任务系统**  
。  
  
在macOS上，cron本身是以com.vix.cron  
的launchd服务形式存在的。在标准安装系统中，它虽然已被标记为"已弃用"状态，但依然静静地躺在那里等待唤醒。系统中通常没有默认的/etc/crontab  
文件，而这个服务的KeepAlive  
保持生存条件精确地定义为：只要该文件存在：  
```
<key>KeepAlive</key><dict>    <key>PathState</key>    <dict>        <key>/etc/crontab</key>        <true/>    </dict></dict>
```  
  
因此，只需创建/etc/crontab  
文件，cron服务就会**立即自动启动**  
，根本不需要系统重启。更有意思的是，这个文件通常不存在，攻击属于"创建"而非"覆盖"操作，完美契合了我们的写入原语。ironically，macOS将cron标记为弃用，反而让它在攻击链中成为了绝佳候选目标。  
  
更好的是，这种方式巧妙规避了导致前两种尝试失败的所有权问题。之前LaunchDaemon路径的最大障碍在于：启动守护进程必须由root所有，否则launchd  
会拒绝加载，这就迫使我们必须赢下那次极其短暂的chown()  
竞争窗口。而**cron对/etc/crontab文件不设置所有权限制**  
：它只读取文件内容，并按照每行第六个字段指定的用户身份来执行对应的命令。即使/etc/crontab  
文件本身由uid 501普通用户所有，只要有一行以* * * * * root …  
开头，该行命令就会以**root权限**  
执行。  
  
最终的攻击载荷就是一个简单的系统级crontab行：  
```
* * * * * root /bin/sh -c 'cp /bin/zsh /tmp/lpe_root_shell; chown root:wheel /tmp/lpe_root_shell; chmod 4755 /tmp/lpe_root_shell'
```  
  
这样就完成了完整的权限提升链条：创建一个名为crontab  
的EICAR"诱饵文件"，将其隔离内容替换为上面这行crontab命令，然后通过跳过授权的方式触发还原操作，将目标指定为/etc  
目录。对端身份验证的竞赛成功后，/etc/crontab  
文件（uid 501，权限0644  
）就被创建出来；launchd  
立即启动cron服务；大约一分钟后，/tmp/lpe_root_shell -p  
命令就能触发一个交互式的root权限shell。由于只需要赢下一次PID重用竞赛，这个攻击方案的稳定性显著提高。  
## 结论与跨平台安全思考  
  
这两个安全漏洞均已向微软安全响应中心（MSRC）报告，并在2026年7月"补丁星期二"获得修复：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">CVE-2026-50657</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">CVE-2026-50658</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">漏洞类别</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">信息泄露/隐私泄露（非授权访问私人信息）</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">TOCTOU时间检查竞争条件</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">安全影响</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">本地信息泄露，可任意读取系统文件</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">本地权限提升，通过任意文件写入获得root权限</span></section></td></tr></tbody></table>  
  
虽然macOS平台上的LPE（本地权限提升）漏洞是独立发现的，但它们与2026年Windows Defender的权限提升漏洞形成"镜像"并非巧合。有趣的是，**相同的两类漏洞**  
在不同的操作系统上，会因为IPC机制和安全原语的差异而被不同方式处理。  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">Windows系统特点</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">macOS系统特点</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">进程身份验证</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">依赖进程句柄，内核将进程对象锁定在内存中</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">必须显式检查审计令牌才能有效防范PID重用</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">授权验证机制</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">内核驱动的访问令牌和完整性级别评估体系，自动与目标ACL（访问控制列表）对比</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">&#34;自我限制型&#34;机制，需要程序员主动调用授权服务API才会生效</span></section></td></tr></tbody></table>  
  
正是这些核心差异，导致在macOS平台上的漏洞利用方式与Windows迥然不同：  
- **身份验证方式**  
。Windows的TOCTOU攻击通常移动的是**文件**  
（在检查和使用之间修改文件内容/路径）；macOS则提供了更强的攻击原语：execve()  
可以在连接建立后、身份验证前**改变自身进程对应的可执行文件路径**  
。这就是PID重用攻击在macOS平台为何如此可靠的根本原因——攻击者能够完全在自己进程内部控制主体切换的确切时机。  
  
- **链接机制安全假设**  
。在Windows系统上，链接创建的门槛相对较高：创建符号链接需要SeCreateSymbolicLinkPrivilege  
特权（默认授予管理员），创建硬链接需要对目标文件拥有写入权限，而创建连接点（仅重定向目录，但不需要特殊权限）现在也可以通过重定向保护机制进行防御。但macOS采用传统的Unix风格链接机制，其link(2)  
系统调用完全不检查是否对目标文件有读取或写入权限，只要源路径可搜索（即有访问权限）、目标目录可写就能成功创建链接。正是这个毫不起眼的特性，让无特权用户在macOS上就能完成特权文件读取攻击。  
  
- **授权机制设计理念**  
。Windows的授权体系通过令牌、完整性级别和ACL多层级自动控制；而macOS的授权服务框架是**自我限制型**  
的，开发者需要手动实现调用，这使得错误配置或安全漏洞特别容易发生。macOS特有的安全问题是：管理员**同意**  
（通过AuthorizationCopyRights  
验证）与调用者**身份真实**  
（通过审计令牌验证代码签名）是**两个完全不同、但常常被混淆的安全问题**  
。  
  
由于平台之间的这些深层差异，仅仅在Windows平台上修复相同的代码原语并不足以确保macOS平台的安全。彻底有效的修复方案应包括：  
1. 采用macOS原生的高级IPC机制如XPC，它能更安全地处理对端认证，提供像xpc_connection_get_audit_token  
和xpc_connection_set_peer_code_signing_requirement  
这样专门防范PID重用的原生API。  
  
1. 不仅要防范符号链接攻击，还需要针对性地防御硬链接攻击。  
  
1. 清晰地分离"授权同意"和"身份验证"这两个不同维度的安全检查。  
  
构建跨平台应用本身就充满挑战，而**确保跨平台应用的安全性**  
更是难上加难。跨平台开发框架虽然给程序员带来了便利，但操作系统底层在IPC处理、文件系统操作和权限管理机制上的细微差异，却为网络安全研究员们提供了持续探索攻防技术的广阔天地。  
  
原文：https://spaceraccoon.dev/race-against-the-machine-defender-macos-lpe/  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
