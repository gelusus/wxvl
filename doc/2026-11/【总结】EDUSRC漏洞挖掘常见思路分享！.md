#  【总结】EDUSRC漏洞挖掘常见思路分享！  
SKYY
                    SKYY  神农Sec   2026-11-06 01:00  
  
  课程培训  
  
  扫码咨询  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b7iaH1LtiaKWXLicr9MthUBGib1nvDibDT4r6iaK4cQvn56iako5nUwJ9MGiaXFdhNMurGdFLqbD9Rs3QxGrHTAsWKmc1w/640?wx_fmt=jpeg&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png&wxfrom=13&wx_lazy=1&wx_co=1&tp=wxpic "")  
  
  
#   
  
专注于SRC漏洞挖掘、红蓝对抗、渗透测试、代码审计JS逆向，CNVD和EDUSRC漏洞挖掘，以及工具分享、前沿信息分享、POC、EXP分享。不定期分享各种好玩的项目及好用的工具，欢迎关注。加内部圈子，文末有彩蛋（课程培训限时优惠）。  
#   
  
文章作者：  
SKYY  
  
文章来源：  
https://xz.aliyun.com/news/91859  
  
01  
  
0x1 【总结】EDUSRC漏洞挖掘常见思路  
  
  
初入 src，大部分新手都会选择教育 src 去作为入门，随着网络安全的兴起，各大高校对网络安全的认知逐渐增强，核心资产基本都存在 waf 保护，也进一步增加了渗透的难度。相对薄弱的站点就比较难找了，这里聚焦 wx 小程序->服务号等 进行收集。  
## 支付逻辑漏洞  
  
支付逻辑漏洞是指在系统的支付流程中存在的业务逻辑层面的漏洞，这类漏洞通常由于服务器端未对客户端请求数据中的金额、数量等敏感信息进行有效验证而产生。  
  
edu站点比较脆弱的支付系统一般就是水电缴费充值之类的小程序、web端。这里涉及到的基本都是四舍五入逻辑漏洞。  
  
支付系统通常将金额精确到分（如0.00格式），但在某些情况下，系统可能会对小数点后的金额进行四舍五入处理。例如，充值0.019元时，付款方会自动识别到分段截止，但服务方则会通过四舍五入将0.019进位为0.2。相当于我们将0.1变成了0.2元，重复这样操作损失也是很大的。  
### 案例1：某高校微后勤电费充值  
  
点击充值按钮  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXic5MyG6X3hsEGGkkcjqibawuJFkuDHdfY6blucx3MzN2DyYgjt6f4n0FP3oZJ7QR5KOaib6XPEHCBn68WXEfEVctwpZzBV4JWDs/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUpI2icFqjib5WLldLibPzHz9sSXfyMzsUaahaCicUXVXBNs3iajibDA9fTsoy7325Sr6ysn42ZEW9xuOWYAILnDIRialeXeLnvcCwGxY/640?wx_fmt=png&from=appmsg "")  
  
这里最少是充值1元  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXWBbibDALtWSKiceicmImsz8USXThYsRXOibzFiaAsONkQFpicMrGickVvg3Zpib9H05XYrvK1VoVk0cRe6Dc3SgrjLP2JhARyoglZogA/640?wx_fmt=png&from=appmsg "")  
  
选择一元点击充值进行抓包  
  
如下数据包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWfvPnMgibPlW8ttfw4ulgE8M6ibjic0VksbZtLg62PnQSiaWpKs3K97PGwQcb4JqZ4tyecosxEkDOjlGMqEqsNmtnicZXxicgbnyJgs/640?wx_fmt=png&from=appmsg "")  
  
我们修改money字段为0.019  
  
成功响应，生成了一个支付链接  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUvoN5vzdiaqpqDic9V6nO1PicAXGsMN0IO135QY2VGEMaAXV6MIxcdvwjqh2NKnvrGrPbanOf1zNcgVZcpAwBbYq0Cfate0eCSibs/640?wx_fmt=png&from=appmsg "")  
  
此时我们发现该链接显示的金额是0.01  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVjekGjRia99uCmHodOFadNOCr4oX30xNespa8MjicuBeTuaibnV6zebdDl5XO7g6ia2l46DiaggvPhYEkiba1pCu53x7RG7BewZRnibE/640?wx_fmt=png&from=appmsg "")  
  
