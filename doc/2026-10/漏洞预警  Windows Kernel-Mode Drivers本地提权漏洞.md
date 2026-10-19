#  漏洞预警 | Windows Kernel-Mode Drivers本地提权漏洞  
浅安
                    浅安  浅安安全   2026-10-18 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-83834  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
mskssrv.sys是Windows操作系统的内核流服务驱动。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SVleaDeU1ibPickZJzpKIF4Mcm9iaHXXSDJfzdooHoG4ZA4iaHupxCYLp8HtE2qPLEqYibUd5u3E3Nmiczw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-83834**  
  
**漏洞类型：**  
本地提权  
  
**影响：**  
提升权限  
  
**简述：**  
Windows内核存在本地提权漏洞，具备本地低权限账户的攻击者可通过构造特定流数据触发UAF条件，结合TOCTOU竞态实现内核代码执行，进而获取高级权限。  
  
**0x04 影响版本**  
- Windows Server 2016  
  
- Windows 11 Version 26H2 for ARM64-based Systems  
  
- Windows 11 Version 26H2 for x64-based Systems  
  
- Windows 11 Version 25H2 for ARM64-based Systems  
  
- Windows 10 Version 21H2 for 32-bit Systems  
  
- Windows Server 2025 (Server Core installation)  
  
- Windows 10 Version 1607 for x64-based Systems  
  
- Windows Server 2016 (Server Core installation)  
  
- Windows 10 Version 1607 for 32-bit Systems  
  
- Windows 11 version 26H1 for x64-based Systems  
  
- Windows Server 2025  
  
- Windows 11 Version 26H1 for ARM64-based Systems  
  
- Windows 11 Version 24H2 for x64-based Systems  
  
- Windows 11 Version 23H2 for ARM64-based Systems  
  
- Windows 11 Version 24H2 for ARM64-based Systems  
  
- Windows 11 Version 23H2 for x64-based Systems  
  
- Windows 11 Version 25H2 for x64-based Systems  
  
- Windows 10 Version 22H2 for 32-bit Systems  
  
- Windows Server 2022  
  
- Windows 10 Version 22H2 for ARM64-based Systems  
  
- Windows 10 Version 22H2 for x64-based Systems  
  
- Windows 10 Version 21H2 for x64-based Systems  
  
- Windows Server 2022 (Server Core installation)  
  
- Windows 10 Version 21H2 for ARM64-based Systems  
  
- Windows 10 Version 1809 for x64-based Systems  
  
- Windows 10 Version 1809 for 32-bit Systems  
  
- Windows Server 2019 (Server Core installation)  
  
- Windows Server 2019  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://msrc.microsoft.com/update-guide/zh-cn/vulnerability/  
CVE-2026-  
83834  
  
  
  
