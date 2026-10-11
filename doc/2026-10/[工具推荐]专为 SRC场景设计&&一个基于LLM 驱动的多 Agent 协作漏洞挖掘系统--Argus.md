#  [工具推荐]专为 SRC场景设计&&一个基于LLM 驱动的多 Agent 协作漏洞挖掘系统--Argus  
R0x7e
                    R0x7e  陌笙不太懂安全   2026-10-11 09:00  
  
免责声明  
```
由于传播、利用本公众号所提供的信息而造成
的任何直接或者间接的后果及损失，均由使用
者本人负责，公众号陌笙不太懂安全及作者不
为此承担任何责任，一旦造成后果请自行承担！
如有侵权烦请告知，我们会立即删除并致歉，谢谢！
```  
  
工具简介  
  
**Argus**  
 是一个基于 LLM 驱动的多 Agent 协作漏洞挖掘系统，专为 SRC（安全应急响应中心）场景设计。系统采用   
**LATS（Language Agent Tree Search）+ ReAct**  
 混合架构，通过蒙特卡洛树搜索（MCTS）策略智能探索漏洞空间，结合 Katana API 端点发现、Playwright 浏览器引擎、mitmproxy 流量分析、crawlergo 深度爬虫和隔离 PoC 沙箱，实现从侦察到验证的全链路自动化漏洞挖掘。  
  
与传统扫描器不同，Argus 不依赖固定规则或签名库，而是通过 LLM 的推理能力生成漏洞假设、自适应构造 payload、智能回溯无效路径，模拟真实安全研究员的漏洞挖掘思维过程。  
## 核心特性  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n277" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">特性</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n278" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n280" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">LATS + ReAct 混合架构</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n281" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">蒙特卡洛树搜索指导探索方向，ReAct 循环执行具体验证，搜索效率远超线性管线</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n283" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">工具驱动攻击面构造</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n284" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">端点发现完全由侦察工具（Katana、crawlergo、Playwright、dir_scan）驱动，LLM 仅作策略顾问，消除端点幻觉</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n286" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">EndpointCapability 统一模型</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n287" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">结构化端点能力画像 + VulnType×Endpoint 声明式兼容性矩阵，杜绝错误类型匹配</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n289" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">MCTS 智能搜索</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n290" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Wilson UCB + 多样性 + 端点多维评分，将有限搜索预算分配到最有价值的方向</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n292" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">渐进式 Level0 快速探测</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n293" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">两轮 9 次探测 + 参数自动回退，精准筛选高价值分支</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n295" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">多类型漏洞检测</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n296" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">SQL 注入、XSS、SSRF、LFI、RCE、IDOR、SSTI、认证绕过、信息泄露等</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n298" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Katana API 端点发现</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n299" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">ProjectDiscovery Katana 被动+主动爬取，JS 端点提取，表单自动发现</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n301" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Playwright 浏览器引擎</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n302" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">支持 SPA/前后端分离站点的动态渲染、表单交互、JS 事件触发</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n304" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">mitmproxy 流量捕获</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n305" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">实时捕获浏览器交互产生的隐藏 API 调用，发现前端看不到的攻击面</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n307" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">crawlergo 深度爬虫</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n308" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">基于 Chromium 的深度爬虫，自动触发 JS 事件和填充表单</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n310" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">隔离 PoC 沙箱</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n311" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">RestrictedPython + Docker 双层隔离执行 PoC 代码，安全验证复杂漏洞</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n313" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">自适应 Payload 变异</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n314" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">WAF 检测 + payload 变异绕过，支持编码、大小写、注释等多种绕过技术</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n316" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">实时事件流</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n317" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">WebSocket 推送 Agent 思考过程和工具执行状态，前端实时可视化搜索树</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n319" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">自动报告生成</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n320" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞确认后自动生成结构化报告，包含复现步骤和修复建议</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n322" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 209.125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">16 款安全工具</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n323" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 699.875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">覆盖被动侦察、主动探测、PoC 执行全阶段，四级风险管控</span></span></span></td></tr></tbody></table>  
界面预览  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSP40hFcTB74n7bAvS9U03ak7N87tGYum4TMic6O9HpiboW4Xmhmuxaib5Ok6oiaicibBPymnfodBStGmwE5yrmAxVrsIs826vPICut8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTkH72NF0aO7djk9lricUoEEjVx2SHUuiac1QVg6z0Td7Y4gADeHDhw5EfSQt1BJutkicasFWE94Hf0ZSbtPX0fAVQUibqFsOYiavZo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSbIPqSrPzkaeZ4nKoZMIXlDXNznhpOlkNYBSFlOZhfXVbpJHvn90vs6JqUq2rXIWVVSSfpLEDVmM3a6H8OSggCUqwPkqT3fmI/640?wx_fmt=png&from=appmsg "")  
  
