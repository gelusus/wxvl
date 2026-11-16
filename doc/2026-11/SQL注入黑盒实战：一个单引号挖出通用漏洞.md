#  SQL注入黑盒实战：一个单引号挖出通用漏洞  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-11-16 00:30  
  
渗透测试里最经典的一幕：一个再普通不过的系统登录框，扔进去一个单引号，数据库报错了。  
  
很多时候，测试到这里就收工了——登录绕过、交差、下线。但从“一个系统的漏洞”到“一批系统的通用漏洞”，中间差的其实只是思维发散这一步。  
  
这篇文章完整记录一次真实的黑盒 SQL 注入：从登录框的单引号探测，到万能密码验证绕过，再到顺着 JS 文件名这个指纹用 FOFA 批量定位同款站点，最终把一个事件型漏洞升级成通用漏洞的全过程。  
## 一、黑盒测试  
  
某次渗透测试过程中，单公子经黑盒测试发现某系统存在 SQL 注入。  
  
系统登录页面大概是这样子的：  
  
![系统登录页面](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqohr1Jm7SibB3vQwvicrxQDNED4hyW31KwlqW6bmoOne5KYOuVh9Sd6QweLwz2ib0p6ictzficyXI8uMcYKmic6UQPyhficIBDf87PWI/640?wx_fmt=png&from=appmsg "")  
  
用户名和密码两处参数都存在注入。这里有个细节：相对而言，密码处存在注入的可能性比较小，因为密码通常会经过加密处理（数据库中的 password 字段值一般是密文）；当然，如果数据库里保存的密码本身就是明文，那同样存在 SQL 注入的风险。  
  
先抓一个正常请求看看基线回显：  
  
![正常登录请求的基线回显](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqERiaCoiaibP8UFF4xpYqicf1O88Rwia8TFHsia8SPBozMeMOu16HojwXLsOktC0etPQyCO3D9iauiaH7nf0SQdzadCJDGEBMMbNfOtws/640?wx_fmt=png&from=appmsg "")  
  
在用户名后面跟一个单引号试试：  
  
![用户名后加一个单引号的回显](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqtpb7nUuo63yTwJfgJYcQme2Ar3CcNPibAAOt2MRTeyg72Tatpkn1d1c5rhjtdaocjHp5F0xXdWzCichxXWcTric2YpSsTjk51Gw/640?wx_fmt=png&from=appmsg "")  
  
再跟两个单引号：  
  
![用户名后加两个单引号的回显](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gor3a4fY9XTVCH1suLzZSwf7Te14Qdicgv0SLsQObTwCAVykpicyUR1coI8KjGDMnzatLBtOpRyGSs6TBAoSUeNgmWnEYNwtzrd4/640?wx_fmt=png&from=appmsg "")  
  
对比两次回显，响应内容有明显区别——注入点坐实。当然，单引号探测并不是对所有注入点都适用，这里刚好命中。  
  
接下来判断数据库类型。根据网站架构加上经验，猜测后端是 MSSQL 数据库：  
  
![根据报错特征判断为 MSSQL 数据库](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1go8LAjpYXU9QWHp5mNbKZ3uibQFc9FiaCgGM66Ckkc62cAUhQwUibK3YYwJFbqEiaK6xujavgx7eWiaGy6cp9BR49aY8yJkzqLv2nZo/640?wx_fmt=png&from=appmsg "")  
  
MSSQL 体系下，当前用户一般就是 dbo：  
  
![查询 user 确认当前用户为 dbo](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqFfKAS9iant0XCcTppV83D5SqJbYVCR74oRZzZIS9OBgLmHsLgLh7G06Y4xvznqAt1H7qrEQkAb9QibWuTQDQtjUX5hD9sPqVvU/640?wx_fmt=png&from=appmsg "")  
  
再参考一下两个 or 的探测姿势：  
  
![利用两个 or 语句做布尔探测](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gphic8udFibENqP8SkHAXtnJAzKtGSvQaITgJGrcFaoYo2PpYovRsBYu2g9X3vWP5iadycx165EhppvRrz7icCPnjepa0SGRVxGHdo/640?wx_fmt=png&from=appmsg "")  
  
然后就是经典操作——万能密码。先记住这句话：**有万能密码就一定有 SQL 注入，但有 SQL 注入不一定有万能密码。**  
  
用户名填 admin'--  
，密码随意：  
  
