#  速查！PHP 核心爆出超危漏洞，长亭已复现验证，全系产品支持检测  
夺旗猫
                    夺旗猫  长亭安全观察   2026-10-22 11:12  
  
我们每天访问的网站，背后大多运行着同一门语言——  
PHP  
。  
  
据 W3Techs 2026 年 10 月 22 日的统计，在已知服务端编程语言的网站中，  
69.3% 使用 PHP  
；全球 39.6% 的网站直接运行在 WordPress 上，而 WordPress 本身由 PHP 编写。内容管理系统、电商平台、论坛、网盘、主机控制面板和企业自建业务系统，大量以 PHP 作为运行环境。PHP决定了页面如何生成、如何处理登录、上传和会话。  
  
这类底层组件一旦出问题，影响面会覆盖大部分互联网。2026 年 10 月，PHP Core 被披露存在两处远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619），公开讨论中以 php2shell 指代该组漏洞，  
官方评级为 CVSS v4.0 9.2（Critical），危害性极高  
。漏洞位于 PHP 引擎层，不依赖上层应用代码，因此上层应用写得是否规范，并不改变风险的存在。  
  
漏洞的成因属于内存安全范畴。简单说，程序在释放一块内存后，仍保留了指向它的地址；后续请求再用这个地址时，读到的可能是其他请求留下的数据，也可能是攻击者构造的内容。由于 Apache mod_php 与 PHP-FPM 会复用 worker 进程，上一个请求残留的失效状态有机会被下一个请求继承，缺陷因此可以在请求之间传递。  
  
攻击者利用这一缺陷，不需要账号，不需要目标用户点击任何链接，也不需要目标系统存在特殊配置，即可读取 PHP 进程内存，其中可能包含其他请求或服务端的口令、会话令牌、API Key 与证书；在已验证的利用链中，可进一步在服务器上执行任意代码。  
  
长亭安全应急响应中心  
对这组漏洞做了连续跟踪。  
10 月 21 日发布首份预警  
[【已复现UAF】php2shell：PHP Core 远程代码执行漏洞（CVE-2026-109618/109619）](https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247493443&idx=1&sn=36f659baea269658659eea7f1f4b482a&scene=21#wechat_redirect)  
  
，并在实验环境稳定复现内存破坏导致的进程崩溃；  
10 月 22 日完成  
 Apache mod_php 部署场景下 WordPress 的  
远程代码执行验证  
，同时确认官方修复版本已经发布，  
给出升级与临时缓解方案  
。  
  
对使用 PHP 的组织，当前的处理顺序是：先确认版本是否在受影响范围内，能升级的尽快升级；暂时无法升级的，按照下面的措施降低可利用性。  
  
长亭科技全系产品已支持对该漏洞的检测  
，请尽快发现并处理。  
  
**漏洞描述**  
 Description   
**01**  
  
### 漏洞成因  
  
PHP Core 在处理请求期间，若内存分配失败或发生其他致命错误，部分内部状态的清理与回滚可能未完成。请求结束后请求内存被释放，但 worker 进程仍保留指向该内存的全局状态；同一 worker 处理后续请求时，这些失效状态可能在正常初始化之前被读取或释放，形成跨请求的 use-after-free 与类型混淆。缺陷位于 PHP Core 本身，不依赖上层应用代码，Apache mod_php 与 PHP-FPM 复用 worker 进程的机制会放大该问题。公开分析将这条利用链称为 php2shell。  
### 漏洞影响  
  
攻击者可在无需认证的情况下读取 PHP 进程内存，其中可能包含其他请求或服务端的口令、会话令牌、API Key 与证书；在已验证的利用链中，攻击者可进一步在受影响服务器上执行任意代码，导致服务器被完全控制、页面被篡改或业务数据被窃取。缺陷位于 PHP 引擎层，不依赖上层应用自身的代码，默认配置下即存在可利用路径。  
### 处置优先级：高  
  
**漏洞类型：**  
 远程代码执行  
  
**漏洞危害等级：**  
 高  
  
**触发方式：**  
 网络远程  
  
**权限认证要求：**  
 无需权限  
  
**系统配置要求：**  
 默认配置  
  
**用户交互要求：**  
 无需用户交互  
  
**利用成熟度：**  
 POC/EXP 已公开  
  
**修复复杂度：**  
 低，官方已提供修复方案  
  
  
  
**影响版本**  
 Affects   
**02**  
  
PHP 5.4.0 - 8.2.34  
  
PHP 8.3.0 - 8.3.35  
  
PHP 8.4.0 - 8.4.26  
  
PHP 8.5.0 - 8.5.11  
  
**解决方案**  
 Solution   
**03**  
  
### 升级修复方案  
  
官方已在各维护分支发布修复版本，建议升级至以下版本或更高：  
- PHP 8.5.12：https://www.php.net/releases/8_5_12.php  
  
- PHP 8.4.27：https://www.php.net/releases/8_4_27.php  
  
- PHP 8.3.36：https://www.php.net/releases/8_3_36.php  
  
- PHP 8.2.35：https://www.php.net/releases/8_2_35.php  
  
升级后重启 PHP-FPM 与 Web 服务，替换残留旧 worker。  
### 临时缓解方案  
  
以下方案为应急临时措施，可能影响业务功能，升级完成后应及时移除。  
- PHP-FPM 部署：关闭会话上传进度功能（session.upload_progress.enabled=0  
），并将 memory_limit  
 设为 -1  
。  
  
- Apache mod_php 部署：将 memory_limit  
 设为 -1  
。  
  
- 在边界防护设备或 WAF 上拦截异常 multipart 请求，并收敛不必要的上传入口。  
  
将 memory_limit  
 设为 -1  
 会移除 PHP 的内存上限，可能增加内存耗尽与拒绝服务风险；上述配置仅能降低部分触发路径的可利用性，不能替代升级。  
  
**漏洞复现**  
 Reproduction   
**04**  
  
长亭安全应急响应中心已复现RCE。  
  
  
官方 php:8.4.26-apache 镜像，完成了未认证的跨请求状态利用 → 196 KB 堆泄露 → ASLR 绕过 → 伪造 HashTable → zif_system 执行命令，以 www-data 身份落地。  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/EqS9GE77r0OsWI5vryfk75eQ3EUIDAn9oRyA7qHHwMvaFdZGZib96stNiakRVqJCibYPgxdnTY82ic1pVNMQqAr1MKcjhWtk79jZclk5aAJQIxA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "")  
  
  
Apache mod_php 部署场景下的 WordPress（  
memory_limit=128M、  
 post_max_size=8M、  
MaxRequestWorkers=5  
）  
  
  
  
  
Apache mod_php 部署场景下的 phpmyadmin（memory_limit=128M、 post_max_size=8M、MaxRequestWorkers=5）  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/EqS9GE77r0O7A1xEzUXLjs5plQe9tjricOugo3QDk355N2icDoA93X9ibWqWkUVVAz4ibGeYib25eAovKPf4gHnHxsBryDGlFxJ2B0gFzMxGREbk/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "")  
  
  
**产品支持**  
 Support   
**05**  
  
**云图： 默认支持该产品的指纹识别，预计2026.10.22支持该漏洞的PoC检测。**  
  
洞鉴：  
   
默认支持该产品的指纹识别，  
预计2026.10.22 发布应急PoC。  
  
无锋：  
   
默认支持该产品的指纹识别，  
预计2026.10.22  
支持该漏洞的PoC检测。  
  
全悉：  
 已发布更新包支持该漏洞利用行为的检测。  
  
雷池：  
 已发布自定义规则支持该漏洞利用行为的检测。  
  
