#  漏洞预警 | Windows Update Stack本地权限提升漏洞  
浅安
                    浅安  浅安安全   2026-11-05 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-81963  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Windows Update Stack是Windows操作系统负责更新下载、暂存、编排与安装的核心组件集合，主要包括 Windows Update Orchestrator服务、Windows Update Medic服务、更新会话编排器以及 MoUsoCoreWorker.exe、usoclient.exe等工作进程。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SVleaDeU1ibPickZJzpKIF4Mcm9iaHXXSDJfzdooHoG4ZA4iaHupxCYLp8HtE2qPLEqYibUd5u3E3Nmiczw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-81963**  
  
**漏洞类型：**  
权限提升  
  
**影响：**  
获取最高权限  
  
**简述：**  
Windows Update Stack存在本地权限提升漏洞，由于Windows更新栈在处理文件路径时未正确解析重解析点，即在访问文件前未对链接目标做充分校验，导致以SYSTEM权限运行的更新组件可能被诱导对攻击者指定的文件执行特权操作。本地低权限攻击者利用该漏洞可将权限提升至SYSTEM。  
  
**0x04 影响版本**  
- Windows 11 23H2（x64/ARM64）< 10.0.22631.7582  
  
- Windows 11 24H2（x64/ARM64）< 10.0.26100.9445  
  
- Windows 11 25H2（x64/ARM64）< 10.0.26200.9445  
  
- Windows 11 26H1（x64/ARM64）< 10.0.28000.2954  
  
- Windows Server 2025（含Server Core安装）< 10.0.26100.33438  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://msrc.microsoft.com/update-guide/zh-cn/vulnerability/  
CVE-2026-81963  
  
  
  
