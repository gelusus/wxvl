#  AI赋能漏洞挖掘案例分享1-某系统从sql到登录绕过再到后台getshell  
原创 陌笙
                    陌笙  陌笙不太懂安全   2026-10-10 09:11  
  
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
  
  
  
**后台回复加群加入交流群**  
  
****  
**广告：********cisp pte/pts &nisp1级2级低价报考**  
  
  
**陌笙安全纷传圈子+陌笙src挖掘知识库+陌笙安全漏洞库+陌笙安全面试题库**  
**简单介绍****（**  
**加入纷传圈子**  
**送****知识库+漏洞库+面试题库****）**  
                          
  
如果觉得合适可以加入,圈子目前价格  
39.9元，价格只会根据圈子内容和圈子人数进行上调，不会下跌。。。    
  
  
**圈子福利**  
   
  
**edu漏洞挖掘1v1指导出洞**  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKQWHxLsRrPqpqdiceX76d7yExQIyOqFmmJAfHQh7qzKvPc2V5z6iaa0RY6Ib8AsGvgS5MKkAk5aaHnJBaSnI10LDKQYMLcQMmg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR8pnPeapLBK4Jsa4ufCvFoGL66t7PKeZyA3AjNxsObjtnCibN2gzGX7NMS7Wo5sj3YYL2iboeRuQDcWqiapc8xuo5fticoBG4DsyY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKIBNIQIVicRWJLbyGRmg92vPzc8375PJpcYVvfywzwqnaeBicZuEbfvuic9KRdjwkahSDic5VqrH2Mb4NkqtkADl5HLIh8gPex60/640?wx_fmt=png&from=appmsg "")  
  
**skill+grok辅助挖掘某企业sr**  
**c****实战效果，能出但是重复多，agent独立挖掘也可以，见仁见智，看个人习惯，好的模型是最重要的。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQwyn779TTwY7vZkePQCL8k3K8dYxNdyzfgADL4dJcNUvpmodLeDVCZ6xDC4RJXEBmO2tcWqgUNdTVicKTW0jpdsg7X6xwP5gIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQnIqagDL2A4BUIXrib9YVmATWuaIDqETqYd9ToHib52mDyoMyqc6Wzh733FRnbsDsGgey7B8s8jr72UtkPY6ich58niaPJqoItKcE/640?wx_fmt=png&from=appmsg "")  
  
****  
**企业src边缘&核心资产实战效果&&有重复但是证明好模型+AI确实够用**  
  
**（图片仅供参考，我出不等于你出，见识到ai神力即可，多去用AI!!!）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT1YNI9U6r6NMO4UMUBROoWeC4XQC4Dge94ODZ7tXY6tbxqb3IJoghve0u1SfkygE5UJ5HTdUBvLZKrn5ps13F71piax5dlHnOE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTGjdxKy4nllaLXIznTvRqicichITccuB8psYFRYakw6ViauCk6iccziahfPw4fnrqhyCp7Zkq7lRI0DOicyrZlNyicYibTVibfGFZVaG0c/640?wx_fmt=png&from=appmsg "")  
  
****  
**不是P图,单洞1.2w记录**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQQpQdR2Ttwqxibyr75Is0kBG2N2tLYQIaau7SS278oyQ4RDpNScviaMt4wtlfgDCibE05WgoMhE5kZUrP8ciaYIdnxA594wsmoAAs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSRn2EsfFkA5mG6dcn7JLQMroc2dy3EQb3ueY2Cspd0WYgicXEnSF68UD43nNd4plkxmkTpEOh2kkQMEWZZIjE0ibA8r1q4IfiaxI/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙src挖掘知识库介绍（内容持续更新中!!!)**  
```
信息收集(主域名信息收集,子域名信息收集等&会永久提供fofa-key助力)
弱口令漏洞&未授权访问漏洞挖掘
任意文件读取&删除&下载&上传漏洞
sql注入漏洞
url重定向漏洞
csrf&ssrf漏洞挖掘
XSS&XXE漏洞挖掘等等常见漏洞
cors&目录遍历&越权漏洞挖掘
EDUSRC(证书站挖掘案例分享&edusrc挖掘技巧分享)
CNVD挖掘技巧分享&实战案例报告编写
公益漏洞挖掘（公益src挖掘漏洞分享&提供补天1权重资产）
SRC挖掘实战(针对各种常见功能总结的常见测试思路等快速提升)
经典常见Nday漏洞(常见中间件&以及各种常见框架)复现
云安全相关漏洞挖掘（云key扫盲&云存储桶&快速识别云环境&云攻防）
AI相关学习（AI基础&AI代码审计实战测试&webLLM攻击等）
APP&小程序漏洞挖掘
等各模块不在一一介绍
```  
  
