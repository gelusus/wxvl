#  【高危漏洞预警】Cacti Guest访客未授权远程代码执行漏洞  
 飓风网络安全   2026-11-27 10:26  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Yd9HAo0qc3rjgYy2pKQKOVBzicDVBXdb6NwSFHbkfgQekCCqL3KRf421iaAtorgIJue6iajiadHZD0ibBvnxgT9qzu6tqY7DqwghyxDF8o04LkYU/640?wx_fmt=jpeg "")  
  
一、预警结论  
  
Cacti（开源网络性能与故障监控框架）出现一个未打补丁的零日 RCE，研究代号**4HTTP2ROOT / 4HTTP2SHELL**  
：  
- **入口前提**  
：站点开启了匿名**guest（访客）账号**  
；攻击者无需登录、无需有效 API Key、无需钓鱼，仅凭一个全新未认证 HTTP会话即可发起；  
- **利用链路**  
：guest 会话向用户资料接口写入未校验的SQL 字符串（二阶 SQL 注入）→ 触发堆叠语句修改 poller 表配置行 → 实时图端点读取被篡改配置并传入popen()  
执行，全程仅需**4 个 HTTP 请求**  
；  
- **影响版本**  
：已确认**Cacti Core 1.2.31**  
（当前最新版）；共用相同资料页与实时 poller 代码的更早 1.2.x 版本可能同样受影响，尚未逐一排除；  
- **补丁状态**  
：**官方尚未发布修复版本**  
（等待 1.2.32）  
- **执行权限**  
：命令以 Cacti 进程运行用户权限执行，实验室演示拿到 root shell 是因为测试 Web 层以 root 运行，常规 Linux 主机下为www-data  
/apache  
等 Web 用户  
- **横向价值**  
：Cacti 普遍保存路由器、交换机、防火墙、主机的 SNMP community、SSH凭据与数据库登录信息，监控服务器常被全网信任、审计薄弱，失陷后可作为横向跳板。  
> 紧急动作：尚未关闭 guest 访客账号的实例，应先在 Cacti 用户管理中禁用 guest，并把 Web 门户收敛到可信管理网段，不要等待补丁。  
  
