#  【提高挖洞效率】Burpsuite杂包强过滤src漏洞挖掘  
原创 神农Sec
                        神农Sec  神农Sec   2026-10-31 01:00  
  
  课程培训  
  
  扫码咨询  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b7iaH1LtiaKWXLicr9MthUBGib1nvDibDT4r6iaK4cQvn56iako5nUwJ9MGiaXFdhNMurGdFLqbD9Rs3QxGrHTAsWKmc1w/640?wx_fmt=jpeg&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png&wxfrom=13&wx_lazy=1&wx_co=1&tp=wxpic "")  
  
  
#   
  
专注于SRC漏洞挖掘、红蓝对抗、渗透测试、代码审计JS逆向，CNVD和EDUSRC漏洞挖掘，以及工具分享、前沿信息分享、POC、EXP分享。不定期分享各种好玩的项目及好用的工具，欢迎关注。加内部圈子，文末有彩蛋（课程培训限时优惠）。  
#   
  
  
01  
  
0x1 【提高挖洞效率】Burpsuite杂包强过滤src漏洞挖掘  
  
### 一、burpsuit抓特定域名数据包  
  
下面记得勾选包含子域名，要不然有些数据包抓不到。  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWlafMs2Dc7Y4cWiaGIEwP2fIYOocsKKroGR7ocakWMOBW1Oic06cFPib3Ete4MJKWqTCiabhCyjjwQfh4ia822eWhwZEfSic6R59kqk/640?wx_fmt=png&from=appmsg "")  
  
img  
  
这样勾选完，就只会抓这个域名、子域名相关的数据包，显示在http数据包中，方便我们测试  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXmIB6WvfMjibX0qsf2RcjuuQ5ASjSGZjcAbC5icJicCKB531V2kOV05A4xOFcuKy7BtjeicNhN424wGxiaagvZ9C4Qsaiauib8LvQdss/640?wx_fmt=png&from=appmsg "")  
  
img  
### 二、指定域名/IP/目录/文件 不抓取  
  
比如目前，测试网站有甚多的杂包、心跳包，想要：指定域名/IP/目录/文件 不抓取  
  
这个我们在数据包抓包，拦截中，可以使用burpsuite自带的功能  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWvgugFGe5lAC6ibaGnj5M6yxPVHBlJGzf9B8yiaiamicuGgCpZjowkHQ1FhJ0jR6aiaq3UEicpKaUqTcpmXVCzuIias5EsOiaIm76KK4Y/640?wx_fmt=png&from=appmsg "")  
  
img  
  
这里不想抓某域名下的目录，就可以这样设置  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWicggoEQiaf2SCAiaFGaMN5IN6EFugKRrHcazo83fOWWHDfN6pyrTicFIFPZ08ibgOB7pwNTGZIjbMNqpiaHXiaMBvIYbojRhdDQvL98/640?wx_fmt=png&from=appmsg "")  
  
img  
  
当我们下次，再选择拦截抓包，就直接跳过这个带/xxxx/路径的数据包了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVKcUZcfBcqSbRZMXLGF8NUgZHjKe0S5HfFKPIIp0z7pGWjEFicePbpb9DeA7Npy5mCdCm1vL1ldUCchMOM9CJsQTc3dIeSYnOs/640?wx_fmt=png&from=appmsg "")  
  
如果想要取消，直接在burp设置里面，把开始设置的这几个给删除即可  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWhib8icibadNcpZBRGISibs0ib4mtWgcgdc2Db6vuAmFlWhPt5lN3vWwt9BhJhB2NROUb6YfBg4T1z8MIZB2iaib3tnAcoDksYXvlfyU/640?wx_fmt=png&from=appmsg "")  
  
img  
### 三、自动修改数据包（类似MAR插件功能）  
  
然后第三点，想给大家分享的是：自动修改数据包（类似MAR插件功能）  
  
其中，新版本的burp自动了部分的HPPT匹配规则：  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVm6jodX5jPlUEt1EiaXI31tZojzDFVX7xWr4oIStFpRsNKPh7fXDb32yg4FxoX3bK66yCSU1cXvicUhia9cyZIkib299OCIxTqnM8/640?wx_fmt=png&from=appmsg "")  
  
img  
  
**1、第一种：**  
  
uid  
在 Cookie 请求头里，配置一条匹配替换规则后，所有经过 Burp 代理的请求都会自动把 uid 改成 101，浏览器正常操作就能直接测越权，不用手动改包。  
```
POST /update HTTP/2
Host: example.com
Content-Type: application/json
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Content-Length: 34
cookie:xxxxx.xxxxx;uid=100

{
  "key1": "value1",
  "key2": "value2"
}

```  
  
