#  五个洞不是五个漏洞：GeoServer 未认证 RCE 链把补丁打穿了  
 TJXFire   2026-11-26 06:59  
  
全文约 13700 字，预计阅读 40 分钟。本文为防守向拆解，**不提供任何攻击 EXP、利用代码或可直接运行的触发载荷**  
。  
  
这条链最该被记住的，是**"五个洞不是五个漏洞"**  
。官方公告写得很清楚：前四步单独存在时只是信息泄露或权限绕过，第五步需要管理员权限才能投递数据，**而第四步恰好提供了伪造的管理员会话**  
——五处在默认配置下环环相扣、缺一不可。**只修补其中一环，不足以消除风险。**  
  
更要紧的是，**这一次"修补"不是一个动作，是一次版本升级**  
。同一次发布的 3.0.2 / 2.28.6 / 2.27.7，实际上一次性关掉了 **17 个 CVE**  
（其中 5 个构成这条链），另有 **1 条独立的 RCE 路径**  
（CVE-2026-120704）也能达成未认证执行。**你不需要打五个补丁，你只需要升一个版本。**  
  
还有一条最反直觉、也最该被写进运维手册的结论：**"改掉 remember-me 密钥"这个动作在这条链上是无效的**  
。官方 GHSA 原文写得很直白——签名密钥从数据目录的 **security/config.xml**  
 读取，发行包自带的默认值是公开可查的固定字符串；而同一条链的第三环恰好能读出这个文件。同一次文件泄露会同时暴露密钥和口令摘要，所以单独改密钥拦不住任何人。  
  
11 月下旬，GeoServer 的一个完整利用链被摆到了台面上。  
  
先看时间线，这件事的节奏比多数漏洞更值得注意：  
  
**8 月 2 日**  
，研究者向 GeoServer 团队提交私有安全公告并附上完整 PoC；**8 月 10 日**  
官方确认；  
  
**10 月 17 / 18 日、11 月 2 日**  
，修复版本 3.0.2、2.27.7、2.28.6 陆续发布——**走的是先修后曝**  
；  
  
**11 月 7 日**  
，原始报告被拆成 5 份独立公告；**11 月 16 日**  
，CVE 编号获分配；11 月 23 日，官方五份 GHSA 公告与一份总更新页同时公开；  
  
**11 月 24 日**  
，研究者公开技术细节与 PoC；**11 月 24—25 日**  
，WatchTowr 在蜜罐网络上观测到针对其中两条漏洞的实际探测；  
  
**11 月 25 日**  
，奇安信 CERT、长亭安全应急响应中心、绿盟科技 CERT 分别发布通告，CNVD 同日收录 5 个漏洞并发布安全公告 CNTA-2026-0008。  
  
也就是说，**从补丁发布到攻击者开始敲门，中间有 38 天**  
。这 38 天里你的 GIS 平台补没补，完全取决于你有没有把 GeoServer 的版本升级纳进常规巡检。  
  
本文要做五件事：一是把事实底座摆齐，包括**五份官方公告各自的评分向量、修复版本、缓解措施**  
，以及**四家国内机构通告之间三处互相打架的口径**  
；二是讲清楚漏洞原理，讲到"为什么第五步需要管理员权限、而第四步恰好能凭空造出一个管理员会话"这一层为止；三是把"处置单位是版本、不是补丁"这件事单独拎出来说，这是本期最该被记住的部分；四是给出一张盘点表和一张核对单——因为这条链真正的难题从来不是"怎么修"，而是"你根本不知道自己部署了什么版本的 GeoServer，而且它多半不是 GIS 部门在管"；五是给江西政企客户一条能直接挂到现有合规动作上的抓手。  
  
**本文不提供任何攻击 EXP、利用代码或触发载荷构造。所有评分口径、版本号、时间点与在野利用状态均标注发布主体，官方未公开的一律标注"未公开"。**  
  
   
## 一、事件档案  
  
下表汇总本次漏洞链的关键档案信息，涵盖编号、评分口径、披露时序、修复状态与官方声明情况。  
<table><caption></caption><colgroup><col style="width:16%;"/><col style="width:84%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">项目</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">内容</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">漏洞名称</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">GeoServer 未认证远程代码执行漏洞链（5 个独立 CVE 组合利用）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">漏洞编号</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">CVE-2026-129244</span></strong><section><span leaf="">（GHSA-4v26-c537-jvxc）、CVE-2026-129243（GHSA-rv5g-g2r9-58rj）、CVE-2026-120705（GHSA-39mw-5chv-jh47）、CVE-2026-129245（GHSA-hgm3-wq4h-8vvq）、CVE-2026-129242（GHSA-9wf2-fcgq-vxhq）。CNVD 同步收录 5 条：CNVD-2026-47961 / 47960 / 47959 / 47958 / 47957（对应上述顺序）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">未认证远程代码执行。</span><strong><span leaf="">根因是五处独立缺陷在默认配置下可被串联</span></strong><span leaf="">，非单一漏洞</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">位置</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">WFS 存储查询与服务调度器、存储查询 XML 二次解析、remember-me 签名密钥、JDBC 用户/角色服务</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">评分口径</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">完整链 CVSS 3.1 = 9.8</span></strong><section><span leaf=""> 由奇安信 CERT给出（其通告同时记高危评级、利用可能性高）。官方单条评分（GeoServer 官方 GHSA，均为 CVSS 4.0，除注明者外）：CVE-2026-129245 9.2 Critical、CVE-2026-129243 8.8 High、CVE-2026-129242 8.6 High、CVE-2026-129244 6.9 Moderate；CVE-2026-120705 官方给 CVSS 3.1 = 8.6 High。CNVD 对五个漏洞的综合评级为&#34;高危&#34;</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">认证要求</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">整条链无需任何账号或凭据</span></strong><section><span leaf="">；无需用户交互；默认配置即可触发（官方公告明确写 &#34;default configuration&#34;）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">网络前提</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">目标 GeoServer </span><strong><span leaf="">网络可达</span></strong><span leaf="">，且默认开放的 WFS 服务入口与 REST 接口可达</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">执行权限</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">以 </span><strong><span leaf="">GeoServer 服务进程的操作系统权限</span></strong><span leaf="">执行命令。官方容器镜像默认以 root 运行（CNVD 与长亭通告口径）——这是官方镜像的默认部署形态，不代表生产环境必然为 root，实际取决于容器权限配置。以普通用户身份部署时，影响受进程权限约束</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">披露时序</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">报告厂商 </span><strong><span leaf="">2026-08-02</span></strong><span leaf=""> → 厂商确认 08-10 → 修复版本发布 10-17（3.0.2）/ 10-18（2.27.7）/ 11-02（2.28.6） → 拆分 5 份公告 11-07 → CVE 编号分配 11-16 → 官方公告公开 11-23 → 研究者公开技术细节与 PoC 11-24 → WatchTowr 观测到探测 11-24—25 → 国内三家机构通告与 CNVD 收录 11-25</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">修复版本</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">GeoServer 3.0.2 / 2.28.6 / 2.27.7</span></strong><section><span leaf="">。CVE-2026-120705 另需配套升级 GeoTools 33.7 / 34.6 / 35.2（官方 GHSA 明确列出 gt-metadata 的受影响与修复版本）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">官方声明状态</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">五份 GHSA 公告均由 GeoServer 协调人于 2026-11-23 发布</span></strong><section><span leaf="">；同日官方站点发布总更新页，公布该批修复涉及的全部 CVE</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">在野利用状态</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">已观测到在野探测，未确认成功利用</span></strong><section><span leaf="">。WatchTowr 观测到针对 CVE-2026-120705 与 CVE-2026-129243 的错误型探测请求（创建并调用存储查询以验证目标是否脆弱），明确未报告任何成功实现 RCE 的案例。奇安信 CERT 通告记&#34;在野利用状态：未发现&#34;，绿盟科技 CERT 通告记&#34;在野利用 未发现&#34;</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">KEV 收录状态</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">截至 catalogVersion 2026.11.25（1805 条），本链 5 个 CVE 均未收录</span></strong><section><span leaf="">。KEV 中 GeoServer 相关历史条目为 CVE-2025-58360、CVE-2024-36401、CVE-2022-24816（JAI-EXT）三条，均与本链无关。本链不存在联邦强制整改截止日期</span></section></td></tr></tbody></table>  
档案里有五行需要单独强调。  
  
