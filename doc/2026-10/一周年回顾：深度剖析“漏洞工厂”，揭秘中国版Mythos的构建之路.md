#  一周年回顾：深度剖析“漏洞工厂”，揭秘中国版Mythos的构建之路  
 安恒信息   2026-10-12 01:06  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icVz8RbowK3zzWCaicK1LPbSTJDDcicxleNaqXXSxYPFppNTsB1z02AlkKRMHgtzdCselqbaOWGGfYJPqibodouxRQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
2025年4月，卫兵实验室一位00后研究员提出了一个大胆的想法：  
  
能不能把漏洞挖掘做成一座“工厂”？  
  
  
这个想法在当时听起来有些异想天开。  
  
漏洞挖掘长期以来是一项高度依赖专家经验的工作。面对一个陌生项目，研究员需要理解代码、梳理调用关系、寻找攻击路径、构造漏洞假设，再通过环境验证证明漏洞真实存在。  
  
一个漏洞的发现，往往是一次复杂的“抽卡”。  
  
如果换一个项目、换一种语言、换一个框架，研究员可能又要从头开始。  
  
但AI的出现，让我们第一次看到了另一种可能：  
  
如果让AI持续理解代码、持续提出漏洞假设、持续完成分析和验证，漏洞挖掘能不能从“专家项目”，变成一种可以持续运行的生产能力？  
  
  
2025年6月，这个想法正式被定义为“漏洞工厂”专项；9月，漏洞工厂开始内部运营。  
  
一年之后，我们已经走过了从技术探索到产品化，再到真实环境验证的完整过程：  
2026年2月，首款产品级应用  
恒脑AI代码审计智能体上线；  
7月，第二个产品级应用  
零日雷达上线。  
  
回头再看，“漏洞工厂”这个名字，已经不再只是一个大胆的设想。  
  
它逐渐变成了一套我们每天都在真实运行、真实验证的漏洞挖掘生产体系。  
  
  
01我们为什么要造一座“漏洞工厂”？  
  
我们并不认为，给代码审计接入一个大模型，就等于拥有了AI漏洞挖掘能力。  
  
真正的问题是：  
  
如何让AI持续找到漏洞，而且找到的漏洞能够被证明、被复现。  
  
  
这背后至少有三个基本判断。  
  
第一，漏洞挖掘真正稀缺的，不是代码，而是上下文  
  
一个孤立的代码片段，很难说明它是否真的存在漏洞。  
  
漏洞可能隐藏在跨文件的调用关系里，也可能隐藏在权限、租户、所有权、状态机和额度等业务关系中。  
  
因此，漏洞挖掘的第一生产力，不是让模型“看更多代码”，  
而是让模型获得足够高质量的上下文。  
  
第二，AI负责“想”，确定性引擎负责“证”  
  
AI擅长生成假设，也擅长组织跨文件、跨模块的语义关系。  
  
但“我认为这里可能存在漏洞”，和“这里确实存在一个漏洞”，是两回事。  
  
前者需要推理，后者需要证据。  
  
因此，我们没有试图让AI包办一切，而是让AI负责漏洞假设和复杂语义推理，再由确定性引擎完成事实构建、验证和复现。  
  
让AI负责“想”，让引擎负责“证”。  
  
第三，业务逻辑才是规模化漏洞挖掘真正困难的地方  
  
SQL注入、XSS等传统漏洞类型，往往能够通过相对明确的模式进行识别。  
  
但真正复杂的业务漏洞，可能来自一个权限判断、一个租户隔离条件、一处所有权校验，或者一个状态机中的逻辑缺口。  
  
因此，漏洞工厂最终要解决的，并不是“让AI会找几个常见漏洞”，而是让它逐渐具备理解真实软件复杂逻辑的能力。  
  
这也是我们定义漏洞工厂的起点：  
  
