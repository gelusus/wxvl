#  美国国土安全部人员数据遭窃取，CBP 与 ICE 大量特工信息泄露；CNNVD周报：新增3806个安全漏洞，PHP高危漏洞可导致远程代码执行| 牛览  
 安全牛   2026-10-29 04:35  
  
**点击蓝字 关注我们**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/wKeDC5RjIzHzEPC3lQlYJ4iajvfC7icvBCHaxmma0rhTcagPEL3cQXgepv3qicV7yXXsQ7RLe2W3ymESlXpbDppOKM78HvibbM0fjCTEW6oB56c/640?wx_fmt=png&from=appmsg "")  
  
  
新闻速览  
  
  
- CNNVD周报：新增3806个安全漏洞，PHP高危漏洞可导致远程代码执行  
  
  
- CNCERT发布网络安全信息与动态周报  
  
  
- Firefox 159 正式版发布：侧边栏密码管理上线，新增 AI 网页监控能力  
  
  
- 超半数互联网流量来自 AI 代理，Socure 推出 Agent Intelligence 补齐风险识别短板  
  
  
- 英国预算提案拟磋商软件厂商网络安全事故责任，或将告别自愿安全准则时代  
  
  
- 美国国土安全部人员数据遭窃取，CBP 与 ICE 大量特工信息泄露  
  
  
- Bitdefender 与 Netgear 联合报告：黑客利用 IoT 设备监听、横向移动并转发攻击流量  
  
  
- Google 推出 Connected Apps Platform，向第三方开放 Gemini 与 Search 接入能力  
  
  
- Intelbras 多款 IP 摄像头曝未授权 DoS 漏洞，本地网络即可让设备离线  
  
  
- 代号 Omegaline 行动：FBI 捣毁防弹托管服务商 Virtualine，但其具备快速换壳重启能力  
  
  
  
  
特别关注  
  
  
**CNNVD周报：新增3806个安全漏洞，PHP高危漏洞可导致远程代码执行**  
  
国家信息安全漏洞库（CNNVD）发布2026年第42期信息安全漏洞周报。数据显示，10月19日至25日期间，CNNVD新增收录安全漏洞3806个，其中超危漏洞491个、高危漏洞1357个、中危漏洞1629个、低危漏洞329个，整体修复率为88.18%。  
  
  
从厂商分布来看，Oracle新增漏洞1078个，占比28.32%，位居首位。漏洞类型中，授权问题最多，共422个，占比11.09%，其次为缓冲区错误和资源管理错误。  
  
  
本期重点通报PHP两项资源管理错误漏洞（CVE-2026-109618、CVE-2026-109619）。前者涉及longjmp()内存处理错误，后者源于请求结束后全局状态未正确重置。未经认证的攻击者可通过恶意HTTP请求造成内存泄露，进而远程执行代码。受影响版本涉及PHP 8.2、8.3、8.4及8.5多个分支。  
  
  
此外，Google Chrome存在释放后重用漏洞，攻击者可通过特制HTML页面在沙箱外执行任意代码。AI领域，Amazon sagemaker-python-sdk漏洞可能导致IAM凭据泄露及代码执行，SGLang和vLLM漏洞则可能引发内存耗尽和服务中断。  
  
  
目前，相关厂商均已发布修复版本。建议企业及时排查受影响资产，优先修复可导致远程代码执行及服务中断的漏洞。  
  
  
原文链接：  
  
https://www.cnnvd.org.cn/group1/M00/03/3A/rBBl8Wrha8uAEUznAAXAFzDjKTU006.pdf  
  
  
**CNCERT发布网络安全信息与动态周报**  
  
10月28日，国家计算机网络应急技术处理协调中心（CNCERT）发布2026年第43期《网络安全信息与动态周报》，披露10月19日至25日期间我国互联网网络安全态势及事件处置情况。  
  
  
报告显示，本周我国互联网网络安全态势整体评价为“良”，互联网基础设施运行总体平稳，未发生影响全国或省级行政区域的重大基础设施运行安全事件。  
  
  
从威胁类型来看，软件高危漏洞、恶意代码传播及网站攻击仍是政府、企业和互联网用户面临的主要安全风险。其中，高危漏洞可能被攻击者用于入侵系统，恶意代码传播可能造成设备感染和数据泄露，网站攻击则可能影响业务连续性。  
  
  
事件处置方面，CNCERT本周共协调处理网络安全事件680起，其中跨境网络安全事件218起。处置工作涉及云服务商、域名注册服务机构、应用商店、各省分中心及国际合作组织。  
  
  
值得关注的是，CNCERT协调境内外域名注册机构、境外CERT等机构，重点处理仿冒投诉事件467起，反映出网络仿冒仍是安全应急处置的重要关注领域。  
  
  
整体来看，我国互联网基础设施运行保持稳定，但网络仿冒、软件漏洞及恶意代码等安全威胁仍需持续防范。  
  
  
原文链接：  
  
