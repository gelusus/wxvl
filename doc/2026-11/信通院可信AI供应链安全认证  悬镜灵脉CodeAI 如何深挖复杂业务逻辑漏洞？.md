#  信通院可信AI供应链安全认证 | 悬镜灵脉CodeAI 如何深挖复杂业务逻辑漏洞？  
领航AI原生安全的
                    领航AI原生安全的  悬镜安全   2026-11-13 07:00  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riajUgdH8tBbjIXJCuzorVZOWV8D2icZHSia1cRiaIBaUpwjricqcNffh2bBA2cAyJt257HN1jqnYHeDW4b5KdX9NwzIIO8GYgNyCHRI/640?wx_fmt=png&from=appmsg "")  
  
...  
  
近日，由中国通信标准化协会主办、中国信息通信研究院（以下简称“中国信通院”）承办的2026 OSCAR开源产业大会在北京正式举行。悬镜安全新一代AI原生安全治理体系核心产品源鉴SCA开源威胁管控平台、问境AgentAI智能体安全卫士平台、灵脉CodeAI代码安全智能体系统系数通过“可信AI供应链”评估认证，成为业内  
**唯一一家**  
在AI供应链安全、智能体安全与AI 代码安全三项关键技术上完成可信体系化认证的AI原生安全企业。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riagKtfXmCsApk8cYU9lDia4KXb68Cjkzl8GHLQ8GwLFwNhTTsCtnzHj3604pEoECiaH6QPCBhKbWxT4yhv6ns1sTkibYGf1l9EuwDo/640?wx_fmt=png&from=appmsg "")  
  
Fig.1悬镜安全 CBO 王越（Sunny，右一）现场领取证书  
  
其中，[灵脉CodeAI](https://sast.xmirror.cn/?lang=zh)  
聚焦代码安全智能体技术创新，融合静态代码分析、多智能体自主推理与深度漏洞挖掘能力，进一步探索传统规则检测难以覆盖的复杂业务安全风险。  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/cC2m1em2riaiaHzD7aCrJIpDfv3bIKLuG4KIC09adH0lncSGsGibiaF5z9SPsBpSyiaQVTjYFWsp9icGPZkp5UbbQnjft9hD5wnbibE3MTmSUmKkmA/640?wx_fmt=png&from=appmsg "")  
  
当每一段代码看起来都没有问题，隐藏在业务流程中的漏洞该如何发现？  
  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riagek2WzNlXiaakGibqz9kiaTKE4z9Vn08AFAQdcmYUiaRYLZia4lX1akkQQJFUVQI0ZqppbSabzy9ibvU7iaIawlXm61j8tV2IGE18ZBA/640?wx_fmt=png&from=appmsg "")  
  
  
**01 一笔付款审批**  
  
为什么有权限的人也可能越权？  
  
以一个典型的金融业务场景为例：  
  
某金融机构的甲、乙两家分支机构拥有独立的付款审批权限。按照业务规则，甲机构的审批人员不能操作乙机构的付款申请。   
  
但在一个存在权限设计缺陷的系统中，甲机构的审批人员尝试审批乙机构的付款申请时，可能顺利通过三项检查：   
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/cC2m1em2riaiakJr2lyiaAkXbRxBibgaU6QvjFOT46fl0wn2icbtviaU6XicoJFpYXd2DfuE6xDPZecG0iaMAyDa4TklR0VqqQ669lyTDKSU8elnyBk/640?wx_fmt=png&from=appmsg "")  
  
三项检查全部通过，却仍然不能证明这次审批合法。   
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/cC2m1em2riagc4zkkVtGsgSpHRibzR30epMvv2rgiaax43VDUu6nymJXGZT345HD3VycjMJOHPwhtJTtZnfVsjIaCykQtgqmFibpMpLCv8mYqYA/640?wx_fmt=png&from=appmsg "")  
  
Fig.2 业务流核心授权关系  
  
因为系统可能遗漏了最关键的一项判断：  
当前审批人员是否有权操作这笔具体的付款申请？  
  
