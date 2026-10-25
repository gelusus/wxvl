#  Web框架漏洞盘点那些年拖垮无数网站的经典漏洞  
原创 围巢安全笔记
                    围巢安全笔记  围巢安全笔记   2026-10-25 01:00  
  
渗透测试实战 · 第 22 篇  
  
做渗透测试的时候，识别出目标用的是什么Web框架，往往就成功了一半——因为很多框架有已知的经典漏洞，识别出框架版本，直接对照漏洞库就能找到可利用的漏洞。这篇把最常见的几个Web框架的经典高危漏洞讲透，从ThinkPHP到Spring再到Struts2，每一个都是真实发生过、拖垮过无数网站的漏洞。了解这些漏洞，不是为了攻击，而是为了知道怎么防御。**本文仅限安全研究与教学，请勿对未授权系统进行漏洞利用**  
。约 2400 字，附 2 个实用干货。  
  
  
01｜ThinkPHP：国内最流行的PHP框架，也是漏洞最多的  
  
ThinkPHP是国内最流行的PHP开发框架，很多中小网站和企业系统都用它。但是ThinkPHP的历史上出过好几个严重的RCE（远程代码执行）漏洞，影响范围极广，直到现在还有大量网站因为这些漏洞被黑。  
  
ThinkPHP 5.x RCE漏洞  
  
这是ThinkPHP最著名的漏洞，影响5.0.x和5.1.x的多个版本。漏洞的原理是ThinkPHP的路由解析功能存在缺陷，攻击者可以构造特殊的URL，让框架把用户输入当成PHP代码执行。这个漏洞的利用非常简单，只需要在URL里加上特殊的参数，就能执行任意PHP代码，包括写WebShell、执行系统命令、读取敏感文件等。  
  
这个漏洞在2018年底被公开之后，立刻成为黑客的"批量扫洞神器"——攻击者用扫描器扫全网的ThinkPHP网站，发现有漏洞的就直接利用，写挖矿木马、留后门、拖库。无数网站因为这个漏洞被黑，很多站长甚至不知道自己用的ThinkPHP有这个漏洞。  
  
ThinkPHP 2.x/3.x 日志泄露漏洞  
  
老版本的ThinkPHP（2.x和3.x）默认开启日志功能，而且日志文件存储在Web可访问的目录下，文件名是按日期命名的（比如Runtime/Logs/Home/23_01_01.log）。攻击者只要知道网站用的是ThinkPHP，直接访问日志文件的URL，就能下载到完整的日志，里面包含了所有的SQL查询、错误信息、用户请求等敏感数据。很多网站的数据库密码、管理员账号密码都因为这个漏洞被泄露。  
  
防御方法  
  
升级到最新版本是最有效的防御方法。ThinkPHP官方在漏洞公开后很快就发布了补丁，升级到最新版本就能修复。另外，关闭调试模式（APP_DEBUG=false）、禁止访问Runtime目录、限制日志文件的访问权限，也能降低风险。  
  
  
02｜Struts2：Java框架的"漏洞之王"  
  
