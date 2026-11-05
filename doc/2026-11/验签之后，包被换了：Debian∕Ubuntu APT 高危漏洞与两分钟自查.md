#  验签之后，包被换了：Debian/Ubuntu APT 高危漏洞与两分钟自查  
 Ots安全   2026-11-05 06:33  
  
**威胁简报**  
  
  
**恶意软件**  
  
  
**漏洞攻击**  
  
软件包管理器是 Linux 世界里信任浓度单点：它以 root 运行、它装的一切都将以 root 落地。11 月 4 日，Debian、Ubuntu 与 APT 上游开发者同步披露 CVE-2026-109518。处在网络路径上的攻击者，可以让 APT **在签名校验通过之后**  
把安装包换成自己的版本，代码以 root 执行。发现者是 Max Justicz，七年前那个让整个 Debian 系连夜关重定向的 apt RCE，也是他报的。  
## 一、先看问题：验签之后、安装之前  
  
**发生了什么。**  
 Debian 于 2026 年 11 月 4 日发布 DSA-6607-1，Ubuntu 与 APT 开发者同日发布公告。按官方措辞（Debian 安全邮件列表原文）：  
> Max Justicz discovered a vulnerability in the APT package manager. APT can merge two different package downloads after an HTTP redirect gives them the same URI. Improper file handling when resuming an HTTP download allows a man-in-the-middle attacker to replace a package after verification, resulting in the execution of arbitrary code as root.  
  
  
翻译过来：HTTP 重定向可以让两个不同的包下载落到同一个 URI 上，而断点续传路径上的文件处理不当，让中间人在**校验通过之后**  
完成替换。最终安装的是攻击者的包，任意代码以 root 执行。  
  
**评分的三种口径（这里必须说清楚）。**  
 你大概率会在不同推送里看到三个不同的"严重性"，它们并不矛盾，是三家的计分习惯不同：  
- **Ubuntu**  
CVSS 4.0 评分 **7.7**  
，定级 High，并称基于 Debian/Ubuntu 的衍生发行版同样受影响（未点名）；  
  
- **APT 上游开发者**  
定性为 critical，未给数值分；  
  
- **Debian**  
官方政策是不提供 CVSS 分、也不采用外部评分做分级依据（Debian Security FAQ 明文）。  
  
所以"9.0+"之类的转述（本文未在任何信源见到该说法）请直接忽略；以公告和你所在发行版的口径为准。  
  
**时间线。**  
 据外媒对 Justicz 的采访（新浪财经 11 月 4 日转载）：他于 9 月下旬把问题报给 Canonical。11 月 4 日三份公告齐发；Ubuntu 于当日 14:25 UTC 把四个受支持版本的新 apt 构建推入安全仓库，14:58 UTC 时 Debian testing 分支仍停留在有洞的 3.3.5，到 15:46 UTC Ubuntu 的漏洞追踪页仍把全部八个受追踪版本标为受影响。**补丁在推、状态页在追**  
，这个时间差本身就是处置窗口的一部分。11 月 5 日上午，中文社媒已出现面向 VPS 用户的实操升级视频（X @ygkkkg，1.9 万粉技术博主，以甲骨文云 Ubuntu 22.04 演示）。  
  
**在野利用。**  
 截至本文，三份官方公告均未描述已出现的攻击，THN、西语技术博客与中文情报日报一致写明"无公开的在野利用报告"。Justicz 本人在采访中也称尚未见到有人尝试利用。  
  
**四源对照与本文标注。**  
 用户提供的三条信源与官方公告互证结论一致：THN（11 月 4 日，Swati Khandelwal）给出公告链接、精确版本号与 UTC 时间戳；西语博客 blog.elhacker.net（自注编译自 cybersecuritynews.com）补出了硬链接复用这一依赖条件和临时缓解思路；X 帖给出面向运维的实操检查与升级命令。**两处出入如实标注**  
：一是 X 帖把 Ubuntu 修复版写成 2.4.15 / 2.8.4 / 3.2.1，而 Launchpad 的完整版本号是 2.4.15ubuntu0.1 / 2.8.4ubuntu0.1 / 3.2.1ubuntu0.1，省略了发行版修订后缀，版本比对时容易误判（见第六节）；二是"漏洞由 OpenAI GPT-5.6 Sol 模型辅助发现"这一细节，目前只有采访这一路信源，Debian、Ubuntu、APT 三份官方公告与 THN 稿件均未提及 AI 参与，本文按单一信源处理。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0EcU96icBhnP1PRVknNaWLhVf7rTQRqIKFx3ByqK7uiavOZRnqhYgPDchDicnAuabkmLUEp0GZhSgaCBdKp5jQVONY9HS1zlBPM3c/640?wx_fmt=png&from=appmsg "")  
## 二、影响范围：谁默认就在射程内  
  