问题在于，这些校验通常分散在不同代码层级中。身份认证由安全框架处理，角色权限由接口控制，付款状态由业务服务判断，而单据归属信息则保存在数据库中。  
  
单独检查每一段代码，可能都不存在明显错误。只有将身份、角色、资源归属与业务状态联系起来，才能发现完整业务链路中的安全缺陷。  
  
“拥有审批权限”不等于“有权审批这笔付款”。   
  
传统静态代码分析能够通过规则检测、控制流和数据流追踪，发现大量已知安全缺陷。但面对企业特有的业务约束，仅依靠通用规则往往难以完整还原业务授权关系。   
  
类似风险还可能出现在金融系统的账户操作、资金划转、支付审批与交易状态流转等关键环节。其隐蔽性不在于代码有多复杂，而在于多个看似正确的局部判断，组合起来却可能违背真正的业务规则。  
  
**02 灵脉CodeAI**  
  
如何从代码中找出缺失的业务规则  
  
业务逻辑漏洞的难点，往往不在于缺少某个安全校验，而在于分散在不同代码层级中的校验条件，未能形成完整的业务授权约束。  
  
灵脉 CodeAI 融合静态程序分析、代码语义理解与多智能体自主推理，围绕业务对象、调用链和授权关系展开深度分析，识别传统规则检测难以覆盖的逻辑缺陷。  
  
**第一步**  
   
  
**业务授权建模，还原代码背后的权限关系**  
  
一次付款审批是否合法，取决于用户身份、审批角色、单据归属与交易状态等多个条件，而这些信息通常分散在安全框架、业务服务和数据访问层中。  
  
灵脉CodeAI 通过代码解析、项目分层索引与上下文检索，关联Spring Security、JWT、RBAC等认证授权机制，提取业务接口、资源对象、权限规则与状态约束，构建业务授权关系。  
  
针对大型代码仓库，系统通过代码索引、任务拆分与按需检索组织上下文分析，而非一次性将全部代码输入大模型。  
  
以付款审批为例，核心授权关系不仅是“用户拥有审批角色”，还包括“用户所属机构与付款单据归属一致，且单据处于允许审批的状态”。这为后续识别缺失的业务约束提供了判断依据。  
  
**第二步**  
   
  
**跨文件调用链分析，定位授权约束的断点**  
  
业务规则明确后，灵脉 CodeAI 结合控制流、数据流和函数调用关系，跨 Controller、Service、DAO 等代码层级追踪业务操作的完整执行路径。  
  
以付款审批为例，一次审批请求通常需要经过多个代码层级，各层分别承担不同的业务处理逻辑：  
  
![](https://mmecoa.qpic.cn/mmecoa_jpg/cC2m1em2riajcSuy3cFL3nauFyTx5rd7r6vIKibYfxTnNZghmV9CApwnc0bnt6P0mV4ONfjrW1eIKqiaTXiaLAE70huxicfiaUrFXpAIXUXA0j9yg/640?wx_fmt=jpeg&from=appmsg "")  
  
虽然角色与状态检查均已通过，但如果最终的数据更新仅以单据 ID 为条件，且其他环节没有有效的机构归属限制，就可能产生跨机构越权风险。  
  
灵脉 CodeAI 的分析重点，是将身份凭证、角色校验、资源归属与最终数据操作关联起来，判断授权条件是否沿完整调用链实际生效。  
  
关键不在于代码中是否存在权限校验，而在于权限校验能否约束最终的业务操作。  
  
**第三步**  
  
**多智能体对抗验证，形成可修复的漏洞证据**  
  
识别疑似授权缺口后，灵脉 CodeAI 通过多智能体协同研判，进一步验证风险是否成立。  
  
攻击分析智能体从跨机构越权场景出发，构建风险假设并追踪可能的操作路径；防御验证智能体则回溯统一鉴权、公共权限组件、数据过滤条件及异常处理逻辑，寻找有效的防御约束。  
  
如果底层数据访问已经强制实施机构隔离，则需要排除或重新评估风险；如果完整调用链中均缺少有效的归属校验，则结合可达性、触发条件与代码证据，对潜在漏洞进行研判。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riahsiaAh4WQVcX8bvaicvf6ic8ZBxxibZdYVRwzCWEnrr3tNf0gkSBhI1pp6FNyibrdNMOHfpI6w5ib2Dichez0ebDE1JI5Fn8jniaRQjqk/640?wx_fmt=png&from=appmsg "")  
  
