#  打穿虚拟化边界！Corvus AI 与 隐藏十年的 Linux 内核漏洞的一场对决  
原创 腾讯朱雀实验室
                    腾讯朱雀实验室  腾讯安全应急响应中心   2026-10-19 09:36  
  
一段自 2017 年起潜伏在 Linux 内核主线近十年的底层缺陷，在真实环境中被 Corvus AI 用时 2 小时捕获。  
> 这处被命名为 Vghost（CVE-2026-74580，CVSS 8.8 高危） 的漏洞，广泛存在于 Ubuntu、RHEL、Amazon Linux、Azure Linux、Oracle Linux 等全球主流企业级服务器系统中。在实际业务场景下，它不仅能让任意普通低权限账号在几秒钟内直接接管整台服务器的最高管理员权限；在开启特定虚拟化加速配置的物理机或 AI 沙箱中，被关在虚拟机内部的程序还能突破隔离，直接篡改宿主机进程，威胁同一台物理机上的其他业务与核心资产。  
  
> 发现该漏洞的 Corvus AI 是由 TencentOS 安全团队构建的漏洞研究智能体（上线至今已发现 100+ 个Linux 内核 0day，1day 检出率达 92%，守护 TencentOS 19 个版本及百万级服务器风险清零）。这一次，Corvus AI 联合腾讯混元 Hy4 preview 模型，全自动完成了从漏洞定位、分析到修复补丁编写与提交的全流程，修复补丁现已合入 Linux 上游主线。  
  
  
  
**前言**  
  
  
  
过去两个月里，Corvus AI 在 Linux 内核深水区连续发现了上百个安全隐患。  
  
今年 8 月，团队刚刚公开了潜伏长达 18 年的高危漏洞 https://mp.weixin.qq.com/s/v4kFYfoO4aFhE-pSpnr7qg （CVE-2026-64564）。利用此漏洞，攻击者在普通 Docker 容器内部即可夺取主机 root 权限。TencentOS 安全团队积极协同社区完成了补丁合入，为大量依赖容器隔离的生产业务提前排除了安全风险。  
  
在多租户等场景中，容器和虚拟机承担着隔离业务、限制代码访问范围的职责。一旦隔离被突破，风险就可能从单个运行环境扩散到其他业务。这一次，Corvus AI 将研究重点转向了虚拟机（VM）与宿主机之间的隔离。  
  
在 Linux 虚拟化相关组件中，Corvus AI 发现了自 2017 年起就存在于主线代码中的 Vghost（CVE-2026-74580）。该漏洞可用于本地提权；在启用特定虚拟化加速配置的环境中，虚拟机内的程序还可借此篡改宿主机进程。  
  
下面是 Vghost 的本地提权效果：实机演示中，普通用户运行利用程序后，在数秒内获得了当前系统的 root 权限。  
  
  
演示视频 -   
本地提权漏洞利用视频  
  
  
**漏洞详解**  
  
  
  
现实世界里，我们访问的大多数网络服务都运行在虚拟机中。同一台物理机会同时承载多个租户，彼此互不影响，单台虚机出故障或被攻破也不能波及宿主机，这种隔离是多租户架构最基本的安全前提。  
  
一台虚拟机，本来应该像一个被完全封闭的房间。但 Vghost 让房间里的数据穿过了隔离墙，直接写进宿主机的内存。因为宿主机在替虚拟机搬运数据时，用了一份没来得及更新的地址缓存，而这份旧的缓存内容可以被攻击者控制，数据被按旧记录写到了宿主机自己身上。在服务器开启了特定的虚拟化加速配置时，这种逃逸危害就更有可能发生。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvbiaLRYic4jEn87c4zkcQ16DUEiaIicpHvqkJTY7EwzcZWEZ90s0AHug6Xj7zUMHq0uqSub79ia9YUDMGdJnk1TuUwYBIj4nGqic3XDs/640?wx_fmt=png&from=appmsg "")  
  
图1  虚拟化逃逸的危害  
  
这意味着如果业务把安全寄托在虚拟机隔离上，比如企业用虚拟机进行业务隔离和AI Agent 运行不可信代码的沙箱等场景，这道被默认信任的边界就可能从内部被击穿：攻击者在一台虚拟机里获得权限后，就有机会控制宿主机，进而威胁同一台宿主机上的其他业务。  
  
下面的视频展示了这一逃逸过程：利用 Vghost，攻击者可突破虚拟机与宿主机之间的隔离边界。  
  
  
演示视频 - 通过虚拟机攻击宿主机  
  
上方视频展示了左右两个终端：左边是虚拟机（Guest），攻击程序正在里面触发 vhost 漏洞；右边是宿主机。运行完PoC 之后，可以看到右边宿主机程序运行的运行轨迹已经被虚拟机篡改了。  
  
