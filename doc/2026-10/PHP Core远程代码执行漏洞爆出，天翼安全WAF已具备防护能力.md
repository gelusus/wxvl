#  PHP Core远程代码执行漏洞爆出，天翼安全WAF已具备防护能力  
网站安全专家产线
                    网站安全专家产线  中国电信安全   2026-10-22 10:04  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/Dh3fqSPAOWekCSIf3ffuFuiaBPl4BSArBsDhFEMSOTbeIfb7mdz4D0mDExZesv4PPicUdsOTxfRUx8QntAMTmTBA/640?wx_fmt=gif#imgIndex=0 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dh3fqSPAOWePMu18J3qdm7D4DgerZrtmzfZ2ThoQZZl4WAHJScyL4c59NPZcjXgnP2YL4HnUicxRibjtBZmqJW8A/640?wx_fmt=other&from=appmsg#imgIndex=1 "")  
  
2026  
年  
10  
月  
21  
日，  
PHP  
官方发布紧急安全更新，修复了两个藏身于  
PHP  
核心引擎多年的严重安全漏洞，编号分别为**CVE-2026-109618**  
与**CVE-2026-109619**  
，安全社区将其统称为  
 "php2shell"。两个漏洞在  
CVSS v4.0  
评分体系中均为**9.2****分**  
，属于最高危级别。攻击者无需身份验证，只需向运行受影响  
PHP  
版本的网站发送特制的  
multipart POST  
请求，即可触发内核内存破坏并最终实现远程代码执行，进而完全控制服务器。  
  
  
PHP是全球使用最广泛的服务器端脚本语言之一，大量内容管理系统、电商平台、论坛与政企网站  
以其为运行时基础，受影响资产量级达千万级。  
中国电信安  
全专家在漏洞信息披露当日即完  
成分析  
与复现验证，并第一时间更新了防护策略——中国电信天翼安全网站安全专家新一代WAF已全面支持对该漏洞攻击的检测与拦截。  
  
  
一、漏洞速览  
  
  
<table><tbody><tr><td data-colwidth="95.125px" width="95.125px" valign="center" style="border-width: 1px;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);box-sizing: border-box;padding: 5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><strong><span leaf="">项目</span></strong></span></span></p></td><td data-colwidth="366.875px" width="366.875px" valign="center" style="border-width: 1px 1px 1px medium;border-style: solid solid solid none;border-color: rgb(204, 204, 204) rgb(204, 204, 204) rgb(204, 204, 204) currentcolor;background: rgb(213, 232, 240);box-sizing: border-box;padding: 5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><strong><span leaf="">内容</span></strong></span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">漏洞名称</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span leaf="">PHP Core 远程代码执行漏洞（php2shell）</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">漏洞编号</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span leaf="">CVE-2026-109618、CVE-2026-109619</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">公开时间</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span leaf="">2026-10-21</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">危害等级</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">严重（</span></span><span leaf="">CVSS v4.0：9.2）</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">威胁类型</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">远程代码执行（内存破坏</span></span><span leaf=""> / 释放后重用）</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">利用条件</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">可访问目标</span></span><span leaf=""> Web 服务即可，无需身份验证、无需用户交互</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">官方修复</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span leaf="">2026-10-21 已发布，修复版本：8.5.12 / 8.4.27 / 8.3.36 / 8.2.35</span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">复现状态</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">电信安全专家已在实验环境完成复现</span></span></span></p></td></tr><tr><td valign="center" style="border-width: medium 1px 1px;border-style: none solid solid;border-color: currentcolor #cccccc #cccccc;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">防护状态</span></span></span></p></td><td valign="center" style="border-width: medium 1px 1px medium;border-style: none solid solid none;border-color: currentcolor #cccccc #cccccc currentcolor;padding:5px 10px;"><p style="text-align:center;"><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">网站安全专家新一代</span></span></span><span style="font-size:16px;font-family:微软雅黑;"><span leaf="">WAF</span></span><span style="font-size:16px;font-family:微软雅黑;"><span style="font-family:微软雅黑;"><span leaf="">已支持检测与拦截</span></span></span></p></td></tr></tbody></table>  
二、漏洞详情  
  
  
  
