#  N-able 紧急发布多个 0day 漏洞的热修复方案  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-11-12 08:50  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**N-able****发布了热修复方案，修复影响****N-central****远程监控与管理（****RMM****）平台的****0day****漏洞。这些漏洞在上周末被公开披露，迫使该公司将存在漏洞的托管环境下线。**  
  
此前，N-able 决定将美洲、欧洲和亚太地区所有托管 N-central 环境下线，直到能够完成补丁修复，以防范针对这些 0day 漏洞的潜在在野攻击。N-able 周一表示：“N-able 正在积极应对 11 月 8 日周日晚间一名独立研究人员发现的 N-central 关键漏洞被不负责任地公开一事。作为预防措施，我们于 11 月 9 日将托管 N-central 环境下线，并向运行本地部署 N-central 服务器的客户提供指导，建议他们暂时将服务器下线，直至另行通知。”  
  
N-central 可使 IT 部门和托管服务提供商（MSP）通过集中式基于 Web 的控制台监控、管理和维护客户网络与设备。N-able 现已发布 N-central 2026.4 HF2，其中包含安全补丁和技术更新，用于修复部分被泄露的 0day 漏洞，包括“最严重的”漏洞：  
  
- CVE-2026-125883 - 多个 XXE 漏洞。  
  
- CVE-2026-125878 - 多个 OGNL 注入漏洞。  
  
- CVE-2026-125879 - 通过 nable 可写的 syslog-ng 配置实现本地权限提升至 root 漏洞。  
  
- CVE-2026-125880 - 通过 MMS3 补丁元数据（管理员上下文）实施存储型 XSS 漏洞。  
  
- CVE-2026-125881 - 通过服务导入模式验证实施基于 schema 提示的 SSRF 漏洞。  
  
- CVE-2026-125882 - 通过文件仓库上传路径遍历实现 RCE（致谢Sprocket Security公司的研究员 Nate Fair）。  
  
- CVE-2026-91000 - 不安全的 IPC 通道导致本地提权漏洞。  
  
  
  
N-able 在周二的更新中表示，所有地区所有托管 N-central 服务器（NCOD）现已成功升级，并建议且“强烈”建议所有本地部署 N-central 客户“尽快”将安装升级至新发布的版本。该公司表示：“客户在恢复正常访问 N-central 环境之前，应仔细遵循已发布的升级指南。早先的事件指南建议本地部署客户在修复程序开发期间保持 N-central 服务器离线。”  
  
尽管 N-able 尚未将这些漏洞标记为已在野利用，但威胁情报公司 Defused 周二表示，攻击者已经开始扫描互联网上的易受攻击系统，并警告管理员立即将 N-central 服务器下线。Defused 公司表示：“在过去 24 小时内，我们观察到有人针对昨天公开披露的 N-able N-central 0day漏洞进行探测。我们看到有人开启匿名代理会话、拉取设备配置，并在未认证的情况下访问远程控制端点。这些路径属于0day漏洞攻击面的一部分。”  
  
互联网安全非营利组织 Shadowserver 报告称，目前仅有略超过 148 台 N-central 服务器暴露在互联网上，较周一的 1338 台大幅下降。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUPcx1mOibkMyOYHSCrvJwciar1VYsNodC0V1ho1avia7OkDom0WgYTQicGAEOPvFUicibjT2OX9Uzm3vaI9KCHdHDdxmeR192tCyxIo/640?wx_fmt=gif&from=appmsg "")  
  
**因此前被忽视，公开漏洞报告**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUJc7Mmxibib8VeB09mwpsXrkic2Y8pGUz3Zzeo2EkL3fFzzfSrH459OuxMK9lgKft5aKmupTZq3lD6ACaiahlczJUI89ub7wgZ8JY/640?wx_fmt=gif&from=appmsg "")  
  
  
  
安全研究员 Sean Bales 于 11 月 8 日周日晚间披露了这些漏洞，并发布了一个名为 Able2Shell 的 GitHub 仓库，其中包含针对这些漏洞的概念验证利用程序。  
  
Bales 表示，这些 0day 漏洞影响 N-central Appliance 的预发布版本2026.4.0.31，他认为大多数（如果不是全部）漏洞也影响更早版本。  
  
已披露的漏洞包括远程代码执行（RCE）、认证绕过、凭据泄露、任意文件写入、目录遍历、XML 外部实体（XXE）注入和跨站脚本（XSS）漏洞。该安全研究员还演示了一个严重的未认证 XXE 到 RCE 利用链，该利用链滥用 N-central 的服务导入功能，窃取受信任的本地会话、重置目标账户密码，并以 nable 用户身份在服务器上执行命令。  
  
攻击者还可以利用该漏洞创建一个由攻击者控制凭据的默认管理员账户，从而获得对 N-central 管理界面的恶意管理员访问权限。Bales表示自己在未通知 N-able 的情况下发布了 N-central 0day漏洞的原因是，该公司据称未能修复他大约五六个月前在另一款 N-able 产品中报告的漏洞（包括两个不相关的远程代码执行漏洞和一个权限提升漏洞）。当被问及 Bales 关于此前披露的安全问题的指控时，N-able 没有回答这些问题，而是回应了一份关于 N-central 安全事件和预防性下线的声明。  
  
N-able 为遍布 140 多个国家的 50 多万家组织机构的托管服务提供商（MSP）和企业 IT 部门提供基于云的远程监控、网络安全和数据保护工具。  
  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[N-able 紧急修复正遭利用的CVSS 满分 N-central RCE 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527055&idx=2&sn=5548cd5e1fbb288e5c71c1c2cbf20b04&scene=21#wechat_redirect)  
  
  
[N-Able Take Control Agent 高危漏洞可用于 Windows 系统提权](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247517665&idx=1&sn=81658f5393b8756f823fdc4fce193db9&scene=21#wechat_redirect)  
  
  
[微软11月补丁星期二值得关注的漏洞，含1个0day](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527726&idx=1&sn=2c5d66ada519c7108099844332f3e46e&scene=21#wechat_redirect)  
  
  
[谷歌修复今年第八个已遭利用的 0day 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527714&idx=2&sn=d6244344c53e751c0c2681cebcab47bb&scene=21#wechat_redirect)  
  
  
[D-Link：注意缓解DGS-3130系列交换机中的多个0day](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527704&idx=2&sn=9e8eaceb5083ff4b5492ca87612729d6&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/n-able-releases-security-hotfixes-for-zero-day-vulnerabilities/  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
