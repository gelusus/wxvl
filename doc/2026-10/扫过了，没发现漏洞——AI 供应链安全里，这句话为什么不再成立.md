#  "扫过了，没发现漏洞"——AI 供应链安全里，这句话为什么不再成立  
 锦岳智慧   2026-10-14 03:56  
  
AIBOM 管理系统：把 AI 资产从“一张清单”变成“一份可复核的证据”  
# 一、案例  
  
某个开源组件爆出高危漏洞，安全团队接到通知，需要在当天回答三个问题：  
  
1)  
我们哪些系统在用它？  
  
2)  
它在这些系统里是怎么被引入的？  
  
3)  
修完之后，怎么证明这批系统确实已经不包含了？  
  
这三个问题在传统软件供应链里不算难题——拉一份 SBOM，按组件名和版本检索，基本能给出答案。  
  
但把场景换成  
 AI 应用，事情就变了。模型的权重、微调适配器、数据集、提示词模板、RAG 知识库、MCP 服务、Agent 插件……这些组成部分散落在源码目录、供应商交付的压缩包里。  
而且  
不是静态的：上游模型更新了小版本，知识库被写入几篇新文档，一个新挂载的插件拿到了新权限——应用代码一行没改，组成已经变了。  
  
于是最常见的结果是：安全团队翻出三个月前那份评估报告，发现里面的清单已经对不上现在的情况。不是当初做错了，而是当初那份报告读的是“当时的清单”，而现在系统里存的是“现在的清单”。这就是 AI 资产治理最容易被忽略的一个陷阱——清单没变，结论变了。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mvkK67dLgZUXwMta9gt5xibWibQEoIVU5vIoneUIjxFtp64q5TNySHrll0G73sPpZHyTiaaVUxxZSmVPcsT23AJpVWpbvWKF6z7YgIchia4b4LQ/640?wx_fmt=png&from=appmsg "")  
# 二、实践过程遇到的困难  
  
AIBOM（人工智能物料清单）描述 AI 系统的组成、来源及关联关系，覆盖数据、模型、软件依赖、工具 / 插件 / MCP、算力 / 运行环境、服务 / API 等类别。具体能识别到什么程度，取决于扫描工具、输入材料与有效元数据；无法从输入中确定的信息，保留"未知"状态或通过人工维护补充——是"保留未知"，而非"默认为安全"。  
  
企业在应用中遇到的问题，可归为四类：  
  
一是资产分散。软件依赖、模型引用与外部交付清单分散于多个来源，人工表格易遗漏组件，且难以区分"扫描发现"与"人工补录"。二者可信度不同，混在同一张表里便失去了意义。  
  
二是版本变化。三个月前判定为安全的组件，今天可能已经被替换，而报告上的数字没变。  
  
三是风险定位。新漏洞披露后，需从组件及版本反查项目、目标与依赖路径。仅给出漏洞数量无法支撑修复排序——哪个先修、影响面多大、哪些实际未被真正引用，都无法从数字本身判断。  
  
四是证据判断。这是最危险的一类：身份缺失、版本未知、知识过期均会影响结论。若报告缺少明确的覆盖说明，读者很容易把"未匹配到漏洞"理解为"没有风险"，而实际上可能只是组件身份未识别、版本号缺失或漏洞知识库版本过旧。  
# 三、五个业务对象，撑起整套治理逻辑  
  
平台用五个核心业务对象组织数据。理解这五个对象，基本就理解了系统的设计思路。  
<table><tbody><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">对象</span></font></span></b></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">业务含义</span></font></span></b></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">与其他对象的关系</span></font></span></b></p></td></tr><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">项目与目标</span></font></span></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">组织权限和具体盘点范围</span></font></span></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">一个项目管理多个目标</span></font></span></p></td></tr><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">来源</span></font></span></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">压缩包或</span></font><span leaf=""> BOM </span><font face="思源黑体"><span leaf="">输入</span></font></span></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">同一目标可配置多个来源</span></font></span></p></td></tr><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">资产代次</span></font></span></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">某一来源一次发布的结果</span></font></span></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">保留该来源的版本轨迹</span></font></span></p></td></tr><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">当前清单</span></font></span></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">当前参与来源与人工信息形成的视图</span></font></span></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">随来源与治理信息变化</span></font></span></p></td></tr><tr><td data-colwidth="200" width="200" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">清单快照</span></font></span></p></td><td data-colwidth="333" width="333" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">某一时点固化的资产与依赖</span></font></span></p></td><td data-colwidth="246" width="246" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">为评估、复核和报告提供基线</span></font></span></p></td></tr></tbody></table>  
这里面最值得说的是两个：资产代次和清单快照。  
  
