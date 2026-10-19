#  OpenCode披露安全漏洞，OpenAI、Anthropic等API密钥或遭泄露  
 FreeBuf   2026-10-19 10:30  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX0Xwo7KrN24UCgNZQC8gyMGKZgh0DNNIFRm6AKCqwK2FcNwgABg7pAk9WQNC5bDGmd5hbRd4QpRRHoS1rrTbutVmglgDDkxkMY/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1FeN5ibEkCkibaiaEGj2BRvTc0N60gLsQrOgjL0Jly0j5GnWayKyfML1qW8ibiaJcmY4XYxvKZTX5JCXq2Fv8z0GiccQUUwXvCJdfe4/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
遗留系统失陷与密钥风险  
  
  
OpenCode披露了一起安全漏洞事件，1.4%的控制台用户受影响，受影响工作区内存储的第三方API密钥可能已遭泄露。在10月17日发布的声明中，该公司将此次事件与大规模蒸馏攻击关联，称正与Anthropic、OpenAI、谷歌合作处置此事。  
  
  
受影响范围仅限绑定了外部服务提供商的部分控制台用户。OpenCode称，事发当天上午已向所有受影响用户发送邮件通知，未收到邮件的用户未受波及。前述1.4%的比例仅针对控制台用户统计，不覆盖OpenCode全量用户。  
  
  
公开截图中的通知显示，OpenCode发现攻击者未授权访问了一套原本准备下线的遗留系统。用户工作区中存储的第三方服务商密钥可能已泄露，其中包括OpenAI、Anthropic的服务密钥。  
  
  
通知要求收件人通过各服务商的控制台吊销并更换相关密钥，更新所有使用这些密钥的应用。同时，用户需要排查密钥近期使用记录，查看是否存在异常活动。通知仅表述密钥存在泄露可能，并未确认所有受影响密钥都已被窃取或滥用。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3tnBqaLia7vhVS1xHav9WWsaJXibtAWurZOccOkn6WrRd7II3ibelADtdxd1yJ5zYkgGCBI8mWUKQWibHtahSAXDMNzbhzj9xueQY/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
蒸馏攻击关联待核实  
  
  
OpenCode的公开声明并未说明攻击者的入侵路径，也未披露其在系统内的潜伏时长。声明没有公布本次事件涉及的密钥总量，也未指明攻击者身份。此外，OpenCode也没有提供技术证据，证明此次漏洞如何支撑了前述蒸馏攻击活动。  
  
  
模型蒸馏指的是用一个AI模型生成的响应训练另一个AI模型。在未授权攻击活动中，攻击者可以未经许可收集大量模型输出。Cybersecurity News此前曾报道Anthropic针对大规模蒸馏攻击的预警，解释攻击者如何通过重复请求实现不同模型间的能力迁移。但OpenCode并未说明泄露的凭证是否被用于发起这类请求。其声明仅提及事件与蒸馏攻击存在关联，并未完整披露攻击全貌。公开通报中也没有给出经确认的损失数据。  
  
  
Part  
03  
  
密钥处置与后续调查  
  
  
此前曾有公开网络数据中发现API密钥的相关报道，已经暴露过凭证泄露的广泛风险。那起独立事件说明密钥的安全存储与及时清理十分重要，但无法证明与本次事件的攻击者或攻击手法存在关联。  
  
  
收到通知的用户需第一时间按照警示要求完成密钥更换流程。用户可以对照服务商使用记录与近期账单，识别需要排查的异常请求。更换密钥后，还要同步更新所有依赖该密钥的应用配置。  
  
  
OpenCode表示，发布声明时事件处置仍在进行中，暂时无法披露更多细节。后续OpenCode将发布完整技术报告，明确攻击入口与凭证泄露范围。报告也会确认攻击者是否发起过未授权的模型调用请求。  
  
  
参考来源：  
  
OpenCode Security Breach Exposes Third-Party API Keys of its Console Users  
  
https://cybersecuritynews.com/opencode-security-breach/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
