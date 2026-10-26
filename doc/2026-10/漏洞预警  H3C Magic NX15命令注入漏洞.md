#  漏洞预警 | H3C Magic NX15命令注入漏洞  
浅安
                    浅安  浅安安全   2026-10-25 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-112370  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
H3C Magic NX15千兆Wi-Fi 6无线路由器是新华三智能终端有限公司全新设计的路由器，面向追求极致性价比网络的用户和群体。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXonhb4Z0OQUPUuPpRHZDL9GpNiauzymWWDcmkyDXzNfIYUMPKQkWyIWafhd7QyEPib3xKFsWUIOTYQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-112370**  
  
**漏洞类型：**  
命令注入  
  
**影响：**  
执行任意代码  
  
**简述：**  
H3C Magic NX15的/api/esps接口存在命令注入漏洞，经过验证的攻击者可通过该漏洞向操作系统注入命令，进而已当前状态执行任意命令，获取系统权限。  
  
**0x04 影响版本**  
- H3C Magic NX15 100R017  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.h3c.com/  
  
  
  
