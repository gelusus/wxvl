#  「AI漏洞挖掘」AutoCVE拆解：多Agent自动审源码，一周提交30个CVE  
原创 句芒安全实验室
                    句芒安全实验室  句芒安全实验室   2026-10-13 06:31  
  
## 从"看着不对劲"到"拿到编号"，中间隔着一条流水线  
  
做过代码审计的人都懂，最磨人的从来不是"看不出问题"，而是"看出来了却推不动"。一个函数里参数没做边界检查，你心里清楚这是个洞，但从这里到一个真正的 CVE 编号，中间还排着一长队活儿：项目得挑有价值的、仓库得拉下来、代码得逐层读、误报得一条条滤、漏洞得动态跑一遍证明它真能触发，最后还得按 CNA 的格式把报告写出来。任何一个环节掉链子，前面全白干。  
  
过去的做法是"人肉串行"：扫描器出候选，人做判断；人判断完，再手动去写 PoC。这两年 Agent 起来了，不少团队的第一反应是"让大模型帮我读代码"。真做过就知道，直接丢一个仓库问模型"有没有漏洞"，得到的多半是似是而非的猜测——它没有工具去验证、没有状态记住自己读到哪、也没有机制逼它把结论落成结构化证据。  
  
句芒这次拆的 larlarua/AutoCVE  
，就是冲这条流水线来的。我核实了一下（2026-10-12）：这个仓库 **1435 个 star、136 个 fork、14 个未关 issue**  
，Python 写的，AGPL-3.0 协议，2026-06-15 建仓，最近一次推送就在 **2026-10-01**  
，最新版本 **v1.0.5（2026-07-12）**  
，大约 **3 位贡献者**  
。  
  
它的自我定位一句话：**把"项目筛选 → 仓库导入 → 审计任务 → Agent 挖洞 → CVE 申报报告"整条链路自动化。**  
 用户最后只需要复制报告、提交，就完成了后续的 CVE 申请。  
  
