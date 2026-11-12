#  苹果 iOS 曝沙盒逃逸漏洞，币安发出预警，恶意APP可读取其他 App 数据  
安全透视镜
                    安全透视镜  网络安全透视镜   2026-11-12 11:48  
  
"  
  
无需越狱、无需内核漏洞，只要申请到一个普通共享容器，  
   
iOS 沙盒隔离墙  
   
就会被撕开一个通往全机应用私有目录的缺口。  
  
  
移动端漏洞的常态是内核利用链：需要越狱、需要漏洞稳定性、需要提权。这些门槛让绝大多数用户与主流攻击者绝缘。但 11 月 8 日由研究团队公开的代号   
worse_query  
   
的概念验证代码，打破了这一惯例。发布者自称是此前   
bad_query  
   
的直接继任者，针对 iOS 26 与 iOS 27，不需要越狱，也不需要任何内核漏洞，仅依赖苹果自带的共享容器机制，便实现了跨沙盒的本地文件读取。  
  
概念验证公布次日，社区即出现基于该方案构建的免越狱文件管理器，可直接浏览、导出乃至修改其他 App 的私有目录。11 月 10 日，  
币安公开发布安全预警，  
   
助记词、私钥、登录凭证  
   
被列为潜在暴露对象，同日慢雾科技首席信息安全官 23pds 发文澄清其技术定性与真实触发条件。  
  
📌 本文看点  
  
01  
  
机制：App Group 校验缺口  
  
02  
  
边界：钱包资产暴露条件  
  
03  
  
处置：终端与开发侧加固  
  
01  
VULNERABILITY REPLAY  
### 机制复盘：共享容器如何成为穿透支点  
  
  
发布者 forcequit 在社交平台披露的内容显示，该方案面向   
iOS 26 与 iOS 27  
   
，并与研究者 @0xjohnny 合作完成。其实现思路并未寻找内核层的提权路径，而是回到苹果为应用共享数据而提供的   
App Group  
   
机制本身——在路径校验环节寻找可被绕过的判断逻辑。研究者同时在说明中强调，这一方案与越狱无关，不能用于提权，也不会成为越狱工具链的一部分。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TiaL5jqa5zrIGicm2J7nzukhl3QZuKq3dwCt9vD5RLcW3lXbicSanQ8LqRTbDZnHMlpxNOwDItaWXv8sgiantnUEibz9AYO7hSkbarL90Ex9I6MM/640?wx_fmt=png "")  
  
— 图 1：forcequit 在社交平台公布 worse_query 项目，指向 GitHub 上的概念验证仓库  
  
从攻击链的角度看，整条路径的门槛相当低。应用只需在自身 Info.plist 中声明一个   
App Group 共享标识  
   
，即可在沙盒扩展中取得容器访问令牌，随后读取本不属于自身的数据域。  
  
.  
.  
.  
attack-path  
  
[第三方恶意 IPA] 声明 App Group 共享标识  
  
　　↓ 申请沙盒访问令牌（wq_grant）  
  
[沙盒校验缺口] 越出自身数据容器边界  
  
