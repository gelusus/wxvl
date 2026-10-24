#  PHP2Shell漏洞如何快速响应，安天发布多平台配置加固工具  
安天测试交付部
                    安天测试交付部  安天垂直响应平台   2026-10-24 11:00  
  
点击上方"蓝字"  
  
关注我们吧！  
  
  
# 导语  
#   
  
针对 PHP2Shell 高危漏洞，安天测试交付部发布 PHP2Shell 漏洞加固工具。该工具覆盖 Linux 与 Windows 两个平台，将上游官方缓解措施与安天纵深加固经验整理为可扫描、可加固、可验证、可回滚的加固项；框架内每条加固规则均配套适用条件、变更影响、校验及回滚方案，执行遵循预检预览、快照留存、不强制覆盖原有配置的原则，适配服务器与桌面运维场景，提供多交互入口，规则库与程序版本独立管理，用于在补丁升级完成前快速收敛风险暴露面。  
  
提示：  
官方补丁仍是根本修复手段  
，  
建议优先使用官方补丁。加固工具不能替代补丁，只能作为叠加防护。工具面向专业用户，用于短期内无法完成全部升级的生产环境。  
# 工具介绍  
#   
  
·  
   
该工具  
是一款免费应急响应工具  
，简称 PHP 专项配置加固工具，由安天测试交付部出品，配置知识由安天测试交付部整理。  
  
  
  
·  
   
工具将系统与应用的加固知识转化为可审查的配置计划，提供受约束的执行、状态记录和快照回滚能力，帮助运维和安全人员明确“改什么、为什么改、如何改、如何验证以及如何恢复”。  
  
  
  
·  
   
Windows 与 Linux 两端保持一致的知识组织、双语呈现、品牌信息和受控变更思路；Windows 采用原生桌面界面，Linux 提供编号式命令行、非交互命令界面，具体执行机制按操作系统及组件差异实现。  
  
  
  
·  
   
Linux 版默认仅执行官方标准模式（official），只包含上游修复公告确认的缓解措施；深度加固模式（deep）需  
人工  
选择，可能影响业务连续性。  
  
  
  
·  
   
工具不运行漏洞利用、不强制触发内存耗尽、不主动外连，只做配置核查与加固。  
  
  
## 适用场景  
##   
  
·  
 漏洞应急期间的配置缓解：  
查阅专项规则，按适用条件收紧入口、权限和执行环境；补丁升级、真实版本核查及效果验证仍需继续完成。  
  
  
  
·  
 PHP 应用上线前审查：  
对照  
 Web 服务、PHP/FPM 及宿主配置逐项检查并调整，并验证业务兼容性和实际加载链。  
  
  
  
·  
 服务器日常配置维护：  
在  
 Linux 编号菜单或命令行中选择方案并设置逐项例外，按组件要求检查语法、加载配置及复核运行状态。  
  
  
  
·  
 测试交付与运维接手：  
以规则编号、目标、操作结果和恢复条件组织交接，在目标环境完成集成测试并保留验证证据。  
  
  
## 核心能力  
##   
  
·  
 结构化配置知识：  
每项规则包含配置名称、说明、位置与存放方式、默认值或默认值范围、调整建议、副作用、验证方法和恢复说明。Windows 现有 195 项规则、16 个分类，Linux 现有 212 项规则、13 个分类，均提供完整中英文对照；两套规则按平台独立维护，数量反映知识或实现覆盖，不能直接换算为漏洞修复率。  
  
  
  
·  
 明确目标后执行：  
可执行条目绑定固定规则标识、操作方式和参数范围。操作者选择实际目标并完成预检，审查计划后再确认执行；知识描述本身不会直接被当成任意命令运行，目标、权限或组件条件不满足时操作会被拒绝。  
  
  
  
·  
 受保护快照与冲突检查回滚：  
工具在变更前保存恢复所需状态，回滚前检查当前目标是否仍符合事务预期，避免覆盖后续合法变更。Windows 使用系统数据保护机制保护相关事务，Linux 使用独立本地密钥保护事务记录；恢复配置不等于恢复进程内存或业务数据。  
  
  
  
·  
 场景方案与逐项调整：  
