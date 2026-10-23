#  紧急！PHP 引擎层爆出两个 9.2 分严重漏洞，速查！  
 魔方安全   2026-10-23 00:48  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/3wlibOpCwqMuz28Txjwx7Y2ETtPKBFFL7nhEdXpzMGdlCETCsEWPpfG2smT8qBj9XNxk6YLFr0ia3Bj0BhIyZoD5uAM70xoEK7liaU40PLewaA/640?wx_fmt=gif&from=appmsg "")  
  
2026年10月21日，PHP 官方披露两项严重漏洞并发布安全更新：**CVE-2026-109618 与 CVE-2026-109619**  
，官方 CVSS v4.0 评分均为 9.2。官方建议所有运行 PHP Web 服务的用户尽快更新，并依据实际运行方式（SAPI）、应用操作与配置评估风险。  
  
  
## 01.事件回顾  
  
》  
披露前约两个月  
  
据 PHP 官方与 OpenAI 披露，漏洞由安全公司 Calif 在 OpenAI"Patch the Planet"计划下借助 GPT-5.6 Sol 发现，Wiz 独立发现并报告，PHP 维护者主导修复，各方协同准备约两个月。  
  
》  
10月20日  
  
PHP 安全团队发布预告，称将于 10月21日 14:00 UTC 发布关键更新，并保留在漏洞细节公开或出现野外利用时提前发布的权利。  
  
》  
10月21日  
  
8.5.12、8.4.27、8.3.36、8.2.35 四个安全版本正式发布。  
  
  
## 02.漏洞解析  
  
  
漏洞信息  
（依据 PHP 官方安全公告）  
  
漏洞编号  
CVE-2026-109618、CVE-2026-109619  
  
公开代号  
PHP2Shell（公开讨论与厂商通告中的称呼）  
  
官方表述  
109618 为内存耗尽（OOM）损坏导致的 RCE；109619 为陈旧全局状态导致的 RCE  
  
官方评级  
严重（Critical），CVSS v4.0 9.2  
  
利用条件  
无需权限、无需用户交互；攻击复杂度（AC）官方评为高  
  
影响部署  
PHP-FPM、Apache mod_php、php-cgi、FrankenPHP、LiteSpeed 等  
  
风险最高  
Apache mod_php，官方已演示单个未授权请求在应用代码运行前触发  
  
  
CVE-2026-109618  
内存耗尽损坏  
  
攻击者可借超大请求触发内存耗尽，使 PHP 在内部请求数据仅部分初始化时被中断，后续清理流程误把这批无效数据当作有效数据处理，进而造成内存破坏并导致远程代码执行。该缺陷发生在应用代码运行之前，与网站自身代码无关。官方已在 Apache mod_php 默认配置下通过单个未授权 HTTP 请求验证完整 RCE，故 mod_php 风险最高；PHP-FPM 在请求启动失败时子进程会直接退出，可阻断这一启动阶段路径  
  
  
CVE-2026-109619  
陈旧全局状态  
  
请求所属的内部状态在其内存被释放后仍可能残留，同一工作进程处理的下一个请求若在初始化重置前复用该状态，即形成释放后重用（UAF）并导致远程代码执行。长亭实验分析指出，multipart 请求解析中会话上传进度机制的多次状态切换，可能使内存对象被提前释放又被再次访问，release 构建多表现为进程崩溃，debug/ASAN 构建可见引用计数异常。官方确认两条路径：PHP 8.5 致命错误回溯路径（mod_php）、会话上传进度生命周期路径（mod_php 与 PHP-FPM，FPM 路径依赖正常启动后的合适应用操作）；该功能默认开启，无需应用主动调用 session_start()。  
  
  
  
官方将攻击复杂度评为"高（AC:H）"；官方验证的未授权 RCE 场景针对 Apache mod_php，风险也以此最高，不同运行方式与配置需分别评估。Wiz 称其在最新版 WordPress、Drupal、Joomla 测试环境复现了未授权攻击。  
  
  
  
## 03.影响范围  
  
  
**受影响版本（依据官方修复分支）**  
  
· PHP 8.2.x < 8.2.35  
  
· PHP 8.3.x < 8.3.36  
  
· PHP 8.4.x < 8.4.27  
  
· PHP 8.5.x < 8.5.12  
  
