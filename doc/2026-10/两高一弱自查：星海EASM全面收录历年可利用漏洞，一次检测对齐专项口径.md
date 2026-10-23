#  两高一弱自查：星海EASM全面收录历年可利用漏洞，一次检测对齐专项口径  
你信任的
                    你信任的  亚信安全   2026-10-23 09:28  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/YUnWCyLjbulwTyxzPs5KoUkRfEyqXTk8qHVo2z6MwXia3Ssiac9snRYlGexhZ9rpLqeLVYnfCa4jEQoB2LaicUfy1bxsqw8n0WMR6Ny3JkDDKU/640?wx_fmt=jpeg "")  
  
  
**把“清单对齐”变成机制，而不是一次临时专项**  
  
  
“两高一弱”清单不断变长，是因为漏洞在累积、端口随业务漂移、弱口令随人员流动反复出现。试图维护一份“最终版清单”来应对，注定滞后。更现实的做法是建立滚动式的差集比对：  
  
- **资产侧：**  
 外部测绘的周期性产出实测资产集合，与台账做差集，差集条目强制归属确认或下线收口。  
  
- **端口侧：**  
以“公网实际可达”为准，而非“台账上登记开放”为准，非标端口必须通过服务指纹识别后才能进入弱口令检测范围。  
  
- **口令侧：**  
把默认口令核查做成配置基线的一部分，覆盖“出厂未改、重置回退、规律口令”三种口径，而不是依赖一次性人工改密。  
  
- **归属侧：**  
每一项资产（包括云资源和第三方组件）明确业务归属、部署位置和责任人，形成发现、修复、复验的闭环，杜绝“发现了但没人认领”的风险滞留。  
  
自查报告的价值，不在于证明“我查过了”，而在于证明“我查的范围和实际暴露的范围是对齐的”。如果报告里只有台账内资产的扫描结果，而没有影子资产差集，就是一份边界不完整的自查。  
  
  
  
这正是星海外部攻击面管理平台（EASM）的切入点：随着2026年两高一弱专项口径的更新，星海EASM已将历年两高一弱相关可利用漏洞全面收录进平台，企业无需再人工核对逐年更新的通报清单，一次检测即可对齐专项口径。  
  
  
  
  
**两高一弱自查  难在清单对不齐**  
  
  
  
  
两高一弱之所以年年查、年年查不透，是因为它对应的是一个持续扩大的检查范围，而不是三道固定题目。  
  
  
但在实际排查中，企业经常面临三个问题：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qEwCoiclic3egFJo5WhAAZsts6fL8XQIATCoc8poySoEH1Bdq7bnKjygnDfM5soEAibt9NF9Rhsx2KibnKcpuMAysGmXx3sGYWO2Qpvjh3eFFJk/640?wx_fmt=png "")  
  
**痛点一：历年漏洞持续累积，人工对照难。**  
  
历年两高一弱相关的通报与漏洞数量庞大且持续新增，安全团队需要逐一核对检查表、通报列表与自身资产，人工对照既耗时也容易遗漏。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qEwCoiclic3egFJo5WhAAZsts6fL8XQIATCoc8poySoEH1Bdq7bnKjygnDfM5soEAibt9NF9Rhsx2KibnKcpuMAysGmXx3sGYWO2Qpvjh3eFFJk/640?wx_fmt=png "")  
  
**痛点二：扫描依赖已知目标，覆盖不到未知资产。**  
  
常规漏洞扫描能够对纳入范围的目标开展检测，但如果某个互联网资产根本没有进入扫描清单，它就很难被纳入后续检查。扫描任务完成，不代表互联网暴露面已经全部覆盖。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qEwCoiclic3egFJo5WhAAZsts6fL8XQIATCoc8poySoEH1Bdq7bnKjygnDfM5soEAibt9NF9Rhsx2KibnKcpuMAysGmXx3sGYWO2Qpvjh3eFFJk/640?wx_fmt=png "")  
  
