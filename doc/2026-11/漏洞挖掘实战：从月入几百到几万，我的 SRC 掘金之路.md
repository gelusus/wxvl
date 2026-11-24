#  漏洞挖掘实战：从月入几百到几万，我的 SRC 掘金之路  
 沧海讲安全   2026-11-24 09:09  
  
 1. 从月入几百到几万，我的 SRC 掘金之路   
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9SAaDE5zM7mhtHDgYmcXuHhZvLBIuRt6kSicb6olQBjrZxeLAXK3vMpfmpZdiawPpur6UkNeewvoFAD5OItvLiazxd7Zibca75WvTI/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=0 "")  
  
刷 SRC 月度榜单时，总羡慕别人单条漏洞奖金几万，反观自己常常只有几百块入账，仿佛同在一座金矿，别人挖到金块，自己只捡到碎石。  
  
我深耕网安近十年，从安全测试员转型专职漏洞挖掘，也曾长期卡在收益瓶颈。今天和大家聊聊 SRC 挖洞：它从来不是能用扫描器就能做好，高手和普通挖洞者的差距，往往不在技术本身，而是  
   
思路、流程、方法和持续迭代的能力   
。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/CsKJlMFPH9SSmXE06Bm2ZUb85HpBVQ1ibia6osJK5n9ibIJBt5yW5sfqANptTWMTlUyVk7ibv6f5M8xmyRxWGgzB9UvXTCdQCqn34unhjryIAIc/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=1 "")  
  
文中会分享我多年踩坑总结的实战经验、提效挖洞技巧。不管是刚入门的新手，还是陷入瓶颈的老研究员，都能找到突破方向。  
  
##  2. 核心差距解析：思维、流程与信息差 2.1 思维模式：猎人思维，而非游客式扫洞不少人挖洞等于开扫描器盲扫，靠 AWVS、Xray 批量跑目标，耗费大量时间核验一堆误报、低危漏洞，提交的漏洞价值很低，这就是游客模式—— 漫无目的，全靠运气。收益稳定的挖手，用的是猎人模式：主动侦察、精准攻坚，不依赖自动化泛扫。  
  
拿到目标后优先做这些事：  
  
|   
资产梳理测绘：  
  
借助 Fofa、Quake 等测绘引擎搜集资产，不止收集子域名、IP 段，还要研判业务关联。遇到后台域名，先判断是否为真实管理入口、和主站账号体系是否打通，参考同厂商历史资产特征。  
  
|   
架构与指纹识别：  
  
依靠指纹工具识别框架、中间件、前端技术栈，提前预判漏洞方向。比如 Spring Boot 带 Actuator 重点排查信息泄露与 RCE；Vue+Node 项目重点盯 API 未授权、JWT 缺陷。  
  
|   
吃透业务逻辑：  
  
这是挖到高价值逻辑漏洞的核心。像真实用户走完注册、下单、支付、退款全链路，不断思考功能可以如何被滥用。扫描器很难识别这类问题。  
> 我之前挖过一个电商漏洞，没有上来就扫描，而是花两天模拟十余种购物场景，研究优惠券核销规则，最后发现合并支付时优惠券可无限复用，拿下高危漏洞，奖金远超扫一个月低危漏洞。  
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9QX0NoAUtkCib6K2DxicC4LzjlKAAsme6uKiaEBZC39ZbOqJicKNv6mNhmntSqdD9SQlAt6DekaibKPddorMTNWCwHLEp2W31CKSNeI/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=3 "")  
  
  
  
## 2.2 工作流程：系统化闭环，而非随机碰运气低收益挖洞大多随性而为：哪里有活动扫哪里、上新厂商碰运气，缺少沉淀与复盘。  
  
  
高效猎人拥有标准化闭环流程：**侦察 → 武器化 → 测试 → 报告 → 复盘**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9S3aSBOwZxw9ssIJApy4BDyNfDTibBCc77RRhvjKmic70qEpNVMhSj06ibGbXQOCnchLSn5SvjQlFc1z3SVnbnIu8wo38bvIUmB5k/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=5 "")  
**| 侦察情报阶段**  
优先筛选高价值 SRC，关注攻防活动奖励倾斜、厂商业务类型、过往漏洞记录，判断安全水位；  
自动化搜集子域后人工筛选维护资产清单，标注业务归属、技术栈、敏感功能点、历史漏洞；  
同时研读同厂商、同类业务公开漏洞，总结通用漏洞模式。  
**| 武器化测试阶段**  
打造专属工具链，不局限现成图形工具：  
- 写 Python 脚本做业务专项测试、Golang 自研 Fuzz 工具、定制 Nuclei 检测模板；  
  
- 搭建个人测试用例库，整理越权、上传绕过、SSRF 等场景清单，测试时对照核验，避免遗漏。  
  
