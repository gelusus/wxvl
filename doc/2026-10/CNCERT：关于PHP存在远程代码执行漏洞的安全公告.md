#  CNCERT：关于PHP存在远程代码执行漏洞的安全公告  
 安全内参   2026-10-22 10:23  
  
安全公告编号:  
CNTA-2026-0007  
  
  
2026  
年  
10  
月  
21  
日，国家信息安全漏洞共享平台（  
CNVD  
）收录了  
PHP  
远程代码执行漏洞（  
CNVD-2026-41674  
、  
CNVD-2026-41675  
，对应  
CVE-2026-109618  
、  
CVE-2026-109619  
）。未经身份验证的远程攻击者可构造恶意  
HTTP  
请求触发内存错误，在特定条件下实现对目标服务器的拒绝服务攻击、敏感数据窃取、远程代码执行。目前，漏洞技术细节与  
PoC  
已广泛传播，  
CNVD  
建议受影响用户立即升级至最新版本。  
  
一、漏洞情况分析  
  
PHP  
是一种广泛使用的开源通用脚本语言，适用于  
Web  
开发并可嵌入  
HTML  
中。  
PHP  
解释器内核为全球多数  
Web  
应用运行提供了基础运行时环境，包括  
WordPress  
、  
Drupal  
、  
Joomla  
、  
Laravel  
等主流框架和内容管理系统。  
  
2026  
年  
10  
月  
21  
日，国家信息安全漏洞共享平台收录了  
PHP  
远程代码执行漏洞。  
PHP  
解释器内核在内存管理环节存在逻辑处理缺陷，攻击者可构造特定  
HTTP  
请求触发程序异常处理流程，导致内部数据状态异常或已释放内存被再次访问，引发内存破坏。  
未经身份验证的远程攻击者，可构造恶意  
HTTP  
请求触发内存错误，并在特定条件下实现对目标服务器的拒绝服务攻击、敏感数据窃取、远程代码执行  
。  
  
CNVD  
对该漏洞的综合评级为  
"  
高危  
"  
。  
  
二、漏洞影响范围  
  
目前，已知受漏洞影响的  
PHP  
产品版本包括：  
  
1  
、  
8.2.35  
版本之前的  
8.2.x  
版本；  
  
2  
、  
8.3.36  
版本之前的  
8.3.x  
版本；  
  
3  
、  
8.4.27  
版本之前的  
8.4.x  
版本；  
  
4  
、  
8.5.12  
版本之前的  
8.5.x  
版本。  
  
厂商发布的修复版本为  
PHP8.5.12  
、  
8.4.27  
、  
8.3.36  
、  
8.2.35  
。  
  
此外，该漏洞还影响  
Magento  
、  
WordPress  
、  
Drupal  
、  
Joomla  
、  
MediaWiki  
、  
Roundcube  
、  
phpMyAdmin  
、  
Nextcloud  
、  
Laravel  
、  
Symfony  
、  
WHMCS  
、  
OpenCart  
等基于  
PHP  
环境的主流  
CMS  
和框架。  
  
三、漏洞处置建议  
  
目前，PHP安全团队已修复漏洞并发布安全公告，  
建议  
PHP  
用户及时开展自查，关注官方信息，通过官方渠道及时更新升级至最新版本（https://www.php.net/archive/2026.php[#2026]()  
  
-10-21-5  
）。  
  
  
**推荐阅读**  
- [网安智库平台长期招聘兼职研究员](http://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247499450&idx=2&sn=2da3ca2e0b4d4f9f56ea7f7579afc378&chksm=ebfab99adc8d308c3ba6e7a74bd41beadf39f1b0e38a39f7235db4c305c06caa49ff63a0cc1d&scene=21#wechat_redirect)  
  
  
- [欢迎加入“安全内参热点讨论群”](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247501251&idx=1&sn=8b6ebecbe80c1c72317948494f87b489&chksm=ebfa82e3dc8d0bf595d039e75b446e14ab96bf63cf8ffc5d553b58248dde3424fb18e6947440&token=525430415&lang=zh_CN&scene=21#wechat_redirect)  
  
  
  
  
  
  
文章来源：  
CNVD漏洞平台  
  
  
点击下方卡片关注我们，  
  
带你一起读懂网络安全 ↓  
  
  
  
  
