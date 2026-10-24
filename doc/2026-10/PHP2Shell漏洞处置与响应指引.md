#  PHP2Shell漏洞处置与响应指引  
安天CERT
                    安天CERT  安天集团   2026-10-24 02:40  
  
点击上方"蓝字"  
  
关注我们吧！  
  
  
# 声明  
  
本文基于安天CERT工程师的分析研判整理而成，结合AVL Code智能体协同分析并进行人工修订。建议读者交叉验证关键信息。如发现错误或质疑，欢迎反馈，我们验证后会对本文持续修订（点击文末“阅读原文”可查看最新版本）。  
# 概述  
  
安天综合研判，鉴于该漏洞的影响及危害，安天发布针对该漏洞的漏洞处置及响应指引。本指引面向运维、安全运营、应急响应人员，用于指导完成PHP2Shell漏洞的资产摸排、风险判定、漏洞验证、安全加固、效果核验以及持续监测，帮助快速完成应急处置，降低漏洞被利用、植入WebShell、服务器失陷、内网横向渗透等安全风险。  
  
说明：  
PHP2Shell漏洞为PHP代码执行类高危漏洞，攻击者可通过构造特殊请求，利用漏洞实现代码执行，直接写入WebShell，获取服务器控制权，存在数据泄露、业务篡改、挖矿、勒索等高风险。部分嵌入式设备（路由器、NAS、工控Web管理界面）内置精简PHP环境，同样存在受影响可能，这类设备资产往往难以通过常规资产扫描识别，统计数据存在偏差。  
# 1. 漏洞基本信息  
## 1.1  漏洞简介  
  
PHP2Shell漏洞指PHP内核session扩展upload_progress（RFC1867）处理路径存在的状态损坏类高危漏洞，涉及CVE-2026-109618与CVE-2026-109619两个编号。攻击者通过发送含PHP_SESSION_UPLOAD_PROGRESS表单字段的multipart/form-data请求，即可在引擎初始化完成前（RINIT之前）触发会话数据的完整反序列化及相关状态操作。  
此阶段若执行用户代码或发生bailout（错误、超时、内存耗尽长跳转），将破坏引擎状态假设，造成状态损坏，可进一步导致内存破坏与远程代码执行。该漏洞CVSS4.0评分为9.2。  
  
漏洞利用无需认证、无需前台账号，目标脚本无需调用session_start()，仅需目标可接收multipart/form-data类型POST请求即可尝试触发。攻击者成功利用后，可在PHP进程权限下执行任意系统命令、植入WebShell、建立持久化访问，并进一步向内网横向渗透。  
## 1.2  受影响版本  
  
本次漏洞影响PHP 8.1至8.5的多个子版本，各分支需升级至官方修复版本。区分官方原版PHP、第三方定制PHP与嵌入式设备内置PHP三类场景：  
  
表1-1 受影响PHP版本与修复版本对照  
  
<table><tbody><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP分支</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">受影响版本</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">修复版本</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">维护状态</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP8.1.x</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">全部子版本</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">无（已</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">EOL）</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">已停止维护</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP8.2.x</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">低于</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.2.35</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.2.35及以上</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">活跃支持</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP8.3.x</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">低于</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.3.36</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.3.36及以上</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">活跃支持</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP8.4.x</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">低于</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.4.27</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.4.27及以上</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">活跃支持</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="132" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP8.5.x</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="142" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">低于</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.5.12</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="139" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">8.5.12及以上</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="163" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">活跃支持</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
·  
   
官方原版PHP：直接通过官方渠道升级至对应修复版本即可完整修复。  
  
  
  
·  
 第三方定制PHP：phpStudy、宝塔、XAMPP等集成环境内置的PHP经过厂商二次打包，需通过对应厂商通道获取适配补丁，直接覆盖官方补丁可能不兼容。  
  
·  
 嵌入式设备内置PHP：路由器、NAS、物联网Web管理后台等设备内置精简PHP环境，版本号可能与官方一致但底层代码已裁剪修改，官方补丁不一定生效，需依赖设备厂商固件更新。  
  
· 漏洞攻击面覆盖所有以PHP作为服务端脚本的Web场景，主要包括Nginx/Apache/Lighttpd等Web服务结合mod_php或PHP-FPM的部署形态，以及命令行（CLI）模式下的PHP运行时。不同部署形态的风险程度存在差异：  
  
·  
 mod_php：官方评估为风险最高的部署形态，已有未经认证请求在应用代码执行前触发攻击的验证利用；CGI形态存在一定风险，取决于Web服务多处理模块配置与进程回收策略。  
  
·  
 PHP-FPM：存在较高风险。工作进程常驻且可复用，攻击者可借助HTTP长连接将多条请求调度至同一工作进程，为跨请求内存状态复用创造条件，应作为重点排查与处置对象。  
  
