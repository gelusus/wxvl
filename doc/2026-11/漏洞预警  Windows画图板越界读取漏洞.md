#  漏洞预警 | Windows画图板越界读取漏洞  
浅安
                    浅安  浅安安全   2026-11-10 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-83844  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Microsoft Paint是Windows操作系统中内置的经典图像编辑应用程序。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30MhyHChHHZibHl3WgBuhLVqJ5bR5zwq9KTUyJGvfAcy1m22amzaTTsvUSeYkcY5AjBgD0g10oOQLfkMcQzgVJWBJRYaFqUcCxkueg/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-83844**  
  
**漏洞类型：**  
越界读取  
  
**影响：**  
执行任意代码  
  
**简述：**  
Microsoft Paint存在越界读取漏洞，未经身份认证的攻击者可通过向受影响的服务发送特制数据包来触发该漏洞，进而执行任意代码。  
  
**0x04 影响版本**  
- 11.0.0.0 <= Microsoft Paint <   
11.2609.401.0  
  
**0x05 POC状态**  
- 未公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.microsoft.com/zh-cn/windows/paint  
  
  
  
