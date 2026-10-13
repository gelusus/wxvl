#  渗透测试从业者看过来：AutoCVE 一键挖 CVE，7天 30 个漏洞  
宝十八
                    宝十八  网络安全老宋   2026-10-13 04:00  
  
**导语：**  
 你好，我是网络安全老宋。  
安全攻防干货准时送达！  
  
网络安全  
老宋  
// 工具实测 · AI安全  
  
// 工具实测 · AI安全  
# 渗透测试从业者看过来：AutoCVE 一键挖 CVE，7天 30 个漏洞  
  
把挖洞做成流水线：Multi-Agent 协同审计，筛项目、审源码、验漏洞、出报告，再聊聊 AI 挖洞的真实水平。  
  
AutoCVE  
漏洞挖掘  
Multi-Agent  
开源  
  
  
![封面：AI 智能体自动挖洞](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flic7MDGVqZfujYk0iaekpDmjBXHjibSp5h8ZlqS7sr8TyUibTNEBiag4ECJJ7kSia8Qxw1RHlGXjD6wtnN0JhEkic3iaUQFclPhU77IgLI/640?wx_fmt=png&from=appmsg "")  
  
🔑 一句话精华  
  
工具能替你跑流程、写报告，但挖没挖到真漏洞，最后还得你拍板——AI 是加速器，不是免责金牌。  
  
## 01它是什么：把"挖洞流水线"做成了开源平台  
  
  
AutoCVE 的 GitHub 地址是 github.com/larlarua/AutoCVE，国人作者 larlarua 开发，AGPL-3.0 协议，GitHub 星标已经破千。技术上它不神秘，就是一套标准的 AI 原生应用：FastAPI 做后端、React 做前端、PostgreSQL 存审计轨迹和漏洞、Redis 管任务，外加一个沙箱跑动态验证。  
  
真正有意思的是它的"打法"。传统 SAST 工具（Semgrep、Bandit 之类）只会按规则扫一遍，扫完给你一堆可能对的报警。AutoCVE 想做的是往上叠一层智能体编排——你丢一个仓库进去，它自己决定先测绘、再扫描、再深挖、最后出报告，全程不用你守着。  
  
项目方在 README 里挂了一份可验证的战绩：为期一周的测试里，发现并提交了  
 30 个安全漏洞  
，覆盖 14 个开源项目，类型涵盖 SSRF、RCE、SQL 注入、XSS、CSRF、访问控制绕过和敏感信息泄露，最高 CVSS 9.9（严重级），每一条都附带官方 CVE 记录链接和完整报告。注意，这是开源项目的负责任披露，不是拿未授权系统练手。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flicm7gwE4ephFibb3IolZVnjSoYM9Elug4EGiasv4QHsSaX4TicNeL1ezxtPm1Hp4vvwpha5F3MrRsYZJfrD9D9zoFel1uJrxSDqns/640?wx_fmt=png&from=appmsg "")  
  
## 02工作机制：Multi-Agent 协同审计  
  
  
整个系统是一个多智能体的 ReAct 循环，工作流长这样：  
  
AutoCVE · Multi-Agent 工作流  
  
Orchestrator 总调度  
  
→ Recon（信息收集 · 测绘）  
  
→ Scan（工具扫描）  
  
→ Triage（误报过滤）  
  
→ Finding（深挖 0Day / CVE，核心）  
  
→ Verification（动态验证 · 漏洞确认）  
  
→ Merge / Finalize → CVE 报告  
  
把"谁先干、干完交给谁"这条路走顺，才是它的价值。三种审计模式对应不同目标：增强扫描  
（Scan→Triage，快速过一遍工具结果、滤误报）、智能审计  
（Finding，专门深挖高价值漏洞，适合 CVE 和 0Day 研究）、综合审计  
（Scan→Triage+Finding，全量铺开）。  
  
![多智能体协同：总调度分流测绘、扫描、深挖、验证](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93fl9bibXzbcTlhDy7YZC7MWbPficFcl1CAcbgKP8iadQlp6HE046lZKcPHFehU7nnKicCCb3Tt3Nl7hHibHVXDZv6ick9m1YJYSibnuY2fQ/640?wx_fmt=png&from=appmsg "")  
  
还有一点很实用：支持给不同 Agent 配置专属 Skills，等于你把自己的审计经验写成规则塞进去，工具面就能扩。审计过程也能可视化——Agent Tree、工具调用、阶段进度都能看到，不是黑盒。  
  
## 03战绩之外，得说点实话  
  
  
30 个 CVE、7 天、最高 9.9，这数字很炸。但老宋得泼盆冷水：AI 挖漏洞这事，业界 benchmark 的真实水平没那么神话。  
  
几个硬数据摆这：学术测试 CVE-Bench 里，最强的智能体在零日场景下只解出 13%、一日漏洞 25%；CyberGym-E2E 这种端到端测试，智能体整体成功率只有 7.6%，而"只做打补丁"的基线高达 82.3%——说明从发现到利用的全链路里，漏洞发现才是最大瓶颈  
。GPT-5 在 CVEFixes 上 F1 到了 62.4%，但一旦碰到没见过的项目，性能会掉 65% 到 80%，这就是所谓的"泛化鸿沟"。连 KDD 2026 的 OmniVul 评测都显示，主流模型在"识别漏洞代码"上的 top-1 准确率平均不到 50%。  
  
⚠️ 注意：把 AI 输出当结论直接用，是这块最危险的坑。误报会浪费你时间，漏报会让你以为"安全了"。  
  
换句话说，AutoCVE 的 30 个漏洞，大概率是在它擅长的中低复杂度开源项目上跑出来的，而且每一条都得人工复核。项目方自己也承认：生成的报告必须人工审、独立验证后才能提交 CVE。  
  