**触发开关是 Acquire::Retries。**  
 攻击依赖"下载中断后重试"这一步，因此只有当 Acquire::Retries  
 大于 0 时才成立。**Debian 12（bookworm）与 Ubuntu 22.04 起，该值默认为 3**  
；更早的发行版（Debian 11、Ubuntu 20.04）在默认配置下不在射程内。但 APT 开发者提醒：不少用户脚本会在老系统上自行开启重试，这些机器同样暴露。  
  
**版本状态一览**  
（依据 Debian Security Tracker 与 Ubuntu Launchpad，2026-11-04）：  
<table><thead><tr style="box-sizing: border-box;"><th data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">系统</span></section></th><th data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">状态与修复版本</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Debian 13（trixie）</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">3.0.3 → 3.0.4（DSA-6607-1）</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Debian 12（bookworm）</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">2.6.1 → 2.6.2（DLA-4880-1）</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Debian 11（bullseye）</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">修复版 2.2.5 已发布，LTS 已于 8 月底结束</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Debian testing / sid</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">3.3.5 → 3.3.6（上游）</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Ubuntu 26.10</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">3.3.3ubuntu0.1</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Ubuntu 26.04 LTS</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">3.2.1ubuntu0.1</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Ubuntu 24.04 LTS</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">2.8.4ubuntu0.1</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Ubuntu 22.04 LTS</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">2.4.15ubuntu0.1</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="208" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">Ubuntu 14.04 LTS</span></section></td><td data-colwidth="309" style="box-sizing: border-box;border: 1px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">有 Ubuntu Pro 订阅即可取修复</span></section></td></tr></tbody></table>  
补一句范围提醒：Ubuntu 明确表示**基于 Debian/Ubuntu 构建的衍生发行版同样需要各自出补丁**  
，所以用 Linux Mint、Kali、Pop!_OS 这类衍生版的读者，要盯自己发行版的安全通告，而不是默认"Debian 修了我就修了"。  
## 三、机制拆解：两个文件处理失误，叠出一个调包窗口  
  
APT 的信任模型一直很朴素：传输层用 HTTP 也行，反正下载完要过签名校验。这个模型成立的前提是：**校验过的文件就是即将安装的文件**  
。CVE-2026-109518 恰好打在这个前提上。  
  
拆开看是两件事的叠加（按官方公告措辞与上游修复说明还原）：  
  
**① 重定向把两个下载指向同一个 URI。**  
 HTTP 重定向响应可以让 APT 把不同的包下载落到同一个目标位置。西语博客的报道另点出一个依赖条件：**硬链接复用**  
：两次下载因此摸到同一个文件。  
  
**② 断点续传的文件处理不当。**  
 下载被中断（攻击者可以用重定向或普通的临时服务器错误制造中断）后，APT 在 Acquire::Retries > 0  
 时自动重试续传；续传路径对目标文件的处理有误，于是形成了一个短窗口：**签名校验已经完成，文件内容却还能被换掉**  
。  
  
注意这里的微妙之处：签名验证并没有被绕过，它确实校验了一个合法的包，只是校验的那份内容和最终装进去的那份内容不是同一份。校验结论只对"校验那一刻的文件"负责，而攻击者改的是"校验之后、安装之前"的文件。等 dpkg 解包安装时，软件包的 maintainer 脚本以 root 身份执行，到这里任意代码执行就成立了。  
  
**本文增补的定位**  
：这是一处典型的"检查与使用分离"（time-of-check / time-of-use）缺陷，只不过发生在包管理器这条全系统信任度的路径上，代价被放大到 root。  
## 四、触发条件：四个前提与利用链全貌  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0GcBxREVdI4JxP7kboXxM64cuy057E4g7d0EpfeQNTejuA9m62ia1RTJ5oeCkicfK5orubGHgogX1I9XzD74UOw21icw7BHtGP73A/640?wx_fmt=png&from=appmsg "")  
  
按 Ubuntu CVE 页面与 APT 开发者公告，把前提归拢一下：**四个条件缺一不可**  
：  
1. **处于 MITM 位置**  
：能读写客户端与包服务器之间的 HTTP 流量。同网主机、恶意代理、被控镜像，三者任居其一即可；  
  
1. **Acquire::Retries > 0**  
：才有"中断后自动重试"这一步可利用；  
  
1. **客户端跟随重定向**  
：攻击者靠 HTTP 重定向把两次下载指向同一 URI；  
  
1. **硬链接复用条件成立**  
：两次下载得以落到同一个文件上。  
  
攻击的节奏是：先用重定向制造中断，再用普通临时服务器错误逼 APT 重试，在续传的窗口里完成替换。THN 的措辞值得注意：攻击发生在"机器安装软件包的时候"，也就是说它不是需要人点链接的钓鱼，而是撞上正常的装包/更新动作。  
  
