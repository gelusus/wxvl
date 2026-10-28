#  记一次学员ai挖洞不知道原理经过深度分析之挖掘通杀0day过程  
湘南第一深情
                    湘南第一深情  湘安无事   2026-10-28 14:12  
  
**声明：**  
**由于传播、利用本公众号湘安无事所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。如有侵权烦请告知，我们会立即删除并致歉。谢谢！**  
  
前言  
  
事情的起因是学员用ai挖到漏洞但是不知道怎么来的，于是有了这篇文章  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Eu4yGp6sZYg48GWyTpOasYqrDfoFz6Ud2lGS70UPibuarlTaPXgvjmbBSwwSBhjL1AnVwS6UB7d9tAwO292TIdhk28SqLYcIPm8/640?wx_fmt=png&from=appmsg "")  
  
后面半手工半ai分析了一下，发现了一些的新洞是ai没有发现的漏洞，还是edu通杀漏洞  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvKXp5mJog4njgLVVur5lt2l95YCiaO7gClniaPqic1dlBT3ma3pLvzB1P1GOk418khaqsgzA5wTdo6hxMxwUw5u89fyUWyUQ8yZE/640?wx_fmt=png&from=appmsg "")  
  
现在不忙了还原一下ai挖掘过程和人工如何分析这个漏洞点的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsYGt92mnG5BfNzdW47ERtbfT4FecITj4PnwUbvLaKRiaE6uH60QK4iberARgwOicHWfscGzW1ZDPgVrWGaQyCorw8978TlFNpLL8/640?wx_fmt=png&from=appmsg "")  
  
模拟ai挖掘过程  
  
首先开局一个登陆宽  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuCjVvbaYHcicG2Wbh9EsUCX4kKtl4ypusL5qwFg0zxOFskRuYtbM1UcFmEcPfFDoWXCZvE4e5MEpLm26Y8UPcrrHVqITRn5V0c/640?wx_fmt=png&from=appmsg "")  
  
ai这里是怎么挖的了，先用js-sqg爬取一下api接口然后再进行测试  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EttlFB0icekjWJiaqgyJY1baQjPiaTnjQWDRtn9rzoYc6oDVes5yZtvKYiasgRzcgMbUo08x3gIMKYxHaRdxKAwiaicMMSARjLSDNRms/640?wx_fmt=png&from=appmsg "")  
  
过了半个小时直接给了一个sql注入给我们  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Etc3zOGlWicrPw4QHOLRTr3Jb9J1BaYMRU3icuFibG5JyrUEicJVz1BeBtpS1DU3cj1t4KKvz9QFNFcVCPYmwicHNxHffsvwFtO5d1E/640?wx_fmt=png&from=appmsg "")  
  
报错注入直接拿下  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EsKlFgeKoibHMCWicz9ibEmCfpRKsYUQibVSBU59YKJjH7Fd5zqibiblKGGFPSEV9ric9pkUTiaeqVRD6r4v0FGZRz5h4Enehyw5swWI1w/640?wx_fmt=png&from=appmsg "")  
  
  
  
使用人工分析ai挖掘过程  
  
首先使用登陆宽发现数据包加了签名，这也是为什么我们一上来就把站丢给ai测试的原因，签名的站真的不好手工测试  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Et0AC67dIhCH7qibrxWRoBr48YUdlK9zQBbwIk55UbicUhibmJzibEw60aJ10wa8nYoiasym8UAhzWbQiciaBWClANibzQozc4dGkrianicU/640?wx_fmt=png&from=appmsg "")  
  
学员的问题是什么？他复现不了这个漏洞是怎么一步一步来的。因为他再js里面找不到这个/list.do的接口  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuyyE35UGj1CY19ddgh2YT5mOOfhfBzKUZXXmJVdtDXtpScXDtHre0gafvtia7rmddTyzOWky1GEd7SPbtlaAmf6ywOVbibmSPfk/640?wx_fmt=png&from=appmsg "")  
  
而且整个页面就3个js，他就很疑惑这个接口怎么来的，所以就让我实战给他讲解一下，于是我开始了分析这个网站。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EtfUSF9D4VUu6viaicpeiaf3icrCYtmQ9JI6m1Zj7N1gAaZlG1aTia4ibJfkn6QPcpojAqibfBB7ES626teXzxIF6v8eVI8dVZFINscgc/640?wx_fmt=png&from=appmsg "")  
  
很明显了vue的框架，大概率是**JS chunk**  
映射导致的，所以这就是为什么找不到接口的原因，js再映射里面并不在显有的js里面先用工具提取一下。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EticBR1LWJlIVuia3y61IONLY1ibIjXOSXWnic9Zz04eSzgrn2bsvPOhlY6QEPOU2C3Zg4XibibkFGIpmicqe0X4LONjM5CCqiczCXiaibKQ/640?wx_fmt=png&from=appmsg "")  
  
提取出来还是不能用，这里其实还是要人工分析一下  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsyRAmlIibqqDzcqTicpjD59X0Rvq3YQEu6Jv7A0Xia9xlJh0RIOib1l9tsp5vZVfSWVk5U2EOmWtfKP2dVsAoN1ibiaALemVyliatjss/640?wx_fmt=png&from=appmsg "")  
  
