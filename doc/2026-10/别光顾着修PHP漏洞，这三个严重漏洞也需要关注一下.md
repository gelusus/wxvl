#  别光顾着修PHP漏洞，这三个严重漏洞也需要关注一下  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-22 05:18  
  
QUOTE  
  
同一天里，  
GitLab 埋着一颗哑弹，LiteLLM 被人从源头换了货，Node.js 的底座上裂开一道缝  
。这三件事的共同点是：  
你很难在日志里看见它发生过  
。  
  
—— 安全观察 · 2026 年 10 月  
  
除了火爆的php远程命令执行漏洞，  
2026 年 10 月 21 日这一天，安全圈还同时收到了三份公告。：  
  
GitLab 修补了一处位于正则表达式引擎中的整数溢出，编号   
CVE-2026-110264  
，CVSS 满分 10.0；  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPqnCgic3QbW9wgzWiaSXAloe9icbO9NzIic3odRbP4jz8a69Qb41wLcWOQsKkddqt1NFZXj9V7cDCuI0xM2VmLKlmZHcyWEfgBIaZs/640?wx_fmt=png&from=appmsg "")  
  
  
AI 网关项目   
LiteLLM  
 官方发布安全事件通报，承认其发布流水线被入侵，1.107.0 版本的 PyPI wheel 与 Docker 镜像中含   
凭据窃取器与远程代码加载后门  
；  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPqN5tr1lI3faQcjcDuS6Qh7kRLDV5bFv5cWWnjuEvNWktxCFstor16aJ0FOGgvKdFXU1Tw7vGCRzRM5U60ozibicH3j3oUvYUFVk/640?wx_fmt=png&from=appmsg "")  
  
  
Node.js 官方发布 10 月安全更新，其中   
CVE-2026-111497  
 是其内置 Brotli 解码器的堆内存越界写，也可导致远程代码执行。  
  
但三者的处置逻辑完全不同。GitLab 那个漏洞影响范围横跨   
12 年发布历史  
，厂商却始终没有说明触发条件；LiteLLM 那件根本不是漏洞，而是   
一次已经确认成立的攻击  
，攻击者拿到的不是某个接口的钥匙，而是整个发布链路的签名发布权限；而 Node.js 这个漏洞藏在   
几乎所有 Node 服务都会走的解压路径  
 上，官方定级只有 high 而非 critical，于是很容易被排进「以后再说」的队列。  
  
本文看点  
  
01  
  
10.0 分那个洞，为什么比想象中更难排查  
  
02  
  
投毒：一次已经成立的入侵  
  
03  
  
运行时底座上那道缝  
  
04  
  
72 小时处置清单  
  
01  
  
OVERVIEW  
### 先认清这三样东西  
  
  
三起事件的严重程度，很大程度上取决于它们各自   
是什么东西、被多少人装在什么位置  
。先把这三样产品的用途与体量摆清楚，后面的风险才好理解。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">产品</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">它是干什么的</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">用户量与下载量</span></section></th></tr></thead><tbody><tr><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(39, 39, 42);font-weight: 700;"><section><span leaf="">GitLab</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">DevSecOps 单体平台：源码托管、CI/CD 流水线、制品库、容器镜像仓库与安全扫描，一个应用装下整套研发工具链</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">官方披露 5000 万+ 注册用户、《财富》100 强中半数企业在用；公网暴露的自托管实例超 130 万个</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(39, 39, 42);font-weight: 700;background: rgb(250, 250, 250);"><section><span leaf="">LiteLLM</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">统一 LLM 调用层：既是 Python 库，也可作为自托管 AI 网关，把 100 多家模型提供商的接口收敛成一套 OpenAI 兼容协议，并集中管理密钥与配额</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">GitHub 星标 6 万+；PyPI 月下载 8900 万次以上；容器拉取超 5 亿次；约 2.6 万个仓库与软件包直接依赖</span></section></td></tr><tr><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(39, 39, 42);font-weight: 700;"><section><span leaf="">Node.js</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">服务端 JavaScript 运行时：Express、NestJS、Next.js 以及大量 CLI 工具与前端构建链路的底座，也是 npm 生态的运行时</span></section></td><td style="padding: 10px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">官方渠道 2026 年 1 至 7 月下载 46.1 亿次，已超过 2025 全年的 44.8 亿次；GitHub 星标约 11.9 万；npm 生态超过 200 万个包，周下载合计超 100 亿次</span></section></td></tr></tbody></table>  
  