Fig.3 跨文件调用链分析  
  
最终，灵脉CodeAI输出关键代码位置、调用链、触发条件及修复建议，并结合 IDE、代码仓库与 CI/CD 集成，支持研发修复和回归验证。对于付款审批等核心金融业务，修复不仅需要补充资源归属校验，还应检查批量审批等复用同一业务服务的操作入口，避免相同缺陷通过其他路径再次出现。从业务规则还原、授权链路追踪到漏洞证据验证，灵脉 CodeAI 将代码安全分析深入到具体业务操作的授权边界，实现复杂业务逻辑漏洞的深度挖掘与精准研判。  
  
**03 金融客户实践**  
  
越权与逻辑漏洞检出率提升80%  
↑  
  
在金融行业，复杂业务权限关系带来的安全治理难题已成为代码安全建设的重要关注点。  
  
金融企业采用 DevOps 开发模式，业务发版周期较短，但长期面临漏洞修复周期长、跨部门沟通成本高等问题。随着业务系统复杂度提升，权限校验缺陷、业务逻辑漏洞等风险逐渐突出。  
  
为解决上述问题，企业引入悬镜安全灵脉 CodeAI，将代码安全检测集成到 CI/CD 流水线，并重点利用 AI 业务逻辑漏洞检测能力，对金融业务中的权限校验缺陷开展深入分析。  
  
在实际治理过程中，灵脉 CodeAI 通过代码调用链追踪、业务权限关联与漏洞证据输出，帮助安全团队识别复杂业务风险；同时结合漏洞报告、责任分配及 IDE 修复能力，推动安全问题快速进入研发修复流程，减少反复排查与跨部门沟通成本。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riaj0Hb02SDUJ1q3GI0iahTAEKrrA84IaRLoDEgMnJgfPU0f8uEtISF9lic38A2NsWGIvCE0onVVl50iagbadQFMENBO0bicwicp3fo0U/640?wx_fmt=png&from=appmsg "")  
  
Fig.4   
多智能体对抗验证与修复  
  
经过实际应用，企业取得了两项显著成效：  
  
越权与逻辑漏洞  
检出率提升80%，漏洞修复周期  
缩短50%。  
  
前者意味着更多隐藏在复杂业务关系中的安全风险能够被识别，后者则反映出风险发现、研判与修复之间的协作效率进一步提升。  
  
对于金融也起而言，代码安全检测的价值不应仅由扫描速度、规则数量或告警规模衡量。能否真正理解业务授权关系，识别隐藏在复杂调用链中的逻辑缺陷，并提供可验证、可修复的风险证据，才是深度漏洞挖掘能力的重要体现。  
  
一段代码没有语法错误，一次接口调用通过权限认证，一笔交易满足状态条件，都不意味着完整业务操作一定安全。  
  
代码审计的边界，不应止于代码是否正确，更应深入业务操作是否被正确授权。  
  
从传统规则检测到 AI 深度漏洞挖掘，悬镜安全灵脉 CodeAI 正通过代码语义理解、多智能体协同推理与业务逻辑分析，推动代码安全从已知缺陷识别，进一步向复杂业务风险发现与验证延伸。  
  
**结 语/Closing Remarks：**  
  
