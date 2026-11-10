#  SDC2026议题回顾 | ZeroMDASH：领域自适应的Agent-Native 通用漏洞挖掘工程  
SDC2026
                    SDC2026  看雪学苑   2026-11-10 09:59  
  
**SDC2026**  
  
议题回顾  
  
  
随着大模型能力不断提升，AI Agent 正在逐步参与复杂的软件漏洞挖掘工作。但面对超大型代码仓库和陌生技术领域，如何让 Agent 理解领域知识、建立有效的威胁模型，并持续完成漏洞调查与验证，仍是一个值得探索的问题。  
  
  
在第十届安全开发者峰会上，**安全研究员——C0ss4ck和Swing**  
分享 ZeroMDASH 项目，介绍其如何通过领域上下文准备、威胁建模、任务规划与多轮调查复核，构建 Agent-Native 通用漏洞挖掘工程，并结合 KVM 虚拟机逃逸和 Milvus 堆溢出两个案例，展示从研究线索到漏洞发现的完整过程。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K1WHnSDB6V9DdxuFW0pDj2klUXdYskktiaLQpt711Az96KLLIORvmQQJQKH6dfQia1MJ5vQzapOTZWncyetpu15URnD9oK3XF8uE/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K12bE4Wc3Zff6FsaxX5lDr4BSuWMWX6VURsCt1SJ5C8bEFXofVnoGACwyEibeaWcpuRhGzXUibk15aRH8MQkGpia2d3TCDJqOiaGr4/640?wx_fmt=jpeg&from=appmsg "")  
  
C0ss4ck（左）, Swing（右）  
  
安全研究员  
  
  
*以下为整理后的速记  
  
  
**01**  
  
**工程启发与系统总览**  
  
这次分享主要围绕五个部分展开：首先介绍已有 AI 驱动漏洞挖掘工程带来的启发，包括微软的 MDASH、谷歌的 Mantis，以及 ZeroMDASH 的整体流程；随后介绍领域上下文准备、威胁建模与任务规划、漏洞调查与攻击链复核三个主要阶段；最后结合漏洞案例，展示系统发现问题的过程。  
  
  
  
**1. 漏洞挖掘工程需要同时发散与收敛**  
  
如何实现一个能够持续运行数天、同时尽可能降低漏报的自动化漏洞挖掘工程？演讲中将漏洞挖掘抽象为一个求解过程，每个漏洞都可以看作一个解。为了尽可能多地获得有效解，系统需要先发散生成漏洞假设，尽可能覆盖攻击面和攻击路径，以降低漏报；同时，还需要从大量假设中进行收敛，筛选出真实存在且具有实际影响的问题，以降低误报。  
  
  
选择 Agent-Native 的方式进行静态代码漏洞挖掘，一个重要原因是传统 SAST，或者由大模型生成 SAST 规则的方式，会损失大量上下文。代码 Agent 已经能够完成搜索函数、阅读实现、编译项目、编写触发器和 PoC、调试 GDB 等专业且长程的工作，因此可以让 Agent 直接参与完整的代码调查过程。  
  
  
不过，简单地向 Agent 提出“帮我找一个漏洞”仍然存在明显不足。任务往往没有说明攻击者是谁、攻击者能够访问哪些入口、关注哪类安全影响，以及项目历史上出现过哪些相关问题。在超大型代码仓库中，尤其是面对不熟悉的领域和工程架构时，这种许愿式挖洞很容易产生漏报，而且漏报通常难以从最终报告中直接看出来。  
  
  
  
**2. 从 MDASH 与 Mantis 获得的工程启发**  
  
微软的 MDASH 将漏洞挖掘流程划分为 Prepare、Scan、Validate、Dedup 和 Prove 五个阶段。准备阶段建立仓库知识库，收集威胁模型、调用关系和历史补丁等信息；扫描阶段根据 CWE 类型选择相应 Agent；验证阶段过滤误报；最后通过复现和测试输入证明问题。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K20fIuzz1Jq89xrwKYF3BQicNqccSOxXTbtibhcGrHiaZZt5y4ucK3Cmm3c2TD7LSt9HH8VepmpPHTOvqC50xNVLYBWVQMmexBlrQ/640?wx_fmt=png&from=appmsg "")  
  
