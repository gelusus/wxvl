#  Anthropic 开源漏洞扫描工具OSS Scanner，发现了HTTP/2 上传功能存在严重漏洞，可导致客户端远程代码执行！  
原创 mmc
                    mmc  AGI安全   2026-10-16 01:31  
  
# Anthropic 开源漏洞扫描工具OSS Scanner，发现了HTTP/2 上传功能存在严重漏洞，可导致客户端远程代码执行！  
  
10 月 14 日，curl 发布 8.23.0，修复了一个 HTTP/2 上传时的释放后重用漏洞 CVE-2026-92392  
。客户端向恶意服务器上传约 135KB 的文件时，服务器可以破坏客户端内存，报告者已经用 PoC 在上传方机器上执行了代码。受影响的是 curl 和 libcurl 8.12.0 至 8.22.0，命令行工具和被大量软件内嵌的库都在范围内。这是 curl 自 2023 年 10 月 SOCKS5 堆溢出以来的第一个高危漏洞，报告者署名为 Luna with Claude, Anthropic  
。  
  
上传 → 服务器卡住并重置流 → 清理顺序出错 → 句柄被释放 → 服务器触发释放后重用① 客户端上传curl / libcurl8.12.0 – 8.22.0HTTP/2 上传约 135KB文件内含伪造请求行证书须通过校验② 恶意服务器窗口设为 0 卡住上传RST_STREAM连接保持不断curl 判定需要重试HTTP_1_1_REQUIRED 等③ 清理顺序出错剩余上传数据被冲刷被当成新请求新流记下句柄指针重试走 HTTP/1.1句柄随后被释放④ 释放后重用旧连接仍持有指针服务器发帧触发泄露地址绕过 ASLR覆盖函数指针执行约 23 字节命令依据 curl 官方公告、Daniel Stenberg 博客及公开的 HackerOne 报告整理本图为攻击链示意，省略了内存布局准备等细节  
  
## 漏洞概况  
  
curl 是几乎所有系统都自带的命令行下载工具，它背后的 libcurl 被编译进了大量软件：编程语言的 HTTP 库、包管理器、CI 工具、各类桌面和嵌入式程序。据 TechTarget 报道，curl 运行在超过 200 亿台设备上。所以 curl 的高危漏洞，影响面不只是敲 curl  
 命令的人。  
<table><tbody><tr><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">项目</span></section></td><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">内容</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">编号</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(185, 28, 28);font-weight: 700;"><section><span leaf="">CVE-2026-92392</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">类型</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">释放后重用（CWE-416）</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">严重程度</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(185, 28, 28);font-weight: 700;"><section><span leaf="">High（curl 四级评级中的第二级）</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">受影响版本</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">curl / libcurl 8.12.0 至 8.22.0（2025 年 2 月起的所有版本）</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">触发条件</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">通过 HTTP/2 向攻击者控制的服务器上传约 135KB 文件，且服务器证书通过客户端校验</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">在野利用</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">暂无报告</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">修复版本</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">8.23.0；旧分支 8.14.3、8.16.2、8.20.2</span></section></td></tr></tbody></table>  
这个漏洞的方向和常见的不一样：**攻击来自服务器，受害的是客户端**  
。平时我们担心的是恶意客户端打服务器，这次是你把文件上传给一个不怀好意的服务器，对方反过来破坏你本机进程的内存。  
  
门槛也不算低。服务器的证书必须通过客户端校验，也就是说攻击者要么控制一个客户端本来就信任的站点，要么拿到了目标域名的有效证书。另外，libcurl 只有在编译时开启了 HTTP/2 支持才会受影响，但主流发行版和大多数打包版本都是开启的，访问 https:// 地址时只要服务器支持就会默认协商 HTTP/2。  
  
