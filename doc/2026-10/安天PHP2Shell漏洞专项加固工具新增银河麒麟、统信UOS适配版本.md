#  安天PHP2Shell漏洞专项加固工具新增银河麒麟、统信UOS适配版本  
安天测试交付部
                    安天测试交付部  安天垂直响应平台   2026-10-25 08:20  
  
点击上方"蓝字"  
  
关注我们吧！  
  
  
# 导语  
  
针对PHP2Shell高危漏洞，安天继昨日发布覆盖Linux与Windows平台的配置加固工具后，今日新增银河麒麟、统信UOS适配版本。该工具覆盖统信  
 UOS 与银河麒麟等 x86_64 国产化操作系统，将上游官方缓解措施与安天纵深加固经验整理为可扫描、可加固、可验证、可回滚的加固项；框架内每条加固规则均配套适用条件、变更影响、校验及回滚方案，执行遵循预检预览、快照留存、不强制覆盖原有配置的原则，适配服务器运维场景，规则库与程序版本独立管理，用于在补丁升级完成前快速收敛风险暴露面。  
  
提示：  
官方补丁仍是根本修复手段，建议优先使用官方补丁。加固工具不能替代补丁，只能作为叠加防护。工具面向专业用户，用于短期内无法完成全部升级的生产环境。  
# 工具介绍  
  
·  
   
该工具是一款免费应急响应工具，简称  
 PHP 专项配置加固工具，由安天测试交付部出品，配置知识由安天测试交付部整理；本期发布适配统信 UOS 与银河麒麟系统的版本。  
  
  
  
·  
   
工具将系统与应用的加固知识转化为可审查的配置计划，提供受约束的执行、状态记录和快照回滚能力，帮助运维和安全人员明确“改什么、为什么改、如何改、如何验证以及如何恢复”。  
  
  
  
·  
   
面向服务器运维场景提供子命令式命令行界面，默认交互确认，加  
 --yes 免确认、便于脚本化；配置布局按发行版自动匹配，具体执行机制按操作系统及组件差异实现。  
  
  
  
·  
   
工具默认仅执行官方标准模式（official），只包含上游修复公告与官方配置指引确认的缓解措施；深度加固模式（deep）需人工选择，可能影响业务连续性。  
  
  
  
·  
   
工具不运行漏洞利用、不强制触发内存耗尽、不主动外连，只做配置核查与加固。  
  
  
## 环境适配  
  
·  
 配置布局自动适配：  
对统信 UOS 等 Debian 系系统、银河麒麟等 RHEL 系系统自动适配系统版本，无需按发行版硬编码路径。当自动识别失败时，支持通过 --ini-dir、--pool-dir 参数手动指定目录；工具通过读取 /etc/os-release 获取系统信息，输出当前系统名称及内核版本。  
  
  
  
·  
 架构边界：  
本版本仅提供  
 x86_64；飞腾、鲲鹏（aarch64）、龙芯（LoongArch）、申威等其它架构不在覆盖范围。  
  
  
## 适用场景  
  
·  
 漏洞应急期间的配置缓解：  
查阅专项规则，按适用条件收紧入口、权限和执行环境；补丁升级、真实版本核查及效果验证仍需继续完成。  
  
  
  
·  
 PHP 应用上线前审查：  
对照  
 Web 服务、PHP/FPM 及宿主配置逐项检查并调整，并验证业务兼容性和实际加载链。  
  
  
  
·  
 服务器日常配置维护：  
在命令行中选择方案并设置逐项例外，按组件要求检查语法、加载配置及复核运行状态。  
  
  
  
·  
 测试交付与运维接手：  
以规则编号、目标、操作结果和恢复条件组织交接，在目标环境完成集成测试并保留验证证据。  
  
  
## 核心能力  
  
·  
 结构化配置知识：  
每项规则包含配置名称、说明与位置、默认值或默认值范围、调整建议、副作用、验证方法和恢复说明。官方标准模式  
 15 项、深度加固模式 212 项、13 个分类，均提供完整中英文对照；数量反映知识或实现覆盖，不能直接换算为漏洞修复率。  
  
  
  
