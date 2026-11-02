#  微信小程序漏洞挖掘详解｜抓包、逆向、反编译与实战技巧，SRC 挖掘可直接参考  
山河学安全
                    山河学安全  渗透安全HackTwo   2026-11-02 00:24  
  
**0x01 简介**  
  
不同于普通 Web 渗透，微信小程序抓包调试存在诸多限制。本文梳理完整小程序渗透测试链路，讲解两种流量代理抓包配置、DevTools 开启以及 wxapkg 反编译审计方法。重点剖析敏感信息泄露、sessionKey 越权登录两类高频漏洞，附带 Burp 插件利用方式，同时分享电脑端兼容、真机投屏等实战小技巧，可直接用于 SRC 场景。  
  
  
  
  
  
  
  
  
> 本文内容仅用于网络安全技术学习与交流，所有操作均在授权测试环境下完成。严禁将文中技术用于未授权检测与非法攻击，违者责任自负。  
  
  
现在只对常读和星标的公众号才展示大图推送，建议大家把**渗透安全HackTwo“设为星标”，否则可能就看不到了啦！**  
  
参考文章  
：  
```
https://xz.aliyun.com/news/17139
https://www.hacktwohub.com/
```  
  
**末尾可领取挖洞资料/加圈子 #渗透安全HackTwo**  
  
**0x02 正文详情**  
  
  
> SRC 挖掘、攻防演练中，微信小程序是很好的突破口。它和普通网页渗透有区别：无法直接抓包、调试困难，加密流量排查棘手，存在专属漏洞，还有各类访问限制。本文从抓包、逆向、漏洞、技巧四个方向讲解，小程序渗透底层和 Web 渗透一致，搞定这些难点就能正常开展测试。  
  
# 抓包篇  
## 方法一：ProxyPin+Burpsuite对小程序进行抓包  
  
proxypin是一个开源免费抓包工具，支持Windows、Mac、Android、IOS、Linux 全平台抓包。  
  
优势：  
  
脚本: 支持编写JavaScript脚本来处理请求或响应。  
  
请求重写: 支持重定向，支持替换请求或响应报文，也可以根据增则修改请求或或响应。  
  
请求屏蔽: 支持根据URL屏蔽请求，不让请求发送到服务器。  
  
①安装证书  
  
安装证书到本系统，选择“受信任的根证书颁发机构”  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAW5ia5BJStPGEADIbyxMukibgH50JnNYcOQzkwLslGFQGzVibic92eE8cMlzUbDMXDUibwCZj7IuC8Qmib9ibCMkvmRP31LwKJkx9iboqI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXedEdDMUcVgCQ2Whj85Iic78Sjt1N6Cn4Lw1vIRM7wVS3YcibMshvOeo2o9cSOlACNp1KIpc5PMJDHeicS1PC6mMlicKPuedfu6Jw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXncPa0J9F078dlEXV0ujvOh2MGcT2VS9mdgBGITQ8pLyzukkYgd0uMDuKtGgLXTMRTkYJJCNNuuBiarfCn7ib3SM9qkBfic9ngTU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUCcnqDqxfmTic7nB26ejaAKCRT4rsbDNSoibdo2B3rMRFSP0Dn2ktxicURxhW7fVMc2GFW9c6L3lDAKrRd6R8BbObaMQEjTgBfjc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXfGES1lIup0ax9PmaPJKdTGG4XJquHdvCOhQz21JNO1qPqTwO2IkrlIUgmBYsfEI6ZsNE2PccUdyQmu9WObuCsD09Rib5w2Ss4/640?wx_fmt=png&from=appmsg "")  
  
②成功抓包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAX5MhsFqTsnRdtftyBEMRhsLiaNgTspiakLbviabxc5YtmlPSnDJ2nSK7c7IP5ibvICxJLIunCpIsWib9czPNLsNNZNWib71OlFiagUk0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVR9QEb6FoOkrsATUzmBOEknMia6CXxQyHJQUZHVj8m6h5rIqgNKxTRg64xm0avqfiabt3icII3xXr95eJZKxxfvlRibItAXmpWGL8/640?wx_fmt=png&from=appmsg "")  
  
