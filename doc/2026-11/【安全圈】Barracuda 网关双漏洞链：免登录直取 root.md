#  【安全圈】Barracuda 网关双漏洞链：免登录直取 root  
 安全圈   2026-11-06 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
在 Barracuda Email Security Gateway 这台邮件过滤设备上，攻击者不需要账号、不需要登录，就能以最高权限 root 执行命令。两个漏洞串成一条链：一个把未认证请求抬到管理员级，一个把配置项变成 shell 命令。厂商周一发布补丁，watchTowr 周四公开完整攻击链  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyGNZg5iaBhDV871VhymNH07d5e19qo9Of4vK1p74jn5ib1uIwz7khDRzP6HReO1lLNiaAWiaOCTVmmBaug9sgRXX03J1MKgp3KmbQs/640?wx_fmt=other&from=appmsg "")  
  
  
## 🧩 两个洞，各司其职  
- **CVE-2026-116606**  
（授权绕过，Mandiant 评分 9.3/10）：未认证攻击者可恢复一份构造的配置备份，改掉设备全局设置。仅影响 Email Security Gateway  
  
- **CVE-2026-111560**  
（命令注入，Mandiant 评分 8.6/10）：以 root 身份执行命令，原本需要管理员账户。该漏洞波及另外五款产品，但在那些产品上仍需管理员权限才能利用  
  
> 两个漏洞在 Email Security Gateway 上叠加，效果是免登录 root RCE  
  
## 🔬 watchTowr 拆解的完整链条  
  
watchTowr 沿着设备代码把链路走了一遍。第一步，一次失败的密码修改不会终止请求，而是给请求者返回一个低权限角色——备份恢复功能就此对未登录者敞开  
  
第二步，恢复的备份数据里带着一个 timezone 字段，设备后续把它当 shell 命令以 root 权限执行。备份本身有校验和保护，但计算校验和的密钥硬编码在代码里，攻击者可以直接伪造出合法校验和  
  
演示中，watchTowr 用这条链以 root 身份向设备写入文件，全程没有登录凭证  
## ✅ 处置清单  
- Email Security Gateway 升级到 9.4.0.030 或更高版本；其余受影响产品按官方公告列出的修复固件版本升级  
  
- 暂时打不了补丁的，把设备管理界面限制在可信内网  
  
- 关闭过自动补丁的，重新打开  
  
watchTowr 开源了检测工具（GitHub 仓库 watchTowr-vs-Barracuda-RCE-CVE-2026-116606-CVE-2026-111560），用于判断设备是否受漏洞影响，未公开利用代码  
## 📌 现状  
  
截至本周，两个漏洞都未进入 CISA 的已知被利用漏洞目录（KEV），也没有公开报告把它们与真实攻击挂钩。Barracuda 安全公告列出了各产品的修复固件版本  
  
这台设备上一次出圈是 2023 年：UNC4841 利用零日漏洞 CVE-2023-2868 发起过一轮大规模攻击。那起事件与本周修复的两个漏洞没有关联，目前也没有证据把新漏洞与任何攻击者挂钩  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】9.8分Linux内核高危RCE！无需认证隔空打穿](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=1&sn=dfbd72f6698a155131d20cada69f5589&scene=21#wechat_redirect)  
  
  
  
[【安全圈】隔空投送秒越狱？安全团队演示攻破iOS锁定模式](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=2&sn=c3a9e1733d1b652c93ff442f8514bdf2&scene=21#wechat_redirect)  
  
  
  
[【安全圈】零知识架构翻车？知名密码管理器mSecure疑似遭拖库](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=3&sn=de8663b781d1914ceb2dc416f348a11a&scene=21#wechat_redirect)  
  
  
  
[【安全圈】面试题下毒pyenv！朝鲜黑客假借招聘攻破币安内网](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079687&idx=1&sn=2698a551a0d839d42d48e0600cd1befd&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
  
