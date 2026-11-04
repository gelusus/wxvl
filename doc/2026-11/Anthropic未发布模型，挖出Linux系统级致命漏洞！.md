#  Anthropic未发布模型，挖出Linux系统级致命漏洞！  
 谈思实验室   2026-11-04 09:53  
  
点击上方蓝字  
谈思实验室  
  
获取更多汽车网络安全资讯  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzOTc2OTAxMg==&mid=2247580896&idx=1&sn=36a65f68d014c79310fdeb0e1e55b6b0&scene=21#wechat_redirect)  
  
  
AI 再次引发网络安全大地震！  
  
  
据报道，在Ubuntu和Red Hat系统中，一个近乎满分的Linux内核漏洞至今仍未修复。更令人毛骨悚然的是，Anthropic尚未发布的一款前沿大模型，竟然成功地将这个原本停留在「理论层面」的漏洞，自动转化为可远程获取root权限的「高危利用链」！  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaBGbPjvwJgdeWNQYjALWGz3tWEdpVV6deqAtc5TZfwJt2LyiapoQdrv59ZiaUlibqiaRgHIogO34gYRWK6lLzsBHDMHReCmgf9t15s/640?wx_fmt=png&from=appmsg "")  
  
想象一下：你的服务器在互联网上默默运行，没有密码、没有交互、甚至不用用户点一下鼠标。一个陌生人发来几组精心构造的TCP包，半个多小时后，对方就能以root权限在你机器上执行任意命令。  
  
  
现在，网络安全防御逻辑彻底崩塌了——AI改变的不是漏洞的数量，而是将漏洞武器化的速度，从  
「数月」硬生生压缩到了  
「数小时」。  
  
  
当AI化身不知疲倦的  
「超级黑客」，我们的数字世界还安全吗？  
  
**01**  
  
**满分级绞肉机：CVE-2026-98432到底有多可怕？**  
  
  
在网络安全界，CVSS（通用漏洞评分系统）满分为10分。  
  
  
任何超过9分的漏洞，都足以让全球各大公司的安全运维人员半夜从床上惊醒，连滚带爬地去加班打补丁。  
  
  
而这次的主角，**CVE-2026-98432，评分高达9.8分。**  
  
  
它网络可达、无需认证、无需用户交互、高机密性/完整性/可用性影响。从Linux 4.4开始就存在，几乎覆盖了过去十年的所有主流服务器内核。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaDKDpQ2sMByU8RWtEd6foI673Tia5NRpKNibz5m2tnKWPXicDic08bp43mStibZtTvmObicak6Xs5tUmlbakPnwrptOmWOddeQ9xzRm0/640?wx_fmt=png&from=appmsg "")  
  
这是一个存在于IPv4 TCP握手过程中的「释放后重用（Use-After-Free, 简称UAF）」漏洞。  
  
  
简单说，内核在创建新连接时，把IP选项内存从「请求套接字」直接转给了新套接字，却没有正确等待所有CPU读完就释放了。  
  
  
另一个CPU还在用这块内存发SYN-ACK，结果就踩到了已经释放的内存。  
  
  
打个比方：  
  
假设服务器是一家酒店，IPv4 TCP握手就是客人（数据包）去前台开房的过程。UAF漏洞相当于：一个客人退房了（内存被释放），但系统因为某种逻辑错误，没有收回他手里的房卡。当下一位无辜的客人（或者核心系统程序）住进这个房间时，那个持有「过期房卡」的黑客，就可以大摇大摆地开门进去，甚至直接接管整个酒店的控制权（即获取Root权限）。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaD6m70144MmoRRuTy5MImtA8hPUIozOWc8IuF4Xkicvv16Cnc8hxq0dRuTMNhjxKicjtO1GhicWwfvB8BFICvCUxYB3zJnGFSuLKM/640?wx_fmt=png&from=appmsg "")  
  
最致命的是，触发这个漏洞**「无需任何凭据」**  
。  
  
