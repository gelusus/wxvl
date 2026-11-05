#  AI挖漏洞爆火，一篇带你了解用Work Buddy挖漏洞的全流程！  
海哥网络安全
                    海哥网络安全  海哥网络安全   2026-11-05 09:55  
  
最近后台被同一类问题刷屏了：  
  
"都用上AI了，那挖漏洞这事儿，它能不能替我干？顺带把副业收入也解决了？"  
  
先把结论放在最前面：**能变现，但只有合规这一条路能走通。**  
  
在补天、漏洞盒子以及各家厂商 SRC 平台的授权范围内，让 WorkBuddy 去干资产梳理、接口抓取、漏洞初筛、报告初稿这些"体力活"，再由人工复核后提交——这条路，已经有人实打实跑通了。🛤️  
  
01  
先弄明白：WorkBuddy 究竟是个啥？  
  
WorkBuddy 是腾讯推出的全场景 AI 智能体桌面工作台，2026 年 3 月上线，和 CodeBuddy 同属一个 AI 工具矩阵。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9TpEFHq3OZdKZMMp4K3z37iawT7L8UvxpMUYlrWMP5U5prZAMGBKT1hU30YK4ERicsprp7QSStAgBsZPe6hYVnlycjjib2O5eqJ0M/640?wx_fmt=other&from=appmsg "")  
  
> 马化腾把它称作腾讯内部的"超级项目"——继 QQ、微信之后的第三个战略级产品，主打 AI 原生办公智能体。  
  
  
一句话：**它不是聊天机器人，是真正能"在你电脑上干活"的 AI 助手。**  
  
你只需要用一句话说清目标，剩下的交给它：自己拆解任务、调用工具、读写本地文件、产出文档表格，最后交付一个能直接验收的结果。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9S46xp8TVvu69FgmYibT6msffZlGDwFf1gHZ0Mvw6TBBOagfMf1ABJ9g55xia2bFqujXZ9Fn7CdpvjJ2VPIAzdT5AWfan8L8F4t0/640?wx_fmt=jpeg "")  
  
把这些能力平移到安全场景，它更像一个**随叫随到的 AI 安全搭子**  
：🔧  
  
🔹 **自然语言派活**  
：不用写复杂脚本，讲清楚需求就行；  
  
🔹 **自主规划执行**  
：自动拆步骤、调工具、持续推进；  
  
🔹 **本地文件操作**  
：在授权目录里读取、整理、输出结果；  
  
🔹 **多 Agent 并行**  
：资产探测、接口抓取、报告整理可以同时跑；  
  
🔹 **技能扩展**  
：支持安装 Skill，接入外部工具和数据源；  
  
🔹 **远程指挥**  
：微信、企业微信、QQ、飞书、钉钉都能远程下指令。  
  
02  
AI 挖洞，为什么偏偏今年火出圈？  
  
因为成绩单已经摆上台面了——2026 年，AI 辅助漏洞挖掘早就不是噱头。📊 360 的漏洞挖掘智能体已累计挖出漏洞 **3432**  
 个，其中 **105**  
 个通过监管确认；ISC.AI 2026 大会上也给出判断：智能体把高价值漏洞的挖掘周期，从"数月"压缩到了"数小时"。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9QwfI4FZdBfdKIObUgOT7IibyuhhXRq7vYfKy4ZktrnsIqMyn99XJ1Z4UM3ibiatOyd1tojcKYtuqpsLBBG2Hv2qekkIX4d4QzYQQ/640?wx_fmt=jpeg&from=appmsg "")  
  
国家网络安全宣传周期间，相关部门也在推动"网络系统及源代码漏洞智能挖掘"测试——AI 挖洞已经不是概念，而是进入了真实系统的验证阶段。  
  
对普通人来说，门槛同样在塌陷。WorkBuddy 支持自然语言派活，能自动拆解、调用工具、交付结果。  
  