③将流量转发到burpsuite  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXFl6RSI0PSJdPfRGqic2RuhVTaWvmwaKlX72aJ0HPeQudfnBj325whQsTbNOugEBRMOyqDlokMgFWcWuUU0cS1GcFF41Eiau8yc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVr3HCtROmFYvdZrDsojvV9pTAwr66Vic6Q763lFl2ic5CwV1ERSHCnMNwGoiaKz7hlKwBQNkYCpcicgGJf88kgibic7m9GuvHchPRfA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAX7snALRYtBS5DXuyTia28h8GjSHiba9YT5BupU4I9BK47VkljiaibWqeGqJ6zxxmSeCab98WjBputHaN0brzNyR3wclCCibMianVGfI/640?wx_fmt=png&from=appmsg "")  
  
④不想被抓包的域名添加到代理域名黑名单后，将不会抓相关域名的数据包，可以减少发送到burpsuite的数据包，减少burpsuite处理的数据，提升速度，尤其是开了被动扫描插件的情况下。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXWaL2kibzoKOibeV9Et7b6oPg6dq6mMttOTTXiab9AaLe362NUO14Tv7iaGXG2NNVEvj9j9tPzHFFh39ofwQibJjFDTM3qgnaF5Qp0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXwZQCx74TSicxtlcDmqbt4icMtxPbRgEldwJicdVQF72GW1ribRTmfZohBR2BRq0kn7qhU7fWt4RMRS5aGydq2PD6IAicTQ1LdsVQo/640?wx_fmt=png&from=appmsg "")  
## 方法二：Proxifier+Burpsuite对小程序进行抓包  
  
Proxifier是一个功能强大的网络代理工具，可以进行流量转发，可自定义灵活度很高。  
  
①添加代理服务器  
  
代理端口需要与burpsuite端口保持一致  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVStpg0Jr70p6haFj4evcHHjcFE5b64hlKdfs1uzRbcbqoibHCibRJK48tefAP9anicvWciaYdr70MaTvw1O1etNd7QGuj8wa3Skg4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAX7GPS96JFBTvTVibLUTvdODWfWiaxhnbXXHpd0ax5gsx5LdkG1eEJHAhZtwEDcAltx8TedBicicTekemCiboBbBW0GWHzpJjAEgcK8/640?wx_fmt=png&from=appmsg "")  
  
②创建规则  
```
WeChatApp.exe;WechatBrowser.exe;WeChatAppEx.exe
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAX1sU4MklLc1t0PM1picjwjM1kgEmKP7GfKxtBjDugTclKHzKaH1QnCckuFuzoHGG1staEDYgtbuLSwvqzcNvNMicicaVlHQgz3h8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWRb3ibr3GkTpKAHNt5BAr0RjtcsRRu9fVn1FpbicbR1TwqrIs2XpLwUpg9rksXicRZ1FS3IyKXkF9NOlQibI9eiaJB2yGmnzkqEBXE/640?wx_fmt=png&from=appmsg "")  
  
③成功抓包  
  
如果配置完成后，无法抓包，将微信退出重新登陆后再进行尝试  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUOXLzmPNYwibYDzKEllYeFcMTuAtFn2icJT85cHFVDV69WZTdaTFictdDDlQUa9eialQZqaInAdDMCgKpsfrzYW9GHsUhFrvzh8X0/640?wx_fmt=png&from=appmsg "")  
# 逆向篇  
## 一、微信小程序开启devtools  
  
使用大佬写的WeChatOpenDevTools工具，即可一键开启devtools。建议使用小号，避免微信被封。不是每个微信版本都可以开启devtools，很多会开启失败，我这里为大家准备了一个可以使用的微信版本  
  
①安装依赖库  
```
pip3  install -r requirements.txt
```  
  
②开启微信小程序中的F12（devtools）  
  
直接运行以下命令  
```
python main.py -x
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWJAicUpXWRlZhjyrUEaNjkczhHiavCd1KoghPeX27oKDP6CAtoYaTCGbBNXgdQZ5pZwMkSZMrFicPk95LhHibjgMpxo7ia6KKMsibrg/640?wx_fmt=png&from=appmsg "")  
  
