#  CVSS 10.0 满分、12 年全中招： GitLab 这波漏洞 把「自托管」盲区照得透亮  
宝十八
                    宝十八  网络安全老宋   2026-10-23 04:00  
  
**导语：**  
 你好，我是网络安全老宋。  
安全攻防干货准时送达！  
  
  
网络安全  
老宋  
// 漏洞应急 · GitLab  
  
// 漏洞应急 · GitLab  
# CVSS 10.0 满分、12 年全中招：GitLab 这波漏洞把「自托管」盲区照得透亮  
  
无需登录、无绕过方案，唯一的出路是把补丁打上  
  
CVE-2026-110264  
CVSS 10.0  
防守清单  
  
🔑一句话精华  
一个被 GitLab 打包进安装包的 Ruby 正则引擎，在 12 年里悄悄埋下了同一个坑。等它爆出来，是 CVSS 满分、无需登录、直接远程执行代码。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93fl9mILkyOztqOiaf9Nxc8AicqJXpsuicAyN0DZM6v4V4doic5zT2fBmibMXj4pm4wSI0skPtKAeKhh0rMjr7KAKjffvuQ5qdwydMFibbc/640?wx_fmt=png&from=appmsg "")  
  
▲ 满分漏洞与 12 年影响跨度：所有自托管实例都在靶心上  
  
10 月 21 日，GitLab 紧急发了一个安全公告，里面这个漏洞的编号是  
 CVE-2026-110264  
，CVSS 评分 10.0，也就是满分。更吓人的是它的影响范围：所有自托管（self-managed）的 GitLab CE/EE，从 2014 年的 7.0 版本一直到今年的 19.5，几乎无一幸免。  
  
换句话说，只要你们公司、你们客户、你们合作伙伴里有一台自己在机房或云上搭的 GitLab，而且还没升到指定版本，那它现在就处在一个"不需要账号、发个特制数据包就能拿下服务器"的状态。  
  
这不是危言耸听，是 CVE 官方、NVD、新加坡 CSA、欧盟 CERT-EU、荷兰 NCSC 同一周内一起亮红灯的事。这篇文章不跟你复述新闻稿，而是站在防守方  
的角度，把它讲清楚：它到底是什么、为什么这次特别要命、根因藏在哪、以及你现在该做哪几件事。  
  
## 01事件速览：一张表说清  
  
  
先把最关键的事实摆出来，省得你再去翻公告。  
<table><tbody><tr style="text-align: left;border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><td data-colwidth="116" style="text-align: left;padding: 9px 12px;font-weight: 700;color: rgb(26, 23, 20);font-size: 14px;line-height: 25px;"><section><span leaf="">项目</span></section></td><td style="text-align: left;padding: 9px 12px;font-weight: 700;color: rgb(26, 23, 20);font-size: 14px;line-height: 25px;"><section><span leaf="">关键事实</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">漏洞编号</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">CVE-2026-110264</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">CVSS 3.1</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">10.0（CRITICAL，满分）</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">漏洞类型</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">CWE-190 整数溢出（Integer Overflow）</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">攻击向量</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">网络可达、复杂度低、无需权限、无需用户交互、影响范围 Changed（可波及宿主机与周边服务）</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">危害</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">未经身份验证的攻击者，可在 GitLab 服务器上执行任意代码</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">根因</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">编译特制正则表达式时，内置 Onigmo 引擎发生整数溢出</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">披露时间</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">2026 年 10 月 21 日（各机构 10 月 22 日跟进告警）</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">公开利用</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">截至 10 月 22 日暂无公开 PoC、暂无在野利用，未进 CISA KEV 清单</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="116" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">临时绕过</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">没有。升级打补丁是唯一办法</span></section></td></tr></tbody></table>  
受影响的版本和对应修复版本，直接照着下面这张表对：  
<table><tbody><tr style="text-align: left;border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(219, 210, 196);"><td data-colwidth="231" style="text-align: left;padding: 9px 12px;font-weight: 700;color: rgb(26, 23, 20);font-size: 14px;line-height: 25px;"><section><span leaf="">你所在的版本分支</span></section></td><td style="text-align: left;padding: 9px 12px;font-weight: 700;color: rgb(26, 23, 20);font-size: 14px;line-height: 25px;"><section><span leaf="">必须升级到</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">7.0 ～ 18.11</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">18.11.13</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.0</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.0.10</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.1</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.1.9</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.2</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.2.9</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.3</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.3.6</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.4</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.4.3</span></section></td></tr><tr style="text-align: left;border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);"><td data-colwidth="231" style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(26, 23, 20);"><section><span leaf="">19.5</span></section></td><td style="text-align: left;padding: 8px 12px;font-size: 13.5px;line-height: 24px;color: rgb(94, 93, 93);"><section><span leaf="">19.5.1</span></section></td></tr></tbody></table>  
注意：  
GitLab.com（官方托管版）和 GitLab Dedicated 已经由官方直接修好，不用你动手。但只要是自托管，就得自己上。别看到服务器名字带"gitlab"就以为安全，得确认到底是不是真在用它家的托管服务。  
  
