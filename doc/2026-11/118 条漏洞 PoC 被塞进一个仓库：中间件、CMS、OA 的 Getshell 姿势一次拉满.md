#  118 条漏洞 PoC 被塞进一个仓库：中间件、CMS、OA 的 Getshell 姿势一次拉满  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-05 02:41  
  
重点导读免责声明
本文仅用于安全技术研究与防御能力建设。所涉及的验证与利用代码仅限在授权环境中测试使用，禁止用于任何未授权目标，使用者自行承担全部责任。
重点导读项目概览
POChouse 是一个公开整理的漏洞验证与利用代码集合。仓库以厂商和产品作为一级目录，将常见中间件、CMS、协同办公系统、网络与安全设备的已知漏洞逐条归档。
当前仓库包含 39 个厂商或产品目录、3 个聚合分类目录、129 个目录节点，其中 118 个节点带有独立说明文档。漏洞条目以 CVE 编号命名的有 53 个，其余以 CNVD 编号或厂商内部编号标识。配套代码涵盖 45 个 Python 验证脚本、8 个 JAR 利用包，以及 NSE、Go、Shell、Java、JSP、PHP 等形态。
项目在结构上把验证与利用拆成两层：POC 负责判定目标是否存在风险，EXP 负责在授权前提下推进利用链，两者分开存放，便于只做检测而无需引入攻击性载荷。
重点导读收录范围

高危可 Getshell 漏洞及配套利用手法
中间件：Weblogic、Tomcat、JBoss、Shiro、Struts2、ActiveMQ、Solr、Spark、Flink、Kylin、Nacos
CMS：WordPress、Discuz、Drupal、Joomla、ECshop、ThinkPHP、ThinkCMF、FastAdmin、骑士CMS
DevOps 与基础组件：Jenkins、Gitlab、Harbor、Nexus、Nacos、Zabbix、Kibana、Jira、XXL-JOB、Spring
协同办公 OA：泛微、致远、通达、蓝凌、禅道
网络与安全设备：F5 BIG-IP、VMware vCenter、VMware View Planner、VMware vROps、Exchange、SAP
Fastjson、phpMyAdmin、Jellyfin 等高频通用组件

重点导读目录结构
PART 01一级分类

厂商目录：39 个
聚合目录：3 个，分别承载协同办公系统、编辑器上传漏洞、网络与安全设备

PART 02漏洞条目

命名规则：产品名 + 漏洞描述 + 编号
编号体系：CVE 优先，其次 CNVD，最后为厂商内部编号
版本约束：目录名直接标注受影响版本区间

PART 03文档模板

应用简介
相关资产指纹
识别方法
环境搭建

PART 04脚本形态

Python 验证脚本：批量下发、响应判定
JAR 利用包：反序列化 gadget 链
NSE 脚本：接入 nmap 扫描流程
Go 工具：SSRF 回连与代理隧道
Shell 脚本：证书提取与 JDK 依赖的复合利用
WebShell 载荷：JSP、PHP

重点导读资产识别
多数条目在文档中给出可直接用于资产测绘的指纹语句，例如 Fastjson 对应 app=Fastjson，VMware vCenter 对应 app=vmware-vCenter 与 title=ID_VC_Welcome，Jenkins、Cacti、Zabbix 等也各自带有区分度较高的标题特征。指纹语句配合批量验证脚本，可以把测绘平台的资产清单直接转成候选目标队列。
识别方法一栏给出了多种判定路径：报错回显用于确认服务端解析行为，DNS 回显用于在无回显环境下确认请求是否出网，响应状态码与特征字符串组合用于区分版本差异。部分条目还给出了无害化的探测载荷，用于在正式验证前先行确认组件指纹。
重点导读使用流程

依据指纹语句从资产测绘结果中筛出候选目标
运行目录内 POC，完成无副作用的存在性验证
依据响应特征与版本区间确认受影响组件
在授权测试环境内使用 EXP 完成后续利用

重点导读集成方式

POC 侧：与 nuclei 模板体系对接，可通过大宝剑 DBJ 直接下发批量任务
EXP 侧：并入统一的 Python 利用框架调度，支持命令执行与回连
插件侧：部分条目按 Goby 插件形式组织，可直接装载

三种形态共用同一套条目目录，新增漏洞只需追加一层产品目录，无需改动调度逻辑。
重点导读界面
项目横幅
仓库中的 EXP 条目支持命令执行与回连两种结果输出：
利用框架输出
重点导读适用场景

授权渗透测试中的漏洞可用性验证
中间件与 CMS 版本暴露面自查
安全基线加固前的历史漏洞排查
检测规则与 POC 集成的回归测试
安全产品规则库的比对基线

重点导读目录索引



分类
代表条目




中间件
Weblogic T3 与 IIOP 反序列化、Shiro 550、Tomcat 未授权上传、ActiveMQ 反序列化


CMS
Drupal Geddon、Joomla 3.9.24 普通管理员 RCE、骑士 CMS 远程代码执行、ThinkPHP 漏洞


DevOps
Jenkins CLI 反序列化、Gitlab 未授权 RCE、Harbor 任意管理员注册、Nexus 访问控制缺失


OA
泛微 Bsh 远程代码执行、通达 OA 文件包含、致远 OA 任意文件上传、禅道文件上传


设备
F5 TMUI 远程代码执行、VMware View Planner 未授权 RCE、Exchange SSRF 转 RCE



重点导读合规边界
仓库条目多以未授权 Getshell 为目标，默认前提是测试对象属于授权范围。建议在隔离内网中使用验证脚本，EXP 环节使用专用测试主机，避免触及生产数据与真实业务链路。默认口令类条目仅应用于自有设备的初始化验证。
重点导读项目地址
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/DawnFlame/POChouse

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
