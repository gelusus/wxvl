#  塔塔车联网App四类漏洞披露：从手机端凭据到车辆服务，风险边界在哪里  
 谈思实验室   2026-11-07 09:40  
  
点击上方蓝字  
谈思实验室  
  
获取更多汽车网络安全资讯  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247580896&idx=1&sn=36a65f68d014c79310fdeb0e1e55b6b0&scene=21#wechat_redirect)  
  
  
  
**01**  
  
**一份11月公告，指向已经推出的更新**  
  
  
11月4日，印度计算机应急响应小组CERT-In发布CIVN-2026-0537，披露塔塔汽车iRA 2.0与iRA.ev的四类漏洞。前者服务于燃油车客户，后者面向电动车客户，两款App承载车辆状态、健康监测、行程信息以及部分远程服务。问题集中在手机应用的凭据、网页组件和网络配置，并非公告认定车内控制器遭到入侵。  
  
  
公告列出的受影响范围为iRA 2.0的26.7.3及以下版本，以及iRA.ev的26.7.2及以下版本；建议分别升级到26.8.2与26.9.1。这些是此次公告给出的修复版本。车主实际更新时，仍应从官方商店取得当前可用版本，并确认没有停留在受影响版本。  
  
  
11月5日，Business Today报道了塔塔的回应，11月6日Autocar India又刊出相关说明。两家媒体均转述塔塔称，漏洞已通过软件更新和安全措施处理，其评估未发现客户凭据、个人信息或车辆相关数据遭到未授权访问的证据。公告描述的潜在风险与企业披露的实际发现由此形成两个层次：目前可确认漏洞已被公开披露，但尚无独立取证材料证明它们已造成客户数据泄露。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaAKjF0nSkA3X0WJzlUE3VPWbqoxcIBEAc3dic8gD5Cic5oXicwicEX4P4hY5EGibiau59ia54ZZHmUbm73nW6iaibVZAibL6QqPlU6M4d8Pg/640?wx_fmt=png&from=appmsg "")  
  
图 T1｜iRA 2.0官方商店中的车辆状态展示图，界面列出锁车、鸣笛等功能；并非漏洞利用现场，不能据此断言这些功能均受影响。  
  
**02**  
  
**四条路径，落在不同的信任边界**  
  
  
第一类是硬编码凭据，编号CVE-2026-121144。CERT-In提交的CVE记录指出，应用包内存在服务账号凭据、API密钥和授权令牌，可能被提取后用于未授权访问关联后台。这类问题的根源是把应由服务端保管的秘密交给了可分发的客户端。应用加固只能增加提取成本，不能让每个用户都能获得的安装包成为可靠的保密边界。  
  
  
第二类CVE-2026-121145涉及WebView。应用内嵌网页时，对URL及来源校验不足，恶意网页内容可能与应用功能发生未授权交互，进而取得仍然有效的持有者令牌。此类令牌通常以持有即代表授权的方式使用，泄露后的问题不只在页面显示层，而是可能延伸到以用户身份访问服务。来源限制、网页与原生功能的接口权限，因而应一并检查。  
  
  
对同类系统而言，登录成功之后的每一次车辆查询、每一条远程指令，都还需要服务端重新确认用户、车辆绑定关系与操作权限。令牌只能证明某种身份或授权上下文，不能让后台省略对象级访问检查。这是沿着公告风险作出的工程分析；现有公开材料没有披露塔塔具体哪些接口受影响，也没有证明所有远程功能均可被调用。  
  
  
第三类CVE-2026-121146是日志泄露：开发调试日志记录了认证令牌，能够接触这些日志的本地攻击者可能借此冒用用户。这与无需接触用户手机即可从公开安装包提取公共秘密的第一类漏洞，前提并不相同。第四类CVE-2026-121147则是证书固定配置失效，攻击者还需处于能够截获网络流量的位置，才可能拦截或修改App与后台之间的通信。  
  
  
这里还要纠正一个容易被新闻标题放大的细节：CERT-In给整份公告的评级是高危，不等于四个CVE各自全部为高危。官方CVE记录的CVSS 4.0基础分依次为8.8、7.1、6.8和8.5，日志令牌泄露一项为中危。分值反映各自的利用条件与影响模型，不能替代对实际账号权限、服务端限制和运行环境的调查。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaBW2yJlhUdKkiaria4aVvV1B2icFtC12Niaov5IFGq8SvDzNPBxicLKd6zCVGD3w4lbDrSvxmZibZLZ9FibRMgGlT6wLibDyrFibXKYvr6I/640?wx_fmt=png&from=appmsg "")  
  
  
图 T2｜iRA 2.0官方健康诊断展示图。应用显示车辆部件状态，不代表这些部件的控制器已被攻破。  
  
