#  【安全圈】黑客利用 Cisco SD-WAN 漏洞部署定制的 KeyDrift 恶意软件  
 安全圈   2026-11-01 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
黑客  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/sbq02iadgfyGNLXTGQsz2wLSJXEQgJHibPyXKtdxSBkE23ua7dibwtvwEOUWjrBxrmNXazRewzcjicjvxJBTy6jQwOhGl5K5zHvBqEPV3iaUjQ6E/640?wx_fmt=png&from=appmsg "")  
  
一个被追踪为 UAT-11473 的新识别高级持续性威胁（APT）组织被观察到利用 CVE-2026-20127——Cisco Catalyst SD-WAN Controller 中的一个身份验证绕过漏洞——来部署一个名为 KeyDrift 的定制恶意软件框架。  
  
CVE-2026-20127 是一个最高严重性漏洞，Cisco 于 2026 年 2 月修补，它允许远程攻击者获得对 Catalyst SD-WAN Controller 和 Manager 的特权访问并添加恶意对等节点。  
  
UAT-8616 至少自 2023 年起将该漏洞作为零日漏洞利用，近期研究还将其利用与中国 Volt Typhoon 国家支持黑客组织关联起来。  
  
据报告该活动的 Cisco Talos 研究人员称，UAT-11473 威胁集群似乎与 Volt Typhoon 的 "Silent Train" 隐秘访问网络存在重叠，但他们未建立确定的关联。  
  
在 Lumen 昨日发布的一份报告中，研究人员揭示 Silent Train 框架让 Volt Typhoon 在进行零日漏洞攻击时得以规避检测。  
  
**部署 KeyDrift**  
  
Cisco 研究人员观察到的攻击针对太空和卫星服务、国防和航空航天技术以及政策研究组织，使用了 KeyDrift 恶意软件，这是一个旨在运行于 Cisco SD-WAN 控制器实例上的定制恶意软件框架。  
  
"Talos 以高置信度评估，该框架旨在提供长期持久访问并自动窃取敏感密钥材料，" Cisco Talos 解释道。  
  
"KeyDrift 在 SD-WAN 控制平面进程内运行，能够拦截解密后的控制流量、提取控制会话元数据和数据平面密钥材料，并使用加密记录暂存或传输收集到的材料。"  
  
KeyDrift 有两个主要组件 PolyFlame 和 LittleFilch，Talos 看到了持续开发的迹象，包括新的反调试机制和多样化的进程冒充。  
  
PolyFlame 使用修改后的服务启动脚本和一个注入器进行部署，在合法的 vdaemon 进程内运行。  
  
它支持通过一个 TinyShell 风格的组件执行命令、文件上传和下载，以及在攻击者的命令与控制（C2）服务器和多个对等节点之间中继通信。  
  
LittleFilch 是一个控制会话拦截模块，它钩住 SSL_read 以在控制平面消息于控制器内解密后检查它们。  
  
对于预定义的边缘节点，它提取 IP 标识符和一个 32 字节字段，Talos 评估其为 AES-256 IPsec 数据平面密钥材料。  
  
它使用 RSA 加密保护收集到的信息，将其保存到 / var / tmp / vdaemon.bin，然后通过 UDP 传输给攻击者。  
  
Talos 以高置信度认为，KeyDrift 旨在维持对 Cisco SD-WAN 控制器的持久访问、拦截控制平面流量、提取加密密钥，并安全地暂存或窃取被盗数据。  
  
Cisco 建议系统管理员应用 CVE-2026-20127 的可用安全更新，并查阅厂商针对 Catalyst SD-WAN 部署的专用加固指南。  
  
  
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
  
  
  
