#  PoC 满天飞的十月：PHP2Shell、NetScaler、Atlassian……  
原创 flower
                    flower  flower安全   2026-10-28 01:45  
  
**0****1**  
  
**前言**  
  
Preface  
  
![PHP2Shell：我受影响吗？该怎么修复？](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvZa3wvnzZ69hUwMdq3UWzbGbzuDcDY5hRe9LcGsntwvUelzDiaSv3NL13fOD0oBoOwK6wpDCZrmnmgagWgBhkbDKYA3Vrnu8icGs/640?wx_fmt=png&from=appmsg "")  
  
   
  
 刚刚过去的这个十月，安全圈过得并不太平。支撑全球约 70% 网站的 PHP 被挖出 CVSS 9.2 的"PHP2Shell"漏洞，PoC 满天飞、攻击数百万次；几乎同一时间，Citrix NetScaler、Atlassian、SonicWall、AhsayCBS 的高危漏洞接连爆出，多个已有公开 PoC 甚至已被在野利用。漏洞从披露到被武器化，已经从"数周"压缩到"数小时"。这篇文章，我们聊聊这批漏洞，也聊聊在没有补丁的窗口期里，雷池（SafeLine）专业版能帮你做什么。  
  
**0****2**  
  
**正文：**  
  
CyberSecurity  
  
一、PHP2Shell：一个请求，变出一台 shell  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvZIviaEr0SZ6GK8VDdzEAovJk6fV71SBficUl3X9Wrn94XM55kQXXP4ZichibktVTMFjIk9eEw6u5Eq348b5Xgz5IcQNMWOPAh9ia08/640?wx_fmt=png&from=appmsg "")  
  
2026 年 10 月 21 日，PHP 官方发布紧急安全更新，修复两个核心引擎漏洞：  
- CVE-2026-109618：内存耗尽（OOM）处理缺陷，导致内存破坏。   
  
- CVE-2026-109619：跨请求残留的全局状态，形成释放后重用（UAF）  
  
两者合并代号 PHP2Shell，CVSS v4.0 评分均为 9.2（Critical）。攻击者无需任何认证，发送特制 POST 请求即可触发远程代码执行，或在不需要执行代码的情况下读取进程内存中的密码、Token、API Key。  
  
受影响版本（几乎覆盖所有在线 PHP）：  
<table><tbody><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">分支</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">状态</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">修复版本</span></span></p></td></tr><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">PHP 8.5.x</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">受影响</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">升级到 8.5.12</span></span></p></td></tr><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">PHP 8.4.x</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">受影响</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">升级到 8.4.27</span></span></p></td></tr><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">PHP 8.3.x</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">受影响</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">升级到 8.3.36</span></span></p></td></tr><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">PHP 8.2.x</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">受影响</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">升级到 8.2.35</span></span></p></td></tr><tr><td data-colwidth="160"><p><span leaf=""><span textstyle="" style="font-size: 15px">PHP 8.1 及更早</span></span></p></td><td data-colwidth="170"><p><span leaf=""><span textstyle="" style="font-size: 15px">已停止维护，无官方补丁</span></span></p></td><td data-colwidth="191"><p><span leaf=""><span textstyle="" style="font-size: 15px">隔离或升级分支</span></span></p></td></tr></tbody></table>  
  漏洞代码最早可追溯至 PHP 5.4（2012 年），潜伏超过十年。WordPress、Drupal、Joomla、Magento、Laravel、MediaWiki、phpMyAdmin、Nextcloud 等主流 CMS 和框架全部暴露在风险之下。国家网络安全通报中心、工信部 CNVD、CNCERT、CISA 均已发布预警，荷兰、丹麦、法国、芬兰和加拿大部分政府网站一度紧急下线。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvZZbJybVvQsOjQS7z9o1LYZhFicjlU0AlpqmsWNH33ABsuCupq8hS6ONMxkFiaqfbwrgreFNvTib11HpA1IEEoYMGbWHEP6eQrb0M/640?wx_fmt=png&from=appmsg "")  
  
  更值得警醒的是漏洞的来源，它由 Wiz Research、OpenAI Codex Security、Calif 借助 AI 完成挖掘与利用。这类内存缺陷 2024 年就被审计注意过，当时被认为"利用难度太高、无法武器化"。过去是世界级团队数月的活，现在一个研究员加前沿大模型，一周搞定。Wiz 将其称为"迄今由 AI 协助发现的最严重漏洞"，与 Heartbleed、Log4Shell 并列。  
  
  
二、不止 PHP：十月这批漏洞，个个都有 PoC  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvbNCsnohyPQia59DCWsVDIcFCl6aUxuI3KztcH889JH8r7ePH1AJibKLQqwocaSP3LoCDE9aKStcc5nxQtgHGdLSQEt935vhbJCg/640?wx_fmt=png&from=appmsg "")  
  
