#  漏洞预警 | 泛微Ecology远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-11-05 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-22679  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
泛微协同管理应用平台是一套兼具企业信息门户、知识文档管理、工作流程管理、人力资源管理、客户关系管理、项目管理、财务管理、资产管理、供应链管理、数据中心功能的企业大型协同管理平台。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/7stTqD182SX9lUEvt3ekNQY5a2YFI3YdXum7T9DHRWW1yCibhTq4ia7dnaWVlTog0nv6gkEa6SNDbo0AUYXwsOTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-22679**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意命令  
  
**简述：**  
泛微e-cology的/papi/esearch/data/devops/dubboApi/debug/method接口处存在远程代码执行漏洞，未经过验证的攻击者可构造特制POST请求即可触发该漏洞，从而在目标服务器上执行任意操作系统命令。  
  
**0x04 影响版本**  
- 泛微e-cology 10.0 < 20260312  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.weaver.com.cn/  
  
  
  
