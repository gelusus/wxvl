#  审代码还自动写 PoC，挖出 49 个 CVE 的 AI 审计平台开源  
原创 大白
                    大白  知白守黑1024   2026-10-21 22:16  
  
MULTI-AGENT · CODE AUDIT · SANDBOX POC  
  
审代码还自动写 PoC  
  
挖出 49 个 CVE 的 AI 审计平台开源  
  
DeepAudit · 四智能体协作 / 沙箱验证 / RAG 增强 / 可本地部署  
  
先看一组战绩：**DataEase 9.8 分 JNDI 注入**  
、**JimuReport 9.8 分反序列化**  
、**禅道 9.1 分提权**  
、**xxl-job、mall、eladmin**  
这些 GitHub 上几万星的国产明星项目，都在它手里栽过跟头——累计**49 个 CVE 编号 + 6 个 GHSA 安全公告**  
，横跨 17 个知名开源项目。  
  
挖出这些洞的不是某个大厂安全团队，而是一个**Multi-Agent 代码审计平台**  
：让一队 AI 智能体模拟安全专家的完整工作流——读代码、找漏洞、**写攻击脚本、扔进沙箱验证**  
，最后输出一份带 CVSS 评分的专业审计报告。这就是今天的主角**DeepAudit**  
，v3 版本已开源，7.2k Star，AGPL-3.0 协议。  
  
