#  AI 智能体漏洞挖掘：让 Agent 跑通一个 SQLi 到 RCE 的复盘  
原创 Sink
                    Sink  船山信安   2026-11-01 16:00  
  
AI安全 · AGENT  
  
AI 智能体漏洞挖掘：让 Agent 跑通一个 SQLi 到 RCE 的复盘  
  
  
我不信「AI 一键打穿」，但让 Agent 做侦察、初筛、初步验证，确实省下大量机械时间。  
  
这篇复盘一次授权范围内的 SQLi 到 RCE 路径：Agent 做了什么，人做了什么，卡点卡在哪。  
  
案例基于一个已公开且已修复的历史 CVE 作教学演示，所有动作在授权测试环境内进行；不提供针对未修复系统的利用链。  
  
01  
01 分工：Agent 做什么，人做什么  
  
我的原则：Agent 负责「重复、可描述、可验证」的活——列参数、跑指纹、比对版本；人负责「判断可利用性、决定下一步、守住边界」。Agent 不帮人做越界的事。  
<table><tbody><tr style="background:#eef4ff;"><td style="padding: 10px 12px; border: 1px solid rgb(212, 227, 255); font-weight: 700; color: rgb(37, 99, 235);"><section><span leaf="">环节</span></section></td><td style="padding: 10px 12px; border: 1px solid rgb(212, 227, 255); font-weight: 700; color: rgb(37, 99, 235);"><section><span leaf="">Agent</span></section></td><td style="padding: 10px 12px; border: 1px solid rgb(212, 227, 255); font-weight: 700; color: rgb(37, 99, 235);"><section><span leaf="">人</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">侦察</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">爬参数、列接口</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">确认授权范围</span></section></td></tr><tr style="background:#eef4ff;"><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">初筛</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">比 CVE、标疑似</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">判断可利用性</span></section></td></tr><tr><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">验证</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">构造请求、看回显</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">确认影响、写报告</span></section></td></tr><tr style="background:#eef4ff;"><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">收口</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">整理证据</span></section></td><td style="padding:10px 12px;border:1px solid #d4e3ff;"><section><span leaf="">守住不越界</span></section></td></tr></tbody></table>  
把分工写清楚，是为了可逆：任何一步 Agent 出了偏差，人能立刻接管，不会「模型自己跑远了」。  
  
02  
02 第一步：让 Agent 把请求参数全列出来  
  
把抓到的请求喂给 Agent，让它输出「每个参数的位置、类型、是否进数据库」。这一步纯体力，Agent 比人快。  
  
# 给 Agent 的输入（脱敏）  
  
POST /api/order/list HTTP/1.1  
  
Content-Type: application/json  
  
{"page":1,"size":20,"status":"PAID","sort":"id desc"}  
  
   
  
# Agent 输出：status / sort 疑似拼接进查询  
  
03  
03 第二步：Agent 跑指纹，人来判断  
  
Agent 根据响应特征识别出后端是某框架的特定版本，并比对出该版本存在的一个已修复 SQL 注入 CVE。但 Agent 只给了「疑似」，没下结论——这正是对的。  
  
人要做的是：确认该 CVE 在目标版本上是否真的可达、注入点是否在授权测试的功能里、回显条件是否满足。这一步没有任何工具能替你拍板。  
  
04  
04 卡点：Agent 把盲注当成误报  
  
关键的卡点来了：这是个盲注，正常响应和注入响应状态码一样，只有时间差。Agent 第一次判定为「无差异 = 误报」。人拉回来，加了时间延迟观测，才确认注入确实存在。  
  
Agent 会按「明显差异」判断，而很多真实漏洞恰恰藏在「不明显的差异」里。人对异常的直觉，是 Agent 目前补不上的部分。  
  
这也解释了为什么纯自动化工具有天花板：它擅长「有差异」，不擅长「差异很弱」。弱信号恰恰是实战里最常见的情况。  
  
05  
05 收口：确认影响，写报告，不越界  
  
验证完注入影响的数据库范围后，立即停止，不读取无关数据、不写任何利用脚本到生产、不尝试提权。报告里写清：触发条件、影响范围、修复建议、复测方法。  
  
关于「到 RCE」：在这个已修复 CVE 的教学链路里，SQLi 配合特定的文件写权限可形成 RCE，但前提是那几个前提条件同时成立——这恰恰说明，绝大多数环境靠基础加固就能断掉这条链。  
  
给团队的建议：把「Agent 标疑似 → 人确认 → 只读到影响边界」当成固定流程，既保留效率，也不放任一脚踩进红线。  
  
参考来源  
  
OWASP Top 10 — A03 Injection  
  
MITRE CWE-89 — SQL Injection  
  
该历史 CVE 厂商安全公告与修复版本（教学演示用，已修复）  
  
本文为 AI 智能体辅助漏洞挖掘的方法论复盘与合规边界说明。案例基于已公开且已修复的历史漏洞作教学演示，不含针对未修复系统的完整攻击链或利用代码。  
  