③打开要开启DevTools的小程序，即可自动hook开启DevTools（如果已经打开小程序，点击重新进入小程序也行）  
```
python main.py -c
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVuG6stdx9UbLWAMnia0ibSRlhz4ovktPGrrTRrtEnTSrdztZfS3JKmFY1fPMtA4j7P1ObjP4ibder61zXbH8Jk0ic38C6liaSrqK1E/640?wx_fmt=png&from=appmsg "")  
  
④开启微信自带浏览器中的F12（devtools）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWicJu5tgZDDvz4IJYzbq7LHbQjb0kWsa0X8mLVEGNxvicbz71bib47zKoCCpGpO4hGwrFxwFVHFBAMswKVtK1uFfPIlkQOSMvpBs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXvrw6z6f1iaibQIlWo8GTPibVLbtEVvtnPryicNw7pw6AibqSNWdic4xBiadj4cUib0TgRRU7BNbxVDc2dPpeAxT7XiaGUicYnp7DSURCec/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUKVQchoas5I2DqGeaZKqiaj3NmzfKMxfwVpaVduItujTzSibsbTxhn4AV0wOpJNFsBGsh00ooFW9equYdWfxqNFpuowVpvYUQeI/640?wx_fmt=png&from=appmsg "")  
  
开启只要三分钟，踩坑却花了三小时。开启操作本身很简单，但过程中的坑不少，下面说说我都遇到了哪些：  
1. 我这边用的是 Python 3.10。某些低版本 Python 可能装不上 requirements 里指定的 frida 16.1.11 依赖库，安装时会直接报错。如果图省事自己装一个低版本 frida，又可能引出其他兼容问题；不怕麻烦的话，可以逐个版本调试试试。  
  
1. 装过新版微信的电脑上直接覆盖安装老版本，是开不了 devtools 的。我的做法是用 Uninstall Tool 把新版微信彻底卸载（连带注册表一起清掉），再安装老版本就能正常开启了。推测是之前的覆盖安装或卸载残留导致的。  
  
1. 开启微信自带浏览器的 F12 时，如果提示"请退出所有微信实例后再执行该命令"，把微信完全退出后重试即可。  
  
## 二、微信小程序反编译  
  
①使用 wxapkg.exe scan 命令来扫描所有小程序。需要联网  
  
获取小程序的名称、路径、wxid（用于后续解密）等信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUB4f3GN9yicQlXgoDEK1kvCXFs7o8GWtQ1Ocee71gvkiaqu0pvc5v6FK8sMlkXiam6qkb0HYmdDkHzstOphDEj9s0YpYxx8klaQ8/640?wx_fmt=png&from=appmsg "")  
  
②使用键盘上下键选中想要处理的小程序，选中以后，点击enter键即可执行解密+解包  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUYVhHyFnCul8Zib1U3rTRMZhKQWbCAIMhss3ibia0kKw7NYx0lRcxWoJvwFfVfNyL5MbLiah1Br1ZtWkpbtfBAOibU2HHiamDMxPYQk/640?wx_fmt=png&from=appmsg "")  
  
③如果这里没有看到你要反编译的小程序，可以将要测试的程序添加到自己的小程序，然后再重新进入查看。添加以后，进入小程序查看，如果还没有发现要测试的小程序，点击重新进入小程序然后再查看  
  
④反编译成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAU8j3CMjiau5QFd6Ts0z8Z8tsrXv8Uj0s0or0lpzPxic54lRjnqfmMSoDPXBWEN1BQm8mM9ibBhadhJQcQUE9hQQO4kAJvibbnd0nY/640?wx_fmt=png&from=appmsg "")  
  
⑤反编译成功后的文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXnQQxuDaP9Gg0ArAiczRZ2qYjqq79g2xicwUjqmqEkHsGbKpgfyO0VFE634T8rSTH2lTBg2E1PicB8wRf9J1ouqHE4VibHp9fE5w8/640?wx_fmt=png&from=appmsg "")  
  
# 漏洞篇  
## 一、敏感信息泄露  
  
反编译得到小程序源码后，就可以检查里面是否硬编码了 AccessKey、身份证号、手机号、密码等敏感信息。不过手工逐文件翻找效率太低，所以我写了个脚本来自动提取（能用，但不算规范）——它可以直接从 JS 文件中批量匹配并输出这些敏感信息。  
①首先筛选出反编译后的js文件  
  
我这里使用的everything进行快速筛选，首先打开全字匹配  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXWO7MZHBNmGNaBfibXmhmxIibaFhpibaBITUsqLwwqrZQd0D5MDzA3kBA94L61ticjgzSgqqOxXEePyURh24oSNuV5rRQdFficmnibQ/640?wx_fmt=png&from=appmsg "")  
  
然后使用查询语句筛选出js文件  
```
path:反编译后的目录 .js
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVRmm0jQANDd7CibicBeMtNUJTM7dWveSgI2CPLLkWrzYfsKGSjtGOTXa3hIlXSWIZPaAYOr9rxzZ3JvRck407AJ6YEYticsEMKyA/640?wx_fmt=png&from=appmsg "")  
  