但它真正的价值从来不是"替你赚钱"，而是**把你从重复劳动里捞出来**  
，让你把省下的时间花在刀刃上：判断漏洞真伪、复现危害、提升提交质量。💰  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CsKJlMFPH9SKILXu9ZljFWmicKnAKx7diby9WicmPF56gQyIjxXEyPXgbBWZLnKJdQ5FKsKK15Xu7bEXh2COKuZuRftMejEfciabDl5l9r9vo9s/640?wx_fmt=png&from=appmsg "")  
  
03  
它到底能帮你干哪些活？  
  
✅ **资产探测**  
：域名、子域名、端口、服务指纹；  
  
✅ **接口抓取**  
：API、参数、请求包、响应包；  
  
✅ **漏洞初筛**  
：未授权、弱口令、信息泄露、基础越权；  
  
✅ **报告生成**  
：复现步骤、请求响应、危害描述、修复建议。  
  
但边界同样要心里有数：⚠️  
  
**复杂业务逻辑、金额篡改、权限绕过、深层越权——这些 AI 目前很难稳定挖出来**  
，仍然要靠人的经验去判断。  
  
04  
一套能落地的挖洞变现流程  
  
**第 1 步｜注册合规 SRC 平台**  
比如补天、漏洞盒子，记住一条铁律：只碰授权资产。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/CsKJlMFPH9STxibmnAX5upLWzOZ6NGRfXKO9xkaodfmymVNzmbW2diaxnVvTdkhPL0V9qbiaTvjzpd3QKQ0pexevrI5EeKgbbqqGzQkZKjWU6E/640?wx_fmt=jpeg&from=appmsg "")  
  
**第 2 步｜安装 WorkBuddy 和挖洞 Skill**  
加载资产探测、接口抓取、漏洞初筛、报告生成等能力。  
  
**第 3 步｜让 AI 自动跑任务**  
资产梳理 → 接口提取 → 参数遍历 → 基础漏洞筛查。  
  
**第 4 步｜人工复核，确认有效漏洞**  
剔误报、去重复，重新复现核心问题。  
  
**第 5 步｜生成报告并提交**  
AI 出初稿，人工补充业务背景和危害证明。  
  
**第 6 步｜审核通过，领取赏金**  
低危、中危、高危赏金不同，建议先从简单漏洞练手，更稳。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/UlibJFdibia1g7B4srRE1fPs1OJMjaibH7cqg8WqW4vIdpQQrWKgcqHnib8BXr3bmgiarQd02ZibiaWJYpqM7Q22Tu3V8UvoRFIPQnpmQo6bREl6nmk/640?wx_fmt=jpeg "")  
  
一句话送给零基础想转行、或想多一条副业收入的朋友：**别只盯着工具。**  
🧭  
  
05  
挖洞之前，先把这门基础课补上  
  
否则很容易变成"AI 巨婴"：工具会点，原理不懂；报告能写，危害讲不清；洞能扫出来，边界守不住。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9SziaJjpicAOJZnBO6eBGXaEYoQJ3nj5EJL1cm2aI2eYOIP7wqMhku30icOm8YXEzZh1MYpYk5BE5mMPSrCGdGPgBnu5zRZ6cJfWM/640?wx_fmt=jpeg "")  
  
真正能落地的挖洞能力，至少要补齐下面 8 块——  
  
**1）法律与授权边界**  
 ⚖️  
  
什么能测、什么不能测，必须刻在脑子里：只测 SRC 或企业明确授权的范围，不扫未授权站点，不下载敏感数据，不做破坏性测试。合规不是可选项，是白帽的第一条底线。  
  
**2）网络与 HTTP 基础**  
  
挖洞的本质是读请求、读响应。GET/POST、URL 参数、Cookie/Session/Token、状态码、请求头、前后端交互流程，都得吃透。很多越权、密码重置、接口篡改问题，都是从这里看出来的。  
  
**3）Web 前端与后端基础**  
  
不用一上来就成开发，但要看得懂页面结构、接口调用、参数传递、数据库查询；HTML/JS、基础 SQL、常见后端逻辑至少混个脸熟。否则 AI 甩给你一堆结果，你分不清哪个是真洞。  
  
