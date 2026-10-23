#  PHP重大漏洞危及多国政府网站：根因、泄露事件与应急处置方案  
原创 安全值班室
                    安全值班室  安全值班室   2026-10-23 01:00  
  
10 月 21 日 14:00 UTC，PHP 连发四个安全版本。  
  
修的是两个 RCE，CVSS v4.0 都是 9.2。不需要登录，远程就能打。  
  
同一天，荷兰国家网络安全中心把自己的官网关了。荷兰众议院、芬兰部分政府站点、法国 CNIL、日本部分政务平台，也陆续预防性下线。  
  
不是被攻破，是怕被攻破。  
  
预防性下线清单　据公开报道  
  
荷兰国家网络安全中心　官网自行离线  
  
荷兰众议院　部分站点暂停  
  
芬兰　部分政府站点下线  
  
法国 CNIL　预防性关闭  
  
日本　部分政务平台与站点  
  
1  
  
两个洞，一个根因  
  
CVE-2026-109618 是内存耗尽时的破坏。请求刚好在 memory_limit 边缘失败，PHP 会留下半初始化的内部数据，之后又把它们当成有效数据使用。  
  
在 Apache 加 mod_php 的环境里，未认证请求可以在应用代码执行之前触发它。官方说法是，默认配置下就演示了完整的远程代码执行。  
  
CVE-2026-109619 是陈旧状态。致命错误发生在错误的时点，内部状态就会指向上一个请求的内存。同一个 worker 处理下一个请求时，用到的就是上一个请求已经释放的那块内存。  
  
报告方在 Apache 加 mod_php 的真实应用上打通了 RCE。PHP-FPM 也能中，走的是应用层代码路径。  
  
不是哪个插件写错了。是解释器自己的地基。  
  
这两个位置，几乎每个请求都要经过。所以严重，也所以难修。  
  
2  
  
为什么拖了两年  
  
线索在 2024 年就有了。PHP 那一年做了多年来第一次外部安全审计，资金来自德国主权技术署，审计由 Quarkslab 执行。修另一个问题时，开发者发现了内存耗尽问题的迹象。  
  
当时只是理论上的可能，没找到具体路径，便放下了。  
  
今年 8 月 11 日，Calif 的研究者报告了同根因的无条件 RCE。OpenAI 在同一天独立报告了同样的根因，之后又补了新的攻击向量，Wiz 也补了一份。  
  
修法有好几套，每一套都有代价。破坏向后兼容，引入新的拒绝服务风险，或者要大量扩展跟着改。  
  
安全团队靠共识决策，一直选不出方案。基金会自己在复盘里写了，这次延迟是结构性失败，既有流程问题，也有人手问题。  
  
3  
  
泄露把节奏打乱了  
  
原定 10 月 22 日 14:00 UTC 发布。10 月 20 日下午，编号先漏了。  
  
10-20 15:00  
两个 CVE 编号在论坛和社交平台流传，附带错误的发布时间和错误的 10.0 评分。没有技术细节。  
  
10-20 19:30  
PHP 提前发预公告，发布提前一天。  
  
10-20 20:00  
报告方之一的 PoC 视频被误发，删除后仍在流传，细节又漏了一层。  
  
10-21 07:41  
Chainguard 提前发布含修复的镜像，比协调时间早六个多小时。  
  
10-21 14:00  
四个版本正式发布。官方说，当时没有发现可用的 exploit。  
  
从编号泄露到补丁落地，不到 24 小时。对防守方是好事，对攻击方也是。  
  
4  
  
今晚能做的三件事  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZNdOU6xPdJeDetDfxhLqZY4BZjGg95tYbQsb84zPUPBh5aHJ42lUcniaBpasptKp0PnD9KgPicnnXPYBGSzYdloia5EHJ3PYS8rxh3ibSTibchaM/640?wx_fmt=png&from=appmsg "")  
  
一  
能升就升。8.5 到 8.5.12，8.4 到 8.4.27，8.3 到 8.3.36，8.2 到 8.2.35。升完记得重启 FPM 和 Apache，让 opcache 失效。  
  
二  
升不了就下线。NCSC 自己就是这么做的。WAF 只能挡已知特征，挡不住内存破坏的变形。  
  
三  
5.x、7.x，还有已经停维护的 8.0 和 8.1，没有官方补丁。只剩迁移，或者下线。  
  
别指望临时缓解。官方说得很直白，完整绕过办法没有，升级是唯一修复。  
  
5  
  
十分钟和两年  
  
这次暴露的不是 PHP 有多少洞。是修洞的人太少。  
  
半个 Web 跑在 PHP 上，政府服务、银行、电商、媒体，都在里面。而维护这门语言安全的，是一个靠基金会养着的小团队，加上一点志愿时间。  
  
基金会把话说得很重。这次延迟是结构性的，流程和资源都出了问题。  
  
你今晚升级的十分钟，背后是别人连轴转的两年。  
  
灯关得再快，也快不过补丁。  
  
  
**MORE**  
  
往期回顾  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJd2VWNzn1zwES9wep9zXMxpd4pzZEX5A2auqIDHXxKQu4mTJjniatK47ib7967dK4mN09zS0mUGlx3yVJdDA9Oty2xsADqJnUGT4/640?wx_fmt=gif&from=appmsg "")  
  
[CTF新手速成](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4616106042689814529#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[靶场实战：从漏洞基础到红队综合](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4694391088487563265#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/ZNdOU6xPdJdeDc9RodEDdrwONAFutCzMLCRuQ9OArXOzQhRTMl9ALSeICh3S5pR88veNF5iaQ5MJKhSPibx156OibdfRWEls16qEiajUDZE3juE/640?wx_fmt=gif&from=appmsg "")  
  
[内网渗透学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4572157479522107394#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJfj1YSdkgqeicpsicCDlBHcAC5Q36YsAtcicZdr9WXT2gh8WDicPUbcHBoibYr5ph4LIFMJ1hibHZ8IfBSTjDkE6PfEmg4eXLnggFeQU/640?wx_fmt=gif&from=appmsg "")  
  
[云安全攻防实战连载](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4544879677747986438#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[Web安全学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4534863005871931394#wechat_redirect)  
  
  