![AutoCVE 官方仓库架构图（larlarua/AutoCVE 仓库真实素材）](https://mmbiz.qpic.cn/mmbiz_jpg/J2hBCjr4Lft2X6qbtiazj6hYnNWW2BElaHJWUYRMVlDvKCLnYLV5cfGbNHfW5547YF9Jo6JFB1MCcw6CRyDfnBHH830Sko2BGvQXmibRv0hGw/640?wx_fmt=jpeg "AutoCVE 官方仓库架构图（larlarua/AutoCVE 仓库真实素材）")  
## 它到底是什么  
  
一句话：**AutoCVE 是一个把 CVE 挖掘 Agent 化的开源平台——你给它一个 GitHub 项目，它派出一组分工明确的 Agent 去读代码、验漏洞、出报告，最后给你一份可以直接提交给 CNA 的漏洞报告。**  
  
技术栈很实在：Python 3.11+，后端 FastAPI，前端 React 18 + Vite，数据落 PostgreSQL 15，Redis 做运行时缓存，沙箱单独一套镜像。部署也简单，一行命令就能起：docker-compose.prod.yml  
 拉起来之后，前端在 3000、后端 API 在 8000、Swagger 在 8000/docs、Adminer 在 8080。  
  
它保留的功能也全：仪表盘、项目管理、审计任务、Agent 审计执行页、审计会话、漏洞管理、一键 CVE、Skills 管理、系统设置。其中"审计会话"这个设计很关键——它把一次审计的全过程当上下文，用户可以围绕某次审计继续追问、补证或让 Agent 扩展分析，而不是每次从零开始。  
## 亮点一：多 Agent 分工，不是一个大模型硬扛  
  
AutoCVE 的核心是一张编排图。Orchestrator  
 统一调度五个节点：Recon  
 做项目侦察（识别语言、框架、入口文件、优先审计路径），Scan  
 调规则扫描、依赖扫描、密钥扫描，Triage  
 对扫描候选复核、滤误报、补证据，Finding  
 直接读代码挖高价值漏洞，Verification  
 用沙箱、测试工具和 PoC 动态验证。  
  
编排不是"全开"：Orchestrator  
 和 Recon  
 默认强制启用；Scan  
 打开后 Triage  
 才有输入来源；Verification  
 只有在 Triage  
 或 Finding  
 启用时才有意义。项目大时 Recon  
 倾向完整分析，目标文件少时倾向轻量分析。换句话说，**工作流是根据任务配置、项目规模和 Agent 开关动态算出来的，不是写死的。**  
  
Agent 之间也不靠"共享一大段无结构文本"协作，而是走结构化 TaskHandoff  
：上一阶段的发现、摘要和置信度，目标文件和推荐审计路径，扫描原始候选，复核后的可信候选，结构化漏洞，验证状态和补充证据，一层层往下传。重复漏洞由 Orchestrator  
 按 file_path + line_start + vulnerability_type  
 三个维度合并。  
## 亮点二：Finding Agent 的 ReAct 循环 + 状态机  
  
Finding  
 是这套系统里最核心的 Agent。它不读扫描器候选再复述，而是直接面向代码挖洞。它的运行时是一条工程化的 ReAct 循环，围绕"模型消息、原生工具调用、工具结果、状态转换、终止动作"转。  
  
它把"为什么继续"变成了可记录、可解释、可调试的状态。Continue 原因有九种：本轮执行了工具调用的 next_turn  
；模型没调终止工具但阶段要求终止的 terminal_action_nudge  
；模型输出了伪工具语法的 legacy_tool_syntax_nudge  
；输出被长度截断后的 max_output_tokens_escalate  
 和 max_output_tokens_recovery  
；上下文过长时的 reactive_compact_retry  
；上下文 collapse 后的 collapse_drain_retry  
；stop hook 判断不能结束的 stop_hook_blocking  
；token 预算判断还应继续的 token_budget_continuation  
。  
  
终止也一样有清单：completed  
、hook_stopped  
、blocking_limit  
、prompt_too_long  
、model_error  
、aborted_streaming  
、aborted_tools  
、stop_hook_prevented  
、max_turns  
。最理想的完成模式是 completion_mode = finalize_tool  
 且 terminal_action = finalize_finding  
。  
  
这里有个细节值得所有做 Agent 的人抄：**如果模型在自然语言里说"我已经完成了"，但没有调用 FinalizeFinding，系统不会直接接受**  
，而是先尝试触发 nudge，失败后把这次审计标记为 incomplete。它区分了"看起来完成"和"结构化提交完成"——这正是大多数 Agent 系统最容易自欺的地方。  
  
FinalizeFinding  
 本身也做校验，必填字段包括 vulnerability_type  
、severity  
、title  
、file_path  
、line_start  
、code_snippet  
、source  
、sink  
、confidence  
、needs_verification  
、exploit_chain  
、poc  
、cve_justification  
 等。payload 不合法时，工具不当作成功终止，而是返回 finalization_rejected  
，把校验错误和缺的字段退回给模型，让它继续补证据。  
## 亮点三：七种 Nudge，把跑偏的 Agent 拽回来  
  
Nudge 是这套系统的纠偏机制。模型偏离流程时，runtime 会提醒并试着把它拉回标准路径。我在架构文档里数出七种：原生 tool calling 提醒（不要输出伪工具语法）、文本伪工具语法 nudge、终止动作 nudge、继续意图 nudge（模型说"继续审查"但没调工具时）、token 预算续跑、stop hook 阻断、以及 finalizer 恢复。  
  
其中终止动作 nudge 上限配的是 2：超过上限模型还没调终止工具，runtime 就把结果记为 natural_end_without_terminal_action  
，并倾向标记 incomplete。这个设计的价值观很直白：**宁可承认这次没收敛，也不把一段自然语言总结包装成"成功结果"。**  
## 亮点四：工具调用有痕迹、有权限  
  
AutoCVE 的工具不是裸执行。所有工具先进 ToolRegistry  
，模型每轮看到的不是后端全部能力，而是"当前 Agent + 当前场景 + 当前配置"下可用的集合。调用进 ToolOrchestrator  
：查定义、校验输入、权限检查、写 AuditToolCall  
、触发 PreToolUse  
 checkpoint、执行、触发 PostToolUse  
、转成 tool result。  
  
一次模型回合里多个工具调用由 StreamingToolExecutor  
 排：只读且并发安全的并行；concurrency_key  
 相同的避免互扰；写入、shell、沙箱这类有副作用的串行；shell 类失败可以中止同批相关调用，防错误扩散。Write  
、Bash  
、PowerShell  
 这类能改文件或执行命令的工具，受更严格的权限和目录限制。  
## 亮点五：Skill 渐进式披露  
  
Skill 机制支持给不同 Agent 配不同能力。它的加载是渐进式的：启动阶段只注入 catalog、匹配结果和 route plan；命中某个 Skill 时，要求模型先调 Skill(action="body")  
 读完整 SKILL.md  
；SKILL.md  
 指向 references、checklists、examples 或 scripts 时，再调 Skill(action="read_resource")  
 加载具体资源。每次 Skill 调用都写进 Audit Session，记 skill_ref  
、action  
、resource_name  
 和加载阶段，runtime 还会记住这个 Skill 已经加载到哪一层，避免重复加载。  
## 战绩：一周 30 个 CVE，我核了几个  
  
官方给的数字是：**为期一周的测试里发现并提交了 30 个安全漏洞，覆盖 14 个开源项目，最高 CVSS 9.9。**  
 句芒不轻信数字，抽查了几个：CVE-2026-48765  
（typebot.io，授权绕过，CVSS 9.9）、CVE-2026-43986  
（Tautulli，未认证 SSRF，9.9）、CVE-2026-40904  
（Chartbrew，访问控制不当，8.1）、CVE-2026-45296  
（OpenReplay，跨租户信息泄露，7.7），我在 MITRE 的 CVE 记录里都查到了，状态都是 PUBLISHED。也就是说这 30 个不是 PPT 数字，是能查到官方编号的真洞。  
  
![AutoCVE 官方仓库界面截图：审计执行页（larlarua/AutoCVE 仓库真实素材）](https://mmbiz.qpic.cn/mmbiz_png/J2hBCjr4LfsPdj0NpEOR2Nrv1pqmL2HNicpD3jT92Mu05KOJqfH9MlbyQeCzjYBOVqudCcxCoygbJRWXicEvgmJE9jO8mZMwYFseqMMtjLJHM/640?wx_fmt=png "AutoCVE 官方仓库界面截图：审计执行页（larlarua/AutoCVE 仓库真实素材）")  
## 上手  
  
最省事的是源码部署：git clone  
 之后 docker compose up -d --build  
。起来之后打开 3000，配模型 → 导入项目 → 建审计任务 → 看实时审计 → 管漏洞 → 编辑或导出报告。模型方案支持为不同 Agent 配不同模型，工作流各节点也能单独开关。  
## 避坑  
  
三个坑先说清楚。第一，**它是 AGPL-3.0**  
，如果你的用法涉及分发或商用，先确认许可边界。第二，**它的审计质量强依赖模型能力**  
。这套系统的核心是让 Agent 去读代码、调工具、验漏洞，模型不行，循环转得再漂亮也挖不出东西；配模型方案时不要只看价格。第三，**"提交了 30 个 CVE"不等于"全部有效"**  
。CVE 从提交到 CNA 确认、再到公开，中间会掉一批；官方给的 30 是提交数，真正落地多少要看后续。另外它自己也在 README 里承认，架构和实现还有不少要打磨的地方。  
  
![AutoCVE 官方仓库界面截图：漏洞管理页（larlarua/AutoCVE 仓库真实素材）](https://mmbiz.qpic.cn/sz_mmbiz_png/J2hBCjr4LfudIz8QpylfSN2THPvB5aFBRRjib8JwDiaoIicWkn65vsDzZqsFQBftcCJJ0fIyjzYJibcQC8JZu1ialruxHVicIeicWvd6m8usonpwho/640?wx_fmt=png "AutoCVE 官方仓库界面截图：漏洞管理页（larlarua/AutoCVE 仓库真实素材）")  
## 适合谁  
  
适合三类人。一类是**做漏洞研究的安全团队**  
：想把"人工读代码"从瓶颈环节变成可并行环节，用它做候选筛选和初筛。另一类是**想学 Agent 工程的人**  
：它的 ReAct 循环、状态机、Nudge、工具编排、Skill 渐进式披露，是一份很完整的 Agent 工程参考实现——它自己也承认参考了 lintsinghua/DeepAudit  
 的工程架构，然后针对 CVE 挖掘场景做了重构。第三类是**平台/DevSecOps 团队**  
：想把代码审计接进自己的流程，它有 API 和 Swagger，能落进既有管线。  
  
不适合只想"扫一下出个报告"的人——它要配模型、要起一整套服务，重；也不适合把它当"全自动挖 0day 机器"的人——它挖的是已有项目里的洞，产出的是候选和报告，最终判断还得人来。  
  
（完）  
  
