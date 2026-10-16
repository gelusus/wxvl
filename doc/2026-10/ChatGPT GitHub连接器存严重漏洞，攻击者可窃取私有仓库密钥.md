#  ChatGPT GitHub连接器存严重漏洞，攻击者可窃取私有仓库密钥  
 FreeBuf   2026-10-16 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3KC14Sjd7KC5dsO0N39kA4lty1rCokDQvnDmbHeypnxjultIfRfDHohmxqRCzMxsowEAqUEBkcEtNX1e62ib8Hia8Wgpqpj9VJg/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX13e3L8E65kbibhDibLpsHB1fwQxlhZGPXO9EqTcrIDL1TzibUawGBEQZHQVOBDOb6UuO3xBWcPsUc8Dl95ficFDrERo2GHEyAlVE8/640?wx_fmt=png&from=appmsg "")  
  
  
ChatGPT的GitHub集成功能存在一处严重漏洞，可将常规仓库审查操作转化为密钥窃取攻击。2026年10月15日发布的一份报告披露了相关攻击场景：攻击者篡改仓库内的隐藏指令，可诱导AI Agent修改其他私有仓库并触发GitHub Actions。  
  
  
研究员Sunil Tripathy披露，漏洞验证环境涉及两个独立GitHub账号。攻击者控制一个存放隐藏指令的公开仓库，受害者账号已将GitHub与ChatGPT绑定，并授权ChatGPT访问账号下所有仓库。  
  
  
该漏洞的CVSS 4.0评分为9.3，这一评分仅为研究人员个人评估，并非厂商官方确认的分级。测试全程使用研究人员自有账号和测试用假密钥，未影响任何用户数据或第三方系统。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX10gwn5uoHMtoy1bPdYEga3wSR6CGwkqgM9GfmwSM17VrL4xx00KWSQgJMNPpaeSTR67xBiaic2B4vXOhOzY9Js7feNsN4QIYicsw/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
隐藏指令劫持GitHub Agent  
  
  
当用户  
发起审查该公开仓库的请求时，AI Agent会将仓库内的恶意指令视为可信命令执行。它不会将操作范围限定在内容分析层面，反而会利用受害者的现有访问权限，向其他无关私有仓库提交GitHub Actions工作流。  
  
  
提交的工作流会自动运行，将测试密钥AUDIT_CANARY发送至外部服务器。整个过程中，用户既没有指定目标私有仓库，也没有批准任何写入操作。  
  
  
该漏洞属于间接提示注入场景，恶意指令通过AI助手读取的内容载体传递。此前AgentFlayer研究已经证实，恶意文档、绑定的Google Drive数据也存在同类安全风险。  
  
  
Part  
02  
  
恶意指令伪装成部署要求  
  
  
Tripathy表示，  
直  
接写入明确恶意的指令会触发AI拒绝机制，将指令包装为部署要求则可成功绕过拦截。攻击者只需在看似正常的工作流中引用一个独立的自定义Action，就可能触发其中隐藏的数据窃取逻辑。  
  
  
测试期间，研究人员还发现了其他潜在影响，包括枚举私有仓库列表、提交攻击者构造的文件，以及从Git历史记录中恢复攻击指令等。不过，这些结果均来自研究人员自行开展的测试和上报，目前尚未经过第三方独立验证。  
  
  
Part  
03  
  
官方权限描述存在冲突  
  
  
目前该漏洞涉及的产品权限边界仍存在疑点。OpenAI的GitHub相关文档显示，标准ChatGPT GitHub应用仅具备只读权限，代码编辑、推送操作需要使用Codex完成。这一描述与测试中观测到的工作流提交行为存在冲突，需要明确具体集成方式、权限配置和执行路径，才能确认准确的受影响范围。  
  
  
此前OpenAI Codex曾存在一处已修补的命令注入漏洞，同样可导致GitHub令牌泄露，但该漏洞源于不安全的分支名处理逻辑，与本次披露的提示注入攻击链无关。  
  
  
相关行为的首次上报时间为9月4日，9月28日研究人员完成密钥窃取场景的演示，10月14日厂商以"不在漏洞受理范围内"为由关闭了上报工单。10月15日研究人员复测时，攻击路径依然有效。目前上述信息均为研究人员单方披露，OpenAI尚未确认该漏洞存在，也未公布相关补丁进展。  
  
  
Part  
04  
  
最小权限配置可阻断攻击  
  
  
对于防御方而言，最直接的防护措施是遵循最小权限原则，为绑定的ChatGPT账号仅分配当前任务必需的仓库访问权限。OpenAI已确认，用户可通过GitHub应用设置调整仓库访问范围。  
  
  
此外，建议用户排查仓库中存在的异常工作流提交，轮换可能已泄露的密钥。对照测试显示，当访问权限收窄至单个仓库时，跨仓库写入操作会被阻断，证明严格限制权限是抵御该攻击路径的可行防护手段。  
  
  
参考来源：  
  
Critical ChatGPT GitHub Connector Flaw Could Allow Attackers to Steal Private Repository Secrets  
  
https://thehackernews.com/2026/10/critical-chatgpt-github-connector-flaw.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