https://mp.weixin.qq.com/s/Ijfuemn6nVa0oQgpWki29A  
  
  
  
热点观察  
  
  
**Intelbras 多款 IP 摄像头曝未授权 DoS 漏洞，本地网络即可让设备离线**  
  
卡巴斯基 GReAT 研究团队披露 Intelbras 多款 IP 摄像头存在拒绝服务漏洞，候选编号 CAN-2026-2035970，CVSS4.0 评分为 5.1，影响 iM3BC、iMX C、iM4 C 等多款机型，配套管理应用 Mibo Smart 下载量超 400 万，用户集中在拉美地区。  
  
  
漏洞根源为固件升级接-/firmwareUpgrade_service/Demo缺少身份校验，处于同一局域网内的攻击者无需账号密码，发送恶意 HTTP POST 请求即可触发固件升级重启流程，单次请求会造成设备约一分钟离线，循环发送请求能够使摄像头长期断连，监控画面与设备管理功能完全失效，无需提供合法固件文件。  
  
  
厂商收到漏洞通报后推出修复固件 R.260804，新版本对该接口增加会话与 Cookie 校验。目前暂无完整 CVE 编号，官方没有可用临时缓解方案，建议用户尽快升级固件；未完成更新前，应限制该管理接口的网络访问权限。  
  
  
原文链接：  
  
https://securelist.com/intelbras-camera-vulnerability/121131/  
  
  
**Google 推出 Connected Apps Platform，向第三方开放 Gemini 与 Search 接入能力**  
  
Google 正式发布 Connected Apps Platform 开发者平台，面向 AI 代理时代，允许第三方服务接入 Gemini 与 Google Search，用户无需跳转独立 App，直接在 AI 交互流程内完成业务操作hongkongse。  
  
  
该平台当前仅限美国开发者以候补名单模式参与 Beta 测试。开发者依托 AppFunctions APIs 对接云端 MCP 服务器或者 Android MCP，通过 Google Connected Console 完成应用注册、测试与上线，可选择将服务部署在 Gemini、Search，或者双端同时启用，还可监控流量、延迟、错误率等运行指标。  
  
  
目前 Gemini 已集成超过 50 款第三方应用，包含 Adobe、Peloton 等产品；Search 端已有 Instacart、Canva、YouTube Music 上线，其余应用将在未来数周陆续迁移至搜索侧。  
  
  
该模式推动使用习惯从独立 App 切换至 AI 代理交互，也带来新安全挑战，涉及第三方身份授权、数据流转、权限管控，和此前 ChatGPT、Muse 等产品的应用扩展机制形成竞争。平台暂未公布全球大规模上线时间表。  
  
  
原文链接：  
  
https://techcrunch.com/2026/10/28/as-ai-agents-reshape-the-web-google-opens-gemini-and-search-to-more-apps/  
  
  
  
安全事件  
  
  
**美国国土安全部人员数据遭窃取，CBP 与 ICE 大量特工信息泄露**  
  
据 TechCrunch 报道，黑客窃取了美国国土安全部（DHS）数千名雇员人事数据，事件波及美国海关和边境保护局（CBP）、美国移民与海关执法局（ICE）特工人员信息。  
  
  
泄露数据集规模超过 4 万份，内容包含雇员姓名、职务、联系电话、电子邮箱、工作地点以及任务分配信息，同时覆盖正式雇员与外包合同人员。黑客在随泄露文件附带的说明中表述了自身行动立场，媒体向黑客发起置评请求，但并未得到回应。  
  
  
本次泄露数据由 DDoSecrets 对外发布。事件发生后，Okta 公开表示没有证据表明自身系统遭到入侵。国土安全部、CBP、ICE 均未对外回应媒体问询，尚未确认泄露数据完整真实性，也未披露泄露入口、攻击路径等技术细节。  
  
  
该事件属于美国联邦执法机构人员信息外泄案例，执法人员个人敏感信息外泄，将给相关人员带来身份暴露、社会工程学攻击等安全风险。截至报道发布，官方暂未发布事件调查进展公告。  
  
  
原文链接：  
  