MDASH 的执行生命周期  
  
  
MDASH 带来的启发包括：先构建仓库知识库，再开展漏洞扫描；知识库不仅包含代码结构，也包含威胁模型和历史补丁等信息。此外，多个模型之间的讨论与投票机制也值得参考。  
  
  
谷歌的 Mantis 将工作拆分得更细，从历史漏洞提取、结构索引、架构分析，一直延伸到威胁建模、研究复现、攻击链分析和修复。它有两条值得关注的反馈路径：一条是在修复后重新验证，另一条是检查补丁是否留下绕过路径。同时，报告中形成的新认知会反馈到架构分析阶段，使下一轮分析能够使用更新后的上下文。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1YGK5jhN5gAtjDlVj370FIDr4IeDE9Rv6ujPmgKsdW9Cz6XDeicvG46jNsxNSoWHC4Lompf4fu2ibGTv14GQic0U6erNsGLDr5lA/640?wx_fmt=png&from=appmsg "")  
  
Mantis 工作流  
  
  
这对 ZeroMDASH 的直接启发是：漏洞挖掘过程应当持续补充对目标软件的理解。新发现的入口、对象关系和状态变化，不应只留在某一个疑似漏洞点的调查过程中，而应沉淀为后续任务也能使用的上下文。  
  
  
  
**3. ZeroMDASH 的整体流程**  
  
ZeroMDASH 从代码仓库及扫描范围开始，既支持扫描整个仓库，也支持针对 Merge Request 的 DIFF 代码开展分析。准备好扫描对象后，系统收集五类领域上下文，再据此生成威胁模型并识别攻击面。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K3oM6UgIicg1KSnqp2xnQlzuDXkM1Q6W0HUXWicickibiaV4uSmvib8b8XJnF6LfA7S69hqbhQqnJ9oRvyiaG21OeqTicgoGpU4ib3gxibl0/640?wx_fmt=png&from=appmsg "")  
  
ZeroMDASH 整体架构  
  
  
随后，系统通过不同规划器生成子分析任务，包括 plan_by_feature、plan_by_spec、plan_by_vuln 和 plan_by_patch，分别围绕新功能、规范、相似漏洞和历史补丁组织调查。每个任务都需要说明调查目标、初始漏洞假设集合，以及必须分析的代码片段和调用路径。  
  
  
调查阶段采用辩论式的调查与复核流程。已经确认的问题会进一步区分为独立漏洞，或可以组合使用的漏洞原语。漏洞原语会记录其前置条件，以及能够破坏哪些安全不变量、带来什么能力；系统随后检查不同原语能否组合成完整的漏洞利用链。每个节点都可以分别配置模型和 Harness。  
  
  
**02**  
  
**Context Preparation：领域上下文准备**  
  
在威胁建模之前，系统希望负责建模的 Agent 能够获得尽可能优质的漏洞研究上下文。演讲采用的思路是：先观察没有 AI 时，安全研究员通常如何开展漏洞研究，再将这些研究线索整理为可供系统使用的上下文。  
  
  
  
**1. 从安全研究员的真实研究过程提取线索**  
  
团队整理了不同领域顶尖安全研究员的公开资料，包括论文、演示文稿、访谈和社交媒体发言等。最初以约 100 名研究员作为抽样范围，最终保留了 78 份公开材料。结合公开作品、深度对话、社区活动和外部视角，研究人员追踪研究过程中的决策及其变化。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1dju2ovmJddz1PiarBLyzhFgRjaK8cebeeMib7g1sXBcAwpK7BmsPzf8icGwnquGyscEY06G65hO5PT24bH8YGVtnv0GyZ7gfU1E/640?wx_fmt=png&from=appmsg "")  
  
研究员研究与复核流程，以及研究员与领域归属网络  
  
  
研究员与领域之间并非严格的一一对应关系。例如，同一位研究员可能同时研究浏览器、内核和虚拟化，因此不同领域之间存在重叠。整理这些资料时，关注的不只是研究员最终发现了什么，还包括研究如何开始、为什么选择某个功能点、为什么沿着某条代码路径继续追踪，以及什么证据促使研究方向发生变化。  
  
  
  
**2. 五类常见研究方向**  
  