核心技术解析  
```
Recon Phase (三层回退提取)
  ├─ Layer1: http_request 工具 → 正则提取 href/action/src
  ├─ Layer2: httpx 直连 → 正则提取 (工具层失败时)
  ├─ Layer3: Playwright headless → DOM 渲染提取 (JS 页面)
  └─ Katana 并行爬取 → 端点 + 表单 + JS API (与 dir_scan 并行)

AttackSurface Construction (零 LLM 驱动)
  ├─ EndpointNormalizer → 清洗 LLM 注释/通配符/非法字符
  ├─ 7 种端点来源合并去重 (categorized_links + forms + dir_scan + katana + ...)
  ├─ P0 智能分类评分 → vuln_page / api_endpoint / auth_page / config_leak
  ├─ P1 能力探测 → GET 每个端点 → 过滤 forbidden/not_found
  ├─ P3 URL 参数推断 → sqli_id.php → ["id"]
  └─ EndpointCapability → VulnType 兼容性矩阵

Branch Value Estimation (语义感知)
  ├─ estimate_branch_value_v2 → 不兼容组合 → score=0.00
  ├─ /.git/HEAD + RCE → 0.00 (is_version_control → 仅 info_disclosure)
  ├─ /vul/rce/rce_ping.php + RCE → 1.00 (路径含 "rce"+"ping"→强信号)
  └─ value_decay_factor → 每次 backprop 衰减 15%, <0.1 → exhausted

Level0 Quick Probe (两轮渐进)
  ├─ Round 1: 3 次探测 → promoted / killed / needs_deeper_probe
  ├─ Round 2: 3 种不同技术 payload → 深度验证
  ├─ 无参端点保护 → 通用参数回退 (sqli→["id","q"], rce→["cmd","ping"])
  └─ 参数发现后自动注入 node.state.current_param
```  
### LATS + ReAct 混合搜索引擎  
  
传统漏洞扫描器使用线性管线（枚举 → 测试 → 报告），存在两个问题：  
1. 无法根据中间结果调整策略（如发现 WAF 后无法智能绕过）  
  
1. 搜索空间爆炸时无法智能分配预算  
  
Argus 采用   
**LATS（Language Agent Tree Search）**  
 架构解决这两个问题：  
```
MCTS Loop:
  1. Select   — 自适应多因素选择 (Wilson UCB + 多样性 + 新近度 + 先验 + accessibility + value_decay)
  2. Execute  — ReAct 循环执行具体验证 (Thought → Action → Observation)
  3. Expand   — 发现驱动的动态扩展 (新端点、WAF 规则、参数推断) + capability 门控
  4. Backprop — 将奖励信号沿路径回传, 更新节点价值估计 + value_decay_factor 衰减
  5. Evaluate — 检查终止条件 (发现漏洞 / 预算耗尽 / 全部穷尽 / 连续无发现)
```  
  
**MCTS 选择因子**  
:  
- exploitation (Wilson score lower bound): 利用已知高价值节点  
  
- exploration (UCB): 探索未访问节点  
  
- prior (value_estimate): 先验价值估计  
  
- diversity: 端点/vuln_type 多样性  
  
- recency: 新近度加权  
  
