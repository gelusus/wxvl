#  漏洞预警 | Windows健康与优化体验服务权限提升漏洞  
浅安
                    浅安  浅安安全   2026-10-21 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-85884  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Windows健康与优化体验服务是Windows 11内置的系统服务，负责监控设备的CPU负载、温度、电源与电池等运行状况，跟踪系统组件、驱动与应用的稳定性，收集非个人诊断数据并协助系统进行后台资源调度与优化。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SVleaDeU1ibPickZJzpKIF4Mcm9iaHXXSDJfzdooHoG4ZA4iaHupxCYLp8HtE2qPLEqYibUd5u3E3Nmiczw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-85884**  
  
**漏洞类型：**  
权限提升  
  
**影响：**  
获取最高权限  
  
**简述：**  
Windows健康与优化体验服务存在权限提升漏洞，由于其在处理Windows诊断Lua工作流事件时缺少认证，具备诊断能力的低权限代码可伪造包含攻击者可控Lua模块路径的ETW工作流事件，该事件被以LocalSystem权限运行的whesvc服务消费后，服务将通过WinDiag Lua引擎加载并执行攻击者指定的Lua模块，攻击者由此可在目标主机上将权限提升至SYSTEM。  
  
**0x04 影响版本**  
- Windows 11 24H2 < 10.0.26100.9711  
  
- Windows 11 25H2 < 10.0.26200.9711  
  
- Windows 11 26H1 < 10.0.28000.3231  
  
- Windows 11 26H2 < 10.0.26300.9711   
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://msrc.microsoft.com/update-guide/zh-cn/vulnerability/  
CVE-2026-  
85884  
  
  
  