付款之后系统端提示充值成功0.02  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXhicTwCz9gMjjGLXVARH626OwnvdascicQvsRYEvX34uPUwMl8pjmg7b839Cia3BJ5ibcgeJqNvACpK6GbjvAoIEk42HHqibcUg7W0/640?wx_fmt=png&from=appmsg "")  
### 案例2 某高校汽车充电桩  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXCulmpdxvfT3IbsnGsNH63UrWMXgiaM6x6jQLQvPOwItUibmcVtqwJm7AiaqDhaYwEf3wtwf0msulMHVxYhOhicAgMKXiczvOEaohM/640?wx_fmt=png&from=appmsg "")  
  
充值的时候可以自定义额度  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QV04Pr6QU1aG50oGTs5mmXqLY675I9HXico64ZdI7au4GSQTTO4dDx7ljOWp6rQUQCX9srMCE5TPKib4C83vt6WqgrzILfOYVggY/640?wx_fmt=png&from=appmsg "")  
  
这里直接写了个0.019，根本没有校验金额的合法性  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QX6y96UI2J6sT0207iaAvBf9HP6Qf6NQ622CPVXEbqaUzpfWQUunMqfbjef9VELpwyYia5UgibT9YbCJSVYQYFGfkPgInTW78yK8k/640?wx_fmt=png&from=appmsg "")  
  
这里也是直接就成功支付验证漏洞存在了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWMe6biaopluWZnlibGKv9L6UxYxjZKyMhKLj78ZwLLl3icwxYEeURibiaUgVtibia4wRTVslnGobveS4bUQjsyMAUUiaYFtVtofjM7VD8/640?wx_fmt=png&from=appmsg "")  
## 水平越权漏洞  
  
水平越权（HorizontalPrivilegeEscalation）发生在用户或攻击者能够访问与自己同一权限级别的其他用户数据或功能时，但这些数据或功能并不属于自己。简单来说，就是同级用户之间的越权访问。例如，一个普通用户A通过修改请求中的用户ID或URL参数，访问到普通用户B的账户信息或操作其资源，这就构成了水平越权。  
### 案例1：报修单撤回接口存在水平越权可撤回任意用户的报修申请  
  
两个账号进行验证  
  
账号1提交报修申请  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXZ74w4PO8284WT8Pup9eXPZDuwjfllg4Um9nr2ZMGs8Vic8UpGvlxdgLc8qCYVhjnjmBicKEXusuF56DvhFYPicWwQ6pyPibJU4Uc/640?wx_fmt=png&from=appmsg "")  
  
抓取流量包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QUkH7YnV1SoNJOPfH8RBWSSGiaeSQjSJU1YgOL38AbZopgEnrUBA2tNib1ZibicAchn3fhwCb7SFTP3sIYYm0viaEb16MBWurAp87nI/640?wx_fmt=png&from=appmsg "")  
  
wxbx_data_id是报修单的id：31358  
  
切换账号2，同样申请保修单，这里的wxbx_data_id：31359  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUHS5tV8I0FGAT97uGmpy1PVsd3U7ZPyqCtJITsZJMXuyHiaibKzzibsA7T3ZoOC5baiaPicJw7ZzyI92JOAYtxUBOIvWibMKBFeje1A/640?wx_fmt=png&from=appmsg "")  
  
也就是说保修单的id是连续性的，那么就很容易存在越权了  
  
使用账号2撤回该账号申请的报修  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUqiavXp2j1VYEW7QRIbiabV9UB1WlicjneqwYSiaibaoeA13c7Ub6syZ1Om03Rmv5VAMbiaFwsOxibTWb1P4UfYpKeWXQorvy9W53wmY/640?wx_fmt=png&from=appmsg "")  
  
然后将该流量包中的id修改为账号1的报修单id：31358  
  
发包，回显撤回成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUvYYy7dJKBHIze7OO2Yh4EPLMAbA2GhUzEtkTAIlc1GEUNDQ5ZMpnaiaf33P9n4GB9ZV4uthqnxoO1Q63h9accnVF6ojLWy0ak/640?wx_fmt=png&from=appmsg "")  
  
此时账号1已经收到通知撤回报修单成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVxlZmXEEBu9cxGMcAHaNFIxWQQxvEIrjiaYLrfEMUwCA5HyPbheiaS8lMs9ibcLdUfUc93Et9ZbAzdFSiaZJDV6go2EYY9dgTias2M/640?wx_fmt=png&from=appmsg "")  
### 案例2：某场馆预约系统存在用户id遍历导致信息泄露  
  
