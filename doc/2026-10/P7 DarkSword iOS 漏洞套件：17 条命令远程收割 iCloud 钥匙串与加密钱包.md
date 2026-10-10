#  P7 DarkSword iOS 漏洞套件：17 条命令远程收割 iCloud 钥匙串与加密钱包  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-10 01:08  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NBwQSeFgcLcJTRDMK7wTjtuaueQD3c8LZR21ac4TrZqzF2w7kviaiaRCiaS51I1owjO7awI8IeJVvqcict6OlbmKKBSnk4ic79nGDQ/640?from=appmsg "")  
> **导语**  
：iOS 漏洞即服务（Exploit-as-a-Service）在野进化的最新证据 —— DarkSword 工具套件推出 P7 变种，17 条 C2 命令直接收割 iCloud 钥匙串、加密钱包与 Apple Notes，连 SpringBoard 进程都被注入作为长期驻留通道。  
  
## 一、DarkSword 是什么  
  
DarkSword 是一个针对 iPhone 的商业级漏洞利用套件，最早 2025 年 11 月在野被 Google Threat Intelligence Group（GTIG）、iVerify、Lookout 三方捕获，2026 年 3 月首次公开。  
  
完整攻击链分三层：  
1. **WebKit 沙箱逃逸**  
——通过浏览器漏洞突破 Web Content 沙箱  
  
1. **内核权限提升**  
——利用 Core Audio 等系统组件 0day 拿下 ring 0  
  
1. **SpringBoard 进程注入**  
——把植入体注入到 iOS 负责应用启动和主屏幕的 SpringBoard 进程，落地为长期驻留  
  
这套工具最早是闭源商业产品，原始供应商不明，但 2025 年底开始在二手市场流转，被多个有财务动机的攻击者采购。在野观察到的使用者包括土耳其商业监控供应商 PARS Defense（通过假 Snapchat 主题站点分发）、俄罗斯背景的 Star Blizzard（COLDRIVER，通过假邀请诱饵）、以及至少一个中文背景的攻击者群体（通过假 Apple ID 登录页分发）。  
  
受影响国家覆盖沙特阿拉伯、土耳其、马来西亚、乌克兰。  
  
![DarkSword 概念图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6OCnby0SDWbEgrh7ZINCGiafMtNIG9OBn3Dm5XyzzUocE6KM9JxQc4JZAsTNStiblMev0GCIjPeBhXKcjG0epj47WGtaiabEEc1E0/640?from=appmsg "DarkSword 概念图")  
## 二、P7 三大升级  
  
iVerify 在 2026 年 10 月 9 日发布的最新报告里，把 DarkSword 的最新变种命名为 P7（命名来自攻击者在修改原代码时用的 p7_  
 变量前缀）。相比之前观察到的版本，三项核心升级：  
- **缩小设备端 footprint**  
——消除 HTTP 调试日志和 syslog 输出，植入体在受害设备上更安静  
  
- **新增 keychain 与加密钱包窃取**  
——之前版本只是把 keychain 数据库拷贝出来到攻击者侧处理；P7 直接在设备上把数据转成 JSON 再外传，效率更高、链路更隐蔽  
  
- **双向 C2 通信**  
——植入体 15 秒轮询一次 C2，支持命令拉取和心跳回传  
  
新增的「用浏览器 localStorage 防重复利用」机制也值得注意——同一台设备被二次感染的概率显著降低，避免操作员重复浪费漏洞。  
  
![DarkSword 攻击链示意图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NTlf3M53MpzGE8zxu7XEUhwC2rxQJINwsTXvHE6CkTwJ2LamoyRfGibgmXQGtMkNYC24CiaLKcuiaibv9UndwRrlx4vtDteia608O4/640?from=appmsg "DarkSword 攻击链示意图")  
## 三、17 条 C2 命令清单  
  
P7 植入体支持的具体命令从 ls  
 到 wallet_extract  
 一共 17 条，覆盖了「侦察 - 提取 - 控制」全链路：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">命令</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">功能</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">execute_command</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">跑系统命令：ls / dir / cat / mkdir / rm / echo / ps / memdump / ipconfig / netstat / whoami</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">ls</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">列目录</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">download</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">读文件后上传 C2</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">photos</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">上传 </span><code><span leaf="">/var/mobile/Media/DCIM</span></code><span leaf=""> 全部照片</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">apps</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">枚举应用沙箱，提取 bundle ID</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exec</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">在植入体运行时内执行任意 JavaScript</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">file_upload</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">递归扫描路径并上传匹配文件</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">basic_info</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">上传设备元数据</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">disk_scan</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">从 </span><code><span leaf="">/</span></code><span leaf=""> 递归扫文件系统，记录文件/目录/符号链接元数据</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">ios_app_data</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">找指定 bundle ID 的应用沙箱和应用组容器，上传选定文件</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">wallet_scan</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">扫已安装的钱包 App</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">wallet_extract</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">抽 imToken 钱包数据</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">memo_scan</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">上传 Apple Notes 数据库</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">photo_scan</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">上传 Apple Photos 数据</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">sleep</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">改信标轮询间隔</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exit</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">终止信标循环</span></section></td></tr></tbody></table>  
最值得说的是 wallet_extract  
 —— 攻击者已经把目标锁定到 imToken 这类中文用户常用的加密钱包，不是泛泛而谈的「加密资产窃取」。  
