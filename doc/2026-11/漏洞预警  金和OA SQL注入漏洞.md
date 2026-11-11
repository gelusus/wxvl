#  漏洞预警 | 金和OA SQL注入漏洞  
浅安
                    浅安  浅安安全   2026-11-10 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-124331  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
金和网络是专业信息化服务商,为城市监管部门提供了互联网+监管解决方案,为企事业单位提供组织协同OA系统开发平台,电子政务一体化平台,智慧电商平台等服务。   
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXOhrIicAqN0TULjm0J7IN5y6sLyKp9DrQOTW803iaDNmGpf9bm43iaggj4WUjsIpZLnLhdic1mhbExTQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-124331**  
  
**漏洞类型：**  
SQL注入  
  
**影响：**  
获取敏感信息  
  
**简述：**  
金和OA C6的/C6/JHSoft.Web.Project/OfficeReadOnly/ProjectAfficheManage.aspx/ProjectManage接口存在SQL注入漏洞，未经身份验证的攻击者可以通过该漏洞执行任意SQL语句，从而获取数据库敏感信息。  
  
**0x04 影响版本**  
- 金和OA C6  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
http://www.jinher.com/  
  
  
  
