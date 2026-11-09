#  漏洞预警 | CyberPanel命令注入漏洞  
浅安
                    浅安  浅安安全   2026-11-08 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-78818  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
CyberPanel是一个开源的Web控制面板，它提供了一个用户友好的界面，用于管理网站、电子邮件、数据库、FTP账户等。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SUNl1K8QYt0kne3TvEh7kEBosyE7bsF75WCk1DtQZnpBGFCzkfBbb2MJeueD2ju6bTUuuKSWJdMsA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-78818**  
  
**漏洞类型：**  
命令注入  
  
**影响：**  
执行任意代码  
  
**简述：**  
CyberPanel存在命令注入漏洞，由于changePHP()函数在处理phpSelection参数时，未进行任何验证或清理，经过身份验证的攻击者可通过发送特制的POST请求触发该漏洞，进而以root身份执行任意系统命令。  
  
**0x04 影响版本**  
- CyberPanel <=   
2.4.7  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://github.com/usmannasir/cyberpanel  
  
  
  
  