**第一行是"漏洞编号"这一行——它有 10 个编号。**  
 5 个 CVE、5 个 GHSA 公告、5 个 CNVD 编号。这不是编号冗余，而是官方在 CVE Numbering Authority 的要求下，**把一条链上的每一步都拆成了独立条目分别修复、分别发布**  
。这带来一个直接的运维后果：你在漏洞库里按产品搜，能搜到 5 条彼此看不出关系的记录；任何一条单独的告警，看起来都只是一个"中危路径泄露"或"高危权限绕过"。只有把它们拼起来，才看得出这是一条完整的未认证 RCE 链。  
  
**第二行是"评分口径"这一行里的三个分。**  
 9.8 是**奇安信 CERT 对完整链**  
给出的 CVSS 3.1；9.2 是官方对 CVE-2026-129245 单条给出的 CVSS 4.0；8.8、8.6、6.9 是官方对另外三条单条给出的分值。这四个数字不是同一把尺子量的，不能互相比较、也不能相加。特别提醒一句：CVSS 3.1 与 CVSS 4.0 的向量结构不同（4.0 增加了 Attack Requirements 维度），跨版本比较分值本身没有意义。  
  
**第三行是"披露时序"这一行里的三个日期簇。**  
 厂商确认在 8 月，修复版本在 10 月中旬到 11 月初，官方公告在 11 月 23 日。三者**不可互换**  
：把 8 月 2 日写成"漏洞公开日"、把 10 月 17 日写成"发现日"，都是把机构动作当成了事件时间。  
  
**第四行是"修复版本"这一行里的 GeoTools。**  
 这一条极容易被漏掉——**CVE-2026-120705 的官方修复方案同时覆盖了 org.geotools:gt-metadata 这个包**  
，受影响版本为 ≤33.6、≥34.0 且 ≤34.5、≥35.0 且 ≤35.1，修复版本为 33.7 / 34.6 / 35.2。如果你的 GeoServer 是自带 GeoTools 的官方发行包，升 GeoServer 即可；如果你用的是自己拼装的部署包，只升 GeoServer 不升 GeoTools，这个洞仍然在。  
  
**第五行是"在野利用状态"这一行。**  
 这是全篇最需要说准的一行。**探测已经开始，成功利用尚未被公开确认**  
——这两件事必须分开写，第三章会展开。  
  
   
## 二、漏洞原理：五处缺陷不是五个漏洞  
### 2.1 GeoServer 是什么，以及为什么它在政企场景里到处都是  
  
**GeoServer**  
 是 OSGeo 维护的开源地理空间服务端（Java 实现），按 OGC 标准对外提供 **WMS**  
（地图服务）、WFS（Web 要素服务）、WCS（网络 Coverage 服务）等接口。奇安信 CERT 通告把它点得很准："常见于政务、测绘、自然资源及行业 GIS 平台的数据发布环节"。CNVD 的表述是"提供政务、测绘、自然资源以及行业 GIS 平台的图层空间数据发布等功能，在政府与行业地理信息基础设施中部署较广"。  
  
它的部署形态通常由三部分组成：**Web 管理台**  
、**REST 管理接口**  
、本地 data_dir 数据目录，再加上插件体系。  
  
有一个业务侧的事实必须先摆出来，因为它决定了后面所有防护动作的可行性：**WFS 存储查询（Stored Query）默认配置下把 CreateStoredQuery 与 DropStoredQuery 限制为管理员操作，而匿名用户默认可以调用 GetFeature。**  
 换句话说，**这条链的起点是一个很多业务系统本来就在用的开放入口**  
——大量政企单位的地图业务系统对接图层，走的就是匿名 WFS 查询。  
### 2.2 第 1 环（CVE-2026-129244，6.9 Moderate）：超长标识把绝对路径吐回给客户端  
  
存储查询 ID 会被映射成数据目录下的一个文件名。**当这个 ID 超过文件系统名称长度限制时，存储查询请求会返回一个异常报告，而异常里带着完整的绝对路径。**  
>   
> "An unauthenticated user learns the absolute data directory path. This is not harmful by itself, but other attacks need it, for example to build `file:` references to data directory files."  
>   
> 单独看它无害，它的价值在于给后面几环提供坐标。  
  
  
这一环的官方缓解措施是 **"None needed beyond upgrading"**  
（除升级外无需额外操作），修复方式是：**超长存储查询 ID 现在被直接拒绝并返回简短错误，文件系统错误改为只写日志、不再回显给客户端。**  
### 2.3 第 2 环（CVE-2026-129243，8.8 High）：x/WFS —— 规则匹配的是另一个名字  
  
这是整条链里最值得防守方记住的一环，因为它**不需要任何精巧技巧，只需要一个斜杠**  
。  
  
请求里的 service 参数如果带路径前缀（比如 **service=x/WFS**  
），**调度器会把 / 之前的部分剥掉，用 WFS 选中服务；但服务安全检查用的却还是未解析的 x/WFS**  
。于是——没有一条规则能匹配上这个名字，本该受限的操作就绕过了角色检查。  
>   
> "The default configuration restricts CreateStoredQuery and DropStoredQuery to administrators, so anonymous users could create and delete WFS stored queries."  
>   
> "Installations that restrict other operations this way, for example WFS Transaction, lose that protection as well."  
  
  
**注意最后那句**  
：如果贵单位用服务安全规则限制了 WFS Transaction（WFS 事务写入），**这条绕过同样让该限制失效**  
。而图层与数据的安全规则仍然有效——所以这条环绕的是"服务级"规则，不是"图层级"规则。  
  
