#  PHP2Shell 漏洞利用链危害分析与修复方案  
 渗透测试网络安全   2026-10-21 15:14  
  
2026 年 10 月 21 日，PHP 官方发布紧急安全更新，修复两个严重的远程代码执行漏洞：CVE-2026-109618 和 CVE-2026-109619。  
  
公开的 PoC 展示了攻击者如何利用 PHP 内部的请求处理、Session 数据解析和反序列化机制，在特定条件下将异常数据处理转化为代码执行。  
  
这类漏洞最危险的地方在于：  
攻击者可能不需要网站账号，也不一定需要上传一个能够直接执行的 PHP 文件，就有机会控制运行 PHP 的服务器。  
  
本文重点分析三条利用链、实际危害，以及如何临时缓解和彻底修复。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/hF2kcyfeuwdDKhwDiajZazR5eLOS5JHgKfdZ2UVY0EELxJofM4wfCBM9qsXo206y3wvnqaFknB908qmiaNtRLv2HibfUYydMAufdz5g6C2AL7w/640?wx_fmt=png&from=appmsg "")  
  
# 一、漏洞原理  
  
PHP2Shell 涉及 PHP 核心的多个安全问题。理解它们，首先需要了解两个机制：  
  
Session 上传进度：PHP 可以在处理 multipart/form-data 请求时，记录上传状态、文件名等信息。  
  
反序列化与对象析构：PHP 将序列化数据还原为对象时，可能执行相关对象的 __destruct() 方法；对象清理阶段也可能触发额外操作。  
  
如果这些机制中的数据校验、状态管理或内存生命周期处理存在缺陷，攻击者就可能让原本应当被视为普通数据的内容进入危险执行路径。  
  
PHP 官方确认的两个问题分别涉及：  
  
CVE-2026-109618：内存分配失败时，请求数据可能尚未完整初始化，后续清理流程却将异常数据当作有效数据处理，造成内存损坏。  
  
CVE-2026-109619：请求级内部状态在底层内存释放后仍然残留，后续由同一工作进程处理的请求可能在正常初始化之前访问这些失效状态。  
  
研究者公开的 PoC 则展示了如何利用 Session 处理、反序列化清理和配置差异形成具体攻击链。  
# 二、三条利用链具体如何工作？  
## 利用链一：Session 注入 + 本地文件包含（LFI）  
  
核心思路：让 PHP 将攻击者可控的内容写入 Session，再寻找能够执行这份内容的入口。  
  
攻击过程如下：  
  
攻击者向目标发送特定的 multipart 上传请求。  
  
PHP 上传进度处理机制将文件名等信息记录到 Session 数据中。  
  
如果攻击者能够控制写入内容，并且应用存在本地文件包含（LFI）漏洞，就可能让 PHP 读取对应的 Session 文件。  
  
如果注入内容被当作 PHP 代码解析，就可能实现远程代码执行。  
  
这条攻击链的关键条件包括：相关上传进度功能处于可用状态、Session 数据可以被攻击者影响，以及存在能够将注入内容转化为代码执行的入口。  
## 为什么危险？  
  
攻击者不一定需要把恶意文件上传到网站的正常上传目录。即使网站禁止上传 .php 文件，如果 Session 文件能够被注入并通过其他路径执行，仍然可能产生严重后果。  
  
但需要注意：存在文件上传功能并不等于存在此漏洞，存在 LFI 也不代表一定可以利用这条链。具体结果取决于 Session 存储、解析方式和应用环境。  
## 利用链二：反序列化清理缺陷 → UAF → RCE  
  
核心思路：利用 PHP 对象清理期间的内存生命周期错误，将内存破坏进一步转化为代码执行。  
  
UAF（Use-After-Free）是指程序已经释放一块内存，却继续通过旧指针访问它。  
  
这条利用链可以拆成几个阶段：  
  
反序列化过程建立包含引用关系和待析构对象的数据结构。  
  
PHP 进入清理阶段，释放部分内部内存。  
  
清理过程中触发对象的 __destruct() 方法。  
  
如果析构函数又触发嵌套反序列化，内部操作可能重新访问已经释放的数据结构。  
  
在满足特定内存布局和运行条件时，攻击者可能进一步控制内存状态，实现代码执行。  
  
这类漏洞的根源不是简单的“使用了 unserialize()”，而是  
内存释放顺序、对象析构重入以及内部引用状态管理之间出现了问题。  
## 为什么危险？  
  
内存安全漏洞有时会直接造成 PHP 工作进程崩溃；但如果攻击者能够稳定控制相关内存状态，影响就可能从拒绝服务升级为任意代码执行。  
  
需要特别说明：这条路径对运行环境和内存布局存在依赖，PoC 在特定实验环境中的成功率不能直接代表所有线上服务器的实际利用成功率。  
## 利用链三：Session 处理器不一致 → 对象析构 → RCE  
  
核心思路：利用不同 Session 序列化处理器之间的解析差异，让同一份数据在不同环境下产生不同解释。  
  
