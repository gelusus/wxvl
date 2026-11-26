#  AI 挖到漏洞就收工？我给它加了一条 “不许交卷” 的规则  
溯风安全
                    溯风安全  陌笙不太懂安全   2026-11-26 09:00  
  
# AI 挖到漏洞就收工？我给它加了一条 “不许交卷” 的规则  
  
漏洞确认了，为什么不算完？  
  
仓库：https://github.com/yuqiuyang12/cyberstrike-zhulong  
## 一、这张证，是 loop 自己跑出来的  
  
11 月 9 日，我收到一张 CNVD 原创漏洞证明：某信息股份有限公司 Minos 数据中台，任意文件上传，中危。  
  
![图 1 · CNVD 原创漏洞证明（姓名与证书编号已打码）](https://mmbiz.qpic.cn/mmbiz_jpg/436qAALMYYnshdWejPib216bYJp7iaSzdg6pvE1aY3DkF3khAateicteiboP2Dwibxzu6J4kbALb9XAvy04mujsJxTCXxNPjjou3Opg8Pg13JvicU/640?wx_fmt=jpeg&from=appmsg "")  
  
图 1 · CNVD 原创漏洞证明（姓名与证书编号已打码）  
  
不是我手工挖的。是烛龙这个 loop，一轮一轮跑出来、自己收口、自己把证据凑齐。  
  
可两个月前，同一套东西只会做一件事：跑完一轮，交卷。  
  
漏洞躺在“已确认”那一栏，然后就没有然后了。  
## 二、loop 成了行业热词，我这边缺的正好是 loop  
  
11 月 23 日，智谱发布 GLM-5.4，第一次把 Loop Transformer 放进旗舰模型。  
  
做法不复杂：把基座中间 28 层共享权重的网络，循环跑两遍。参数一个没加，有效计算深度翻倍。智谱自己的表述是“智能水平达到 2-3T 级模型”——这是厂商标称，不是第三方跑分，先把前提记着。  
  
同一个月，AI 渗透这条线上也在变天。  
  
CyberStrikeAI 被微步收购，开源版保留，另推企业版；更早的 Artex 开源版，10 月 8 日已经转闭源。理由都差不多：开源的东西被拿去做未授权攻击，厂商要收口。  
  
一边是 loop 在模型层被验证，一边是平台在往企业版收。  
  
我的判断很朴素：平台会越来越闭，但“loop 该怎么跑”这件事，能在开源侧做完。  
  
所以我把我加的那一层，开源了。  
## 三、先说清楚我不是谁  
  
我不是 CyberStrikeAI 的作者。  
  
AI 渗透测试现在大概三条路：ARTEX 走资产图，Cairn 走黑板，CyberStrikeAI 走工具集成加流程编排。AI安全社那篇解析里的原话是“CyberStrikeAI 侧重工具集成和流程管理，适合整合已有工具和测试流程的团队”。  
  
我就是那种团队：手上一堆工具、一堆流程、一个永远跑不完的清单。  
  
CSAI 自带一个 eino 单代理：一次对话，一次产出，结束。它够用，但它不会沿着利用链往上爬。  
  
所以我没重写平台。我在它上面加了一层，叫 eino2，也就是烛龙。  
  
![图 2 · 模式下拉里的“烛龙（Zhulong）”，与 Eino 单代理并列](https://mmbiz.qpic.cn/mmbiz_jpg/436qAALMYYkcCDwDCQP3xArrXkzk3RJ5DHicRG5ORVg2WPMzYzplnic5UgBecYUT8I2q2Doquj0MhCZ9fX9vMmZXaF0rLGg057PANKTUOgs5g/640?wx_fmt=jpeg&from=appmsg "")  
  
图 2 · 模式下拉里的“烛龙（Zhulong）”，与 Eino 单代理并列  
  
一个细节：仓库里保留了 internal/zhaoheng 这个包。昭衡是烛龙的前身，包名和命名都没改。你在截图里看到“无人值守（昭衡）”，说的就是同一个东西。  
## 四、eino 和 eino2 的差别，一句话  
  
eino 是“跑完一轮”，eino2 是“跑到链清完”。  
  
<table><tbody><tr><th style="padding:9px 10px;border:1px solid #e6e4dd;background:#f2f1ec;font-size:14px;font-weight:600;color:#141413;text-align:left;"><section><span leaf="">维度</span></section></th><th style="padding:9px 10px;border:1px solid #e6e4dd;background:#f2f1ec;font-size:14px;font-weight:600;color:#141413;text-align:left;"><section><span leaf="">eino（v1）</span></section></th><th style="padding:9px 10px;border:1px solid #e6e4dd;background:#f2f1ec;font-size:14px;font-weight:600;color:#141413;text-align:left;"><section><span leaf="">eino2 / 烛龙（v2）</span></section></th></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">交卷标准</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">跑完一轮</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">open 链 = 0 才许 PASS</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">优化目标</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">面覆盖率</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">链深度（0→6 台阶）</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">blocked</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">记一笔就过</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">≥3 个变体才许收口</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">无人值守</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">无</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">会话级勾选，自动续轮</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">止损</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">无</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">连续 3 轮无 exploited → 止损轮</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">链记账</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">无</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">四工具记账，每次尝试落证据</span></section></td></tr></tbody></table>  
  
引擎没换。换的只有一条：交卷标准。  
## 五、把“确认”变成坐标：链深度 0 → 6  
  
先定义“链”。一条链 = 一个漏洞的利用尝试线。中高危漏洞一旦确认，烛龙就给它开一条链。  
  
<table><tbody><tr><th style="padding:9px 10px;border:1px solid #e6e4dd;background:#f2f1ec;font-size:14px;font-weight:600;color:#141413;text-align:left;"><section><span leaf="">台阶</span></section></th><th style="padding:9px 10px;border:1px solid #e6e4dd;background:#f2f1ec;font-size:14px;font-weight:600;color:#141413;text-align:left;"><section><span leaf="">含义</span></section></th></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">0</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">漏洞已确认</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">1</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">读到敏感数据</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">2</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">认证绕过</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">3</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">写入（创建 / 修改业务数据）</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">4</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">上传成功</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">5</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">执行</span></section></td></tr><tr><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">6</span></section></td><td style="padding:9px 10px;border:1px solid #e6e4dd;font-size:14px;line-height:1.65;color:#3a3a38;"><section><span leaf="">横向</span></section></td></tr></tbody></table>  
  
![图 3 · 链深度台阶：0 是起点，6 是横向](https://mmbiz.qpic.cn/sz_mmbiz_jpg/436qAALMYYlOxJMAzeEuee7JRBdgLYJMrpISxLttWdV4dpD8iciaibibCaPV0MZWHib5shQicG19GrLYNfbLMAZVA5icPWm2sk3baCpregXWhDPQho/640?wx_fmt=jpeg&from=appmsg "")  
  
图 3 · 链深度台阶：0 是起点，6 是横向  
  
四个工具记账：zl_chain_open 开链，zl_chain_attempt 记一次尝试，zl_chain_close 收口，zl_chain_list 查账。  
  
每次尝试必须落三样东西：payload、证据的绝对路径、verdict（打穿 / 被拦 / 存疑）。  
  
这像作战日志。记的不是“这块打过了”，是“几点、用什么、打哪、结果如何”。好处很实在：别人拿到账本，能顺着证据复核一遍，而不用只信我一句“打过了”。  
## 六、三条硬规则  
  
第一条，blocked 不是免死金牌。同一台阶，必须试满 ≥3 个不同变体才允许收口。试一次就说“打不动”，那不叫结论，那叫偷懒。  
  
第二条，deliver_gate 硬门。open 链 > 0，就不许 PASS。像批奏折，没清完的折子不能合上。漏洞确认了、链没走完，引擎会把 PASS 拦下来，逼你继续爬。  
  
第三条，止损阀。连续 3 轮没有 exploited，转止损轮；6 轮还没有，停排。油表红灯亮了，不是让你接着踩油门。  
  
![图 4 · 三条硬规则](https://mmbiz.qpic.cn/sz_mmbiz_jpg/436qAALMYYkFjlia9ic4vYdPJgUXbwBzxOYILvrib3JZKLzyn0FwibX7z7umicvibDYDmFV9TZnGJogBb1ABWUgLrGHGNULD4dqpYRwAgtwkOQ84o/640?wx_fmt=jpeg&from=appmsg "")  
  
图 4 · 三条硬规则  
  
这三条凑起来，是烛龙的性格：不轻易认输，但也绝不无脑烧预算。  
## 七、真跑一轮：open=0 之后还发生了什么  
  
先说明：这一轮跑在一个已授权的项目上。  
  
![图 5 · 第 3 轮，zl_chain_list 回来：open=0，exploited=11](https://mmbiz.qpic.cn/mmbiz_png/436qAALMYYny4WWKt2DwUQ3ZjKxiczh4jPaF9kPhsRGFRVl2GfLoaEiaRu9TmLwUS6icJibEERsPTqf4sYMSMibYg8wBLv5gQrlaD7hvmo4bYBg0/640?wx_fmt=png&from=appmsg "")  
  
图 5 · 第 3 轮，zl_chain_list 回来：open=0，exploited=11  
  
第 3 轮，zl_chain_list 回来：open=0，exploited=11。  
  
open=0 是旧链全部收口了；exploited=11 是这次会话累计打穿的 11 个点，不是 11 张 CNVD。  
  
链清完，任务并没结束——这一轮的指令是“找新的中高危，看能不能在别的站点复现”。  
  
![图 6 · 第 4 轮自动续跑：同目标、同链、未收口](https://mmbiz.qpic.cn/mmbiz_png/436qAALMYYny4WWKt2DwUQ3ZjKxiczh4jPaF9kPhsRGFRVl2GfLoaEiaRu9TmLwUS6icJibEERsPTqf4sYMSMibYg8wBLv5gQrlaD7hvmo4bYBg0/640?wx_fmt=png&from=appmsg "")  
  
图 6 · 第 4 轮自动续跑：同目标、同链、未收口  
  
第 4 轮自动续跑，承接上一轮的 next_actions：minos 路径穿越，download.html?fileName=。续跑前它先查了 facts，确认这条路没被测过，避免重复劳动。  
  
![图 7 · 收口六闸门全绿（依据栏涉及客户信息，已像素化）](https://mmbiz.qpic.cn/sz_mmbiz_jpg/436qAALMYYmHltq0zeLBfAtF3icvUc4uZlb29a9sY6Lr45B9mRtiaVEd085OKwlnrAYd5sEYVYyvTAEmo8P5vhwqOP6E4wqMoAuWiax2wA5zCE/640?wx_fmt=jpeg&from=appmsg "")  
  
图 7 · 收口六闸门全绿（依据栏涉及客户信息，已像素化）  
  
收口走六道闸门：G0 资质、G1 资产、G2 原生性、G3 定级、G4 查重、G5 证据。  
  
最硬的是 G2。判据是“16 站逐字节同构”：同一个漏洞在 16 个站点上响应体 md5 一致，才算原生，不算碰巧撞上一个站。定级口径也写进账本：任意扩展名写入 = medium。  
  
最后是那张证明：中危，11 月 9 日收录。  
  
这里要诚实一句：从开链到收口，没有人工拍板。但报送是我自己做的——填贡献者、提交材料，都是人。loop 负责把证据凑齐，人负责签字。  
## 八、它不好用的时候  
  
先说边界，这是可信度的来源。  
- 只管增量，不管本体。只适配已经装了 CyberStrikeAI 的机器，且树必须是 loop2（eino2）布局、含 internal/zhaoheng；不提供平台本体。  
  
- 烧钱。截图里那一轮，耗时 10 分 53 秒、5.1M tokens。无人值守不是免费的，没有预算就别勾 18 轮。  
  
- 前提是资产清单。没有目标清单，开链无从谈起——它擅长把已知漏洞往下钻，不擅长替你找入口。  
  
- 六闸门是降噪，不是除噪。它把“16 站逐字节同构”这种判据固定下来，但误报依然存在，最后仍要人看一眼。  
  
- 前端开关是可选项。不接前端，用 API 也能跑。  
  
- 只用于授权的安全测试与研究。  
  
## 九、怎么装  
  
git clone https://github.com/yuqiuyang12/cyberstrike-zhulong zhulong && cd zhulong bash scripts/apply.sh /path/to/your/CyberStrikeAI   # 合并式覆盖，先备份 cd /path/to/your/CyberStrikeAI && go build ./cmd/server bash /path/to/zhulong/scripts/verify.sh .           # 静态检查接线是否齐  
  
包内是完整 eino2 引擎：loop2/ 216 个文件、zhulong/ 27 个文件（零平台依赖，可单独 go test）、chain_tools.go、schema、hooks/ANCHORS.md 的 17 处挂钩点、docs 六篇。  
  
前端那个模式下拉和无人值守勾选，是 hooks/ANCHORS.md 的第 17 条，可选。  
  
![图 8 · 仓库首屏：Apache-2.0，与上游同协议](https://mmbiz.qpic.cn/sz_mmbiz_jpg/436qAALMYYnViaBaSWGjfOjOeyGbK7OGkDx43icLibcQ1MpParOAGOoykAWv7pTUwUFtc2RdU6G17oZqckCLSKLR19IOKb66ZZo2tN3ZC8CFI4/640?wx_fmt=jpeg&from=appmsg "")  
  
图 8 · 仓库首屏：Apache-2.0，与上游同协议  
## 十、结尾  
  
智谱用 GLM-5.4 证明了 loop 在模型层有效。我想证明的是：loop 在渗透测试这行同样有效——只不过这里的“循环”不是多跑几遍网络层，是多走几层利用链。  
  
仓库在 GitHub：https://github.com/yuqiuyang12/cyberstrike-zhulong  
  
装上跑一轮，把你的 open 链数发我。  
  
参考资料：智谱 GLM-5.4 官方说明（zhipuai.cn）、InfoQ《GLM-5.4 编程能力暴涨》报道、微步 RedAgents 平台页、仓库 README 与 UPSTREAM.md。  
  
免责声明  
  
本文涉及的所有测试，均在已授权的项目环境中进行，文中工具仅用于合法的安全测试与研究。读者因传播、利用本文信息造成的任何直接或间接后果及损失，均由使用者本人承担，本公众号及作者不为此承担任何责任；请勿将相关技术用于未授权目标。如有侵权，烦请告知，我们将立即删除并致歉。  
  
  
