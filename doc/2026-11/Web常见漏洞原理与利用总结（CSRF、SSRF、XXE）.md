#  Web常见漏洞原理与利用总结（CSRF、SSRF、XXE）  
点击关注👉
                    点击关注👉  马哥网络安全   2026-11-17 09:00  
  
一、SSRF服务端请求伪造漏洞  
  
1. 漏洞核心概念  
  
SSRF（Server-Side Request Forgery，服务端请求伪造）是一种  
服务端主动发起请求  
的高危Web漏洞。漏洞根源是网站业务存在  
用户可控的URL参数  
，后端代码未对请求地址、访问协议、内网IP做严格的过滤与校验，直接调用服务端网络请求函数（如file_get_contents、curl_exec、fsockopen等），以服务器自身的网络权限去访问攻击者构造的任意目标地址。  
  
通俗理解：攻击者无需直接访问内网，利用存在漏洞的Web服务器作为网络跳板，突破外网边界限制，访问服务器本机、内网网段、外网任意资源，实现信息探测、数据窃取、内网攻击等操作。  
  
2. 受害机参与时机与流量逻辑  
  
在SSRF攻击全流程中，  
受害服务器是唯一的请求发起主体  
，全程参与攻击，攻击者仅负责构造恶意参数、接收回显结果，不直接与目标资源产生交互。  
  
1. 接收阶段：受害服务器接收攻击者提交的包含恶意URL的HTTP请求，正常解析参数中的恶意地址；  
  
2. 发包阶段：服务器以自身IP、自身网络权限，主动向恶意地址对应的本机、内网、外网资源发起网络请求；  
  
3. 回显阶段：服务器接收目标资源的响应数据，整理后返回给攻击者，完成攻击。  
  
完整流量链路：攻击者浏览器/工具 → 漏洞服务器 → 内网/本机/外网目标资源 → 漏洞服务器 → 攻击者  
  
3. 完整详细利用流程  
  
第一步：寻找、判断漏洞点位  
  
SSRF漏洞只会出现在  
服务端需要主动请求外部资源  
的业务功能中，常见特征参数：url、link、src、target、domain、api、remote等。常见业务场景：远程图片抓取、图片在线预览、网站内容采集、URL短链接生成、第三方登录接口、在线资源下载、站点内容爬虫。  
  
第二步：初步验证漏洞存在  
  
构造正常外网URL（如https://www.baidu.com）传入可控参数，若服务器能够代为访问并返回百度页面内容，即可确认SSRF漏洞存在。随后依次测试协议支持情况，不同协议对应不同攻击能力：http/https用于端口扫描、内网Web服务访问；file用于读取本地文件；gopher/dict用于内网服务命令执行、高危攻击。  
  
第三步：判断服务器过滤规则  
  
探测服务器是否拦截回环地址（127.0.0.1）、内网网段（192.168/10/172）、是否禁用非HTTP协议，为后续绕过、精准利用做铺垫。  
  
第四步：构造对应恶意Payload  
  
根据业务场景和过滤规则构造载荷：内网端口扫描批量IP端口、file协议读取系统敏感文件、gopher协议构造内网服务攻击指令等。  
  
第五步：发送恶意请求触发漏洞  
  
通过浏览器直接访问、Burp Suite抓包修改参数等方式提交恶意载荷，由服务端代为执行网络请求。  
  
第六步：根据回显/状态判断攻击结果  
  
根据页面回显内容、页面报错、请求超时时间、响应长度，判断端口开闭、文件是否读取成功、内网服务是否攻击成功，获取敏感信息或完成渗透攻击。  
  
4. 通用实战利用场景与Payload  
  
场景一：服务器本地任意文件读取（高频）  
  
适用条件：服务端支持file协议，无本地文件读取限制。可读取系统核心配置、账号密码、网站源码、数据库配置等敏感文件。  
  
Linux常用Payload：file:///etc/passwd（用户列表）、file:///etc/group（用户组）、file:///etc/shadow（密码哈希）、file:///var/www/html/config.php（网站数据库配置）  
  
Windows常用Payload：file://C:/Windows/System32/drivers/etc/hosts（本地域名解析）、file://C:/Windows/win.ini（系统配置）、file://网站目录/config.ini（程序配置）  
  
场景二：内网端口扫描与存活探测  
  
利用http协议遍历内网常见端口，根据响应状态判断端口是否开放：端口开放会返回页面内容、服务Banner；端口关闭会提示连接拒绝；端口被防火墙拦截会出现请求超时。常见扫描端口：80、8080、3306、6379、22、21、11211等。  
  