Linux 采用内置场景方案加逐项例外的方式，支持策略 JSON 导入导出、预览和确认执行；内置基础、生产、严格和应急方案目前是少量 PHP 默认集合，不会自动启用全部系统配置，外部策略文件用于复用选择和参数，不能加入任意命令。  
  
  
  
·  
 适应桌面与服务器操作：  
Windows 采用原生桌面界面，Linux 提供编号式命令行、非交互命令和本机浏览器界面；默认中文，可切换英文，语言切换不改变规则身份及执行参数。Linux 浏览器入口用于本机操作，不是集中管理多台服务器的远程控制平台。  
  
  
  
·  
 记录结果并支持后续核验：  
界面提供报告导出，命令行提供适合记录和自动化衔接的结构化结果；配置源值、写入结果与运行时有效状态分别解释，便于运维与测试人员继续完成组件加载、业务检查和恢复验证。  
  
  
  
·  
 加密嵌入知识与动作数据：  
运行所需知识和动作数据在构建时加密封装后嵌入，加载时检查完整性；维护源与运行包分离，规则更新通过维护、验证和重新构建完成。该机制用于保护静态内容及检查损坏，不能抵抗逆向提取，也不提供发布者数字签名保证。  
  
  
# Linux 版：phphard 命令行工具  
#   
  
该工具  
是一个单文件可执行文件，用于扫描并对  
 Linux 上 PHP 运行环境实施 针对 CVE-2026-109618（PHP OOM）、CVE-2026-109619（PHP stale state）及相关上传进度残余路径的加固。可执行文件约 1.2 MB，复制到目标机即可运行，不依赖目标机 glibc 版本与外部数据文件，也无需安装运行时或解释器。  
  
  
  
兼容范围：  
在  
 CentOS 6.6（glibc 2.12）上静态编译，可在 CentOS 6/7/8、Ubuntu 18.04/20.04/22.04、Debian 等 x86_64 Linux 上运行，低版本编译产物兼容高版本系统；所有加固可回滚，每项加固写入前先快照原值或原文件。  
  
  
  
下载地址：https://vs.antiy.cn/downloads/SecurityConfiguration/phphard-linux-x86_64.tar.gz  
  
  
## 规则库与加固模式  
  
表  
 1 双加固模式对比  
  
<table><tbody><tr><td data-colwidth="138" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">模式</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="89" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">加固项数量</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="162" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">内容来源</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="178" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">业务影响</span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="138" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">official 官方标准（默认，无需参数）</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="89" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">11 项</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="162" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">仅上游修复公告中的官方缓解措施与事件处置步骤</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="178" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">最小</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="138" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">deep 深度加固（加 --deep 切换）</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="89" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">212 项</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="162" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf="">官方措施加本方纵深加固整理</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="178" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">含容量裁剪、功能限制、身份与路径硬限制等，可能影响业务连续</span>性</span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
official 集合由 5 个官方处置工作流、PHP-001/002/003/037 四个加固点及 2 项 RLIMIT 误用审计构成，为默认模式、无需参数；deep 模式加 --deep 切换，执行前会打印业务连续性警告，需运维显式确认。  
  
适配器覆盖：  
已实现  
 PHP INI（43 项）、FPM 池（20 项，加固限低危键）、内核参数（20 项）、systemd 沙箱指令（43 项），以及补丁状态判定、旧 worker 检测、启动诊断采集等官方处置工作流检查；Nginx、Apache、LSM、网络、容器等适配器尚未实现，相关项扫描结果如实标注为需人工复核，加固命令拒绝执行。  
## 运行前检查  
##   
  
·  
 权限检查：  
扫描与加固要读取/写入系统配置与状态目录（/etc/php、/var/lib/phphard 等），必须以 root 运行，非 root 直接拒绝执行；list / info / env / --help / --version 免 root，可随时使用。  
  
  
  
·  
 漏洞存在性检查：  
工具按  
 PHP 分支与上游修复版本下限（8.2≥8.2.35 / 8.3≥8.3.36 / 8.4≥8.4.27 / 8.5≥8.5.12）判定补丁状态：已高于修复版本时加固命令拒绝执行并提示未检测到相关漏洞；版本无法确认（PHP 未安装或已 EOL 分支）时同样结束，可用 --php-version 手动指定，不臆断安全。确需作为安全基线加固时可跳过门禁。  
  
  
