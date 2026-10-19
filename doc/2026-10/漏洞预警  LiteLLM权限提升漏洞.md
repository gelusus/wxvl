#  漏洞预警 | LiteLLM权限提升漏洞  
浅安
                    浅安  浅安安全   2026-10-18 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-35029  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
LiteLLM是一个创新的代理和库，旨在简化多种大型语言模型API的集成过程。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXHFbO3xcOeRiaK3sbZrDm7oyfyl8QVlPicuib8zgT4bzh0opHnqK9UemFKnXTOMEjdcePpxiacPGKSAQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-35029**  
  
**漏洞类型：**  
权限提升  
  
**影响：**  
执行任意代码  
  
**简述：**  
LiteLLM存在权限提升漏洞，由于其/config/update端点未能正确强制执行管理员角色授权，已认证的低权限用户可利用该漏洞修改代理配置及环境变量，注册指向攻击者控制Python代码的自定义端点，进而实现远程代码执行。  
  
**0x04 影响版本**  
- LiteLLM < 1.83.0  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.litellm.ai/  
  
  
  