这个匹配规则不会的，问AI就行  
```
(Cookie:.*?)uid=\d+(.*)
$1uid=101$2

```  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QWLsPrsbnqafSSRqctlvYNK0aicrt3PDT2Vkfrm7JqsakxCmrhXy4sHNSuSkz6j27nZ2TMkQbzLy6aw1iaCeFTV4LU5sECiaEsyYA/640?wx_fmt=png&from=appmsg "")  
  
img  
  
**第二种：**  
  
uid 放在 json 请求体"uid":100  
```
POST /update HTTP/2
Host: example.com
Content-Type: application/json
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36

{
  "uid":100,
  "key1": "value1",
  "key2": "value2"
}

```  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QX16ovJI8R5orNmdX7uwTqCpDThiaDWHb8vTEfM019J8ibiaia0ibxUPiclw45mmabr8Riak0E9XHsmgQdxAgAE3diaw2rEcw4DHotKcpE/640?wx_fmt=png&from=appmsg "")  
  
img  
  
然后fales → true常见  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QUylnnzhgYibG2jibGP73eLghgXcd7jyI8zRm4HgMEoZQjPMj8OG6wwQkRrvlyhDNicib3IfvA4iamEJ72PzMumoH4yMJhxH5bnWxWQ/640?wx_fmt=png&from=appmsg "")  
  
img  
  
**第三种：**  
  
假设这个 uid 传参是 GET  
```
GET /update?uid=100 HTTP/2
Host: example.com
Content-Type: application/json
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36

{
  "key1": "value1",
  "key2": "value2"
}

```  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QXp29WLDicibsMXG3zaFmfcbgZjqJfXU4DDfn7hy13BicVvjL5RBntJ6iblYzw9DPic8lBfjOF1t9xDz1PQVOZ8f1qVyN4j9MPQ6Ats/640?wx_fmt=png&from=appmsg "")  
  
img  
### 四、浏览器插件过滤数据包  
  
相信大家用的最多的一个过滤指定的域名，不抓这个相关数据包的  
  
通过Zero Omega浏览器插件即可：  
https://chromewebstore.google.com/detail/proxy-switchyomega-3-zero/pfnededegaaopdmhkdmcofjmoldfiped?hl=zh-CN&pli=1  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QV487U0lujWlL9rzkmL4q5pYbeNF9PMUfgAK0kO5r49xMgqRkiaQw3ocFHvVmg2eNviaRTjEYaiaPGghUXInCsaRfoFB6nA6FwpQ4/640?wx_fmt=png&from=appmsg "")  
  
img  
  
然后进行把一些域名，丢下面就行了  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QUyzYLfvLQ9pDjmujKfOicoibLFNKHic3j4ic0Ze5nT2FJicbhowUkI9ON03xibMrf9uMYymHsf5Lw6w2FtrHBOexyLxLIFfhy2NBTaw/640?wx_fmt=png&from=appmsg "")  
  
img  
  
这个是为了能够在chrome浏览器上，抓127.0.0.1和localhost的数据包的  
  
<-loopback>  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWXFClM2usWXVgzK7jbqwiaZZXjNbtaiafNUbV7Io2Zp2YoclAbnCdGX06oY18eHsqdiclg5OdARmQUwg8BQPgnLHjPL2vgczo85Y/640?wx_fmt=png&from=appmsg "")  
  
img  
### 五、burpsuite杂包过滤+bambda脚本  
  
这里脚本使用的地图大师的模版，大家可以自己下载：  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QX4xQqEb2jib30cWJv9hlXK2mzxuvVrEXbiaN1kAic801Y2haPkhxyFLlo1nVfBTKuHUpJXY8lSCBVPl1G5ohGEFAjJVgcpPWMm3Q/640?wx_fmt=png&from=appmsg "")  
  
img  
  