示例载荷：http://192.168.1.1:6379、http://127.0.0.1:3306  
  
场景三：内网高危服务攻击（高危）  
  
利用gopher、dict协议向内网未授权服务发送自定义TCP数据包，可攻击Redis、MySQL、Memcached等服务，实现写入WebShell、创建系统账号、命令执行、服务器沦陷。  
  
场景四：外网恶意流量转发  
  
利用漏洞服务器批量向外网地址发送请求，将服务器沦为攻击肉鸡，发起CC攻击、恶意爬虫、接口刷量，造成目标业务瘫痪或自身IP被封禁。  
  
5. YzmCMS专项SSRF漏洞详细利用  
  
该漏洞主要存在于YzmCMS 5.3、5.5等经典版本的  
后台采集管理功能  
，属于二次解析SSRF漏洞，利用门槛低、危害极大。  
  
漏洞原理  
：后台新增采集节点时，用户可控“测试URL”参数，服务器会首先访问该用户指定的URL页面，随后自动解析页面HTML代码，提取页面中第一个  
标签的href属性值，并  
自动发起第二次请求  
。全程无IP黑名单、无协议过滤、无特殊字符拦截，可直接调用file、gopher等高危协议。  
  
  
完整利用步骤  
：  
  
1. 准备公网恶意页面：搭建一个公网可访问的HTML文件，写入恶意a标签链接，可自定义读取文件、扫描内网、攻击Redis等载荷；  
  
2. 登录YzmCMS后台，进入【模块管理】-【采集管理】-【新增采集节点】；  
  
3. 自定义节点名称、编码格式，在“测试URL”处填写自己搭建的公网恶意HTML地址；  
  
4. 点击【测试采集】，服务器先访问公网恶意页面，解析获取a标签中的恶意地址；  
  
5. 服务器二次发起请求，执行文件读取、内网扫描、服务攻击等操作，结果直接回显在后台页面。  
  
  
6. 常见过滤绕过方式  
  
1. 禁止127.0.0.1回环地址：使用十进制IP（2130706433）、十六进制IP、域名解析跳转绕过；  
  
2. 禁止内网网段：使用外网域名解析指向内网IP、302跳转跳转内网地址；  
  
3. 过滤高危协议：使用URL编码、双重编码、换行拼接、特殊分隔符绕过协议拦截。  
  
7. SSRF漏洞详细危害  
  
1. 高危信息泄露：读取服务器系统文件、网站配置文件、数据库账号密码、源码源码、日志记录等核心敏感数据；  
  
2. 内网横向渗透：突破外网防火墙边界，扫描全网段内网存活主机、开放端口、服务版本，绘制内网拓扑，为后续内网渗透铺路；  
  
3. 服务器完全沦陷：攻击内网无防护的Redis、MySQL、FastCGI等服务，写入后门、执行系统命令、获取服务器最高权限；  
  
4. 业务与流量风险：服务器被滥用为攻击肉鸡，发起恶意攻击，导致IP封禁、业务追责、网站被拦截；  
  
5. 攻击溯源困难：所有攻击流量来源均为漏洞服务器IP，攻击者真实IP完全隐藏，安全排查难度极大。  
  
二、CSRF跨站请求伪造漏洞  
  
1. 漏洞核心概念  
  
CSRF（Cross-Site Request Forgery，跨站请求伪造）是一种  
身份劫持类被动漏洞  
。漏洞核心原理：浏览器存在同源凭证携带机制，用户登录网站后，浏览器会保存该站点的Cookie、Session等登录凭证，后续访问该站点任意接口时，会  
自动携带登录凭证  
。  
  
攻击者通过构造恶意页面、钓鱼链接，诱导已登录用户访问，在用户无感知、无需授权的情况下，冒用用户身份向目标网站执行业务请求。  
  
核心特性：  
只能代为发起请求、执行业务操作，无法窃取用户Cookie、无法读取页面数据、无法获取用户隐私信息  
。  
  
2. 漏洞利用必备前置条件  
  
1. 状态条件：目标用户必须处于网站登录状态，Cookie、Session未过期、未退出登录；  
  
2. 防护条件：目标网站无有效CSRF防护措施（无随机CSRF Token、无Referer/Origin校验、无操作二次确认、无验证码验证）；  
  
3. 接口条件：存在可直接调用的业务接口，参数可被攻击者完全控制；  
  
4. 触发条件：攻击者可诱导用户点击链接、访问恶意页面。  
  
3. 详细利用方式与实战Payload  
  