首先找到返回内容多的js  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Eufwu7eyibzLrmK844EngOMaGUkiaAKUoss9ab2O263ac3Pv6ia1pnkEOicprjFo0dD4cczhfgX0RxVrDUtQAVicNOkrqjDh8l5DZCk/640?wx_fmt=png&from=appmsg "")  
  
找到映射的地方  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EtNuic6ubMw6uCIS6VIzX6dpERvicIVGVYic9Aic14bBkEhKWkcQdYXnqLGPat7D3vygJAkhV50Vn1N72QGam1ycicPVoS2EibIibLuyw/640?wx_fmt=png&from=appmsg "")  
  
直接把关键数据给我们的  
deepSeek好大哥  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Evniaoy2z0fDR1yTxNBg3Gy4I0CeicqPZ6KVCsiaa88RxdgllicG9PNlf2RhtNbW0RIiclPItnrbJxRrWcSpr1tcU6hXPHMsr8JDBWA/640?wx_fmt=png&from=appmsg "")  
  
给我的数据还是不能用  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvjuPchXeiag2ICftMe4C8Qa4GC8FuXzNHn0U3Qh3v79lMvOYtVTlL9e6fmB1GUMVn7ib1BQyC0eS58wNxlk6wvUwD5lib3fycxRU/640?wx_fmt=png&from=appmsg "")  
  
再控制台跑完发现一个都没有  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvibrTiad73GwSHh9KBbib2ww6H0W6o7CS3aoLdYc3B3RoWqGWtv9efaFWPw5CPbu5m9LQPcsF2ibI3tMZuEuPHFaQgFwBb6OlJ7XA/640?wx_fmt=png&from=appmsg "")  
  
崩不住了，只能先找个存在的js，我们先看看提出来的js为什么不能访问  
  
https://SQG5200/web/mgr/MgrForm.e3b155c4.js 这是其中一个，还是不死心用刚刚Webpack映射提取器提取出来的路径全部跑一下  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EubqBhib4FqdUQicPibKZBIm4tGpfMfUib2CbkqaDiat7s0oMsT2KHmzdoeIj3g4PeGFFcv7Wo2h3M3ufcrg98yXG6LjwNGGsnotgI8/640?wx_fmt=png&from=appmsg "")  
  
无果只能人工分析，发现其中一个chunk  
映射的js前面存在js文件夹  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EtianD4vHooYrxBf4ianjKmxw9YeUs1DjZ0RwaQIwSus1Dajdc3gENKGXK0plkjibYibKZ715gemtQ79VicbM0v2ibbwpredsXbf1XWg/640?wx_fmt=png&from=appmsg "")  
  
于是把用  
刚刚Webpack映射提取器提取出来的js路径全部加上/js/  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EtdGlbyP6DpHABfxnSl3PMQWgxR7zkP6yMzSHjlfC6yazTckLlKgHor6lysAM5ibFUL9ag0z08C4l6QAQaWeAgnyzA1nfPuSdsc/640?wx_fmt=png&from=appmsg "")  
  
跑完发现全部都有200  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Es078eeWJeEVaE3kiaDfuSicGG2TgsETA8lplg1wZibtppK0tjzPtE5RkvEcmhOhtdfjprkBlbicoicwnPIlJjdLqpchYH2zKjWF1WE/640?wx_fmt=png&from=appmsg "")  
  
接下来直接从这里面找到发现sql注入的那个接口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EtzIVhicrysnBBxtRQoIHuPRhBe937yvCC5BicfBeibTZXrFIoq6RXIOYqILXEj03Rt3084qCMWPtun3PHqImn9wKAPZINrtVmaicg/640?wx_fmt=png&from=appmsg "")  
  
然后让ai构造数据包直接注入成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EsMdwKIks3nDvcZeg9wibwZyMpxxDYmXKXh25ZU5IicYZQ7I9PUcJMR6UIicrs19OoZXFOzjH6HzquXeLIk0qoYdFAcLpKznMHkpw/640?wx_fmt=png&from=appmsg "")  
  
但是你会发现如果你不知道什么叫chunk  
映射，当你看到ai挖出来的报告会一头雾水的。这时候就应该找sqg探讨一下了。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EtlZkcH3MYQWMDJzptoAibF9ibIpfAZMZu0XicMJ62Q4qFZyqCPauIlpF2jyK0fsFQp9mnIia0rHOdTknDiceY14wskicXZqmykvwPVA/640?wx_fmt=png&from=appmsg "")  
  
手工挖掘ai挖不倒的漏洞  
  
我看ai是先去js逆向才去进行测哪些sql注入的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsCj7LOTTggC3MLknHsqdAWJCKDrffrbzbXibLcRvlHfkMddxhgiaeGVj5eibcUPJb1uML8icKYuB60w0JQ6vZECujm6EEpnFt3GB0/640?wx_fmt=png&from=appmsg "")  
  
