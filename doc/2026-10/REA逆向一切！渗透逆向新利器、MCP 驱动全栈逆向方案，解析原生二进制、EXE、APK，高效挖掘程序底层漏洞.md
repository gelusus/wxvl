#  REA逆向一切！渗透逆向新利器、MCP 驱动全栈逆向方案，解析原生二进制、EXE、APK，高效挖掘程序底层漏洞  
morluto
                    morluto  渗透安全HackTwo   2026-10-12 16:00  
  
0x01 工具介绍  
  
REA 是一款基于 MCP 协议的本地逆向分析工具，专为网络安全与渗透测试场景打造。支持 APK、原生二进制、Electron、.NET、固件等多类目标，可联动 Ghidra、Hopper、JADX 完成静态反编译与运行时行为捕获。依托 AI 智能体自动追踪调用链路，输出带完整证据链的分析结果，省去人工通读大量汇编的繁琐。适用于红队漏洞挖掘、二进制审计、安卓 App 逆向取证，所有分析在本地完成，兼顾效率与数据安全，助力安全研究人员快速拆解闭源程序，定位底层安全风险。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWH9XEOoGr7fkic6sUibSjI6dGh2icPTXoW8JsHicJOTLZkCHcPrKIL1xWrY7T2lwbiaKy7e1ZRFDiaARmbxs9xbuJibls2dqOydOKodU/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
  
  
  
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
  
> 在应用中看到一个想加入自己产品的功能？让智能体用 REA 调查它。即使没有源代码，智能体也可以检查应用、解释功能的工作方式、展示证据，并为你的项目实现类似功能。  
  
## REA 的工作原理  
  
**智能体通过 MCP 调用 REA，检查目标并追踪相关代码。REA 返回发现及对应证据。智能体据此提出后续问题、解释行为，或编写并测试实现。CLI 命令使用同样的工作流。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWicITzF0aicRn4FxVD2VSbR4PfJ9CeAAtq3WnpUl7cvLbYiagaicMYupLGp67wQ6GPfaa7N75LUhWkHToF5CvFsibrRk7ribqdtpswY/640?wx_fmt=png&from=appmsg "")  
## 可以分析什么  
  
**REA 需要 Node.js 22.x（>=22.19）、24.x（>=24.11）或 26+，以及 npm。额外工具和主机支持取决于分析目标：**  
<table><thead><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">目标</span></section></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">REA 返回的内容</span></section></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">要求与指南</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">原生二进制文件</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">伪代码、汇编、字符串、符号、调用和引用</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Hopper、Ghidra 或 IDA；原生分析</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">离线 ELF 布局</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">节、段、原始符号/重定位信息及静态防护机制候选项</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Linux x64 上由调用方提供的 pwntools；二进制诊断</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">EVM 字节码</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">分派选择器、字节偏移、推断的参数和状态可变性</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Linux x64 上的本地原始字节/十六进制输入载体；离线 EVM 指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">已记录的 Linux 崩溃</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">原始 note 记录、每个已记录线程的寄存器/信号及可选的映射候选项</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Linux x64 上由调用方提供的 pwntools；可选的 GDB/pwndbg；已记录崩溃</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">JavaScript / Electron</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">模块、导入、source map、路由、IPC 和原生扩展关系</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Node.js 和 npm；应用分析</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">网站</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">页面结构、脚本、网络观察结果和按请求获取的截图</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Chrome 系浏览器；浏览器分析</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">已保存的网络捕获</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">请求、响应、可访问的载荷和来源位置</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">HAR；原生 mitmproxy 捕获需要 Linux 上的 mitmdump；捕获指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">.NET 程序集</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">元数据、CIL 指令、声明的原生依赖和构建对比</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">静态检查；托管代码指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Android APK</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">清单声明、类、反编译的方法和引用</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Linux/macOS/Windows x64 上的无界面 JADX 和完整 JDK；Android 指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Android 设备</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">设备、软件包、进程、日志、转储、屏幕截图以及 APK/文件传输</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">调用方提供的 adb（模拟器或实体设备）；ADB 指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">JEB 项目</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">引擎支持的项目单元、工件摘要和类型/方法的伪代码</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">由调用方启动、提供 MCP 的 JEB 客户端，通过 REA_JEB_MCP_URL 选择；JEB 指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Android 资源</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">解码后的清单、字符串表、语言区域和版本信息</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">调用方提供的 Apktool 启动器；资源指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Flutter 应用</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Dart 快照哈希、按 ABI 的库摘要及引擎构建溯源</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">无（静态 APK 解析）；Flutter 指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">固件</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">区域、提取结果和转交原生分析的内容</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Linux 上的 Binwalk / Unblob；固件指南</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">软件包与资源</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件清单、摘要、plist、Apple bundle 结构和提取的资源</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">制品与 JavaScript 指南、Apple 应用</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">进程行为</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">终端输出、交互、退出和文件系统观察结果，以及运行对比</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">支持原生 PTY 的 Linux/macOS；进程捕获</span></section></td></tr></tbody></table>  
****  
0x03 更新介绍  
```
Android：新增设备检测、分包APK获取、日志、截图、文件传输以及通过ADB执行显式应用生命周期操作，同时支持使用Apktool进行清单文件和本地化字符串的解码。
```  
###   
  
