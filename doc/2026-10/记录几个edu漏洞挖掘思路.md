#  记录几个edu漏洞挖掘思路  
小轩
                    小轩  C4安全   2026-10-04 01:54  
  
#### 0x01前言  
  
原作者：  
响应云sec小轩，今天分享的是edu漏洞挖掘思路。  
#### 信息收集  
  
很多师傅都抱着一颗出漏洞决心，但是发现一圈下来没有毛都没有发现，这个时候自己就开始怀疑自己。师傅们有没有想过不是自己没有能力或许是没有找对应脆弱资产？？？```
比如你想搜索充值入口 你可以在微信搜索 职业学院充值/缴费 （微信小程序的最多了）或者 xxx技术学院
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeJnzMLypz1tw6RDMpB1ibj42GUQ32gzRTRBxLibCgdGX27GBJkPbUDIvFs4LHgx4VQJYHqibUrolYvfAqFUZceqRGZVAqib0SccpxE/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
### 文件上传  
  
发现该证书站点小程序存在文件上传接口，我就测试了一下  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/R9d0DzJpTeLs0aPo1xrfUibOn64WF2iawXxxe7h9p9ribl4ic09TicMcDSxibfeywJ4SHYdqicOonKYpDY7g0kXich4G1LqYauxIBN31ojChKhSY05w/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
文件上传入口  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/R9d0DzJpTeJ9jRC0nSnFnzzXaRibEkRRAlIiccWo2iaXA0WxaE3wzjwzRJVicnI7L4QWFpG2YqdMNBDd3vJsHbvVntNZDkL4Rs8ciadbsQXUcEdg/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
构造上传文件  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeJlY4GiboJfR9IMtESVicNvO8xcPLjjvpJtF7Mu9T0m0GKjJqU1kshFhrDUeqhXJbjvv08ckC4t7xGkvTDmKupAehjppC1LAib62Q/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
回显上传绝对路径 拼接路径访问  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeKo7ue8PryKasuTJtWc7Tbia8D4CAUF6RzpPPX3mcZJfgRLB34thgyORj9fVZeOJXLQichYANTUNWF9o5ZhCrUQNHbUhXNmicxiar4/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
  
发现任意上传文件，可惜的是网站不是php，jsp，aspx的都可以上传但不执行，后来就上传了存储型的xss，没有拿下GETshell还是挺可惜的。  
### 未授权访问  
  
话说回来这种漏洞就有点意思了哈，可以不需要管理员权限就可以   
增 改 查  
,但是需要构造数据包。关于这个漏洞是怎么来的呢（通过反编译小程序提取敏感信息路径出来的）  
工具推荐:https://github.com/Mstce/Onyx  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeLUAHvljS0aezYZOZBlxLLvMib7p6mK7GehiaD3LtJlOvWXqzYuSc3mFng2EOIn06ebiaQyia26r3ibBUiaUVUkoibWFBsBd8FLz3micPw/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
  
增加用户的数据需要自己构造（后面我会说这个构造是泄露出来）  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/R9d0DzJpTeKajsduNibVz5jPjy4OR5SkMLlVic8BtX4muj8qm2Im0RLtWo00YrIBafhUC1p8GpHNMFzOoBAtgfpcfWU5eIrwrsxthG248v9Y4/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
这个呢是修改用户信息的，但是不能把用户改为管理员权限。 主要是可以修改用户的基本信息（名字 学校 手机号 性别 专业等） 这里我就用 谢志强的信息修改学校 改为91xueyuan，提交数据包，发现也是返回修改成功  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeIaNXpBOVvg2ia7Nu9woI5WicZYGmbrSCib9AcAagyb7r8CP72gGqpyfIA6nK86QCbCFpVEsqcjsMXMcbZ12g5J5QVzjuIMo2SibsI/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
这个是查询接口可以通过（手机号 id  state:"5"）来查询。师傅们还记得前面我说的那个构造标准参数怎么来的吗就是提交了 'state' 这个返回 志强的信息去构造的 也是水洞一个，其他的接口也没有什么好玩的了  
### 支付逻辑漏洞  
  
某大学财务系统的费用缴纳网站，老演员了开局一个登陆框（首先测试了sql注入无果后还是老实的注册完测试里面的功能吧） 注册成功后 发现功能点还是比较少的  
《功能点》 在测试过程中就去点击了，小额缴费发现有一个叫   
汽车检测与维修技术服务师（高级）  
的缴费项目（要100蚊），用burp记录了相关的流量包，果然发现了东西  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeLMdofiaVJGCDQXiaFnhic34Ed8LvDvOV9ccdOZoGF0uNicicu0uLziaPqE3unVEOdcuGsicOxFKhibQxTd4LnSHkSiaAQM5y0gL3W3u8R0/640?wx_fmt=png&from=appmsg&watermark=1 "")  
```
usercode=61032419xxxxxxxxxx&zje=0.01&banktype=abc&clientType=wx&businessId=1493990161191665664//usercode=用户账户也是身份证//zje=0.01 //这个是需要缴纳的金额 我修改成了0.01返回结果{"errorMessage":"操作成功","traceId":"1497959654137921536","success":true,"showType":0,"data":{"data":"https://xxxx.xxxx.com/mpay/mobileBank/zh_CN/EBusinessModule/BarcodeH5Act.aspx?token=17772118150139582323","orderid":"ahzypt2026042600074","rtnMsg":"操作成功","rtnCode":0,"status":0},"errorCode":"0","host":"172.17.255.133"}返回成功了，发现有一个url心想这个肯定是支付接口把他复制到了浏览器（他说需要到微信），打开了微信点了进去需要支付0.01（果然和我前面修改的一样，但是系统会不会有订单还要支付后回去看看）
```  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/R9d0DzJpTeJLsIoetlGjk67NOMAOIMwApozzGncwgX4r1nyn7oibFaV7NbA81HdWKUW6LsSSabEIiaZV3qChlI0yNichx5w5otcxNNszyNHFx0/640?wx_fmt=jpeg&from=appmsg&watermark=1 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/R9d0DzJpTeIE16UMzA78UxnYwpCw2NdPAAfRhs9f2OXuRXRG8csc9Kp3qePtXWRTF5M6R0kjOPNyWrFrqMWXRZvRHPSLbNm9U1KHuPtgmm4/640?wx_fmt=png&from=appmsg&watermark=1 "")  
  
很给力，系统返回了支付成功的订单！！！  
  
  
END  
  
  