这里js逆向挺复杂的，我也手工逆向给学员看了一下，放到录屏里面了，学员自行查看。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvKuRVJU4eIu4YhxiaRrXV7LztQJ6KAqeHempnAqlVfpNy2QZRyibgeE5nA6mzumceyOydybxHfSyYBIQtJpdtFw2jhdQ2SYnciaw/640?wx_fmt=png&from=appmsg "")  
  
这里感觉ai直接就去js逆向挺好token的，想想能不能绕过签名，我们来看看请求体  
```
{
  "appKey": "uY29lgRz",
  "secure": 0,
  "param": "{\"offset\":1,\"limit\":10,\"name\":\"noexist' OR EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()))) OR '1'='1\"}",
  "time": 1791016815769,
  "sign": "dbe17da8e042667dc2326b22ef0c95e5"
}
```  
  
发散一下思维+经验看看能不能改成下面这个样子，不知道会不会绕过签名  
```
{
  "offset": 1,
  "limit": 10,
  "name": "noexist' OR EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()))) OR '1'='1"
}
```  
  
发现成功了！！！！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsRKHsc8h5yO88qc9ZiciaFFh1ic86HicjKlb8vKUZibZAa38mANAr6Oe7dfW08xQLzMzPNuqAWf7PyX8DNpcFicEAugjh8M2tu8fOXA/640?wx_fmt=png&from=appmsg "")  
  
后面发现登陆框数据包再签名下面是没sql注入的  
```
{
  "appKey": "uY29lgRz",
  "secure": 0,
  "param": "{\"schoolId\":\"573\",\"username\":\"111111\",\"password\":\"111111\",\"customerId\":\"573\",\"openId\":null,\"projectId\":\"136000697\",\"device\":\"desktop\"}",
  "time": 1793194518494,
  "sign": "bd4bc7b6c7878acb88dab24b6b59fd81"
}
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Ev6toiaf0LKauOGl6NjfibuWWhbtT73n8auiafnMtUnQvn8eyZyQRz92x3jfsD88BaLwk6EXPTY5LpVINW3sd4czEyussYhvOFVTU/640?wx_fmt=png&from=appmsg "")  
  
但是改成不带  
sign的请求体就有注入的，还挺有意思的  
```
{
  "schoolId": "573",
  "username": "111111'",
  "password": "111111",
  "customerId": "573",
  "openId": null,
  "projectId": "136000697",
  "device": "desktop"
}
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EsWcmhvswANjWLkUSPuoA4LEOwxKpCU7tLxTOiavzZXbox5FfpasjCw7SAcfvpZ8LYOyEmiclBiczVfDQ7L9oChcj0Spudc5D30PM/640?wx_fmt=png&from=appmsg "")  
  
所以ai也不一定能挖出这样子的漏洞，大家可以多半手工和半ai挖掘，把这个经验复盘加入到你的skills  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuUh4icA3txDUEMz6oAXuhDIbpH0bog1DDK4Gufm6zRnyEjoHqgGULj9HxCib1HpdsMI4WyC24tpXPITnn9oN16eQuh1LG1HYNOs/640?wx_fmt=png&from=appmsg "")  
  
## 深情版edu+src培训  
  
  
深情哥这里是有edu+src培训,更新了ai的一些，将以前的思路和ai进行的结合，交大家如何使用ai挖edu或者挖赏金src,我们更注重交学员如何使用ai快速挖洞和学习ai的思路，而不是异想天开让ai猛猛出洞，这样子会让自己的渗透能力退化的，我们不能成为ai的奴隶，而是ai的注入。课表如下：(或者加我wx:azz_789了解)  
  
新增加AI渗透挖掘笔记  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsVe1PjoXg740CSB1TwEht5hVvlyWoTVobNXDUt6rV3hvkonVvnnS7rKqSvStnaDD2yiaoAnPuMOOG50fICul22Qc2q6JRX7kjM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvMBWXVhw6iaRM3CCcjDT8njjYVibgARPmfgv7hO8iayaaibPYbvJQRF4WJCjoQBBG94IqfUibd25EnyKWdHzxANyTTG98eeQjhSIWQ/640?wx_fmt=png&from=appmsg "")  
```
https://www.yuque.com/denghanhan/qqby5b/tmde5e8qwdysm5zz?singleDoc# 《深情版edu+src培训课表(2688)且赠送审计版+ai版》
```  
### 【edu漏洞挖掘实力展示top2】  
  
湘安无事团队一直都是edu漏洞平台的top2如图，都是离不开深情哥和区长的培养。(  
学员专享：对于表现好的同学，贡献比较多的会打印团队证书发放  
)2023年到2025年的证书~  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tlibgKYKL9EtE4c5sEdKhahpgNo5vWm2ckJrvhzg3SU9C8g0HB8C5omRlNdZKWp8icZrYWgwAwqkEVe4gzq46jeWYkuTfek8KQswebXF4O8t0/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=21 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9EvV2W17qJkXWvShJRU1hOJudXUrps6cN0vulTXAHNlqWMbKpdAUUQK1YJfD46jzoK0gvzuFkAT8OPics24yvtic3kgpg2sqdAtcU/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=22 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Eu5sFqDUaFMOwx8mZg5MOx1Xdu2ibclFWrqakbhia1s9WnXYkRa5vVwqRMWU723L2E72n40y7NVZ5Fj8HtzictMryEAeiceQN7dJkg/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=23 "")  
#### 深情哥edu漏洞挖掘成果  
  