**痛点三：风险发现与整改之间缺少衔接。**  
  
扫描报告可能列出了大量问题，但安全团队仍需进一步确认风险对应的资产、责任部门、整改进度及复测结果。如果缺少统一的管理机制，容易出现问题已发现、整改跟踪却未同步推进的情况。  
  
  
归根结底，企业需要的不只是一次扫描任务，而是一套从资产发现、风险排查到整改复核的工作机制。  
  
  
  
  
**全面收录历年两高一弱可利用漏洞**  
  
**让专项口径直接落地**  
  
  
  
  
星海EASM已全面收录历年两高一弱可利用漏洞，并将其转化为对互联网资产的检测能力：高危漏洞依托漏洞库全面覆盖检测，高危端口按暴露口径识别，弱口令按常见服务入口排查。专项要求不再依赖人工对照清单，直接落地为可执行的检测任务。  
  
  
围绕两高一弱相关场景，可以将排查工作理解为三个层次。  
  
  
  
  
**高危端口：先看清哪些入口暴露在外**  
  
  
  
高危端口排查的关键，不只是识别哪些端口需要重点关注，更要确认这些端口是否仍然对外开放，以及对应的服务是否有必要暴露在互联网侧。  
  
  
星海EASM通过互联网资产发现、端口探测与服务识别，帮助安全人员掌握目标资产的对外开放情况，识别需要重点关注的远程管理入口、数据库服务及其他高风险暴露点。  
  
  
结合企业现有资产台账，可以进一步核对新增资产、未知入口和不合理暴露，辅助安全团队明确排查重点。  
  
  
对于发现的高风险端口，企业可结合业务需求和安全策略，采取关闭不必要端口、限制访问来源、调整网络策略等措施，并通过后续复测确认暴露状态是否发生变化。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/YUnWCyLjbum6Ap4XZUXW83quKxFpkgcBUSicj6jHkGicYxK15RvmboNK9NQzVvlXPLNia895ahpIEO05yKgMoSGIJfqMns53ibv6ydCtgtEXVOc/640?wx_fmt=png "")  
  
互联网资产的端口与服务暴露识别  
  
  
**核心价值：从“知道哪些端口有风险”，进一步走向“知道哪些入口不该暴露、哪些问题需要优先整改”。**  
  
  
  
  
**高危漏洞：不仅关注漏洞告警 更关注风险是否成立**  
  
  
  
高危漏洞排查的难点，在于如何从检测结果中识别真正需要优先处置的问题。  
  
  
对于互联网暴露资产，仅依赖已有台账开展扫描，可能无法覆盖全部外部入口；而仅依据组件版本或漏洞特征产生的告警，也需要结合实际环境进一步研判。  
  
  
星海EASM以资产发现和服务识别为基础，依托对历年两高一弱可利用漏洞的全面收录，帮助安全人员识别互联网资产中的潜在高危漏洞，关联相应资产与风险信息，为风险分析和整改优先级判断提供依据。  
  
  
针对需要进一步确认的漏洞，应结合实际检测条件、授权范围及相应验证能力进行核实，避免将未经确认的风险线索直接等同于已验证的可利用漏洞。  
  
  
通过将漏洞风险与具体互联网资产关联，安全团队能够更清晰地回答：风险出现在哪个系统、对应哪个对外入口、需要由谁负责处置。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/YUnWCyLjbumibnslENiak0rxIciaZQoUT4nDrQnr0kMkibzRhHEHZe8rSeZpBwhmic37OCOIokIpMyBLkWa9n4xKdLLVkz5liaiaQ3ial4kKbQibiaD2k/640?wx_fmt=png "")  
  
历年两高一弱漏洞检测与资产关联  
  
  
**核心价值：让漏洞排查从分散的告警列表，转向与互联网资产关联的风险识别和处置。**  
  
****  
  
  
  