面向明确授权的软件与数字资产，通过标准化资产接入、可组合检测、AI辅助推理、隔离验证、证据固化和知识回流，持续生产高置信漏洞证据。  
  
  
  
02实践是检验真理的唯一标准  
  
  
代码切片  
  
对于大型软件项目而言，代码规模本身就是AI漏洞挖掘面临的第一道门槛。  
  
以我们测试的某大型ERP项目为例，项目压缩包达到269.5MB，解压后源码规模达到数GB。面对这样的项目，即使模型本身具备较强的代码理解能力，也很难仅依靠原生上下文窗口一次性理解完整项目，更难建立跨文件、跨模块的代码关联。  
  
这意味着：  
  
模型“会审代码”，不等于模型“能审大型项目”。  
  
  
针对这一问题，漏洞工厂研发了自研切片分析引擎，不再要求模型一次性“吞下”整个代码库，而是将大型项目按照代码结构、调用关系和分析任务进行智能切片，再通过上下文关联与结果汇聚，让模型能够在有限上下文中持续获得完成漏洞分析所需的关键信息。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapHLIL6bUH4olH8wssQKMj2ky6qia9SAmgBruwWvby5bgVLia0dOklicNV3T9a0Tj8lfsJnLVZUPWfHKD2MngZxAib9pViaON7uU8oXU/640?wx_fmt=png&from=appmsg "")  
  
最终，我们成功完成了对该ERP项目的整体漏洞审计，识别出的代码执行、SQL注入、反序列化等高危漏洞，经过人工复核，相关漏洞均真实存在。  
  
这一项目成功证明：  
  
上下文的边界，可以通过工程能力突破。  
  
  
让模型看到代码，只是第一步；让模型看到与漏洞有关的整个上下文，才是真正的能力。  
  
  
框架的意义  
  
我们选取了  
国内两家具有代表性的大模型厂商的4款模型，以真实漏洞发现能力为核心评价指标，通过统一评测集、统一测试流程和统一评价标准，系统对比相同漏洞挖掘框架在靶场环境与真实环境中的检出表现。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapHZWDFShIwkMWKoDsILJlNW45gktTFsTFU9nKtqPsrV4uUp9p51meYqQIOlbyEGfibMB9VXDYnzwIKASGJR4W0M7bXiaQXAI9Xu4/640?wx_fmt=png&from=appmsg "")  
  
结果显示，在相同框架下，4款模型的漏洞检出率整体保持稳定，模型间波动极小。  
  
这一结果说明，模型本身是漏洞挖掘能力的基础，但稳定、可复用的漏洞发现能力，并不完全取决于模型能力本身。  
  
为了进一步验证漏洞工厂框架的作用，我们引入 Claude Code 作为参照，在完全相同的模型条件下，对比不同框架在漏洞挖掘任务中的实际表现。  
  
结果显示，框架差异会显著影响模型最终的漏洞发现效果。  
  
在包含144个漏洞的靶场测试中：  
- 漏洞工厂框架 + GLM-5.2：检出126个，检出率87.5%；  
  
- Claude Code + GLM-5.2：检出109个，检出率75.7%。  
  
当测试从标准化靶场进入真实项目，差距进一步扩大。  
  
在包含87个真实漏洞的测试中：  
- 漏洞工厂框架 + GLM-5.2：检出50个，检出率57.5%；  
  
- Claude Code + GLM-5.2：检出13个，检出率14.9%。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapHcPKmlqx522Vezn3Cm7db9Jx0sEicNiaO2k9GGr3uOQhctywKNzMqRrictDnqEkBVnaRzL7u61qryqXiamKnicDWAXnNGT8bnKBV4g/640?wx_fmt=png&from=appmsg "")  
  
这组数据对我们最大的意义，并不是证明“谁赢了”。  
  
而是验证了一个更重要的判断：  
  
大模型提供了漏洞挖掘的能力基础，但真正决定这些能力能否转化为漏洞发现能力的，是整个框架。  
  
  
  
