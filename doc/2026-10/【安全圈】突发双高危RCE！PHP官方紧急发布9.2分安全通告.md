#  【安全圈】突发双高危RCE！PHP官方紧急发布9.2分安全通告  
 安全圈   2026-10-21 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事件快报**  
：北京时间 10 月 21 日，PHP 安全团队紧急确认两个关键远程代码执行（RCE）漏洞，CVSS v4.0 评分为 9.2。安全社区将该漏洞组合称为“php2shell”。官方排定于 UTC 时间 10 月 21 日 14:00 发布计划外应急补丁。目前在野利用尚未见报，但全球仍有超过 88 万台暴露在公网的 PHP 8.1 及更早版本主机处于停止支持（EOL）状态，无法获得任何官方补丁。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyFn81KwibNUXlMxDQ6ozTHOTTmsDbBjRNyNbBNXkeu59hlf30gOpvAa5GJ292mby4AK9NINadichkBQWuymcA8iaSWtGtCdqCtQ2E/640?wx_fmt=other&from=appmsg "")  
## 🔍 泄露引发突发预警：从社区爆料到官方提前发版  
  
此次安全事件最初始于 10 月 20 日 Hacker News 社区的一则匿名爆料。爆料称 PHP 存在两个 CVSS 10.0 的 RCE 漏洞，波及 PHP 5 及之后的全系版本，并声称将于 10 月 22 日全网公开。  
  
随后，知名网络安全机构 watchTowr 在社交平台证实了漏洞的真实性，并指出该漏洞原处于禁运协同披露状态（Embargo）。  
  
信息外泄直接打破了披露节奏。PHP 官方团队在 UTC 时间 10 月 20 日 19:33 迅速上线预警通告，将正式安全补丁的发布时间提前至 10 月 21 日 14:00 UTC，并明确表态：若细节被进一步提前公开或出现野外利用迹象，官方将立即跳过倒计时直接释出修复。  
## ⚡ 漏洞特征与受影响版本阵列  
  
安全研究界当前为两个漏洞分配的追踪标识为 **CVE-2026-109618**  
 与 **CVE-2026-109619**  
。综合 watchTowr、Huntress 及安全专家 Kevin Beaumont 的研判，技术面核心事实如下：  
- **漏洞危害等级**  
：PHP 官方评定为 CVSS v4.0 9.2 分（属于 Critical 严重级别），攻击者可远程触发未授权代码执行。  
  
- **在野利用与 PoC 状态**  
：截至发稿，CISA KEV 目录尚未收录，公开网络尚未监测到可用的公开 PoC 代码，但已有大量自动化测绘探测活动。  
  
- **官方受支持分支**  
：仅有 PHP 8.2、8.3、8.4 和 8.5 四个分支将获得维护更新，对应新版本预计为：  
  
• PHP 8.5.12  
  
• PHP 8.4.27  
  
• PHP 8.3.36  
  
• PHP 8.2.35  
  
## ⚠️ 88万台 EOL 资产暴露：无厂商补丁可用  
  
本次事件最为致命的痛点在于存量资产的弃维现状。所有早于 PHP 8.2 的分支（包括 PHP 8.1、8.0、7.x 及 5.x）均已结束生命周期（End of Life），PHP 官方明确不会向这些旧分支反向移植安全补丁。  
  
根据安全分析团队 Mallory 基于 Shodan 检索数据的统计：  
- 全球公网可检索到约 150 万台在 HTTP Banner 中明文回显 PHP 版本的主机。  
  
- 其中多达 **88.4 万台**  
主机运行的正是 PHP 8.1 或更早版本，占比高达 **59%**  
。  
  
- 另有约 74 万台主机隐藏了具体次版本号。一旦攻击载荷全网扩散，这批未获维护的旧系统将直接沦为无防御攻击目标。  
  
## 🛡️ 应急缓解与网络隔离清单  
  
在官方更新完全就绪、技术细节正式揭晓前，企业安全运维团队应参考 Huntress 与 CERT-EU 给出的应急隔离指引完成处置：  
- **第一步：全面清点 PHP 运行时基线**  
：梳理全网所有生产服务器、反向代理后端的 FastCGI、PHP-FPM 与容器镜像版本，标记所有运行 PHP ≤ 8.1 的遗留实例。  
  
- **第二步：收紧公网接入访问控制**  
：非必要严禁将管理后台或 PHP 处理节点直接对公网开放。结合堡垒机、内网 VPN 或云安全组实施白名单访问控制。  
  
- **第三步：升级至受支持分支**  
：将业务环境平滑升迁至 PHP 8.2+ 分支，并做好拉取 8.2.35、8.3.36、8.4.27、8.5.12 补丁包的自动化测试准备。  
  
- **第四步：WAF 规则与行为监控就位**  
：关注主流安全厂商基于 PHP 异常调用链路的虚拟补丁规则，监控 PHP 子进程异常衍生（如 bash/sh 衍生、反向 shell 建立等异常行为）。  
  
 END    
  
  
阅读推荐  
  
  
[【安全圈】双11前紧急叫停！京东JSRC全面暂停外部漏洞测试](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=1&sn=64985dae9e27b64b33a95e031a7e8784&scene=21#wechat_redirect)  
  
  
  
[【安全圈】克隆代码就中招！94个GitHub开源库遭假字体投毒](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=2&sn=5347b52ee72fe6b67a3af750295fed29&scene=21#wechat_redirect)  
  
  
  
[【安全圈】四漏洞连环击穿！Plex遭武器化利用植入挖矿木马](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=3&sn=c59fa44ef0a5791429f5ba4c7b36e7e4&scene=21#wechat_redirect)  
  
  
  
[【安全圈】连密码都不用！博通WiFi曝0day隔空秒拿ROOT](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=1&sn=21c4f345cc81f0721be1f3a465c9026c&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
