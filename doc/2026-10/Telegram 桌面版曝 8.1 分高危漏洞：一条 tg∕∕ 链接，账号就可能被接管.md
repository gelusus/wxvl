#  Telegram 桌面版曝 8.1 分高危漏洞：一条 tg:// 链接，账号就可能被接管  
原创 威胁猎人
                    威胁猎人  OSINT情报分析师   2026-10-09 08:14  
  
安全研究人员 BeakSec 披露了一个高危漏洞：  
CVE-2026-107181，CVSS 评分  
8.1。问题出在 Telegram Desktop 的 IPC 记录处理上。攻击者可以构造恶意  
tg:// 链接，一旦用户触发，就可能发生  
IPC 记录注入，进而接管受害者的 Telegram 桌面账号，并窃取本地会话数据。  
  
更值得警惕的是：  
整个过程不需要获取用户密码。  
# 漏洞速览  
  
漏洞编号：CVE-2026-107181  
  
严重级别：高，CVSS 8.1  
  
影响产品：Telegram Desktop  
  
影响版本：早于 7.2.9  
  
漏洞类型：IPC 记录注入 / 账户接管  
  
攻击方式：恶意 tg:// 链接  
  
潜在影响：未授权访问账号、窃取本地会话数据、暴露敏感账户信息、账号被接管  
  
修复版本：7.2.9 及以上  
  
当前状态：尚未见到在野利用证据  
  
虽然目前还没有公开的在野利用证据，但这类漏洞一旦被利用，后果很直接：你的聊天记录、联系人、本地会话数据，甚至账号控制权，都可能落入攻击者手中。  
# 现在要做的几件事  
  
立即更新 Telegram Desktop 到 7.2.9 或更高版本。  
  
不要点击来源不明的 tg:// 链接，尤其是聊天群、私信、邮件或网页里突然出现的“邀请”“验证”“会议”链接。  
  
审查活跃会话：设置 → 隐私与安全 → 活跃会话，发现不认识的设备立即退出。  
1. 如果你曾在旧版本上点过可疑链接，建议尽快检查登录设备、修改相关密码，并提醒重要联系人。  
  
1. 关注官方发布渠道，避免下载第三方“修复版”“绿色版”。  
  
技术再强，也怕手快。  
  
转发给身边用 Telegram 桌面版的朋友：先升级，再点链接。  
  
[#Telegram]()  
[#高危漏洞]()  
[#CVE2026107181]()  
[#网络安全]()  
[#账号安全]()  
  
  
[#CyberSecurity]()  
[#Telegram]()  
[#CVE2026107181]()  
[#AccountTakeover]()  
[#Vulnerability]()  
[#InfoSec]()  
[#ThreatIntel]()  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/PbIBsQGgFOkn25rSh3qCsextDiahqmZlicZOlZ6Mv3ol8YZFIIbibJUYHkABygofwDS1Yrvia57eesSmAHB1CJECmvn4Fmx8mfhHXElHTdibZS7c/640?wx_fmt=jpeg "")  
  