- accessibility: 端点可达性 (forbidden/not_found → 0.0)  
  
- vuln_penalty: vuln_type 全局失败惩罚  
  
**奖励信号设计**  
：  
- 确认漏洞: +0.4 ~ +1.0（按严重性分级）  
  
- 发现有价值线索: +0.1 ~ +0.3（鼓励继续深挖）  
  
- 无信息增益: -0.12（鼓励换方向）  
  
- 明确死路: -0.15（鼓励回溯）  
  
**跨分支共享知识库**  
：  
- 端点指纹、WAF 规则、漏洞信号在各搜索分支间实时同步  
  
- 避免独立 ReAct Agent 的信息孤岛问题  
  
- 支持 Graveyard 节点复活（已剪枝节点在有信号时重新探索）  
  
- 端点步数配额 (单端点 ≤12 步, ≤10%, 连续 3 步无信息截断)  
  
- 端点×vuln_type 组合配额 (同一组合 ≤3 次完整 ReAct)  
  
工具使用  
### 环境要求  
- Docker & Docker Compose (v2.0+)  
  
- 至少 4GB 可用内存（Chromium + crawlergo + Katana 消耗较大）  
  
- AI API Key（Anthropic Claude）  
  
### 一键启动  
```
# 克隆项目
git clone <repo-url> argus && cd argus

# 配置 API Key
export ANTHROPIC_API_KEY="sk-ant-..."

# 构建并启动所有服务（9 个容器）
docker compose up -d

# 查看所有服务状态
docker compose ps
```  
### 使用流程  
1. 访问   
http://localhost:3000  
 并注册账户  
  
1. 进入   
**系统设置**  
 页面配置 LLM API Key  
  
1. 创建扫描任务，填入目标 URL（例如   
https://target.example.com  
）  
  
1. 启动任务，在任务详情页实时查看：  
  
1. 搜索树展开过程（MCTS 节点选择与扩展）  
  
1. Agent 思考链（Thought → Action → Observation 循环）  
  
1. 工具执行结果与奖励信号  
  
1. 漏洞确认后自动生成报告，可导出为结构化格式  
  
工具链接  
```
https://github.com/R0x7e/Argus
```  
  
  
  
**后台回复加群加入交流群**  
  
****  
**广告：********cisp pte/pts &nisp1级2级低价报考**  
  
  
**陌笙安全纷传圈子+陌笙src挖掘知识库+陌笙安全漏洞库+陌笙安全面试题库**  
**简单介绍****（**  
**加入纷传圈子**  
**送****知识库+漏洞库+面试题库****）**  
                          
  
如果觉得合适可以加入,圈子目前价格  
39.9元，价格只会根据圈子内容和圈子人数进行上调，不会下跌。。。    
  
  
**圈子福利**  
   
  
**edu漏洞挖掘1v1指导出洞**  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKQWHxLsRrPqpqdiceX76d7yExQIyOqFmmJAfHQh7qzKvPc2V5z6iaa0RY6Ib8AsGvgS5MKkAk5aaHnJBaSnI10LDKQYMLcQMmg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR8pnPeapLBK4Jsa4ufCvFoGL66t7PKeZyA3AjNxsObjtnCibN2gzGX7NMS7Wo5sj3YYL2iboeRuQDcWqiapc8xuo5fticoBG4DsyY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKIBNIQIVicRWJLbyGRmg92vPzc8375PJpcYVvfywzwqnaeBicZuEbfvuic9KRdjwkahSDic5VqrH2Mb4NkqtkADl5HLIh8gPex60/640?wx_fmt=png&from=appmsg "")  
  
**skill+grok辅助挖掘某企业sr**  
**c****实战效果，能出但是重复多，agent独立挖掘也可以，见仁见智，看个人习惯，好的模型是最重要的。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQwyn779TTwY7vZkePQCL8k3K8dYxNdyzfgADL4dJcNUvpmodLeDVCZ6xDC4RJXEBmO2tcWqgUNdTVicKTW0jpdsg7X6xwP5gIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQnIqagDL2A4BUIXrib9YVmATWuaIDqETqYd9ToHib52mDyoMyqc6Wzh733FRnbsDsGgey7B8s8jr72UtkPY6ich58niaPJqoItKcE/640?wx_fmt=png&from=appmsg "")  
  