这三行有一个共同点值得先点出来：它们的部署位置都在   
业务代码之外，又在业务数据之内  
。GitLab 存着源码与流水线凭据，LiteLLM 握着一整张模型调用凭据平面，Node.js 则跑着所有对外提供接口的服务。也就是说，它们被攻破之后不需要额外的横向移动，攻击者已经站在了最有价值的位置上。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">维度</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">CVE-2026-110264</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">LiteLLM 1.107.0</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">CVE-2026-111497</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">类型</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">正则编译器整数溢出</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">供应链投毒（已确认）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Brotli 解码器堆越界写</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">官方定级</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">CVSS 3.1 评分 10.0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">官方定性为已投毒发布</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">high，非 critical</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">受影响起点</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">GitLab 7.0（2014 年）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">仅 1.107.0 一个版本号</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">全部在支持与已停止支持分支</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">利用条件</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">厂商称「特定条件」，未公开</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无门槛，装了即中</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">需送达压缩数据，可靠性依赖构建形态</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">公开 PoC</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">载荷样本已被安全厂商公开分析</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">在野利用</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">未发现，未入 KEV</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无证据，但凭据外泄按失陷处理</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无公开报告</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">典型后果</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">GitLab 主机被接管，源码与 CI 凭据外泄</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">整机凭据被拖走加持久后门</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">服务进程被控制，可致 RCE</span></section></td></tr></tbody></table>  
  
这张表最值得注意的一行是最后一行。CVE-2026-110264 的爆炸半径是   
一整套研发资产  
；LiteLLM 事件的爆炸半径是   
整台主机上所有能拿到的凭据  
，并且攻击者已经拿到了持续投递新代码的能力；而 CVE-2026-111497 的爆炸半径是   
所有对外提供接口的 Node 服务  
，包括那些没有在依赖清单里显式出现过 Node 自身的项目。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPqKicnzWenejuVBIQiar9oXwbm90icubcp2utshDoJqvfpnOqzPntO1sXFJ746Ap8EJNLIOuOrTRV4ibhMZjRI006lpPd8aoD2ETuo/640?wx_fmt=png&from=appmsg "")  
  
— 三起事件的横向对比：类型、门槛、证据与防守动作  
  
  
02  
  
GITLAB  
### CVE-2026-110264：埋在正则引擎里的十二年老弹  
  
  
2.1　产品与用户量  
  
GitLab 是全球使用最广的私有化代码托管平台之一。它的形态是   
单一应用  
：代码托管、CI/CD 流水线、制品库、容器镜像仓库、漏洞扫描与 wiki 全都在同一个 Ruby on Rails 应用里，由同一个团队统一发布。对企业来说，这意味着「一套系统承载全部研发资产」；对防守来说，意味着   
一个 Web 入口后面就是全部源码与流水线凭据  
。  
  
官方投资者关系页面披露，平台拥有   
超过 5000 万注册用户  
，并服务了   
《财富》100 强中的半数企业  
。但在本次事件中，真正决定风险规模的是其中占比更大、也更难被外部监测到的   
自托管（Self-Managed）部署  
。根据此前针对 GitLab 另一路径穿越漏洞的公开统计，全球互联网暴露的自托管 GitLab 资产超过   
130 万个  
，这些实例由企业自建在自有服务器或云主机上，多数不在任何外部扫描视野内。  
  
2.2　影响范围与危害程度  
  
漏洞编号 CVE-2026-110264，CVSS v3.1 评分   
10.0  
，向量为   
AV:N/AC:L/PR:N/UI:N/S:C  
，即网络可达、低复杂度、无需权限、无需用户交互，且影响范围跨越安全边界。根因是   
服务器编译特制正则表达式时发生整数溢出  
（CWE-190），进而导致以 GitLab 服务账户权限执行任意代码。  
  
真正放大这个漏洞的，是它的   
受影响版本区间  
。GitLab 官方公告写得很直白：所有自托管 CE 与 EE 版本，  
从 7.0（2014 年发布）起全部受影响  
。原因在于出问题的组件并不是 GitLab 独有代码，而是   
Ruby 内置的正则引擎 Onigmo  
——GitLab 将其随自己的软件包一同分发并自行打补丁，引擎的弱点因此被带进了此后每一个版本。  
  