**4）核心漏洞原理**  
  
别背名字，要懂"它为什么成立"：👇  
  
▪ SQL 注入：输入被当成数据库语句执行；  
  
▪ XSS：恶意脚本在浏览器里执行；  
  
▪ 文件上传/包含：服务器错误处理了用户文件；  
  
▪ CSRF：伪造用户身份发起请求；  
  
▪ 越权访问：改个 ID 就能看到别人的数据；  
  
▪ 以及密码重置、支付、验证码逻辑漏洞，信息泄露、未授权访问、弱口令等。  
  
**5）会用工具，但不迷信工具**  
  
Burp Suite、浏览器开发者工具、Nmap、Wappalyzer 要熟练，但顺序不能反：先手工理解请求，再让工具辅助。AI 和扫描器适合做初筛，最终判断必须是人。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9QxSoweiaqd3yDHSEX8OLwNZ7K4vDqB8FAvBKSd4kpKmBSIJImhppqM1m9t8aOwkz0C135ickqEJQBL6ZINwx0OGdp6W65IXTfb8/640?wx_fmt=other&from=appmsg "")  
  
**6）靶场实操**  
  
千万别拿真实网站练手。先在 DVWA、Pikachu、SQLi-Labs、Upload-Labs 这类靶场里反复复现，练出"看到参数就想到测试点"的肌肉记忆。🎯  
  
**7）报告与危害证明能力**  
  
挖到洞 ≠ 能变现。你要能讲清楚：洞在哪、怎么复现、危害多大、影响哪些数据、怎么修。AI 可以写初稿，但业务背景、复现细节、危害判断，必须自己把关。  
  
**8）复盘思维**  
  
每次提交后多问自己三个问题：这个洞为什么存在？AI 哪里误报了？下次能不能更快定位？真正拉开差距的从来不是手速，而是**能不能持续沉淀自己的挖洞方法论**  
。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9RxfmHfVfR85wq9yZUq1cDic6BZcnvhDdNCZjzvMVAYNxtuBbojuoqzq7XLQxySo3H9T7FLibAxBXxjSU1FrO9ibqGLEEGTuQQxkc/640?wx_fmt=png&from=appmsg "")  
  
先把 Web 安全、渗透测试、SRC 规范学扎实，再让 WorkBuddy 帮你提效——变现路径才走得稳。  
  
写在最后  
  
真想学好一门手艺，先掂量一下自己对它的兴趣。天赋不够，时间和努力还能补；但没有兴趣托底，很难撑到出结果的那天。  
  
如果你正打算入坑网安，或者想再搏一次，我这些年攒的视频课和学习笔记也全都整理好了，现在免费分享，需要的文末自取。  
  
现在哪个行业都不轻松。没学历、没天赋，那就拼努力和坚持，剩下的交给相信的力量。共勉！  
  
想看更多学习思路？我主页的历史文章里有几篇入门干货：零基础转行网安、渗透测试入门教程、自学路线规划，扫下方的二维码就能拿。  
  
  
**黑客/网络安全学习包**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/TiaI8Dth4IiaRCFva2ZibMZKuNBEDOAEmkUGiakynth3MRTicLcHaV4MAvjubiaIicUx4ZrMxuSdSicjzT5HfEAzJy782g/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=2 "")  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/TiaI8Dth4IiaRCFva2ZibMZKuNBEDOAEmkU7VZiaRU6vdoIQC9ToNyrFNvkWmp92gn3R2RWyGVEiaxjTlDjic3dPsW6g/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=3 "")  
  
**资料目录**  
1. 成长路线图&学习规划  
  
1. 配套  
视频教程  
  
1. SRC&黑客文籍  
  
1. 护网行动资料  
  
1. 黑客必读书单  
  
1. 面试题合集  
  
  
  
**282G**  
《**网络安全/黑客技术入门学习大礼包**  
》，可以**扫描下方二维码免费领取**  
！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/UlibJFdibia1g4d5GFIWq9ypg0WGrTtgH8Y7MlLyGO27qMDNPwnoMHVB7hLJXxRZt5mapa4vo5eOibLvibpJXOXyDlPQGPuA1tfK4KBdG1eBt36Q/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
1.成长路线图&学习规划  
  
