#  CISA：这个思科 RCE 漏洞已遭活跃利用  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-11-24 08:14  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**美国网络安全和基础设施安全局（****CISA****）提醒联邦机构称，攻击者正在利用一个近期已修复的思科****Expressway****严重漏洞****CVE-2026-76466****实施在野攻击。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
思科Expressway系列设备是企业协作网关（以虚拟机或物理硬件设备形式部署），通过安全连接将内部语音和视频系统与外部网络及远程用户连接起来。  
  
CVE-2026-76466源于使用硬编码凭据以及未能完整验证客户端证书，使得未经身份验证的攻击者可以远程在配置为集群的受影响思科Expressway-C和Expressway-E设备上以root权限执行任意操作系统命令。  
  
思科在10月下旬发布安全补丁解决该漏洞时表示：“拥有这些cookie的攻击者可以通过连接到数据库服务并调用特定的远程过程调用 (RPC) 命令来利用该漏洞。”尽管思科表示其产品安全事件响应团队（PSIRT）并未发现该漏洞存在在野活跃利用或公开可用的概念验证利用代码，但CISA于周五将其标记为已被活跃利用。  
  
CISA还于周二将其加入已知被利用漏洞 (KEV) 目录，并依据具有约束力的操作指令 (BOD) 26-04，要求美国联邦民事行政部门 (FECB) 机构在三天内保护其系统。尽管BOD 26-04仅针对联邦机构，但CISA敦促所有网络防御者，包括私营部门安全团队，优先修复该漏洞，以阻止可能针对其网络的持续攻击。该机构警告称：“这类漏洞是恶意网络行为者的常见攻击载体，对联邦企业构成重大风险。”  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXONiaBmQAZdXA2W2XsQLBcia9bBKwYDcH4MWkZunibeukSCgRq5lepXT7VqSdt5YxjG3WxcYcKGDQCbUoRToPSA3NhhyXpr827ug/640?wx_fmt=gif&from=appmsg "")  
  
**获取互联网暴露设备的 root 权限**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUxcaUTE1SKlMDeNGKGgLhOwsrbTia2Ck8UL9H3PgB5pIS1LT8v4zy4l2E0IaKlfe9cIMHtc7xTI5BEpMZnUCHuOtvwAHHUeKJM/640?wx_fmt=gif&from=appmsg "")  
  
  
  
尽管CISA尚未分享这些持续攻击的细节，但俄罗斯网络安全公司F6周四表示，其发现攻击者使用一个未具名的AI助手成功利用了CVE-2026-7646，并在互联网暴露的思科Expressway设备上获得具有完整root权限的远程命令执行能力。  
  
F6安全研究人员指出：“攻击者最初攻陷了两台面向互联网的思科Expressway设备。通过其中一台，他们获得了对位于该组织内部网络上的另外两个集群节点的访问权限。在获得root访问权限后，攻击者开始准备工具，以便在基础设施中进行进一步的横向移动。他们使用定制的Python脚本扫描网络、发现可访问的SMB资源、与SQL Server和WinRM交互，并探查GitLab仓库，以窃取凭据和其它敏感信息。”  
  
两周前，CISA还将一个高严重性的思科BroadWorks漏洞（CVE-2026-20320）标记为在攻击中被活跃利用，并要求联邦机构在11月24日前修复系统。  
  
自2021年11月以来，CISA已在其活跃利用漏洞清单中纳入103个思科漏洞，其中7个还被勒索软件团伙在野使用。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[CISA：Linux 严重漏洞已用于 PHP2Shell 攻击活动](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527580&idx=1&sn=86377c86b3c45bfd471679fd7a089ec4&scene=21#wechat_redirect)  
  
  
[CISA：PHP2Shell 漏洞已遭利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527536&idx=1&sn=6c207a14ce6a2416be690ad8bcc931a9&scene=21#wechat_redirect)  
  
  
[2026 Q3 CVE 漏洞利用加速，CISA KEV 新增100个新条目](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527332&idx=2&sn=bf3e1e822b634ed0a2d9fb0665e5655b&scene=21#wechat_redirect)  
  
  
[CISA：三个Linux 内核漏洞已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527207&idx=1&sn=ee496b1b2360baa934d52866a4de28eb&scene=21#wechat_redirect)  
  
  
[CISA：速修复已遭活跃利用的 TrueConf Server 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526945&idx=1&sn=49c1fbcc51b2b499e529c7d3643569b9&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/cisa-flags-critical-cisco-expressway-rce-flaw-as-actively-exploited/  
  
  
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
  