PHP Session 支持不同的序列化格式。如果多个目录或请求处理路径采用不一致的处理器，就可能出现数据边界和解析规则不匹配的问题。  
  
攻击过程如下：  
  
攻击者能够影响某个 Session 中保存的数据。  
  
数据先后经过采用不同序列化处理器的 PHP 处理路径。  
  
某些原本应被视为普通字符串的内容，在另一处理器中可能被解释为序列化对象。  
  
如果对象包含可触发危险操作的析构行为，便可能在解析或清理过程中产生副作用。  
  
如果应用环境中存在可利用的对象链，可能进一步升级为远程代码执行。  
  
这条利用链尤其需要关注多站点、多目录、多 PHP-FPM Pool 或存在不同 Session 配置的环境。  
  
PoC 还指出，即使某些反序列化操作最终返回失败，清理阶段仍可能触发已经创建对象的析构函数。因此，  
解析失败不一定意味着没有副作用。  
  
不过，处理器配置不一致本身并不必然导致 RCE；仍然需要满足数据可控、对象行为可利用等条件。  
# 三、危害到底有多大？  
  
这类漏洞的风险不能仅仅理解为“网站可能被黑”。如果远程代码执行成功，攻击者可能直接获得 PHP 服务进程权限，并进一步影响整个业务系统。  
## 1. 服务器被直接控制  
  
攻击者可能在 PHP 工作进程的权限范围内执行系统命令。  
  
如果 PHP-FPM 以 www-data 等低权限账户运行，攻击者首先获得的是该账户的权限；如果服务进程权限过高，或者系统权限配置不当，后果可能更加严重。  
  
这意味着，攻击者可能不再局限于访问网页，而是能够在服务器上执行实际操作。  
## 2. 网站被植入后门，修复漏洞后仍可能失陷  
  
成功执行代码后，攻击者可能修改网站 PHP 文件、植入 WebShell、篡改页面或写入其他恶意脚本。  
  
如果攻击者还建立了持久化入口，即使管理员随后升级 PHP，也不意味着已经植入的后门会自动消失。  
## 3. 数据库、用户信息和密钥可能泄露  
  
PHP 应用通常需要访问数据库配置、API 密钥、缓存凭据和第三方服务令牌。  
  
如果攻击者获得相应访问权限，就可能读取敏感文件、访问数据库、窃取用户信息，或者利用泄露的凭据攻击其他服务。  
  
因此，RCE 的影响范围取决于 PHP 进程能够访问什么，而不只是网站本身存储了哪些文件。  
## 4. 从一台服务器扩展到整个业务环境  
  
如果服务器保存了 SSH 密钥、云平台凭据、内部 API 令牌或其他主机的访问凭据，攻击者可能尝试进一步访问关联服务器和内部服务。  
  
这可能使一个 PHP 网站的漏洞演变为更大范围的安全事件。  
## 5. 服务中断与业务损失  
  
攻击者可能利用异常请求导致工作进程崩溃，或通过恶意操作消耗服务器资源。  
  
对于电商、支付、会员系统和 API 服务，这可能造成业务中断、数据恢复成本增加，以及客户信任受损。  
## 6. 风险等级如何判断？  
  
可以将风险分成三个层次：  
  
高风险暴露：公网提供 PHP 服务，实际运行版本未确认是否包含修复。  
  
严重安全事件：存在异常请求、工作进程异常或可疑文件等迹象，需要进一步调查。  
  
确认失陷：有证据表明发生未授权代码执行、后门植入或敏感数据泄露。  
  
需要强调：PHP 官方给出的 9.2 分代表漏洞的严重程度，并不代表每一台服务器都有相同的利用成功率，也不能仅凭漏洞存在就认定服务器已经被入侵。  
# 四、如何临时修复？  
## 方案一：立即升级 PHP  
  
这是最重要的措施。  
  
PHP 官方于 2026 年 10 月 21 日发布了以下修复版本：  
<table><tbody><tr><th><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="91"><span leaf="">PHP 分支</span></section></th><th><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="92"><span leaf="">修复版本</span></section></th></tr><tr><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="93"><span leaf="">PHP 8.2</span></section></td><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="94"><span leaf="">8.2.35</span></section></td></tr><tr><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="95"><span leaf="">PHP 8.3</span></section></td><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="96"><span leaf="">8.3.36</span></section></td></tr><tr><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="97"><span leaf="">PHP 8.4</span></section></td><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="98"><span leaf="">8.4.27</span></section></td></tr><tr><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="99"><span leaf="">PHP 8.5</span></section></td><td><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="100"><span leaf="">8.5.12</span></section></td></tr></tbody></table>  
应根据实际部署选择包含官方补丁的版本或发行版回补补丁。  
  
如果使用 PHP-FPM，需要确认实际运行的 Pool 已加载更新后的 PHP；如果使用 Apache mod_php、PHP-CGI 或容器，也必须更新对应运行时。  
  
