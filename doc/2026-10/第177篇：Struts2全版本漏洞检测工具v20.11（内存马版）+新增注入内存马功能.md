#  第177篇：Struts2全版本漏洞检测工具v20.11（内存马版）+新增注入内存马功能  
原创 abc123info
                        abc123info  希潭实验室   2026-10-31 12:19  
  
![](https://mmbiz.qpic.cn/mmbiz_png/OAz0RNU450ATcz6jUJnFNeOxRzVZ9LbcCCMJ6Af2WYicgMPA32IwibF8mI2ibC9h8jaHkhxnZzZuqctMLRTxDudicA/640?wx_fmt=png "")  
  
 Part1 前言   
  
大家好，我是ABC_123  
。Struts2全版本漏洞检测工具于2016.05.05 完成第1个版本，时间过得很快，一晃10年过去了。更新这个工具已经变成了一种情怀，不忘初心，也算是一种研究性和实验性的工作。本次更新加入了大家期待的注入内存马功能。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2Zc77PyLkuDupKmynbgOEwUUWTzUVjztJ8lNBUiaU9icoEOBhMD2DdLFiazribOgicGo0O9910qnCUP5NsLGWiabqBwOm1v6GShlmRj0/640?wx_fmt=png&from=appmsg "")  
  
  
 Part2 技术研究过程   
  
点击  
"  
检测漏洞  
"  
，软件会给出存在的Struts2框架的漏洞编号。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2byLBe10bTBaQIKQlibnZcRJLjlRBWWLspKcskX6Bv27kjLhP8GTtgVdf3vxbXRXTaRAcbQrSLBQmoLbiadXv5XrvI4bvEwnvMzs/640?wx_fmt=png&from=appmsg "")  
  
  
切换到"注入内存马"标签页，可选择冰蝎、哥斯拉、NeoreGeorg、Suo5等，中间件类型目前支持Tomcat，后续会添加WebLogic、WebSphere中间件。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2aibWWGNUasRTqxbvwAHLgXfs4tyKd9RHQIBAKP42vl6SvrfDPqc2icaqE6y41qpknd7fwV2yQiabkChSiapJ1EMxp8ZaXMHXFIMibY/640?wx_fmt=png&from=appmsg "")  
  
  
如下图所示，植入冰蝎内存马完全可用。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uPOMOKjLe2bVg4qvru33z2tT8Tfzk4iaWUCox7eknW2mxxw1k6Jfg4Zk5iccNgyMplsnE3G1bGdsNBXhOlcq1KE0ibZhZgvS2j5Yia4JhWLasI8/640?wx_fmt=png&from=appmsg "")  
  
  
将"内存马类型"切换到哥斯拉，植入后发现也完全可用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2ZI90Yc29uoZicwU3gyxz3H9KrjCP66yqcBRpMbIM6xkm7ycsu9Feibv4No9NMoqUicp6hP355I8UXR9BtqqLT6dt0Nej572OdPT4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2ZTRL08zl2fHtJyB4TNibWvdgEialQkOjVTVg5JDCMnHTl0NInEqhswVicIrOuHZibicSZ95c8laFOX8aUbHrrF7Z94K2ianiaw6hZc8g/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uPOMOKjLe2YoJey6RntIneeFfQmzHwuwZclmASZprMFDLUyODZjib0tgrflroMK42ljBiaFKPrd7S7icQ0jWs3QhBJgwYyobpkgBjKXv6icP5J4/640?wx_fmt=png&from=appmsg "")  
  
##  Part3 总结   
  
1.  
    
后续会添加WebLogic、WebSphere等中间件的支持，敬请期待  
。  
  
2.  
    
工具可进知识星球下载；不方便加入星球的网友，可以联系知识星球的用户获取  
。  
   
欢迎大家扫码加入知识星球  
希水涵@信安知识库  
，进入星球内部交流群，一起学习进步！  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/uPOMOKjLe2YU6YdILmbv6CbQQjPzT5fTMH918e5lLTyuBcJxypX32Ihkyu2ab3ibjhnq7gTk7dvnJ4HibQpSziaOoDwpQj83t9OMtJicPErfKbo/640?wx_fmt=jpeg&from=appmsg "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/OAz0RNU450A5qqg2iaK6KIYYR8y6pF5Rh3JHDibOKOop204nXz618iawdRb8dABicMPtHb2PkJE8x6koJO5HyuwZJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=18 "")  
  
  
**公众号专注于网络安全技术分享，包括APT事件分析、红队攻防、蓝队分析、渗透测试、代码审计等，每周一篇，99%原创，敬请关注。**  
  
**Contact me: 0day123abc#gmail.com**  
  
**OR 2332887682#qq.com**  
  
**(replace # with @)**  
  
  
