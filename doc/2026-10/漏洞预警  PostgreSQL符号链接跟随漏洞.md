#  漏洞预警 | PostgreSQL符号链接跟随漏洞  
浅安
                    浅安  浅安安全   2026-10-12 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-6475  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
PostgreSQL是一个开源、强大的关系型数据库管理系统，支持SQL标准及扩展，广泛应用于企业级应用。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SU3hGH08FibTuicaaGzPwn7gicCRVfVonzx5pwtdRjBSvL89OHTSyWnGqGmTUoov8quGRvzN0BgXiaXpA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-6475**  
  
**漏洞类型：**  
符号链接跟随  
  
**影响：**  
任意文件写入  
  
**简述：**  
PostgreSQL存在符号链接跟随漏洞，在解包tar备份时，若归档中存在指向绝对路径的符号链接，程序会跟随该链接进行写入，而非拒绝或限制路径。攻击者可通过构造恶意备份归档，诱导备份过程将文件写入目标系统的任意位置。  
  
**0x04 影响版本**  
- PostgreSQL 18 < 18.4  
  
- PostgreSQL 17 < 17.10  
  
- PostgreSQL 16 < 16.14  
  
- PostgreSQL 15 < 15.18  
  
- PostgreSQL 14 < 14.23  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.postgresql.org/  
  
  
  