![DeepAudit 实战演示](https://mmbiz.qpic.cn/mmbiz_gif/j7ZnQr1RD1js2pplkicaoqiaLsEia8rS9lDE46EjgDtdTQrQDxfJQbxcwkUqfKpf5zXBe53UH5AgfhNfoAiagu5CIH9dMj67VZt1B90ibwrednrE/640?wx_fmt=gif&from=appmsg "")  
  
导入项目 → Agent 审计 → 沙箱验证 → 报告导出全流程  
  
先花三十秒认识它  
  
$ whoami  
  
▸  
DeepAudit — Multi-Agent 代码安全审计平台（v3）  
  
▸  
底座：FastAPI + LangGraph · 前端 React + TS · 协议 AGPL-3.0  
  
▸  
四智能体：Orchestrator / Recon / Analysis / Verification  
  
▸  
杀手锏：Docker 沙箱跑 PoC，验证不过的漏洞直接剔除  
  
▸  
模型：GPT / Claude / DeepSeek / Qwen / GLM / Kimi / Ollama  
  
$ 战绩: 49 CVE · 6 GHSA · 17 个知名开源项目 ▊  
  
它不是"又一个扫描器"  
  
用过传统 SAST 工具的都知道三大痛：**误报率高**  
——不懂语义，几千条告警里九成是噪音；**业务逻辑盲**  
——跨文件调用、复杂业务流看不明白；**没有验证手段**  
——报完就完了，洞到底能不能利用没人知道。  
  
DeepAudit 的回答是：**别做规则匹配了，让 AI 把安全专家的审计流程完整演一遍**  
。官方那句口号说得挺准——**"让 AI 像黑客一样攻击，像专家一样防御"**  
。你只管导入项目（GitHub / GitLab / Gitea 一键拉取，或者传 ZIP），后面全自动：识别技术栈 → 分析风险 → 生成脚本 → 沙箱验证 → 出报告。  
  
![DeepAudit Agent 审计入口](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1iaj5IXOIMCXPPSYI5twXZhViapPSwtjLNgmgdSt3qm1bGdvm0EIRVNjpCiasxQPKib5ciajRXk1cDKPicPewliaoMDu0og7r6xlpsDY0/640?wx_fmt=jpeg&from=appmsg "")  
  
Agent 审计入口：选择项目与审计模式，一键进入 Multi-Agent 深度审计  
  
四个 Agent，一条审计流水线  
  
打开源码目录，backend/app/agents/  
下四个文件就是这个平台的大脑，分工像一支正规审计小队：  
  
<table><tbody><tr style="background: rgb(238, 240, 251);"><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">阶段</span></section></th><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">负责 Agent</span></section></th><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">干什么</span></section></th></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;color: rgb(51, 51, 61);"><strong><span leaf="">① 策略规划</span></strong></td><td style="padding: 10px 12px;color: rgb(61, 78, 214);"><strong><span leaf="">Orchestrator</span></strong></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">分析项目类型，制定审计计划，给下属派活</span></section></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);background: rgb(247, 248, 253);"><td style="padding: 10px 12px;"><strong><span leaf="">② 信息收集</span></strong></td><td style="padding: 10px 12px;color: rgb(61, 78, 214);"><strong><span leaf="">Recon</span></strong></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">扫结构、识别框架/API、提取攻击面入口</span></section></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;"><strong><span leaf="">③ 漏洞挖掘</span></strong></td><td style="padding: 10px 12px;color: rgb(61, 78, 214);"><strong><span leaf="">Analysis</span></strong></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">RAG 知识库 + AST 分析，深挖可疑代码</span></section></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);background: rgb(247, 248, 253);"><td style="padding: 10px 12px;"><strong><span leaf="">④ PoC 验证</span></strong></td><td style="padding: 10px 12px;color: rgb(61, 78, 214);"><strong><span leaf="">Verification</span></strong></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">写攻击脚本，Docker 沙箱实跑，失败自我修正</span></section></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;"><strong><span leaf="">⑤ 报告</span></strong></td><td style="padding: 10px 12px;color: rgb(61, 78, 214);"><strong><span leaf="">Orchestrator</span></strong></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">汇总发现，剔除误报，生成专业报告</span></section></td></tr></tbody></table>  
  
![DeepAudit 系统架构图](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1hGtjKaSgLcSCF8p240BKKEhW670bbfEhuiaIM2MR7TxwNygCxd4VjtzCViaTyWsqMOuNs2s81LXryarI2icj36icEGtMVFj6xMtls/640?wx_fmt=jpeg&from=appmsg "")  
  
官方架构图：Multi-Agent 引擎驱动，RAG 知识库与沙箱执行环境支撑（点击放大）  
  
灵魂功能：报漏洞之前，先打一遍给你看  
  
这是 DeepAudit 和所有传统工具拉开差距的地方。普通扫描器的输出是**"这里可能有问题"**  
；DeepAudit 的 Verification Agent 干的是**"我写个攻击脚本打给你看"**  
——针对每个疑似漏洞生成 PoC，扔进**Docker 隔离沙箱**  
里真实执行：  
  
**验证成功**  
→ 漏洞坐实，攻击路径和证据一起写进报告，附带 CVSS 评分；  
  
**验证失败**  
→ Agent 自我修正：换思路、改 payload、重写脚本再来，反复尝试后仍打不通的，**按误报直接剔除**  
，不进最终报告。  
  
等于用"实弹演习"替代"纸上谈兵"，误报率被机制性摁住。审计过程全程可视：**审计流日志实时展示每个 Agent 的思考与执行**  
——它在看哪个文件、为什么可疑、PoC 第几版跑通了，像看一个安全专家直播工作。  
  
![DeepAudit 审计流日志](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1grWJxC26SUpncicwncNsPia5SPibD43LRVgGWhurscFfUXragdqXkCgib3rdtAY8HRNKAq8ga2OBo5c4JU5rsOydgKYG5tk1wib0zM/640?wx_fmt=jpeg&from=appmsg "")  
  
审计流日志：实时查看 Agent 思考与执行过程，验证环节全程透明  
  
战绩单：49 个 CVE 的含金量  
  
空谈架构没意思，直接看战果。这里必须说清楚：**这批 CVE 是 DeepAudit 团队用其闭源内测版挖的**  
，开源版与闭源版同架构，能力以官方演示为准。挑几个 9 分以上的：  
  
<table><tbody><tr style="background: rgb(238, 240, 251);"><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">受影响项目</span></section></th><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">漏洞类型</span></section></th><th style="padding: 10px 12px;text-align: left;color: rgb(43, 47, 85);"><section><span leaf="">CVSS</span></section></th></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;color: rgb(51, 51, 61);"><section><span leaf="">DataEase（36k+ Star）</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">JNDI 注入 / SSRF ×3</span></section></td><td style="padding: 10px 12px;color: rgb(214, 69, 69);"><strong><span leaf="">9.8</span></strong></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);background: rgb(247, 248, 253);"><td style="padding: 10px 12px;"><section><span leaf="">JimuReport / H2O-3</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">反序列化 ×4</span></section></td><td style="padding: 10px 12px;color: rgb(214, 69, 69);"><strong><span leaf="">9.8 / 6.5</span></strong></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;"><section><span leaf="">Litemall</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">硬编码凭证</span></section></td><td style="padding: 10px 12px;color: rgb(214, 69, 69);"><strong><span leaf="">9.8</span></strong></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);background: rgb(247, 248, 253);"><td style="padding: 10px 12px;"><section><span leaf="">禅道 ZentaoPMS</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">提权 / SSRF ×2</span></section></td><td style="padding: 10px 12px;color: rgb(214, 69, 69);"><strong><span leaf="">9.1</span></strong></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);"><td style="padding: 10px 12px;"><section><span leaf="">OpenClaw（6 个 GHSA）</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">签名绕过 / RCE / 凭证泄露</span></section></td><td style="padding: 10px 12px;color: rgb(224, 120, 48);"><strong><span leaf="">8.6 / 8.5</span></strong></td></tr><tr style="border-top: 1px solid rgb(232, 234, 246);background: rgb(247, 248, 253);"><td style="padding: 10px 12px;"><section><span leaf="">jshERP / xxl-job / mall / eladmin…</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">提权 / 注入 / 信息泄露 等 30+</span></section></td><td style="padding: 10px 12px;color: rgb(85, 90, 110);"><section><span leaf="">3.7–8.8</span></section></td></tr></tbody></table>  
  
一份完整的 CVE 清单在仓库CVEList.md  
里，每条都带 NVD 编号可查。对国产开源项目的覆盖尤其狠——O2oa 一个项目就交了 16 个 XSS。挖洞的还是团队两个人，这产出效率值得玩味。  
  