对研究员样本进行整理后，演讲归纳出五类反复出现的研究方向，并将其作为 ZeroMDASH 准备领域上下文的主要来源：  
  
  
第一类是目标软件历史漏洞。关注目标软件过去出现过哪些漏洞，以及修复或重构过程中是否留下回归问题。  
  
  
第二类是相似软件历史漏洞。通过相似软件中已经发生过的问题，寻找目标软件中对应的实现和信任边界。迁移的是安全问题及查证思路，而不是直接迁移漏洞结论。  
  
  
第三类是 Fuzz Driver 开发。关注现有 Fuzz Harness 能够生成哪些输入，以及这些输入与产品真实入口可达的输入之间存在多大差距。  
  
  
第四类是相关功能与标准之间的 Diff。对照规范承诺的安全语义，检查实现中的各条代码路径是否一致满足。  
  
  
第五类是新 Feature 安全研究。新功能可能引入此前不存在的能力或默认行为，需要检查它是否改变了既有安全假设。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0CXViaVG3GbZhfibP6emIib7goocSZweRSiaO8dLrrY5zSwecKibjEWcWBP6ocicyDNdWqVmaicfpxqnuBjUwLSVA0ZoicGVsl1KZH7wI/640?wx_fmt=png&from=appmsg "")  
  
研究员样本中的五类常见研究方向及统计  
  
  
在演讲展示的样本统计中，不同领域对五类方向的关注比例有所差异。这些比例用于描述样本中的研究分布，并不意味着某一类研究方向天然比另一类更容易发现漏洞。重要的是，这五类研究方向在真实研究中反复出现，而且研究结果通常能够进一步转化为具体的代码分析任务。  
  
  
  
**3. 五类上下文如何转化为下游任务**  
  
目标软件历史漏洞的代表案例是 regreSSHion（CVE-2024-6387）：相关保护机制在 2006 年得到修复，2020 年重构时保护被移除，最终在 2024 年再次暴露。对应的查证问题是：哪次重构删除或绕过了哪条安全不变量？ZeroMDASH 会通过目标软件历史漏洞搜索生成补丁相关任务，并在仓库内验证 commit SHA，进一步派生针对缺失保护或绕过路径的复核子任务。  
  
  
相似软件历史漏洞的代表案例是 0.0.0.0 Day。多家浏览器最初都未将 0.0.0.0 识别为本地目标。对应的查证问题是：同一信任边界在目标软件的实现中，具体由哪段代码负责把守？系统会搜索相似软件并生成同类实现对照清单，迁移安全问题，而不是直接认定目标软件存在相同漏洞。  
  
  
规范与实现 Diff 的代表案例是 MadeYouReset（CVE-2025-8671）。HTTP/2 层已经关闭请求，但后端任务仍继续消耗资源。对应的查证问题是：规范承诺的安全语义，是否在每一条代码路径上都得到一致实现？系统会把规范条款逐条转化为核查任务，并与具体实现对照。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K3lqlBSOhWHGibZeVB93TA4GBmJkY9NyL4OPdu5mjU4rIaSgzibel9MzKmtNSAr2ctC5qJxGibZLV0gDquwxaYP7AdtyT79bny2Mc/640?wx_fmt=png&from=appmsg "")  
  
五类研究方向、代表案例及 ZeroMDASH 下游任务  
  
  
Fuzz Driver 开发方面，演讲以 OSS-Fuzz-Gen 与 OpenSSL（CVE-2024-9143）为例：发现的问题在 Harness 中成立，但产品路径不可达，因此影响有限。对应的查证问题是：Harness 能生成的输入，与产品真实入口可达的输入之间有多大差距？Fuzz Driver 的研究结果会用于指引攻击面分析，区分已经明确可达和仍未覆盖的攻击面。  
  
  
新功能安全研究方面，演讲提到 MSG_OOB 与 CVE-2025-38236：默认开启的新 flag 与沙箱放行组合后，使 renderer 能够向内核提权。对应的查证问题是：新功能默认开启或放行了什么能力，又打破了哪些既有安全假设？缺少规范约束、Fuzz Driver 覆盖或历史漏洞研究的新功能，可能成为新的攻击面。  
  
  
  
**4. Harness 与 Model 的横向评估**  
  
演讲还展示了不同 Harness 与 Model 组合的横向评估。图中横轴表示 Deep Research 结果的上下文丰富度，纵轴表示基于这些上下文生成的漏洞假设质量，评分经过归一化处理。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0ncsibcLNrEWI5MCb90y701gyG7WjyeibtG2K6gog503RsGeaZ9EBwLccDBGXYZ40uBXkfw3v4ghyKKbv739XHhPFjzAyKS5Nzc/640?wx_fmt=png&from=appmsg "")  
  
