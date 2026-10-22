#  OpenAI推出五阶段高危漏洞响应框架，应对AI发现漏洞增长趋势  
 FreeBuf   2026-10-22 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3zOBN5lCIxgETGWicSse2XcuHlo21eGfIicdGCOFC2CQm2pUJfqRJuhPML5icVSSzY9zXVRxbx3EFXl9q8nX8JbpwVOyJGj25JIU/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0EDJWJvbWMrXoVpUOTIVB5xy1RicEBq7e7JyTJ8dP4aXn9ohopdoqL093Co3MZklYoLpb4heTWaf33RY38LA4T49J4reQEPTAg/640?wx_fmt=png&from=appmsg "")  
  
  
OpenAI近日提出一套五阶段流程，用于处置广泛使用软件中的高危漏洞。这套流程源自团队历时两个月处置PHP漏洞的经验——PHP是支撑全球网站与Web应用运行的开源编程语言。  
  
  
本次披露的PHP漏洞中，一个由Calif使用OpenAI的GPT-5.6 Sol模型发现，相关工作隶属于OpenAI面向开源安全的Patch the Planet计划；Wiz则独立发现了全部两个漏洞并完成上报。  
  
  
OpenAI写道："这些漏洞影响范围极大，只是我们目前已发现并正在修复的多个高危漏洞中的一部分。我们预计未来几个月，同类漏洞的发现数量将大幅增长。"  
  
  
这套框架专门针对会同时影响大量产品的漏洞，这类漏洞需要软件发行方、服务提供商预留时间做修复准备。目前前沿模型可率先发现这类漏洞，但攻击者也能轻易获取能力相近的公开AI工具，留给防御方的准备时间可能非常有限。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX2tqCic7mGyHstZw9iajYGIg8KOqtJYV7NpcdUr6UvliaG5zQiaQ7HYs7IAAPJFDaeicE1IzyDO46OAh8wMxkqNHMnAPy2YcwWU29ibw/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
设五阶段响应流程  
  
  
第一阶段，维护人员、安全研究人员与技术合作伙伴共同确认漏洞存在，评估影响范围，开发并测试修复补丁。测试环节会专门调用前沿模型验证补丁有效性，排查可绕过补丁的攻击路径，以及漏洞的相关变种。  
  
  
第二阶段，受影响机构提前启动准备工作。发行方可为旧版本软件适配修复方案，主机服务商可提前测试缓解措施，安全厂商可编写Web应用防火墙（WAF）规则——这类规则相当于过滤器，可在恶意流量抵达应用前直接拦截。  
  
  
第三阶段，响应方对接政府机构。本次处置PHP漏洞时，OpenAI及合作伙伴就对接了白宫Super Intelligence Force、国家网络总监办公室、CISA及国际合作伙伴。  
  
  
第四阶段，待修复方案与披露计划确定后，维护方将通知范围扩大至更多发行方、服务商与运营方。  
  
  
第五阶段，补丁正式发布后，响应方公开处置指引，在运营方部署补丁的过程中持续提供支持。  
  
  
Part  
02  
  
明确知情优先级  
  
  
OpenAI的框架通过三个维度判定哪些机构可以提前获取漏洞信息：机构承担的防护职责、完成处置工作所需的准备时长，以及机构的敏感信息保护能力。  
  
  
同类机构适用相同判定标准，最终决定权在维护方手中。所有参与方仅获取履职必需的信息，范围从包含失陷指标的网络日志，到可直接运行的漏洞PoC。  
  
  
OpenAI表示："即便是有限的细节，也可能泄露漏洞的核心信息。"如果漏洞已在野被利用、出现信息泄露、公开补丁暴露了漏洞细节，或是修复进度停滞导致用户面临的风险升高，执行这套流程的团队会重新调整披露时间线。  
  
  
Part  
03  
  
优化框架落地规则  
  
  
OpenAI目前正向维护人员、安全研究人员、运营方征集三类问题的意见：漏洞披露窗口期设置为多长更合理、如何平衡扩大提前通知范围与信息泄露风险、如何修复已停止安全更新的旧版本软件包。  
  
  
它还提出了一个更具前瞻性的问题：如果政府或运营方失去了自有或授权管理系统的管理员权限，是否可以利用相关漏洞进入系统完成补丁部署。  
  
  
OpenAI在框架中描绘了最终愿景：实现常态化全网范围扫描，发现漏洞后要么向网站所有者发送告警，要么直接为系统部署补丁。  
  
  
参考来源：  
  
AI-found bugs are expected to rise, and OpenAI has a five-stage response plan  
  
https://www.theregister.com/2026/10/22/openai_five_stage_vulnerability_response/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
