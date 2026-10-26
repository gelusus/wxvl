#  被同一个白帽子连续破防4次？详解GitLab最新10.0高危漏洞  
原创 Kit Chung
                    Kit Chung  安全圈动向   2026-10-26 00:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bHeteOibsicdg0rUMpeePYWBicOia6rmxSUcZBghLz82wUDxtBf4Hujq50NverQSg1C0zqh9LfOia17I1rUzlKWXN9o1ZJa3Vzs4qyYxSicibamVjA/640?wx_fmt=png&from=appmsg "")  
  
**导语：**  
如果你以为上周刚打完GitLab的补丁这周就能安稳度日，那可能要让你失望了。10月21日，GitLab官方又扔出一枚“核弹”——一个CVSS评分满分（10.0）的严重漏洞。更夸张的是，这个漏洞一路可以追溯到2014年发布的7.0版本。今天，我就带大家深挖一下这个潜伏了12年的RCE漏洞到底是怎么回事。  
## 01 漏洞全景：满分威胁与12年的潜伏期  
  
这次爆雷的漏洞编号为 **CVE-2026-110264**  
。简单来说，它允许**未经身份验证**  
的攻击者在满足特定条件的情况下，直接在服务器上执行任意代码（RCE）。  
  
大家都知道，RCE加上“无权访问”基本就是渗透测试里最具破坏力的组合了。这也是为什么它的CVSS评分直接拉满到了10.0。  
  
如果你在10月12日刚刚辛苦升级到了19.2以上的版本，抱歉，你依然在射程范围内。这次的新补丁正好卡在上次更新版本的上方，这意味着几乎所有的自托管实例都需要立即采取行动。  
## 02 核心技术溯源：成也正则，败也正则  
  
作为技术人，我们最关心的肯定不是官方通告的套话，而是这玩意到底是怎么触发的？为什么会影响长达12年发布的所有版本？  
  
根本原因出在GitLab底层的**正则表达式编译器**  
上。  
  
GitLab在打包自身应用时，内置了Ruby自带的正则引擎——**Onigmo**  
。这次的漏洞正是一个典型的**整数溢出**  
。当攻击者向服务器发送一段经过精心构造的恶意正则表达式时，Onigmo在对其进行编译的过程中会导致内存中的整数溢出。这种底层的内存破坏直接为执行任意代码打开了后门。  
  
💡 **技术思考：**  
因为Onigmo引擎是深深捆绑在GitLab自身的安装包中，并由GitLab自己进行构建和补丁维护的。只要你用的是包含了这个引擎的版本（也就是从7.0至今的所有版本），底层缺陷就一直存在。这就解释了为什么一个漏洞能拥有如此恐怖的跨度。  
  
目前官方虽然还没有公开触发编译器的所有“前置条件”，但这种依赖底层引擎的漏洞，通常很难通过简单的WAF规则或修改配置文件来完美防御，**打补丁是唯一确定的出路**  
。  
## 03 漏洞检测与排查路径  
  
既然漏洞已经处于公开状态，作为防守方，我们需要立刻查看系统是否已经遭受探测或攻击。GitLab官方通过其开源的 **TLDR框架**  
 提供了两条非常关键的检测规则（基于Sigma格式）：  
- **"Onigmo native worker crash"**  
 (Onigmo原生工作进程崩溃)  
  
- **"Org mode render error burst"**  
 (Org模式渲染错误激增)  
  
**排查建议：**  
建议大家立刻去翻一下集中式日志平台（比如Graylog）里的应用日志。如果发现短时间内出现了大量与Org模式渲染相关的报错，或者底层的Onigmo进程频繁崩溃重启，这极有可能是攻击者在进行Payload测试，尝试触发整数溢出边界。  
## 04 修复方案与避坑指南  
  
GitLab这次是一口气发布了7个版本的修复补丁，甚至破例将补丁向后移植到了18.11.13、19.0.10等老版本上。官方正常的维护策略通常只覆盖当前和前两个月的版本，这种罕见的向后移植动作，足以说明该漏洞的严重性。  
  
**升级对照表如下，请对号入座：**  
<table><thead><tr style="background-color: rgb(242, 242, 242);"><th style="border: 1px solid rgb(221, 221, 221);padding: 10px;text-align: left;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">你当前运行的分支</span></span></section></th><th style="border: 1px solid rgb(221, 221, 221);padding: 10px;text-align: left;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">需要升级到的安全版本</span></span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.5  </span><span textstyle="" style="font-size: 15px; font-weight: normal">(早于 19.5.1)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.5.1</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.4  </span><span textstyle="" style="font-size: 15px; font-weight: normal">(早于 19.4.3)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.4.3</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.3  </span><span textstyle="" style="font-size: 15px; font-weight: normal">(早于 19.3.6)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.3.6</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.2  </span><span textstyle="" style="font-size: 15px; font-weight: normal">(早于 19.2.9)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.2.9</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.1 </span><span textstyle="" style="font-size: 15px; font-weight: normal"> (早于 19.1.9)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.1.9</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">19.0 </span><span textstyle="" style="font-size: 15px; font-weight: normal"> (早于 19.0.10)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">19.0.10</span></span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><strong><span leaf=""><span textstyle="" style="font-size: 15px">7.0 至 18.11  </span><span textstyle="" style="font-size: 15px; font-weight: normal">(早于 18.11.13)</span></span></strong></td><td style="border: 1px solid rgb(221, 221, 221);padding: 10px;"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 15px">18.11.13</span></span></section></td></tr></tbody></table>  
⚠️ **运维避坑高能预警：**  
  
  
对于单节点服务器的管理员，升级 19.4.3  
 和 19.3.6  
 时要特别注意，这两个版本**包含数据库结构的变更**  
。这意味着升级过程中一定会产生停机时间。大家在跑升级脚本前，一定要做好全备，并提前向业务部门申请维护窗口。如果是多节点集群架构，则可以参考官方文档执行零停机升级。  
## 05 还有哪些值得关注的更新？  
  
有意思的是，这已经是同一个安全研究员（ID: **joaxcar**  
）在一个月内向GitLab报告的**第4个**  
正则引擎相关的代码执行漏洞了。前面三个（包括10月12日修复的CVE-2026-97229，以及9月23日修复的两个9.9分漏洞）只影响到19.2版本，而这次的漏洞直接穿透到了十年前。这位白帽子师傅可谓是把GitLab的正则引擎底裤都给看穿了。  
  
除了这个满分漏洞，本次10月21日的更新包中还顺带修复了另外10个安全问题，其中有两个高危漏洞同样值得我们警惕：  
- **CVE-2026-15723 (CVSS 8.2)：**  
在管理员同时启用了SAML和LDAP自动链接的场景下，已登录用户可以借此接管他人的账户。  
  
- **CVE-2026-112212 (CVSS 7.5)：**  
OAuth和OpenID Connect端点的缓存中毒漏洞。在使用共享缓存的架构中，攻击者可以将特定登录流重定向到他们控制的恶意服务器。  
  
网络安全永远是动态博弈的过程。底层的依赖组件往往是企业安全防线中最容易被忽视的暗门。赶紧检查实例资产吧，祝大家的服务器都平平安安  
  
