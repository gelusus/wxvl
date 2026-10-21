#  【修复指南】PHP双RCE补丁已发布：怎么升级，怎样确认真正修好  
Yu
                    Yu  TJXFire   2026-10-21 14:44  
  
全文约 2100 字，预计阅读 7 分钟。  
  
PHP已发布修复CVE-2026-109618、CVE-2026-109619的安全版本，政企单位应从提前排查转入升级、验证与整改闭环。本文说明不同部署如何处理、临时措施有哪些局限，以及怎样验收，避免软件包更新了，网站却仍在使用旧运行时。  
  
一、补丁到了，先核对正确的修复依据  
  
PHP官方已确认两项漏洞可导致未认证远程代码执行，并发布以下安全版本。[1]  
<table><thead><tr><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">使用分支</span></section></th><th style="border:1px solid #d9d9d9;padding:8px 6px;background:#f0f7f8;font-weight:700;color:#006d7a;text-align:left;word-break:break-all;line-height:1.5"><section><span leaf="">上游首个修复版本</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">PHP 8.2</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#cf1322;font-weight:700;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">8.2.35</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">PHP 8.3</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#cf1322;font-weight:700;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">8.3.36</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">PHP 8.4</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#cf1322;font-weight:700;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">8.4.27</span></section></td></tr><tr><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#333;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">PHP 8.5</span></section></td><td style="border:1px solid #d9d9d9;padding:8px 6px;color:#cf1322;font-weight:700;line-height:1.55;word-break:break-all;vertical-align:top"><section><span leaf="">8.5.12</span></section></td></tr></tbody></table>  
以上是各分支的首个修复版本。使用上游发行包的，应更新到对应修复版或经确认包含修复的后续版本；由发行版或商业维护方供包的，则按供应商安全公告核对。  
  
**不能只看版本数字大小来判断是否修复。**  
  
例如，Ubuntu的USN-8948-1公告已列出修复这两个CVE的软件包：Ubuntu 24.04 LTS的PHP 8.3包为8.3.6-0ubuntu0.24.04.13  
；Ubuntu 22.04 LTS的PHP 8.1包为8.1.2-1ubuntu2.28  
。这些是特定发行版的修复包，不能移作其他系统的安全版本。[4]  
  
因此，扫描工具仅凭“8.3.6低于8.3.36”报警时，需要结合完整软件包版本及公告复核；也不能仅凭“供应商会维护”就直接关闭工单。  
  
二、哪些系统先处理  
  
PHP官方要求所有处理Web请求的PHP部署关注此次风险，包括PHP-FPM、Apache mod_php、php-cgi、FrankenPHP和LiteSpeed，并认为mod_php风险最高。[1]  
  
安全团队可按以下顺序组织排查：先处理公网可达且未修补的实例，再覆盖其他业务、测试环境及厂商内嵌组件。**公网mod_php部署应优先核实和安排变更**  
，但不能据此将其他部署标为“不受影响”。  
  
资产清单应同时记录业务入口、主机或容器、PHP部署方式、完整版本、软件包来源、业务负责人和运维责任人。**网站由第三方建设或托管的，也要落实到具体维护方。**  
  
两个漏洞的根因已经公开：109618涉及内存耗尽后的异常处理及无效状态使用；109619涉及工作进程复用时访问先前请求遗留的失效状态。[2][3]  
  
对政企管理者，关键是不要用“业务代码没改过”“后台要登录”来代替运行时核查。应用正常访问，也不能作为漏洞不存在的证据。  
  
三、按部署方式升级，别只更新一个PHP  
  
以下为建议实施流程，具体服务名称、升级方式及维护窗口应由本单位运维人员或供应商确认。  
  
**操作系统软件包部署。**  
 核实包来源和对应安全公告，备份配置及业务数据，在维护窗口更新实际使用的PHP运行时与相关组件。**不要仅安装CLI包便认定Web服务已修复**  
；升级后按供应商要求重启或重载，并**确认旧工作进程已退出**  
。  
  
**面板或多版本部署。**  
 逐站核对绑定的PHP版本。面板默认版本变更，不代表所有站点同步切换；同机多个PHP-FPM池、独立站点配置和历史安装目录都要检查。先选择可控实例验证，再分批推进。  
  
**容器部署。**  
 使用包含修复的可信基础镜像重新构建应用镜像，完成测试后替换运行实例。Docker文档说明，镜像是构建时的快照，需要定期重建获取更新，构建时可用--pull  
获取较新的基础镜像。[5]  
  
建议记录镜像摘要和运行实例标识，**确认所有副本都已替换**  
；同时更新部署模板，防止扩容或重新部署时再次使用旧镜像。只拉取新镜像、只修改标签，都不能作为运行实例已升级的验收证据。  
  