![万能密码 admin'-- 成功登录](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gp8oS3WZ6Sbqg2BNUia9kBl8VYXiczbhvicE2E2FKSbdFQghcT8icD9MmOQicpr0PhvQ7aaNRDriavAYibach9E2o18O5N94W55HQNuFw/640?wx_fmt=png&from=appmsg "")  
  
或者反过来：用户名随意，密码填 555'or+'a'='a  
，同样能进去。  
  
到此，SQL 注入实锤，登录也绕过去了。大部分情况下，这个点的测试就到这里收工了：  
  
![证实系统存在 SQL 注入并可绕过登录](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goLmzOl3YNueZibmT9uliaK7YSfjsLrH872Yiau4CDoBTwzicaN25UZZWuadgvcbPOT9ftnNatcOShAM3WEficSXtz9ib7fBhLxzO0LA/640?wx_fmt=png&from=appmsg "")  
  
但思维需要发散……  
## 二、通用漏洞挖掘  
  
在挖掘通用漏洞之前，先弄清楚概念：什么是通用型漏洞，什么是事件型漏洞。  
  
拿补天平台来说，漏洞类型分两种：  
  
![补天平台的漏洞类型分类](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grt5PSnocQpYO1xj6JibAde8DntRnwcDeXic5nfRIyex9tBxic8deuwwzlK427FIWJ63Qg0DNvoRbqhD1wicb1BMQNSgaaOSYBUERk/640?wx_fmt=png&from=appmsg "")  
  
**通用型**  
：顾名思义，PoC 通通适用，类似于 0day 的概念，指第三方软件、应用、系统的漏洞；**事件型**  
：非通用型，只在当前应用存在。补天官方的定义见其帮助文档：https://www.butian.net/Help/plan  
  
回到上文的 SQL 注入：如果思维不发散，那它就是一个事件型的漏洞；思维发散了的话，说不定……也还是一个事件型的漏洞。开个玩笑，言归正传——既然要挖通用漏洞，就势必需要大量的案例来验证。  
  
问题来了：怎么寻找使用了该产品的站点呢？常见思路有这么几条：  
  
1. 图标 hash  
  
2. 页面关键字（如：技术支持：xx单位）  
  
3. url 链接（如 http://22.22.22.22/kkkk/yyyy/zzz/xxx.html）  
  
4. web 指纹识别  
  
5. js 文件  
  
6. ……  
  
但这个站点似乎没有啥比较突出的标识：  
  
![目标站点页面上没有突出的指纹特征](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqbWSL3uuXLxeaDPYmhogQraYtgYrfzicFbzvpXvUDHfu9HokPW7WYm0CibMfDicibgaXKMd6MdBT0j4c6piaW8vfIb56Y0mexicu4ib0/640?wx_fmt=png&from=appmsg "")  
  
那就看看网页源代码：  
  
![查看网页源代码寻找特征 JS 文件](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gocwyfzGice8bKHBs9QvFbsiatMD8hZnicF2JtNycdGap60CwxnKOmzVdmz8GZ6Cib1Q1JxYr40P4vfJGuhrNczAcNQqSibc5m35niaY/640?wx_fmt=png&from=appmsg "")  
  
搜索页面引用的 js 文件名——实在不晓得哪个是关键的 js 文件名的话，就全部试一遍，反正不多。这里搜索的是 /Coyyy/js/core/knocxxxxxxxx-2.2.1.js  
，对应 FOFA 语法：  
```
js_name="knocxxxxxxxx-2.2.1.js"
```  
  
![FOFA 按 js 文件名搜索出大量同款站点](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpaiay0Dxf4eKj0zVoJ7C0LbfTJFibm1PI8Ar209vhWznklovg4mBC6EwX8PoMc4Vz0PowzrkziaGibVC0MdjQhSvvJvmbUd84ITBE/640?wx_fmt=png&from=appmsg "")  
  
随便点开几个搜索结果：  
  
![搜索结果站点一，页面与目标站点高度相似](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gquPqbgjmXSRTZibwT6ia3TPLPnfEnibVFib1OKCibozZ6lGzZOoGIkSFr6NLzJiaWGMkTL1uicPRdDMHcC4RfgyAdDCuw1AGDsRuUraU/640?wx_fmt=png&from=appmsg "")  
  
![搜索结果站点二，同样具备相同产品特征](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpt4bK1MRlgnic4ODkUP6pmTQJibufhFRMITqBcicy7kGmm7ygjCfsTcsr4qiae3nWp5ehia2JgQBjwlevs3Qr8Ijp0qr0PDv0xfFjY/640?wx_fmt=png&from=appmsg "")  
  
发现页面大同小异，且都存在同样的 SQL 注入。至此，一个通用漏洞到手：  
  
![多个站点均存在相同注入，通用漏洞成立](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goCWLGQFBgoZGD1g32pYkuoAJt2g0bjIjbAIhPPibYuaj4LoWg8yrmg7RmRZ3rSmiazeCo3ts3W1k0Pq4HFGgWibaqdkZydAnmJYI/640?wx_fmt=png&from=appmsg "")  
  
话说回来，平常在黑盒测试过程中挖到的漏洞，都可以顺手去试一下：它是不是通用型的漏洞。  
  
如果是，并且运气够好——该产品公司的注册资金大于 5000w，那么一张 CNVD 证书就来了（运气再好些，还可以在同一个系统里多薅几个）。  
  
当然，也可以换个方向，去搜国内 500 强公司的产品，挖他们产品的漏洞，同样是一种方法。  
## 写在最后  
  
回看整个案例，链路其实很短：登录框一个单引号 → 单双引号回显对比确认注入 → 架构加经验判断 MSSQL → 万能密码实锤并绕过登录 → JS 文件名当指纹 → FOFA js_name  
 批量定位同款站点 → 通用漏洞到手。  
  
真正值钱的是中间那次“思维发散”：绝大多数人证明注入存在之后就收工了，而通用漏洞的钥匙，往往就藏在目标站点最不起眼的特征里——页面没有指纹特征，就去源代码里找它专属的 js 文件名，这就是这个产品在全网最精准的“身份证”。平常黑盒挖到的每一个漏洞，都可以顺手问自己一句：这是不是通用产品？如果是，注册资金 5000w 以上厂商的产品，一张 CNVD 证书可能就在路上。  
  
对防御方来说，这是同一件事的另一面：通用型漏洞意味着一发命中、全量沦陷，厂商必须在源头做参数化查询、统一发版升级，否则每个客户站点上都埋着同一颗雷。  
  
