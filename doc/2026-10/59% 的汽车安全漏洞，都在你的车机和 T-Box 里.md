#  59% 的汽车安全漏洞，都在你的车机和 T-Box 里  
 谈思实验室   2026-10-16 09:40  
  
点击上方蓝字  
谈思实验室  
  
获取更多汽车网络安全资讯  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247580896&idx=1&sn=36a65f68d014c79310fdeb0e1e55b6b0&scene=21#wechat_redirect)  
  
  
摘要：一项 ICSE 2026 的大规模实证研究分析了 649 个真实漏洞，发现其中 59% 集中在远程通信和信息娱乐系统。而 Pwn2Own 2026 上，一根 USB 线就让特斯拉的车机交出了 root 权限。  
  
先说一个数字。  
  
649。  
  
这是一篇发表于 ICSE 2026 的论文收集到的**真实可被利用的智能网联汽车漏洞**总数。不是理论推演，不是问卷调查，是 649 个有利用代码、有复现记录的漏洞。  
  
研究团队做了一件此前没人做过的事：他们把其中 592 个漏洞，来自 8 届"匿名杯"车联网漏洞挖掘赛（2023 年 1 月到 2024 年 4 月），覆盖 48 款不同车型；剩下 57 个来自研究者日常提交。基于这个数据集，他们第一次系统性地回答了那个一直没人说清的问题——  
  
**一辆智能汽车，到底哪里最容易被攻破？**  
  
论文标题很朴素：《Towards Understanding and Characterizing Vulnerabilities in Intelligent Connected Vehicles through Real-World Exploits》，发表在 ICSE '26 论文集第 1571–1583 页。  
  
但结论很不朴素。  
  
**01**  
  
**漏洞不是均匀分布的，它们挤在两个地方**  
  
  
研究给出的攻击面分布是这样的：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaDhRiaEXnicPicKBffkpx4xJGSvKXbQbu87vv6bBiaORKiaSibnW4tuPaW3H8iaIPbYKftdfxz2AAEcUrn1LfsmuoyX60D9H2RUzOibqwE/640?wx_fmt=png&from=appmsg "")  
  
前两项加起来，59%。  
  
换句话说，一辆车最脆弱的地方，既不是发动机控制单元，也不是刹车控制器——**而是你的车机，和那个你根本没注意过的 T-Box（车载远程信息处理单元）**  
。  
  
为什么是这两个？因为它们处在同一个位置上：**外部网络与车内总线之间的交界面**  
。  
  
这是工程上无法绕开的矛盾。T-Box 要连蜂窝网、要给云端上报数据、要支持远程解锁和 OTA；IVI 要连 Wi-Fi、连蓝牙、读 U 盘、跑第三方 App。**它们必须是"开放的"，否则车就不智能了**  
。但它们连的那一端，是 CAN 总线——一个 1980 年代设计出来、**报文本就没有身份认证**  
的协议。  
  
研究团队还补了一刀：现有文献里那些流传很广的攻击面分类法，覆盖面其实不够。他们在这个数据集上发现了**一个新的漏洞位置和 13 种新的漏洞类型**  
。  
  
**02**  
  
**一个比任何演示都更刺人的案例**  
  
  
论文引言里提到一个 2024 年 9 月的发现：研究者在一个韩系品牌的智能网联汽车上找到一个**授权漏洞**  
。  
  
结果是什么？**攻击者只用车牌号，就能远程控制这辆车。**  
  
不需要芯片、不需要拆车、不需要接触车辆。一个印在车尾、谁都能看到的车牌号，成了控制这辆车的钥匙。  
  
这背后的技术本质是：很多车企的云端后台，把"能查到车辆"和"能控制车辆"这两件事的权限边界划得太浅了。你只要能证明"我知道有这辆车"，系统就默认你有权操作它。  
  
这类漏洞在数据集中占比不低——研究把 649 个漏洞归成了**6 类威胁和 4 个风险等级**  
（从低到严重）。而那些最终能触达车辆控制功能的漏洞，风险等级往往落在"严重"那一头。  
  
**03**  
  
**Pwn2Own 2026：一根 USB 线，root 权限**  
  
  
理论说完，看实战。  
  
