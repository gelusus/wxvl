#  记一次SRC挖掘—审计js还原隐藏接口产出2个未授权漏洞  
原创 zkaq- 诚信友善
                    zkaq- 诚信友善  掌控安全EDU   2026-10-30 06:05  
  
扫码领资料  
  
获网安教程  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/BwqHlJ29vcrpvQG1VKMy1AQ1oVvUSeZYhLRYCeiaa3KSFkibg5xRjLlkwfIe7loMVfGuINInDQTVa4BibicW0iaTsKw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=1 "")  
  
  
# 本文由掌控安全学院 -  诚信友善 投稿  
  
**来****Track安全社区投稿~**  
  
**千元稿费！还有保底奖励~（  https://bbs.zkaq.cn   ）**  
# 一、前言  
  
    本次测试从一个运维系统开始，通过审计 JavaScript 中的 Chunk 映射，还原出未在首页直接加载、也未被常规插件识别的js文件，并沿着请求封装继续定位到多个隐藏接口，挖掘多个未授权漏洞   
# 二、正文  
  
开局一个登录框，出洞漏洞全靠肝  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoJmpoYglZMc3IneYYwUbLYqVHyyFjic9AfKrmAibmzxkbeUQj02WgsN8IOzrkWOOxxhib8MHOmSbSrgeHJUGqqfywGknZibPnWmyG4/640?wx_fmt=png&from=appmsg "")  
  
按照正常流程把该测的功能点都测试了一遍  
  
雪瞳和小熊猫插件识别的接口也全部都跑了一遍，没有任何产出  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoJx6UIZDt7sdibZS0aO9Ofa9OadwbkUCiawWpujbgvP2FliaPMUI8TIia1ibQXZEtTkiauTklc6Hiak7xs4eHWVsGfmSBq54WLsAE1Qtk/640?wx_fmt=png&from=appmsg "")  
  
查看引用的js内容，发现baseURL /api ，雪瞳和小熊猫里面是没有这个baseURL /api 相关接口的，所以如果我们能找到相关接口大概率有戏  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoLJ2OY25xePPVZOia179xymJUhnZerEyDY10P1lHt0LXxNgiaIeribyu58L7DkjRfGu3Bib5sZcXWawCwYsTyqlsEkRYMrxTMkuR5I/640?wx_fmt=png&from=appmsg "")  
  
继续查看js内容，发现 Chunk 映射---类似：  
"chunk-xxxxxx":"xxxxxx"  
  
计算出的完整地址：  
https://xxxxx.xxxxx.edu.cn/static/js/chunk-xxxxx.xxxxx.js  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoJNiaPcib5qLalGPic3Uicue3RRd6CfH7RvBJSk4llsm5z2PmAnDCuFbtruvb2zaCEFIX0o4bCfyvAVlwSNibuz7ibPBewtQ6Gz9tcak/640?wx_fmt=png&from=appmsg "")  
  
打开  
https://xxxxx.xxxxx.edu.cn/static/js/chunk-xxxxx.xxxxx.js  
  
发现众多插件未识别出来的接口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoKTecF86wRelnofr8bF012F1nkNNvQiaYia9DLAjv9DwhApJyibnrT8giaEiaYPCWZ4lfRmrZUqicNA8Fibqia170uEjwx7eRaSbkGibAFs/640?wx_fmt=png&from=appmsg "")  
  
最终恢复接口发包成功获取 全站用户名和MD5密码等敏感信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoL6vxvLcHUSJE1K6icbZhEkb8icMGccapia49e20eXEfmEdVDUuNKfEjkbAQiaLFBQbF2uJHFS3CKvJq2yQick0wbOfNgLDgRUYic5jc/640?wx_fmt=png&from=appmsg "")  
  
还有一个getxxx接口是查询人员排班信息的，不过查询出的信息没什么敏感的  
  
我们通过查询一个未来的日期，可以发现是没有排班信息的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoJMqOpib7fDJRs2sGOhbEYWbibib6Yy2qFA1QTyicicFLNW7fhYRVpibiaBialMcicPhibp1LJKIQfM7TQZlraibicDQTWvLiaX2FH01LuhoQw4/640?wx_fmt=png&from=appmsg "")  
  
通过savaxxx接口给两位用户写入了2099-12-31的排班信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ianpxKPnLHoJCwYj0u8BhhmBib0g6PL4boBf4yt5SslNlBllEMa5k0usdWoVz4XDPuY94x6FwtllAdm6iaTcRaUdonBRibqOefHBP0DaWc9h1Kw/640?wx_fmt=png&from=appmsg "")  
  
再次查询2099-12-31的排班信息，成功查询出刚刚写入的排班信息，证明存在未授权排班信息写入漏洞  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ianpxKPnLHoJKDDy1WHqibFsclQibJIJIqzicpYNACJGVmUtRwibzaFIjSLl9BOr4Rl7b69Ez1vDHWeBsUHXH4NciaHwzxwDLppIzeriaH3U29M9uc/640?wx_fmt=png&from=appmsg "")  
# 三、总结  
  
对于采用 Vue/Webpack 的系统，测试时不应该只依赖自动化插件，应该把Chunk 映射当作必查项，逐一还原隐藏模块对其中接口进行验证，或许可以发现前端未暴露但服务端缺少授权控制的未授权接口。  
  
申明：本公众号所分享内容仅用于网络安全技术讨论，切勿用于违法途径，  
  
所有渗透都需获取授权，违者后果自行承担，与本号及作者无关，请谨记守法.  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=34 "")  
  
**没看够~？欢迎关注！**  
  
  
**分享本文到朋友圈，可以凭截图找老师领取**  
  
上千**教程+工具+交流群+靶场账号**  
哦  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/BwqHlJ29vcrpvQG1VKMy1AQ1oVvUSeZYhLRYCeiaa3KSFkibg5xRjLlkwfIe7loMVfGuINInDQTVa4BibicW0iaTsKw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=35 "")  
  
******分享后扫码加我！**  
  
**回顾往期内容**  
  
[网络安全人员必考的几本证书！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247520349&idx=1&sn=41b1bcd357e4178ba478e164ae531626&chksm=fa6be92ccd1c603af2d9100348600db5ed5a2284e82fd2b370e00b1138731b3cac5f83a3a542&scene=21#wechat_redirect)  
  
  
[文库｜内网神器cs4.0使用说明书](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247519540&idx=1&sn=e8246a12895a32b4fc2909a0874faac2&chksm=fa6bf445cd1c7d53a207200289fe15a8518cd1eb0cc18535222ea01ac51c3e22706f63f20251&scene=21#wechat_redirect)  
  
  
[重生HW之感谢客服小姐姐带我进入内网遨游](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247549901&idx=1&sn=f7c9c17858ce86edf5679149cce9ae9a&scene=21#wechat_redirect)  
  
  
[手把手教你CNVD漏洞挖掘 + 资产收集](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247542576&idx=1&sn=d9f419d7a632390d52591ec0a5f4ba01&token=74838194&lang=zh_CN&scene=21#wechat_redirect)  
  
  
[【精选】SRC快速入门+上分小秘籍+实战指南](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247512593&idx=1&sn=24c8e51745added4f81aa1e337fc8a1a&chksm=fa6bcb60cd1c4276d9d21ebaa7cb4c0c8c562e54fe8742c87e62343c00a1283c9eb3ea1c67dc&scene=21#wechat_redirect)  
  
##     代理池工具撰写 | 只有无尽的跳转，没有封禁的IP！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=36 "")  
  
点赞+在看支持一下吧~感谢看官老爷~   
  
你的点赞是我更新的动力  
  
