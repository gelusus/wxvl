#  多 Agent Harness 如何把漏洞告警变成可复核证据  
原创 QAC
                    QAC  Qsec   2026-10-27 03:52  
  
# 多 Agent Harness 如何把漏洞告警变成可复核证据  
  
Ulysses 是论文提出的一套面向 Web 应用漏洞验证的多 Agent Harness，也就是编排多个 Agent 分工测试的工作框架。2026 年 10 月 22 日发布在 arXiv 的同名论文介绍了它的设计，并用 OWASP Benchmark 和 WebExploitGym 评估它生成可复核利用证明（Proof of Exploit，PoE）的能力。  
  
这套流程把规划、尝试、运行时效果核验和复盘交给不同组件。Effect Oracle 检查目标是否出现预设效果；只有核验通过，PoE Node 才生成可复现脚本。  
## 告警的证据边界  
  
扫描告警指出一段可疑代码，却未必证明攻击者能触达该路径，更不能说明目标已出现安全影响。LLM 自述“利用成功”，仍只是文字结论。  
  
Ulysses 把最终产物称为 Proof of Exploit，简称 PoE。论文也使用 Proof of Exploitability 这一说法。在这篇论文里，PoE 指**可复现的利用验证脚本，以及独立 Oracle 对目标运行效果的确认**  
。这里的“证明”对应可复现证据，不是数学形式化证明；它说明特定测试部署上的路径可达、预期效果发生，无法直接推出其他生产环境也可利用。  
<table><thead><tr><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">产物</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">能支持的判断</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">仍然缺少什么</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">静态扫描告警</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">有一条值得检查的代码路径</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">运行时能否触发、是否造成预期影响</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">模型自报成功</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">模型提出了一种解释或验证结果</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">独立证据；模型可能把推测当作事实</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Oracle 核验的 PoE</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">受控目标出现预设效果，且有脚本可复现</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不能自动证明其他部署也可利用；还需确认效果是否由所测漏洞造成</span></section></td></tr></tbody></table>## Harness 的角色分工  
  
Hypothesis Graph 串起规划、验证与复盘。  
  
