#  PHP 紧急发布安全更新：两个严重 RCE 漏洞如何应对  
 中新赛克   2026-10-23 06:40  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/UcuFpnTAbbAiajfeYgYk6xCia3O7OAyC3FKQdBrs3EQ8Gk24FqnfXTqys0ibkzmyMfZXTPwrUDr0gYoKtowdnAWrg/640?wx_fmt=gif "")  
  
01  
  
漏洞概述  
  
2026 年 10 月 21 日，PHP 官方披露两个严重（Critical）级远程代码执行漏洞，编号   
CVE-2026-109618  
 与   
CVE-2026-109619  
。安全社区将本次事件称为   
php2shell  
，直观反映了其危害特征：未经身份验证的远程攻击者可构造恶意 HTTP 请求触发内存错误，在特定条件下实现对目标服务器的拒绝服务攻击、敏感数据窃取、远程代码执行。目前，漏洞技术细节与 PoC 已广泛传播。  
  
披露时间  
  
2026-10-21  
  
危害等级  
  
Critical（严重）  
  
攻击前提  
  
未经身份验证  
  
核心后果  
  
远程代码执行  
  
· · ·  
  
02  
  
漏洞原理  
  
CVE-2026-109618：内存耗尽时的提前退出导致内存破坏  
  
PHP 使用 longjmp() 处理内存耗尽（OOM）异常以放弃当前请求。但在部分上下文中，请求内存在完成初始化之前即处于可访问状态，此时若发生内存分配失败，PHP 未能正确处理该次提前退出，请求内存将停留在无效状态；随后的请求关闭流程若读取该无效状态，即造成内存破坏并可导致远程代码执行。  
  
官方列举的触发路径包括：multipart 请求缺少有效 Content-Length 而绕过 post_max_size 初始检查；单个分段可携带不限数量的头部，其在 PHP 内部的内存开销显著大于线上体积。需重点提示的是，mod_php 在请求启动失败时仍会无条件执行请求关闭流程，且该阶段早于业务代码运行，因此应用自身的登录校验无法作为有效防护依据。  
  
CVE-2026-109619：跨请求的陈旧全局状态被提前消费  
  
PHP 工作进程会保留错误、会话、对象及扩展等全局状态，这些状态指向某一请求所拥有的内存，必须在该请求内存释放前完成重置。若致命错误（包括 OOM）中断了重置流程，或诊断信息在关闭阶段重新填充了已清空的错误状态，则请求内存释放后工作进程中的全局字段仍指向该区域，后续请求在正常初始化之前读取该陈旧状态即构成释放后使用。攻击者可通过精心构造的堆内存布局控制触发顺序，实现未经身份认证的远程代码执行。  
  
已确认的消费路径有两类：一是 Session upload progress，该功能自 PHP 5.4 引入且默认开启，无需应用显式调用 session_start()，即可在常规会话初始化之前接触会话处理逻辑；二是 PHP 8.5 的 fatal_error_backtraces 机制。  
  
· · ·  
  
03  
  
影响范围  
<table><tbody><tr style="background: linear-gradient(135deg, rgb(239, 246, 255) 0%, rgb(236, 253, 245) 100%);"><th style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);font-weight: bold;color: rgb(26, 26, 26);text-align: center;"><section><span leaf="">PHP 分支</span></section></th><th style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);font-weight: bold;color: rgb(26, 26, 26);text-align: center;"><section><span leaf="">受影响版本</span></section></th><th style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);font-weight: bold;color: rgb(26, 26, 26);text-align: center;"><section><span leaf="">安全版本</span></section></th></tr><tr><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(51, 51, 51);"><section><span leaf="">8.2</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(153, 153, 153);"><section><span leaf="">&lt; 8.2.35</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;font-weight: bold;background-image: linear-gradient(135deg, rgb(37, 99, 235), rgb(5, 150, 105));background-clip: text;color: transparent;-webkit-text-fill-color: transparent;"><section><span leaf="">≥ 8.2.35</span></section></td></tr><tr><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(51, 51, 51);"><section><span leaf="">8.3</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(153, 153, 153);"><section><span leaf="">&lt; 8.3.36</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;font-weight: bold;background-image: linear-gradient(135deg, rgb(37, 99, 235), rgb(5, 150, 105));background-clip: text;color: transparent;-webkit-text-fill-color: transparent;"><section><span leaf="">≥ 8.3.36</span></section></td></tr><tr><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(51, 51, 51);"><section><span leaf="">8.4</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(153, 153, 153);"><section><span leaf="">&lt; 8.4.27</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;font-weight: bold;background-image: linear-gradient(135deg, rgb(37, 99, 235), rgb(5, 150, 105));background-clip: text;color: transparent;-webkit-text-fill-color: transparent;"><section><span leaf="">≥ 8.4.27</span></section></td></tr><tr><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(51, 51, 51);"><section><span leaf="">8.5</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;color: rgb(153, 153, 153);"><section><span leaf="">&lt; 8.5.12</span></section></td><td style="padding: 10px 8px;border: 1px solid rgb(224, 224, 224);text-align: center;font-weight: bold;background-image: linear-gradient(135deg, rgb(37, 99, 235), rgb(5, 150, 105));background-clip: text;color: transparent;-webkit-text-fill-color: transparent;"><section><span leaf="">≥ 8.5.12</span></section></td></tr></tbody></table>  
官方按运行方式（SAPI）给出的验证情况：Apache mod_php 下两个漏洞均已验证远程代码执行；PHP-FPM 下 CVE-2026-109619 已验证可远程执行，CVE-2026-109618 经源码分析认定受影响；FrankenPHP 很可能受影响；LiteSpeed (LSAPI) 尚未验证。采用 Nginx + PHP-FPM 架构的业务同样受影响，WordPress、Drupal、Magento、Laravel 等建站系统可能间接受影响。  
  
