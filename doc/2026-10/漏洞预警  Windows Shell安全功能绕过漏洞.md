#  漏洞预警 | Windows Shell安全功能绕过漏洞  
浅安
                    浅安  浅安安全   2026-10-20 00:00  
  
**0x00 漏洞编号**  
- # CVE-2026-96951  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Windows Shell是Windows的图形界面与文件处理核心组件，负责文件关联、Shell扩展、快捷方式、设置文件解析和应用程序启动等。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SVleaDeU1ibPickZJzpKIF4Mcm9iaHXXSDJfzdooHoG4ZA4iaHupxCYLp8HtE2qPLEqYibUd5u3E3Nmiczw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-96951**  
  
**漏洞类型：**  
安全功能绕过  
  
**影响：**  
执行任意命令  
  
**简述：**  
Windows Shell存在安全功能绕过漏洞，由于其在处理命令中的特殊元素时存在命令注入缺陷。攻击者可通过邮件、即时消息或网页投递特制的.SettingContent-ms/.appcontent-ms文件，诱导用户打开。由于该文件类型未被Windows识别为危险类型，文件即使带有Mark of the Web标记，也不会弹出安全警告，成功利用可在当前用户上下文中执行任意命令，并绕过SmartScreen、Office Protected View等依赖MOTW的安全控制。  
  
**0x04 影响版本**  
- Windows 10 1607 < 10.0.14393.9612  
  
- Windows 10 1809 < 10.0.17763.9376  
  
- Windows 10 21H2 < 10.0.19044.7807  
  
- Windows 10 22H2 < 10.0.19045.7807  
  
- Windows 11 23H2 < 10.0.22631.7652  
  
- Windows 11 24H2 < 10.0.26100.9711  
  
- Windows 11 25H2 < 10.0.26200.9711  
  
- Windows 11 26H1 < 10.0.28000.3231  
  
- Windows 11 26H2 < 10.0.26300.9711  
  
- Windows Server 2012 R2 < 6.3.9600.23470  
  
- Windows Server 2016 < 10.0.14393.9612  
  
- Windows Server 2019 < 10.0.17763.9376  
  
- Windows Server 2022 < 10.0.20348.5761  
  
- Windows Server 2025 < 10.0.26100.33650  
  
- Outlook.com < 20260925.100  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://msrc.microsoft.com/update-guide/zh-cn/vulnerability/  
CVE-2026-96951  
  
  
  
