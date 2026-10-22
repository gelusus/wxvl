#  【漏洞风险通告】PHP远程代码执行漏洞  
 迪普科技   2026-10-22 07:56  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/wDpVcZDWF6Ywu3a4RB4GjpKYBJSa4g6TR0QNMQgiafFgJHpibxHzH4Bwo2icib4Lyzxs6U1dKNMvhWlX9WKiaoQge2iclJrx46T23sZKjRPXsiaycY/640?wx_fmt=png&from=appmsg "")  
  
PHP是全球广泛使用的服务器端编程语言。该漏洞是PHP核心（PHP Core）中存在的严重漏洞，CVSS评分高达9.2（严重）。攻击者无需身份认证，通过构造恶意HTTP请求即可触发内存破坏（释放后重用UAF）或内存耗尽缺陷，在服务器上以PHP进程权限执行任意代码，可能导致服务器被完全控制、敏感数据泄露或业务中断。  
  
目前官方已经发布了修复版本。迪普科技建议相关用户及时进行修补更新，做好相关防护措施。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/wDpVcZDWF6Zmkl2OiaW0sychZ3YfoibIUPhA6MpXg8cPjoBAURDhFSnnGLibJRIgA6iaUib5gDe7aekMPDQUXX9xQW7mia7VdQ1jiboib2TPallVgo4/640?wx_fmt=png&from=appmsg "")  
  
**严重等级**  
  
高危  
  
**漏洞描述**  
  
CVE-2026-109618：该漏洞是PHP内存耗尽缺陷漏洞，利用PHP在内存耗尽（OOM）时longjmp()提前退出的机制，导致未完全初始化的请求内存在关闭阶段被错误消费，触发内存破坏。  
  
CVE-2026-109619：该漏洞是PHP跨请求陈旧状态缺陷，利用PHP工作进程在请求结束后全局状态未正确重置，导致后续请求在初始化前访问已释放的内存，形成释放后重用（UAF）。  
  
**影响版本**  
  
PHP 8.2.x < 8.2.35  
  
PHP 8.3.x < 8.3.36  
  
PHP 8.4.x < 8.4.27  
  
PHP 8.5.x < 8.5.12  
  
**漏洞原理**  
  
CVE-2026-109618：PHP在处理内存耗尽（Out of Memory，OOM）错误时，使用longjmp()进行非本地跳转以提前退出当前执行上下文。在某些场景下，请求内存尚未完全初始化就已经处于可访问状态；一旦此时出现内存分配失败，PHP未能正确处理该提前退出逻辑，将会造成请求内存状态无效。  
  
CVE-2026-109619：PHP工作进程会复用以提升性能，但每次请求拥有独立的内存空间。某些内部全局状态保留在工作进程中，却指向单次请求持有的内存（如错误、会话、对象或扩展相关状态）。这些引用必须在请求内存释放前正确重置，否则形成释放后使用（Use-After-Free，UAF）。  
  
**漏洞检测**  
  
**版本检测：**  
  
相关用户可通过查看PHP的版本来判断应用是否在受影响范围内：  
  
**PHP -v**  
:  
  
![](https://mmbiz.qpic.cn/mmbiz_png/wDpVcZDWF6Zvp9ZKFUa4FiaNGwMWdUNO9hVb5Urt6an4ueibLOYjt50BHG8ia7bn90P2iaTNsrGdIQYDJqb4wu093ufy5DtkicgO6yBy3ZqjibSB0/640?wx_fmt=png&from=appmsg "")  
  
**漏洞验证**  
  
在PHP8.2.34上成功复现PHP2Shell：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/wDpVcZDWF6Zg9aqQ4IGQ5joADhLIYww5wn2EEedNG60prgOwV69GGjOlDARK7eI8fiaonGhNY6FBW5d46fZ9BasMs2LF96nqR4Rw7AcDfTwc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wDpVcZDWF6ZHV29xjV3lNyRz5dVJwdicG4wjNyRlPIJjQHa7JEXMrcZR7KXUbPa5q4GBvq0s5BGsMJdic8qvoeYXljSnIxMMEzicuHgnzgYpTk/640?wx_fmt=gif&from=appmsg "")  
  
  
**解决方案**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wDpVcZDWF6bot9jZnpVNoHxyaHYSfzYwsksPbqfC4PYTfse8IjlubSdzbKKwmgQBribrvhPufPzficTR9KVeP2kHfoqrD8EibOW3ibWCgbkMYX0/640?wx_fmt=gif&from=appmsg "")  
  
  
  
  
**官方解决方案**  
  
  
当前官方已发布修复版本，及时升级。升级后需重启Web服务与PHP工作进程，确保旧进程不再处理请求。  
  
