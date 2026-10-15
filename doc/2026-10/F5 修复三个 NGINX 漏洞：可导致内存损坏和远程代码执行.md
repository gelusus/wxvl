#  F5 修复三个 NGINX 漏洞：可导致内存损坏和远程代码执行  
sec随谈
                    sec随谈  sec随谈   2026-10-15 01:54  
  
**摘要**  
F5 于 2026 年 10 月 14 日发布了三份安全公告，解决 NGINX Plus 和 NGINX Open Source 中的高风险 NGINX 漏洞。这三个漏洞分别为   
CVE-2026-97205  
、  
CVE-2026-93479  
 和   
CVE-2026-15176  
，每个漏洞的 CVSS v3.1 评分均为 8.1（高危）。此外，每个漏洞的 CVSS v4.0 评分均为 9.2（严重）。未经身份验证的攻击者可利用这些内存损坏漏洞使工作进程崩溃，或执行任意代码。  
  
**这些 F5 NGINX 漏洞为何重要**  
NGINX Open Source 和 NGINX Plus 为全球数以百万计的反向代理、API 网关和 Kubernetes 入口控制器提供支持。然而，这三个新披露的 NGINX 漏洞都会使数据平面暴露于远程拒绝服务（DoS）和潜在的代码执行风险之下。在每份公告中，F5 都警告称，在禁用了地址空间布局随机化（ASLR）的系统上，或攻击者能够绕过 ASLR 时，攻击者可以执行代码。目前，F5 确认这三个漏洞均没有公开的概念验证（PoC）利用代码，也没有真实环境中的利用。  
  
**攻击原理**  
  
**异步文件 I/O 和 SSI 中的释放后使用漏洞（CVE-2026-97205）**  
该漏洞由研究员 Cyberlanc3r 报告，F5 内部编号为 NPS-187。F5 针对 CVE-2026-97205 的安全公告 K000162608 详细说明了 ngx_http_core_module 模块中的一个释放后使用（Use-After-Free，CWE-416）漏洞。根据公告，在某些配置下启用异步文件 I/O（AIO）时，处理包含 include virtual 命令的服务器端包含（SSI）页面，可能会在 NGINX 工作进程中触发释放后使用（UAF）。因此，这种情况会损坏内存或使工作进程终止。  
  
**正则 map 指令中的堆缓冲区溢出（CVE-2026-93479）**  
该漏洞由 Tolmo, Inc. 的 Matt Suiche 和 Mateusz Gierblinski 报告。F5 针对 CVE-2026-93479（NPS-180）的安全公告 K000162607 说明了一个基于堆的缓冲区溢出（CWE-122）。当 map 指令使用正则表达式匹配时，就会出现该缺陷。如果 root 指令或访问日志在引用 map 输出变量之前，先引用了正则捕获变量，也会出现该缺陷。此外，在字符串表达式中使用不可缓存的变量，同样可能触发同一个堆溢出。这发生在处理特制的 HTTP 请求时。  
  
**alias 和 rewrite 指令中的堆缓冲区溢出（CVE-2026-15176）**  
该漏洞由 F5 内部发现，编号为 NPS-139。F5 针对 CVE-2026-15176 的安全公告 K000162416 描述了 ngx_http_core_module 中的另一个基于堆的缓冲区溢出（CWE-122）。具体来说，当 location 块将 alias 指令与 rewrite 结合使用，并同时有引用 $request_filename 的 set 或 return 指令时，就会触发该缺陷。此外，在访问日志 log_format 指令中引用 $request_filename，也会使工作进程暴露于堆溢出风险之下。  
  
**受影响的产品与版本**  
BIG-IP、BIG-IQ、F5OS 和 F5 Distributed Cloud Services 不受影响，而以下软件分支在这三份公告中均存在漏洞：  
- **NGINX Open Source：**  
 1.31.0 至 1.31.6 和 1.30.0 至 1.30.5（针对 CVE-2026-15176，还包括旧版本 1.0.0 至 1.30.5 和 0.1.29 至 0.9.9）。  
  
- **NGINX Plus：**  
 37.0.0 至 37.0.4 和 R33 至 R36。  
  
- **生态系统组件：**  
- NGINX Instance Manager（2.18.0 至 2.23.0）  
  
- F5 WAF for NGINX（5.9.0 至 5.15.0）  
  
- NGINX App Protect WAF（针对 CVE-2026-97205 和 CVE-2026-15176：4.12.0 至 4.16.0 和 5.4.0 至 5.8.0）  
  
- F5 DoS for NGINX（4.8.0 至 4.10.0）  
  
- NGINX App Protect DoS（针对 CVE-2026-97205 和 CVE-2026-15176：4.5.0 至 4.7.0）  
  
- NGINX Gateway Fabric（2.1.0 至 2.7.2；针对 CVE-2026-93479，还包括 1.5.0 至 1.6.2）  
  
- NGINX Ingress Controller（5.6.0 至 5.6.3；针对 CVE-2026-93479，还包括 5.0.0 至 5.6.3、4.0.0 至 4.0.1、3.7.1 至 3.7.2，以及 2026-lts-r3 至 r8）  
  
**补丁详情与缓解措施**  
管理员应通过升级到 NGINX Open Source 1.31.7 或 1.30.6 来消除这些 F5 NGINX 漏洞，也应升级到 NGINX Plus 37.1.0.1 或 37.0.5.1。以下组件也已有修复版本：NGINX Instance Manager 2.24.0、F5 WAF for NGINX 5.16.0 和 5.14.2、F5 DoS for NGINX 4.11.0、NGINX Gateway Fabric 2.8.0，以及 NGINX Ingress Controller 5.7.0。  
  
如果团队无法立即打补丁，F5 针对每个漏洞都提供了有针对性的配置变通方案：  
- **CVE-2026-97205：**  
 在配置文件中设置 aio off;  
 或 ssi off;  
。  
  
- **CVE-2026-93479：**  
 将未命名的正则捕获替换为仅限于正则匹配块内使用的命名捕获。  
  
- **CVE-2026-15176：**  
 避免在 location 块中使用 alias 指令，或移除对 $request_filename 变量的引用。  
  
参考链接：  
  
https://my.f5.com/manage/s/article/K000162608  
  
https://my.f5.com/manage/s/article/K000162607  
  
https://my.f5.com/manage/s/article/K000162416  
  
