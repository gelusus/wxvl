#  漏洞预警 | Apache Ofbiz远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-11-17 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-83542  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Apache OFBiz是Apache的一套企业资源计划系统，提供了一整套基于Java的Web应用程序组件和工具。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SWb8d466BsKYMQRW3jrCjGvEV4QWVVDKTqwx3KR0YI4OpBchpucKVnUt3vic275ibHTInsxFWehWFDA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-83542**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意代码  
  
**简述：**  
Apache OFBiz存在远程代码执行漏洞，由于系统在处理JSON请求时，未对请求体中的属性进行有效校验，导致攻击者能够覆盖应用程序原有的Widget装饰器配置，并通过构造恶意XML文件触发Groovy表达式执行。未经身份验证的远程攻击者可利用Multipart文件上传及进程文件描述符等机制，将恶意XML加载至Widget引擎执行，进而执行任意操作。  
  
**0x04 影响版本**  
- Apache OFBiz < 24.09.08  
  
**0x05****POC状态**  
- 未公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://ofbiz.apache.org/  
  
  
  