![Struts2架构图](https://mmbiz.qpic.cn/sz_mmbiz_jpg/8m3icSM3btkf4tEFCUDIdNVxAGmv3RlFL4Xqv7GDfPKlFUjNPrhUlibDhliadP0wJks3xDH9EurBuicM3HXwT1Q621Ky4p6WxuNm1A0bxZqAF1M/640?wx_fmt=jpeg&from=appmsg "")  
  
Struts2架构：请求经过FilterDispatcher、拦截器、Action，漏洞就出在这些环节  
  
Struts2是Apache基金会的Java Web框架，曾经是Java Web开发的主流框架，大量企业级应用都用它。但是Struts2的历史上出过几十个RCE漏洞，是当之无愧的"漏洞之王"。几乎每年都有新的Struts2漏洞被发现，每个都能让大量企业系统沦陷。  
  
S2-045：最著名的Struts2漏洞  
  
S2-045是2017年公开的一个RCE漏洞，影响Struts2 2.3.5-2.3.31和2.5-2.5.10版本。漏洞的原理是Struts2的文件上传功能在解析Content-Type头的时候存在OGNL表达式注入，攻击者可以在HTTP请求的Content-Type头里插入恶意的OGNL表达式，从而执行任意Java代码。  
  
这个漏洞的利用极其简单——只需要在请求头里加一段恶意的Content-Type值，就能执行任意命令，不需要任何特殊条件。漏洞公开之后，立刻成为黑客最爱的漏洞，大量政府、企业、金融网站因为这个漏洞被黑。直到现在，还有很多老系统没有升级，依然存在这个漏洞。  
  
S2-057、S2-061等后续漏洞  
  
S2-045之后，Struts2又陆续出了S2-057（2018年，REST插件的OGNL注入）、S2-059（2019年，标签库的OGNL注入）、S2-061（2020年，再次绕过S2-059的补丁）等多个RCE漏洞。每个漏洞的原理都类似——OGNL表达式注入，但是触发点不同，有的在URL参数里，有的在表单字段里，有的在标签属性里。  
  
Struts2的漏洞之所以这么多，根本原因是它的OGNL表达式功能太强大了——OGNL可以调用任意Java方法、访问任意对象属性，而Struts2在很多地方都把用户输入传给了OGNL解析，只要过滤不严，就会导致RCE。虽然官方一直在打补丁，但是总能找到新的绕过方法。  
  
防御方法  
  
最根本的防御方法是——不要用Struts2了。这个框架已经过时，而且漏洞层出不穷，维护成本很高。如果必须用，一定要升级到最新版本（2.5.x的最新版或6.x），并且关注官方的安全公告，有新漏洞及时升级。另外，在WAF里配置Struts2漏洞的特征规则，也能拦截大部分攻击。  
  
  
03｜Spring：Java企业级开发的事实标准，也有大漏洞  
  
Spring是现在Java企业级开发的事实标准，几乎所有的Java后端都在用Spring（包括Spring Boot、Spring Cloud）。Spring的代码质量比Struts2高很多，漏洞也少很多，但是历史上也出过几个严重的漏洞。  
  
Spring4Shell（CVE-2022-22965）  
  
这是2022年爆出的一个严重RCE漏洞，影响Spring Framework 5.3.0-5.3.17和5.2.0-5.2.19版本，以及基于这些版本的Spring Boot应用。漏洞的原理是Spring的参数绑定功能存在缺陷，攻击者可以通过构造特殊的请求参数，修改Tomcat的日志配置，从而写入恶意的JSP文件，最终实现RCE。  
  
这个漏洞被称为"Spring4Shell"，因为它和当年的"Shellshock"一样影响广泛。漏洞公开之后，安全界一片哗然，因为Spring的使用范围太广了，几乎所有的Java企业应用都可能受影响。不过这个漏洞的利用条件比较苛刻——需要JDK 9+、Spring MVC、Tomcat作为Servlet容器、WAR包部署，所以实际能利用的系统比预想的少。  
  
![Spring4Shell漏洞](https://mmbiz.qpic.cn/sz_mmbiz_jpg/8m3icSM3btkegeAtZXgP0DkVG7zQfRibjQYriaj1iaovRha8YK5dkmdOVSb4xcBM6DKWhlZhRUibtnSKKlchhACRI6TSBzl3OQm6Oe06pwB6MJBg/640?wx_fmt=jpeg&from=appmsg "")  
  
Spring4Shell（CVE-2022-22965）是Spring Framework的远程代码执行漏洞，2022年曝光后引发安全界震动  
  
Spring Cloud Gateway RCE（CVE-2022-22947）  
  
这是Spring Cloud Gateway的一个RCE漏洞，影响3.1.0-3.1.1和3.0.0-3.0.6版本。漏洞的原理是Spring Cloud Gateway的Actuator端点没有做权限控制，攻击者可以通过Actuator API添加恶意的路由，路由的过滤器中可以包含SpEL表达式，从而执行任意代码。  
  
这个漏洞的利用需要Actuator端点暴露在公网上，而且没有做认证。很多开发者为了方便，把Actuator的所有端点都暴露出来，而且不设密码，就给了攻击者可乘之机。  
  
防御方法  
  
及时升级Spring Framework和Spring Boot到最新版本。Spring官方对安全漏洞的响应很快，一般漏洞公开后几天内就会发布补丁。另外，Actuator端点一定要加认证，不要暴露在公网上，只开放必要的端点（不要用*开放所有端点）。  
  
  
04｜其他常见框架的经典漏洞  
  
Laravel（PHP框架）  
  
Laravel是现在最流行的PHP框架之一。它的经典漏洞是Laravel Debug模式RCE（CVE-2021-3129）——当Laravel开启Debug模式时，攻击者可以通过构造特殊的请求，利用Laravel的日志文件和PHP的phar反序列化，实现RCE。这个漏洞的利用条件是开启了Debug模式，很多开发环境和测试环境都开启了Debug模式，而且暴露在公网上，就会被利用。  
  
Django（Python框架）  
  
Django是Python最流行的Web框架，以安全著称。但是它也出过漏洞——比如Django的Debug模式信息泄露（当DEBUG=True时，错误页面会泄露大量敏感信息，包括数据库密码、SECRET_KEY、文件路径等）、Django的SQL注入漏洞（ORM的某些用法可能导致SQL注入）、Django的CSRF绕过等。Django的漏洞总体来说比较少，而且大部分需要特定条件才能利用。  
  
Flask（Python微框架）  
  
Flask是Python的轻量级Web框架。它的经典漏洞是Flask Debug模式RCE——当Flask开启Debug模式时，错误页面有一个交互式的Python控制台，攻击者可以直接在浏览器里执行任意Python代码。这个漏洞非常严重，但是利用条件是开启了Debug模式而且知道Debug PIN（Flask 1.0+需要PIN才能用控制台）。不过Debug PIN可以通过信息泄露获取，所以依然有风险。  
  
Shiro（Java安全框架）  
  
Apache Shiro是Java的安全框架，用于认证和授权。它的经典漏洞是Shiro反序列化RCE（CVE-2016-4437）——Shiro的RememberMe功能用AES加密Cookie，但是默认的加密密钥是硬编码在代码里的，攻击者知道密钥之后，可以构造恶意的序列化对象，加密后放在Cookie里，服务器解密后反序列化，从而执行任意代码。这个漏洞影响极广，因为很多项目用了Shiro的默认配置，没有修改默认密钥。直到现在，Shiro反序列化还是内网渗透中最常用的漏洞之一。  
  
干货**Web框架指纹识别速查表**  
  
做渗透测试的时候，第一步就是识别目标用的是什么框架。下面是常见框架的指纹特征：  
```
# ThinkPHP
# - Cookie: PHPSESSID（不一定）
# - 错误页面有"ThinkPHP"字样和版本号
# - URL特征：/index.php/Home/Index/index
# - 特定路径：/Runtime/Logs/、/Application/
# - Header: X-Powered-By: ThinkPHP（有些版本）

# Struts2
# - URL特征：.action、.do后缀
# - Cookie: JSESSIONID
# - 错误页面有"Struts Problem Report"
# - 特定路径：/struts/webconsole.html（S2-001的控制台）
# - Header: X-Powered-By: Struts（有些版本）

# Spring / Spring Boot
# - Cookie: JSESSIONID
# - 错误页面有"Whitelabel Error Page"（Spring Boot默认错误页）
# - 特定路径：/actuator、/actuator/health、/actuator/env
# - Header: X-Application-Context（有些版本）
# - favicon.ico是Spring的绿色叶子图标

# Laravel
# - Cookie: laravel_session
# - 错误页面有"Laravel"字样（Debug模式下）
# - 特定路径：/telescope（Laravel Telescope调试工具）
# - Header: X-Powered-By: PHP/7.x（不一定）

# Django
# - Cookie: csrftoken、sessionid
# - 错误页面有"Django"字样（Debug模式下）
# - 特定路径：/admin/（Django后台）
# - Header: X-Frame-Options: SAMEORIGIN（默认）

# Flask
# - Cookie: session（Flask的session是加密的Cookie）
# - 错误页面有"Werkzeug"字样（Debug模式下）
# - 特定路径：/console（Debug控制台，需要PIN）
# - Header: Server: Werkzeug/x.x.x（开发服务器）

# Shiro
# - Cookie: rememberMe（Shiro的RememberMe功能）
# - 删除Cookie后访问需要登录的页面，
#   响应头会有Set-Cookie: rememberMe=deleteMe
# - 这是Shiro最明显的指纹特征

# 识别工具：
# - 手动：看Cookie、Header、错误页面、URL特征
# - 工具：whatweb、wappalyzer、cmseek、EHole
# - 在线：Wappalyzer浏览器插件、BuiltWith
```  
  
框架识别是渗透测试的第一步，识别出框架和版本，就能对照漏洞库找到可能存在的漏洞，事半功倍。  
  
  
05｜框架漏洞的通用防御方法  
  
不同框架的漏洞各不相同，但是防御方法有一些通用的原则：  
- **及时升级**  
：这是最有效的防御方法。框架的安全漏洞公开后，官方一般会很快发布补丁，及时升级到最新版本就能修复大部分漏洞。不要用已经停止维护的老版本（比如Struts2 2.3.x、ThinkPHP 3.x）。  
  
- **关闭Debug模式**  
：几乎所有框架的Debug模式都有信息泄露甚至RCE的风险。生产环境一定要关闭Debug模式，不要图方便开着Debug上线。  
  
- **修改默认配置**  
：很多框架漏洞是因为默认配置不安全——比如Shiro的默认加密密钥、Actuator的默认开放端点、Redis的默认无密码。部署的时候一定要修改这些默认配置，不要用出厂设置。  
  
- **WAF防护**  
：在Web应用前面加一层WAF（Web应用防火墙），配置常见框架漏洞的攻击特征规则，能拦截大部分已知漏洞的攻击。WAF不能替代升级，但是能作为额外的防护层。  
  
- **定期扫描**  
：定期用漏洞扫描工具（比如Nessus、AWVS、Xray）扫描自己的系统，及时发现存在的框架漏洞。不要等被黑了才知道自己的系统有漏洞。  
  
- **关注安全公告**  
：关注所用框架的官方安全公告和CVE数据库，有新漏洞公开时及时评估影响，必要时紧急升级。  
  
干货**框架漏洞学习路径与资源推荐**  
  
想深入学习框架漏洞，按照下面的路径来：  
```
# 第一阶段：打基础
# - 掌握至少一门编程语言（Java或PHP或Python）
# - 理解Web开发的基本原理（HTTP、MVC、ORM、路由、中间件）
# - 用一个框架做一个完整的Web项目（比如用Spring Boot做博客）
# - 理解框架的工作原理（请求是怎么从浏览器到代码的）

# 第二阶段：学漏洞原理
# - 学习常见漏洞类型的原理：
#   RCE（远程代码执行）、反序列化、SQL注入、
#   路径穿越、SSRF、XXE、模板注入、表达式注入
# - 理解每种漏洞的成因和利用条件
# - 学习OGNL、SpEL、MVEL等表达式语言的语法
#   （很多Java框架漏洞都是表达式注入）
# - 学习Java反序列化的原理和利用链
#   （CommonsCollections、Fastjson、Jackson等）

# 第三阶段：漏洞复现
# - 用Vulhub、VulApps等漏洞环境，
#   逐个复现经典框架漏洞
# - 每个漏洞都要理解：原理、利用条件、利用方法、防御方法
# - 不要只复制粘贴POC，要理解每一步在做什么
# - 尝试自己写POC，而不是用现成的

# 第四阶段：代码审计
# - 学习框架的源码，理解漏洞是怎么产生的
# - 尝试自己在框架代码中发现漏洞
# - 学习补丁分析，看官方是怎么修复漏洞的
# - 尝试绕过补丁，发现新的利用方式

# 推荐资源：
# - 漏洞环境：Vulhub（github.com/vulhub/vulhub）、
#   VulApps（github.com/Medicean/VulApps）
# - 漏洞库：CVE Details（cvedetails.com）、
#   CNVD（cnvd.org.cn）、Exploit-DB（exploit-db.com）
# - 技术博客：P牛的博客（govuln.com）、
#   先知社区（xz.aliyun.com）、FreeBuf（freebuf.com）
# - 书籍：《Java安全漫谈》、《PHP安全之道》、
#   《白帽子讲Web安全》
# - 工具：Burp Suite、Yakit、Xray、Nuclei

# 记住：
# 学习框架漏洞不是为了攻击，而是为了防御。
# 只有理解了漏洞是怎么产生的、怎么被利用的，
# 才能知道怎么写出安全的代码、怎么防御攻击。
# 始终在合法合规的框架内学习和实践。
```  
  
框架漏洞的学习是一个长期的过程，需要持续关注新的漏洞和技术。打好基础、理解原理、动手实践，这三点做到了，就能成为一个合格的安全研究员。  
  
  
Web框架漏洞是渗透测试中非常重要的一个方向——很多高危漏洞都出在框架层面，而且利用简单、影响广泛。作为开发者，要了解这些漏洞，写出安全的代码，及时升级框架版本；作为安全从业者，要掌握框架指纹识别和漏洞利用的方法，在授权测试中发现这些漏洞。  
  
**最后再次提醒**  
：本文所有内容仅限安全研究与教学，请勿对未授权系统进行漏洞利用。未经授权利用他人系统漏洞属于违法行为，将承担法律责任。  
  
围巢安全笔记  
  
不生产焦虑，只分享踩过的坑和挖过的洞  
  
你被哪个框架漏洞坑过？评论区聊聊 👇  
  
有想聊的话题，后台留言告诉我  
  
  
往期回顾  
- 验证码绕过：那些看起来很安全的验证码是怎么被破的  
  
- Redis未授权访问：一个配置错误就能丢服务器  
  
- 密码破解：你的密码在黑客眼里能撑多久  
  
- Nmap：用了三年才搞懂的端口扫描神器  
  
