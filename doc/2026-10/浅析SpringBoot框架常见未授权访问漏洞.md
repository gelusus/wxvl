#  浅析SpringBoot框架常见未授权访问漏洞  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-10-11 12:30  
  
前言  
  
在对 Java 站点进行渗透测试的过程中，经常会遇见各类未授权访问漏洞，本文来总结学习下常见的几类未授权访问漏洞的检测和利用方法。  
  
SpringBoot 站点的简单识别方法：  
  
1） 通过 web 应用程序网页标签的图标（favicon.ico）：如果 web 应用开发者没有修改 springboot web 应用的默认图标，那么进入应用首页后可以看到如下默认的绿色小图标：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goafiaEDomvz6yIOJXYiaoTlicxtaAU6DLBH4ZeKzvcdkCXVDYDrKoQJ4RiaMpKYWXz9l5MEFlRKaexSz5w5lCCDia3woFl0vNfKcKM/640?wx_fmt=png&from=appmsg "")  
  
2）通过 springboot 框架默认报错页面；如果 web 应用开发者没有修改 springboot web 应用的默认 4xx、5xx 报错页面，那么当 web 应用程序出现 4xx、5xx 错误时，会报错如下（此处仅以 404 报错页面为例）：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grf0SxMicuXP5gJDNDARm7mwnB6NNA308KAg5ialsKXRArYWcjOQjFGibYv5567PQw5uWbrIfEibgF8YGOicMQ17iaodVMiacvicYlst20/640?wx_fmt=png&from=appmsg "")  
## Swagger未授权访问  
  
Swagger 是一个用于生成、描述和调用 RESTful API 接口的 Web 服务。通俗的来讲，Swagger 就是将项目中所有（想要暴露的）接口展现在页面上，并且可以进行接口调用和测试的服务。  
  
在平时渗透测试的的时候，经常会发现 Swagger ui（swagger-ui 是将 api 接口进行可视化展示的工具）接口泄露，如下，在这个页面中暴露了目标站点中所有的接口信息，所以可以对这个接口进行漏洞测试，看是否存在未授权访问、sql 注入、文件上传等漏洞。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqtKkQUo9DQygm3ZSl9qGrwDqhpcvNCiboQ6VD58X2s88Xc9p9hsHl3sU8To2vKk8SZkbzaDt68McZfN2eenOBVIIYsbuatUX0o/640?wx_fmt=png&from=appmsg "")  
```
```  
  
修复 Swagger 未授权访问漏洞的方案：  
  
配置 Swagger 开启页面访问限制；  
  
排查接口是否存在敏感信息泄露（例如：账号密码、SecretKey、OSS配置等），若有则进行相应整改。  
  
Swagger 未授权访问地址可能存在于以下默认路径：  
```
```  
## RESTful API 设计风格  
  
REST，全名 Representational State Transfer (表现层状态转移)，它是一种设计风格，一种软件架构风格，而不是标准，只是提供了一组设计原则和约束条件。RESTful 只是转为形容词，就像那么 RESTful API 就是满足 REST 风格的，以此规范设计的 API。  
  
介绍请参见：《一杯茶的时间，搞懂 RESTful API》、《一文搞懂RESTful API》。  
  
https://apifox.com/blog/a-cup-of-tea-time-to-understand-restful-api/  
  
https://www.cnblogs.com/bigsai/p/14099154.html  
  
举个例子：提供一个订单信息 API，早期程序员为了更方便传递信息全部使用了 POST 请求，使用了定义了 method 表明调用方法：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grpY7uj1H8F6xXCGTCoqHCntS6yvrTRd82TzOK68Hr8eMmy7PVcPn5QTiadsLQ3VkxfCGhMUnQs8SppqjSop3icTIBwHOv51O6Nc/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpROCIrFal5KNJKBIN7asX37nVBicJ8rqDJ4c2vK2H00gBg0Pa1wZSlf24NQALQxuZLW8huL0K26Ywm6NE5qib1Nria2aS7n0p2eA/640?wx_fmt=png&from=appmsg "")  
  
现在来看上述例子会觉得设计上很糟糕，但是在当时大量的 API 是这样设计的。操作资源的动作全部在数据体里面重新定义了一遍，URL 上不能体现出任何有价值的信息，为缓存机制带来麻烦。对前端来说，在组装请求的时候显得麻烦不说，另外返回到数据的时候需要检查 HTTP 的状态是不是 200，还需要检查 status 字段。那么使用 RESTful 的例子是什么样呢：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpmAJNia7oic5WNPV0gwD7S78mEfULtP6ZnjOmfViaTh2cOnY0xrrj09U5gPalopIJibwyJcd2W2IalXm1YQp9kibpXNuxHOg9vNeCU/640?wx_fmt=png&from=appmsg "")  
  
上述例子体现的 RESTful API 的优点：  
## swagger-ui 未授权访问  
  
下面使用 Java 安全综合靶场搭建 Swagger 漏洞环境：  
https://github.com/JoyChou93/java-sec-code  
，直接下载源码后在虚拟机基于 Docker 快速搭建：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqfKmiaHxOmyWdVibmD3g55r0FMuiaEyXTIBwNHZ7Eyqf9B2iaXcwNkr6XnwTk3ia40hFibU0eukNbib1iaQxLjB6ciacIS2monbWXFI8fo/640?wx_fmt=png&from=appmsg "")  
  