**为什么是 High 而不是 Critical。**  
curl 官方认为，现实中的攻击尝试"最可能的结果只是崩溃"。要真正执行代码，攻击者需要精确掌握目标的 curl 版本、CPU 架构、libc 分配器和堆布局，还要让两路并行传输的时序保持稳定，最后能塞进去的命令只有约 23 字节。curl 不发布 CVSS 分数，截至 10 月 14 日 NVD 也还没有给出评分。  
  
## 恶意服务器是怎么一步步拿下客户端的  
  
先补一个背景。HTTP/2 可以在一条连接上同时跑多个传输（流），libcurl 用一个叫 easy handle 的对象（struct Curl_easy  
）记录每个传输的状态。漏洞的本质是：一个传输已经结束、句柄已经被释放，连接上却还留着指向它的指针。  
  
1. 卡住上传，再重置流  
  
客户端开始上传后，服务器把 HTTP/2 流控窗口设为 0，让上传停下来，然后返回一部分响应，接着发送 RST_STREAM  
 重置这个流，错误码为 HTTP_1_1_REQUIRED  
 或 REFUSED_STREAM  
，但连接本身不断开。curl 收到这类重置，会认为应该重试。  
  
2. 清理顺序错了一步  
  
清理时，curl 先通知 HTTP/2 层"这个传输结束了"，然后再把还没发完的上传数据冲刷到同一条连接上。此时连接上已经没有登记的传输，这些剩余字节就被当成一个新请求来解析。报告者在上传文件里放了特定的行格式，让这段数据看起来像请求行。解析成功后，连接上开了一个新流，并且**记下了一个指向 easy handle 的指针**  
。  
  
3. 句柄被释放，指针还在  
  
重试在一条新的 HTTP/1.1 连接上完成，应用程序随后正常释放了这个句柄。但旧的 HTTP/2 连接还在，上面那个新流依然拿着已经失效的指针。只要服务器在这个流上再发一个帧，curl 就会去用这块已经释放的内存。这一步甚至可以发生在程序退出的过程中。  
  
4. 从释放后重用到代码执行  
  
在报告者的利用里，服务器发来的数据落进了原句柄占用过的内存。客户端发出的一个请求泄露了本地地址，服务器据此绕过地址空间随机化（ASLR）。随后，那块内存里的一个函数指针被服务器发来的字节覆盖，curl 接着调用了它。公开的 HackerOne 报告附带了恶意服务器脚本和一个可复现代码执行的 Docker 容器。  
  
修复由 curl 核心开发者 Stefan Eissing 完成，只是 lib/multi.c  
 里一处六行的顺序调整：先冲刷剩余的上传数据，再让 HTTP/2 层放弃对句柄的引用。curl 官方把它归为"C 语言错误"，即换成内存安全语言大概率不会犯的那一类，但项目没有用内存安全语言重写 curl 的计划。  
  
## 从发现到修复  
<table><tbody><tr><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">时间</span></section></td><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">事件</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">2025 年 2 月</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">curl 8.12.0 发布，漏洞代码随之引入</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">2026 年 9 月 15 日</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">报告经 HackerOne 提交，curl 安全团队数小时内确认</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">2026 年 10 月 7 日</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">curl 预告下个版本将修复一个高危漏洞，发布周期从 8 周缩短为 6 周</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">2026 年 10 月 8 日</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">Anthropic 公开推出 OSS Scanner</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(185, 28, 28);font-weight: 700;"><section><span leaf="">2026 年 10 月 14 日</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">curl 8.23.0 发布，公告和 HackerOne 报告（含利用脚本）同步公开</span></section></td></tr></tbody></table>  
8.23.0 一共修复了 24 个 CVE，除这一个之外其余 23 个都是 Low，这个版本还移除了 SMB 协议支持。据 TechTarget 报道，其中 18 个 CVE 来自安全公司 Aisle 用 AI 做的长期代码审计。  
  
## 漏洞是 AI 找到的  
  
curl 公告里的致谢写的是 **"Luna with Claude, Anthropic"**  
。Daniel Stenberg 在博客中说明，报告者是 Anthropic 的 Luna Tong，漏洞来自公司新推出的 OSS Scanner，他推测其中用到了 Anthropic 的 AI 模型。  
  
