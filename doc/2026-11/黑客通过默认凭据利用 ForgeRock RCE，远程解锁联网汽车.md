#  黑客通过默认凭据利用 ForgeRock RCE，远程解锁联网汽车  
Guru Baran
                    Guru Baran  代码卫士   2026-11-10 09:31  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**研究人员将默认的****admin:admin****凭据纳入利用链在两家汽车制造商的生产环境****ForgeRock****服务器上运行代码，并远程解锁联网汽车。**  
  
这些漏洞于  
2026  
年初被发现，并于  
11  
月  
7  
日披露，它们还暴露了车辆位置，并支持远程发动机指令。研究人员提到，两家未具名的汽车制造商在收到报告后约两小时内修复了该问题，并轮换了受影响的凭据。它们的身份系统服务于多个地区，但研究人员仅使用美国账户和车辆测试了完整的账户接管和车辆访问链。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXWpuUw4vWqzScg24icMKjJWOzicckG0ewiaq5K1a4mVoCMHAF15OIWHlAu4gkb5BgMa6cD7LjjPnX1cXOIxqW2IqCZAqyITicejHo/640?wx_fmt=gif&from=appmsg "")  
  
**ForgeRock攻击链**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXib0OZhmSZgNiabnaj4iamBVJQpCHD0mQtx84BWl2FnfYRbMIG9ichY2I7PoE3fS9FUBTrkmqw2liccRXnrchO18DhkpLaibHCv386k/640?wx_fmt=gif&from=appmsg "")  
  
  
  
调查始于车主使用的  
Android  
应用。这两款应用都依赖  
ForgeRock Access Management  
处理登录会话，依赖  
OpenIDM  
处理客户记录。研究人员追踪电子邮件验证请求后，将目标转向管理服务，包括与  
OpenIDM  
一同运行的  
Apache Felix Web Console  
。  
  
Web  
应用防火墙阻止了对这些服务的直接请求。然而，经过特殊编码的路径使请求看起来指向静态文件，而后端  
Web  
服务器  
Jetty  
却将其解释为控制台请求。这种  
URL  
处理上的不一致暴露了隐藏的管理界面，而其中文档记载的默认用户名和密码仍然有效。  
  
研究人员通过访问  
Felix  
的  
ConfigMgr  
界面的权限，创建了一个运行  
Groovy  
代码的  
OpenIDM  
计划任务，并利用它在运行中的服务内安装了一个小型  
Java  
组件，从而获得了一个受控的命令端点。这证实了远程代码执行发生在生产身份认证进程内，而非汽车本身内。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVfTOHT11Oqh1wTt7azibSV6a8sc882a7A5Hv66UvfXtecFsbTNrqD2sjbudtibpgjlWd6KRU0THlZ6SsUiaCGr9iac2iaSbooHYYfY/640?wx_fmt=gif&from=appmsg "")  
  
**从服务器访问到车辆指令**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXRwoGlKicJpc7VMonDkFvqKQX7TRRVMjAxrLnUWdN5wsEDHLY8LLdZmcmNichSnP8AaibCOHUQkfJtoT49qP0xgibFK2rPrMztvxI/640?wx_fmt=gif&from=appmsg "")  
  
  
  
研究人员发布技术报告表示，服务器访问暴露了允许向客户记录发起本地请求的管理凭据。研究人员通过更改一个测试账户的电子邮件和密码，收到了验证码，并完成了正常登录流程。随后将会话兑换为车辆  
API  
所接受的令牌。  
  
这些令牌暴露了该账户的车辆、位置、状态以及支持的远程控制功能。在车主位于测试车辆旁边的情况下，研究人员证实了解锁、发动机启动、发动机停止和锁车功能。  
  
恢复原始电子邮件并未使新令牌失效。此前研究人员曾开展起亚远程访问研究和斯巴鲁  
STARLINK  
安全漏洞研究。  
  
Ping Identity  
公司的生产环境加固指南建议移除或保护  
Felix  
控制台，并更改默认凭据。该公司还提醒称，控制台身份验证与  
IDM  
身份验证是分开的。该案例表明，为什么在防火墙上阻止管理  
URL  
不能替代对背后服务本身的安全加固，尤其是当该服务控制着客户身份和对联网车辆的访问时。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[15年前的雷诺汽车 Linux 导航系统软件被破解 可执行任意 root 命令](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527686&idx=1&sn=f75e5ce428d9d4a5a9d306af91430362&scene=21#wechat_redirect)  
  
  
[在线阅读版：《智能网联汽车云平台漏洞分析报告》全文](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524907&idx=1&sn=56ac025aa6a6fcc0f1243163f93a12bc&scene=21#wechat_redirect)  
  
  
[XCharge C6电动汽车充电桩存在多个严重漏洞，可导致任意代码执行](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526201&idx=2&sn=94f249f0b82ed4528faa20db71165ca7&scene=21#wechat_redirect)  
  
  
[电动汽车充电网络告警：Everon OCCP 后端系统存在严重漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525349&idx=1&sn=352cbd47ed06c60804138658fa37fece&scene=21#wechat_redirect)  
  
  
[2026 Pwn2Own 东京汽车大赛落下帷幕，Master of Pwn 诞生](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524969&idx=1&sn=fbd3e630ba31d2f0118b90ad0779d309&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/forgerock-attack-chain/  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
