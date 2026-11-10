#  Notepad++ 发布 8.9.9 安全更新：修复 6 项漏洞，含本地插件命令执行缺陷  
 网安百色   2026-11-10 10:49  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WibvcdjxgJnvNLq5PvVEY5AvbgicOEAiazOxib5tW3sgkrkcBemWQibuE1NE3ou0oBqt0q9N2mnvBMMEhGKBYTNK4EP3z7fkaJ5THt5ibTggSgsnA/640?wx_fmt=png&from=appmsg "")  
  
Notepad++ 于 2026 年 11 月 6 日发布了 8.9.9 版本，修复了涉及 Windows 消息处理、受保护文件保存、剪贴板操作以及 MIME 工具解码等领域的六个安全漏洞。  
  
此次更新修复了包括**特定条件下的插件命令执行**  
、**内存信息泄露**  
以及缓冲区溢出在内的多个缺陷。官方发布说明中未为这些问题分配 CVE 编号。  
  
该命令执行问题涉及 WM_COPYDATA  
，这是一种用于在 Windows 应用程序之间交换数据的消息机制。在 Notepad++ 8.9.8 中，**消息接收方**  
仅检查调用者提供的窗口句柄是否受信任，而未验证实际发送该消息的进程身份。本地攻击者可以指定另一个合法的 Notepad++ 窗口来绕过这些检查。  
  
这可能会让攻击者将插件命令注入到高权限的编辑器中。演示的攻击需要满足严格的先决条件：已安装并初始化了可选的 NppExec 插件、目标进程已处于提权状态，且存在另一个从相同可执行文件路径运行的 Notepad++ 实例。因此，这属于特定条件下的**本地提权（Local Privilege Escalation）**  
 漏洞，而非基于网络的远程代码执行（RCE）。  
  
发布说明中分别列出了 WM_COPYDATA  
 发送方身份验证绕过和插件命令执行这两个漏洞。两者均凸显了在不同权限级别之间，盲目信任由调用者控制的消息数据所带来的危险。  
  
此次更新紧随此前 Notepad++ 修复的涉及配置文件的代码执行漏洞，尽管那些早期漏洞的成因与此不同。  
  
另一个漏洞影响需要用户账户控制 (UAC) 批准的文件保存操作。Notepad++ 会先将文档内容写入临时文件，随后关闭该文件，并将其路径传递给高权限的辅助进程（elevated helper）。  
  
以同一用户身份运行的恶意进程，可以在辅助进程将内容复制到用户选择的受保护文件之前，替换该临时文件的内容。不过，该攻击仍需用户点击 UAC 批准弹窗。  
### 内存缺陷与插件风险  
  
关于**堆内存信息泄露（Heap Disclosure）**  
 的公告指出，“二进制粘贴”功能中存在未校验剪贴板长度的问题。本地进程可以提供一个较小的文本缓冲区，同时声明一个较大的尺寸，从而导致 Notepad++ 将相邻的进程内存复制到文档中。测试证实，该操作确实能提取出超出所提供缓冲区范围的堆数据。  
  
NPPM_MENUCOMMAND  
 漏洞在选择存储的命令时使用了未校验的索引。测试表明，越界数据会传递至 ShellExecute  
 并导致界面阻塞，但并未演示攻击者能够实际控制最终执行的命令。  
  
3.1 及更早版本的 MIME Tools 插件存在 SAML 解码缓冲区溢出漏洞。精心构造的输入在解码时可能导致编辑器崩溃，甚至可能执行任意代码。MIME Tools 3.2 版本已修复此问题。  
  
此前 Notepad++ 的插件滥用问题涉及替换插件文件，这与本次新修复的漏洞属于完全不同的攻击路径。用户应通过官方下载页面安装 8.9.9 版本，并确认 MIME Tools 插件已同步更新。  
  
该版本还提升了“在文件中查找”的性能，更新了 Scintilla 和 Lexilla 核心组件，增加了对话框字体控制功能，并修复了剪贴板历史记录错误及**内存泄漏（memory leaks）**  
 问题。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