OSS Scanner 于 10 月 8 日公开上线，是一个需要项目主动加入的服务：用 Anthropic 的模型扫描登记的开源项目，结果不经人工审核直接邮件发给维护者，每份报告都附带复现程序，能给出的会附上候选补丁。curl 这份报告比公开上线早了三周。Anthropic 的发布文章引用了 Stenberg 的话，说扫描器发现了"近几年报告给 curl 的最严重漏洞之一"，当时没有点名是哪一个。  
  
**反差在于，Stenberg 一直是"AI 垃圾报告"最有名的批评者之一。**  
  
2025 年 7 月他写过，当年 curl 收到的安全报告里大约五分之一是他所说的"AI slop"，真正成立的只有约二十分之一；2026 年初他还因为报告质量下降暂停了 curl 的漏洞赏金计划。一年多以后，同一个项目最严重的漏洞和大部分 CVE 都来自 AI 工具。区别不在于用不用 AI，而在于报告有没有可复现的证据。这份报告附带了能跑通的利用脚本和 Docker 环境，安全团队数小时内就确认了。  
  
## 该怎么处理  
  
1. 升级到修复版本  
  
首选升级到 8.23.0，或者把修复提交合入自己的构建。还在旧分支上的，可以使用以下版本：  
<table><tbody><tr><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">当前版本</span></section></td><td style="padding: 8px 6px;border: 1px solid rgb(229, 231, 235);background: rgb(243, 244, 246);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="">升级到</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">8.12.0 – 8.22.0</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">8.23.0</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">8.14.x 分支</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">8.14.3</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">8.16.x 分支</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">8.16.2</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">8.20.x 分支</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">8.20.2</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);"><section><span leaf="">Debian 13（trixie）</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(21, 128, 61);font-weight: 700;"><section><span leaf="">8.14.1-2+deb13u7（DSA-6558-1）</span></section></td></tr><tr><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);background: rgb(240, 253, 244);"><section><span leaf="">8.12.0 之前的版本、Debian 12（bookworm）</span></section></td><td style="padding: 7px 6px;border: 1px solid rgb(229, 231, 235);color: rgb(55, 65, 81);background: rgb(240, 253, 244);"><section><span leaf="">不含漏洞代码</span></section></td></tr></tbody></table>  
2. 别只看系统里的 curl  
  
很多程序自带一份 libcurl，系统包升级管不到它们，要等各自的厂商发版。容器镜像、语言运行时、CI 工具、桌面客户端都值得排查一遍。下面几条命令可以快速确认版本和是否启用了 HTTP/2：  
```
```  
  
3. 暂时不能升级：关掉 HTTP/2  
  
curl 官方给的临时办法是不用 HTTP/2，强制 HTTP/1.1 就不会走到出问题的代码路径：  
```
```  
  
要注意，curl 文档把这个选项定义为"请求"：如果连接池里已有一条其他协议版本的连接，仍可能被复用。在应用里设置时，最好让所有传输都统一配置。另外，编译时不带 HTTP/2 支持的 libcurl 本身就无法协商 HTTP/2，不受此漏洞影响。  
  
4. 重点关注"向外部上传"的场景  
  
这个漏洞只在上传时触发。优先排查会把文件上传到第三方或用户指定地址的服务：Webhook 回调、文件同步、日志和备份上传、制品发布脚本，以及允许用户填写目标 URL 的功能。上传目标越不受控，风险越高。  
  
## 写在最后  
  
一个六行的顺序问题，在 curl 里存在了一年半，跨过了十多个版本。Stenberg 说，curl 这几年一直在减少内部指向句柄的指针，改成需要时按标识符查找，但这并不能保证以后不再出现释放后重用。留几个问题供讨论：  
  