·  
 CLI：风险相对较低，进程单次执行后退出，难以构造跨请求利用链。  
  
从网络位置看，外网可直接访问的资产面临实时扫描与批量利用风险；仅内网可访问的业务服务器风险次之，但存在被内网攻击者横向利用的可能。需重点说明的是，路由器、NAS、物联网设备等嵌入式资产的内置PHP环境具有特殊性：此类设备通常直接暴露管理端口，缺少基础访问控制，且厂商往往不再提供更新，风险长期存在。  
# 2. 漏洞影响分布  
  
说明：  
本章基于网络空间测绘数据开展影响面分析。需要说明的是，测绘数据对标准云主机与独立服务器覆盖较好，而对路由器、NAS等嵌入式设备覆盖有限，此类设备实际暴露规模可能被低估，统计结果存在偏差。  
## 2.1  全球资产分布概况  
  
据测绘数据显示，两组测绘数据分别匹配到PHP相关记录约5079万条与4700万条，对应独立IP约318万个与191万个，反映出受影响资产在全球范围内具有相当大的暴露规模。端口分布上，443与80端口占比合计超过60%，表明绝大多数受影响资产通过标准Web服务暴露；8080、7080等非标准端口占比较低，但仍构成不可忽视的攻击面。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHkic744kw3Jc0z3o1CflHJZL6tfibqicmQ1qNQh5eA0tXnuriaAjeiciaiaoj44QjeE8PcmvAAWjJzXC9FOhBqbpU3tyv0ENlYo1gX0AZE/640?wx_fmt=png&from=appmsg "")  
  
 图2-1 双平台测绘总量对比  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk9FP6Pvybzlt49t4UyxuuJI8hibtSUr35EIGxJSJ7g7jH93nlM0dGSFXWSDHPV13d0S42b17uPrGMV08IAJcOS6q4AvVnXca9Dc/640?wx_fmt=png&from=appmsg "")  
  
   
图2-2 受影响资产端口分布对比  
  
从国家与地区分布看，美国在两个测绘数据集中均占绝对主导，占比分别约为43.57%与63.48%；德国、新加坡、日本、法国等国家次之。这一分布特征与全球云计算资源分布高度吻合，说明受影响资产以海外云主机与独立服务器为主。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XBFaicYdOHkibS5zDPIr4wPrJ6IoDPsDw7VZPslYzeaSSSXDSpMHvLDEH4ibCZyDsaOsrVa8AQzTSU8YLh32zfXSa1oYMYOmRv3Aia1rk00vM7o/640?wx_fmt=png&from=appmsg "")  
  
 图2-3 受影响资产国家/地区分布  
  
从承载平台看，Cloudflare、Oracle、Hostinger等云服务商与CDN厂商在测绘结果中占据显著份额，其中Cloudflare在部分数据集中占比超过50%。这进一步印证海外受影响资产高度集中于云环境，通常具备相对完善的更新通道与基础防护能力。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk9v4K722B55yOjdkibE3CWMkcHooLeicK9IibSBFUzu7SiaBgpAokLkFic52O0KfsaxfPSD2bkaHYzWYlb9q8qjU6yRn3pPvcX0CYt8/640?wx_fmt=png&from=appmsg "")  
  
   
图2-4 云服务商与CDN集中度  
## 2.2  国内外受影响资产差异特点  
  
综合测绘数据与安天威胁情报分析，国内与国际在受影响资产暴露特征上呈现明显差异，具体对比如下：  
  
表2-1 国内与国际受影响资产特征对比  
  
<table><tbody><tr><td data-colwidth="106" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">对比维度</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="205" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">国际（海外）</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">国内</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="106" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">暴露资产类型</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="205" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">以云主机、独立服务器为主</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">除云业务外，大量嵌入式设备、小型办公网关、</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">NAS内置PHP环境暴露</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="106" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">暴露端口与访问控制</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="205" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">部分存在</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">WAF、云厂商基础防护</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">大量设备直接公网暴露管理端口，缺少基础</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">IP访问限制</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="106" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">资产运维特征</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="205" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">云实例版本更新相对更及时</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">中小单位、嵌入式设备长期不升级，版本老化严重</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="106" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">测绘数据覆盖</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="205" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">覆盖度较高</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">嵌入式设备难以通过常规资产扫描识别，统计数据存在偏差</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
需要特别指出的是，国内大量路由器、NAS、物联网Web管理后台等嵌入式设备内置PHP环境，由于设备指纹非标准、管理端口多样，常规测绘手段难以有效识别，其实际暴露规模可能显著高于测绘数据所见。此类资产应作为国内处置工作的重点关注对象。  
## 2.3  高风险资产清单识别优先级  
  
