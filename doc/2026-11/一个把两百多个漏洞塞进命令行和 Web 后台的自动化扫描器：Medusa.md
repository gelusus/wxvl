#  一个把两百多个漏洞塞进命令行和 Web 后台的自动化扫描器：Medusa  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-09 02:50  
  
免责声明：本文内容仅用于技术学习与授权范围内的安全测试，禁止用于任何未授权的目标与用途。
Medusa 是一套以 Python 编写的自动化漏洞扫描框架，同时提供命令行版本与基于 Django 的 Web 后台。仓库内共有 22 个一级插件分类、62 个细分插件目录，Modules 目录下累计 327 个插件脚本文件，其中 CMS 方向占 202 个，覆盖 53 个开源内容管理系统。框架把中间件漏洞检测、CMS 漏洞检测、敏感信息泄露探测、子域名收集、端口扫描与漏洞利用整合进同一套插件协议，配合 SQLite 结果存储、DNSLog 回连验证与 Word 报告导出，形成从资产探测到漏洞利用的完整链路。
重点导读项目定位
Medusa 面向的是授权测试场景中的批量目标排查工作。它解决的不是单点漏洞研究问题，而是同一批目标需要在多套中间件、多套 CMS 之间反复试探的问题。
框架的适用场景集中在三处：资产规模较大、需要按组件批量派发检测；需要同时输出检测结果与修复建议；需要在检测基础上进入交互式命令验证环节。
重点导读功能全景
PART 01漏洞检测
中间件方向按厂商划分子插件，Apache 分支下包含 ActiveMQ、Flink、Log4j、Solr、Tomcat、Shiro 六个方向。Struts2 分支覆盖 S2_001、S2_005、S2_008、S2_009、S2_015、S2_016、S2_045、S2_046、S2_052、S2_057 共十个编号。Weblogic 分支包含十三个检测脚本，覆盖 XMLDecoder 反序列化与远程命令执行。
CMS 方向是插件数量最多的部分，仓库内的 Cms 目录覆盖 74CMS、AfterLogicWebMail、B2Bbuilder、BaijiaCMS、BearAdmin、BEESCMS、BlueCMS、Bocweb、BugFree、Cacti、ChamiloLMS、ChanZhiEPS、CmsEasy、CMSimple、CMSMS、CmsTop、Coremail、CraftedWeb、CsCMS、CSDJCMS、CTSCMS、CuteCMS、Cyberwisdom、DaMall、DamiCMS、Destoon、DRUPAL 等五十余套系统，检测类型集中在 SQL 注入、命令执行、任意文件包含与反射型跨站脚本。
OA 方向分出 Seeyou、Tongda、Weaver、Ruvar 四个厂商分支，Spring 方向覆盖八个组件，Jenkins 分支四个脚本，FastJson 分支四个脚本。
PART 02信息收集
信息泄露探测集中在 InformationLeakage 目录，包含 Git、SVN、SFTP、Druid、JetBrains、PHPApc、Java 压缩包解析、敏感文件扫描与目录遍历等方向。
子域名收集由 Subdomain 模块实现，基于 tldextract 解析根域，依赖 dnspython 完成解析，配合 IPy 处理网段。
端口扫描调用 python-nmap 驱动外部 nmap 工具，扫描结果写入独立数据表。
PART 03漏洞利用
Exploit 目录维护交互式漏洞利用列表，当前收录 WebLogic CVE-2019-2729、WebLogic XMLDecoder 反序列化 CVE-2017-10271、Solr CVE-2019-0193、Solr Velocity 模板 CVE-2019-17558、PHPStudy 后门与 Citrix 认证绕过 CVE-2020-8193。
命令行通过 -l 查询可交互模块，通过 -e 指定模块名，进入交互式命令输入循环。
PART 04报告输出
Web 版本内置 Word 报告生成链路，基于 docxtpl 渲染模板文件，字段包含目标地址、漏洞数量、漏洞名称、危害等级、发现时间、漏洞描述、漏洞细节与修复建议，首页嵌入图片，生成后按时间戳命名写入下载目录。
重点导读核心架构
PART 05分层结构

入口层：MedusaScan.py
调度层：ProcessPool、ThreadPool
插件层：Modules 目录下的独立脚本
公共层：ClassCongregation.py
存储层：SQLite
外联层：Dnslog