## 使用方式  
##   
  
·  
 环境发现：  
phphard env，查看发行版、内核、PHP 版本、补丁状态、SAPI、运行单元与容器信息，免 root。  
  
  
  
·  
 扫描：  
phphard 或 phphard scan 执行官方标准扫描（默认）；scan --deep 切换深度加固模式 212 项全集；scan --all 显示全部加固项，含需人工配置的明细与原因。  
  
  
  
·  
 查看加固点：  
phphard list 列出加固点；phphard info <编号> 查看单项的适用环境、建议值、副作用、验证方法、回滚原则与生产风险定级。  
  
  
  
·  
 单项加固：  
phphard apply <编号>，--dry-run 预演不落盘，--yes 确认执行；每次加固先快照后写入；加固后如需立即生效加 --reload。  
  
  
  
·  
 一键加固：  
phphard harden-all --yes 对 official 安全集合逐项独立快照并执行；--include-caution 纳入 caution 级；--deep 切换深度集合，执行前会打印业务连续性警告。  
  
  
  
·  
 撤销回滚：  
phphard rollback <编号> 单项撤销；rollback --all --yes 一键回滚全部，按后建先撤恢复，只恢复本工具的修改。  
  
  
  
·  
 验证与历史：  
phphard verify <编号> 验证单项效果；phphard status 查看加固事务历史。  
  
  
  
说明：apply 与 rollback 默认需交互确认；脚本化使用请加 --yes，stdin 非 TTY 时未应答按取消处理，不会误执行。  
  
使用示例  
：环境确认（env）→ 官方扫描 → 预演（harden-all --dry-run）→ 实执行（harden-all --yes）→ 查看历史（status），必要时一键回滚（rollback --all --yes）。回滚只恢复本工具自己改过的内容；若他人后续修改过同一文件，工具会拒绝覆盖并报告冲突，需人工处置。以下截图依次为使用说明、环境发现、深度加固与回滚操作的输出。  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03YpkYJ4v3gGFl7qoaMtqZn3qxhBOoA8L9GiauUZY16pTCViaCoVVqjDoF11ukudEmRJxP2Po42VWljQKZ9PdhicsFo2fmSmOuq05IIo/640?wx_fmt=png&from=appmsg "")  
  
图  
 1 phphard：使用说明  
   
  