这里我们准备两个账号，分别添加预约人信息  
  
这里第一个账号添加的人员信息id为643033  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QX9YIVfQ6qRbWKheFPMLjTZA7Xz7EbVOWuOIUGnf7JNFXaPSGFl82V8JHmvDoBqzVaWgURibDyEKzLIAn0s6HC1VUKrNedpXzZU/640?wx_fmt=png&from=appmsg "")  
  
接下来我们用另一个账号进行预约  
  
该账号我们添加的人员Id为643034  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QUc9eYBvNXibYKQIN2tianwDFzJyIWD7lHcMudpEZh9vL6hxb58SibsriaBqRHwVw7mlJWSytbhFn6DapVp8WXbY4wzeh3libPOQGLg/640?wx_fmt=png&from=appmsg "")  
  
我们随便预约一个场馆，人员选择我们账号填写的人员信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUicxKt148drNBq2wwE6ib8iaT3AYtD76tDbVlotMjGMjb9davcRibSmSQ8rmDAHjksXwKOYI2LXaibZAwYfwPib6WNQIyNQWwgqsScM/640?wx_fmt=png&from=appmsg "")  
  
接下来我们只需要修改id号为我们上一个账号的id，然后修改日期就可以对其他账号的人员进行预约，并且该人员的信息并不在当前账号存储  
  
可以发现已经成功预约，并且获得该人员的身份信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWDyHZ5bVky4EPKuXq3NfmNQtyO0aA3NFr1f3xYa3EftHqO2ibuhqn6I6PGO0HMiaMtGCYe5h52b4XOh3Eptkxj1PnhdlFbYkRqY/640?wx_fmt=png&from=appmsg "")  
  
那么我们只需要重复上述操作即可将系统中存储的个人信息全部泄露出来  
## 云安全-OSS存储桶漏洞  
  
对象存储服务（Object Storage Service，简称 OSS），以 HTTP RESTful API 的形式对外提供服务，是阿里云提供的海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。  
  
Bucket：用户用来管理所存储Object的储物空间。  
  
Object：OSS存储数据的基本单元。  
  
Key：当存储文件（Object）时，需要指定此Object的名称（Key），后续您将通过这个Key来获取该Object的内容。 Key也可以用来模拟文件夹的一些属性。  
  
Data：存储的数据本体。  
  
oss常见的路由minio  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVblh8nNoR9TWByibUuMAibT2LsfcFI0O6jfu7OFficdop21tuTwojwXG7sNJqoykHdeDONRvLlyyCwraic2Le5E5kaSqQevYVTJck/640?wx_fmt=png&from=appmsg "")  
  
code:nosuchkey  
  
这种情况十有八九是oss权限设置为公开可读写了。  
  
尝试put上传文件、delete删除文件等操作  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVq57RgoBF5QZalyibKpcALia5dtVF9RBjkcZbCesDLbzYNfFKLJfNNYDLEgAGj1KTzSwKmxLexibEXeZIwpYEc84cKuEGnoAbRicU/640?wx_fmt=png&from=appmsg "")  
  
204即为成功删除的相应码  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QW3swHOOlUdZ0oQCEYRrkSWTbQW9fv3B8iaDwtFaVuibtTvWnajVCVKQqiaYxMet1zHuq0UZ3Sibn7YI65YdNxswW5nL5Jtjj7m9QQ/640?wx_fmt=png&from=appmsg "")  
  
如果能够成功上传文件的话，我们还可以尝试去刷一波xss  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVB58bLicb7gOAsZHxSRoqqZ84VyWfr6STyk37BJYUFnMFY1dAPgBA4JMPwA8HRDhnfnux6OEeFdL0htm5BmST6EhfImsKPHPH8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWkXEedZFibYGXQdDfLQOyovB1T0vJnjb5gdqwbHoNSoyUjycABAZxg1fUJUHLOxXYFX20xFhHjFUoOnhQkWXRhwpibuANHbssFI/640?wx_fmt=png&from=appmsg "")  
  
有时直接访问无法触发弹窗，是上传的时候没指定返回头中的content-type，我们可以在请求地址上增加response-content-type=text/html让浏览器将返回内容当html解析。  
  
这个漏洞估计有90%的站都存在，但是很多站点用的都是公司的oss站点，很容易出现归属存疑。  
  
