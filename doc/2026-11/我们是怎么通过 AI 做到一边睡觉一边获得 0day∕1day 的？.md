#  我们是怎么通过 AI 做到一边睡觉一边获得 0day/1day 的？  
原创 ChinaRan404
                    ChinaRan404  知攻善防实验室   2026-11-09 01:16  
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXFicibCKdGukQbkfItCErnzibF0x4sW1JeoIKdxhpNpz0IzjtusuibZtsiaZs3BkV2rP7OGYYb3MwFr5j8Ry2Dnbe8yx5eU12oP0BXs/640?wx_fmt=gif&from=appmsg "")  
  
前言  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXGH9VbOpyk37mFgRlzIXx6xqjnficrDr4mz8kLqrCibKC4IOictw0q935fmPZXiceB2n7Lo2WathiarDGz1kicStLQia9qxttYchUIcu0/640?wx_fmt=gif&from=appmsg "")  
  
  
原理：监控 commit 来进行漏洞情报监测  
  
这个想法源于哪个技术沙龙，我忘了，某个议题提的，我们也把它实现了，效果比较好。  
  
大概是这样实现的：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXGppQx32nIz3epTNDibECJDK2j7RicrFqakaSwQ60AjsB7y6d840ITZoM2bUSC1XZpJllb6eb4JlTygqvicPjfV7icAxHCGyx9vzzc/640?wx_fmt=png&from=appmsg "")  
  
作为一名安全工程师，你需要先去学习基础的 git 和 commit 是什么，不然很难理解 23333333  
  
简而言之，在源代码管理中，90% 的人都是使用 git 进行的管理，Github或者私有化的 git 服务等等，commit 是什么呢？就是每次更新代码提交的内容。  
  
那么什么是   
diff 呢，简而言之就是提交代码前和提交代码后的文件变化等等。  
  
两者同时我们就可以实现监听 commit 提交，让 LLM 来判断是否为安全更新，如果是则记录，自动提交工程开始复现漏洞或者人工审批来复现。  
  
如何使用  
  
  
你可以在“情报”->“开源情报”，里面一键导入 Github 链接.  
  
该功能兼容的是 Github REST API，公司如果有自己私有化的 git 仓库，也可以自定义。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a3etiafIAYXE58iamHE8iaFqro8u9sBuEvtqhcemOjzjVBznnR8fPVskHUBb7uj3cykvTgWq5j7BwpogBmwTX1lSLm4b1WDZQdLvOwaTz4lDd8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXEZYt90IrHkuGI2DnlicXib15a1vmEneADHC7e52bnlnh4Tm3tfviazQecwMHtyg3yzwTicMYTm7MGCUKCW0zF4FwOswiavib3PEHQfk/640?wx_fmt=png&from=appmsg "")  
  
  
检测到安全更新 commit 你可以自行选择自动提交工程进行全自动漏洞复现，也可以进行手动提交工程队列，总之爽的很  
  
0day/1day按斤批发  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXGIw5nrcf6gh37qsXicNOezyvpw6oXAu3WpHyN4qp1Fe2kp2mUPQzaBbXpmf7ZOgJicaZ8Pg2IuiawgcSXs0ia3el9jg2XOH2VT5CY/640?wx_fmt=png&from=appmsg "")  
  
注意，这些漏洞全都是在睡梦中或者其他摸鱼中产出的（看后面的时间）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a3etiafIAYXHjzOb26OPVTVMVtZ3bqogsicBjSvC87uxKRdzJcWT88T1IBkHjwsFcAZqnNNjT0sCDRmkq16QtnJS734PKANF0AicsMEhNVP3H4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXE8fbSyLCrMwzAU16KHPxkiaOlHE0K5AKHKhgHGqEtPnPL6lYa3NBKlYpgB5WXkB80oYPicg1fcobWqoNiaQ3tfhGdiaJaPMiclRicyk/640?wx_fmt=gif&from=appmsg "")  
  
实战效果  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXH9seDsSiaxWIj23KeImbE6h1DKiahzSZSp5ZYRDBWZ3LNl1FTFsSC5AOYxicaHTyDJgKHVqo07CiaziazrCRCsZn7vn9kNklibbUElM/640?wx_fmt=gif&from=appmsg "")  
  
  
随便截点图，不可否认的是，经过我们的一段时间监测，你可以获取到的漏洞情报非常多，但是确实是可以实打实的第一时间获取开源漏洞情报，你可以在监控策略里面仅复现 RCE、文件上传、SQL 注入这种单项的，方便第一时间获取 1day/0day 漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXHTNwjBjKG3icXmfiasYmJz0mh4TdYoVYJSPwbykib1dLdQe90iaIWYoHPWPGLpfZfXJcffrFCckwpfOfKsubpZKqlcjYeBEXEshas/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXFUYCSnTNWicOaKXSdpD8XJ1ficRnvswrhEW5o2YEYgXFdeIeoB1g6aMnLGzWpib6PdkoE1gaNicJa6HeabibL52xrsEJXVn0YibNock/640?wx_fmt=gif&from=appmsg "")  
  
项目介绍  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFJlPufwBDMkXMPU7IiaPkPf7unb2lK5pfprl25vfV9Mia9vzuUICty0nY68oIYHNvFRG4ArWlhKflBUf5lEDghVhwDY9hJzNtdk/640?wx_fmt=gif&from=appmsg "")  
  
  
全链哨兵，是集Web 渗透、安卓 App 安全测试、iOS APP安全测试、攻防批量打点、自动化应急响应、开源漏洞监测为一体的网络安全 Agent，用于安全服务交付自动化辅助，禁止用于非法用途。  
  
https://github.com/zgsfsec/Full-Chain-Sentinel-System  
  
 希望大家多多帮忙 star🌟 转发，您的 star 是作者更新最大的动力。  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXHsthiafFtQyeFFYzmU9Ep4avq7SFAcuNTovJL5zszYD3PPJJib2d5kMdoVCHeVF5ol5ug6rVRBeiaS3ibhje7E3GZ1YmuPhST6xjY/640?wx_fmt=gif&from=appmsg "")  
  
全链哨兵交流群  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXF2LPkYS8X7K2hqcbDEz70mzA50PiapiaZrQk6ZZxianKWx3ogw1oOkRueTTtCRgebvicf58HqvXssWYej9lREicp0zzQqTDOsKTKjY/640?wx_fmt=gif&from=appmsg "")  
  
  
加群条件  
  
1.Github 帮我们点一个 star  
  
2.扫描下方，加好友“打招呼”：发送“  
全链哨兵  
”   
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXGyGsfgt3voQOzS57ia5iacyIKBT3QkkQhCF7kh7ULOFjMGyv3hafcB0qVoloiclPhPPo9NtfRlR4w6fxoQMibvibzDIK6LHwY50hgc/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=29 "")  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/a3etiafIAYXGOqLicOy68hTTqx8jiarNk0aMOcE3RhXjF6EESIKKibKNMkWniaCc2SarhbLRfme4yVYVLiaeiaPic5OudzKsRzFKysk2fYicTj7iaicfDE/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=30 "")  
  
