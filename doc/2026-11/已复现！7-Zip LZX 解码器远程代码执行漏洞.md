#  已复现！7-Zip LZX 解码器远程代码执行漏洞  
微步情报局
                    微步情报局  微步在线研究响应中心   2026-11-12 09:47  
  
漏洞概况  
  
  
7-Zip 是一款广受欢迎的开源文件压缩/解压缩工具，由 Igor Pavlov 开发维护，支持 7z、ZIP、CAB 等多种归档格式，广泛运行于 Windows、Linux 等多种操作系统。  
  
近日，7-zip官方发布通告，修复了7-Zip LZX 解码器远程代码执行漏洞（CVE-2026-125742）。  
微步情报局已成功复现该漏洞。  
经分析，  
该漏洞存在于多卷 CAB 归档的续卷解码流程中。当 LZX 解码器跨卷续接历史时，代码会根据新卷声明的字典位数重新分配解压窗口，并更新 _winSize；但位置计数器 _pos 未随之重置，仍保留上一卷大窗口下的值。由于边界检查采用无符号减法（outSize > _winSize - _pos），当 _pos 大于缩小后的 _winSize 时，该表达式会下溢为一个极大值，导致正负判断结果的边界检查被绕过，造成堆越界写。  
  
（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-93913）  
  
攻击者可构造特制的多卷 CAB 归档，诱导用户使用受影响版本的 7-Zip 解压，通过精确控制越界写入内容覆盖堆上的对象虚表，进而劫持控制流实现任意代码执行。  
建议受影响用户  
尽快修复。  
  
漏洞处置优先级(VPT)  
  
  
**综合处置优先级：**  
中风险  
<table><tbody><tr><td rowspan="3" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">基本信息</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;"><section><span leaf="">微步编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">XVE-2026-93913</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE-2026-125742</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程代码执行</span></section></td></tr><tr><td rowspan="5" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用条件评估</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的网络条件</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">本地</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要绕过安全机制</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">对被攻击系统的要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无特殊要求</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的权限要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无须用户权限</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要受害者配合</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是</span></section></td></tr><tr><td rowspan="2" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用情报</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否有POC</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><span style="color:#d93025;font-weight:bold;"><span leaf="">是</span></span></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">已知利用行为</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">暂无</span></section></td></tr></tbody></table>  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">产品名称</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">7-zip</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">0&lt;=version&lt;26.06</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
漏洞复现  
  
  
在已运行官方 7-Zip 26.05 的 Linux arm64 目标上执行 PoC，脚本针对 libc 基址 byte2 进行有限次重试，命中后进入多卷 CAB 触发流程，实现任意代码执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEMlBBnhZpZlpVnyJSD8PMlB0OtlqicB2ZGiarVHqBEjEq9YoMtYg1vD8CkLr1Im0d28fT4ltyKnziaZknyGNACXia0b8k8v6MjpElg/640?wx_fmt=png&from=appmsg "")  
  
  
修复方案  
  
### 官方修复方案  
  
官方已发布修复方案，请访问链接下载：  
  
https://github.com/ip7z/7zip/releases/tag/26.06  
  
### 临时缓解措施  
  
升级前，避免使用受影响版本的 7-Zip 解压不可信的多卷 CAB 归档。必要时仅在受控隔离环境中解压。  
  
微步产品支撑  
  
  
微步漏洞情报于  
2026-11-10  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