基于暴露面与可利用性，建议按以下优先级开展资产识别与处置：  
  
表2-2 高风险资产识别优先级  
  
<table><tbody><tr><td data-colwidth="89" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">优先级</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">资产特征</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="229" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">风险说明</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="89" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">优先级</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">1</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">公网直接暴露、使用受影响</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP版本、可直接访问PHP入口页面</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="229" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">面临实时扫描与批量利用，需立即处置</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="89" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">优先级</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">2</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">内网业务服务器，可被横向访问，存在内网攻击者访问入口</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="229" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">一旦内网被突破，将直接面临利用风险</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="89" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">优先级</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">3</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="258" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">路由器、</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">NAS、物联网设备内置PHP，通常无升级通道</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="229" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">风险长期存在，需依赖缓解措施与持续监测</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
   
# 3. 漏洞如何排查  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk8jNZArgs6oLEb5hZ2naXeMunX0lgRL1gmaToQQ9qlJpJaiciaeGS57y5W8yLbudCynXhU3ybLIyFBKs8VQ38No9KvKrZY0QyNxI/640?wx_fmt=png&from=appmsg "")  
  
图3-1 漏洞排查工作流程  
## 3.1  资产测绘与资产梳理  
  
·  
   
梳理业务资产：全面梳理所有搭载PHP的Web服务器、虚拟主机、容器实例及嵌入式设备，建立资产清单。  
  
·  
   
识别PHP部署形态：区分独立PHP、PHP-FPM、容器内PHP与嵌入式内置PHP四种形态，不同形态的排查方法与升级路径存在差异。  
## 3.2  本地主机自查  
  
对可登录的服务器资产，通过以下方式开展主机内部检测：  
  
·  
   
查看PHP版本：执行php -v或php-fpm -v获取版本号，对照受影响版本范围判定。  
  
·  
   
查看php.ini配置：确认session.upload_progress.enabled等关键配置状态，识别是否存在异常改动。  
  
·  
   
查看服务日志：审计Web服务访问日志、PHP-FPM日志、PHP错误日志，排查异常请求与异常脚本写入行为。  
## 3.3  远程漏洞探测  
  
·  
   
非破坏性POC探测：优先使用非破坏性验证请求，避免对业务造成影响；禁止在业务高峰期执行。  
  
·  
   
主动扫描工具探测：使用扫描器批量识别受影响资产时，应控制扫描速率与并发，避免引发服务异常。  
  
提示：  
所有探测行为须获得资产所有方授权，禁止未授权测试。生产环境验证前应在测试环境完成预验证。  
## 3.4  可疑行为排查  
  
·  
   
查找WebShell：检查Web目录下新增的未知PHP文件、异常PHP文件，重点关注包含eval、assert、system、shell_exec等函数的文件。  
  
·  
   
审计访问日志：检索含PHP_SESSION_UPLOAD_PROGRESS字段的multipart请求、异常POST参数、异常文件写入行为。  
  
·  
   
检查系统层面：核查系统账号、计划任务、运行进程，排查异常后门、反弹Shell等恶意行为。  
## 3.5  排查注意事项  
  
嵌入式路由器/NAS设备的排查存在特殊性：此类设备通常无法执行命令查看PHP版本，只能通过设备型号、固件版本结合POC探测验证；同时设备日志留存时间短，溯源难度大。对确认受影响的嵌入式设备，应优先采用网络隔离与访问控制措施，并持续监测其外联与访问行为。  
# 4. 漏洞如何修补  
## 4.1  官方升级下载  
  
从PHP官方渠道下载对应分支的安全更新包，优先升级至官方修复后的稳定版本。不同运行环境的升级操作存在差异：  
  
·  
   
源码编译环境：获取官方源码包，重新编译安装。  
  
·  
   
rpm/deb包环境：通过系统包管理器或官方仓库升级。  
  
·  
   
容器镜像：重新构建镜像，替换基础镜像后重新发布，禁止直接在容器内打补丁。  
  
·  
   
Windows环境PHP：通过官方Windows构建或Web平台安装器升级。  
  
提示：  
容器场景下，直接在运行容器内修改PHP文件或打补丁的方式不可取，容器重建后修改将丢失，且可能引入镜像层不一致问题。必须通过重新构建镜像的方式完成修复。  
## 4.2  第三方打包PHP  
  
对于phpStudy、宝塔、XAMPP等集成环境内置的PHP，应通过对应集成环境厂商的升级通道，使用厂商提供的安全版本更新。直接覆盖官方补丁文件可能因编译参数、补丁路径差异导致不兼容或修补不全。  
## 4.3  嵌入式设备  
  
