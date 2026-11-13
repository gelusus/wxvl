#  漏洞复现 | 大蚂蚁 (BigAnt) 即时通讯系统 uploadMultipleFile 任意文件上传漏洞  
 实战安全研究   2026-11-13 13:27  
  
**免责声明**  
<table><tbody><tr style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;"><td valign="top" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;visibility: visible;"><span style="color: rgb(255, 0, 0);letter-spacing: 0.544px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;visibility: visible;font-size: 14px;"><span leaf="">本文仅用于技术学习和安全研究，请勿使用本文所提供的内容及相关技术从事非法活动，由于传播和利用此文所提供的内容或工具而造成任何直接或间接的损失后果，均由使用者本人承担，所产生一切不良后果与文章作者及本账号无关。如内容有争议或侵权，请私信我们！我们会立即删除并致歉。谢谢！</span></span></td></tr></tbody></table>  
1  
  
**漏洞描述**  
  
  
  
杭州九麒科技大蚂蚁 (BigAnt) 即时通讯系统是一款企业级IM通信管理系统，提供多种功能支持。该系统的 uploadMultipleFile 接口存在目录遍历+任意文件写入/上传漏洞，攻击者可以通过上传特制的 php 文件，执行恶意代码，实现服务器的远程控制，可能导致敏感信息泄露、数据篡改等危害。  
  
2  
  
**影响版本**  
  
  
  
   
杭州九麒科技大蚂蚁 (BigAnt) 即时通讯系统  
  
3  
  
**测绘语法**  
  
  
  
fofa语法  
```
(body="/Public/static/admin/admin_common.js" && body="/Public/lang/zh-cn.js.js") || title="即时通讯 系统登录" && body="/Public/static/ukey/Syunew3.js"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yIciaKAicYtorwCezQ4vuI3SzdRqqrjzGh2Mc22Z1iaZOp98BGpDaGRVOF5snYMCB0qApqzKHbUgzXqIoPd6wdHc5hyzuJRlqae6Vw7YGiaTBqw/640?wx_fmt=png&from=appmsg "")  
  
  
4  
  
**漏洞复现**  
  
  
  
上传文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yIciaKAicYtor4mToOsbUFy61YpWFyKdZVwkDibiaqicTHFokXFAjQcicBQtVMkANA2lG3obhO5RoOrKYgUHghr2Em2N3icWoqJomiaz4tEOzlTwwhk/640?wx_fmt=png&from=appmsg "")  
  
访问文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yIciaKAicYtoprZE4kHf8YXp52sRSbBj4ATKT5bZrw3iaEXpNNhPb1VWnsicdV0ZDnAjfZnAzMkO2hxVpgzsOafHL3yurJPLbsVdYZXxmgmABoU/640?wx_fmt=png&from=appmsg "")  
  
  
5  
  
**检测POC**  
  
  
  
nuclei  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yIciaKAicYtooPNbRhpR9IBleiaciavZfVqXPyFyOibA69MTWgnUMlexRksVyv4ryh8rLQk1lYxqqadTicOhQ20MDaibRphibvDztkknw5JTAzquJK4/640?wx_fmt=png&from=appmsg "")  
  
afrog  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yIciaKAicYtooTKeLBDA0ic3AeOSM5djL56wFy1u7s5v02ibniaezbProMs4eBjrodrwUGT4ibaHtdXOKygpvpBQ9hR9e1RInLnbj6OibgGt7TlwhM/640?wx_fmt=png&from=appmsg "")  
  
  
6  
  
**漏洞修复**  
  
  
1、建议联系厂商打补丁或升级版本。  
  
2、增加Web应用防火墙防护。  
  
3、关闭互联网暴露面或接口设置访问权限。  
  
7  
  
**内部圈子**  
  
  
**现在已更新POC数量 2500+（中危以上）**  
  
🔥 **1day/Nday 漏洞实战圈上线**  
 🔥  
  
还在到处找公开漏洞 POC？  
  
这里专注整合全网公开1day/Nday漏洞POC和复现，一站式解决你的痛点！  
  
🔍   
圈子福利  
  
✅ 整合全网 1day/Nday 漏洞POC，附带复现步骤，新手也能快速上手  
  
✅ 每周更新 7-15 个POC测试脚本，经过实测验证，到手就能用  
  
✅ 完美适配 Nuclei/Afrog 扫描工具，脚本无需额外修改，即拿即用  
  
✅ 临时福利：免费 FOFA 高级会员查询，无需账号也能高效资产测绘  
  
✅ 专属权益：提供指纹识别库，指纹库持续更新  
  
💡   
适合对象  
  
渗透测试🔹攻防演练🔹安全运维🔹企业自查  
🔹SRC漏洞挖掘  
  
⚠️   
重要提醒  
  
仅限授权范围内的合法安全测试，严禁用于未授权攻击行为！  
  
本服务为虚拟资源服务，一经购买概不退款，请按需谨慎购买！  
  
目前圈子已满200人，价格由66.9调整为69.9元（  
交个朋友啦  
），250人后调整为71.9元。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/yIciaKAicYtooOcDoeEYZtETDFQ4rjVvy92CZjibia4I6kmdiakLGDf8DbibXuyXo44ic8YGePNBib59bWUHegCYsgPATaVUw8r11f9B7uzwfczOyt4/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