**2.****1**  
  
**漏洞成因**  
  
本次披露的两个漏洞均位于PHP核心的**会话上传进度（session upload progress）处理机制**  
中。该机制自PHP 5.4引入且默认开启，用于在上传文件时向页面反馈上传进度，其处理发生在应用程序脚本执行之前，属于PHP内核与Web服务器交互的底层路径。  
  
  
分析表明，该机制在解析multipart请求的过程中会触发多次会话状态切换。两个漏洞的成因分别为：  
  
• **CVE-2026-109618**  
：极端内存压力（OOM）场景下，内部数据结构在构建过程中被中断而处于不完整状态，请求清理阶段将无效数据当作有效数据处理，可导致内存破坏并最终实现远程代码执行；  
  
• **CVE-2026-109619**  
：请求相关的全局状态在内存释放后仍然残留，后续由同一worker进程处理的请求会访问已释放的内存（use-after-free），同样可导致内存破坏与远程代码执行。  
  
  
由于上述处理路径先于应用逻辑执行，即使业务系统本身不存在上传功能页面，或已部署登录校验等应用层防护，也**不能据此认定系统不受影响**  
。这也正是该漏洞影响面如此之广的根本原因。  
  
**2.****2**  
  
**漏洞危害**  
  
一旦攻击者成功利用完整利用链，可在 PHP 服务进程权限下执行任意代码，典型危害包括：  
  
• 服务器被完全控制，部署WebShell并建立持久化访问；  
  
• 网站页面被篡改、业务数据与敏感信息被窃取；  
  
• 业务中断，Web服务大面积崩溃；  
  
·   
以失陷主机为跳板，向内网横向渗透，扩大攻击战果。  
  
三、影响范围  
  
  
  
**•**  
******直接影响**  
：PHP 5至PHP 8.5（官方修复版本为 8.5.12、8.4.27、8.3.36、8.2.35；PHP 8.1 及更早版本已停止安全支持，预计无法获得官方修复，需优先迁移或隔离）；  
  
  
**•**  
******间接影响**  
：以PHP为运行时的应用、框架与建站系统，包括WordPress、MediaWiki、phpBB、Magento、Laravel、Symfony、Drupal 等；  
  
  
**•**  
******部署形态**  
：mod_php、PHP-FPM（含 Docker PHP-FPM 镜像）、CGI 等均可能受影响；  
  
  
**·****使用操作系统发行版、容器镜像或托管服务提供 PHP 的用户**  
，应等待并采用供应商标注已修复相关 CVE 的安全更新。  
  
  
四、电信安全专家已完成复现验证  
  
  
  
漏洞信息披露后，电信安全专家第一时间启动应急分析，  
复现成功截图：  
  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OrnFicDIeOOH1b9oCMYIibacq5515e4ecic1kweUiatVDic717YxQzUichsfZbVezfyOibownopAd9cRCez0S802GDibVL1lkTLbGXXMMo/640?wx_fmt=png&from=appmsg "")  
  
五、外置建议  
  
  
  
**5.****1**  
  
**升级修复（首选）**  
  
PHP 官方已于2026年10月21日22:00（北京时间）发布安全更新，请立即依据官方公告（  
https://www.php.net/archive/2026.php  
）将PHP升级至以下修复版本：8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
  
升级时应优先处置直接暴露于互联网的Web服务，使用发行版、容器镜像或托管服务的，请采用供应商侧已修复版本，并在更新后按要求重启或替换服务进程，确认所有业务实例均已运行修复后的 PHP 版本。  
  
**5.2**  
  
**临时缓解措施**  
  
