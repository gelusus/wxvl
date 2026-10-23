#  漏洞预警 | WordPress plugin Booktics SQL注入漏洞  
浅安
                    浅安  浅安安全   2026-10-22 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-109894  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Booktics是一款功能强大的预订日历和预约管理插件。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30MhzDoHU8RmicPoaMeicdCBavVDfjOH3DDSWb7T0tyfpa7YLxkHqwtlvOyA7S03pCKiaFbQyovV3DQjv6jF1HOlNXp8BHOFIaSvdc9w/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-109894**  
  
**漏洞类型：**  
SQL注入  
  
**影响：**  
获取敏感信息  
  
**简述：**  
Booktics  
存在SQL注入漏洞  
，未经身份验证的攻击者可利用该漏洞注入SQL语句，进而从数据库中提取敏感信息。  
  
**0x04 影响版本**  
- Booktics  
 <=   
1.0.29  
  
**0x05****POC状态**  
- 未公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://cn.wordpress.org/plugins/booktics/  
  
  
  