官方给出的缓解措施有两条，都很具体：**在 GeoServer 前面的代理上，阻断 service 参数中含 / 的请求**  
（无论在查询串里还是在 XML 请求的 service 属性里）；**审查数据目录下的 wfs/query 与 workspaces/*/wfs/query 目录，找出你自己没有创建过的存储查询。**  
### 2.4 第 3 环（CVE-2026-120705，8.6 High）：一次未认证请求，读文件、发外连  
  
GeoServer 通过公开的 GetFeature 操作暴露了内置的 WFS 2.0 存储查询 GetFeatureById。这个存储查询把调用方传入的 ID 参数**未经转义直接替换进一个 XML 模板**  
，然后**在没有使用 GeoServer 实体解析器的情况下解析结果**  
。  
  
官方 GHSA 的 Impact 段把后果写成了三条，本条链的所有后续环节都建立在它们之上：  
  
**读取 GeoServer 进程可读的本地文本文件**  
；  
  
**抓取攻击者指定的 HTTP 或 HTTPS 地址（服务端请求伪造，SSRF）**  
；  
  
**通过 Illegal property name 错误响应把抓到的文本回显出来**  
。  
  
官方特别注明：**回显与否取决于图层是否对匿名用户发布**  
——即使不回显，**文件仍然被读取了**  
。这一条区分很重要："响应里没看到内容"不等于"没被读走"。  
  
这一环的官方缓解措施是：**用服务安全限制或禁用匿名用户的 WFS 服务；如果不需要 WFS，直接禁用它。**  
 修复方案为**存储查询解析改用 GeoServer 实体解析器，并且 GeoTools 的 PreventLocalEntityResolver 现在会检查重定向**  
。  
### 2.5 第 4 环（CVE-2026-129245，9.2 Critical）：默认密钥是公开的  
  
**这一环是全链最高分，也是最反直觉的一环。**  
  
remember-me Cookie 用一个密钥签名，签名内容包含用户名、过期时间和**存储的口令摘要**  
（不是口令本身）。这个密钥从数据目录的 **security/config.xml**  
 读取，而发行版数据目录自带的密钥是一个公开可查的固定值，官方 GHSA 原文写的是 "the release data directory ships with the key geoserver, which most installations never change"  
——绝大多数安装从未改过它。  
  
于是第 3 环读出 security/config.xml 的那一刻，**签名密钥和口令摘要一起被读走了**  
。  
  
**这里必须把官方那句最狠的话原样讲出来**  
："Changing the key in security/config.xml does not help when the same file disclosure also reveals the key."  
——**改密钥这个动作，在这条链上是无效的**  
，因为同一次文件泄露会同时暴露密钥本身。奇安信 CERT 的通告也用了同一个定性："仅修复其中任一环节虽可提高利用门槛，但不足以从整体上消除该组合利用风险。"  
  
这一环的官方缓解措施有两条，优先级是明确的：  
  
**首选：从 Web 过滤器链里移除 rememberme**  
（管理界面路径：**Security > Authentication**  
，过滤器链 web）；  
  
**其次：把 security/config.xml 里的密钥改成一个足够长的随机值，并保护数据目录不被泄露**  
。  
  
官方修复后的行为也值得记下来：**密钥不再存放在数据目录中，每个节点使用随机密钥，升级会使已有的 remember-me Cookie 全部失效，集群部署通过 GEOSERVER_REMEMBERME_KEY 应用属性设置共享密钥。**  
### 2.6 第 5 环（CVE-2026-129242，8.6 High）：反序列化执行  
  
**JDBCUserGroupService**  
 与 **JDBCRoleService**  
 用 ResultSet.getObject() 读取用户和角色的属性值。对 HSQLDB 的 OTHER 类型列，驱动会用不受限制的 ObjectInputStream 反序列化该值。  
  
官方公告指出两个让这一环成立的前提：**JDBC 安全模块与 HSQLDB 驱动都在标准发行包里**  
，而且 **REST API 允许管理员上传数据库文件到数据目录、并创建一个指向它的 JDBC 用户组服务**  
。  
  
所以这一环**要求管理员权限**  
——而**第 4 环恰好凭空造出了一个管理员会话。**  
 官方对第 5 环的 Impact 定性是："An administrator, or anyone who takes over an administrator session, can execute code with the operating system privileges of the GeoServer process."  
  
这一环的官方缓解措施是：**限制管理员访问给可信用户，并把 REST API 的 /rest/resource 与 /rest/security 端点限制在可信网络内。**  
 修复方式是**用户和角色的属性值现在按文本读取，无需任何手工操作。**  
### 2.7 为什么"只修一个"等于没修  
  
把五环串起来看，这是一条设计得相当干净的攻击链：  
<table><caption></caption><colgroup><col style="width:26%;"/><col style="width:44%;"/><col style="width:30%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">环节（CVE · 官方评分）</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">缺陷</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">单独存在时的后果</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">1</span></strong><section><span leaf=""> CVE-2026-129244</span><span leaf=""><br/></span><span leaf="">6.9 Moderate</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">WFS 存储查询超长标识缺少长度限制，异常回显完整文件系统路径</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">泄露数据目录绝对路径</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">2</span></strong><section><span leaf=""> CVE-2026-129243</span><span leaf=""><br/></span><span leaf="">8.8 High</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">服务调度器对带路径分隔符的服务名截断解析，访问控制规则仍按原始值匹配</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">匿名用户可创建/删除存储查询</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">3</span></strong><section><span leaf=""> CVE-2026-120705</span><span leaf=""><br/></span><span leaf="">8.6 High</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">存储查询文本重载时二次解析，实体解析允许本地文件协议</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">读本地文件 + SSRF + 错误回显</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">4</span></strong><section><span leaf=""> CVE-2026-129245</span><span leaf=""><br/></span><span leaf="">9.2 Critical</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">remember-me 签名密钥沿用固定默认值</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">配合泄露的口令摘要伪造管理员会话</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">5</span></strong><section><span leaf=""> CVE-2026-129242</span><span leaf=""><br/></span><span leaf="">8.6 High</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">JDBC 用户服务以对象方式读取用户属性，底层驱动无过滤反序列化</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">以进程权限执行任意命令</span></section></td></tr></tbody></table>  
**链的逻辑是这样的**  
：第 1 环给出路径坐标 → 第 2 环让匿名用户拿到"创建存储查询"这个能力 → 第 3 环读出安全配置文件 → 第 4 环用公开的固定密钥 + 读到的口令摘要伪造管理员会话 → 第 5 环以管理员身份投递反序列化载荷，执行命令。  
  
**关键在第 4 与第 5 环的咬合**  
：第 5 环本身要求管理员权限，这不是缺陷，是设计——**JDBC 用户服务本来就该只有管理员能配。正因为它设计得对，它成了整条链的收口点；而第 4 环的存在，让"我是管理员"这件事变得不再需要真的拿到管理员口令。**  
  
所以官方的结论才是那句重话：**"四处铺路，第五处完成命令执行——五者在默认配置下环环相扣，缺一不可，只修补其中一环不足以消除风险。"**  
### 2.8 一条必须说清的边界：这还不是"已经被拿下"  
  
这一节单独开，是因为这条链最容易被写成"已被大规模利用"，而事实不支持。  
  
**已确认的事实**  
（第三方观测口径）：WatchTowr 通过其蜜罐网络观测到攻击者针对 **CVE-2026-120705**  
 与 CVE-2026-129243 发起请求，其公开表述是攻击者发送了错误型探测请求，通过创建并调用存储查询来验证服务器是否脆弱。  
  
**WatchTowr 同时明确表示：未报告任何攻击者成功实现远程代码执行的案例。**  
 SecurityWeek 的报道也写明，截至其发稿时**没有其他利用尝试的报告**  
。  
  
**未确认的部分**  
：攻击者身份、归属、动机——**无任何机构作出归因，本文不作任何暗示**  
；实际受害规模——未公开；是否存在成功利用的案例——截至本文写作时无公开确认。  
  
**因此本文的表述是：探测已经开始，成功利用尚未被公开确认。**  
 这两句必须一起讲。只讲前半句会制造恐慌且缺乏依据，只讲后半句会让人以为还没轮到自己是错的——**探测阶段恰恰是防守窗口最宽的时候**  
，因为此时攻击者还在筛目标。  
  
   
## 三、处置断层：处置单位是版本，不是补丁  
### 3.1 第一层：修复版本早于公告 38 天，中间靠你自己盯  
  
先看这一组日期，它是这条链最容易被忽略、也最该写进运维流程的部分：  
  
**10 月 17 日**  
：GeoServer 3.0.2 发布  
  
**10 月 18 日**  
：GeoServer 2.27.7 发布  
  
**11 月 2 日**  
：GeoServer 2.28.6 发布  
  
**11 月 23 日**  
：官方五份 GHSA 公告与总更新页公开  
  
**从 3.0.2 发布到公告公开，中间 37 天；从 2.28.6 发布到公告公开，中间 21 天。**  
  
这是**协调披露的标准做法**  
，GeoServer 官方在其更新页中明确说明了策略："vulnerabilities are fixed for both stable and maintenance releases, prior to vulnerability details being published. The goal is to give everyone an opportunity to update prior to public disclosure."  
——**先修后曝，让所有人有机会在公开前更新。**  
  
但这个策略有一个明确的代价：**它把"看公告"变成了"看公告"，而不是"看版本"。**  
 因为在 10 月 17 日到 11 月 23 日这 37 天里，**没有任何一条公告告诉客户"你必须升它"**  
。你的 GIS 平台补没补、资产台账有没有更新，完全取决于你有没有把 GeoServer 的版本号纳入常规巡检项。  
  
对照另一条时间线就更刺眼：**11 月 24—25 日，探测就开始了。**  
 从补丁发布到攻击者敲门，**38 天**  
。  
### 3.2 第二层：这一次"修补"是一次升级，因为它关掉的不止 5 个洞  
  
这是本期最该被记住的一层。  
  
GeoServer 官方 11 月 23 日的总更新页，把 **3.0.2 / 2.28.6 / 2.27.7 这批版本修复的全部 CVE 都列了出来**  
。按官方口径逐条看：  
<table><caption></caption><colgroup><col style="width:24%;"/><col style="width:76%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">类别</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">CVE 与官方定性</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">本链 5 条</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-129244、CVE-2026-129243、CVE-2026-120705、CVE-2026-129245、CVE-2026-129242</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">独立 RCE</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">CVE-2026-120704</span></strong><section><span leaf=""> — JDBC 安全服务加载 SQLite JDBC 扩展导致 RCE（High）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">任意文件写</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">CVE-2026-120710</span></strong><section><span leaf=""> — GeoWebCache FileRasterFilter ZIP 更新导致任意文件写（High）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">SSRF / 文件读</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-120701（WPS DownloadMap 未认证 SSRF）、CVE-2026-120706（WPS 内联 VRT 至原生 GDAL）、CVE-2026-120709（图例图标 URL）、CVE-2026-129240（严格 schema 校验期间经 file:// schemaLocation 的本地文件探测）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">访问控制绕过</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-120707（GeoFence 源 IP 规则经 X-Forwarded-For 绕过）、CVE-2026-120708（ACL 源 IP 规则经 X-Forwarded-For 绕过）、CVE-2026-120703（JWT Headers 角色头未经验证即被信任）、CVE-2026-120702（CSW DirectDownload 在本已禁用的图层上仍被提供）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">样式与 XSS</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-129294（OGC API Styles 未认证样式管理）、CVE-2026-129293（样式编辑器图层预览中样式名导致存储型 XSS）</span></section></td></tr></tbody></table>  
**加起来 17 个 CVE，一批版本。**  
 而且从定性上看，**这 17 个里有 2 个独立的 RCE/任意文件写、4 个 SSRF 或文件读、4 个访问控制绕过**  
——单看这条 5 洞链，你甚至意识不到这批版本还修了别的东西。  
  
**这带来一个非常实用的结论**  
：**你不需要打五个补丁，你只需要升一个版本。**  
 反过来说，如果你只按"我是否受这 5 个 CVE 影响"来判断要不要升，你会低估这批版本的价值；如果你只按扫描器报出来的某一条来升，你可能升了 GeoServer 却漏了 GeoTools。  
### 3.3 第三层：还有另一条独立路径，不在这 5 个 CVE 里  
  
**CVE-2026-120704**  
 值得单独拎出来说，因为它构成了**第二条未认证 RCE 路径**  
。  
  
Eye Security 独立发现了这条链中的部分漏洞，并**用 CVE-2026-120705、CVE-2026-129245、CVE-2026-120704 三个缺陷的组合，在 Linux 和 Windows 上都演示了未认证 RCE**  
。这与 CryptoCat 的五环链是**不同的组合方式**  
——它跳过了第 1 环和第 2 环，直接用第 3 环读文件、第 4 环伪造会话、第 5 环换成 SQLite JDBC 扩展路径来落地执行。  
  
**这条路径的意义有两层**  
：其一，它**证明了这条链不是某个研究者的独门发现**  
，两家独立团队各自跑通了完整 RCE，这是"这不是理论风险"的有力信号；其二，它说明"升级"这件事的收益比"堵这 5 个洞"大得多——因为 CVE-2026-120704 同样在 3.0.2 / 2.28.6 / 2.27.7 里被修掉了。  
  
Eye Security 方面表示其发现于 **2026 年 9 月底**  
报告给厂商；该公司还识别出**超过 3000 个互联网可达的 GeoServer 实例**  
。  
### 3.4 第四层：KEV 没有收录这五个，不会有谁来敲你的门  
  
本次直读 CISA KEV 官方 JSON（catalogVersion **2026.11.25**  
，dateReleased 2026-11-25T19:48:12.4344Z，总计 **1805**  
 条）核验：  
  
**本链 5 个 CVE 均未收录**  
；  
  
KEV 中与 GeoServer 相关的历史条目共 3 条——**CVE-2025-58360**  
（2025-12-11 收录）、**CVE-2024-36401**  
（2024-07-15 收录）、CVE-2022-24816（JAI-EXT）（2024-06-26 收录），均与本链无关。  
  
**这意味着：没有联邦强制整改截止日期，没有 BOD 26-04 的取证排查要求，也不会有任何外部机制来提醒你"该修了"。**  
  
结合第三章第一层的时间线看，这一层断层的含义就清楚了：**补丁是厂商给了，公告是厂商给了，探测是攻击者发起了——中间那个"你该升级了"的推力，没有人替你施加。**  
 它只在你自己的台账上。  
### 3.5 第五层：受影响区间本身在滚动，别把"升到最新"当成一次性动作  
  
官方更新页同时列出了更早两批版本的修复情况，这两行值得抄进台账规则里：  
  
**GeoServer 3.0.0 / 2.28.4 / 2.27.6**  
 修复了 **CVE-2025-53607**  
（OWS 请求中 xsi:schemaLocation 导致的未认证 XXE 文件读与 SSRF，High）  
  
**GeoServer 3.0.0 / 2.28.5 / 2.27.6**  
 修复了 **CVE-2024-35233**  
（REST coverage store 文件上传导致 ImageMosaic 远程代码执行，High）  
  
**注意这两条的修复版本区间和本链高度重叠。**  
 这意味着：**2.27.6 这个版本号，同时踩在 CVE-2025-53607、CVE-2024-35233 和本链的多个洞上。**  
  
还有一个细节：**GeoServer 官方对 2.27.x 这条老分支没有给出受影响版本的下界**  
（第三方情报机构的核查结论是：需自行与官方公告逐条比对老版本）。**所以"我的是 2.26，应该没事"这句话在 GeoServer 上不成立。**  
  
结论是：**GeoServer 的版本升级不是一次性任务，是一条长期基线。**  
 一个单位如果这次升到 3.0.2 就把这件事结案，那它和上次升到 3.0.0 的动作没有本质区别——**下一次官方批量修复发布时，它又会落在同一个区间里。**  
### 3.6 一个反向参照：GeoServer 不是第一次被打  
  
这一点必须写进去，因为它直接改变"要不要立刻动手"的判断。  
  
**2026 年 8 月**  
，攻击者把一个 GeoServer 漏洞当作**零日漏洞**  
进行利用（当时尚未打补丁）；  
  
**2025 年**  
，一家**美国政府机构**  
被曝因利用 GeoServer 漏洞而遭入侵。  
  
**同一平台，同一年，零日与在野入侵各发生过一次。**  
 加上这次已经有公开 PoC、且探测已经出现——**"这个平台不容易被打"的假设，在事实层面站不住。**  
  
   
## 四、受影响范围：版本、测绘口径与历史同类问题  
### 4.1 受影响版本与前提条件  
  
官方五份 GHSA 公告给出的受影响版本区间完全一致：  
<table><caption></caption><colgroup><col style="width:18%;"/><col style="width:52%;"/><col style="width:30%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">分支</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">受影响版本</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">修复版本</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">3.x</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">&gt;= 3.0.0, &lt;= 3.0.1</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">3.0.2</span></strong></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">2.28.x</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">&gt;= 2.28.0, &lt;= 2.28.5</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">2.28.6</span></strong></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">2.27.x</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">&lt;= 2.27.6</span></strong><section><span leaf="">（官方未给出下界）</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">2.27.7</span></strong></td></tr></tbody></table>  
**GeoTools 配套版本**  
（仅 CVE-2026-120705 需要）：org.geotools:gt-metadata 受影响为 **≤33.6、≥34.0 且 ≤34.5、≥35.0 且 ≤35.1**  
，修复为 33.7 / 34.6 / 35.2。  
  
**整条链的利用前提（官方与各机构通告口径一致）**  
：  
  
1. **网络可达**  
 GeoServer 的 WFS 端点与 REST API；  
  
2. **默认配置**  
——官方公告明确写明链的触发条件是 "default configuration"；  
  
3. **默认开放 WFS 服务入口**  
——链的起点是默认开放的 WFS；  
  
4. **默认用户服务中 admin 账号处于启用状态**  
；  
  
5. **执行载荷依赖 /bin/sh -c**  
——这是研究者在 Linux 环境下验证的前提，**这一环说明该链在 Linux 上的验证路径最直接**  
。  
### 4.2 三套测绘口径，统计对象完全不同  
  
关于"有多少台 GeoServer 暴露在公网上"，目前有三套数字：  
<table><caption></caption><colgroup><col style="width:28%;"/><col style="width:30%;"/><col style="width:42%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">来源</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">口径</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">数字</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">奇安信鹰图资产测绘平台</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">风险资产</span></strong><section><span leaf="">总数与关联 IP</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">国内</span><strong><span leaf="">风险资产 3852 个</span></strong><span leaf="">、关联 IP 345 个；全球风险资产 18412 个、关联 IP 1536 个</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">Eye Security</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">互联网可达实例</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">超过 3000 个</span></strong></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">长亭安全应急响应中心</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">公开分析给出的修复版本</span></strong><section><span leaf="">（未给出资产数）</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">3.0.2 / 2.28.6 / 2.27.7</span></section></td></tr></tbody></table>  
**这三套数字不能相互比较，更不能相加。**  
 统计对象定义不同（"风险资产"是按指纹匹配的口径，"互联网可达实例"是按暴露面口径），指纹规则不同，统计时点也不同。  
  
**而所有这些数字都只是测绘口径，不等于确认可利用。**  
 这里必须补一条反向的事实：**测绘只能看到公网可达的实例。**  
 内网横向可达、只对特定业务系统开放的 GeoServer 实例，根本不在公网测绘的覆盖范围内。而按 2.1 节讲的，这条链的起点恰恰是业务系统在用的那个 WFS 入口——真实的暴露面只会比测绘更大，不会更小。  
### 4.3 GIS 平台失守后，空间数据侧的后果比主机侧更重  
  
这是给江西政企客户的一条判断，因为 GeoServer 的特殊性在于：**它不是"一个 Web 应用"，它是"空间数据的出口"。**  
  
主机侧的后果是标准的：命令执行、进程权限范围内的横向移动、官方镜像 root 部署下的最高权限。**这一条所有人都会想到。**  
  
但**数据侧的后果更容易被漏掉**  
：  
  
**第 3 环本身就能读文件**  
——GeoServer 进程可读的本地文本文件，包括数据目录中的安全配置；  
  
**SSRF 让攻击者能把内网地址当探针**  
——从 GIS 服务器向内网发起请求，而 GIS 服务器通常位于**能同时访问业务数据库与政务内网的位置**  
；  
  
**图层发布范围决定了泄露面**  
——如果图层对匿名用户发布，读取到的内容会通过错误响应直接回显；**即使不回显，文件也已经被读了**  
；  
  
**空间数据的敏感性是独特的**  
——高精底图、地下管线、规划红线、禁限建区、勘测定界坐标，这些数据的敏感性不在"个人信息"这一栏里，而在**国家安全与关键基础设施**  
那一栏里。江西的自然资源、应急、测绘三条线都是这个量级。  
  
**这决定了处置优先级**  
：对 GIS 平台来说，"读文件"这一环的后果可能比"执行命令"更难估量。  
### 4.4 历史同类问题清单：这条链落在哪一片区域  
  
按官方更新页与历史通报，GeoServer 的高危问题长期集中在四个区域，本文逐条标注以避免重复踩坑：  
<table><caption></caption><colgroup><col style="width:24%;"/><col style="width:38%;"/><col style="width:38%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">历史 CVE</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">问题区域</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">与本链的关系</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2025-53607</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">OWS 请求 xsi:schemaLocation → 未认证 XXE 文件读与 SSRF</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">修复版本与本链重叠</span></strong><section><span leaf="">（3.0.0 / 2.28.4 / 2.27.6），同一个版本号同时踩两个雷</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2024-35233</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">REST coverage store 文件上传 → ImageMosaic RCE</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">修复版本与本链重叠</span></strong><section><span leaf="">（3.0.0 / 2.28.5 / 2.27.6）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-120704</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">JDBC 安全服务加载 SQLite JDBC 扩展 → RCE</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">与本链同批修复</span></strong><section><span leaf="">，且是 Eye Security 演示的第二条路径</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-120710</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">GeoWebCache FileRasterFilter ZIP 更新 → 任意文件写</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">与本链同批修复</span></strong></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">CVE-2026-120701 / 120706 / 120709 / 129240</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">WPS、打印图例、schemaLocation → SSRF 与文件读</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">与本链同批修复</span></strong><section><span leaf="">，与本链第 3 环同属&#34;读文件与外连&#34;家族</span></section></td></tr></tbody></table>  
**这张表的用途是防止重复劳动**  
：如果你单位历史上处理过 GeoServer 的 XXE 或 ImageMosaic 上传漏洞，**那次的修复版本号大概率仍然落在本链的受影响区间内**  
——不需要重新排查漏洞，需要重新确认版本。  
  
   
## 五、落地清单：一张盘点表，加一张核对单  
### 表一：《GeoServer 资产盘点表》  
  
这条链的排查难点不在"怎么修"，而在**"你不知道自己有这个东西"**  
。GeoServer 有一个特殊属性：**它通常不是以"Web 服务器"或"中间件"的名义登记在资产台账里的**  
——它多半是某个 GIS 业务系统的后端组件，登记在"XX 空间信息平台"名下，归 GIS 部门或建设单位运维，安全部门按"服务器/中间件"扫一遍，很可能扫不到它。  
  
所以盘点表的第一维不是版本，而是**"怎么找到它"**  
。  
<table><caption></caption><colgroup><col style="width:22%;"/><col style="width:26%;"/><col style="width:52%;"/></colgroup><tbody><tr><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">盘点维度</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">判定字段</span></section></th><th style="font-size:14px;font-weight:700;color:#006d7a;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;background:#f0f7f8;text-align:left;word-break:break-all;"><section><span leaf="">判定标准</span></section></th></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">① 是否存在</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">是否部署 GeoServer</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">按</span><strong><span leaf="">组件名</span></strong><span leaf="">而非&#34;服务器&#34;口径清点：GIS 平台、空间信息平台、地图服务、图层发布、遥感影像服务等业务名下逐个核实；重点查自然资源、应急、测绘、水利、规划、公安与城管等系统</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">② GeoServer 版本</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">主版本 + 补丁号 + 完整构建号</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">受影响判定：3.0.0–3.0.1、2.28.0–2.28.5、≤2.27.6。</span><strong><span leaf="">2.27.x 官方无下界，老版本一律按待核实处理</span></strong></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">③ GeoTools 版本</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">gt-metadata 版本</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">独立于 GeoServer 单独核。CVE-2026-120705 要求 ≥33.7 / 34.6 / 35.2。</span><strong><span leaf="">自拼部署包必查</span></strong></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">④ 部署形态</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">官方镜像 / 自建包 / 容器</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">官方镜像默认 root 运行；自建包需确认运行账号。</span><strong><span leaf="">&#34;进程是否 root&#34;必须实测，不能靠配置文档推断</span></strong></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">⑤ 匿名 WFS 开放面</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">是否发布图层给匿名、是否启用 WFS</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">匿名 WFS 是链的起点。</span><strong><span leaf="">同时决定第 3 环的读取结果能否被回显</span></strong></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">⑥ 数据目录访问控制</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">data_dir 是否被 Web 侧引用、是否有越权读路径</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">第 1、3 环都指向数据目录。</span><strong><span leaf="">数据目录与 Web 根目录同权限部署，等于把配置文件放在攻击者够得着的地方</span></strong></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">⑦ REST 接口暴露面</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">/rest/resource、/rest/security 是否可达</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">第 5 环的投递路径。官方缓解明确要求把这两个端点限制在可信网络</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">⑧ remember-me 配置</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">是否启用、密钥是否已改</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">关键判定：若仍在用发行版默认密钥，则&#34;改密钥&#34;这个动作无效</span></strong><section><span leaf="">（同一次文件泄露会同时暴露密钥与摘要）</span></section></td></tr><tr><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">⑨ 业务归属</span></strong></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><section><span leaf="">运维责任人、是否有外包或第三方运维</span></section></td><td style="font-size:14px;color:#333333;line-height:1.5;padding:8px 6px;border:1px solid #d9d9d9;word-break:break-all;vertical-align:top;"><strong><span leaf="">很多 GIS 平台由集成商或设计院交付并代维，版本基线不在你的台账上</span></strong></td></tr></tbody></table>### 操作便签：《GeoServer 处置核对单（按顺序做，不要跳步）》  
  
**第 1 步｜先定暴露面，再动版本**  
  
先回答一个问题：**这台 GeoServer 的 WFS 与 REST 接口，从公网能访问吗？**  
 若不能，本链风险大幅下降，但仍需按第二步升版本（因为第 2 环的"服务名绕过"在**内网横向**  
场景同样成立，攻击者不需要从公网进来）。若能——把第 3 步的临时缓解当作今晚就能做的事，不要等排期。  
  
**第 2 步｜升级，且要核对 GeoTools**  
  
GeoServer 升到 **3.0.2 / 2.28.6 / 2.27.7**  
（按当前分支）；**独立核对 GeoTools**  
 是否达到 33.7 / 34.6 / 35.2；  
  
**升级后必须做的两件事**  
：确认运行版本号、确认旧 remember-me Cookie 已失效。**升级后集群要确认 GEOSERVER_REMEMBERME_KEY 的共享配置**  
，否则节点间会话行为不一致；  
  
**务必在升级后改一次管理员口令**  
（官方在 CVE-2026-129245 的公告中明确写：若怀疑已被暴露，请更换管理员口令）。  
  
**第 3 步｜在代理层做四件事（这是"临时缓解"，不是替代升级）**  
  
在 GeoServer 前面的反向代理或 WAF 上：  
  
1. **阻断 service 参数中含 / 的请求**  
——查询串里的和 XML 请求 service 属性里的都要挡；  
  
2. **阻断 XInclude / file:// 关键字**  
，以及**超长 storedQuery_id**  
；  
  
3. **阻断异常的 remember-me Cookie**  
；  
  
4. **限制 /rest/resource 与 /rest/security 的访问来源**  
。  
  
同时**在 GeoServer 侧禁用或限制匿名 WFS**  
（若业务允许）。  
  
**官方与长亭通告都明确写：这些措施只能降低利用成功率或阻断链路入口，无法替代升级，且可能影响依赖匿名 WFS 调用的业务，升级完成后应及时移除。**  
  
**第 4 步｜查痕迹，顺序不能反**  
  
查失陷痕迹**必须放在升级之后**  
——因为升级会让旧 Cookie 失效、行为特征变化，先查后升会丢失关键证据。重点看四处：  
  
1. **数据目录 wfs/query 与 workspaces/*/wfs/query 下你自己没有创建过的存储查询**  
（这是官方 GHSA 点名的检查项）；  
  