## 02为什么这次特别要命  
  
  
满分漏洞每年都有几个，但这一次要格外紧张，原因有四个。  
  
第一，影响跨度 12 年，旧实例逃不掉。  
以往很多严重漏洞只影响最近几个大版本，老系统还能靠"反正跑着旧版、攻击者懒得打"蒙混。这次从 7.0（2014 年）一路到 19.5，意味着那些被遗忘在开发网、测试实验室、甚至并购来的子公司里的老 GitLab，全在靶心上。盘资产时最容易漏掉的，恰恰是这些"影子实例"。  
  
第二，没有 workaround。  
很多漏洞还能靠改配置、关功能、加 WAF 规则先顶一阵。这次 GitLab 明确说没有临时缓解措施，因为触发点藏在正则编译的底层逻辑里，你没法靠"改个设置"把它绕开。没打补丁就等于暴露，没有中间态。  
  
第三，前置条件没公开，你没法自己排除风险。  
官方只说"在特定条件下"可被利用，但没说清楚条件是什么。这就意味着：你无法通过分析自己的配置来认定"我这个部署是安全的"。这种"未知的安全感"比已知风险更危险，管理员容易误判。  
  
第四，也是最容易被坑的一点：10 月 12 日你打过的补丁，并不等于安全。  
就在 10 天前（10 月 12 日），GitLab 刚修过一个几乎一模一样的漏洞  
 CVE-2026-97229  
（同样是正则引擎整数溢出、同样 CVSS 10.0、同样是研究员 joaxcar 通过 HackerOne 报的）。但那次只修了 19.2 及以后的分支。很多团队在 10 月 12 日勤快地升了级，心想"本月安全了"，结果这次的新漏洞直接打穿了那些他们认为"已合规"的环境。这叫"虚假合规感"，是这次事件里最现实的一个坑。  
  
更值得玩味的是：据 The Hacker News 统计，这是同一个人一个月内上报的、GitLab 正则引擎上的第四个代码执行漏洞  
，而且这是第一个能波及 19.2 之前老版本的重磅漏洞。一个引擎，一个月，连续四个 RCE。这已经不是"偶发 bug"，而是这条技术路线本身在反复出问题。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93fl8NVd5u414iapE4V37GU4Sialg9zqsS7D8bpSQyoibKGwMdJwTpc71hmUwVqF9lXtqF4IU2Y23cCib2Jen0oW5y3vQRHN3QJkHKdNQ/640?wx_fmt=png&from=appmsg "")  
  
▲ 从 2014 年的 7.0 到 2026 年的 19.5：12 年版本全在影响范围内，按分支对表升级  
  
## 03根因不在 GitLab 自己，在它打包的那个引擎  
  
  
想真正吸取教训，得看清根因。这个漏洞不是 GitLab 业务代码写错  
，而是它打包内置  
的一个组件，也就是 Onigmo，出问题了。  
  
Onigmo 是 Ruby 语言默认用的正则引擎。GitLab 因为是 Ruby on Rails 写的，就在自己的安装包里把 Onigmo 一起打了进去。问题是：只要 GitLab 打包了它自己定制的 Onigmo，这个底层缺陷就会跟着每一个 GitLab 版本一起发布。哪怕你底层系统的 Ruby 早就单独修过了，只要 GitLab 装的是它自带的那个旧 Onigmo，照样中招。  
  
这就是教科书级别的传递性供应链风险  
。你的暴露面，不是由你自己的代码定义的，而是由你"顺手打包进来、从没想过要去审计"的那个依赖定义的。  
  
这也解释了为什么 SBOM（软件物料清单）不是合规摆设。平时大家觉得"搞 SBOM 是为了应付检查"，但这次恰恰说明：如果你有一份准确的 SBOM，你就能在第一时间回答"我环境里哪些东西用到了 Onigmo 这个正则引擎、分布在哪些版本、哪些实例上"，而不用等漏洞爆了再全网地毯式排查。看懂依赖关系，和看懂自己写的代码，同样重要。  
  
## 04防守方现在该做的四件事  
  
  
光紧张没用，给你四件事，按优先级排好。  
  
第一步，盘资产：把"所有" GitLab 找出来。  
别只查生产网。开发网、测试实验室、离职同事留下的虚拟机、并购来的子公司、甚至供应商代管的实例，全部登记。重点排查那些没人认领的"影子实例"。每找出一台，就给它指派一个明确的负责人（owner），这是后面能推动升级的前提。  
  
第二步，查版本：逐台对表。  
登录每台实例，执行：  
  
   
   
   
⚙ Shell · 查看本机 GitLab 版本  
  
cat /opt/gitlab/version-manifest.txt 2>/dev/null | head -1  
  
# 或在 Rails 控制台 / 管理员面板查看，也可直接看页面底部版本号  
  
把结果和本文 01 节的表格对照。只要在某行"受影响范围"内、且低于"必须升级到"那列，就是待修对象。提醒一句：10 月 12 日升过级的，务必重新核对  
，很可能还在受影响区间。  
  