理解业务逻辑  
  
为了验证漏洞工厂是否具备理解复杂业务逻辑的能力，我们让AI生成了一个完整的电商系统，再将其交给漏洞工厂进行自主分析。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapEbZsEVia8V0zF4dUiaeHsaPvY2vGs9j9utJ8uoxj4ZkmAUhKZ86pKXGXdXTmhdlO9lTQia7BntJkoqrF7Meib4lRjnEMt4cecg6AM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapFJFic1CuRXXKAFe5baLNCAP96ovbgbibkmqEUmwyF5RZh0YCmOQYicZZXznprVPG9lZQc0vibB7pJlTJjiabOWeX6CibUaZYNapvC14/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapFanhjE1s6TF5dTZLHHDgzFBsHY6XtU3BZoFuqsbwqPuyunuTRRkjpf1yZ08icRsSfaSMhfmkEEPF1n2ibibjRnftqibgq6Z5JEj34/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapHyFT91o2MHIlly57YZjoIN7Jb1Mia223kumnqiaDwniaSIUw2PWNgibSr4VrGpk1av8qMpSk2BfuO98HaNwGzpLQjI639up4Rkz7o/640?wx_fmt=png&from=appmsg "")  
  
<<  滑动查看下一张图片  >>  
  
在回放过程中可以清晰看到，漏洞工厂并不是简单扫描某一行代码，而是主动围绕支付金额、订单金额、支付流程等关键业务关系展开分析，沿着参数传递和跨模块调用进行追踪，最终发现：  
  
用户传入的支付金额未与订单实际金额进行校验，导致攻击者可以篡改支付金额，以远低于商品实际价格的金额完成购买。  
  
  
这类漏洞很难通过单一规则直接命中。  
  
因为从代码层面看，支付接口本身可能完全“正常”；  
真正的问题在于，用户输入的金额与系统真实业务状态之间缺少一个关键的不变量校验。  
  
而这恰恰是  
业务逻辑漏洞最难、也最有价值的地方：  
  
漏洞不一定藏在代码里，也可能藏在代码没有遵守的业务规则里。  
  
  
进一步来看，AI漏洞挖掘真正需要解决的，不只是“看懂代码”，而是：  
  
看懂代码背后的业务。  
  
  
  
03离开实验室，走到客户现场  
  
真实项目验证之后，下一个问题自然出现了：  
  
如果漏洞工厂要真正走到客户现场，它能不能面对比实验室更复杂的软件环境？  
  
  
作为国内首个AI原生代码审计应用，恒脑AI代码审计智能体一经上线便收到近百家客户单位申请试用。  
  
激动之余，担忧接踵而至。  
  
第一个难关就是准确率  
  
在某能源客户现场，客户以java-sec-code标准靶场为考核依据，恒脑AI代码审计智能体给出78份报告，命中74个标准答案。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapFWZVtjANicDlTUhh8YgUUacY2wd7XyvretAcpIocjVicVdB8SJ5zrniaIt461KSH8hpr6fib79ffNf9xo1tgpbSYptacWLjmJX0u0/640?wx_fmt=png&from=appmsg "")  
  
（VF为漏洞工厂英文翻译简称）  
  
另有额外检出为非标准漏洞但真实存在的安全问题：AntObjectInputStream反序列化黑名单绕过、FileUpload路径遍历上传、ReferFilter被禁用、LoginFailureHandler敏感信息泄露。  
  
74个标答+4个非标答，准确率达到惊人的  
100%。  
  
但对于漏洞挖掘来说，我们更关心的是：  
  
当标准答案消失之后，它还能不能找到真正的问题？  
  
  
在某金融客户真实业务系统中，共生成83份报告，其中76份被验证为正确，准确率达到91.6%。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapHiaR3rPI2oqK1aTxXS7KxSYQE2pswfowEGQkdO34zeosZOCLsj4wMd7LESORicaSdsHibpvtHP0zT0vzciaMdLfRvcbic4eefOUqf4/640?wx_fmt=png&from=appmsg "")  
  
