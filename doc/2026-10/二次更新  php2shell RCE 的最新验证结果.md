#  二次更新 | php2shell RCE 的最新验证结果  
原创 微步情报局
                    微步情报局  微步在线研究响应中心   2026-10-22 07:46  
  
最新结论  
  
  
**目前微步情报局已复现mod_php部署下的稳定崩溃和完整RCE。**  
  
**但PHP-FPM部署时的RCE限制较多，该场景下的完整RCE利用链正在进一步分析中。**  
  
  
关于PHP不同部署方式的验证效果  
  
  
PHP   
主流的部署方式有：  
  
1.   
最主流的部署方式是  
nginx  
/  
Apache + PHP-FPM。  
目前发行版、官方php:fpm 仓库和各类面板（宝塔、  
1Panel、Plesk  
等）基本都是这种方式。  
  
2.   
Apache mod_php：主要是部分老 LAMP、部分共享主机，以及部分应用（例如wordpress Docker）还在使用这种方式。  
  
3.   
其他方式：  
L    SAPI  
（商业虚拟机较多）、  
FrankenPHP  
等。  
  
PHP官方已经验证的部署方式和漏洞效果  
：  
<table><tbody><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">部署方式</span></span></span></p></td><td data-colwidth="163.93333333333334" width="163.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">CVE-2026-109618</span></span></span></p></td><td data-colwidth="190.06666666666666" width="190.06666666666666" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">CVE-2026-109619</span></span></span></p></td></tr><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">mod_php</span></span></span></p></td><td data-colwidth="163.93333333333334" width="163.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">已验证完整RCE</span></span></span></p></td><td data-colwidth="190.06666666666666" width="190.06666666666666" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">已验证完整RCE</span></span></span></p></td></tr><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">PHP-FPM</span></span></span></p></td><td data-colwidth="163.93333333333334" width="163.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">源码分析认为受影响，但RCE未证实</span></span></span></p></td><td data-colwidth="190.06666666666666" width="190.06666666666666" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">已验证完整RCE</span></span></span></p></td></tr></tbody></table>  
目前微步情报局已经验证的部署方式和漏洞效果  
：  
<table><tbody><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">部署方式</span></span></span></p></td><td data-colwidth="163.93333333333334" width="163.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#FF0000;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">CVE-2026-109618</span></span></span></p></td><td data-colwidth="190.06666666666666" width="190.06666666666666" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#FF0000;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">CVE-2026-109619</span></span></span></p></td></tr><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">mod_php</span></span></span></p></td><td data-colwidth="163.93333333333334" width="163.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#FF0000;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">已验证CRASH POC</span></span></span></p></td><td data-colwidth="190.06666666666666" width="190.06666666666666" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#FF0000;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">已验证完整RCE</span></span></span></p></td></tr><tr style="height:58.266666666666666px;"><td data-colwidth="79.93333333333334" width="79.93333333333334" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#333333;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">PHP-FPM</span></span></span></p></td><td colspan="2" data-colwidth="354" width="354" style="box-sizing: border-box;vertical-align: middle;padding: 0px 7.2px;border-width: 1px;border-style: solid;border-color: rgb(203, 205, 209);"><p style="text-align:left;line-height:1.3;margin-top:3pt;margin-bottom:3pt;margin-left:0pt;margin-right:0pt;"><span style="font-size:11pt;font-weight:normal;font-style:normal;color:#FF0000;letter-spacing:0pt;mso-font-width:100%;vertical-align:baseline;" data-font-family="default"><span leaf=""><span textstyle="" style="color: rgb(255, 0, 0);">根据现有信息，完整RCE限制条件较多，完整利用链在进一步分析中</span></span></span></p></td></tr></tbody></table>  
  
漏洞原理  
  
  
PHP 处理一次 HTTP 请求时，可能在应用代码尚未执行（解析请求体）阶段就 OOM，并经 zend_bailout() longjmp 中断。被跳过的是中断点之后尚未完成的初始化或扩展 RSHUTDOWN，request shutdown 在 mod_php 上仍会执行。由此出现两类独立缺陷：  
- CVE-2026-109618（同一请求内）：OOM longjmp 后请求内存可能尚未初始化完成；request shutdown 将这份无效状态当作有效对象访问。这是PHP引擎在 “一次请求从启动到 shutdown” 过程中的缺陷，不依赖具体应用代码。  
  
  
- CVE-2026-109619（跨请求）：这是引擎处理跨请求状态的缺陷。一次请求期间，worker 进程中的全局状态（session、错误回溯等）会保存指向该请求堆的指针；请求结束本应先重置这些全局状态，再释放请求堆。若此时发生 fatal 或 OOM longjmp，扩展的请求清理（RSHUTDOWN）可能被跳过，重置不会发生，而请求堆仍会被释放，指针悬空。shutdown 开始后的非 fatal 诊断也可能再次写入错误相关的全局状态。同一 worker 的下一请求在正常初始化之前访问这些过期指针，即构成 UAF。  
  
  
  