****  
**企业src边缘&核心资产实战效果&&有重复但是证明好模型+AI确实够用**  
  
**（图片仅供参考，我出不等于你出，见识到ai神力即可，多去用AI!!!）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT1YNI9U6r6NMO4UMUBROoWeC4XQC4Dge94ODZ7tXY6tbxqb3IJoghve0u1SfkygE5UJ5HTdUBvLZKrn5ps13F71piax5dlHnOE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTGjdxKy4nllaLXIznTvRqicichITccuB8psYFRYakw6ViauCk6iccziahfPw4fnrqhyCp7Zkq7lRI0DOicyrZlNyicYibTVibfGFZVaG0c/640?wx_fmt=png&from=appmsg "")  
  
****  
**不是P图,单洞1.2w记录**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQQpQdR2Ttwqxibyr75Is0kBG2N2tLYQIaau7SS278oyQ4RDpNScviaMt4wtlfgDCibE05WgoMhE5kZUrP8ciaYIdnxA594wsmoAAs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSRn2EsfFkA5mG6dcn7JLQMroc2dy3EQb3ueY2Cspd0WYgicXEnSF68UD43nNd4plkxmkTpEOh2kkQMEWZZIjE0ibA8r1q4IfiaxI/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙src挖掘知识库介绍（内容持续更新中!!!)**  
```
信息收集(主域名信息收集,子域名信息收集等&会永久提供fofa-key助力)
弱口令漏洞&未授权访问漏洞挖掘
任意文件读取&删除&下载&上传漏洞
sql注入漏洞
url重定向漏洞
csrf&ssrf漏洞挖掘
XSS&XXE漏洞挖掘等等常见漏洞
cors&目录遍历&越权漏洞挖掘
EDUSRC(证书站挖掘案例分享&edusrc挖掘技巧分享)
CNVD挖掘技巧分享&实战案例报告编写
公益漏洞挖掘（公益src挖掘漏洞分享&提供补天1权重资产）
SRC挖掘实战(针对各种常见功能总结的常见测试思路等快速提升)
经典常见Nday漏洞(常见中间件&以及各种常见框架)复现
云安全相关漏洞挖掘（云key扫盲&云存储桶&快速识别云环境&云攻防）
AI相关学习（AI基础&AI代码审计实战测试&webLLM攻击等）
APP&小程序漏洞挖掘
等各模块不在一一介绍
```  
  
信息收集  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTu9DGyTubluhYicFynwVBKa4V06sDfEVKOyk5Q4ghZzLMDAuLb1M1oR4RJumGWrADPapFjTrOjpksKQ8q0YYCnl3ZWLof8Knzg/640?wx_fmt=png&from=appmsg "")  
  
src挖掘基础  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboR45bibbJEb28a1gS5yth3r5HyOsgPiaOUHHYriahZyIyrk0LMOsHW4VoDibyBRibTNzptGiaLWX62UwykicwvbxCJPopvklqiaxML8lS8/640?wx_fmt=png&from=appmsg "")  
  
src挖掘实战  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTKWnTsN6CXf3djhXIlMKNRjVmJn3g5b23ur9E6Cx3O68f0hXVjCiaj8J4RYeTGBecqf1k99phG0ice2wtd5lKgR46OeqeLQfMpk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRbZ1HYm7R7YEiaxRVQibGWyricx9l7HpGjS4ZfWRdlft8iacwkpzYyZfmYEkWdJgYRORPkNFR6dADR5MyE524tWX6cAwN8MmrCZu0/640?wx_fmt=png&from=appmsg "")  
  
  
edusrc  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQVVlTXhibjR8UiakZBQicXRZrQ7hdoOz5G8MQrcuDBGbqJdO0kIz6R9IU4ObAeOiabT8pr6lc7jibdIkKoTjiaXNHPLAwAB3BV2UvLM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSCrvarBbzP4L9kS6P0LVH9JMdmcbFDKiaicHqMFgTxq3x4iatjDJQicmc7NPC14C9Fk3icFjrouSgNVaN8Byuf0C0Iq9O6D1XPvFvY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRALwXmgZ4mh2LW0RdicrKjBCP7P1iaF14G0Eq2v3KRnTJORpwXZlF58WEz6QicxLJpyJaA5iah5CF2rHjBz4JzOELFRaZTAKOQ2tQ/640?wx_fmt=png&from=appmsg "")  
  
