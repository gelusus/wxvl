#  日口令绕过到 RCE：拆 Danfoss 超市冷柜控制器三条 CVE  
原创 黑卷
                    黑卷  黑卷的IoT攻防日记   2026-10-16 02:19  
  
最近一条新闻在圈里传得很开：2026 年 10 月 12 日前后，法国多家连锁超市的冷柜被人远程改了温度，设定值被拉到约 50℃，大量冷藏食品直接报废。超市冷柜本来恒在几摄氏度，一旦被拉到五十度，几个小时就是一地损失。  
  
受影响的名单不短——Intermarché、Carrefour、Monoprix、E.Leclerc，还有 Super U、Hyper U 这些连锁。其中 Intermarché 对法国媒体称有 150 家门店中招，损失以几十万欧元计。多篇报道都提到，攻击者动的是门店里的 Danfoss 制冷系统。  
  
![法国超市冷柜被改温的热点背景](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84WrRT29VMjFaPtYfFla57QhDCFl19hEIUUkBzvLRfT5q6brs1XG94z6kLjmDDO6rh9Tm5tkN54fbGpQ3ql73600wsqIibYVOfWk/640?wx_fmt=jpeg&from=appmsg "")  
  
这里得先把话说清楚：这起事件的**具体攻击手法还没有公开确认**  
，到底是不是用了下面要拆的几个 CVE，目前没有定论。CERT-FR 10 月 13 日的通报还把范围扩到了 Copeland、Dixell 的 XWEB 平台，说已知有多起这类系统被入侵，建议统统撤离公网。  
  
所以本文不蹭结论，只拆一个确定的东西：Claroty Team82 在 2026 年 8 月披露、并在 DEF CON 上讲过的 **Danfoss AK-SM 800A**  
 攻击面。研究员是 Amir Zaltzman。热点归热点，技术归技术，两头别混。  
  
![Danfoss AK-SM 800A 管理面板](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WeFjuQiclWCCXpnWOYrFbQiciaicU5670JqZvMevLVTKok09olmMBBiaVow1aXI5DWIFhLicYOTrmaldKIXCMB0NmnWibdFYXAUuNibVM/640?wx_fmt=jpeg&from=appmsg "")  
## 这台控制器管的是整店冷链  
  
Danfoss 是全球做制冷和暖通设备的大厂之一，产品铺在超市、冷库、仓储和商业楼宇里。AK-SM 800A 是它的一款**制冷集中管理器**  
：一台机器管着一整片门店的制冷回路。  
  
从一块控制台上，管理员能盯温度、调温度阈值、翻历史告警、排运行时段、配通知服务、管用户和权限，还能远程做维护。一句话——它就是整店冷链的操作中枢。  
  
正因为维护工经常远程管，设备自带一个 Web 管理界面，省得每家门店都跑一趟，维护成本确实降了不少。可方便是双刃剑：一旦这个「中枢」挂上公网，它就成了攻击者最省事的入口。  
  
![三条 CVE 与关键数字](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84WQGD0JrV2uxBZs3Qkic1EMOz8YFNuYYHNcXldLe0ryFBicAibhJ0neWapuOOUs9J0zO0VzGUV2TGmaAfaucJhXwnN6zSPlXrw784/640?wx_fmt=jpeg&from=appmsg "")  
## 先问一句：这些机器真能被摸到吗？  
  
工控系统常被默认「躲在隔离网里很安全」，但现实往往不是。研究的第一步不是翻固件，而是先确认：攻击者到底够不够得着这些设备？Team82 去 Shodan、Censys 上按管理界面的指纹搜了一圈，很快就搜到了——一大批控制器直接暴露在公网上。  
  
![Censys 扫到约 2765 台公网暴露设备](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84XvpjiaJ99l8Yohib94mjDmw2K569phZgGxdWHODWIicYQPhsySny6trmHraw0rwznMJMljv80W33W8e5EjfuHOOib6PM4ibubH49v4/640?wx_fmt=jpeg&from=appmsg "")  
  
Censys 的结果是约 **2765 台**  
，很多还带着 Danfoss StoreView 管理界面那个标志性的 HTTP 标题，说明管理员是主动把 Web 控制台开到了外网做远程运维。对攻击者来说，门槛一下就低了：不用先攻企业网、也不用进超市摸机器，扫一扫、挑出跑着老固件的，直接开打。  
  
这也决定了研究方向。设备虽然支持多种工业通信协议，但对远程攻击者来说，这个面向公网的 Web 界面才是最现实的入口，于是重点先放这儿。  
## Web 架构：四层链路串起来  
  
抽出固件翻文件系统，才发现这个 Web 服务没那么简单——不是一个 HTTP 服务器闷头干活，而是多层反向代理 + CGI 处理器 + 应用逻辑，分散在好几个独立进程里。  
  