　　↓ 遍历 /var/mobile/Containers/Data/Application/*  
  
[跨应用读取] 目标 App 的 Documents / Library / Caches  
  
研究者给出的受影响范围为   
iOS 26.0 至 26.7.2 与 iOS 27.x  
   
。在 iOS 27 环境下，方案还能列出部分系统级目录，包括   
SystemGroup  
   
与描述文件相关目录。社区开发者随后基于该代码推出的   
terminal_worse  
   
项目，在 README 中完整列出了可访问的路径范围。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TiaL5jqa5zrIZeM274hw1npibxry0V0MoFEUibREQ2KdKTCLIEibfm0bNuBwylicK2uS7Bn2ibxo9A3cLk2ecGuDXHQc8leOmo0vwHC5zOdUibIr9c/640?wx_fmt=png "")  
  
— 图 2：terminal_worse 项目说明中列出的可触达路径，覆盖全部应用数据容器与共享容器  
  
使用门槛低到什么程度？普通用户在借助 AltStore、SideStore 等侧载工具进行个人签名安装时，签名工具会   
自动为应用补齐所需的 App Group 容器  
   
，利用环境自动成立。基于该方案构建的图形化工具   
WQ Files  
   
进一步提供了完整的文件浏览、导出、修改与清除界面。11 月 11 日，命令行版本 terminal_worse 在社交平台走红数小时后删库，页面转为 404。  
  
还需要注意一个容易被忽略的背景：App Group 本身是苹果的正当功能，小组件与扩展机制均依赖它实现数据共享，应用商店内的应用同样可以合法使用。因此这一漏洞   
并不能等同于“所有第三方商店应用都不安全”  
   
，其风险边界仍需结合具体授权与商店审核机制来判断。  
  
  
02  
IMPACT ASSESSMENT  
### 威胁边界：钱包私钥暴露的真实条件  
  
  
11 月 10 日，币安在社交平台发布预警，列出的受影响版本为   
iOS 26.7.1、27.0.1 与 27.2  
   
，并明确该问题属于 iOS 系统层面，生态内的助记词、私钥与登录凭据均面临风险，建议用户删除来源不可信的应用、升级系统与全部应用，若怀疑已被入侵则转移资产并重置登录会话。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TiaL5jqa5zrJpiaWOicCciaDgv6KOaG08DwSQInSYibyrOSZAbGje2C9veG0T0qGjicfWd5G1MUh0vCQMFRxiboRQsLGcsl40gibAlRJsQgltTJ3lWg/640?wx_fmt=png "")  
  
— 图 3：币安官方安全预警原文，包含受影响版本与三条处置建议  
  
同日，慢雾科技首席信息安全官 23pds 发文给出更精确的判断：该问题属于   
容器沙盒逃逸，并非越狱  
   
，漏洞本身不会自动窃取资产。真正的触发需要两个条件同时成立——设备上安装了来路不明的 IPA 文件，且目标钱包将助记词、keystore 或 WebView 缓存以明文形式存放在本地。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TiaL5jqa5zrK3s7M2gkOhxdwaqz2cRFQNQRkcx1QEhMZ0EVA2lH2p7uypHTLS3I9ibJI4XBf3gibemYiaUowhfaBDaXcQOzG5rIHU2aqQNduQks/640?wx_fmt=png "")  
  
— 图 4：慢雾 23pds 公开解读文本，明确“并非越狱”与两项触发前提  
  
依据这套判定逻辑，实际风险的分布并不均匀。下表按三个维度对高危暴露情境与相对安全的防御情境做了对照。  
  
<table><thead><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">风险维度</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">高危暴露情境</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">相对安全情境</span></section></th></tr></thead><tbody><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">应用来源</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(153, 27, 27);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">手动侧载未知 IPA、第三方助手签名安装</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">仅通过 App Store 官方渠道安装</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);font-weight: 700;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">密钥存储</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(153, 27, 27);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">明文存放于 Documents、SQLite 或 WebView 缓存</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">由系统 Keychain 或 Secure Enclave 保护</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);font-weight: 700;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">钱包形态</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(153, 27, 27);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">纯软件热钱包，本地常驻未加密凭据</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 12px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">硬件钱包、冷存储、多签方案</span></section></td></tr></tbody></table>  
  
国际媒体对此事的跟进相对克制。CoinDesk 在当日的报道中引用币安表态称，该问题   
影响的是整个 iOS 生态，而不限于币安应用本身  
   
；报道同时提到，截至发稿，交易所与研究团队均未报告任何利用该漏洞实施的攻击或盗窃事件。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TiaL5jqa5zrLjVvgmmITT2mN6kqkM43qwIaE83bFXxETcrpPGia2LEz84Xory1vl73fLiaHV7NXtNAjShZo8o3iahiaYF1uDHa6nnwO9nEIt6u3c/640?wx_fmt=png "")  
  
— 图 5：CoinDesk 报道原文，指出问题影响 iOS 生态整体，且暂无实际盗取案例  
  
  
03  
MITIGATION  
### 官方响应与处置：终端与开发侧动作  
  
  
时间线上有一处值得注意的错位。苹果于 11 月 9 日推送了   
iOS 27.2.1 与 iOS 26.7.3  
   
更新，安全公告列出信息组件、无线配对、蓝牙、AirPlay、App Intents 与 brotli 等共 9 项漏洞，但更新说明中   
并未出现 worse_query 或沙盒逃逸相关条目  
   
。慢雾方面判断升级后可以阻止该问题，属第三方推断；亦有开发者在新版 README 中标注“基本可以确定 27.2.1 仍受影响”。这一分歧尚无官方结论。  
  
9 项  
  
27.2.1 公告修补项  
  
3 个  
  
币安点名受影响版本  
  
0 起  
  
已确认资产损失案例  
  
综合各方口径与实测结论，终端用户与客户端开发团队可执行以下处置动作。  
  
1  
**清理未知侧载应用**  
排查并卸载所有经非官方渠道安装的手动签名应用，重点删除来源不明的文件管理器、调试工具与自动化插件。  
  
2  
**跟进系统与客户端更新**  
升级至机型对应的最新正式版本（iPhone 11 及以后对应 27.2.1，26.x 分支对应 26.7.3，XS、XS Max、XR 及七代 iPad 对应 18.7.12），并同步更新全部应用。  
  
3  
**高风险设备上的资产迁移与会话吊销**  
若设备曾长期使用侧载工具且运行过存放高价值资产的钱包，建议在干净设备上以全新助记词重建钱包并转入资产，同时重置全部活跃登录会话。  
  
4  
**开发侧：将敏感凭据迁出本地明文存储**  
移动端开发团队应停止把认证 Token、会话凭据与密钥材料以明文写入应用沙盒，全面迁移至系统钥匙串与加密硬件。  
  
∞  
EPILOGUE  
### 结语：沙盒隔离不是不可绕过的假设  
  
  
移动终端安全始终处于动态对抗之中。封闭生态与沙盒机制长期承担了绝大部分防御职责，但本次事件表明，  
   
信任不应完全建立在单一边界之上  
   
。当系统级隔离的校验逻辑出现缺口时，最先被暴露的往往是那些主动打破分发纪律的用户——他们为便利性支付了权限成本，而这一成本在此刻被一次性兑现。  
  
对普通用户而言，克制侧载、对来历不明的安装包保持警觉，仍是目前最有效的自保手段；对开发团队而言，把密钥交给系统钥匙串而不是自己的沙盒，是同样应当被写进基线要求的一条准则。至于苹果何时在公告中正式确认这一漏洞的修补状态，目前尚无定论，后续进展仍需持续跟踪。  
  
END  
  
我是 网络安全透视镜  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
