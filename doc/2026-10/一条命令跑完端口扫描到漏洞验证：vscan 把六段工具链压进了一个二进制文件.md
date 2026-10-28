#  一条命令跑完端口扫描到漏洞验证：vscan 把六段工具链压进了一个二进制文件  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-28 02:41  
  
免责声明：本文仅用于技术学习与研究，所有扫描、爆破与漏洞验证操作必须在获得明确授权的目标上进行，非法使用产生的后果由使用者自行承担。
vscan 是一个用 Go 编写的 Web 资产漏洞扫描工具。它把端口探测、HTTP 存活探测、服务指纹识别、后台与中间件弱口令爆破、敏感文件发现、POC 漏洞验证这六件事收敛进同一个可执行文件，使用者只需提供 IP、域名、C 段或 URL 列表，探测、识别、验证、落盘由程序内部串联完成，不需要在不同工具之间手工搬运结果。
重点导读定位

资产范围：IP、域名、C 段、URL、文本列表、标准输入
探测阶段：端口扫描、HTTP 存活、CDN 识别
识别阶段：wappalyzergo、eHole 规则、本地规则三路指纹
验证阶段：Go POC、Xray YAML、Nuclei 模板
收尾阶段：弱口令爆破、敏感文件发现、结果导出
输出形态：STDOUT、TXT、JSON、CSV

重点导读架构
代码整体由四个可执行单元构成，端口层与 HTTP 层分别内联了 naabu v2 与 httpx 的源码副本，扫描器层承载指纹、爆破、POC 三类能力，公共层提供 HTTP 请求封装与 JNDI 辅助服务。
PART 01链路

目标归一
IP 展开
CIDR 合并
端口集合解析
组合顺序打乱
SYN 探测
CONNECT 探测
TCP 二次校验
HTTP 存活请求
跳转链跟踪
指纹匹配
敏感文件探测
差异技术栈提取
POC 校验
结果聚合
导出

重点导读能力
PART 02端口
端口层位于 pkg/naabu/v2，具备特权时通过 gopacket 原始套接字发送 SYN 包，无特权环境回退到 CONNECT 全连接探测。IP 与端口的组合顺序由 blackrock 洗牌，避免固定顺序导致的漏扫与探测集中。扫描进度写入 resume.cfg，中断后可续扫；命中 CDN 网段的 IP 默认只保留 80 与 443 两个端口，通过 cdncheck 判断。并发由 sizedwaitgroup 控制，发包速率由 ratelimit 限速。被动模式从 Shodan IDB 拉取历史端口用于快速扩面，-nmap-cli 可把存活结果交给 nmap 做服务版本识别。
PART 03指纹
指纹来源分三路：wappalyzergo 基于响应头、响应体与 Cookie 做通用识别；pkg/fingerprint/dicts/eHoleFinger.json 提供 650 条 eHole 规则；pkg/fingerprint/dicts/localFinger.json 提供 3131 条本地规则。规则以位置与方法的二维组合描述，位置覆盖 body、header、title，方法覆盖 keyword 关键字包含、regular 正则匹配、faviconhash 图标哈希。favicon 路径先从 HTML 的 href 属性中正则提取，提取失败回退到 /favicon.ico，内容按 Shodan 风格的 mmh3 哈希后参与匹配。三路结果在写入输出前排序去重。
PART 04爆破
brute 目录负责凭据检测。后台登录页的表单结构靠正则推断：用户名输入框匹配 user、log、account 的各种大小写变体，密码输入框匹配 pass、pwd、mima、word 变体，提交地址从 form 的 action 属性或页面中的 ajax url 中解析。检测前先用 admin 与 test 账号搭配错误密码各打一次请求，建立 302、401、200、500 四类响应基线，之后凭响应差异判定成功：401 场景比对状态码，302 场景比对 Location 跳转地址，200 场景比对响应长度差并追加一次确认请求。页面含验证码、锁定、次数超限等提示时立即终止该目标的爆破。检测到 md5.js 时密码在提交前做 MD5 处理。中间件凭据检测覆盖 Basic 认证、Tomcat 的 /manager/html、JBoss 与 WebLogic 的管理控制台，各自带独立字典。
PART 05文件
brute/dicts/filedic.txt 收录 1136 条路径，覆盖各类备份包、压缩包、数据库文件、编辑器残留、上传目录、测试文件。执行时先请求一个不存在的路径取得 404 基线，再逐条比对状态码、标题与响应长度，避免把站点自定义的错误页当作命中。备份包类路径改用 HEAD 请求，并依据 Content-Type 判定是否命中。命中路径的响应特征会反推技术栈，/manager/html 返回 401 判定为 Tomcat，/console/login/LoginForm.jsp 含 Oracle 字样判定为 WebLogic，/wls-wsat 系列路径判定为 WebLogic，/jmx-console/ 含 jboss.css 判定为 JBoss，/seeyon/、/zabbix/、/grafana/、/zentao/、actuator 系列路径分别对应 Seeyon、Zabbix、Grafana、禅道与 Spring 环境端点。反推出的技术栈与首轮指纹取差集，只对新增项做 POC 校验。
PART 06POC
Go POC 分布在 21 个目录中，覆盖 Shiro、Tomcat、WebLogic、JBoss、Jenkins、Confluence、GitLab、F5 BIG-IP、Spring 系、ThinkPHP、Fastjson、Seeyon、通达 OA、蓝凌 OA、禅道、Zabbix SAML、Spark、Sunlogin、MCMS 等组件，累计 40 余个 CVE 与未授权访问场景。Xray 侧内嵌 354 个 YAML 模板，Nuclei 侧内嵌 1795 个模板，按目录组织为 cves、default-logins、vulnerabilities 等分类。模板与指纹按名称前缀或标签匹配，Nuclei 侧排除 apache、java、php 标签以减少与 Go POC 的重复探测。需要回连的验证走 DNSLog，接口参数为 ceye.io 的 api 与 domain。pkg/jndi 另提供 LDAP 握手伪造与日志采集的辅助实现，用于 JNDI 类验证的回连观测。
重点导读规则
PART 07指纹库