2026 年 1 月 21 日至 23 日，Pwn2Own Automotive 在东京举办，赛场就在 Automotive World 展会现场。这一届的规则有个关键设定值得注意：**所有目标系统都是已经完全打补丁的最新版本**  
。这考验的是漏洞挖掘的原创性，而不是"捡漏"。  
  
第一天，参赛队伍一共演示了 37 个零日漏洞，拿走 516,500 美元奖金。  
  
其中最受关注的是 Synacktiv 团队对特斯拉车机的攻破。路径非常清晰，也很有代表性：  
  
1. 入口：USB 接口（物理接触，模拟真实攻击场景——比如借车、维修、插充电线）  
  
1. 第一个漏洞：通过特制的 USB 设备触发信息泄露，拿到内存布局的关键信息  
  
1. 第二个漏洞：利用内存布局信息实现越界写入，覆盖关键内存区域  
  
1. 提权：把两个漏洞串联起来，成功获得系统 root 权限  
  
1. 结果：在 root 权限下执行任意代码，完全控制信息娱乐系统  
  
  
  
奖金 35,000 美元。同一个团队还用三个漏洞链，在索尼 XAV-9500ES 数字媒体接收器上实现了 root 级代码执行，再获 20,000 美元。  
  
其他队伍的战果同样值得看：  
  
- Fuzzware.io：攻破 Alpitronic HYC50 充电桩、Autel 充电器、Kenwood DNR1007XR 导航接收器，合计 118,000 美元  
  
- Team DDOS：攻破 ChargePoint Home Flex、Autel MaxiCharger、Grizzl-E Smart 40A 三款家用充电桩，72,500 美元  
  
- PetoWorks：串联三个零日漏洞，在 Phoenix Contact CHARX SEC-3150 充电控制器上拿到 root，50,000 美元  
  
  
  
把这组数字放进时间线看，趋势更清楚：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaBTUBJhqwdwJaInWPj1DaN07fjtC7o7YPNMf8X4t2cnVdIyYxJqLAyMKicScsibaETeECwoDyqhD9k4sLRIWhVwVUmoHNVj3njVU/640?wx_fmt=png&from=appmsg "")  
  
首日就有 37 个零日。 而且注意攻击目标里充电桩的密集程度——EV 充电基础设施正在快速成为和车机并列的高价值目标。  
  
按赛事规则，厂商有 90 天窗口期来开发并发布补丁，之后 Trend Micro 旗下的 Zero Day Initiative（ZDI）才会公开漏洞细节。也就是说，你现在读到这篇的时候，其中大部分细节仍然没有公开。  
  
**04**  
  
**真正的危险不在"攻破车机"，而在攻破之后**  
  
  
如果只是车机被控，那顶多算个隐私问题。但渗透测试行业里有一条被反复验证的路径，被称为头号渗透测试发现：  
  
IVI → CAN 提权。  
  
完整链条是：先通过蓝牙、Wi-Fi、蜂窝网络或 USB 媒体处理攻破车机，然后以车机为跳板，向车内 CAN 总线发送指令，最终触达安全关键 ECU。  
  
2015 年那台被远程劫持的 Jeep 确立了这条范式。十年过去了，厂商做了大量加固——但同类提权路径至今仍在不断被发现。  
  
为什么这么难堵？因为根因在协议层。CAN 总线设计时考虑的是速度和可靠性，不是安全。在总线上传输的指令不携带认证信息。只要能上总线，你就能让车执行"软件能力范围内的几乎任何动作"。  
  
而 T-Box 的情况更微妙。作为对外的常连通道，它的攻击面包括：2G/3G/4G/5G 基带漏洞、车内 Wi-Fi 热点、蓝牙钥匙、APP 云端接口（车辆控制 API、用户数据接口）、USB 物理接入后的权限提升。学术界已识别的 T-Box 攻击向量还包括：  
  
- 基带处理器漏洞——通过伪基站发送恶意短信触发  
  
- 基带协议栈内存损坏漏洞导致的远程代码执行  
  
- OTA 更新机制不安全导致的中间人攻击  
  
  
  
第三个尤其危险。因为 OTA 本来是修漏洞的通道，一旦它本身被劫持，就变成了规模化的投毒渠道。  
  
**05**  
  
**防守侧正在做什么**  
  
  
面对这条链路，行业的安全架构基本沿着五层铺开，这套分法对理解"钱花在哪"很有帮助：  
  
