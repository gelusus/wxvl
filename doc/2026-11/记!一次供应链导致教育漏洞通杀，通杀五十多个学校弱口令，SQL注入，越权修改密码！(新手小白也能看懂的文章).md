#  记!一次供应链导致教育漏洞通杀，通杀五十多个学校弱口令，SQL注入，越权修改密码！(新手小白也能看懂的文章)  
xhys
                    xhys  HACK之道   2026-11-11 02:12  
  
   
  
```
作者：xhys
原文：https://xz.aliyun.com/news/14882
```  
  
  
### 前言  
  
  
   
  
小编认为哈，就现在来讲，对于教育系统来讲，信息收集还是很重要的！但是像什么什么大学比较显眼的 "统一认证登录"||"学生综合管理",基本上弱口令是不太行,别的漏洞的话好多也都被大佬们打完了或者利用上不太容易(菜鸟的内心崩塌)，所以我们可以扩大咱们信息收集的范围-->**供应链**  
，利用一些边缘资产混一下Rank！hh  
  
这里小编亲自整理了一套信息收集语法，宝子们捡漏就完了！  
  
谷歌语法常用搜索参数：  
  
site：  
  
指定域名，如：site:edu.cn 搜索教育网站  
  
inurl：  
  
用于搜索包含的url关键词的网页，如：inurl:uploads 文件上传，搜索关于公司有关的网址中含有login的网页，  
  
intitle：  
  
搜索网页标题中的关键字，如：  
  
intitle:“index of /admin” robots.txt  
  
intitle:“robots.txt”  
  
intext：  
  
搜索网页正文中的关键字，如：intext:登陆/注册/用户名/密码  
  
filetype：  
  
按指定文件类型即文件后缀名搜索，如：filetpye:php/asp/jsp  
  
cache：  
  
已经删除的缓存网页，推荐组合使用  
  
谷歌语法拓展  
  
查找后台  
  
site:xx.com intext:管理|后台|登陆|用户名|密码|系统|帐号|admin|login|sys|managetem|password|username  
  
site:xx.com inurl:login|admin|manage|member|admin_login|login_admin|system|login|user  
  
查找sql注入漏洞  
  
inurl:.php?id=23 公司  
  
inurl:.asp?id=11  
  
查找上传点：  
  
site:xx.com inurl:file| uploadfile  
  
查找敏感信息泄露  
  
intitle:“Index of /admin”  
  
intitle:“Index of /root”  
  
intitle:“Index of /” +password.txt  
  
intitle:phpinfo()或者inurl:phpinfo.php  
  
查找未授权访问phpmyadmin  
  
inurl:.php? intext:CHARACTER_SETS,COLLATIONS, ?intitle:phpmyadmin  
## 信息收集  
  
site:edu.cn ("初始密码表"）& （附件）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic69K36GsomN4ibARDKbCyTDjbZl5OXqHlTibpNGCiaCpMrXqibXH2TDcb46EVkGPa6ZkVd5yJLZpVFyXPQomrMdwGhd6P2DkPvmiauO8/640?wx_fmt=png&from=appmsg "")  
  
一次意外经历找到了一个学校运动会平台在报名操作手册上直接有了初始登录密码xxxxxx  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6icCb3oCcOdTmicxQKuJicBzYCzaFskRu3SQNHjK2ovxXxakgcFwOGQvLic6ebCudKSvUXJ0QFTiaFkgxmPsQGRibX2zOEeUUHTG44wY/640?wx_fmt=png&from=appmsg "")  
  
发现不只一个学校在用所涉及学校单位一共58所  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic696TXtSbTibQz3mxo7VrG2T22IEFCwQTZ0FXxibCUg9vZ1PiboWePB9zmzAMtMWQj0C4FUCwXylPPQrelYxgJq2hubUpictzz5nibsw/640?wx_fmt=png&from=appmsg "")  
## 漏洞利用  
### (1)弱口令,信息泄露  
  
利用收集到的弱口令发现直接可以正常进入  
  
造成部分敏感信息泄露!  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic68S8XIVdGTblicZp1UXia0yqeprHjW4DPC2myInHo7yMvTSMfzy1cQk3reeokqxPn6SibBjPJBGO7gOrpwZ2NHx5qtTJSBDWNmVVk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6icwT5ml0ZumSBUvdpHb8rXKnyyGn9Zu1Xc9aNOv41pibanyRFVXrSndoCicDuhDuicfV5Iexic0uZRtxurC1x57GicpvZ11Z7wEyicDo/640?wx_fmt=png&from=appmsg "")  
  
一共58所学校单位,经过测试超过20所都使用默认密码!  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6icfZ1gsMBfdCoc6DpkK4QxHd9lFE7wtkBCnP5ndd4ibicicofNreKOB5rs4gy0sgCuTH9RvECA2ibfdByJaOdjgKQdlgJrjIwBJcJ8/640?wx_fmt=png&from=appmsg "")  
  
完成一波弱口令信息泄露小通杀  
  