而比单次攻击演示更令人细思极恐的，是这道虚拟化隔离屏障背后长达十年的攻防暗流。  
  
  
   
  
**打破虚拟化的隔离屏障**  
  
  
  
回看虚拟化的发展历程，这一点反复被验证——2014 年，AWS 为修 Xen 逃逸，在漏洞公开前重启全球近 10% 的 EC2；2017 年起，虚拟化逃逸在国际赛事高频出现；2026 年，KVM 内核接连曝出 ITScape、Januscape、Zapscape。内核漏洞研究，正从用户提权走向突破系统边界。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9icASLQUQzvZY25719Sxia2F0kn4CvtRbIPpP5DWfhWCDiaoLvosfAer4GC6vaamaYFicxBJ84sxrwXeJG3H11Sx3f9jJtzQGKOsNnokn8XMTEU/640?wx_fmt=png&from=appmsg "")  
  
图2  2016—2026 虚拟化逃逸能力样本  
  
Vghos  
t 正是  
这种情况。围绕这一缺陷，团队完成了本地提权验证，并进一步确认了一条新的虚拟机逃逸向量。在指定的虚拟化配置下，客户机能够越过原有内存访问边界，向宿主 QEMU 进程内存发起越界写入。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvYcOic9yyG7VjXI0DTyrtYLc4KEngKwcRPjQGEfaPqmv4viaNO2Df4WqXywxRzV9A0LelAcmib1MZ1WR8QYYtomsaVsqcQxOIXn2s/640?wx_fmt=png&from=appmsg "")  
  
图3  逃逸向量示意图  
  
由于该  
内核缺陷既能实现宿主机本地确定性提权，又能跨越虚拟化边界改写宿主进程内存，Linux Kernel 官方 CNA 为 CVE-2026-74580 给出了 8.8 分高危评级。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvbbOLm0SggMNiaA7YW8a3Dz2958CAMC2Mypy7BA7cciaH6alZZFia06L820AAQGe7YStbKcHmBhZH8C5VkX8azKwryynQjeaIo6QM/640?wx_fmt=png&from=appmsg "")  
  
图4   CVE-2026-74580 记录  
  
既然这处足以击穿虚拟化边界的缺陷自 2017 年起就躺在 Linux 主线最核心的组件里，为什么过去近十年间，无数的静态扫描工具和全天候运行的内核模糊测试（Fuzzing），以及人工代码审计都没能抓住它，而 Corvus AI 却能在 2 小时内精准锁定？  
  
  
  
**Corvus AI 如何两小时发现 Vghost**  
  
  
  
这条跨越边界的路径，起点藏在 Linux 内核复杂的代码与运行状态之中。  
  
美国国防高级研究计划局（DARPA）发起的 AIxCC（AI Cyber Challenge）竞赛曾把在 Linux 内核中寻找高危漏洞的难度直观地画了出来：  
  
  
如视频所示，仅 AIxCC 比赛涉及的局部 Linux 内核代码，就包含三万多个函数、七万多条调用关系。在决赛的 8 小时挑战中，只有一个全自治系统发现了 Linux 内核漏洞。该系统的核心开发者，如今也参与了 Corvus AI 的建设。  
  
我们现在把视野放到完整的 Linux 内核源码树，要在真实环境中捕获 Vghost 并完成全自动验证，Corvus AI 在研究过程中跨过了三道技术难关：  
1. 跨时间尺度的逻辑盲区：  
   
对缓存全生命周期进行完整状态推演。  
  
1. 多步触发时序：  
传统 Fuzzing 几乎不可能撞中触发漏洞的组合，AI 需要自己读懂代码条件并倒推出触发剧本。  
  
1. 稳定漏洞利用：  
Corvus AI 需要在严格的权限隔离下，自主推理出一条可行的利用路径，并完成跨多个主流发行版系统的自动化验证与补丁回归测试。  
  
**模型能力升级带来的研究提速——Hy4 preview**  
  
  
  
要连续跨过上述三道难关，多智能体流水线解决的是研究流程的调度与协作，而决定系统能否跑通端到端漏洞研究流程的，则是底层大模型的工程推理能力。  
  
Corvus AI 的整体架构，我们在上一篇（[TencentOS 科维斯AI首秀：SCTPhantom——潜伏18年的Linux内核提权与容器逃逸漏洞](https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651208584&idx=1&sn=fe28c3d4e1ee43b16379d36785ebe1b9&scene=21#wechat_redirect)  
  
）里已经详细介绍过，此处不再展开。  
SCTPhantom 公开后，netdev 上围绕 SCTP 的补丁与讨论明显增多。风险曝光会迅速带来集中排查，而 Corvus AI 持续挖掘的，正是那些长期被忽视的组件及其安全问题。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvYr3eqc0Ah1o6lYEr4fPRXqFKXYl8s5dygk6df83yKlonjWLpibsFW7E8L4ep3ea2JC4sONXq6QS7j7PRQSwOSEjt4YSVP4aPYM/640?wx_fmt=png&from=appmsg "")  
  
