#  【严重通告】PHP双RCE预警：两个9.2分漏洞，今晚22时准备更新  
 TJXFire   2026-10-21 06:41  
  
全文约 2300 字，预计阅读 7 分钟。  
  
PHP 官方已确认两项 CVSS v4.0 9.2 分严重漏洞的安全更新计划，CERT-EU 将其与 CVE-2026-109618、CVE-2026-109619 关联并提示潜在未认证远程代码执行风险。本文梳理已确认事实、仍待披露的条件，以及更新发布前后的处置清单。  
  
通告等级：严重预警｜完整技术细节待公布｜未取得可核验的在野利用证据  
  
一、先把今晚的维护窗口留出来  
  
10月240日，PHP 安全团队提前发布通知：将修复两项严重漏洞，两者的 CVSS v4.0 评分均为9.2。  
  
计划发布时间是 **2026年10月21日14:00 UTC，即北京时间今晚22:00**  
。官方同时保留提前发布的可能：**如果技术细节公开，或出现实际利用证据，就可能提前放出更新。**  
完整细节、受影响版本与修复版本将随更新公布。[1]  
  
这意味着，“等到今晚再开始找服务器”会压缩处置时间。现在适合做的是清点资产、联系维护方、备份和准备回归测试。  
  
CERT-EU 已发布安全预警，提示这两个编号存在潜在未认证 RCE 风险，并建议组织准备及时修补；**无法立即修复时，应考虑隔离面向互联网的应用。**  
[2]  
  
这里的“未认证”仍须等完整技术条件确认。不能仅凭9.2分倒推出请求格式、默认配置可利用性或具体攻击链。  
  
二、漏洞档案：哪些确定，哪些还不能写死  
<table><thead><tr><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">项目</span></section></th><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">当前核验结果</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">漏洞编号</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">CVE-2026-109618、CVE-2026-109619</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">涉及产品</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">PHP 运行时</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">严重程度</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#cf1322;font-weight:700;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">官方预告两项漏洞均为 CVSS v4.0 9.2</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">漏洞性质</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">CERT-EU 提示潜在未认证 RCE；完整条件待公布</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">精确影响版本</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">尚待 PHP 官方公布</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">修复版本</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">尚待 PHP 官方公布</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">技术细节</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">本次核验未取得完整官方说明</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">公开利用代码</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">本次未取得可核验的公开 PoC / EXP</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">在野利用</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">已核验来源未提供可核验事件证据</span></section></td></tr></tbody></table>  
严重预警已经成立，精确受影响清单仍需等待。  
  
watchTowr 的 FAQ 将两者描述为 PHP RCE，但明确指出“PHP 5起所有版本受影响”来自未经官方确认的报告，具体配置要求也未明确。[3]  
  
因此，PHP 5、7及各个8.x分支都应纳入待核查资产范围；这是一项排查建议，不能改写为“官方确认所有版本、所有部署均可被直接攻破”。  
  
“PHP2Shell”是公开讨论中流传的称呼。利用演示、研究者确认、公开可用代码和真实受害事件，是不同层次的证据，不能相互替代。  
  
Huntress 在10月20日的文章中说明，其合作伙伴环境当时未观察到相关在野利用，也未复现这两个漏洞的 PoC。这只代表该团队当时的观察范围，不能推导出全球没有攻击。[4]  
  
三、先找到真正处理请求的 PHP  
  
本次处置对象应落到运行时。建议以网站或业务入口为起点，追踪到实际处理请求的主机、PHP-FPM池、Apache模块、容器实例或托管服务，而不是只查看建站系统、框架及插件版本。  
  
需要排查的对象包括**公网网站、内部业务、测试站点、旧域名对应的源站，以及厂商软件自带的 PHP 环境**  
。是否受影响，最终都要结合官方版本范围和部署条件判断。  
  
PHP 官方文档提供了本机只读检查方式：php -v  
查看版本，php --ini  
查看配置文件路径。这些输出仅能说明当前执行的二进制及其配置，不能自动证明网站使用同一套 PHP。[5]  
  
建议让维护人员逐项填写：**业务入口、处理请求的运行时、部署方式、版本及软件包来源、维护责任人、更新渠道、允许维护时间**  
。对无法确认的实例单独列为待核实，不填“无需处理”。  
  
PHP 官方当前支持列表包含8.2、8.3、8.4和8.5分支；8.1及更早分支已不在该列表中。**是否由发行版或商业维护方提供回移补丁，需要向相应供应商核实。**  
[6]  
  
旧系统的困难往往是业务兼容性。现在就确认能否迁移、谁负责测试、无法及时迁移时如何限制暴露，比等修复版本发布后再找合同和联系人更有效。  
  
四、关于“上传进度”的说法，证据到哪里  
  
公开讨论把部分注意力放在 PHP 的会话上传进度机制上。本文尚未取得足以确认这一路径与两个 CVE 对应关系的一手技术披露，因此不将具体函数、内存破坏类型或关闭某项配置写成已验证结论。  
  
