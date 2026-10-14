#  AI赋能漏洞挖掘案例分享1-某系统从sql到登录绕过再到后台getshell  
 GG安全   2026-10-14 00:39  
  
免责声明  
```
由于传播、利用本公众号所提供的信息而造成
的任何直接或者间接的后果及损失，均由使用
者本人负责，公众号陌笙不太懂安全及作者不
为此承担任何责任，一旦造成后果请自行承担！
如有侵权烦请告知，我们会立即删除并致歉，谢谢！
```  
  
漏洞挖掘  
  
某次渗透测试过程中，发现这个系统，看起来就老的一批  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTX0hYgzv9CiaE8olLDszNia9AC7kQF3LfckMUy1zQnml6upfESOyr5dFFWckYLJLTblVFXnyJVmK9SmYvKriaxumT1Or8CYpia0ho/640?wx_fmt=png&from=appmsg "")  
  
简单看看架构，框架是thinkphp  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQVjMkZekCUmj7pJzGnWmial0Myibia87MYppPtsshfibZPmJ4hVznT9FQricrOEOEeuz6Gic6x6iaJmWaNanLwicpY8f7xSBmZS0zJQ4A/640?wx_fmt=png&from=appmsg "")  
  
直接先打一手nday  
  
多用两个工具试试，有可能一个工具，poc不够全，测试的也不够准  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQBkqAicibvyrXwQsf3PXm3XVA42YXHCIcfiakPSsZP0CY0Cm5pDI9m1EGVGnxCWjMkTUvrOcicp7LkYXZNEQVwdLrFIXEuUrwNlibc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRyMCtmt5M9gxM2Lsw1yNuLegP1oKRpIFVWLofFzbg5UevTA6t7jIdbexI52PTEvO8wEFITdnd0ic1zs9FUfgADvD1DSYtB78nw/640?wx_fmt=png&from=appmsg "")  
  
也可以用afrog试试  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRF9IfR3NFABPn7ibIIdcuuV8aEV2BmUUeNv4yhXhxkCfuCyGHVLfym3qtV5zPMqFyg6kLnBBibHt6OicOVQCXGw5tuQwK5vN5q3Q/640?wx_fmt=png&from=appmsg "")  
  
这里我试了，没有tp的nday  
  
回归系统本身，雪瞳这里直接0接口  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSZscoiahInOxaluILF7A5zD6BoPoObhb1yAVVjDeQb2ubqpaCIyKVcia1m5Jr2aicuLNicjhPJptXUVRWnopOKm9rHFwiaLTwL8JHM/640?wx_fmt=png&from=appmsg "")  
  
直接抓包看看登录接口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboS5WvkbpyucNPLqNlOibYrKmh6Evc0FFPgqckKa6y7F3SNRwibY40HFB3U27dYLYleC8CPIa9WcnSZrf953hNeHQ48kMgDMjyay0/640?wx_fmt=png&from=appmsg "")  
  
这里可以看到这个图形验证码，是前端校验，只要输入正确之后  
  
后面就不会有这个图形验证码的限制，所以可以直接爆破弱口令  
  
这个图形验证码前端检测，如果是项目上的话，本身也可以水一个低危漏洞，更神奇的是，某些src竟然也收，哈哈  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRgXBLAic0AYMTj6y6JOHsibOzJq3hpAgb6FpxYTIbhCQon6ib11FMrvsrzVDsFUfRH5Xicfuia5RotQN5aZQl570PBdLESzN1D4MlM/640?wx_fmt=png&from=appmsg "")  
  
我这里不爆破了，直接让AI搞，其他的一些图形验证码思路可以参考这个  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTakkRIVHBNuciahyM1BHnrLNsmicZr7DzH5yo3dTeoS2U4I78h4EZ8KvqjRL9dhkWYYccX4ha7QBsvSX8iasNibWoIyPQ4AecHV2Y/640?wx_fmt=png&from=appmsg "")  
  