要学习一门新的技术，作为新手一定要**先学习成长路线图**  
，**方向不对，努力白费**  
。  
  
对于从来没有接触过网络安全的同学，我们帮你准备了详细的学习成长路线图&学习规划。可以说是最科学最系统的学习路线，大家跟着这个大的方向学习准没问题。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/UlibJFdibia1g5cyo7EEuXLkIqP8HyFpIWWWQicnKwZTib4aIJe98EwnBoo6LvQDUKTFQ85LLaxhhDicmibv2LhCibwjrGWqiaczhHg9oFooCK4LZnpk/640?wx_fmt=jpeg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=3 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/evTLxnBbHv6fa8BCJ5052WLSGZjTIfEDgymVV6FeniaFszgpka15xzMolFmtXDdiaaDJMwXSqTQgRgBicvbYv4tNw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=6 "")  
  
2.视频教程  
  
很多朋友都不喜欢  
**晦涩的文字**  
，我也为大家准备了视频教程，其中一共有  
**21个章节**  
，每个章节都是  
**当前板块的精华浓缩**  
。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dnMVja8hzZpia0AkKu6AWrQn8E3Yp6lXhRo3D1Bttpiao3a0poRH29MC1MBC0hk5gKMCiaicy3wOiaUviag/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=7 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dnMVja8hzZpia0AkKu6AWrQnHBEMEd0W8dr6zFFQetPOhwiax5u8YYm0YZtWJSmyJ7d85QmuVQEicLVQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=8 "")  
  
3.SRC&黑客文籍  
  
大家最喜欢也是最关心的  
**SRC技术文籍&黑客技术**  
也有收录  
  
**SRC技术文籍：**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dkY8ctWgyFKc2oWZY3ibCDm5lMpjofvtGCicHTLibsOF8b841UOfozGsdjDvJKiaFgibdTunKlgC9kzrTQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=9 "")  
  
  
**黑客资料由于是敏感资源，这里不能直接展示哦！**  
  
  
4.护网行动资料  
  
  
其中关于  
**HW护网行动，也准备了对应的资料，这些内容可相当于比赛的金手指！**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dnMVja8hzZpia0AkKu6AWrQnaPKJSI9dNKiaR4vaJf0hqApKNbJeZnCpsQSElEicDrlAMLkRXHoyKN8A/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=10 "")  
  
  
5.黑客必读书单  
  
****  
6.面试题合集  
  
  
当你自学到这里，你就要开始  
**思考找工作**  
的事情了，而工作绕不开的就是  
**真题和面试题。**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dnMVja8hzZpia0AkKu6AWrQnXxPNhSSySbwUMEWOicYYS62D1UOQExv0cYuVQ68gk2uFF2xJ4TPmRHA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=12 "")  
  
**更多内容为防止和谐，可以扫描获取~**  
  
****  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NAkrkExZ3dnMVja8hzZpia0AkKu6AWrQnGktIUCicPreibR6b3sx1Qu0CsCZP0sZtCP4RHlMdxXuE4icCFSoL2yyBg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=13 "")  
  
  
朋友们需要全套共  
**282G**  
的《**网络安全/黑客技术入门学习大礼包**  
》，可以**扫描下方二维码免费领取**  
！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/UlibJFdibia1g4d5GFIWq9ypg0WGrTtgH8Y7MlLyGO27qMDNPwnoMHVB7hLJXxRZt5mapa4vo5eOibLvibpJXOXyDlPQGPuA1tfK4KBdG1eBt36Q/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=18 "")  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/TiaI8Dth4IiaRCFva2ZibMZKuNBEDOAEmkULH6MxzBRGa9Fibvuic8pv9cEjY0HWQbamrjGDz4jUgPS7TpprXiagZe6A/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=15 "")  
  
**END**  
  
  
****  
  