https://techcrunch.com/2026/10/28/hacker-stole-personnel-data-about-thousands-of-homeland-security-employees-including-cbp-and-ice-agents/  
  
  
**代号 Omegaline 行动：FBI 捣毁防弹托管服务商 Virtualine，但其具备快速换壳重启能力**  
  
近日，在美国代号为 Omegaline 的联合执法行动中，FBI 主导查封防弹托管服务商 Virtualine 的virtualine.net域名，德国 Polizei Hessen、荷兰 Politie、英国 NCA 及 OverRIPE、Lumen 等政企机构协同参与调查。  
  
  
防弹托管（bulletproof hosting）指刻意无视执法处置请求，为网络犯罪提供服务器、域名等基础设施的服务商Cyber.gov。Virtualine 拥有 Secury、Omegatech 等多个别名，在黑客论坛招揽客户，提供可承载钓鱼、盗卡、垃圾邮件等黑产活动的服务器服务，还推出额外保护方案，承诺收到执法函后将犯罪业务迁移至无关空壳公司继续运行。尽管对外宣称禁止儿童色情内容，该平台仍明知故犯托管传播未成年性剥削素材的犯罪论坛，即便多国警方与儿童保护机构多次预警，仍维持站点运营，还协助犯罪者搭建新上传售卖站点。  
  
  
该平台采用无反洗钱校验的比特币地址收款，Chainalysis 区块链溯源显示其资金链路与勒索软件、信息窃取恶意程序、网络犯罪论坛高度关联。  
  
  
域名被查封后，运营方迅速在virtualine.org发布公开声明否认指控，并在 Telegram 告知客户数据安全，坦言可在一日内完成品牌重塑重建服务。安全厂商 Silent Push 提示，此类防弹托管可在 25 个工作日获取新 ASN 自治系统编号，迁移 IP 网段实现死灰复燃，Omegatech 部分网络当前仍可访问。  
  
  
原文链接：  
  
https://www.helpnetsecurity.com/2026/10/28/fbi-seizure-virtualine-bulletproof-hosting-domain/  
  
  
  
安全攻防  
  
  
**Bitdefender 与 Netgear 联合报告：黑客利用 IoT 设备监听、横向移动并转发攻击流量**  
  
网络安全公司Bitdefender与网络设备厂商NETGEAR联合发布《2026年IoT安全态势报告》，指出攻击者正利用存在漏洞的物联网（IoT）设备实施监控、网络横向移动及流量代理攻击，家庭网络正成为攻击企业的新跳板。  
  
  
报告基于2026年1月至9月全球约1040万户智能家庭的遥测数据，分析了54亿次安全事件及12亿次漏洞观测记录。智能电视、智能插座和网络附加存储（NAS）设备分别占漏洞观测记录的22.7%、22.4%和22%，合计达67.1%。  
  
  
研究发现，99.97%的设备漏洞观测记录涉及已公开的CVE漏洞。约54%的漏洞可能引发溢出错误，拒绝服务和内存损坏分别占约18%和17%，反映出设备固件在内存安全及输入验证方面存在不足。  
  
  
值得关注的是，攻击者可利用智能电视麦克风进行窃听，或以受控设备为跳板入侵其他网络资产，还可能将设备变为住宅代理，通过家庭IP地址转发恶意流量，隐藏攻击来源。  
  
  
研究人员警告，共享固件组件、连接库及底层硬件平台可能导致跨厂商漏洞集中暴露。建议企业加强IoT资产盘点、固件更新及持续监测，重点排查长期在线且缺乏安全维护的设备。  
  
  
原文链接：  
  
https://www.cybersecuritydive.com/news/internet-of-things-vulnerabilities-proxies-netgear-bitdefender/873033/  
  
  
  
产业动态  
  
  
**英国预算提案拟磋商软件厂商网络安全事故责任，或将告别自愿安全准则时代**  
  