**| 报告复盘阶段**  
漏洞报告直接影响评级与奖金。  
- 优质报告清晰展示复现链路、原理、危害、修复方案，降低审核成本；  
  
- 漏洞无论是否收录都要复盘，提炼可复用的测试思路，持续优化工具。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/CsKJlMFPH9QjdWhOKUjpz9Fo76uyCYibXt40RgoRaXfobhcRj1ibj7ibBa71OlkoCN4ueibclI789MgXTQZBibuANP2mTBX43CgTZibauscK5WK84/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=6 "")  
  
  
##  3. 实战技能精讲：四大核心方向 3.1 信息收集：资产的广度深度，决定收益上限新学很多人只搜集子域名，远远不够。  
**| 资产拓展：**  
  
**利用 DNS 历史记录、crt.sh 证书日志、JS 外链挖掘历史资产；**  
  
**全端口探测，不要只盯着 80/443，8080、6379、9200 等端口经常藏着未授权风险；**  
  
**追踪企业并购、子公司信息，收购遗留系统往往防护薄弱。******  
  
**| 架构源码分析：**  
  
**自建指纹库，快速匹配带已知漏洞的组件；**  
  
**留意.git、备份压缩包等泄露，拿到源码能快速定位隐藏接口与硬编码密钥；**  
  
**爬取前后端分离项目的 API 接口，是越权、未授权漏洞高发区。**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9Rvl6hqtQ95prCS79qvyKr48hronXC8z1qAe1PeXDRaDt0RspYcWkrBnfwtcrdiaHL8LHWqoS3EXxYoibTOnichqTm5Hrh7V87GAs/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=8 "")  
  
**3.2 手工挖洞：扫描器为辅，手工测试才是核心业务逻辑漏洞（溢价最高，只能手工）**  
|   
业务逻辑漏洞  
  
越权  
：分水平越权、垂直越权。抓包修改 ID、订单号等标识，同时测试查询、修改、删除接口。  
  
流程绕过  
：跳过业务步骤、重复提交请求、逆向调用后端接口。  
  
竞争条件  
：借助 Turbo Intruder 或多线程脚本并发请求，测试余额、优惠券、库存等资源时序问题。  
#### | 常规 Web 漏洞深度测试  
  
SQL 注入  
：不靠 sqlmap 默认扫描，掌握注释混淆、编码变换、等价语句替换等 WAF 绕过技巧，优化盲注请求量。  
  
XSS：  
不止测试基础弹窗，关注存储型漏洞利用链、DOM 型 XSS 风险点，熟悉各类编码与标签绕过方式。  
  
SSRF  
：多协议测试，掌握 DNS 重绑定、地址解析差异等绕过手法。  
  
文件上传  
：绕过前后端双重校验，利用后缀变异、解析漏洞、图片马等方式测试。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9TXpnKZx2qxM0pv0Ev3QummNpHhf5scx7hOUpOA95LiagAclMnHHYeplIC4yE2C2qDknEgebm39ecDNcFs0YpsxE92OEFV7AyWg/640?wx_fmt=jpeg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=10 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/icS6QYnhpJckHlIj5d2BibCpyUibsiaIrSyMpiaAMNTNVtSeaGekkbMzokxl3zuzasdFupV8icJiaofSa5GLsVuV7pkCw/640?wx_fmt=gif&tp=wxpic&wxfrom=5&wx_lazy=1#imgIndex=11 "")  
  
3.3 定制工具链，提升挖掘效率  
  
工具贵在适配自己的流程，而非堆砌软件。  
  
我自用方案：  
  
① 信息收集自动化脚本  
 ：我用   
Python  
 写了一个脚本，输入一个主域名，自动调用 subfinder   
,  amass   
,  assetfinder   
 收集子域，然后用   
httpx  
 探测存活，接着用   
nuclei  
 跑一遍基础指纹和漏洞检测，最后将结果整理成一个清晰的   
HTML  
 报告。整个过程一键完成，解放双手。  
  
② 漏洞Fuzz专用字典 ：  
我维护着几个字典文件：  
  
目录字典   
：融合了 dirsearch   
,  dirb  
 的大字典，并加入了根据目标行业（如教育、金融）特点自定义的路径（如/admin/edu/   
,  /financial/upload/   
）。  
  
参数字典   
：除了常见参数（ id   
,  page   
,  file   
 ），还收集了各种API接口常见的参数名（如 access_token   
, userId   
, orderNo  
 ）。  
  
Payload字典   
：针对不同漏洞类型、不同WAF的测试Payload，分类存放，随时调用。  
  
③ Burp Suite的深度定制  
 ：   
Burp   
不仅是抓包工具，更是测试平台。  
  