不同 Harness 与 Model 组合的横向评估  
  
  
从展示结果看，Codex 与 GPT 系列模型的表现相对稳定；部分国产模型在漏洞研究领域上下文的 Deep Research 任务中，结果出现了较明显的幻觉，也有组合表现不够稳定。因此，在准备漏洞挖掘上下文时，预算充足会优先选择 GPT；预算有限时，则会增加 DeepSeek 的运行轮次。  
  
  
**03**  
  
**Threat Modeling：威胁建模与任务规划**  
  
不同领域的软件代码差异很大，因此需要一种相对通用的描述方式来表达威胁模型。ZeroMDASH 参考既有威胁建模方法并作了拓展，将可能出现漏洞的位置、代码对象及其状态变化组织为结构化模型。  
  
  
  
**1. 统一描述威胁建模与疑似漏洞点**  
  
在威胁建模的基础描述中，系统关注入口点（Entry Points）、信任边界（Trust Boundaries）、解析器或状态机热点、Sink 类别，以及漏洞假设清单。这里的 Sink 类别并不是固定的 CWE 类型，而是对潜在漏洞影响或漏洞类型的一种宽泛描述。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0qJ7yvterlNOCbyBzawuXl5OMMOXict87fVOsf7FyfkHXpI9GAzPw02TNMAdvvoxlIz3jia9m1X9ZmQe5auXKY9rQxN3mOxgx90/640?wx_fmt=png&from=appmsg "")  
  
威胁建模：统一描述威胁建模与疑似漏洞点  
  
  
漏洞假设不能脱离代码凭空产生，因此还需要从目标代码的疑似片段中提取实体（Entity）、表示（Representation）、状态迁移（Transition）、环境（Environment）和 Seam 等信息。Entity 表示当前代码片段涉及哪些对象；Representation 表示对象以什么形式存在；Transition 描述对象处于什么阶段，以及什么操作会使其进入下一阶段；Environment 表示状态迁移前后必须保持的安全约束；Seam 则是状态迁移边界上可能出现理解差异的位置。  
  
  
Representation 还可以区分为权威态与派生态。权威态可以理解为判断时以其为准的状态，派生态则是从权威态派生出来的对象。例如，磁盘上实际存在的文件内容是权威态，内核中的 inode 和 page cache 是派生态；数据库中记录的用户权限是权威态，页面展示的用户权限列表则是派生态。  
  
  
系统会利用上述描述生成结构化领域模型，并将模型中的 ID 注册到攻击面 Markdown 文档中，供后续任务引用。  
  
  
  
**2. MySQL 示例：从领域模型生成漏洞假设**  
  
演讲以 MySQL 的一个案例说明领域自适应威胁建模。演讲者说明，自己并没有数据库领域的漏洞挖掘经验，这个案例是 ZeroMDASH 自动运行得到的结果。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K1Dibjfl9NFqibn5ktXwzU5q5LRQ9lYKNouwz2jVx8WXKjJQoK7RV87n8ACH3sIGVEiaPshfGKjYiaGMXoOhRIBPlATd9VPR3rDljc/640?wx_fmt=png&from=appmsg "")  
  
以 MySQL 某漏洞为例的领域威胁建模  
  
  
案例涉及 MySQL 相关版本引入的 JSON Duality View 特性。其 DML 路径会将用户提供的 JSON 值转换为内部 SQL 语句。字符串值先经过 escape_string_for_mysql 进行反斜杠转义，随后被封装为 UTF8MB4 字符串字面量。  
  
  
问题在于，转义过程假设反斜杠具有转义语义，而后续 SQL 解释器实际遵循当前会话的 SQL mode。当低权限 MySQL 租户设置 NO_BACKSLASH_ESCAPES 模式时，解析器会把反斜杠视为普通字符。此时，转义器生成的反斜杠加单引号不再能够保护单引号，攻击者控制的引号可能提前结束内部字符串，并生成新的 SQL Token。  
  
  
JDV DML 入口在生成和执行语句前会安装相关 View Context，因此注入语句可能在 View Definer 的安全上下文中执行，而不是以租户本人的权限执行，从而形成 MySQL 内部的权限提升。  
  