2. **访问日志中针对 GetFeatureById 的异常请求**  
，以及 service 参数带前缀的请求；  
  
3. **数据目录中 security/config.xml、users.xml 的读取痕迹与异常修改时间**  
；  
  
4. **管理员账号的异常登录与异常会话**  
——第 4 环伪造出来的会话，特征与正常登录不同。  
  
**第 5 步｜补上三处"配置即漏洞"的根**  
  
这一步是把这次的教训固化成配置，而不是固化成一次性动作：  
  
· **Web 过滤器链中移除 rememberme**  
（官方首选缓解）；  
  
· **数据目录与 Web 根目录分离**  
，收紧数据目录权限；  
  
· **以非 root 或专用服务账号运行 GeoServer**  
，限制数据目录写权限——**注意这一步同时限制了"数据目录被写"和"命令以 root 执行"两个后果**  
。  
  
**第 6 步｜把版本纳入常规巡检**  
  
**这一条是这份清单里唯一能防住"下一次"的。**  
 建立规则：**GeoServer 与 GeoTools 版本列入季度资产巡检必检项**  
，与操作系统、中间件、数据库同级。原因见第三章第五层——受影响区间在滚动。  
### 一条给江西政企客户的落地建议  
  
前五步是通用的，但有一条对江西客户特别重要，而且它可以**直接挂到已有的合规动作上，不用新设流程**  
：  
  