下面这个是模版，具体自己可以调整：  
```
//以下排除内容都为半匹配，即包含以下任意内容都会被过滤掉

//需要排除的域名
String[] domainExclude = {
    "freebuf.com",
    "h5.youzan.com"
    };

//需要排除的请求路径或文件后缀
String[] pathExclude = {"WebSocket", ".css", ".png"};

//需要排除的HTTP方法
String[] methodExclude = {"OPTIONS", "HEAD"};
    
//需要排除的响应类型，包括图片、字体文件、二进制、CSS、脚本文件
//MimeType[] mimetypeExclude = {
//    MimeType.APPLICATION_UNKNOWN, MimeType.UNRECOGNIZED,
//    MimeType.FONT_WOFF2, MimeType.FONT_WOFF, 
//    MimeType.VIDEO, MimeType.SOUND,  
//    MimeType.IMAGE_TIFF, MimeType.IMAGE_BMP,  MimeType.IMAGE_PNG, MimeType.IMAGE_GIF, MimeType.IMAGE_JPEG, MimeType.IMAGE_UNKNOWN, 
//    MimeType.CSS
//    };

//需要排除的请求体内容
String[] bodyExclude = {""};
String host = requestResponse.request().httpService().host();
String path = requestResponse.request().path();
String method = requestResponse.request().method();
String body = requestResponse.request().body().toString();
var mimeType = requestResponse.mimeType();
return Arrays.stream(domainExclude).noneMatch(it -> host.contains(it))
     && Arrays.stream(pathExclude).noneMatch(it -> path.contains(it))
     && Arrays.stream(methodExclude).noneMatch(it -> method.contains(it))
//   && Arrays.stream(mimetypeExclude).noneMatch(it -> mimeType == it)
     && Arrays.stream(bodyExclude).filter(it -> it != null && it.length() != 0).noneMatch(it -> body.contains(it));
       

```  
  
这个使用，直接在burp点击下面的这个过滤规则 → 点击**脚本模式**  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXZqh509W002hj89k89Q8usX436fGaSjpibMK222jvynbicTPwic2olias53iblKCPXbcml8gdxX2Mo4FQz8BoEEGTopzunwtiak3xDE/640?wx_fmt=png&from=appmsg "")  
  
img  
  
然后每次你重启burpsuite，这个脚本的内容都会清空，所以你需要自己保存本地  
  
使用这个脚本后，可以发现http数据包记录这里，看着就更加清楚，不会那么多没有的杂包，耽误我们时间  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QXTAfPY4CabPrIm8mp2FCEQ1u7exBCgP7fVZvT0ZdibKvyia8xek1lBKmDvEGcXPvtnoT8S1D5ucqwq8K5N8Okpib2ZWPCGxgKoibI/640?wx_fmt=png&from=appmsg "")  
  
如果你不想使用这个脚本过滤，那么选择设置模式（就是自带默认的），点击保存  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/mcko8AHj6QVl0K9IeiaQcHSKEQbJRLdghiasdwxSibyOvs3vYkQiaeSzgs8lNxqWBmiaOMW5rwGTCDImtLU8UDic3QeSPjrMBaItT9Rbh3eMt0nog/640?wx_fmt=png&from=appmsg "")  
  
img  
### 六、WebSocket包过滤  
  
在一些测试中，WebSocket包很多都没有用，但是burp默认还是会抓  
  
想要不抓这个WebSocket相关数据包，直接设置里面：取消勾选☑️  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QV13UVBMDYQRHc0glB0JaMPTHaOiaX9bX5TJrplgAdJ7QaEWgfu5trn7Fbquuf1gJyvWTicKiaf5cL7BQpLxmgW3v1Pia8ZrDfdIKI/640?wx_fmt=png&from=appmsg "")  
  
img  
  
默认这里会抓WebSocket的杂包  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QWOE15KdibPfCA2xE8ibvLK46ySMmiaBeBWBdkeqYnhnZ1OXhkjxA8r5FkIIwfYMHibnRHDa5THjvSnLtnAZ7xGDtUbb9FgurxyT8k/640?wx_fmt=png&from=appmsg "")  
  
img  
  
取消勾选之后，就没有了  
  
![img](https://mmbiz.qpic.cn/mmbiz_png/mcko8AHj6QVbNw3nRoZQl0I1cY0XstWG2z4VVhBEOElhmxMOCnpQt0ReMkHOwIoELfkaZvDJWC1gUjt6o4FrMZJ631WuqAiaJozltSK1OUak/640?wx_fmt=png&from=appmsg "")  
  
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
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mcko8AHj6QVDF0tAQcNE5ZX2ribDK5K6YJibqD4YbTxRC1ic2pvoGqJYN0SbuEpQFXL5t1SmeIC5ra0dPWUL1rGvcZuytdiaAlAnCHyqaXe1KUM/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/mcko8AHj6QV9grs7NOhSTCfTpCc4xrxdnlISIReNNCKR2EOyWvhMpyIzbma8nuelSg8LicKF5yYZ7hgyODlWgMmhViaE8Ahhs7PZlnmA0VFcY/640?wx_fmt=jpeg&from=appmsg "")  
```
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/b7iaH1LtiaKWW8vxK39q53Q3oictKW3VAXz4Qht144X0wjJcOMqPwhnh3ptlbTtxDvNMF8NJA6XbDcljZBsibalsVQ/640?wx_fmt=gif "")  
  
  
