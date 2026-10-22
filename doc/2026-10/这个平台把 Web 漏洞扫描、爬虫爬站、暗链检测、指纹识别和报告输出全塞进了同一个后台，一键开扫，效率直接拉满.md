#  这个平台把 Web 漏洞扫描、爬虫爬站、暗链检测、指纹识别和报告输出全塞进了同一个后台，一键开扫，效率直接拉满  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-22 03:13  
  
免责声明：本文仅用于技术学习与交流。WDScanner 属于安全检测工具，对任何目标进行扫描、探测前须获得目标所有者合法授权，禁止用于未授权测试，使用者需自行承担相应后果。
WDScanner 是一套面向批量安全检测场景的 Web 漏洞扫描平台，代码以 PHP 5 与 Python 2 混合实现，控制端由 MySQL 支撑任务与资产数据，执行端通过多节点协同完成实际探测。平台把客户资产建档、周期扫描策略、信息搜集、整站爬取、专项漏洞检测、代理池维护与中文报告导出整合在同一套后台中，适合甲方安全团队、渗透服务团队和系统运维团队在同一界面内管理检测任务与产出交付物。开源版本已收录 670 条中文化漏洞描述条目，扫描结果可直接转成 Word 与 Excel 报告交付客户。
重点导读项目定位

分布式 Web 漏洞扫描平台
资产与客户集中建档
周期任务自动派发
中文漏洞库与中文报告
开源免费二次开发

