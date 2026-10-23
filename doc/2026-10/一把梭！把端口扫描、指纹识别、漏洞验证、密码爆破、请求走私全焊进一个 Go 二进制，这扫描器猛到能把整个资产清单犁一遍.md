#  一把梭！把端口扫描、指纹识别、漏洞验证、密码爆破、请求走私全焊进一个 Go 二进制，这扫描器猛到能把整个资产清单犁一遍  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-23 02:44  
  
免责声明：本文仅用于安全技术研究与学习交流，请勿用于任何未经授权的测试行为，使用者需自行承担全部法律责任。
scan4all 是一个用 Go 编写的自动化安全扫描框架，它把端口发现、Web 指纹识别、漏洞验证、目录与敏感文件探测、服务弱口令爆破、HTTP 请求走私、蜜罐识别、供应链信息提取等环节串成一条流水线，对一批目标跑完之后，输出结构化的结果，而不是一堆需要人工翻检的终端日志。
项目的定位不是再造一个单点工具，而是做一个调度层：底层复用 naabu、httpx、nuclei、subfinder、nmap 等成熟组件的探测能力，上层由自研的事件引擎统一编排调度、去重、限流、结果落库。
重点导读定位

目标：批量资产的自动化测绘与验证
输入：STDIN、HOST、IP、CIDR、URL、TXT 列表
输出：JSON、TXT、CSV、STDOUT
载体：单个 Go 二进制，配置文件通过 embed.FS 打包进程序
平台：Linux、Windows、macOS
存储：Elasticsearch、SQLite、本地文件
入口：lib/api.StartScan

重点导读组成
仓库按能力拆分为若干目录，模块边界比较清晰：



目录
职责




engine/
事件引擎，goroutine 池调度


lib/api/
启动入口，参数解析与阶段编排


lib/util/
配置、缓存、ES 落库、蜜罐识别、供应链提取、HTTP 封装


lib/Smuggling/
HTTP 请求走私探测


lib/socket/
裸 TCP 连接与检测目标构造


lib/crawlergo/
页面爬取与 chromedp 渲染


pkg/portScan/
nmap、masscan 端口扫描封装


pkg/fingerprint/
指纹匹配与 favicon 计算


pkg/hydra/
多协议弱口令爆破字典与执行


pkg/httpx/、pkg/naabu/
端口与 Web 探测


pkg/ksubdomain/、pkg/kscan/
子域名枚举


pkg/WhatWaf/
WAF 识别


pkg/jndi/
JNDI 相关载荷构造


pocs_go/
Go 版 POC 集合，按厂商与漏洞分类


pocs_yml/
YAML 版 POC 校验器


brute/
通用爆破、403 绕过、文件模糊、指纹模糊


webScan/
URL 层的 Web 扫描任务


config/
配置、字典、模板、Elasticsearch 初始化脚本



重点导读架构
PART 01事件引擎

全局单例 Engine
ants.PoolWithFunc 工作池
EventData 事件队列
sync.Map 事件类型注册表
SendEvent 事件级联
每类事件独立并发控制
事件内自去重
信号中断与优雅退出
延迟清理 DoDelayClear
pprof 性能分析端口

引擎把每一种扫描动作抽象成一个事件类型：启动时通过 EngineFuncFactory 把函数注册进 sync.Map，运行时从队列取出事件数据，按类型分发到对应回调。这样新增一种检测能力只需要注册一个函数，不必改动主流程。
PART 02阶段编排
启动链路是线性的：
参数解析 → 端口扫描 → HTTP 探测 → Web 扫描 → POC 校验 → 结果落库

端口扫描环节做了三条分支处理：直接读取 nmap XML 结果文件、跳过扫描直接复用已有结果、或者调用 naabu 执行实时枚举。端口扫描结束后立即触发 httpx 探测，把存活 Web 服务交给后续流程。
PART 03结果落库

initEs 初始化索引
按工具类型分索引
ESaveType 区分数据来源
SendReq 统一写入
独立线程池限速
JSON 序列化后批量提交

重点导读能力
scan4all 功能总览
PART 04端口发现

naabu 实时枚举
nmap 结果复用
masscan 探测封装
优先级可配置 priorityNmap
端口到协议的映射表
146 种协议识别
90000 条端口规则
nmap XML 解析
noScan 复用已有结果

nmap 扫描过程
PART 05指纹识别

eHoleFinger 指纹库
localFinger 本地指纹库
httpx 指纹结果融合
favicon 哈希计算
模糊匹配算法
指纹学习与缓存
7000 余条指纹规则
WAF 识别
异常页面容错

指纹识别同时走三条来源：httpx 返回的框架特征、vscan 的 eHoleFinger 与 localFinger 指纹库，以及 scan4all 自身维护的指纹数据。匹配过程带模糊比对，用于降低因页面改版导致的漏判。
PART 06漏洞验证

