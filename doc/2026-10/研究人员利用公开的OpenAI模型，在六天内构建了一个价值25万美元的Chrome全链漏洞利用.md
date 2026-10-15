#  研究人员利用公开的OpenAI模型，在六天内构建了一个价值25万美元的Chrome全链漏洞利用  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-15 00:35  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
## AI安全研究的新里程碑  
  
"六天时间内，一名研究人员仅凭公开的OpenAI模型就成功构建了一个影响10亿用户的谷歌Chrome全链条漏洞利用，并获得了25万美元的最高漏洞赏金。" 这听起来像科幻小说情节，但这是安全公司XBOW在2026年10月刚刚完成的一项里程碑式研究。   
  
这一突破性成就不仅刷新了漏洞发现的效率纪录，更昭示着**AI与网络安全领域正在深度融合**  
，传统的安全攻防平衡正被彻底打破。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZjPjWQtAfZ0UghNQsFZFQ40kicRgKZb0SNqjicqAeibodzjFM2Ne1g0I2ZjjAw1VaS6GuEficflibtAoBF2p0Hy6lcVABct6ibIpgBqg/640?wx_fmt=png&from=appmsg "")  
## 何为"全链漏洞利用"？  
  
正如XBOW团队在技术报告中强调的那样：  
  
**"全链漏洞利用是一套组合使用的漏洞集合，它们共同突破Chrome保护措施的每一层，从网页层面一直到计算机系统本身。"**  
  
Chrome浏览器为网页代码运行设置了两层嵌套的沙箱保护机制，这种架构设计旨在防止受攻击的网页访问机器其他部分。XBOW构建的攻击链成功穿越了这两层沙箱，最终能够在无需用户点击或下载任何内容的情况下，在Windows系统上运行任意代码。  
## 六天的破纪录时间线  
  
这六个破纪录的日期发生在2026年8月2日至8月7日，正值拉斯维加斯的Black Hat和DEF CON安全会议期间。  
  
**历史对比令人震惊**  
：XBOW研究人员Brendan Dolan-Gavitt直言："在大型语言模型（LLM）时代之前，构建一个完整的Chrome全链漏洞利用需要4人团队花费6-9个月时间。我们的第一次提交耗时2个月，而这次获奖的成果仅**6天**  
。"  
  
更为惊人的是成本效益分析。根据XBOW创始人Oege de Moor的估算，这六天的攻势大约花费了**4万美元的AI计算资源**  
。Dolan-Gavitt警告防守者："当回报是一个Chrome全链攻击时，六天时间和4万美元并不是很多时间或金钱！门槛已经大幅降低了。"  
## 三层攻击架构  
### 第一阶段：V8 JavaScript引擎漏洞（CVE-2026-85045）  
  
攻击的起点在Chrome的V8 JavaScript引擎Maglev优化编译器中发现。这个漏洞允许攻击者控制Chrome JavaScript引擎内部的内存。研究人员详细解析了漏洞机理：优化的代码基于特定假设运行，当假设不再成立时，V8必须重建普通JavaScript对象并继续在更通用的代码中执行。在这个过程中，Maglev错误地创建并复用了可变HeapNumber  
对象。   
  
**关键技术细节**  
：  
- 漏洞位置：V8的Maglev中层级优化编译器中的deoptimization路径  
  
- 根本原因：编译器错误地将可变对象转换为字面量，然后将其标识复制到每个重建所有者中  
  
- 最终效果：攻击者获得了对V8沙箱内4GB内存区域的可靠访问权限  
  
### 第二阶段：动画WebP图像解码器漏洞  
  
这个阶段是**V8沙箱逃逸**  
，攻击者控制V8沙箱内存后，找到一个漏洞可以突破引擎的内层沙箱，并对计算机内存进行精确写入。研究人员选择了一种聪明的策略：避免直接攻击沙箱本身，而是寻找沙箱"允许"但可能产生问题的区域。   
  
**TOCTOU攻击的精妙设计**  
： 研究人员在两个有效动画WebP文件之间切换，利用ImageDecoder的持久化解码器缓存了第一个WebP的帧几何参数和有效载荷位置，然后切换为具有不同几何参数的第二个WebP，导致解码器读取第二个WebP的压缩数据，但使用第一个WebP缓存的几何参数描述输出缓冲区。  
  
这种组合导致了边界越界写入，为下一阶段提供了关键衔接。  
### 第三阶段：Crashpad崩溃处理器漏洞（CVE-2026-78952）  
  
这是最精妙的跨沙箱利用，Crashpad作为Chrome的崩溃处理器，在沙箱外部运行。当请求快照时，它暂停渲染器并从渲染器的地址空间读取数据结构。研究人员发现了一个微妙的竞争条件：  
  
**核心漏洞**  
：Crashpad在假设渲染器进程地址空间保持不变的条件下执行两次读取操作之间的推理。但当渲染器能在两次读取之间改变映射时，这个假设失效了。  
  