**弱口令：不要只检查已经纳入台账的系统**  
  
  
  
弱口令风险往往具有隐蔽性。企业可能已经对部分业务系统开展口令检查，却忽略了其他独立部署、长期运行或对外暴露的服务入口。  
  
  
例如，远程管理服务、运维入口及部分业务管理界面，都需要结合实际暴露情况和认证机制进行排查。  
  
  
星海EASM围绕已发现且处于授权检测范围内的互联网资产，结合当前版本支持的检测对象与检测方式，辅助开展弱口令风险排查，帮助安全团队识别存在风险的相关入口，并将发现的问题关联到对应资产。  
  
  
对于检测发现的风险，应进一步结合资产用途、访问范围和业务影响进行评估，及时推动口令加固、认证策略优化、访问权限收敛等整改措施。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/YUnWCyLjbum1nfjteW24qfibP7tov651iaLeOFvibDl2TlHWWDLgo9ty0EdkSt5xfTicfGn7n7mbROh806I0B68icVl9geLjXSa6AJoaopFeWZkc/640?wx_fmt=png "")  
  
弱口令风险排查  
  
  
**核心价值：将弱口令排查与互联网资产发现结合起来，减少因资产遗漏造成的检查盲区。**  
  
  
  
  
**从发现资产到整改复测**  
  
**形成完整排查链路**  
  
  
  
  
如果说高危端口、高危漏洞和弱口令是检查重点，那么资产发现、风险关联和整改复核，就是让这些检查工作真正发挥作用的基础。  
  
  
围绕两高一弱相关排查需求，星海EASM可以通过以下工作链路提供支撑：  
  
  
  
**第一步：**  
发现资产。结合企业名称、域名、IP、备案信息、证书及其他线索，开展互联网关联资产发现，辅助梳理外部暴露面。  
  
  
**第二步：**  
识别暴露。对纳入范围的资产开展端口探测与服务识别，掌握互联网开放情况，发现需要重点关注的暴露入口。  
  
  
**第三步：**  
开展风险检测。结合实际支持的检测能力，对相关资产开展高危漏洞、弱口令等风险排查，形成与资产关联的检测结果。  
  
  
**第四步：**  
推动整改。结合风险等级、资产重要性和业务影响，明确处置优先级，推动责任部门落实整改。  
  
  
**第五步：**  
复测核验。对已整改问题开展复测，核对风险状态及暴露情况变化，为整改结果确认提供依据。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/YUnWCyLjbulFPTLflhbgpZcCcNBGqm3ribHNFGq1uaVwFue1r20HXkw59nb0e1Q17ngicuSWCxO5rJzyCXMQKgGSdWwrs7W9GTIP9CHmBEicw8/640?wx_fmt=png "")  
  
从资产发现到复测核验的排查链路  
  
  
通过这条链路，企业可以将原本分散的资产盘点、漏洞扫描和人工跟踪工作串联起来，减少重复核查，让风险发现、处置与复核之间形成更清晰的衔接。  
  
  
排查之外，结果汇总与整改管理同样是安全负责人的压力来源：梳理出多少资产、哪些属于高优先级、哪些已整改、哪些待复核。星海EASM通过资产管理、风险中心、任务管理及报表等能力，与具体资产和处置过程一一对应；企业可结合自身检查表，整理资产清单、风险明细、整改状态及复测结果，提升自查材料准备与内部协同效率。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/YUnWCyLjbuk84HdnHIjwyFN36JaFjxkc2B4MwQicJGbtuqRYtV64bt9icAwbdia9Y8HQiawpfFyFwiclBZCacImibUlCGmeXDqZRWsY94Z3QqdbUA/640?wx_fmt=png "")  
  
