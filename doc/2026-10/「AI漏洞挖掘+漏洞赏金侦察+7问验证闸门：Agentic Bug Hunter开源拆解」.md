#  「AI漏洞挖掘+漏洞赏金侦察+7问验证闸门：Agentic Bug Hunter开源拆解」  
原创 句芒安全实验室
                    句芒安全实验室  句芒安全实验室   2026-10-15 03:39  
  
挖漏洞最耗时间的，从来不是「找」，是「证」。  
  
一个看着像漏洞的输入，你可能要花三四十分钟写复现、写报告，最后被平台一句 N/A 打回来。真正吃掉时间的，是那些「看起来能打、其实打不动」的弱发现。GitHub 上这个叫 Agentic Bug Hunter 的项目，就是冲着这件事来的：它把「侦察→狩猎→验证→报告」整条链路做成 AI Agent 流水线，中间卡一道「7 问验证闸门」，过不了闸门的发现直接杀掉，不让你浪费时间去写报告。仓库目前 5329 星、943 fork，被 Trendshift 收录，最近一次推送是 2026 年 10 月 9 日。  
### 它是什么：把整条挖洞链路塞进一个终端  
  
Agentic Bug Hunter 由 AwareXone 开源，MIT 许可，Python 3.10+。一句话定位：AI 驱动的漏洞赏金狩猎工具，从侦察到报告，全在终端里跑。  
  
它有两种跑法。一种作为 Claude Code 插件，用 /recon  
、/hunt  
、/validate  
、/report  
 这些斜杠命令驱动；另一种是完全独立的命令行 bughunter  
——作者专门强调，独立模式不需要 Claude Code，也不需要任何付费 AI 订阅，接一个本地模型就能跑。  
  
仓库结构我核过（GitHub 上真实文件）：skills/  
 下 19 个技能知识库（bug-bounty 主流程、bb-methodology 方法论、web2-recon、web2-vuln-classes、triage-validation、report-writing、web3-audit、llm-redteam、mcp-server-audit、cloud-pentest、cicd-security、graphql-audit 等）；bughunter/agents/  
 下 9 个专职 Agent；commands/  
 下三十多条斜杠命令；tools/  
 下约 35 个 Python/shell 扫描器；外加 memory/  
 跨会话记忆和 mcp/  
 MCP 服务器。  
  