再想了一下,好不容易都进来了,总不能只拿个低位低危吧!  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic68VqMHsahrN2j7p1hIxH3bDtRrTqtNIHupZot6ia3HBmWanVlrYOYHSp0ThfyBKyUqHibkJzOQia6HGv7BKY5k5bUoFA6usZ8MYBE/640?wx_fmt=png&from=appmsg "")  
  
于是就有了下面的测试  
### (2)SQL注入  
  
在经历了多次抓包放包功能点后,发现了一个功能点极有可能存在sql注入出库名  
  
(经过测试后也是验证了自己的猜想!)  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6ic4ibwcwTOgwj76LHJyWJnibEsCTGJoGwF5RD6yBq3D0A5SARl9L8Ig4fNNxV7seJ8VW9Nf4IZQEVCmLMGsgJYAEQk1uNkC3Coibk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6ic8YppYq50SIuMRaib0ks5JV77MHQshTiblneHia5NRgGzVdBIRb2GohxA0GM0SrtyicgBtbN3Rne1aDlMgibAxDOQECCg9XeGbibIIs/640?wx_fmt=png&from=appmsg "")  
  
在查询成绩页面,由于显示的都是表格模式,所以载流量包li面有一个比较显眼的数据  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6icsffziarb5TCtpw8HV6y14IjehB5SXAg9BlVO40awibO0vSe6RKpELicOWribUGm7KM7D37rWYOnhx9IJA3QFRV6ScLUNSz4y45ia8/640?wx_fmt=png&from=appmsg "")  
  
sortname=hm  
  
对sortname参数的怀疑：  
  
sortname参数值为hm，看似正常，但如果这个参数的值没有被正确地转义或处理，它可能导致SQL语句出错，从而引发SQL注入。  
  
测试:  
  
1.修改sortname=1s  
  
这是一个简单的输入，尝试引入一个语法错误。  
  
如果应用程序未能妥善处理此输入，可能会导致SQL解析错误。  
  
例如，在某些SQL语句中，这会导致如下错误信息：  
  
SQL syntax error near '1s' at line 1  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic687Zb6ZCEb5IL203Qo4sGM8ibKHcnaHrhOyicUn8u1KV6yDZJhw5d4cqh5Ys612Dicic8UfSXc9Hhnic4gGGMxK1yWiaLR3qsrauRiaibk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6ic450hMMWCV0ORaJn7tTOl9cibW7y4zX1ia7ia0ciavoPLrricA5kRPeu1Y1mcg7b5Qia1LzfmEDAojfQZ7hNfEbKduI6dUWaHTzicH5Y/640?wx_fmt=png&from=appmsg "")  
  
不是哥们!  
  
2.修改sortname=exp(11111)  
  
exp是许多SQL数据库中的数学函数，用于计算指数。  
  
将不合理的参数传递给exp函数可能会引发错误。  
  
利用数学函数来引发错误，检查数据库是否返回错误信息。  
  
错误信息可能泄露数据库类型和版本。  
  
这里不太行  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69Cjbzeavb7rh7wPVmvtfJr81oq2x9ia7jUf0iaJN3u799rkg2dRoiaTb12UWKX384ZXGIe4LhdIfAHP5p3N0I7a0kVhDQIWHicqYI/640?wx_fmt=png&from=appmsg "")  
  
3.修改sortname=update(11111)  
  
分析：update通常是SQL中的关键字，而这里被错误地用作函数。  
  
这样会导致SQL语法错误。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic68BrKZsctFar4vtfna7UrYfenA0Wt3XXxEXJrTyLOpeFCFYH9MuslZibjyibJQND0uskph655SXhMePZicDW8ykQIlKlgP4MubY9w/640?wx_fmt=png&from=appmsg "")  
  
4.修改sortname=updatexml(1,1,1)  
  
分析：updatexml是MySQL中的XML函数，用于更新XML文档中的节点。  
  
传递非法参数会引发错误。  
  
测试原理：使用函数引发错误，并观察返回的错误信息。  
  
错误信息可能包含数据库用户、版本等敏感信息。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic68hfj4702VMKX3kDNnQzSeD4nDWia5CibKRzDXfmz8iboQria9k39nug22AGUqWNJmjkcKf3vXFH3nx9NQEGjtV7ofVBEKmqff23bE/640?wx_fmt=png&from=appmsg "")  
  
5.修改sortname=updatexml(1,0x7e)  
  
分析：0x7e是十六进制表示的波浪号（~）。  
  
传递非法的XML路径会引发错误。  
  
测试原理：利用非法参数引发错误，观察数据库返回的信息。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6ibaMGn7lq8EUAqJTfE6QH3bqVICQQErMp4a93Qsx1pK0KZ95XYC3PpgyTSCN11okVv2xFQibAtgc1mC8ia0dD6gKQ3TjdLb1b29w/640?wx_fmt=png&from=appmsg "")  
  
6.修改sortname=updatexml(1,concat(0x7e,user()),1)  
  
分析：concat函数用于连接字符串，这里连接~和当前数据库用户。  
  
引发错误时，返回的信息可能包含连接后的字符串。  
  