第三步，速打补丁，但走正规升级路径。  
先备份，再沿 GitLab 官方支持的升级路径升到对应修复版本。注意数据库迁移可能带来停机，提前规划维护窗口。另外，官方把 18.11.13、19.0.10、19.1.9、19.2.9 这几个"超出正常补丁窗口"的老分支也做了回溯修复，说明这事真的很严重，别嫌分支老就不升。  
  
第四步，上检测规则，边修边盯。  
GitLab 官方在 TLDR 框架里放了两套检测规则，建议立刻导入你的 SIEM 或日志系统，专门抓"还没来得及修的那些实例"被尝试利用的迹象：  
  
🔍 Onigmo native worker crash  
🔍 Org mode render error burst  
  
这两条规则描述的，正是漏洞被触发时服务端会留下的异常特征。在修复完成前，它们是你唯一的"哨兵"。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93fl9uRXy3QGFBia2WVdYoNtCOZssYQvgzTrPT0tr2TaRBsLPMdgTL6ibvBS0Y5IIl0c9NWaTrgCVSO8icnWDqCc6IfiadGyvWJSXjFeA/640?wx_fmt=png&from=appmsg "")  
  
▲ 防守三步：盘资产、打补丁、上检测规则，边修边盯  
  
## 05老宋说  
  
  
做安全运营这么多年，我越来越觉得：真正可怕的漏洞，往往不在你盯着的地方，而在你"以为没问题"的依赖里。  
  
这次 GitLab 满分漏洞给防守方的启示，不只是"赶紧打补丁"，那是基本功。更值钱的是三件事：  
  
一是把"影子资产"当成常态化问题。  
12 年跨度的漏洞最狠的一刀，就是砍在那些没人维护的老实例上。资产台账不全，再快的应急响应也救不了看不见的东西。  
  
二是 CI/CD 里的密钥和部署钥匙，才是真正的"皇冠"。  
GitLab 不只是代码仓库，它往往还握着你的构建流水线、部署凭据、云账号密钥。一旦被 RCE，攻击者的下一跳不是这台服务器，而是你下游所有通过它发布的服务，这才是供应链投毒的入口。所以这类系统的修复优先级，必须排在"面向公网"那一档的最前面。  
  
三是建立"补丁不等于安全"的复盘习惯。  
10 月 12 日那次升级制造了虚假合规感。以后每次大漏洞出来，都要反问一句："我上次打的补丁，覆盖这次的范围了吗？"别让"已升级"三个字变成盲点。  
  
顺手把 SBOM 这事儿真正做起来吧。不是为应付审计，而是为了下一次类似 Onigmo 这样的底层组件暴雷时，你能用十分钟回答"我哪里用了它"，而不是花三天全网翻箱倒柜。  
  
## 06结尾  
  
  
CVE-2026-110264 目前还没有公开利用，但这恰恰是最该行动的时候。等 PoC 满天飞、等 CISA KEV 点名，你的响应窗口就只剩"救火"了。  
  
满分漏洞、12 年跨度、无绕过方案，这三个词加在一起，结论只有一个：盘点、核对、升级、上规则，这四步今天就该动。  
别让你的 GitLab 成为那个"以为安全、其实一直在靶心上"的实例。  
  
如果你负责的环境里真有自托管的 GitLab，现在就去 01 节的表格对一眼版本。多花半小时，可能就少一次"服务器被人拿下了才知道"的事故。  
  
// 老宋说：你的暴露面，不是由你自己的代码定义的，而是由你从没想过要去审计的那个依赖定义的。  
  
推荐阅读  
  
这几篇相关的，建议一并看看：  
  
1. GitLab 官方补丁公告：19.5.1 及系列修复版本  
（升级路径与版本对照的权威来源）  
  
https://docs.gitlab.com/releases/patches/patch-release-gitlab-19-5-1-released/  
  
2. CVE-2026-110264 官方记录（CVE Program）  
（受影响版本范围、CVSS 向量、CWE 分类的原始条目）  
  
https://www.cve.org/CVERecord?id=CVE-2026-110264  
  
3. 新加坡 CSA 安全告警 AL-2026-152  
（政府机构视角的风险定级与处置建议）  
  
https://www.csa.gov.sg/alerts-and-advisories/alerts/al-2026-152  
  
4. 欧盟 CERT-EU 安全通告 2026-019  
（含 GitLab 官方两套检测规则（Onigmo worker crash 等）链接）  
  
https://cert.europa.eu/publications/security-advisories/2026-019  
  
5. Protect.Computer：CVSS 10 RCE 横跨 12 年版本  
（「同研究者一个月内第四个正则引擎 RCE」来龙去脉与逐分支对照）  
  
https://protect.computer/news/2026-10-21-gitlab-cve-2026-110264-regex-rce  
  
  
防御，不是在演练期间发现攻击，而是在演练开始前就把攻击面收敛到最小。  
  
end  
  
  
  
不想错过文章内容？读完请点一下**“在看**  
**”**  
，加个**“****关注”**  
，您的支持  
是我创作的动力  
  
期待您的一键三连支持（点赞、在看、分享~）  
  
