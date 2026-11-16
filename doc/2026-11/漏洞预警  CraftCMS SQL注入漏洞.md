#  漏洞预警 | CraftCMS SQL注入漏洞  
浅安
                    浅安  浅安安全   2026-11-15 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-115934  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
CraftCMS是一个灵活的、易于使用的内容管理系统。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SU8Hf5YibCK8obmSCq5lcwuicW45vQDp2TGuPUldZibXaj87iaPNkrGu3D3R2ib7oc5lK6rPab4859G6gQ/640?wx_fmt=png&randomid=xu0uxjj1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-115934**  
  
**漏洞类型：**  
SQL注入  
  
**影响：**  
账号接管  
  
**简述：**  
CraftCMS存在SQL注入漏洞，由于前端搜索功能中的attribute参数被未经清理便直接拼接进 SQL查询，且底层PDO使用模拟预处理，未经验证的攻击者可以通过该参数执行堆叠查询，从而对应用程序数据库获得读写访问权限，进而将权限提升至管理员。  
  
**0x04 影响版本**  
- 4.0.0 <= Craft CMS < 4.18.9  
  
- 5.0.0 <= Craft CMS < 5.11.2  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
******目前官方已发布漏洞修复版本，建议用户升级到安全版本****：******  
  
https://craftcms.com/  
  
  
  