第一层，物理与硬件信任根。 集成在 SoC 或独立安全芯片内的硬件信任根（HRoT），包含不可更改的 Boot ROM 和唯一设备密钥。从 Boot ROM 开始，逐级验证引导加载程序、Hypervisor、操作系统内核和关键应用的电子签名——任何一级验证失败，系统中止启动或进入安全恢复模式。  
  
第二层，通信保护。 针对 CAN/CAN FD 这类没有原生安全机制的总线，靠上层协议弥补：SecOC（AUTOSAR 标准）为关键消息添加消息认证码（MAC）和新鲜度值，防伪造、防重放、防篡改。但要清醒地认识到一点——SecOC 保护的是认证和防重放，并不保护机密性。通信内容是明文的。如果报文涉及位置轨迹这类隐私数据，还得额外叠加链路加密。  
  
第三层，网关与域隔离。 在内网各安全域之间做监控隔离，按黑白名单策略只放行合法节点通信。一个典型的例子：如果车机突然试图发送"关闭刹车"指令，网关会判定其违反规则直接丢弃，刹车控制器根本看不到这条消息。  
  
第四层，入侵检测与防御（IDPS）。 这里有个工程上的现实取舍。基于规则的检测开销低、准确度高，但只能发现已知攻击模式；基于行为/异常的检测对未知攻击识别率更好，但车载嵌入式平台算力通常跑不动复杂模型，误报率也是真问题。所以实际项目里比较务实的策略是：规则为主、少量轻量异常检测为辅。检测对象包括报文频率是否符合预期、是否出现未知 ID、新鲜度值是否异常跳变；以太网侧则看异常的端口扫描和 ARP 响应。  
  
第五层，OTA 与供应链。 固件包的端到端加密、签名校验、防回滚、失败保护，这是底线。  
  
**06**  
  
**如果你负责这件事，可以从这里开始自查**  
  
  
那篇 ICSE 论文研究的落脚点很实际，它给出的自查方向可以浓缩成七条：  
  
1. T-Box 与 IVI 的固件与依赖更新——这两个是 59% 的所在，优先级最高  
  
1. OBD-II 访问控制——诊断接口直通 CAN 总线，任何插上去的设备都值得警惕，包括保险公司的 UBI 盒子  
  
1. CAN 总线入侵检测——至少要能发现未知 ID、异常频率、新鲜度跳变  
  
1. V2X 的 PKI 强度——证书体系的强度决定了能否被伪造  
  
1. 充电桩安全审计——Pwn2Own 已经用奖金说明了它的价值  
  
1. OTA 签名验证——防回滚和失败恢复是否真的可用  
  
1. 动态道路测试——静态测试覆盖不了真实路况下的攻击场景  
  
  
  
**07**  
  
**结语：攻击面还在扩大**  
  
  
这篇论文的贡献不只是那 649 个数字，更是把漏洞数据集公开了。这意味着后来的人可以在同一个基准上做对比，而不是各说各话。  
  
但坏消息也很明确：车联网的攻击面仍在扩大，新的漏洞形态还在涌现，而且攻击已经从云端一路延伸到物理控制层——危及的是真实的人身安全。  
  
59% 这个数字真正值得琢磨的地方在于：它指向的不是某个技术难点，而是一个结构性选择——我们为了让车"智能"，主动在车机和 T-Box 上开了朝向互联网的窗。窗户开得越大，防护就得越厚。  
  
参考资料：  
1. Wang Y., Ning Y., Sun Y., Xie X., et al. Towards Understanding and Characterizing Vulnerabilities in Intelligent Connected Vehicles through Real-World Exploits. ICSE '26: Proceedings of the 2026 IEEE/ACM 48th International Conference on Software Engineering, pp. 1571–1583. DOI: 10.1145/3744916.3773223；预印本：arXiv:2601.00627  
  
1. ZONE.CI 全球网《车联网渗透的六层攻击面：从 T-Box 到 CAN 总线，从 V2X 到充电桩》（含攻击面分布数据整理）  
  
1. Bleeping Computer / SC World / 安恒威胁情报中心《Pwn2Own Automotive 2026 首日：特斯拉等车载系统遭破解，37 个零日漏洞曝光》（2026 年 1 月）  
  
1. 21IC 电子网《电子电气架构相关安全体系》（内外网安全分层架构）  
  
  
  
