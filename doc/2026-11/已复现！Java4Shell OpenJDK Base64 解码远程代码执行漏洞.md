#  已复现！Java4Shell OpenJDK Base64 解码远程代码执行漏洞  
微步情报局
                    微步情报局  微步在线研究响应中心   2026-11-18 01:01  
  
漏洞概况  
  
  
OpenJDK 是 Java 平台标准版的开源参考实现，也是当今使用最广泛的 Java 开发工具包。  
  
近日，Oracle官方发布通告，修复了OpenJDK Base64解码远程代码执行漏洞（  
CVE-2026-114938  
）。  
微步情报局已成功复现该漏洞。  
经分析，  
漏洞影响JDK17及以上版本，  
漏洞位于 OpenJDK HotSpot 针对 x86-64 处理器 AVX-512 VBMI 指令集优化的 Base64 解码汇编代码中。当处理包含特定尾部填充字符输入时，输出长度计算会发生下溢，导致 evmovdqub 指令的写入掩码变为全 1，从而将 64 字节数据写入一个仅 50 字节的缓冲区，造成约 62 字节的堆越界写入，满足特定利用条件时，可造成远程代码执行。  
  
（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-99068）  
  
攻击者成功利用此漏洞可  
在受影  
响  
服务器上执行任意代码，亦可导致子进程崩溃、业务中断。  
建议受影响用户  
尽快修复。  
  
漏洞处置优先级(VPT)  
  
  
**综合处置优先级：高**  
风险  
<table><tbody><tr><td rowspan="3" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">基本信息</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;"><section><span leaf="">微步编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">XVE-2026-</span><span leaf="" data-pm-slice="1 1 [&#34;para&#34;,{&#34;tagName&#34;:&#34;section&#34;,&#34;attributes&#34;:{&#34;style&#34;:&#34;color:#0f172a;font-size:15px;font-family:-apple-system, BlinkMacFont, \&#34;Segoe UI\&#34;, \&#34;PingFang SC\&#34;, sans-serif;&#34;,&#34;data-pm-slice&#34;:&#34;0 0 []&#34;},&#34;namespaceURI&#34;:&#34;http://www.w3.org/1999/xhtml&#34;,&#34;data-layout-id&#34;:null},&#34;para&#34;,{&#34;tagName&#34;:&#34;p&#34;,&#34;attributes&#34;:{&#34;style&#34;:&#34;margin:0 0 14px;margin-left:0;padding:0;line-height:1.6em;&#34;},&#34;namespaceURI&#34;:&#34;http://www.w3.org/1999/xhtml&#34;,&#34;data-layout-id&#34;:null}]">99068</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE-2026-</span><span leaf="" data-pm-slice="1 1 [&#34;para&#34;,{&#34;tagName&#34;:&#34;section&#34;,&#34;attributes&#34;:{&#34;style&#34;:&#34;color:#0f172a;font-size:15px;font-family:-apple-system, BlinkMacFont, \&#34;Segoe UI\&#34;, \&#34;PingFang SC\&#34;, sans-serif;&#34;,&#34;data-pm-slice&#34;:&#34;0 0 []&#34;},&#34;namespaceURI&#34;:&#34;http://www.w3.org/1999/xhtml&#34;,&#34;data-layout-id&#34;:null},&#34;para&#34;,{&#34;tagName&#34;:&#34;p&#34;,&#34;attributes&#34;:{&#34;style&#34;:&#34;margin:0 0 14px;margin-left:0;padding:0;line-height:1.6em;&#34;},&#34;namespaceURI&#34;:&#34;http://www.w3.org/1999/xhtml&#34;,&#34;data-layout-id&#34;:null}]">114938</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程代码执行</span></section></td></tr><tr><td rowspan="5" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用条件评估</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的网络条件</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要绕过安全机制</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">要</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">对被攻击系统的要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><p><span leaf="">受影响硬件是支持 AVX-512F、AVX-512BW、AVX-512 VBMI 和 BMI2 指令集的 x86-64 处理器。</span></p></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的权限要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无须用户权限</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要受害者配合</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td rowspan="2" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用情报</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否有POC</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><span style="color:#d93025;font-weight:bold;"><span leaf="">是</span></span></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">已知利用行为</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">暂无</span></section></td></tr></tbody></table>  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">产品名称</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">OpenJDK</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><p><span leaf="">Oracle Java SE: JDK&lt;=8u511b33-perf(仅 perf 企业性能包版本；JDK 8 标准线不受此漏洞影响)、17.0.6&lt;=JDK&lt;=17.0.21.0.3、21&lt;=JDK&lt;=21.0.13.0.3、25&lt;=JDK&lt;=25.0.5.0.3、27&lt;=JDK&lt;=27.0.1.0.3</span></p><p><span leaf=""><br/></span></p><p><span leaf="">Oracle GraalVM for JDK: 17.0.6&lt;=JDK&lt;= 17.0.21.0.3、21&lt;=JDK&lt;= 21.0.13.0.3</span></p><p><span leaf=""><br/></span></p><p><span leaf="">Oracle GraalVM Enterprise Edition: JDK&lt;=21.3.20.0.3</span></p><section><span leaf=""><br/></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
漏洞复现  
  
  
目标为已运行的 OpenJDK 17.0.6 且  
硬件是支持 AVX-512F、AVX-512BW、AVX-512 VBMI 和 BMI2 指令集的 x86-64 处理器。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEMrVNx38hewJww2CUicauO33tILLmiamJmmKlOBvD7LYwBHJHHLHomg4okTFDLUypVDfGfSHFm0ib8owGmtpOJuAk3HkZO5psY98A/640?wx_fmt=png&from=appmsg "")  
  
修复方案  
  
### 官方修复方案  
  
官方已发布修复方案，请访问链接下载：  
  
https://jdk.java.net/27/  
  
  
微步产品支撑  
  
  
微步漏洞情报于  
2026-11-18  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