②将筛选出的js文件复制出来放到一个目录中，使用脚本进行敏感信息查询  
```
python secertinfo.py js文件所在目录
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAX2YA1UBpQWCOpnCcPEV8V5JbmMu0ibZs4NHIhswpauSIqQKGPpPHgVktGeNXQdvs57V3BVg8uqUcvrFZNVBrJa4et90icRJrPMU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVZ3nS2ZzMsMTP1JOevqxM968OWg4BhwfbVyZiaGeTRI5EfTVb3ib90bJ5XU1ibzXZV64jGzsI0dCVpgCBcVQ9qaIwQtu9eAgDhGc/640?wx_fmt=png&from=appmsg "")  
  
③获取到的手机号、密码、ip等可用于后续渗透测试。  
## 二、任意用户登录  
  
微信小程序如果依赖开发者后台对开放数据进行校验和解密，可能埋下任意用户登录的隐患。当数据包中泄露了 session_key 和 iv 时，可以借助 AppletPentester 插件对数据包进行解密和重加密：把手机号（或其他用户信息）替换成任意目标值后再提交，服务端校验通过，就能以任意用户身份登录。  
①如何快速发现数据包中的sessionKey，从而进行后续的判断  
  
这里使用burpsuite的hae插件规则进行匹配  
```
（sessionKey）
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWtiaFjh3avFriaRkocVDBFpKiacHDfmEFBjmQhOeZCFbicQLCSSQcrWxUT7ciatnap8lRyOQZBvr9X0GclE2icRpniaIQia5LrXqWoT6o/640?wx_fmt=png&from=appmsg "")  
  
②获取到sessionKey和iv以后如何对数据包进行加解密  
  
这里使用burpsuite的AppletPentester插件进行加解密  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWalmW0umEozIad6PjzhmEvutNAadwvNHS8CjpugI7YnRZEjx9iazLibvvdTDR8wvpWRS4yxiaNThXSjIQoycSbKbKrIbh6l9zMaw/640?wx_fmt=png&from=appmsg "")  
# 技巧篇  
  
测试微信小程序可能会遇到很多奇奇怪怪的问题，这里做一下记录，希望对师傅们有所帮助，当然你们遇不到更好。  
## 一、解决功能点无法加载的问题  
  
有些小程序在手机端打开正常，但是电脑端打开一直显示在加载，或者某些功能点点击没反应  
  
我们可以在手机上点击要测试的功能点，然后复制链接，再从电脑端进去，就可以解决这个问题（注意是复制链接，分享小程序不可以）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXzptb48QV5Bs4wsUJ1uvsqjv8Z99CGkDBm9AVwIueuRos4C9jRicoKyQuoNx4FYqSKC032YicycuXUJrl1WC0NdOt4WZYTmgcibQ/640?wx_fmt=png&from=appmsg "")  
  
链接是这种形式的  
```
#小程序://中国电信App/8zsyCcxJscGz0QA
```  
## 二、微信小程序只能手机打开  
  
有些小程序直接用电脑打开会提示"请在手机上使用"。这时可以先在手机端打开一次该小程序，然后在电脑端微信的"最近使用"小程序列表里就会出现它，从列表中点进去即可正常打开，绕过这个限制。  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAU5U7ibWwBVeTzklMpKW8DHibtOZxnRgC7gxQmMia3WeqsrlfmicjVauWR7pic5ic9qSBpDiaHWjMLHXZ5ibrPTEDzKulKBqmpw9QevG4o/640?wx_fmt=png&from=appmsg "")  
## 三、如何更便捷的进行真机测试  
  
