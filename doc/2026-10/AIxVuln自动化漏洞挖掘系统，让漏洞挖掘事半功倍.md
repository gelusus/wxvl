#  AIxVuln自动化漏洞挖掘系统，让漏洞挖掘事半功倍  
youki
                    youki  C4安全   2026-10-10 06:47  
  
Technology  
  
AIxVuln自动化漏洞挖掘系统  
  
漏洞挖掘事半功倍  
  
  
AIxVuln自动化漏洞挖掘系统  
  
  
  
**点击上方蓝字关注我们**  
  
现在只对常读和星标的公众号才展示大图推送，建议将  
C4安全“设为星标”~  
  
  
  
  
工具介绍  
  
  
  
  
**0****1**  
  
**AIxVuln**  
 是一个基于大模型（LLM）+ 工具调用（Function Calling）+Docker 沙箱的自动化漏洞挖掘与验证系统。  
  
系统通过 Web UI / 桌面客户端管理"项目"，为每个项目自动组织多个数字人协作完成环境搭建、代码审计、漏洞验证与报告生成，并在隔离的 Docker 环境内完成依赖安装、服务启动、PoC验证与证据采集，最终产出可下载的漏洞报告。  
  
目前已通过该项目在真实开源项目中发现数十个漏洞。  
  
  
工具特性  
  
  
  
  
**0****2**  
  
**首次启动引导**  
— 首次运行自动进入初始化向导，引导创建管理员账户并一键构建所需 Docker 镜像，开箱即用  
  
  
**单二进制部署**  
— Dockerfile、前端 UI 等资源全部嵌入可执行文件，无需额外文件即可运行  
  
  
**项目化管理**  
— 支持从 Git 仓库、压缩包上传、压缩包 URL 三种方式创建项目，一键启动/取消，实时查看漏洞列表、容器、事件日志与报告  
  
  
**数字人协作**  
— 每个数字人拥有独立人格（姓名、性别、年龄、性格、头像、自定义提示词），绑定特定 Agent 能力类型，以持久化实例运行，跨任务复用记忆  
  
  
**决策大脑（DecisionBrain）**  
— 全局调度中枢，维护状态面板与记忆体，自动编排数字人、汇聚碎片化利用点（exploitIdea）并组装攻击链（exploitChain）  
  
  
**团队聊天（Team Chat）**  
— 用户可通过 @数字人名 或 @全体 与任意数字人 / 决策大脑实时对话，数字人之间也可通过 TeamMessage 机制广播消息  
  
  
首页 — 项目管理与创建  
  
  
  
  
**0****3**  
  
支持从 Git 仓库、压缩包上传、压缩包 URL 三种方式创建项目，一键启动漏洞挖掘任务。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/g1pTgnczBiaGmk0xg3yz0vBk41xuCsqFxax34Mybns3nu5iawTIQJnobhLfOscmQxssib73trXKw0GGh48LRm4uGjtdEt23txNAgoR3iaiaiavib5c/640?wx_fmt=png&from=appmsg "")  
  
  
  
项目详情 — 实时状态总览  
  
  
  
  
**0****4**  
  
运行中的项目详情页，左侧展示数字人工作状态与容器列表，右侧展示环境信息（登录凭证、数据库信息、路由示例等），所有信息实时更新。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaHZdjqKcY9bC5jnUt50TAm1jEnP4C2bMbCy4wk2eHrQGUTnpoYLBXtrCLh0FDeicc26TNY0micRdhX2qmib9NeoAdkJzxZkVp4myU/640?wx_fmt=png&from=appmsg "")  
  
  
  
数字人管理  
  
  
  
  
**0****5**  
  
管理 Agent 数字人角色，每个数字人拥有独立人格（姓名、性别、年龄、性格、头像）和自定义提示词，绑定特定 Agent 能力类型。支持增删改，修改后重启项目生效。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaE9L2PersPRyO00w2aXBUguqYTzxOkf5mN5tSSD3bxBWsEicH7d2pVRCYBgKBocN6rJ0ib2ArncqUkvwXtkHmBU6HXyibFJP8jPiaA/640?wx_fmt=png&from=appmsg "")  
  
  
  