图5   Kernel 邮件列表，sctp 组件相关补丁已经有上万次交流  
  
在本次 Vghost 的完整研究闭环中，我们进一步接入了腾讯混元最新发布的  
  
Hy4 preview  
 模型，将其部署在考验代码能力的漏洞利用验证与修复补丁检查两个核心节点。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvbGOAyQx9WSrITxiagmice8zXUQUM4xMI3ibnUt376QpticnPO34SFvuYQZveibahMDSUoAxTaDZib0ZzJfYE8vwpzDAYNVCojzGF7po/640?wx_fmt=png&from=appmsg "")  
  
图6   Hy4 preview 模型能力评估  
  
从涵盖 12 项前沿基准的评测结果来看，Hy4 preview 相比前代 Hy3 展现出了跨代式的能力跃迁。  
  
评测数据表明，Hy4 preview 的提升本质上是底层软件工程与自治执行能力的全面跃迁：衡量真实软件修复的 DeepSWE 效果翻倍，真实命令行操作的 Terminal Bench 2.1 追平了 Claude Opus 5（已收录于   
Papers With Code  
等第三方公开榜单）。这种在复杂代码重构、自主终端调试与长程时序追踪上的综合突破，正是它能够在无人干预下完成 PoC 动态利用验证与高精度补丁差分分析的关键所在。  
  
这种在长程代码推理与复杂工具调度上的底层突破，直接转化为了 Vghost 研究中的实战效率：接入 Hy4 preview 后，Corvus AI 不仅在 2 小时内锁定了潜伏十年的缓存失效根因，还全自动完成了跨内核版本的通用利用验证，并对上游修复补丁进行了严密的回归检查，确认其彻底切断了漏洞触发路径且不损耗正常虚拟化性能。  
  
  
  
**跨生态验证与协同修复**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9icASLQUQzvZsQYyLia0icPz2tJtWbmN7BXyC1JPpyh1OHqm62wabzwWPwjIiaMJpneL4oU8p57fcCJtbS9zN2I1buTtkWGBPu3jicXnyn47ZDag/640?wx_fmt=png&from=appmsg "")  
  
团队使用同一份 x86-64 利用代码，在上述六个发行版的测试环境中验证了本地提权链。修复补丁已合入 Linux 上游并回移植至相关维护分支，Amazon Linux、Oracle Linux 和 TencentOS 已发布安全更新（注：腾讯云 CVM 经全面排查不受该虚拟机逃逸配置影响）。  
  
关于受影响内核版本范围、运维自查步骤及安全补丁升级指引，详见   
TencentOS 官方安全公告（  
https://cloud.tencent.com/document/product/1397/138440  
）  
。  
  
  
**结语 - 从一次发现到持续交付能力**  
  
  
  
过去一个月，Corvus AI 独立发现的多个内核漏洞已获得官方 CVE 编号，另有上百个候选高危缺陷正在协同披露与修复中。通过多 Agent 协作，并在利用验证与补丁检查环节接入腾讯混元 Hy4 preview，Corvus AI 正持续完成从漏洞分析、利用验证到修复检查的研究工作，向社区提供可复现的验证结果与修复支持。  
  
攻击者可以集中资源寻找一处可用的突破口，防守方则需要覆盖不同组件、配置与部署环境。当攻击方也借助 AI 加速漏洞挖掘和利用构造时，这种不对称会让防守更加被动。防守方需要把自动化能力用于持续排查、验证和修复，尽可能在漏洞被利用之前缩小风险暴露范围。  
  
这正是 TencentOS 安全团队构建 Corvus AI 的初衷：让开源基础设施拥有与代码演进速度相匹配的自检与自愈能力。  
  
待上游社区与各发行版完成本轮补丁同步后，我们将继续公开后续一系列内核安全案例的技术细节。  
  
  
关于  
Corvus AI  
  
本案例由科维斯AI（Corvus AI）跑通完整漏洞研究流程，其中修复补丁检查与漏洞利用验证由 Hy4 preview 提供模型能力。  
  
科维斯AI（Corvus AI）是由 TencentOS 安全团队构建的 Linux 发行版漏洞研究智能体。面向真实复杂的内核安全场景，系统依托多Agent 协同架构与 Harness 深度优化，实现了从漏洞挖掘、根因分析、稳定利用构建、跨环境验证到补丁开发的全链路自动化研究。Corvus AI 始终坚持负责任的漏洞披露原则，积极携手上游社区推动协同修复，致力于从源头提升开源 Linux 发行版生态的整体安全水位。  
  