![Web 服务架构示意](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84XicSAvxibhh2btSxxz2cLKnEEeUS0HrricjXBJN6oELors4MWcum8kiaxUIru70AXlaufoX4e7t6Wpy0XjHhTAqrgUtESlH0zhxiaI/640?wx_fmt=jpeg&from=appmsg "")  
  
最外面是一个公网可达的 Nginx，纯做反向代理，nginx.conf  
 里规定带 .cgi  
 后缀的请求转给 CGI 实例 sm_fcgi_app  
。  
  
![sm_fcgi_app.conf 配置](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84W4eCtWa8S2MiahPd4964MAxOtHE5AsRGj0riaicibewHVOzp2ichYwlV3Xjghe2ZSIg36cLV9mJP27Ag4vNHEaJEeAbwLzS5sgPY98/640?wx_fmt=jpeg&from=appmsg "")  
  
sm_fcgi_app  
 按自己的配置 sm_fcgi_app.conf  
 做初步处理，再用 forward_xml_request  
 把请求转给**内层 Nginx**  
。  
  
![sm_fcgi_app 调用 forward_xml_request](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84Xd8w9NVJIRU3lf0096ssbq1tDJtoAJg488ajM2AWM7yLb8JlEawQZ9liaiaX0bsrag26cYdrxAARhiap1icBrQbWdvzNAnPShrwlU/640?wx_fmt=jpeg&from=appmsg "")  
  
内层 Nginx 只监听本地回环，继续把 .cgi  
 请求转给 sm_app  
 里的一个任务 Task_Web  
——这才是 Web 应用真正解析、处理、分发请求的核心。几层套下来看着绕，但理清这条链，正是后面每个洞的起点。  
  
![内层 Nginx 监听本地](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84VMXhgUqjUgV3C7gSJ593CqfPlr0tZtlcxFQAhsJ7LklqFlFSwibGPQpu6cWoX5GCKveWsAQeibmTsvjruznZ2ojay7UV89pOUF4/640?wx_fmt=jpeg&from=appmsg "")  
## CVE-2025-41450：藏在日期里的「当日口令」  
  
认证逻辑往往是管理平台里最敏感的一块。Team82 盯上了发往 xml.cgi  
、带 action="getauth"  
 的认证请求。  
  
![Danfoss 登录页与 getauth 请求](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84XYThBPRvwaVKy0qfu4o6VboW8fBuBcliadviaUzcTfLbhrmPtKDvoGJh2a7ZCV4KGUWgTFwUACOoHzxejibcDZ26zqpU3yeGgkCw/640?wx_fmt=jpeg&from=appmsg "")  
  
顺着处理流程看，请求会进到 xml_get_data  
，再按操作类型分发到对应处理函数。分析 getauth 这条时，发现请求里有两个用途不明的属性。一旦把它们置为 1，认证流就不再走正常的密码比对，而是拐进另一条分支，最终调到一个叫 check_code_of_day  
 的函数。  
  
![当日口令绕过流程](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84X6eqILrwZcEZejsq2HO2IibK8gBPNKL56hicHl3JbzTn6dWEBUDM84htf8Rr56h9Az9F4a1h5Ria9Cqhibzm2yuPiaCuibSianTAib25U/640?wx_fmt=jpeg&from=appmsg "")  
  
这显然是一条隐藏的认证通道，大概率是厂商留给售后/维护用的。它不去比对你配置的管理员密码，而是拿设备**当前日期**  
算出一个「当日口令」，让你提交的值去跟它比。  
  
![check_code_of_day 口令生成函数](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84XJljccfoicCBIp8HAPs400OQ7ovEXUo4H37OztKnyF1YrcH4CI16Wt0UCOgXzHXTl3xicCsX9k0CHZILkfJF0t1BQeGzrHbDdfY/640?wx_fmt=jpeg&from=appmsg "")  
  
把 check_code_of_day  
 逆出来后，算法就清楚了：它用 Get_syst_current_tds  
 取当前日期，再按 "%02d%02d%02d%02d"  
（月、日、年、月）这类格式拼出口令。换句话说，**只要复现这套算法，谁都能算出任意一天的口令**  
，根本不用知道管理员密码，就能拿到一个完整的管理员会话。隐藏维护口令，本质上就是一个后门。  
## CVE-2025-41451：邮件密码字段拼进 shell  
  
绕过认证之后，攻击面一下就铺开了——原本只有少数几个未认证接口能碰，现在整个管理界面全开：网络、告警、用户、邮件通知、日志、系统维护，全都能点。逆嵌入式 Web 应用时，第一件事就是找「会去调系统命令」的功能，因为这类地方常常用用户输入拼 shell 命令。  
  
![三条 CVE 的关系](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84VHYtlmJHvFJSa6H53kRNHNc23CdGf6EDgxBiaW4Pp7ia5GTgmqH0dmJFThCggkd3PL6aO8sboY1ZMlRVePYzB3VdR9cibecpyq7o/640?wx_fmt=jpeg&from=appmsg "")  
  
