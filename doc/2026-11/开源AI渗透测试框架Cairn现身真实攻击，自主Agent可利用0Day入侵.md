#  开源AI渗透测试框架Cairn现身真实攻击，自主Agent可利用0Day入侵  
e安在线
                    e安在线  e安在线   2026-11-25 08:03  
  
某中国背景威胁组织运行的自主AI Agent耗时近一整天尝试入侵高校服务器未果，随后在一款开源医学影像AI应用中找到并利用一个0Day漏洞，成功拿下据点。  
  
  
亚马逊威胁情报团队披露，这些Agent运行在开源AI渗透测试框架Cairn上，几乎不需要人工引导。操作者仅向其提供目标IP地址列表，设定的目标为：“获取shell访问权限。”  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9nEI1ZPicACxrmxPrn64FFKLD3vYZ5W1ib1Qc6eNYFibYRSDGSGbV5OFCEyP3o9MkAMxnqqXhGad0q529dCYsyPo5iaJKoKAibrxcm4/640?wx_fmt=png&from=appmsg "")  
  
**01**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9kbJFZZBVFfk82fRsUonnbzGpn7cZUDzaHOkoa2AcGYWoKgceibUWHEssKA98mSianNaL0QF5iaibmtbXs9DbL5tYr7fXrphhgnffs/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9nA4yGmjRO5Kdoqq7NVacUbwoOgiaXMGT8WTSlfzSSDSibHibTF6OcAeZFILSTwKP5pozbGDOwRyvyW1vaZLSATA35y7ibECUiaZJJQ/640?wx_fmt=png&from=appmsg "")  
  
  
**AI Agent自主开展多轮探测**  
  
**利用0Day攻入高校网络**  
  
此次攻击活动始于2026年9月8日，目标为中国大陆以外东亚地区高校的IP地址。  
  
  
亚马逊方面表示，针对两所高校的攻击过程显示，这些Agent在近一整天的时间里自主调整攻击手段，全程无需人工指示下一步尝试方向。  
  
  
在针对其中一所高校的攻击中，Agent先是花了约16小时攻击一台允许未授权访问的Redis服务器。它们尝试利用主从复制功能加载未授权代码、绕过脚本沙箱、写入配置文件、破坏内存，还对同一台机器上的另一服务发起Log4Shell注入攻击。  
  
  
上述尝试全部失败后，Agent利用同一台服务器上部署的Qdrant实例存在的服务端请求伪造（SSRF）漏洞，测绘内部Docker网络，随后对PostgreSQL、SSH及其他服务进行凭据喷洒攻击，再次未能得手。  
  
  
随后，Agent从教职工名录、实验室网站、存档页面及研究论文中收集到约80个真实用户名。接下来的6个多小时里，它们针对SSH、管理后台、网络存储、VPN服务器及校园门户尝试了超过3万组凭据组合，依然没有成功。  
  
  
最终，Agent通过一款开源医学影像AI应用找到突破口：它们利用该应用上传接口文件名参数中一个此前未知的路径遍历漏洞，仅用约11分钟就完成了从测绘应用API到在服务器上写入任意文件的全流程。  
  
  
亚马逊综合安全部门CISO CJ Moses补充：“在后续会话中，Agent利用同一漏洞向_sitecustomize.py文件写入Python反向shell载荷——Python解释器启动时会自动导入该文件。随后Agent触发训练API接口生成新的Python进程，新进程导入载荷后建立了root权限的反向shell。”  
  
  
此后人工操作者介入了其中一次会话，为Agent提供了通往第二所高校网络的代理隧道——该网络同样运行着存在漏洞的软件——并指挥Agent攻击网络内的其他机器。亚马逊威胁分析师认为，攻击者正是利用同一个0Day漏洞攻陷了这台服务器，搭建了上述隧道。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9niakFr3U6iafaHkmGQWD62Libb29NHKB3uJxquSx8ntFFNxhpzZE1wEQMNsq6RLfxMaCibzxsaeKIBWnIUW09KTSmZetF5omaG23g/640?wx_fmt=png&from=appmsg "")  
  
**02**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9kia7DBJvRqTWRlkr3SEwvCjdcMDu3y2hLQYMIgzLUFYK7iaDqO3g6dJm6dscGjuYUF6VFbCNw6x3iaAib5ESds85PSpoib693U7nDE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9n3nvsFwcW7Vc2Oxtb4UwRY0IoVeaEH9kOVM5hEKx08hicmhoO3x6IJ0hgcVmKlzyBj6YC4hia6cOq0RUIIsd51533LxNPW3rsm8/640?wx_fmt=png&from=appmsg "")  
  
  
**Cairn已被多类威胁组织滥用**  
  
**野外攻击事件频发**  
  
