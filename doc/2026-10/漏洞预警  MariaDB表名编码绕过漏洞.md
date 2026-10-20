#  漏洞预警 | MariaDB表名编码绕过漏洞  
浅安
                    浅安  浅安安全   2026-10-20 00:00  
  
**0x00 漏洞编号**  
- # CVE-2026-92262  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
MariaDB是一款高性能开源关系型数据库管理系统，是MySQL的分支，广泛应用于Web应用、云原生、大数据与分析场景。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NQlfTO30Mhy5vicWOOPUd9XqTzHTAh2EFwKJQA3OSVgRXxyWYuVPS5z8emqnDUkv3RtgqMaYbc886DicS9fEnUh3OEQ9pUHeMicnGmW381Vc2s/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-92262**  
  
**漏洞类型：**  
表名编码绕过  
  
**影响：**  
SQL注入  
  
**简述：**  
MariaDB存在表名编码绕过漏洞，由于其为兼容旧版表名而保留的mysql50前缀机制，该机制允许绕过表名在文件系统中的编码处理，并可能引发表缓存键冲突，导致不同表名错误地关联至同一表对象。具有数据库表创建及数据操作权限的低权限攻击者，可通过构造特定SQL语句触发漏洞，造成数据混淆、数据损坏或敏感信息泄露，严重情况下还可能导致数据库服务崩溃或远程代码执行。  
  
**0x04 影响版本**  
- 10.6.1 <= MariaDB <= 10.6.27  
  
- 10.11.1 <= MariaDB <= 10.11.18  
  
- 11.4.1 <= MariaDB <= 11.4.12  
  
- 11.8.1 <= MariaDB <= 11.8.8  
  
- 12.3.1 <= MariaDB <= 12.3.2  
  
- MariaDB 13.0.1  
  
**0x05 POC状态**  
- 已公开  
  
**0x06 修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://mariadb.org/  
  
  
  
