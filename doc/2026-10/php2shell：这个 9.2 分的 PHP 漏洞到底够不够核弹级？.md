#  php2shell：这个 9.2 分的 PHP 漏洞到底够不够"核弹级"？  
原创 WorkBuddy
                    WorkBuddy  海狼风暴团队   2026-10-22 06:31  
  
蓝队防御  
技术分析  
  
php2shell：这个 9.2 分的 PHP 漏洞  
  
到底够不够"核弹级"？  
  
CVE-2026-109618 / CVE-2026-109619 全拆解 · 始末、定级、利用方式与真实风险  
  
海狼风暴团队｜蓝队防御 · 技术分析  
  
2026年10月21日 13:40 UTC，PHP 官方一次性发布 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 四个版本，修复核心层的两个远程代码执行漏洞——CVE-2026-109618与CVE-2026-109619，社区合称php2shell。CVSS v4.0 双 9.2，官方措辞罕见地不留余地："更新是唯一安全的选择。" 随后 48 小时，"核弹级""PHP 史上最严重""全球七成网站沦陷"的说法在中文社区刷屏。  
  
这篇不跟着喊。我们只回答四个问题：事件始末、国内定级、利用方式，以及——它到底够不够"核弹级"。  
  
01  
  
事件始末：一个沉睡了六年的缺陷  
  
这次事件最值得注意的一点，不是它爆发得多快，而是它被发现得多早、又被搁置了多久。根据 PHP 基金会的公开说明，相关缺陷早在 2020 年就被发现，2024 年 PHP 代码审计时又被重新发现一次——两次都因为"无法证明可利用"而没有修补。  
  
直到 2026 年，借助前沿 AI 模型的辅助分析，这个"理论缺陷"才被完整推导成一条可执行的 RCE 链。这本身就是一个信号：存量代码里那些"看起来打不通"的低危缺陷，正在被 AI 重新估值。  
  
9.2  
  
CVSS v4.0（双 CVE）  
  
5.4.0→8.5.x  
  
影响版本跨度  
  
4  
  
官方修复版本  
  
510万+  
  
公网暴露 IP（第三方扫描口径）  
  
-4天  
  
最早利用早于披露  
  
百万级  
  
公告后攻击尝试量  
  
  
  
从发现到在野利用  
  
  
2020 年 / 2024 年  
  
PHP 的 OOM 处理缺陷两次被发现，均因"无法证明可利用"而未修复  
  
  
2026 年 8 月 11 日  
  
多家研究团队（含 AI 模型辅助分析）独立报告了 OOM → RCE 的完整路径，随后又补充了更多攻击向量。修复工作被列为 PHP 基金会最高优先级  
  
  
2026 年 10 月 17 日 🔥  
  
有安全团队的蜜罐捕获到最早的利用尝试——比公开披露提前 4 天，来源为匿名 IP。攻击者如何在 embargo 期内拿到细节，至今没有答案  
  
  
2026 年 10 月 21 日 02:00 UTC  
  
最早的公开 PoC 出现，比官方解禁早约 12 小时；同日 07:34 UTC，已有安全机构从托管服务商处收到多起在野利用报告  
  
  
2026 年 10 月 21 日 13:40 UTC  
  
PHP 官方发布 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35；CNVD 收录 CNVD-2026-41674 / 41675 并发布风险提示；加拿大网络安全中心发布 AL26-026；多家 CDN 与防护服务同步上线 WAF 规则  
  
  
2026 年 10 月 22 日  
  
ENISA、欧盟委员会、CERT-EU、Europol EC3 与 EU CSIRTs 网络发布联合声明，要求各组织立即处置；国内多所高校与安全厂商密集发布通告  
  
02  
  
利用方式：两个缺陷，一条请求生命周期  
  
两个 CVE 的共同点，都落在同一个位置：PHP 请求生命周期的状态管理边界。当请求以一种"非正常"的方式结束时（内存耗尽、致命错误），PHP 没有把状态收拾干净，下一个请求就可能在初始化之前踩到上一轮留下的垃圾。  
  
  
  
CVE-2026-109618 — OOM 路径上的内存损坏  
  
PHP 处理内存耗尽时，会用longjmp()  
提前退出当前上下文。问题在于：某些上下文里，请求内存"尚未完全初始化"就已经处于可访问状态。此时若内存分配失败，PHP 没能正确处理这次提前退出，请求内存就被留在了**无效状态**  
。随后的请求关闭流程可能读取、或基于这块状态做分支判断——内存损坏，进而 RCE。  
  