英国在财政预算文件中披露，将于明年启动咨询，探讨因产品安全缺陷引发网络安全事件时软件厂商的责任划分，这是该国首次正式研究网络泄露的责任归属，将依托英格兰及威尔士法律委员会正在开展的产品法修订工作推进。此前英国长期依靠自愿性行业规范约束软件安全，捷豹路虎遭网络攻击事件给本国造成 19 亿英镑经济损失，推动政策转向。  
  
  
政策方向包含扩展《Product Security and Telecommunications Infrastructure (PSTI) Act》适用范围，新规将力求与欧盟 Cyber Resilience Act 兼容，避免企业承受多重合规成本。该提案或将遭遇商业软件联盟等行业团体的阻力，企业担忧严苛追责会抑制创新。  
  
  
本次预算同步投入 1.54 亿英镑强化网络防御。其中 7000 万英镑用于 NCSC 的 NERV 中心，依托 AI 识别关键基础设施漏洞；6000 万英镑投向 Cyber Shield 项目，借助智能 AI 代理发现、修复安全弱点；4000 万英镑设立国家级智能代理安全部门，专门处置恶意 AI 代理带来的威胁。  
  
  
原文链接：  
  
https://therecord.media/uk-budget-proposal-software-companies-liable-cyber-breaches  
  
  
  
新品发布  
  
  
**Firefox 159 正式版发布：侧边栏密码管理上线，新增 AI 网页监控能力**  
  
Mozilla 基金会于 2026 年 10 月 28 日正式发布 Firefox159 稳定版，新版本将已保存账号凭证直接集成至浏览器默认侧边栏，用户浏览网页时可直接查看密码，无需手动挂载组件，减少操作打断。  
  
  
本次更新调整了快捷键体系，Windows、Linux 平台使用 Ctrl+Shift+A 检索已打开标签页，macOS 对应 Command+Shift+A；扩展与主题管理器快捷键变更为 Ctrl+Shift+F（macOS 为 Command+Shift+E），全部快捷键均可在 about:keyboard 页面自定义配置。  
  
  
针对 Linux 用户，版本修复 Wayland 会话两项标签交互缺陷，解决点击标签意外分离、拖拽标签无法正常拆分为独立窗口的故障，同时修复 AMD 显卡渲染 Mapbox 元素时出现的内存泄漏问题，提升平台稳定性。  
  
  
在 AI 实验性功能方面，Smart Window 新增/watch指令，支持设置轮询间隔监控网页动态变化。网页内容发生变动时，浏览器将推送桌面通知并点亮工具栏标记，可用于跟踪库存变动、价格波动等场景。  
  
  
用户可通过浏览器内置关于对话框完成升级，运维人员可从官方站点获取独立安装包。  
  
  
原文链接：  
  
https://securityonline.info/mozilla-firefox-159/  
  
  
**超半数互联网流量来自 AI 代理，Socure 推出 Agent Intelligence 补齐风险识别短板**  
  
身份风险情报厂商 Socure 数据显示，截至 2026 年 9 月，AI 代理每日请求量同比暴涨 1700%，互联网半数流量已不再来自人类用户。Accenture 调研显示，85% 消费者愿意使用 AI 代理协助线上业务，74% 允许 AI 代为执行特定任务。  
  
  
当前传统反欺诈系统仅能识别自动化行为，无法区分 AI 代理背后是合法用户还是欺诈者。为此 Socure 推出 Agent Intelligence 套件，以身份图谱为基础，分三层解决 AI 代理的身份核验难题。  
  
  
Agent Detection 现已正式可用，不采用简单拦截判定，输出置信分数与原因编码，实验室环境下识别 Muse、Instinct 等消费级代理准确率达 99.99%。Agent TrustScore 处于测试阶段，从身份、授权、风险三个维度综合评估代理可信度，企业可按自身风险偏好加权。Agent Credential 预计 2027 年第一季度上线，属于 W3C 可验证凭证，可将代理与实名主体绑定，支持跨站点离线校验与委托链路密码验证。  
  
  
该产品支持异构代理与凭证来源，厂商称其可在同一身份图谱完成检测、评分、凭证签发全流程，企业需结合自身流量实测验证相关能力。  
  
  
原文链接：  
  
https://www.expresscomputer.in/news/half-the-internet-is-no-longer-human-does-your-fraud-stack-know-which-half-it-is-talking/140564/  
  
  
  
  
  
  
**联系我们**  
  
合作电话：18610811242  
  
合作微信：aqniu001  
  
联系邮箱：bd@aqniu.com  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wKeDC5RjIzHLaRNDqXv8EBKmAlUNc963SKtboonO5zqaKpOeh7Pub6LsDJQ7llRskjdrNfmEkz7dFZib0rbBjIIgJoKTrzOa77KR24Pf1rl4/640?wx_fmt=gif&from=appmsg "")  
  
  