**托管或厂商软件。****要求服务方书面提供受影响判断、两个CVE的修复依据、实施范围及生效记录。**  
不要自行替换封装产品里的PHP文件，以免破坏依赖或失去供应商支持。  
  
四、老版本怎么办，暂时不能升级怎么办  
  
对于已经停止上游维护的分支，应优先评估迁移或由维护供应商提供回移补丁。PHP此次也提供了7.4、8.0、8.1的最小源码回移分支，供有源码构建能力或缺少LTS支持的使用者参考；这不等于旧分支恢复正常维护。[1]  
  
不要把自行编译回移补丁作为所有政企单位的默认操作。涉及商业系统、扩展兼容和业务可用性时，应由具备能力的维护团队负责构建、测试及后续更新。  
  
**临时配置只能减少部分攻击路径，不能替代完整升级。**  
  
109619的官方公告列出了针对特定路径的缓解，例如在请求解析前生效的配置中关闭session.upload_progress.enabled  
，并重启全部工作进程；PHP 8.5还涉及fatal_error_backtraces  
配置。但这些措施都不能覆盖全部变体，运行中用ini_set()  
调整也不能替代相应的前置配置。[3]  
  
109618的临时措施涉及内存失败路径、请求大小及资源边界，应结合其完整公告和实际部署评估。[2] 不建议脱离配套资源限制，直接把取消PHP内存限制的参数复制到生产环境；这可能引入可用性问题。  
  
确实无法及时修补的，由业务和安全负责人共同确认限制来源、缩小暴露面或暂时隔离的范围，**同时记录补丁计划、责任人和剩余风险。**  
已有WAF规则、关闭某个功能或隐藏错误提示，都不能直接作为“两个漏洞已修复”的证明。  
  
五、升级完成后，至少验收四件事  
  
**第一，修复依据匹配。**  
 记录系统分支、完整包版本或镜像摘要，以及明确覆盖两个CVE的官方或供应商公告。不能把另一分支、另一发行版的包号拿来验收。  
  
**第二，实际Web运行时生效。**  
 PHP文档中的php -v  
、php --ini  
可用于本机只读检查，但它们反映的是当前命令所调用的PHP及配置。[6] 建议结合服务配置和受控检查核对实际Web运行时，逐节点确认，**不把CLI输出当作整站证明**  
。  
  
不要为方便验收而长期向公网开放phpinfo()  
页面。需要临时诊断时，应控制访问范围，并在完成后删除。  
  
**第三，关键业务可用。**  
 用本单位正常业务用例验证登录、查询、提交、上传和接口调用等功能，观察错误率与资源使用。无需在生产系统上运行来源不明的EXP来证明“补丁成功”。  
  
**第四，异常排查有记录。**  
 检查暴露期间是否出现Web进程异常子进程、非预期脚本、可疑外联和账号或计划任务变更。它们是通用入侵线索，不是本次漏洞的专属指纹；发现异常后另立处置工单，保留日志及时间线。  
  
**补丁阻止后续利用，不负责清除已经存在的入侵痕迹。**  
  
六、可直接转给运维商的整改要求  
  
请逐项回复：  
1. 本单位哪些主机、站点、容器及代管系统运行PHP，哪些尚未核实；  
  
1. 每个实例的部署方式、升级前后完整版本，以及覆盖两个CVE的修复依据；  
  
1. 实施时间、执行人、工作进程或实例替换记录，以及业务回归结果；  
  
1. **尚未修复实例的限制措施、责任人和明确完成时间**  
；  
  
1. **暴露期间异常排查的范围、结果及日志留存情况**  
。  
  
建议采购人或业务责任人核对实例清单，运维方提交实施证据，安全人员复核修复依据和遗留风险。把“已通知供应商”“已安排升级”和“已验证修复”分别记录，避免工单状态掩盖实际进度。  
  
**验收落到每一个实际服务实例，才能说明修复范围。**  
  
信息来源与核验方式 [1] PHP官方安全公告及修复版本：https://www.php.net/archive/2026.php#2026-10-21-5 [2] PHP官方公告，RCE via OOM corruption：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w [3] PHP官方公告，RCE via stale global state：https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp [4] Ubuntu USN-8948-1：https://ubuntu.com/security/notices/USN-8948-1 [5] Docker镜像构建最佳实践：https://docs.docker.com/build/building/best-practices/ [6] PHP命令行选项手册：https://www.php.net/manual/en/features.commandline.options.php 核验截至2026年10月21日22:40（北京时间）。漏洞机制、修复版本及缓解边界来自官方公告，发行版包号来自其安全通知；分批实施、责任分工、验收清单及异常排查为本文建议。本文未进行利用复现，不以官方利用演示推导全球在野攻击规模。后续版本及供应商修复状态可能变化，实施前请复核。  
  
安全通告 · 核验截至 2026年10月21日22:40（北京时间） · 版本以官方及供应商公告为准  
  
