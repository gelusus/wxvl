#  Cacti曝出4HTTP2ROOT零日漏洞: 无需认证，4个HTTP请求即可实现远程命令执行  
tuto
                    tuto  杂杂咱谈   2026-11-28 02:22  
  
## 一、漏洞概述  
  
2026 年 11 月 26 日，安全研究人员 ek0ms savi0r 公开披露了 Cacti 网络监控平台中的一个零日漏洞，命名为 **4HTTP2ROOT**  
。  
  
该漏洞属于未经身份验证的存储型 SQL 注入漏洞，攻击者可以利用访客权限下的配置参数验证不足问题，将恶意 SQL 注入数据持久化保存，随后通过二阶 SQL 注入影响 Cacti 的轮询配置，最终实现操作系统命令执行。  
  
Previdian 在实验环境中验证了公开 PoC，并确认该漏洞能够影响 Cacti 1.2.31。研究人员展示了仅通过 4 个 HTTP 请求即可触发命令执行的攻击过程。  
  
**漏洞基本信息：**  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">详细信息</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">漏洞名称</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">4HTTP2ROOT</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">影响产品</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Cacti 网络监控平台</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">CVE 编号</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">暂未分配</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">已确认受影响版本</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Cacti 1.2.31</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">漏洞类型</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">未经身份验证的存储型 SQL 注入</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">潜在影响</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">远程命令执行（RCE）</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">身份认证要求</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">无需有效账户凭据</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">利用前提</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">目标启用了 Guest 访客访问</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">公开 PoC</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">已公开</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">官方补丁</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">暂未发现确认的修复版本</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">CVSS 评分</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">暂无官方评分</span></section></td></tr></tbody></table>## 二、漏洞技术原理  
  
4HTTP2ROOT 的核心问题在于 Cacti 对访客用户可控制的配置参数缺乏充分验证，导致攻击者能够通过特定请求向数据库写入恶意 SQL 内容。  
  
该漏洞利用了二阶 SQL 注入机制。  
  
与普通 SQL 注入不同，二阶 SQL 注入并不一定在恶意数据提交时立即执行。攻击者首先将恶意内容存储到数据库中，随后等待应用程序在其他业务流程中重新读取并使用这些数据。当数据未经安全处理地进入 SQL 查询时，恶意语句才会被触发。  
  
在该漏洞中，攻击者可以利用这一机制影响 Cacti 的轮询配置，进一步将 SQL 注入转化为操作系统命令执行。  
  
主要问题包括：  
1. **访客配置参数验证不足：攻击者可以通过 Guest 访问入口提交经过构造的恶意配置数据。**  
1. **恶意数据持久化：恶意 SQL 内容被保存到数据库中，为后续二阶注入创造条件。**  
1. **轮询配置遭到操纵：Cacti 在后续处理相关配置时触发 SQL 注入。**  
1. **实现命令执行：攻击者最终能够利用受影响的配置处理流程执行任意操作系统命令。**  
研究人员在实验环境中展示了 Root 权限级别的命令执行，但这并不意味着所有部署环境都能够直接获得 Root 权限。实际命令执行权限取决于 Cacti 应用程序及其轮询进程运行时所使用的系统账户。  
## 三、漏洞利用过程  
  
公开研究表明，攻击者可以通过 4 个 HTTP 请求完成漏洞利用。  
  
整体攻击链可以概括为：  
```
攻击者  |  v访问 Cacti Guest 入口  |  v提交恶意配置参数  |  v恶意 SQL 数据写入数据库  |  v触发二阶 SQL 注入  |  v操纵 Cacti Polling 配置  |  v执行操作系统命令  |  v获取目标主机控制权限
```  
  
该攻击链的关键在于恶意数据的持久化与后续调用。攻击者不需要通过传统登录流程获得有效账户，而是利用启用的访客访问功能进入漏洞触发路径。  
  
**攻击特点：**  
- 无需有效身份认证。  
  
- 不依赖用户交互。  
  
- 利用存储型、二阶 SQL 注入实现攻击。  
  
- 通过操纵轮询配置实现命令执行。  
  
- 公开 PoC 展示了仅需 4 个 HTTP 请求的利用方式。  
  
这里的 4 个 HTTP 请求指公开 PoC 展示的攻击流程，并不代表所有部署环境都必然能够以相同请求数量完成利用。  
  
## 四、关键漏洞触发点分析  
  
根据研究人员进一步披露的技术分析，漏洞利用涉及 Cacti 中三个主要功能文件。  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">作用</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">漏洞利用中的角色</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><p><span leaf="">auth_profile.php</span></p></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">用户个人配置管理</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Guest 用户获取会话并修改配置</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><p><span leaf="">graph_view.php</span></p></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">图形预览功能</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">将数据库中保存的恶意值拼接进 SQL 查询</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><p><span leaf="">graph_realtime.php</span></p></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">实时图形初始化</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">触发轮询进程执行被篡改的命令</span></section></td></tr></tbody></table>  
具体利用过程如下：  
  
**1. 获取 Guest 会话**  
  
攻击者访问 auth_profile.php，  
在无需账户凭据的情况下获取 Guest 会话 Cookie 和 CSRF Token。  
  
**2. 写入恶意配置**  
  
通过 auth_profile.php?action=update_data 修改 preview_graphs_per_page 参数。  
  
该参数原本用于控制图形预览页面的显示数量，但应用程序没有强制验证其必须为整数，导致攻击者可以提交恶意字符串。  
  
**3. 触发二阶 SQL 注入**  
  
攻击者访问 graph_view.php?action=preview。  
  
应用程序重新读取数据库中保存的参数，并将其拼接到 SQL 查询的 LIMIT 子句中  
。  
  