从100%到91.6%，我们在客户现场验证了：  
  
当环境从标准化测试走向真实业务，漏洞工厂依然能够保持较高的准确性。  
  
  
第二个难关是语言与框架的复杂性  
  
客户现场的软件，往往不是单一语言、单一框架构成的标准化项目，而是由多种语言、框架、组件和技术栈长期演进形成的复杂系统。  
  
同一个业务系统中，可能同时存在 Java、Python、Go、C/C++、JavaScript 等多种语言，也可能同时使用 Spring Boot、Django、Gin、Kubernetes、LangChain、n8n 等不同类型的框架与平台。  
  
这意味着，客户现场的漏洞挖掘不能只在某一种语言或某一个框架中有效。它需要理解不同技术栈之间的调用关系，也需要适应不同项目结构、开发模式和业务逻辑。  
  
这一阶段，漏洞工厂的关键指标就是扩展漏洞挖掘能力的覆盖范围。  
  
目前，漏洞工厂完成了对  
16种后端核心语言的系统性覆盖：  
- 企业级主力  
：Java、PHP、Python、C/C++  
  
- 现代云原生栈  
：Go、Rust、TypeScript、JavaScript、C#、Ruby  
  
- 小众 / 专业领域  
：Lua、Erlang、Scala、Clojure、Elixir  
  
比语言更见功力的，是框架层的理解深度，当行业多数审计工具仍停留在 “传统 Web 框架”的舒适区时，我们的审计对象已深入PyTorch、LangChain 等AI应用本身。  
  
以AI治理AI，不是一句口号，而是一整年的实战。  
  
第三个难关是打破安全“常识”  
  
在软件安全领域，有一些长期存在的“常识”：  
  
知名厂商有专业的安全团队，核心产品经过多年安全建设，风险应该已经被充分发现；热门开源项目拥有庞大的社区和大量研究人员，甚至已经被反复审计过，再进行同样的安全检查，价值似乎已经有限。  
  
但过去一年漏洞工厂的实践，让我们开始重新审视这件事。  
  
我们把漏洞挖掘目标扩展到  
全球主流商业软件、基础设施级开源项目、AI大模型生态以及国产信创生态：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapErz7SJZsl7LFXRpF2PUN6wNmQCicukJmJqpiaWFcoNU6GHhCHcHlIs7g8Pdddfdwxa9bczzTWzB5c2jY99L8PZmVC7yiallY5shc/640?wx_fmt=png&from=appmsg "")  
  
这些项目的共同特点是：成熟、知名、拥有长期的安全投入，很多项目也经历过大量安全研究和审计。  
  
但在漏洞工厂持续的AI辅助挖掘中，我们依然不断发现新的安全问题。  
  
这意味着一个值得重新思考的问题：  
  
“已经被审计过”，并不意味着“不会再发现新的漏洞”。  
  
  
代码在变化，功能在变化，依赖在变化，业务逻辑在变化，新的攻击路径也可能随之产生。  
  
过去没有被发现，并不等于漏洞不存在。  
  
作为长期以安全性和稳定性著称的成熟开源操作系统，FreeBSD同样经历了数十年的持续开发与安全维护。漏洞工厂在对其开展研究时，发现了一个潜伏长达32年的安全漏洞。  
  
它说明，软件的成熟度和安全团队的专业程度固然重要，但它们并不能成为“无需再次审计”的理由。  
  
安全没有“审计完成”这一刻。  
  
AI不是在重复过去的安全检查，而是在尝试重新扫描那些“我们以为已经看过”的软件世界。  
  
04从“作茧”到“破茧”  
  
当AI开始具备规模化漏洞发现能力之后，一个新的现实也随之出现：  
  