**end**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/3g8Dklb9Twicgqayv6EVjeHah3Bpvw2ZJlH8rNickiaaHhLM4PaibcicFO9usS5xIOrWYjZibuvwV8g9DwnI6xZ4RvHg/640?wx_fmt=jpeg&from=appmsg "")  
  
**谈思汽车媒体门户**  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzkyODQzMDI3Mw==&mid=2247549590&idx=1&sn=b5ea25965c057d1ca2913d900f77799d&scene=21#wechat_redirect)  
  
  
  
**精品活动推荐**  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247572036&idx=3&sn=2410465a682d6b6c1f8b801eb583cdae&scene=21#wechat_redirect)  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247575659&idx=3&sn=1b3acb3a33e0fc992b67b37bc4d04a0e&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247580896&idx=1&sn=36a65f68d014c79310fdeb0e1e55b6b0&scene=21#wechat_redirect)  
  
  
  
  
  
  
  
  
**AutoSec系列沙龙**  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247548574&idx=1&sn=11f37456b4f45c0fdbf795c21e201c03&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247551934&idx=2&sn=50785b76c512a88b30455fc1e8fa188c&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247557132&idx=2&sn=2e44d4c2d77a2eec377d0553442d2c1b&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247561775&idx=1&sn=948a9e7f8d4fbed363c6a6a5479cd39e&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247561260&idx=2&sn=0ca6395502487515a921f32288b7e8df&scene=21#wechat_redirect)  
  
  
  
  
  
**专业社群**  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247535223&idx=1&sn=e30e07a44accd5b0e9ada3d8b537f977&scene=21#wechat_redirect)  
  
**部分入群专家来自：**  
  
  
**新势力车企：**  
  
特斯拉、理想、极氪、小米、零跑汽车、阿维塔汽车、智己汽车、小鹏、岚图汽车、蔚来汽车、吉祥汽车、赛力斯......  
  
**外资传统主流车企代表:**  
  
大众中国、大众酷翼、奥迪汽车、宝马、福特、戴姆勒-奔驰、通用、保时捷、沃尔沃、现代汽车、日产汽车、捷豹路虎、斯堪尼亚......  
  
**内资传统主流车企：**  
  
吉利汽车、上汽乘用车、长城汽车、上汽大众、长安汽车、北京汽车、东风汽车、广汽、比亚迪、一汽集团、一汽解放、东风商用、上汽商用......  
  
**全球领先一级供应商：**  
  
博世、大陆集团、联合汽车电子、安波福、采埃孚、科世达、舍弗勒、霍尼韦尔、大疆、日立、哈曼、华为、百度、联想、联发科、普瑞均胜、德赛西威、蜂巢转向、均联智行、武汉光庭、星纪魅族、中车集团、潍柴集团、地平线、紫光同芯、字节跳动、......  
  
**二级供应商(500+以上)：**  
  
中科数测  
、ETAS、  
BlackDuck  
、NXP、上海软件中心、Deloitte、奇安信、为辰信安、云驰未来、信长城、泽鹿安全、纽创信安、复旦微电子、天融信、奇虎360、中汽中心、中国汽研、上海汽检、  
加特兰微电子  
、浙江大学......  
  
**人员占比**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9Twicgqayv6EVjeHah3Bpvw2ZJVW2JR9ib5icMR4wIs58nO6ia3OicH5l6vONnmuhfLqMKqj8T2AnD7W1vqQ/640?wx_fmt=png&from=appmsg "")  
  
  
**公司类型占比**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9Twicgqayv6EVjeHah3Bpvw2ZJU6yKtYSJu4oPaJABYuCSyTpLXjRNbVv7OUTUUCxmB1OuPhtcM4j1kw/640?wx_fmt=png&from=appmsg "")  
  
  
**文章**  
  
# 不要错过哦，这可能是汽车网络安全产业最大的专属社区！  
  
