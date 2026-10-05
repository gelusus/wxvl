#  据称黑客在某论坛出售 Fortinet FortiGate的 1day 漏洞  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-03 16:19  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8LJSS98NBibFIzOwllWZ731QwTLeib7dnt8iaxtDRcbqs1cJbF9YyFtJq1yQtR0b947FYkUSS5c5VodjGoZJG9BEK8CTM4abDrJSU/640?wx_fmt=png&from=appmsg "")  
  
据称，有攻击者正在提供针对 Fortinet FortiGate SSL VPN 设备的私有远程代码执行漏洞利用程序，并声称该漏洞影响 FortiOS 7.2.x 和 7.4.x 版本。该信息尚未经过独立验证，也无法证实存在新的 FortiGate 零日漏洞。  
  
该广告由暗网情报机构分享，宣传的是一个卖家所称的“1day”漏洞，该漏洞具有远程代码执行能力。  
  
据报道，该漏洞利用程序旨在对暴露的 FortiGate SSL VPN 服务进行初始访问。攻击者还声称拥有概念验证视频，并表示可通过私下沟通获取定价信息。  
  
然而，该列表并未列出 CVE 编号，未指明受影响的确切固件版本，未解释是否需要身份验证，也未提供所称漏洞的技术描述。  
  
这些遗漏使得无法确定所声称的漏洞利用是针对未公开的漏洞、较早的已修补漏洞、绕过现有修复程序，还是针对欺诈产品。  
  
黑客出售 Fortinet FortiGate 1day 漏洞  
  
FortiGate 设备仍然是高价值目标，因为它们通常部署在企业网络边缘，以提供防火墙、VPN 和远程访问服务。  
  
此类设备中存在的预身份验证远程代码执行漏洞可能允许攻击者获得初始立足点、建立持久性、窃取凭据、转移到内部网络或部署后续恶意软件。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8LXydRjaVGW6M6FrY5wFsvf0YWajNqKPfQWHbCTfdumia8foJKJwamcTric9BaXwdCtkcfZvING76y0ZCvZA3KdYbty9jCpFCxGQ/640?wx_fmt=png&from=appmsg "")  
  
此外，该声明发布之际，此前披露的 Fortinet 漏洞正持续遭到利用。安全研究人员近期报告了涉及 CVE-2025-25249 的攻击，该漏洞存在于 FortiOS 和 FortiSwitchManager 中，是一个未经身份验证的基于堆的缓冲区溢出漏洞，攻击者可以通过精心构造的请求执行命令。  
  
Fortinet 于 2026 年 1 月发布了补丁，但有报告显示，攻击者于 2026 年 7 月开始在实际操作中利用该漏洞。受影响的 FortiOS 版本分支的修复版本包括 7.4.9 和 7.2.12。  
  
此外，攻击者继续滥用 CVE-2024-21762，这是 FortiOS 和 FortiProxy SSL VPN 组件中的一个严重越界写入漏洞。  
  
该漏洞允许通过精心构造的 HTTP 请求执行未经身份验证的远程代码，并且与最近针对暴露的 FortiGate 系统的入侵有关。  
  
根据暗网情报网站X上的一篇文章， Fortinet此前曾建议企业在无法立即升级的情况下禁用SSL VPN。企业应将此次疑似漏洞利用程序出售事件视为威胁情报线索，而非新发现漏洞的确认。  
  
安全团队应立即清点所有面向互联网的 FortiGate 设备，验证 FortiOS 版本是否受支持且已完全打补丁，尽可能将管理和 VPN 访问限制在受信任的网络，并检查日志中是否存在异常的 SSL VPN 活动。  
  
管理员还应检查是否存在新的管理员帐户、无法解释的配置更改、可疑的 VPN 会话、不熟悉的进程以及来自防火墙设备的出站连接。  
  
由于边缘设备可以提供对内部环境的特权访问，因此一旦怀疑遭到入侵，就应该触发凭证轮换、配置审查和更广泛的事件响应调查。  
  
  
信息来源：C  
yberSecurityNews  
  