黑客不需要知道你的账号，不需要知道你的密码，甚至不需要你在服务器上运行什么复杂的高危服务。  
  
  
不需要钓鱼、不需要用户点链接、不需要已有权限。只要你的机器对外暴露了IPv4 TCP服务（几乎所有公网服务器都是），理论上就可能中招。云厂商的Network Load Balancer如果透传IP选项，后面的目标机器也会受影响。  
  
  
因为漏洞存在于最底层的IPv4 TCP握手中——只要你的服务器还连着网，还在接收网络请求，你就暴露在枪口之下。  
  
  
最危险的一点在于：失败的竞态只是让握手失败，系统不会崩溃。攻击者可以无限重试，直到成功。  
  
Anthropic的实验里，大约48分钟就能泄露内核指针（绕过KASLR），再花5分钟就能拿到root shell。整个过程大约53分钟。  
  
  
Anthropic明确说，针对的是开启了Multipath TCP（MPTCP）监听的服务器。而Go 1.24及以后版本，默认就会在Linux上为监听套接字开启MPTCP。  
  
  
无数用Go写的微服务、API网关、云原生服务，可能正默默开着这扇门。  
  
  
目前，**Ubuntu和Red Hat（RHEL）这两大占据企业级市场半壁江山的操作系统，依然没有发布修复该内核漏洞的正式补丁！**  
  
  
这意味着，全球可能有数百万台乃至上千万台服务器，正处于大门洞开的「裸奔」状态。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaDsiaf21WRDIdzuwPkWpicbTHB9G9fNmBVyvxZACExBiaHq3C8Z9MjO95G310xZO7GxR4RibG8gZjq8Aotswqod7oqzRsiajoxPTzh4/640?wx_fmt=png&from=appmsg "")  
  
面对9.8分的底层漏洞，任何「偏方」都是自欺欺人。  
  
  
目前，**只有Debian、Amazon Linux和Oracle UEK等少数发行版从内核层面真正修复了这个问题。**  
  
  
广大Ubuntu和RHEL用户，正坐在一个引信已经点燃的火药桶上。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaCwbzz5xWC4ECpCA3kaNsgvjhCicIlyvbLrEZMSiauVfqiaoP5vaX0zuJOdacVxulmT2seTGwGyUashkribrjsbgAJEYkkmIpkoQhA/640?wx_fmt=png&from=appmsg "")  
  
**02**  
  
**AI并没有「发现」漏洞，但它做的事更可怕**  
  
  
这场危机的真正内核不是AI 发现了又一个漏洞，而是 AI 彻底颠覆了行业——有趣的部分并不是一个未发布的模型找到了远程 root 路径。而在于该模型完成了缓慢、机械的工作：把一个已知的 use-after-free 转成无需凭据即可工作的利用链。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zQ19N6bPViaDWXnZ94ibWzpFu4ibOldrE3qn5nDnfoqdIk4KkHtib7pYXekRAx1XOboTFyhX7TRuBNofkf4j5myr6ticIRm2Tk94wbOrFBJ8fEcU/640?wx_fmt=png&from=appmsg "")  
  
长久以来，网络安全界存在一个微妙的「时间差保护伞」。  
  
  
当一个极其复杂的内核漏洞被公开时，它通常只是概念证明，或者仅仅停留在理论层面。为什么？  
  
因为要把一个底层的内存漏洞变成一个百分之百好用的**利用链**  
， 极其缓慢、极其考验黑客耐心，是一项「机械性手艺活」。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaCibPkGJ63icYEdOFFZXMqnnDL8r5aEte8pFbxO6FBMgOicvJx3GNc0ibArMiaiasHRrIEx92nVSxb0FNEBhAE2hEAAylO2lntYMTemA/640?wx_fmt=png&from=appmsg "")  
  
要打磨出一把真正能用的「钥匙」，顶尖黑客可能要几周甚至几个月，不断尝试内存空间的布局，编写繁琐的代码，串联多个微小的逻辑缺陷。  
  
正是这「几周到几个月」的时间差，给了全球各大软件厂商开发补丁的时间，也给了各大系统管理员更新系统的时间。  
  
  
**但是，Anthropic的未发布AI模型，亲手捏碎了这个保护伞。**  
  
  
AI最擅长的是什么？就是不知疲倦、极其精准地处理那些缓慢、复杂、需要巨大工作量的机械性逻辑推演。  
  
  
这个Anthropic未发布的AI模型是一个「怪兽」，既拥有着人类顶级黑客直觉，同时又有着机器般算力和耐心。  
  
  
当CVE-2026-98432的理论漏洞摆在它面前时，它可以在极短的时间内，自动推演千万次内存状态，把那些散落的理论碎片，拼接成了一把直插服务器心脏的利刃！  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zQ19N6bPViaCX0A0n7ID3abSHtC1icIw3WErqJia6G0aTuFfJ98aTsUacqggwXDxibTvLo46UBSQZHjpCAqgwyOhk2CicvWvkPoibuU0lBibiaZRUPo/640?wx_fmt=png&from=appmsg "")  
  