实际上 java-sec-code 靶场并不存在 Swagger-ui 未授权访问漏洞，需要用户登录以后才能访问到 swagger-ui，此处我们假设无需登录即可访问：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1go8GxPIibFbtbWu9aSK8XkC8E09aTfsPYWJnoEwdnNFVDozicgfa4FqKHicuaPuC432XZm3ibufloQu4gFplzG0kQWu8qZQATzpKII/640?wx_fmt=png&from=appmsg "")  
  
路由 swagger-ui.html#/ 列举了所有的 api 接口，以 login 接口为例：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gprMZuiaG4lS1NPsicIF116icDQqdGqwxYknicjfpK2Lw6Dg4Hib2TIXlje6l2jibqvA5H7Uhq32uyzibYUFP6T4A7A5z75c7gmpTTQHw/640?wx_fmt=png&from=appmsg "")  
  
点击 Try it out --> Execute 可以调用接口并查看服务器返回数据：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqRK3jLjYMQR6h3V9OaSDonyPicsEOzQv532SNt36t4tnpicrYC7ty2VKyKUZqhZ5SRY0kkok5HLS5n67Wcqq4PZ5GSUPV9lZCwk/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grRWibqNo5KFqHibjq3UUsNezVFn94zyDzwicv0SYyiaGU3ywNRYfHPJ7bmopYArl0sibkiaCV59sYAFDozLA1cneOVIcdic6STAib4ia9Q/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqjKxICal9KZiaIQFp8w5AWyb9HuL5z436ewISfibGP3guaKiaaLgNrAagevuSoPkA7CBHdFb6zZ0CBuaBicmicYtO0k6r6WGfGA13s/640?wx_fmt=png&from=appmsg "")  
  
Swagger-ui 提供的 Restful API 接口也会同步提供相应的，发起模拟请求时可以填入具体参数值后再发送请求，比如 SSRF 漏洞示例接口   
http://ip:8080/ssrf/urlConnection/vuln?url=XXX  
：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpEGsx9LyxJPp3kEM278MLNSRZmp7U51IM6mYlv6VHWms65D7hk59zN2euvSc128wDrDl1ibjBTTeTicEicEgqGiauEartgSLiaex4k/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1go6kBdONvXl9kfdGXMicNxA6d4mSciahPNWbeDyPhIl3qkfUZNFBY8t27hB5ZKJ73A0L68NF6LRW3icavvAZqTFTpwh7btur4f7wY/640?wx_fmt=png&from=appmsg "")  
## swagger 接口批量探测  
  
通过访问 api-docs 或者 swagger.json 可以直接获取 Json 格式的全部接口文档：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grVGpWfDQYKK09tDXT3N9Xma4icWj8zicPed1vicmFoSxDqJzHCWUSC3LM1xQb5aBSib5O0ic0UR6Fia4ibWgjL4weeCPrkZ0nUVywiayQ/640?wx_fmt=png&from=appmsg "")  
  
具体地址可在 swagger-ui 页面上找到：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grEHQKricO3SjZicMClGTcibdjvAGJvrYBSvadvjBUMWMSIg3admzdzsGTPIMqqnaxIZ1JHWCTlbdpmTGGEzbmyhDEicjGyHbFU2Oc/640?wx_fmt=png&from=appmsg "")  
  
在目标系统存在大量 API 接口的情况下，逐一进行手工测试的话会消耗大量精力，可以使用一些现成的自动化工具来快速完成此项工作。  
### 0x01 Postman API工具  
  
第一种方案是借助 Postman 工具（或者 Apifox 工具：  
https://apifox.com/  
），利用思路：  
```
```  
  
具体演示请参见：《  
Swagger ui接口自动化批量漏洞测试  
》、《  
Actuator内存泄露及利用&Swagger未授权&自动化测试实现  
》。  
  
Postman 导入数据总是失败，此处用 Apifox 应用导入目标 Swagger API 接口：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1go9lbQ8fclM5TQvIwG2dtmHmYZZHHvOxhQ1JZV4tlZYJVdEkd5ibbaVyTstOVrJbgrCYuWx0AiaArMPxV8a9XSPuST3lLDojOHcc/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqJmrh7K5dCfvGd31UdiaGkS1FQPswlicQW4n6aHmzricRmJGJjr5LK35wK5D21lZmQ7jN55CKUvB4IxGpbO6ZT0RFAmDP0LicZB7k/640?wx_fmt=png&from=appmsg "")  
  
发送测试请求（下面是单独一个接口，实际上可以批量发送请求）：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goXxFXptFJ8O1JvP0S1cZbZMbFUjpP6EqIDJNwv9wRFY2WOzc1wErrEicMgt8YGVCl3s72K5ex65J1GWYAHI8kGlxviaNiapBWia9E/640?wx_fmt=png&from=appmsg "")  
  