该场景中的攻击者是已经认证的低权限 SQL 租户。前置条件包括：租户能够设置 Session SQL Mode，并能够对相关 JSON Duality View 执行 INSERT、UPDATE、DELETE 等语句；同时，View Context 有效，Definer 对底层对象具有高于租户的权限。攻击输入则是设置 NO_BACKSLASH_ESCAPES 后提交的恶意 JSON 字符串值。  
  
  
在领域模型中，相关实体包括转义器、词法器和模式守卫；权威态是由 SQL mode 决定的转义语义，派生态是生成 SQL 文本中的转义序列。需要保持的不变量是：生成的 SQL 不应将租户输入重新解释为 SQL Token。演讲中提到，生命周期迁移打破了这一不变量，并由此形成了若干 SQL 提权漏洞假设。该问题当时尚未公开，演讲者表示暂未提交报告。  
  
  
  
**3. 将威胁模型转化为扫描任务包**  
  
得到初始漏洞假设后，系统通过规划阶段生成调查任务包。演讲展示了 KVM 案例中的任务包生成过程：plan_by_spec 会结合规范与威胁模型形成相对详细的初始漏洞假设，并引用领域环境注册信息、攻击面 Markdown 等结构化材料。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0D7B9OHzw8jYt7sAc2JM2ZW2C6bkj2lQNt5hz2I3t8sSKYJyvWAGaAtvqyMsXNAunJlOE4SQXAjyUIPgfJCnibDHbUAzHsfze8/640?wx_fmt=png&from=appmsg "")  
  
将威胁建模转化为漏洞扫描任务包  
  
  
任务包会明确调查目标，列出必须覆盖的代码分支，提出初始的、可证伪的具体漏洞命题，并提供供后续子任务使用的提示信息（Hints）。任务规划的重点，是让后续调查知道需要验证什么、必须覆盖哪些代码，以及哪些条件决定假设是否成立。  
  
  
**04**  
  
**Investigation：漏洞调查与攻击链复核**  
  
任务规划完成后，系统进入调查与复核阶段。这里关注的核心问题是：如何让每一个肯定或否定结论都能够经受多轮复核。  
  
  
  
**1. 围绕 Hypothesis 展开调查与复核**  
  
系统首先生成初始漏洞假设，由 Investigator 执行调查。调查过程中如果发现新的假设，会将其提交给 Critic。Critic 判断新假设应当被接受、合并到已有假设，还是直接拒绝。被接受的新假设会进入假设处理流程，继续开展递归式调查。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1yibUgRnRp7lefcpXLe5QqjB5co34zsqLkMnKxicC1q4doqJ0unxoorfiblxIQKn51F5IY960bFFum7MlhKicbpZtcm9CW6zrKx9U/640?wx_fmt=png&from=appmsg "")  
  
围绕 Hypothesis 的调查与复核流程  
  
  
除新假设的派生外，Investigator 与 Critic 还会围绕证据和结论进行多轮讨论。最终，一个假设可能被认定为独立漏洞；如果不足以构成独立漏洞，则进一步判断是否构成可利用的漏洞原语；如果既不是独立漏洞，也不能形成漏洞原语，则该假设会被放弃。  
  
  
  
**2. 组合漏洞与漏洞原语**  
  
漏洞原语并不会在确认后直接丢弃。系统会尝试组合不同漏洞原语，判断它们能否形成新的漏洞利用链，并进一步评估组合后的实际危害。每个原语会保留自身的前置条件和能够提供的能力，以便后续分析其是否能够与其他原语组合。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K20t5zM0lOTwwa9YgsQfPBPZAy1ZFknugFBBuib4m1OZ82hJuGqjicwGJlXibQT4ODCWGlHxodj1osaMNYdXq2Z0ISSZxY56icJDR0/640?wx_fmt=png&from=appmsg "")  
  
组合漏洞与漏洞原语的调查流程  
  
  
**05**  
  
漏洞案例展示  
  
演讲准备了三个案例，但受现场时间限制，主要展开了 KVM 和 Milvus 两个案例。演讲者表示，第三个案例未能在现场详细介绍。  
  
  
  
**案例一：KVM CVE-2026-64555——SPSR 污染导致 ERET 到 EL2**  
  
该案例涉及 ARM64 KVM 的嵌套虚拟化场景。Guest VM 内能够触发特权级越权，最终导致虚拟机逃逸。演讲中指出，ARM64 的相关嵌套虚拟化场景当时尚未在云厂商中广泛普及，因此该问题属于提前发现的漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3cfgs2LY7RJRHibwluO0GFibpSyTqb8an7GTJoIGwvksLdH8pj4m1BCTTLsuum0oy1y8672FtzOV11ENuicB7nz63ImNJ210N7vM/640?wx_fmt=png&from=appmsg "")  
  
