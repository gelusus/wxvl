#  LXD 与 Incus 曝出严重证书校验漏洞：可致攻击者绕过认证并完全接管 Linux 主机  
 网安百色   2026-11-03 10:50  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJnv9s88DwUOAz4FTzdImdQQA4u5UsMbQ6D6mBdoMySHSDkxYUbYeT3BdCESTKSno490uzATlkQzIRwlXCJXnUDJsdf1RUkCZxrM/640?wx_fmt=png&from=appmsg "")  
  
LXD 和 Incus 中存在一处严重的证书验证缺陷，可能允许远程攻击者绕过登录验证，进而接管集群及受管的虚拟机。  
  
该漏洞影响 HTTPS 管理监听器（HTTPS management listener），暴露在外部的 Linux 系统面临极高风险。攻击者利用此漏洞无需密码、无需窃取私钥、也无需任何用户交互即可入侵。Incus 将此漏洞的严重等级评定为“严重（Critical）”，CVSS 评分为满分 10.0。  
  
Incus 于 10 月 29 日发布的安全公告指出，该漏洞自项目初始发布以来便已存在。Canonical 于 10 月 30 日发布了其相关的 LXD 漏洞记录 CVE-2026-112816。尽管 Ubuntu 将该记录的优先级定为“中（Medium）”，但其描述的攻击路径允许未授权用户获得对集群和虚拟机的完全访问权限。  
### LXD 与 Incus 证书验证缺陷详情  
  
问题的根源在于服务在 HTTPS 连接期间检查客户端证书的方式。TLS 握手过程仅能证明客户端持有其提交的**第一个**  
证书对应的私钥。然而，该证书链中剩余的证书均由客户端控制，且未经过验证。  
  
随后的身份验证检查会将提交的证书链中的**每一个**  
证书与服务端的信任存储（Trust Store）进行比对。这就在“证明持有证书”与“接受其身份”之间制造了一个安全缺口。攻击者可以将受信任证书的公钥部分附加到自己的证书链中，即便他们没有对应的私钥，也能被系统接受为该受信任证书的所有者。  
  
公钥证书并非机密。服务器在 TLS 握手期间会返回其证书，这意味着攻击者无需拥有现有账户即可获取。根据公告描述，出示服务器自身的证书可获取数据库网关（database gateway）的访问权限，而出示集群中其他成员的证书则可获取内部集群访问权限，并完全绕过授权检查。  
  
这与《网络安全新闻》（Cyber Security News）此前报道的 runc 漏洞中的容器逃逸（container escape）缺陷截然不同。后者旨在突破隔离的工作负载，而此漏洞攻击的是控制环境访问权限的**可信管理层**  
。  
  
Incus 公告指出，只要攻击者持有相关的公钥证书，即可冒充受信任的客户端、监控指标（metrics）或服务器证书。同样的缺陷还允许攻击者通过自助服务路径更新或删除信任存储中的条目。此外，该漏洞还影响了主机与虚拟机之间代理连接（agent connection）的身份验证。  
  
另一个受影响的领域是授权脚本（authorization scriptlet），这是一种用于做出访问决策的脚本。Incus 将未经验证的证书链传递给了该脚本，导致攻击者选择的证书能够影响其决策结果。这也影响使用令牌（tokens）或 OpenID Connect 的连接，因为这些客户端在握手期间仍然可以出示证书。  
### 修复版本与即时防护措施  
  
Incus 列出 7.6.0 之前的版本为受影响版本，7.6.0 及更高版本为已修复版本。Canonical 的 LXD 公告指出，受影响的发布分支已在版本 4.0.15、5.0.11、5.21.9 和 6.10.1 中修复。管理员应对照相关厂商的公告检查其已安装的版本。  
  
在应用更新之前，Incus 建议将 HTTPS 监听器的访问权限限制为仅受信任的主机。目前不存在能够修复底层验证错误的配置类临时缓解措施（configuration-based workaround）。网络层面的访问限制可以降低暴露面，但绝不能替代对漏洞服务的补丁修复。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