对于路由器、NAS等嵌入式设备，首先查看设备厂商是否发布固件更新，优先升级设备固件。对厂商已不再维护的老旧设备，无法通过升级修复，需启用缓解措施（详见第7章），并评估设备替换或下线方案。  
# 5. 补丁有效性分析  
  
说明：  
本章基于安天AVLCode代码分析平台开展源码层面验证，区别于业务环境POC动态测试，用于验证补丁底层修复逻辑完整性，评估是否存在潜在绕过缺口。由于源码  
及二进制  
静态分析对人员能力与工具门槛要求较高，一般业务运维侧不具备开展条件，建议优先采用  
6.2章节  
POC动态测试结合业务回归的方式完成补丁有效性核验。  
## 5.1  验证环境与验证方法  
  
·  
   
依托安天AVLCode代码分析平台，对漏洞原始源码与官方补丁diff进行对比分析。  
  
·  
   
静态代码审计：对比补丁前后关键代码分支，定位漏洞触发点的代码修改逻辑。  
  
·  
   
代码层面校验：判断补丁是否完整阻断漏洞触发条件，评估多场景下绕过风险。  
## 5.2  补丁代码变更分析  
  
以PHP8.5.11至8.5.12的升级为例，官方共发布287个文件变更，其中绝大部分为常规bugfix，安全相关变更集中在5个代码文件与2个测试文件。  
补丁修复主题为“Avoid state corruption via bailout”，核心修改位于ext/session/session.c与main/rfc1867.c。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XBFaicYdOHk8zYwTVvPdatII6vhH65ouia0nniaJIviczxpdawkTIXetVhmFEWHiaAic1aRhVvSiblpK3tTCJWibibpTtzrO6nlxoXSCPB66ib11JqXgk/640?wx_fmt=png&from=appmsg "")  
  
   
图5-1 补丁代码变更构成分析  
  
从漏洞机制看，攻击链为：攻击者发送含PHP_SESSION_UPLOAD_PROGRESS字段的multipart请求，PHP在RINIT之前由RFC1867上传进度钩子进入php_session_rfc1867_update()，其中调用php_session_initialize()读取并完整反序列化会话文件。  
修复前，该反序列化允许autoload、实例化真实类、触发__wakeup()/__unserialize()，导致用户代码在引擎尚未完成初始化的阶段执行；此阶段的对象析构、错误处理或bailout与引擎状态假设冲突，造成状态损坏。  
  
官方补丁通过四道防线封堵该路径：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk8v6oF78bqfLyIycf9TMfibhyLroMUib9ypFPBYXhoHXyoNM2MQOIK0c6ROOquzA2ntsu1K9j8TevzV8CIa5a62YbicIZy9omRo2o/640?wx_fmt=png&from=appmsg "")  
  
图5-2 补丁四道防线与攻击面对应关系  
  
防线一：pre-RINIT反序列化强制incomplete-class模式。新增php_session_unserialize_init()包装，在in_rfc1867_update标志下强制对象保持__PHP_Incomplete_Class，禁止autoload、unserialize_callback_func、__wakeup()/__unserialize()等用户代码执行。  
  
防线二：update/cleanup路径失败早退。php_session_rfc1867_update()与php_session_rfc1867_cleanup()在initialize失败或会话状态异常时立即返回，不进入后续逻辑。  
  
防线三：防止request_parse_body二次进入损坏会话，避免重复解析破坏已建立的会话状态。  
  
防线四：状态清理。pre-RINIT阶段提前调用php_rinit_session_globals()清理会话全局状态，$_SESSION以incomplete模式填充后显式删除，避免污染后续session_start()。  
  
配套修复方面，main/rfc1867.c将全部POST变量注册函数由void改为bool返回，失败路径经ZEND_HAS_OOM()判定，并补充post_max_size超限检查，属于同一bailout安全主题的加固。  
新增回归测试rfc1867_sid_invalid.phpt包含官方确认的触发流量样本，可作为检测规则锚点。  
## 5.3  补丁有效性结论  
  
结论：该补丁在原生PHP环境下，可完整修复漏洞触发条件，阻断漏洞利用入口。  
  
补丁有效性边界如下：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHkiciayPKjT3ibIvgenHYibVjdWcn8RfB953ZEV2CWVo5JibvuZVsCanIYJdSfmXue0GcaTpfJYdwtBtC9Qaybgm3kYXlaKN0ossroAs/640?wx_fmt=png&from=appmsg "")  
  
图5-3 补丁有效性边界示意  
  
·  
   
仅针对官方原版PHP：第三方修改版、厂商二次定制PHP，补丁可能存在不兼容或修补不全的情况。  
  
·  
   
