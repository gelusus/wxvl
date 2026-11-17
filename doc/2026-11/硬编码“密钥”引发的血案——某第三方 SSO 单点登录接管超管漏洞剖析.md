#  硬编码“密钥”引发的血案——某第三方 SSO 单点登录接管超管漏洞剖析  
原创 zkaq-腾风起
                    zkaq-腾风起  掌控安全EDU   2026-11-17 07:19  
  
扫码领资料  
  
获网安教程  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/BwqHlJ29vcrpvQG1VKMy1AQ1oVvUSeZYhLRYCeiaa3KSFkibg5xRjLlkwfIe7loMVfGuINInDQTVa4BibicW0iaTsKw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=1 "")  
  
  
# 本文由掌控安全学院 - 腾风起 投稿  
  
**来****Track安全社区投稿~**  
  
**千元稿费！还有保底奖励~（ https://bbs.zkaq.cn）**  
# 免责声明  
  
本报告仅用于安全研究和授权测试，未经授权的渗透测试行为是违法的，切勿用于其他用途，请确保在合法授权的情况下进行安全测试，出现任何后果与本人无关。  
## 0x00 前言  
  
一个真实的历史漏洞 PoC，某第三方 SSO 登录接口是如何因为一个极其低级的错误——**算法密钥硬编码**  
，导致攻击者能够伪造任意身份，并在秒级时间内接管系统超级管理员权限。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoKKMaadsIAvTh2MzCFk7uock8Ziaqx1BmVrGjdk6RFRXQTTibIK97iaPELlfDgLHP2DWxzEGyxLglj7mmMEQgME1tiayonNw3DR3nQ/640?wx_fmt=png&from=appmsg "")  
## 0x01 漏洞PoC 分析  
  
这是一个异常的第三方 SSO 登录请求引起的。该接口的设计初衷是允许受信任的第三方系统通过签名校验直接免密登录，但其校验逻辑存在致命缺陷。  
  
以下是触发漏洞的原始 HTTP 请求（PoC）：  
  
HTTP  
```
GET /XXX-login/XXXXXd/ssologin?username=xxxy_admin&timestamp=1697765009348&auid=A5AXXXXXXXXXXXXXXXXXF6E2 HTTP/1.1Host: {{Target_IP}}User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng, */* ;q=0.8Connection: close
```  
  
从结构上看，这是一个非常典型的基于凭据签名的 API 请求，该接口需要3个参数，username（要登录的用户名）、timestamp（时间戳）、auid（签名）；系统默认管理员用户名为 XXXX_admin，时间戳使用当前时间的毫秒级时间戳即可，签名构造方式为md5(username+ timestamp+”密钥”)  
  
懂行的兄弟一看这三个参数就知道套路了：  
1. username：目标用户，这里是系统默认超管 XXXX_admin。  
  
1. timestamp  
：13位毫秒级时间戳。  
  
1. auid：32位的字符串，一股浓浓的 MD5 味，绝对是签名。  
  
## 0x02 硬编码密钥  
  
一般来说，这种签名的逻辑都是 MD5(参数1 + 参数2 + 密钥)。如果密钥在服务端动态生成或者配置在独立的文件里，这接口基本打不进去。但是这个密钥确实硬编码。固定为 "XXXXssomd5"， 既然算法公开、密钥已知、而且超管用户名 XXXX_admin 也是默认的，那这事儿就简单了。 根本不需要爆破，伪造当前时间戳，算出一个合法的 auid  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoIf6iaf7lYVDSV2l5bbaxm4bM3tM40H1CrbgyDWQO332hoibQ17VuSoMMBMs1zdqeUV5IwibM2mCoAv286vsGtktT987t1RKxuPdM/640?wx_fmt=png&from=appmsg "")  
  
当服务器接收到上述伪造的请求后，由于拼接计算出的 MD5 值与后端完全一致，服务器被成功欺骗。它会误认为这是合法第三方系统发送的免密登录请求，并直接下发代表 XXXX_admin（超级管理员）权限的 Session 或 Token。实现超管接管。  
###   
  
## 0x04 漏洞危害  
  
配置修改  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoIlHqIOzciaHdfIjbNiccvDBTF8BZAibibwesRPYDLJZET7XjdJtyo8cd8CPy6QDwR8PFUCnP7IaU06snfwhyZU9XwZ1OAm0KWfKmI/640?wx_fmt=png&from=appmsg "")  
  
数据库账号密码  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoIQbf4iaiciaia8BbakFgBSrVjlpAHsODibyCkKuN5rS3uaXqXKPIYv1ib1nDpaY21Y9Dj7UMgt61mXwWS2ktm77OWjqvy3mPkgRbKwc/640?wx_fmt=png&from=appmsg "")  
  
操控数据库  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoLNAurd8jXepTPT1ADZFibT4LL8V2ibmYP9bhgibAhyhrPt6ccMPcJyI6vKo8NUItvBpTmz921WvOuSlw5xJCiaq1Eic2Xdx7IV4DIE/640?wx_fmt=png&from=appmsg "")  
  
## 0x05 扩大危害 打新 0day  
  
  
既然有了超管，那肯定要找接口测垂直越权了。  
  
找一个普通用户（比如学生或教职工），进入系统后，后台能点的菜单寥寥无几。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoLXxo4IZXnNMkFQkTzuqibFD8uoIrCAkDCZ9lvWs12o1EtfQgUdmUZ2hYUxxQYiaoAIBMIn6g8KLoa0b0p1K9HFlrkRlicwHlg80k/640?wx_fmt=png&from=appmsg "")  
  