在完成升级前，可参考采取以下临时措施（可能影响业务功能，升级完成后应及时评估是否保留）：  
  
•  
   
若业务不依赖上传进度条，在  
php.ini  
、  
.user.ini  
 或   
.htaccess  
 中设置   
session.upload_progress.enabled = Off  
，并重新加载PHP-FPM或Web服务器，可切断目前已验证的攻击路径；  
  
•  
   
在WAF、反向代理或网关层**收紧对****multipart/form-data 请求的检查与速率限制**  
，重点保护不需要文件上传的业务端点；  
  
•  
   
启用  
session.use_strict_mode = 1  
，降低攻击者指定任意会话 ID 的风险；  
  
•  
   
对PHP 8.1及更早已停止支持的版本，采取下线、网络隔离或白名单访问控制，直至完成迁移；  
  
•  
加  
强  
Web  
服务日志与主机行为监测，重点关注  
PHP  
进程异常退出、异常子进程创  
建、非预期文件写入及可疑外联行为。  
  
六、网站安全专家新一代WAF已支持防护  
  
  
  
基于对漏洞成因与攻击特征的深入分析，**中国电信天翼安全对网站安全专家新一代 WAF的防护策略完成了紧急升级，已全面支持对 php2shell（CVE-2026-109618 / CVE-2026-109619）攻击的检测与拦截**  
，并在网站安全专家新一代WAF平台复现环境中完成了拦截效果验证。截图如下：  
  
  
![image.png](https://mmbiz.qpic.cn/mmbiz_png/NicWxqLvw6Oo0tv4l8ldBXWQe0uBSD0QKTrn7LazQY8KrnLGTgYTFWSh6OXf04tRaWZibotlZjYlbWonVgQaZf6pExibMqA4qlOFPAy02mHm70/640?wx_fmt=png&from=appmsg "")  
  
  
网站安全专家新一代WAF的防护能力体现在以下方面：  
  
**•**  
******深层请求解析**  
：对multipart请求进行结构化解析，可精确识别会话上传进度字段中的异常构造，覆盖攻击者触达内核缺陷路径的必经环节；  
  
**•**  
******智能语义分析**  
：不依赖单条特征规则，结合请求序列、会话状态切换等上下文特征识别异常访问模式，对变种与混淆攻击同样有效，误报率低；  
  
**•**  
******虚拟补丁防护**  
：在用户完成官方版本升级之前，即可在网络边界提供前置防护，为补丁窗口期争取宝贵时间；  
  
**•**  
******威胁联动响应**  
：拦截事件实时告警，可与安全运营流程联动，帮助用户快速定位受攻击资产。  
  
对于已接入网站安全防护的用户，无需额外操作即可获得上述防护能力；尚未接入的用户，建议尽快开通，尤其应优先保障对外暴露的  
PHP  
业务系统。同时提醒：**WAF****防护不能替代官方补丁，完成版本升级仍是彻底消除风险的根本措施**  
，建议两者结合、分层设防。  
  
供稿：网站安全专家产品线  
  
排版：马子豪  
  
编辑：陈师慧  
  
校对：李雪  
  
执行主编：田金英  
  
主编：冯晓冬  
  
**推荐阅读**  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzkxNDY0MjMxNQ==&mid=2247542192&idx=1&sn=578772d5248dffd11e35f876358a8f06&scene=21#wechat_redirect)  
  
**重磅升级！见微大模型安全护栏**  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzkxNDY0MjMxNQ==&mid=2247541733&idx=1&sn=3d171e39aa8109c75e289fd0f812a1c2&scene=21#wechat_redirect)  
  
**网站安全专家｜云WAF重点功能升级！**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/z7xPqlc0GbBZdrnibkon0HxogO9iazwQy5cqsw6fRdkPujrZZCuVnk7ywgyYA7yTfRIIkEXpdpDfQlkFMENO9P0Q/640?wx_fmt=gif&from=appmsg "")  
  
  
  