我们导师深情哥实力，深情哥edu漏洞平台top10发送证书奖励  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuuM2MUfibVT7atxYg80WzSeicQ2QXLkldJYAWp5pzPIQJoETahbuFfjE5xA4vIgiaiaptFFhITEoJMy70Uml7bWbxicRh0UwsM2HRY/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=24 "")  
  
深情哥都挖了快2000多个漏洞，难道还不能带飞嘛，嘿嘿。我的edu证书太多了，可以看后面放的截图。现在不怎么挖了，因为现在都是带学员挖edu漏洞~  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvSxmTg3k0YdEJ6CBsjmX5UgODw6DbiaYMexJHdgANibIpZCR1AAWKMDWOsHLE0bbToCqToCQ2jJNKDCOYRjLqib5UXmMtKLqSmfE/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=25 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsBbA1qX6vE5ibRvickZ6QD0guvbyI7drmXX4JcMRbyHfxLyfNFiadUMCZDfd2UxtJfH2VHb6Ps3Qf51rDgCMu0nHEy8HkAVxkmq8/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=26 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuM0cyTlM4EMREN5goXVWfBJib1UicyJ8ELQZZpcibVkTDGq0z58hia1KQ6MwQXWY4FXhSGicynwdmeQnicYf9DdDiat3uEOkK6N4sjmg/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=27 "")  
#### 中国地质大学聘书  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvWgbZn9o03xWscH1lzc2VLORicwcwsF7Kicknv8x9CqUT6ODvNeQibNnBHIrWA91YTl77YFwBYUT36Wia1kzLHldEeOwic6eKfTKvk/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=28 "")  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Eu70licJW7maDLPRmYiaYCDwddofebHkYL7Ln8Rl5T11KI7hzicUmdhlUTCYknFbboWbxZZNliboa4x2xxZP1NgwIMFs3ibyL97Uuxc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=29 "")  
#### 各个学校的感谢信  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9EuaMGASUebGVHrugwlwtkwfziaR8vU7jBTlmgvR32ibyWzibw02aYK0Qv4sI65jM0U2RLiaJicibjqCGPPibEq1qSTRGPibPsAX3zk3uhQ/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=30 "")  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Ets7icUCva2Xpo2Niafb2vCo6PvCwHbwAe00ADLVI23Toicm1ItLfOoqBuHwf4KlXJxbzJjFfs3v37LteCmSiaKicXj6qojsslWsvVk/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=31 "")  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EtIDOfhyJZDKPzaH43plmKlxcs0ReibgjSnqrPDuO2VeEQsbia9WoibicMLWwlIDCno3iaXx9kNeb9lYb0tXBFq2PBISPruiarNPD3u8/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=32 "")  
#### cnvd实力展示  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EsMXY5cYHWMRzK3qzRcj0bMZPOE8dW9ZC5cNBoqsmZlxslhflqxKkRHCCzGBB4cXZHGzgrjP1Gpy6KYHsHKvlR72DWkUibNDia28/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=33 "")  
#### edu攻防实力展示  
  
2025年带学员打了两次攻防，都是第二名，第一次奖金1w，第二次奖金2w。  
####   
  
其中拿到的edu攻防奖金都和学员成员瓜分了。**2026年也打了两次但还没打钱**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvDnqFJRticrtV2r432RF9CJdhfXjgc0AokDJMmd7CmHwW5DCVSH5uum0GJKTrpfI2MrgRVmPNhPIS7nOQPhwMFoIzOMA7iaKdHA/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=36 "")  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Etg1fGiavI6d4O4Q6kWzaPlnrDntNRibKtmtHTmIltgh7Y5grHCyNytZsYt6AyiaqrhXrpziavxEiargviaibVqDxHclcrPPWL56x3eZM/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=37 "")  
#### 个人年榜top10  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EsUQuRaE5H3jzsaNbe3fiaL1jFiaUqLOEcvy1SsrD4WQAgxHIqJllhv6OFvnBVU291k3cIvjWPc6QwVeqohJe2ibnQ7XHn4VegJg4/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&retryload=1&tp=webp#imgIndex=38 "")  
#### edu漏洞平台带领团队第二名  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvlJ6mVeBKdtcRb3RnPAMHJVibBopnjQyIgVt3ERibQAzyNXPvALabmHLDmgAw4vHsBqsBia6iaulQvZLm9UMNBvPbe3myJhlET1ac/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=39 "")  
### 【赏金src漏洞挖掘实力展示】  
#### 深情哥漏洞盒子赏金50w挖掘展示,我不只是只挖edu鸦！！！！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuQI23b15FW1Unqo6RHs78CfmAncOqWDialcdVDGApAJy66eIUyExqRM7Ul1RIRYwIXLq6PLZniazW81qicSlNoAIz5iaicfyibicq4t8/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&retryload=1&tp=webp#imgIndex=40 "")  
  