## 四、配套 Coruna 在野  
  
Censys 同期披露另一个 iOS 漏洞套件 Coruna，定位是 DarkSword 的「companion payload kit」：  
- 目标 iOS 13.0 - 17.2.1（比 DarkSword 覆盖范围更老）  
  
- 攻击阶段在浏览器会话内执行（与 DarkSword 的系统级注入不同）  
  
- 钱包收割模块：加密货币恢复助记词、余额、keystore 数据  
  
DarkSword + Coruna 通过「DS-Fusion v1.0」合并包一起分发，攻击者用一套基础设施同时打两套目标。  
## 五、5 个开放主机暴露的数据规模  
  
Censys 找到 5 个开放目录主机直接暴露了 DarkSword 组件，托管基础设施运营者的实际作业痕迹：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">IP</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">角色</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">关键证据</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">43.134.165.205</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">DS-Fusion v1.0（DarkSword + Coruna 合并包）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">整套商业级套件在线分发</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">166.88.95.90</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">C2 服务器</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9-6 有 2 个中国 iOS 设备每 3 秒轮询信标页数小时</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">23.148.212.237</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">iOS 26 漏洞链开发工作区</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">含 CVE-2026-31001 等新 0day 开发痕迹</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">47.102.192.23</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Coruna 暂存主机</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">—</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">156.239.230.120</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">完整 C2 平台</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9-15 观察轮询设备，admin 面板暴露 agent/reseller 模型</span></section></td></tr></tbody></table>  
从生产服务器复原出来的受害者数据：**11 个加密钱包恢复助记词 + 179 个设备 loot 目录 + 75 个账户的控制面板名单**  
。Censys 研究员 Aidan Holland 直接定性：「The platform runs a Chinese-speaking exploitation-as-a-service operation（这是一个中文背景的漏洞即服务平台）」。  
## 六、两个新披露的 CVE  
  
Censys 在生产服务器的 exploit registry 中找到两个此前未公开的 CVE 编号，证实 DarkSword 仍在用真实未公开漏洞：  
- **CVE-2025-24201**  
——WebKit 引擎越界写漏洞，可突破 Web Content 沙箱。iOS 18.3.2 / iPadOS 18.3.2 已修  
  
- **CVE-2025-31200**  
——Core Audio 框架内存损坏漏洞，处理恶意构造的音频流时可执行代码。iOS 18.4.1 / iPadOS 18.4.1 已修  
  
如果你的 iPhone 还停在 iOS 18.3.1 / 18.4.0 区间，这两个漏洞就是真实威胁。  
## 七、iOS 26.x 升级失败与 LLM 辅助  
  
iVerify 2026 年 9 月报告里提到，DarkSword 公开后短短几周内，业内观察到「多次显然由 LLM 辅助的尝试」想把工具迁移到 iOS 26.x，但都失败了。失败变种的特点：  
- 专注稳定性、隐蔽性、偷数据质量  
  
- 放弃了部分激进功能换取存活率  
  
- 由开源 LLM 直接拼凑漏洞链，缺少对 iOS 内核机制的理解  
  
这给业界提了个醒：漏洞套件一旦泄露，攻击者会大量尝试用 AI 工具改造，防御方必须假设「下周就有 iOS 26.x 的 DarkSword 变种出现」。  
## 八、红队四点观察  
1. **闭源工具一旦泄露就再也收不回**  
——DarkSword 从 2025-11 首次在野到现在不过 11 个月，已经迭代出至少 4 个变种  
  
1. **DS-Fusion 合并包模式是趋势**  
——把两个套件打包成单一商品卖，攻击者采购成本进一步降低  
  
1. **生产服务器开目录是低级失误**  
——攻击者团队技术不一定强，运营管理才是真正的 P0 漏洞  
  
1. **加密钱包是新战场**  
——imToken、BitKeep 已经在野被收割，币圈用户的安全意识要跟上来  
  
## 九、思考题  
1. 如果 DarkSword 团队真的成功升级到 iOS 26.x，Censys 提到的那些 iOS 18.3.2 / 18.4.1 补丁会失效吗？为什么？  
  
1. P7 用浏览器 localStorage 防重复利用，这对受害用户的检测有什么正面/负面影响？  
  
1. 中文背景「漏洞即服务」平台曝光后，国内加密钱包厂商（imToken、BitKeep）应该立刻做什么？  
  
## 素材出处  
- Ravie Lakshmanan / The Hacker News《P7 DarkSword iOS Exploit Kit Adds Crypto Wallet Data Theft and Remote Commands》2026-10-09  
  
- iVerify《P7 DarkSword iOS Exploit Kit》报告 2026-10-09  
  
- Censys《DarkSword and Coruna open directory cluster》2026-10-09  
  
- Apple Security Updates（iOS 18.3.2 / 18.4.1 修复 CVE-2025-24201 / CVE-2025-31200）  
  
- Google Threat Intelligence Group / iVerify / Lookout《DarkSword 首次公开报告》2026-03  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OjsibwC3agVUVfoK1bib3raDEa4gspKCvXxD77wEBeAOIfcDlh83PYCzQdxUbNlekHvPmdEFx1B1ic4pGNemzibJHiblOas7xVRGm4/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