关键在于"怎么把 OOM 打出来"。官方列出了三种能在不超 post_max_size的前提下耗尽 memory_limit 的 multipart 手法：  
  
**① 缺省 Content-Length**  
— 不带可用声明长度的 multipart 请求，可以通过 post_max_size 的初始检查，继续投递解码后的 body  
  
**② 海量 header**  
— 单个 part 可塞入不受限数量的 header；这些 header 在 PHP 内部表示所吃的内存，远大于它们在报文里的体积  
  
**③ 空文件控件 + 嵌套字段名**  
— 空文件控件不计入 max_file_uploads，但照旧创建完整的文件元数据数组；嵌套字段名会让数组工作量成倍放大  
  
为什么 mod_php 被官方列为最高风险：Apache mod_php 在请求启动失败时，仍会无条件调用请求关闭流程。也就是说，攻击可以在任何应用代码开始执行之前完成。  
  
  
  
CVE-2026-109619 — 陈旧全局状态导致的 UAF  
  
PHP worker 会保留指向"上一个请求内存"的全局状态——错误、会话、对象、扩展状态。请求内存释放之后，这些全局字段仍指向那块已释放的区域，形成**悬垂指针**  
。某些在"下一个请求正常初始化之前"就执行的操作（INI 诊断、会话上传进度），会访问这块陈旧状态，造成释放后重用（UAF）。  
  
这条链需要凑齐"生产者 + 消费者"，而且两个请求必须落在同一个 worker上：  
  
**生产者（留下脏状态）**  
— 解析/编译失败、未捕获异常、require 失败、执行超时、内存耗尽等致命错误，都会让清理环节被跳过  
  
**消费者（踩到脏状态）**  
— 请求初始化前的 INI 诊断；或 session.upload_progress 路径。注意：修改 error_reporting、加 @、关 display_errors 都拦不住，因为状态在被访问之后 PHP 才决定要不要报  
  
  
  
关键：不同 SAPI 的可利用性差异很大  
  
Apache mod_php  
已确认 RCE  
  
两条路径已验证：PHP 8.5 的 fatal_error_backtraces 致命错误回溯、会话生命周期路径  
  
PHP-FPM  
有条件暴露  
  
会话路径已通过验证；需要前端下发 PHP_VALUE / PHP_ADMIN_VALUE 等早期消费者。请求启动失败时子进程会退出，阻断其中一条路径  
  
LSAPI（LiteSpeed）  
条件性候选  
  
可早期应用特殊环境 INI 值，属于条件性候选，尚无完整链路验证  
  
FrankenPHP  
默认配置未证实  
  
默认配置下未建立等效的早期配置消费者，但官方明确"不代表免疫"，可能通过应用或扩展行为暴露其他消费者  
  
⚠️  
澄清一个正在广泛流传的混淆  
  
中文社区目前流传着一种说法：php2shell 的触发机制是"停滞窗口（stall window）+ PHP_SESSION_UPLOAD_PROGRESS"，完整 RCE 还需要LFI / FastCGI / PEAR 等额外条件。  
  
这套描述真实存在，但它描述的是第三方 PoC 的利用路径，不是 PHP 官方定义的漏洞根因。两者不能划等号。  
  
那套说法从哪来的  
  
它源自安全研究者的独立分析，以及社区 PoC 仓库对其的封装。其机制是：rfc1867 上传进度回调会把zval*指针缓存进 session HashTable；当 multipart 上传停滞超过session.upload_progress.min_freq（默认 1 秒）后，后续的 FILE_DATA 事件会重新进入 session 初始化并释放该 HashTable，缓存指针随即变成悬垂指针。  
  
而该 PoC 的 RCE 链确实需要额外条件，按可靠性排序依次是：会话投毒 + LFI、日志投毒 + LFI、FastCGI 直连（php-fpm 端口可达）、PEAR 写 Webshell。这些是那个 PoC 的落地路径，不是漏洞本身的必要条件。  
  
官方口径与社区口径的关键差异  
  