信息收集  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTu9DGyTubluhYicFynwVBKa4V06sDfEVKOyk5Q4ghZzLMDAuLb1M1oR4RJumGWrADPapFjTrOjpksKQ8q0YYCnl3ZWLof8Knzg/640?wx_fmt=png&from=appmsg "")  
  
src挖掘基础  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboR45bibbJEb28a1gS5yth3r5HyOsgPiaOUHHYriahZyIyrk0LMOsHW4VoDibyBRibTNzptGiaLWX62UwykicwvbxCJPopvklqiaxML8lS8/640?wx_fmt=png&from=appmsg "")  
  
src挖掘实战  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTKWnTsN6CXf3djhXIlMKNRjVmJn3g5b23ur9E6Cx3O68f0hXVjCiaj8J4RYeTGBecqf1k99phG0ice2wtd5lKgR46OeqeLQfMpk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRbZ1HYm7R7YEiaxRVQibGWyricx9l7HpGjS4ZfWRdlft8iacwkpzYyZfmYEkWdJgYRORPkNFR6dADR5MyE524tWX6cAwN8MmrCZu0/640?wx_fmt=png&from=appmsg "")  
  
  
edusrc  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQVVlTXhibjR8UiakZBQicXRZrQ7hdoOz5G8MQrcuDBGbqJdO0kIz6R9IU4ObAeOiabT8pr6lc7jibdIkKoTjiaXNHPLAwAB3BV2UvLM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSCrvarBbzP4L9kS6P0LVH9JMdmcbFDKiaicHqMFgTxq3x4iatjDJQicmc7NPC14C9Fk3icFjrouSgNVaN8Byuf0C0Iq9O6D1XPvFvY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRALwXmgZ4mh2LW0RdicrKjBCP7P1iaF14G0Eq2v3KRnTJORpwXZlF58WEz6QicxLJpyJaA5iah5CF2rHjBz4JzOELFRaZTAKOQ2tQ/640?wx_fmt=png&from=appmsg "")  
  
经典nday复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRu8Gf849iaCkSBxLL8IlzJTRs185QicEe9l5UGI1dEVKISt2IGGveZynXBW9tIUsxNsz4adSTib7rib50uSJdjNfTvVRFrbPJhzL4/640?wx_fmt=png&from=appmsg "")  
  
  
云安全&AI安全  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQ9qiavETNjaaX162czpNCqpw3uJqVpicbI15AXzhf5x8icmHxBdTGOgRgzNPGF3Aw2gglT4Fx09JGXYibQC6U7CQKVmoH08l3meia4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQgcsjiaZ4S26TWowHfpBkhSeHf2pjrcDyicJuia3uqvRBauLEOicibibEMqibnBMtjopFL8No7UXNibbURvzeJ3dQHTibvGxRQGnorb4co/640?wx_fmt=png&from=appmsg "")  
  
**陌笙安全漏洞库介绍**  
```
最新漏洞查看
1day&0day分享
EDU学校相关漏洞
Web应用漏洞
CMS漏洞
OA产品漏洞
中间件漏洞
云安全漏洞
人工智能漏洞
其他漏洞
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTFBRMa8XwYxfcZMyXicx94xSKxawPcqFia2rJKOL7fSLYXiccwHc868XxNGIQ5z7ibiaI1MNAGRrK7U6wXJTsZOCAu2I5XV1boTAL4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSBzdakI9XI33ReAm2dxO8vgzw3JicQmUuWCb5ayBlKR1PoQHEHFETteBnicyupwU0mXvXibfrDoyg8nSWBGoK1p2YXY3ElhcvOQ0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSajCclDhuRpaLic9Ld915CHU7RqSC1LCrPGfNZiavdPEVeDedDWOPBhtMLCicTp3RNd1lT0Pmfo3mx5B0hUxbQg3ic6Via90NMtZVk/640?wx_fmt=png&from=appmsg "")  
  
****  
****  
**陌笙安全面试库**  
```
渗透测试基本问题一汇总
渗透测试基本问题二汇总
渗透测试基本问题三汇总
微步护网面试题目
长亭科技面试
深信服护网面试
启明星辰渗透测试面试题目
安恒面试题目
绿盟笔试题目
360面试
奇安信护网面试
运维面试题目
运维面试题库
网安面试相关文档大全
相关面试文章推荐
等等
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSoLqEzH0a3A4LQrvTIkGx81Sh5pf6fCoEQJhYg715vrJicSkfBuCoAmV2Kp4uOMe5jcUZutPwicibFibtJ1ZmyiaAibCg0XicWnsNcicE/640?wx_fmt=png&from=appmsg "")  
  