有的站点只能上传、覆盖不能删除或者能删除不能上传的，总之权限设置的都很离奇。  
  
oss存储桶还存在ak/sk泄露导致oss存储桶被接管的漏洞  
  
"**AK/SK**" 是对象存储服务（OSS）中用于**身份认证和访问控制**的核心安全凭证，全称是 **Access Key / Secret Key**（访问密钥/秘密密钥）。  
  
如果系统泄露了这两个key我们就可以完全接管oss服务器  
  
阿里云官方工具  
  
https://github.com/mrknow001/aliyun-accesskey-Tools  
## 敏感信息泄露  
  
敏感信息泄露是指应用程序由于配置不当、代码缺陷或逻辑错误，导致本不应公开的数据被攻击者获取。是SRC漏洞挖掘中最常见且危害面广的漏洞类型之一。  
### 案例1：报修系统配置不当泄露全体维修成员登录凭证  
  
属于是抓包就有  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXw4ffPvoDBodLstI3LUicC2FwtjlM53UbSTxUbpp3PzxTXddo590pfLT9eGzc1wLGsodj4VVN9rcwhM54K7ScDUVHPrpPVKlAc/640?wx_fmt=png&from=appmsg "")  
  
这里GetAllWorkers接口直接把所有信息全部泄露了，小公司提供的服务真的漏洞百出  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWzLKPsGvSWfVLF8aVzJ4ibTFgwvLVauSHNnqRQJo4njQZnhHudTBorP1sLfRibh4bwy0VmicVwmJv3DSwxNggHY3BJcibyfbNrh5o/640?wx_fmt=png&from=appmsg "")  
  
这里也是直接利用泄露的凭证即可绑定维修账号，获取全部信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWIJtqKcLYwQLb5GicnicBlm3fVWITicXwIV5cSmPht9mlACTqWM3c2Rb8icajlHDJMAAibptePzVHva1Vg8EERibiar9dQ3WygUA21Iw/640?wx_fmt=png&from=appmsg "")  
### 案例2：校友系统认证配置不当导致敏感信息泄露  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXOoWyjcr8JqVn63iaxwfegozCV7OLpOSCnjSd7lzEQWfVpuWk4D6iaNruw0qRyicRDsJASQoj7Laic9TLJ1nowLUU1mX1s6UmooNQ/640?wx_fmt=png&from=appmsg "")  
  
这里我们利用谷歌语法进行信息收集  
  
site:xxx.edu.cn 姓名  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWevsgPmET16AzEnuPvSmBW5ibtnT3iceKiaG8tZpoGaOKtU0wCUZ0cpticVK91VAQcDT0r5EyWzxXmJf6xlibVR1jeibX2YcYKo6ZwY/640?wx_fmt=png&from=appmsg "")  
  
这里将姓名填写进去其他信息任意填写尝试，选择认证信息  
  
抓包发现把用户的所有信息全部泄露出来了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUe6tibx1rg9vPFHPTdicRnvFpsW5zcqibmxdWbc5wbeYjNycQSqMicNaEWwMgpOGGWTo7mmsnc7PU0UrOu3BUTwtvzTmjPmzvMeWw/640?wx_fmt=png&from=appmsg "")  
  
这里尝试直接替换该流量包的 xm 字段看看能不能遍历  
  
这里提示需要人工审核，前端回显该用户已经被绑定过了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QU2pMtIFhOHDnrmoS0o8W9CMgRvmBjAU4w5meTPJpyyTrMjJibn4k0Haib5sO3v2dLecF86rjravc8IIFwAbicZpxZicbSMEoK9OxE/640?wx_fmt=png&from=appmsg "")  
  
那就是可以遍历了。只需要学生没有绑定该小程序即可  
  
尝试其他姓名，提示我们验证码失效了，发现短信上写的有效期是 2 分钟  
  
所以我们需要重新获取验证码这里直接将验证码包重新发包即可  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QX4yiaxcibFZXhSubF7R12s2icyMGxmC6l0via1BiaDPQLwUJB05ev2ZosHCxRUN3NpmDSXs6ibaibTU17ibNMicZOdkJ2niawtFe9vqREzE/640?wx_fmt=png&from=appmsg "")  
  
