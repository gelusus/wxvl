#  漏洞复现 | 泛微E-cology9 doc 存在SQL注入漏洞  
 实战安全研究   2026-10-27 13:11  
  
**免责声明**  
<table><tbody><tr style="-webkit-tap-highlight-color: transparent;outline: 0px;visibility: visible;"><td valign="top" style="-webkit-tap-highlight-color: transparent;outline: 0px;word-break: break-all;hyphens: auto;visibility: visible;"><span style="color: rgb(255, 0, 0);letter-spacing: 0.544px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;visibility: visible;font-size: 14px;"><span leaf="">本文仅用于技术学习和安全研究，请勿使用本文所提供的内容及相关技术从事非法活动，由于传播和利用此文所提供的内容或工具而造成任何直接或间接的损失后果，均由使用者本人承担，所产生一切不良后果与文章作者及本账号无关。如内容有争议或侵权，请私信我们！我们会立即删除并致歉。谢谢！</span></span></td></tr></tbody></table>  
1  
  
**漏洞描述**  
  
  
  
泛微E-cology9是基于现代企业管理理论与数字化转型实践深度融合的综合性协同管理解决方案，由上海泛微网络科技股份有限公司精心打造。泛微E-cology9 /api/doc/out/more/list 接口存在SQL注入漏洞，攻击者可以通过构造恶意参数进行注入攻击，可能导致数据库敏感信息泄露，甚至进一步获取服务器系统权限。  
  
2  
  
**影响版本**  
  
  
  
   
泛微E-cology9  
  
3  
  
**测绘语法**  
  
  
  
fofa语法  
```
app="泛微-协同商务系统"
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yIciaKAicYtooSYGiaVjfnjjS01DiceHggD0b3fm5KUiajAlqYUriaQCxTxM5YFZ8jibG1dUJx2TDiaiaGjAftqicXqQXg7poAJDQKQHhuOhNEShWAM70/640?wx_fmt=png&from=appmsg "")  
  
  
4  
  
**漏洞复现**  
  
  
  
注入  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yIciaKAicYtor5gKrYCibF70WzLuaf4ETK5MnWdlYA3O62kYR9mvGjLzEjEWBe4SpVkX5ghSdBozmyT4gy9TI4njapZOelYrsBwHUz6pOxrXqw/640?wx_fmt=png&from=appmsg "")  
  
  
5  
  
**检测POC**  
  
  
  
nuclei  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yIciaKAicYtor4iayCywwNMGLBpibJPyGSAawIicj4pnBWEB7YWn5rSeG4KrmZ9JjSoXIOAb1kmEtwwS7eZCr2ictmunKjedXwD6MPeovhaSUHoia4/640?wx_fmt=png&from=appmsg "")  
  
afrog  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yIciaKAicYtopDTeZzOKMicTlMFPmGVBKUeb3BGJibYqtar1e28quU8wlLcV0jeiav1icCiaQAsCnmNHXibyWM6III5oibpByc4LceibwZO6f0wqCJWicM/640?wx_fmt=png&from=appmsg "")  
  
  
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
  
  
  
