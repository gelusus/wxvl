#  大蚂蚁即时通信系统 loginByToken 身份认证绕过漏洞  
Superhero
                    Superhero  Nday Poc   2026-10-27 08:02  
  
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
  
  
大蚂蚁即时通信系统 loginByToken 接口存在身份认证绕过漏洞，未授权的攻击者可利用该漏洞造成信息泄露，使系统处于极不安全状态。  
**02******  
  
**搜索引擎**  
  
  
fofa:  
```
product="即时通讯系统"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1gklKzee6OKicqG22WZtia08NLtkU9Sq194YR4rzG542dvM2WdzXxZrZTia82RN5FEScltuCWj0fHjdvLgIb6LpScehVGwibG97Vew/640?wx_fmt=png&from=appmsg "")  
  
  
**03******  
  
**漏洞复现**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1juwx3GmfBIMxOWAd4UrsSdjXJwAmeqobEx1niaFxFhkYxlH4grSEkRvs6uoI2v7glHrW32M6O3BMGU3goAGR6AhutxmpFia1Yek/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1grxeQEibjIguTxUCScRyiaNQdQenVF5tlbicDjlMn6g24rkFCMXVBic9uVSYME1iczNTXJAqavrzUNViab2gRSPNCnhwVAlE3a1P3ib8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/8o9dnzNIF1jar0SUbM2JJ2Pf0UuMibpRwYDiaPv0NPVUXJ5eVwMc54KJBln3KfTUk7P4fDyk6Vju8M5iatmFWIK56SnbQ0fYXQEaLmeiaAjmntM/640?wx_fmt=png&from=appmsg "")  
  
  
**04**  
  
**自查工具**  
  
  
nuclei  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1jsI1hyj6BcvwCKFREaVLY67icnZ5nz5GPp4Khnn3e2Sianl79Wvh668ibXYVDjPHPuibAQVNG1gnL9pQRe0t2HzvVGtQMbV94hWpo/640?wx_fmt=png&from=appmsg "")  
  
afrog  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8o9dnzNIF1iaSWBm1GznMgTk4I9sF8bUNljXHVOjBkmKsNmcWQiaBfE4ag00aQ6Kjyt1vHyowJFrobGz6uqMxAn4VNKQibo8yysxCLXh4iaWBWU/640?wx_fmt=png&from=appmsg "")  
  
  
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
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/8o9dnzNIF1hQEJ0IE80HT23ZV8N1caFSickeY6Q2M3zw0nJibeDicDDdjgcprb9YzccQN6gVEFkqaAb14v3IpLlKUdacYbpSP5Rollu83vHflc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
