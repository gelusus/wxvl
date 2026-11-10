#  漏洞提交到telegram应急响应中心?为什么我不再挖国内src了  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-11-09 12:16  
  
我提交的漏洞都是可以对系统中业务数据进行增删改查的，很多可以水的漏洞都懒得提的。  
  
你说漏洞重复，我可以理解，  
毕竟用ai挖的越来越多了。但你说没危害，感觉就像再说我是个外行，简直是侮辱。  
  
src驳回理由千千万，下面是网友投稿的又一个src驳回漏洞的骚操作。  
  
看点总结  
  
src偷偷修复漏洞，然后驳回漏洞理由：无法复现。  
  
最搞笑的是，漏洞修复记录居然出现了前端代码里面，开发在前端js中加了备注，修复xx漏洞  
  
投稿内容：  
  
  
这个漏洞一个内联注入的XSS  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPqibKEMoadaewxiaDcnohibg1u25gNUFkQd9yevictX4B499XpJAickw3ECUzdKZoMLUTmVHXNN8BFTXF2aa3IBsLpKQ6SmYfMOciaN0/640?wx_fmt=png&from=appmsg "")  
  
可以修改登录表单，伪装成该页面的登录输入窗口，被害人点击链接后，输入自己的账号密码，攻击者可收到被害人输入的内容  
  
这里起一个接受的服务，就可以看到受害者输入的账号密码信息  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPr2QF3XFMQLkSLOkSjBNiav9qnMDS5gDXUwPtLWDAZBbcsRZGG6aFqibrJ8Y4JJibic4pWjEXtL0RVEl8xuhaytgENK6sKJShP6Re8/640?wx_fmt=png&from=appmsg "")  
  
其实到这里都比较正常  
  
就是普通平常的一个漏洞，这个漏洞的payload是有时效性的，在构造一次就可以了  
  
接下来就是我最不能理解的地方  
  
厂商驳回，给我的理由是，漏洞无法复现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPrpicHAptQBENyPicLlD4gsJBicf1dgyJQtjsibSBoib5ib0Ztvwp2oqQGKibOz65FsSHZaPFYX1T8FZ88ic8dJNbot4dR6Jyp2uIIpzm0/640?wx_fmt=png&from=appmsg "")  
  
我是10月份交的洞，你墨迹墨迹到11月份才审核，你说复现不了，说我步骤不清楚，我也无话可说  
  
然后你偷偷摸摸的把漏洞修了，我也理解你，我都没话说，我都理解  
  
但是，哥们，你修洞的时候，能不能把你js里的注释也删除一下呢，你注释特地标明了我提交给你的漏洞名称是不是有点把我当傻子了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPrFlqZBtXMXjyHiaRhmNLviccqiciavqfC4NVuGuibvXq4GYPADjLSuZGhEbMS1XmjsyqC05pwqfuPmialLAJxxk5gfeQya4uWRyMP6o/640?wx_fmt=png&from=appmsg "")  
  
  
咱就是说，你既然复现不出来，那你给我驳回了，肯定就是不认这是个漏洞，那你又为什么要修他呢？你偷偷摸摸修就算了，我麻烦你能不能把注释也删除一下，把你爹当傻卵呢？挑衅我？谢谢白帽辛苦提交漏洞，你爹我不认，这口饭不想给你吃，我还要当你面修你的洞，还要告诉你，这就是你的漏洞，但是你爹我不给你过，你爽不爽？不爽来干我？是这意思么  
  
当然，被坑漏洞已经不是一次两次了，正常我也已经习惯了，但是碰到这种修洞js里还这样注释的我也是第一次见，我也是真没招了，忍不了  
  
  
  
  
  
  
  
  
  
  
  
  
