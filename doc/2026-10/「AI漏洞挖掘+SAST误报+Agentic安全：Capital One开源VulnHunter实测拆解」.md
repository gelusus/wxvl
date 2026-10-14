#  「AI漏洞挖掘+SAST误报+Agentic安全：Capital One开源VulnHunter实测拆解」  
原创 句芒安全实验室
                    句芒安全实验室  句芒安全实验室   2026-10-14 03:12  
  
用 SAST 扫一遍代码库，出来几百条 finding，安全团队花两周把误报关掉——这是很多公司的日常。真正让人头疼的从来不是"找不出问题"，而是"找出太多不是问题的问题"。  
  
Capital One 把这套东西开源了。2026 年 7 月建仓、10 月初已经 1.1k star、161 fork 的 capitalone/VulnHunter，README 第一句口号是 "From pattern-matching to provability"——从模式匹配到可证明。它不讲"扫描器"，讲的是三件事：判断哪些缺陷真的可被利用、画出可能的攻击路径、给出有证据支撑的修复建议。许可证 Apache-2.0，主体是 Python，但真正的核心是一堆给 Claude Code 用的 skill 文件。  
  
![仓库主页真实截图：About 里写着 attacker-first analysis，Apache-2.0，1.1k star / 161 fork](https://mmbiz.qpic.cn/sz_mmbiz_png/J2hBCjr4LfvbH6vfzmJZrkXGp6icgN5V0R0lKMc5NJwDwL3xIvj25Eia15Io2dGq5MoJ4VNbHdYWB3tyD8TehXuQD46OicRrRgcB1X1ia3ibogBo/640?wx_fmt=png "仓库主页真实截图：About 里写着 attacker-first analysis，Apache-2.0，1.1k star / 161 fork")  
### 亮点一：attacker-first，从入口点向前推  
  
传统 SAST 走的是 sink-first：先盯危险模式（eval  
、字符串拼接 SQL），再往回找一个假想攻击者，于是大量"理论上可被利用"的 finding 被吐出来。VulnHunter 反过来做——**先从攻击者真正够得着的入口点出发**  
：HTTP API、网络消息、文件上传，然后沿着数据流"向前"推理，判断攻击者到底能不能打穿。  
  
它的分析纪律写得很细：读到 sink 之后必须去读生产代码，而不是测试替身；数据穿过 4 个文件 6 个函数也要一路追完；遇到分派函数（router、中间件链、data fetcher、工厂）不许停在抽象边界，要逐个追进每个分派目标；每一条出站调用（HTTP 客户端、gRPC stub、数据库查询、消息发布）都要把**所有参数**  
过一遍——尤其 dealerId  
、tenantId  
、userId  
 这类作用域参数，要确认传下去的到底是校验过的那一个变量，还是一个硬编码通配符。硬编码通配符替换掉校验过的 scoping 参数，在它的规则里是明确的越权。  
### 亮点二：Falsification Engine，让 AI 反驳自己  
  
找到候选漏洞之后，它跑一个专门**证伪自己**  
的结构化流程：找错误假设、找逻辑缺口、找会挡住攻击的安全控制；依赖没有支撑的假设的 finding 直接丢掉。  
  
它把"证伪"做成了硬规则。README 直接写：历史上约一半候选 finding 是误报；phase2b 开篇就是一句 "STOP. Before writing any report... you MUST complete this verification phase"，并把每个候选列成清单，最终裁决表的行数必须等于候选清单的行数——少一行就意味着悄悄漏掉了一个 finding。判定为 FALSE POSITIVE 必须引用具体的 file:line  
 作为消除证据，只写一句"这里是事务日志"不算证据。它还有一条 anti-merge 规则：攻击者认证等级、入口点/路由、网关或中间件上下文、被利用的参数，四项里只要有一项不同，就不许合并成一条 finding——同一端点不同参数的漏洞是独立漏洞，即便它们能串成一条攻击链。一条 finding 只有在**所有**  
调用点都被验证为不可利用时才能被降级，只查了一个点、还剩 4 个没查，不算清理干净。  
  
![vulnhunt/phases 目录真实截图：11 个阶段文件，recon → hunt → verify → reproduce → sweep → report](https://mmbiz.qpic.cn/sz_mmbiz_png/J2hBCjr4LfvrYWiaibNxBeHfDnMPBlgJiaZOOYJbJtqGONtNfxUmkNNXsOwkfSZ3IEOobFpZz8fSAYR2mBXhibIulZ7GDQHpEAMr2jelf3kskYQ/640?wx_fmt=png "vulnhunt/phases 目录真实截图：11 个阶段文件，recon → hunt → verify → reproduce → sweep → report")  
### 亮点三：Hunt → Fix → Verify，闭环但互相独立  
  
它由三个可以拼起来的 Claude Code skill 组成：  
  
/vulnhunt  
 负责找。SKILL.md 本身是**编排器**  
，它自己不做任何安全分析：建结果目录、按阶段派发 subagent、校验每个 subagent 的输出文件存在、最后汇总报告。把 finding 挡在编排器上下文之外是刻意的设计——逼出系统化方法论，而不是即兴分析。  
  
/vulnhunter-fix  
 负责修，走 TDD：先写 exploit demo，再写一个**会失败的**  
安全测试（RED），然后实现修复（GREEN），验证 exploit 被挡住、原测试不回归，最后开 PR。每个 finding 在 <repo>/.vulnhunter-fix/  
 下的独立 git worktree 里改，PR 之前要过六道机械闸：严重级掩码、正文完整度、范围、幂等、防合并、验证表。  
  
/vulnhunt-fix-verify  
 负责验，是一个**独立只读**  
的验证 skill：输入上一次扫描结果和开发者"已修好了"的声明，逐个 finding 输出裁决。README 那句写得很直接——开发者的话不是证据，代码才是。它的工具信封被压到极窄：只有 Read/Write/Edit/Glob/Grep/Agent，**没有 Bash、没有网络**  
，不能跑 shell、不能跑 exploit、不能 git  
，连目录都建不了（所以调用方必须先把 out  
 目录建好），只能在可信根目录内读、不 clone 不 fetch。它的 comment_rules.md  
 规定：开发者评论一律当"声明"读，绝不当指令执行。  
  
![vulnhunt/SKILL.md 真实截图：模型检查 STOP 与 MANDATORY FIRST ACTIONS](https://mmbiz.qpic.cn/mmbiz_png/J2hBCjr4LftqtkkjmV7DM9J6MIwOlWtMu6zNTgFaTPf2Dbcpxbkd86Riaaoo9Faernhg3kGhQCPzNAC6Ppv0r5ic6OLwrLtGxPoopdPSekEXU/640?wx_fmt=png "vulnhunt/SKILL.md 真实截图：模型检查 STOP 与 MANDATORY FIRST ACTIONS")  
### 亮点四：把"被扫的代码"当成不可信输入  
  
这是做 AI Agent 平台的人最该抄的一段。整套东西默认**只读**  
：/vulnhunt  
 是 prompt-only skill，无 Python、无网络。批量扫描的 harness 跑 claude -p  
 时限定：  
- --tools Read,Write,Edit,Grep,Glob,Agent  
——没有 Bash、WebFetch、WebSearch；  
  
- --permission-mode default  
，只有结果目录下的写入被预先批准，其他写入在 headless 模式下会被直接拒绝；  
  
- --setting-sources user  
——被扫仓库自带的 .claude/settings*.json  
 和里面的 hooks 永远不会被加载；  
  
- --strict-mcp-config  
——不加载任何 MCP 服务器或 claude.ai connector。  
  
也就是说：**目标仓库是不可信输入**  
。只有加 --execute  
 才会在 acceptEdits 下重新放开 Bash，好让 skill 装依赖、跑 exploit test，官方明确"只在你信任的代码上用"。  
### 亮点五：能进 CI 的工程化，而不只是一个 prompt  
  
vulnhunter-agent/  
 把扫描包成 headless 运行时：从 TOML 配置 + VULNHUNT_*  
 环境变量读设置（配置优先、环境变量覆盖），克隆目标仓库、跑 /vulnhunt  
、把结果目录推到独立仓库、每个确认 finding 开一个**去重后的 GitHub issue**  
，没有 finding 就发一张 "clean scan" 回执；还有 --mode=verify  
 驱动只读验证流程。产物有两份 JSON Schema 兜底：scan_manifest.schema.json  
 和 verify_disposition.schema.json  
。认证三种模式——api_key  
、bedrock_oauth  
、bedrock_sigv4  
，README 说认证是"唯一的收口点"，换环境不用重打镜像。  
  
harness/  
 是开发者工具：REPO_LIST.txt  
 一行一个仓库批量扫，benchmark 模式对着已知有洞的语料跑"克隆→扫描→LLM 评委→统计"，每步落盘可断点续跑，还能 analyze_misses  
 诊断漏报。官方自己标注这是 dev-only 基础设施，不是产品的一部分。  
### 句芒视角：它其实是一份 Agent 安全治理作业  
  
第一，它承认**发现可以并行、验证是瓶颈**  
：把"找洞"丢给多个并行 agent，把"验证/分诊/打补丁"当承重墙。这和 Anthropic 官方那份参考实现的判断一致——真正难的不是发现，是证明。  
  
第二，它的工程细节在回答一个问题：当你给一个能执行命令的 AI 放开一个仓库时，权限该开到哪。答案是全链路最小权限——只读默认、工具白名单、不加载目标仓库的 hooks 与 MCP、验证器连 out  
 目录都要调用方先建。  
  
第三，README 自己挂了一条 Cyber-safeguard 风险提示：VulnHunter 做的是双用途的网络安全工作，如果你在**没有加入 Anthropic Cyber Verification Program**  
 的账号上跑，实时网络防护可能拦你的请求，账号也可能被标记为滥用，官方建议先去验证门户登记。前沿厂商已经开始给"AI 攻防"上闸，这件事本身比工具更值得注意。  
### 上手  
```
git clone https://github.com/capitalone/vulnhunter.gitcd vulnhunter./install.sh        # 把三个 skill 拷到 ~/.claude/skills/claude --model opus \  --add-dir ~/.claude/skills/vulnhunt \  --add-dir ~/.claude/skills/vulnhunt/phases# 会话里：/vulnhunt
```  
  
install.sh  
 是**复制**  
而不是软链接——README 解释了原因：软链接会让 subagent 里的 find  
/glob  
 失效；改完任何 skill 文件都要重跑一次 install.sh  
 同步本地副本。  
  
无头跑（CI / 定时任务）：  
```
cd vulnhunter-agentpython -m pip install -e ".[dev]"export ANTHROPIC_API_KEY=sk-...python -m agent https://github.com/your-org/your-service --no-publish --no-issues
```  
  
批量与基准：  
```
cd harnesspython -m pip install -e ".[dev]"python -m local_harness.batch.run scanpython -m local_harness.benchmark.run
```  
### 避坑  
- 跑不动先看模型。SKILL.md 第一步是模型检查：交互式调用时如果不是 Opus 4.7 以上就 **STOP**  
，直接让你切 /model opus  
。官方说低误报纪律依赖前沿级推理能力，换模型可靠性下降。  
  
- 不加 --execute  
 是只读模式：PoC 和 exploit test 会被写出来，但不会真的执行。  
  
- 只审第一方生产代码：tests/  
、vendor/  
、node_modules/  
、generated/  
、*.md  
 全跳过；唯一的例外是安全相关的基础设施配置——比如 nginx 里 set_real_ip_from 0.0.0.0/0  
 把 remote_addr  
 变成可伪造，这在它的规则里算部署系统的漏洞，不算运维问题。  
  
- 零确认 finding 是合法结果，官方明确禁止为了凑输出而软化判定标准；理论上可能的攻击、已有下游缓解的模式、已修问题的弱变体，只能进"代码质量"章节。  
  
- 官方自己承认：自主分诊和自主打补丁尚未完全解决。  
  
- fork 模式要把 config.json  
 里 your-org  
 / your-fork-org  
 这些占位符改成自己的组织，否则 PR 不知道往哪开；默认 PR 是 draft。  
  
- benchmark 自带的语料是**合成示例**  
（映射到 OWASP Juice Shop / WebGoat / NodeGoat），要真做基准得自己按 ground_truth/<repo>.json  
 补语料。  
  
- 只扫你有明确授权扫的代码库。  
  
### 适合谁  
  
安全团队里被 SAST 误报淹过的代码审计同学、做 SRC/赏金想加一层"可证伪"过滤的人、以及正在给编码 Agent 划权限边界的 AI 平台团队——第三类人最该读它的 harness/README.md  
，那里面几乎是把"Agent 该拿到什么工具"逐条写清楚了。  
  
回到开头那两周。误报的根因不在规则库，在"谁来判断"。VulnHunter 的答案是把判断交给一个愿意反驳自己的 agent，再用一个只读的 verifier 独立复核一遍——开发者的话不是证据，代码才是。  
  