验证成功，只需获取用户姓名即可泄露出所有敏感信息数据。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVZfic0yNvPKPicJ7z21JJAXZ9QrJDXsENNWZNTdq2WuxQBofoZmqUEfWfO93RuqV5HuP2Vlic9oiaGPEvVxWU6BadEW4I6QJmP55I/640?wx_fmt=png&from=appmsg "")  
  
  
02  
  
0x2 培训课程介绍  
  
26  
  
**SRC漏洞挖掘培训课程**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6cIuvSQkkicOHhYFkQLTibYAMUR9rfZ9eUrI78toIC4V2304G909O6s6CnVrAGiaYLEJM9XuUARhzNfxCtYKQfQ83wfPSlqpshSScfoYzSKzgY/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&watermark=1&tp=wxpic#imgIndex=4 "")  
  
  
**1.课程价格目前是575（后面也会随着人数越多，涨价）🌟师傅们还可以上车补票，冲冲冲！**  
  
**2.报名成功送知识星球一个，拉内部小圈子交流群+SRC直播通知群！✨**  
  
**3.一周2节课程，直播+录播形式，课程内容大家可以看课表，目前是第一期，一次报名永久无限听课！❤️**  
  
**4.目前是第一期课程，后面比如说开了二、三期，都是不用在花钱的！**  
  
**5.上课结束后，会把视频录播+课件笔记一起打包发直播群！**  
  
**6.哔哩哔哩SRC课程公开课，链接🔗直达：**  
  
**https://space.bilibili.com/642258933**  
  
SRC课程详情🔎：  
[学了一堆理论，还是挖不到漏洞？你缺的是实战！](https://mp.weixin.qq.com/s?__biz=Mzk0Mzc1MTI2Nw==&mid=2247510755&idx=1&sn=6214a7f5fa76f846db1f5f8808455767&scene=21#wechat_redirect)  
  
  
内部小圈子知识星球详情🔎：[50 元封顶！渗透攻防 + SRC 漏洞星球限时开放！](https://mp.weixin.qq.com/s?__biz=Mzk0Mzc1MTI2Nw==&mid=2247509408&idx=1&sn=2e12452dfc2d34631af5109af28a6758&scene=21#wechat_redirect)  
  
  
欢迎关注公众号：  
神农Sec  
，报名咨询添加VX：  
routing_love  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mcko8AHj6QVcCkxIUpaBmNic17zibGfXMWrr9z89gE0DFtbOu3QYzD5d62zsp6qwc38Pssk60mLq8VKthcMOmctVlHU716S5G4KYmrKVrEj5c/640?wx_fmt=other&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=6 "")  
  
开课快五个月时间  
，课程目前已经  
累计加入了1000+个学员  
了，课程培训招生任火热持续中，师傅们  
对于我们课程感兴趣的，想要学习技术，找工作的可以咨询我报名  
。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVmdLBDlbl5p4Teyw5qqFOIFTIUxIxRay83I5qDXG690XI61gRj8MXTvTaibC4q2cCb1CbM4XS2FK6X4KYhPTX2ibgvA363YYwcE/640?wx_fmt=png&from=appmsg "")  
  
课程培训记录📝，每次上车在1-3小时之间，上课包括课程内部群大家  
交流氛围很好！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mcko8AHj6QV7iczVHowN13BzCTraG8jDUoe5hluiaZ90RUy7FjW398DictcrZhHrYpMgw4polRqvlGua6iakYdARPI3Jkiahhjvrvkviblm19U4F0/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "")  
  
课程上课笔记课件📒都会打包给师傅们，笔记都非常详细，很多几k价格的培训机构哪怕是课件笔记都没有的，我这里都是下课第一时间把  
录播+笔记打包发给大家！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWbRV4mBn8GZHrvHocPMYYcBuAM3gyIKOM0SicBWQhywMehkXInvEerRLySOPPMzEmM2GLSlOMFREx6QItqtCgCibGs2MeY6yvu0/640?wx_fmt=png&from=appmsg "")  
  
平常也都会给学员进行一些项目发布，包括后面的  
工作、护网内推等，经常上麦交流，大家互相学习，简历优化等。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXvjjkgJibDEUhdDjErjibiangGsN0rqb0Av59xfyxBbDrTMNdfIAhNXlx0HQKvxIVBIEGAAbYrEENzd77j65asejlD4a50Sb4U7o/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20 "")  
  
SRC漏洞挖掘课程培训已经两个星期了，期间也是创建了  
“回本小群”，希望学员回本越来越多，创建这个群主要是鼓励学员学习进步，以及不定时发小项目！  
  
