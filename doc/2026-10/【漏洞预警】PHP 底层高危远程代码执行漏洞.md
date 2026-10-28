#  【漏洞预警】PHP 底层高危远程代码执行漏洞  
原创 安博通
                    安博通  安博通   2026-10-28 08:18  
  
近期，PHP 被披露存在php2shell 高危漏洞（CVE-2026-109618、CVE-2026-109619），该漏洞属于无需认证即可远程代码执行（RCE）的严重缺陷。  
  
  
安博通已完成漏洞分析，提醒各单位及时做好防护。  
  
  
**漏洞风险说明**  
  
  
**1、攻击门槛低：**  
无需账号密码，构造特殊 multipart POST 请求即可触发内存破坏。  
  
  
**2、危害严重：**  
利用成功后，攻击者可实现远程代码执行、窃取数据、植入后门，甚至完全接管服务器。  
  
  
**3、影响面广：**  
波及 WordPress、Drupal、Laravel 等大量 PHP 系统，政企及互联网业务普遍面临威胁。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/xd7XLzz7r177O4NO7tuTlKL9KibxG4jMP9WqFL95Ydiaic0Xqk7p9TkSjpU2q1oSI8ey9sRb4l930pTSUU21HQGtYJ6sRZKswpbxQibY4UeLHEo/640?wx_fmt=png&from=appmsg "")  
  
  
特别提示：**该攻击流量会穿越网络边界，若边界设备未实施拦截，攻击流量可直达后端业务服务器。**  
  
  
**IPS 防护规则与使用建议**  
  
  
安博通已发布专项 IPS 入侵防御规则，可识别并拦截php2shell 漏洞攻击。请登录网关管理界面，及时更新威胁特征库，在安全策略中启用 IPS，并将相关规则设置为“阻断+告警”；同时建议将业务侧 PHP 升级至官方修复版本（8.2.35、8.3.36、8.4.27、8.5.12 及以上）。  
  
  
通过安博通检测设备添加IPS规则配置：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/xd7XLzz7r14zRibClziaDrNnQHtRy6gre6hZbKelibQ4icTBIdaI6XJdKxicgYiaxFBgOics4L2oAicIADs9dMiaCRbHwMx8ibNI5A8n5kUD9tUQvlEjY/640?wx_fmt=png&from=appmsg "")  
  
  
发现外部攻击行为：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xd7XLzz7r14BX2wLNkfZlIOLOTkDW7vRfV9KRTDZ0Hku23UWWEsRMe5h5cJx0k9fJicUal2sUch07sTN7oRZODWdjlMaw1ZoCPOdq0uCk5Dw/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**安博通网关设备风险提醒**  
  
  
经评估，部分网关产品内置的 PHP 组件受该风险影响。  
请使用安博通 SG-8000系列深度安全网关、工控防火墙产品的用户，第一时间联系安博通技术支持，核查产品版本并完成固件升级，以消除设备自身安全隐患。  
  
  
**综合处置建议**  
  
  
1、网关侧：核查版本并升级固件，同步更新 IPS 特征库。  
  
2、  
服务器侧：  
将  
 PHP   
升级至官方修复版本，并收紧  
 Web   
业务公网暴露面。  
  
3、运维侧：  
持续关注  
 IPS   
告警，定期审计日志，做好应急处置准备。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Ljib4So7yuWia4IQpuyWIEIZicClqUuz9np8uozS8ewYBCNeDv2icvrFmTgI3lXHyoNuHLhnO3SibjmMibfibLwMlSiaRg/640?wx_fmt=gif "")  
  
  
安博通将持续跟踪该漏洞的动态，及时评估影响范围，迭代优化 IPS 检测规则与防护策略，并通过官方公告和技术支持渠道同步最新进展。如用户在核查、升级或应急处置过程中遇到问题，可随时联系安博通技术支持获取协助。我们将一如既往守护用户网络安全。  
  
  
  
    
**安博通**  
  
**，AI时代安全算力生态构建者**  
  
  
  
  
  
  
近期热门文章  
  
[1024程序员节丨代码有解，热爱无限](https://mp.weixin.qq.com/s?__biz=MzIyNTA5Mzc2OA==&mid=2651139781&idx=1&sn=fc4f61ad0c83a57ac46893c0e6a354ec&scene=21#wechat_redirect)  
  
  
  
[跨越云与网的银河，一分钟get云网安全策略管理](https://mp.weixin.qq.com/s?__biz=MzIyNTA5Mzc2OA==&mid=2651139735&idx=1&sn=7e46d8a21f3141f6a6f57dda4334749e&scene=21#wechat_redirect)  
  
  
  
[省掉70%重复工作量，晶石+鲁班智能体，策略运维再升级！](https://mp.weixin.qq.com/s?__biz=MzIyNTA5Mzc2OA==&mid=2651139734&idx=1&sn=e94df89c17548c2edeb09d3be6d9ded2&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/cZkI8M0nbCnqFdBSZTPngyO5KHUZhk4AED9YficV2s0zic3EHmqH0icpiczkrKGdQKiaUciaiaDIP4R4gG6wdV3HoSRkw/640?wx_fmt=gif#imgIndex=6 "")  
  
点击**阅读原文**  
，了解安博通  
  
  
  