**把"GIS 平台版本基线"挂到数据安全与测绘成果管理这两条既有台账上。**  
  
理由很实在。GeoServer 承载的是**空间数据**  
——高精底图、地下管线、规划红线、勘测定界坐标。而**江西测绘与自然资源两条线的成果数据，本身就是敏感资源**  
。这两个台账已经在管"数据在哪、谁在用、怎么共享"，但它们通常只记数据，不记承载数据的那个组件的版本。  
  
具体挂法建议三条：  
  
1. **在空间数据成果台账里加一列"承载组件版本"**  
——记 GeoServer 版本 + GeoTools 版本。这条数据的价值在下一轮官方批量修复发布时会立刻显现；  
  
2. **在测绘成果涉密/受限数据的分发环节加一个卡口**  
——**分发的是数据，但数据出口往往就是那个 WFS 入口**  
。凡是通过 GeoServer 对外提供受限图层的分发链路，分发前确认该实例的版本基线；  
  
3. **把"第三方代维的 GIS 平台"单列一类**  
——**版本基线不在你手里，是这条链最常见的失守形态**  
。处置动作不是催厂商出报告，而是在合同或运维协议里写明版本基线与升级责任。  
  
另外一条可以并入年度检查：**这次的三条国内机构通告（奇安信 CERT、长亭、绿盟科技 CERT）与 CNVD 公告 CNTA-2026-0008，都明确把"政务、测绘、自然资源及行业 GIS 平台"点名为典型受影响场景**  
。**这三个词几乎就是江西政企 GIS 系统的三个主战场**  
——这句话可以直接写进年度网络安全自查的通知里，比泛泛要求"排查开源组件漏洞"有用得多。  
  
   
## 六、几点判断  
  