方式一：GET型CSRF（最简单、无门槛）  
  
适用场景：接口数据通过URL传参，访问链接即可触发操作，无需表单提交。常见于修改资料、注销账号、修改站点配置、绑定手机号等接口。  
  
利用原理：将恶意操作接口URL拼接成钓鱼链接，或嵌入隐藏的img、iframe标签，用户打开页面瞬间自动发起请求，完成操作。  
  
  
方式二：POST型CSRF（主流、最常用）  
  
适用场景：接口仅接收POST请求，无法通过URL直接触发，是网站最常见的CSRF场景。  
  
利用原理：构造自动提交的HTML表单，页面加载完成后自动提交POST数据，无需用户点击，无感触发漏洞。  
  
方式三：JSON高阶CSRF（特殊接口绕过）  
  
适用场景：前端接口仅接收Content-Type:application/json格式数据，普通表单无法提交。通过AJAX构造JSON请求，绕过常规限制，实现自动攻击。  
  
4. CSRF漏洞详细危害  
  
普通用户层面危害：  
篡改个人昵称、头像、手机号、收货地址；绑定攻击者第三方账号、篡改登录密码；发起转账、消费、兑换虚拟资产，造成直接财产损失；公开私密动态、泄露个人隐私。  
  
管理员后台高危危害：  
恶意新增永久管理员账号、删除原有管理员；篡改网站核心配置、关闭站点服务、修改网站源码；批量清空网站文章、用户数据、订单数据；发布违规、违法内容，导致网站被封、追责；完全接管网站后台控制权。  
  
衍生复合危害：  
可与XSS漏洞组合利用，XSS持久化植入恶意代码，批量触发CSRF攻击，实现全站用户账号批量篡改、批量接管，扩大漏洞危害范围。  
  
5. 常见防护与绕过思路  
  
标准防护：随机CSRF Token、SameSite Cookie限制、Referer/Origin校验、操作二次确认、验证码防护。  
  
常见绕过：空Referer绕过、域名后缀一致绕过、前端JS校验可直接抓包绕过、固定Token可暴力破解绕过。  
  
三、XXE外部实体注入漏洞  
  
1. 漏洞核心概念  
  
XXE（XML External Entity Injection，XML外部实体注入）是针对XML解析功能的高危注入漏洞。漏洞核心成因：Web业务接收前端传入的XML数据，后端使用不安全的XML解析器，  
默认开启DTD文档类型解析、允许外部实体加载  
，且未做任何过滤与拦截。  
  
攻击者可通过自定义XML外部实体，调用本地文件协议、网络请求协议，强制服务器读取本地文件、访问内网资源、发起网络请求，攻击能力  
同时覆盖文件读取+SSRF内网攻击  
，危害性高于普通SSRF。  
  
2. 漏洞触发场景  
  
所有接收、解析XML数据的业务功能均可能存在漏洞：接口接收XML格式请求参数、后台支持XML文件上传、数据导入功能使用XML格式、小程序/接口数据传输采用XML封装。  
  
高危环境：PHP5.x全系、旧版Java、Python原生XML解析库、未做安全加固的C#解析组件。  
  
3. 漏洞原理与详细Payload  
  
XML支持通过DOCTYPE自定义外部实体，实体可绑定本地文件或远程URL，解析器会自动加载实体对应的资源并输出内容，攻击者利用该特性构造恶意代码。  
  
SSRF拓展Payload（内网探测）  
  
将实体绑定内网IP端口，可实现内网端口扫描、存活主机探测，效果与SSRF完全一致。  
  
4. XXE漏洞分类  
  
1. 回显型XXE：页面可直接输出实体内容，可直接读取文件、获取内网响应，利用简单直观；  
  
2. 盲XXE：页面无直接回显，需通过DNS外带、HTTP请求外带的方式，带出内网数据、文件内容，属于高阶利用方式。  
  
5. XXE漏洞详细危害  
  
1. 任意文件读取：无权限限制读取服务器系统文件、网站源码、数据库配置、账号密码、密钥证书等所有敏感资源；  
  
2. 全方位内网探测：扫描内网所有网段、端口、存活服务，精准获取内网拓扑结构，为横向渗透提供支撑；  
  
3. 内网服务高危攻击：对内网Redis、FastCGI、Jenkins、Zabbix等服务发起恶意请求，实现命令执行、写入后门、服务器沦陷；  
  
4. 服务器拒绝服务攻击：构造超大恶意实体、递归实体，占用服务器CPU、内存资源，导致XML解析卡死、全站服务瘫痪。  
  
