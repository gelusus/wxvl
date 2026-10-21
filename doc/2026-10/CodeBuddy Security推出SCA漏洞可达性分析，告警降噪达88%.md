#  CodeBuddy Security推出SCA漏洞可达性分析，告警降噪达88%  
 云鼎实验室   2026-10-21 07:21  
  
# 告警越来越多，先修谁仍靠人判断  
  
AI 智能体写代码、引入库依赖的速度，正在超过团队审查和分诊这些产出的速度。SCA 能快速找出受影响组件，但扫描后的问题仍留给团队：这么多告警，先修哪条？  
  
只按严重性排序还不够，真正决定"先修谁"的信息其实在你的项目里：代码调没调那个入口，配置踩没踩那个前提，项目有没有把官方给的缓解开关拉上。版本匹配告诉你“依赖有风险”，却不足以说明它与你的业务代码有多大关系，研发仍要逐条翻代码、查配置。  
  
ICSE’23 的 Maven 生态研究分析了约 4977 万次下游到上游库的调用，结论是：86.1% 的漏洞函数无法被任何下游项目访问到；真正落在风险路径上的比例只有 4.4~4.7%。另一项 2025 年的研究（VFArchē）做了对照实验：拿 50 个新 CVE 去 25 个真实依赖它们的项目里验证，主流 SCA 工具的告警里，漏洞真正可达的只占约一成。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/iczOE5KEJun65xjLvSH5cj54DIbawOoAaDv655kR6fTAViab1w7It92OOpCExZ2HyX5G3Wdj3ZNNp3FMRePib3veQJ1Vxf0TAttykbqpZAQzzs/640?wx_fmt=png&from=appmsg "")  
# CodeBuddy Security：为告警补上项目证据  
  
CodeBuddy Security 推出 SCA 漏洞可达性分析，新开发了 ReachX 引擎，通过 AI 提取证据、规则给出判定，将漏洞信息与项目源码、配置关联起来，帮助团队缩小重点复核范围，并为修复排序提供依据。  
  
核对触发入口：检查漏洞 API 的真实调用，也覆盖配置驱动的触发条件，区分直接调用、别名引用与同名误匹配。  
  
检查缓解措施：核对适用于该漏洞及版本的配置项、启动参数等措施，补充已有防护的证据。  
  
提供可复核结论：给出代码位置和判定理由，证据不足标为“不确定”，让研发能从具体调用点开始核查。  
# 看效果：从告警概览，到单条证据  
  
➢  
概览：缩小重点队列，让排序有依据  
  
在真实开源项目 510 条去重后的告警里，判为可达的 59 条，占 11.6%——落在生态级研究给出的区间内。如果先将这 59 条列为重点复核队列，相比逐条处理全部告警，队列规模缩小 88.4%  
  
降噪的价值不止是数量变少，而是让排期有据可依。同为高危，有明确调用证据的问题值得优先核查；证据不足的告警仍需复核，不能自动排除。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/iczOE5KEJun6vQK2RRo6DlefbGzS4vu5kO6GsgFoAuZialduNvnhPddbM4GPl1YLrb1YJhicWxsXiazaPpUdb4iagcy85namsq4nMaK3WhShJv68/640?wx_fmt=png&from=appmsg "")  
  
➢  
详情：从一条告警，定位到真实调用  
  
以 golang.org/x/crypto 的一条解析型 DoS 漏洞（CVE-2026-39829） 为例，项目在 go.mod 声明 v0.22.0，并直接引用 ssh 子包。7 处真命中锁定核心攻击面，ParseAuthorizedKey 命中 5 处、ParsePublicKey 命中 2 处，本次复核的 7 条均确认为真实直接调用，包括 gossh 别名导入的调用，每条命中都有具体代码位置和判定理由。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/iczOE5KEJun6IxttWbanIeXkQ4wczUqibHyJrWJkDYdiayiaYG7ytrAMBsbAHzrRLP0qCwNQfmmRHnvagyzEbtDLPWicvOSjTxlicGraz3x5XGNCM/640?wx_fmt=png&from=appmsg "")  
# 结语  
  
可达性分析不是免修金牌，而是给告警补上项目证据。安全团队最稀缺的是注意力：让有触发证据、缺少有效缓解的问题更早得到处理，才是这条排序键的价值。  
  
CodeBuddy Security 由腾讯云鼎实验室、腾讯科恩实验室与腾讯云代码分析（TCA）团队联合研发，支持 SaaS、VPC、私有化部署与 CI/CD 接入，SCA 漏洞可达性分析已随平台开放。  
  
> 参考：  
> ICSE’23: Understanding the Threats of Upstream Vulnerabilities to Downstream Projects in the Maven Ecosystem  
> arXiv 2506.18050: VFArchē（600 CVE 漏洞函数人工标注与下游对照实验）  
  
  
