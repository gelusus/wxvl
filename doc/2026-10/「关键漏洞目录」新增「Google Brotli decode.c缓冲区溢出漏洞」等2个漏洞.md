#  「关键漏洞目录」新增「Google Brotli decode.c缓冲区溢出漏洞」等2个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-10-28 07:53  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/sz_mmbiz_png/0D2ibczlp7VicpE0kPxHjZtWe6mmJgGHv8PYoyUicAlhJ8qHBkXJS2ibEX0clD3via5m15lIXQLeRrHzsc7KPhc4W56C3Odu0Rc27qHjw81dE7W0/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 2 个漏洞  ·  2026-10-28  
  
01  
## Google Brotli decode.c缓冲区溢出漏洞  
  
  
**CVE-2026-111497**  
超危  
  
**公开日期**  
：2026-10-21 21:07:00  
  
CVSS 4.0：**9.2**  
  
AV:N/AC:L/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  
  
远程  
无需认证  
公开PoC  
供应链漏洞  
CISA KEV  
关键漏洞  
开源组件漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
智能分析漏洞  
  
**漏洞描述**  
  
Google Brotli 1.0.3到1.2.0版本在所有平台的Brotli解码器 (c/dec/decode.c) 中存在缓冲区溢出漏洞，允许远程未认证的攻击者破坏相邻的解码器状态内存、泄露敏感内存内容、导致服务拒绝，或可能通过构造的Brotli压缩流执行任意代码。  
  
**修复建议**  
  
按版本分支升级Brotli至1.0.10、1.1.1或1.2.1及更高版本。参考链接：  
  
https://nodejs.org/en/blog/vulnerability/october-2026-security-releases#nodejs-brotli-decoder-out-of-bounds-write-permits-remote-code-execution-cve-2026-111497  
  
https://github.com/google/brotli  
  
02  
## Adobe Commerce和Magento任意文件上传漏洞  
  
  
**CVE-2026-48356**  
超危  
  
**CNNVD**  
：CNNVD-2026-65616492  
  
**公开日期**  
：2026-07-15 03:44:17  
  
CVSS 3.1：**9.6**  
  
AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:L  
  
远程  
无需认证  
公开PoC  
CISA KEV  
关键漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
开源组件漏洞  
  
**漏洞描述**  
  
Adobe Commerce（含Adobe Commerce B2B）、Magento Open Source及Adobe Commerce Events（Webhooks插件） 存在危险类型文件不受限制上传漏洞。这是由于上传处理器缺失或未有效执行文件类型白名单校验，应用依赖客户端侧检查，或盲信攻击者可控的属性（文件名扩展名、Content-Type 头），导致带危险服务端可执行扩展名（.php/.phtml/.phar 等）的文件被存储到Web可访问路径。攻击者无需任何账户权限，通过诱导受害者访问恶意URL或受控页面触发上传/脚本执行。恶意脚本在受害者浏览器会话上下文中运行，进而窃取会话令牌、接管账户或会话。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://helpx.adobe.com/security/products/magento/apsb26-73.html  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
