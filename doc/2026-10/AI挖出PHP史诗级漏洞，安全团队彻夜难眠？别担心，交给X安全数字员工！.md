#  AI挖出PHP史诗级漏洞，安全团队彻夜难眠？别担心，交给X安全数字员工！  
 安恒信息   2026-10-22 10:19  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/mc7Mmwou4T80KsMmxBtGSfhFMqDlick5aBXnmREbkZ2osGbIJze4yq4iaUTJx9WIh75sW1lQEvuhCibfPJkLf5GZw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
  
一场波及全球互联网的重大安全危机，正在引发全球泛关注。  
  
10 月 21 日，PHP 官方发布紧急安全更新，修复两个被统称为「PHP2Shell」的严重远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619，CVSS 均为 9.2）。这两个漏洞影响 PHP 5.4 至 8.5.11 的所有版本——  
覆盖全球约 70% 的网站，且攻击不依赖任何应用代码，一个普通请求即可触发。  
  
最值得关注的是发现过程：OpenAI 的 GPT-5.6 Sol（安全团队 Calif 使用）与谷歌 Wiz 借助Gemini 4 Argon  
分别独立  
找到了这组漏洞，随后由PHP维护者主导，联合Cloudflare、CISA等机构展开了长达两个月的协调修复，赶在攻击者之前补上了缺口。  
  
OpenAI称这是  
「前沿 AI 在攻击者之前发现高危漏洞」的标志性案例，为防御方争取到宝贵的「防御者之窗」；但它也同时承认，同样的能力完全可以被攻击者用来更快地开发利用工具——  
AI挖洞攻防两端同时提速的时代，已经到来。  
  
  
  
  
当AI加速漏洞发现  
  
安全防御也需要全新模式  
  
  
  
此次PHP重大漏洞事件，揭示了AI时代网络安全攻防的一个重要趋势。一些潜伏多年、过去难以发现的深层安全漏洞，正在AI的帮助下被重新挖掘出来。  
  
过去，一次重大漏洞应急，往往需要多个专业团队接力：情报团队追踪漏洞，研究人员分析复现，渗透测试人员验证风险，安全运营人员排查资产、响应处置。  
  
但在AI加速的网络攻防的今天，企业需要的不仅是更快的漏洞扫描工具，而是一套从情报发现、漏洞验证到风险处置、持续运营的协同安全工作模式。  
  
这正是安恒信息X安全数字员工所探索的新模式。通过零日雷达与X安全数字员工协同应战，让不同专业安全能力协同运转，帮助企业更高效地应对AI时代不断涌现的安全挑战。  
  
  
情报先行：  
  
零日雷达，捕获漏洞情报，启动RCE验证  
  
  
面对PHP2Shell这样的高危漏洞，第一步是  
尽快掌握准确的漏洞情报。  
  
安恒信息零日雷达持续追踪全球漏洞动态，捕获高危漏洞信息，分析漏洞成因、影响版本、利用条件及潜在风险，并结合技术研究开展RCE可利用性验证。  
  
但发现漏洞只是开始。更重要的是，如何让情报迅速转化为面向客户的安全行动？  
  
依托零日雷达的情报赋能，X安全数字员工能够围绕同一漏洞事件开展专业分工与协同响应。  
  
一条漏洞情报，触发一支AI安全团队的协同作战。  
  
  
X安全数字员工协同应战：  
  
从漏洞复现到客户风险闭环  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/UDBWiaGVgFVic47nw3B7EFLm2tQYqxOzvbFvZV0O4Kxc6iaRyPZhjYoLOPlNnnVC2ic4nliasLHhKrZYSZTvSs6lxlQ/640?from=appmsg "")  
  
安小虎  
  
定向复现，把漏洞研究透  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/cyaBr7eyibPLNHKPHBiaPibtj0mE6Bpp3F1gre9m8mvoWed3f2HVAib4YtGwT6DL0MwGhvCsObyU2CkZDWOHXiavIicg/640?from=appmsg "")  
  
  
代码审计专家安小虎，依托“特定漏洞复现”能力，根据指定CVE编号、漏洞描述及相关代码文件，开展针对性漏洞分析与复现。  
  
针对PHP2Shell这样的高危漏洞，安小虎可围绕漏洞触发条件、利用方式和潜在影响开展定向研究，为后续安全测试提供技术依据。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapE6uqYqE5n3iboyQZn0N4w5En3utAzxLrFx5FXPiaoensadErFWpC6FLBQrK4mZ3X7gCpCXtO7yuYkOkUiaV5fUL4G64IKbKH3f5M/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapFF1xncGsV23MIzYCUgjHpzVmQHJVnFrh2dWxvsfN84A7ErW81fjwS1dDcdKficMDdPnNIknkGwWhGFsI7PpWFTnuicgRyeTibspk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapEg4a1gccmBupCoILXXfa2nNd1iaGnliccWZYmf4qicylVjtzdHSsxIn3e40pTf0LsVeDgicfgn2EAQeWgVjKNiafRtaHgcbYTC6iaj0/640?wx_fmt=png&from=appmsg "")  
  
  
复制粘贴至浏览器查看复现回放：  
  