最后也是希望大家都可以赚钱，找到好工作🎉  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVhlprHaGy2FoegRFSmNoGqGia3qPeafwId8US77OLOvMGttCDacH67NTuNbZdSGe2Tw97Z9VvfgnMva1pvc61bS6Iibbpcl4a7A/640?wx_fmt=png&from=appmsg "")  
  
培训时间不长，感谢🙏师傅们的  
喜报  
，很开心看到师傅们给我分享自己的成果，  
希望师傅们越来越强！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUMAtEWv3xXZPDsGBRhESmwGRciaasCGibU8TtbP2U0YVZPBdf5tlLqpWAtQKBh5oFwgETyvicKBeW1JSsekAyJ5cbRlSdjooQkSM/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUrS4N68nZ0EyE76Wkib7ZDrpnZWw2Q1RJQvFEdIOu5XvFGCwpz9lziabKyo9C9d5ZiamibuSXlibhXLHb7b8QJhqEIs3hXvqktkkyA/640?wx_fmt=png&from=appmsg "")  
  
  
平常也会分享项目，下面是一些  
学员项目成果  
，群里报课的学员都是不抽成的，主要是帮助学员进行  
回本  
，  
让大家都可以进步！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWS8lR4pmzZrczwr6YjtG48EqF9q4FlAUH78M7DXkiboqF8Q1HkeWJLzpFPOQBToO3auj8r4rU9x3fuafXUDVMEcFj5EI6U3P9w/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23 "")  
  
上课结束后，会把  
视频录播+课件笔记  
一起打包发直播群  
  
**「神农安全」**  
知识星球目前已经  
累计2500+网络安全爱好者的加入！  
  
后面也是小圈子做大起来了，师傅们也都喜欢看我文章，想着给大家教下src漏洞挖掘思路，所以自己花了很长时间做了✨  
课件和课表，都是纯自己手搓的，大家也可以看下课表的内容。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b7iaH1LtiaKWXNmpV89Zxcm1J56eeHltthM2sjuWQFbmvWv79V058KwI0DswFF9LysewGtULj81Vp5bX9nTEK78A/640?wx_fmt=png&from=appmsg "")  
  
03  
  
0x3 课程特色  
  
课程  
主打真实，  
一线SRC漏洞挖掘师傅是如何学习和挖掘SRC漏洞的，让你真正了解SRC漏洞挖掘，助力在岗人员和大学生的能力提升，掌握新的技能树，为下一次  
跳槽涨薪做好准备。本  
课程内容覆盖企业  
SRC、众测项目挖掘、护网HVV红蓝攻防技巧、CVE、CNVD、EDUSRC等平台通杀案例技巧挖掘方法。  
  
本课程  
适合人群  
（光看不挖啥也不会）  
```
1、有计算机经验，想从0转行入行的大学生或自学者
2、想从CTF比赛/Web或SRC进阶到项目实战的选手
3、想参与项目/找工作/提高收入的转型者
4、想通过挖SRC赏金做副业的师傅们
5、挖SRC漏洞遇到瓶颈的师傅们
6、想学习AI安全自动化渗透测试漏洞挖掘的师傅
```  
  
课程价格：575 元  
  
报课成功的师傅们直接免费送内部小圈：一个知识星球+内部小圈子交流群  
```
1、课程价格真心实惠，绝不割韭菜
2、四五百的课程价格让你体会大几千的培训课程内容
3、带着大家从0到1，本人上课坚持手搓课件（实战案例+知识体系）
4、拒绝使用PPT演讲模式（无实操，很枯燥）
```  
  
直播培训教学方式  
  
课程  
一周1-2节课，课程特色涵盖直播多人上麦活跃回答，直播过程中有问题随时解决或私信我。  
拉群：一个知识星球内部小圈子交流群+课程培训直播通知群。有项目/工作/护网第一时间内推报课的师傅，  
一对一简历优化，助力在岗人员和大学生的能力提升。  
  
一次报名每期均可永久学习，并且赠送内部「神农安全」知识星球，一对一永久解答、无保留教学！  
  
欢迎关注公众号：  
神农Sec  
，报名咨询添加VX：  
routing_love  
  
课程均为线上交付，报名成功后  
不支持退款  
  
内部小圈子  
（知识星球+内部小圈子交流群+知识库）  
  