**安全版本：**  
8.2.35、8.3.36、8.4.27、8.5.12，或对应分支后续安全版本  
  
PHP 8.1 及更早版本已不再从 PHP 项目获得更新；官方为 7.4、8.0、8.1 提供了最小源码回移，供自行编译或通过发行版、长期支持厂商获取修复，但不代表恢复官方支持。官方建议这部分用户联系发行版或升级到受支持版本。  
  
  
  
## 04.解决方案  
  
  
► 优先更新（官方列为完整修复方案）  
  
更新至 8.2.35、8.3.36、8.4.27、8.5.12 或对应分支后续安全版本，可从 PHP 官方或操作系统发行版、长期支持厂商获取。使用发行版回补包时，不能只比对上游版本号，应按发行版安全公告确认更新已包含两个修复：  
  
GHSA-w8hw-w9fx-j79wGHSA-6wmf-4rv5-p3rp  
  
更新后须重启全部 PHP 工作进程，并核对实际 Web 运行方式（SAPI）生效的版本，而非只查看命令行 php -v。  
### ► 无法立即更新时的临时缓解  
  
官方说明，以下措施只能减少针对特定路径的暴露，并不能让安装变得安全。  
  
针对 CVE-2026-109618  
  
在应用无法覆盖的层级（FPM 进程池 php_admin_value 或 Apache 配置）设置 memory_limit=-1，并确认无配置重新设回有限值；设置较低的 post_max_size（不上传可设 8M，需上传可设 32M），使 upload_max_filesize 小于 post_max_size；在 Web 服务器或代理层限制解码后请求大小、字段数、multipart 分段数、标头数与嵌套深度。官方同时提示，不要用 RLIMIT_AS 或 RLIMIT_DATA 替代 PHP 内存限制，以免诱发相应分配失败；可用容器或 cgroup 限制作为可用性边界。  
  
  
针对 CVE-2026-109619  
  
在请求解析之前生效的 php.ini、FPM 进程池或服务器配置中设置 session.upload_progress.enabled=0 并重启全部工作进程，不能依赖脚本内 ini_set() 或 .user.ini；PHP 8.5 可另设 fatal_error_backtraces=0 并重启。官方同时说明，用 error_reporting、display_errors 或 @ 隐藏诊断信息并无帮助。  
  
对 PHP 8.1 及更早停更版本，按官方建议升级到受支持版本，或向发行版、长期支持厂商核实回补修复。  
  
  
  
## 05. 魔方安全应急响应  
  
  
针对此次漏洞，魔方安全第一时间启动应急响应机制，组织安全专家开展漏洞原理分析、影响范围评估与检测能力建设。  
  
  
引擎层漏洞处置的前提，是先掌握自身有多少 PHP 资产、分别运行什么版本与运行方式、哪些直接暴露在互联网，尤其是不在台账内的影子 PHP 资产。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3wlibOpCwqMt3eIOib1DDPCqay2Y0ICBBCyZ7UBQGAazqFbD6lVxcTJ6XmqicrXFUE36GVibQlZg0kZfX9bUg3ib86K1vwCYoP4dwo9bcxiahPWww/640?wx_fmt=png&from=appmsg "")  
  
### ▎魔方安全可协助用户：  
  
✓ 暴露资产清点  
：识别互联网暴露 Web 资产及 PHP、WordPress、Drupal、Joomla 等应用指纹与版本，定位受影响资产与影子资产。  
  
✓   
持续风险监测：  
结合暴露面与风险信息持续监控，对高暴露、高风险资产优先预警。  
  
✓   
漏洞闭环运营：  
对漏洞资产进行分派、跟踪、复测与闭环，确保处置责任到人、过程留痕。  
  
✓   
资产漏洞一体：  
打通"资产、风险、责任人"，在补丁窗口内有序完成处置。  
  
  
已采购  
魔方安全相关产品的客户，可登录平台查看资产与风险检测结果及处置建议。  
  
尚未采购  
相关产品的客户，可联系魔方安全服务团队，获取专项暴露资产排查与风险评估支持。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3wlibOpCwqMta7OMzVa0MmwnUvEhB76qyibpDnqGKVT21iahDt8lickkibv9nWMibr73Hrx1nX6vu1JvDMlAbtS78UEVHEmsROBEwn9Pkw2Ntaenc/640?wx_fmt=png&from=appmsg "")  
  
  
