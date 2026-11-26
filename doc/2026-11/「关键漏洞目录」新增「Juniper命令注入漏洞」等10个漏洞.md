#  「关键漏洞目录」新增「Juniper命令注入漏洞」等10个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-11-26 05:11  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/sz_mmbiz_png/0D2ibczlp7VicoC9icnukIFDO9TSxFpsf3JahaMfwAQCiaW57d60qy2riaUZPprdrDvaTvQOFLzceUEHHnDL1RMXZz2hqXQicw0leKiaY3U6pdwVPA/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 10 个漏洞  ·  2026-11-26  
  
01  
## Juniper Networks Session Smart Conductor命令注入漏洞  
  
  
**CVE-2026-103406**  
超危  
  
**CNNVD**  
：CNNVD-2026-33581536  
  
**公开日期**  
：2026-10-29 00:35:53  
  
CVSS 3.1：**9.8**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Juniper Networks Session Smart Conductor是美国Juniper Networks公司的一款集中管理网络会话与路由策略的软件。Juniper Networks Session Smart Conductor 7.0.5之前版本、7.1.7版本之前的7.1.x版本和7.2.3版本之前的7.2.x版本存在命令注入漏洞，该漏洞源于AutomatedProvisioner对OS命令中的特殊元素中和不当，远程攻击者可通过特制的salt minion ID以root身份执行任意命令。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://supportportal.juniper.net/JSA112183  
  
https://supportportal.juniper.net/s/article/2026-10-On-Demand-Security-Bulletin-Session-Smart-Router-and-Session-Smart-Conductor-Multiple-internally-discovered-vulnerabilities-resolved  
  
02  
## Vates Xen Orchestra软件供应链问题漏洞  
  
  
**CVE-2026-133469**  
超危  
  
**CNNVD**  
：CNNVD-2026-62643966  
  
**公开日期**  
：2026-11-24 01:07:16  
  
CVSS 3.1：**9.8**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
开源组件漏洞  
  
**漏洞描述**  
  
Vates xen-orchestra是Vates组织开源的一款虚拟化环境集中管理与备份平台。Vates xen-orchestra 6.9.2之前版本和6.10.1之前版本存在安全漏洞，该漏洞源于REST cookie解析将authenticationToken=j:%7B%7D转换为{}，REST认证未验证字符串即将该对象传给XO token provider，token集合将{}视为未过滤查询并返回第一个token，导致攻击者可以该token所有者身份通过认证，并通过/rest/v0/users/me/authentication_tokens路由获取可重用登录令牌ID。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://github.com/vatesfr/xen-orchestra/security/advisories/GHSA-2r3w-c2vx-fv5g  
  
https://docs.vates.tech/security-advisories/advisories/2026/vates-sa-2026-062/  
  
03  
## Danfoss AK-SM 800A Series安全漏洞  
  
  
**CVE-2025-41450**  
高危  
  
**CNNVD**  
：CNNVD-202505-1267  
  
**公开日期**  
：2025-05-08 17:41:20  
  
CVSS 3.1：**8.2**  
  
AV:N/AC:H/PR:N/UI:R/S:C/C:H/I:L/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Danfoss AK-SM 8xxA Series是丹麦Danfoss公司的一系列工业级智能压力传感器，专为制冷与空调系统设计，支持Modbus通信。Danfoss AK-SM 800A系列包含不正确的身份验证漏洞，可能允许攻击者生成暴露敏感信息的web报告，包括内部IP地址、用户名和存储名称。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.danfoss.com/en/service-and-support/coordinated-vulnerability-disclosure/danfoss-security-advisories/dsa-2025-03-01  
  
04  
## Danfoss AK-SM800A路径遍历漏洞  
  
  
**CVE-2023-25914**  
超危  
  
**CNNVD**  
：CNNVD-202308-1714  
  
**公开日期**  
：2023-08-22 04:30:03  
  
CVSS 3.1：**9.9**  
  
AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Danfoss AK-SM800A是丹麦Danfoss公司的一个系统管理器。提供安全的系统控制和监控Danfoss AK-SM800A 3.3及之前版本存在安全漏洞，该漏洞源于限制不当，攻击者利用该漏洞可以通过XML接口检索并读取底层服务器的系统文件。  
  
**修复建议**  
  
厂家已发布漏洞修复程序，参考链接：  
  
https://www.danfoss.com/en/service-and-support/coordinated-vulnerability-disclosure/danfoss-security-advisories/dsa-2023-08-01  
  