更新地址：https://www.php.net/downloads.php   
  
  
  
**迪普科技解决方案**  
  
  
迪普科技安全研究院监测到PHP远程代码执行漏洞后，迅速采取了应急措施。  
  
**1**  
  
使用迪普科技“**慧眼检测平台**  
”检测现网环境中是否存在PHP远程代码执行漏洞。  
  
**2**  
  
使用迪普科技“**态势感知平台**  
”检测现网环境中是否存在PHP远程代码执行漏洞攻击行为。  
  
**◆ 态势感知特征库版本**  
：2026100501  
  
**3**  
  
迪普科技**DPtech IPS2000、FW1000、WAF3000**  
安全产品将  
在以下特征库版本中对PHP远程代码执行漏洞进行有效防护：  
  
◆ **产品系列**  
：IPS2000、FW1000  
  
◆ **漏洞库版本**  
：IPS-R3.1.494、IPS-R3.3.121、IPS-R4.1.208  
  
◆ **产品系列**  
：WAF3000  
  
◆ **漏洞库版本**  
：WAF-R3.1.384、WAF-R2.1.384  
  
**4**  
  
迪普科技安全服务团队可协助客户完成**现网安全风险评估**  
，针对网络安全入侵事件，提供快速应急响应支撑服务以及专业的安全建设建议。  
  
**5**  
  
**迪普科技官网特征库下载地址：**  
  
http://www.dptech.com/index.php?m=content&c=index&a=lists&catid=57  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wDpVcZDWF6YywO8XItK2BfFntUG9iaelK3fsAYCWfdcMuGbB7mTGNPLpico1I2jPqpe5J9yqIPwOic19dZXyFn44icvIgtqSAgcSu8gDeZiaFAwI/640?wx_fmt=gif&from=appmsg "")  
  
**迪普科技正在全力跟踪相关漏洞最新进展，请启动设备自动更新特征库功能，如有疑问，可联系迪普科技当地办事处售后人员或拨打客户服务热线：400-6100-598，进一步了解相关情况。**  
  
  
**往期推荐**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wDpVcZDWF6ZoJ6lcnSLHlTOhJrSia6ShGZJ3HaW36h76AnkmJaIqZKdrMicfZESEjznA5rCssib0LHprmrftBNBF9foEKAxNQ8sBSAC3aQqBJ4/640?wx_fmt=gif&from=appmsg "")  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzA4NzE5MzkzNA==&mid=2650398595&idx=1&sn=aaaee4a46d61439befb3ace31e7945b2&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA4NzE5MzkzNA==&mid=2650391491&idx=1&sn=ba3755359ae06115b601657668c0961d&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA4NzE5MzkzNA==&mid=2650378353&idx=1&sn=0442c3ecd4a7672c04fe8809e0b5a3bc&scene=21#wechat_redirect)  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wDpVcZDWF6ZpT7hLaxnn4g8yoYRBXRvPvkibjp2ZGF7v2bLib9mcicIoQRZ5nFg97nuD8ArMv5kEDehHYibiafUlf5ENeUY573TUmZ9LRRJw6Lb4/640?wx_fmt=gif&from=appmsg "")  
  
  
END  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wDpVcZDWF6Y1jlb6n1FR1pMUJ9es1By0Qar9A9f6ju8hLaFvG425ZuCqYkiaqrUU2wHhERM0gzrOq3Ie5LSQS1qAzAHoYGicMx54uoRU8PnqQ/640?wx_fmt=gif&from=appmsg "")  
  
  