**第一，处置的单位是版本，不是补丁。**  
  
这条链给防守方最大的提醒不是技术细节，而是**动作粒度**  
。面对"5 个 CVE"这个数字，很多人的第一反应是排 5 个工单；正确的动作是**一次升级**  
，因为这批版本一次性关掉了 17 个洞，其中还包含一条你原本不知道的独立 RCE 路径（CVE-2026-120704）和一条任意文件写（CVE-2026-120710）。版本基线对了，风险面自然收敛；版本基线错了，排多少工单都在原地。  
  
**第二，"改配置"不等于"降风险"，要看配置在哪一环上。**  
  
这是本期最反直觉的一条。**在大多数漏洞里，改密钥、改口令是有效的临时缓解；但在 CVE-2026-129245 上，官方明确写"改密钥没有用"**  
——因为同一次文件泄露会同时暴露密钥和口令摘要。**凡是"攻击链上另有一环能读到你改的那个配置"，单独加固这一环就是无效的。**  
 判断标准很简单：你改的那个值，会不会同时出现在别的漏洞能读到的地方？ 会，就得连版本一起升。  
  
**第三，"探测"和"得手"之间隔着的那段距离，就是防守窗口。**  
  
这条链目前的真实状态是：**探测已经开始，成功利用未被公开确认。**  
 这两句话必须一起讲——只讲前半句是无依据的恐慌，只讲后半句会让人以为还没轮到自己是错的。**探测阶段恰恰是窗口最宽的时候**  