PHP2Shell 不是孤例。就在同一时间窗口，还有一批高危漏洞已被公开 PoC 或确认在野利用：  
- Citrix NetScaler（CVE-2026-107406） —— 10 月 9 日 Citrix 催促立即修复的严重漏洞，可导致远程代码执行或拒绝服务。NetScaler 通常部署在企业网络边缘，一旦失陷就是进入内网的跳板。  
  
- Atlassian Data Center（CVE-2026-21589） —— 10 月 8 日 PoC 公开后数小时内就开始被攻击者利用。自建 Data Center 产品的用户是重点目标。这是"PoC 公开即攻击开始"的典型案例：攻击代码一旦公开，连低水平攻击者都能拿去扫全网的脆弱系统。  
  
  
- SonicWall（CVE-2026-102255） —— 10 月 12 日被报出无需认证的 SSRF 漏洞，披露后数天即在野利用。不需要登录就能利用，对暴露在互联网上的设备尤其危险。  
  
- AhsayCBS 备份软件（CVE-2026-105133 / 105134） —— 认证绕过 + 操作系统命令注入，10 月 9 日披露后至今未修复完全，持续被在野利用。备份系统是攻击者的心头好：数据敏感、补丁节奏还慢。  
  
- Cisco —— 10 月 8 日一次性修复 12 个严重漏洞，涵盖未授权访问、信息泄露、提权、RCE。  
  
把这几个案例放在一起，规律非常清晰：  
```
披露 → PoC 公开 → 扫描全网 → 在野利用，这条链已经压缩到"小时"级。留给防守方的补丁窗口，几乎为零。
```  
  
  
三、怎么防：把"真空期"变成"安全期"  
  
  
补丁是根治，但升级永远需要时间，排期、测试、重启、兼容性验证，每一环都是天级的。在这段真空期里，雷池（SafeLine）就是你手里最硬的一张牌。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvaxr7hOWoWibAynMqqDZ8huPDkVPTKo9WvBZicq9p3kvSvMwmN2zHGMKvGXFBN6g89cETcEqNeib4m9HLNKicL1jAN8JrsXd9PA0qo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvbjPeticJFd2BBibDakqzo4GflhxTAib96mYnpjzn59RK2H3iaetg0UxHw8W7ib8rk4steBOicEz8DxjCh1g2AOczAe13hEr40PtT8yY/640?wx_fmt=png&from=appmsg "")  
  
（雷池技术架构）  
```
雷池官网：
https://waf-ce.chaitin.cn/
开源地址：
https://github.com/chaitin/SafeLine
```  
  
雷池环境要求：  
```
操作系统：Linux
CPU 指令架构：x86_64, arm64
CPU 指令架构：x86_64 架构必须支持 ssse3， 推荐支持 avx2指令集
软件依赖：Docker 20.10.14 版本以上
软件依赖：Docker Compose 2.0.0 版本以上
最低资源需求：1 核 CPU / 2 GB 内存 / 5 GB 磁盘
```  
  
可以根据以下命令来查看相关信息  
```
uname -m                                    # 查看指令架构
cat /proc/cpuinfo| grep "processor"         # 查看 CPU 信息
lscpu | grep ssse3                          # 确认 CPU 是否支持 ssse3 指令集
lscpu | grep avx2                          # 确认 CPU 是否支持 avx2 指令集
docker version                              # 查看 Docker 版本
docker compose version                      # 查看 Docker Compose 版本
docker-compose version                      # 查看老版本 docker-compose 版本
free -h                                     # 查看内存信息
df -h                                       # 查看磁盘信息
```  
  
