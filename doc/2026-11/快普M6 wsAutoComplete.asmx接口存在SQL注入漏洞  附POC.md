#  快普M6 wsAutoComplete.asmx接口存在SQL注入漏洞  附POC  
2026-11-21更新
                    2026-11-21更新  南风漏洞复现文库   2026-11-21 11:52  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 快普M6简介  
  
微信公众号搜索：南风漏洞复现文库  
该文章 南风漏洞复现文库 公众号首发  
  
快普M6  
## 2.漏洞描述  
  
快普M6 wsAutoComplete.asmx接口存在SQL注入漏洞  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
快普M6  
![快普M6 wsAutoComplete.asmx接口存在SQL注入漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/b9KQYsB8q6wZuGvYiacKLq9jBb2rsTS35DMvZPrGqeCNjFYVhYWkVngqjxw4AeYP6b6JOia1c933DP4icfKlmrGscpYCL3rnnYwu102K5ch32o/640?wx_fmt=png&from=appmsg "null")  
  
快普M6 wsAutoComplete.asmx接口存在SQL注入漏洞  
## 4.fofa查询语句  
  
body="Resource/JavaScript/jKPM6.DateTime.js"  
## 5.漏洞复现  
  
漏洞链接：http://xx.xx.xx.xx/WebService/wsAutoComplete.asmx  
  
漏洞数据包：  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yJY1ZGRScCo8Sa5g9jf5zuatAJWCkHervFvbia9XoJK9ae6VI1T2U7WNMsQ668NxYOuGn14koWlUqP5wjt0Dvd9arqXVdzwcRo/640?wx_fmt=jpeg&from=appmsg "null")  
  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
2: 免登录，免费fofa查询。   
3: 更新其他实用网络安全工具项目。  
4: 免费指纹识别，持续更新指纹库。  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6yuIrUMNsS0RngGVveXz2W1jk9XHQun8WXr5KZXP7QaYMJ4slmKk06pk3iaHUK3L0euOGJXaPqiaHeYRugVC4495UFx3EiabEC2Ns/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6wOrrJ01YJogxEIPhsibJDLNLnpZpicAmDGR2bnLgXN9wCsqeGkiaXPQTibd6Wf4XseIZ7vS3s3vibMP4B45iacHNQuWU9oSVWjSz4vE/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6yf85FACWSbobOrpr2AnMu4Dog478TOzlYhte8fUmERicekPkbtmfnDYgWUA1tj9ZQ7HKYFM6ic9XgkPEUhOcOwic2xFBKxy9kEWQ/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6zqKn3ztiaP5abcsvwYFAN94nRKuIPqypjib31xLuRUm3EAdO2Uqu9GnkYicib3UYnzCua08iaka2EPsDr1qJgBSSldKwc9oJJvksF4/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6xS25WmqlTLauPvXo2Htk2lEzu9Rhj0N2V44Be2UoA3icVfBjogEOXto6I2qicIc3FTKnMV05QVgugMQEJ8pKfCTjWLnDWtO5ianE/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6xfAVIiaqkSORQ2ldF6icZ1KF8cgQ6jO37icvCgxCq6KS1yKzDPK7OTmMMv6PUhAibZHCKmXIsw9D3PA5IaQcY0CEfa4Pu3ma7k4OA/640?wx_fmt=jpeg&from=appmsg "null")  
  
## 7.整改意见  
  
打补丁  
## 8.往期回顾  
  
  
   
  
  
[用友NC doApply接口存在SQL注入漏洞  附POC](https://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247491041&idx=1&sn=100c143c84cf39ef60ed25544f8b254c&scene=21#wechat_redirect)  
  
  