嵌入式设备厂商自研裁剪PHP：即使版本号与官方一致，底层代码可能不同，补丁不一定生效，需依赖设备厂商适配。  
  
·  
   
补丁仅修复PHP2Shell漏洞本身，不修复其他独立漏洞，也不能抵御业务层代码漏洞、弱口令等其他攻击。  
## 5.4  补丁局限性说明  
  
·  
   
补丁仅封堵漏洞利用入口，无法清除服务器上已存在的WebShell、系统后门。若主机已被入侵，仍需开展恶意文件查杀、后门清理等处置工作。  
  
·  
   
若Web服务存在WAF、代理或业务代码层面的特殊改写，可能影响补丁的实际生效场景，需在升级后开展利用验证。  
# 6. 修补效果验证  
## 6.1  基础环境验证  
  
重启相关服务（php-fpm、nginx/apache等Web服务），查看PHP版本，确认升级或补丁包已成功加载。  
  
检查php相关配置文件是否被升级覆盖，确认原有业务配置不丢失。  
## 6.2  漏洞利用验证  
  
使用POC进行利用测试，查看利用是否失败，观察PHP行为与返回包。  
  
对比打补丁前后的响应差异，确认漏洞利用链路被阻断。  
  
建议在测试环境先行验证，再在生产环境执行验证，避免业务风险。  
## 6.3  业务回归验证  
  
验证业务功能可用性，升级或补丁后检查网站业务、接口是否正常，排除升级带来的兼容性问题。  
## 6.4  日志观测  
  
持续观测Web日志、PHP错误日志，确认不再出现漏洞相关特征请求的成功执行记录。重点关注PHP-FPM工作进程异常退出（signal 11）与pre-RINIT阶段“Failed to read session data”告警激增等触发侧特征。  
# 7.  其他缓解措施  
  
重要提示：  
缓解措施为临时应急手段，不能替代官方补丁修复，存在被绕过风险，具备条件仍需尽快升级。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XBFaicYdOHkibnWbdVkKKft36XWKavhQ3dWWXWwsGGoicpAvS8gl3coLlAXiaWQluJch4SeW7PPeDQm0ojEwHppzfwth8oARU8F1tPaV6PBk3Yo/640?wx_fmt=png&from=appmsg "")  
  
   
图7-1 临时缓解措施层级  
## 7.1  网络访问控制  
  
·  
   
配置防火墙或安全组，限制外网对PHP管理页面、高危PHP入口的访问，仅允许可信IP访问。  
  
·  
   
关闭不必要公网端口，关闭不必要的PHP站点与虚拟主机。  
## 7.2  WAF防护规则  
  
在Web应用防火墙中添加漏洞特征防护规则，拦截含PHP_SESSION_UPLOAD_PROGRESS字段的multipart请求及异常超长文件名请求。需注意WAF基于流量特征检测，载荷变形后存在绕过可能。  
## 7.3  PHP配置加固  
  
·  
   
禁用危险函数：通过disable_functions限制system、exec、eval、passthru等代码执行类函数。  
  
·  
   
关闭危险配置项：关闭远程文件包含等高危特性，管控文件写入能力。  
  
·  
   
限制目录写入权限：  
Web站点目录不赋予PHP脚本写入权限，遵循最小权限原则。  
  
·  
   
关闭上传进度统计：设置  
session.upload_progress.enabled=Off，阻断当前已知的漏洞利用路径。  
  
·  
   
内存上限配置说明：未完成补丁修复的环境，官方临时方案可将  
memory_limit配置为 - 1，移除可控内存上限带来的失败触发路径。该操作必须搭配请求限流、资源隔离（cgroup）保障业务稳定；该方式无法根除漏洞，底层内存分配异常仍存在触发风险。  
不建议使用ulimit、LimitAS、LimitDATA替代资源隔离，禁止不加限流直接在生产环境取消内存限制。  
  
·  
   
上传进度开关持续管控：即便已经升级安装本次漏洞补丁，受同系列其他会话序列化漏洞影响，建议暂时保持  
session.upload_progress.enabled=Off，等待后续版本完善。  
  
·  
   
补充防护边界说明：禁用函数、目录访问限制仅作为叠加防护手段，官方不认可二者为可靠的安全边界，无法约束该内存破坏漏洞被成功利用后的进程行为。  
## 7.4  业务层面加固  
  
·  
   
移除不需要的PHP测试页面、phpinfo页面、后台管理演示页面。  
  
·  
   
对Web目录做权限管控，禁止上传目录执行PHP脚本。  
## 7.5  系统层面加固  
  
对承载PHP业务的服务器实施操作系统安全基线加固，清理冗余账号、关闭无用系统服务与监听端口，完善系统审计策略，降低漏洞被利用后攻击者实施权限提升、横向移动的风险。  
  
