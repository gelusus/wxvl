#  CISA：Linux 严重漏洞已用于 PHP2Shell 攻击活动  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-10-30 09:31  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**美国网络安全和基础设施安全局****(CISA)****提醒修复一个高严重性的****Linux****内核权限提升漏洞（****CVE-2026-46300****），称攻击者正在组合****PHP2Shell****漏洞形成利用链。**  
  
该  
 Linux   
内核提权漏洞被称为  
 Fragnesia (  
该词意为：分片处理中的状态丢失  
/  
标记丢失  
)  
，源于  
 Linux XFRM ESP-in-TCP   
子系统中的一个逻辑缺陷，可导致攻击者以  
 root   
权限运行恶意代码。  
Fragnesia   
属于  
 Dirty Frag   
漏洞类型，已有  
PoC  
，可被本地攻击者用于在  
 2026   
年  
 5   
月  
 13   
日之前发布内核的主要  
 Linux   
发行版上获取  
 root   
权限。  
CISA  
本周三将该漏洞纳入  
KEV  
，要求联邦机构在周六  
 10   
日前完成修复。  
  
虽然  
CISA  
未透露正在进行的  
 Fragnesia   
攻击的细节，但指出威胁行为者可能将其与两个严重的  
 PHP   
漏洞（  
CVE-2026-109618   
和  
 CVE-2026-109619  
，统称  
 PHP2Shell  
）链式利用，在未修复的系统上部署  
 webshell  
。  
CISA   
表示：  
“  
该漏洞可与  
 CVE-2026-109618   
和  
 CVE-2026-109619   
链式利用，植入  
 PHP webshell   
并以  
 root   
身份运行命令。  
”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUyTa4MsSSJbQVib8UZCnzXnduib9ekbHfnTvEdqBBUWJ3qyTichDoWiafofKOdGRzcltCAcBEAUuHkeOttT9dE9KjEVHnrXuwjxhU/640?wx_fmt=gif&from=appmsg "")  
  
**Fragnesia 与 PHP2Shell 的利用链**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfU27Qw8dq2kmQG8wEV61H3CuLt3mEg5UWbCtkm1OxnjydeFicjibqRRLMI6Nh2TB0LiaG087iaSVzZwyibGqicm5bpWYiaNWSErfM64P8/640?wx_fmt=gif&from=appmsg "")  
  
  
  
虽然  
 CISA   
未就攻击者链式利用  
 Fragnesia   
和  
 PHP2Shell   
提供更多信息，但  
 Dream Security   
公司分析师在周一的一份报告中表示，这些漏洞被用于入侵二十多台服务器，其中包括三台属于美国某政府附属机构的服务器。  
  
他们表示：“  
Dream   
研究人员识别出一次完整的攻击行动，包括对  
 PHP2Shell   
利用链（  
CVE-2026-109618/CVE-2026-109619  
）的定制武器化，搭配  
 Fragnesia   
内核权限提升（  
CVE-2026-46300  
）、目标列表、利用日志以及约  
 40   
个定制后渗透脚本。这包括一场已在位于美国、日本、中国台湾和韩国的  
 25   
台服务器上获取  
 root   
权限的攻击行动的痕迹。获取  
 root   
权限后，攻击者将目标锁定在美国某政府附属环境中的三台特定服务器上。  
”  
  
CISA   
还在周一将  
 PHP2Shell   
漏洞纳入  
 KEV  
，并要求联邦民事行政部门  
 (FCEB)   
机构在周四前确保网络安全。虽然美国网络安全机构尚未就正在进行的  
 PHP2Shell   
攻击提供进一步细节，但  
 Perplexity   
公司安全工程师  
 Adel Karimi   
在周末确认了该漏洞已遭在野利用，此前其蜜罐记录到针对运行  
 PHP 8.4.26   
的  
 Apache mod_php   
安装的远程代码执行证据。  
  
目前，  
CISA  
未能立即就可能在攻击中链式利用  
 Fragnesia   
和  
 PHP2Shell   
的威胁行动者提供更多信息。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[CISA：PHP2Shell 漏洞已遭利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527536&idx=1&sn=6c207a14ce6a2416be690ad8bcc931a9&scene=21#wechat_redirect)  
  
  
[XSS2Shell：WordPress Core 漏洞利用预认证登录XSS实现 PHP RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526848&idx=1&sn=b29b6dc57efb2687f969776b26ce49fe&scene=21#wechat_redirect)  
  
  
[2026 Q3 CVE 漏洞利用加速，CISA KEV 新增100个新条目](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527332&idx=2&sn=bf3e1e822b634ed0a2d9fb0665e5655b&scene=21#wechat_redirect)  
  
  
[CISA：三个Linux 内核漏洞已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527207&idx=1&sn=ee496b1b2360baa934d52866a4de28eb&scene=21#wechat_redirect)  
  
  
[CISA：速修复已遭活跃利用的 TrueConf Server 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526945&idx=1&sn=49c1fbcc51b2b499e529c7d3643569b9&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/cisa-warns-of-fragnesia-linux-flaw-chained-in-php2shell-attacks/  
  
  
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
  