对内部小圈子感兴趣的师傅们也可以看下下面的这个  
跳转链接，里面有对小圈子的详细介绍，报名课程成功的师傅们直接免费送一个（直接点击下面直接可以跳转）。  
  
[强烈推荐一个永久的SRC挖掘、渗透攻防内部知识库！](https://mp.weixin.qq.com/s?__biz=Mzk0Mzc1MTI2Nw==&mid=2247510184&idx=1&sn=3f1f8405d9bd8d029b00c74958a7560b&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QX3wmlibyqswnPFFJbo2EofN3fKNhsKibmicBUAkkriazg3ViciaIt9ArbhlIHzpAlJY7uKboEVKSp06HicEFn5sz4sAVSByyduK5rhUk/640?wx_fmt=png&from=appmsg "")  
  
讲师介绍  
  
id：一个想当文人的黑客  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QX6UX8mhQtia4qnfEiasbq2R3KjlwQg2ysg4ibj744R4DF0BXZQZBjHc3qNgPKkqG7msub5w6WjSmoElCibibTp6qImS3FkupITqJUk/640?wx_fmt=png&from=appmsg "")  
  
欢迎关注公众号：神农Sec，报名咨询添加VX：  
routing_love  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b7iaH1LtiaKWXLicr9MthUBGib1nvDibDT4r6iaK4cQvn56iako5nUwJ9MGiaXFdhNMurGdFLqbD9Rs3QxGrHTAsWKmc1w/640?wx_fmt=jpeg&from=appmsg "")  
  
04  
  
0x4 第一期挖洞培训课表内容  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXdFkU8hwaxia8XQ7EyshqMb1BUOknbNI4lhtliaE0iakNZ0PRmjBUocUGbGDmEaGwuZDDP4sXkOrjicxI1exTafD6wdNUTj66wCWw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0F0PmZricIVE4aZnhtO9Ap086iau0Y0jfCXicYKq3CCX9qSib3Xlb2CWzYLOn4icaWruKmYMvqSgk1I0Aw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**内部圈子介绍（报课赠送）**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0F0PmZricIVE4aZnhtO9Ap08Z60FsVfKEBeQVmcSg1YS1uop1o9V1uibicy1tXCD6tMvzTjeGt34qr3g/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
  
  
**圈子专注于更新src/红蓝攻防相关：**  
  
```
1、维护更新src专项漏洞知识库，包含原理、挖掘技巧、实战案例
2、知识星球专属微信“小圈子交流群”
3、微信小群一起挖洞
4、内部团队专属EDUSRC证书站漏洞报告
5、分享src优质视频课程（企业src/EDUSRC/红蓝队攻防）
6、分享src挖掘技巧tips
7、不定期有众测、渗透测试项目（一起挣钱）
8、不定期有工作招聘内推（工作/护网内推）
9、送全国职业技能大赛环境+WP解析（比赛拿奖）
10、十个专栏会持续更新~提前续费有优惠，好用不贵很实惠
11、每日内部资料分享，内部圈子资料1000+
12、联系圈主获取：内部漏洞知识库+圈子使用手册+内部圈子交流群
13、VX：routing_love，技术交流+疑问解决
```  
  
  
**内部圈子**  
**专栏介绍**  
  
知识星球内部共享资料截屏详情如下  
  
（只要没有特殊情况，每天都保持更新）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b7iaH1LtiaKWWYcoLuuFqXztiaw8CzfxpMibgpeLSDuggy2U7TJWF3h7Af8JibBG0jA5fIyaYNUa2ODeG1r5DoOibAXA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b7iaH1LtiaKWUw2r3biacicUOicXUZHWj2FgFxYMxoc1ViciafayxiaK0Z26g1kfbVDybCO8R88lqYQvOiaFgQ8fjOJEjxA/640?wx_fmt=png&from=appmsg "")  
  
  
05  
  
0x5   
优秀学员报喜  
  
下面是最近两个月培训期间，很多  
优秀学员进行报喜，看到师傅们有收获，也是感到很开心的！  
拉回本小群，就是为了促进大家学习，在群里发学员成果，也是为了让大家学习优秀的师傅们。  
  
