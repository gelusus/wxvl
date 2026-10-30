#  漏洞预警 | Cisco FMC反序列化漏洞  
浅安
                    浅安  浅安安全   2026-10-29 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-20131  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Cisco Secure Firewall Management Center是一款用于集中管理和配置Cisco Secure Firewall产品的安全管理平台。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SU7yiaA9KHbxNq1iaCtatkAMRdMxh9GQW5qsybAMVCfgV1icAvzV2Y7tKZDV9WTF3l0dMcaKaNR0jllQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-20131**  
  
**漏洞类型：**  
反序列化  
  
**影响：**  
执行任意命令  
  
**简述：**  
Cisco FMC存在反序列化漏洞，未经验证的攻击者可通过该漏洞向受影响接口提交恶意序列化数据，从而执行任意代码，并以root权限控制设备。  
  
**0x04 影响版本**  
- 6.4.0.13 <= FMC <= 6.4.0.18  
  
- 7.0.0 <= FMC <= 7.0.0.1  
  
- 7.0.1 <= FMC <= 7.0.1.1  
  
- 7.0.2 <= FMC <= 7.0.2.1  
  
- 7.0.3 <= FMC <= 7.0.5  
  
- 7.0.6 <= FMC <= 7.0.6.3  
  
- FMC 7.0.7  
  
- 7.0.8 <= FMC <= 7.0.8.1  
  
- 7.1.0 <= FMC <= 7.1.0.3  
  
- 7.2.0 <= FMC <= 7.2.0.1  
  
- 7.2.1 <= FMC <= 7.2.2  
  
- 7.2.3 <= FMC <= 7.2.3.1  
  
- 7.2.4 <= FMC <= 7.2.4.1  
  
- 7.2.5 <= FMC <= 7.2.5.2  
  
- 7.2.6 <= FMC <= 7.2.7  
  
- 7.2.8 <= FMC <= 7.2.8.1  
  
- FMC 7.2.9  
  
- 7.2.10 <= FMC <= 7.2.10.2  
  
- FMC 7.3.0  
  
- 7.3.1 <= FMC <= 7.3.1.2  
  
- FMC 7.4.0  
  
- 7.4.1 <= FMC <= 7.4.1.1  
  
- 7.4.2 <= FMC <= 7.4.2.4  
  
- 7.4.3 <= FMC <= 7.4.5  
  
- 7.6.0 <= FMC <= 7.6.1  
  
- 7.6.2 <= FMC <= 7.6.2.1  
  
- FMC 7.6.4  
  
- FMC 7.7.0  
  
- 7.7.10 <= FMC <= 7.7.10.1  
  
- FMC 7.7.11  
  
- FMC 10.0.0  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.cisco.com/  
  
  
  
