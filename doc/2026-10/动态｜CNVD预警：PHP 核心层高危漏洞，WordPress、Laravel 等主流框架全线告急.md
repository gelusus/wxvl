#  动态｜CNVD预警：PHP 核心层高危漏洞，WordPress、Laravel 等主流框架全线告急  
 信息安全研究   2026-10-23 07:00  
  
**点击蓝字 关注我们**  
  
  
  
  
    
  
  
**▼▼▼**  
  
  
2  
026年10月21日，据**国家信息安全漏洞共享平台（CNVD）**  
安全公告，PHP 官方披露并修复了两枚高危远程代码执行漏洞（对应CVE-2026-109618、CVE-2026-109619）。本次漏洞深埋 PHP 解释器核心层，影响面覆盖WordPress、Drupal、Joomla、Laravel等主流CMS 与框架，几乎波及全球绝大多数Web应用运行环境。尤其值得警惕的是，漏洞技术细节与PoC已公开传播，攻击门槛被大幅拉低，针对 RCE 利用链的研究仍在推进，留给运营单位的修复窗口有限。  
  
漏洞无小事，防线须前移。各信息系统运营单位应尽快对照**CNVD公告**  
列出的影响范围完成版本自查，通过正规渠道升级至PHP 8.5.12、8.4.27、8.3.36、8.2.35及更新版本；暂不具备升级条件的，务必落实FPM内存限制、阻断跨请求状态传递、WAF拦截异常请求等缓解措施，并遵循 "非必要不上网" 原则收缩公网暴露面。隐患早一日排除，风险少一分蔓延。望各单位以最快速度完成整改，守住网络安全防线。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/buQvkZfsTH9TYRwDu0AzRoBfVcPPvXfw5WiaYYsGiatmEYm0Gmzic2G67JMAAEmXOYEbNUkDQFIKjBnic4l8u9TLsAJo0LjNOCU0MPLNap6xicz4/640?wx_fmt=jpeg "")  
# ●●●●●●●●●●●●●●●●●●●●●●  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTHibRkE5IThcTPT9D6VDbPpOXK8icJu3Xv73Wl4nWToKnWdrrMry5OaepkBJbcL3MIyFvH2iaEsPjVLJB0qbzYM957oYCBMyATf2ib0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH9H2ufsJBWXRtkr9jialahHfiayRCbIz7ibHMUCBDlt2Y2u6NdhQEefN0K4vFnS90lasB59H8k65icUcIHreSdeotF6gje5T5pZicIk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9n48ToOJPdpAwX9rGG2oTUE7JdVr8YxB23l3XECgiboTLCYEsdicAgjgfoMlK2wIJeXOCXpP2r4aJzVUllcq2eY8k3LYlOCiaeEM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/buQvkZfsTH94aJCCN76Goia8EeBXxH5ZdAmoia4GUxyviavA7Kc1o62hiaPoNhugHoE32kkm6H8xMW1t0jOKAvHzb4ucuRUeh0d25HcoRKjYyeo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/buQvkZfsTH9djEo6zicwiavARLkmCXcn2iaP9k09hJ2FXqGKK38EoqAoBxiaiaUISeUNZu1Z66BIIHZvCc1tEaMAUcuZgzFvREY7jClLoDMtc22c/640?wx_fmt=png&from=appmsg "")  
  
（来源：CNVD，国家信息安全漏洞共享平台）  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WicaKuMLeYl3l23eSpOn1tBIbdC0Z6pRUzaRLw6Xdg6H5JggcVxceibhpPE7GLBwib6bIDMiaKsouqKQaJQWc2cF5Q/640?wx_fmt=jpeg&from=appmsg#imgIndex=15 "")  
  
  