漏洞正在变得越来越多。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapFdRMeGAfibdh2gQqOtMbAPGgXC8RjtqfvjJQOBKvtibKkOooicoGVj4568f9eVbgYaBoWdZwaLHwonr2VT43Qriabxgq7gmqknYHY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapGJiaBwbuqvafhibKLzpH2vXVl4DWC57X6dryWoZIDoiaCYxDlIlicBn8qd7LlbxWm81Ct0ehgjibkjLnq6YOfBicHUxysc4OctguY3o/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/QsTClNuIiapGU6LRX9CbTicL8QkqPBav0GuFfzVcxHWGDMAUBv0JcVbEa1FADujUlOZCIFWcZeLibaNDtaMFDUM8aHWebnyHypv5BicM4G7kGH8/640?wx_fmt=jpeg "")  
  
  
AI把漏洞挖掘能力释放出来，也让漏洞研究进入一个全新的数量级。  
  
如果仍然用过去完全依赖人工的方式逐个分析，能力越强，反而越容易被海量信息困住。  
  
所以，漏洞工厂开始了下一步进化。  
  
2026年7月，第二款产品级应用——零日雷达正式上线。  
  
它进一步利用漏洞工厂的关键能力，将零散的漏洞情报进行AI分析、环境验证，并转化为可以使用的防御规则。  
  
作茧，是AI导致漏洞数量的增长。  
  
破茧，是让AI开始具备理解这些漏洞的能力。  
  
真实漏洞事件，则让我们看到这套能力真正进入现实世界之后是什么样子。  
  
2026年7月18日，零日雷达自动监测到IBM Langflow相关漏洞情报。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapH7AevoTAgVlCkFej07m1PicpTJVWzTE3libFrgDySgrObJJyC3JHY7mZVqqzESBwmXvFwMD8GuM7bq9ctcWwTVS0ib7CWcsCaB3k/640?wx_fmt=png&from=appmsg "")  
  
随后完成AI漏洞分析与环境验证，并在T+1天内将规则配置到客户现场。  
  
20天后，也就是8月7日，攻击者开始利用该漏洞发起攻击。  
  
客户侧流量检测设备命中此前生成的规则，识别到相关攻击行为，客户随即完成IP封禁。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapGDWiblPq79GF16zoCicm8NR1cajPSvibuIOPl8RuiaDmbUrsAv9QGRJDvhfrD7HlcibUXgvko42ES5cKoa8cQwWhff1lZaVP3vcUqs/640?wx_fmt=png&from=appmsg "")  
  
8月14日，IBM官方发布修复方案。  
  
从漏洞情报出现，到分析、验证、规则落地，再到真实攻击发生，这一次我们第一次完整地看到了一条新的链路：  
  
漏洞出现 → AI分析 → 环境验证 → 规则生成 → 真实攻击 → 防御命中  
  
  
它让我们更加确信：  
  
真正的实战，不是实验室里发现了多少漏洞，而是当一个真实漏洞出现在真实世界时，我们能不能尽可能早地理解它。  
  
  
  
05一年之后，再看“漏洞工厂”  
  
2025年4月，我们第一次提出“漏洞工厂”这个想法。  
  
那时，我们想解决的只是一个问题：  
  
能不能让AI持续找漏洞？  
  
  
一年之后，这个问题已经有了越来越具体的答案。  
  
我们把它放进真实的开源软件里；  
  
把它放进不同语言、不同框架和不同技术栈里；  
  
把它与不同的AI代码智能体放在相同模型条件下进行比较；  
  
把它交给第三方验证；  
  
把它放进真实业务系统；  
  
再把它带到真实漏洞和真实攻击事件中。  
  
从“偶然发现一个漏洞”，到“持续生产高置信漏洞”。  
  
从一个研究员的单点能力，到一套可以不断运行、不断迭代的系统能力。  
  
这就是漏洞工厂上线一周年，我们真正想回答的问题。  
  
我们造的，不是一个更大的扫描器。  
  