接下来设置网络代理将流量转发到 Burpsuite：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gooib6ic5DcVic9jg91rTQRMxcR1jBfUzicib9QiaENAUq5OzdCVDELf6AEqbMwibeOu47buMGiaTf7YIucS0EdGUsiapzAmfqRaaIQhno8/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqZFicVLykpQXNichWHYPjtibSV6NaqHymxicZouYtcJ02bc8m0RDViaia26fpHQ2X5a1WERaiaqQwMZ6uQsC21n3OUyLRTpfXol5KuvI/640?wx_fmt=png&from=appmsg "")  
  
BurpSuite 联动 Xray 进行漏洞扫描就不演示了，参见 Xray 官方文档即可：  
https://docs.xray.cool/  
### 0x02 Github开源工具：swagger-hack  
  
项目地址：  
https://github.com/jayus0821/swagger-hack  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grklLvp981m292ryWgYzQDJ5gKk40gmsKkWzkt4ZKjIvlBsX54mlIyGzlodiapibwdM122XhOAvLCK4ss9xMLYe9EnIK1ye2RrMI/640?wx_fmt=png&from=appmsg "")  
  
会自动构造参数并发送请求包，同时记录返回数据到本地统计表格：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gowQbKAfickBxic7G1lC4st91eOvsFSC3jWHgR0MhVcG5Y9LP9BMZVlF2piaFia5Ud5EchO9X1c4jTRkSmibjTFMnCFiblVKpbWkoC4k/640?wx_fmt=png&from=appmsg "")  
  
【More】 swagger-ui 还存在过 XSS 漏洞，详情请参见：《  
Hacking Swagger-UI - from XSS to account takeovers  
》、《  
渗透技巧基于Swagger-UI的XSS  
》。  
## Springboot Actuator  
  
Actuator 是 springboot 提供的用来对应用系统进行自省和监控的功能模块，借助于 Actuator 开发者可以很方便地对应用系统某些监控指标进行查看、统计等。在 Actuator 启用的情况下，如果没有做好相关权限控制，非法用户可通过访问默认的执行器端点（endpoints）来获取应用系统中的监控信息，从而导致信息泄露甚至服务器被接管的事件发生。  
  
【影响版本】  
```
```  
  
Actuator 提供的执行器端点分为两类：原生端点和用户自定义扩展端点，原生端点主要有：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpnVbV7V0TcvvGIBhCiblW28Ucn6pD38iarcCSrRFlo1RCSpUduagIPYaz1R3iaRSwfePj4PGqLeg22jia4XhEEzby1cYB44mBqwXw/640?wx_fmt=png&from=appmsg "")  
  
需要注意的是：  
```
```  
  
Actuator-api 的官方文档：  
https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/  
，官方文档对每个端点的功能进行了描述。  
```
```  
  
未授权访问数据利用  
  
对于寻找漏洞比较重要的接口和其可能的利用方式可总结如下：  
## 未授权访问数据利用  
  
对于寻找漏洞比较重要的接口和其可能的利用方式可总结如下：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grAjQpicqNDwj66EmUBAOu1MjjFlC7sV9D6QKxtia21R4PaM1Frhj4iaiabvwJWCSfwK8jDw2QYrqfeDPGBFF6F1SwficVysyz1XlxM/640?wx_fmt=png&from=appmsg "")  
  
下面使用 Vulhub 靶场的 CVE-2022-22947 环境，存在 actuator 未授权访问：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goticyW7wO08QUEZzgbsIsSEZiasd2WnExMYaMQvRmp9DCFdVG9fZWAAT7iaaKVoiblPibTDIArnkMT88SHAHzwhntXD5aSm6AXAqdU/640?wx_fmt=png&from=appmsg "")  
### 0x01 /actuator/heapdump  
  
先来看下 /actuator/heapdump 接口，访问后可以下载到一个 hprof 格式的堆转储文件 heapdump：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goO2bKIVwibv2LOzcyFdnvPkKBQ9q5OCXRFkSeP4ZXLIHu7AsBQSO0XibgnPJTGGtpTsUWp6PatOCXwNAOF9VZaWMZx1KXAz5OKI/640?wx_fmt=png&from=appmsg "")  
  
heapdump 文件的敏感数据查看工具：  
```
```  
  
第一款工具 JDumpSpider 食用方法很简单：  
```
```  
  
会自动识别、提取敏感信息并进行分类，可惜本案例没有什么敏感信息：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpjl70BJ4m3wTePtRc4SDervDg7wes03fxHGRQfEc405ux19yiauyNLZwQUabJR9lESKsIJyHJna5rof72a5pfnqBW7ncdajXZE/640?wx_fmt=png&from=appmsg "")  
  
但是 java-code-sec 靶场的 heapdump 文件（请求：  
http://ip:8080/heapdump  
) 则存在敏感数数据泄露：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqFnhFV7BdRX4nENT7xslNPSW2UakjThR2hTKUBNAF2U7Bcl3fncRgQrWibkjz7PwDGJBQdcdibpF5O4z4wjBHhfYFxjzBAymya0/640?wx_fmt=png&from=appmsg "")  
  
第二款工具 Eclipse MemoryAnalyzer 需要 Java 17 以上版本，用法参见：《  
Springboot信息泄露以及heapdump的利用  
》。  
### 0x02 /env 路径敏感信息  
  