## 0x06 敏感信息垂直越权  
  
  
 这时候老规矩，祭出 Burp Suite 开启被动扫描，在前端瞎点一通。很快，我在 HTTP 历史记录里抓到了一个接口：  
```
GET /xxx-auth/xxxxxxx_user/v1/query?querykey1=&roleid=&keyword=&typex=&isauth=&accountstatus=&isneedupdpwd=&_search=false&nd=1776067458549&rows=20&page=1&sidx=&sord=asc HTTP/1.1Host: 218.21.168.206:18080
```  
  
  
  
这是系统超级管理员才配使用的“统一人员管理”查询接口！我把这个请求送到 Repeater 里替换成普通用户的 cookie 重发，好家伙，居然直接返回了 200 OK，而且拉出了系统里几百个用户的详细列表  
  
这意味着系统的接口存在垂直越权漏洞  
  
通过这个接口，我成功拿到了大量用户的核心信息，最关键的是拿到了目标用户的 id 和 username。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoJGpcWgndoY60DcrvcWzguibA9FRyT2xWSQqt23G8s6meetsEyficnzwgTFWfjcHw0XghkMH0rWePicBWKv5Sl5grktFTusEicFwS4/640?wx_fmt=png&from=appmsg "")  
  
## 0x07 任意用户提权为超级管理员  
  
  
抓取编辑用户权限的报文  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoIlwdfrNFf0BujCYM5DMdHKm7OeSrcMczH6iahC9ef5LrypOKfUzENErib64iaG1nFjtNIHMIicFHYGibrHxXQJZoIAWn7CIDia7LbT0/640?wx_fmt=png&from=appmsg "")  
  
  
注意看 POST body 里的 roleid=40289xxxxxxxxxxxxxxxxxxx6000d。 这个长串的 Hash 值，正是在第一个接口里查询到的系统超级管理员的角色权限 ID  
  
分析其需要的参数发现，与垂直越权的接口泄露的信息能对应上，再次替换为普通用户的 cookie 发包，成功  
  
可以将任意用户提权为超级管理员  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoJjVtc7Bs0bAibqULqMDAW1y9Zeuicv4EtdpIRAhoicXHVTpsVdUdDEbG51Mm7Fauf7fU6vSukLgibPGTfvdFUUXrYmhibP0RbOYR9U/640?wx_fmt=png&from=appmsg "")  
  
  
 我退回登录界面，用那个被提权的 JS00023 账号重新登录，发现左侧菜单瞬间全开，系统配置、权限分配、数据库管理功能全部暴露。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoLFMZFafqI8Etb0AfXNsMRXLLH7zGJHoo4RqKw4EvAnFgW30Ua2tMrVg7VGvp1xaibhtjBLPN5h9H1IVCNljKl2FPdKQ5YsO9BU/640?wx_fmt=png&from=appmsg "")  
  
  
申明：本公众号所分享内容仅用于网络安全技术讨论，切勿用于违法途径，  
  
所有渗透都需获取授权，违者后果自行承担，与本号及作者无关，请谨记守法.  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=20 "")  
  
**没看够~？欢迎关注！**  
  
  
**分享本文到朋友圈，可以凭截图找老师领取**  
  
上千**教程+工具+交流群+靶场账号**  
哦  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/BwqHlJ29vcrpvQG1VKMy1AQ1oVvUSeZYhLRYCeiaa3KSFkibg5xRjLlkwfIe7loMVfGuINInDQTVa4BibicW0iaTsKw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=21 "")  
  
******分享后扫码加我！**  
  
**回顾往期内容**  
  
[我与红队：一场网络安全实战的较量与成长](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247550558&idx=1&sn=589aa46a61b9ab02ab953ccb9539b1d3&scene=21#wechat_redirect)  
  
  
[网络安全人员必考的几本证书！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247520349&idx=1&sn=41b1bcd357e4178ba478e164ae531626&chksm=fa6be92ccd1c603af2d9100348600db5ed5a2284e82fd2b370e00b1138731b3cac5f83a3a542&scene=21#wechat_redirect)  
  
  
[文库｜内网神器cs4.0使用说明书](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247519540&idx=1&sn=e8246a12895a32b4fc2909a0874faac2&chksm=fa6bf445cd1c7d53a207200289fe15a8518cd1eb0cc18535222ea01ac51c3e22706f63f20251&scene=21#wechat_redirect)  
  
  
[重生HW之感谢客服小姐姐带我进入内网遨游](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247549901&idx=1&sn=f7c9c17858ce86edf5679149cce9ae9a&scene=21#wechat_redirect)  
  
  
[手把手教你CNVD漏洞挖掘 + 资产收集](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247542576&idx=1&sn=d9f419d7a632390d52591ec0a5f4ba01&token=74838194&lang=zh_CN&scene=21#wechat_redirect)  
  
  
[【精选】SRC快速入门+上分小秘籍+实战指南](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247512593&idx=1&sn=24c8e51745added4f81aa1e337fc8a1a&chksm=fa6bcb60cd1c4276d9d21ebaa7cb4c0c8c562e54fe8742c87e62343c00a1283c9eb3ea1c67dc&scene=21#wechat_redirect)  
  
##     代理池工具撰写 | 只有无尽的跳转，没有封禁的IP！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=22 "")  
  
点赞+在看支持一下吧~感谢看官老爷~   
  
你的点赞是我更新的动力  
  
  