eHole 规则：650 条
本地规则：3131 条
wappalyzergo：3000 条以上

PART 08POC 库

Go POC：21 个组件目录，40 余个 CVE
Xray YAML：354 个模板
Nuclei 模板：1795 个

PART 09字典

敏感文件路径：1136 条
通用口令：106 条
Tomcat 账密：290 组
WebLogic 账密：13 组
JBoss 账密：4 组
Shiro 默认密钥：内置常用组合

重点导读输入
目标来源包括 -host 单目标、-l 列表文件、-eh 与 -ef 排除项，列表内允许混合 IP、域名、C 段与 URL，支持标准输入管道。端口范围由 -p 指定区间或列表，-top-ports 使用常用端口集合，默认取 HTTP 常用端口，另有 -ep、-pf 排除项。输入中形如 URL 的条目不参与端口扫描，直接进入 HTTP 层。
重点导读输出
每条结果记录 URL、状态码、标题、最终跳转 URL、跳转链状态码、技术栈列表、favicon mmh3、Web 服务器标识、内容长度、响应行数与词数、A 记录、CNAME 记录、CDN 名称、JARM 指纹、TLS 与 CSP 头、敏感文件命中列表与 POC 命中列表。结果可打印到标准输出，也可写为 TXT、JSON 或 CSV。
vscan 运行输出
重点导读部署
依赖 Go 1.17 与 libpcap 开发库，go build 即可产出单一可执行文件，Windows 平台会自动关闭彩色输出。官方同时提供各平台发行版二进制。运行时若出现 libpcap.so.0.8 缺失，需按系统实际版本建立软链接。
重点导读扩展
新增 Go POC 时在 pocs_go 下按组件建目录编写校验函数，返回布尔或结构化结果，再回到 pocs_go/go_poc_check.go 的技术栈分支中挂载调用。新增 YAML POC 时参照 pocs_yml/xrayFiles 中的写法，文件名以对应指纹名开头。指纹规则的调整入口是 pkg/fingerprint/dicts/localFinger.json，凭据与路径字典的调整入口是 brute/dicts。
重点导读边界
弱口令字典规模固定，复杂口令体系需要按业务场景扩充。SYN 扫描依赖特权与 libpcap，非特权环境退化为全连接扫描，速度与准确性均有折损。Nuclei 模板数量较大，命中结果需要人工复核以排除误报。目标侧存在 WAF 封禁时，可先加 -np 参数只产出指纹清单，再对 URL 单独执行 POC 验证，避免整条流程无结果。内网 B 段整段扫描会产生大量端口探测，建议先用存活探测工具筛出存活 IP 再交给 vscan。
重点导读地址
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/veo/vscan

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
