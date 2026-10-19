#  【安全圈】OpenCode披露安全漏洞，控制台用户第三方API密钥或遭泄露  
 安全圈   2026-10-18 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
安全漏洞  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyGE0XbMibZoV3ibPSkFkpmvuffxtibzHTUtQzzSFGicLP35ytOkzl5DsfIONCibZIuTqT4m8hrrIoyAcoCvWOE5LseCnxWSIl0bT6Ng/640?wx_fmt=jpeg&from=appmsg "")  
  
OpenCode披露了一起安全漏洞事件，1.4%的控制台用户受影响，受影响工作区内存储的第三方API密钥可能已遭泄露。在10月17日发布的声明中，该公司将此次事件与大规模蒸馏攻击关联，称正与Anthropic、OpenAI、谷歌合作处置此事。  
  
受影响范围仅限绑定了外部服务提供商的部分控制台用户。OpenCode称，事发当天上午已向所有受影响用户发送邮件通知，未收到邮件的用户未受波及。前述1.4%的比例仅针对控制台用户统计，不覆盖OpenCode全量用户。  
## 攻击者入侵遗留系统，第三方服务商密钥存泄露风险  
  
公开截图中的通知显示，OpenCode发现攻击者未授权访问了一套原本准备下线的遗留系统。用户工作区中存储的第三方服务商密钥可能已泄露，其中包括OpenAI、Anthropic的服务密钥。  
  
通知要求收件人通过各服务商的控制台吊销并更换相关密钥，更新所有使用这些密钥的应用。同时，用户需要排查密钥近期使用记录，查看是否存在异常活动。通知仅表述密钥存在泄露可能，并未确认所有受影响密钥都已被窃取或滥用。  
  
OpenCode的公开声明并未说明攻击者的入侵路径，也未披露其在系统内的潜伏时长。声明没有公布本次事件涉及的密钥总量，也未指明攻击者身份。此外，OpenCode也没有提供技术证据，证明此次漏洞如何支撑了前述蒸馏攻击活动。  
  
模型蒸馏指的是用一个AI模型生成的响应训练另一个AI模型。在未授权攻击活动中，攻击者可以未经许可收集大量模型输出。Cybersecurity News此前曾报道Anthropic针对大规模蒸馏攻击的预警，解释攻击者如何通过重复请求实现不同模型间的能力迁移。但OpenCode并未说明泄露的凭证是否被用于发起这类请求。其声明仅提及事件与蒸馏攻击存在关联，并未完整披露攻击全貌。公开通报中也没有给出经确认的损失数据。  
  
此前曾有公开网络数据中发现API密钥的相关报道，已经暴露过凭证泄露的广泛风险。那起独立事件说明密钥的安全存储与及时清理十分重要，但无法证明与本次事件的攻击者或攻击手法存在关联。  
## OpenCode敦促用户更换密钥，事件处置中细节待披露  
  
收到通知的用户需第一时间按照警示要求完成密钥更换流程。用户可以对照服务商使用记录与近期账单，识别需要排查的异常请求。更换密钥后，还要同步更新所有依赖该密钥的应用配置。  
  
OpenCode表示，发布声明时事件处置仍在进行中，暂时无法披露更多细节。后续OpenCode将发布完整技术报告，明确攻击入口与凭证泄露范围。报告也会确认攻击者是否发起过未授权的模型调用请求。  
  
  
 END    
  
  
阅读推荐  
  
  
[【安全圈】程序员为女友抢号搞瘫医院挂号系统](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=1&sn=62452dec7ffbb0c77a86ee1e051297e4&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Linux内核爆出9.3分超高危漏洞，可“越狱”虚拟机攻击宿主机](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=2&sn=870c96014815465341569810ecc94d94&scene=21#wechat_redirect)  
  
  
  
[【安全圈】黑客利用 Collabora RCE 漏洞为 Nextcloud 服务器植入后门](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=3&sn=83fe8f1d0aa273a7e5c0ca6d4a83d64e&scene=21#wechat_redirect)  
  
  
  
[【安全圈】调价后，DeepSeek还是崩了5小时](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079367&idx=1&sn=55f57a7fbf76ce793069a3ed8391fc56&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