·  
 明确目标后执行：  
可执行条目绑定固定规则标识、操作方式和参数范围。操作者选择实际目标并完成预检，审查计划后再确认执行；知识描述本身不会直接被当成任意命令运行，目标、权限或组件条件不满足时操作会被拒绝。  
  
  
  
·  
 受保护快照与冲突检查回滚：  
工具在变更前保存恢复所需状态，回滚前检查当前目标是否仍符合事务预期，避免覆盖后续合法变更。事务记录保存于独立状态目录  
 /var/lib/phphard，回滚前校验文件是否被他人改动；恢复配置不等于恢复进程内存或业务数据。  
  
  
  
·  
 场景方案与逐项调整：  
采用内置场景方案加逐项例外的方式，支持策略  
 JSON 导入导出、预览和确认执行；内置基础、生产、严格和应急方案目前是少量 PHP 默认集合，不会自动启用全部系统配置，外部策略文件用于复用选择和参数，不能加入任意命令。  
  
  
  
·  
 适应服务器操作：  
提供编号式命令行与非交互命令；默认中文，可切换英文，语言切换不改变规则身份及执行参数。  
  
  
  
·  
 记录结果并支持后续核验：  
命令行提供适合记录和自动化衔接的结构化结果（--json）；配置源值、写入结果与运行时有效状态分别解释，便于运维与测试人员继续完成组件加载、业务检查和恢复验证。  
  
  
  
·  
 知识嵌入与动作数据：  
运行所需知识和动作数据在编译期内嵌于二进制，加载时检查完整性，不含外部数据文件；维护源与运行包分离，规则更新通过维护、验证和重新构建完成。该机制用于保护静态内容及检查损坏，不能抵抗逆向提取，也不提供发布者数字签名保证。  
  
  
  
下载地址：https://vs.antiy.cn/downloads/SecurityConfiguration/phphard.tar.gz  
## 规则库与加固模式  
  
沿用Linux  
版工具  
标准  
 15 项与深度加固 212 项两套规则库，构成与业务影响见表 1；国产化环境建议先以 official 起步，再按信创基线要求评估 deep  
。  
  
表  
 1 双加固模式对比  
  
<table><tbody><tr><td data-colwidth="140" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">模式</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="96" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">加固项数量</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="158" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">内容来源</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="183" width="193" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">业务影响</span></span></font></span></b><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="140" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">official 官方标准（默认，无需参数）</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="96" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">15 项</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="158" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">仅上游修复公告中的官方缓解措施与事件处置步骤</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="183" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">最小</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr><tr><td data-colwidth="140" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">deep 深度加固（加 --deep 切换）</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="96" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">212 项</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="158" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">官方措施加本方纵深加固整理</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td><td data-colwidth="183" width="193" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">含容量裁剪、功能限制、身份与路径硬限制等，可能影响业务连续性</span></span></font></span><span style="font-family:微软雅黑;font-size:10.5000pt;"><o:p></o:p></span></p></td></tr></tbody></table>  
  
official 集合由 5 个官方处置工作流（INCIDENT-001~005）、8 个 PHP 配置加固点（PHP-001/002/003/027/028/030/031/037）及 2 项组件级 RLIMIT 误用审计（APACHE-012 与 SYSTEMD-033）构成，为默认模式、无需参数；deep 模式加 --deep 切换，执行前会打印业务连续性警告，需运维显式确认。  
  
适配器覆盖：已实现  
 PHP INI（43 项）、FPM 池（20 项，加固限低危键）、内核参数（20 项）、systemd 沙箱指令（43 项），以及补丁状态判定、旧 worker 检测、启动诊断采集等官方处置工作流检查；Nginx、Apache、LSM、网络、容器等适配器尚未实现，相关项扫描结果如实标注为需人工复核，加固命令拒绝执行。  
## 运行前检查  
  
·  
 权限检查：  