建议部署安天智甲终端防御系统（企业版），借助产品的安全基线核查、漏洞资产监测、内核主动防御、主机防火墙与行为审计能力，构建服务器端点纵深防护。智甲可持续监测PHP组件版本风险，捕获漏洞利用触发的异常命令执行、恶意文件写入等攻击行为，阻断漏洞利用后的后续攻击链路，集中管理多台服务器安全状态，作为临时缓解措施的有效补充。  
  
Linux平台加固要点：  
  
•运行身份隔离：每站点使用独立Unix用户与独立FPM池，高价值站点采用独立服务或容器部署，PHP worker进程不以root身份运行，抑制跨站读取、篡改其他业务与直接接管主机。  
  
•文件权限收敛：代码、依赖与PHP配置由部署账号持有，PHP运行账号只读，写权限仅开放上传、缓存与会话目录，抑制代码篡改、配置篡改与WebShell持久化。  
  
•上传目录限制：挂载noexec只能限制直接运行落地二进制，PHP仍可能读取并解释文件；应同时在Web服务器禁止上传目录的脚本处理，最好将上传内容置于Web根目录之外。  
  
•强制访问控制：SELinux保持Enforcing，或为PHP配置AppArmor profile，只授权必要的目录、程序与网络行为，进一步收紧PHP进程的行为边界。  
  
•systemd沙箱：通过ProtectSystem=strict配合ReadWritePaths=写目录白名单限制PHP进程的文件系统写入范围，并按兼容性启用NoNewPrivileges=yes、ProtectHome=yes、PrivateTmp=yes、PrivateDevices=yes；需注意NoNewPrivileges不能阻止所有经IPC服务完成的动作，也不能修复内核漏洞。  
  
•内存写执行限制：MemoryDenyWriteExecute=yes可增加部分内存载荷执行难度，但可能与PHP JIT及扩展冲突，也存在绕过条件，须经兼容性测试后启用，不能视为必然阻断。  
  
•能力与系统调用收敛：移除PHP服务不需要的capabilities，在独立PHP服务中按实际需求配置seccomp/SystemCallFilter，限制挂载、调试其他进程、命名空间操作等高危行为。  
  
•出站访问控制：通过nftables、容器网络策略或出口网关限制PHP服务器出站连接，仅放行必要的数据库、缓存、DNS与业务API，抑制反连、下载载荷、数据外传与内网扫描。  
  
•宿主机隔离：不向PHP容器挂载Docker socket、宿主根目录与部署密钥，容器以非特权方式运行且根文件系统只读，避免站点失陷扩大为宿主机或集群失陷。  
  
Windows平台加固要点：  
  
•服务身份隔离：IIS每站点使用独立应用池并采用ApplicationPoolIdentity，Apache使用专用低权限服务账号承载PHP，避免以LocalSystem或管理员账户运行，抑制主机接管、跨站访问与高权限凭据获取。  
  
•NTFS权限管控：IIS应用池按池SID设置资源ACL，实际按站点授权，避免将多个站点开放给宽泛的共享组；代码、PHP安装目录与配置仅可读，写权限限定在业务数据目录，抑制WebShell持久化与DLL替换。  
  
•令牌权限检查：核查实际PHP进程令牌，移除不必要的模拟、调试等权限；调整前需验证FastCGI与身份模拟的兼容性。  
  
•子进程限制：对无需调用外部工具的php-cgi.exe，测试启用Exploit Protection的Do not allow child processes；限制应作用于实际PHP进程，直接禁止w3wp.exe创建子进程可能影响FastCGI正常启动及图片处理、PDF转换等外调工具。  
  
•应用程序控制：部署App Control for Business或适用的AppLocker策略，限制可执行文件、DLL与脚本宿主；需注意AppLocker不会阻止PHP解释.php WebShell，其控制的脚本类型不包含任意PHP代码。  
  
•利用缓解：保持DEP、ASLR等利用缓解机制，对PHP单独测试ACG等额外策略；ACG与Code Integrity Guard需兼容性测试（尤其是JIT），第三方PHP模块可能无法加载，不宜直接套用仅允许微软签名模块的策略。  
  
•出站访问控制：Windows Firewall或网络出口默认拒绝非必要连接，仅允许指定目标与端口；出站白名单需设计默认动作与规则优先级，避免显式阻断规则覆盖允许规则。  
  
•DEP策略：64位Windows上原生64位进程默认始终启用DEP，将系统策略改为覆盖所有程序通常不会为64位PHP增加新保护；32位PHP进程才需要评估策略调整或按程序单独强制DEP。  
  
