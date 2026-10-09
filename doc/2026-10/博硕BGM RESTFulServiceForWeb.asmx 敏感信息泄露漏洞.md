#  博硕BGM RESTFulServiceForWeb.asmx 敏感信息泄露漏洞  
Superhero
                    Superhero  Nday Poc   2026-10-09 02:20  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCkYN4sZibCVo6EFo0N9b7Kib4I4N6j6Y10tynLOdgov9ibUmaNwW5yeoCbQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=8n1b48rw&tp=webp#imgIndex=0 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCkhic5lbbPcpxTLtLccZ04WhwDotW7g2b3zBgZeS5uvFH4dxf0tj0Rutw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=vx7xykg3&tp=webp#imgIndex=1 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Melo944GVOJECe5vg2C5YWgpyo1D5bCk524CiapZejYicic1Hf8LPt8qR893A3IP38J3NMmskDZjyqNkShewpibEfA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=x3261qcu&tp=webp#imgIndex=2 "")  
  
内容仅用于学习交流自查使用，由于传播、利用本公众号所提供的  
POC  
信息及  
POC对应脚本  
而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号Nday Poc及作者不为此承担任何责任，一旦造成后果请自行承担！  
  
  
**01**  
  
**漏洞概述**  
  
  
博硕BGM RESTFulServiceForWeb.asmx 存在信息泄露漏洞，未授权攻击者可利用该漏洞获取系统用户信息，可能导致进一步的攻击。  
**02******  
  
**搜索引擎**  
  
  
fofa:  
```
body="Libs/Platform.lib"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1ia7kibLCColtv6ic4fZ4DCE68IA3qjmhKCsick4sZdqzcg9a7CpnRp76zbqd9quza6vDa7PtnJBnwWyn1NBDfvaib3ic6bYu6jcxicfs/640?wx_fmt=png&from=appmsg "")  
  
  
**03******  
  
**漏洞复现**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/8o9dnzNIF1jmL4ofWFicpTSXJJ8BU7j5PvhWX3yP7DQtga7g8XCxvpAHoicr8n2ryZJgTj7vCU6NnrqiboqvBKYN7JbufnxicTpShVwhWdQRpdM/640?wx_fmt=png&from=appmsg "")  
  
  
**04**  
  
**自查工具**  
  
  
nuclei  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/8o9dnzNIF1hvmugI87eFeTBQS9gF0FgD2LKF8PC8JeGj6CjEq9kVHIfpGmXT92H8JCibFLKr3twt35rOGia1Sprmpv4HSNuCTA8HtECXtfATs/640?wx_fmt=png&from=appmsg "")  
  
afrog  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1gQSXYJEUGJSKcdyohhq0dDn8ESkhCtvxVFk3smIAfOy9dmvpyugoUpPWpbCCwAiaCDZydRICIIYakGGgUWiakQsIkVqtdycNfzg/640?wx_fmt=png&from=appmsg "")  
  
  
**05******  
  
**修复建议**  
  
  
1、关闭互联网暴露面或接口设置访问权限  
  
2、升级至安全版本  
  
  
**06******  
  
**内部圈子介绍**  
  
### 【Nday漏洞实战圈】🛠️  
  
专注公开1day/Nday漏洞复现 · 工具链适配支持  
  
✧━━━━━━━━━━━━━━━━✧  
  
🔍 **资源内容**  
  
▫️ 整合全网公开1day/Nday漏洞POC详情  
  
▫️ 适配Afrog/Nuclei检测脚本  
  
▫️ 支持内置与自定义POC目录混合扫描  
  
🔄 **更新计划**  
  
▫️ 每周新增7-10个实用POC（来源公开平台）  
  
▫️ 所有脚本经过基础测试，降低调试成本  
  
🎯 **适用场景**  
  
▫️ 企业漏洞自查 ▫️ 渗透测试 ▫️ 红蓝对抗 ▫️ 安全运维  
  
✧━━━━━━━━━━━━━━━━✧  
  
⚠️ **重要声明**  
  
▫️  
仅限合法授权测试，严禁违规使用  
  
**▫️虚拟资源服务，购买后不接受任何形式退款**  
  
▫️  
付款  
前请评估需求，慎重考虑  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1hILCW3mHNlSNdxTMVqsucEtoRnEVB3aGeZlvJmvuwTqtjIjJJHtO2udLejdEJxzicHNWibP6ibWu7hADCZbFjTGvsk4rBGHlpPqE/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