仅更新 WordPress、插件或网站业务代码，不能替代 PHP 核心安全更新。  
## 方案二：关闭 Session 上传进度功能  
  
如果暂时无法升级，可以先关闭相关功能，减少部分已知攻击路径的暴露面。  
  
在实际生效的 php.ini 中设置：  
  
session.upload_progress.enabled = Off  
  
如果使用 PHP-FPM，也可以在对应 Pool 配置中强制设置：  
  
php_admin_flag[session.upload_progress.enabled] = off  
  
修改后应重启相关 PHP-FPM 服务，并确认全部工作进程已经应用新配置。  
  
注意：关闭上传进度功能不是完整修复。它不能修复所有反序列化 UAF、跨请求状态残留或其他独立缺陷。  
## 方案三：限制 HTTP 请求大小  
  
针对内存分配失败相关攻击路径，可以根据官方建议评估 post_max_size。  
  
不需要文件上传的应用，可以考虑：  
  
post_max_size = 8M  
  
需要文件上传的应用，可以考虑：  
  
post_max_size = 32M  
  
同时，应确保 upload_max_filesize 小于 post_max_size，并在 Nginx 层配置合理的请求体大小限制。  
  
如果采用官方针对 CVE-2026-109618 给出的临时措施，还需要评估将 memory_limit 设置为 -1 的资源影响，并通过容器或 cgroup 限制系统资源。  
  
这些配置仅能降低特定攻击路径的风险，不能代替补丁。  
## 方案四：Nginx 层限制 multipart 请求  
  
如果网站完全不需要文件上传，可以临时拒绝不必要的 multipart/form-data 请求。  
  
如果业务需要上传头像、附件或文件，则不建议全站直接封禁，而应按接口设置规则。  
  
这种方法只是一层额外防护：请求头过滤不能修复 PHP 内部的内存安全问题，也不能保证其他请求路径不会触发漏洞。  
## 方案五：检查 Session 配置一致性  
  
检查不同站点、目录和 PHP-FPM Pool 是否使用不一致的 session.serialize_handler。  
  
尽量统一相关配置，并避免让攻击者可控的数据进入不可信的反序列化流程。  
  
这有助于降低部分攻击链的风险，但仍不能替代 PHP 官方补丁。  
# 五、修复后还需要检查什么？  
  
如果服务器曾经暴露于相关漏洞，建议检查：  
  
Nginx、Apache 和 PHP-FPM 日志中是否存在异常 POST 请求。  
  
PHP 工作进程是否异常崩溃、频繁重启或启动非预期子进程。  
  
网站目录、上传目录和临时目录是否出现来源不明的 PHP 文件。  
  
数据库配置、SSH 密钥、云凭据和 API 令牌是否存在异常访问。  
  
计划任务、系统服务和网站文件是否发生未经授权的修改。  
  
如果确认存在未授权代码执行，应考虑隔离受影响主机、保留证据、轮换可能泄露的凭据，并从可信的软件包和备份恢复系统。  
  
不要认为升级 PHP 就能自动清除此前已经植入的后门。  
# 六、总结  
  
PHP2Shell 的危险性在于，它涉及 PHP 核心运行时，而不是某个特定网站插件。公开研究展示了 Session 注入与 LFI、反序列化 UAF，以及 Session 处理器不一致等利用路径。  
  
如果攻击者成功实现远程代码执行，可能造成服务器控制、网站后门、数据库泄露、凭据窃取、内网横向移动和业务中断。  
  
建议按照以下优先级处理：  
  
立即确认 PHP 实际运行版本及官方补丁状态。  
  
升级到包含安全修复的版本。  
  
暂时无法升级时，关闭上传进度功能，并结合官方建议实施请求限制及其他临时缓解。  
  
检查 Session 配置、异常请求和可疑文件。  
  
如果有成功入侵迹象，开展完整的安全事件响应。  
  
最终结论：关闭某项 PHP 功能或增加 Nginx 规则只能降低部分风险，不能保证系统安全。升级到包含官方修复的 PHP 版本，才是解决这类核心漏洞的首选方案。  
  
参考资料：  
  
PHP 官方安全公告：https://www.php.net/index.php  
  
PoC 技术分析：https://github.com/ikow/php2shell-poc/blob/main/docs/ANALYSIS.md  
  
关注本公众号回复“718619”  
  
可以免费领取全套网络安全学习教程，安全靶场、面试指南、安全沙龙PPT、代码安全、火眼安全系统等  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=8 "")  
 还在等什么？赶紧点击下方名片关注学习吧！  
  
  
**欢 迎 私 下 骚 扰**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hF2kcyfeuwcaXeQq6GPBG6GmnyF281FOAFnChNS3ZwNE5hcIUWW1JFUx4lGmkxtmPtoDNywbfnx5iaetbYYBT5wV5czlHS82AreKabc6ZN9s/640?wx_fmt=png&from=appmsg "")  
  
  
