#  Brotli 漏洞暴露严重，HTTP2Shell 漏洞触发紧急补丁  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-27 16:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8J4qQXKHtDf9hN7X9H5EFnhABfMQ6m706YyPU7dsgolVrZVFmDSPqaVYDdpubFFdm1RQaDuN6BuKSG3pSAib0x2a94iaZJm2icHYo/640?wx_fmt=png&from=appmsg "")  
  
内置于 Node.js 中的 C 解码器库 Brotli 存在一个名为“HTTP2Shell”的严重漏洞，攻击者可以利用该  
漏洞  
破坏内存并访问敏感数据。  
  
该安全问题编号为CVE-2026-111497，也称为“Broccoli2Shell”和“Node2Shell”。它影响 Brotli 版本 1.0.3 至 1.0.9、1.1.0 和 1.2.0。  
  
该漏洞曾以多个名称被报道，因为它最初是在 Node.js 中发现的，后来研究人员发现 Brotli 解码器中的相同  
漏洞  
可能会影响多个生态系统中的软件。  
  
Brotli 是由 Google 开发和维护的开源项目，用于无损数据压缩。它可以缩小文件大小，从而加快传输速度，同时又不影响原始数据。广泛影响  
  
HTTP2Shell 这个名称反映了它对 HTTP 客户端和服务器的影响，尽管该漏洞并非 HTTP/2 特有，它还会影响其他解压缩攻击者控制的 Brotli 数据的应用程序。  
  
除了 Node.js 之外，可能受影响的软件还包括 Python HTTP 客户端，例如urllib3[brotli]、requests、httpx和aiohttp[speedups]，当它们配置了易受攻击的 Brotli 解码器时，以及其他使用易受攻击的 Brotli 实现的应用程序和工具。  
  
周日，微软推送了一个紧急安全更新来解决 .NET 中的问题，并警告称，成功利用该漏洞可以在具有易受攻击的 .NET 版本的系统上提升权限。  
  
Node2Shell 漏洞是一种缓冲区溢出漏洞，它允许一个特制的 Brotli 压缩流触发一个无效的块切换，解码器会将其视为有效切换，并错误地解释为请求更多输入。  
  
通过不断向内部 8 字节缓冲区追加输入字节，写入操作会超出缓冲区的边界，从而损坏附近的内存。  
  
Node.js在一份公告中解释说：“无效的块切换被报告为暂时需要更多输入，而不是永久性的格式错误，因此流解码器不断地将输入字节附加到 8 字节的内部缓冲区。”  
  
攻击者可以利用此漏洞，让应用程序解压缩恶意数据，例如向获取 URL 的应用程序提供压缩的 HTTP 响应，或者直接向服务器发送压缩的请求。  
  
Node.js 指出，这可能会导致进程崩溃、内存泄漏，并启用远程代码执行 (RCE)。  
  
网络安全公司Imperva 最近的一份报告证实，利用 CVE-2026-111497 Node2Shell 漏洞可能导致远程代码执行 (RCE)。  
  
Brotli 上游的修复程序最早于 2 月份提交到项目源代码中，但 Node.js 和 Google 于 10 月 21 日发布了面向用户的稳定修复程序，并在第二天向公众公开披露。包含这些修复程序的 PyPI 包于 10 月 23 日发布。  
  
已打补丁的版本如下：  
- Google Brotli 1.0.10、1.1.1、1.2.1  
  
- Node.js 22.23.4、24.22.1、26.12.1  
  
- Python Brotli 1.2.1  
  
- Python brotlicffi 1.2.1.0  
  
  
Node2Shell 的概念验证 (PoC) 漏洞利用程序已经发布，但目前尚未有恶意利用该漏洞的报告。尽管如此，用户仍应优先安装可用的安全更新。  
  
更新后，管理员必须重启受影响的进程，并且必须重建并重新部署受影响的容器镜像。  
  
一个专门跟踪该  
漏洞  
影响的在线门户网站 也警告说，更新系统 Brotli 库并不能解决 Node.js 部署或其他使用该库的捆绑副本的应用程序中的风险。  
  
此外，即使阻止了入站 Brotli 请求，恶意 HTTP 响应路径仍然暴露在外。  
  
如果无法立即更新，建议从 Python 应用程序中删除 Brotli 支持，并在其代理或防火墙级别阻止传入的 Brotli 压缩请求。  
  
  
信息来源：BleepingComputer  
  