然后ai开始库库发力  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSjaZf8A5L54QR6IMyFVQhibLGgKAU983Y9x8lgv6M1tNdLUFJPpSdZib9yESAuK22uFFqBfTNnf7u2Y3VrRaAVibeHZias9lcU6PI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTqLg20trh3mm3tbHzliczAp3ibC1NibfMssjvUtxzmZHfX527DN12ZXwYIHhWeRZs3aPp4BEuN7JpVCNXe9we6UETMj8jcTMANgA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSAGquJM6SXicUmSiaS7cR6ELXKOzfHlXGtdFspbC459JWNjBPN2zibTaOy6RH7k3npjibZm5bHybdJX2zgxicQbjWYHRARsFAprF5s/640?wx_fmt=png&from=appmsg "")  
  
几分钟后报告出来了，我们开始复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRYX8Eb7ac5ic3NWXGzh9scGoOG6hibDbX5laUsNUMoGtoAFEzANsWrhiaF5pOIv5xjp1nMH0N2Gu7kXsVmpDzCoicRl0lic7icy2cM8/640?wx_fmt=png&from=appmsg "")  
  
还是复现爽，复现几个危害高的，顺便看看他是怎么打的  
  
ai打出来一个sql注入导致的登录绕过,其实这个我是试过的，可惜不太行  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSbZWuKJcOcf4xP3f6A7WE0EIGaB8ZebNzAicYqdD1k4eqQ6uHJ6OjkyZrDGjVrSuHCLxOVegFmuM0XT11r9ibxn3fkFW0dicdfKU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRcbiccdPqJXy2eNZ6350foI9L9T2hO8xhhPeiaCXKv2x2qPwicj6naqTYGczwic7EDoo1EFOIe8BcxsUdJh6x5icVHQ0wyDeQRsNYU/640?wx_fmt=png&from=appmsg "")  
  
ai打出来的思路是  
  
ThinkPHP 3.2.3 中因将用户可控的 account 参数以数组形式直接传入 where() 条件，攻击者通过 account[0]=exp&account[1]=or 1=1-- - 触发框架表达式查询分支，把任意 SQL 片段原样拼接进登录查询，从而注释掉口令校验实现免密登录（可登录管理员或指定 uid 用户），并借助 Debug 模式报错回显结合 extractvalue() 等报错注入外带数据库版本、账号、库名、表名及用户明文口令，形成「未授权 → 任意用户登录 → 全库数据泄露」的完整攻击链，属于可直接入侵系统的入口级 SQL 注入漏洞。  
  
我们简单复现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQXlsaHMPMx14icWfXpS25szTQDnf1ARmub88W4Lr6x8J4c0Bsic3JoBluYyriaLHtJHpH1XyOZC2GyKgVnQNj6J4RElJ36rvxNKE/640?wx_fmt=png&from=appmsg "")  
  
