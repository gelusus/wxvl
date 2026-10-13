#  漏洞预警 | Telegram Desktop IPC记录注入漏洞  
浅安
                    浅安  浅安安全   2026-10-12 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-107181  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Telegram是提供开源客户端的跨平台即时通信软件。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXZIwUGcg2RhOe7RB9ubibNZBWOMsRnWCiaGqswAxMs8ItRUG3aNWgYyzguasmaAGdpXm2m8ZW12aHw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-107181**  
  
**漏洞类型：**  
IPC记录注入  
  
**影响：**  
账号接管  
  
**简述：**  
T  
elegram Desktop存在IPC记录注入漏洞，新启动的实例将系统调起的tg://链接序列化为OPEN:记录写入单实例IPC本地套接字时未转义URL内部的记录分隔符分号，已运行实例在Sandbox::readClients中按分号盲目切分记录，攻击者构造含分号的链接可注入任意OPEN:记录；注入的OPEN:interpret:路径记录会进入内部interpret: scheme处理流程，Support::InterpretSendPath无授权校验与确认弹窗，按指令文件把指定本地文件上传到攻击者指定的频道或超级群组，造成本地任意文件读取与账号接管。  
  
**0x04 影响版本**  
- Telegram Desktop <= 7.2.8  
  
**0x05 POC状态**  
- 已公开  
  
**0x06 修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://telegram.org/  
  
  
  
