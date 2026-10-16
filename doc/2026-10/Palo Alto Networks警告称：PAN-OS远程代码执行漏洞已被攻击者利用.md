#  Palo Alto Networks警告称：PAN-OS远程代码执行漏洞已被攻击者利用  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-16 02:11  
  
![](https://mmbiz.qpic.cn/mmbiz_png/vO1zY1O9p8IZZITRUzMw4rzmyY2WicxZcOslLZRojwqmdDh6eysQiccv3hAQQ0pzngYlicIKpGibicrG698a6ntRBUACXv7sl66V2vHsXNSHF7os/640?wx_fmt=png&from=appmsg "")  
  
Palo Alto Networks 发布了针对高危 PAN-OS RCE 零日漏洞（编号为 CVE-2026-0322）的安全更新，该漏洞已被用于针对防火墙的攻击。  
  
该漏洞影响 PAN-OS 管理接口，允许已通过身份验证的攻击者利用网络访问权限在易受攻击的设备上以 root 身份执行代码。  
  
Palo Alto Networks在周三发布的一份安全公告中证实，他们已经发现该漏洞被有限地利用于攻击。  
  
该公司表示：“目前尚未发现针对 Palo Alto Networks 管理界面的攻击行为。”  
  
该漏洞的严重性评分为 8.6，会影响 PA 系列和 VM 系列防火墙，以及 Panorama 虚拟设备和 M 系列设备。Palo Alto Networks 表示，Cloud NGFW 和 Prisma Access 不受影响。  
  
要利用此漏洞，攻击者需要现有的凭据和对 PAN-OS 管理界面的网络访问权限。  
  
因此，Palo Alto Networks 建议仅允许受信任的内部 IP 地址访问管理界面。  
  
该公司已发布针对受影响的 PAN-OS 10.2、11.1、11.2、12.1 和 12.2 版本的修复程序，预计将于 10 月 15 日发布针对 PAN-OS 11.2.16 的修复程序。  
  
使用较旧、不受支持版本的用户必须升级到受支持的版本。  
  
Palo Alto 表示，目前尚无针对此漏洞的已知解决方法，但拥有威胁防护订阅的客户可以启用签名，从而针对已知的攻击模式提供有限的保护。  
  
Palo Alto 警告说：  
“威胁防御覆盖范围有限，这意味着虽然可以使用当前的威胁 ID 识别已知的攻击模式，但可能存在目前无法检测到的变种”。  
  
“如果此 CVE 和暴露所需配置影响到您的环境，我们建议您升级到不受影响的版本。”  
  
该漏洞是由 Palo Alto Networks 的高级首席产品安全研究员 Suraj Siddharudh 在公司内部发现的。  
  
该公司还公布了三个利用该漏洞的 IP 地址：66.42.72[.]14、208.83.239[.]114 和 149.248.47[.]234。  
  
BleepingComputer 联系了 Palo Alto Networks，以了解攻击何时开始，幕后黑手是谁，以及攻击者是否攻破了防火墙或利用防火墙访问了内部网络。  
  
美国网络安全和基础设施安全局 (CISA) 已将 CVE-2026-0322 添加到已知利用漏洞目录中，并要求联邦民事机构在 10 月 17 日之前缓解该漏洞。  
  
  
信息来源：  
BleepingComputer  
  
