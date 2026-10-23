#  漏洞预警 | Fastjson远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-10-22 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-16723  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Fastjson是阿里巴巴开源的一款高性能Java JSON解析库，以其出色的解析速度和简洁的API设计在国内Java生态系统中占据主导地位，广泛应用于企业后端服务、微服务网关、数据接口层等核心业务场景。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30Mhyt1JG8BbzoAkZn5iaVib2tlW1hJrVy8s088P7RMGTcdDMe09mbjh7Zas2TTNyAexm5e0VktR1oguFFVHKOZagRSx023Pn0u2OZo/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-16723**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意代码  
  
**简述：**  
Fastjson存在远程代码执行漏洞，由于其反序列化内部类型校验逻辑存在缺陷，突破传统AutoType黑白名单防护机制。未经验证的攻击者可构造携带恶意JSON载荷，通过网络请求传入目标服务，触发Fastjson原生内部逻辑完成任意代码执行。  
  
**0x04 影响版本**  
- 1.2.68 <= Fastjson 1.x <= 1.2.83  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://github.com/alibaba/fastjson  
  
  
  