快速安装指令：  
```
bash -c "$(curl -fsSLk https://waf-ce.chaitin.cn/release/latest/manager.sh)"
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvbGDkQh9DA2pG4ia6M8niaAFzibpq796icAnQOic7O58mdDEpG2PG4iaMuV1emmS8ibI5HpKedA29cK7r4H7OBoXicH6ibCV0SHRtwLZ2DM/640?wx_fmt=png&from=appmsg "")  
  
![首页概览](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvZtEL8vHBZJbrOwtbavgRDmFdkDJ0b7icwg1ib30mpufibvfAuWQS41pKTJzNkU10QTTKVaMHckpRoG8uOyicBCiaTZMXUxzBCOY6EQ/640?wx_fmt=png&from=appmsg "")  
  
  
雷池区别于普通的防火墙有以下优势：  
  
  
1. 语义分析引擎  
  
雷池的核心检测能力是智能语义分析，不依赖"已知攻击特征"的规则库，而是对 Payload 深度解码、分析攻击意图。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvaUWkGGwOQub2WOqcQNjgE0R8HntnMveOVUsQ6fiaDRDEmDPk2IVLANHbBkxIyaiaNhASs7JyXnqorkcn0CzECeb8Sz05ZObJHKA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvaibwEfGJnpGMfHHwjplvIzicf72lUmdIrdtX9NwTibDgfmx2LvRmpZbu2soDCvjh2vtSsm7mf3uFUyRgyzKRhh0pmlsKrqxvMQxE/640?wx_fmt=png&from=appmsg "")  
  
这意味着面对 PHP2Shell 这类刚爆发的 0day，即使引擎还没有针对性规则，攻击载荷中的命令执行、WebShell、内存异常触发等意图依然能被识别拦截。Atlassian 漏洞那种"PoC 一公开全网开扫"的场景，恰恰是语义分析引擎最擅长应对的：不管利用姿势怎么变，攻击意图跑不掉。补丁是根治，但升级永远需要时间排期。  
  
  
2. 规则广场：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvZQFKYbfnDdibDGQOpjpPbYZ7VKBLwtR11CCJ5Q61L9jJhkt07F7PQv9Trjc5zLM2aaw8QIb9UAIkw7Qva0SfG2DGericLQ4iatKA/640?wx_fmt=png&from=appmsg "")  
  
官方规则模板，一键导入 针对本次 PHP2Shell，雷池规则广场已提供防护规则模板，可直接导入，也支持在控制台手动配置： 登录雷池控制台，进入 防护配置 → 自定义规则 → 新建规则； 规则名称建议填写 CVE-2026-109618_109619_php2shell，匹配范围选择需要保护的站点或 API； 动作建议先设为观察（仅记录不拦截），确认无误报后再切换为拦截； 导入/启用后，在攻击检测日志中确认规则命中情况。  
  
  
3. 雷池专业版：  
  
不止是挡在前面 作为雷池专业版用户，除了边界拦截，还能获得更完整的纵深能力： 自定义规则 + 观察模式先行：灰度验证、零误报上线，不影响业务； 攻击检测日志与告警：实时掌握针对这批漏洞的扫描与利用尝试，及时发现异常 multipart POST、异常 Cookie、PHP-FPM 异常重启的前兆流量；   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvZFRosoexyJZ6XNqZw7ZthEkMCN6v6ZzELcyNQvpBxBia2sz5ibMXjR16gibJpjmyHoh11sDkiabOmjygUic8IXiacW6EicLDuKVuVtNg/640?wx_fmt=png&from=appmsg "")  
  
威胁情报联动：自动识别并拦截已知恶意扫描源、僵尸网络节点 IP——像 AhsayCBS 这类持续被扫描的暴露面，情报联动能先一步掐掉探测流量；  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHvbBsJbCAia3WOcrBI7ZWorj8AMJkYpvovQNgxy2mrI9TMRGRKIDt4gDKSVgsicz9S5Jg5OyzIR4736v4Aloniaibiay9bWkSpBiagcOQ/640?wx_fmt=png&from=appmsg "")  
  
API 防护与精细化访问控制：收敛暴露面，把不需要对外开放的接口先关掉，SSRF 类漏洞的可打面也随之缩小； Bots 管理：过滤自动化扫描器，PoC 公开后铺天盖地的扫描噪音直接降噪。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CyYbk1vmHva8aibrLrpbIL9qkEhxDWjgCJX1NXmRCNt87sqxlLRo3BlIoBbYCqekHbg259Fxt7p9LGCaONXC6vP7ojKdWmbcphuMibTRakVIQ/640?wx_fmt=png&from=appmsg "")  
  
一句话总结：补丁是根治，雷池是保险。在"不能立刻升级"的现实世界里，WAF 就是你漏洞修复窗口期的安全带。  
  
  
四、现在就该做的清单 立即（今天）：  
   
  
在雷池控制台导入/启用 PHP2Shell 防护规则（先观察、后拦截）； 盘点互联网暴露的 PHP 资产，包括设备内嵌、控制面板、厂商软件自带的 PHP；同时排查 NetScaler、Atlassian 自建产品、SonicWall、备份系统是否在列； 检查日志中是否有异常的 Allowed memory size exhausted、fatal error、worker 崩溃激增。  
  
**0****3**  
  
**写在最后**  
  
Final thoughts  
  
PHP2Shell 会过去，但它揭示的趋势不会：AI 让攻击者的武器化速度从"数月"压缩到"一周"，PoC 公开到全网开扫只隔几个小时，而修复永远需要时间。这道时间差，就是每一位网站管理员必须面对的新常态。 与其在漏洞爆发后手忙脚乱地"拔网线"，不如现在就把边界防线立起来。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvaC9cPJiaYxLicebanrWm5hhJ9MjSiaKT7icEROH3uRCVO9biciae6YMJdYF4YM3VicXmRs9MJUntpAKS52arJbxdWfKEvDEUickrVPCtw/640?wx_fmt=png&from=appmsg "")  
  
  
不让黑客越雷池半步，这一次是字面意思。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CyYbk1vmHvb1A9wYPn6aljS6jMakA1k78zYtE86WK5AQ4RmbAfTlibnKNy1hz88HibiadLS7n4h5ibt3MDTeVH4T0BLLln1NftZm5yDMlCHxP7Y/640?wx_fmt=png&from=appmsg "")  
  
点击  
阅读原文  
，体验雷池专业版  
  
  