[关于涉嫌仿冒AutoSec会议品牌的律师声明](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247531034&idx=2&sn=e466ca3e7c2927a91dd9a81be705afe1&chksm=e9273ec1de50b7d7f540ae2e4c255bfb42f842228a87f7dbc65297027a878544a9e796e09cf6&scene=21#wechat_redirect)  
  
  
[一文带你了解智能汽车车载网络通信安全架构](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247517280&idx=2&sn=8bfafb17871598c9cc0041bc9ee5f65d&chksm=e927c0bbde5049ad8cdb3647f6cdfce00c2db7a7b484941027bb7edf3128e4eaa74d6727dd46&scene=21#wechat_redirect)  
  
  
[网络安全：TARA方法、工具与案例](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247502093&idx=1&sn=ec4b373a33ca04d79afbb0b0b880bd4e&chksm=e9278dd6de5004c01bdd83ad0dd89c3549c7ae2ceb362959dbcb159324b2593d70bce78d82a9&scene=21#wechat_redirect)  
  
  
[汽车数据安全合规重点分析](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247519068&idx=1&sn=78c66e13bd8798afd46c766b8f18abe7&chksm=e927cf87de504691c816f78b55daf93bdfb72fc1cb870d926de8b471eb3e1be61058498327b1&scene=21#wechat_redirect)  
  
  
[浅析汽车芯片信息安全之安全启动](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247512151&idx=1&sn=7fabbeeec206ce615a5a3c574bed4c43&chksm=e927f48cde507d9ab6bfd4b8389b5eafea37586707682bfe60f294feb54e1c36cb07bad4d26d&scene=21#wechat_redirect)  
  
  
[域集中式架构的汽车车载通信安全方案探究](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247519952&idx=2&sn=709860de942501f20e923d15330ced9a&chksm=e927ca0bde50431df0b47ad1a2da63bf98ee637c9c00482145fbdb8755851b61421357aab4bf&scene=21#wechat_redirect)  
  
  
[系统安全架构之车辆网络安全架构](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247520446&idx=1&sn=27e10e455264cecb2a1b49d91484d036&chksm=e927d465de505d73c59a6fb4cb066c7c7d07a96ef49a841ffe598c23d28be545c5874dec7de4&scene=21#wechat_redirect)  
  
  
[车联网中的隐私保护问题](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247521010&idx=1&sn=94ef379e2b877551093a869cf9d4897e&chksm=e927d629de505f3f3cbc102682f7a21a82372108776d3484d8ce619f7db1aae0ab0a001b9b41&scene=21#wechat_redirect)  
  
  
[智能网联汽车网络安全技术研究](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247521302&idx=1&sn=01e9311cb2c84f3e64902abf5f6e7a9e&chksm=e927d0cdde5059db5fe18c5e27f830bbb6ea6df327088082e7844aa056b05f840ad4cf6e3b5a&scene=21#wechat_redirect)  
  
  
[AUTOSAR 信息安全框架和关键技术分析](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247521661&idx=1&sn=a72381e326e3a226059954c74698e0dd&chksm=e927d1a6de5058b0297b91ba77fcf34bd3c581476a0790c5e0cfbcbe026b5a7c27d700bfb1ca&scene=21#wechat_redirect)  
  
  
[AUTOSAR 信息安全机制有哪些？](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247522056&idx=1&sn=bbd03def212d085f533e0301f8c86f18&chksm=e927d3d3de505ac57099d5e42fb6726cf152de9aaa9590b095895874e7a4cc806abc84cc4ebf&scene=21#wechat_redirect)  
  
  
[信息安全的底层机制](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247522886&idx=1&sn=77103702d98e3788beae34b8ea3c31d0&chksm=e927de9dde50578b3dce0bba65599da38844310edd8554f43c9f1c354eaa0487b7c8b4f65c3c&scene=21#wechat_redirect)  
  
  
[汽车网络安全](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247523567&idx=1&sn=1b1d83f339de81a0dc396dd0bd6e6893&chksm=e927d834de50512246f63e47a32f7b934e64eb2b6138053ef43485b871736a122db1340bc437&scene=21#wechat_redirect)  
  
  
[Autosar硬件安全模块HSM的使用](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247527177&idx=1&sn=984bfc845ef51ec1f32cd12d37430621&chksm=e9272fd2de50a6c4013f84ed2257f634a505a04a27b4b27c30e5af4492d5fc3b0099216b1f7d&scene=21#wechat_redirect)  
  
  
[首发!小米雷军两会上就汽车数据安全问题建言：关于构建完善汽车数据安全管理体系的建议](http://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247519331&idx=1&sn=925d48164f1c7d2d109ee433cde6805b&chksm=e927c8b8de5041aea58f73aed311cdd3bf913bbb73d8e175ac80ae643d944709e06ec418fb52&scene=21#wechat_redirect)  
  
  
