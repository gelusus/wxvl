#  ArgusAI自主 SRC 挖洞Agent 多智能体协同代理抓包 + 浏览器自动化请求重放，提升高危漏洞产出效率  
HG-ha
                    HG-ha  渗透安全HackTwo   2026-11-02 16:00  
  
0x01 工具介绍  
  
ArgusAI 是达瓦科技开源的 Agent 原生安全工作台，主打 SRC 漏洞挖掘与持续渗透攻防。依托自主 Agent 能力，可将授权目标交给工具自动规划审计流程，内置 MITM 流量代理、浏览器接管、HAR 导入、请求重放与逆向分析能力，支持数十小时长时任务、定时巡检与断点续跑。可联动 Nuclei 等外部扫描器协同检测，自动甄别疑点、验证漏洞、归档全套可复现证据，导出规范报告，一站式完成从资产审计、漏洞验证到材料整理，大幅减少人工重复操作，提升 SRC 挖洞产出效率。  
  
  
  
  
注意：  
现在只对常读和星标的公众号才展示大图推送，建议大家把  
**渗透安全HackTwo**  
"**设为****星标⭐️**  
"  
**否****则可能就看不到了啦！**  
  
**下载地址在末尾 #渗透安全HackTwo**  
  
0x02   
功能介绍  
  
✨核心特点  
> 项目面向企业安全团队、应用安全工程师、安全研究人员和明确授权的测试人员。底层 Agent 运行时也支持文件处理、日常信息整理和桌面协作，当前内置工具与工作流重点覆盖安全分析和逆向研究。  
  
  
**桌面 Agent**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXG8w5B2yDmV8KgybrtACLaMBQYZhlZUXu1nztubD0f1nlEwfe0xgDxxkR8hKShD3SV1kye9rFX6VKrticjMWc3nxeJ4aRoHQlo/640?wx_fmt=png&from=appmsg "")  
**命令行**  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWia9oKItdjVRmbz3HgamH1fJFJzQ7TvAzvjHf9ECtCLO1srUjnEaqsIvxm69Mu4ia2YWYl6U5mVVLjEPich6dkuyn8dTStNsjicn8/640?wx_fmt=png&from=appmsg "")  
**沙盒预览**  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVeVUyecVG9iayiaCcUG3UGaTDWjuj9VMWYl3HHOSl1qGUDIVibQ3wM6r8S812SJeNeaHIK94v3FTosAhKzoXJ8XP73gVIuq64kQM/640?wx_fmt=png&from=appmsg "")  
## 一个工作区，连接分析与执行  
  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;"><span cid="n96" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">Agent 对话、悬浮球与小队支持数十小时级单轮自主执行，持续组织工具调用、Skills、计划与待办，配合上下文压缩、证据归档和进展检查推进长任务。持久小队可按角色、模型与工具范围分工协作。通过对话菜单或悬浮球进入同一 Agent 工作区。浮球显示运行与未读状态，展开后可继续对话、查看工具步骤和处理确认；每个对话独立保存模型与自动审批状态。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">Agent 入口、小队与会话设置 →</span></span></span></span></span></th><th style="box-sizing: border-box;"><span cid="n97" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">持续任务与主动发现固定间隔、Cron、签名 Webhook、文件变化四类触发；支持新建对话、任务专属对话或继续现有对话。按检查点续跑，安排周期巡检，对比基线、发现变化并报告。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">调度、续跑与巡检 →</span></span></span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;"><span cid="n99" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">Web 审计与流量分析内置 Web 检查、浏览器实时观看与接管、原始请求与重放；HTTP/1.1、HTTP/2 MITM、WebSocket、断点及可选改写。支持 HAR 与 Burp 日志导入，把已有材料接入分析流程。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">扫描、浏览器与代理 →</span></span></span></span></span></td><td style="box-sizing: border-box;"><span cid="n100" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">统一逆向工作台JS、WASM、APK、IPA、PE、ELF 和 HAR 进入 ReverseJob，关联静态初筛、解包、反编译、动态观察和证据。按需连接 JADX、APKTool、Ghidra、Frida、capa、yara 等工具。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">样本与逆向任务 →</span></span></span></span></span></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;"><span cid="n102" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">证据、记忆与知识Case、Artifact、Evidence 与发现相互关联；会话记忆、知识库和大结果归档支持复查与跨轮延续。扫描报告可导出为 HTML、Markdown 和 JSON，便于整理交付。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">企业工作流与交付 →</span></span></span></span></span></td><td style="box-sizing: border-box;"><span cid="n103" mdtype="table_cell" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">工具生态与开放扩展Python Check、Agent Tool、Markdown Skill、MCP、本地 ACP Agent、QQ/飞书/Telegram 等配对通信渠道，以及桌面自动化。QQ 支持官方扫码绑定，也保留 App ID / Client Secret 手动配置。渠道回复支持平台适配的流式阶段消息和 </span></span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">channel_reply</span></span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;"> 多气泡回复；表达遵循全局用户习惯，软件内对话与悬浮球同样生效。编排外部扫描器，接入云、IaC、SBOM、离线流量、固件等专项工作流。</span></span></span><span md-inline="link" style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 12px;">扩展、渠道与工具依赖 →</span></span></span></span></span></td></tr></tbody></table>### 数十小时级单轮自主执行  
  
ArgusAI 支持数十小时级的单轮自主执行。这里的“单轮”指一次用户任务内连续进行的多次模型与工具调用，不是单次 HTTP 或模型请求。运行时没有固定的模型/工具轮次上限，默认单轮紧急时限为 7 天，并保留进展检查、死循环检测、取消与截止机制。  
  
