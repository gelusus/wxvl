#  VMware ESX 和 vCenter 的严重漏洞使攻击者能够绕过身份验证并执行任意代码  
原创 ZM
                    ZM  暗镜   2026-11-26 23:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zdwoicOrrJb0WaPsOqLY7eHEStmrT5SnCe3Tg6zmbhYdn910xnMByHenojiaUcbMBxW9x6Wktdm6KePViaq6GePbGJp0ibXDtgjPoB6FwCW9C5w/640?wx_fmt=png&from=appmsg "")  
  
博通公司于 11 月 24 日发布的安全公告 VMSA-2026-0011 列出了 VMware 产品中的 23 个漏洞，其中包括一个严重的 vCenter 身份验证绕过漏洞，该漏洞只需要对受影响的实例进行网络访问即可绕过。  
  
vCenter 的另一个漏洞可能允许拥有备份存储库访问权限的攻击者以 root 用户身份写入文件，从而可能导致设备上的任意代码执行。这两个漏洞的先决条件不同，而且安全公告摘录并未说明它们可以相互关联。该安全公告对 CVSS v3 基本评分的取值范围为 3.4 到 9.8。它涵盖 VMware ESX、vCenter、Workstation、Fusion、Tools 和 Avi Load Balancer，以及 VMware Cloud Foundation、vSphere Foundation、Telco Cloud Platform 和 Telco Cloud Infrastructure。现有摘录详细列出了七个 vCenter 漏洞，但并未说明 ESX 漏洞的影响。此前VMware ESXi 和 Workstation 的代码执行缺陷涉及其他漏洞，不应被视为此次版本中主机被入侵的证据。  
  
CVE-2026-123903 是摘录中评分最高的漏洞，它影响 VMware Directory Service（也称为 vmdir）。可通过网络访问的攻击者可以绕过身份验证，未经授权访问 vCenter。该漏洞评分为 9.8 分，表明利用该漏洞既不需要现有权限，也不需要用户交互。博通公司感谢三组研究人员独立报告了该问题。摘录内容并未说明攻击者是否已在实际环境中利用该漏洞。  
  
此次披露是在CISA发布关于另一个vCenter身份验证绕过漏洞CVE-2026-123890的警告之后进行的，该漏洞已被报告为正在被积极利用。这两个漏洞都影响VMware目录服务，但之前的攻击活动并未证实CVE-2026-123903已被利用。管理员在审查漏洞暴露情况和补丁状态时，应分别跟踪这两个漏洞标识符。  
  
CVE-2026-123904，评级为7.5，会影响vCenter处理某些归档文件的方式。拥有备份存储库访问权限的攻击者可以以root权限将文件写入任意位置，这可能导致设备上的代码执行。这一访问要求将该漏洞与可通过网络访问的身份验证绕过机制区分开来。它还使得存储库权限和访问日志与事件分类相关，而不仅仅是与设备的网络暴露情况相关。  
  
另外三个 vmdir 漏洞（CVE-2026-123905 至 CVE-2026-123907）的评分最高为 7.5，可能导致目录服务中断。CVE-2026-123908 的评分为 6.7，允许已攻破 vpxd 服务帐户的攻击者提升至 root 权限。  
  
CVE-2026-123909，评级为 5.2，涉及 vSphere 分布式交换机策略执行失败，这可能导致虚拟机内部的参与者使用策略应该限制的功能进行操作。博通建议客户参考安全公告响应矩阵中列出的修复版本。但并未提供针对 vCenter 问题的详细解决方法。由于提供的摘录中不包含该矩阵，因此管理员在安排升级之前，应核实每个已部署产品和发布分支的目标版本。博通还针对 vCenter 问题提供了一份补充安全公告常见问题解答。  
  
  