扫描与加固要读取/写入系统配置与状态目录（/etc/php、/var/lib/phphard 等），必须以 root 运行，非 root 直接拒绝执行；list / info / env / --help / --version 免 root，可随时使用。  
  
  
  
·  
 漏洞存在性检查：  
工具按  
 PHP 分支与上游修复版本下限（8.2≥8.2.35 / 8.3≥8.3.36 / 8.4≥8.4.27 / 8.5≥8.5.12）判定补丁状态：已高于修复版本时加固命令拒绝执行并提示未检测到相关漏洞；版本无法确认（PHP 未安装或已 EOL 分支）时同样结束，可用 --php-version 手动指定，不臆断安全。确需作为安全基线加固时可跳过门禁。  
  
  
## 信创环境使用要点  
  
·   
强制访问控制识别  
：银河麒麟V10默认搭载KYSEC，部分环境启用SELinux 强制模式。工具识别系统保护状态后再执行配置变更，避免被系统强制防护机制阻断，相关操作遵循操作系统厂商安全规范。  
  
  
·   
环境版本兼容控制  
：部分systemd沙箱类防护能力存在版本依赖，工具自动检测环境支持情况；对当前环境不支持的配置项自动跳过，并在结果报告中明确标注。  
  
  
·   
分级处置策略  
：部分LSM、网络边界、容器相关基线项依赖现场业务拓扑研判，工具输出标准化处置指引，由运维人员结合场景人工复核落地。  
  
  
·   
变更可追溯  
：加固过程留存快照、执行状态记录与校验信息，支撑变更审计与项目验收归档。  
  
  
·   
发行版适配与生效机制  
：工具适配统信UOS、银河麒麟组件管理体系；完成PHP相关配置加固后，需根据系统对应的php-fpm 服务名称重启服务使配置生效。  
  
  
## 使用方式  
<table><thead><tr><th><section><span leaf=""><span textstyle="" style="font-size: 14px;">命令</span></span></section></th><th><section><span leaf=""><span textstyle="" style="font-size: 14px;">说明</span></span></section></th></tr></thead><tbody><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard env</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">环境发现：发行版 / 内核 / PHP 版本 / 补丁状态 / SAPI/unit/ 加固模式与集合</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard scan</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">全量扫描（默认 official 官方标准模式）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard scan --mode deep</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">深度加固模式全量扫描 212 项</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard scan --prod-risk safe</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">只看可一键加固的安全项</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard info PHP-001</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">加固点详情：环境 / 建议 / 副作用 / 验证 / 回滚 / 生产风险</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard apply PHP-001 --dry-run</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">单项预演（不落盘）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard apply PHP-001 --yes</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">单项加固（先快照后写入）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard harden-all --dry-run</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">一键加固预演（official）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard harden-all --yes</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">一键加固（official safe 集合，逐项独立快照）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard harden-all --mode deep --yes</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">深度加固（⚠ 可能影响业务连续性）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard rollback PHP-001 --yes</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">单项撤销（只恢复本次修改）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard rollback --all --yes</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">一键回滚全部（后建先撤）</span></span></section></td></tr><tr><td><code><span leaf=""><span textstyle="" style="font-size: 14px;">sudo ./phphard status</span></span></code></td><td><section><span leaf=""><span textstyle="" style="font-size: 14px;">事务历史</span></span></section></td></tr></tbody></table>##   
  
·  
 环境发现：  
phphard env，查看系统名、内核、PHP 版本、补丁状态、SAPI、运行单元与容器信息，免 root。  
  
  
  
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
  
使用示例：环境确认（env）→ 官方扫描 → 预演（harden-all --dry-run）→ 实执行（harden-all --yes）→ 查看历史（status），必要时一键回滚（rollback --all --yes）。回滚只恢复本工具自己改过的内容；若他人后续修改过同一文件，工具会拒绝覆盖并报告冲突，需人工处置。  
  
以下为统信UOS系统上的运行输出示意：  
  
