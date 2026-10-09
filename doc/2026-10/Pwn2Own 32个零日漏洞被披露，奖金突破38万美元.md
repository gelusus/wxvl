#  Pwn2Own 32个零日漏洞被披露，奖金突破38万美元  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-09 00:40  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6Ojzg2x3Oan41k3EwE3hhI5hacQMspdibaf6RHJ69Ribc6unn0IYx8sl0XfSO4Zyxkl37VLr9tD9WNtS9BDx7rkCibA0n88HuBicUM/640?from=appmsg "")  
> **导语**  
：由 Trend Micro 旗下 Zero Day Initiative（ZDI）主办的 Pwn2Own Ireland 2026 在爱尔兰科克开幕，首个比赛日共完成 21 次现场攻击演示，披露 32 个此前未公开的零日漏洞，单日发放奖金达 388,500 美元。三星 Galaxy S26 成为首日被攻破次数最多的目标。  
  
## 一、赛事概况  
  
Pwn2Own Ireland 2026 于 10 月 6 日在爱尔兰科克（Cork）开幕，是该项赛事在爱尔兰举办的第二届。本届规模为历届最大，组委会共收到超过 60 个参赛条目，覆盖手机、智能家居、打印机、AI 基础设施、可穿戴设备等多个消费与企业级品类。首个比赛日安排了 21 次现场攻击演示，每个参赛团队需要在限定时间内完成对指定目标的完整漏洞利用链，并在现场向裁判演示从初始入口到任意代码执行的全过程。  
  
赛事沿用 ZDI 一贯的协调披露机制：被攻破的厂商将在 90 天（或厂商与 ZDI 协商确定的更短周期）内获得漏洞细节，再由 ZDI 公开披露漏洞详情。攻击成功后获得的奖金与「Master of Pwn」积分决定了团队的最终排名，冠军队伍将在最后一天的 Pwn2Own 颁奖环节获得额外奖励。  
  
![赛事舞台效果图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NGCz63cCS8AG0KPNcCpcyh5oLBVvsZDonL7K59xZOUbeAmDtnvZp7UOSLCFruJuNRCWrtCF4buLo9wPabKnIaor6pNsP2EDpc/640?from=appmsg "赛事舞台效果图")  
## 二、首日战果盘点  
  
首日 21 次演示的整体结果大致如下：  
- 13 次完全成功（披露的漏洞全部为新零日）  
  
- 6 次部分成功（含部分碰撞，即漏洞已被厂商知晓但尚未修复）  
  
- 2 次未能完成演示，未能取得奖金  
  
按品类分布看，手机与 AI 基础设施是首日最「热门」的靶子。三星 Galaxy S26 在首日被 4 支团队先后攻破 6 次（含多次碰撞），其中 Viettel Cyber Security 与 Interrupt Labs 各贡献了一次成功演示；这是 Galaxy S 系列连续第二年在 Pwn2Own Ireland 上成为手机品类的重点目标。AI 基础设施品类是本届新增亮点，LiteLLM、OpenAI Codex、Oracle Autonomous AI Database 都成为被攻破的目标。  
  
按金额看，单笔最高奖金 50,000 美元由独立研究员 McCaulay Hudson 拿下，他在 Sonos Era 300 上组合使用 OOB Write 与格式化字符串漏洞完成利用。紧随其后的是多笔 40,000 美元奖金，分别由 VinSOC（Philips Hue Bridge Pro、Oracle Autonomous AI Database）、Xint（LiteLLM）、Ikotas Labs（OpenAI Codex）获得。  
## 三、几个值得关注的细节  
  
首日亮点中，VinSOC 越南安全研究团队表现最为抢眼，旗下成员 Vũ Chí Thành 与 Huỳnh Đức Tin 在 Philips Hue Bridge Pro 上一口气披露了 7 个零日漏洞，是本届单场披露量最大的一次尝试。Summoning Team 的 Sina Kheirkhah 在第二轮演示中只用一个漏洞就攻破了 Lexmark CX532adwe 打印机，再次拿下 10,000 美元奖金。Interrupt Labs 凭借对 Garmin Index BPM 血压监测设备的 OOB Read + OOB Write 组合漏洞，成为本届「健康可穿戴」品类的首位获胜者。  
  
碰撞情况同样普遍。Out of Bounds、VinSOC、Xint 等多支团队在演示中使用了已被厂商知晓但尚未修复的漏洞，仍可获得 ZDI 发放的部分奖金与积分，但不再计入「新零日」统计。这意味着大量厂商即便参与了 Pwn2Own，仍有未修复的旧漏洞在赛场被反复使用——这是协调披露机制下常态化的现象。  
  
![赛事奖金与奖杯概念图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6Mxo3riaVeHZvicXrm2K5OyrbtfwyibPhtdWDibINdjOuzncAVMZrMPCM6klyyicO83TJuNddexYmEL45048XsVkGWu6a2eDmU8m6oc/640?from=appmsg "赛事奖金与奖杯概念图")  
## 四、与去年爱尔兰州对比  
  
作为参照，2025 年首届 Pwn2Own Ireland 共披露 73 个零日漏洞、发放奖金 1,024,750 美元，其中 Summoning Team 以 187,500 美元的成绩拿下总榜第一，主要来自对 Samsung Galaxy S25、Synology DS925+ 与 CC400W 摄像头、Home Assistant Green、QNAP TS-453E 等多目标的连续攻破。2026 年仅在首个比赛日就发放 388,500 美元，按比例看本届的节奏与金额都要快于去年。考虑到本届赛程还有 2 到 3 个比赛日，最终奖金总额很可能刷新去年纪录。  
## 五、对厂商与安全行业的意义  
  
Pwn2Own 一直是消费与企业级设备安全的「年度体检表」。本届首日的几个趋势值得关注：  
  
一是手机厂商对供应链与第三方组件的加固仍不充分，Galaxy S26 在 4 个团队的连番攻击下都难以守住——攻击面主要来自基带、图像处理、调制解调器等组件层，而非传统意义的「系统漏洞」。  
  
二是 AI 基础设施成为新的攻击面。LiteLLM、OpenAI Codex、Oracle Autonomous AI Database 等被列入靶子，意味着大模型相关的开发与部署组件已正式进入主流攻击研究的视野。  
  
三是健康可穿戴品类首次纳入，意味着攻击者对个人健康数据的关注正在上升。从演示结果看，Garmin 这类设备的固件与无线协议仍存在可被利用的内存破坏漏洞。  
  
随着本届赛事推进，ZDI 将在赛后统一发布技术细节与漏洞编号。对于普通用户而言，最直接的建议是关注自己所用厂商的固件更新——大多数被攻破的产品都会在 90 天披露窗口内推出修复补丁，及时升级仍是规避相关风险的最简单手段。  
  
**参考链接**  
- ZDI Day One 官方记录：https://www.zerodayinitiative.com/blog/2026/10/6/pwn2own-ireland-2026-day-one-results  
  
- BleepingComputer 报道：https://www.bleepingcomputer.com/news/security/hackers-exploit-32-zero-days-on-first-day-of-pwn2own-ireland/  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6P4OFvoklcZUKzbac7jgAvXEYZUgj64ttP6vL1eu8a7oM933EExEqJZC2AS7CBu45dib0zxMvnBkGBZdiacyTAR0ptdtFibNhoyeU/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