nuclei YAML 模板
内置 3000 余条模板
281 个模板目录
3922 个模板文件
xray 2.0 的 300 余条 POC
Go 编写 POC
按厂商与 CVE 分类
模板标签统计
自定义模板目录

PART 07密码爆破
pkg/hydra 覆盖 23 类服务的弱口令检测，每种服务独立维护用户名字典、密码字典与默认口令字典：

SSH
RDP
VNC
Socks5
MySQL
MsSQL
Oracle
PostgreSQL
Redis
MongoDB
FTP
SMB
Telnet
SNMP
Elasticsearch
RouterOS
WebLogic
Tomcat
JBoss
WinRM
POP3
SVN
WebDAV

SMB 爆破过程中顺带检测 MS17-010 系列与 SMBGhost。WebLogic 路径可配合 nuclei 开启 T3、IIOP 等协议探测。HTTP 侧默认开启智能爆破，在识别出需要口令的场景时自动激活。
PART 08请求走私
lib/Smuggling 实现了多种 HTTP Request Smuggling 变体：

CL-TE
TE-CL
TE-TE
CL-CL
BaseErr

每种变体独立实现载荷构造与响应判定，判定逻辑基于响应体特征匹配，例如识别服务端返回的 Unrecognized method GPOST 标记。基础结构统一收敛到 Base 类型，通过 GetPayloads、CheckResponse、GetVulType 三个方法暴露接口。
HTTP 请求走私检测
PART 09目录与敏感文件

敏感文件字典探测
后缀名扫描
备份文件定位
目录爆破
404 页面识别
内容类型模糊匹配
落地页检测
403 绕过

PART 10子域名

ksubdomain 集成
subfinder 集成
证书域名提取
SSL 信息关联分析
通配符域名匹配
结果自动回填扫描列表
正则可控的域名过滤

PART 11蜜罐与供应链
蜜罐识别读取响应的 Server 头，对超长或畸形的 Server 值做判定，结果写入内存缓存避免重复计算，命中后自动跳过该目标。供应链提取针对首页与二级页面做上下文识别，从响应体中提取开发商相关信息，相同 URL 上下文只提取一次。
PART 12POC 校验
Go 版 POC 以目录组织，覆盖 Apache、Confluence、Fastjson、GitLab、JBoss、Jenkins、Landray、Log4j、MCMS、Oracle、PHPUnit、Seeyon、Shiro、Springboot、Sunlogin、ThinkPHP、Tomcat、Tongda、VMware、WebLogic、Zabbix、Zentao 等厂商与组件。YAML 侧通过 pocs_yml 的校验器加载 ymlFiles 下的模板文件。
重点导读配置
配置以 config/config.json 为中心，通过环境变量覆盖同名键：

priorityNmap：是否优先使用 nmap
noScan：跳过端口扫描
enableNuclei：是否启用 nuclei
enableMultNuclei：多实例并发
UrlPrecise：精确匹配 URL
ParseSSl：解析证书信息
EnableSubfinder：证书子域名遍历
EnableKsubdomain：子域名爆破
EnableHoneyportDetection：蜜罐识别
enableEsSv：Elasticsearch 落库
ScanPoolSize：扫描线程池大小
MaxErrorTimes：错误上限
LimitReader：读取上限

字典路径同样在配置中声明，涵盖各服务的用户名、密码、默认口令，以及敏感文件字典、目录字典、404 URL 列表。
重点导读落地
典型用法：
bashgo install github.com/GhostTroops/scan4all@2.8.9

# 精确扫描 URL 列表
UrlPrecise=true ./scan4all -l xx.txt

# 跳过端口扫描，直接复用已有结果
noScan=true ./scan4all -l list.txt -v

# 关闭 nmap 适配，改用 naabu 内置端口表
priorityNmap=false ./scan4all -tp http -list allOut.txt -v

Elasticsearch 侧通过 config/initEs.sh 初始化索引，各工具的结果按索引分别存储，查询时可直接按目标 IP 或域名检索：
bashhttp://127.0.0.1:9200/nmap_index/_doc/_search?q=_id:192.168.0.111

扫描流程如下：
scan4all 工作流程
重点导读短评
优点：

调度层设计清晰，事件引擎让新增检测能力成本很低
覆盖面广，单工具难以同时覆盖端口、指纹、漏洞、口令四类问题
单一二进制交付，配置内嵌，部署成本低
结果结构化存储，适合接入后续分析流程

需要注意的地方：

组件集成度越高，依赖版本与上游行为变化带来的维护成本越大
字典爆破与漏洞验证的扫描速率需要按授权范围控制
nmap 与 masscan 在网络条件不佳时可能漏扫
强依赖外部工具与外部服务（nmap、Elasticsearch、log4j-scan），环境准备不全会直接影响链路

重点导读源码
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/GhostTroops/scan4all

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