资产代次解决的是“来源自己的历史”。一个来源可以形成连续的代次，你可以回看这个来源什么时候更新过，也可以对比两次代次之间资产和依赖发生了什么变化。  
  
清单快照解决的是“结论的基线”。当前清单是活的，会随来源和治理信息变化；而快照是某一时点冻结下来的，专门用来给评估、复核和报告兜底。  
  
# 四、五项核心能力  
## 1. 多来源接入：把不同批次的材料放进同一个目标  
  
同一个检测目标可以登记多个来源，把不同交付批次、不同采集方式的材料纳入统一管理。来源有独立的名称、类型、启停状态和时间线。  
  
这样做的好处是，任何时候都能解释清楚“这条资产信息由哪份材料产生、何时进入清单”。  
  
两  
种主要接入方式：源码压缩包（接收研发或供应商交付的材料，上传建源后再提交任务）、  
BOM 文件（接收外部工具或供应商生成的清单，解析校验与源码扫描分别处理）。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mvkK67dLgZXQBMS42auv6Oy3Xw4XpxmR6u7jBYcB0uzdBqqwJvqgYza4vxsKFxJmVQFRx3Ja3xLEKMwo82t2jw2fyqYtKqRnicGq2nBrLxPk/640?wx_fmt=png&from=appmsg "")  
## 2. 资产身份管理：名字一样，不等于同一个东西  
  
名称相同的组件，可能来自不同生态，也可能属于不同版本。系统围绕  
 PURL 等身份信息与归一化标识组织记录，同时保留来源事实。  
  
有两条原则值得单独强调：  
  
保持身份含义。组件的生态、名称、版本及限定信息共同影响识别结果；按完整身份查询，能定位到具体实例；按包族查询，能观察同一组件的多个版本分布。  
  
保留来源差异。资产详情和使用位置可以关联来源、代次、  
BOM 制品及原始引用；不应把某个来源的声明，自动解释成其他来源也提供了相同证据。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mvkK67dLgZVibfofnnAmWmDsEFunMYVq7zumC6B0U3DsbsrtmfxQn2bibVfZiaTibiaGZ2Ub59yxEZRKmBQkqXHDnTrWyfTypFX31U4JNQNf2DBU/640?wx_fmt=png&from=appmsg "")  
## 3. 依赖图谱与组件：回答“它是怎么进来的”  
  
系统提供资产代次图、当前清单图和清单快照图，分别对应来源结果、当前组合及历史基线。  
  
图谱以资产为节点、依赖关系为边，支持查看依赖项、查看被谁依赖，以及双向展开。组件出现漏洞或需要升级时，可以从资产入口查看使用范围，按完整身份或包族观察版本分布，再定位到可见的项目、目标和具体使用位置。  
  
反向也一样：从使用位置解释引入关系，查看声明根、直接引入者或自定义起点，沿依赖路径定位目标组件，最短路径会高亮，用来解释“哪个上游组件把它引进来的”。  
  
有一条使用约束需要注意：查询期间如果当前清单发生了变化，应当刷新范围及路径信息，避免把不同版本的组件位置、来源事实和图结构，拼接成一条结论。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mvkK67dLgZWonyltKCL78CAut4VBCq9UiabIkR6qCubvLwuaFnt5hdzAVxsb7Jo05B2YAjkvM2e2bvSnqXQzJeoQZMR590ic5ClVtjqwBDM6Q/640?wx_fmt=png&from=appmsg "")  
## 4. 多引擎漏洞匹配：把“未匹配”和“无风险”分开  
  
资产清单为漏洞匹配提供输入。系统把知识库维护和项目风险发现分开组织，可管理  
 OSV 公共知识版本及私有漏洞记录，并展示知识源状态、活动版本、同步情况和匹配引擎可用性。OSV 以开放结构描述开源组件漏洞及受影响版本，为组件版本匹配提供基础信息。  
  
漏洞匹配任务围绕冻结清单开展，记录参与引擎与知识版本。多引擎用于汇集可用知识来源，但结果质量仍然取决于组件身份、版本信息和知识覆盖。  
  
系统区分三个统计口径：漏洞条目、风险发现、受影响资产。同一个漏洞可能影响多个组件位置，同一个资产也可能关联多个漏洞，不能混成一个“风险总数”来报。  
  