https://vlf.das-ai.com/assistance?share=29125112b5954823ace51a30df744c15  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/UDBWiaGVgFVic47nw3B7EFLm2tQYqxOzvbFvZV0O4Kxc6iaRyPZhjYoLOPlNnnVC2ic4nliasLHhKrZYSZTvSs6lxlQ/640?from=appmsg "")  
  
安小龙  
  
自动检测，快速验证真实风险  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/cyaBr7eyibPLNHKPHBiaPibtj0mE6Bpp3F1gre9m8mvoWed3f2HVAib4YtGwT6DL0MwGhvCsObyU2CkZDWOHXiavIicg/640?from=appmsg "")  
  
  
渗透测试专家安小龙，结合零日雷达漏洞情报、安小虎的复现成果及安全能力中心的技术支持，已集成针对本次PHP高危漏洞的专项检测Skill。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapHQIkMyNePLs8PNxtf2SsBOSj0kvTzgMAPDtjeNlMYeibuDgVvzl2wrBYRlAg00yiaht3VmBRnagzhajJwsiaapyHd7qfSQaewN98/640?wx_fmt=png&from=appmsg "")  
  
安小龙已内置对应漏洞的渗透skill  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapE5eOSp6f5BjRlH0zSl9NrTQ66x1rqvGU5HZmo3BUrvo2FRHPOPniadP5uLKZn5ICO9vtiaYicRlJoQWC1RQWp5jTiadnDdCSwChIM/640?wx_fmt=png&from=appmsg "")  
  
可顺利挖掘出对应漏洞  
  
  
在客户授权环境中，安小龙可自动开展目标资产漏洞检测，快速识别潜在风险、验证漏洞情况，并生成检测报告，为风险处置提供更精准的依据。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/UDBWiaGVgFVic47nw3B7EFLm2tQYqxOzvbFvZV0O4Kxc6iaRyPZhjYoLOPlNnnVC2ic4nliasLHhKrZYSZTvSs6lxlQ/640?from=appmsg "")  
  
安小风  
  
情报直达客户资产，快速锁定影响面  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/cyaBr7eyibPLNHKPHBiaPibtj0mE6Bpp3F1gre9m8mvoWed3f2HVAib4YtGwT6DL0MwGhvCsObyU2CkZDWOHXiavIicg/640?from=appmsg "")  
  
  
与此同时，安全运营专家安小风已经可以启动另一条响应线。  
  
通过对接云端零日雷达情报，安小风直接关联客户侧资产信息，快速开展漏洞排查，定位潜在受影响的服务器和业务系统。  
  
结合漏洞验证结果，协助安全团队开展风险研判、确定处置优先级，推动修复和复测。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapETyTIlW1TuBYqWEMoHnKB0zPOvr9DEpgDuEUdicdIzboAiaMO8Fyf6632toTvKy1IOlo6KicicWXnjZsdssdciaaS16sIJmq9s7b3g/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/UDBWiaGVgFVic47nw3B7EFLm2tQYqxOzvbFvZV0O4Kxc6iaRyPZhjYoLOPlNnnVC2ic4nliasLHhKrZYSZTvSs6lxlQ/640?from=appmsg "")  
  
安小云  
  
一键联动，盘活现有安全工具  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/cyaBr7eyibPLNHKPHBiaPibtj0mE6Bpp3F1gre9m8mvoWed3f2HVAib4YtGwT6DL0MwGhvCsObyU2CkZDWOHXiavIicg/640?from=appmsg "")  
  
  
安全托管专家安小云，同样可以获得零日雷达的情报赋能。  
  
通过联动安小龙、安小虎、EASM等安全能力，并对接客户已有的GOBY、FOFA及各类漏洞扫描工具，安小云能够协助完成资产与漏洞风险摸排。  
  
针对本次PHP漏洞，还可自动分析SAPI部署方式及应用配置，辅助识别潜在风险，推动后续处置与持续安全运营。  
  
对于  
专业安全力量有限的企业，无需重新搭建庞大的安全体系，也能充分利用已有工具，降低安全运营门槛。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/QsTClNuIiapGmiceyMb9KYEXbo5PFMZz29QIYGZmGyCAWoB2fDX1Au5ZonQQopvia7twz6ibIQcpBQ4Mg4JFHpeiacJYDCVXB1353DNBADtibqnL0/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QsTClNuIiapEm3xHloFIglbQDia0Xqu3MwoFqcccfuCxyWKsdbkuOcEribbicfBNdQCetxqbgXAnCvshg2LNK1dvEE7ibkXYHpXOhWILrxg7zFSI/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/kmrotDJESpLvI0twmANRmwSGRtJHOicIiapj9OQuGoHUeRmr7mshEDcRibYnNMpdkYO3d8NR1CwKXfTyOtSKgBH7Q/640 "")  
  
重大漏洞当前，怎么办？  
  
  
重大漏洞面前，不是一个AI在战斗，而是一直AI安全团队在协同。  
  
