#  已复现！Apache HTTPD mod_http2 条件竞争远程代码执行漏洞  
微步情报局
                    微步情报局  微步在线研究响应中心   2026-11-13 13:55  
  
漏洞概况  
  
  
Apache HTTP Server 是 Apache 软件基金会开发的开源 Web 服务器，是全球部署最广泛的 HTTP 服务端之一。  
  
近日，Apache HTTP Server官方发布通告，修复了Apache HTTP Server mod_http2 assign_headers() 竞争条件漏洞（CVE-2026-119618）。  
微步情报局已成功复现该漏洞。  
经分析，Apache HTTP Server mod_http2 的 assign_headers() 函数存在并发同步缺陷，多个线程并发访问共享资源时缺少适当同步，特定 HTTP/2 请求序列可触发竞态条件造成内存破坏，满足特定利用条件时，可造成远程代码执行。  
  
（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-96011）  
  
攻击者成功利用此漏洞可  
在受影  
响  
服务器上执行任意代码，亦可导致子进程崩溃、业务中断。  
建议受影响用户  
尽快修复。  
  
漏洞处置优先级(VPT)  
  
  
**综合处置优先级：**  
中风险  
<table><tbody><tr><td rowspan="3" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">基本信息</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;"><section><span leaf="">微步编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">XVE-2026-96011</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE-2026-119618</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程代码执行（条件竞争）</span></section></td></tr><tr><td rowspan="5" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用条件评估</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的网络条件</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要绕过安全机制</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">对被攻击系统的要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">需加载 mod_http2 模块且 Protocols 配置包含 h2/h2c（仅提供 HTTP/1.1 的服务器不受影响）</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的权限要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无须用户权限</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要受害者配合</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td rowspan="2" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用情报</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否有POC</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><span style="color:#d93025;font-weight:bold;"><span leaf="">是</span></span></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">已知利用行为</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">暂无</span></section></td></tr></tbody></table>  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">产品名称</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">Apache HTTP Server</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">2.4.55&lt;=version&lt;=2.4.70</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
漏洞复现  
  
  
目标为已运行的 Apache HTTP Server 2.4.70 且启用了 HTTP/2，发送特制的 HTTP/2 请求，  
竞态触发后 prefork 子进程在 apr_allocator_free 处 SIGSEGV  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdENyAicoerh1w86hhDyNicHrOBIOKlw5OpSctl0uVOr5mlzSm7AF3y3UrYPIrLM6gpgH7uajDLu4ic7KWreXHGZyYykgP5qIqiaDCNI/640?wx_fmt=png&from=appmsg "")  
  
修复方案  
  
### 官方修复方案  
  
官方已发布修复方案，请访问链接下载：  
  
https://httpd.apache.org/download.cgi  
### 临时缓解措施  
  
1、在反向代理、负载均衡或 WAF 上对不可信来源禁用 HTTP/2（移除 h2/h2c，仅保留 http/1.1），或对短时间大量创建 HTTP/2 流、请求头数量接近上限的客户端限速/阻断  
  
 2、禁用 HTTP/2 协议：在 Apache 配置中将协议限制为仅使用 HTTP/1.1，避免触发 `mod_http2` / `mod_h2` 漏洞路径。  
  
 3、以低权限账户运行 httpd 子进程，限制 Web 根目录、配置文件和日志目录的写权限。  
  
微步产品支撑  
  
  
微步漏洞情报于  
2026-11-11  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
