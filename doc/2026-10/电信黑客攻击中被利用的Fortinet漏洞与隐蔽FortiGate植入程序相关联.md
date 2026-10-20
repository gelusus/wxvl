#  电信黑客攻击中被利用的Fortinet漏洞与隐蔽FortiGate植入程序相关联  
爱拍照的老李
                    爱拍照的老李  爱拍照的老李   2026-10-20 01:04  
  
**导****读**  
  
  
  
Palo Alto Networks 报告称，一名疑似受国家支持的威胁行为者在针对某电信机构的攻击中利用了 Fortinet 的多个漏洞。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/PaFY6wibdwyJHB5rTpXzXfuzHiaUvatyTC6tCcqtYdhibd8mNH28CTI12ey8etY2293Xjibljm4SDK2h76MzqQeyuhZNP1hYeF3wpgGMSV9hKNY/640?wx_fmt=webp&from=appmsg "")  
  
  
此次归因于UNC5820（Palo Alto Networks将其标记为CL-STA-0954）的入侵事件发生于2024年9月，导致攻击者得以长期访问受害者的系统环境。  
  
  
最初的访问权限是通过利用 FortiManager 中的一个关键远程代码执行（RCE）漏洞获得的，该漏洞编号为 CVE-2024-47575，并于 2024 年 10 月作为零日漏洞被修复。  
  
  
补丁推出后不久，Mandiant 警告称，UNC5820 自至少 2024 年 6 月起就一直在利用这一被命名为 FortiJump 的漏洞。  
  
  
Palo Alto Networks 发布的一份最新报告显示，作为同一入侵事件的一部分，UNC5820 还利用了 CVE-2025-61624——这是一个影响多款 Fortinet 产品的路径穿越漏洞。  
  
  
该安全缺陷影响 FortiOS、FortiPAM、FortiProxy 和 FortiSwitchManager，并已于2026年4月完成修复，它可能让经过身份验证的攻击者写入或删除任意文件。Fortinet 在该漏洞披露时就将其标记为零日漏洞。  
  
  
周五，美国网络安全机构赛博安全与基础设施安全局（CISA）将CVE-2025-61624加入其已知被利用漏洞（KEV）列表，敦促联邦机构立即对其进行补丁修复。  
  
  
UNC5820 似乎将路径遍历漏洞作为零日漏洞加以利用，以部署一款此前不为人知的名为 PeerStitch 的 FortiGate 后门程序。  
  
  
这款ARM64恶意软件驻留在内存中，避免打开监听端口或发起出站网络连接，并通过设备上合法进程产生的流量进行通信。  
  
  
在此次被观察到的攻击中，UNC5820 利用 FortiJump 访问了 FortiManager 中央实例，注册了一个恶意的虚拟 FortiGate 设备，并窃取了一份配置归档文件，该文件包含了 FortiManager、500 多台受管理的 FortiGate 防火墙以及受害者网络的相关详细信息。  
  
  
UNC5820利用CVE-2025-61624获取了底层防火墙操作系统的root权限，并投放了一个脚本以启动PeerStitch部署。  
  
  
该植入程序由C语言编写，是一款“一次性进程内存补丁注入器”，针对FortiOS中的中央系统与流量日志模块以及IKE/IPsec VPN服务守护程序，在它们之间构建一条隐蔽的内存驻留通信桥梁，同时绕过FortiOS的默认安全验证机制。  
  
  
作为攻击的一部分，还发现威胁行为者在 FortiGate 到 FortiManager（FGFM）通信中冒充 FortiManager，从外部基础设施发出命令，并“采取了大量措施来消除法证痕迹”。  
  
  
据 Palo Alto Networks 称，UNC5820 要么通过利用 CVE-2024-26008 终止正版 FortiManager 的会话，要么通过复用泄露的密钥并以合法 FortiManager 的身份进行身份验证，从而声明了管理身份。  
  
  
“PeerStitch 植入程序的发现表明边缘设备恶意软件的技术成熟度不断提升。Palo Alto Networks 指出，攻击者通过将其作为一次性注入器运行、避免留下持久的磁盘痕迹，并在进程内存内部构建隐蔽的进程间通信桥接，实现了持续的持久化访问，同时避开了传统监控手段。”  
  
  
新闻链接：  
  
https://www.securityweek.com/exploited-fortinet-bug-linked-to-stealthy-fortigate-implant-in-telecom-hack/  
  
****  
扫码关注  
  
爱拍照的老李  
  
**讲述普通人能听懂的安全故事**  
  
  