![](https://mmecoa.qpic.cn/mmecoa_png/frAs2k03YpljtYQXBoCicRNIAPMwXKfJAVpkicQvGylQLiacp6ekj1ggQhdDSiaObialjvWvphFldNNJAmrTh075HRyVQ6g4KuNLvfXnXsy2fJPE/640?wx_fmt=png&from=appmsg "")  
  
  
图  
 2 phphard：环境发现  
  
![](https://mmecoa.qpic.cn/mmecoa_png/frAs2k03Ypm9SOmaFaJ8O9Tv72qcHcFnpicBMPKyiaI84tOTD21gFavecoJCSiaJ5BUDjR1D9cjRAe6WfMlwSicdDOQFzYukA7qgM6zmSj0PDO0/640?wx_fmt=png&from=appmsg "")  
  
  
图  
 3 phphard：深度加固  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03YpkYicic6f6KvczdyS44OeNaNwkB31kFibpibZNGezY4aPmugAuHe59MERFzObcicKaztOeftmrg62GWGORGhgWg62GlhbMvcjE5KmaM/640?wx_fmt=png&from=appmsg "")  
  
图  
 4 phphard：回滚操作  
## 扫描结果解读  
##   
  
扫描结果按“能不能自动加固”分为四组：可自动加固，仅收紧暴露面与信息泄露、不改变业务行为，harden-all默认一键加固；可自动加固，可能改变业务行为，需  
执行  
--include-caution  
参数  
才纳入；需人工配置，不能由配置写入完成，按  
 phphard info <编号> 查看人工步骤与不能自动加固的原因；已符合 / 不适用，无需处理。默认只展示可自动加固的项，加 --all 才展示全部。  
# Windows 版：PHP-CVE-2026-109618+109619修复工具  
#   
  
该工具是面向  
 Windows 平台的 PHP 安全配置审查与加固工具，服务 CVE-2026-109618 与 CVE-2026-109619 修复相关配置。工具采用图形界面，默认中文、可通过“语言 / Language”菜单切换英文，切换时保留目标、扫描结果和已暂存计划；交付为单文件 EXE，离线运行，不需要 Python 或外部数据文件；标准用户即可启动，不会自动申请提升权限。  
  
下载地址：https://vs.antiy.cn/downloads/SecurityConfiguration/AConfig.zip   
## 规则库  
##   
  
工具内置  
 195 项加固规则、16 个分类，其中 Web 相关 117 项、系统策略与部署 78 项，每项均建立真实的写入与恢复程序映射。规则知识在构建时加密封装后内嵌，运行时验证并解密到内存，旁置数据包不是热更新入口。  
## 使用方式  
##   
  
1.  
在左侧选择分类，或按编号、名称和说明搜索；双击条目查看默认值范围、建议、副作用、激活、验证、回滚及官方来源。  
  
  
  
2.  
点击“选择 INI / 版本”，选择本地 PHP INI 文件并声明部署 PHP 版本；版本未知可留空，只进行只读检查。声明的版本不是程序探测到的 Web 版本。  
  
  
  
3.  
点击“只读扫描”，快速观测 7 个 PHP 布尔项以及本机 DEP、防火墙概况；该扫描不等于所有动作的预检。  
  
  
  
4.  
任意条目使用“配置选中项”，填写目标与参数，阅读双语帮助和恢复限制，只读预检后审查并确认真实执行；原 7 个 PHP 布尔项也可“加入计划”，批量预览后执行。  
  
  
  
5.  
根据动作说明激活并验证业务进程。纯配置动作不自动重启服务；服务状态和容器部署等动作会按明确参数改变运行状态。成功不等于漏洞已修复。  
  
  
  
6.  
“快照回滚”选择本工具的 .cta 或 .ctb 事务，预检真实恢复目标后确认。后续改动会阻止覆盖；恢复配置不会复活原进程或恢复业务数据。  
  
  
  
7.  
“导出报告”生成 JSON 或 Markdown，包含有限观测、暂存计划及本次会话的逐项动作结果；请选择新文件名，不导出完整参数和配置快照。  
  
  
  
工具  
使用示例及  
主要界面如下图所示  
：  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03YpnKwa9iahI4kyuo0BsmXSDjpWic6UbZtu93oWc4UuYAGcKVLx286hN8iaJUuEkyvA3jufP9BsibJ4e1LzgsmOSRiaQK0hljaOicsvtQU/640?wx_fmt=png&from=appmsg "")  
  
图  
 5 Windows 版加固工具：指定配置文件  
   
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03YploJkDmibibNSBic6naibjde0blGtqkDTmeibLS87D9GSiaVNOnnuQM6iblsjdpz60kFDUbvbY6veicQNqzBiatfOTAcVl307NqhAHugBdo/640?wx_fmt=png&from=appmsg "")  
  
  
图  
 6 Windows 版加固工具：只读扫描  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03Ypkr31uxYdJp6OT7KXSHicB5m0AdSRSoMOP42JOmmJzFx0iakb46q6M32qhOaVhyVgrzToItu7YTpP0dShfFrQrtf8yiacKc0FdCP8/640?wx_fmt=png&from=appmsg "")  
  
图  
 7 Windows 版加固工具：加入加固计划  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03YpmTkzAiaCtNB6xheDIP7tmbn1edvUoKEzK89BBA5Ro1obQwZu71Nbs50WgviamoIk0U1micg0tG9nBGhtRjnbOMcEUQb0shWricefk/640?wx_fmt=png&from=appmsg "")  
  
  
图  
 8 Windows 版加固工具：预览执行与回滚配置  
## 扫描结果解读  
##   
  
扫描与动作结果以“未知”“人工核验”“不适用”“读取失败”“源值符合”“源值待调整”“已写入待激活”分别显示，统计数量不能换算成漏洞修复率。默认值仅作知识参考，缺失配置不能当作 0。本版不自动发现完整 Web 配置加载链，也不验证所有运行时有效值；配置已写入不代表运行生效，成功不等于漏洞已修复。  
# 常见问题及补充说明  
#   
  