：攻击者还在筛目标，说明大规模自动化利用还没铺开。但"铺开"这件事在 GeoServer 上有先例：2025 年有美国政府机构被曝因 GeoServer 漏洞遭入侵，2026 年 8 月它还被当作零日打。"还没得手"不等于"不会得手"。  
  
**第四，先修后曝的策略，代价落在防守方的日常里。**  
  
GeoServer 官方的协调披露做法本身是对的——它给了所有客户 37 天的窗口期。**但这 37 天里，没有任何一条公告告诉你"你必须升它"。**  
 这个推力本该由客户自己的资产巡检提供。**如果一个单位的资产台账里查不到 GeoServer 版本，那这 37 天对它是 0 天。**  
 所以真正该问的不是"这次补了没有"，而是"下次官方发一批修复，我的台账能不能在一天之内说出哪些系统该升"。  
  
**第五，测绘数字只能当线索，不能当结论。**  
  
奇安信鹰图说国内风险资产 3852 个，Eye Security 说全球互联网可达实例超 3000 个。**这两个数字不能比大小，也不能相加。**  
 更重要的是方向：**测绘只看公网，而这条链的起点是业务系统在用的 WFS 入口——内网横向可达的实例不在测绘覆盖范围内。所以真实的暴露面只会比测绘更大。**  
 用测绘数字做风险分级可以，用测绘数字做"我们只有这么多台、那我们没问题"的结论不可以。  
  
   
## 信息来源与核验方式  
### 官方一手源（本文评分、版本、缓解措施的主要依据）  
  
· GeoServer 官方安全公告 GHSA-4v26-c537-jvxc（CVE-2026-129244）：github.com/geoserver/geoserver/security/advisories/GHSA-4v26-c537-jvxc  
  
· GeoServer 官方安全公告 GHSA-rv5g-g2r9-58rj（CVE-2026-129243）：github.com/geoserver/geoserver/security/advisories/GHSA-rv5g-g2r9-58rj  
  
· GeoServer 官方安全公告 GHSA-39mw-5chv-jh47（CVE-2026-120705）：github.com/geoserver/geoserver/security/advisories/GHSA-39mw-5chv-jh47  
  
· GeoServer 官方安全公告 GHSA-hgm3-wq4h-8vvq（CVE-2026-129245）：github.com/geoserver/geoserver/security/advisories/GHSA-hgm3-wq4h-8vvq  
  
· GeoServer 官方安全公告 GHSA-9wf2-fcgq-vxhq（CVE-2026-129242）：github.com/geoserver/geoserver/security/advisories/GHSA-9wf2-fcgq-vxhq  
  
· GeoServer 官方总更新页《GeoServer Security Update - November 2026》（2026-11-23，列出该批修复的全部 17 个 CVE 与协调披露策略）：geoserver.org/vulnerability/2026/11/23/cve-update.html  
  
· GeoServer 官方下载页：geoserver.org/download/  
### 国内机构通告（口径与在野状态依据）  
  
· 奇安信 CERT《GeoServer 未认证远程代码执行漏洞（CVE 攻击链）安全风险通告》（2026-11-25，公开时间 2026-11-24）：secrss.com/articles/95982  
  
· 长亭安全应急响应中心《【已复现】GeoServer 未认证远程代码执行漏洞（CVE-2026-129242 等 5 个 CVE）》（2026-11-25）：cn-sec.com/archives/5611038.html  
  
· 绿盟科技技术博客《GeoServer 多个高危漏洞通告》（通告编号 NS-2026-0039）：blog.nsfocus.net  
  
· CNCERT / 国家互联网应急中心安全公告 CNTA-2026-0008（CNVD 收录 5 个漏洞，2026-11-25）：secrss.com/articles/95999  
### 第三方研究与观测（在野状态、历史与独立路径依据）  
  
· CryptoCat（Jonah Burgess，Rapid7）《GeoServer Pre-Authentication RCE via WFS and HSQLDB》（披露时序、技术细节、PoC，2026-11-24）：cryptocat.me/blog/research/analysis/geoserver_preauth_rce/  
  
· SecurityWeek《Two Newly Disclosed GeoServer Vulnerabilities Under Attack》（2026-11-25，WatchTowr 观测）：securityweek.com/two-newly-disclosed-geoserver-vulnerabilities-under-attack/  
  
· Eye Security《Unauthenticated Remote Code Execution Vulnerability in GeoServer》（独立三洞路径、互联网可达实例数量）：labs.eye.security/unauthenticated-remote-code-execution-vulnerability-in-geoserver/  
### 核验方式说明  
  
**KEV 状态**  
：本文直读 CISA KEV 官方 JSON，核验时 catalogVersion 为 **2026.11.25**  
、dateReleased 2026-11-25T19:48:12.4344Z、总计 1805 条；确认本链 5 个 CVE 均未收录，KEV 中 GeoServer 相关历史条目为 CVE-2025-58360、CVE-2024-36401、CVE-2022-24816（JAI-EXT）三条。KEV 目录处于持续更新状态，本文数据以核验时点为准。  
  
**评分与向量**  
：全部取自官方 GHSA 公告页，并逐条标注版本（CVSS 3.1 或 CVSS 4.0）；国内机构给出的独立评分口径单独标注。**跨版本分值不可比。**  
  
**厂商原文引用**  
：官方 GHSA 中关于 Impact、Mitigation、Resolution 段落的引述，均为英文原文的中文转述，语义以官方英文公告为准。  
  
   
## 官方确认事实与未公开事项  
### 已由官方与机构确认的事实  
  