Cairn的开发者将其描述为一款通用问题求解引擎，能够“穿越未知状态空间”，寻找从起点到目标的路径。  
  
  
虽然Cairn并非专门为渗透测试开发，但实际使用效果已充分证明其在该场景下能力出众。需要注意的是，它与思科Talos用于狩猎AI集成恶意软件的框架CAIRN并非同一项目，不要混淆。  
  
  
亚马逊是在发现一个公开部署、未设置身份验证的Cairn实例后，才掌握了此次攻击的完整细节。该公司表示，项目配置、操作者的提示词以及AI输出内容均指向某中国背景威胁行为者。  
  
  
近两个月来，亚马逊是第四家记录到威胁组织使用Cairn攻击真实目标的安全厂商。  
  
  
Gambit Security曾从攻击者的中转服务器上还原了一场以在线零售商为目标的牟利攻击活动。操作者将Cairn与另外两款开源AI工具框架Hermes、Strix搭配使用，共攻陷至少27家企业，窃取超过60万条未过期银行卡记录，还植入了刷卡窃密程序。  
  
  
今年10月，NorForge发现了超过20个暴露在公网的Cairn面板，这些面板分属三个以牟利为目的的攻击集群。其中规模最大的是一个中文使用者集群，共运行超过1100个攻击项目，目标主要位于美国、日本和澳大利亚。这些攻击任务中约有十分之一达成目标，成功任务的平均耗时不到23小时。  
  
  
本月早些时候，派拓网络（Palo Alto Networks）Unit 42团队披露了一个疑似中国背景的威胁组织（追踪编号CL-STA-1238），该组织针对半导体、国防及民间团体目标发起25次基于Cairn的攻击，共攻破10个目标系统。  
  
  
目前，勒索团伙、间谍组织滥用的开源攻击框架远不止Cairn一款。攻击者也已将ARTEX、PentAGI、HexStrike AI、CyberStrikeAI等框架用于攻击活动。谷歌分析师表示，威胁行为者还在测试自主搭建的Agent运行框架。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9ngnG4kRvBNKl0qibpcoueVe9ibxykkoUqkqxOmTlYNibXp3HAxArQUsx7DmBlqdiblUd91wdAG6n1yhUrYiaibrmDiaa4j6x94QaC90E/640?wx_fmt=png&from=appmsg "")  
  
**03**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9mJyrU5unl8IoSTMKXWk4SYpZRw1ncqjPPJVqZbKLNlRicA8jWLib2GqXic5ODxhb8f356ibaIYvRl4moIFOFs5NjbUpPxv8S8EEbw/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9nBY6sEl3aOcu8Ajic9sAGP9etia5xqMVeQJWSpwbUV3SfxvFJzPdyxibsljDTZd7iavdX9YH161KUqiaTlncNZ0JEZxTHOaFEqaFVA/640?wx_fmt=png&from=appmsg "")  
  
  
**传统补丁策略已显不足**  
  
**防御需适配AI攻击节奏**  
  
防御方应优先部署高置信度检测规则，并联动自动化响应，再由人工审核最终结果，而不是逐一审批Agent的每一步操作。毕竟，Agent的行动速度往往远超SOC团队的告警分诊效率。  
  
  
面对这类攻击，企业还需要做好长期应对高强度攻击的准备，毕竟Agent不需要休息，也不会产生疲劳。  
  
  
在具体防护措施上，企业应加强身份管理、收敛攻击面，并及时安装安全补丁。其记录的攻击案例显示，攻击者主要利用弱口令、认证绕过漏洞以及尚未修复的旧CVE漏洞。  
  
  
不过，仅靠快速打补丁已经难以应对这类攻击。AI工具不仅成本低廉，还能长时间持续行动，不断调整攻击方式，其耐心、持续性和攻击效率都给传统防御体系带来了新的挑战。企业因此还需要梳理维持业务正常运转所必需的核心系统，并提前验证遭受攻击后能否及时恢复。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9mnDPAVHlZkJm3esQDPibZicKHichePUnOJf7TUO5f2hSCBQnojumyX3QIjz2dqSiasZWwmJuIWBX45H2j7XlZDR2DpduPquY8fPrk/640?wx_fmt=png&from=appmsg "")  
  
声明：除发布的文章无法追溯到作者并获得授权外，我们均会注明作者和文章来源。如涉及版权问题请及时联系我们，我们会在第一时间删改，谢谢！文章来源：FreeBuf  
  
Open-source AI pentesting framework Cairn keeps turning up in real-world attacks  
  
https://www.helpnetsecurity.com/2026/11/23/cairn-ai-agents-autonomous-attacks/  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9nBKic8V1o3g3ekx1SwrKwyTiacZLKIYYHtZJIjIrw61zrqAibFwgFdfRBoxdTtNwOXxfnCqCz5Z2XLyXcnUqRButebXu2muV7I94/640?wx_fmt=png&from=appmsg "")  
  
  
  