首先正常抓包，然后把请求体，替换成这个，放包  
```
account[0]=exp&account[1]=or+1=1--+-&password=x
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT4Yru1pXaj5CiaicRpIVGxRLwCiaiaujs2Pt8KXSXn0ibwgaiaHjFllGv76ib75rNkwm7Bm1lVFFgVibqtjwFDZvsVbAHQtAbYM2v4qJA/640?wx_fmt=png&from=appmsg "")  
  
直接以管理员的身份进入后台  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ52sDPiab4Yoqrof8mA9aAUUzFmOaehjv82GGoibjhSwIrsLbFM55NohTG23UUTmCwLcW3O8QeGjEz9WIUYUQPMecvQ7PLUJiaUc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTia2c90BR0mU5tarKlBpU3SskibvmljNjmpB3Sp6upQxK6XWiceSUCgN8UIFIfD9VXiafJ5ekqk5D3EbupPoicfH49JXjuLhzkHhcg/640?wx_fmt=png&from=appmsg "")  
  
问了一下ai，为啥登录框的万能密码，不可以直接干进去  
  
解释是  
  
输入框里直接打字打不进去——我补测了账号框和密码框的所有字符串型 payload（admin'、admin'#、密码框 x' or '1'='1、' or 1=1-- -），全部返回"用户名或密码错误"，无 SQL 报错。原因：这个注入点要求 account 参数以数组形式提交（account[0]=exp），普通表单永远只会发出 account=<字符串> 这种形式，字符串路径会被转义，而表单不可能产生数组参数——所以必须改参数结构，不是改输入内容。  
  
这个框本身也有sql注入漏洞  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTnPiby9ic3JTtcIqmyBUHEuJ0icT4z3qCicuF2ZlnwHf53DVJKZO3FquXgGdyvka1YaGLC6QdjmUh8KAxqAicKo1ec2teyVu0pRgMA/640?wx_fmt=png&from=appmsg "")  
  
继续深入测试，ai又发现  
  
站点在静态资源目录下遗留了一整套第三方 jQuery 上传组件演示程序（路径   
/Lib/uploadFile/  
，组件为 CreativeDream/php-uploader 0.2，目录内   
readme.txt  
 自述出处），其中处理上传的   
/Lib/uploadFile/php/upload.php  
 未做任何身份校验、也未限制文件扩展名，实测在完全不带 Cookie 的情况下以 multipart 字段   
files[]  
 提交一个内容为   
<?php echo "SEC_TEST_RCE_EVIDENCE_9d31"; ?>  
 的 PHP 文件即可上传成功，服务端响应会回显保存路径为   
../uploads/sec_test_rce2.p.php  
（保存目录   
/Lib/uploadFile/uploads/  
 可通过目录列表公开访问）；随后直接请求该文件   
GET /Lib/uploadFile/uploads/sec_test_rce2.p.php  
，服务端返回体为   
SEC_TEST_RCE_EVIDENCE_9d31  
（而非 PHP 源码文本），证明 Apache 的 PHP 处理器在   
/Lib/uploadFile/uploads/  
 目录下真实执行了上传的 PHP 代码，即攻击者在无任何账号的情况下获得服务器端任意代码执行能力，实测完成后已通过同目录的   
remove_file.php  
 将测试文件删除并验证（404）。该上传点同时允许任意扩展名与任意内容，实际影响等同直接获得服务器控制权（可写入任意 webshell、读取数据库配置、横向渗透内网）。  
  
继续复现，我这里直接搞个phpinfo,不传马子了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRuvvSt1LnZQalGKlXbNO3x2jKBwhqXv9mkzn3Gy4S6fzl5UnLEiangj6N6fIN2WgzytEXiaMy0pkibSwGolFREEOYa69eVnAoTqk/640?wx_fmt=png&from=appmsg "")  
  
拼接路径进行访问，直接getshell，写报告，跑路跑路  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQpknnFPmz8jb8iaOuTibwQ4CAKk3Xjq82MuBMMYnNLwwxBg59PLfYY8rb01naVialib424B7J9m11njC2NN892KUyMAJZbmJGknUQ/640?wx_fmt=png&from=appmsg "")  
  
太高效了，其实根本，不用手动测试，简单确认一下，这个站点的价值  
  
如果有价值，直接ai梭哈就行，ai都干了，那我？？？  
  
  
**1**  
►  
  
**福利放送**  
  
    再次声明：本公众号及其发布的内容的使用者需自行承担由此产生的任何直接或间接的后果和损失，GG安全公众号和原文章作者不承担任何责任。  
  
   
  
  
欢迎加入我们的安全技术交流群，  
后台回复  
进群  
加入交流群  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ONkxMdpEUyic0icTQ8TBWric1q7Z2cpocAalvBqjtHTZbemBNd0ge4fibbDeG3Ht9le7ZVvRFv7qjF26CpzgfgE2CKupicohk0UjrI9ibMppB86zI/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=16 "")  
  
  
  
**edusrc邀请码 | 无问AI 积分兑换码**  
  
    免费不限量提供edusrc邀请码及  
玄机邀请码  
，可在的菜单栏  
资源获取-edusrc邀请码   
| 无问AI 积分兑换码  
中获取。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ONkxMdpEUy96qfCnuOkeGwVjGiaLeKCYWKIrPbicMrIobriaT5RRhm7ydHpiaibJBL5AhZqlIdNVhCGkGsOaKDia9ibiaD8q1WCGbhfAXNUw7ZFnkHc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=17 "")  
  
****  
**无问AI 积分兑换**  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ONkxMdpEUy9CGAfOa0NFZNsNE6gBN6iabv1IobeCxOgicJP0hAgEgiadPOC9DGeMoSGO9cxg0FibiahfrUg5onqxEouDZL6gxU7ZJYmLKyG3Ekro/640?wx_fmt=jpeg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=18 "")  
  
  
  
广告：  
nisp1级2级低价报考  
  
  
  
**2**  
►  
  
**往期精彩**  
  
```
```  
  
  
  