四、三大漏洞核心区别与对比总结（超详细）  
<table><tbody><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">对比维度</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: Arial;font-variant: normal;text-transform: none;"><span leaf="">CSRF</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: Arial;font-variant: normal;text-transform: none;"><span leaf="">SSRF</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: Arial;font-variant: normal;text-transform: none;"><span leaf="">XXE</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">攻击发起主体</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">用户浏览器</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">漏洞Web服务器</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">服务器XML解析器</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">核心攻击能力</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">冒用用户身份执行业务操作</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">服务端内网探测、外网发包、端口扫描</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">文件读取+内网SSRF攻击双重能力</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">能否窃取数据</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">不能，仅可操作、不可读取</span></span><page></page></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">可读取请求响应数据</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">可直接读取服务器本地文件数据</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">攻击边界</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">仅局限于目标网站自身业务接口</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">突破防火墙，访问内网、本机、外网</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">完全突破边界，权限最大、范围最广</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">利用前提</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">用户登录+无CSRF防护+钓鱼诱导</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">存在可控URL参数+服务端主动发包</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">接收XML数据+未禁用外部实体</span></span></p></td></tr><tr><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">危害等级</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">中高危（业务篡改、账号劫持）</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">高危（内网渗透、服务器肉鸡化）</span></span></p></td><td data-colwidth="138" width="138" valign="top" style="border-width: 1pt;border-style: solid;border-color: rgb(222, 224, 227);padding: 3pt 6pt 1.5pt;"><p style="margin-top:6.0pt;margin-bottom:6.0pt;text-align:left;margin-left:0.0pt;line-height:120%;"><span style="font-size: 11pt;font-family: 等线;font-variant: normal;text-transform: none;"><span leaf="">超高危（文件泄露+内网沦陷）</span></span></p></td></tr></tbody></table>  
核心一句话总结  
  
CSRF是  
借用户的手改数据  
，SSRF是  
借服务器的网扫内网  
，XXE是  
兼具读文件和扫内网的全能高危漏洞  
。  
  
  
文章内容转自归不去的光影，侵删  
  
![](https://mmbiz.qpic.cn/mmbiz_png/INa3lxHH4I2aV3zCmfiaj4cXeQ2HQd6s53wJS36HYI65ib48fujDK8najfWiahicsljzsdT3dfVS8HHyxaviaSd8g2g/640?wxfrom=5&wx_lazy=1&wx_fmt=png&wx_co=1 "")  
  
  
**今日福利**  
  
为了帮助大家入门网安，给大家推荐一份**《新手Web安全入门到精通》**  
，共474页，包括**代码审计、web漏洞、靶场实例分析、信息收集、渗透思路**  
等，将Web安全攻防知识点一网打尽。  
  
代码配图，简单明了，攻防思路清晰透彻，关键是里面还配有多张思维导图，通俗易懂，实用性非常强，很适合新手学习参考~  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/UkV8WB2qYAnknFkWScXBu74vvYnrsKQeSxTJrcrLN5T9jWmO0WejeeRJzjYSo3QGnGGfibOmCRIZ8EGptiaTnKdQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/UkV8WB2qYAnknFkWScXBu74vvYnrsKQeF72M9DhzWE0U5434e5vlcoySfD4kicEibGeMVZpdnNRQqqWubWRUPHjg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/UkV8WB2qYAnknFkWScXBu74vvYnrsKQeoy1Raic5D6tP84jY8W0WQhGOnVCxrSsy1hAuOj9YIcsDnbXicRXxDyyg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/UkV8WB2qYAnknFkWScXBu74vvYnrsKQe0TTJ1MOQuQjyAMbyichkoSYxqLTCbNnIEJ8osu9dso3rJULxpwr6nUQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/NUwbCdTuQQxsJibSZGyA8akP9TVyJfPcpJ4uIZJDj3akRUfv6cNbnksGJQsibq1aH8iaGDic7TvOaSwNGXLdQ8PC9A/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp "")  
  
  
以上资料获取请扫码  
  
![](https://mmbiz.qpic.cn/mmbiz_png/2I159AwKj55AdUNSDkmXgRsU8ETjqV03OBkobNxJhv1icJYYI7ZvQibyLyTuw0xF2ibw7d8c13rhsicUWIHyOsn1NvGf3kLt0OtGjjzHtApNUG0/640?wx_fmt=png&from=appmsg "")  
  
识别上方二维码  
  
备注：  
**web安全入门到精通**  
  
100%免费领取  
  
（是扫码领取，不是在公众号后台回复，别看错了哦）  
  
  
  
  
