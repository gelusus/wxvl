#  「关键漏洞目录」新增「Check Point Quantum Security Gateway访问控制问题漏洞」等2个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-11-16 01:00  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/mmbiz_png/0D2ibczlp7VicbY3biaGA3E4Nc59YBvlAKxWvmGQZd0yNnUYiahkJcnA2u5jPfSVYIibiaWpac75COloibpVTr3uMibsudQq1icdoppkncHibUKXKGF8M/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 2 个漏洞  ·  2026-11-14  
  
01  
## Check Point Quantum Security Gateway权限许可和访问控制问题漏洞(CVE-2026-123039)  
  
  
**CVE-2026-123039**  
高危  
  
**CNNVD**  
：CNNVD-2026-54804314  
  
**公开日期**  
：2026-11-12 22:02:13  
  
CVSS 3.1：**7.8**  
  
AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H  
  
本地  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Check Point Quantum Security Gateway是以色列Check Point公司的一款提供高级威胁防护与统一安全策略管理的网络安全网关。Check Point Quantum Security Gateway存在安全漏洞，该漏洞源于Gaia OS的DynamicCLI守护进程以root权限运行并监听全局可写的本地套接字/tmp/dynamic_socket，且未对连接进程进行身份验证，可能导致经过身份验证的本地用户向该套接字发送命令并以root权限执行。  
  
**修复建议**  
  
厂商已发布漏洞修复程序，参考链接：  
  
https://support.checkpoint.com/results/sk/sk1000362  
  
02  
## UnrealIRCd输入验证错误漏洞  
  
  
**CVE-2026-114713**  
高危  
  
**CNNVD**  
：CNNVD-2026-42781955  
  
**公开日期**  
：2026-11-08 10:31:48  
  
CVSS 3.1：**8.6**  
  
AV:N/AC:L/PR:N/UI:N/S:C/C:N/I:H/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
UnrealIRCd是UnrealIRCd组织开源的一个开源IRC服务器。UnrealIRCd 6.0.0至6.2.7.1之前版本存在输入验证错误漏洞，该漏洞源于labeled-response模块未转义IRCv3  label消息标签，可能导致任何用户注入以服务器权限执行的服务器间命令。  
  
**修复建议**  
  
升级到UnrealIRCd 6.2.7.1或更高版本。参考链接：  
  
https://forums.unrealircd.org/viewtopic.php?t=9487  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