测试原理：通过触发错误并返回包含数据库用户的信息，以确认SQL注入的存在  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic68mgQWIRicryILEByKO8E4zZSVo78KPzDD8Sv0JZkJRofocMqov8fmtjaibnBOCtiaOR72GqsVHF4kibdfIM3ydkaYhAB5pj5hHFLU/640?wx_fmt=png&from=appmsg "")  
  
这里不太行猜测是user的问题  
  
7.修改sortname=updatexml(1,concat(0x7e,exp(1)),1)  
  
分析：这里使用concat连接~和exp(1)的结果。  
  
exp(1)是一个合法的数学表达式，但在updatexml中使用会引发错误。  
  
测试原理：通过触发错误观察返回的错误信息，利用数学函数确认SQL注入的存在。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6ic4NTNNBrHHiawAzhENUbibppkfg1O4ibR8eWESqhYiba0iarzuBjPJUFhthyoWLoHVrianw62sdrOqUvlpic9cYT6X1HLOG3p4Da7qWQ/640?wx_fmt=png&from=appmsg "")  
  
发现就是user的问题  
  
8.修改sortname=updatexml(1,concat(0x7e,database()),1)  
  
分析：concat连接~和当前数据库名。  
  
通过触发错误，返回的信息可能包含当前数据库名。  
  
测试原理：利用错误消息泄露数据库名，确认SQL注入的存在并获取数据库信息。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic68ic0IR2pq6zTaXxiaicmOmMpfS9IC232dah6TqWgFmLxqnjichtiaWLAlCqtqhgkH4bQHPtu9beBpO8sd9kTIWtsOzvbOerCRIP47o/640?wx_fmt=png&from=appmsg "")  
  
这里可以看到成功注出了数据库名,这危害不一下上升起来了吗hh  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic68uEbf7vRN55aev66YhGBVgJ9uBWFIib9KmNkZN27n6XeYLd6r6ZNPfpK5utGet7uhnFn06rxxwCg2jUobIU1Wu1gXeSBsHwT78/640?wx_fmt=png&from=appmsg "")  
### (3)越权  
  
前面测试弱口令的时候已经说了,拿到的弱口令是本校内的001队  
  
001/xxxxx  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6icZtuM4icC4NKujia672gYiaR4ib53CvB6dhE9lDPuMCXqJJTq2SP861UAosia8icEG9MZOqicIQgniaguUIxT4kOhfHbaNGQttrIia6dpY/640?wx_fmt=png&from=appmsg "")  
  
要是能越权岂不美哉!  
  
通过分析cookie:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic69Gnzdc6Jiasb2Jl51xNlurzm1FLaSvK6jjXMPzvibE2ARlBs1ic9ia6DxcKWAWPXjvAKfgVDTb1BuYphIChk2T4VSHFVEdsIah2qs/640?wx_fmt=png&from=appmsg "")  
  
发现  
  
yh_dm=008代表学校编号  
  
dw_id=001代表校内队伍编号  
#### 校内越权  
  
点击刷新抓包!  
##### !!注意!!  
  
多个数据包都要修改:dw_id=2  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6ibXVyGRfWTMibXIowHeJaD4fibS3zTeK48ic7gJdlzT52RSMlzZc5C55UzibAicJk4UtdsmK3Qk1SMZ2WdibxTplrBgGFYNcVvGYibibicM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69iaKI6HwahzQvPvqXJuw2Ipl2WAwsLkiaQZkvWCbxqXhUvGkicuzic8wk3Q1JjLicELWIKwEIbcx97YbqRUe1TYvfvFXRxLUKNmZaM/640?wx_fmt=png&from=appmsg "")  
  
成功越权至校内002队,并且可以进行修改密码!  
#### 校外越权  
  
类比校内越权:  
  
修改  
  
yh_dm=00x代表学校编号(经测试001-058都可以)  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic69uMBCkZFzCdMAaicw6ZoTyUd2vJh934WKK7ylTmPxiaqRhqM6ibD5W0nIok3EPrg4pibBeEf8R1IgJBjaIOsXKqsvicbqRWGtmPp7c/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6ic7bJvOhZj0Re4fShwkK53n2K3x01DtEvCnVB3rYLF2ia3uzye585aiczRAExCbIXnozRkCydv8cNCBuafA6vmlYyIHicQwI84cxQ/640?wx_fmt=png&from=appmsg "")  
  
发现成功越权至别的学校!  
  
漏洞危害直接由原本弱口令导致的20多所扩大至所有学校-->直接上交edusrc酷酷刷分了hh  
  
算是一次酣畅淋漓的供应链通杀教育漏洞弱口令、SQl注入、越权修改密码.  
  
![](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69uf30oqVervOmOeMawlldKnxt2zicwlAT1OQ0TPfiaiccACyodd7M5MLeicZh3Xn5LXIAWw4Ibd3WaarczxNFfPHaO6UYVZMAlyyU/640?wx_fmt=png&from=appmsg "")  
  
   
  
### 总结  
  
  
   
  
师傅们像这样的边缘资产可以说是数不胜数，这种思路普遍利用一下刷一波rank还是蛮可以的，注释师傅们早日拿到漏洞证书！！！  
  
  