·  
 规则数量代表漏洞修复率吗？  
规则数量表示知识或实现覆盖，不能直接换算为漏洞修复率；Windows 与 Linux 两套规则按平台独立维护，不宜把数量简单相加并称为互不重复的安全能力。  
  
  
  
·  
 工具是否可以直接修复漏洞？  
工具主要提供配置审查和加固能力。具体措施可能用于预防、缓解、降低利用机会或限制后续动作，但不能由“配置已写入”直接判定漏洞已修复。补丁状态、实际运行版本和业务有效态仍需核验。  
  
  
  
·  
 是否支持一键加固整台服务器？  
当前按明确规则、目标和参数实施变更。Linux 场景方案提供一组起始选择，操作者仍需确认适用范围并审查计划。工具不承诺自动发现所有组件或对整机实施无需确认的加固。  
  
  
  
·  
 配置改变后是否立即生效？  
取决于提供程序与目标组件。部分原生策略动作会改变相关状态；许多文件配置需要重载、重启或外部激活。界面和报告中的写入结果不能代替实际运行验证。  
  
  
  
·  
 回滚是否能够恢复所有影响？  
回滚恢复的是支持范围内、由本工具保存的变更前状态。它不能自动恢复已经丢失的业务数据、原进程内存或被外部操作激活的不可逆限制。多文件操作可能部分完成，需按事务结果执行恢复。  
  
  
  
·  
 是否依赖外部策略文件或联网服务？  
运行规则由程序内嵌，核心操作不依赖联网服务。Linux 外部 JSON 策略用于复用规则选择和参数，属于可选方式，不能加入任意命令。部分动作仍需要目标系统已有的组件、工具、权限和运行条件。  
  
  
  
·  
 Linux 是否已经可以直接下载运行？  
Linux 版已提供下载，下载地址见“Linux 版：phphard 命令行工具”一节；首批面向 Ubuntu/Debian，目标环境的集成验收尚待完成，正式运行包以发布说明为准。  
  
  
## Linux 版  
##   
  
·  
   
deep 深度加固模式含容量裁剪与功能限制，执行前应在测试环境验证。  
  
  
  
·  
   
数值型建议（如  
 post_max_size、pm.max_children）必须按业务容量本地压测后配置，工具示例值不是通用阈值。  
  
  
  
·  
   
Linux 版已在 Ubuntu 20.04（PHP-FPM + systemd）实测；其他发行版、内核、systemd 组合及 FPM+Nginx、FPM+Apache、mod_php、容器、SELinux/AppArmor 场景尚未实测，上线前应按随附文档验证。  
  
  
  
·  
   
目标  
 systemd 版本过旧时部分沙箱指令不支持（如 ProtectClock 需 systemd 246 及以上），加固后验证会如实报告失败。  
  
  
  
·  
   
事务状态目录默认  
 /var/lib/phphard，需 root 可写。  
  
  
## Windows 版  
##   
  
·  
   
工具面向  
 Windows 10/11 x64 与 Windows Server 2016 及以后版本，尚未覆盖全部目标系统的真机验证。  
  
  
  
·  
   
195 项均已建立写入与恢复映射，但实际目标环境的逐项验收交由外部集成测试，映射数量不等于已验收数量。  
  
  
  
·  
   
7 个 PHP 布尔项适配 PHP 8.2–8.5（fatal_error_backtraces 仅 8.5），该门槛是执行器审核边界，不表示这些分支仍在安全支持期。  
  
  
  
·  
   
本版不自动发现完整  
 Web 配置加载链，不验证所有运行时有效值，不提供整机“一键恢复默认”；配置写入成功不等于漏洞已修复，统计数量不能换算成漏洞修复率。  
  
  
  
·  
   
事务机制为带预写备份的独占写入，不是断电原子事务，极端情况下可能留下部分内容，应保留部署方独立备份。  
  
  
  
·  
   
回滚须由执行变更的同一  
 Windows 用户完成；受保护目录的写入可能需要管理员权限。  
  
  
  
·  
   
内置知识为静态加密，密钥随程序存在，不能抵抗逆向提取，也不提供发布者数字签名保证。  
  