0x04 使用介绍  
  
📦安装与使用指南  
### 设置智能体  
  
安装 Node.js 和 npm 后，运行：  
```
npx rea-agents setup
```  
  
选  
择智能体，检查计划中的变更并批准。设置流程会添加 REA 的 MCP 服务和匹配的工作流指引，并备份已有配置。完成后重启智能体。  
  
设置流程支持 Claude Code、Codex、Cursor、Gemini CLI、Grok Build 和其他智能体。提供方配置和手动注册 MCP 的方法见安装与设置。  
### 询问智能体  
```
弄清 Notes 应用的搜索功能如何工作，展示证据，并为我的项目实现类似功能。
```  
  
把 Notes 替换为目标应用，并描述你想了解的功能。  
### 使用终端  
  
检查已解包的 JavaScript/Electron 应用目录或 ASAR 文件：  
```
npx -y rea-agents@latest analyze-javascript-application /absolute/path/to/app --json
```  
  
结果包含模块、导入、Electron 边界及相关证据。将路径替换为你的目标路径，例如 Windows 上的 "D:/apps/example"  
。  
  
如果需要经常使用，可以安装 rea  
 命令：  
```
npm install --global rea-agents
rea --help
```  
### 更新 REA  
  
REA 更新较快，新版本经常包含错误修复。请保持安装的版本为最新。  
  
通过 npm 安装的 CLI 使用：  
```
rea update
```  
  
要更新智能体注册和 skill，请运行更新命令输出的设置命令。  
  
如果使用 npx  
，通过以下命令更新智能体设置：  
```
npx rea-agents@latest setup
```  
  
检查设置变更并重启智能体。单次 CLI 操作可使用 npx rea-agents@latest  
，后接所需命令。  
  
  
**0x05 内部VIP星球介绍-V1.5（福利）**  
  
          
如果你想学习更多**渗透测试技术/应急溯源/免杀工具/挖洞SRC赚取漏洞赏金/红队打点等**  
欢迎加入我们**内部星球**  
可获得内部工具字典和享受内部资源和  
内部交流群，  
**每天更新1day/0day漏洞刷分上分****(2026POC更新至12922+)**  
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
  
  
**公众号回复20261013获取下载、回复 加群 获取交流群**  
  
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
  
**4. 最新xray1.9.11高级版下载Windows/Linux**  
  
**5. 最新HCL AppScan Standard**  
  
  
渗透安全HackTwo  
  
微信号：关注公众号获取  
  
后台回复星球加入：  
知识星球  
  
扫码关注 了解更多  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/RjOvISzUFq6qFFAxdkV2tgPPqL76yNTw38UJ9vr5QJQE48ff1I4Gichw7adAcHQx8ePBPmwvouAhs4ArJFVdKkw/640?wx_fmt=png "二维码")  
  
  
  
  
上一篇文章：  
[Nacos配置文件攻防总结|揭秘Nacos被低估的攻击面](https://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247492839&idx=1&sn=b6f091114fbd8e8922153a996c8f4f1c&scene=21#wechat_redirect)  
  
  