![DeepAudit 审计报告](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1iaX9OlMx1Bic7LticOGmvAZuFZYaaVu2TdCOIOG6pR1pxA2BLUGPeVXhVWibBTnAG1Pl9oUqP7rkdibAObr2aVZgQvQsbxGMeCPQL4/640?wx_fmt=jpeg&from=appmsg "")  
  
审计报告：漏洞详情 + 修复建议，一键导出 PDF / Markdown / JSON  
  
技术底座拆解  
  
**编排层：LangChain + LangGraph。**  
四个 Agent 的协作状态机用 LangGraph 驱动，LiteLLM 做统一模型网关——换模型只改配置，OpenAI / Claude / Gemini / DeepSeek / 通义 / 智谱 / Kimi / 文心 / 豆包都能接，还支持 API 中转站。  
  
**知识层：RAG + AST。**  
ChromaDB 向量库存 CWE/CVE 知识库，Tree-sitter 做多语言语法树解析——Analysis Agent 找漏洞时不是靠运气，是"这个模式在历史 CVE 里出现过 + 语法树确认数据流可达"的双保险。  
  
**验证层：Docker 沙箱。**  
专门的沙箱镜像隔离执行 PoC，打不通不罢休；路线图显示沙箱正从 function_call 迁移到稳定 MCP 服务。  
  
**检测面：12 类漏洞。**  
SQL 注入、XSS、命令注入、路径遍历、SSRF、XXE、反序列化、硬编码密钥、弱加密、认证/授权绕过、IDOR；外加 Bug / 安全 / 性能 / 风格 / 可维护性五维检测和 OWASP Top 10 规则集，报告按 What-Why-How 组织——在哪、为什么、怎么修。  
  
三步跑起来  
  
官方预构建了 Docker 镜像，不用克隆代码，一行命令启动（国内用南京大学镜像站加速版）：  
  
# 国际版  
  
curl -fsSL https://raw.githubusercontent.com/lintsinghua/DeepAudit/v3.0.0/docker-compose.prod.yml | docker compose -f - up -d  
  
# 国内加速版（南京大学 GHCR 镜像）  
  
curl -fsSL https://raw.githubusercontent.com/lintsinghua/DeepAudit/v3.0.0/docker-compose.prod.cn.yml | docker compose -f - up -d  
  
# 启动后访问  
  
http://localhost:3000 ▊  
  
进浏览器先配模型（运行时配置、不用重启服务）：接云模型 API Key，或者填**Ollama 地址走全本地**  
——Llama3、Qwen2.5、DeepSeek-Coder 都行，**代码不出内网**  
，这是给合规敏感企业准备的路。项目导入支持 GitHub / GitLab / Gitea，也吃 ZIP 包，10+ 语言通吃。  
  
![DeepAudit 项目管理](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1gdgOU4mMzXw6uWZhv2gOicqTLiaUQgqadeiawTymuThJm4xaHpJSZN22UXLSKL4KMC2QiayI2VpJ6MOS0WhOYCicJelnnw3NZ2fibok/640?wx_fmt=jpeg&from=appmsg "")  
  
项目管理：GitHub / GitLab / Gitea 一键导入，多项目协同与安全态势仪表盘  
  
泼点冷水：上手前想清楚这几件事  
  
**开源版 ≠ 战绩版。**  
49 个 CVE 是闭源内测版的成绩，开源版同架构但能力差异没公开量化，预期放理性点。  
  
**AGPL-3.0 有传染性。**  
个人学习和企业内部审计没问题，但要是二开后对外提供服务，源码开放义务会找上门，商用前找法务过一眼。  
  
**代码出境问题。**  
接云模型意味着你的源码会发到模型服务商——核心代码请务必走 Ollama 本地模式，官方文档也反复强调这点。  
  
**Token 消耗不小。**  
多 Agent 反复推理加沙箱重试，一个大项目审下来 API 账单不会好看，日常建议小步快跑：先审增量代码和高危模块。  
  
**合规红线。**  
只能测自己的或已授权的项目，这既是项目声明也是法律底线。  
> 一句话总结：**DeepAudit 把"审计报告可信"这个稀缺品做成了自动化流水线**  
——沙箱 PoC 验证是同类工具里少见的硬功夫，RAG + AST 的组合拳也透着学术团队的严谨。对企业安全部、外包验收和白帽研究者，这都是一个值得部署的工具。  
  
  
适合谁  
  
**企业安全部**  
——外包项目交付前跑一遍，PoC 验证过的漏洞直接进整改单，扯皮空间为零；**研发团队**  
——发版前自检，五维检测连代码风格都管；**白帽与赏金猎人**  
——SRC 项目批量初筛的高效放大器；**安全研究者**  
——Multi-Agent + 沙箱验证这套架构，本身就是 AI 安全工程的优质学习样本。  
  
这个项目完全免费开源，7.2k Star 持续迭代中。  
  
觉得有用的话，**点赞、在看、转发**  
三连，  
  
帮好工具找到更多需要它的人。  
  
项目地址与部署文档已整理  
  
公众号后台回复关键字  
  
20261022  
  
自动获取下载方式  · 建议收藏本文  
  