漏洞处置优先级(VPT)  
  
  
**综合处置优先级：**  
高风险  
<table><tbody><tr><td rowspan="3" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">基本信息</span></section></td><td data-colwidth="192" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;"><section><span leaf="">微步编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><p><span leaf="">XVE-2026-84070</span></p><p><span leaf="">XVE-2026-84069 </span></p></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><p><span leaf="">CVE-2026-109618</span></p><p><span leaf="">CVE-2026-109619</span></p></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程代码执行</span></section></td></tr><tr><td rowspan="5" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用条件评估</span></section></td><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的网络条件</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程</span></section></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要绕过安全机制</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">对被攻击系统的要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无</span></section></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的权限要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无需权限</span></section></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要受害者配合</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td rowspan="2" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用情报</span></section></td><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否有POC</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是</span></section></td></tr><tr><td data-colwidth="192" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">已知利用行为</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">暂未捕获</span></section></td></tr></tbody></table>  
漏洞影响范围  
  
  
**PHP 8.5.x： < 8.5.12 受影响**  
  
**PHP 8.4.x： <8.4.27 受影响**  
  
**PHP 8.3.x： <8.3.36 受影响**  
  
**PHP 8.2.x： <8.2.35 受影响**  
  
**PHP 8.1 及更早版本也受影响。PHP 8.1 及更早的版本 PHP 项目不再发版本。官方给了仅供自行编译的最小 backport 分支：**  
- PHP 7.4：https://github.com/php/php-src/tree/PHP-7.4-CVE-2026-109618-109619  
  
- PHP 8.0：https://github.com/php/php-src/tree/PHP-8.0-CVE-2026-109618-109619)  
  
- PHP 8.1 ：https://github.com/php/php-src/tree/PHP-8.1-CVE-2026-109618-109619  
  
  
  
漏洞复现  
  
  
CVE-2026-109619：在 PHP 8.4.26 的 Apache mod_php上  
（prefork 单工作进程，memory_limit=8M、post_max_size=64M）  
，对任意 PHP 页面发送特殊的  
 multipart POST 请求，实现远程任意代码执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEPFicNm5fyaXhSp4ImIf70PHibF69NBd4DbagPdrwn4K1iaUQUyiaNe2sAUgZwcxRzFBicQ6dT2RtiakLA6ANxoY4pzJCffE0RS8CAiaw/640?wx_fmt=png&from=appmsg "")  
  
  
CVE-2026-109618：  
在PHP 8.5.11 的 Apache mod_php（prefork，单工作进程）上，对任意 PHP 页面  
发送特殊的  
 multipart POST 请求，造成内存损坏，工作进程 SIGSEGV 并产生 core dump。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEPOejyY2O6PMhk20u9RXcdoXBt6L8J8THLqeLRZuUiaeUAdiaSvrMU5ibY2P6ibscvFcewcKMBvmVBJSicfUnyXbGDQUfMh6g1yLkmE/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
修复方案  
  
### 官方修复方案  
  
  
### PHP 安全团队已发布安全版本修复上述漏洞，具体通告链接：  
  
- https://www.php.net/archive/2026.php#2026-10-21-5  
  
- https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
- https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
### 临时缓解措施  
  
  
### 以下为PHP官方提供的临时缓解措施：对于PHP-FPM，禁用会话上传进度session.upload_progress.enabled=0并进行设置memory_limit=-1。对于mod_php，设置memory_limit=-1。但PHP官方建议这些更改无法完全取代补丁，并且设置memory_limit=-1会移除 PHP 的内存限制，增加内存耗尽的风险，建议最好通过升级的方式来缓解漏洞。  
  
微步产品支撑  
  
  
微步漏洞情报于  
2026年10月21日  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
微步威胁感知平台TDP已于  
2026年10月22日  
支持检测，检测ID：   
S3100185607、S3100185611、S3100185616  
，模型/规则高于：  
20261022100000  
可检出。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdENlHopibaoozrXtoS0BAVkRZicZ2rabenpor079Q003oia0QemXqfGFZHwHHTLxaeGibAh7g2cwqZicL4t9slIFSZxnYIoaEZNJGOIc/640?wx_fmt=png&from=appmsg "")  
  
微步威胁防御系统OneSIG已支持防护规则ID为：  
3100185607、3100185611、3100185616。  
  
  
  