![](https://mmecoa.qpic.cn/mmecoa_png/frAs2k03YpnCKGeJgxV6ibvicPYVluicx3wibeHNV6LxXUeFMrWuGpUkumyNUQ79TuEEPOAnrUTxpjWaLSZia3xZribIMFJ8oRMMQyzwru1YYIMzk/640?wx_fmt=png&from=appmsg "")  
  
图  
 1 phphard：环境发现  
   
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/frAs2k03Ypm1nicvRjoZNbvCJxRtlRtIicj9ibxm8b0vESfSQWmeZwq6F9WTSdLy3WicEMBtnibpCDib4vcv6817vV6eoDQGkicfvSicav8npgAW8Hc/640?wx_fmt=png&from=appmsg "")  
  
图  
 2 phphard：扫描结果  
## 扫描结果解读  
##   
  
扫描结果按“能不能自动加固、要不要人工介入”分为五组，各组含义与对应操作见表 2：  
  
表2 扫描结果分组与处置  
<table><tbody><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">分组</span></span></font></span></b></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">含义</span></span></font></span></b></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(217, 226, 243);"><p style="text-align:center;"><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">需</span></span></font></span></b><b style="mso-bidi-font-weight:normal;"><span style="font-family:微软雅黑;mso-ansi-font-weight:bold;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">要做什么</span></span></font></span></b></p></td></tr><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">可加固</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">· 一键加固</span></span></font></span></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">不符合建议值，改了不影响业务</span></span></font></span></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">harden-all --yes 一键做完</span></span></font></span></p></td></tr><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">可加固</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">· 确认后加固</span></span></font></span></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">改了可能影响业务（容量</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">/性能/功能裁剪）</span></span></font></span></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">评估后</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">harden-all --include-caution --yes</span></span></font></span></p></td></tr><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">需给值</span></span></font></span></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">工具能改，但值要按现场定（如</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">memory_limit）</span></span></font></span></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">apply &lt;ID&gt; --value &lt;值&gt;</span></span></font></span></p></td></tr><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">需人工</span></span></font></span></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">工作流类（打补丁、隔离入口），或本版本未实现自动适配</span></span></font></span></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">info &lt;ID&gt; 看步骤后人工处理</span></span></font></span></p></td></tr><tr><td data-colwidth="106" width="175" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">已达标</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">· 不适用</span></span></font></span></p></td><td data-colwidth="251" width="300" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">已符合建议值，或本机没有对应对象（版本</span></span></font><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">/组件不支持）</span></span></font></span></p></td><td data-colwidth="219" width="299" valign="top" style="padding: 0pt 5.4pt;border-left: 1pt solid windowtext;border-right: 1pt solid windowtext;border-top: none;border-bottom: 1pt solid windowtext;"><p><span style="font-family:微软雅黑;font-size:9.5000pt;"><font face="微软雅黑"><span leaf=""><span textstyle="" style="font-size: 14px;">无需处理</span></span></font></span></p></td></tr></tbody></table>  
默认只展示可加固的项，加--all 才展示全部；五组数量相加等于本机命中项数。  
# 常见问题及补充说明  
  
·  
 统信 UOS、银河麒麟是否已经可以直接下载运行？  
已提供下载，下载地址见“统信 UOS / 银河麒麟版：phphard 命令行工具”一节；首批  
适配  
 x86_64 的统信 UOS 与银河麒麟系统，目标环境的集成验收尚待完成，正式运行包以发布说明为准。  
  
  
  
·  
 支持飞腾、鲲鹏等其它架构吗？  
本版本仅提供  
 x86_64；飞腾、鲲鹏（aarch64）、龙芯（LoongArch）、申威等其它架构不在覆盖范围，相关环境请关注后续发布说明。  
  
  
  
·  
   
目标  
 systemd 版本过旧时部分沙箱指令不支持（如 ProtectProc、ProcSubset 需 247 及以上），加固后验证会如实报告失败。  
  
  
  
·  
   
银河麒麟  
 V10 默认启用 kysec，部分发行版启用 SELinux 强制模式，写入受保护位置需按厂商规范确认放行。  
  