## 二、漏洞基本信息  
<table><thead><tr><th><span leaf="">项目</span></th><th><span leaf="">内容</span></th></tr></thead><tbody><tr></tr><tr><td><span leaf="">研究别名</span></td><td><span leaf="">4HTTP2ROOT / 4HTTP2SHELL</span></td></tr><tr></tr><tr><td><span leaf="">受影响产品</span></td><td><span leaf="">Cacti Core（开源网络监控框架）</span></td></tr><tr><td><span leaf="">确认受影响版本</span></td><td><span leaf="">1.2.31（最新发布版，2026 年 6 月发布）</span></td></tr><tr><td><span leaf="">可能受影响版本</span></td><td><span leaf="">共用相同</span><code><span leaf="">auth_profile.php</span></code><span leaf="">/ 实时 poller 代码的更早 1.2.x 版本（待官方逐一确认）</span></td></tr><tr><td><span leaf="">漏洞类型</span></td><td><span leaf="">二阶 SQL 注入 → 配置篡改 → 操作系统命令注入（CWE-89 SQL 注入；CWE-78 OS 命令注入）</span></td></tr><tr><td><span leaf="">前置条件</span></td><td><span leaf="">① guest 访客账号处于启用状态；② Web 界面对攻击者可达；③ 对应配置代码路径可被访客触达</span></td></tr><tr><td><span leaf="">认证要求</span></td><td><strong><span leaf="">无需认证</span></strong><span leaf="">（guest 会话即入口）；无需用户交互</span></td></tr><tr><td><span leaf="">利用复杂度</span></td><td><span leaf="">链路仅 4 个请求；研究侧已公开完整技术说明与 PoC</span></td></tr><tr><td><span leaf="">补丁状态</span></td><td><strong><span leaf="">无官方补丁</span></strong><span leaf="">；跟踪 Cacti GitHub 后续 1.2.32 发布</span></td></tr><tr><td><span leaf="">在野状态</span></td><td><span leaf="">截至 2026-11-27 未确认生产环境在野利用；公开 PoC 已落地，需按即将被批量扫描处置</span></td></tr><tr><td><span leaf="">发现 / 披露</span></td><td><span leaf="">研究者 ek0ms savi0r；watchTowr 情报通告；Previdian 实验室复现验证；2026-11-26 公开</span></td></tr></tbody></table>### 版本与处置状态对照  
<table><thead><tr><th><span leaf="">版本分支</span></th><th><span leaf="">状态</span></th><th><span leaf="">处置口径</span></th></tr></thead><tbody><tr><td><span leaf="">1.2.31（最新）</span></td><td><span leaf="">确认受影响（guest 启用时）</span></td><td><span leaf="">无补丁；禁用 guest + 网络收敛 + 排查 poller 配置，等待 1.2.32</span></td></tr><tr><td><span leaf="">1.2.x 更早版本</span></td><td><span leaf="">可能受影响，未逐一排除</span></td><td><span leaf="">同上；同时评估是否已含 1.2.31 之前的历史修复</span></td></tr><tr><td><span leaf="">非 1.2.x 分支</span></td><td><span leaf="">待官方确认</span></td><td><span leaf="">暂按版本测绘结果跟踪公告</span></td></tr></tbody></table>> 背景对照：2026 年上半年 1.2.31 已集中修复一批历史缺陷（未授权 LFI/RCE、未授权 SQL 注入、RRDtool 命令注入、存储型 SQL 注入、会话固定等，对应 CVE-2026-39938 / 39893 / 40079 / 39902 / 39951 / 40082 等）；本次 4HTTP2ROOT 是 1.2.31 之上新披露的未修复缺陷，**升级到 1.2.31 并不能覆盖本漏洞**  
。  
  
## 三、背景：Cacti 为何是高价值跳板目标  
  
Cacti 是被广泛部署的开源网络监控平台，通过 SNMP、脚本、数据采集轮询器（poller）持续采集路由器、交换机、防火墙、服务器等设备的性能指标。其高风险属性在于：  
- **凭据集中**  
：平台内保存被监控设备的 SNMP community、SSH 凭据、数据库登录串、数据源连接信息；  
- **信任位置**  
：监控服务器通常位于内网受信区域，可直连核心网络设备，却常因"内部工具"而审计薄弱；  
- **执行面多**  
：poller 以脚本 / 命令方式采集数据，Web 侧与后台轮询器共享数据库配置；  
- **历史包袱**  
：近年多次出现未授权 SQL 注入、命令注入、LFI 类漏洞，guest 访客图表面向匿名开放的部署方式进一步放大了入口风险。  
研究者在公开分析中指出：在不少组织里，Cacti 服务器"被全网信任，却几乎无人审计"，其价值往往不低于域控。  
## 四、根因与攻击链路拆解  
### 4.1 四个被串联的信任缺口  
  
