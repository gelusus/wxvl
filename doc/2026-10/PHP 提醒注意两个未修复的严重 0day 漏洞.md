#  PHP 提醒注意两个未修复的严重 0day 漏洞  
Lawrence Abrams
                    Lawrence Abrams  代码卫士   2026-10-21 06:44  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**PHP****安全团队正在提醒管理员称，准备在明天安装紧急安全更新，修复两个严重漏洞，因为据称这些漏洞的信息在计划披露日期之前就已经泄露。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
PHP   
为许多网站和   
Web   
应用程序提供支持，包括内容管理系统、电子商务平台和自定义   
Web   
服务等。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVnbgNpx2LoeXORomcxsFIobfd7j6fuLqZrabfcHYUcriaNlaibnlSUVsc1tzen9BEvTQ2JeMYXKHuODibsOMVquDaJpGbJoXcqkY/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞的 CVE 编号和评分**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXknicwyU99S2UTJiaRicdchfiaAU9NC9z85Py0niawU8hZXHFUCtQOIqJcMmwC8u4mFIE30h7yZu74eBfice5DVlhrlQuywSO0UMQrw/640?wx_fmt=gif&from=appmsg "")  
  
  
  
PHP   
在本周二发布公告但尚未披露这些漏洞的任何技术细节，包括受影响的是哪些  
PHP  
版本、如何利用这些漏洞，以及哪些版本将包含修复。  
PHP为  
这两个漏洞给出的CVSS v4评分为9.2。  
  
而网络安全公司   
watchTowr   
的安全研究人员表示，这些被泄露的漏洞编号为  
CVE-2026-109618  
和  
CVE-2026-109619  
，是远程代码执行  
 (RCE)   
漏洞，据信影响  
PHP 5  
、  
6  
、  
7  
和  
8  
版本，且  
CVSS  
评分为满分  
10  
分。另外，该公司表示已经成功复现其中一个漏洞  
CVE-2026-109618  
，但在补丁发布前不会发布这些细节。该公司还表示并未有证据表明这两个漏洞已遭在野利用。  
  
而在此之前，  
Y Combinator 旗下  
的  
Hacker News  
论坛上有一篇现已被删除的帖子，其中一位名为  
“petecooper”  
的用户声称，两个严重的  
PHP  
远程代码执行漏洞将于  
10  
月  
22  
日披露。这篇帖子声称，这两个漏洞的  
CVSS  
评分均为  
10.0  
，并且影响可追溯至  
PHP 5  
的各个  
PHP  
版本。该帖子还将这些漏洞标识为  
CVE-2026-109618  
和  
CVE-2026-109619  
，与  
watchTowr  
后来分享的编号一致。  
  
PHP  
开发团队宣布计划于  
10  
月  
21  
日周三  
14:00 UTC  
（美国东部时间上午  
10:00  
）发布安全更新，修复这两个漏洞。  
PHP  
安全团队还向   
ANNOUNCE   
邮件列表发送了通知，敦促管理员准备在安全更新一经发布后立即安装。  
  
PHP  
安全团队警告称：  
“  
如果任一漏洞的详细信息被公开，或出现在野活跃利用的证据，我们保留提前发布的权利。  
”  
  
PHP  
安全团队表示，当安全更新可用时，将发布完整的技术细节、受影响版本和修复版本。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[XSS2Shell：WordPress Core 漏洞利用预认证登录XSS实现 PHP RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526848&idx=1&sn=b29b6dc57efb2687f969776b26ce49fe&scene=21#wechat_redirect)  
  
  
[PHP Composer 多个新漏洞可导致任意命令执行](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525778&idx=1&sn=f575dcb35ac2b3091d3f1a2359e1ae4d&scene=21#wechat_redirect)  
  
  
[黑客在Laravel 生态系统中隐藏恶意PHP包，发动供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525357&idx=1&sn=8a6f6163370e2aa35430f8dcf25ac760&scene=21#wechat_redirect)  
  
  
[这个严重的PHP漏洞正遭大规模利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522450&idx=1&sn=d780d2de95939199fea4d699bb1d2080&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/php-warns-admins-of-two-critical-flaws-emergency-patches-coming-tomorrow/  
  
  
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
  
