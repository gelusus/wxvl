#  漏洞预警 | Magento/Adobe Commerce模板注入漏洞  
浅安
                    浅安  浅安安全   2026-11-01 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-75650  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Adobe Magento Open Source是Adobe公司的一套开源的PHP电子商务系统，Magento Open Source提供所有基本的商务功能，可用于从头开始建立独特的线上商店。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXaqpBYiaDTgZPe9XeqSxh7yOLFz89okSbeNCBvI3EH8Uib9OBmh9gZnjOuukml79kZ7pXrfwQsicNnw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-75650**  
  
**漏洞类型：**  
模板注入  
  
**影响：**  
执行任意命令  
  
**简述：**  
Magento/Adobe Commerce存在模板注入漏洞，由于模板渲染环节对用户可控输入过滤不足，导致模板表达式被解析执行。未经验证的攻击者可通过构造恶意模板内容注入指令，进而执行任意代码。  
  
**0x04 影响版本**  
- 2.4.6 <=   
Magento Open Source <=  2.4.9  
  
- 2.4.4 <= Adobe Commerce <= 2.4.9  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.adobe.com/  
  
  
  
