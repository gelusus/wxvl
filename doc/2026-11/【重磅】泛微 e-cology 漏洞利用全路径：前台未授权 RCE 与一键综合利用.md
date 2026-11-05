#  【重磅】泛微 e-cology 漏洞利用全路径：前台未授权 RCE 与一键综合利用  
原创 CabinQ
                        CabinQ  逆熵寻生   2026-11-05 08:07  
  
 泛微 e-cology 漏洞利用全路径：前台未授权 RCE 与一键综合利用  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Uhia8L5ia7ha7dzOjz00NVm9iagQV1iaxLy0nPqLic8eFmJeBHcU1THmoYwlAnQgicIZz3WiaRRibZcCcsAE0WSSZXtTNibo0yK5rycMB0yy4r7pcMZM/640?from=appmsg "")  
  
  
【深度长文】全文约 10000 字 | 覆盖 11 个 CVE 与 3 大经典攻击面 | 含一键综合利用工具 | 含完整工具资源包  
  
核心结论  
  
泛微 e-cology 是国内部署量最大的 Java 企业协同办公平台（OA）之一，从政府、央企到银行、券商、制造业，只要有内部审批、公文流转、门户集成的需求，几乎都能看到它的身影。正因为覆盖面足够广，e-cology 也随之成为攻击者眼中的"高价值、低门槛"目标：一方面大量历史实例停留在旧版本、旧补丁上；另一方面 e-cology 历史上暴露过大量**前台未授权**  
（无需登录）即可触达的漏洞，从 SQL 注入、XXE、源码泄露，到 XMLDecoder/Beanshell 反序列化、Dubbo 调试端点 RCE，一条完整的"信息收集 → 前台打点 → 一键 RCE → 内存马/冰蝎持久化"攻击链早已被完整跑通并举一反三。  
  
本文以纯攻击视角，系统梳理泛微 e-cology 从 2019 年到 2026 年最值得掌握的 11 个 CVE 与 3 大无标准 CVE 编号的经典攻击面，每条均经过多源权威核验（NVD、CVE.org、阿里云 AVD、奇安信、VulnCheck、启明星辰、Shadowserver、Vega 等），并给出可直接复现的 PoC 请求。重点落在 2026 年 3 月大规模野外利用的 **CVE-2026-22679**  
（Dubbo 调试端点未授权 RCE），以及一键综合利用工具 **Apt_t00ls**  
——它把 BshServlet、WorkflowServiceXml（写冰蝎内存马）、KtreeUploadAction、workrelate/page_uploadOperation 等多项泛微 RCE 做成了图形化一键利用。读者读完本文，即可掌握一条从资产确认到拿下 e-cology 服务器权限的完整攻击路径。  
  
CVE 速览  
  
🔹 **CVE-2026-22679**  
：/papi/esearch/data/devops/dubboApi/debug/method  
 调试端点未授权直达 Dubbo RPC，任意命令执行，CVSS 3.1=9.8，e-cology 10.0 < 20260312，已野外利用  
  
🔹 **CVE-2025-34038**  
：/js/hrm/getdata.jsp  
 前台 SQL 注入（getSelectAllId），拖管理员密码哈希，CVSS 4.0=8.7  
  
🔹 **CVE-2024-48069**  
：条件竞争绕过文件上传限制（CWE-362），e-cology 9.0，CVSS 9.8  
  
🔹 **CVE-2024-48070**  
：构造特殊请求插入恶意代码 RCE（CWE-94），e-cology 9.0，CVSS 9.8  
  
🔹 **CVE-2024-48071**  
：/importmould/deletefolder  
 目录遍历任意文件删除（CWE-22），CVSS 6.5，需认证  
  
🔹 **CVE-2024-48072**  
：/mobilemode/Action.jsp  
 MECAction SQL 注入（CWE-89），e-cology 8.0/9.0，CVSS 9.8  
  
🔹 **CVE-2024-7704**  
：/cloudstore/ecode/setup/ecology_dev.zip  
 开发源码包泄露（CWE-200），e-cology 8，CVSS 5.3  
  
🔹 **CVE-2023-3793**  
：/weaver/weaver.file.FileDownloadForOutDoc  
 fileid 时间盲注（CWE-89），< 10.58.0  
  
🔹 **CVE-2023-2806**  
：/rest/ofs/RequestInfoByXml  
 XXE 外部实体注入（CWE-611），e-cology 9.0  
  
🔹 **CVE-2023-51892**  
：FrameworkShellController 任意代码执行 RCE，e-cology 10.0.2310.01，CVSS 9.8  
  
🔹 **CVE-2019-10272**  
：/workflow/request/ViewRequestForwardSPA.jsp  
 CRLF 注入（CWE-93），CVSS 6.1  
  
🔹 **经典攻击面①**  
：WorkflowServiceXml XMLDecoder 反序列化 RCE（写冰蝎内存马，e-cology ≤ 9.0）  
  
🔹 **经典攻击面②**  
：BshServlet Beanshell 未授权 RCE（/weaver/bsh.servlet.BshServlet  
，≤ 9.0）  
  
🔹 **经典攻击面③**  
：uploadOperation.jsp 任意文件上传 RCE（写冰蝎 webshell）  
  
收益清单  
  
1. 掌握泛微 e-cology 版本指纹识别与前端暴露面测绘方法，快速锁定可用的攻击面  
  
2. 掌握 CVE-2026-22679 完整利用链：debug 端点探测 → interfaceName/methodName 注入 → Dubbo RPC 命令执行 → PowerShell 免杀落地  
  
3. 掌握 CVE-2025-34038 与 CVE-2023-3793 两类前台 SQL 注入的验证与数据拖取手法  
  
4. 掌握 2024 年 e-cology 9.0 四连击（CVE-2024-48069/48070/48071/48072）的原理边界与利用价值排序  
  
5. 掌握三大经典攻击面（WorkflowServiceXml / BshServlet / uploadOperation）的完整 PoC 与内存马、冰蝎持久化  
  
6. 掌握一键综合利用工具 Apt_t00ls 的运行、五项泛微 RCE 的调用方式与冰蝎连接技巧  
  
7. 获得完整工具资源包：Apt_t00ls（一键综合利用）+ FileDownloadForOutDoc SQLi 专项 PoC + CVE-2026-22679 检测器  
  
**适用读者**  
：渗透测试工程师、红队攻击队员、安全研究员、SRC 漏洞挖掘者、攻防演练参赛选手  
  
**免责声明**  
：本文所有技术内容仅面向授权安全测试与教育研究，任何未经授权的渗透测试行为均违反《中华人民共和国网络安全法》，滥用造成的后果由使用者自行承担。  
  
**下载说明**  
：文末资源包包含完整工具集，包括 Apt_t00ls（图形化一键综合利用工具，覆盖泛微 5 项 RCE）、ecology_oa_FileDownloadForOutDoc_sql.py（CVE-2023-3793 SQLi 检测与 sysadmin 密码遍历）、CVE-2026-22679 检测器（Python 检测脚本 + Nmap NSE 脚本）。  
  
  
一、泛微 e-cology 攻击面总览  
  