上述靶场环境不存在敏感数据，以下借用网上案例，/env 路径泄露多个敏感账户密码：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqtiaG2hdwK3fWEKPvQl66xmJ0XiaMEiaibwZA4NCEubeKArloLRcb7SvolUVPmMic2nHqlccNmXibbYLyeLcMr1qjnzXuhicQicL4icDbg/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goeV41Nr4QUy9kXoQ9TdnPTmmia8J3CYmBXaSSGDVZLahC4SZUFply5zevh2icqaETS0MR3eTTSSubLzca6lSurvNJOCRUicFfdOU/640?wx_fmt=png&from=appmsg "")  
### 0x03 /trace 路径泄露认证凭据  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goC3SZiaHh1zztM3J5ScWnYnvtDsRZjph9AzbRJQs8CQtXYZORnuVBicPGKCBVicAtpS1byllxDjklYxWaVqAkcuNGCZ8ANe1nGb8/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpbEnPNsWia4Kica9dJmA8YpQTZnJr7Ad3H44FjYFibARVyPBBuOWjpYcW2geoUvXXlpjqeY2IryoVpDib2icmYib0iaUv2jtWS7rFTvo/640?wx_fmt=png&from=appmsg "")  
### 0x04 /health 路径泄露 Git 项目地址  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1god5Rxnkl1GlXsHn7ia418e3fX6yQ1B8efzibrTGBibpf8467OntZBog2DNWibFmKRJdIQdI4kFCK0icGgWDuxmC6DD4TwAaMicLWAWI/640?wx_fmt=png&from=appmsg "")  
### 0x05 /mappings 路径泄露所有 API  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqJIUsRhq0WkYeXgicGTlElNSJCk9Q5j8BLAOj8WAjhk44L5PXr98QIbe9Ulajz0AgIh3fPW2CKaHFKS6ibEqy940bSZA0NCEqBQ/640?wx_fmt=png&from=appmsg "")  
  
【More】SpringBoot Actuator 未授权访问漏洞导致的危害不仅仅是信息泄露，结合其他相关组件漏洞可以实现 RCE 的效果，可进一步参见：  
  
https://github.com/LandGrey/SpringBootVulExploit  
；  
  
奇安信攻防社区-Springboot攻击面初探  
；  
  
Spring Boot Actuator 未授权的测试与利用思路  
。  
  
相应的靶场环境与复现方式位于 SpringBootVulExploit，这里头实际上涉及多个 CVE 漏洞，此处暂不展开分析。  
## 未授权访问防御手段  
  
作为一名安全dog，不能只挖不修。  
  
【方案一】  
  
在项目的 pom.xml 文件下引入 spring-boot-starter-security 依赖：  
```
```  
  
然后在 application.properties 中开启 security 功能，配置访问账号密码，重启应用即可弹出认证界面：  
```
```  
  
【方案二】  
  
可以在 application.properties 配置文件修改配置，开启业务需求上必须的端口（建议全部禁用），通过配置 management.endpoint.<端点名称>.enabled 为 true/false 来开启/禁用端口。  
```
```  
### 漏洞自动化检测工具  
  
SpringBoot 框架漏洞探测工具：  
https://github.com/0x727/SpringBootExploit  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gofq8sKbc2FQ2yQWJY3KHpJmCQkTyLq1Dv92R7zbgqiaUK2s9pVRibM98XSzoH4qOgM45vU9y06WIBYwgC2HfEqHka3LqHiaWdPGE/640?wx_fmt=png&from=appmsg "")  
  
一款基于 YAML 语法模板的定制化快速漏洞扫描器：  
https://github.com/projectdiscovery/nuclei  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gozWI1Z4y09Zd58aibQJH1x3CcCI5mzCibFbrf0XRauZnfme31a6Z0zgicwhRzFYxEvwBmt7ShTqLTicBnxGq1MuXjQQ3FQZZiaffoI/640?wx_fmt=png&from=appmsg "")  
  
一款快速、稳定的高性能漏洞扫描器 afrog：  
https://github.com/zan8in/afrog  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpKcYuyeqC5iccynXYoCy1B8PjQDqicJpqKQlbBMCR72lYDmAfDkKvsDU8v6ElxGYibFrgE3AXRibautMyLneMqurVFSZhaLC1lNyk/640?wx_fmt=png&from=appmsg "")  
### CVE-2022-22947 RCE  
  
Spring Cloud Gateway 是 Spring 中的一个 API 网关，旨在提供一种简单而有效的方法来路由到 API 并为其提供横切关注点，例如：安全性、监控/指标和弹性（英文直译，可参见官方介绍文档）。客户端发起请求给网关，网关处理映射找到一个匹配的路由，然后发送该给网关的 Web 处理器，处理器会通过一条特定的Filter链来处理请求，最后会发出代理请求，Filter 不仅仅做出预过滤，代理请求发出后也会进行过滤。  
  
Spring Cloud Gateway 在其 3.1.0 及 3.0.6 版本（包含）以前存在一处 SpEL 表达式注入漏洞，当 Gateway Actuator 端点启用、暴露且不安全时，使用 Spring Cloud Gateway 的应用程序很容易受到代码注入攻击。远程攻击者可能会发出恶意制作的请求，从而允许在远程主机上进行任意远程执行。  
  
受影响的 Spring Cloud Gateway 版本（当前官网版本已达 v4.0.9）：  
```
```  
  
