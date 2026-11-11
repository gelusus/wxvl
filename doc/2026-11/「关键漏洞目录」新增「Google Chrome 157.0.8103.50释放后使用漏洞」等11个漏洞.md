#  「关键漏洞目录」新增「Google Chrome <157.0.8103.50释放后使用漏洞」等11个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-11-11 03:15  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/sz_mmbiz_png/0D2ibczlp7V9lKDdib1an4SLZ0RlVRwcibCakkMzySpc0WKFH2B3WicyP0nTds4oQBzJFp9Jw6tIEeu8VllbYCjVXzBtpuSfUQuS7T2ofXFmrqU/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 11 个漏洞  ·  2026-11-11  
  
01  
## Google Chrome <157.0.8103.50释放后使用漏洞  
  
  
**CVE-2026-125350**  
高危  
  
**公开日期**  
：2026-11-11 01:20:21  
  
CVSS 3.1：**8.8**  
  
AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
Google Chrome 157.0.8103.50之前版本的XML组件libxslt的pattern.c 、中的 xsltTestCompMatchDirect() 函数存在use after free漏洞，允许远程攻击者通过特制的HTML页面在沙箱内执行任意代码。此漏洞影响使用Chromium的多种浏览器，包括不限于Chrom、Egde、Opera等。  
  
**修复建议**  
  
请升级Chrome到157.0.8103.50或更高版本，参考链接：  
  
https://chromereleases.googleblog.com/2026/11/stable-channel-update-for-desktop_018223192.html  
  
02  
## Barracuda Email Security Gateway身份验证绕过漏洞  
  
  
**CVE-2026-116606**  
超危  
  
**CNNVD**  
：CNNVD-2026-19833037  
  
**公开日期**  
：2026-11-02 20:59:11  
  
CVSS 4.0：**9.3**  
  
AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
漏洞攻击链  
  
**漏洞描述**  
  
Barracuda Email Security Gateway 9.4.0.030之前版本存在身份验证绕过漏洞，攻击者可通过替代路径或通道绕过认证，允许未经身份验证的远程攻击者调用配置导入功能并恢复经过精心制作的兼容备份归档，从而导致对设备全局配置的未经授权修改。此漏洞与CVE-2026-111560形成漏洞攻击链。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://trust.barracuda.com/security/pre-authentication-authorization-bypass-command-injection  
  
03  
## Barracuda Networks多款产品操作系统命令注入漏洞  
  
  
**CVE-2026-111560**  
高危  
  
**CNNVD**  
：CNNVD-2026-17806528  
  
**公开日期**  
：2026-11-02 20:59:03  
  
CVSS 4.0：**8.6**  
  
AV:N/AC:L/AT:N/PR:H/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
远程  
公开PoC  
CISA KEV  
关键漏洞  
漏洞攻击链  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
CWE Top 25 (2025)  
  
**漏洞描述**  
  
BBarracuda Email Security Gateway 9.4.0.27及之前版本、Barracuda Load Balancer Application Delivery Controller (ADC) 6.4.0及之前版本、Barracuda Web Application Firewall (WAF) 11.0.1及之前版本、Barracuda Web Security Gateway (WSG) 16.1.100.100及之前版本、Barracuda Message Archiver (BMA) 5.5.4.0及之前版本和Barracuda WAF Control Center (WCC) 2.2.1及之前版本存在命令注入漏洞，该漏洞源于时区配置字段对Shell元字符中和不当，可能导致经过身份验证的管理员注入以root权限执行的操作系统命令。此漏洞与CVE-2026-116606形成漏洞攻击链。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://trust.barracuda.com/security/pre-authentication-authorization-bypass-command-injection  
  
04  
## Cisco BroadWorks XML外部实体引用漏洞  
  
  
**CVE-2026-20320**  
高危  
  
**CNNVD**  
：CNNVD-2026-11775196  
  
**公开日期**  
：2026-08-20 00:39:04  
  
CVSS 3.1：**7.5**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Cisco BroadWorks是美国Cisco公司的一款提供呼叫控制与PBX功能的交换机。 Cisco BroadWorks的开放客户端接口 (OCI) XML解析器中存在漏洞，可能允许未经身份验证的远程攻击者读取受影响系统上的敏感配置信息。漏洞存在的原因是XML条目由于默认允许外部实体解析而被不正确解析。攻击者可以通过向开放客户端接口 - 配置 (OCI-P) 服务发送精心构造的XML消息来利用该漏洞。成功利用该漏洞可能允许攻击者以Cisco BroadWorks用户的权限查看文件系统中的敏感文件。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-bworks-xxe-uwUd7CEt  
  
05  
## Openwave Webtop Media Transcoder权限许可和访问控制问题漏洞  
  
  
**CVE-2026-111793**  
中危  
  
**CNNVD**  
：CNNVD-2026-52028904  
  
**公开日期**  
：2026-10-17 02:28:10  
  
CVSS 3.1：**4.3**  
  
AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:N/A:L  
  
远程  
公开PoC  
CISA KEV  
关键漏洞  
智能分析漏洞  
  
**漏洞描述**  
  
Openwave Webtop Media Transcoder是Openwave公司的一款媒体转码软件。Openwave Webtop Media Transcoder 1.1.22.0之前版本存在权限许可和访问控制问题漏洞，该漏洞源于未强制执行管理员定义的文件类型限制，可能导致经过身份验证的用户绕过Webmail体验处理和转换非预期文档类型。  
  
**修复建议**  
  
https://github.com/GISOpenwave/security-advisories-public/blob/main/CAN-2026-2032674.md  
  
