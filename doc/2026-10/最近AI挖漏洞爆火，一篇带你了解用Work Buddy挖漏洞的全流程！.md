#  最近AI挖漏洞爆火，一篇带你了解用Work Buddy挖漏洞的全流程！  
 沧海讲安全   2026-10-22 09:28  
  
### 最近WorkBuddy火了，就很多人问我：用 WorkBuddy 挖漏洞，到底能不能赚到钱？  
  
直接说：**能赚，但要走合规路线。**  
  
在补天、漏洞盒子、各厂商 SRC 平台授权范围内，用 WorkBuddy 做资产梳理、接口抓取、基础漏洞初筛、报告生成，再人工复核提交，确实是一条可以跑通的副业路径。  
  
**01**  
  
**先搞清楚：WorkBuddy 到底是什么？**  
  
  
  
WorkBuddy 是腾讯出品的   
全场景 AI 智能体桌面工作台  
，2026 年 3 月上线，和 CodeBuddy 同属一个 AI 工具矩阵。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9Qo3KibDfoH8xxFs7FJoicQbBpKu72bqicZl8JvhQKQcuPxAcFlyMtRUDz3PDvdFFcjmcNBv3chJJcAlHD6N56aGIVB0iaQAfJqN3c/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
> 马化腾将 WorkBuddy 定位为腾讯内部的“超级项目”，是继 QQ、微信之后的第三个战略级产品，重点打造 AI 原生办公智能体。‌‌‌  
  
  
简单说，它不是普通的聊天机器人，而是能真正“替你在电脑上干活”的 AI 助手。  
  
你给它一句话指令，它能自己拆解任务、调用工具、读写本地文件、生成文档和表格，最后交付一个可以直接验收的结果。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9RNB39w9JWDn9mSJUHwtOIPRWOw1MXxENxiaNKxibt2j8DWFyfHrIKGVwySvxYUibt2EK90XWjpfgNwoM0tbxheaNrvP1JWx5pCtw/640?wx_fmt=jpeg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "")  
  
而它的核心能力很适合拿来辅助挖洞：  
  
• **自然语言下任务**  
：不用写复杂脚本，说清楚目标就行；  
  
• **自主规划执行**  
：自动拆步骤、调用工具、持续执行；  
  
• **本地文件操作**  
：在授权目录里读取、整理、生成结果文件；  
  
• **多 Agent 并行**  
：可以同时跑资产探测、接口抓取、报告整理；  
  
• **技能扩展**  
：支持安装 Skill、接入外部工具和数据源；  
  
• **远程操控**  
：通过微信、企业微信、QQ、飞书、钉钉也能远程指挥电脑执行任务  
  
放到漏洞挖掘场景里，它就像一个会调用安全工具的 **AI 安全助手**  
。  
  
以前一个人要反复翻资产、抓接口、整理参数、写报告，现在 AI 能先把大量重复工作跑一遍，你再把精力放在最值钱的环节：**判断漏洞真伪、复现危害、提升提交质量**  
。  
  
**02**  
  
**为什么今年 AI 挖洞突然火了？**  
  
  
  
2026 年，AI 辅助漏洞挖掘已经不是噱头。  
  
360 漏洞挖掘智能体已累计挖掘漏洞 3432 个，其中 105 个经监管确认；ISC.AI 2026 上也提到，智能体让高价值漏洞挖掘时间从数月缩短到数小时。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9RP2avkiarjcEia7I64AEibWvNsQ80ibddsuuU3BDpIWpwA91NWiahiaibPMf79RraL7frlpGsjoQMnuWeTQk9G68nLoicicnWpuFswapbg/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "")  
  
国家网络安全宣传周期间，相关部门也在推动“网络系统及源代码漏洞智能挖掘”测试，说明 AI 赋能漏洞挖掘正在进入真实系统验证阶段。  
  
- 对普通人来说，门槛也变低了。  
  
WorkBuddy 支持自然语言下任务，能自动拆解、调用工具、交付结果。  
  
它真正的价值不是“替你赚钱”，而是**把你从重复劳动里解放出来**  
，让你有更多时间去判断漏洞真伪、复现危害、提升提交质量。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/CsKJlMFPH9SjBmSpPdnzcqYWMUyHqibmpPBNkpibseHh9ibMMeibA9AEG6xwsgfJHKR5YlriaiaqyEQD10CL1Qm4ddsXX5JzMPStddMa7VomHicQW4/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3 "")  
  
**03**  
  
**WorkBuddy 能帮你做哪些事？**  
  
  
  
|   
资产探测  
：域名、子域名、端口、服务指纹；  
  
**| 接口抓取**  
：API、参数、请求包、响应包；  
  
**| 基础漏洞初筛**  
：未授权、弱口令、信息泄露、基础越权；  
  
**| 报告生成**  
：复现步骤、请求响应、危害描述、修复建议。  
  
但它的边界也很清楚：  
**复杂业务逻辑、金额篡改、权限绕过、深层越权，AI 很难稳定挖出来。**  
  
这些仍然需要你有人工经验来判断。  
  
**04**  
  
**一套能落地的挖洞赚钱流程**  
  
  
  