深情哥也是挖洞累计快50w的男人，漏洞盒子战绩可查，也带了很多学员赚钱了，可以看后面的截图。  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Eub3ZiaGSZtETxrhOozJ0F2sYyKia6oOyrasvDaze2d48ViaeA5icdsakib2oWWzY78c2eL5VPiap1E3r4QUG2W5MlU39cD9icj0IX6bY/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=41 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Evic6F8gB7fAlk6RufxWAEdicaQCzyo8ytTFRibC21hRLJiaxmD809L9zY9s6Uh9SuCQ1XxYiaVUwzF5ELStw3yNpGrVtLTXDMroyg0/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=42 "")  
#### 银联src  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Eu8xnxtZP6byfLgCibgxXxYicDDobSk8Z6xWwr8JMTv0jkwcFdBlB38XAIRfC4cSg9BvWWAA3GwBNwotVSOS8G4JcGiben4p0d7K0/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=43 "")  
#### 漏漏洞盒子两个赛季top3  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Etaicr0KnXKjLCfZ0sbk5bNKZtGmSnTiaL1bf3z0HTQaJtNKD1ttj4No0aUNjLiccfPyAxMciaIscicJoOB0vDD0ibib7xayC5qDGFLWU/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=44 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvWczGMUarvmbTawkibVBbxajD1Nia6nu71PfCiaNoyBIvYX0HTAhic7lmicBukiaIKorI6eHduv3rqR3MQeh67iazx7DI31fjrIicYFcg/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=45 "")  
  
漏洞总榜33名  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EtYaEQiaqXpWLKjp6xQ08VEkQX38Luz5aYQRdDbf2tKUq5Q1ea2gatI0xJJibaGJibHkRE9Pic0dO8ibDmylnRKicoicSC8g3nmXFrnxQ/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=46 "")  
  
其他平台也就一点点  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Esn6CIp271E8J56WKVsTfHCQnjbePLBkQCFodteMk2AGKeEIEwiaic3CV8n9dWLLJDnNKCmNibCtYTdm5uDaQRLRCwaJ0bWthpLlI/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=47 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuzOl0CwkpBNWrT0OEZicyo2XMXiawpRtwaDYjZAg14VCZMMSSeIg3mQ10QuN8OibxVk3gKC6y5HFkeqth6EY4q65cjfNKwlhbt5Y/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=48 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuFqHiaBg3Kmn49nSkhVen3lQmUESqfNpPp6rXCuJK9G2Mjnibw87Qg6ENicFXW6fxOA3HLRHCHgLegqE6xVCj25WlV8zqiaib1aD2Y/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=49 "")  
#### 带领学员和成员一起众测赏金累计10w+  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EtdpfwmGb5gviblvBn0TL1geMkxhCYh9ARXGFbqwE7KXG9d20WojTdHjj4bLoXj9f2aH2ynpqoV5caP58nNv5opArRcGxCsic5z8/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=50 "")  
  
  
【部分学员实力展示，太多了其他的可以看我的朋友圈】  
  
一堆千分rank的学员  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuSymcJje0j4qQRwcpCQemtIkAx6OSFbNGeDib82cCibjyrSl3SudEiaQB0LZrZicZz8UY4CyFf0iccN0HrnAaT8h8jqydokLKCN3vc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=51 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Euuibheka7GNEnb2HrVYEnoQDxGX8T1xSh21O9z1AF00x60pKHU3V5fW5Vz8Z6Zv0OyyBCCaNuIC6kvq4N56skkMEYNZuaCU0qw/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=52 "")  
  
就是太卷了，希望他多注意身体，交流到3点，他妈的他8.00还能起床上班，我懵逼了。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvpKAEib9oicEBAd5gjf1icsyicyqJ3ic9P8KGsuTlELLWs6FwSEjglyZc6gqrqUQt9DWeRcPTBk1BostfkDIy2AliaDZLZeBYtLMS3U/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=53 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EudicO1dz7N4PjzOwtiavCXV6gQPVlHaPveYeIQjrXhGVl3MAmOibobhKRMeM3WX6ibGdJhUZNCKb72bp5ORaj7ucTfCMAXVhlVcGE/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=54 "")  
  
你以为深情哥只带学员挖edu嘛？深情哥盒子赏金也挖了快50w(真实可查),所以我们培训内容也有赏金src的内容，没有挖到赏金漏洞我们也不会轻易出来瞎搞的。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9Euv31JH8N7UBGZyAW9ib6BgibiaIVcpB6QglbuKyHsMtPSu3GDgicdhbYPsW9Bbpt8OmcuuxwvUsP2w6WQ3ice63Fribrn1N74JLegOk/640?wx_fmt=other&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=55 "")  
  
去年跟着我一年的学员差不多赚了1w多，真心服务好学员，有问必答。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9Eu3GrM3h96RianpdDAKjnSEmazNY8zVlG36LPNFf8DaQ0a1f6EVDiau0cFemJDUC7jxqFOsSmzic4Tfp6QW9MbecroKiaJaahvib5BI/640?wx_fmt=other&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=56 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9EsB1xr63pk9rnZHPQDH63gicGG0jtdDIIQHGfLFMBWsuZZlpRH8LFnVA25N2ovIlpfb982hoLD2F29ghtZrxE2rhHV2z6xpiaB7M/640?wx_fmt=other&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=57 "")  
  