这里有一个极易被忽略的   
二次伤害  
。就在 9 天前的 10 月 12 日，GitLab 已经因同类的正则编译器溢出漏洞 CVE-2026-97229（同样 10.0 分）发布过一次紧急更新，但那次更新只覆盖到 19.2 及以后的分支。本次修复版本正好比它高一个补丁号——换句话说，  
凡是 10 月 12 日按指引完成升级的运维，仍然处在本次漏洞的影响范围内  
。资产清单上写着「已升级至最新」而实际版本号落后的实例，在这轮事件中数量可能相当可观。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPq6QnQTmlRwukT7z8X64SzHLVI5JRha48VEhY7JPDJZ6PWUMpyK5bK4tP2MpaiaE2z3rQibOvPoXpD4xKy6GNWhfVwQwMc1OuLTc/640?wx_fmt=png&from=appmsg "")  
  
— 从 GitLab 7.0 到 19.5：受影响的整条发布历史  
  
危害等级判定为「严重」：CVSS 10.0 加未认证、加无需交互、加无临时缓解措施（官方未提供 workaround）。  
  
2.3　利用条件  
  
GitLab 的表述是「在特定条件下」，而   
这些条件至今未被披露  
。这一句话直接决定了防守的难度：如果触发前提是公开的，管理员可以通过关闭某项功能、检查某类配置来自证清白；而在前提未知的情况下，  
任何配置层面的自查都无法证明自己不受影响，唯一的确定防护就是打补丁。GitLab 自己也明确表示，管理员不应假设该漏洞必须先获得某个项目或某项功能的访问权限。  
  
从公开的旁证看，攻击面至少横跨两条路径：其一是   
CI/CD 配置中的正则表达式编译  
，其二是   
Org-mode 文件渲染  
——GitLab 官方发布的检测规则名称「Onigmo native worker crash」与「Org mode render error burst」正对应这两处。  
  
2.4　有无在野 PoC 与利用  
  
截至 2026 年 10 月 22 日，公开信息呈现的状态是   
无公开 PoC、无在野利用证据  
。GitLab 官方表示未发现该漏洞被利用的迹象；该编号   
尚未进入 CISA 已知被利用漏洞目录（KEV）  
；CANSIE 与 CERT-EU 也未发布与之相关的在野预警。  
  
但有一组对比值得警惕。同一研究者（网名 joaxcar）在一个多月内已四次报告 GitLab 正则引擎的代码执行漏洞：9 月 23 日的 CVE-2026-93577 与 CVE-2026-89078（均为 9.9 分，但需要登录账户）、10 月 12 日的 CVE-2026-97229（10.0 分，未认证），以及本次 CVE-2026-110264。其中 CVE-2026-97229 已被 Rapid7 转化为可用利用代码。前三次的修复区间均止于 19.2，  
这一次则一路回溯到 7.0  
。  
  
一个合理的判断是：同一位置在短期内被反复挖出四个洞，  
攻击工具化的时点通常距离披露并不遥远。GitLab 提供的两条 Sigma 规则（隶属其开源 TLDR 检测框架）可用于在升级窗口期内做日志侧排查，但官方同时提醒，  
成功利用并不一定导致进程崩溃，没有告警并不能证明环境干净。  
  
2.5　修复建议  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">当前分支</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">升级至</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.5</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.5.1</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.4</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.4.3</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.3</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.3.6</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.2.9</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">19.1.9</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">19.0.10</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">7.0 至 18.11</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">18.11.13</span></section></td></tr></tbody></table>  
  
除升级外，官方与第三方的补充建议包括：  
  
1  
升级窗口期内，把   
Onigmo native worker crash  
 与   
Org mode render error burst  
 两条 Sigma 规则部署进 SIEM，覆盖 GitLab 应用日志。  
  
2  
在升级完成前，用   
VPN 白名单、WAF 规则、网络分段  
 收敛 GitLab Web 入口的可达范围。  
  
3  
升级完成后，  
轮换实例内的 CI/CD Token、部署密钥与存储凭据，并检查是否存在异常的 CI 任务运行、陌生进程与被修改的部署令牌。  
  
4  
19.3.6 与 19.4.3 包含数据库迁移，单节点部署会中断服务，多节点可走零停机流程；18.11 之前的分支已无对应修复版本，  
只能迁移到受支持分支。  
  
  
03  
  
SUPPLY CHAIN  
### LiteLLM 1.107.0：一次已经成立的供应链入侵  
  
  
3.1　产品与用户量  
  
LiteLLM 是 BerriAI（YC W23）出品的   
统一 LLM 调用层  
。它的用途可以拆成两层：作为 Python 库，它给应用提供一把统一的调用入口；作为自托管服务，它就是一个   
企业内部的 AI 网关  
，把 OpenAI、Anthropic、Azure、Vertex 等 100 多家提供商的接口收敛成一套 OpenAI 兼容协议，同时承担密钥托管、虚拟密钥、配额与审计日志。它通常不是业务系统的主角，却是   
所有模型调用的必经之路  
。  
  