严格审核机制 — 防止AI幻觉  
  
  
  
  
**0****6**  
  
ExploitIdea 经过"审核失败 → 正在整改"等多轮状态流转，决策大脑对每个候选漏洞进行严格审核，防止 AI 幻觉导致的误报。ExploitChain 组装后同样需要经过验证流程。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/g1pTgnczBiaElKoWQKUznFcuQT65so6JGm5PA583crpZUWT9s5c6y0nf5afHTopIOsjrZumv4aOUQ6TZ62jOVZbYVz4xKaMSqZ9OHVaXfPfg/640?wx_fmt=png&from=appmsg "")  
  
  
  
高效的团队沟通机制  
  
  
  
  
**0****7**  
  
数字人之间、数字人与决策大脑之间通过 Team Chat 实时协作。以下展示了一次真实项目中的多轮沟通过程：  
  
环境搭建阶段 — Ops 数字人汇报编译问题，决策大脑给出修复指令，数字人自主执行修复：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaFYW2bAdkLWgnStL3O3lgEUqIheWKgjfJgJhDHBUu6MnYjpdSV8BgbIa9btMaUNfbsiaOv4D2ol6JSsgLJRbeLPNYAHUsyK3p94/640?wx_fmt=png&from=appmsg "")  
  
用户实时介入 — 用户通过 @数字人名 直接下达指令（绿色气泡），决策大脑同步协调其他数字人处理环境问题：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/g1pTgnczBiaG0u6PurZbeSUtuqbM8vMafx7Yibc0FFQSIs76NicZXfKjicJwJGsaxXebzKzWzlNib6ULW1yibPsfqrHq3xLTyM3Hdbe5YNodFJAwU/640?wx_fmt=png&from=appmsg "")  
  
决策大脑指导 — 决策大脑针对 MySQL 连接问题给出详细排查步骤和重建方案，数字人据此自主执行：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaGbaCKcAUCbOHgrEDVetDjcRYkvbc0SO7aicZHh2mqn05RbcS91Qa3318RibeSuVX7tHn4oN9I4w00PhlZn2GFugg0uHamX69abk/640?wx_fmt=png&from=appmsg "")  
  
多线并行 — Analyze 数字人广播发现的漏洞线索（@all），Ops 数字人同步处理 Maven 依赖问题，Verifier 数字人等待环境就绪后立即开始验证：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaHQuLzX8N7WxWZZKRGBXficTXtqI0vXQwxBxOVQibwSSBMa6Ribn4k4stfVSIIsgUBVfwibibHUWKRMM87rkrvPwXvNIcmjn7ibF2HYU/640?wx_fmt=png&from=appmsg "")  
  
自主协作 — 多个数字人同时汇报进展、分配剩余配额、用户可随时 @任意数字人 进行干预：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaHrx0qNM1dxXT9Aiat50MtDTYbibLuUiaseFHq76q8uIU9RKCL52ic4Sd8YcHKGV6kibLnllMic6HDl7jvOiasAvNCF6ohVPleHEicIJlM/640?wx_fmt=png&from=appmsg "")  
  
  
  
漏洞报告示例  
  
  
  
  
**0****8**  
  
系统自动生成结构化漏洞报告，包含完整利用链分析、攻击流程图和验证证据：  
  
完整利用链分析 — 自动绘制从攻击入口到最终危害的完整调用链路图：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/g1pTgnczBiaG0OzGb6VFS5o6WnKkbwRn3sDP7J13OGmdBVGWGbkv8Tn0WSbLgiccV4gSrEgdQKkwF25SGRE6DyDe2P8Y0padQrrawS35Khco4/640?wx_fmt=png&from=appmsg "")  
  
验证证据与 PoC — 包含时间盲注验证、UNION SELECT 探测等详细测试过程和关键证据（HTTP 请求/响应）：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/g1pTgnczBiaFJbgCDlymKHouawJCaMPXdE173zl1mial6FFJC9rZSmiaFR3XzvKfTjgt99GqUF0UAwUg6nWXcuoG5AdC9MicyArcPDyRyqfX4F4/640?wx_fmt=png&from=appmsg "")  
  
  
  