两点需要特别关注：  
  
其一，  
PHP 8.1 及更早版本已结束官方常规安全支持  
，官方预期不提供修复，部分发行版与 LTS 服务商可能提供回移补丁，需主动向供应商确认。仍在运行 PHP 5.x、7.x、8.0/8.1 的环境，应尽快迁移至受支持的 PHP 8 分支，或采取下线、网络隔离等措施。  
  
其二，  
容器化部署是排查中易被遗漏的环节  
。Docker PHP-FPM 镜像及各类环境的版本一致性，以及扩容时是否重新拉取旧版本，均需纳入核对范围。  
  
· · ·  
  
04  
  
处置建议  
  
1. 升级至安全版本（唯一完整的修复方式）  
  
PHP 官方明确指出，升级到包含补丁的版本是唯一完整的修复方式。建议立即评估业务影响并升级至上述安全版本；使用发行版、容器镜像或托管服务提供的 PHP 时，应采用供应商明确标注已修复相关 CVE 的安全更新。升级前应完成备份与兼容性验证；升级后需重启全部 PHP 工作进程，确认提供 Web 服务的进程已加载修复版本，容器化部署需同步更新镜像并替换旧实例。  
  
2. 临时缓解措施（不能替代升级）  
  
针对 CVE-2026-109618：  
由管理员强制设置 memory_limit=-1，并确认进程池、目录或应用配置未恢复有限内存上限；该措施可移除可预测的单请求内存上限，但不完整，mmap() 等底层分配失败时仍可能进入风险路径。同时在 Web 服务器或反向代理侧限制解码后的请求体大小、输入变量数量、multipart 分段数、每分段头部数与上传文件数，限制应覆盖缺少有效 Content-Length 的请求；调低 max_input_nesting_level，不使用 multipart 表单的业务可在前端直接阻断此类请求。不应使用 RLIMIT_AS 或 RLIMIT_DATA 作为替代防护，达到上限后 mmap() 失败会触发风险路径。  
  
针对 CVE-2026-109619：  
设置 session.upload_progress.enabled=0，PHP 8.5 可另行设置 fatal_error_backtraces=0；性能允许时，可设置 pm.max_requests=1 以切断跨请求状态复用；同时消除攻击者可触达路径上的致命错误，包括上传、导入、错误页面、析构、序列化、输出处理及插件扩展等环节。  
  
3. 收缩暴露面并加强监测  
  
清点对外提供服务的 PHP 主机与容器实例，建立版本台账，优先处置直接暴露于互联网的 Web 服务；对公网暴露的 PHP 业务系统加强访问控制，收敛不必要的外部可达接口与调试入口，并遵循最小权限原则运行 PHP 服务进程。  
  
同时加强 Web 服务日志与主机行为监测，重点关注 Web 进程创建的异常子进程、非预期文件写入及可疑外联行为。从已知利用行为看，典型特征包括 php-fpm、httpd 等工作进程派生 shell 或网络工具子进程，命令行中出现 /dev/tcp/、fsockopen、mkfifo 等反向 shell 模式，以及 whoami、id 等侦察命令。  
  
上述配置是对已确认攻击路径的针对性缓解，PHP 官方已说明 CVE-2026-109619 不存在完整的通用配置缓解。所有配置均须在请求解析前生效（不应依赖 ini_set() 或 .user.ini），并重启全部 PHP 工作进程。  
  
· · ·  
  
05  
  
相关检测能力说明  
  
针对本次漏洞事件，  
星河AI安全管家  
服务已具备相应的检测与防护能力：  
  
漏洞检测  
：安全能力平台已支持对该漏洞的扫描检测，可定位受影响资产并输出修复建议。  
  
资产梳理  
：通过流量探针镜像采集持续发现公网暴露的域名、IP 与端口服务，支撑 PHP 资产台账建设。  
  
行为监测  
：具备对 Web 进程异常派生子进程、可疑外联、异常文件写入等利用特征的识别能力，告警经 AI 分级研判后降噪输出，并可联动防火墙完成封禁。  
  
专家支撑  
：配备云端安全专家与专属服务通道，可就排查、修复与应急处置提供远程协助。  
  
需要说明的是，工具能力不能替代补丁升级。建议用户以升级至安全版本为首要动作，同步开展资产排查与行为监测，形成  
“修复 + 监测”  
的处置闭环。  
  
· · ·  
  
06  
  
结语  
  
php2shell 这组漏洞的特别之处在于，其受影响组件是 PHP 运行时本身，而非某个可选引入的依赖包——任何一条能够到达解释器的请求路径，在特定条件下都具备触发可能。在补丁发布前，企业难以构建精确的检测特征，建议按以下顺序推进：  
  
优先完成资产盘点与版本台账建立，优先处置互联网暴露的 Web 服务；尽快升级至安全版本，并重启全部 PHP 工作进程；在全过程内保持对 Web 进程异常行为与可疑外联的持续监测。  
  
如需了解相关检测能力的部署方式，或需协助开展资产排查与处置，欢迎与我们联系。  
  
· · ·  
  
参考资料  
  
1. PHP 官方安全公告（CVE-2026-109618）：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
2. PHP 官方安全公告（CVE-2026-109619）：https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
3. PHP 官方版本发布记录：https://www.php.net/archive/2026.php  
  
   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ucxsqNKHOCibWNt6TuicpHke1wLpYMib5FUXxpAibJ9y7nlXUwva4jQWSicibW0By9xETickldUlNCwhE0P2HwHh3px6Q/640?wx_fmt=gif "")  
  
  
