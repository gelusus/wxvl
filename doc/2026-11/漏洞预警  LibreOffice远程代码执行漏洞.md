#  漏洞预警 | LibreOffice远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-11-26 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-104692  
  
- # CVE-2026-104007  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
LibreOffice是一款免费开源的办公套件，由The Document Foundation维护，提供文字处理、电子表格、演示文稿、数据库和绘图等完整功能。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NQlfTO30MhzYiaFXhQcjZfQ0nIiaprvtziafibU0vxpNicAxQXXhUvm4XMOAol5cFdGPqCMnHicVNzEoicLHpznNtxVZy0gFDfoc7Xhg35h9Eaibej8/640?wx_fmt=png&from=appmsg "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-104692**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意代码  
  
**简述：**  
LibreOffice存在远程代码执行漏洞，由于其HSQLDB数据库引擎在通过通用JDBC驱动使用时，未能正确强制执行Java方法白名单机制，导致SQL语句可调用任意Java方法，进而导致可执行任意命令，从而完全控制系统。  
  
**CVE-2026-104007**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意代码  
  
**简述：**  
LibreOffice存在远程代码执行漏洞，由于数据库文档中的JDBC驱动路径可被恶意文档控制，且在通过表单数据源外部引用时绕过了属性剥离过滤。攻击者可构造特制的ODS电子表格文件，利用表单自动加载、HTTP外部ODB下载、SQL转义绕过等特性，在用户仅打开文档时实现任意代码执行，从而完全控制系统。  
  
**0x04 影响版本**  
- 26.2 <= LibreOffice < 26.2.7  
  
- 26.8 <= LibreOffice < 26.8.2  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.libreoffice.org/  
  
  
  