①CVE-2026-109618 的官方公告从头到尾没有提到 session。它的触发靠 multipart 解析把 memory_limit 打爆，全程不涉及 session，也不需要 LFI——官方明确「完整 RCE 已在默认配置的 mod_php 上通过 HTTP 验证」。②session.upload_progress 属于 CVE-2026-109619，且只是它的两条消费者路径之一（另一条是请求初始化前的 INI 诊断，与 session 完全无关）。③ 早期大量报道把 PHP_SESSION_UPLOAD_PROGRESS，以及「关掉 session.upload_progress」这条缓解建议，错误地同时挂到了 109618。两家漏洞需要的缓解措施并不相同，混淆会导致防护做错方向。  
  
一句话：社区 PoC 借 LFI / FastCGI / PEAR，是因为纯内存利用需要针对每个构建版本计算偏移、还要赢下堆竞争，可靠性太差——那是「打不通主链时的降级路径」，不是「漏洞本来就打不通」。  
  
03  
  
国内定级：CNVD 是"高危"，不是"严重"  
  
很多人会看到两个说法打架：官方说 9.2 Critical（严重），国内通告写"高危"。这不是矛盾，是两套刻度体系。先把各方定级摆清楚：  
  
<table><tbody><tr><td style="background: rgb(46, 97, 221);color: rgb(255, 255, 255);font-weight: 700;padding: 10px 12px;border: 1px solid rgb(46, 97, 221);"><section><span leaf="">来源</span></section></td><td style="background: rgb(46, 97, 221);color: rgb(255, 255, 255);font-weight: 700;padding: 10px 12px;border: 1px solid rgb(46, 97, 221);"><section><span leaf="">编号 / 评分</span></section></td><td style="background: rgb(46, 97, 221);color: rgb(255, 255, 255);font-weight: 700;padding: 10px 12px;border: 1px solid rgb(46, 97, 221);"><section><span leaf="">定级</span></section></td></tr><tr><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">PHP 官方 / GHSA</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">CVE-2026-109618 / 109619</span><span leaf=""><br/></span><span leaf="">CVSS v4.0</span><strong style="color: rgb(229, 62, 62);"><span leaf="">9.2</span></strong></section></td><td style="background: rgb(255, 255, 255);color: rgb(229, 62, 62);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">Critical（严重）</span></section></td></tr><tr><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">CNVD 国家信息安全漏洞共享平台</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">CNVD-2026-41674</span><span leaf=""><br/></span><span leaf="">CNVD-2026-41675</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(194, 65, 12);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">高危</span></section></td></tr><tr><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">国内高校 / 机构通告</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">同上</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(194, 65, 12);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">高危 / 高风险</span></section></td></tr><tr><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">国内安全厂商</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">综合处置优先级</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(229, 62, 62);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">紧急</span></section></td></tr><tr><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">加拿大 Cyber Centre</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">AL26-026</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(229, 62, 62);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">Critical</span></section></td></tr><tr><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">ENISA / CERT-EU / Europol</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">EU 联合声明</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(229, 62, 62);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);font-weight: 700;"><section><span leaf="">Critical</span></section></td></tr></tbody></table>  
  
**怎么看这个差异：**  
CNVD 的"高危"已经是它常规业务分级里的高段位，并不代表风险被低估。CVSS v4.0 的 9.2 是国际通用的技术评分，两者统计口径和业务语境不同。**在实际处置上，请直接按"紧急"对待**  
——不需要纠结字面级别，官方那句"更新是唯一安全的选择"已经把话说尽了。  
  
04  
  
"核弹级"该不该给？把两边论据都摆出来  
  
先说实话：这个说法不是全无依据，但它把"影响的广度"和"利用的难度"混为一谈了。分开看会更清楚。  
  
  
  
支持"核弹级"的七条理由  
  
❶面积太大——缺陷在 PHP 核心层，不是某个应用组件。WordPress、Drupal、Joomla、Laravel、MediaWiki 全部波及，PHP 支撑全球约七成以上站点❷暴露量惊人——第三方公网扫描口径约 510 万个 IP 运行 PHP；另一份扫描为 438 万个 PHP 服务、189 万台主机；另有遥测口径约 2260 万实例❸免认证 + 前置触发——mod_php 上，攻击可在应用代码运行之前完成，业务层做得再干净也没用❹版本跨度十余年——从 PHP 5.4.0 一直到 8.5.x，7.4 等 EOL 分支还有海量实例在跑，且不会收到官方更新❺在野扩散极快——某头部 CDN 服务商披露：公告前数千次尝试，公告后 45 分钟数万次，2 小时数十万次，目前已是百万级❻泄露价值高——可读取进程内存，里面可能有凭证、API Key、会话令牌❼权威背书——漏洞发现方给出的评价是"PHP 历史上最严重的漏洞之一，甚至可以说是软件史上最严重的之一"  
  
  
  