KVM ARM64 嵌套虚拟化场景与异常级关系  
  
  
VHE 是 ARM64 的 Virtualization Host Extensions（虚拟化宿主扩展）。它使运行 KVM 的宿主 Linux 内核可以直接运行在物理 EL2，同时尽量保持原来运行在 EL1 时的编程模型。在嵌套虚拟化场景中，Guest 还可以运行自己的 Hypervisor，因此 KVM 需要保留 Guest 中虚拟 EL2 的相关状态。  
  
  
案例涉及 Armv8.8 的 FEAT_MOPS。MOPS 为 memcpy/memset 提供了一组指令，一次操作通常由三个阶段组成：CPYP/SETP prologue（初始化）、CPYM/SETM main（执行主要复制或填充）以及 CPYE/SETE epilogue（收尾）。主指令或尾声指令可能因 CPU 迁移、对齐或大小要求等原因发生 MOPS 异常。按照 Arm 架构要求，异常处理程序需要恢复源地址、目标地址和剩余长度，使其符合 prologue 的要求，将 PC 回退到 prologue，并重新执行整组 MOPS 指令。KVM 需要在 Hypervisor 内处理此类异常。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K21bwN17nUUiacAicgF8XroSSafxVKuoC2J0kve8cg5ylMKwtg3ej8JdyQa5vXz2k9S7dGevZuCHKQN6QVJr8IVapH6xicw7EWicnw/640?wx_fmt=png&from=appmsg "")  
  
内存中的 pstate 副本与硬件 SPSR_EL2 的两种表示  
  
  
漏洞的关键在于：内存中的 pstate 副本与硬件寄存器 SPSR_EL2 是同一份状态的两种表示，正常路径必须经过翻译与检查。正常情况下，软件中的虚拟 EL2 状态需要转换为硬件能够正确执行的状态；如果没有经过翻译，就可能把虚拟 EL2 状态直接写入硬件寄存器。  
  
  
在漏洞路径中，同一趟退出处理先将状态改回虚拟表示，随后 MOPS 异常处理程序又将其原样写回硬件寄存器。结果是，原本应保持 EL1h 的状态被错误写成 EL2h，Guest 指令因此能够以 EL2 特权级执行。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2ZCRw06r6IzepvgCiaWkUkwicic98edoZicdW3fdRtqBiccIccqEHgN3fHmqh2ic0XzKYNx36f1xzpibvPI6TBiaakxtfmw3N9vrpKuibc/640?wx_fmt=png&from=appmsg "")  
  
vEL2 未经翻译并被写入 SPSR_EL2 的漏洞路径  
  
  
演讲展示的代码路径涉及 vhe/switch.c、switch.h 和 entry.S。相关处理先读取硬件状态，再清除 SS 位，随后将状态写回 SPSR_EL2。问题在于，写回时使用了未经正确翻译的虚拟状态，导致硬件 SPSR_EL2 中的异常级信息被污染。  
  
  
修复思路是避免将错误的虚拟状态直接写回硬件寄存器，而应按照正常路径单独处理需要清除的状态位，并保持正确的异常级表示。演讲将其归纳为：新功能引入时没有复用既有 mitigation。PPT 还列出了 SPSR_EL2 的多个写入点，其中 MOPS 特殊处理路径是本次问题所在。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0JuJnf1xoll5ONfd4XYeK7xzbL81wgTyQdds0ialFV0KiaypVIfMhBvgK88fNKWUx8uIN5GS19C9NGibhOueNvow3pbWthVNsN4M/640?wx_fmt=png&from=appmsg "")  
  
SPSR_EL2 写入点及 MOPS 特殊路径  
  
  
PoC 展示了 Return to ArbitraryAddress 的效果。Kernel Panic 日志中的 PS 值为 00000009，对应 PSTATE=PSR_MODE_EL2h，说明执行流已经通过 ERET 返回到 EL2；PC 值为 0x400000028，来自 Guest 且由 Guest 控制，因此可以实现 ret2addr。演讲者进一步说明，后续能够以 Host 的 EL2 指令执行，并找到可用于 ROP 的 gadget；完整 POC 未公开。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2NYVf4wEej5k4taPEtw2RDL6YRGqkWRjoLhRiaxIw5yiadbZQQQWMP77TbgnrUZ1Bd2ogcNyZXhHkcW82HKuAJxnDoAb2bQozmE/640?wx_fmt=png&from=appmsg "")  
  