【很多学员找到实习或者秋招上岸了，今年尤其是老妹辣】  
  
其实我们不仅仅交技术，也帮学员修改简历鸦，面试指导，实习内推，手下有很多学员去了绿盟、微步、安恒、360、长亭等大厂实习和就业。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Ev7mySQax0Ko9qamlrs4zQ896U2NUWtZW33urlhlWzFDdfHbwseSlh34fUwib60wcZwia1oJ0tl6HPdxQvkDuBwWImL8pMP1UCwU/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=58 "")  
  
以下是部分offer截图：  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuNQsTCBBicdibEebKnKQnSSzjQoibkcAzXlDTg5pj55qMESc11kPibTnnNhXmiaDUfxmVwVSIYKjtpTe0cia7pjMnLaGQvqLNb4icL00/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=59 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuPJS6yia7zqDT3Baibu6uoib6Kmql6ibldgWhSFRUShJT2kHYAr5bfSCTVtSKCjfk0f3Uic0oLgXJLwlkYLLia6Imn0FHYhQ7W5YOkM/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=60 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvWUHytUZlMxuB0IiblJ1Fa4vhz1eb0KT9w4Ufiamy0OMMiaBDalMRwPbjIAibTTqV9u7lSrPZuJVLmB6cAr4VQV7up4x9ianMon6SU/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=61 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuXkOgnrqFnpURxS7ibJUrKAqw713yGnce7ickj6eX1CIxia4JiaaOxY5MTMwn9icEddImgoicgKZ8YbVgFl3bHjpRMh1yyK47foLw1k/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=62 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EvXK6Oe2GZQiaDCoGsp2qHKia9qA147eJXvF26TNuLmyd4zib15K9U4Yto3bJWU5vEz7WM4icFwNBgWIDjJWG2fOdEc0DethhDhTXc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=63 "")  
  
最后总结  
  
感兴趣的可以联系深情哥进群  
，有公开课会在群里面通知，包括审计和src。  
edu邀请码获取，咨询问题，hvv渠道推荐，nisp和cisp考证都可以联系深情哥  
。  
  
**内部edu+src培训，包括ai渗透，src挖掘，edu挖掘，小程序逆向，js逆向，app渗透，导师是挖洞过50w的奥特曼深情哥，edu上千分的带头大哥！！！联系深情哥即可。**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EuZqEUJ1ibr1DZsnu0le2J5HZTMYC3uBJ19arjDsyNP5pOrQlCPq2UFAuFJbrmKLdOcBRFibpxavuz69hWrYoRkVcMCfMjXIpQBE/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=22 "")  
####   
  
往期文章  
  