而是一套让AI持续发现那些原本不该存在的漏洞的生产系统。  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6Ppczr1X4OOTmbg4rENrqwqbYqRtgl3icicic9an9TicNrOnKOT4t2icSyx7w/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=7 "")  
  
**点点赞**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6P0OFJt7aruYwYjIWic5WCu7iaE9ZYWmW6TKPcvrib6Itmpc0dnMqlANmow/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
**点分享**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PwKHSiaCHQrj4D3mJJZ7QGPX1zbt3rJEKjhdBkX12A8r5L47fI28upaA/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=9 "")  
  
**点喜欢**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/QsTClNuIiapFeCRD4zgEoc58Gq0ibgvWGD7m7Ck9aLXxoiaGTeFsSFqKVsibCKG5fBUndxJiaC5icibp244byDxxdRn3wiaffY6B1RQyA1lw5gxD9ibM/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=15 "")  
  
  
点击下方名片立即关注  
  
不走丢哦！  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PzUETDErlviazPRtpVtc98iasfL8RCCib8yzmeGJ9HrBlJASFn5qz95QwQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=9 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6Pb2aOfdTZnZZbozL1mvicIWsdWicdDcibz2SAuHblLHicWQc8CmX6WT6OMA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=10 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PBQVNtt5ynIG7UjUHbRJFvFXgKjZW6mzjjhlxfjwtXfrdJyPgroKWQA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=11 "")  
  
  
**往期**  
**精彩****回顾**  
  
  
  
  
  
[让安全检测不再拖慢网络 ，安恒明御防火墙新一代AI+FPGA架构发布](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652959&idx=1&sn=21ccce534007dedc933fcfe30b16bcb4&scene=21#wechat_redirect)  
  
  
2026-10-10  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652959&idx=1&sn=21ccce534007dedc933fcfe30b16bcb4&scene=21#wechat_redirect)  
  
  
[国家数据局率先批复试点｜安恒参编安徽省汽车行业数据资源开发利用任务书](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652934&idx=1&sn=bc31a7c52a73a6e43a3fd8bcfe3b113f&scene=21#wechat_redirect)  
  
  
2026-10-09  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652934&idx=1&sn=bc31a7c52a73a6e43a3fd8bcfe3b113f&scene=21#wechat_redirect)  
  
  
[【全球首个捕获AI黑客的AI原生蜜罐】——猎影·AI威胁狩猎诱捕系统正式发布！](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652925&idx=1&sn=b6eb5bb80a344d868ca3021029c67d16&scene=21#wechat_redirect)  
  
  
2026-10-08  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650652925&idx=1&sn=b6eb5bb80a344d868ca3021029c67d16&scene=21#wechat_redirect)  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3wm5VicXg2ibVyMsjPZ3OJSzTwdeSU207GIcBicQDzkDVgFNvXD0npWKhFtBb2VtiaczibVqM8HE0vRoNw/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=14 "")  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/cbge2dLXia4bbQyBibIkuZz7hZVFlib5Oib5VZCiacheOLBHzKc88VEQYkXdC7BfdzLSVml0HWO6RjrP7FHL8oL0ONw/640?from=appmsg&wxfrom=5&wx_lazy=1&wx_fmt=other&tp=webp#imgIndex=43 "")  
  
法律声明  
  
本文数据均来自内部统计、媒体报道、公开信息整理等，仅供信息分享，可能存在统计口径差异或误差，敬请理性看待，我们不对其准确性承担责任。  
股市有风险，投资需谨慎。阅读者在作出任何投资决定之前，应当咨询各自的顾问。本公众号发布内容仅代表内容创作视角，不构成任何投资建议或投资依据。在任何情况下，本公众号及运营主体不对任何人的投资结果承担法律责任。  
本公众号原创内容，欢迎合法合规复制、转载，转载时请务必注明出处，不得断章取义、以偏概全或进行有悖原意的引用。  
  