PoC：Return to ArbitraryAddress  
  
  
发现过程方面，系统受到 KVM 历史漏洞 CVE-2018-18021 的启发。相关 writeup 将其归类为 Guest 设置 PSTATE.M 或 core register file 时的输入验证问题。ZeroMDASH 根据这些历史上下文提出假设，再通过多轮调查和复核逐步收敛，最终确认本次漏洞。  
  
  
演讲展示了两轮审计与复核：第一轮调查未能确认问题，Critic 提出需要进一步核查 SPSR 与 MOPS 异常处理之间的状态关系；第二轮调查补充了关键证据，复核方确认漏洞成立。演讲者还介绍，发现过程本质上是沿着需要状态翻译或检查的路径逐一排查，最终找到缺少检查或返回约束的位置。  
  
  
  
**案例二：Milvus #53759——Sparse Search 堆溢出**  
  
第二个案例来自向量数据库 Milvus。漏洞位于 Search/HybridSearch 的查询行 C++ 解码过程：查询行在进入检索之前就可能触发越界写。Sparse Search 只保存和匹配非零维度，输入为稀疏向量 q 与 topK，输出为 Top-K 文档 ID 与得分。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K02Heo0Co50GLNz3zm3blRibjwY2WibWvNxiaphrR64lbAfzYfO8huRiaunsiaicTckjglDaXNvIwxUJ3XR7XtwvIMz4gk1EHdRSnd40/640?wx_fmt=png&from=appmsg "")  
  
Milvus Sparse Search 的查询流程与行编码  
  
  
Sparse 查询行通过 PlaceholderGroup 传入，编码形式为（uint32 index，float32 value）× NNZ，每项占 8 字节。行长度由输入控制。漏洞代码按整项数量分配内存，却按照原始长度复制数据，并且长度校验发生在 FastMemcpy 之后。  
  
  
具体而言，代码先计算 n = size / 8，再创建 SparseRow(n)，因此分配的空间为向下取整后的整项长度；随后 FastMemcpy 按原始 size 复制数据。若输入长度为 8k+r（k≥1，1≤r≤7），复制操作就会越界写入 r 字节。以 9 字节输入为例，程序只分配 8 字节，却复制 9 字节，造成 1 字节越界写。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0Esrggyia4ZRYG4dkzriaHYFYe2C1qcjNUnT2DKEF7eyWynjYssfuMf6r3F4AYN7kdYNhh9NBa79hNBQ8tna9fq5Na8KTGmFSMc/640?wx_fmt=png&from=appmsg "")  
  
按整项分配、按原始长度复制导致堆越界  
  
  
修复方式是调整执行顺序：先检查 size 是否能够被 8 整除，不满足时直接返回数据格式错误；通过检查后再计算数量、分配内存并复制数据。也就是将原来的“分配—复制—校验”调整为“校验—分配—复制”。  
  
  
演讲还展示了漏洞利用过程，并说明最终能够实现任意命令执行，同时绕过 ASLR。现场演示视频未能正常播放，因此没有展开完整利用细节。  
  
  
  
**Milvus 案例的发现过程**  
  
该问题的发现受到 Qdrant CVE-2023-38975 以及未覆盖 Fuzz 的情况启发。相似软件历史漏洞提示，攻击者控制的 vector size/count 可能触发 allocation 或 arithmetic 相关问题；同时，Milvus Query Parser 当时没有 Fuzz coverage。  
  
  
系统早期生成的攻击面和初始假设看起来与最终结果相距较远。它从公共 Search/HybridSearch 请求中的 PlaceholderGroup 入手，分析 SearchRequest、PlaceholderGroup protobuf、repeated Placeholder 和 repeated bytes values。Agent 注意到，values_size() 和 values(0).size() 都来自客户端提供的数据，并参与 C++ 内存分配，因此提出需要确认 Proxy 是否在进入 C++ 前约束了 nq、dim 和 value_size 的组合。  
  
  
经过三轮调查与复核，系统最终定位到 SparseFloat 的 Search 路径，并确认了上述堆溢出问题。演讲中展示了三轮调查和 Critic 复核的结果，说明初始攻击面并不一定直接指向最终漏洞位置，调查过程需要不断补充证据、调整假设并进行复核。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3EUY9hKO6jl3YBFQkMElftc7zP8EfaLz2s0SUvBgdkQ5dnwCaAX76QTd3FibgEm9PpSOa7D7Wpsu9oYumMsicKJLy8sbCicvstZ0/640?wx_fmt=png&from=appmsg "")  
  
