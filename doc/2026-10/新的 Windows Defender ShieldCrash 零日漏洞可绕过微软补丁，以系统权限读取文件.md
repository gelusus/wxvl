#  新的 Windows Defender ShieldCrash 零日漏洞可绕过微软补丁，以系统权限读取文件  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-04 16:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8IPtYWialqicOMSKVgUiabdsaCicFib2bGcCKpUC6tfibqjqm2SPre70bGHjXAXoaaNmicINNQHicWYqPfhyP1LNSGTluW2REh8vXSBCiaI/640?wx_fmt=png&from=appmsg "")  
  
研究人员 MSNightmare 最新发布的 ShieldCrash 概念验证声称，尽管微软此前已修复了 ShieldBreak漏洞（编号为 CVE-2026-69414），但 Microsoft Defender 仍然容易受到任意文件读取漏洞的攻击。  
  
研究人员表示，该问题可能使本地攻击者能够让 Defender 在完全更新、受支持的 Windows 系统上以系统级权限读取文件。  
  
根据 MSNightmare 的说法，微软虽然修复了 ShieldBreak 漏洞的几个方面，但仍留下了一条特定的攻击路径。在特定条件下，这条剩余的攻击路径据称会重现先前漏洞的核心安全影响。  
  
报告显示，此次事件的影响十分显著，因为 SYSTEM 帐户的权限比普通用户和大多数管理员帐户都要广泛。Windows 服务、安全软件组件和受保护的操作系统进程通常都在 SYSTEM 帐户下运行。  
  
如果攻击者能够强制 Defender 组件访问受保护的文件并暴露其内容，他们可能会获得其现有帐户不应该访问的敏感数据。  
## Windows Defender ShieldCrash 零日漏洞  
  
可能泄露的数据包括应用程序配置文件、凭据相关材料、安全产品设置、私钥、浏览器或服务密钥，或者属于其他 Windows 用户的文件。  
  
具体影响取决于攻击者可以攻击哪些文件，他们是否能够可靠地恢复文件内容，以及攻击者在发起攻击之前已经拥有哪些权限。  
  
现有的概念验证被描述为结构实现，而不是完整的 SYSTEM 权限提升漏洞利用。  
  
研究人员表示，该漏洞演示了在2026 年 9 月 Windows 安全更新后，攻击者可以以 SYSTEM 权限任意读取文件，并指出稍后可能会发布更完整的概念验证。读取文件并不意味着可以运行代码或系统命令，但仍然会削弱 Windows 的安全性。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/vO1zY1O9p8KcayiaL7MRa69o5FtHM5n3yNjcqSaPrbAKzvVcMnvF4Z2OwNYia2BA9cK6KibHiaV285u0I3hhJicJd8ia5SMibD9lCsxqt1Xq5b5siac/640?wx_fmt=png&from=appmsg "")  
  
ShieldCrash 代码库包含 C++ 项目文件、名为 Warden.dll 的 DLL 文件、资源文件以及 EICAR 测试存档。EICAR 文件表明，该研究可能涉及 Defender 的恶意软件检测或文件处理工作流程。  
  
但是，组织应避免在生产终端上运行不受信任的公共概念验证代码，尤其是与防病毒服务或特权 Windows 组件交互的代码。  
  
GitHub ShieldCrash PoC 声称，微软针对 ShieldBreak (CVE-2026-69414) 的修复未能完全解决根本问题，允许在已打补丁的 Windows 系统上以 SYSTEM 身份读取任意文件。  
  
微软尚未公开证实这一新的绕过方法，目前这仍只是研究人员的报告，尚待独立复现或微软发布安全公告。此前的漏洞编号为 CVE-2026-69414，而新的绕过方法尚未获得单独的 CVE 编号。  
  
防御者应监控端点，以发现与 Microsoft Defender 扫描路径交互的可疑本地工具、未签名 DLL 的意外创建或加载、涉及受保护文件的异常访问尝试，以及与 Defender 服务相关的子进程或文件操作。  
  
安全团队还应保持 Microsoft Defender 平台和情报更新为最新状态，及时应用未来的 Microsoft 补丁，并通过应用程序控制策略限制不受信任的代码执行。  
  
  
信息来源：C  
yberSecurityNews  
  