最关键的是覆盖状态，它直接参与结果解读：  
  
未评估——表示尚无评估依据；  
  
结果过期——表示既有结论与当前清单或匹配条件不再一致；  
  
部分覆盖——表示仅对有效部分形成结论。  
  
三个状态都在提醒同一件事：不要因为报告里没写“有风险”，就认为真的没风险。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mvkK67dLgZXMAV0t5161vCicUiaacEQAWlCYNBmicVzZhFfMxDKsvuyqAhbLuOfzUK8W5MfKy5JUeiaJR3mPq112ddzUcXpDL6owNV1X62S8jZM/640?wx_fmt=png&from=appmsg "")  
## 5. 资产治理与风险处置：人工确认不能替代技术验证  
  
系统支持人工维护资产和治理信息，区分扫描发现、  
BOM 导入、手工创建及多来源记录。人工确认与说明用于补足管理信息，已有事实按来源保留，人工补录不作为自动识别证据。  
  
两个状态字段的含义经常被误解，这里明确一下：  
  
“有效”表示属于当前清单，不表示正在运行。  
  
“已确认”属于人工审核结论，不表示已通过漏洞评估。  
  
风险处置要关联具体资产及匹配证据，需要核对影响条件、记录判断依据。组件或知识版本变化后，要通过新一轮清单和匹配结果复核——人工标记完成，不能替代技术验证。  
# 五、应用场景  
<table><tbody><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">应用场景</span></font></span></b></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">工作方式</span></font></span></b></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(204, 204, 204);background: rgb(213, 232, 240);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><b><span style="font-family: &#34;Times New Roman&#34;;font-weight: bold;font-size: 12pt;"><font face="思源黑体"><span leaf="">可形成的业务输出</span></font></span></b></p></td></tr><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><span leaf="">AI </span><font face="思源黑体"><span leaf="">应用资产盘点</span></font></span></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">汇集源码扫描与外部</span></font><span leaf=""> BOM</span></span></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">目标清单、来源事实和质量缺口</span></font></span></p></td></tr><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">新漏洞影响排查</span></font></span></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">按组件身份与版本定位使用范围</span></font></span></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">相关项目目标及依赖路径</span></font></span></p></td></tr><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">供应商交付审核</span></font></span></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">解析</span></font><span leaf=""> BOM</span><font face="思源黑体"><span leaf="">、核对身份与格式</span></font></span></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">问题明细、原始制品和审核依据</span></font></span></p></td></tr><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">版本升级复核</span></font></span></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">比较代次并重新开展漏洞匹配</span></font></span></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">资产变化与新评估结果</span></font></span></p></td></tr><tr><td data-colwidth="216" width="216" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">周期性治理复盘</span></font></span></p></td><td data-colwidth="300" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">固化清单快照并生成报告</span></font></span></p></td><td data-colwidth="263" width="263" valign="top" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(204, 204, 204) rgb(204, 204, 204);"><p style="margin-bottom: 0px;line-height: 1.6em;margin-top: 0px;"><span style="font-family: &#34;Times New Roman&#34;;font-size: 12pt;"><font face="思源黑体"><span leaf="">可追溯的阶段基线与处置记录</span></font></span></p></td></tr></tbody></table># 总结  
  
回到开头那个场景。三个问题——哪些系统用了它、它是怎么被引入的、修完怎么证明——之所以在 AI 时代变得难回答，不是因为工具不够多，而是因为“清单”这件事本身的性质变了。  
  
传统软件交付后即定型，清单是一张静态的快照。  
AI 应用交付之后仍在持续变化，清单更像是一条不断流动的河。所以治理的重点，也从“能不能扫出来”，转向了“能不能说清楚”：说清楚这条信息来自哪份材料、什么时间进入清单、当时的结论基于哪个版本的清单、现在的结论和当时相比变了什么。  
  
AIBOM 管理系统做的，本质上就是这件事——不是替你判断有没有风险，而是让每一次判断都有可核对的依据。毕竟在安全这件事上，最危险的不是报告里写着“有风险”，而是报告里什么都没写。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mvkK67dLgZXszoEze7Pfjad7DgHl8uINDBNgoaLxMrmE3tnSn2pMfcABviaJlQ3LJulu6fFDyjRX3Rp3em6eYGNI45EA9HrqVcFChRticw460/640?wx_fmt=jpeg&from=appmsg "")  
  
  