漏洞披露与分析文章链接：  
https://wya.pl/2022/02/26/cve-2022-22947-spel-casting-and-evil-beans/  
。  
  
漏洞原理分析与复现  
  
从 Spring官方的修复代码 可以直观看出这个 SpEL 表达式注入的位置：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grA8ibrXariamFwTrALHMgjt7mqJHF4EmRcWthHFSxTibLMeu4GyGRacBBOzx0NibT2PGudy6mEqxrYvqIhPlLY3jLIE7YuVH7JNicQ/640?wx_fmt=png&from=appmsg "")  
  
修复方法是使用安全的上下文 SimpleEvaluationContext 替换了不安全的 StandardEvaluationContext（SpEL 表达式注入漏洞的相关知识可以参见我的另一篇文章：《  
Java代码审计之SpEL表达式注入漏洞分析  
》）：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grANFeK1LxGOL5a4osTUIpUah6GlQgnfEia3cnmaqpNIMGEiauR0EYYvq55XMDqJ7IibP0DEvlTDibPWtO0UREwkQGYlVETZ46EuzE/640?wx_fmt=png&from=appmsg "")  
  
至于入参传递到漏洞触发点的过程请参见《  
Spring Cloud GateWay 远程代码执行漏洞(CVE-2022-22947)  
 》，此处不展开。  
  
直接使用 Vulhub 靶场环境和   
指导文档  
 快速进行漏洞复现，前面已经提到了 Vulhub 靶场的 CVE-2022-22947 环境，存在 actuator 未授权访问且启用了 Actuator Gateway ：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpt4hSMCVknYJtto9atn8iby39EnlLkVAPkjNvoQRlUqdL7y3hEUicrB759Dwor0ADGDzyShZ47bMgMtdbQ8ibfuzpr6GWgbXaYTc/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gricBiaIzGPkgvu604NJdruBIUyH2yeFetN3dwRLlsqBywezv3eBf62GTgBGhkcL7dfzIbzm2WL59mXgLOeGYk7RklCrmTJHlvU4/640?wx_fmt=png&from=appmsg "")  
  
从官方文档   
https://docs.spring.io/spring-cloud-gateway/docs/3.0.4/reference/html/#actuator-api  
 可以看到，如果配置了暴露 actuator 端点，允许 jmx 或者 Web 访问，则可以通过 /gateway 接口与网关进行交互，并创建新的自定义路由。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpp2g3eic1C8nFF7OsWgKibibrNBtahhGdNn1jvPFx1HABYnQibVv6hYRY2skaaicc52AUkO1YD5yMjkVCW84c7UPgpXXy0rHxqeiawc/640?wx_fmt=png&from=appmsg "")  
  
利用这个漏洞需要发送两个 HTTP 数据包。  
  
首先，发送如下数据包即可添加一个包含恶意 SpEL 表达式的路由：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grjibyibZ9EtxTM8nMUOApkaRia8ML7o5cdv6L1nKLAJmS4Cng8ibOQJWQyhelvrHsepyGqRVmaJichGuQiakn6s2HCG7INAtzqnsbZs/640?wx_fmt=png&from=appmsg "")  
  
然后，发送如下数据包应用刚添加的路由，这个数据包将触发 SpEL 表达式的执行：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqtQRtT48wJAnJXRoCaJJ68TvZE9kMpmox6UbPuwMb7RX4ocB6fdz7yuNn7PcdyHiazufRedz3uhdiaWgqlCO2csdAQ4wujW0NmQ/640?wx_fmt=png&from=appmsg "")  
  
最后发送如下数据包即可查看执行结果：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpFKBrMa9XSw3faYHlHhvqdympxgty1pASVKTlGQFj17hWaSpGlK7qV1BBibQmo7yE1faTrpRsRTSpBhEVcX4tWEdkXsW0ibDKV8/640?wx_fmt=png&from=appmsg "")  
  
上面的演示只是为了简单起见，也可以将恶意路由的 SpEL 表达式替换为 basse64 的反弹 shell 命令。  
  
【漏洞修复方案】  
  
除了升级 Spring cloud Gateway 到最新版本以外，也可以禁用 actuator gateway。通过前面的漏洞利用过程可以看到，首先需要通过 /actuator/gateway/routes/{id}API 创建一条路由。因此将此 API 禁止，也可实现漏洞的修复。根据 Actuator 的 API 文档可知，启用 actuator gateway 需要设置以下两个配置的值：  
```
```  
  
因此只要这两个选项不同时满足，就不会启用 actuator gateway。  
### 漏洞自动化利用工具  
  
沿用前文提到的 SpringBoot 框架漏洞探测工具：  
https://github.com/0x727/SpringBootExploit  
：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqzOd3K1tOlomYFbWCa92hzNMfEFqTc2F6vvFUqjibANibTZAcMjsKiaiboxwVp3rJoU8GvTicniaQ0GXFOYvbgOcHvlAOycRCcXK6QU/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpuKIbiaa7q4K6nE8jv9P3y9gsPpWIwrTjUqMjEHApo7Jh9adaW7awhnsy3z1TgJRfiaUgiaVny3kdDic5V8exc7ZqEoiaEBJ7dRp5U/640?wx_fmt=png&from=appmsg "")  
  