要求"降级"的七条事实  
  
❶CVSS 向量自己写着 AC:H + AT:P——攻击复杂度"高"，且明确存在攻击前提条件。这不是"发一个包全网通杀"❷完整 RCE 只在 mod_php 上确认——FPM 是有条件暴露、会话路径已验证；FrankenPHP 默认配置未证实；LSAPI 仅候选❸需要同一 worker 内的状态复用——生产者与消费者必须落在同一个进程，多 worker、进程回收都会降低单次成功率❹可移植性受限——利用可靠性取决于 PHP 构建、扩展、前端行为、内存分配器布局，换个环境未必能打通❺公开 PoC 良莠不齐——部分仓库基于已被提出者本人质疑的利用理论；即便是有效的实现，也需针对每个构建版本计算偏移、并赢下堆竞争。有代码 ≠ 能打通❻不可自传播——没有蠕虫式自动扩散能力，仍需逐个目标投递❼修复路径清晰——四个官方版本 + 7.4/8.0/8.1 最小 backport 分支都已给出，升级即闭环  
  
我们的结论  
  
"面"是核弹级的，"点"不是。  
  
它配得上 CVSS 9.2 的 Critical，配得上"紧急处置"，也完全配得上官方那句"更新是唯一安全的选择"。这是一次不折不扣的系统性风险。  
  
但它不是"一键通杀"。称作"核弹级"，容易让防守方产生两种误判：一是以为无差别通杀、防御无望，索性放弃精细化处置；二是以为只有 mod_php 危险、自己跑 FPM 就安全，从而放松警惕——而 FPM 的会话路径其实已经被验证了。  
  
更准确的说法是：一枚影响半径极大、但引信不算短的炸弹。覆盖面和后果是核弹级的；触发条件、SAPI 依赖、worker 命中率，决定了它不是"按下按钮就全灭"。  
  
