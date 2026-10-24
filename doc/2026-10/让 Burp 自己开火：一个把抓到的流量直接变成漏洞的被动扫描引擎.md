#  让 Burp 自己开火：一个把抓到的流量直接变成漏洞的被动扫描引擎  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-24 02:44  
  
本文仅用于安全技术学习与授权测试环境的交流，禁止用于任何未经授权的目标攻击与探测。
myscan 是一个用 Python 3 编写的被动式漏洞扫描器。它的定位很直接：不再主动爬取目标站点，而是接管 Burp Suite 已经在真实业务流量中产生的请求与响应，在这些数据包上挂载 POC 进行检测。流量从哪来、检测怎么跑、结果往哪去，全链路都可以在代码里逐段读出来。
重点导读定位
被动扫描。检测对象是流量，不是页面。
工作方式是复用 Burp Suite 扩展抓取到的 HTTP 交互，在其之上执行漏洞验证逻辑，输出结果并落库。
这一思路带来两点现实收益：检测发生在真实会话上下文中，可以直接复用 Cookie、Token 与业务参数；检测目标范围等于实际访问过的接口，而不是站点目录树推测出的路径。
重点导读组成
PART 01组件
项目按功能拆成若干独立目录：

myscan/cli.py：命令行入口，分发 webscan、hostscan、reverse 三种模式。
myscan/lib/core/：调度、进程管理、结果存储、HTML 输出等核心逻辑。
myscan/lib/parse/：命令行参数解析与数据包解析。
myscan/pocs/：POC 目录树，按检测维度分四层。
myscan/plugins/：插件目录，webscan 与 hostscan 各一套。
myscan/reverse/：反连平台服务端。
myscan/web/：Flask 查询面板。
myscan/exp/：独立功能模块，如 Shiro 密钥识别、DNS 区域传送。
myscan/data/：字典与口令数据。
myscan/lib/bin/：Java 侧 POC 的可执行产物。

PART 02依赖
外部依赖集中在三类：Burp Suite 及其扩展 myscan_burp_extension.jar、Redis 服务、Python 依赖包（requirements.txt 中包含 redis、requests、flask、dnspython、pysmb、pymysql、elasticsearch-dsl、pyjnius 等）。
Redis 在这里不是可选缓存，而是整个系统的总线：数据入口、任务队列、结果存储三件事都由它承担。
重点导读数据流
PART 03入口
Burp 扩展监听代理流量，把每个 request/response 整理成 JSON 写入 Redis 列表 burpdata。扫描端在 webscan 模式下循环 rpop 取包，取到后生成一个随机任务 ID，用这个 ID 挂载后续所有衍生任务。
hostscan 模式的入口不同，数据来自 nmap 的 XML 报告、nmap 文本报告或自定义 JSON 文件，由 start_input 推入 hostdata 列表。
myscan 运行流程图
PART 04分流
数据包在 dictdata_parser 中被拆解成四个检测维度：

perfile：按单个 URL 文件发起检测。
perfolder：按 URL 中的目录层级发起检测，同一目录下多个 POC 共享一份目录数据。
perscheme：按整个请求方案发起检测，适合参数级漏洞。
perserver：面向端口与服务的检测，只在 hostscan 模式使用。

分流结果被序列化成任务，写入 work_data_py。perfile 与 perscheme 两个维度带去重标记，同一个 URL 与同一套方案不会重复入队。
PART 05调度
调度由多进程完成，process_start 按 --process 拉起 POC 进程池，按 --process-plugins 拉起插件进程池，全部为守护进程。
任务取出有两种策略：random_test 为真时用 Redis 集合 SPOP 随机取任务，避免短时间内对同一主机形成密集请求；为假时用列表 LPOP 顺序消费。
每个进程消费到任务后，通过 signal.alarm 设置单任务超时，默认 1800 秒，防止单个 POC 阻塞整个进程。任务引用计数用 Redis HASH 维护，计数归零即删除，防止 Redis 内存膨胀。
PART 06熔断
block_info 维护 host:port 维度的失败计数，默认连续 500 次请求失败即把该主机端口加入黑名单，后续任务直接跳过。
重点导读POC
PART 07目录
POC 目录树本身就是检测能力的索引。以 Web 框架为例，Struts、Spring、SpringBoot、WebLogic、Tomcat、Shiro、Jolokia 各有独立分支；中间件与业务系统覆盖 Apache OFBiz、Flink、Cacti、Citrix、ColdFusion、Confluence、Druid、Drupal、Elasticsearch、Kibana、Kong、Laravel、Nexus、Oracle EBS、PHPStudy、SAP、Seeyon、SharePoint、Solr、SonarQube、ThinkPHP、ThinkAdmin、Tongda、UEditor、VMware、WordPress、Zabbix 等；通用检测包括目录扫描、XSS、SQL 注入、XXE、CORS、JSONP、CRLF、命令注入、SSRF、SSTI、越权、敏感信息泄露与重定向。
PART 08筛选
筛选条件有三层，全部作用于文件名匹配：