![Ulysses 的多 Agent 验证闭环](https://mmbiz.qpic.cn/mmbiz_png/icVSvicxhfMd7a45bhvsNbUJhmII1HMebuCnm24R1Ut9F0TLlNP8U5bKdeuBo46vBnyo2ZVNlLBDVavTu96NUAoGTL8vGUBXS65qyZIfKUEib8/640?from=appmsg "")  
  
Ulysses 的多 Agent 验证闭环  
  
图 1｜从候选线索到 PoE 的循环。PoE Node 只有在 Oracle 核验通过后才会启动；流程图依据论文描述原创绘制。  
  
Planning Agent 先读取漏洞线索。输入可以来自静态扫描器、LLM 扫描器或已有 advisory；论文也测试了不提供 advisory 定位信息的条件。规划 Agent 将候选攻击路径拆成 Hypothesis Graph。每个节点代表一个待验证假设，边记录抵达下一节点所需的上下文与前置关系。  
  
Hypothesis Graph 把验证假设和必要的依赖关系明确存下来。每个 Exploit Node 只拿到所需上下文，不必携带所有历史对话。候选路径按置信度分批处理；一轮结束后，Planner 可以结合累计结果重建假设图，让多个分支并行测试，也能把前一轮留下的线索接到后续假设上。  
  
Ulysses 为每个假设节点创建一个 Exploit Agent。它只测试对应路径，可以调用工具与受控 Web 应用交互，不能自行把文本结论写成“已利用”。论文把节点标为“可利用”“存在漏洞但当前路径不能利用”或“良性”。每个 Agent 有轮次上限，整套流程也受总时间预算约束。  
  
Oracle 返回本轮结果后，Reflection Agent 汇总哪些效果已验证、哪些路径失败、哪些效果还没有覆盖，并将摘要交给下一轮 Planner。Reflection 负责整理过程，不负责裁决真假。  
<table><thead><tr><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">组件</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">主要职责</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">不负责的事</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Planning Agent</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">读取线索，构建或更新假设图</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不判定漏洞是否真实可利用</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Exploit Node / Agent</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">独立测试一个图节点中的假设</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不以自身文字回复作为成功证据</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Effect Oracle</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">比对目标侧效果与预置事实，返回通过或未通过</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不替 Agent 寻找利用路径</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">PoE Node</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Oracle 通过后生成可单独复现的验证脚本</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不决定 Oracle 是否通过</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Reflection Agent</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">汇总本轮成功、失败与未覆盖路径，反馈给 Planner</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">不负责批准 PoE</span></section></td></tr></tbody></table>  
Oracle 确认效果后，PoE Node 才会生成可独立运行的 Python 脚本。作者人工检查并重放了生成的 PoE，最终产物包含脚本与目标侧效果证据。  
## Oracle 的验证路径  
  
Ulysses 把 Agent、目标应用和 Oracle 的权限边界拆开。Agent 在 Host A 的 Docker 沙箱里运行；目标应用、Oracle 和效果监听器位于另一台主机。Agent 可以读取测试源码并通过 Web 接口访问目标，但无法直接读取目标数据库、文件系统或操作系统。  
  
Agent 可以向 Oracle 的提交接口送交待核验的证据，却不能读取 Oracle 保存的预置秘密，也不能直接连接效果监听器。目标应用一侧发生的效果由监听器接收，Oracle 再将它与预置事实比对，只向 Agent 返回匹配或不匹配。  
  
![Agent、目标应用与 Oracle 的验证边界](https://mmbiz.qpic.cn/mmbiz_png/icVSvicxhfMd4R0iaMnFv0NSo7a6TxFlxFzsW1YNFkXCTTLeLW1jAw7zZqbxwseN2FMPibcKgxBHibiaCeOCkMNv3icsOxYy5BjE0Hsx72FeyZ7jxU/640?from=appmsg "")  
  
Agent、目标应用与 Oracle 的验证边界  
  
图 2｜提交证据的 API 对 Agent 开放；预置事实和效果监听器不开放。目标侧信号由 Oracle 独立核对，图为依据论文威胁模型原创绘制。  
  
这套验证机制覆盖八类运行时效果。Oracle 为目标随机植入标记，Agent 无法靠猜测内容伪造成功。  
<table><thead><tr><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">效果类别</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">Oracle 核验的信号</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">未授权读取文件</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">提交内容是否匹配目标中的预置值</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">未授权创建文件</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">指定位置是否出现预期文件</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">数据库读取</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">返回值是否匹配预置数据行</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">数据库修改</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">目标数据行是否变为预设值</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">绕过认证</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">提交的会话令牌能否通过运行时校验</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">权限提升</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">该令牌能否访问管理员专属功能</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">数据外传</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">受控监听端是否收到目标侧信号</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">远程代码执行</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">受控监听端是否观察到预期回连效果</span></section></td></tr></tbody></table>  
Oracle 只判断“效果有没有发生”，不判断效果是否由待测 CVE 导致。论文称它 mechanism-blind，意思是只认结果，不辨成因。为减少错误归因，在 advisory-informed 和静态扫描器 seeded 条件下，系统还用目标路由、漏洞类型信号及认证前置条件做路径与利用方式校验。zero-knowledge 条件没有这道校验，因此不能把任意 Oracle 信号都归到某条已知漏洞上。  
## 实验结果  
  
论文构建了 WebExploitGym，包含 25 个已披露的 Web 漏洞实例，覆盖 Go、Java、JavaScript、PHP、Python 五种语言和七个框架。作者把受影响版本的应用部署到本地 Docker，并为每个样本预置 Oracle 可识别的状态。25 个 advisory 中，14 个此前已有公开 PoE，另 11 个没有。  
  
“zero-knowledge”只表示不给 Agent advisory 或漏洞定位信息。Agent 仍能读取测试源码并访问运行中的目标；这组实验测的是已披露漏洞上的验证能力，并非零日测试或未知生产系统的纯黑盒攻击。  
  
OWASP Benchmark Python 提供 1,230 个测试用例；论文选取 339 个可评估样本，其中 129 个为真实漏洞，覆盖六类效果。三组实验各测不同问题，结果不能合并成一个“总体成功率”。  
<table><thead><tr><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">实验问题</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">设置与主要结果</span></section></th><th style="border:1px solid #d0e1fa;padding:8px 16px;background:#eef3fd;font-weight:600"><section><span leaf="">正确解读</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">RQ1 验证误报</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">CodeQL 候选精确率 48%，LLM 检测器 Absentia 为 54%；Graph 自验证 42.2%，单轮 ReAct 自报 19.3%；Oracle 核验后的精确率为 1.000（100%）</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Oracle 确认效果后才计为阳性，所以 1.000 不代表整体准确率或召回率 100%；六类 CWE 中保留了五类召回</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">RQ2 比较架构</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">Oracle 开启、每条路径预算同为 300 秒时，图式规划 Graph 对单轮 ReAct 的召回率分别为 Kimi K2.5 56.6% 对 8.5%，MiniMax M2.5 55.0% 对 7.0%，Sonnet 5 56.6% 对 5.4%</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">这些数字只适用于论文的 OWASP 设置，说明该测试中的图式规划优于单轮 ReAct；结论不能外推到其他任务</span></section></td></tr><tr><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">RQ3 实测 Web 应用</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">提供漏洞公告的 advisory-informed 为 10/25；以 Semgrep 告警为线索的条件为 3/25；不给公告定位信息的 zero-knowledge 为 4/25</span></section></td><td style="border:1px solid #e0e9fa;padding:8px 16px"><section><span leaf="">10/25 只对应第一种条件，不是总体成功率；每个目标、每个条件只运行一次</span></section></td></tr></tbody></table>  
作者为四个此前没有公开 PoE 的已披露 advisory 补上了可工作的验证证据。新增的是四份 PoE，目标仍是已披露的 N-day 漏洞。作者人工检查并重放了生成结果。  
## 结果适用范围  
  
WebExploitGym 只测本地 Docker 部署的开源 Web 应用和已披露漏洞，Agent 可读取源码。论文覆盖服务端 Web 效果，未测试 XSS、拒绝服务、内存安全漏洞或非 Web 应用。  
  
每个 WebExploitGym 目标在每种条件下只运行一轮。作者人工复核并重放了全部生成 PoE，但没有因为成本再做多轮随机重复。因此，10/25、3/25、4/25 是这次实验条件下的观察值，不能当成稳定的产品成功率。  
  
Oracle-gated pipeline 的 precision 1.000 源于准入规则，系统只接受 Oracle 确认预设效果的 PoE。召回仍有限。Oracle 也不自动证明效果由指定 advisory 的漏洞造成；论文只在部分输入条件加入路由和向量约束，具体案例仍需人工复核。  
  
作者只在本地测试已披露的 N-day 开源应用，并与维护者协调补充 PoE，没有测试生产目标。论文承认 Harness 有双重用途，建议仅用于自有或授权系统。论文 v1 将代码和数据列为后续提供，本文按这一版本标注，不称它们已公开。  
## 安全流水线的落点  
  
Ulysses 把 Agent 的权限和裁决拆开。Planner 提出假设，Exploit Agent 负责尝试，Oracle 确认目标效果；PoE Node 整理已验证结果，生成可重放产物。每个环节都有明确的输入、权限和输出。  
  
对实际 AppSec 流程，有三项可以直接参考。扫描器告警先作为候选，经过独立验证后再报告为已确认漏洞。验证环境应与 Agent 隔离，用 Agent 无法伪造的信号确认影响。最终报告保存复现脚本、目标版本、测试前置条件和独立核验结果，方便另一位工程师复查。  
  
我更关注 Oracle 把成功判定放在 Agent 之外。目标端效果与可重放脚本给复核者留下证据；Oracle 仍无法独立还原因果路径，报告里要保留这层限制。  
## 参考  
- •  
Aditya Oke 等：《Ulysses: A Security Harness For Synthesizing Verifiable Web Exploits》（arXiv:2610.31831，2026-10-22）  
  
- •  
OWASP Foundation：《OWASP Benchmark Project》  
  
- •  
GitHub：《About CodeQL》  
  
- •  
Semgrep：《Semgrep Docs》  
  
  
  