•DEP启用方式：仅针对Windows平台的专用PHP服务器，完成兼容性验证后，可选用排除策略（OptOut），或在防护组件内对承载PHP的宿主进程单独开启DEP；配置变更后需要重启对应进程并核查业务运行状态。  
  
•DEP防护边界：DEP仅防护数据内存区域直接执行代码，无法阻止漏洞本身造成的内存损坏，需要搭配低权限运行账号、代码目录只读、子进程创建与对外出站访问限制一起实施。  
## 7.6  入侵监测加固  
  
·  
   
部署WebShell检测与异常文件监控，对Web目录新增文件进行告警。  
  
·  
   
开启Web访问日志，延长日志留存时间，配置告警规则，监控漏洞特征请求。  
# 8. 后续需要重点关注的情况  
## 8.1  漏洞变种与绕过监测  
  
·  
   
持续关注是否出现漏洞绕过POC、新的利用变种，关注厂商后续补充补丁。  
  
·  
   
关注第三方定制PHP、嵌入式设备的特殊绕过场景。  
## 8.2  资产持续监测  
  
·  
   
定期资产巡检，持续监测受影响PHP资产，防止新增上线的未加固服务器。  
  
·  
   
重点关注长期无法升级的路由器、NAS等物联网设备，持续监测访问日志。  
## 8.3  威胁情报监测  
  
监测互联网上利用该漏洞的攻击活动、恶意样本、挖矿或勒索载荷，关注安天威胁情报更新。  
## 8.4  运营与复盘  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk8bjqNdH7NfYzrgJ6RpggMQlpkCRdQvrfZ8HQn3byA0EzTr86FHicpCu0PyF6uia3uq22qSzlD1vGPTMNlg7sO6g3fltiaQZy1MtM/640?wx_fmt=png&from=appmsg "")  
  
   
图8-1 漏洞闭环管理流程  
  
·  
   
建立漏洞闭环台账，记录资产、处置时间、加固方式、验证结果。  
  
·  
   
定期复盘，优化资产发现与漏洞应急响应流程。  
## 8.5  长期安全建议  
  
·  
   
建立PHP环境版本管理与定期更新机制。  
  
·  
   
上线前安全测试，避免老旧版本PHP直接上线公网。  
# 附录A  自查命令清单  
  
表A-1 自查命令清单  
  
<table><tbody><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">场景</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">命令</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">说明</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">查看</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP版本</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php -v</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">获取</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP版本号</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">查看</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">PHP-FPM版本</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php-fpm -v</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">获取</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">FPM版本号</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">查看加载配置</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php --ini</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">定位</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php.ini路径</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">查看上传进度配置</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php -i | grep upload_progress</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">确认</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">session.upload_progress状态</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">查看禁用函数</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">php -i | grep disable_functions</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">确认危险函数禁用情况</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="148" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">检索特征请求</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="218" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">grep &#34;PHP_SESSION_UPLOAD_PROGRESS&#34; /var/log/nginx/access.log</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="211" width="258" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px">检索攻击触发字段</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
   
# 附录B  POC测试注意事项  
  
授权声明：  
所有POC测试须获得资产所有方书面授权，禁止未授权测试。生产环境验证前应在测试环境完成预验证，避免引发服务崩溃或业务中断。  
  
·  
   
使用非破坏性验证请求，避免触发内存耗尽导致进程崩溃。  
  
·  
   
控制测试频率与并发，避开业务高峰期。  
  
·  
   
保留测试记录，包括请求样本、响应差异、时间戳，用于后续溯源。  
# 附录C  临时缓解配置参考  
  
网络访问控制配置示例：  
  
location ~ ^/(admin|phpmyadmin) {  
  
    allow 203.0.113.0/24;  
  
    deny all;  
  
}  
  
管理页面路径与可信IP范围按实际环境替换（示例IP为文档保留地址段）；关闭不必要的公网端口、站点与虚拟主机，需在服务器防火墙与云安全组侧同步实施。  
  
php.ini配置示例：  
  
session.upload_progress.enabled = Off  
  
disable_functions = system,exec,shell_exec,passthru,eval,assert  
  
allow_url_fopen = Off  
  
allow_url_include = Off  
  
memory_limit = -1  
  
open_basedir = /var/www/html:/tmp  
  
配置说明：memory_limit = -1仅适用于尚未完成补丁修复的环境，必须配合请求限流与cgroup等资源隔离手段保障可用性，不建议以ulimit、LimitAS、LimitDATA替代隔离，禁止不加限流直接在生产环境取消内存限制；open_basedir为补充防护手段，官方明确其不是可靠的安全边界，目录范围按实际站点路径调整。  
  
WAF规则要点：  
  
·  
   
拦截含PHP_SESSION_UPLOAD_PROGRESS字段的multipart/form-data请求。  
  
