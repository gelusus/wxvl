#  iOS 27.0.1完整攻击链：从消息一键RCE到Chollima持久化越狱  
 白帽子   2026-10-19 00:38  
  
刚发布的 iPhone 18 Pro，在黑客大会上被当场 "越狱"，还能扛住重启  
  
10月16日的巴黎Hexacon安全大会上，安全公司Calif.io一口气展示了四段现场demo，覆盖了从消息应用一键远程代码执行到设备持久化越狱的完整攻击链。这不是单点突破，而是一套可以串起来用的组合拳。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibO9kiauylaDrtQhHadKvotxw39sceu1yuOWhHMKax3ytrbxoNAHlW6REflp63dGEia1ww73Dg8E9S4y3eJNbKwibVjpvQ6Ogh1cibepV3Pk4djk/640?wx_fmt=png&from=appmsg "")  
  
第一段是Signal攻击。演示里一个名为“resume-ash-katchem-2026-v6.pdf”的简历附件在Signal中被打开，PDF渲染出来的不是求职信息，而是Calif的标志，紧接着主屏幕上自动冒出了Calif应用图标。用户只做了“打开文件”这一个动作，代码就已经在手机上跑起来了，应用也被悄悄装好了。  
  
  
第二段是iMessage攻击，代号Glyph Grift。研究人员在iMessage里打开恶意PDF之后，旁边的电脑屏幕上弹出一个实时窗口，标题写着“Glyph Grift live proof”，下面列出一条清晰的攻击路径：Look PDF → SpringBoard（桌面管理器）→ IMDPersistence（消息持久化服务）→ HTTPS。手机里存的聊天记录被一条条外发到远程服务器，其中一条还写着“With <3 from Calif”。  
  
  
这两个demo指向的技术底座是同一个方向。Calif此前公开分析过CVE-2026-86950，一个存在于CoreGraphics（图形渲染框架）里的字体字形渲染越界写漏洞，恶意PDF里嵌入特制字体就能触发内存破坏。iMessage和Signal都会自动渲染PDF附件，这意味着只要用户点开那个看起来像简历的文件，攻击链就启动了。  
  
第三、四段才是真正的重头戏。团队演示了名为Chollima的untethered jailbreak（完美越狱）工具，目标设备是iPhone 18 Pro，系统版本iOS 27.0.1。研究人员当着观众的面重启手机，走完正常开机流程，输入密码解锁，然后通过SSH 连上手机，直接拿到root（最高权限）shell。  
  
  
  
终端输出把技术细节摆得明明白白：id命令返回uid=0确认是root身份，uname显示Darwin Kernel 27.0.0，底层构建号XNU 13432.2.10，跑在T8160芯片上，设备型号标识iPhone19,2，sw_vers确认系统是iOS 27.0.1 build 24A446。neofetch列出67个通过dpkg安装的软件包，uptime显示开机才两分钟。  
  
整个流程覆盖了沙盒逃逸、内核读写、SPTM（Secure Page Table Monitor，安全页表监控）绕过和持久化几个关键环节。  
  
拿到root之后，团队安装了Sileo越狱商店，还装了tweaks（系统插件）比如Gravity Lite。最能说明问题的是重启之后的画面：锁屏界面出现Calif标志和“From Calif with LOVE”字样，设置里多了一个Calif图标，而关于本机里系统版本仍然显示“iOS is up to date”。  
  
这种重启之后越狱自动恢复的能力就是untethered。它不需要插线，不需要电脑引导，用户每次开机都是越狱状态。上一次在现代iOS上做到这件事还要追溯到iOS 9时代，而这次是在最新款硬件加最新系统上完成的。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibO9kiauylaDqekS9UHzUovxmnOs84ea3uDlFBdoicUNtcgPesSp940cC0WSpHUuEq2kbFq9obQtciafyElIZnJ8tLmyFG4xDNVMMqI152q2EAA/640?wx_fmt=png&from=appmsg "")  
  
要理解这件事为什么难，得先搞清楚现代iPhone到底叠了多少层防护。每个app都被关在sandbox（沙盒）里只能访问自己的文件，所有运行的代码必须经过Apple数字签名，启动链上每一步都会校验下一步合法性，内核还有PAC（Pointer Authentication Codes，指针认证码）这种硬件级防护防止内存指针被篡改，MIE（Memory Integrity Enforcement，内存完整性防护）则专门用来挡住内核内存利用。想从一个PDF附件一路拿到持久化root权限，需要把这些层全部绕过去。  
  
AI这次直接参与了漏洞链的发现过程。这也不是Calif第一次这么干，今年4月Thai Duong就分享过Claude如何帮他拆解一个iOS Safari exploit。5月的时候，Calif用Anthropic的Claude Mythos Preview在不到一周的时间里做出了针对Apple M5芯片MIE防护的macOS内核exploit。Duong的评价是，Mythos Preview帮助识别bug并贯穿了整个exploit开发过程。  
  
Thai Duong在Google做了12年安全和密码学，参与创建了Tink和Wycheproof密码学库，还和别人一起发现了BEAST、CRIME、POODLE这几个著名的SSL/TLS漏洞，拿过Pwnie Award的最佳服务端漏洞奖。整个Calif团队的定位就是用AI加速漏洞研究，同时给AI模型本身做红队测试。  
  
Apple自己的iOS 27安全说明里其实已经提到了Calif.io，写的是“in collaboration with Claude and Anthropic Research”，归因于两个导致crash的bug。Hexacon大会是Synacktiv主办的offensive security（攻防安全）会议，为期两天。之前Cellebrite曾经公开表示iOS 27在任何当前物理设备上都无法越狱，iOS 18以上也没有可靠的公开越狱方案，这次Calif直接在最新硬件加最新系统上跑通完整持久化链，等于当场推翻了那个判断。  
  
当然，对普通用户来说现在不用慌。方法还没公开，没有下载链接，网上任何声称能越狱iOS 27的工具基本都是恶意软件。但问题在于，能在一台完全更新的iPhone上从一个PDF附件打到持久化root，这种能力和商业spyware厂商卖给政府的东西是同一类。Citizen Lab前几天刚发了AdFection的研究，那是一种通过定向广告投递的zero-click（零点击）攻击，Hexacon上展示的技术和它属于同一个能力层级。  
  
Calif在大会最后只留了一句话，“Everything will be released in due time”。Apple通常在补丁就绪之前不会确认安全问题，iOS 27.1目前还在beta阶段，里面有没有修掉Calif用的那几个bug，要等Calif公布细节或者Apple自己说。对于记者、活动人士、企业高管这些高风险人群，建议照旧：系统更新第一时间装，重要账号开2FA，考虑启用Lockdown Mode（锁定模式）。对大多数人来说，保持系统更新就够了。  
  
AI正在把漏洞挖掘的门槛往下拉，苹果系统的防线当然也在升级，但攻防这盘棋，节奏已经不一样了。  
  
