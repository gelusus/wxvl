#  漏洞预警 | Nodejs Brotli远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-10-28 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-111497  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Node.js是一个基于Chrome V8引擎的JavaScript运行时环境，它打破了传统“JavaScript仅能在浏览器中运行”的限制，允许开发者使用JavaScript编写服务器端代码，实现全栈开发。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXhHE2o9zeyXNKic0dQJd3coTnZG5WCxywOYgRUjmib6FID1sxG65gMQYQRbbcRCgtQ8SERcdeIGeibA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-111497**  
  
**漏洞类型：**  
代码执行  
  
**影响：**  
执行恶意代码  
  
**简述：**  
Node.js Brotli解码器存在远程代码执行漏洞，由于其处理块类型切换时未正确传播校验失败，导致畸形Brotli数据可在解码过程中触发越界写漏洞，进而导致内存破坏并可能以Node.js进程权限执行任意代码，造成服务器失陷。  
  
**0x04 影响版本**  
- Node.js 22.x < 22.23.4  
  
- Node.js 24.x < 24.22.1  
  
- Node.js 26.x < 26.12.1  
  
**0x05 POC状态**  
- 未公开  
  
**0x06 修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://nodejs.org/  
  
  
  
