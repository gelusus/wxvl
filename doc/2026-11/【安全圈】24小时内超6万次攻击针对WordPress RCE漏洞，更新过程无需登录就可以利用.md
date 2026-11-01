#  【安全圈】24小时内超6万次攻击针对WordPress RCE漏洞，更新过程无需登录就可以利用  
 安全圈   2026-11-01 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyEsKmls298Gh14Lica4YlISTN9r1JicLfRWBtLlnSwMo3hia3k67hlyTt9ePDia0VQkEDnQgZJXdGhYXHRR6PDRaMX0kbblnouWdNg/640?wx_fmt=jpeg&from=appmsg "")  
  
攻击者正瞄准WordPress核心的一处远程代码执行（RCE）漏洞，站点在软件更新过程中可能因此被入侵。根据Wordfence提供的威胁情报快照，相关防护规则24小时内共拦截60471次攻击尝试。  
  
上述数据仅统计被拦截的攻击尝试，不代表已确认的站点入侵事件，也无法反映实际遭受攻击的独立站点总数。  
  
WordPress于2026年10月28日发布7.1.4版本，共修复22个安全漏洞，除本次的代码执行漏洞外，还覆盖存储型跨站脚本、信息泄露、权限配置不当等多类风险。  
  
WordPress官方公告敦促管理员立即完成更新，并向报告该升级流程漏洞的研究员Umut Bayram致谢。  
## 漏洞源于更新流程竞态条件，无需登录就可以发起攻击  
  
该漏洞属于竞态条件类漏洞，攻击者必须精准命中临时安装流程的特定时间窗口才能利用成功。与需要窃取密码或持有站点账号才能利用的漏洞不同，攻击者无需登录就可以发起攻击。不过，漏洞成功利用的前提是目标站点正处于完整的WordPress核心升级过程中。升级过程中，WordPress会将更新文件解压到临时暂存目录。这一阶段，由于临时文件夹名称可被预测、安装文件可公开访问，会短暂暴露一个独立的WordPress安装程序。  
  
攻击者一旦访问到该安装程序，就可以将其连接到自己控制的数据库。完成这一步后，攻击者就能在服务器上执行任意代码。  
  
据Cybersecuritynews.com此前对WordPress 7.1.4安全更新的报道，本次补丁主要做了三处调整：为临时文件夹名称添加16位随机字符、恢复index.php的防护机制、禁止从工作目录复制wp-config.php文件。上述修复针对的是核心升级暂存流程的暴露问题，并非第三方插件漏洞。  
  
这类漏洞的攻击窗口通常仅持续数秒，利用难度较高但并非完全无法成功。攻击者可以通过发送大量重复请求，提升命中站点更新文件暴露窗口的概率。Wordfence已监测到补丁发布后出现多起漏洞利用尝试，说明尽管有严格的时间条件限制，攻击者仍在积极测试该漏洞的可行性。Cybersecuritynews.com此前也报道过这批针对WordPress的活跃利用尝试。需要明确区分防火墙拦截的攻击请求与成功的代码执行事件：高请求量仅说明攻击活动活跃，并不代表攻击者已经成功入侵了同等数量的网站。  
  
根据Wordfence发布的公告，其Premium、Care、Response订阅客户在10月28日（即版本发布约3小时后）就已收到对应的防火墙防护规则。免费用户则按照标准的30天延迟规则，将于11月27日收到该规则。管理员不应等待防护规则推送，应优先安装官方补丁。  
## 及时安装官方安全补丁，多维度验证防护效果  
  
站点所有者应登录后台，进入更新页面，确认站点已安装WordPress 7.1.4版本。使用旧分支版本的站点，需安装对应分支的安全更新包。官方版本文档列出了所有已修复该漏洞的版本，包括7.0.8、6.9.11，向后移植补丁的覆盖范围最低至4.7.39。WordPress 4.6及更早版本已停止接收安全更新。  
  
符合支持条件的站点会通过后台自动更新推送对应安全版本，但管理员不能默认站点已受保护，必须手动确认更新完成。官方文档明确说明，只有最新版WordPress会获得持续主动支持。旧分支的修复补丁仅提供过渡更新路径，不能作为推迟版本维护的理由。  
  
管理员还可以限制公众对wp-content/upgrade/目录的访问权限，作为额外防护手段。仅靠插件防火墙可能无法覆盖解压后的安装程序，因为该程序的加载流程独立于主站点。安装核心安全更新，仍是修复该升级流程漏洞的最核心措施。  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】Gitee 725 万个公开仓库被扫出 26400 个有效凭证，9473 个是阿里云密钥](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079646&idx=1&sn=b67092a6e295dc9135d4f0304a210caa&scene=21#wechat_redirect)  
  
  
  
[【安全圈】无印良品遭黑客入侵，约 2726万条客户信息外泄](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079646&idx=2&sn=cb164b43f8161ed86d04fc5f85f660b7&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Hugging Face 恶意软件活动卷土重来，劫持 GPU 进行加密货币挖矿](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079646&idx=3&sn=d482d796b6a983027c050b0687351a7c&scene=21#wechat_redirect)  
  
  
  
[【安全圈】官方插件遭投毒！Claude Code被植入SSH后门](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079634&idx=1&sn=5f118197d2668a55db0755ec56254013&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
  