从零日雷达捕获漏洞情报，到安小虎定向复现、安小龙渗透测试，再到安小风精准研判、安小云联动排查，四位X安全数字员工围绕同一安全事件协同作战。  
  
从风险发现、验证到响应处置、持续治理，形成完整的安全工作闭环。  
  
##   
  
过去，一次重大漏洞响应，往往需要多个专业岗位、多支团队协同完成。  
  
今天，专业安全能力正在以“团队”的方式被组织和交付：一个数字员工做好一件事，多个数字员工协同做好一件复杂的事；安全专家则从重复劳动中抽身，回到判断、决策与统筹的位置。  
  
这不是多了一个工具，而是  
安全能力的组织方式变了。  
  
当漏洞产生的速度超过人力的速度，比拼的就不再是谁的人更多，而是谁的响应体系更快成型。  
  
如果您的团队也正被“告警看不完、漏洞验不完、报告写不完”困住，欢迎联系我们，用一次真实环境的演示，看看一支  
安全数字员工团队能在您的业务里跑出什么结果。  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/QsTClNuIiapGrAcsnj0YwyBljH8k0tBrQMT2nZsJWbC7hht449j842Rp4GZSgovlmYhkX6D91icVicYZ1icvUUmp0BF4UJe1gN8ov5S8Ewqs0G8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8 "")  
  
  
  
扫码获取免费试用  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6Ppczr1X4OOTmbg4rENrqwqbYqRtgl3icicic9an9TicNrOnKOT4t2icSyx7w/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=7 "")  
  
**点点赞**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6P0OFJt7aruYwYjIWic5WCu7iaE9ZYWmW6TKPcvrib6Itmpc0dnMqlANmow/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
**点分享**  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PwKHSiaCHQrj4D3mJJZ7QGPX1zbt3rJEKjhdBkX12A8r5L47fI28upaA/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=9 "")  
  
**点喜欢**  
  
  
点击下方名片立即关注  
  
不走丢哦！  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PzUETDErlviazPRtpVtc98iasfL8RCCib8yzmeGJ9HrBlJASFn5qz95QwQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=9 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6Pb2aOfdTZnZZbozL1mvicIWsdWicdDcibz2SAuHblLHicWQc8CmX6WT6OMA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=10 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icVz8RbowK3w0BggvHRq4iaXMuxkRPMb6PBQVNtt5ynIG7UjUHbRJFvFXgKjZW6mzjjhlxfjwtXfrdJyPgroKWQA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=11 "")  
  
  
**往期**  
**精彩****回顾**  
  
  
  
  
  
  
[客户问：想做代码审计，但没源码，只有部署程序，可以吗？](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653110&idx=1&sn=464a985e86d7603429f71711109e1204&scene=21#wechat_redirect)  
  
  
2026-10-20  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653110&idx=1&sn=464a985e86d7603429f71711109e1204&scene=21#wechat_redirect)  
  
  
[第六届奇点论坛｜王欣分享：X安全数字员工 = 安全专家X机器速度](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653092&idx=1&sn=01e17d1b42a30691b65d8dcdeedec4af&scene=21#wechat_redirect)  
  
  
2026-10-19  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653092&idx=1&sn=01e17d1b42a30691b65d8dcdeedec4af&scene=21#wechat_redirect)  
  
  
[首届全球具身智能应用创新挑战赛|百余支队伍角逐，冠军公布，圆满收官！](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653082&idx=1&sn=10485f6bb02c722266c4647ce2fdda0b&scene=21#wechat_redirect)  
  
  
2026-10-18  
[](https://mp.weixin.qq.com/s?__biz=MjM5NTE0MjQyMg==&mid=2650653082&idx=1&sn=10485f6bb02c722266c4647ce2fdda0b&scene=21#wechat_redirect)  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/icVz8RbowK3wm5VicXg2ibVyMsjPZ3OJSzTwdeSU207GIcBicQDzkDVgFNvXD0npWKhFtBb2VtiaczibVqM8HE0vRoNw/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=14 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/cbge2dLXia4bbQyBibIkuZz7hZVFlib5Oib5VZCiacheOLBHzKc88VEQYkXdC7BfdzLSVml0HWO6RjrP7FHL8oL0ONw/640?from=appmsg&wxfrom=5&wx_lazy=1&wx_fmt=other&tp=webp#imgIndex=43 "")  
  
法律声明  
  
本文数据均来自内部统计、媒体报道、公开信息整理等，仅供信息分享，可能存在统计口径差异或误差，敬请理性看待，我们不对其准确性承担责任。  
股市有风险，投资需谨慎。阅读者在作出任何投资决定之前，应当咨询各自的顾问。本公众号发布内容仅代表内容创作视角，不构成任何投资建议或投资依据。在任何情况下，本公众号及运营主体不对任何人的投资结果承担法律责任。  
本公众号原创内容，欢迎合法合规复制、转载，转载时请务必注明出处，不得断章取义、以偏概全或进行有悖原意的引用。  
  