**攻击手法创新**  
： 研究人员利用Windows的THREAD_CREATE_FLAGS_BYPASS_PROCESS_FREEZE  
标志创建线程，在Crashpad认为渲染器已冻结时，该线程可以继续运行并动态修改页面保护属性。通过精心设计的竞争攻击，最终实现了完全的代码执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZjpDmoEaBaLOkywtxweMLjHUaBicFjicLnpoI7rdnHDc3X0YSxQhj4MfiajW8VnkMFvzljrbpAE4AWREYNwXFibj8MPv0LvnicFOVUw/640?wx_fmt=png&from=appmsg "")  
## AI与人类协同开发模式  
  
XBOW团队强调了**清晰的职责分工**  
："AI代理搜索Chrome源代码、测试想法、构建概念验证并编写漏洞利用的第一稿。研究人员选择目标、提供背景、挑战代理结果，并决定哪些部分需要连接。"  
  
这种模式展示了AI作为"高级漏洞挖掘助手"的潜力，而非完全替代安全研究人员。AI负责繁重的代码搜索和模式识别任务，人类专家则提供战略方向、理解复杂交互，并做出关键的连接决策。  
  
**项目架构的智慧**  
：XBOW的设计策略也值得关注。他们将工作分解为并行活动，开发了备份路线，并为每个阶段设计了干净的接口，从而能够在需要时替换组件而无需重建整个利用链。  
## 版本碰撞的惊险时刻  
  
获奖的攻击链差点失败，因为其中一个依赖的漏洞后来被发现是谷歌已经知晓的。XBOW团队在技术报告中坦承："由于现在每个人——包括谷歌——都在将LLM指向像Chrome这样的代码库，撞洞会是一个重大风险。"  
  
值得称道的是，XBOW已经为类似情况做好了准备："我们设计了具有模块化阶段的漏洞利用，并试图为每个部分准备好备份漏洞。"这种策略最终拯救了项目。  
## 10亿用户面临的风险  
  
根据XBOW的评估，在这些漏洞被修复前，大约有**10亿运行Chrome的Windows用户**  
面临此类攻击的风险。  
  
**修复状态更新**  
：  
- CVE-2026-85045: 已于Chrome 152.0.7977.82 修复（9月3日）  
  
- CVE-2026-78952: 已于Chrome 152.0.7977.65 修复（8月25日）  
  
- WebP漏洞: 在同一Chrome 152周期中修复  
  
虽然Chrome会自动更新，但修复仅在浏览器重启后生效，因此建议重新启动Chrome并确认版本为152.0.7977.82或更高版本。  
## 行业影响深远  
  
谷歌的Chrome漏洞奖励计划在新规则下，每年最多颁发四个这样的全链利用奖励，单个报告最高奖金为**25万美元**  
。XBOW称这是第一个在新规则下的Chrome全链奖励，也是第一个授予自主安全公司的奖励。  
  
这一成就不仅刷新了漏洞奖励纪录，更标志着：  
1. **AI驱动的安全性研究已经成熟**  
，而不仅仅是概念验证  
  
1. **漏洞发现的效率已经大幅提升**  
，传统的时间周期被大幅压缩  
  
1. **防御方需要重新评估安全假设**  
，AI可能比人类更快地找到复杂漏洞  
  
## AI时代的网络安全新范式  
  
XBOW的研究成果标志着AI与网络安全融合的重要转折点，这不仅仅是技术突破，更是对整个安全行业思维模式的挑战：  
1. **效率革命**  
：从半年到六天的时间压缩，显示了AI在复杂系统分析中的巨大潜力  
  
1. **成本下降**  
：4万美元的成本使得尖端漏洞研究变得更加"大众化"  
  
1. **新协作模式**  
：AI与人类专家的协同工作将成为未来安全研究的标配  
  
1. **防御觉醒**  
：防守方需要意识到攻击者的工具和能力正在发生质的变化  
  
谷歌已修复所有漏洞，XBOW也确认这些漏洞从未被用于攻击Chrome用户。但这次研究的真正价值在于向我们展示了未来网络安全的发展方向：一个AI辅助、效率极高、变化极快的攻防时代已经来临。  
  
对于广大IT从业者和安全研究人员而言，理解AI工具的能力边界，掌握AI-人类协作的新模式，将是这个新时代的关键竞争力。毕竟，在安全领域，唯一不变的是变化本身，而现在这个变化正在以AI的方式快速发生。  
  
**参考资料**  
：  
- The Hacker News: Researchers Used a Public OpenAI Model to Build a $250,000 Chrome Full-Chain Exploit in Six Days   
  
- XBOW Labs: Zero to Calc in Six Days: Inside XBOW's Chrome Full Chain  
  
- Forbes: It Cost $40,000 To Find A Google Chrome Zero-Day Bug With AI   
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