可以独立核实的是：PHP 手册说明，上传进度信息会在应用脚本执行之前写入会话；配置手册也说明，session.upload_progress.enabled  
默认开启。[7][8]  
  
机制事实不能替代漏洞根因证明。  
  
即使某个功能在脚本前执行，也不能据此直接证明这两个 CVE 都经过该路径。反过来，**仅凭“网站没有上传页面”或“后台需要登录”，也不足以排除运行时层面的风险。**  
  
现阶段不建议把单项配置调整当作两个漏洞已经完成修复的证据。待官方或维护供应商明确给出缓解措施后，再按适用条件验证实施。  
  
五、补丁发布前后，按这个顺序处理  
  
以下为面向运维和安全团队的处置建议，并非官方已经验证的特定漏洞绕过方案。  
  
**现在清点。**  
 将公网可达的 PHP 实例列为优先核查对象，补齐源站、容器、旧节点和第三方代管系统。**明确每个实例的更新责任人**  
，避免只登记域名而找不到实际维护方。  
  
**提前准备。****备份业务数据及配置，准备回退方案和关键业务测试项**  
；联系发行版、镜像或托管服务提供方，确认获取安全更新的渠道。不要预填猜测的“最低安全版本”。  
  
**收敛暴露。**  
 对非必要公网入口，经业务确认后限制来源或暂时隔离。WAF可辅助监测和限制访问，但在触发条件尚不明确时，**不能仅凭已有规则认定完成防护。**  
  
**更新运行时。**  
 安全更新发布后，核对公告明确修复的 CVE、版本分支及软件包公告。采用发行版回移补丁的系统，**应依据供应商修复说明判断，不能只比较上游版本号。**  
  
**确认实际生效。**  
 按维护方要求重启、重载或替换相关服务及容器，**再核查每个业务实例实际加载的版本**  
。保留升级记录和关键业务回归结果，避免只更新磁盘上的包或镜像标签。  
  
**继续排查异常。**  
 修补与入侵排查应分别记录。如出现可疑行为，保留日志和时间线，再开展隔离、凭证处置及恢复工作，**不将“升级成功”直接等同于“没有被入侵”。**  
  
六、目前能监测什么  
  
现阶段缺少本文已验证的专属攻击指纹。以下是通用入侵线索，不是命中即能归因至这两个 CVE 的检测规则。  
<table><thead><tr><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">观察位置</span></section></th><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">建议关注</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">Web与PHP进程</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">异常启动命令解释器、下载程序或其他非预期子进程</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">文件与持久化</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">非预期脚本落地、计划任务和账号新增</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">网络与运行状态</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">异常外联、进程崩溃与可疑请求在时间上的关联</span></section></td></tr></tbody></table>  
Huntress 同样建议关注 PHP / Web 服务的异常子进程和 WebShell 行为，并明确区分其既往观察到的 PHP 攻击与本次漏洞，不能把旧案例当作本次在野利用证据。[4]  
  
建议**立即保留现有反向代理、Web服务器、PHP及主机审计日志**  
，并确认日志是否连续、时间是否一致。日志缺失或没有告警，都不足以证明没有发生攻击。  
  
七、可直接转给维护方的核验单  
  
请确认本单位所有处理Web请求的PHP实例及责任人；以PHP官方公告和供应商安全通知核对受影响条件；准备安全更新、备份与回退窗口；更新后逐实例确认运行时版本及业务可用性；**提交修复依据、实施时间、遗留风险和异常排查结果**  
。  
  
当前最需要准备的，是可执行的更新流程和完整的实例清单。  
  
“核弹级”不是通用漏洞分级。本文采用官方的严重等级，同时保留对影响范围、利用条件和攻击状态的核验边界。预警可以提前，版本和技术结论不能靠猜测补齐。  
  
信息来源与核验方式 [1] PHP安全更新预告：https://www.php.net/archive/2026.php#2026-10-20-1 [2] CERT-EU安全预警2026-018：https://cert.europa.eu/publications/security-advisories/2026-018/ [3] watchTowr漏洞FAQ：https://watchtowr.com/intelligence/php-cve-2026-109618-faq/ [4] Huntress响应说明：https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities [5] PHP命令行选项：https://www.php.net/manual/en/features.commandline.options.php [6] PHP支持周期：https://www.php.net/supported-versions.php [7] PHP上传进度手册：https://www.php.net/manual/en/session.upload-progress.php [8] PHP会话配置手册：https://www.php.net/manual/en/session.configuration.php 核验截至2026年10月21日14:05（北京时间）。PHP官方预告用于确认评分、计划发布时间和披露安排；CERT-EU用于确认预警与其风险判断；研究团队资料保留自报及观察范围；PHP手册仅用于确认机制事实。排查顺序、资产表和核验单为本文处置建议。发布前应重新核对官方公告；本文未进行漏洞利用复现。  
  
安全通告 · 核验截至2026年10月21日14:05（北京时间） · 发布前请复核官方更新  
  