这比"一键远程打穿"的门槛高得多，但西语博客给出的场景判断是现实的：**公共 Wi-Fi、被劫持的路由器、被控的镜像源**  
都满足前提。Justicz 在采访里还给出一个更冷门但对机房有意义的场景：接入数据中心互联光纤的攻击者可能借此在数据中心服务器上执行代码。这个说法来自采访，未见官方公告背书，仅作场景参考。  
  
**一个官方没回答的问题。**  
 三份公告都只描述了 HTTP 场景下的攻击，**没有说明 HTTPS 源是否同样暴露**  
（THN 明确指出了这一点）。在官方口径补充之前，稳妥的做法是把"换 HTTPS 源"当作降险动作而不是完整修复（见第六节）。  
## 五、补丁与历史回响：同一个信任缺口的第三次  
  
**上游怎么修的。**  
 APT 3.3.6 把下载方法改为**以独占方式打开目标文件**  
，让两次下载再也摸不到同一个文件，这是针对两处失误叠加的关键修复。同一版本还拒绝缺少必需 Version  
 字段的 .deb 包，并修了一个依赖解析器的排序错误。各发行版的修复版本见第二节表格；Debian 同步指出 trixie 的修复还顺带关闭了若干越界访问错误。  
  
**历史回响才是这一节的重点。**  
 apt 的 HTTP 下载与重定向处理，这已经是同一个信任缺口的第三次：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0EqsYym6Al7pic6lmPicp0IPPFjMmibdLauJXfibWzKJdE1KbFibdYvCLSpwpwnyQ6SSAe6gGJparv7BwNCia93lYsicicIayghqwLsuDA/640?wx_fmt=png&from=appmsg "")  
- **2016 年 12 月**  
，Google Project Zero 的 Jann Horn 报告 apt 对 InRelease 文件的签名校验处理不当，MITM 可绕过签名（修复于 1.0.9.8.4 / 1.4~beta2）；  
  
- **2019 年 1 月**  
，Max Justicz 报告 CVE-2019-3462：HTTP 重定向处理代码未净化传输字段，注入内容被当作合法包执行。当时的临时缓解是 -o Acquire::http::AllowRedirect=false  
，The Register 当年那篇报道的标题至今读来贴切："apt for root-level malware injection"；  
  
- **2026 年 11 月**  
，还是 Max Justicz，换了一类 bug（重定向合并下载 + 续传文件处理不当），结论还是那一个：MITM 在验签后换包，root 执行。  
  
七年三撞，同一个组件、同一个信任模型、两次同一个报告人。这不能全怪维护者：重定向、断点续传、临时文件复用，本就是任何更新工具里风险密度的地带。但连续三次也说明：**把"校验过的等于将要装的"当作默认成立，代价就是每隔几年被现实纠正一次。**  
  
**本文增补：一个值得玩味的自举难题。**  
 修复这个洞的方式，是用 apt 更新 apt 自己。社区（gslin 博客）把这层怪味点了出来："不然用 apt 更新 apt 的安全漏洞，很怪啊。"这和 2019 年那次是同一个自举难题。当时 Debian 官方还真给过绕开自举的路径（手动下载、核对哈希、dpkg 安装）。而这次三份公告**均未给出无法升级时的缓解方案**  
（THN 明确指出），只能靠社区建议的临时措施降险。七年后处置手册反而更薄了，这大概是公告之外值得行业记一笔的事。  
  
**发现方式的另一重背景。**  
 据外媒对 Justicz 的采访（新浪财经 11 月 4 日转载），他这次是用 OpenAI 面向安全研究人员的 **GPT-5.6 Sol**  
 模型在 apt 代码里发现的问题，9 月下旬报至 Canonical。同一信源提到 OpenAI 6 月的"Patch the Planet"与 Anthropic 10 月的"Cyber Mission"漏洞修复计划。这和我们此前解读过的那份 Anthropic 红队报告是同一股趋势：AI 辅助挖洞正在从演示走向真实公告的致谢栏。需要重复一遍标注纪律：**三份官方公告均未提及 AI 参与，此细节目前只有采访这一路信源。**  
## 六、结束语：更新工具自己就是信任单点  
  
CVE-2026-109518 的技术故事不新：检查与使用之间留了条缝。新的地方在于它落在 apt 身上：一个以 root 运行、装什么都以 root 落地、连自己更新自己都要走同一条路的组件。七年三次，坑一次比一次偏（签名校验 → 重定向字段 → 续传与硬链接），但结论一直没变：**更新工具的下载与校验链路，配得上和它权限同等严格的审计**  
。  
  
**END**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zNsFJyIuL0HjMZYicaZshg5Jdq9ibDBL42t0GwOZ2YiahiaBakUTOX9vZ1JPncHw10HTnRDXhVswL2uBKS64uTXDpa8nuAiblJpamltmt2w8d9fA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
公众号内容都来自国外等平台- 搜索的内容通过结合编写 -   
  
公众号 |   
AnQuan7 (Ots安全)  
  