项目官方在 10 月初的社区里程碑通报中称，其 GitHub 星标已   
超过 6 万  
，PyPI 月下载量   
8900 万次以上  
，容器拉取量   
超过 5 亿次  
。第三方仓库画像平台的数据显示，其周下载量约 2300 万，被   
2.6 万个仓库与软件包  
直接依赖。  
  
这个体量放在供应链视角下会显得格外刺眼。LiteLLM 处在绝大多数 LLM 部署的   
凭据平面  
上：它天然持有 OpenAI、Anthropic、Azure、Vertex 等多家提供商的 API Key，配置里通常还有虚拟密钥、数据库口令与内部服务地址。依赖量越大，  
一次自动安装的爆炸半径就越广。  
  
3.2　影响范围与危害程度  
  
10 月 21 日清晨，LiteLLM 官方发布《Contained: Oct 21, 2026 LiteLLM release v1.107.0 Compromised》，确认发布流水线被入侵。按官方时间线还原如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPpA0IszwtX8DNnB10ls4qMIWjsPqOqSRU5U1hYVx07qLh74bgocSDaQ6CZXYcOByPJZnic7uCy4GH7I2EzwPIKIJUr8VPV2Ycyo/640?wx_fmt=png&from=appmsg "")  
  
— 从一枚泄露的 OAuth Token 到一次已签名的正式发布  
  
10:08  
  
GitLab 分支保护拒绝了针对私有发布工作流的一次未授权修改；几秒后，属于   
组织所有者之一 的凭据被用于关闭分支保护并强制推送该改动。  
  
  
10:46 至 11:08  
  
被篡改的工作流启动正式 staging 运行，源码指向攻击者控制的   
仿冒仓库 Berri-AI/litellm  
（官方为 BerriAI/litellm），staging 顺利通过。  
  
  
11:39 至 13:23  
  
七个恶意 wheel 被上传至 PyPI，随后 PyPI 将该版本下架。官方 GitHub 上的 v1.107.0 源码 tag   
本身不含恶意文件，污染发生在构建阶段。  
  
  
11:46 至 14:20  
  
Docker Hub 与 GHCR 的 v1.107.0、latest、main-latest、main-stable 四个标签被指向受污染镜像索引，其中 Docker Hub 侧直到 14:20 UTC 才完成移除或重指。  
  
  
11:53  
  
第三个开发者账号的凭据被用于删除生产与 staging 的工作流记录，直接减少了可供溯源的工作流日志。  
  
攻击链的另一端同样值得记录。官方在通报中确认了   
三套开发者 GitHub CLI OAuth Token 泄露  
，泄露路径是：LiteLLM 内部部署实例暴露在公网且存在一个不起眼的异常账号，攻击者进入后从日志中捞出了这些 Token——而某些场景下 Claude Code 会把用户的 GitHub CLI OAuth Token 写进响应体、响应体又被记录进日志。官方同时承认，  
Token 是如何被取得的至今尚未查清，也未确认相关账号已无残留访问权。  
  
从防守视角看，恶意载荷的设计把危害放大到了主机级别。Upwind、SafeDep、Socket、StepSecurity 等团队的分析显示，载荷通过滥用 Python 的   
.pth 机制与 sitecustomize.py 自动执行  
，意味着   
即便从未 import 过 litellm，只要该环境启动过一次 Python，恶意代码就会执行。收集范围包括云访问令牌、SSH 与 GPG 私钥、浏览器数据、AI 编码工具配置、LiteLLM 代理数据库中的凭据，随后发往   
仿冒域名 www.litellm.dev（与官方 litellm.ai 无关）。同时它会派生后台进程、创建   
/tmp/remote_code_loader.lock，并每小时从 GitHub 拉取   
domain.txt 解析新域名再下载执行新的 Python 代码——这意味着攻击者在包被下架之后，依然可以持续向已感染环境投递新载荷。  
  
NOTE  
  
危害等级判定为「严重」：这是本轮三起事件中唯一一起   
已被官方确认成立  
 的攻击事件，且同时具备凭据窃取、远程代码执行与持久化驻留三重能力。  
  
3.3　利用条件  
  
这是三起事件中利用条件最低的一个。触发条件只有两条：  
  