插件生态   
：   
Autorize  
 （自动越权测试）、   
Turbo Intruder  
（高性能并发攻击）、   
Collaborator  Everywhere  
 （自动发现SSRF/Out-of-band漏洞）、   
Logger++   
 （增强日志记录）。  
  
项目级配置   
：为每个重要目标创建独立的Burp项目文件，保存所有的请求历史、爬虫站点地图、自定义的扫描配置和插件设置。方便下次继续深入测试。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/icS6QYnhpJckHlIj5d2BibCpyUibsiaIrSyMpiaAMNTNVtSeaGekkbMzokxl3zuzasdFupV8icJiaofSa5GLsVuV7pkCw/640?wx_fmt=gif&wxfrom=13&tp=wxpic#imgIndex=12 "")  
  
3.4 漏洞报告：做好呈现，拉高评级  
  
一份高分报告包含：明确标题、自评等级与理由、目标 URL、原始请求包、分步复现截图、漏洞危害证明、原理分析、可落地修复建议。  
> 避坑提醒：  
只证明漏洞存在远远不够，要展示完整攻击链与实际危害。早年我提交存储 XSS 仅附 alert 截图，被定为低危；补充窃取 Cookie 登录他人账号的演示之后，直接升级高危，奖金大幅上涨。  
  
  
SRC 挖洞没有捷径，从月入几百到稳定高额收益，是思维、技术、习惯、耐心的综合比拼。从被动跑扫描器，转变为主动发现风险，打破固有思维定式，你才能看见别人发现不了的漏洞。  
  
「最后」如果你真的想学好一门本事，首先就要考虑自己对这门技术的兴趣，没有天赋还能靠时间和努力去弥补，但如果没有兴趣加持，就很难坚持到最后。你要是正打算尝试网安或者想努力一次，我把这些年用过的视频教程和学习笔记都梳理出来了，现在都无偿分享给大家，需要的找我拿就行（文末自取）。现在哪个行业都不好走，如果没有学历也没有天赋，那就只有努力和坚持了，请相信相信的力量，共勉！  
  
**沧海专属黑客/网络攻防技术资料**  
  
  
  
@沧海讲安全：  
在安全圈待了十多年，已经积累了很多的技术教程，在计算机这个行业，如果不会主动学习，手里没点学习资料，注定是走不远的。我整理的这些资料包含了市场上主流的攻防技术，不说让你成为黑客大佬，帮助你从0到进阶网络安全技术问题不大。  
  
  
**平台铭感，拿资料、学技术看⬇（无偿共享）**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCW28TYVbicW1icR88lb2fLYfLS6ib2Mfic96c3gX0VBFarDLjM2sjicYFE6SVtcyF5DHLPwyUgE4lyzDxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
**部分技术资料预览**  
  
  
**01**  
  
**视频教程**  
  
  
从0到进阶主流攻防技术视频教程（包含红蓝对抗、CTF、HW等技术点）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcYaRKqWc1cxP8sBrX6KZasFTJEVibWmdyoGAuRO4AbzaVjUJ8guoWAzQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcCP8oaOCQm8Cp2qhpCxWiaOjzYrOoA1iac5eSafBicPxSQcpYtchyfVvxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****2**  
  
**书籍Pdf**  
  
入门必看攻防技术书籍pdf（书面上的技术书籍确实太多了，这些是我精选出来的）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcUumOTUmUznuo7MzKl1JiaEQIeSh4ibkO6jxY68zVZz7iayrwGRtGu2bHw/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****3**  
  
安装包/源码  
  
  
主要攻防会涉及到的工具安装包和项目源码（防止你看到这连基础的工具都还没有）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcvsT9h4B1hS9VEPengMcOtNL24949kb4cibKLS9HkIb1k2htW8GYqzMQ/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****4**  
  
**面试试题/经验**  
  
  
网络安全岗位面试经验总结（谁学技术不是为了赚$呢，找个好的岗位很重要）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6Kqcm6J0eAql29R6DIM8bJW4rweVBicM8ibGMOmLNFTpdcQ0gFvefMTOg9dA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
**平台铭感，拿资料、学技术看⬇（无偿共享）**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCW28TYVbicW1icR88lb2fLYfLS6ib2Mfic96c3gX0VBFarDLjM2sjicYFE6SVtcyF5DHLPwyUgE4lyzDxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
@沧海讲安全：  
只要你是真心想学黑客/网络安全技术，我这份资料就可以无偿共享给你学习，但是想学技术去乱搞的人别来找我，目前全球网络环境日益紧张，我国在这方面的相关人才比较紧缺，网络安全行业确实也需要更多的有志之士加入进来，我也真心希望帮助大家学好这门技术，如果日后有啥学习上的问题，欢迎找我交流。  
  