测试越权漏洞时往往需要用两个微信同时验证。可以选电脑加模拟器，但我习惯用电脑加真机。真机测试的痛点是要在手机和电脑两块屏幕之间来回切换，很不方便，所以我一般把手机投屏到电脑，测试全程只看一块屏幕就够了。  
  
我用的投屏工具是 QtScrcpy：先用 USB 把手机连上电脑，识别到设备后选择"一键 WiFi 连接"（我比较喜欢这个模式），连接成功后拔掉 USB，手机放一边，就能直接在电脑上完成真机操作了。对于那些带模拟器检测的 App，用真机投屏这种方式也能正常测试，挺好用的。  
# 相关工具末尾下载  
  
  
## 0x03 总结在SRC 挖洞和攻防演练里，小程序可是个宝藏突破口！它不像普通网页那么好拿捏，抓包、调试处处是坑，加密流量让人头大，还有专属漏洞和各种限制卡你。本文分抓包、逆向、漏洞、技巧四块唠干货，其实小程序渗透底子和 Web 差不多，搞定这些拦路虎，咱们就能愉快挖洞！🔥喜欢这类文章或挖掘SRC技巧文章师傅可以点赞转发支持一下谢谢！内部星球VIP介绍V1.5（更多未公开挖洞技术欢迎加入星球）如果你想学习更多另类渗透SRC/AI挖洞技术/攻防/免杀/应急溯源/赏金赚取/工作内推，欢迎加入我们内部星球可获得内部工具字典和享受内部资源/内部群🔥🚀1.每周更新1day/0day漏洞刷分上分，目前已更新至23794+;🧰2.包含网上的各种付费工具/各种Burp漏洞检测插件/fuzz字典/SRC挖洞SKILL等等;3.Fofa/Hunter/Ctfshow/360Quake/Shadon/零零信安/Zoomeye各种账号VIP会员共享等等;🎥5.最新SRC挖洞文库/红队/代审/免杀/逆向视频资源等等;🧪6.内部自动化漏扫赚赏金捡洞工具，免杀CS/Webshell工具等等;💡7.漏洞报告文库、利用AI挖洞技巧、共享SRC漏洞报告学习挖洞技巧；🎯6.最新0Day1Day漏洞POC/EXP分享地址（同步更新）;https://t.zsxq.com/jVcxV(全网最新最完整的漏洞库)🔥7.详情直接点击下方链接进入了解，后台回复" 星球 "获取优惠先到先得！后续资源会更丰富在加入还是低价！（即将涨价）以上仅介绍部分内容还没完！点击下方地址全面了解👇🏻👉点击了解加入-->>2026内部VIP星球福利介绍V1.5版本-1day/0day漏洞库及内部资源更新结尾免责声明获取方法回复“20261102"获取文章涉及的全部资源工具回复“渗透字典" 获取 一些字典已重新划分处理（需要内部专属fuzz字典可加入星球获取，内部字典多年积累整理好用！持续整理中！）回复“书籍" 获取 网络安全相关经典书籍电子版pdf最后必看    文章中的案例或工具仅面向合法授权的企业安全建设行为，如您需要测试内容的可用性，请自行搭建靶机环境，勿用于非法行为。如用于其他用途，由使用者承担全部法律及连带责任，与作者和本公众号无关。本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用。如您在使用本工具或阅读文章的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具或文章或来源于网络，若有侵权请联系作者删除，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！往期推荐1.内部VIP知识星球福利介绍V1.5版本0day推送2.最新Nessus2026.2.9版本下载3.最新BurpSuite2026.1.1专业版下载4.最新xray1.9.11高级版下载Windows/Linux5.最新HCL AppScan_Standard_10.9.1下载渗透安全HackTwo微信号：关注公众号获取后台回复星球加入：知识星球扫码关注 了解更多上一篇文章：Nacos配置文件攻防思路总结|揭秘Nacos被低估的攻击面喜欢的师傅可以点赞转发支持一下  
  
  
  