1  
在 11:39 至 13:23 UTC 之间，以   
未固定版本  
 的方式从 PyPI 安装了 litellm（即拉到了 1.107.0）。  
  
2  
在 11:46 至 14:20 UTC 之间拉取并运行了受影响标签的容器镜像，包括   
v1.107.0、latest、main-latest、main-stable  
，涉及 litellm、litellm-database、litellm-non_root 等镜像变体。  
  
不需要任何认证、任何配置改动、任何可利用链，  
也不需要攻击者做后续侦察。相比之下，CVE-2026-110264 至少还要求攻击者构造出能触发整数溢出的特定正则与调用路径；CVE-2026-111497 至少还需要把一段特制压缩数据送到解码器面前，并依赖运行时构建形态才能稳定触发。从门槛角度看，  
投毒是唯一「装完就中」的那个。  
  
3.4　有无在野 PoC 与利用  
  
需要区分两件事。  
  
其一，恶意载荷的样本与分析是公开的。Upwind 是首个识别出该投毒的研究团队，并向维护者提交了 GitHub issue [#47826]()  
；StepSecurity 进一步发现产物构建自仿冒仓库，恶意内容叠加在干净的 v1.107.0 tag 之上；SafeDep 记录了凭据窃取器与后门的完整行为。相关 IOC 也已公开：  
  
.  
.  
.  
ioc  
  
cook.pth / cook.py / sitecustomize.py  
  
/tmp/remote_code_loader.lock  
  
www.litellm.dev  
  
raw.githubusercontent.com/letters9813/pythonclient-test/main/domain.txt  
  
http://{domain}/codepy  
  
litellm-1.107.0.dist-info  
  
其二，就「是否有真实攻击者利用」这个问题，公开信息并未给出确认结论。官方通报的措辞是「事件已控制、调查进行中」，没有报告具体的攻击活动；GitHub issue [#47826]()  
 下的讨论同样以取证与披露为主。因此更准确的表述是：  
攻击能力已经就位并被完整公开，但尚未有公开证据表明已被大规模武器化到具体受害主机。  
  
不过 LiteLLM 官方在处置建议中给出了一条非常关键、也容易被低估的判断：  
只要 1.107.0 曾在具备外网访问能力的主机上安装过，就应当假定数据外泄已经成功。官方特别说明，标记文件不存在并不能证明环境干净——容器重建与临时目录清理都会抹掉痕迹。这是本次事件中最难回答的一个问题：  
入侵范围不可验证。  
  
另有一条容易被忽略的时间线背景：这是 LiteLLM   
同年内的第二次供应链事件  
。3 月的投毒事件（1.82.7 与 1.82.8 区间）被公开报道与 TeamPCP 系列攻击关联在一起；官方在本次通报中承认，3 月事件之后已加固发布流水线，改为私有仓库、Trusted GitHub Publisher、镜像不可变标签与镜像签名，但   
加固措施在本次攻击中仍被逐个绕过  
。  
  
3.5　修复建议  
  
1  
不要用 pip 清理环境。在受影响环境里，任何一次 Python 启动都会重新执行 .pth 钩子，包括运行 pip 本身。  
  
2  
从 shell 手工删除所有 site-packages 中的   
cook.pth、cook.py、sitecustomize.py  
 及对应的 __pycache__ 缓存文件，然后杀掉持有   
/tmp/remote_code_loader.lock  
 的分离 Python 守护进程。  
  
3  
重新安装并   
固定到 litellm==1.106.2  
，且按 digest 锁定而非仅锁版本号；更稳妥的做法是直接重建镜像与虚拟环境。  
  
4  
在   
干净环境  
 中轮换该主机能触达的全部凭据：实例或任务 IAM 角色、存储在 LiteLLM 代理数据库中的全部提供商 Key 与主密钥及虚拟密钥、DATABASE_URL 口令、主机上任意进程环境变量中的密钥、SSH 与 GPG 私钥、浏览器密码、AI 编码工具令牌、CI 与 Vault 机密。  
  
5  
在 CloudTrail 中检索安装时间前后实例角色的   
GetCallerIdentity  
 调用，并追查该角色凭据是否在组织外部被使用；同时排查异常出站连接、异常 Python 进程、GitHub 上的可疑发布与审批活动。  
  
  
04  
  
RUNTIME  
### CVE-2026-111497：底座上的一道缝  
  
  
4.1　产品与用户量  
  
Node.js 是服务端 JavaScript 运行时，也是 npm 生态的底座。Express、NestJS、Next.js 这些框架，以及大量 CLI 工具、前端构建链路与 serverless 函数，都跑在它之上。第三方统计显示，官方渠道在   
2026 年 1 至 7 月累计下载 46.1 亿次  
，已经超过 2025 全年的 44.8 亿次；仅 7 月单月，Node 24 约 3.588 亿次、Node 22 约 2.839 亿次，而   
已停止支持的 Node 20 仍有约 1.037 亿次  
、Node 18 约 3270 万次。项目在 GitHub 上约有 11.9 万星标，整个 npm 生态超过 200 万个包，周下载合计超过 100 亿次。  
  
这组数字里最该被记住的不是 46 亿，而是那条   
已停止支持却仍有月度千万级下载  
 的旧版本线。它意味着在任意一台跑 Node 服务的机器上，都存在「装得上补丁、但装的不是受支持版本」的现实，而这类资产在任何依赖清单里往往只写着一条模糊的版本区间。  
  
4.2　影响范围与危害程度  
  
CVE-2026-111497 是 Node.js 内置   
Brotli 解码器中的堆内存越界写  
，可导致远程代码执行。官方在 10 月安全更新中将其定级为   
high  
，并明确说明「本次发布中修复的最高严重度问题为 high」。影响范围覆盖 22.x、24.x、26.x 三条受支持分支；官方同时写明，  
已停止支持（EOL）的版本始终受影响  
，18.x、20.x 与 25.x 没有补丁可打。  
  
根因在代码层面相当明确：在 Brotli 解码过程中进行   
块类型切换时校验失败没有被向上传播，三个返回值为 void 的封装函数直接丢弃了失败结果，解码器带着一个尚未赋值的块类型继续工作，随后越界索引 Huffman 表与上下文映射结构。该漏洞由 Node.js 技术委员会成员 Matteo Collina 报告，由 Antoine du Hamel 修复。  
  
需要留意的旁注是：同一编号   
CVE-2026-111497  
 也出现在 Istio 与 Envoy 侧的 Brotli 库公告中（CVSS 7.7，影响 Istio 1.29 至 1.31）。两边根因不同、修复版本不同，检索与处置时不要混淆。  
  
4.3　利用条件  
  
与前两个事件不同，这个漏洞的利用条件相当「日常」。攻击者只需要让一段特制的 Brotli 压缩数据走到解码器面前，而这条路径在现代 Node 服务里几乎无处不在：  
  
1  
内置   
fetch()  
 会自动解压收到的 HTTPS 响应体。任何一处消费外部 API 响应并把结果交给业务逻辑的代码，都是潜在入口；恶意内容只要来自一个被劫持或伪造的上游即可。  
  
2  
框架侧同样会自动解压：  
body-parser 2.1.0 及以上 与   
@fastify/compress 会在鉴权之前先解压请求体。也就是说，一个未认证的攻击者只要能把特制的 br 请求体发到接口，解码就已经发生。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPqyCeH5qoR6opBialPic6wezQANP1iaRXcovgem4yVGgRWrfpVD7ibEn7ePiaTibrnYCqu2p1Q5L8NWaF5pdKvl1UnsShPoNCrPAUaW4/640?wx_fmt=png&from=appmsg "")  
  
— 从特制压缩数据到堆内存越界写，再到代码执行  
  
门槛不在「能不能到达」，而在「能不能稳定拿到控制权」。官方与研究者均指出，  
利用的可靠性取决于运行时版本、构建形态与内存分配器布局，这正是它被定为 high 而非 critical 的原因：内存破坏类缺陷能否转化为稳定的代码执行，很大程度上取决于目标进程的堆布局。这也解释了为什么它不能被简单当作一个普通的远程代码执行来对待——  
攻击成功率不稳定，不等于攻击门槛高。  
  
NOTE  
  
危害等级判定为「高」：远程、无需认证、可致代码执行，且命中面覆盖主流 Web 框架的默认解压路径；但因利用可靠性受构建形态影响、官方定级为 high，处置优先级应排在同批 critical 之后、常规高危之后。  
  
4.4　有无在野 PoC 与利用  
  
截至 2026 年 10 月 22 日，  
没有任何公开的利用代码，也没有在野利用报告。官方在公告中未提及任何已被利用的迹象，该编号同样   
未进入 CISA 已知被利用漏洞目录。对于内存破坏类漏洞，这个状态属于常见且正常的窗口期：从补丁发布到首个公开利用出现，往往只有数周。  
  
这次同批更新里还有三个同为 high 的条目值得一并记录，因为它们描述的正是「补丁日」的实际工作量：CVE-2026-105482（DNS 响应中的 NUL 字符截断）、CVE-2026-105483（全角数字形式的 IP 地址导致 TLS 主机名校验绕过）、CVE-2026-106069（宽松解析导致的同类 TLS 绕过）。此外本次还更新了 c-ares、ca-cert、OpenSSL、SQLite 与 Undici 等依赖，其中 ca-cert 的更新与同一天被披露的   
eMudhra 中间证书颁发机构密钥泄露  
 直接相关。  
  
4.5　修复建议  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">当前分支</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">升级至</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">22.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">22.23.4</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">24.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">24.22.1</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">26.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">26.12.1</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">18.x、20.x、25.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">已停止支持，无补丁，需迁移</span></section></td></tr></tbody></table>  
  
除升级外，官方与第三方的补充建议包括：  
  
1  
用实际运行进程的基础镜像版本而不是   
node:latest 标签核对版本；把 22.23.4、24.22.1、26.12.1 三个版本号写入资产清单的硬约束。  
  
2  
为仍在 EOL 分支上的服务制定迁移计划。在 Node 20 仍有月度千万级下载的情况下，  
「等下一个大版本」本身就是一个长期风险敞口。  
  
3  
在网关或反向代理层限制对外部输入的压缩编码白名单，例如只接受 gzip 与 identity，避免把任意 br 数据直接送到应用层解码。  
  
4  
本次更新同时带上了三个 TLS 与 DNS 相关的 high 条目与一批依赖更新（c-ares、ca-cert、OpenSSL、SQLite、Undici），  
建议整批升级而不是只挑一个版本号跳上去，以免留下同一批次的已知问题。  
  
  
05  
  
ANALYSIS  
### 为什么这三件更值得占用你的时间  
  
  
以下为分析观点，与上文事实部分分开陈述。  
  
5.1　防守动作的性质不同  
  
CVE-2026-110264 的处置是   
排查型动作  
：除了升级版本，还必须回答「我这台到底有没有被动过」，而官方的检测规则又明说成功利用不一定崩溃。LiteLLM 的处置是   
重建型动作  
：不能靠清理，需要重建，需要全量轮换凭据，需要在组织内部横向核查其他 AI 工具是否共享了同一批凭据。CVE-2026-111497 则介于两者之间，它是   
平台型动作  
：一次基础镜像或运行时版本的全量抬升，决策成本低、执行成本高、覆盖范围取决于你的节点清单有多准。三者的防守成本不在一个量级上，优先级也不该一样。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPqmlYU68Cwjuyku3yFSVhXaZuYBzhry6j2o5n7QRicQQCoS8wWibCm7Vq9GcHDuQCZjey9Jvt4ia6uId1PkdusdntXxG8GWLRG3Og/640?wx_fmt=png&from=appmsg "")  
  