经典nday复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRu8Gf849iaCkSBxLL8IlzJTRs185QicEe9l5UGI1dEVKISt2IGGveZynXBW9tIUsxNsz4adSTib7rib50uSJdjNfTvVRFrbPJhzL4/640?wx_fmt=png&from=appmsg "")  
  
  
云安全&AI安全  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQ9qiavETNjaaX162czpNCqpw3uJqVpicbI15AXzhf5x8icmHxBdTGOgRgzNPGF3Aw2gglT4Fx09JGXYibQC6U7CQKVmoH08l3meia4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQgcsjiaZ4S26TWowHfpBkhSeHf2pjrcDyicJuia3uqvRBauLEOicibibEMqibnBMtjopFL8No7UXNibbURvzeJ3dQHTibvGxRQGnorb4co/640?wx_fmt=png&from=appmsg "")  
  
**陌笙安全漏洞库介绍**  
```
最新漏洞查看
1day&0day分享
EDU学校相关漏洞
Web应用漏洞
CMS漏洞
OA产品漏洞
中间件漏洞
云安全漏洞
人工智能漏洞
其他漏洞
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTFBRMa8XwYxfcZMyXicx94xSKxawPcqFia2rJKOL7fSLYXiccwHc868XxNGIQ5z7ibiaI1MNAGRrK7U6wXJTsZOCAu2I5XV1boTAL4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSBzdakI9XI33ReAm2dxO8vgzw3JicQmUuWCb5ayBlKR1PoQHEHFETteBnicyupwU0mXvXibfrDoyg8nSWBGoK1p2YXY3ElhcvOQ0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSajCclDhuRpaLic9Ld915CHU7RqSC1LCrPGfNZiavdPEVeDedDWOPBhtMLCicTp3RNd1lT0Pmfo3mx5B0hUxbQg3ic6Via90NMtZVk/640?wx_fmt=png&from=appmsg "")  
  
****  
****  
**陌笙安全面试库**  
```
渗透测试基本问题一汇总
渗透测试基本问题二汇总
渗透测试基本问题三汇总
微步护网面试题目
长亭科技面试
深信服护网面试
启明星辰渗透测试面试题目
安恒面试题目
绿盟笔试题目
360面试
奇安信护网面试
运维面试题目
运维面试题库
网安面试相关文档大全
相关面试文章推荐
等等
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSoLqEzH0a3A4LQrvTIkGx81Sh5pf6fCoEQJhYg715vrJicSkfBuCoAmV2Kp4uOMe5jcUZutPwicibFibtJ1ZmyiaAibCg0XicWnsNcicE/640?wx_fmt=png&from=appmsg "")  
  
****  
**POC库****&&更新适配afrog&&nuclei&&dddd的POC&1day/Nday等&&**  
**dddd二开****工具[助力渗透测试&&红蓝攻防]**  
  
**工具截图**  
  
****  
**实战效果**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSKqLXNcOPE07xOwOUCjRGuFphopPumW9RaticmNCuEUXu52GtdTTfpTUicrBj80kMcZzJsnps3abyvXIvLHEIhvMoXUApOqZCe4/640?wx_fmt=png&from=appmsg "")  
  
