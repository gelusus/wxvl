#  WordPress 请更新7.1.4，已经有多漏洞  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-11-01 01:04  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6P2SaNy8iczbu1X9uLAOwlibicOVhZzgyDIJI9icT13NqibniaBIy9r5wYWuHx8ShlBev3wFRFQ1MSwwic9Zo5G6FMV91p12O4Hx9tozY/640?from=appmsg "")  
> **导语**  
：打补丁这件事本身，就可能就是攻击面。WordPress 7.1.4 修掉的正是一个藏在核心更新里的未认证竞态 RCE：攻击者只要抢到解压窗口的那几秒，就能在你的文档根目录里跑起他自己的安装器。补丁落地不到 24 小时，Wordfence 已经拦下 62,969 次尝试。  
  
  
![WordPress 升级窗口里的 RCE 竞态](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6O38Nfr7rUiatqhtQUjEsrH6OwaNBSJhJU9XSJACZHwGWbre1YwdD9AAUhn58fqUKPZiad5bgYx7jDp313r31kgTsSGIkNN0I8w8/640?from=appmsg "WordPress 升级窗口里的 RCE 竞态")  
## 一、攻击面出现在补丁到达的那一刻  
  
漏洞编号 CVE-2026-119977（Wordfence 公告页已挂编号，10 月 30 日 The Hacker News 报道仍称未分配），CVSS 3.1 评分 8.1，影响 WordPress Core 4.7 到 7.1.3，10 月 28 日随 7.1.4 一起修掉，同时回移植到 4.7.39 及以上的全部分支。  
  
它的特别之处不在难度，而在位置：漏洞不在业务逻辑里，在**核心升级流程**  
里。这意味着你越是“赶紧打补丁”，越是会在一个可预测的时间点主动打开这个洞。补丁本身就是触发器。  
  
7.1.4 一次性修了 22 个漏洞、17 份公告，但只有这一个不需要账号。  
## 二、解压目录是公开的，写入点也是公开的  
  
用完整安装包做核心更新时，WordPress 会把新文件解压到 wp-content/upgrade/  
 下的一个目录。7.1.4 之前，这个目录名直接取自包名——从后台上传的包，名字完全可预测。  
  
更糟的一环在清理逻辑：先删除该目录下的所有文件，**包括那个专门用来阻止目录列表的 index.php**  
。于是升级进行中的几秒里，一整套 WordPress 就摊在一个可被列目录、可被直接访问的路径上。  
  
解压出来的副本里带着完整安装器 wp-admin/setup-config.php  
。它不依赖站点的 wp-config.php  
 就能自己起来。攻击者在这个窗口内请求它，就等于在自己控制的数据库上、在别人的文档根目录里跑了一遍安装流程，一路走到 PHP 代码执行。  
  
Patchstack 补了一刀：这个写入点从 WordPress 2.8 就存在，而**每一次更新——包括无人值守的自动更新——都会重新打开这个窗口**  
。  
  
![WordPress 升级窗口 RCE 的四步攻击链](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6PCZE7mmicQ8frYG3dulgOlc9IVp53aRJiaIdia1o7a57Pa07EMCv5AxoQurTuh4siakCAdVqx3xmuaAjicKacVd5J4pnIRsE5cYadg/640?from=appmsg "WordPress 升级窗口 RCE 的四步攻击链")  
## 三、窗口只有几秒，攻击者靠轮询解决  
  
Wordfence 实验室里，可利用的暂存目录通常只存在几秒。Patchstack 因此把它评为 AC:H、优先级低，理由是“实践中不实用”。  
  
但攻击者不需要一次命中。Wordfence 的遥测显示，两个最大的来源对一小撮站点持续轮询了约 14.5 小时，其他来源在穷举各种暂存目录名和版本号变体。不是每个请求都打中正确路径，可“持续轮询 + 精确目录结构探测”说明对方已经读懂了这个竞态。  
  
数字是这么滚起来的：  
- 10 月 28 日 20:30 UTC，Wordfence 威胁响应团队上线防火墙规则（距发布约 3 小时）  
  
- 上线 9 分钟后，出现第一个匹配请求  
  
- 前 21 小时，累计超过 18,500 次指向 setup-config.php  
 暂存路径的请求  
  
- 24 小时口径下，Wordfence 拦截了 62,969 次尝试  
  