****  
**POC库****&&更新适配afrog&&nuclei&&dddd的POC&1day/Nday等&&**  
**dddd二开****工具[助力渗透测试&&红蓝攻防]**  
  
**工具截图**  
  
****  
**实战效果**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSKqLXNcOPE07xOwOUCjRGuFphopPumW9RaticmNCuEUXu52GtdTTfpTUicrBj80kMcZzJsnps3abyvXIvLHEIhvMoXUApOqZCe4/640?wx_fmt=png&from=appmsg "")  
  
****  
**poc库【后续持续更新】**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTG9Lyp44aFffUOxQKtHjToGfqFWTjswYft0VtAPINtV5MqmrTTj8GWrVb6yowvHURubPgOqdribmibWEb0Fcj3YdN4iahUwItcxE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRAMJIIvexOOJa5KhrsKmlsx8bkwib9SPoK72Q0OSPWR5qx67yvl8scMQ5bg8caBXZH01kM39RDnKpnWSaTicgobRmLygERGFWls/640?wx_fmt=png&from=appmsg "")  
  
  
**AI赋能-**  
**skill辅助**  
**漏洞挖掘（免责&&慎用）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR3Dib0RVxVhUOzS6ibC6BvkfulXQAclic0XCXMS35C4EPoqX1b2eMVj2CFiaLCelVs1szGibaHiaAq7WibRdwHUg0IwO8fjDdWxNv6eY/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSXZZVels2NibmHgyxntlCRNIkgoqMPfUPwSM9O43OqniaZLDEJic9QRkW01gNTydFkibdI6yBRkJJ1sDUmfl7iaicoibz1QLp0J2pWE4/640?wx_fmt=png&from=appmsg "")  
  
  
**圈友skill+ai辅助渗透**  
**实战效果**  
**，支持打假！**  
  
证书站  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQibuxKAyHBZicB1t5yGVKyV82Teo8C2MbjKPytKziaXUcjPiao8ylHbD4vicAld8equC9alic3NksvWJ09wArXaPXZD10vjPtfoia4Vg/640?wx_fmt=png&from=appmsg "")  
  
普通站点  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ1Xt6gdLgd2d1mf8QURQX4YZjHA2uIw9GdTuxzSafBVOJzrQVmHJlqdhVWdVDj3OQsiaQhYOaoiabXc6EgajvBMvB6xBZwdJkIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙**  
**纷传****圈子介**  
**绍**  
```
1、src挖掘思维导图，信息收集思维导图，edusrc挖掘思维导图，以及后续的红队&面试思维导图&自己网安笔记等持续更新
2、2025-2026的edusrc实战报告包含证书站和非证书站以及2025之前的各种优质报思路分享
3、各种src报告思路分享（内部&外部）
4、分享各种src挖掘&edusrc挖掘培训资料&视频
5、不定期分享通杀、0day
6、有圈子群可以技术交流以及不定期抽取证书&免费rank
7.分享各种护网资料各家安全厂商讲解视频&精选实战面试题目
8、各种框架漏洞技巧分享
9、各种源码分享（泛微、正方系统、用友等）
10、漏洞挖掘工具&信息收集工具&内网渗透免杀等网安工具分享
11、各种ctf资料以及题目分享
12、cnvd挖掘技巧&CNVD资产&src资产分享&补天1权重资产分享&fofakey共用
13、免杀、逆向、红队攻内网防渗透等课程分享
14、漏洞库&字典以各种内容不在一一说明
15、cisp-pte/pts&nisp一级&nisp二级&edusrc证书内部价格
15、如果有漏洞挖掘问题或者工具资料需求可以找群主(尽量满足)
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSY2pbvbP3qGAlW8O43bRvAISCxZm4UDTRsaMVbJKTsjfTMTDlq6qNBcVs4tkl4UzgqGz5ag81baU1rusKE09J9T6cMVliaibibwQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/MSDUaqtwboTrLRQpTicOR7bzyNiajiapVJgyMiaYlEDBVU87YXMnanOFWsCYN3cCVGsKkibzV9dMryvbFXBb4Z3472ib27RJ1Xq1HnKJIp5u49GYQ/640?wx_fmt=jpeg&from=appmsg "")  
  
**目前800多条内容，扫描下方二维码查看详情以及加入圈子，持续更新中。。**  
  
**如果觉得合适可以加入，价格不定期会根据圈子内容和圈子人数进行上调。。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRngNCK3ae5W1nw179icr078zj1eBrn2aP1FVic1yJHdvbJ1sIKcZzNKlhXsbTfcUYdo0miblHoJpdcfG10cZfp6HbbHQfkfiajU40/640?wx_fmt=png&from=appmsg "")  
  
