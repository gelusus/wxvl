#  PHP代码审计-PHPCMS 前台RCE漏洞  
原创 wolfsec
                    wolfsec  风铃Sec   2026-10-31 12:25  
  
如果你也刚好在学习代码审计，不妨下载如下源码跟着文章一起跟一下漏洞调用链，源码打包如下：  
```
```  
  
系统部署、路由分析部分请看笔上一篇文章《PHP代码审计-PHPCMS-SQL注入》。  
  
来到注册界面，具体URL为：  
```
```  
  
输入对应的信息，完成注册流程，抓包，得到如下数据包，将数据包发送至Burpsuite的重放模块：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZltM5cnAnAoeaddLXyVhtib2ct7NpxIaSkAObVjzwIjw0ibtM1KNJj5XOLTnDychc1ia2ViaPb0ibIxg5IO7t56y6pPkue5fWy9TavTQ/640?wx_fmt=png&from=appmsg "")  
  
打开phpstorm开始，根据注册界面的URL定位到对应的代码片段。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZltTLVLFSFmotSELvl0icflj5DO4MWKejRxrmFicvtoUCDf0nBPnOsx51Ric0ibVJWcEKiabAuvEdQ99WC2ic7sXc96592gkdZibMstexQ/640?wx_fmt=png&from=appmsg "")  
  
下断点，Burp重放模块发包，开始动态调试。  
  
因为该功能点存在异步请求，所以必须要获得整体的注册数据包后再重放该数据，否则执行的控制流会出现偏差。  
###### 注：php代码动态调试具体配置可参考笔者PHP代码审计第一篇文章《PHP代码审计-动态调试配置（附环境）》，虽然使用的PHP版本可能不一样，但是过程都是一样的。  
  
紧接着就可以开始进行动态调试，一路按  
Step over  
，来到核心代码：  
```
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZltxw3JxY3dEUzxheF9ialUgKQ9ORNKxL2PPnfSDKicwcUWymXHsvdobzsialfahQEIMrAfudtIJwcV2y9cbS9BoOjbJnmJy3lX1tU/640?wx_fmt=png&from=appmsg "")  
  
跟进get()函数：  
```
```  
  
在迭代最后有一个函数动态调用：  
```
```  
  
$func  
最后的值为  
datetime  
:  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZlvTKvUOBEjBCK5tpN8rU0khiafQNEWSoMYFaxic15btWqDrM24WsumOMRVpLwpe7sHZWDW6zxsp0fGmrtmrN83P0OpNMPDQCxHlo/640?wx_fmt=png&from=appmsg "")  
  
这个  
datetime  
函数是从  
fields  
数组中提取出来的，此时我们可以先确定一下  
fields  
中的数据是从哪里来的，重点是要确认除了  
datetime  
之外，还有哪些函数是可以被取到的。在当前文件中搜索  
fields  
，得到如下代码（  
member_input.class.php  
的构造函数）  
```
```  
  
关注  
fields  
，它不是在   
member_input  
 构造函数中现场查询数据库得到的，而是通过   
getcache()  
 从缓存系统中读取的。缓存数据可能来源于该数据库表，但需要找到生成缓存的代码才能确认两者的关系。这个时候可以先找一下缓存是在哪里写入的，在源码中搜索：  
```
```  
  
找到五段代码：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZlsxQBBXlrZcQdgQEQibHdExPqvQRYKoHia00Te9X2ibIpmeItk9lfuF0R3jZgzX21Jib7XOrF0OVLKHBgpC5KdDAaIekcnXoLPEjxY/640?wx_fmt=png&from=appmsg "")  
  
根据这五段代码分析，基本可以断定  
model_field  
 数据库表中的字段配置会被读取出来并整理成数组，再写入   
model_field_模型ID  
 对应的缓存中。整个调用链：  
```
```  
  
现在进入  
model_field  
 查看  
formtype  
字段：  
```
```  
  
得到如下函数：  
```
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZlvYXKFuWMMGMdnVBK9wzF86ZMUGOosNCVj1ibE9T4Yd824iciaYv1ibVYSJLJiaic8pRZrYz8j248XfssVbhKnrY5lsiciafMP5ERb54xk/640?wx_fmt=png&from=appmsg "")  
  
对应的modelid应为1，3，11。  
  
接着通过上述函数名，在php代码中搜索函数代码具体实现，这边聚焦editor函数：  
```
```  
  
**注意：动态调试要进入这个函数有一个坑点，要想让控制流进入editor，仅仅讲modelid修改为1、3或者11是不够的。假设你将 $modelid 修改为 1，那么程序读取的是：**  
```
```  
  
而在   
get()  
 中遇到的判断是：  
```
```  
  
