#  「关键漏洞目录」新增「ATLASSIAN多款产品任意文件读取漏洞」等3个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-10-19 06:12  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/mmbiz_png/0D2ibczlp7V8z8A2fM5jV6VQagmJ8hJQwfLM7j4Rp6rWhxs7tskX33cFTgywPtia8btggPBuDeXoQD95Vt5oztZv3LYjwcWibyUp72ic9HZTHJo/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 3 个漏洞  ·  2026-10-19  
  
01  
## ATLASSIAN多款产品任意文件读取漏洞  
  
  
**CVE-2026-21589**  
超危  
  
**公开日期**  
：2026-10-06 05:30:00  
  
CVSS 4.0：**9.3**  
  
AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:N/VA:N/SC:H/SI:H/SA:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
ATLASSIAN Bitbucket Data Center、Confluence Data Center、Jira Service Management Data Center、Jira Software Data Center、Bamboo Data Center、Crowd Data Center、Crucible和Fisheye的atlassian-plugins-webresource组件存在任意文件访问漏洞，该组件使用::作为URL中/的转义表示，攻击者可构造包含..::的恶意请求绕过路径安全检查，在无需任何身份认证的情况下访问受影响版本中Web应用程序根目录中的特定文件。在部署 Atlassian Crowd的环境中，还可进一步读取Crowd明文凭据并通过Crowd REST API创建管理员账户，实现从匿名访问到完全接管。利用该漏洞需要事先知道目标文件的确切名称和路径，该漏洞不允许攻击者枚举或列出目录内容。  
  
**修复建议**  
  
ATLASSIAN已发布针对此漏洞的修复程序，参考链接：  
  
https://confluence.atlassian.com/security/cve-2026-21589-arbitrary-file-access-vulnerability-impacts-multiple-products-1870495748.html  
  
https://github.com/advisories/GHSA-gr66-x5pq-8g2g  
  
02  
## Fortinet多款产品路径遍历漏洞(CVE-2025-61624)  
  
  
**CVE-2025-61624**  
中危  
  
**CNNVD**  
：CNNVD-202604-3014  
  
**公开日期**  
：2026-04-14 23:39:51  
  
CVSS 3.1：**6.0**  
  
AV:L/AC:L/PR:H/UI:N/S:U/C:N/I:H/A:H/E:P/RL:O/RC:C  
  
本地  
公开PoC  
供应链漏洞  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
  
**漏洞描述**  
  
FortiOS、FortiPAM、FortiProxy、FortiSwitchManager 包含路径遍历漏洞，可能允许具有管理员配置文件和至少读写权限的经过身份验证的攻击者通过特定的CLI命令写入或删除任意文件。漏洞同时影响使用Fortigate NGFW的西门子RUGGEDCOM APE1808 Devices设备。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://fortiguard.fortinet.com/psirt/FG-IR-26-122  
  
03  
## BeyondTrust Remote Support身份验证绕过漏洞  
  
  
**CVE-2026-102764**  
超危  
  
**公开日期**  
：2026-10-16 04:30:32  
  
CVSS 4.0：**9.3**  
  
AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
BeyondTrust Remote Support 26.2.3及以前版本内部消息组件信任性地接收消息，未校验消息来源是否合法。攻击者发送特制请求即可让设备执行本应由可信内部组件发起的文件写操作。  
  
**修复建议**  
  
请升级到24.3.1或更高版本，参考链接：  
  
https://www.beyondtrust.com/trust-center/security-advisories/bt26-05  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
