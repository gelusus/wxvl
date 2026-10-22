#  漏洞预警 | Cisco Secure Email Gateway路径遍历漏洞  
浅安
                    浅安  浅安安全   2026-10-21 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-76519  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Cisco Secure Email Gateway是商业邮件安全网关产品，运行Cisco AsyncOS软件，用于对进出邮件进行过滤、防病毒与内容安全处理。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SU7yiaA9KHbxNq1iaCtatkAMRdMxh9GQW5qsybAMVCfgV1icAvzV2Y7tKZDV9WTF3l0dMcaKaNR0jllQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-76519**  
  
**漏洞类型：**  
路径遍历  
  
**影响：**  
执行任意命令  
  
**简述：**  
Cisco Secure Email Gateway高级恶意软件防护功能存在路径遍历漏洞，未认证的远程攻击者只需发送一封带特制附件的邮件，即可写入任意文件并以root权限执行命令。  
  
**0x04 影响版本**  
- 15.5 <= Cisco AsyncOS < 15.5.5-014  
  
- Cisco AsyncOS 16.0  
  
- 16.5 <= Cisco AsyncOS < 16.5.0-780  
  
**0x05****POC状态**  
- 未公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.cisco.com/  
  
  
  