相关地址  
  
  
  
  
**1****9**  
  
关注公众号联系微信备注**“入群”**  
，即可进入**C4安全交流群**  
！  
  
关注微信公众号后台回复“**20261010**  
”，即可获取项目下载地址！  
  
  
**内部圈子介绍**  
  
  
  
  
**1****0**  
  
**《安全渗透感知》**  
是FreeBuf知识大陆的重量级帮会，帮会致力于漏洞POC/EXP、红队攻防实战，是系统化从基础入门到实战漏洞挖掘的教程社区，包含团队自整的挖掘注意点和案例，还包含分享的渗透经验、SRC漏洞案例、代码审计、挖洞思路等高价值资源。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9CN4ib56JKcvxcsL6E1P18PFaSyoXwAciarkLxvclTsbolRZk7lfT2UMB59AfgwiaRC4BTGoObmC9aN2P3E0Ezlu1yeCUsqGMC09hU/640?wx_fmt=png&from=appmsg "")  
  
******内容框架（持续新增中）**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CPSibSD0AcRjajAQNicfrnNH1ibPwWebdiak5z8O3Romc6jGVfPt9qCgEfef6MI35OtlwdgfrI5nwfXO74obVj3IUoDwBCOw6sQWow/640?wx_fmt=png&from=appmsg "")  
  
目前已有「**770+**  
」小伙伴加入了圈子有意向的师傅们可以扫码加入我们，共同进步。  
  
  
目前星球已满770人，价格由79.9元调整为99.9元(交个朋友啦)，1000名以后涨价至129.9元。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/niasx7fyic9CMibaTvQrX7MvrHTFWRP7iarEGVKWE1Bicv6bKHvI7W3J4NIQY8eMG1PycgLlia8eZLvHFMHOJr2MV2Co7QNyMhGfmhSCqr3bGvicjE/640?wx_fmt=png&from=appmsg "")  
  
  
  
往期推荐  
  
  
  
  
**1****1**  
  
[1.用友U8C nDay手到擒来，Provena-Audit自动审计出洞](https://mp.weixin.qq.com/s?__biz=MzkzMzE5OTQzMA==&mid=2247491031&idx=1&sn=a9575091925efcb630f0b0e7d5c3a4c7&scene=21#wechat_redirect)  
  
  
  
[2.基于Ai自主代码审计的红队Agent，真正做到审计出漏洞](https://mp.weixin.qq.com/s?__biz=MzkzMzE5OTQzMA==&mid=2247491064&idx=1&sn=e7a949d5ed1cab94f9f8a1a874b37faf&scene=21#wechat_redirect)  
  
  
  
[3.记录几个edu漏洞挖掘思路](https://mp.weixin.qq.com/s?__biz=MzkzMzE5OTQzMA==&mid=2247491070&idx=1&sn=af1e65ce3ac3bb3f6dafba9533a02cb9&scene=21#wechat_redirect)  
  
  
  
[4.记一次红队Agent自主挖掘漏洞的过程](https://mp.weixin.qq.com/s?__biz=MzkzMzE5OTQzMA==&mid=2247490921&idx=1&sn=9953ff4768fb430a61e3a0b1eff09acf&scene=21#wechat_redirect)  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/niasx7fyic9CMZcUlnsiclOFRAqoS7ZIlGic59teLhDC8DmRgIN8ocYKUs35Mw319JG8nAT9FQKUvATnyIqrcpHPJPC5w386b2IvjcuqQPjLQN0/640?wx_fmt=png&from=appmsg "")  
  
END  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/niasx7fyic9CMBrGOj3UPwVKe4V8FEegoSVVEjicGWokLuXId2HjhlUJTDjiaWib0pRzzwerRHE1uLbsaJHHOPMLN7XHEKUU6jv7eicWDSfc1fHxs/640?wx_fmt=jpeg&from=appmsg "")  
  
  
关注我们  
  
  
  
  
  
