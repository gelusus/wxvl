#  Notepad++发布8.9.9版本，修复6个可触发插件命令执行的漏洞  
 FreeBuf   2026-11-10 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3xTqgoicOjKNbraMq8T2ktgSoswyurhY59HNx7HWtNnW2NnlRYoVbCQ1kJNCd5zLJPuNX8cXLetmBqMBuM80bRLkCrB1EG7AFA/640?wx_fmt=gif "")  
  
  
![Notepad++ 8.9.9发布](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0L1avDykOuTQwaZfa2HIYsH9RzbEecptMsXOkHDyXAZXpB8KYbr1ocicbKojIia1ZdCDW1E37jDdYm5VSoFUIzLWBEOvyRzZHnQ/640?wx_fmt=png "")  
  
  
Part  
01  
  
Notepad++消息处理缺陷  
  
可导致插件命令执行  
  
  
Notepad++于2026年11月6日发布8.9.9版本，修复了Windows消息处理、受保护文件保存、剪贴板操作、MIME Tools解码四大场景下的6个安全漏洞。  
  
  
本次更新修复的问题包括条件性插件命令执行、内存信息泄露、缓冲区溢出三类风险。根据官方发布说明，这些漏洞暂未分配对应的CVE编号。  
  
  
本次修复的命令执行漏洞涉及WM_COPYDATA消息，这是Windows系统提供的跨应用数据交换机制。在8.9.8版本中，程序仅校验调用方提供的窗口句柄是否可信，并未验证消息的实际发送进程。本地攻击者可指定一个独立的合法Notepad++窗口，绕过上述校验逻辑。  
  
  
攻击者可借此向提权运行的编辑器实例传入插件命令。不过已公开的攻击验证场景需要满足多个前提：一是目标设备已安装并初始化可选的NppExec插件，二是目标程序已处于提权运行状态，三是与目标实例同可执行文件路径下，还运行着第二个Notepad++进程。该漏洞属于特定条件下的本地提权问题，并非可通过网络利用的远程代码执行漏洞。  
  
  
发布说明将WM_COPYDATA发送方身份认证绕过、插件命令执行两类漏洞单独列出。这两类问题都说明，跨权限层级直接信任调用方可控的消息数据，存在极大安全隐患。  
  
  
本次更新前，Notepad++也曾修复过涉及配置文件的代码执行漏洞，但那些漏洞的成因与本次问题完全不同。  
  
  
另一类漏洞存在于需用户账户控制（UAC）审批的文件保存流程中。原流程下，Notepad++会先将文档内容写入临时文件，关闭文件句柄后，再把临时文件路径传递给提权运行的助手程序。  
  
  
与当前用户同权限运行的恶意进程，可在助手程序将临时文件内容复制到用户选定的受保护文件前，替换临时文件内的内容。整个攻击过程仍需要用户手动确认UAC弹窗才能完成。  
  
  
Part  
02  
  
剪贴板与菜单处理缺陷  
  
引发内存安全风险  
  
  
堆信息泄露漏洞出现在"粘贴二进制"功能中，问题根源是程序未校验剪贴板数据长度。本地进程可向程序传入体积极小的文本缓冲区，同时声明一个远大于实际体积的尺寸值。这种情况下，Notepad++会把进程相邻内存区域的数据复制到文档中。测试显示，利用该漏洞可读取缓冲区之外的堆数据。  
  
  
NPPM_MENUCOMMAND漏洞的成因是程序选取存储命令时，未对索引值做校验。测试过程中，越界数据可传入ShellExecute接口，造成程序界面卡死。但目前研究人员尚未验证，攻击者能否控制最终执行的命令。  
  
  
Part  
03  
  
MIME Tools插件存在缓冲区溢出  
  
  
此外，3.1及以下版本的MIME Tools插件存在SAML解码缓冲区溢出漏洞。攻击者可构造专门的恶意输入，用户触发解码操作时，轻则导致编辑器崩溃，重则可能触发任意代码执行。目前MIME Tools 3.2版本已修复该问题。  
  
  
此前Notepad++曝出的插件滥用漏洞主要通过替换插件文件实现，与本次修复的漏洞攻击路径完全不同。建议用户通过官方下载页面安装8.9.9版本，同时确认MIME Tools已更新至最新版本。  
  
  
除安全修复外，本次版本还带来了多项功能优化与问题修复。程序优化了"在文件中查找"功能的性能，更新了内置的Scintilla与Lexilla组件。新版本还新增对话框字体控制功能，同时修复了剪贴板历史报错、内存泄露等非安全问题。  
  
  
参考来源：  
  
Notepad++ 8.9.9 Released With Fixes for 6 Vulnerabilities Allowing Plugin Command Execution  
  
https://cybersecuritynews.com/notepad-8-9-9-released/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
  