— 排查型、重建型与平台型动作的成本差异  
  
5.2　三种不同的失效半径  
  
CVE-2026-110264 是「窄但深」：触发条件未知、无法用配置自查排除，但自托管实例往往承载着企业内部最敏感的代码库与流水线凭据。CVE-2026-111497 是「宽但浅」：命中面几乎覆盖全部 Node 服务，可它的利用可靠性受构建形态影响，单个实例上的成功率并不确定。LiteLLM 则是   
广且深  
：8900 万次月下载量级、2.6 万个直接依赖仓库、5 亿次容器拉取，命中一个装在 CI 或开发机上的实例，就可能同时拿到云 IAM、代码仓库、浏览器会话与全部模型提供商的 Key。对应的处置策略也不一样：窄而深的要靠资产清单加日志溯源，宽而深的要靠平台侧的批量抬升版本，广而深的只能按失陷处理并重建。  
  
5.3　三者都难以靠日志证明「没被攻击」  
  
这是它们共同的、也是最容易被忽视的一点。GitLab 明确表示成功利用不必然导致 worker 崩溃，所以没有匹配到 Sigma 规则不等于安全；LiteLLM 官方则明确表示标记文件不存在不代表环境干净，容器与临时目录清理会抹除痕迹；Node.js 这个内存破坏漏洞的利用成功率本身就不稳定，  
攻击发生与否，本来就不一定在日志上留下痕迹  
。三个事件都不提供「阴性证据」，安全团队无法用「没查到日志」来结案。  
  
