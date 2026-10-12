#  Veeam Agent 漏洞被积极利用，以获取 Windows 系统权限  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-12 16:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8IVZtpqMdUYc2esRRlLWFOJL5AoPGz1f3CzrlV6E1Eaic7rrypMtOibtJxl4Iiafd2InZUXKSUu0dEsFAXjCgeLM6oQicqibVPFfzE8/640?wx_fmt=png&from=appmsg "")  
  
Veeam Agent for Microsoft Windows 中的一个严重本地权限提升漏洞在公开的概念验证漏洞利用代码发布后引起了人们的关注。  
  
该漏洞编号为 CVE-2026-32996，可能允许低权限的本地用户在受影响的 Windows 设备上以 NT AUTHORITY\SYSTEM 权限运行命令。  
  
技术细节和漏洞利用代码已于2026年9月14日公开，这增加了攻击者将此漏洞纳入入侵后攻击链的风险。该漏洞影响Veeam Agent for Microsoft Windows 13.0.1.2067及更早的13版本。  
  
问题存在于 Veeam Endpoint Backup 服务中，该服务通过位于 \\.\pipe\Veeam\VAW\ServiceConnectionPipe 的本地 gRPC 命名管道处理特权客户端活动。Veeam Agent 漏洞已被积极利用  
  
GitHub 研究员 suce0155 发现，该服务将提升的管理员身份存储在客户端控制的会话 UID 下，而该会话 UID 与请求用户或原始管道连接没有安全关联。  
  
这意味着本地攻击者可以重用提升权限的会话标识符，并诱使 Veeam 服务以 SYSTEM 级权限运行命令。据报道，所需的会话标识符可以从 C:\ProgramData\Veeam\Endpoint\ 目录下的 Svc.VeeamEndpointBackup.log 文件中恢复。  
  
由于普通本地用户可以读取此日志文件，因此具有有限访问权限的攻击者可以搜索其中的有效 UID，然后将该标识符传递给暴露的服务接口。  
  
公开的漏洞利用代码通过在 Veeam 日志文件中找到有效的 GUID 并执行Windows whoami 命令来演示此问题。命令输出被写入一个文件，证实该进程以 NT AUTHORITY\SYSTEM 帐户运行。  
  
虽然利用漏洞需要本地访问权限，但攻击者通常通过网络钓鱼、窃取凭据、恶意软件感染或被攻破的远程访问帐户来获取低级别访问权限。在大多数 Windows 系统中，SYSTEM 权限代表最高权限级别。  
  
成功的攻击者可以利用此访问权限关闭安全工具、访问受保护的数据、修改系统配置、建立持久性、窃取凭据或在企业环境中横向移动。  
  
Veeam 已在 Veeam Agent for Microsoft Windows 版本 13.0.3.1220 中解决了该问题。各组织应将 Veeam Backup & Replication 升级到 13.0.2.29 或更高版本，该版本会将 Windows 代理更新到已修复的版本。  
  
安全团队应识别运行 Veeam Agent 版本 13.0.1.2067 或更早版本的系统，并优先修复共享工作站、服务器、管理员终端以及备份操作员或技术支持人员使用的设备上的问题。这些系统风险更高，因为多用户或特权工作流会增加本地访问的可能性。  
  
目前尚未找到针对 CVE-2026-32996 的可靠厂商支持的解决方案。主要的缓解措施是将受影响的 Veeam Agent 安装升级到已修复的版本。  
  
在修补完成之前，各组织应通过限制对受影响端点的交互式访问、审查本地帐户权限、限制备份操作员和管理员权限以及监控涉及 Veeam Endpoint Backup 服务、其日志文件和以 SYSTEM 权限启动的意外子进程的可疑活动来降低风险。  
  
  
信息来源：C  
yberSecurityNews  
  