风险明细与整改状态汇总  
  
  
需要强调的是，技术检测结果是风险排查和整改管理的重要依据，但不能直接等同于监管检查结论。具体报送格式、检查口径及材料要求，仍应以相关主管部门的正式文件和实际要求为准。  
  
  
  
  
**专项排查结束  但漏洞还在累积**  
  
  
  
  
两高一弱相关排查有明确的工作周期，但企业的互联网暴露面并不会因为一次检查结束而停止变化。  
  
  
新系统上线、业务版本迭代、云资源调整、第三方服务接入，都可能带来新的域名、端口和服务入口。如果资产台账没有及时更新，风险排查也没有持续开展，原本已经整改的问题可能再次出现，新的风险也可能长期处于未知状态。  
  
  
因此，企业需要将专项排查与日常安全运营结合起来。  
  
  
依托星海EASM的外部资产发现和风险管理能力，安全团队可以围绕互联网资产变化开展持续梳理，结合业务需要制定周期性排查计划，并对重点风险进行跟踪与复测。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/YUnWCyLjbuk2kJavNBSliaOGpqJzDu6qnf2Hb6PHfQiaX7XPcNVC6LBSjicVcxtSdqo4gkDcggCowriaJHHXHpwuHvhxichw6LHkj6vnlZ71L2pY/640?wx_fmt=png "")  
  
互联网暴露面持续监测  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
**真正重要的**  
  
**是你是否看清了自己的互联网暴露面**  
  
  
  
  
两高一弱排查，表面上是在检查高危漏洞、高危端口和弱口令，背后考验的却是企业对互联网资产及其安全风险的掌握程度。  
  
  
如果资产清单不完整，扫描就可能存在盲区；如果风险与资产缺少关联，整改就容易失去抓手；如果缺少复测，问题是否真正解决也难以确认。  
  
  
星海EASM全面收录历年两高一弱相关可利用漏洞，以外部攻击面为切入点，将互联网资产发现、端口与服务识别、风险检测及整改管理串联起来，帮助企业更全面地掌握外部暴露情况，为专项安全排查和常态化风险治理提供支撑。  
  
  
**先把资产找全，再把风险查清，最后让整改形成闭环。**  
  
  
这不仅是做好专项排查的工作思路，也是企业持续提升互联网安全防护能力的重要基础。  
  
  
如果您正在准备这一轮两高一弱自查，不妨从一次互联网暴露面盘点开始：先把对外的域名、IP、服务与端口重新摸一遍，再决定从哪里下手。报送节点临近，暴露面盘点越早启动，留给整改和复测的时间越充分。如需了解星海EASM在两高一弱场景下的检测能力，欢迎联系亚信安全服务团队，预约产品演示并获取《两高一弱自查要点清单》。  
  
  
**往期推荐**  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633497&idx=1&sn=f3949a64ebbec0e5b28a60abf43c29ed&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633452&idx=1&sn=6750e2d034f2a571d34433bc8d7bae3f&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633415&idx=1&sn=a594dab138e6f21d36153f1295d3c752&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633330&idx=1&sn=a8dfe939b299b22b8dcbcc4b8c651864&scene=21#wechat_redirect)  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjY2MTIzMw==&mid=2650633340&idx=1&sn=34b40a3c44c85b193c75c566d24bd2f8&scene=21#wechat_redirect)  
  
  
了解亚信安全，请点击  
**“阅读原文”**  
  
  
**求点赞**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoG8KmicxOYyR9Em8f5BFRia2jia66l1HibEyCKXqUq6bGLUCj7uDtZS58pg/640?wx_fmt=gif "")  
  
  
**求分享**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoHFSQRwicdBWfDiaNibTtUyQ2lPiaicDV5pZaUgZTzY3TJQ3ZbmR1Tj5iciaYg/640?wx_fmt=gif "")  
  
  
**求喜欢**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0FZMB3vsw4wxyNMbXwC42FoAV9dDSmbcAxOs8iaCgcpDEBjGNObDFpgTXjZSyjr9DzTgMblPrUYILA/640?wx_fmt=gif "")  
  
  
  