此漏洞扫描和利用也可以使用 Goby 社区版：  
https://github.com/gobysec/Goby  
，Goby是一款基于网络空间测绘技术的新一代网络安全工具，它通过给目标网络建立完整的资产知识库，进行网络安全事件应急与漏洞应急，官方文档：  
https://gobysec.net/faq  
。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1graYfHBqaId1VcqYEH2vXNsLBIdhXQGHEiaRO4V6ZcXKvpxlFvUqjRx2omInibKvtd8CsSUZicqTzZjgeA8p7EH62yKsoYZCAwI0w/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goBT8G16UC3OxsKX8EwhaW7cXDFj9BicyCIY3n2s6IMdOhp8xqywj4QrpzZeYCw8tiaoB8iah1C5vGiczdSjL8JHRiaCEPWLPDpaCxc/640?wx_fmt=png&from=appmsg "")  
  
点击扫描出来的 CVE-2022-22947 进去快速验证漏洞：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gr29YCicr44SLMtBGupYdBicp6YKmacXPaDY6tFNZzMIMYib1aeWCBBPTgJ9WLPYVCrWMtlLibIZo3fBQbeebeCiaoh7AMrd2AghaWo/640?wx_fmt=png&from=appmsg "")  
  
Goby 跟前面的 nuclei、afrog 的共同特点是基于内置的漏洞 poc 扫描模板库，且都支持使用者自定义新增的 poc 规则。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gooZ0Rrlr46TBqlGjE0DiaD7ibwxveGnhFCBQHVjaBIcslIlUAGSbbQ3569qE2NAbT0NDTYx0WZHWhqcqwdwNXFCJSPDlD6JJPdA/640?wx_fmt=png&from=appmsg "")  
  
优势是 Goby 提供了较为丰富的图形化界面、支持漏洞快速验证功能，以及支持丰富的扩展插件。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1greic91BWTadxTQp6uicD4u5ESzNRCicz5icRW9GOaPGP8dhzp2YjDAnDor78jRIkvu3sic4xrqiauaFvXcm3KECq7aCatwlsq4VqWmA/640?wx_fmt=png&from=appmsg "")  
  
缺点也很明显，专业版收费很贵：  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpHaLCIPbZFbTH3KeQjoRMibYOKZ1ibfIibq7ExG8FoKicU4Ovhz04KDshvOQHwg5ffhSqDVt7NEaiagoibcqjibVm1JibWqsa4bPkECvg/640?wx_fmt=png&from=appmsg "")  
  
### 其他常见未授权访问  
  
网上总结的常见的渗透测试常见的未授权访问漏洞：《二十八种未授权访问漏洞合集》，下文仅挑部分展开描述。  
### Druid未授权访问漏洞  
  
Druid 是阿里巴巴数据库事业部出品，为监控而生的数据库连接池，官方项目地址：  
https://github.com/alibaba/druid  
。Druid 提供的监控功能，包括监控 SQL 的执行时间、监控 Web URI 的请求、以及 Session 监控等。  
  
当开发者配置不当时就可能造成未授权访问漏洞，即项目中引入 druid-spring-boot-starter，且 spring.datasource.druid.stat-view-servlet.enabled 配置为 true 时，可以直接访问 Druid Monitor 监控平台，可能会造成企业机密信息被攻击者获取。  
```
```  
```
```  
  
漏洞默认路径：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grRbMxJI3YLuicfy4Uia514oOQgnhJ6y8u1M3vfdou192NtZLJLs5ZwCC9WCLFIAazRUickwLHPnLUBcspWHzicGTSIgNFmOwVAFicg/640?wx_fmt=png&from=appmsg "")  
  
Fofa 资产搜索语法：  
```
```  
  
Google 搜索语法：  
```
```  
  
【漏洞利用】  
  
Druid 未授权访问在没有进一步利用的情况下仅仅是一个低风险的信息泄露漏洞，但是攻击者也可以进一步利用利用该漏洞来提高危害。  
  
首先可以借助未授权访问收集一下服务器的 API 接口相关信息：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1golicsEtLJFfYeLMD7cia35TxZecDJVQUuBAUX7bL8YjoRpIWb28pe6GzSlnOPic9Ah6PwKqsWUuzZQBSvPPKAYsktUPEkAnyQ8ac/640?wx_fmt=png&from=appmsg "")  
  
该接口泄露了网站功能模块的 API 接口，可以进一步探测这些 API 接口是否存在未授权访问、SQL 注入、XSS 等漏洞。  
  
与此同时，应当关注可能泄露用户 session 的路径：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpFllxfxffuiaUYapEAZGm01u097mQsML7syHANiawXN39t7XaUCuMECXdzANWah7kAwLByoAuVvEgicT5lcI2T3qC41iazAAwVh0w/640?wx_fmt=png&from=appmsg "")  
  
此处泄露的主要是登录用户的 session，不管是登陆成功的、没登陆成功的，还是失效的都会储存在这里。  
> 【漏洞利用思路】当 /druid/websession.html 页面存在数据时，我们可利用该页面的 session 伪造用户身份访问 /druid/weburi.html 泄露的 API 接口路径，来进一步访问敏感业务接口，甚至登录系统后台。请注意应当点击最后访问时间，然后复制一条离现在时间最为接近的 session 进行伪造登录，尽量避免因 session 已失效而导致利用失败。  
  
  
拿到 session 后登录后台的方法可以参见《  
Kali Linux-BeEF浏览器渗透框架  
》中的 XSS 利用章节，网上成功借助 Druid 未授权访问窃取 session 并登录后台的案例：《  
Druid未授权访问实战利用  
》。  
  