这是运行时配置与控制流提供的能力，实际时长受任务、模型额度、工具、资源和授权限制。长调查、样本批分析和多阶段验证可以在同一轮内持续推进；需要跨轮或在下一次事件发生时继续的工作，由持久任务与调度器接续。  
### 跨轮续跑与事件驱动  
  
Agent 将进度保存为待办、检查点、知识和产物，调度器在后续触发时接上工作。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAV2QZXJ2tTyXdQNbbyYn2icDrYgajJesHkLb83CBibFUwPcdBaY5YqicbIny0MJ8LKIEicYcbP2lP9KCLwPypyY46HINb5COS7RStk/640?wx_fmt=png&from=appmsg "")  
  
  
0x03 更新介绍  
```
修复小队成员忙碌时遗漏 @ 消息，群聊消息排队接续，由成员判断是否回复。
修复小队并发消息与已读状态覆盖，保持私聊和审批续接的可见范围。
修复主 Agent 编辑重发和排队消息直接发送失败，保留附件、语音及引用。
修复本地 Agent 编辑失败回滚及取消时序，避免取消后继续启动。
调整发布流程，仅推送版本标签时打包发布并同步官网更新信息。
```  
  
0x04 使用介绍  
  
📦安装与使用指南  
## 源码运行需要 Python 3.12+、uv、Node.js 18+ 和 npm；在项目根目录执行：  
  
  
```
uv sync --locked
npm --prefix argusai/webui ci
npm --prefix argusai/webui run build
uv run argusai --gui
```  
  
首次启动后，在设置页配置 AI 端点、模型和数据目录，按需要检查浏览器与外部工具路径。接着打开完整 Agent 对话或点击悬浮球，把目标、材料和约束交给 Agent，让它建立计划并开展工作。第一次可以提供一份允许分析的 HAR 或 JS，让 Agent 整理接口、标注疑点并归档证据，再到相应功能页核对结果。  
  
Windows 桌面  
  
需要 WebView2 Runtime，Windows ARM64 还需要原生 Microsoft .NET Desktop Runtime；macOS 桌面自动化需要系统权限；Linux 桌面需要 GTK 与 WebKitGTK。  
构建、开发与发布三个平台统一使用 Python 构建入口，在目标系统和目标架构环境中运行：```
uv run --locked --group build python build.py --help
uv run --locked --group build python build.py --dry-run
uv run --locked --group build python build.py --jobs 2
```  
  
Windows 可加 --nsis  
，macOS 可加 --dmg  
，--onefile  
 生成单文件；Linux 同时生成发布压缩包。  
##   
  
  
**0x05 内部VIP星球介绍-V1.5（福利）**  
  
          
如果你想学习更多**渗透测试技术/应急溯源/免杀工具/挖洞SRC赚取漏洞赏金/红队打点等**  
欢迎加入我们**内部星球**  
可获得内部工具字典和享受内部资源和  
内部交流群，  
**每天更新1day/0day漏洞刷分上分****(2026POC更新至21922+)**  
**，**  
包含全网一些**付费扫描****工具及内部原创的Burp自动化漏****洞探测插件/漏扫工具等，AI代审工具，最新挖洞技巧、SKILL等**  
。shadon/  
Hunter  
/  
0zone  
/  
Zoomeye  
/Quake/  
Fofa高级会员/AI账号  
/CTFShow等各种账号会员共享。详情点击下方链接了解，觉得价格高的师傅后台回复"   
**星球**  
 "有优惠券名额有限先到先得  
**❗️**  
啥都有  
**❗️**  
全网资源  
最新  
最丰富  
**❗️****（🤙截止目前已有3000+多位师傅选择加入❗️早加入早享受）**  
  
****  
最新漏洞情报分享：  
https://t.zsxq.com/DSAvv  
  
****  
  
**👉****点击了解加入-->>内部VIP知识星球福利介绍V1.5版本-1day/0day漏洞库及内部资源更新**  
  
****  
  
  
结尾  
  
# 免责声明  
  
  
# 获取方法  
  
  
**公众号回复20261103获取下载、回复 加群 获取交流群**  
  
****  
  
# 最后必看-免责声明  
  
  
      
文章中的案例或工具仅面向合法授权的企业安全建设行为，如您需要测试内容的可用性，请自行搭建靶机环境，勿用于非法行为。如  
用于其他用途，由使用者承担全部法律及连带责任，与作者和本公众号无关。  
本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用。  
如您在使用本工具或阅读文章的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具或文章或来源于网络，若有侵权请联系作者删除，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！  
  
  
  
# 往期推荐  
  
  
**1.内部VIP知识星球福利介绍V1.5（AI自动化）**  
  
**2.CS4.8-CobaltStrike4.8汉化+插件版**  
  
**3.全新升级BurpSuite2026.4专业(稳定版)**  
  
**4. 最新xray1.9.11高级版下载**  
  
**5. 最新HCL AppScan Standard**  
  
  
渗透安全HackTwo  
  
微信号：关注公众号获取  
  
后台回复星球加入：  
知识星球  
  
扫码关注 了解更多  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/RjOvISzUFq6qFFAxdkV2tgPPqL76yNTw38UJ9vr5QJQE48ff1I4Gichw7adAcHQx8ePBPmwvouAhs4ArJFVdKkw/640?wx_fmt=png "二维码")  
  
  
  
上一篇文章：  
[Nacos配置文件攻防思路总结|揭秘Nacos被低估的攻击面](https://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247492839&idx=1&sn=b6f091114fbd8e8922153a996c8f4f1c&scene=21#wechat_redirect)  
  
  