![Agentic Bug Hunter 仓库官方 banner（awarexone/Agentic-Bug-Hunter 仓库真实素材）](https://mmbiz.qpic.cn/mmbiz_png/J2hBCjr4LfuhCRjvNOTuBljRvibemDaW6x2oNmLwlpA08rbqqfy21UgUjiamJicQ6kuBTXtddZohNujJfUPSNG3PhDbgYMs6kEePY37Zz7KrYM/640?wx_fmt=png "Agentic Bug Hunter 仓库官方 banner（awarexone/Agentic-Bug-Hunter 仓库真实素材）")  
### 亮点一：9 个专职 Agent，各干各的活  
  
README 里把 9 个 Agent 列得很清楚，是分开的，不是一个大模型从头扫到尾：recon-agent  
 负责子域枚举、存活探测、URL 爬取；validator  
 专跑 7 问验证闸门，专门杀弱发现；report-writer  
 写「以影响为先」的报告，目标是别被 N/A；chain-builder  
 是发现漏洞 A 之后，去找能跟 A 串联的漏洞 B、C；autopilot  
 是带安全检查点的全自动狩猎循环；recon-ranker  
 按「最有价值目标优先」给攻击面排序；web3-auditor  
 做智能合约 10 类漏洞审计；token-auditor  
 扫 meme 币/代币的 rug pull；credential-hunter  
 是字典生成→OSINT→泄露库比对→喷洒，注意它在「喷洒」之前有一个硬停。  
  
「一个 Agent 只干一件事」这种分工，跟「一个大模型从头扫到尾」是两种思路：前者的好处是每一步的输出都可以单独检查、单独换模型。  
### 亮点二：7 问验证闸门，把弱发现拦在提交之前  
  
这是它跟多数「AI 扫一遍给你一堆发现」的工具最不一样的地方。/validate  
 会跑一张 7 问清单：攻击者现在能真的做出来吗？需要大多数人没有的特殊权限吗？有真实影响（数据泄露、账号接管、资金损失）吗？是否已经被报过或披露过？在这个项目范围内吗？有清晰复现步骤吗？严重级别准确吗？  
  
过不了，工具会杀掉它，或者要求你降级。README 说得很直白：这样能避免你提交一堆被平台拒的弱发现、拉低信誉分。仓库里还有一条常开的规则：「30 秒检查省下几小时的无效报告」。对做过漏洞赏金的人来说，这条闸门其实是把老手脑子里的直觉，写成了可执行的清单。  
### 亮点三：DOM XSS 用无头浏览器「真跑」才算数  
  
CHANGELOG 里我比较关注的一条新增，是 /domxss  
 确认框架。做前端安全的人都知道，「反射」不等于「执行」——一个 payload 出现在响应里，不代表浏览器真的跑了它。这个命令用 Playwright 驱动无头 Chromium，往每个查询参数和 URL 片段参数注入带唯一标记的 canary payload，只有浏览器**真的执行了**  
（弹窗、命中钩子、控制台）才报 [CONFIRMED]  
；只是反射但被中和掉的，报 [POSSIBLE]  
。它还会截图留证。  
  
这一条把工具原本最容易「确认不足」的缺口——客户端/JS 执行类漏洞——从「疑似」变成「已证」。对一份要提交给平台的报告来说，一张执行成功的截图，比十句「可能存在」有用得多。  
### 亮点四：跨会话记忆 + AI 接口红队  
  
每次狩猎结束，工具会把「测了什么、找到什么」自动存到 ~/.claude/projects/[项目]/hunt-memory/  
。下次对同一目标，它会告诉你哪些已经测过；对技术栈相似的新目标，它会推荐之前有效的技术。/memory-gc  
 用来查看或轮换记忆文件（10MB 上限、3 个备份）。  
  
跟 AI 安全直接相关的是 /llm-redteam <endpoint>  
：用 LLM 红队语料打提示注入、越狱、数据外泄；/sast <path>  
 用 Semgrep 安全规则包扫拉下来的 JS/源码并按风险排序。这两个命令，跟句芒一直盯的 AI 安全方向是重合的。  
### 上手  
  
最快是 PyPI：  
```
pip install agentic-bug-hunterbughunter setup        # 选一个免费 AI providerbughunter recon target.combughunter hunt  target.combughunter validate "finding"bughunter report
```  
  
作为 Claude Code 插件：  
```
git clone https://github.com/Awarexone/Agentic-Bug-Hunter.gitcd Agentic-Bug-Hunterchmod +x install_tools.sh && ./install_tools.sh   # subfinder·httpx·nuclei·katana·ffufchmod +x install.sh && ./install.sh               # 装进 ~/.claude/claude/recon target.com/hunt target.com
```  
  
零成本全离线（Ollama 本地跑，下完模型后不需要联网）：  
```
curl -fsSL https://ollama.ai/install.sh | shollama pull qwen2.5:14b./install.sh --agent standalonebughunter setup --provider ollama --model qwen2.5:14bbughunter hunt target.com
```  
  
免费 provider 的自动探测顺序是：Ollama（本地免费）→ Groq → DeepSeek → …；付费的 Claude、OpenAI、Grok、OpenRouter、LiteLLM 也能接。输出写在 ~/.bughunter/  
。  
### 避坑  
- **AI 狩猎开箱即用，完整侦察要装外部工具**  
：/hunt  
、/validate  
、/report  
 这部分不需要外部工具；但完整侦察要用 subfinder、httpx、nuclei、katana、ffuf、nmap，靠仓库里的 install_tools.sh  
 装。缺工具会被跳过而不是报错——这点要注意，不然你以为扫了，其实没扫全。  
  
- **/hunt 是交互式，/autopilot 是全自动**  
：新手先用 /hunt  
（AI 建议、你逐步确认）；/autopilot  
 会自动跑完侦察→排序→测试→验证→起草报告，只在检查点停；想更保守用 /autopilot --paranoid  
（每个发现都停）。  
  
- **/autopilot 要看清安全设置**  
：README 自己写了「设置正确才安全」——它会真的对目标发起请求。  
  
- **只测授权范围**  
：仓库规则第一条就是「先读完整 scope」，只测项目授权的资产；/scope <asset>  
 可以在测之前检查域名/URL 是否在范围内。仓库自己写明「For authorized security testing only」。  
  
- **记忆是本地的**  
：存在 ~/.claude/projects/[项目]/hunt-memory/  
，换机器不会带走。  
  
- **只支持 macOS / Linux**  
：Windows 官方不支持，可能要 WSL。  
  
- **README 营销味偏重**  
：仓库 README 里有赞助商 banner 和合作方链接（Infistar、Sidrune 等），看的时候跳过即可；工具本身是 MIT 开源、能独立跑。  
  
- **版本/许可**  
：MIT，Python 3.10+；最新 release v6.0.0「Provider Freedom」（2026-08-21），PyPI 版本 6.0.2；仓库 2026-03-08 建、最新推送 2026-10-09，5329 星、943 fork、9 个未关 issue。  
  
### 适合谁  
- **漏洞赏金猎人**  
：新手有 AI 一步步告诉你测什么、为什么；老手拿到自动化的侦察和报告，把时间省下来找创意型漏洞。  
  
- **安全团队/渗透测试**  
：把侦察、扫描、验证、报告串成一条流水线，记忆跨会话。  
  
- **AI 安全研究者**  
：/llm-redteam  
、/domxss  
、/sast  
 直接对应 AI 接口和前端安全。  
  
- **想零成本上手的人**  
：Ollama 本地模型免费，独立模式不需要任何订阅。  
  
一句话总结：它把挖洞这件事里最容易被忽略的一环——「验证」——做成了流水线里的硬闸门，而不是留给你手工兜底。  
  