【修复方案】  
  
方案一：直接禁止页面访问，SpringBoot 项目修改配置文件 application.properties（或者不配置，此配置默认为 false）。  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gprmULtWeWpk29Une8KPBAaLZ4LgFZFrZEVJcRt4a6FtCQibDEV2icmYIic2a1SfPOzIXC2rW2gw01rHJkIOoQ8qZKSsGqRPIEtWI/640?wx_fmt=png&from=appmsg "")  
  
方案二：增加账号密码登录，账号密码可自定义配置，与数据库无关。  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grV9trYn4PXKjRKujJdwKXJE8YJiaamn9ibodKfz2e2y3QRzHtibu42oDoNVyDrBSiaKVLb1Ex9NBSs10vcgRica9Vb2xyfc8dicz3fA/640?wx_fmt=png&from=appmsg "")  
### Webpack 源代码泄露  
  
Webpack 是一个强大的前端资源模块打包工具，可以将多个 JS、CSS、JSON等 文件打包成一个或多个文件，使代码更加模块化，便于编程和使用。目前前端部署的代码一般都是经过 webpack 压缩的，压缩的目的一般如下：移除无用代码、混淆代码中变量名称、函数名称等，以及对结构进行扁平化处理。Vue 使用 webpack 静态资源打包器的时候，如果未进行正确配置，会产生一个 js.map 文件，而这个 js.map 可以通过工具来反编译还原 Vue 源代码，导致前端源代码泄露。  
#### 0x01 SourceMap 的作用  
  
SourceMap 在其中扮演了一个十分重要的角色，用来作为源代码和编译代码之间的映射，方便开发定位问题。一般在压缩 js 的过程中，会生成相应的 sourcemap 文件，并且在压缩的 js 文件末尾追加 sourcemap 文件的链接 ，如：  
//# sourceMappingURL=xxxx.js.map  
。这样，浏览器在加载这个压缩过的js 时，就知道还有一个相应的 sourcemap 文件，也会一起加载下来，运行的过程中如果 js 报错，也会给出相应源代码的行号与列号，而非压缩文件的。总而言之，Sourcemap 初衷是方便开发排错，它不应该用在生产环境，如果用在生产环境，攻击者就可以通过 sourcemap 文件中的映射，还原出前端完整代码。  
#### 0x02 Webpack 源码泄露危害  
  
Webpack 前端源码泄露漏洞可能导致以下危害：  
```
```  
#### 0x03 Webpack 站点的识别  
  
前端打包工具 Webpack 所生成的是一个纯 JS 的网站，即源码之中并无多余的 HTML 内容，所有的前端内容都依赖于浏览器对 JS 文件的解析，并且此类网站之中通常存在一行 noscript 提示告诉没有启用JS的访问者启用 JS 功能才能正常浏览网站。  
  
以雷神众测为例，它便是一个标准的 Webpack 站点（但是不存在前端源代码信息泄露）：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpE4f6HViboZS0J8wAxtxYEh3z7a3KhYlXZyhyAmEtLqAEHqp1tdRfHFxMTsRs208QHqneyGgzhVqQQO4hlJmOdBxz8gx3awAjM/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goicHZkbbYIkVz5vudDqibOxEpb7WmLvlS0ic9TfRs8AYV1ZaJ7nib1zee1xs8E2AFf7xFXOPEeIZwbtm5uxVIDn54NviccI2RhanIQ/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gq7TUkgvtnrx5wvw9ibJ5OsaSotNcicia9ZS0q5QBYVBNRTGZubWwibBIHHmaIR4Ow22Lg5bJSaGZM0SuuzPoQx3rmicnwnkTYGsBlw/640?wx_fmt=png&from=appmsg "")  
  
而在 webpack 项目源码泄漏的情况下，则可以在浏览器控制台中的 Sources->Page->webpack:// 中查看到前端源代码：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqU67u2JJK20CjXIhhcQUA7xgeibUxFcJwThg8n1vloic6KdGYSBIN1s1AGh9GnVUnekvbam7ExXeDfvESTg9NpwLXT1y68dyTtM/640?wx_fmt=png&from=appmsg "")  
#### 0x04 Webpack 源码泄露利用  
  
举个例子：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpydmK4k67iaiaQ5jFv4rnCViatG0jHiagAWib0saMqqknT3A11JKvJL2qRxl7oTQTW6TZBjgJIdmpuZQiamYZDyoHRNBaJsxrenToTk/640?wx_fmt=png&from=appmsg "")  
  
直接查看网站的 js 文件，可以在末尾处看到均有 js.map 文件名：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpQlqx3hxceaJnLqV62h1xWibFibprsnXpLt1ya6ib8LFyMYZ87R2Ol6fOicqApicicCgBxgk0uXb2wib5CfsygLjEUHI9U2XFEqsuLc4/640?wx_fmt=png&from=appmsg "")  
  