研究者将漏洞归因于四个先后失效的校验环节：  
1. **访客可把数值型设置写成任意字符串**  
：preview_graphs_per_page  
（预览模式每页展示图数）本应为整数，但 guest 可在资料更新接口写入非数字文本；  
1. **存储默认值绕过请求侧校验**  
：应用在请求阶段未校验输入是否为整数，存入数据库后，后续读取该存储值时又跳过了请求侧数字检查；  
1. **PDO 驱动经 LIMIT 子句接受堆叠语句**  
：存储的文本被直接拼入 SQLLIMIT  
子句，数据库驱动支持堆叠查询，从而执行额外 SQL；  
1. **poller 执行数据库中存储的命令字符串**  
：实时 poller 读取 poller 表配置行，将其中命令值直接交给 PHPpopen()  
，无白名单、无过滤。  
### 4.2 四步攻击链路（公开口径还原）  
```
未认证攻击者（全新 HTTP 会话，guest 入口）
   │
   │ ① GET 用户资料页
   │    取得会话 Cookie 与 CSRF Token（guest 身份）
   ▼
② POST auth_profile.php?action=update_data
   │    将任意 SQL 文本写入数值设置 preview_graphs_per_page
   │    （未做整数校验，文本落库 settings_user）
   ▼
③ GET graph_view.php（图表面板，默认过滤器）
   │    读取 guest 存储设置 → 跳过数字校验
   │    文本拼接进 SQL LIMIT 子句；PDO 堆叠语句
   │    执行 UPDATE，篡改 poller 表中一条配置记录（poller_item）
   ▼
④ GET graph_realtime.php（实时图端点，同一 guest 会话）
   │    实时 poller 读取被篡改的 poller 配置行
   │    命令值直接传入 popen()，无白名单/转义
   ▼
以 Cacti 进程权限执行操作系统命令 → 站点接管 / 横向内网
```  
### 4.3 关键接口与数据表（狩猎与排查定位用）  
<table><thead><tr><th><span leaf="">环节</span></th><th><span leaf="">接口 / 位置</span></th><th><span leaf="">关联数据表</span></th><th><span leaf="">说明</span></th></tr></thead><tbody><tr><td><span leaf="">会话与 Token</span></td><td><span leaf="">用户资料页（auth profile 页面）</span></td><td><span leaf="">session</span></td><td><span leaf="">访客获取会话 Cookie、CSRF Token</span></td></tr><tr><td><span leaf="">配置写入</span></td><td><code><span leaf="">auth_profile.php?action=update_data</span></code></td><td><code><span leaf="">settings_user</span></code></td><td><span leaf="">写入</span><code><span leaf="">preview_graphs_per_page</span></code><span leaf="">非预期文本</span></td></tr><tr><td><span leaf="">二阶触发</span></td><td><code><span leaf="">graph_view.php</span></code><span leaf="">（默认过滤器）</span></td><td><code><span leaf="">settings_user</span></code><span leaf="">→</span><code><span leaf="">poller_item</span></code></td><td><span leaf="">存储值拼入 LIMIT，堆叠 UPDATE 篡改 poller 行</span></td></tr><tr><td><span leaf="">命令执行</span></td><td><code><span leaf="">graph_realtime.php</span></code><span leaf="">（实时图端点）</span></td><td><code><span leaf="">poller_item</span></code></td><td><span leaf="">poller 读取配置行 →</span><code><span leaf="">popen()</span></code><span leaf="">执行</span></td></tr></tbody></table>### 4.4 根因归类  
- **CWE-89**  
：SQL 命令中特殊元素未正确中和（二阶 / 存储型 SQL 注入）；  
- **CWE-78**  
：OS 命令中特殊元素未正确中和（poller 配置值进入popen()  
）；  
- 衍生问题：数值型配置缺少服务端类型校验；数据库驱动堆叠语句未在业务层禁用；后台执行面直接消费数据库内命令串，缺少命令白名单与权限隔离。  
### 4.5 危害面  
- **机密性**  
：可读取监控平台内设备凭据、SNMP community、拓扑与业务指标数据；  
- **完整性**  
：可篡改 poller 采集配置、抑制告警、植入常驻采集项；  
- **可用性**  
：可破坏监控数据采集与告警链；  
- **横向**  
：以 Cacti 进程为跳板，使用窃取的设备凭据访问路由器、交换机、防火墙与主机。  
## 五、在野威胁情报与暴露态势  
- **披露时间**  
：2026-11-26 研究者公开完整技术说明、演示视频与 PoC；watchTowr 发布情报通告，Previdian 在实验室完成复现；  
- **在野情况**  
：截至 2026-11-27，未收到生产环境被利用的确认报告；公开 PoC 落地后，面向互联网开放、guest 启用的实例预计会被快速批量扫描；  
- **流量特征**  
：Web 侧访问形态接近正常 guest 浏览流量，日志中可能仅留下graph_view.php  
非数值相关的 PHP 警告，隐蔽性较强；  
- **情报跟踪重点**  
：Cacti 官方 GitHub 仓库的安全公告与 1.2.32 发布；CVE 编号分配；IOC 与批量扫描特征。  
## 六、检测与威胁狩猎  
### 6.1 资产与版本排查  
1. 盘点全部 Cacti 实例（Web 站点、容器化部署、监控集群），记录版本、guest 账号状态、Web 暴露范围、责任人与变更窗口；  
1. 核对版本：界面**About**  
页面或cacti_version  
相关文件 / 包信息；重点标记 1.2.31 及更早期 1.2.x；  
1. 确认 guest 访客账号是否启用、Web 界面是否对公网或不可信网段开放。  
### 6.2 配置与数据库核查  
  
