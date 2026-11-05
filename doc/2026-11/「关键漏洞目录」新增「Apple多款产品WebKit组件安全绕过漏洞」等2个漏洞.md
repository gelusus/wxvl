#  「关键漏洞目录」新增「Apple多款产品WebKit组件安全绕过漏洞」等2个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-11-05 05:22  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/sz_mmbiz_png/0D2ibczlp7VicMIYtwfSlnEkzbQibBlvIv76WK49VXbagMYjh8nS7HENpRdF6gYJgTicymicpic7P1CUBb90AWKKkj5CY5MEGdbmD1PzWfxGs8JgU/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 2 个漏洞  ·  2026-11-05  
  
01  
## Apple多款产品WebKit组件安全绕过漏洞  
  
  
**CVE-2024-27834**  
中危  
  
**CNNVD**  
：CNNVD-202405-1869  
  
**公开日期**  
：2024-05-14 07:00:50  
  
CVSS 3.1：**6.5**  
  
AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N  
  
本地  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
开源组件漏洞  
CWE Top 25 (2024)  
  
**漏洞描述**  
  
Apple iOS 17.5、iPadOS、tvOS、Safari、watchOS、macOS Sonoma可能允许本地攻击者绕过由WebKit组件中的问题引起的安全限制。通过使用特制的应用程序，攻击者可以利用此漏洞绕过指针身份验证。  
  
**修复建议**  
  
此问题已在iOS 17.5和iPadOS 17.5，Apple tvOS 17.5，Safari 17.5，watchOS 10.5，macOS Sonoma 14.5中修复。有关修补程序、升级或建议的解决方法信息，请参阅如下链接：  
  
https://support.apple.com/en-us/HT214104  
  
https://support.apple.com/en-us/HT214101  
  
https://support.apple.com/en-us/HT214102  
  
https://support.apple.com/en-us/HT214103  
  
https://support.apple.com/en-us/HT214106  
  
02  
## WordPress Core 4.7.0- 7.1.3远程代码执行漏洞  
  
  
**CVE-2026-119977**  
高危  
  
**CNNVD**  
：CNNVD-2026-20189835  
  
**公开日期**  
：2026-10-28 08:00:00  
  
CVSS 3.1：**8.1**  
  
AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
WordPress Core在4.7到7.1.3版本中易受远程代码执行攻击。这是由于在完整包核心升级期间使用了可预测的、可通过网页访问的暂存路径。这使得未经身份验证的攻击者可以在升级窗口期间在服务器上执行代码。  
  
**修复建议**  
  
厂家已发布漏洞修复程序，参考链接：  
  
https://wordpress.org/news/2026/10/wordpress-7-1-4  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