SMTP 邮件通知配置页很快冒了头。跟着「更新邮箱设置」的请求往下走，追到了拼接 OS 命令的那段代码：取自 item 0x1be1 的值——也就是**邮箱认证密码字段**  
——被直接塞进 shell 命令，一路没做消毒。  
  
![邮件密码拼 shell 命令注入示意](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UWlzwfxRelw4n6vBJuunp49wMVicWbF9VQNKEYFYKaIlwVz9QePMTX0CByN62xoWofXsicnVJgofCp7d93tZuLrTAOPCzq5souA/640?wx_fmt=jpeg&from=appmsg "")  
  
这意味着，在密码字段里塞进反引号、分号、管道之类的 shell 指令，就能让设备去跑攻击者的命令。而认证这关已经被 41450 绕过了，两条一叠，就是一条无需口令的远程代码执行链。能在冷链控制器上执行任意命令，意味着什么就不用多说了。  
## CVE-2025-41452：运行时改 Nginx 配置  
  
还有第三条。那个公网 Nginx 的 server 块里，include  
 了一个 headers.conf  
。  
  
![外层 Nginx include headers.conf](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WQRHTbqyDFFFmOEDPOIcoiaxl6CMgEMwRyFO5UgQHYYUhkbKxAD7Qdy5OiaWj4NeFeS8JTk05fT6EDXPXmgZ4fH2QicM52YdyuSk/640?wx_fmt=jpeg&from=appmsg "")  
  
问题是，Web 应用竟然暴露了一个替换这个文件、并在运行时 reload Nginx 的功能——action="load_nginx_headers"  
，认证后以文件上传的方式传一份新的 headers.conf  
 就行，连设备都不用重启，新指令立刻生效。  
  
![xml_load_nginx_headers 处理函数](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84WFpQGtibgH7NQOGMvCJIx9oQVgZwfKAuKtt7KySjaaiab7MrHlFa0kxod7xp5tPBU9ybctZG4dYiaJpOwDthnCnsMRgJia13nTFRk/640?wx_fmt=jpeg&from=appmsg "")  
  
于是攻击者可以往公网 Nginx 里植入任意路由指令，改请求的处理方式，操控与设备交互的客户端。测试中还撞到一个边角：访问被植入的恶意路由，会触发 sm_app  
 的未处理异常，Web 服务直接崩——认证用户就能反复触发，把管理界面打成持续的 DoS。  
  
![Nginx 配置注入示意](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WDNWr7UeicxAffCIMZ0XcJcZpuRzUPtgibp77VzwW7vzkmZ5ibToloEgrWZAibADQ6xNIIibtQiaJGia9AVEPlnf9elt5r3KdMykznia4/640?wx_fmt=jpeg&from=appmsg "")  
## 修复与建议  
  
Team82 走的是协调披露流程，私下把问题报给 Danfoss，厂商调查后发布了固件 **R4.3.1**  
 来修复这几个洞。还在用受影响 AK-SM 800A 的，尽快升到 R4.3.1 或更高。  
  
更要紧的是那条老规矩：别把管理界面直接挂公网，把管理服务收到可信网段里，或用 VPN、网络分段挡在前面。Team82 当初扫到的 2765 台里，有多少已经升级、多少还露在外面，没人数得清——这也是为什么同类暴露面一出事就是一大片。  
## 黑卷点评  
  
一台管超市冷链的控制器，能被从公网一路打到 RCE，根子上是几类老问题凑齐了。  
  
![黑卷点评：三条实战提醒](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WP2CFOk8oicVA6qiaqP9Ud40s7MPxWoPmzeoWsHiaFutFXYUOJpZO4CIYEcAlaBcLHYkGTYag2SY1OH2yBAThmqddQvpHplgEE3A/640?wx_fmt=jpeg&from=appmsg "")  
  
① **CPS 管理面别挂公网**  
——制冷、工控这类集中控制器，一旦暴露就等于把整店冷链交到扫描器手里；② **隐藏维护口令就是后门**  
——按日期算的「当日口令」被逆出来，管理员密码形同虚设，这种「方便售后」的设计迟早反噬；③ **配置别拼 shell、别动态改 Nginx**  
——字段进命令必须消毒，运行时可替换的配置文件本身就是一张提权面。  
  
换到车联网充电桩、楼宇暖通这些远程管理面，题目其实是同一套：收口公网、去掉维护后门、关键配置做白名单校验。法国超市这波热点会过去，但这几条教训不会。守好自己手里那块管理面，比追新闻实在得多。  
  
![星球「黑卷的IoT攻防日记」](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84XOrSrMDoIk3K3rqhh9jUiaDNmFZZodMOuhMHtP1rvNZDyC5ptnn9OGFbsKiaUdRddeicicwXDdgOQqwWZ5lLwE3T1ydj7bHd5pJQY/640?wx_fmt=jpeg&from=appmsg "")  
  
  