<table><tbody><tr><td style="background: rgb(46, 97, 221);color: rgb(255, 255, 255);font-weight: 700;padding: 10px 12px;border: 1px solid rgb(46, 97, 221);"><section><span leaf="">评价维度</span></section></td><td style="background: rgb(46, 97, 221);color: rgb(255, 255, 255);font-weight: 700;padding: 10px 12px;border: 1px solid rgb(46, 97, 221);"><section><span leaf="">我们的判断</span></section></td></tr><tr><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">漏洞严重性</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><strong><span leaf="">9.2 / Critical，毫无争议</span></strong></td></tr><tr><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">应急紧迫度</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><strong><span leaf="">最高级，立刻打补丁</span></strong></td></tr><tr><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">技术神话程度</span></section></td><td style="background: rgb(255, 255, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><strong><span leaf="">远达不到&#34;核弹&#34;</span></strong><section><span leaf="">——它是严谨的、多前提的复杂利用链，不是魔法</span></section></td></tr><tr><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><section><span leaf="">是否值得刷屏</span></section></td><td style="background: rgb(248, 250, 255);color: rgb(51, 51, 51);padding: 10px 12px;border: 1px solid rgb(221, 228, 245);"><strong><span leaf="">值得关注，不值得恐慌</span></strong></td></tr></tbody></table>  
  
05  
  
蓝队处置清单  
  
① 升级并重启（最重要，没有之一）  
  
升级到8.5.12 / 8.4.27 / 8.3.36 / 8.2.35，然后重启全部 PHP worker。不重启等于没修——陈旧状态还留在内存里。7.4 / 8.0 / 8.1 有官方最小 backport 分支，或联系发行版与 LTS 厂商。  
  
② 清点"隐性 PHP"  
  
第三方产品、设备固件、容器镜像、面板工具、CMS 插件生态里都可能捆着 PHP。ENISA 特别提示：有些产品即使"看起来不用 PHP"，后台也依赖它。  
  
③ 短时间无法升级：先隔离  
  
把互联网暴露的 PHP 应用与公网隔离，直到补丁能上。这比裸奔强得多。  
  
④ 临时缓解（不构成完整防护）  
  
针对 109618：管理员强制memory_limit=-1，前端限制解码后的请求体大小；不用 multipart 的应用直接在前端阻断这类请求。针对 109619：PHP 8.5 设fatal_error_backtraces=0；关闭session.upload_progress.enabled（即使打补丁后，不用也该关）；必要时设pm.max_requests=1。两者做完都要重启全部 worker，并确认没有池、目录或应用配置把限制又恢复回来。  
  
⚠️ 一个容易踩的坑：不要用 RLIMIT_AS 或 RLIMIT_DATA 当作防护。这会让 mmap() 在触限时失败，反而走进那个有漏洞的 longjmp() 路径。  
  
  
  
检测与狩猎：从现在往回查  
  
· 进程链：Web 服务进程（php / httpd / nginx）派生出 shell（/bin/sh、cmd.exe）。正常 Web 服务几乎不会这么干· 流量特征：multipart boundary 中出现 php2shell 相关标识；session cookie 带 ph2sh 前缀· 文件落地：Web 进程向 /tmp、htdocs 写入文件，典型 WebShell 落点· 内存泄露迹象：异常响应中混入可疑的内存片段、指针地址、非预期二进制内容· 时间范围：从 10 月 17 日往回查——那是公开报道中蜜罐观测到最早尝试的日子  
  
**日志审计配置：**  
开启log_errors=On  
、有效 error_log、error_reporting=E_ALL  
，重点看重复出现的 warning / deprecated / fatal error / 执行超时——尤其是只出现在攻击者可控路由、内容类型或插件选项上的那些。  
  
**提醒：日志里没有错误，不等于安全。**  
历史日志可能把 deprecated 级别过滤掉了，而 E_ALL 也只覆盖已执行到的具体路由。  
  
最后说一句题外话。这个漏洞真正让人后背发凉的地方，不是 9.2 这个数字，而是它在代码里躺了六年——两次被发现，两次因为"证明不了能利用"而被搁置。现在 AI 把这个门槛拉低了。那些我们一直没修的"理论缺陷"，可能正在被重新估值。对蓝队来说，这比任何一个单点漏洞都更值得记住。  
  
  
参考来源  
  
[1]严重PHP · GHSA-w8hw-w9fx-j79w：RCE via OOM corruption（CVE-2026-109618）github.com/php/php-src · CVSS v4.0 9.2 · 2026年10月21日  
  
[2]严重PHP · GHSA-6wmf-4rv5-p3rp：RCE via stale global state（CVE-2026-109619）github.com/php/php-src · CVSS v4.0 9.2 · 2026年10月21日  
  
[3]官方PHP 官方发布公告 · php.net/archive/20262026年10月21日 13:40 UTC · 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
[4]CNVD国家信息安全漏洞共享平台 · CNVD-2026-41674 / CNVD-2026-41675 风险提示2026年10月21日 · 危害级别：高危  
  
[5]境外加拿大网络安全中心 · Alert AL26-026cyber.gc.ca · 2026年10月21日  
  
[6]境外ENISA / 欧盟委员会 / CERT-EU / Europol EC3 · 关于 PHP 严重漏洞的联合声明enisa.europa.eu · 2026年10月  
  
[7]分析漏洞发现方研究团队 · PHP2Shell 技术深度分析2026年10月 · 已于 8 月上报 PHP 官方  
  
[8]厂商国内多家安全厂商 · php2shell 风险通告与复现验证2026年10月21-22日  
  
[9]在野海外多家安全团队（蜜罐观测 / WAF 遥测 / 托管服务商通报）2026年10月17-21日 · 在野利用与暴露面统计  
  
[10]辨析海外安全团队 · PHP2Shell 观测与处置分析明确指出 session.upload_progress 归属 109619、109618 公告从未提及 session  
  
[11]PoC社区安全研究者 · stall-window 机制分析与 php2shell 复现仓库社区 PoC 路径：zval* 缓存悬垂 + LFI/FastCGI/PEAR 降级链  
  
—— 技术分享，实战为本 ——  
  
海狼风暴团队将持续深耕新疆网络安全社区  
  
与行业爱好者交流互进，共同推动地域网络安全水平发展  
  
Hacking Group 0991B（海狼风暴团队）  
  
关注我们，技术路上不迷路  
  