--enable：只加载文件名包含指定关键字的 POC。
--disable：排除文件名包含指定关键字的 POC，all 表示全部关闭。
--level：按 POC 自声明的等级过滤，等级从 Info 到 Critical 共五档。

启动阶段会遍历目录树实例化每个 POC 的类，据此读取其 level 属性做过滤，之后按维度登记进内存中的模块表，供多进程按需取用。
PART 09插件
插件与 POC 是两套并行的机制。POC 负责判定漏洞，插件负责对流量做旁路处理，两类任务分别进入 work_data_py 与 plugin_data_py 两个队列，由不同进程池消费。
plugins/webscan/es_import.py 是典型的插件实现：把请求响应数据写入 Elasticsearch，便于事后按条件检索。连接地址、认证信息与去重开关在 config.py 的 db_set 中配置。
重点导读主机
hostscan 面向端口与服务层，覆盖 MongoDB 未授权、MySQL 与 MSSQL 弱口令、Redis 弱口令、SSH 与 SMB 弱口令、SMB 信息泄露与 MS17-010、RMI 反序列化、Samba CVE-2017-7494，以及多个 WebLogic 反序列化漏洞。
这些 POC 中有一部分依赖 Java 侧产物，myscan/lib/bin/weblogic/ 下保留了对应的 jar 文件，通过 pyjnius 在 Python 进程内调用。
重点导读反连
反连平台是独立模式，cli.py reverse 启动，不参与扫描调度。
它同时提供四类回连通道：HTTP、RMI、LDAP 与 DNS。HTTP、RMI、LDAP 各自作为独立进程运行，DNS 服务在主进程中运行。
配置集中在 config.py 的 reverse_set：HTTP、RMI、LDAP 各自的监听地址与端口，反连域名，以及访问密码。密码未配置时随机生成。
所有记录写入 SQLite 的 reverse.db。DNS 通道的载荷采用十六进制编码，回连记录由 decode_dns 还原为原始字符串。
反连平台可部署在内网或外网，反连域名通过 NS 记录指向指定的解析服务。
重点导读报告
PART 10落地
POC 判定命中后，结果以 pickle 序列化写入 Redis，键名形如 result_<随机串>，字段包含名称、URL、等级、发现时间与详情。
start_write_results 负责把结果转成 HTML 报告，默认输出 myscan_result.html，单个报告的漏洞条目上限为 120 条，超出后另起新文件。报告内嵌 Prism.js，对原始请求做语法高亮。
PART 11查询
除被动匹配已知特征的搜索模式外，myscan/web/app.py 提供一个独立的 Flask 面板，读取 Redis 中的漏洞列表，支持关键词检索与分页，默认监听 5000 端口。
重点导读环境
平台要求为 Linux。Windows 不受支持，原因在于多进程之间无法共享变量。
Python 版本建议 3.7.5。部分高版本存在兼容性问题。
部署顺序为：启动 Redis、安装依赖、在 Burp Suite 中加载扩展、运行扫描命令。
重点导读边界

强依赖 Burp Suite 与 Redis 两条外部链路，任一环节缺失则整个流程无法运转。
数据入口决定了检测覆盖面，未被访问过的接口不会进入检测范围。
Python 进程模型决定了跨平台实现的困难度。
内置反连平台需要公网可达的资源才能完成闭环验证。

重点导读源码
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/amcai/myscan

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
