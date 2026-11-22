#  群晖紧急修复两项 NAS 平台严重漏洞：CVSS 评分高达 9.8 与 8.1  
 网安百色   2026-11-21 10:52  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJnt7a7hnpyWJ6RsBAo3CsYdw1JWM5yxT4KYnUKQLmUaEibk9pc2VVEeFgjU1CjXyAjbKaiajdKzrpC0SvUvxuvSPb8mKaJldWTY3s/640?wx_fmt=png&from=appmsg "")  
  
群晖（Synology）宣布，其 DiskStation Manager (DSM) 和 BeeStation OS 操作系统中存在两个可被远程利用的安全漏洞。针对这两个 DSM 分支的修复补丁计划于 2026 年 11 月 20 日发布。  
  
11 月 19 日发布的安全公告指出，这两个漏洞分别为严重的 OAuth API 漏洞（CVE-2026-120382）和 NGINX HTTP/2 内存破坏漏洞（CVE-2026-123007）。群晖将这两个漏洞在产品层面的严重等级均评定为“严重（Critical）”，CVSS 3.1 评分分别为 9.8 和 8.1。值得注意的是，利用这两个漏洞均无需任何账户凭证或用户交互。  
### 群晖严重安全漏洞详情  
  
CVE-2026-120382 源于 DSM OAuth API 中对命令参数分隔符处理不当。远程攻击者可借此读取或写入任意文件，或导致系统拒绝服务（DoS）。  
  
根据公告的评分向量，该漏洞的攻击复杂度较低。其影响不仅限于未授权的数据泄露：与此前仅允许任意文件读取的群晖 NFS 漏洞不同，该漏洞还允许攻击者直接修改系统文件。在官方修复发布前，群晖建议暂时停用 OAuth Service（OAuth 服务）套件。  
  
CVE-2026-123007（包含在 NGINX2Root 公告中）是 NGINX Plus 和 NGINX 开源版本在启用 HTTP/2 配置时存在的堆缓冲区溢出漏洞。该内存破坏问题可能导致 NGINX worker 进程崩溃；如果地址空间布局随机化（ASLR）被禁用或可被绕过，则有可能实现远程代码执行（RCE）。  
  
与涉及上游连接（upstream connections）的另一项 NGINX HTTP/2 漏洞不同，此问题主要涉及 HTTP/2 的流控制（flow control）机制。群晖建议用户通过“控制面板 > 网络 > 连接性”关闭 HTTP/2 功能。  
  
两份公告均指出，DSM 7.1、7.2.1、7.2.2、7.3 和 7.4，以及 BeeStation OS 1.5 均受影响。此外，NGINX 漏洞还影响 APM 1.2、APM 2.0 和 DSM Enterprise 1.0；而 OAuth 公告则明确这些产品不受该漏洞影响。Synology Router Manager (SRM) 1.2 和 1.3 均不受这两个漏洞的影响。关于 BeeStation 的发现，与此前报道的涉及 DSM、BeeStation Manager 和 Synology Drive Server 的 BeeStation 漏洞属于独立事件。  
  
11 月 19 日的公告明确指出，DSM 7.4.1-90080-2 和 7.3.2-86009-6 分别是各自分支的最低修复构建版本（build），均计划于 11 月 20 日发布。群晖强烈建议运行 DSM 7.2.1 和 7.2.2 的用户直接升级至 7.3.2-86009-6 或更高版本。  
  
OAuth 公告在技术描述中为旧版 DSM 分支指定了修复版本的阈值，但其可用性状态表仍将那些版本的更新列为“处理中（ongoing）”。仅凭版本号并不能保证这些更新当前已可供实际安装。  
  
管理员应通过“群晖产品安全公告门户（Synology Product Security Advisory portal）”仔细核对设备当前安装的构建版本，并优先处理那些可从不受信任网络直接访问的设备。  
  
在官方修复版本正式可用之前，请针对受影响的漏洞分别应用临时缓解措施（workaround）：**禁用 HTTP/2 无法解决 OAuth 漏洞，停用 OAuth Service 也无法解决 NGINX 漏洞**  
。若系统同时受这两个漏洞影响，在等待确认的更新期间，必须同时应用这两种缓解措施。完成更新安装后，在重新启用任一功能之前，务必先验证当前运行的构建版本号是否确已更新。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
