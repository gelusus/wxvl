#  据称，苹果Safari浏览器零日漏洞正在出售  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-11-26 00:58  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M9rfR2aHAnvVNVugLiaksTvaicvyJwfwAorgFpt3nZy4HR8JsCLqEzZUGKRTONtw93tAXxj4rwZFyDZvlwB09ibmAmNqibLcUfHG0/640?from=appmsg "")  
> **导语**  
：一条 X 帖子，一张售卖单。名为 "3R1C" 的账号公开叫卖一条 Safari 零日利用链：专打 iOS 27.2.1 的 iPhone 16 全系，点开链接就落地，落地就是远程 shell，顺手把钱包容器、助记词截图和剪贴板一起搬走。目前苹果没有对应公告，这条链也没有 CVE、没有公开 PoC——按卖家的说法，它"从未被上报"。  
  
## 一、卖的是什么  
  
DarkWebInformer 在 11 月 25 日转述了这条售卖帖，附了三张原文截图（原始出处：https://x.com/DarkWebInformer/status/2125632639751077973 ）。卖家的帖子标题很直白：**"[SELL] iPhone16 & iOS 27.2.1 Safari drive-by 0day → remote shell A18 only: Crypto wallet targeting"**  
，正文同时给了俄语版。  
  
![漏洞售卖帖原文](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6O8b3K8qtvInXLKvmRljyicX843RpCORiamsSgrClR1xIibgd2Few1DMr510yib124iaImeGve3ia8CZDAdquu6Jv2wLugF1OKx53q4w/640?from=appmsg "漏洞售卖帖原文")  
  
几个自曝细节值得记下来：账号 10 月 3 日注册，只发过三条推文，标注付费注册；帖子状态是"Available"，价格"私信面谈"；截至 2026-11-25（iOS 27.2.1，build 24B156）仍属**未打补丁**  
，卖家明确写明"未上报苹果、无 CVE、无 writeup、无公开 PoC，首次公开出售"。  
  
卖家的动机写得很坦率——他要换目标了，这条链"留着不赚钱"，所以今天谁买归谁，还附赠一个 60 天保证：万一苹果自己独立撞上并修了，保你重新交付。  
## 二、这条链怎么走  
  
技术路线是 WebKit 内部的一条类型混淆：**JSC DFG/FTL 类型混淆，单个 bug，触发确定性**  
。从渲染器 RCE 起步，经由用户态 XPC 服务逃逸，落到 mobile user context，再通过 TLS 通道反向连回攻击者自建的 C2 面板。  
  
卖家列出的"已击穿"清单挺唬人：arm64e 的 PAC、StructureID 随机化、Gigacage、IsoHeap、WebContent 沙箱 profile。**不需要内核 RAW 权限**  
，shell 是会话级的——标签页关了就死，靠重新感染延续，而不是留后门。  
  
设备的挑法也很讲究，**按 SoC 锁而不是按型号锁**  
：利用链要吃 PAC 鉴别器、kalloc zone 布局、JSC Gigacage 基址、IsoHeap 槽位几何。支持 iPhone 16 / 16 Pro / 16 Pro Max；同为 A18 的 16 Plus 未验证、不在保障范围；A18+C1 的 16e 直接不支持。  
  
版本覆盖面上，设备下限是 iOS 18.0（22A3351 / 22A3354），当前版本已验证 27.2 与 27.2.1，27.0.x 到 26.x 确认能触发 RCE 但逃逸环节要按版本重调（另收费）。**开了 Lockdown Mode（JIT 被禁）或 Private Relay 加自定义 DNS 的设备，这条链直接失效**  
——这是卖家自己写的限制，对受害者反而是道防线。  
## 三、卖家真正想卖的不是 shell  
  
交付物包含 shell 控制台、按设备划分的文件浏览器、上传下载、命令历史。但真正的重点是"钱包模块"，五件事：  
1. **容器横扫**  
——落地即遍历每个已装钱包的数据容器，拉走 vaults、keystore JSON、AsyncStorage/LevelDB、备份、SQLite、日志，按 bundle ID 自动识别，在目标关掉页面之前就导完  
  
1. **相册收割**  
——助记词截图，"加密设备上收益最高的一类产物"，自动打包  
  
1. **剪贴板监听**  
——抓复制走的地址和助记词，后台一键替换收款地址  
  
1. **深度链接劫持**  
——触发 walletconnect://  
、metamask://  
、trust://  
，把目标拽进攻击者控制的流程  
  