**03**  
  
**修复发布与风险消除，需要分别核对**  
  
  
据上述媒体转述，塔塔称iRA 2.0与iRA.ev的强制更新分别于2026年8月、9月推出，用户打开App时会收到更新提示；所有活跃用户已经完成更新，公司正联系其余尚未更新的客户。这意味着公告日期晚于企业所述补丁推出时间。公开披露时间不等于漏洞刚刚被发现，更不能直接据此计算所谓三个月无人处理的暴露期。  
  
  
官方Google Play页面提供了一个可交叉核对的时间线索：检索时iRA 2.0显示8月27日更新，iRA.ev显示9月11日更新。不过，商店更新时间只能证明页面显示的发布记录，不能独立证明某项漏洞在哪一天完成修复，也不能验证全部用户的安装覆盖率。后两项仍应准确归于企业说法，避免把不同层次的证据拼成确定结论。  
  
  
从修复机制看，更换手机App只是其中一环。如果旧版曾带有可复用的后台密钥，工程团队还需确认旧秘密是否失效、权限是否收紧；如果会话令牌可能流出，则要检查撤销与过期策略。上述内容是针对这类漏洞的工程核查建议，公开材料尚未披露塔塔具体的密钥轮换、历史令牌吊销范围和独立复测结果，不宜替企业补写已经完成的措施。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaB2jgd0rXZuwHXC62ufR8IL21BDhDDqYCdmXw8L7DS2Bl9Mcj93mSE6HMX3A1bP0DAq7WfSHic0uRp6lpgNzg4676yFK0oRhEkA/640?wx_fmt=png&from=appmsg "")  
  
图 T3｜iRA.ev官方车辆状态展示图，显示Curvv.ev的电量、续航及服务按钮；图中数值为开发者宣传演示，并非本事件真实用户数据。  
  
**04**  
  
**连接车辆的入口，不能只按普通App验收**  
  
  
此次公告确认的影响上限表述，是可能取得敏感凭据、冒用用户，并访问车辆相关信息和服务。它没有展示绕过车辆端安全条件、接管制动或转向的证据。即使App具备远程功能，某个令牌实际能调用哪些接口、车辆端是否另设状态约束，仍需逐项验证。把账号冒用直接等同于任意远程控车，会掩盖真正需要调查的授权链路。  
  
  
公开材料同样没有给出受影响账号总数或实际受害者名单，也未展示完整利用链。没有这些证据，就无法把潜在风险转换成已经发生的损失规模。后续值得追问的是车企如何证明更新真正阻断了原有路径，以及客户端修复后服务端是否仍接受旧凭据；这些问题比笼统宣称系统已经绝对安全，更能检验修复是否形成闭环。  
  
  
对车企而言，这起披露的启示在于验收范围。移动端安全审计应沿着安装包、网页组件、日志、网络通信一直追到后台授权：移除一个泄密点之后，已有凭据是否还有效，同一个账号是否仍能越过车辆绑定关系，都需要复核。对车主而言，现阶段最明确、可执行的动作是完成官方更新。修复覆盖率、旧凭据失效情况及独立复测结果，则是判断这次风险是否充分收敛仍需追踪的证据。  
  
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
  
  
