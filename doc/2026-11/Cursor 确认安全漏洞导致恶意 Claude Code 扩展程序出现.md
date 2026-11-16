#  Cursor 确认安全漏洞导致恶意 Claude Code 扩展程序出现  
爱拍照的老李
                    爱拍照的老李  爱拍照的老李   2026-11-16 01:12  
  
**导****读**  
  
  
  
Cursor 证实其遭遇了安全漏洞，黑客在从人工智能基础设施提供商 Fireworks AI 窃取凭证后，通过其应用商店分发了恶意 Claude Code 扩展程序。  
  
  
此次数据泄露还让攻击者得以进入内部系统并窃取客户数据，其中包括未做脱敏处理的历史AI编码智能体日志，这些凭证随后被用于攻破其他额外服务。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PaFY6wibdwyLSY6hqeRND7K61pvuiaxG2wiaIY9UHrwqlsaS8XUGxtcic3A0y9RAN6K8rAoa5Gz8PItqC5iae0eS2YGCPSWCXLzS5GvVTPlHYzBg/640?wx_fmt=png&from=appmsg "")  
  
此次数据泄露事件在周五发布的一份新事件报告（  
https://cursor.com/cn/blog/security-incident-update）中被披露，该报告详细说明了攻击者如何于10月29日通过Cursor的应用市场分发Anthropic公司Claude代码扩展程序的恶意版本。  
  
  
此次攻击使多达5000名开发者暴露于安装了持久SSH后门的恶意软件之下。  
  
Cursor 是一款基于微软开源的 Visual Studio Code（VS Code）打造的人工智能代码编辑器，支持开发者借助人工智能助手编写、修改和调试软件。  
  
  
据Cursor称，这次攻击始于黑客获取了存储在Fireworks AI上的访问密钥，Fireworks AI是一家用于AI模型训练和推理的第三方供应商。  
  
  
攻击者利用暴露的密钥，从 Cursor 用于与 Fireworks 共享信息的只读存储桶中复制了数据。  
  
  
该存储桶包含历史 AI 编码智能体追踪数据，大多来自 2025 年，收集自未启用隐私模式且数据可用于模型训练的用户。部分追踪数据包含未编辑的 API 凭证，这使得攻击者能够访问更多 Cursor 系统。  
  
  
Cursor 表示，暴露代码智能体痕迹的用户占比低于 0.05%，而 API 凭证遭泄露的客户占比低于 0.01%。  
  
  
黑客利用泄露的凭证，从 Cursor 的多款软件即服务（SaaS）工具中窃取了数据，包括：  
- 已归档的非生产环境 Cursor 源代码仓库，部分仓库包含业务联系信息。  
- 有限的客户支持信息以及邮件对话内容。  
- 产品使用指标和客户联系详情。  
- 部分AI编码助手对话的摘要。  
- 客户此前与 Cursor 支持团队共享的凭据。  
Cursor 表示其监控系统检测到了未经授权的活动，导致该公司撤销了 Fireworks 访问密钥、轮换了泄露的凭证，并对额外的未经授权访问展开调查。  
  
  
随着调查推进，为防患于未然，Cursor 暂时关闭了部分内部及产品相关服务。然而，攻击者已经获取了一项凭证，凭借该凭证他们可以通过 Cursor 的应用商店分发扩展程序。  
  
  
10月29日，黑客利用该凭据在Cursor的VS Code扩展市场上发布了Anthropic公司Claude代码扩展的恶意版本。  
  
  
“10月29日，攻击者利用我们内部系统泄露数据中被盗的凭据，将Anthropic的Claude代码扩展程序的恶意未授权副本直接放置在用于提供Cursor VS Code扩展程序市场的缓存中。”Cursor解释道。  
  
  
此次攻击并未影响Anthropic的合法扩展程序，也未影响Cursor所使用的扩展市场Open VSX。  
  
  
恶意扩展程序的2.1.312和2.1.313版本在Cursor将其移除前，已存在约11小时。  
  
正如 BleepingComputer 此前报道过的（  
https://www.bleepingcomputer.com/news/security/malicious-claude-code-extension-on-cursor-planted-ssh-backdoors/），攻击者篡改了合法的 Claude Code 扩展，以执行恶意 JavaScript，试图在开发者的电脑上安装持久化的 SSH 后门。  
  
  
该恶意软件收集系统信息，安装攻击者控制的SSH密钥，尝试开启SSH访问，并配置持久化机制以维持远程访问权限。  
  
  
不过，安全研究人员发现了一些编码错误，这些错误很可能导致后门无法在许多未启用 SSH 的 Windows 和 macOS 系统上运行。  
  
  
Cursor 估计，在曝光窗口期内请求受影响扩展版本的多达 5000 名用户，可能已经下载到了恶意安装包。  
  
  
该公司表示，其生产基础设施、官方应用版本、客户代码仓库，以及启用隐私模式的用户所属的代码和提示词均未受到影响。  
  
  
Cursor 表示，作为预防措施，它已将所有推理流量从 Fireworks 转移出去，撤销并轮换了暴露的凭据，重建了其扩展市场，并重新部署了生产基础设施。  
  
  
黑色十月  
  
  
这一事件发生在网络安全公司 Wiz 称之为“黑色十月”的针对人工智能开发者平台的一系列攻击浪潮期间。  
  
  
这些攻击以开发基础设施为目标，窃取身份验证凭证，尤其是那些能访问人工智能模型的凭证。  
  
  
被盗的凭据被用于未经授权的人工智能推理，给受影响的客户造成了巨额费用，部分事件还与人工智能模型提取的尝试有关联。  
  
  
这些攻击已影响到众多公司，包括铁路、Northflank、Lightning AI、OpenCode和Warp。  
  
  
在Railway平台，攻击者窃取了客户的身份验证信息，这些信息随后被用于产生约74万美元的人工智能欺诈消费。OpenCode还报告了API密钥泄露以及未经授权的人工智能使用情况。  
  
  
Fireworks AI 也受到了影响，披露了多起安全漏洞，其中一起涉及暴露的 API 密钥和令牌，另一起则是攻击者通过被入侵的第三方云服务商访问了开发机器。  
  
  
此次 Fireworks 事件还影响了其他客户，包括Notion，该公司近日披露，少数工作区的 AI 聊天记录遭到泄露。  
  
  
技术报告：  
  
https://www.wiz.io/blog/black-october-for-ai-developer-platforms  
  
  
新闻链接：  
  
https://www.bleepingcomputer.com/news/security/cursor-confirms-security-breach-led-to-malicious-claude-code-extension/  
  
****  
扫码关注  
  
爱拍照的老李  
  
**讲述普通人能听懂的安全故事**  
  
  