![](https://mmecoa.qpic.cn/mmecoa_png/cC2m1em2riajq78Yn8xcSCtsmZ0hWkAyStm5VjibiaH8xRswzKvdthvicf2Y4Srd46yzCma8jyBJxOymGeAmH6jIr0Co14aoO9t23G38usibzqjU/640?wx_fmt=png&from=appmsg "")  
  
**以AI深度推理，拓展代码安全治理边界。随着金融等核心业务系统的逻辑关系日益复杂，安全风险往往隐藏在身份权限、资源归属与业务流程之间的约束缺失中。代码安全技术亟需从已知漏洞检测，进一步走向对复杂业务逻辑的深度理解、自主推理与风险验证。**  
  
悬镜安全灵脉 CodeAI 融合静态程序分析、代码语义理解与多智能体自主推理技术，结合攻击面建模、漏洞假设生成、跨文件污点追踪与攻击路径推演等 AI 代码漏洞挖掘技术，构建面向复杂业务场景的深度漏洞挖掘能力，实现从代码缺陷识别到业务逻辑风险发现、证据验证与智能修复的持续治理，推动代码安全从规则驱动的被动检测向 AI 驱动的主动漏洞挖掘演进，持续拓展企业核心业务的安全治理边界。  
  
👇  
点击「申请试用」即刻体验产品   
»  
  
[](https://www.xmirror.cn/apply)  
  
  
**最新推荐阅读**  
  
Recommended Reads  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647796987&idx=1&sn=907acfb12d04ffa59bc5098c63288c87&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647796987&idx=1&sn=907acfb12d04ffa59bc5098c63288c87&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647796987&idx=1&sn=907acfb12d04ffa59bc5098c63288c87&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647801006&idx=1&sn=2a44b6b8433f40255b49df56cf9f524c&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647801099&idx=1&sn=1823faec2f4dad842d33c7a1bb30efb2&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NzE2ODk1Mg==&mid=2647801027&idx=1&sn=d203e9d949a8e20e315684e021667b1d&scene=21#wechat_redirect)  
[](https:///#%E5%B0%8F%E7%A8%8B%E5%BA%8F://%E4%BA%AC%E4%B8%9C%E8%B4%AD%E7%89%A9%E4%B8%A8%E7%82%B9%E5%A4%96%E5%8D%96%E9%A2%86%E5%9B%BD%E8%A1%A5/iqqtiHNY3s8mdGe)  
  
[](https:///#%E5%B0%8F%E7%A8%8B%E5%BA%8F://%E4%BA%AC%E4%B8%9C%E8%B4%AD%E7%89%A9%E4%B8%A8%E7%82%B9%E5%A4%96%E5%8D%96%E9%A2%86%E5%9B%BD%E8%A1%A5/iqqtiHNY3s8mdGe)  
  
[](https:///#%E5%B0%8F%E7%A8%8B%E5%BA%8F://%E4%BA%AC%E4%B8%9C%E8%B4%AD%E7%89%A9%E4%B8%A8%E7%82%B9%E5%A4%96%E5%8D%96%E9%A2%86%E5%9B%BD%E8%A1%A5/iqqtiHNY3s8mdGe)  
[]()  
  
**部分标杆客户**  
  
Selected Reference Clients  
  
  
  
![](https://mmecoa.qpic.cn/mmecoa_jpg/cC2m1em2riagHOeQXKWHqR0lSkz1KF535WBf6icCA329RaW5rj4MOdFkfoy09TlOiaYiaRRJ9EF0HjtiaG6icAytRTOsExMUy8ezg1TPyEvbFctTA/640?wx_fmt=jpeg&from=appmsg "")  
  
**关于“悬镜安全”**  
  
**Xmirror**  
  
  
  
  
悬镜安全，起源于北京大学网络安全技术研究团队“XMIRROR”。作为新一代数字供应链安全开拓者，首创基于“AI原生安全+软件供应链安全+AI供应链安全情报”的新一代数字供应链安全治理体系，以AI治理AI，从源头治理AI数据准备、模型训练调试、智能体开发部署到运行迭代等关键环节面临的AI原生安全风险，帮助企业用户构筑一套从“代码与组件”到“数据与智能”的智能体AI全生命周期内生安全治理体系，持续守护新一代数字供应链安全。  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/cC2m1em2riagXJuMtKmXSFwqs9UBRh6HVTsUFez3YeicPlfaHT8mcEesnQJ4PMsplIzL71icGEPnhVlIe2NibN7WrPhQAviaMzdK1uvQhj5ETYKI/640?wx_fmt=png&from=appmsg "")  
  
  