重点导读核心架构
PART 01控制层
PHP 入口 index.php 依据 m 参数路由到 include/ 下的动作函数，再渲染 html/ 目录中的模板页面，动作层通过 include/common.fun.php 中的 __autoload 机制按类名自动加载 *.class.php 与 *.php 库文件，数据库访问统一收敛在 include/mysql.class.php。
PART 02执行层
Python 2 编写的执行脚本常驻运行于扫描节点，负责实际探测动作，脚本通过 Windows 注册表读取本地工具路径，再拉起外部扫描器完成工作。
PART 03数据层
MySQL 数据库 wdscan 承载全部业务数据，核心表包括客户表、扫描任务表、目标信息表、漏洞表、爬虫结果表、代理表与节点表，另有 vul_cn 表存放汉化漏洞条目。
重点导读任务调度
PART 04轮询模型
控制端的 taskscan.php、taskspider.php、taskinfo.php 三个 HTTP 接口充当任务队列，节点脚本以固定间隔发起请求，命中待处理任务时返回 Base64 编码的 JSON 描述，节点解码后执行，再将结果写回数据库，对应任务状态由 new 流转到 ing 直至 ok。
PART 05节点鉴权
include/config.inc.php 中定义的 authkey 参与节点请求校验，point_server 表维护允许请求任务的节点标识，未在登记范围内的节点无法领取任务，扫描任务不会被未授权主机取走。
PART 06批量回写
taskscan.py 将扫描命令写入 bat/ 目录下的批处理文件后调用执行，扫描产物落在 report/<hash>/ 目录，hash 作为任务在数据库与文件系统之间的一致性标识。
重点导读漏洞扫描
PART 07扫描内核
节点通过注册表键 SOFTWARE\Wow6432Node\Acunetix\WVS10 定位 Acunetix WVS 10 的 wvs_console.exe，以 /Scan 参数发起扫描，配合 /ExportXML、/SaveLogs、/SaveFolder 输出结构化结果文件，目标地址、站点认证账号与扫描规则随任务一同下发。
PART 08漏洞汉化
vul_cn 表收录约 670 条漏洞条目，字段包含漏洞名称、漏洞描述、危害等级与加固建议，扫描结果回填时以中文条目覆盖原始英文描述，用于降低报告交付门槛。
PART 09策略选择
扫描任务可绑定不同扫描策略，节点按策略参数裁剪检测范围，在覆盖度与耗时之间做取舍，适用于不同交付周期下的检测安排。
重点导读信息收集
PART 10指纹识别
TaskPython/TaskInfo/whatcms.py 调用 whatweb 与 whatcms 完成指纹识别，输出开发语言、框架、CMS 名称与中间件类型，采集结果以独立文本文件形式落盘。
PART 11端口服务
taskinfo-win.py 解析 nmap 输出文本，识别开放端口及对应服务，从 Running 与 OS details 段落提取操作系统版本信息。
PART 12子域名
子域名采集在日志目录下以 -subdomain.txt 独立留存，结合暴力枚举与互联网检索两种途径完成。
PART 13弱文件与防护
敏感文件扫描结果写入 weakfile 字段，WAF 识别结果独立入库，中间件类型与站点标题同步记录，为漏洞结果提供上下文。
重点导读网站爬虫
PART 14遍历策略
TaskPython/taskspider.py 基于 requests 与 sgmllib 的 SGMLParser 自行实现解析，linkQuence 类维护已访问、未访问与外部链接三份队列，深度优先的层级控制配合去重逻辑完成整站遍历。
PART 15并发模型
WorkManager 与 Worker 组成线程池，默认 20 个工作线程并发处理页面任务，请求超时与异常均被捕获，单页失败不影响整体爬取进度。
PART 16暗链检测
爬虫对站外链接调用腾讯网址安全接口校验，返回结果中白名单类型为 2 时判定为暗链，记录链接地址与所在父页面。
PART 17敏感词与坏链
敏感词匹配读取 TaskPython/filters.txt 关键词表，对页面内容逐条比对并定位命中词，请求失败的链接记入坏链清单，命中页面与外链页面分别保存快照。
重点导读代理资源池
PART 18代理采集
TaskPython/proxy/ip_pool.py 通过 lxml 的 XPath 解析代理站点页面，批量抽取候选代理地址写入本地集合。
PART 19质量评分
assess_quality.py 对代理做多轮连通性测试，按成功率、失败次数与平均响应时间计算分值，超时次数与成功率低于阈值的代理从 valid_ip 表中淘汰，剩余代理按分值降序供扫描任务取用。
重点导读报告输出
PART 20导出格式
include/doc.class.php 的 MhtFileMaker 类将 HTML 内容打包为 MHT 文档用于生成 Word 文件，include/XmlExcel.php 与 include/IExcel.php 负责 Excel 表格输出。
PART 21报告结构
报告按任务 hash 组织，涵盖概述信息、漏洞展示、敏感字命中与信息泄露四类内容，汇总数据由 report.action.php 中的 report_general 等函数从 info 表与 spider 表联表统计得出。
报告概述
漏洞展示
重点导读检索中心
平台提供多维度检索入口，检索维度覆盖漏洞类型、操作系统类型、开放端口、中间件类型与开发技术，并支持按 URL 特征做站点级检索，用于在批量结果中快速定位同类风险。
检索中心
重点导读客户与任务管理
客户与其持有系统以表结构对应建档，站点条目可配置专属扫描策略与周期性执行计划，任务列表按状态流转，节点仅执行被授权的任务。
客户管理
分布式扫描
重点导读节点管理
平台界面提供节点列表，展示各扫描节点的在线状态与所属分组，节点分组策略与 point_server 表中的登记信息相对应。
节点管理
重点导读登录与界面
后台以会话方式维护登录态，Checklogin 函数在路由前对未授权请求做拦截，业务页面通过 html/ 目录下的模板文件渲染。
登录界面
重点导读输入过滤
include/sqlsafe.php 定义的 sqlsafe 类在应用启动阶段遍历 GET、POST 与 COOKIE 参数，以正则匹配注入特征，命中后记录日志并终止请求，配合 addslashes 构成双层参数防护。
重点导读技术栈

服务端 PHP 5.x
数据库 MySQL
脚本层 Python 2.7
扫描内核 Acunetix WVS 10
探测工具 nmap、whatweb、whatcms
前端原生 HTML 与 jQuery

重点导读部署要点
安装依赖 PHP 运行环境、MySQL、Python 2.7、nmap、whatweb 与 WVS 10.5，数据库脚本位于 wdscanner.sql.zip 中，导入后需修改 include/config.inc.php 的数据库连接配置。节点侧分别运行 TaskPython/Tide-proxy-bat.bat 与 TaskPython/Tide-WDScanner-bat.bat 拉起后台任务与代理采集进程。代码中存在较多硬编码绝对路径，二次开发时需相应调整。
重点导读开源地址
本公众号非项目作者，仅做技术分享。
本文介绍的项目开源地址如下：
https://github.com/TideSec/WDScanner

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