![从补丁到 6.3 万次攻击的时间线](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NzUOFxa19Mdcv3CADCvulCJ0qBou2Vot3ZxTgJmrlFyIiadbvEuWEYzU3U23sdicPoalf2H5U3r5s1XKXibq77Vib8udTrHQ6WWH8/640?from=appmsg "从补丁到 6.3 万次攻击的时间线")  
## 四、最扎心的一条：PHP 层防火墙正好是盲区  
  
Patchstack 的分析里藏着这条：攻击请求**直接请求解压出来的 PHP 文件，并不加载 WordPress**  
。官方安装包不带 mu-plugins 目录，所以站点自己的安全插件在这个请求里压根不会执行。  
  
换句话说，绕过防火墙的方式不是打穿它，而是让请求根本不经过它。Wordfence 因此要求使用 Extended Protection（扩展保护），让防火墙在 PHP 文件被访问前先跑起来——免费版用户要等到 11 月 27 日才拿到这条规则。  
  
正确的防御位置在 web 服务器层：拒绝访问 wp-content/upgrade/  
，并确认该目录不可被列目录。两次更新之间，这个目录里应当只剩一个 index.php  
。  
## 五、7.1.4 到底改了什么  
  
三处改动：暂存目录名追加 16 位随机字符；重建 index.php  
 并且不再删除它；阻止暂存配置数据被不安全地“提升”。还有一处发布说明没提的改动——update-core.php  
 把 wp-config.php  
 加进了永不复制出工作目录的文件清单。  
  
其余 21 个漏洞同样有看头。Interactivity API 一口气修了 7 条，根因在 KSES 放行任何 data-*  
 属性，而每条 Interactivity 指令就是 data-*  
 属性——Contributor 账号就能把指令塞进文章绕过过滤。Customizer 未发布的 changeset 可被未认证用户发现，Notes REST 接口的授权判断也漏了口子。  
## 六、处置清单  
  
确认版本已经到 7.1.4 或对应分支的安全版本（4.6 及更早不再收安全更新，只能升）。在 web 服务器层封掉 wp-content/upgrade/  
，确认它不可列目录。用 Wordfence 的话，确认 Extended Protection 已开启；免费版要么买付费，要么自己写规则。还没升级的站点，翻查 wp-content/upgrade/  
 下的异常 PHP 请求、可疑的管理员账号、陌生插件和你没放进去过的 PHP 文件。升完顺手跑一次 Wordfence 扫描，看核心文件有没有被动过。  
## 七、给红队的三条复盘  
  
**内部流程不是安全边界。**  
 只要某个流程会把文件放到 web 可达的位置，它就是攻击面，升级器、构建机、CI 产物目录同理。  
  
**可预测就意味着可自动化。**  
 攻击复杂度高不等于不可利用，只意味着要靠耐心、并发和足够多的目标去摊薄概率。  
  
**安全插件可能正好是盲区。**  
 最好的绕过不是击穿防护，而是让请求绕开防护——不加载 WordPress，就不加载你的插件。  
  
WordPress 这次最值得记住的不是 22 个漏洞，而是它默认“我正在升级”是安全的。防守方最该记住的也只有一句：**补丁上线的时间点，本身就是公开情报。**  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。文中数据与技术细节引自 Wordfence、Patchstack、WordPress 官方发布说明与 The Hacker News。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6MUuaYKVVuibrdhVsCHayvaHfNdU4Siay59Dxv7GwqibuOiaSuia2wQxMIoiaibGfLRsYAdhlb9Tftib1iaOHuRVwwaFGjhbYhkiau9VM4YM/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NmpFJKb2Xyicp619xOlrD4x0tCC0BOIjYCYzpCSOVEcPGakWwYwxKgPGfZIxv0DKDce2WqMPXHOoPiaOib9iafSicTU1c7qs1Lkgd0/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NZvDdXVPWbicVYiaxLgq5ypT1DPicrmyAQiaf7uibXvDJQ4YRTev0QLYzvHhj2RPtIJG7M0LHibedqvmoiaD8FAa7ZVk2I9AFQJ2PdYc/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6MiaicfAb4hichA6DWCUs3XqT0b4JmOPFRG9uuaZF5KPydBIyG1Y4iaxO5aCUOCAAzKEyFf5Qt8JtfUPbEo1MSB6d2TdQbTp8HiaTYs/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
