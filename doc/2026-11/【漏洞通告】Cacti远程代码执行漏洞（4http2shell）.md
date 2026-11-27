#  【漏洞通告】Cacti远程代码执行漏洞（4http2shell）  
NS-CERT
                    NS-CERT  绿盟科技CERT   2026-11-27 10:35  
  
**通告编号 NS-2026-0042**  
  
2026-11-27  
  
<table><tbody><tr><td data-colwidth="26%" width="26%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;white-space: nowrap;padding: 5px;"><strong><span style="font-size: 14px;color: #333;"><span leaf="">TAG：</span></span></strong></td><td colspan="3" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;padding: 5px;"><p><strong style="caret-color: red;"><span style="font-size: 14px;" microsoft="microsoft" yahei="yahei"><span leaf="">Cacti、远程代码执行、4http2shell</span></span></strong></p></td></tr><tr><td data-colwidth="26%" width="26%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;white-space: nowrap;padding: 5px;"><strong><span style="font-size: 14px;color: #333;"><span leaf="">在野利用：</span></span></strong></td><td data-colwidth="22%" width="22%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;padding: 5px;"><p style="vertical-align: inherit;"><strong style="caret-color: red;font-size: 14px;line-height: 1.5em;" microsoft="microsoft" yahei="yahei"><span leaf="">未发现</span></strong></p></td><td data-colwidth="24%" width="24%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;white-space: nowrap;padding: 5px;"><strong><span style="font-size: 14px;color: #333;"><span leaf="">PoC状态：</span></span></strong></td><td data-colwidth="28%" width="28%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;padding: 5px;"><p style="vertical-align: inherit;"><strong style="caret-color: red;font-size: 14px;line-height: 1.5em;" microsoft="microsoft" yahei="yahei"><span leaf="">已公开</span></strong></p></td></tr><tr><td data-colwidth="26%" width="26%" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;white-space: nowrap;padding: 5px;"><strong><span style="font-size: 14px;color: #333;"><span leaf="">漏洞危害：</span></span></strong></td><td colspan="3" valign="top" style="border-color: rgb(216, 216, 216);margin: 0;word-break: break-all;padding: 5px;"><p style="vertical-align: inherit;"><strong style="caret-color: red;font-size: 14px;line-height: 1.5em;" microsoft="microsoft" yahei="yahei"><span leaf="">攻击者利用此漏洞，可实现远程代码执行。</span></strong></p></td></tr></tbody></table>  
  
**1**  
  
  
**漏洞概述**  
  
  
近日，绿盟科技CERT监测到到网上披露了Cacti远程代码执行漏洞（4http2shell），又称“4HTTP2ROOT”；由于Cacti未正确验证用户配置项preview_graphs_per_page且数据库默认配置可被直接拼接至SQL LIMIT子句中，在启用访客账户的情况下，未经身份验证的攻击者可利用堆叠查询将poller_item.arg1修改为恶意命令，并通过访客可访问的实时图表接口触发轮询器执行，从而以Cacti进程权限执行任意代码。CVSS评分9.8，目前漏洞细节与PoC已公开，请相关用户尽快采取措施进行防护。  
  
Cacti是一款基于PHP、MySQL、SNMP及RRDTool开发的网络流量监测图形分析工具，提供数据存储、图形展示、阈值告警和集中监控等功能，广泛应用于企业网络及基础设施监控场景。  
  
  
参考链接：  
  
https://watchtowr.com/intelligence/4http2root-cacti-zero-day-faq/  
  
  
**SEE MORE →**  
  
**2****影响范围**  
  
### 受影响版本  
- Cacti = 1.2.31（已验证版本）  
  
注：影响启用Guest账户且接口可被访问的Cacti实例。  
  
  
**3****漏洞检测**  
  
### 3.1 人工检测  
  
相关用户可以通过检查Cacti版本及访客账户配置情况，判断当前Cacti实例是否在受影响范围：  
  
1、检查Cacti版本：登录Cacti Web控制台，在“Utilities > System Utilities”页面中查看Cacti Version；或查看页面底部显示的版本信息。  
  
2、检查匿名访客访问配置：在“Settings > Authentication > Special Users”中查看“Guest User”配置。若设置为“No User”，表示未启用匿名访客访问；若已指定具体用户，还需在“User Management”中确认该用户的“Enabled”状态为启用。  
  
若当前Cacti版本在受影响范围，且访客账户处于启用状态，则可能存在安全风险。  
  
  
**4****漏洞防护**  
  
### 4.1 官方升级  
  
目前官方暂未发布更新修复此漏洞，请受影响的用户关注官方安全动态并及时更新，下载链接：https://www.cacti.net/download_cacti.php  
  
### 4.2 其他防护措施  
  
在不影响业务的情况下，相关用户可采取以下措施进行临时缓解：  
  
1、禁用Cacti访客账户，并对Cacti Web界面进行访问限制。  
  
2、检查poller配置中是否存在异常命令，关注日志中图表显示设置出现非数字值的告警。  
  
  
**END**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qR4ORTNELImFwJM2rh6GKbnrurdFA28jJ8chUPyC1U6aW3jhenqEiaXkmeGVmfOnvAJy8j3My901JQ7emHaicYzA/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/qR4ORTNELImFwJM2rh6GKbnrurdFA28jib7icfic0lJJHh3eLRpIXiaia08KqOSEibBsz64vlOH9aqicu3lmjccEeAFWQ/640?wx_fmt=jpeg "")  
  
**声明**  
  
本安全公告仅用来描述可能存在的安全问题，绿盟科技不为此安全公告提供任何保证或承诺。由于传播、利用此安全公告所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，绿盟科技以及安全公告作者不为此承担任何责任。  
  
绿盟科技拥有对此安全公告的修改和解释权。如欲转载或传播此安全公告，必须保证此安全公告的完整性，包括版权声明等全部内容。未经绿盟科技允许，不得任意修改或者增减此安全公告内容，不得以任何方式将其用于商业目的。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/qR4ORTNELImFwJM2rh6GKbnrurdFA28jib7icfic0lJJHh3eLRpIXiaia08KqOSEibBsz64vlOH9aqicu3lmjccEeAFWQ/640?wx_fmt=jpeg "")  
  
  
**绿盟科技CERT**  
  
∣  
微信公众号  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tZgxn6yByvKXsABGjbFCll2soNtEvW7Ws2RpV33IGH3qtz7wSDStg3GNz23W8ibCCPOzyuD76oDhAtKhMacZTqK9B7fos0iacTG8olq5gicjicE/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz/Hu8hctxHqSW0nSJn8p8OHVEQwHicSwTibFJMBE650AxdzfISoeY8woe2QsgCINIBrccBOOUft2HuU0GsNQWibSG7g/640?wx_fmt=png "")  
  
长按识别二维码，关注网络安全威胁信息  
  