1. **会话窃取**  
——Safari 里 web 钱包与交易所会话的 cookies 和 localStorage，覆盖 MetaMask、Trust、imToken、Phantom、Coinbase Wallet、Rainbow、Zerion、Exodus、Ledger Live、OKX、Bitget  
  
![钱包模块能力说明](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6PTZjM2AO8Lg4WYWYQj8QFYQqicTj3xwosmzUDTbRpY05t2VjSDGmq3gibfHdWYPCpwSrLXXkbGP4ebQlwVaibutDUE12ABMvS9ro/640?from=appmsg "钱包模块能力说明")  
  
边界也写得很清楚：无内核 RAW、不越狱、不碰 MDM 和描述文件、除用户点击外无授权、**不碰 Keychain**  
、不取 Secure Enclave 和硬件钱包种子、不碰 iCloud 数据、不截其他 App 界面、无持久化。还有一条实战提示：容器读取要求设备处于解锁状态，Complete Protection 类文件必须解锁才能读——**所以要趁目标正在用手机时落地**  
。  
## 四、交易条件与验证玩法  
  
漏洞走托管（escrow）交易，币种 BTC，买家付手续费，交付二进制加面板源码。面板是 Node.js + Docker，跑在买家自己的 VPS、自己的域名、自己签的 TLS 上，无回调。里面带 UA / 机型 / iOS 过滤、跳转伪装、按活动统计、多设备 shell 视图、扫描结果、剪贴板日志和外传日志。  
  
验证方式是全流程直播：买家指定 build，对方视频通话里用 Signal 或 Telegram，全程录屏，先报系统版本、sysctl 硬件型号和 UDID，再现场触发、走容器列表、打开 vault 文件、演示剪贴板替换，之后买家自己带设备复现成功才放款。也可以先花 **1500 美元**  
买一次单独的验证运行。卖家还提到 iOS 27.3 beta 1（24C5037）正在适配中，承诺 60 天补丁保障。  
## 五、这笔交易该怎么看待  
  
值得警惕的是"完美受害者"叙事：卖家反复强调苹果从未见过这条链，**没 CVE、没白皮书、没公开 PoC**  
——这既是它值钱的理由，也是它无法被验证的理由。在暗网买的零日，买到的是对方的自述，链子是不是真的、打的是不是 A18、钱包模块是不是演示时临时拼的，全凭买家自己承担风险。  
  
链子本身写得像模像样：从 JSC 类型混淆到 XPC 逃逸到 mobile user context，PAC、StructureID、Gigacage、IsoHeap 这些词摆得像行话，而且"非 A18 硬件上 WebKit 干净退出、无崩溃无残留"这种描述，反而符合真做过 WebKit 研究的人的表达习惯。但正因如此，更需要冷静：现在没有任何一手证据能证明这条链存在，能确认的只有"有人在卖"。  
  
对读者来说，眼下能做的动作很朴素：**把 iOS 更新到最新，锁屏模式打开，别在主力机上放大量代币资产，看到加密钱包相关的引导链接别随手点**  
。另外值得对照的是同一时间线的两起事件——Google 威胁情报组在 3 月记录的 DarkSword 六漏洞链、CertiK 11 月警告其已转向加密钱包，以及 Confiant 披露的 MoldyRoost 恶意广告投放行动，用了半年时间把普通广告变成 iPhone exploit 的分发管道，专打钱包助记词。**iOS 加密钱包已经是 drive-by exploit 的主战场**  
，这条待售的 Safari 零日，不过是同一块蛋糕上最新切下来的一片。  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OOEqchOFc6JSgicIJRzoKfnFDnZVFauLeVFR5fJMRIJ0dfUrjgakMvlo5rLspnyW8Tica0x480TCFpRrGicfJ7P9YaXZ7OLpGzYU/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OY91hXs6mOcHiavATbh8AuLcGibnHH1ptNsVJvOUSib97PWMrfxrsBB5bxjKOOnSXZyaZzlribW8DKqaC2qNCF0XibMrJgGHib7dFz0/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6N7cP1ibhtck5RBAgiamf9CprclRkjgGLUs8NmxdaM0XEsXXVicnctjglZe0bxU3ibg5zDXjibNmBckvcsZGOoribQFTiaPWJfwiafxib6k/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6P5nNCZRVzkNedUXsdibibqoYqaTrH4He9HADfx8EN8Dj70BZsXeOw8za0WKRYic3tJiaLOOU3H4IIdLcJNAXMPPRbelnjaeyXusPY/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