06  
## Microsoft Outlook安全漏洞(CVE-2026-124194)  
  
  
**CVE-2026-124194**  
中危  
  
**公开日期**  
：2026-11-11 02:22:22  
  
CVSS 3.1：**6.5**  
  
AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:N/A:N/E:H/RL:O/RC:C  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Microsoft Office Outlook在打开特制邮件消息文件时，未能弹出正常的"外部内容/不安全操作"警告对话框，导致安全防线（用户确认环节）被静默跳过。允许未经授权的攻击者在网络上执行欺骗。  
  
**修复建议**  
  
厂家已发布漏洞修复程序，参考链接：  
  
https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-124194  
  
07  
## IBM Aspera Faspex授权问题漏洞  
  
  
**CVE-2026-123218**  
超危  
  
**CNNVD**  
：CNNVD-2026-36695834  
  
**公开日期**  
：2026-11-07 11:47:19  
  
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
  
IBM Aspera Faspex是美国IBM公司的一套企业级文件高速传输平台。IBM Aspera Faspex 5.0.0版本至5.0.19版本存在授权问题漏洞，该漏洞源于授权用户可构造特定消息绕过用户授权，可能导致获得完整管理员权限。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://www.ibm.com/support/pages/node/7295921  
  
08  
## SignalWire FreeSWITCH代码注入漏洞  
  
  
**CVE-2026-116779**  
超危  
  
**CNNVD**  
：CNNVD-2026-77416984  
  
**公开日期**  
：2026-11-06 03:30:25  
  
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
开源组件漏洞  
  
**漏洞描述**  
  
SignalWire FreeSWITCH是SignalWire组织开源的一款支持语音、视频和即时消息的通信软交换平台。SignalWire FreeSWITCH 1.11.3之前版本存在安全漏洞，该漏洞源于sofia_presence_handle_sip_i_message()对入站SIP MESSAGE的To用户按加号拆分并将左侧作为聊天协议选择器，选择api chat proto可达到switch_api_execute()，允许消息被接受的对端以守护进程权限执行FreeSWITCH API命令及操作系统命令，导致配置和凭据泄露、任意呼叫控制和命令执行。  
  
**修复建议**  
  
升级到1.11.3或更高版本，参考链接：  
  
https://github.com/signalwire/freeswitch/security/advisories/GHSA-mg63-h38w-8p9p  
  
09  
## OpenSSH musl环境密码认证绕过漏洞(sshhol)  
  
  
**CVE-2026-123158**  
高危  
  
**CNVD**  
：CNVD-2026-44652  
**CNNVD**  
：CNNVD-2026-29362983  
  
**公开日期**  
：2026-11-05 23:46:26  
  
CVSS 3.1：**8.1**  
  
AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
开源组件漏洞  
  
**漏洞描述**  
  
在Linux上使用musl libc且不使用PAM的情况下，OpenSSH 10.6p2之前的sshd缺乏针对密码认证绕过的解决方法(与是否将字符串解释为密码哈希有关)。对于某些帐户配置，任何长度在256到1024个字符之间的密码都能成功。  
  
**修复建议**  
  
建议升级到OpenSSH 10.6p2及以上版本。参考链接：  
  
https://www.openssh.org/releasenotes.html#10.6p2  
  
https://github.com/openssh/openssh-portable/commit/4353fdc43ef646c43a8dda7025cc1a898caf1482  
  
https://alpinelinux.org/posts/2026-11-05-openssh-password-authentication-bypass/  
  
10  
## GPUStack授权问题漏洞(CVE-2026-102895)  
  
  
**CVE-2026-102895**  
超危  
  
**CNNVD**  
：CNNVD-2026-90103029  
  
**公开日期**  
：2026-07-31 19:58:14  
  
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
  
GPUStack GPUStack是GPUStack团队的一个管理GPU资源的服务器软件。GPUStack 2.2.0之前版本存在安全漏洞，该漏洞源于gpustack/api/auth.py中的get_current_user在force_auth_localhost为false（默认）时将环回请求源视为内置管理员，远程请求可通过同主机反向代理或客户端提供的X-GPUStack-Real-IP环回值伪装成来自127.0.0.1，导致未经验证的攻击者获得管理访问权限，可修改用户和模型、创建持久管理员API密钥、泄露worker和云提供商机密，并通过POST /v2/inference-backends/from-yaml执行代码。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://github.com/gpustack/gpustack/security/advisories/GHSA-gj9m-f8ph-pmhr  
  
https://github.com/gpustack/gpustack/commit/53f42a6a553542974396d16a4ff6b12bcbedb48d  
  
https://github.com/gpustack/gpustack/commit/c6e6b91d56ce12f3ce7293c56be8a363b7a41618  
  
11  
## SonicWall SonicOS SSLVPN安全漏洞(CVE-2026-19868)  
  
  
**CVE-2026-19868**  
高危  
  
**公开日期**  
：2026-11-07 03:03:00  
  
CVSS 3.1：**7.5**  
  
AV:N/AC:H/PR:N/UI:N/S:C/C:H/I:L/A:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
SonicOS SSLVPN包含使用单因素身份验证漏洞，该漏洞可能允许具有替代身份验证方法的有效凭据的用户绕过SAML身份验证控制。成功利用该漏洞也可能绕过SAML强制的多因素身份验证（MFA）。  
  
**修复建议**  
  
厂家已发布漏洞修复程序，参考链接：  
  
https://psirt.global.sonicwall.com/vuln-detail/SNWLID-2026-0014  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