1）注册合规 SRC 平台  
  
比如补天、漏洞盒子，只测试平台授权资产。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9T9bM7kYwXraVHXc2YtkO10wU9PCsEoHicZMH6rMCtNiab28of1ftUzqfOclic3yt7u5J4XLOoicUicflqibIGvVn1UVS8nnBhibLLPIE/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "")  
  
**2）安装 WorkBuddy 和挖洞 Skill**  
  
加载资产探测、接口抓取、漏洞初筛、报告生成等能力。  
  
**3）让 AI 自动跑任务**  
  
资产梳理 → 接口提取 → 参数遍历 → 基础漏洞筛查。  
  
**4）人工复核，确认有效漏洞**  
  
剔除误报、重复、无效结果，重新复现核心问题。  
  
**5）生成报告并提交**  
  
让 WorkBuddy 辅助写报告初稿，人工补充业务背景和危害证明。  
  
**6）审核通过后领取赏金**  
  
低危、中危、高危赏金不同，先从简单漏洞练手更稳。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9Qn0FekiaouEbxD3MpLP43TUialuFJ8FmVYWzw8rOEgzgpmG6R85qtUohTepradWT2NQMnzvcCFvrTGgibiaEcXqm4pajk3TicQnDRI/640?wx_fmt=jpeg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5 "")  
### 如果你零基础、想转行网络安全，或者想给自己多一条副业收入，建议别只追工具。  
  
**05**  
  
**挖洞前的基础课**  
  
  
  
不然就会：工具会点，原理不懂；报告能写，危害说不清；漏洞能扫，边界守不住。  
这就是典型的“  
AI 小子  
”。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9SUW5sjtEuKMtMtMz0PI4PWHFcvic7Aq0p6C6sQYfhQ93C87GCmwoPtoka1tTib5BCKhrqLPkLs8gKGLGBEBd4kxxkHvSl0zBNxM/640?wx_fmt=jpeg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "")  
  
  
所以真正能落地的挖洞能力，至少要补这几块：  
  
1） 法律与授权边界  
  
先搞懂什么能测、什么不能测。必须只测 SRC 或企业明确授权的范围，不扫未授权站点，不下载敏感数据，不做破坏性测试。  
合规不是“附加项”，是白帽的第一条底线。  
  
2） 网络与 HTTP 基础  
  
挖洞本质是在看请求和响应。要懂 GET/POST、URL 参数、Cookie/Session/Token、状态码、请求头、前后端交互流程。很多越权、密码重置、接口篡改，都是从这里看出来的。  
  
3） Web 前端与后端基础  
  
不用一开始就成开发，但要能看懂页面结构、接口调用、参数传递、数据库查询。  
  
至少了解 HTML/JS、简单 SQL、常见后端逻辑。否则 AI 给你一堆结果，  
你很难判断它是不是真漏洞  
。  
  
4） 核心漏洞原理  
  
不要只背名字，要理解“它为什么成立”。  
  
重点从这些开始：  
SQL 注入  
：输入被当成数据库语句执行  
  
XSS  
：恶意脚本在浏览器里执行  
  
文件上传/包含  
：服务器错误处理用户文件  
  
CSRF  
：伪造用户身份发起请求  
  
越权访问  
：改个 ID 就能看到别人的数据  
  
密码重置/支付/验证码逻辑漏洞  
  
信息泄露/未授权访问/弱口令  
  
5） 工具使用，但不依赖工具  
  
Burp Suite、浏览器开发者工具、Nmap、Wappalyzer 这些要会用，但必须先手工理解请求，再让工具辅助。AI 和扫描器适合做“初筛”，不适合替你做最终判断。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9TM9UDfgE4MA1zs0J5hvPdzBbnJicwU2NsSbXO4Yu0YSZLDcRe387eMcFblicDpe3AAL4RokK18jibUugagiaj5GLPq7BIQibzrkde8/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7 "")  
  
6） 靶场实操  
  
别直接拿真实网站练手。先在 DVWA、Pikachu、SQLi-Labs、Upload-Labs 这类靶场里复现漏洞，形成“看到参数就想到测试点”的直觉。  
  
7） 报告与危害证明能力  
  
挖到漏洞不等于能变现。你要能说清楚：漏洞在哪、怎么复现、危害多大、影响哪些数据、怎么修复。AI 可以帮忙写初稿，但业务背景、复现细节、危害判断必须你自己把关。  
  
8） 复盘思维  
  
每次提交后都要问自己：这个漏洞为什么存在？AI 哪里误报了？我下次能不能更快定位？真正拉开差距的，不是谁跑得快，而是谁能持续沉淀自己的挖洞方法论。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/CsKJlMFPH9RDfAwz9sKDl1hwLWsFojqQt2hB7xtu8TLtkPcmV8vkyjhbVoGMvmg0HCpCDAVscQWDCfcIUmbYM0dEfFf2cyTgB24f6kh7ILo/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8 "")  
### 先把 Web 安全、渗透测试、SRC 规范学扎实，再用 WorkBuddy 提升效率，这样变现路径才更稳。  
  
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
  