手动下载 js.map 文件：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goxExvd5la4HDGL1pCQOo8Kj8IYAwicicHmRLibTR1xPaafyJQzFD4RnCMC463WZQUfg8HicsMpCjzgiaB6icnOM0rYxgLAzBxJiaQkLY/640?wx_fmt=png&from=appmsg "")  
  
安装 npm（Windows 下载并安装   
https://nodejs.org/en/download  
 即可），使用 npm 安装 reverse-sourcemap：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqGO941BBsdQy8YySOAIjLniahmxUGuLnu3sAFXngNg5nO04Wo00UOg8sjhVhEqn1o2ah8S6R3EiaH9NhmiaI9Xfq0ibrnpUPQV160/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goeDaWPZl3FDmS4B7OLAvjhricxibUXW9c8IHExibyZnnGic6yE1OoLhDT4z0HgicX7ic8VqIb5WZ254OroP39UCJpckjvZR23UAoZKQ/640?wx_fmt=png&from=appmsg "")  
  
接下来即可使用 reverse-sourcemap 进行 js.map 文件还原操作：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gq4kRwxArMSyeS1vEftViaSBHkia8N0eQ8q7picJDFH1QKo4mxYpEjwPpklAsoffDG63ianrqPlrRCqkcCVKMNcuiciaEUCP1iaGOD69U/640?wx_fmt=png&from=appmsg "")  
  
随后拖入 VSCode 查看源代码即可：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpXks898X7cicJr3f1vILIuaFHNGPLkbaV2OpMTadlKS6icea3M9cAuP6WvWgiakvSg8SxyL956SorvtlOic5pky81ckicjHwdZz4HY/640?wx_fmt=png&from=appmsg "")  
#### 0x05 Webpack 自动探测工具  
  
【检测工具 1：HaE】  
  
BurpSuite 开源插件 HaE （  
https://github.com/gh0stkey/HaE  
）支持检测 SourceMap 文件：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grEYGygyKibSsMZGdlkTRFZeWdd12IIU1QicZzYNJOqh7fNicroC1ia9bYHI5PfAcJhEpz6ib7KjkluUlkPGXANcricaGH5UzekBJR3w/640?wx_fmt=png&from=appmsg "")  
  
HaE 是一个基于 BurpSuite Java 插件 API 开发的辅助型框架式插件，旨在实现对 HTTP 消息的高亮标记和信息提取。该插件通过自定义正则表达式匹配响应报文或请求报文，并对匹配成功的报文进行标记和提取。  
  
【检测工具 2：Packer Fuzzer】  
  
Packer Fuzzer 工具：  
https://github.com/rtcatc/Packer-Fuzzer  
。Packer Fuzzer 工具目前支持自动化提取 JS 资源文件并利用现成规则和暴力提取模式提取其中的 API 及对应参数，在提取完成之后支持对：未授权访问、敏感信息泄露、CORS、SQL 注入、水平越权、弱口令、任意文件上传七大漏洞进行模糊高效的快速检测。在扫描结束之后，本工具还支持自动生成扫描报告，您可以选择便于分析的HTML版本以及较为正规的 doc、pdf、txt 版本。  
> 但注意此漏洞利用工具请注意需要单独执行 pip install python-docx==0.8.11 -i   
https://pypi.tuna.tsinghua.edu.cn/simple  
 后才能正常运行。  
  
  
执行扫描并查看结果：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grK8lPvicEhmGDErVFgnv3EVp1q7mzrTSfrkGb28S5PlPHILVhJyEy7joTEGMibg53ppsk0MGN6IzGsGklHsic6ia1uicoLaCZdEcdE/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goj7uvLztSeVyJQL2UT2rTahTVHW6FAxuiave1mMkbqX267ssj219zhoeg4v6ReLyPpiciccgPdWsZibvuXhwjG4xUotsXSYzS3QFw/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpI7k8DDEZQmPWZXm5ZpOwTdd9SQhxJicKZ5AYo2iaO9P36h8uK9h3nd50LZYuDjkjLicM6k7nZYn8jLbJEcLYdSkYAeZkIGJvwgQ/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grOsRMNISaCDrjUFyp9FgiaEuNMf9mZ3qXMfTjuH8WOBPJEwJNUPWVajSfo3Aqx2xoHHRU7p5K6FQLXCwKqzlrITjgLBnbbAIoo/640?wx_fmt=png&from=appmsg "")  
#### 0x06 Webpack 源码泄露防御  
  
如何防范 Webpack 前端源码泄露漏洞？  
```
```  
  
最后，附上奇安信安全社区几个关于 Webpack 信息泄露的实战利用案例：《  
奇安信攻防社区-浅谈Webpack导致的一些问题  
》。  
### 总结  
  
本文总结学习了 Swagger-UI、SpringBoot Actuator、Druid、Webpack 组件的未授权访问漏洞基本原理与漏洞探测方法，在介绍了几款自动化扫描工具的同时，也简要分析了 CVE-2022-22947 Spring Cloud Gateway 系统 RCE 漏洞。  
  
SpringBoot 安全漏洞总结与利用工具：  
  
SpringBootVulExploit/README.md  
  
奇安信攻防社区-Springboot攻击面初探；  
  
SpringBoot漏洞利用工具的介绍与实例演示。  
  
  