GeoServer 官方于 2026-11-23 发布五份安全公告及一份总更新页，确认存在一条由五个独立缺陷组成的未认证 RCE 链：存储查询超长标识缺少长度限制导致异常回显完整文件系统路径（CVE-2026-129244，6.9 Moderate）；服务调度器对带路径分隔符的服务名截断解析而访问控制规则仍按原始值匹配，导致服务级安全规则被绕过、匿名用户可创建与删除存储查询（CVE-2026-129243，8.8 High）；内置 WFS 2.0 存储查询 GetFeatureById 将调用方 ID 参数未经转义替换入 XML 模板且解析时未使用实体解析器，可读取本地文本文件、发起 SSRF 并经错误响应回显（CVE-2026-120705，CVSS 3.1 8.6 High）；remember-me 签名密钥自数据目录 security/config.xml 读取、发行包自带固定默认值，配合泄露的口令摘要即可伪造管理员会话，改密钥在同一次文件泄露场景下无效（CVE-2026-129245，9.2 Critical）；JDBC 用户组与角色服务以 ResultSet.getObject() 读取属性值，HSQLDB 驱动对 OTHER 类型列执行不受限反序列化（CVE-2026-129242，8.6 High）。受影响版本为 3.0.0–3.0.1、2.28.0–2.28.5 及 ≤2.27.6，修复版本为 3.0.2、2.28.6、2.27.7，CVE-2026-120705 另需 GeoTools 33.7 / 34.6 / 35.2。官方协调披露策略为先修复后公开，3.0.2 于 2026-10-17 发布、2.27.7 于 10-18、2.28.6 于 11-02。  
  
CNVD 于 2026-11-25 收录该链五个漏洞（CNVD-2026-47957 / 47958 / 47959 / 47960 / 47961），发布安全公告 CNTA-2026-0008，综合评级为"高危"，确认"漏洞技术细节与 PoC 已传播"，确认以官方容器镜像默认配置部署时 GeoServer 进程以 root 运行、以普通用户部署时影响受进程权限约束。  
  
奇安信 CERT 于 2026-11-25 发布通告（其记载公开时间为 2026-11-24），给出完整链 CVSS 3.1 = 9.8、高危评级、利用可能性高，POC 状态"已公开"、在野利用状态"未发现"、EXP 状态"已公开"、技术细节状态"已公开"，确认利用条件为默认配置下 WFS 与 REST 接口网络可达且无需任何凭据，给出五条临时缓解措施，并披露鹰图平台关联的国内风险资产 3852 个、关联 IP 345 个，全球风险资产 18412 个、关联 IP 1536 个。  
  
长亭安全应急响应中心于 2026-11-25 发布通告并完成复现验证，确认五处缺陷在默认配置下可依次触发、最终以 GeoServer 进程权限执行命令（其复现环境以 root 权限执行），给出修复版本与四条临时缓解措施，并明确"仅更新单个组件不足以消除风险"。  
  
绿盟科技 CERT 通告 NS-2026-0039 记"在野利用 未发现、PoC 状态 已公开"，并给出各条 CVE 的独立评分口径。  
  
WatchTowr 通过蜜罐网络观测到攻击者针对 CVE-2026-120705 与 CVE-2026-129243 发送错误型探测请求，通过创建并调用存储查询验证目标是否脆弱；**WatchTowr 明确未报告任何成功实现远程代码执行的案例**  
。Eye Security 独立发现了其中部分漏洞，并以 CVE-2026-120705、CVE-2026-129245、CVE-2026-120704 三者组合在 Linux 与 Windows 上演示了未认证 RCE，其发现于 2026 年 9 月底报告厂商，另识别出超过 3000 个互联网可达的 GeoServer 实例。  
### 各机构通告之间存在的口径差异（本文如实并列，不作合并）  
  
· **在野利用状态**  
：WatchTowr 观测到针对两条 CVE 的在野探测并已公开报道；奇安信 CERT 记"未发现"、绿盟科技 CERT 记"未发现"。**三者的差异源于观测范围与发布时点不同，并非事实冲突。**  
 本文统一表述为"已观测到在野探测，未确认成功利用"。  
  
· **PoC 状态**  
：奇安信 CERT 记"已公开"、绿盟科技 CERT 记"已公开"；长亭通告的"利用成熟度"字段填"POC 未公开"。**该字段在长亭通告中指其自身未公开；研究者 PoC 已于 2026-11-24 随分析文章公开。**  
 本文按"研究领域 PoC 已公开"表述。  
  
· **评分口径**  
：奇安信 CERT 对完整链给出 CVSS 3.1 = 9.8；官方对单条给出 CVSS 4.0（129245 = 9.2、129243 = 8.8、129242 = 8.6、129244 = 6.9）与 CVE-2026-120705 的 CVSS 3.1 = 8.6；绿盟科技 CERT 给出同量级的独立表述。**本文分主体标注，不作跨版本比较，也不写"官方评分 9.8"。**  
### 仅由第三方观测与研究者自报、官方未逐条确认的内容  
  
· 攻击者探测行为的具体形态（错误型探测、通过创建与调用存储查询验证脆弱性）来自 WatchTowr 的观测表述与社交媒体帖文引用，非 GeoServer 官方或 CNVD 的确认；  
  
· 五环链的完整技术路径、预认证条件（含默认 admin 账号启用、Linux /bin/sh -c 前提）、测试环境（OpenJDK 17 + Jetty 12 与 Java 21 官方 Docker）来自 CryptoCat 的公开分析；  
  
· Eye Security 的三洞组合路径、Linux 与 Windows 双平台演示结果、超 3000 个互联网可达实例的统计，来自其自身研究与披露；  
  
· 各家的资产测绘数字来自各自平台在各自时点的统计口径；  
  
· 修复版本号在长亭通告中被表述为"公开漏洞分析给出"，其官方公告入口由本文以 GeoServer 官方公告核实。  
### 官方未公开、本文不作推测的事项  
  
· **成功利用的确认**  
——截至本文写作时，无任何机构公开确认攻击者通过本链成功实现了远程代码执行；本文不对后续趋势作任何预测。  
  
· **攻击者身份与归属**  
——无任何机构作出归因，本文不作任何暗示。  
  
· **实际受害规模**  
——本文引用的一切数字均为测绘口径或第三方统计，不涉及任何真实受害单位的数量、名称、行业与损失，这些信息均未公开。  
  
· **江西各单位是否实际部署了 GeoServer、版本分布与暴露面**  
——无公开数据可查，第五章的盘点表与核对单为通用排查框架，具体资产情况须由各单位自行核实。  
  
· **2.27.x 分支受影响版本的下界**  
——GeoServer 官方未给出，本文不推测。  
  
· **完整 IoC 清单**  
——各机构未集中公布，本文不逐条抄录，核对入口见"信息来源"一节。  
### 本文的归纳性内容  
  
表一《GeoServer 资产盘点表》的九个盘点维度与判定字段、操作便签六步的顺序设计与"先查失陷痕迹再动版本"的整体判定顺序、"把 GeoServer 与 GeoTools 版本挂到既有空间数据台账与第三方代维合同"的江西本地落地建议（三个维度及其具体挂法）、以及第六章五点判断中的整体解读，均为本文基于官方公告与公开研究所作的分析与归纳，**不代表 GeoServer、奇安信 CERT、长亭、绿盟科技 CERT、CNVD 或任何厂商的官方分类与统计口径**  
。  
  
**免责声明**  
：本文为防守向风险通告与自查指引，仅用于风险说明、影响范围分析与企业自查，**不提供任何攻击方法、不复述漏洞利用步骤、不给出触发载荷构造**  
。所有评分口径、版本号、时间点与在野利用状态均以各厂商及官方机构公告为准，且官方文档与厂商处置指引在披露后仍在变动，执行前请以各家最新安全公告为准。涉及具体合规判断与处置决策时，请结合本单位实际情况并咨询专业意见。  
  