**这意味着：当前提交的字段名 $field 必须存在于所选模型的字段配置数组中，否则就会报错。**  
所以这个时候要想进入控制流就得先去  
'model_field_(1,3,11).cache'  
或者数据库中查看查看对应  
field  
字段。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZlvdc1TicBpRyqqrKEDOVTCc3UhqlbiaibW5kuiblbT4iaYvQXh0BAchfXFYrpicG8vEHNTk40xTsibAtzyR97XFFB4uavibwChabUl564g/640?wx_fmt=png&from=appmsg "")  
  
modelid  
为1时，field需要为  
content  
，这个时候才能够进入  
editor  
函数，所以构造如下数据包：  
```
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZlvBeVFadgwO0eKcfbl1saDYVGibygq9KFRqc8SUhz7EnQeSOPZDVFmDeJWdFYkwZBUSgticxgiaBLnYr1Lwm4Ilazq9vbY5c6L210/640?wx_fmt=png&from=appmsg "")  
  
   
editor()  
 函数主要用于处理编辑器提交的内容，并调用附件组件的   
download()  
 方法处理内容中的资源。这边继续追踪  
download()  
的具体实现。  
```
```  
  
到这里我们就可以进行文件上传了，因为download函数是从 $string 中查找符合条件的 href 或 src 属性后提取URL后进行文件下载，所以这个时候可以构造如下数据包：  
```
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZlvSZDLQeGFbL9cp3e2V7FNPFebMuuwpBbaQjebQNdiaOu14Nfu0zmAUotibqBcv0GUX4GtB7ZVicPygKTdibIESpEfym3eBl8FRSQ8/640?wx_fmt=png&from=appmsg "")  
  
成功下载图片文件。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZltZZjiczOXgxE7231Bkq91QcQRbjtpMQ7h2icv02piaN8kjB1yfsNrHcC0U3wRAhpekHibLvkBjjibgkpUCezMNc1BdfZd5KKow80Xc/640?wx_fmt=png&from=appmsg "")  
  
文件可以正常上传之后，下一步就要摸索如何上传php文件了。上传文件的类型是根据  
download  
函数中的正则表达式来过滤的：  
```
```  
  
该正则只允许匹配  
$ext  
中定义的这几个类型。再次审计download()函数后发现在文件下载之前，调用  
fillurl  
又对下载地址进行进一步处理。  
```
```  
  
跟进  
fillurl  
函数：  
```
```  
  
这个时候可以尝试设置如下地址：  
```
```  
  
此时该地址进入  
fillurl  
之后就会将  
#  
后面的内容截断掉  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZlvdgwXop7XQjpYZ9Vydp8YjUUPbN1uGxQInQM5mmNKgRjUbK9EduMhjCLZqGS26vQ08SJ5EgibWyibYOD7t1CERIy01ynxKOCET8/640?wx_fmt=png&from=appmsg "")  
  
完整数据包如下：  
```
```  
  
持续步进，发现最后保存的文件为php文件。  
**别忘记将exploit server中的文件重命名为xxxx.php**  
，下图标红内容为download函数下载地址：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZls6pUpchbia0cv5UczOnzDibUOXBJcyoxTytFHOTAwbZZhFbBiaNCm5v2R1WKS1lgpvGUcjIxR7kicVicBK5B91vKSzSV1ib2tsjK7sk/640?wx_fmt=png&from=appmsg "")  
  
把剩下的代码走完，最后WebShell被download函数下载到服务器中。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZluiaxA1fMTKf17ecAF2KAuicle0WOs0SiazL2cvr3u3lMicRQpxlaBEUsL8j2VuGaYSOcAETYkNjkP0hMMFQruIguQOHvibqSpNe4Rk/640?wx_fmt=png&from=appmsg "")  
  
走到这里我们还需要解决一个问题，就是怎么确认Webshell的连接地址？这会儿就需要回到  
register  
方法中继续走。  
```
```  
  
核心作用是  
**在开启 PHPSso 单点登录时，将新用户同步注册到 PHPSso 用户中心，然后再保存到本地会员表**  
。可以看到最后插入会员模型数据得这段代码：  
```
```  
  
我们在注册页面提交得注册信息会被插入数据库中，但是由于请求包已经增加了如下数据：  
```
```  
  
这个时候在进行数据插入的时候会报错，从而将完整地址爆出来：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bhUibV7MPZlseuS79icwHLDic9cjRErOE8qvFT0COdKG3Jb0juDWFug5HU0NbSVYtpeYnL8hgjt1Rt9WxcbP9IXGdBMJR4HxDnIicjDCZ1Z7tia8/640?wx_fmt=png&from=appmsg "")  
  
最后成功RCE：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bhUibV7MPZlusNbm3ADOEHQYy2pDU81MckqUst9pKeia7JyOXib07w2I4g8KMTNJ3Oy1r6zH5TZpRTo3kjibx3ict7jKHz8SKoqAF9cDMpUSXgbI/640?wx_fmt=png&from=appmsg "")  
  
  
  