在 Cacti 数据库中核查两张关键表是否存在异常字符串或被篡改的命令值：  
- settings_user  
：检查访客 / 用户设置项，尤其preview_graphs_per_page  
是否为非数值内容；  
- poller_item  
：检查采集项命令、参数字段是否存在非预期命令拼接、异常二进制路径或陌生字符串。  
> 核查动作应在受控窗口内执行，先备份相关表再比对基线；发现可疑记录不要直接在线删除，先固定证据。  
  
### 6.3 网络与日志侧  
- 访问日志检索对以下端点的匿名 / guest 会话访问：auth_profile.php  
、graph_view.php  
、graph_realtime.php  
，尤其update_data  
动作与实时图请求；  
- PHP / Web 错误日志关注graph_view.php  
关于非数值（non-numeric）取值的警告；  
- 排查 guest 会话下异常的 profile 更新、poller 配置变更记录；  
- 监控 Cacti 进程向外发起的异常命令执行、对外连接、陌生子进程与临时文件落地。  
### 6.4 ATT&CK 映射参考  
- T1190 Exploit Public-Facing Application（Web 入口利用）  
- T1078 Valid Accounts（滥用 guest 访客身份）  
- T1190 / T1059：二阶 SQL 注入篡改配置后落地命令执行  
- T1552 Unsecured Credentials（收集设备 SNMP / SSH / 数据库凭据）  
- T1021 横向远程服务（借监控服务器信任位置访问核心网络设备）  
### 6.5 失陷初判信号  
- guest 账号被启用且 Web 对外可达；  
- settings_user  
中出现非数值的preview_graphs_per_page  
；  
- poller_item  
出现无基线对应的命令 / 参数；  
- graph_view.php  
、graph_realtime.php  
存在异常匿名请求；  
- Cacti 进程出现陌生子进程、外联或可疑文件；监控告警被静默或采集项被改动。  
## 七、授权复测概念验证框架（不含武器化利用代码）  
### 7.1 复测环境要求  
- 独立隔离网络中的 Cacti 实验实例，仅授权测试人员可达；  
- 记录基线版本、guest 账号状态、settings_user  
与poller_item  
基线快照；  
- 全程留存 HTTP 会话、数据库变更与进程日志，便于复盘。  
### 7.2 四阶段验证框架  
<table><thead><tr><th><span leaf="">阶段</span></th><th><span leaf="">验证目标</span></th><th><span leaf="">观测点（成功 / 失败信号）</span></th></tr></thead><tbody><tr><td><span leaf="">① 会话建立</span></td><td><span leaf="">未认证访客能否取得有效会话 Cookie 与 CSRF Token</span></td><td><span leaf="">取得 guest 身份会话；若 guest 已禁用则该阶段失败（链条断开）</span></td></tr><tr><td><span leaf="">② 配置写入</span></td><td><span leaf="">数值型设置是否接受非整数文本落库</span></td><td><span leaf="">查表确认</span><code><span leaf="">preview_graphs_per_page</span></code><span leaf="">被写入非预期内容；若服务端强制整数校验则失败</span></td></tr><tr><td><span leaf="">③ 二阶 SQL 触发</span></td><td><span leaf="">存储值进入 SQL 后能否触发堆叠语句并篡改 poller 行</span></td><td><span leaf="">观察</span><code><span leaf="">poller_item</span></code><span leaf="">目标行是否被修改；日志 / 慢查询中出现堆叠语句特征；若驱动禁用堆叠或参数化绑定则失败</span></td></tr><tr><td><span leaf="">④ 命令执行</span></td><td><span leaf="">实时 poller 是否读取被篡改配置并进入</span><code><span leaf="">popen()</span></code></td><td><span leaf="">在隔离环境用无害标记命令验证执行回显；确认执行身份（Web 用户 / root）；命令执行需在授权范围内进行</span></td></tr></tbody></table>### 7.3 复测后核查  
- 验证完成后恢复基线配置，重置 guest 与 poller 配置；  
- 核对实验环境未残留可疑采集项、文件与外联记录；  
- 将复测中观测到的请求特征、日志告警映射到第六节检测规则。  
## 八、处置与缓解建议（按优先级）  
### 8.1 第 1 优先：断开链路入口（补丁前立即执行）  
1. **禁用 guest 访客账号**  
：在 Cacti 用户管理 / 认证设置中删除或禁用指定 Guest User，使匿名会话无法建立、无法保存资料字段；业务确需对外看图的，改为认证访问；  
1. **网络收敛**  
：将 Cacti Web 门户置于 VPN 或可信管理 VLAN 之后，防火墙 / 反向代理仅放行管理网段；禁止公网直接暴露；  
1. 禁用 guest 后，从非信任网段实测确认匿名访问被拒绝（401/403/404）。  
### 8.2 排查与补偿防护  
1. 备份并核查settings_user  
、poller_item  
表，清理异常命令值与非预期采集项；  
1. 确认 Web 进程与 poller 进程**不以 root 运行**  
，按最小权限配置运行用户；  
1. 监控graph_view.php  
非数值警告、graph_realtime.php  
异常访问与进程外联；  
1. 审计 Cacti 保存的设备凭据，必要时轮换 SNMP community、SSH 与数据库口令；  
1. 关注 Cacti 官方 GitHub 安全公告，1.2.32 修复版本发布后第一时间升级。  
### 8.3 补丁后复核  
- 升级后回归验证：guest 默认不可用、数值设置服务端类型校验生效、poller 配置命令走白名单 / 严格转义；  
- 复核网络收敛策略与权限隔离仍然生效；  
- 将本实例纳入周期漏洞核查与资产台账。  
### 8.4 疑似失陷处置  
1. 先隔离网络、保留日志与快照，再处置；  
1. 从可信备份恢复poller_item  
、配置文件与数据库；  
1. 轮换数据库口令、hash_salt  
、Cacti 保存的全部设备凭据与密钥；  
1. 排查 Web 目录、sites/*/files  
、poller 执行目录中的可疑脚本与常驻采集项；  
1. 以 Cacti 服务器为起点，向内网设备与主机做横向排查。  
## 九、风险研判小结  
- **现实风险**  
：未认证 guest 入口 + 二阶 SQL 注入 + poller 命令执行 + 公开 PoC，监控服务器又是内网高信任跳板，属当前需紧急处置的 Web 高危项；  
- **重点暴露面**  
：对公网 / 不可信网段开放且启用 guest 的 Cacti 实例、1.2.31 及早期 1.2.x、以 root 运行 Web/poller 的部署；  
- **注意盲区**  
：无官方补丁不等于只能被动等待——禁用 guest 与网络收敛可直接切断链路前置条件；guest 流量外观接近正常浏览，不能只靠访问日志肉眼判断；  
- **研判口径**  
：先按"版本 + guest 状态 + 暴露面"分级，对外暴露与 guest 启用实例优先处置；链路切断、配置排查、凭据轮换与补丁跟踪同步推进。  