第一，AI 挖洞的成本在快速下降，curl 一个版本就修了 24 个 CVE。维护者的修复能力跟得上吗？下游厂商的补丁节奏跟得上吗？  
  
  
第二，大多数企业的漏洞管理只盯着服务端。像这样"服务器打客户端"的漏洞，你们的资产清单里有内嵌 libcurl 的客户端程序吗？  
  
  
第三，利用脚本已经和公告同步公开。从"PoC 公开"到"有人拿来用"，留给企业的窗口期有多长？  
后续我们会继续分享开源组件安全、AI 漏洞挖掘与企业安全体系建设的实践经验，欢迎收藏转发！  
  
AGI 安全 · 企业 AI 安全专项培训  
  
针对开源组件供应链安全、AI 漏洞挖掘、智能体攻防与企业安全体系建设  
  
我们提供完整的企业 AI 安全培训体系，助力企业建立完整的安全防护能力  
<table><tbody><tr><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.14);border-radius: 10px;padding: 10px 8px;text-align: center;border: 1px solid rgba(153, 246, 228, 0.45);"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(153, 246, 228);line-height: 1.5;"><span leaf="">大模型安全</span></p></td><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.14);border-radius: 10px;padding: 10px 8px;text-align: center;border: 1px solid rgba(153, 246, 228, 0.45);"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(153, 246, 228);line-height: 1.5;"><span leaf="">智能体安全</span></p></td></tr><tr><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.1);border-radius: 10px;padding: 10px 8px;text-align: center;"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(255, 255, 255);line-height: 1.5;"><span leaf="">AI 合规审计</span></p></td><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.1);border-radius: 10px;padding: 10px 8px;text-align: center;"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(255, 255, 255);line-height: 1.5;"><span leaf="">组件供应链安全</span></p></td></tr><tr><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.14);border-radius: 10px;padding: 10px 8px;text-align: center;border: 1px solid rgba(153, 246, 228, 0.45);"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(153, 246, 228);line-height: 1.5;"><span leaf="">数据安全</span></p></td><td data-colwidth="50%" width="50%" style="background: rgba(255, 255, 255, 0.1);border-radius: 10px;padding: 10px 8px;text-align: center;"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(255, 255, 255);line-height: 1.5;"><span leaf="">AI 编程工具安全</span></p></td></tr><tr><td colspan="2" style="background: rgba(255, 255, 255, 0.1);border-radius: 10px;padding: 10px 8px;text-align: center;"><p style="margin: 0px;font-size: 13px;font-weight: 700;color: rgb(255, 255, 255);line-height: 1.5;"><span leaf="">企业安全体系搭建</span></p></td></tr></tbody></table>  
如需定制企业内部安全培训、对外系统与 AI 工具使用风险评估，欢迎扫码咨询  
  
![](https://mmecoa.qpic.cn/mmecoa_png/3o01zBiam5M2Pt2hdYUBUCxbOkmfZ5UxlSEDKxiboiagicNKshq47pUZbqZFFc4CZrvn2cxxO6WUZ5KltQKVHnAqxwMnicgWicPRoWg3mVNSgYKxE/640?wx_fmt=png&from=appmsg "")  
  
  
联系人：马老师 ｜ 微信：AICodingC ｜ 公众号：AGI安全  
  
## 参考资料  
  
curl 官方公告：CVE-2026-92392 HTTP/2 upload Use-After-Free ｜ HackerOne #4030333  
  
The Hacker News：curl Patches HTTP/2 Upload Flaw After PoC Demonstrates Client-Side Code Execution（2026 年 10 月 14 日）  
  
oss-sec：[SECURITY ADVISORIES] curl 8.23.0 ｜ curl 安全公告列表  
  
TechTarget：Historic Curl flaw shows AI risk to legacy supply chain  
  
heise：New curl version removes SMB support and fixes 24 security vulnerabilities ｜ JFrog：CVE-2026-92392 curl HTTP/2 upload RCE  
  
Daily Security Review：curl 8.23.0 Fixes High-Severity HTTP/2 Flaw CVE-2026-92392  
  