加油，你我皆是黑马！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWS8lR4pmzZrczwr6YjtG48EqF9q4FlAUH78M7DXkiboqF8Q1HkeWJLzpFPOQBToO3auj8r4rU9x3fuafXUDVMEcFj5EI6U3P9w/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWkBFHe0S1MayHGboNyYGhNR94Fic11frXxdUGBgjjIx6dnJ6lgWxw7iajkmFTiczQq5DHN1bwUchcVzatv95E5gibAMUiaZ7fHlypw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QU6ibZutPq43zUiap7IgDmJq7kwUKJBCa2IDujYiadMJfe9fFH9DOfUEOM2TibibYRuFiahDqMnBX1MVjLw5XIdNDSuR5P3g7XibaUkBo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVz2wFpVfer0uAFVpLKyicMaaLkmJDdg5bWnOotuzN3S9r2FMKpEKrJy8ND7icWVzNgqyYS2J6XElVN43vGca4X6HcEqapwGcNX0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWoKYrxQzob221aCmicmemD3aVPbk5dvVIaEic4TNPXrnkRazOTHnIbq87Jbk0GREdlI4iaZUmVU3c6K6rBbyZzBnkicooOUVtzN1I/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWdSycCicsfpn14HlUgEibU04lpXJ4a70L4D6oSWx5s0tLgnLnLiaRAdclxNVicYKFRD1mGn40jQ4t4ic8XZzVoOSTTCzY9xz9auzJI/640?from=appmsg&wxfrom=12&wx_fmt=other&tp=webp&usePicPrefetch=1&watermark=1 "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXB1O6Fx3ia62NNWITh9vUQaEKp7epibLWeEsdobibvBvqNDoTCAvfyQFHw597O24naJAIpM4QALgfqMWWc4E1KHrxBoaGRxE4Ajc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXnjwIRWjJOVSuN4X4HjmEFtCVqCHZ05M77sXqzmVjibaJbLUw3ApOuz7iaH8OCCnRmTRYVtKC5NajGKVkI4pnKZsJaj0T4iaYibq0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWRdtEUY2aeAZh34wDle515j7UwnibFQeCibWSeDKGnIZ2YH5VGX64cYeXgPGdCwHLKdsMY07EIVliapxh10gzQ2EO3bks7bxhmVs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QW7rozFqSBVNRDE2kbfUSB4FefOPm9LXM2B9bV4n9VPM7Kt11rfw284Ejn4AHUc1Uc1r1gZs3FF6umgPk1QejcC6zrOAYEyegY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QV0epFsicbJrNPgGwNXcXRCDrC1sGQySI2ylkfs2Hdic6d6unjwqNiby5DfhtfT6ezabX13bNeR53pOW3BUqLaZrIvPM8Z4IsBpgI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXwfrgic5XLseOxkPOWkjm1yicAW2ZiaAqzxtbjPok4Yhic2Wiblic93SSGN5BtT77AFuZt6ySuRL09icqIicPuOUUbL5NbWgMCHgLictiak/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mcko8AHj6QU7jGMRbvGyeDmHE6KjibHDNmuqO2NDaG4soIjTtg6uQoy4H5x0FntPDicjnUtibVgFMTNvNRaA9SJicNj2BIWQNz2vfRaQDfkKsibI/640?wx_fmt=jpeg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUyyXAWlMf8dHcspnMDucUzRBbeXWW58yMWQncvENPDmIpEKr4HlZ0cyWZSLGiakB03zRmvicfIF79LdWQ6s6VOyxl44WgvMwzf4/640?wx_fmt=png&from=appmsg "")  
  
  
  
**神农安全公开交流群**  
  
有需要的师傅们直接扫描文章二维码加入，然后要是后面群聊二维码扫描加入不了的师傅们，直接扫描文章开头的二维码加我（备注加群）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mcko8AHj6QVnjACnCl9MjxdH77Bw7CKAPD5WU3FSp7FEwMjJ90LibUP9XckWuKWeATTqZCV5siaBo3dhIFmnzsnJdFDDhlhRfyOF5fLOj5jdk/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/mcko8AHj6QV9grs7NOhSTCfTpCc4xrxdnlISIReNNCKR2EOyWvhMpyIzbma8nuelSg8LicKF5yYZ7hgyODlWgMmhViaE8Ahhs7PZlnmA0VFcY/640?wx_fmt=jpeg&from=appmsg "")  
```
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/b7iaH1LtiaKWW8vxK39q53Q3oictKW3VAXz4Qht144X0wjJcOMqPwhnh3ptlbTtxDvNMF8NJA6XbDcljZBsibalsVQ/640?wx_fmt=gif "")  
  
  