5.4　修补的是信任边界，不是代码  
  
CVE-2026-110264 修的是一个第三方引擎在某个发行版里的长期分发；CVE-2026-111497 修的是一个以「失败即返回 void」的方式层层传递校验结果的设计惯例；LiteLLM 事件里被攻破的则是   
「谁能向公共注册表发布这个项目的正式版本」这个信任边界  
。官方在 3 月事件后已经做了加固——私有仓库、Trusted GitHub Publisher、不可变镜像标签、镜像签名——但攻击路径依然从「内部部署实例暴露在公网 + 日志里躺着 OAuth Token」这一环切入，绕到了「临时放宽发布权限以支持客户每日热修复」这一步上。  
这提示一个更普遍的问题：安全加固如果只覆盖代码路径而不覆盖凭证暴露面与权限授予节奏，链路仍然可被绕过。  
  
5.5　一个值得留意的巧合  
  
10 月 21 日这一天，还同时出现了 eMudhra 证书颁发机构密钥可被破解、TiDB Cloud 数据泄露、Railway 密钥暴露等事件。安全研究者 Mudit Gupta 将这一天形容为网络安全界的「噩梦日」。  
同一天内多起重大事件的密度，本身就是一个信号：AI 工具正在同时改变两件事——发现漏洞的速度，以及实施攻击的成本。这对防守方是同向的：谁先把「资产清单、版本核验、凭据轮换、供应链固定」这条流水线建起来，谁就能把单点事件的影响压到最小。  
  
  
06  
  
