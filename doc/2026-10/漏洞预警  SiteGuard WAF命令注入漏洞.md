#  漏洞预警 | SiteGuard WAF命令注入漏洞  
浅安
                    浅安  浅安安全   2026-10-20 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-106931  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
SiteGuard WAF是日本EG Secure Solutions的主机型WAF。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NQlfTO30MhyGqJSmOduhHzRqGY827RZrgiaqVysDiaDF6AjPnNXT9wJrPBzWkGQSPhDrf0DcH5jfWnS2sXhrVrPQ4MiaBSwNK7xxnBtiaEKemow/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-106931**  
  
**漏洞类型：**  
命令注入  
  
**影响：**  
执行  
任意命令  
  
**简述：**  
SiteGuard WAF存在命令注入漏洞，由于其统计处理逻辑对外部可控字段没有充分过滤，未经授权的攻击者可构造特定请求把OS命令注入到处理流程里，最终以root或SYSTEM权限在服务器上跑任意命令。  
  
**0x04 影响版本**  
- Linux/Windows Apache、Nginx、IIS < 8.00 Update 5  
  
- FreeBSD Apache、Nginx < 7.00 Update 6  
  
**0x05 POC状态**  
- 未公开  
  
**0x06 修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.eg-secure.co.jp/  
  
  
  