三轮审计与复核后定位到 SparseFloat Search  
  
  
**06**  
  
**项目与作者**  
  
ZeroMDASH 项目仓库： https://github.com/JANUS-SEC/ZeroMDASH  
  
  
作者主页：  
  
https://github.com/Cossack9989  
  
https://github.com/winmin   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0yVPbkLHz8oj4ZdjWIl5Qaa2AVNY3aicW1g6LicDa7EmPRXXF3xu2OpAJASfxTVjSJ3WXibwr2wkhLrQicmRNllcZc7OxNWq9Ptg8/640?wx_fmt=png&from=appmsg "")  
  
  
以上为本次演讲中介绍的 ZeroMDASH 工程流程、领域上下文准备方法、威胁建模与任务规划机制，以及 KVM 和 Milvus 两个漏洞案例。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2rrnFpZ0qK9BTVs8LctiblwcO6KdnebBtZ21U0n12j2Eiacdiaicar1IpF4iauWCVyJuoEzBrVcicTp1sBD1qWcpmTTEcX7cMoxdkc0/640?wx_fmt=gif&from=appmsg "")  
  
往期议题回顾  
  
[SDC2026议题回顾 | 从 LLM 到 Agent：AI 安全攻击面的过去、现在与未来](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623168&idx=1&sn=c067079a13953815453d84054be125f5&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | 一次渗透 Agent 的架构减法实践：大模型、工具和Harness谁更重要？](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623198&idx=1&sn=a2f7444454ef6dd0e7fd9774e731b005&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | Storm: AI审计时代下，Fuzzing还有用吗？](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623398&idx=1&sn=5313300d03566b0a72a9824d15c96d4e&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | 从漏洞检测到利用：知识驱动的 RISC-V 硬件安全分析技术](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623417&idx=1&sn=6ba9895d501494c5a154b6a8ce5e02ab&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | 大语言模型驱动的移动软件安全分析实践](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623607&idx=1&sn=01da4724621cdfbc360ddb2122f69bab&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | 从字节到神经元——一名二进制安全研究员的 AI 安全研究](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623854&idx=1&sn=477001d7aa1d29a1da1ae75c6d8457c3&scene=21#wechat_redirect)  
  
  
  
  
**PPT下载及视频回放**  
  
  
峰会议题PPT及回放视频已上传至  
看雪课程-【看雪2026 SDC】：  
  
https://www.kanxue.com/book-section_list-107.htm  
  
  
【已购票的参会人员可免费获取】：  
我方已通过手机短信发送“兑换码”  
  
  
免费获取方式：  
  
1. 登录看雪账号，进入上述  
课程页面。  
  
2. 点击“立即购买”  
  
3. 输入大会于 2026年10月27日 通过手机短信发送的  
兑换码。  
  
4. 提交订单后，即可免费开通课程查看权限，获取大会演讲PPT及视频。  
  
  
*温馨提示：使用兑换码无需支付任何费用。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K2fMW7qHwG4aJUYnVWHgibFtXQM22vADs9JvK6AicxAfnV8pAd2g51tDRSTFACejoaQwYRaenR1XqicdIPek4BpibH5RcmQ063s4ac/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0SMzOMxQskNljba2qUU2f3EduYGF67Z89U6Snnup3fVgl5hEibS0uCuA8Ko7M9grYcviarQeLA1QePtQY8hHnSDDmzV2QiaQXpMQ/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K1yxO0k4aN2j9X6NxaRcNOzZB0AbPmhD1UzZxyAnNrb9PuxrN5UOw1wfktha6cFZPMbfZApT4yrzE8gBDQScY3LCo8RdFytXTc/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2XUsKibTUTJe7AIWA6lwQ5oc3bUzWaicRKE16ghuxE17pc1iayuXSr8KNENxyM6mHo87LvCxicUyric3ia9PhIKh31VaAicgia5FcuUBs/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0MyviadWicfiaWqH5zhQMqxUWZ7wmN98ohCyoklERr0CYm9ftPLLSbzM3pichm2e3uIoziaWpyq0MQyibLGuKHv18jkLBCm21N8yNo0/640?wx_fmt=gif&from=appmsg "")  
  
点击阅读原文查看更多  
  