CHECKLIST  
### 72 小时内的处置清单  
  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">优先级</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">动作</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">适用对象</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">P0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">查版本、查镜像 digest、查 1.107.0 残留文件</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">所有 LiteLLM 使用方</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">P0</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">在干净环境轮换云 Key、IAM 角色、SSH 与 GPG 私钥、代理库中的全部提供商 Key</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">确认装过 1.107.0 的主机</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">P1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">全量盘点自托管 GitLab 实例的实际版本号，而非按补丁日期</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">所有 GitLab 运维方</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">P1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">部署两条 Sigma 规则，保留升级前的日志证据</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">GitLab 运维方</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">P1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">重建镜像与虚拟环境，固定到 1.106.2 并按 digest 锁定</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">所有 LiteLLM 使用方</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">P1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">核对实际运行的 Node 版本，抬升至 22.23.4、24.22.1 或 26.12.1</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">全部 Node 服务运维方</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">P2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">统计仍跑在 18.x、20.x、25.x 分支的 Node 服务，制定迁移计划</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">平台与架构团队</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">P2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">在网关层限制对外输入的压缩编码白名单，避免任意 br 数据直达应用层</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">有公网入口的 Node 服务</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">P2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">审查依赖内部 AI 工具链路的其他出网服务是否共享了同一批凭据</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">使用 AI 网关的团队</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">P2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">排查内部自托管 AI 网关是否同样暴露在公网、日志是否记录了敏感令牌</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">自建 AI 网关的团队</span></section></td></tr></tbody></table>  
  
给同时使用这几类产品的团队一句实话：在确认 LiteLLM 1.107.0 是否曾在你的一台外网可达主机上运行过之前，请优先把这一条当作「已经失陷」来处理。凭据轮换的成本，永远低于事后证明没有失陷的成本。  
  
∞  
  
THE END  
### 写在最后  
  
  
把这三起事件放在一起看，会发现一个不太舒服的结论：  
在当前的披露节奏下，防守方真正稀缺的不是补丁，而是可信的资产清单  
。CVE-2026-110264 的问题在于有多少自托管实例以为自己已经升到最新；CVE-2026-111497 的问题在于有多少服务跑在已经停止支持、根本拿不到补丁的 Node 分支上；LiteLLM 事件的问题在于有多少 CI 流水线在无人确认的情况下拉过   
latest 标签  
。  
  
SOURCES  
  
信息主要来源：LiteLLM 官方安全事件通报《Contained: Oct 21, 2026 LiteLLM release v1.107.0 Compromised》（docs.litellm.ai）；GitLab 官方补丁发布说明 19.5.1 / 19.4.3 / 19.3.6 及官方检测文档（docs.gitlab.com）；Node.js 官方博客《October 2026 Security Releases》与官方下载统计（nodejs.org）；Upwind、SafeDep、Socket、StepSecurity 的分析报告；GitLab 投资者关系页面、NodeSource 与第三方仓库画像平台的下载量快照；W3Techs 2026 年 9 月技术统计。  
  
END  
  
我是 网络安全透视镜：专注源码安全审计与漏洞复现。  
  
如果你觉得今天这篇有收获，欢迎**点赞、在看、转发**  
三连，我们下篇见。  
  
  