![AI 输出必须过人工复核，才能确认漏洞](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93fl9h9Exs8BQ4MczTCSRhSQKpjhnREL80dDViaVskibfxNJ7k5L04CXyFh67qwVoycUS3iaS3FKDk69E58WF99YSz7tsT5MVliamPzSg/640?wx_fmt=png&from=appmsg "")  
  
## 04同类横评：三条路线，各走一边  
  
  
AutoCVE 不是独一份。把几个摆一起看，选型就清楚了：  
<table><tbody><tr><td data-colwidth="103" style="padding: 9px 12px;font-size: 14px;line-height: 25px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><section><span leaf="">维度</span></section></td><td style="padding: 9px 12px;font-size: 14px;line-height: 25px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><section><span leaf="">AutoCVE</span></section></td><td style="padding: 9px 12px;font-size: 14px;line-height: 25px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><section><span leaf="">Mandiant AVDH</span></section></td><td style="padding: 9px 12px;font-size: 14px;line-height: 25px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><section><span leaf="">DeepAudit</span></section></td></tr><tr><td data-colwidth="103" style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">形态</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">开源自助平台</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">谷歌内部商用框架</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">开源多智能体审计</span></section></td></tr><tr><td data-colwidth="103" style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">定位</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">挖 CVE / 0Day 研究</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">事件响应·代码审查</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">企业代码安全审计</span></section></td></tr><tr><td data-colwidth="103" style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">人机协同</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">人工复核报告</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">强制人工验 PoC</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">人工追问补证</span></section></td></tr><tr><td data-colwidth="103" style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">战绩</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">7天 30 CVE</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">2天 100+ 高危·12 CVE</span></section></td><td style="padding: 8px 12px;font-size: 14px;line-height: 25px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><section><span leaf="">沙箱自动 PoC 验证</span></section></td></tr></tbody></table>  
Mandiant 的 AVDH 最猛：在被盗企业源码里两天揪出 100 多个高危真漏洞，跑了十个月、扫了数千万行代码，产出 12 个 CVE（含 CVE-2026-13242、CVE-2026-55803），但它不对外卖，是谷歌内部框架。DeepAudit 是另一类，重点在微服务架构加树状 AST 切片再加沙箱自动验证 PoC。一句话：想自己搭一套挖洞流水线、成本可控、数据不出域，选 AutoCVE；企业级内部能力买不到；DeepAudit 偏企业代码审计。  
  
## 05老宋的实话：能不能用，怎么用  
  
  
先说结论：能用，但别神化，更别越界。  
  
🔴 授权第一。  
AutoCVE 是负责任披露工具，只对自己有权审计的代码（自有项目、获授权的渗透、开源项目提交 CVE）用。拿它扫未授权目标，跟用别的一键工具一样，是违法红线，别碰。  
  
🟠 协议坑。  
AGPL-3.0 很硬：你把它当网络服务改了再部署，必须把改动开源。想做闭源商业化，先找法务过一遍。  
  
🟡 成本与依赖。  
它深度绑 LLM API——额度、费用、延迟、掉线都得管；部署还要 Docker 加 PostgreSQL 加 Redis，不是点开即用的小脚本。  
  
✅ 谁适合。  
漏洞赏金猎人、做负责任披露的研究者、企业内部代码审计团队。它真正的价值是把"发现到报告"的脏活压缩掉，让你把精力放在验证和深挖上。  
  
💬 老宋说  
  
这件事的本质，不是又出了个"一键出洞神器"，而是安全圈终于把老师傅的审计思路，从某个人的脑子里搬进了可复用的 Agent 工作流。AutoCVE、Mandiant AVDH、DeepAudit 走的是同一条路：让 AI 跑流程、人做决策。  
  
对读者来说，现在最该做的不是囤工具，而是挑一个能驾驭的，把你的审计经验写进 Skills、把人工复核变成铁律。记住，再神的自动化也替不了那句"我验证过了"——没授权、没复核，再快也是空。  
  
```
项目地址：https://github.com/larlarua/AutoCVE
docker 安装sudo usermod -aG docker $USER
curl -fsSL https://raw.githubusercontent.com/larlarua/AutoCVE/v1.0.0/docker-compose.prod.cn.yml | docker compose -f - up -dhttp://192.168.23.137:3000
邮箱：demo@example.com
密码：demo123
```  
  
推荐阅读  
  
这几篇相关的，建议一并看看：  
  
1. [渗透测试从业者：把AI智能体接进Burp 数据包分析交给它](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487199&idx=1&sn=8753581ca8cbda3b25dcdfbe6f0dce40&scene=21#wechat_redirect)  
  
  
2. [乙方安全工程师开心了：长亭MonkeyCode开源 代码审计+私有化](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487197&idx=1&sn=077725bcf727c00911a9f24763538a47&scene=21#wechat_redirect)  
  
  
3. [腾讯开源 AI 红队平台 1900+ CVE 一键扫完](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487273&idx=1&sn=6ebf3f8ebb974534207ba27480a07761&scene=21#wechat_redirect)  
  
  
4. [乙方安全工程师的新武器： AI解CTF已能挖出真实0day](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487592&idx=3&sn=51e6cde01f929e2094f33f08d9aecf8e&scene=21#wechat_redirect)  
  
  
🔐 我是网络安全老宋，专注把安全威胁翻译成你听得懂的实操建议。觉得有用，点个在看，转发给身边做安全的朋友。  
  
防御，不是在演练期间发现攻击，而是在演练开始前就把攻击面收敛到最小。  
  
end  
  
  
  
不想错过文章内容？读完请点一下**“在看**  
**”**  
，加个**“****关注”**  
，您的支持  
是我创作的动力  
  
期待您的一键三连支持（点赞、在看、分享~）  
  