Anthropic这次只是披露了结果，没有公开利用代码。但一旦类似能力扩散，漏洞从「发现」到「武器化」的时间会急剧缩短。  
  
  
**「如果模型能够可靠地完成这项工作，那么从『公开漏洞』到『可用漏洞利用』的窗口就会从数周或数月缩短到数小时。」**  
  
  
这就是本次事件最令人胆寒的地方！它改变的不是网络世界里漏洞的绝对数量，而是极其残忍地改变了防守方的「实际安全态势」。  
  
  
以前，系统管理员看到一个漏洞公告，可以喝杯茶，排进下个月的更新计划里；现在，漏洞公告发出的几个小时后，由AI全自动生成的自动化攻击武器可能就已经在暗网上泛滥了。  
  
  
人类的肉身运维速度，如何拼得过AI的代码生成速度？  
  
**03**  
  
**我们正在被迫卷入一场「降维打击」**  
  
  
诺贝尔经济学奖得主Daron Acemoglu曾警告：  
  
AI可能让普通劳动者沦为二等公民。  
  
而在网络安全领域，这种降维打击已经提前上演。如果不掌握顶级AI防御工具，传统的安全工程师正在沦为网络战场上的「二等公民」。  
  
  
在过去，只有国家级的高级持续性威胁组织，才有财力和人力去将复杂的0day或Nday漏洞武器化。因此，普通企业遭到这类高级利用链攻击的概率并不高。  
  
  
但现在，AI正在将「制造顶级网络武器」的能力**平权化、廉价化、瞬时化。**  
  
  
试想，一旦这种类似Anthropic前沿模型的能力流出，或者被黑客组织逆向工程实现，会发生什么？  
  
一个初级脚本小子，只需要输入一段提示词：「帮我针对昨天刚爆出的Linux CVE漏洞写一个远程Root攻击脚本」，AI就能在几分钟内生成一套极具杀伤力的代码。  
  
  
防御方还在慢吞吞地走流程、测兼容性、等厂商发补丁；攻击方的「AI军工厂」已经将最新的导弹量产并倾泻而下了。  
  
  
这种不对称的战争，怎么打？  
  
  
AI时代的车轮正在碾压一切旧有的规则。在这场由大模型掀起的网安降维打击中，没有人能独善其身。  
  
  
我们要么学会在AI的辅助下建立起「分钟级」甚至「秒级」的新一代自动化防御体系，要么，就只能眼睁睁地看着我们在互联网上建立起来的数字大厦，被AI生成的炮火轰得粉碎。  
  
  
真实世界里谁最危险？  
  
  
目前公开情报显示，还没有大规模在野利用的确认报告，但扫描已经出现了。有安全人员观察到针对MPTCP端口的探测流量明显上升。漏洞窗口期一旦被真正武器化，后果不堪设想。  
  
  
**现在立刻能做什么？**  
  
1.优先打补丁并重启  
  
Red Hat/AlmaLinux/Rocky：尽快装上11月初发布的kernel包并reboot；Debian/Amazon Linux：直接更新并重启；Ubuntu：关注官方安全公告和-proposed，正式发布后第一时间上。  
  
2.临时缓解  
  
如果不需要MPTCP：sysctl -w net.mptcp.enabled=0。用nftables/iptables过滤带异常IP选项的TCP包，对不需要公网访问的服务严格限制源IP。  
  
3.盘点资产：  
  
立刻扫一遍（uname -r），检查是否有MPTCP监听（ss -tlnp | grep mptcp），哪些机器跑着Go 1.24+服务。  
  
补丁已经有了，只是有些发行版还在路上。现在不是观望的时候，是行动的时候。去检查你的服务器吧。别等别人替你「检查」。  
  
  
**留给人类打补丁的时间，不多了。**  
  
****  
**来源：新智元公众号**  
  
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
  
  
