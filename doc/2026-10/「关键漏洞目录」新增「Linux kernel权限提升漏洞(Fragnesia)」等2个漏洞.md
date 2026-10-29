#  「关键漏洞目录」新增「Linux kernel权限提升漏洞(Fragnesia)」等2个漏洞  
摄星科技
                    摄星科技  关键漏洞目录   2026-10-29 05:05  
  
![关键漏洞目录 Logo](https://mmbiz.qpic.cn/sz_mmbiz_png/0D2ibczlp7V8ZzO5r8bKsXo4ejN6ibZibXHUyO3icFwFH4iaSRu7qc0TibTObEiclhhda2icL31adNu0UIzu3m0amX64SXJycPo0AheSE3v3goEdyec/640?from=appmsg "")  
  
摄星科技  
# 关键漏洞目录  
  
新增 2 个漏洞  ·  2026-10-29  
  
01  
## Linux kernel权限提升漏洞(Fragnesia)  
  
  
**CVE-2026-46300**  
高危  
  
**CNNVD**  
：CNNVD-202605-4847  
  
**公开日期**  
：2026-05-14 02:04:12  
  
CVSS 3.1：**7.8**  
  
AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H  
  
本地  
公开PoC  
CISA KEV  
关键漏洞  
漏洞攻击链  
开源组件漏洞  
CWE Top 25 (2023)  
CWE Top 25 (2024)  
  
**漏洞描述**  
  
Linux内核XFRM ESP-in-TCP子系统存在本地权限提升漏洞。net/core/skbuff.c的skb_try_coalesce()函数在合并数据包的paged frags时未传播SKBFL_SHARED_FRAG标记，导致ESP输入处理（esp4.c/esp6.c）误判skb为非共享状态而跳过skb_cow_data()私有拷贝，直接在页缓存支撑的内存页上执行解密操作。ESP解密本质是异或密钥流写入，攻击者可通过构造恶意ESP数据流，向任意只读文件的页缓存写入攻击载荷，随后执行该程序即可获得root权限。此漏洞可与CVE-2026-109618和CVE-2026-109619形成漏洞攻击链，植入PHP网络后门并以root权限执行命令。  
  
**修复建议**  
  
建议应用各稳定分支回移提交补丁或最新版本。参考链接：  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/760e1addc27ba1a7beb4a0a7e8b3e9ec49e7a34e  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/78bf6b6bb19541d19fbda6242e7cfe2c682763c0  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/3884358a9286b17f389a72b1426fc4547c23c111  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/3599e6b3cc1ada96883d496a50a210d3afbb6987  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/9d3e5fd19fe1063bf607219e8562fbd567b8e8d5  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/3bd9e113d50034db99d7ef69fd8e5242d15e414a  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/2f2b16022a2e10ca7bccfb98db5ed2ec0f72641c  
  
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux/commit/f84eca5817390257cef78013d0112481c503b4a3  
  
02  
## Broadcom VMWare ESX日志记录不足漏洞  
  
  
**CVE-2026-41709**  
低危  
  
**CNNVD**  
：CNNVD-2026-80292976  
  
**公开日期**  
：2026-07-30 20:45:02  
  
CVSS 3.1：**2.7**  
  
AV:N/AC:L/PR:H/UI:N/S:U/C:N/I:L/A:N  
  
远程  
公开PoC  
CISA KEV  
关键漏洞  
  
**漏洞描述**  
  
Broadcom VMWare ESX存在日志记录不充分的漏洞，可能允许拥有管理员访问权限的恶意行为者在未被记录的情况下执行某些操作。  
  
**修复建议**  
  
当前厂商已发布了补丁信息，请及时关注更新：  
  
https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/38017  
  
**服务和支持**  
  
“关键漏洞目录”由摄星科技维护，可通过 https://www.vulinsight.com.cn/keyFlawCatalog 查看详细内容。并可下载 CSV、JSON 格式全量漏洞数据。  
  
企业用户可通过玄猫定制化情报 SaaS 平台（xm.vulinsight.com.cn）、摄星漏洞管控平台获取服务和支持。  
  
