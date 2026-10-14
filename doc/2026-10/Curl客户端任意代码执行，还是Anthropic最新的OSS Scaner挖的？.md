#  Curl客户端任意代码执行，还是Anthropic最新的OSS Scaner挖的？  
原创 玄月调查小组
                    玄月调查小组  玄月调查小组   2026-10-14 18:27  
  
昨晚，curl项目维护者Daniel Stenberg亲自撰文，披露了curl修复的一个高危漏洞——CVE-2026-92392。该漏洞并非人工审计挖出，而是 Anthropic 自研代码扫描器发现。  
## 漏洞本质：内存不安全语言的老问题  
  
这次曝出的漏洞本质是**UAF漏洞**  
，存在于 curl 和 libcurl 对 HTTP/2 协议上传功能的处理逻辑。  
  
恶意服务器可以在接收客户端 HTTP/2 上传的过程中，通过操纵 HTTP/2 流窗口与流重置机制，破坏客户端的内存管理逻辑，最终实现对上传端的内存篡改。  
  
HTTP/2 协议的多路复用特性是此次漏洞的触发前提 —— 多条传输可以复用同一条 TCP 连接，libcurl 通过名为 “easy handle” 的对象跟踪每一次传输任务。  
  
漏洞的完整利用链路可以拆解为以下步骤：   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/xKicFZ2TIFt91UvNDLiaSKqMVhRyWdiaeWOeNMZvAWNzyDY0k3bNAD9icdLDMbn5uaK1iaEbTjX1ic0g3TgABx0uq8pk98yD9YGD6pxFd4YTB1mZI/640?wx_fmt=png&from=appmsg "")  
1. **服务器暂停上传流**  
：当客户端通过 HTTP/2 上传约 135KB 的文件时，恶意服务器将 HTTP/2 流窗口设置为 0，强制暂停上传传输，同时返回部分响应内容。  
  
1. **流重置触发重试**  
：服务器在保持连接不断开的前提下重置当前流，curl 将其识别为传输中断，自动触发重试逻辑。  
  
1. **清理顺序出错留隐患**  
：在清理中断传输的过程中，curl 先通知 HTTP/2 层传输已结束，再将剩余未发送的上传数据冲刷到同一条连接上。此时连接中已无注册的传输任务，剩余数据会被解析为新的请求。  
  
1. **构造伪造请求头**  
：研究者在上传文件中嵌入符合 HTTP 请求格式的行数据，使剩余数据被识别为新的 HTTP 请求，进而创建新的流并存储指向 easy handle 的指针。  
  
1. **悬空指针触发内存篡改**  
：重试任务在新的 HTTP/1.1 连接上完成后，应用层会释放对应的 easy handle 内存，但旧的 HTTP/2 连接中仍保留着指向该内存的指针。当服务器向旧流发送帧时，程序会引用这块已被释放的内存。  
  
1. **绕过 ASLR 实现代码执行**  
：攻击者通过构造请求泄露本地内存地址，绕过ASLR防护，进而覆写释放内存中的函数指针，最终触发**代码执行**  
。  
  
## 极难利用？  
  
虽然，这是 Curl 近三年首个被评为高危的漏洞。  
  
但是，实战攻击门槛极高。很难利用。  
  
因为攻击者需要精准掌握目标客户端的 curl 版本、系统架构、libc 内存分配器特性与堆内存布局，同时控制好两条并行传输的时序，且payload仅能压缩至约 23 字节。  
## AI 立功！Anthropic OSS Scanner  
  
这个漏洞的发现者是 Anthropic 的 Luna Tong。  
  
依靠了 Anthropic 全新开源代码扫描器，底层依托 AI 模型完成挖掘。    
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/xKicFZ2TIFt8YtibPicLynuicngjh8xFz7ev8SQmVXjNkfhT13GFvRiaHmn2FhhDMbcYIqVsbibA3hEdJEIFSco0VoJDgI9cIsI91wzvZ4m3Dgl4Q/640?wx_fmt=png&from=appmsg "")  
  
几个月前，curl项目维护者还认为：围绕Mythos模型的巨大炒作到目前为止主要是营销。  
  
而目前Anthropic的OSS Scanner  
已经有大量开源项目申请。  
  
例如OpenSSL、SpiderMonkey等知名组件  
## 前沿大模型围绕cyber 能力激烈竞争  
  
近一年，国外头部大模型代码审计能力快速提升。  
  
就在一天前，OpenAI 总裁Greg Brockman还转发了XBOW的一篇推文  
  
内容是：XBOW使用 GPT-5.6 Sol 发现 Chrome Full chain RCE  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/xKicFZ2TIFtib8qZGF95msHtjfkCDPzwEzuwXJV7VibiaOVHFRrvQV3hLGJibxIPLJWIem24WPaaMQicl7Gu6TvPW4qiaQiaiaxFiaPBKlqK8ZxMuf8gM/640?wx_fmt=png&from=appmsg "")  
  
  
当然，国内的朋友们大可不必焦虑。  
字母表一头一尾：美国有 A➗，中国有 Z.AI。  
  
GLM 此前就早于 OSS Scanner  
推出了OpenVuln  
项目  
  
虽然目前Scanned repositories的影响力，和OSS Scanner  
还有点差距。  
  
但也是就在今天，智谱官宣了古河的加入  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/xKicFZ2TIFticEjz6UqyickEO9hA3XJonYuG71wkic79BGcEd0nhLKOZe17iagRSon0FgHrS5dZdSsAWfGI7Le7QIrMUxbt4QQGjLYPMJgibwCopM/640?wx_fmt=png&from=appmsg "")  
  
畅想一下，有了专业安全研究员的加持。  
  
下一代 GLM，会不会可以直接挖出 Windows 无条件 RCE？  
  
最后，0day漏洞挖掘或许才是大模型靠谱的 benchmark。  
  
已知的benchmark 都存在刷分空间，  
  
而关键软件的 0day 漏洞，基本无法造假。  
  
  
参考资料： https://daniel.haxx.se/blog/2026/10/14/a-uaf-turned-rce/ https://hackerone.com/reports/4030333  
  