****  
**poc库【后续持续更新】**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTG9Lyp44aFffUOxQKtHjToGfqFWTjswYft0VtAPINtV5MqmrTTj8GWrVb6yowvHURubPgOqdribmibWEb0Fcj3YdN4iahUwItcxE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRAMJIIvexOOJa5KhrsKmlsx8bkwib9SPoK72Q0OSPWR5qx67yvl8scMQ5bg8caBXZH01kM39RDnKpnWSaTicgobRmLygERGFWls/640?wx_fmt=png&from=appmsg "")  
  
  
**AI赋能-**  
**skill辅助**  
**漏洞挖掘（免责&&慎用）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR3Dib0RVxVhUOzS6ibC6BvkfulXQAclic0XCXMS35C4EPoqX1b2eMVj2CFiaLCelVs1szGibaHiaAq7WibRdwHUg0IwO8fjDdWxNv6eY/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSXZZVels2NibmHgyxntlCRNIkgoqMPfUPwSM9O43OqniaZLDEJic9QRkW01gNTydFkibdI6yBRkJJ1sDUmfl7iaicoibz1QLp0J2pWE4/640?wx_fmt=png&from=appmsg "")  
  
  
**圈友skill+ai辅助渗透**  
**实战效果**  
**，支持打假！**  
  
证书站  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQibuxKAyHBZicB1t5yGVKyV82Teo8C2MbjKPytKziaXUcjPiao8ylHbD4vicAld8equC9alic3NksvWJ09wArXaPXZD10vjPtfoia4Vg/640?wx_fmt=png&from=appmsg "")  
  
普通站点  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ1Xt6gdLgd2d1mf8QURQX4YZjHA2uIw9GdTuxzSafBVOJzrQVmHJlqdhVWdVDj3OQsiaQhYOaoiabXc6EgajvBMvB6xBZwdJkIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙**  
**纷传****圈子介**  
**绍**  
```
1、src挖掘思维导图，信息收集思维导图，edusrc挖掘思维导图，以及后续的红队&面试思维导图&自己网安笔记等持续更新
2、2025-2026的edusrc实战报告包含证书站和非证书站以及2025之前的各种优质报思路分享
3、各种src报告思路分享（内部&外部）
4、分享各种src挖掘&edusrc挖掘培训资料&视频
5、不定期分享通杀、0day
6、有圈子群可以技术交流以及不定期抽取证书&免费rank
7.分享各种护网资料各家安全厂商讲解视频&精选实战面试题目
8、各种框架漏洞技巧分享
9、各种源码分享（泛微、正方系统、用友等）
10、漏洞挖掘工具&信息收集工具&内网渗透免杀等网安工具分享
11、各种ctf资料以及题目分享
12、cnvd挖掘技巧&CNVD资产&src资产分享&补天1权重资产分享&fofakey共用
13、免杀、逆向、红队攻内网防渗透等课程分享
14、漏洞库&字典以各种内容不在一一说明
15、cisp-pte/pts&nisp一级&nisp二级&edusrc证书内部价格
15、如果有漏洞挖掘问题或者工具资料需求可以找群主(尽量满足)
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSY2pbvbP3qGAlW8O43bRvAISCxZm4UDTRsaMVbJKTsjfTMTDlq6qNBcVs4tkl4UzgqGz5ag81baU1rusKE09J9T6cMVliaibibwQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/MSDUaqtwboTrLRQpTicOR7bzyNiajiapVJgyMiaYlEDBVU87YXMnanOFWsCYN3cCVGsKkibzV9dMryvbFXBb4Z3472ib27RJ1Xq1HnKJIp5u49GYQ/640?wx_fmt=jpeg&from=appmsg "")  
  
**目前800多条内容，扫描下方二维码查看详情以及加入圈子，持续更新中。。**  
  
**如果觉得合适可以加入，价格不定期会根据圈子内容和圈子人数进行上调。。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRngNCK3ae5W1nw179icr078zj1eBrn2aP1FVic1yJHdvbJ1sIKcZzNKlhXsbTfcUYdo0miblHoJpdcfG10cZfp6HbbHQfkfiajU40/640?wx_fmt=png&from=appmsg "")  
  
