#  漏洞预警 | Flowise服务器端请求伪造漏洞  
浅安
                    浅安  浅安安全   2026-10-29 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-56275  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Flowise是开源的低代码/无代码工具，帮助用户快速构建和部署基于大语言模型的应用程序。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SWaLrCuvt3sbYl6cPYBUicGKLn78CrvapNa0twtqozX7QSMLxKkEHtdSZ1zQdgfJJVvCIWXfp7qHhQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-56275**  
  
**漏洞类型：**  
服务器端请求伪造  
  
**影响：**  
获取敏感信息  
  
**简述：**  
Flowise存在服务器端请求伪造漏洞，由于httpSecurity.ts中缺少secureFetch验证，攻击者能够向内部网络地址发起HTTP请求，访问云元数据，并利用此漏洞枚举内部服务。  
  
**0x04 影响版本**  
- flowise <= 3.0.13  
  
- flowise-components <= 3.0.13  
  
**0x05****POC状态**  
- 未公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://flowiseai.com/  
  
  
  
