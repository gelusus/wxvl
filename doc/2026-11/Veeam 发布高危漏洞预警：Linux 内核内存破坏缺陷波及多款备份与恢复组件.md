#  Veeam 发布高危漏洞预警：Linux 内核内存破坏缺陷波及多款备份与恢复组件  
 网安百色   2026-11-10 10:49  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJnsQxO7QC3APbYPNA9qFicsOK3KYA2RxgrLl88NE3eFic32Z94mR2xOibKLwic4tp67LoITaYEHtz7vhrmN44FKlN26UdjOP45nZB9Y/640?wx_fmt=png&from=appmsg "")  
  
一个 CVSS 评分为 9.8 的严重 Linux 内核漏洞，可能导致其备份和恢复产品出现拒绝服务 (DoS) 或内存破坏。  
  
Veeam 在 11 月 6 日发布的影响声明中指出，TCP/IPv4 连接建立过程中存在一处 Linux 内核“释放后使用”（Use-After-Free, UAF）漏洞（CVE-2026-98432），并概述了受影响系统的更新路径。  
  
其安全公告并未报告存在在野利用（in-the-wild exploitation），也未确认存在远程代码执行（RCE）风险。公告指出的主要风险是：若受影响的设备（appliance）因漏洞变得不可用，可能会直接中断备份和恢复工作流。  
  
受影响的产品包括 Veeam Software Appliance 13.1 和 Veeam Infrastructure Appliance 13.1。基础架构角色（Infrastructure roles）涵盖备份代理（backup proxies）、强化存储库（hardened repositories）、挂载服务器（mount servers）、网关（gateways）以及 Cloud Connect 组件。此外，通过 Veeam 虚拟机管理程序（hypervisor）插件部署的工作节点（workers）同样需要更新内核。  
### Veeam 警告严重 Linux 内核漏洞  
  
基于 Windows 的 Veeam Backup & Replication 部署环境同样需要引起重视，因为它们会部署 Linux 辅助组件（helper appliances）。这些组件用于支持文件级恢复、虚拟实验室（Virtual Labs）和云恢复。公告明确列出了受影响的 13.1 和 12.3 版本辅助组件，而非指出 Windows 内核本身存在漏洞。  
  
此次披露紧随 Veeam 此前修复的独立授权绕过漏洞及更早的 Backup & Replication 安全更新之后。这些问题的成因和补丁要求各不相同，因此管理员不应想当然地认为早期的产品更新已修复了此内核漏洞。  
  
自 11 月 2 日起，Software Appliance 和 Infrastructure Appliance 13.1 已提供包含修复补丁的内核。修复后的软件包版本为 5.14.0-570.60.1+27.2.el9_6_ciq  
，通过 Veeam Updater 进行分发。  
  
管理员可以从 Veeam Host Management 控制台手动安装可用更新，而无需等待自动调度计划。安装后可能需要重启系统。  
  
更新程序每 12 小时检查一次存储库。在默认设置下，强制更新将在备份服务器所在时区的最近一个星期日 14:00 进行安装。  
  
运维团队应检查更新历史记录以确认安装状态。使用本地存储库镜像（local repository mirror）的系统，只有在镜像完成同步后才会接收到该软件包。  
  
对于 Windows 部署环境，13.1 版本的专属修复补丁（private fix）要求 Backup & Replication 版本至少为 13.1.2.57。重新打包的 13.1.2 安装介质已包含此修复。  
  
Veeam 在 11 月 6 日的公告中表示，针对 12.3 版本的专属修复补丁即将发布。临时辅助组件在重新部署时将自动获取更新后的镜像。  
  
AWS、Azure 和 Google Cloud 插件组件通过 Veeam Updater 接收 Ubuntu 内核更新。管理员应安装可用更新，并查阅 Ubuntu CVE 跟踪器以确认软件包状态。  
  
Veeam Kasten 容器镜像本身不包含内核，但其底层的 Kubernetes 节点仍需进行供应商层面的安全检查。由客户自行管理的 Linux 代理、存储库和网关同样需要执行发行版（distribution）级别的更新。完成这些独立的检查，有助于团队在更新主设备后，避免遗漏未打补丁的恢复组件。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