[这个星期四太疯狂了！继PHP RCE又出现了node js，nginx ，Litellm都爆出有漏洞](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247496187&idx=1&sn=b97fa2905c074fbedaeebca29aaa3237&scene=21#wechat_redirect)  
  
  
[湘安无事甲方工作内推](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247496187&idx=2&sn=30bbbc6752d102cf6da2c47acc1c21ed&scene=21#wechat_redirect)  
  
  
[fastjson1.2.83面试被拷打古法研究一下链](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247496165&idx=1&sn=a83938d9dfd461e511d1820b92b72124&scene=21#wechat_redirect)  
  
  
[节后开卷！湘安无事SRC培训(ai版)AND学员近期成果 | 文末抽奖](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247496162&idx=1&sn=4ca17c9b422aeb1805803d1e5c21a1df&scene=21#wechat_redirect)  
  
  
[知识大陆 | 网安人的知识基地 · AI安全前沿阵地(免费送20个帮会名额)](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247496139&idx=1&sn=6f286870933afba984571da1b00c0e9c&scene=21#wechat_redirect)  
  
  
[2026年edu教育攻防奖励已发放](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495922&idx=2&sn=53d40bfe97d04512fea5f92faaba16a1&scene=21#wechat_redirect)  
  
  
[学员分享之记一次省攻防拿分思路](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495922&idx=1&sn=2da75a2d4c821d0a59433d48092f16bc&scene=21#wechat_redirect)  
  
  
[区长分享之AI配合手工价值1700的src报告](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495954&idx=1&sn=7707ace9484c27a7f8277ca42c1df818&scene=21#wechat_redirect)  
  
  
[方班实战演示的挖洞案例分享](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495816&idx=1&sn=5cb11b72feb40d607967bc0bf029423c&scene=21#wechat_redirect)  
  
  
[网安专业规划分享｜深情哥上千学员血泪总结](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495740&idx=2&sn=08816bdabb2e31ab216d391397a5e676&scene=21#wechat_redirect)  
  
  
[记一次ueditor的ssrf+dns重绑定+云主机元数据泄露临时凭证拿下2k赏金](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495718&idx=1&sn=b88b194035851246a02996147b436470&scene=21#wechat_redirect)  
  
  
[Claude最新版与deepseek兼容问题解决小技巧](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495680&idx=1&sn=c3bbb77ead4eb1f9dd5be766b7910820&scene=21#wechat_redirect)  
  
  
[Swagger接口文档配合ai-skills进行未授权渗透](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495623&idx=1&sn=a38c19d8955fd60ebbe57b25bbd51115&scene=21#wechat_redirect)  
  
  
[五月学员优秀edu漏洞挖掘案例分享](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495352&idx=1&sn=866fa5b97b821d22100632cfab11de8a&scene=21#wechat_redirect)  
  
  
[记一次edu攻防演练又拿下top2(湘安无事ai辅助版)](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495313&idx=1&sn=c5de1dd35d54b28a9e1349ac16c58dc5&scene=21#wechat_redirect)  
  
  
[2026最新版burp破解教程+ai操作burp挖洞](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495278&idx=1&sn=37563ab9c3aa7dbd0032d668c444579c&scene=21#wechat_redirect)  
  
  
[湘安无事首推的0基础web安全课程](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495233&idx=1&sn=c213276f0a0e3114fadb2ee693967c5b&scene=21#wechat_redirect)  
  
  
[学员挖掘母校实战案例+4月湘安漏洞库平台优秀实战案例](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495226&idx=1&sn=964c5d38716a725c251affa609579d9c&scene=21#wechat_redirect)  
  
  
[从逆向加密逻辑到一键明文改包：我的 Yakit Hotpatch Skill 实战](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495182&idx=1&sn=c00c7fed753d2eb5c28c7609ce1706e6&scene=21#wechat_redirect)  
  
  
["深入探究JWT：解锁身份验证的挖洞小技巧"](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495180&idx=1&sn=6d13c243db60ec14db5c9e8dae12b8b0&scene=21#wechat_redirect)  
  
  
[敏感信息泄露漏洞总结：深情哥提醒你aksk正在“裸奔”](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495067&idx=1&sn=47d93faf7fa2632bdedef8da9396bf04&scene=21#wechat_redirect)  
  
  
[985–edu证书案例之有意思的报告](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495097&idx=1&sn=dcb8c4a665b2030bf0e86d042f687ef6&scene=21#wechat_redirect)  
  
  
[五一弯道超车！深情版Edu+SRC培训限时低价，文末免费抽奖(两份kfc+五个无影激活码)](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495149&idx=1&sn=9250a9e7ac52ec71c4a492ec77645426&scene=21#wechat_redirect)  
  
  
[记母校漏洞测试一次waf绕过经历](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494963&idx=1&sn=a3402602ab048ef32c1bc7f36d702dde&scene=21#wechat_redirect)  
  
  
[Codex-AI 道德审查绕过进行js逆向](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494987&idx=1&sn=7344cac0a26228596f0e267793aca865&scene=21#wechat_redirect)  
  
  
[究极无敌的srcAI-xss手法（快看过来）](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247495003&idx=1&sn=fcd19fb5f7f950a3e7a15785577cd738&scene=21#wechat_redirect)  
  
  
[记一次差点进编制的漏洞测试](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494898&idx=1&sn=9f9c5a3527c8330f2e4312eadf0ce863&scene=21#wechat_redirect)  
  
  
[新版微信强开f12和新版本微信反编译](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494896&idx=1&sn=ac8ebcaecd5e18d45f79c7b6d8e3b10c&scene=21#wechat_redirect)  
  
  
[湘安无事2025团队和培训总结-福利抽奖](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494849&idx=1&sn=a7084a23faa401c4fbcb06af4650846a&scene=21#wechat_redirect)  
  
  
[985证书漏洞越权成为教授 + EDU/RCE漏洞实战解析｜湘安内部平台月榜 TOP3 案例](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494786&idx=1&sn=9ee660a005413f34e6d89f728df59803&scene=21#wechat_redirect)  
  
  
[服务号存在注入之有意思的edu漏洞](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494659&idx=1&sn=e357db59d806c93f3a5a36b5c312b335&scene=21#wechat_redirect)  
  
  
[湘安无事之湘潭大学冬令营总结](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494727&idx=1&sn=1d17ba9ee9bcd5fc4cfabe12d463e4da&scene=21#wechat_redirect)  
  
  
[赏金src报告分享&&edu证书站漏洞分享](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494759&idx=1&sn=65084c580bb13fa31f82985721147e7e&scene=21#wechat_redirect)  
  
  
[edu小灶案例之泄露上w敏感信息&有意思的证书站报告案例](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494702&idx=1&sn=5f0d05b6bea8550eda36ed2ce8ddbf7a&scene=21#wechat_redirect)  
  
  
[学员投稿之edu漏洞的JS逆向解密导致任意密码重置](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494634&idx=1&sn=7cc83a90badea88d7d7d25583bd0c419&scene=21#wechat_redirect)  
  
  
[最近学员小灶总结之双十二特惠](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494604&idx=1&sn=80f030c8114ee3287036586c670289c9&scene=21#wechat_redirect)  
  
  
[卡顿页面导致三本edu证书现世之学员案例分享](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494572&idx=1&sn=14c21f1cae87dbdbf2ca5c4b10533f5d&scene=21#wechat_redirect)  
  
  
[看完这场EDU通杀刷屏，连我自己都沉默了](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494535&idx=1&sn=113dd5e17065def94a5dacad361176b3&scene=21#wechat_redirect)  
  
  
[edu证书站挖掘之学员分享案例](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494524&idx=1&sn=bff39f27ea27bccb884e832603acda92&scene=21#wechat_redirect)  
  
  
[如何快速挖掘低微漏洞-项目挖掘总结版](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494500&idx=1&sn=2da7aa0d40075b462eb27695f2da9e83&scene=21#wechat_redirect)  
  
  
[公众号接管漏洞之偷偷加](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494345&idx=1&sn=6d9ab80cde99e95c4d1ac710e091ffdd&scene=21#wechat_redirect)  
  
[小姐姐微信](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494345&idx=1&sn=6d9ab80cde99e95c4d1ac710e091ffdd&scene=21#wechat_redirect)  
  
  
[辅助学员审计案例-php代码审计](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494284&idx=1&sn=2feaf2786717c2c577fe0b5230cbc58f&scene=21#wechat_redirect)  
  
  
[学员-补天800赏金报告分享](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494284&idx=2&sn=c54ea91e0b3dc79f48f54f931df413c0&scene=21#wechat_redirect)  
  
  
[从js逆向到sql注入waf绕过到net审计-edu证书漏洞](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494171&idx=1&sn=0d589d01dfbb068e53f9ad9c22676d7a&scene=21#wechat_redirect)  
  
  
[空白页面引起的高危src漏洞-再次绕过](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494082&idx=1&sn=6d1cc92a049d28b5395d417e0a5203a5&scene=21#wechat_redirect)  
  
  
[空白页面引起的高危src漏洞](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494061&idx=1&sn=bfb1406138041fcba8286811aa7ac6e1&scene=21#wechat_redirect)  
  
  
[难忘的优惠劵漏洞(深情哥破防版)](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247494035&idx=1&sn=422326cc617c33c78b3d518471e0862d&scene=21#wechat_redirect)  
  
  
[什么？又日母校？竟然还有表扬](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493987&idx=1&sn=f8edc2f3a113591e6e4cc57fdf821c8f&scene=21#wechat_redirect)  
  
[信](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493987&idx=1&sn=f8edc2f3a113591e6e4cc57fdf821c8f&scene=21#wechat_redirect)  
  
  
[sql server注入靶场搭建](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493947&idx=1&sn=885ee489f74211ba26a4a097f7ba30d6&scene=21#wechat_redirect)  
  
  
[记一次学校ai沦为我的宠物之拿下e](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493890&idx=1&sn=380e9dd566b29b713993b26f524fe0e4&scene=21#wechat_redirect)  
  
[du证书](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493890&idx=1&sn=380e9dd566b29b713993b26f524fe0e4&scene=21#wechat_redirect)  
  
  
[疯狂星期四两本证书漏洞合集](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493926&idx=1&sn=06ac298909e27d0e37fef90f64b205da&scene=21#wechat_redirect)  
  
  
[湘安无事之深情哥版edu+src培训](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493855&idx=2&sn=bb025b6a32bc022319f8dc8057a6436d&scene=21#wechat_redirect)  
  
  
[记两次js逆向拿下985证书站(学员投稿)~](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493855&idx=1&sn=0d507bc1f90b04588393698ee1c2d4b8&scene=21#wechat_redirect)  
  
  
[重生之我在教育园暴打小朋友](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493410&idx=1&sn=17df8a525145bf062ed5d6ecc3e09539&scene=21#wechat_redirect)  
  
  
[某医院微信小程序签名机制绕过分析](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493408&idx=1&sn=672ff76188ab6c97bc1b8d688ac7228f&scene=21#wechat_redirect)  
  
  
[记二次帮学员拿下edu证书站](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493399&idx=1&sn=425e84fa877f789598a2e8afdf8320c6&scene=21#wechat_redirect)  
  
  
[记一次难忘的net直播审计](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493369&idx=1&sn=e10ae1cd25ece5fe3c3ca4c2b455cb51&scene=21#wechat_redirect)  
  
  
[记一次小米-root+简易app抓包(新手)](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493374&idx=1&sn=7dea1086256be1d5a2b1a18fa4ab3be0&scene=21#wechat_redirect)  
  
  
[记一次带学员渗透母校](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493292&idx=1&sn=7b1881174201094a5449fe1400ddc8a6&scene=21#wechat_redirect)  
  
  
[同学，你试过交edu漏洞交两天两夜嘛](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493292&idx=2&sn=0d706a4064111b3e131f24460b2f2c56&scene=21#wechat_redirect)  
  
  
[手把手带学员拿下浙大edu证书](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493292&idx=4&sn=cd157b6471a1ab1e63b8dbfe0f62b42e&scene=21#wechat_redirect)  
  
  
[记一次手把手带学员拿下600赏金](https://mp.weixin.qq.com/s?__biz=MzU3Mjk2NDU2Nw==&mid=2247493151&idx=1&sn=3e1a791a4d14b9ad21d2f0afe8a455a2&scene=21#wechat_redirect)  
  
  
  
  