·  
   
拦截文件名超过2KB的multipart上传请求。  
  
·  
   
对同一源IP短时间内的异常multipart请求实施限速。  
  
·  
   
对明确不具备上传功能的接口，直接阻断multipart/form-data请求。  
  
Web服务器上传目录配置示例：  
  
# nginx  
  
location ^~ /uploads/ {  
  
    location ~ \.(php|php5|phtml)$ { deny all; }  
  
}  
  
  
# apache  
  
<Directory "/var/www/uploads">  
  
    <FilesMatch "\.(php|php5|phtml)$">  
  
        Require all denied  
  
    </FilesMatch>  
  
</Directory>  
  
上传目录路径按实际环境替换，按所用Web服务器选择对应示例；上传内容最好置于Web根目录之外，并在Web服务器侧禁止上传目录的脚本处理。  
  
Linux系统配置示例：  
  
# systemd service unit  
  
[Service]  
  
ProtectSystem=strict  
  
ReadWritePaths=/var/www/html  
  
NoNewPrivileges=yes  
  
MemoryDenyWriteExecute=yes  
  
  
# /etc/fstab  
  
/dev/sdb1  /var/www/uploads  ext4  defaults,noexec  0  2  
  
写入目录、磁盘设备与文件系统类型按实际环境替换；MemoryDenyWriteExecute=yes可能与PHP JIT及扩展冲突，须经兼容性测试后启用；systemd配置修改后执行systemctl daemon-reload并重启对应服务；noexec仅限制直接运行落地二进制，PHP仍可能读取并解释文件；代码目录只读可通过属主与权限调整实现（如chmod -R a-w /var/www/html），写操作仅保留必要目录。  
  
Windows平台配置示例：  
  
·  
   
DEP策略：64位Windows上原生64位进程默认始终启用DEP，通常无需额外调整；专用PHP服务器经兼容性验证后，可以管理员身份执行bcdedit /set nx OptOut并重启采用排除策略，或在系统“漏洞防护”的程序设置中对承载PHP的宿主进程单独强制DEP，调整后重启进程并核查运行状态。  
  
·  
   
IIS应用池：使用应用程序池标识（池SID）设置资源ACL，按站点实际授权，避免将多个站点开放给宽泛的共享组。  
  
·  
   
子进程限制：限制策略作用于实际PHP进程（如php-cgi.exe），直接禁止w3wp.exe创建子进程可能影响FastCGI正常启动及图片处理、PDF转换等外调工具。  
  
·  
   
ACG与Code Integrity Guard：在“漏洞防护”中逐项启用并验证第三方PHP模块可正常加载，不宜直接套用仅允许微软签名模块的策略。  
  
·  
   
出站白名单：在Windows防火墙中按默认阻止、显式允许业务必需目标的方式设计规则，并梳理规则优先级，避免显式阻断规则覆盖允许规则。  
# 附录D  术语说明  
  
表D-1 术语说明  
  
<table><tbody><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">术语</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">说明</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP2Shell</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP内核session扩展upload_progress路径状态损坏漏洞的社区指代</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">RINIT</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP请求初始化阶段，早于业务脚本执行</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">bailout</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP引擎错误、超时或内存耗尽时的长跳转机制</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">incomplete-class</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP反序列化时类未定义时的占位对象模式</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">RFC1867</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">PHP处理multipart/form-data文件上传的内部实现模块</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">WebShell</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">以</span></font><font face="微软雅黑"><span leaf="">Web脚本形式驻留服务器的后门，可远程执行命令</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="174" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">EOL</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="387" width="387" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">End of Life，停止维护，不再提供安全更新</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
   
  
**往期推荐:**  
  
  
  
  
  
[关于PHP核心远程代码执行漏洞的风险提示及研判思考](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215619&idx=1&sn=f05d1b820db4feca2afaa773b09fe016&scene=21#wechat_redirect)  
  
  
  
[CVE-2026-109618、CVE-2026-109619（PHP2Shell）漏洞FAQ](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215632&idx=1&sn=58bb9d14311e8757f28334bddf6394be&scene=21#wechat_redirect)  
  
  
  
[抗战胜利纪念日，安天澜砥模型N2.6正式上线](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215429&idx=1&sn=90a38343331dcf53aace361ea618edfb&scene=21#wechat_redirect)  
  
  
  
[2026年10月微软补丁日风险提示](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215613&idx=1&sn=3e5649d5630933cccfa8626987b1efd4&scene=21#wechat_redirect)  
  
  
  
[2026中国网络安全创新创业大赛 | 安天AVL Code获奖](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215515&idx=1&sn=1dea52c669e041d35f049f0fdce5bb1e&scene=21#wechat_redirect)  
  
  