由于缺乏有效的输入约束，攻击者能够在测试环境中利用堆叠 SQL 语句执行额外的数据库更新操作，  
修改 poller_item 表中的 arg1 字段。  
  
**4. 触发命令执行**  
  
攻击者访问 graph_realtime.php?action=init，触发实时轮询流程  
。  
  
相关代码最终通过 exec_poll() 调用 popen() 执行数  
据库中已经被篡改的命令参数，从而实现操作系统命令执行。  
## 五、受影响范围与利用条件  
  
目前已确认的情况如下：  
- **Cacti 1.2.31：已由研究人员在实验环境中验证存在漏洞。**  
- **Guest 访问功能：必须处于启用状态，才能满足目前公开 PoC 展示的未认证攻击条件。**  
- **其他版本：完整受影响版本范围尚未确定。**  
- **数据库环境：原始 PoC 的 SQL 堆叠执行行为经过特定实验环境验证，不能直接推断所有数据库驱动及部署配置均可利用。**  
Cacti 默认情况下关闭匿名 Guest 访问，但管理员可以为了提供监控图表访问而启用该功能。  
  
因此，启用了 Guest 访问的 Cacti 实例应当优先排查。  
## 六、漏洞是否已遭到实际攻击？  
  
已观察到攻击尝试  
  
根据 Previdian 于 2026 年 11 月 27 日发布的更新，该公司在其真实 Cacti 应用蜜罐中观察到了针对 4HTTP2ROOT 的攻击活动。  
  
主要情况：  
- 监测到来自同一个源 IP 地址的 22 次 SQL 注入尝试。  
  
- 攻击请求包含基于时间延迟的 SQL 注入 Payload。  
  
- 攻击者似乎正在测试新披露漏洞是否可以被利用。  
  
- 尚未确认这些攻击尝试成功实现远程命令执行。  
  
Previdian 于 11 月 26 日将该漏洞纳入自动化监测，约 14 小时后便观察到首次攻击尝试，并将其加入自身的 KEV 漏洞跟踪列表。  
  
目前已经观察到针对漏洞的攻击尝试，但尚无充分证据证明攻击者已经在真实生产环境中成功利用该漏洞。  
## 七、修复建议与安全防护  
  
截至 2026 年 11 月 26 日，尚未发现 Cacti 官方针对 4HTTP2ROOT 发布的确认修复补丁。  
  
在官方补丁发布之前，建议管理员优先采取以下临时防护措施。  
### 1. 禁用 Guest 访客访问  
  
这是目前最直接的临时缓解措施。  
- 删除 Cacti Authentication Settings 中指定的 Guest User。  
  
- 进入 Console > Configuration > Users，  
禁用 Guest 用户账户。  
  
- 使用全新的未认证浏览器会话，验证访客无法继续访问受保护的应用功能。  
  
禁用 Guest 访问能够移除公开 PoC 攻击链中的关键前提，但并不意味着底层 SQL 注入漏洞已经得到修复。  
### 2. 限制管理平台网络访问  
- 不要将 Cacti 管理界面直接暴露在公网。  
  
- 将访问权限限制在可信管理网络。  
  
- 必要时通过经过身份认证的 VPN 访问管理平台。  
  
- 使用防火墙限制非授权来源的 HTTP/HTTPS 请求。  
  
### 3. 排查潜在入侵行为  
- 用户配置是否出现异常修改。  
  
- preview_graphs_per_page 等配置参数是否存在异常字符串。  
  
- Cacti 轮询配置是否被意外修改。  
  
- 数据库 poller_item 表中的 arg1 字段是否出现异常变化  
。  
  
- 是否存在异常 PHP Shell、WebShell 或其他新创建的可执行文件。  
  
- Cacti 相关进程是否启动了非预期的系统命令。  
  
特别需要关注 poller_item.arg1，因为该字段是公开利用链中被修改并最终传递给命令执行流程的关键位置  
。  
### 4. 限制应用程序运行权限  
  
确保 Cacti 应用程序及轮询进程遵循最小权限原则。  
- 避免使用 Root 账户运行 Cacti。  
  
- 为 Web 服务和轮询进程配置独立的低权限账户。  
  
- 限制应用程序对系统关键目录的写入权限。  
  
- 避免将数据库及网络设备管理凭据暴露给不必要的进程。  
  
### 5. 排查凭据泄露风险  
  
如果确认服务器可能已经遭到入侵，应重点检查 Cacti 中保存的网络设备认证信息。  
  
必要时轮换可能已经泄露的设备管理凭据，并检查相关网络设备是否出现异常登录及配置变更。  
### 6. 关注官方安全更新  
  
持续关注 Cacti 官方安全公告及版本发布信息，在正式修复版本发布后及时升级。  
## 八、总结  
  
4HTTP2ROOT 展示了一条比较典型的 Web 应用攻击链：攻击者利用 Guest 用户可修改的配置参数，将恶意数据写入数据库，再通过二阶 SQL 注入影响后续业务逻辑，最终借助实时轮询进程实现操作系统命令执行。  
  
该漏洞值得关注的地方不仅在于无需认证和公开 PoC，还在于它将看似普通的图形显示配置参数与底层命令执行机制联系起来。  
  
对于使用 Cacti 进行网络设备及基础设施监控的企业而言，一旦监控服务器被攻陷，攻击者可能进一步获取其中保存的网络设备凭据，为横向移动和内部网络渗透创造条件。  
  
**建议安全团队立即检查 Cacti 是否启用了 Guest 访问功能，优先关闭匿名访客入口，并对轮询配置、异常进程及网络设备凭据开展安全排查。**  
  
****  
引用来源:   
Previdian  
  
poc:   
https://github.com/ekomsSavior/Sad-Girls-0day-Club  
  
