#  Palo Alto Networks 警告：黑客利用了 Fragnesia 漏洞攻击了 PAN-OS 系统  
Rhinoer
                    Rhinoer  犀牛安全   2026-11-27 16:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/vO1zY1O9p8IJpXMHTBNKkBJumjdPl2eGXr6v6N8nncIDdGmO2g4wsibh0gic0Pcdlau5U6a5KvmqngDdH74MJxwShmthHMeROrgyvCibRYzZpM/640?wx_fmt=png&from=appmsg "")  
  
攻击者正在串联利用PHP2Shell和Fragnesia漏洞，针对Palo Alto Networks设备，这些设备管理接口暴露于互联网。  
  
PHP2Shell 是指两个关键的 PHP 漏洞，编号为 CVE-2026-109618 和 CVE-2026-109619，这些漏洞允许未经认证的攻击者以 PHP 网络工作者的权限远程运行代码。  
  
Fragnesia 漏洞，编号为 CVE-2026-46300，是 XFRM ESP-in-TCP 子系统中的一个 Linux 内核漏洞，允许低权限的本地攻击者将权限提升到 root 级别。  
  
CISA 于 10 月 28 日发布的警报警告称，这两项安全问题正被串联起来，用于在易受攻击的系统上部署 PHP webshell，并以 root 身份执行命令。  
  
大约在同一时间，Dream Security的研究人员报告称，他们发现了一场使用同一链条的活动，目标是美国、日本、台湾和韩国的服务器，包括一家美国政府附属机构。  
  
Palo Alto Networks 今天在一份通告中确认，这类攻击同样针对运行 PAN-OS 的防火墙。  
  
安全公告写道：已观察到针对暴露于互联网的Palo Alto Networks管理接口遭到CVE-2026-46300的利用。  
  
这家网络公司表示，其观察到的所有利用活动都涉及将管理界面暴露在公共互联网中的 PAN-OS 设备，这与其推荐的“最佳实践”相悖。  
  
Palo Alto Networks 没有提供关于 PHP2Shell 漏洞串接的技术细节，但指出这是目前已知唯一可以利用 Fragnesia 的方法，而且他们并不知道该漏洞还有其他可利用的路径。  
  
公司已为 PAN-OS 10.2、11.1、11.2、12.1 和 12.2 发布了 Fragnesia 的修复，具体的补丁版本因版本而异。  
  
鉴于这两个漏洞是同时被利用的，应用了修复 PHP2Shell 漏洞的更新的管理员应当免受这些攻击。  
  
除了应用安全更新外，Palo Alto Networks 还建议需要保留远程管理接口的用户限制对受信任内部 IP 地址的访问。  
  
  
信息来源：BleepingComputer  
  
