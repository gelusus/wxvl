#  ModSecurity WAF漏洞允许攻击者使用精心构造的HTTP请求绕过文件上传限制  
原创 ZM
                    ZM  暗镜   2026-11-15 23:00  
  
开源 Web 应用防火墙 ModSecurity (WAF) 中存在两个高危漏洞，攻击者可利用精心构造的 HTTP 请求绕过上传过滤器或导致服务器工作进程崩溃。维护人员于 2026 年 9 月 30 日发布了这两项安全公告，作为涵盖ModSecurity 多个可绕过 WAF 漏洞的一系列安全公告的一部分。  
  
第一个漏洞GHSA-5pww-8rfg-9crf的 CVE 编号为 CVE-2026-104269，评分为 8.6（高危）。该漏洞影响 ModSecurity 3.0.0 及更高版本，但公告中未列出任何已修复的版本。  
  
问题在于解析器不匹配。ModSecurity 的多部分解析器没有实现 RFC 2231，该规范定义了 Content-Disposition 标头中的扩展 filename* 参数。因此，OWASP CRS 规则（例如 920120、922100 和 922110）仅检查纯 filename 值。  
  
许多符合 RFC 规范的后端在同时存在 `filename` 和 `multipart` 参数时，更倾向于使用 `filename*` 格式。该规范确认了 Go（net/http）、Python（Flask/Werkzeug 和 FastAPI/Starlette）、Node.js（busboy 和 @fastify/multipart）以及 Java（Spring）的这一特性。PHP 的原生 multipart 处理仅读取 `filename=`，因此不受影响。  
  
攻击者可以发送 filename=”safe.jpg”; filename*=UTF-8”shell.php。CRS 会将 .jpg 文件识别为无害文件并允许上传，而后端则会解码 shell.php 文件并将其写入磁盘。  
  
如果该文件被放置在可通过网络访问的目录中，则会导致远程代码执行。CVSS 向量为 AV:N/AC:L/PR:N/UI:N/S:C/C:N/I:H/A:N，这意味着无需身份验证或用户交互。研究人员 0xkalawy 和 ZeyadZonkorany 发现了该漏洞，B0YK4 协调了漏洞披露工作。这是WAFFLED 研究中发现的解析器差异问题的另一个例子，其中 WAF 和 Web 框架对同一请求的解释方式不同。  
  
第二个漏洞GHSA-jx3r-phvx-2jmj的 CVE 编号为 CVE-2026-73857，评分为 7.5（高危）。该问题涉及 ModSecurity v3 XML 请求体处理器中未初始化的指针解引用，该漏洞也属于 CWE-787 和 CWE-824 的范畴。  
  
Tobias Klein 通过电子邮件报告了此漏洞。攻击者只需一次请求，即可操纵工作进程，向其控制的内存地址写入一个常量值，从而导致受限的写入操作。潜在影响包括拒绝服务攻击、内存泄露，甚至可能执行代码。此漏洞仅在 SecParseXmlIntoArgs 指令设置为“On”或“OnlyArgs”时才会触发；默认情况下该指令处于关闭状态。受影响的版本为 3.0.15、3.0.16 以及当前的 v3/master 分支。3.0.x 的早期版本和 ModSecurity v2.x 不受影响。该安全公告中列出的修复版本为 3.0.17。  
  