PART 06插件模型
单个插件文件承载两个结构：VulnerabilityInfo 类负责声明元数据字段，包含 CVE 编号、插件名称、漏洞名称、影响组件、漏洞描述、危害等级、修复建议、影响版本与披露时间；medusa() 函数负责发起检测请求并返回结果。
元数据字段由 VulnerabilityDetails 类统一写入 SQLite 表，插件本身不直接接触数据库。
PART 07调度模型
跨插件调度使用进程池，规避解释器全局锁对并发度的限制；单个插件内部的多目标派发使用线程池。
PART 08公共能力
ClassCongregation.py 集中提供随机 User-Agent 生成、代理配置、URL 解析、DNSLog 域名生成与回连判定、错误日志写入、Nmap 数据读写、CVE 数据写入、终端横幅渲染等能力。
PART 09依赖清单
requirements 以 Medusa.txt 形式维护，涵盖 requests、urllib3、beautifulsoup4、fake-useragent、python-nmap、PyMySQL、IPy、scrapy、tqdm、dnspython、tldextract、Django、Celery、django-redis、eventlet、pyDES、docxtpl、pycryptodome、PySocks、django-cors-headers、django-simple-captcha 与 dubbo-py。
重点导读目录结构
Medusa/
├── MedusaScan.py        命令行入口
├── ClassCongregation.py 公共能力类集合
├── Exploit/             交互式漏洞利用
├── Modules/             插件目录
├── Web/                 Django 后台
├── docs/                文档站与插件目录清单
├── config.py            DNSLog、线程数、调试模式配置
└── Dockerfile

重点导读快速部署
PART 10Docker 方式
镜像基于 alpine 构建，入口命令为 MedusaScan.py，日志与临时目录通过卷挂载。
bashdocker run --rm -it \
    -v $(pwd)/Log:/usr/src/app/Log \
    -v $(pwd)/Temp:/usr/src/app/Temp \
    registry.cn-beijing.aliyuncs.com/rapid7/medusa \
    python3 MedusaScan.py -u http://172.21.7.225:80 -t 3

PART 11依赖安装
bashpip3 install -r Medusa.txt

端口扫描依赖系统层 nmap 二进制，需另行安装。
重点导读命令行用法



参数
作用




-u
指定单个目标 URL


-f
指定批量目标文件


-m
指定单个插件


-e
指定交互式利用模块


-l
列出可交互插件


-t
设置进程数


-p
设置代理地址


-a
指定请求头或启用随机头


-s
开启子域名探测


-d
指定反序列化利用方式



单目标扫描示例：
bashpython3 MedusaScan.py -u https://www.example.com

重点导读Web 后台
PART 12技术选型
Django 3.0.5 提供路由与视图层，Celery 配合 Redis 承担异步任务分发，django-simple-captcha 引入验证码，django-cors-headers 处理跨域。
PART 13接口清单
路由表注册十三个接口，覆盖扫描任务下发、主动扫描列表查询、扫描关系查询、漏洞明细查询、注册登录、Word 报告生成与下载、用户资料查询与密码、显示名、密钥更新。
/api/vulnerability_scanning/
/api/active_scan_list_query/
/api/scan_information_query/
/api/medusa_query/
/api/registered/
/api/login/
/api/generate_word/
/api/download_word/
/api/user_info/
/api/update_password/
/api/update_show_name/
/api/update_key/

PART 14鉴权模型
登录接口校验用户名与密码后签发 token，token 带时效性，扫描类接口依赖 token 识别调用方身份。
PART 15框架联动
MSFModules 目录通过 msgpack 与 Metasploit 标准 API 通信，维护 Token 数据表完成会话管理。ProxyScan 目录提供代理扫描服务，配合 mitmproxy 与 Celery 支撑被动扫描模式。
重点导读工具集成
项目在二次开发中并入了 pocsuit3、鲲鹏、巡风与 TideSec/Mars 的部分 PoC/Exp 脚本，并在 Web 侧调整了页面组织方式与后台管理控件，使多套自动化渗透工具的脚本与扫描流程可以在同一套面板中调度。
重点导读演示
Medusa 扫描过程演示
重点导读使用边界
框架包含命令执行、文件写入与认证绕过方向的检测与利用逻辑，运行前需确认目标归属与授权范围。建议在隔离网络环境中部署，并对 DNSLog 与 ceye 等回连通道使用独立配置。
重点导读项目地址
本文介绍的项目开源地址如下：
https://github.com/xx-zhang/Medusa

本公众号非项目作者，仅做技术分享。
## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