05  
## Danfoss AK-SM800A安全漏洞  
  
  
**CVE-2023-25915**  
超危  
  
**CNNVD**  
：CNNVD-202308-1713  
  
**公开日期**  
：2023-08-22 04:30:02  
  
CVSS 3.1：**9.9**  
  
AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Danfoss AK-SM800A是丹麦Danfoss公司的一个系统管理器。提供安全的系统控制和监控Danfoss AK-SM800A 3.3及之前版本存在安全漏洞，该漏洞源于输入验证不当，攻击者利用该漏洞可以在目标系统上执行任意命令。  
  
**修复建议**  
  
厂家已发布漏洞修复程序，参考链接：  
  
https://www.danfoss.com/en/service-and-support/coordinated-vulnerability-disclosure/danfoss-security-advisories/dsa-2023-08-01  
  
06  
## Mitel MiCollab异常处理不当漏洞  
  
  
**CVE-2026-133324**  
中危  
  
**CNNVD**  
：CNNVD-2026-95980405  
  
**公开日期**  
：2026-11-20 00:31:38  
  
CVSS 3.1：**5.3**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Mitel MiCollab是加拿大Mitel公司的一款企业级统一通信与协作平台。Mitel MiCollab 10.3 (10.3.0.18)及之前版本存在异常处理不当漏洞，该漏洞源于AWV组件认证处理不当，可能导致未经身份验证的攻击者绕过身份验证并在应用程序环境中执行未经授权操作。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-misa-2026-0012  
  
07  
## Mitel MiCollab任意文件上传漏洞  
  
  
**CVE-2026-133275**  
中危  
  
**CNNVD**  
：CNNVD-2026-67552283  
  
**公开日期**  
：2026-11-20 00:09:46  
  
CVSS 3.1：**5.3**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Mitel MiCollab是加拿大Mitel公司的一款企业级统一通信与协作平台。Mitel MiCollab 10.3 (10.3.0.18)及之前版本存在任意文件上传漏洞，该漏洞源于Audio,Web,and Video Conferencing (AWV)组件文件类型验证不足，可能导致未经身份验证的攻击者上传任意文件到目标系统。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-misa-2026-0012  
  
08  
## Mitel MiCollab授权问题漏洞  
  
  
**CVE-2026-133311**  
中危  
  
**CNNVD**  
：CNNVD-2026-51952755  
  
**公开日期**  
：2026-11-20 00:22:22  
  
CVSS 3.1：**5.9**  
  
AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:N/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Mitel MiCollab是加拿大Mitel公司的一款企业级统一通信与协作平台。Mitel MiCollab 10.3 (10.3.0.18)及之前版本存在授权问题漏洞，该漏洞源于Audio,Web,and Video Conferencing (AWV)组件身份验证机制不当，未经身份验证的攻击者在知晓特定系统生成的随机标识符或预置内容的情况下可进行未授权访问，可能导致获取敏感信息。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-misa-2026-0012  
  
09  
## Mitel MiCollab输入验证错误漏洞  
  
  
**CVE-2026-133303**  
超危  
  
**CNNVD**  
：CNNVD-2026-27755238  
  
**公开日期**  
：2026-11-20 00:16:03  
  
CVSS 3.1：**9.0**  
  
AV:N/AC:H/PR:N/UI:N/S:C/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Mitel MiCollab是加拿大Mitel公司的一款企业级统一通信与协作平台。Mitel MiCollab 10.3 (10.3.0.18)及之前版本存在输入验证错误漏洞，该漏洞源于Audio,Web,and Video Conferencing (AWV)组件参数验证不足，可能导致未认证攻击者利用预置恶意内容进行XML注入攻击并在系统环境中执行任意命令。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-misa-2026-0012  
  
10  
## Mitel MiCollab操作系统命令注入漏洞  
  
  
**CVE-2026-133323**  
超危  
  
**CNNVD**  
：CNNVD-2026-36784089  
  
**公开日期**  
：2026-11-20 00:27:10  
  
CVSS 3.1：**10.0**  
  
AV:N/AC:L/PR:N/UI:N/S:C/C:N/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Mitel MiCollab是加拿大Mitel公司的一款企业级统一通信与协作平台。Mitel MiCollab 10.3 (10.3.0.18)及之前版本存在命令注入漏洞，该漏洞源于Audio、Web、Video Conferencing (AWV)组件参数清理不当，可能导致未经身份验证的攻击者进行盲命令注入并在系统环境中执行任意命令。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-misa-2026-0012  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
