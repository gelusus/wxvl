#  漏洞预警 | JeecgBoot SQL注入漏洞  
浅安
                    浅安  浅安安全   2026-11-15 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-124554  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
JeecgBoot是一款基于代码生成器的低代码开发平台。前后端分离架构 SpringBoot2.x，SpringCloud，Ant Design&Vue，Mybatis-plus，Shiro，JWT，支持微服务。强大的代码生成器让前后端代码一键生成，实现低代码开发。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SW25cvic5s59KsEGyS6fTiaaogPXrsntzHQO87dEEoxdTnicwvibHLUm8wvIaRo5oQWGzJqicZPr2Q7G9w/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-124554**  
  
**漏洞类型：**  
SQL注入  
  
**影响：**  
获取敏感信息  
  
**简述：**  
JeecgBoot的  
/jeecg-boot/drag/onlDragDatasetHead/getAllChartData  
接口存在SQL注入漏洞，未经验证的攻击者可通过远程HTTP请求构造注入语句，绕过原有查询条件，进而读取、修改数据库内容，甚至结合数据库权限进一步利用。  
  
**0x04 影响版本**  
- JeecgBoot   
3.9.5  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
http://www.jeecg.com/  
  
  
  
