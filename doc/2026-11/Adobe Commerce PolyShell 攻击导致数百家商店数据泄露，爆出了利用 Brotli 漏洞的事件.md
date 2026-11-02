#  Adobe Commerce PolyShell 攻击导致数百家商店数据泄露，爆出了利用 Brotli 漏洞的事件  
Rhinoer
                    Rhinoer  犀牛安全   2026-11-01 16:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/vO1zY1O9p8LhnMZBcG3z1hlgNaeL1HiaiasqQhnONdh10PNqRZgiceS5iaic95W8D4KJGDumTaaFCZbvHOK8xGHqtA1KyZ3WF97pFcz9pTbg6icSw/640?wx_fmt=png&from=appmsg "")  
  
美国网络安全和基础设施安全局 (CISA) 周二将影响 Adobe Commerce 和 Magento 的两个漏洞以及谷歌的 Brotli 解码器库添加到其已知利用漏洞 ( KEV )目录中，并指出有证据表明这些漏洞正在被积极利用。  
  
漏洞列表如下：  
- CVE-2026-111497（CVSS 评分：9.2）- Google Brotli 中存在一个未检查的返回值漏洞，该漏洞可能允许远程未经身份验证的攻击者破坏相邻的解码器状态内存、泄露敏感内存内容、导致拒绝服务，或者可能使用精心构造的 Brotli 压缩流执行任意代码。  
  
- CVE-2026-48356（CVSS 评分：9.3）- Adobe Commerce 和 Magento 中存在不受限制的文件上传漏洞，该漏洞可能导致在当前用户的上下文中执行任意代码，从而可能获得对受害者帐户或会话的更高访问权限或控制权。  
  
  
就在几天前，Sansec表示，他们观察到攻击者利用 CVE-2026-48356（也称为 PolyShell）入侵数百家电子商务商店，通过 Magento CMS 模块注入恶意加载器。  
  
该加载器旨在获取并执行攻击者控制的有效载荷，但迄今为止尚未恢复任何后续阶段的恶意软件，因此难以确定此次攻击活动的确切目标。据评估，该活动至少从2025年7月开始持续进行。  
  
关于 CVE-2026-111497（又名 Broccoli2Shell、HTTP2Shell 和 Node2Shell），威胁情报公司 Previdian 的遥测数据显示，截至 2026 年 10 月 27 日，其蜜罐传感器已记录到一次来自美国 IP 地址的攻击尝试。目前尚不清楚这些攻击活动的范围是否广泛。  
  
Rapid7表示： “攻击者如果能够提供 Brotli 数据，并让应用程序使用 node:zlib 解压缩这些数据，就可以破坏进程内存。根据 Node.js 的说法，这种破坏可能导致进程崩溃、敏感进程内存泄露或远程代码执行。因此，暴露程度取决于应用程序是否将攻击者控制的压缩数据传递给受影响的解码器。”  
  
ELLIO周一还报告称，其传感器“已记录到CVE-2026-111497漏洞利用流量，该流量携带了我们已确认可从易受攻击的版本中获取shell的公开漏洞利用程序”。该活动源自“ 94.154.43[.]7 ”，这是一个位于荷兰的IP地址，该地址也曾参与针对PHP2Shell的漏洞利用活动。PHP2Shell是PHP中的两个严重漏洞，可能导致未经身份验证的远程代码执行。  
  
ELLIO表示：“攻击者的有效载荷是一个真实且结构正确的Node2Shell攻击尝试，目标版本为Node v24.22.0——Brotli流、分阶段位移、枢轴对和交付方式均正确无误，与公开的工具完全一致。这表明远程代码执行（RCE）攻击已被专注于大规模攻击的攻击者所采用。”  
  
Brotli 是一个广泛分发的开源库，被众多项目采用，这一事实引发了紧急响应，促使各个供应商发布补丁或评估他们的产品是否受到该缺陷的影响。  
  
这包括PHP（已发布8.5.12-1、8.4.27-1、8.3.36-1 和 8.2.35-1 版本）、Node.js、Bun、Envoy Proxy、Istio、.NET、Python、ClickHouse、PHP Brotli 扩展以及 Ruby 的“brotli” gem。以下是一些安全公告的链接 -  
- Amazon Linux  
  
- Bun  
  
- Cisco  
  
- Cloudflare  
  
- CloudLinux  
  
- Envoy Proxy  
  
- F5  
  
- Fortinet  
  
- Imperva  
  
- Istio  
  
- Microsoft Azure  
  
- Microsoft .NET  
  
- Node.js  
  
- Palo Alto Networks  
  
- PHP  
  
- Qualys  
  
- Red Hat  
  
- Safe Security  
  
- Strapi  
  
- Tanium  
  
- Tenable  
  
- Ubuntu  
  
- Upwind  
  
Akamai表示： “Node.js已在22.23.4、24.22.1和26.12.1版本中修复了该漏洞。Node.js的公告并未提供临时解决方案，因此升级是唯一彻底的补救措施。使用已停止维护版本的组织应迁移到受支持的版本。”  
  
谷歌旗下的 Wiz警告称，“91% 的云环境存在易受 CVE-2026-111497 攻击的资源，82% 的云环境存在公开暴露的 Node.js 实例。”  
  
Sysdig表示： “Node2Shell并非所有Node.js服务都存在的漏洞，而是那些解码受攻击者影响的Brotli的服务的漏洞。Express 5默认的JSON解析方式将大量此类服务归入此类。目前已证实的影响是规模化崩溃和在特定配置下执行代码，因此补丁发布顺序应遵循漏洞可及性原则。”  
  
联邦民事行政部门 (FCEB) 各机构有时间在 2026 年 11 月 10 日之前对这两个缺陷进行必要的修复。  
  
  
信息来源：T  
heHackerNews  
  
