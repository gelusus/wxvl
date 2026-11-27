#  据称黑客声称已找到利用 VMware vCenter 零日漏洞的方法  
原创 ZM
                    ZM  暗镜   2026-11-26 22:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zdwoicOrrJb1DIq4bdhJhlNVIhhfxtGuECbgkDwd4ftl1zOrGTbD76tOicplcGaPicD6dSjmcds9BllJsTG3PQzL3T6icPudSEic9qYnUK34ibXho/640?wx_fmt=png&from=appmsg "")  
  
一位名为 Matilda 的论坛卖家声称，他拥有针对 CVE-2026-123890（VMware vCenter Server 的一个严重漏洞）的漏洞利用程序，并标价 30,000 美元。  
Dark Web Informer 分享的一张截图显示，卖家声称该工具可以创建新的 vCenter 管理员帐户。该商品信息是在 Broadcom 于 2026 年 11 月 19 日发布修复程序后出现的。该工具声称的功能和兼容性尚未得到独立验证。  
  
卖家将该工具描述为一个 Python 程序，其中包含用于检查服务器是否存在漏洞和利用该漏洞的独立函数。该帖子声称支持 vCenter 7.0 至 9.1 的受影响版本，并向潜在买家提供演示。但提供演示并不代表测试成功，且该帖子也未证明卖家曾使用该工具攻击过任何组织。  
  
博通发布的 VMSA-2026-0009 安全公告将该漏洞的严重程度评为 9.8 分（满分 10 分）。正如网络安全新闻在其对VMware vCenter 身份验证绕过的报道中所述，该漏洞会影响 VMware Directory Service，而该服务支持 vCenter 的单点登录系统。  
  
攻击者如果能够通过网络访问受影响的服务器，就可以绕过登录验证，而无需有效的凭据。这一点至关重要，因为 vCenter 提供了一个集中管理 VMware 主机和虚拟机的平台。  
  
谷歌威胁情报小组和 Mandiant 追踪到利用该漏洞发起的攻击最早可追溯到 2026 年 9 月初，当时补丁尚未发布。此前，《网络安全新闻》在报道 vCenter 零日攻击时曾指出，一个名为 ARMORY CASTLE 的疑似与中国有关联的组织使用了名为 BONESAW 的独立工具。  
  
Broadcom 列出了 vCenter 9.1.1.0200、9.0.2.0200 和 8.0 Update 3m 中的修复程序。VMware Cloud Foundation 5.x 的客户应遵循供应商针对 5.2.5 版本的指导。  
  
该公告未列出 vCenter 7.0 的公开修复方案，并建议拥有延保服务合同的客户联系博通技术支持。管理员应参考供应商的响应矩阵，确认适用于其部署的正确更新。  
  
国家漏洞数据库条目记录了该漏洞于 11 月 20 日被添加到 CISA 的已知已利用漏洞目录中。博通没有列出任何解决方法，因此团队应该立即应用相关更新，无需等待论坛上的说法得到验证。  
  
他们还应检查 vCenter 登录记录和管理员帐户变更，以发现异常活动，尤其是在补丁安装前可从不受信任的网络访问的服务器上。更新可以修复漏洞，但无法确定攻击者是否在修复程序安装前利用了该漏洞。  
  
