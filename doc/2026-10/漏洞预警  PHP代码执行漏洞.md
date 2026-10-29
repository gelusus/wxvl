#  漏洞预警 | PHP代码执行漏洞  
浅安
                    浅安  浅安安全   2026-10-28 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-115560  
  
**0x01 危险等级**  
- 中危  
  
**0x02 漏洞概述**  
  
PHP是一种广泛使用的开源服务器端脚本语言，主要运行于Web服务器端。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30Mhy3sACcIQ5TKia2gqkAib2Rjrw6cp2gvZW51KdebWYfpFO9CNicePzuhq48GqjXVh5ibKclxyiaQiadMF8R43q6hrEdQiaAMia9qr0UvL8/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-115560**  
  
**漏洞类型：**  
代码执行  
  
**影响：**  
执行任意代码  
  
**简述：**  
PHP存在代码执行漏洞，当启用会话上传进度功能，且应用代码中同时采用特殊的会话序列化配置与运行时切换方式时，未经验证的攻击者即可通过该漏洞执行任意命令，进而获取权限。  
  
**0x04 影响版本**  
- 暂无  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方暂未发布漏洞修复版本，建议用户关注官网动态****：**  
  
https://www.php.net/  
  
  
  
