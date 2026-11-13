#  Palo Alto 修复PAN-OS RCE 漏洞，可导致任意代码执行  
Abinaya
                    Abinaya  代码卫士   2026-11-13 09:11  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Palo Alto Networks****公司披露了运行在****Panorama****上****PAN-OS****软件中的一个远程代码执行漏洞（****CVE-2026-0330****），它导致具有网络访问权限的已认证管理员以****root****权限执行任意代码。**  
  
该公司在2026 年 11 月 11 日发布公告，敦促用户修复受影响的 Panorama 部署，Cloud NGFW、Prisma Access 和独立的 PAN-OS 防火墙不受影响。  
  
Palo Alto Networks 表示未发现恶意利用行为，并提到该漏洞由公司内部研究员 Daniel Voce 发现并报送。该漏洞被归类为 CWE-94，即代码注入弱点。成功利用该漏洞可让管理员以 root 权限运行代码，而不是停留在正常的管理限制内。已发布的评估将高权限列为利用条件，并表示不需要用户交互。  
  
受影响的 Panorama 安装不需要特殊配置就易受攻击，不过利用该漏洞需要管理员身份验证和网络访问权限。它并非互联网上任何人都能直接对可访问系统发起的未认证攻击。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfU0fsQ05s4PMB0MEVjHpPyZqosA1Qw3waOXsAk5plcPnYdgicDCrTWmQMcI0dyykdOY7CK65LrkxHd13sn6hZ25330xIdvNTiccM/640?wx_fmt=gif&from=appmsg "")  
  
**PAN-OS 远程代码执行漏洞**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXyEfXclrxcrdISMjyZkVcJqAnH4MOhXjY6mibaKNVnNcEIRLUdgalf9zYD8y5Z7E1qFftNl8cxYdvmlK7wmjpAEMCcvHgpke28/640?wx_fmt=gif&from=appmsg "")  
  
  
  
Palo Alto Networks 公司将该漏洞评为中危级别，CVSS 威胁评分为 6.1，基础评分为 8.6。供应商列出的紧迫性为中等，但警告称当允许从外部互联网地址进行管理访问时风险最高。  
  
该漏洞与之前报告的 PAN-OS root 代码执行漏洞 CVE-2026-0322 无关。后者已存在利用报告。这些攻击不应被视为攻击者正在利用这个新披露的 Panorama 漏洞的证据。  
  
修复方案覆盖 Panorama 上的 PAN-OS 12.2、12.1、11.2、11.1 和 10.2 版本。对于每个分支中列出的最新维护线，修复版本包括 12.2.4-h2、12.1.10-h4、11.2.16-h2、11.1.16-h5 和 10.2.18-h14。管理员应使用公告中针对自身已安装版本的升级指南。  
  
早期维护线也获得额外修复方案。PAN-OS 12.1 的选项包括 12.1.4-h14 和 12.1.7-h12。PAN-OS 11.2 的选项包括 11.2.4-h24、11.2.7-h24、11.2.10-h18 和 11.2.13-h5。单独的修复程序还涵盖较旧的 11.1 和 10.2 维护版本。  
  
不受支持的版本应迁移到受支持的修复版本。目前不存在可以修复底层问题的应变方法。Palo Alto Networks 公司建议仅允许受信任的内部 IP 地址访问管理权限。该公司在管理安全指南中警告不要将防火墙或 Panorama 管理接口暴露到互联网或其它不受信任的网络。将访问限制到专用跳板机可以进一步减少暴露面。之前关于被暴露的 PAN-OS 管理接口的报道强调了为什么这个边界值得关注。虽然访问限制可以降低风险，但不能替代安全更新。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[Palo Alto：PAN-OS RCE 漏洞已遭利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527392&idx=1&sn=44cddd7e186a3bcdca81b7ae6690583f&scene=21#wechat_redirect)  
  
  
[Palo Alto 提醒注意严重的 PAN-OS RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525932&idx=2&sn=a1f8acec7865ec3eec445777c4ad6251&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks User-ID 凭据代理漏洞以明文形式暴露密码](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524006&idx=2&sn=7d834750d24e4c7125f06fe6c19d570e&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks 修复多个提权漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247523281&idx=2&sn=c4ed265127f77866bb9d4f397428f552&scene=21#wechat_redirect)  
  
  
[Palo Alto Networks 修复PAN-OS 中的认证绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522232&idx=2&sn=3bc7a4466c3c33ff643ca604524fa401&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/palo-alto-networks-pan-os-rce-vulnerability/  
  
  
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
  
