#  从几千到几百万，只用了两小时：PHP2Shell漏洞全解析  
天唯科技
                    天唯科技  天唯信息安全   2026-10-22 08:53  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZibWfCgzicQNbU68NXCNH8sw9R1wBYiaT6icvH7moZbnkDB7UPWcP57YnEr5sDNDh6pssbCmuxvzQERZeMhN6Dknw/640?wx_fmt=png "")  
  
  
  
Cloudflare的CEOMatthew Prince在社交平台上公布了一组数据：  
  
补丁公布前，他们监测到的攻击尝试是**几千次**  
。  
补丁公布后的45分钟里，**几万次**  
。  
两小时内，**几十万次**  
。  
截至10月22日，  
几百万次  
。  
  
这不是一次普通的漏洞披露。这是一场在补丁上线那一刻同步开始的大规模扫描。这个漏洞的名字叫**PHP2Shell**  
，包含两个编号：CVE-2026-109618和CVE-2026-109619，严重程度都是9.2分（满分10分）。  
  
下面分三部分讲清楚：影响范围有多大、能造成什么破坏、现在该怎么处理。  
  
## 第一部分：影响范围—为什么说"几乎没人能置身事外"  
### 一、七成网站跑在它上面  
  
PHP是互联网上最普遍的网站运行环境。全世界大约七成的网站由它驱动，包括WordPress、Drupal、Joomla、Magento、Laravel、维基百科用的MediaWiki，以及大量企业内部系统和自建商城。问题出在PHP这个"运行环境"本身，而不是某个网站程序。这句话有两层含义，第二层更重要：  
  
**第一层：不挑网站。**  
 你不用WordPress、不用任何开源CMS，自己写的系统一样受影响。哪怕只有一页"你好世界"，只要PHP版本在范围内，就在射程内。  
  
**第二层：网站做得再安全也没用。**  
 攻击发生在网站程序开始加载之前。保安还没上班，门已经被撬开了。登录验证、权限控制、框架里的安全中间件—这些都要等程序跑起来才生效，而攻击在它们之前就完成了。官方已经确认：在一种叫mod_php的常见部署方式下，一个未经登录的请求就能触发攻击。  
### 二、真正的暴露量：2200万以上  
<table><thead><tr><th style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;font-weight: 700;background: transparent;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="letter-spacing: 1px;">来源</span></span></section></th><th style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;font-weight: 700;background: transparent;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="letter-spacing: 1px;">数字</span></span></section></th><th style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;font-weight: 700;background: transparent;"><section><span leaf=""><span textstyle="" style="letter-spacing: 1px;">统计口径</span></span></section></th></tr></thead><tbody><tr><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">Palo Alto Unit 42</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><strong style="font-weight: 700;"><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;font-weight: normal;">2260万</span></span></strong></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">互联网上可探测到的 PHP 实例</span></span></section></td></tr><tr><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">VirusTotal</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">510万</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">指纹库识别出的公网 IP，其中约380 万开放端口</span></span></section></td></tr><tr><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">Shodan 检索</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">190万</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: 1px solid var(--ww_base_gray_010);text-align: left;word-break: keep-all;max-width: 50%;"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">直接在HTTP头里暴露 PHP特征的</span></span></section></td></tr><tr><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: none;text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">Silas Cutler</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: none;text-align: left;word-break: keep-all;max-width: 50%;border-right: 1px solid var(--ww_base_gray_010);"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">130万</span></span></section></td><td style="color: var(--mp-color-text);padding: 8px 12px;border-bottom: none;text-align: left;word-break: keep-all;max-width: 50%;"><section><span leaf=""><span textstyle="" style="font-size: 15px;letter-spacing: 1px;">研究员个人估算</span></span></section></td></tr></tbody></table>  
  
建议优先记住2260万这个数字。Unit 42用的是主动探测，比被动指纹识别更接近真实。作为对照：过去几年闹得很大的NetScaler、BeyondTrust边界设备漏洞，涉及的是几万台级别。**这次是三个数量级的差距。**  
  
为什么这几个数字差这么多？因为绝大多数服务器的响应头里根本不报PHP版本。VirusTotal的数据显示，五分之四以上的主机是不报版本的。**这意味着你手上的资产清单，大概率是不完整的。**  
  
**三、补丁覆盖不到的"半个互联网"**  
  
这是本次事件最麻烦的地方。官方只给四个版本发了正式修复：  
- PHP 8.5 → **8.5.12**  
  
- PHP 8.4 → **8.4.27**  
  
- PHP 8.3 → **8.3.36**  
  
- PHP 8.2 → **8.2.35**  
  
8.1、8.0、7.4已经停止支持，官方只提供了需要自己编译的源码补丁—对绝大多数用户来说等于没有。按公开的版本分布  
数据折算，跑在8.1及更早分支上的 PHP 网站接近一半。再补一个时间点：**PHP 8.2将在今年12月31日停止支持。**  
现在拿到补丁的分支，两个多月后又要少一个。  
### 四、而且它在很多你没想到的地方  
  
PHP不只在网站上。它还在路由器、监控设备的管理界面、厂商打包出售的商业产品、一键部署的服务器面板镜像里。这部分设备的特点很一致：没人维护、没人知道它跑的是什么版本、也永远等不到补丁。计算机取证专家Hal Pomeranz的判断很直接：这次会留一条很长的尾巴。  
##   
## 第二部分：破坏性—不只是"服务器被打穿"  
  
多数人对RCE（远程代码执行）的理解停留在"网站可能被黑"。这次需要往深看一层，因为它有三种不同性质的破坏力。  
### 一、服务器被完全接管  
  
攻击者拿到的是网站进程的权限。接下来是标准剧本：植入后门（也就是webshell）、读取源码和配置文件、拿到数据库账号密码、把服务器当跳板往内网走。在共享主机环境下，一台机器上多个站点共存，一个站点失陷可能连累同机的其他客户。对托管服务商来说，这是最头疼的场景。  
### 二、它不需要"控制"你，就能偷走密码  
  
这个漏洞除了能执行代码，还能直接读取服务器内存里的内容—包括其他用户的密码、登录凭证、API密钥、证书。而且**不需要先拿到代码执行权限**  
。也就是说，攻击者可以只做"读取"，不做"写入"。为什么这很可怕？因为它不留痕迹。不用植入木马，不用落地任何文件，不会产生可疑进程。  
  
而现在绝大多数企业的安全监控，盯的都是"有没有恶意程序在跑"。一个程序都没跑，密码已经被拿走了—这套体系是看不见的。类似的手法在2014年Heartbleed 事件里出现过，当年让整个行业重新思考了内存安全。  
### 三、网站会被当成"信誉资产"来滥用  
  
Malwarebytes的研究负责人Shahak Shalev有个说法很到位：这对攻击者来说是座金矿。他们可以劫持真实网站，利用这个网站的声誉去做钓鱼、分发恶意软件。一个经营了十年的企业官网，突然开始给客户发钓鱼链接—损失的不只是服务器，是品牌信任。  
### 四、利用难度：有门槛，但正在快速下降  
  
目前没有公开证据表明出现了大规模的成功入侵。watchTowr的说法是"看到了攻击尝试，还没看到成功，但我们不太相信这个状态能维持住"。Red Hat也认为在默认的FPM配置下，实际利用偏难；有安全团队复现后判断，在开启了地址随机化（ASLR）的x86-64机器上，可用的公开利用代码还不多。  
  
但有几个信号值得警惕：亚马逊的蜜罐在**10月17日**  
就记录到了第一次攻击尝试—距离漏洞公开还有四天。10月21日凌晨2点左右，第一批公开的概念验证代码出现，比官方发布补丁早了约12小时。补丁上线后几小时内，多种变种载荷出现，说明不止一个攻击团伙在跟进。亚马逊的结论是：PHP2Shell已经进入"广泛利用"阶段。  
  
## 第三部分：解决方案  
### 一、唯一的完整答案：升级  
  
官方把话说得很死—不存在完整的缓解措施，升级PHP运行时是唯一安全的选择。下面这些只能降低风险，不能让一台没打补丁的服务器变安全。  
  
四个版本号再列一次，数字对上才算修好：8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
8.1及更早的：找操作系统厂商或LTS服务商要回溯补丁，或者直接升到受支持的分支。5.4到7.3官方已经不管了，只能升级。  
### 二、但"升级"这两个字，坑比想象中多  
  
以下五条是实际排查里最容易翻车的，建议直接抄进清单：  
  
**1. 别只敲一句php -v就以为查完了。**  
  
服务器上常常并存多个PHP，命令行用的那个，和真正处理网页请求的那个，往往是两个不同的程序、两份配置文件。查完一个不等于查完。  
  
**2. 版本号"看起来没修"可能只是误判。**  
  
以Ubuntu为例，它打的修复包版本号还是8.3.6-0ubuntu0.24.04.13，页面上显示8.3.6。拿8.3.36去比对，会以为没修——其实修了。要看更新日志里有没有对应的漏洞编号。  
  
**3. 面板和容器是重灾区。**  
  
宝塔这类面板显示的版本，未必是真在跑的那个版本。用Docker的必须重新拉取基础镜像重建，缓存里的旧镜像不带补丁。  
  
**4. 更新了软件包不等于修好了。**  
  
服务器上处理请求的程序是长期运行的。文件更新了但进程没重启，跑的还是旧代码。**必须重启PHP-FPM、Apache以及所有长驻的PHP进程。这一条是最常见的"假修复"。**  
  
**5. 修完要验证业务。**  
  
官方说明这次修复引入了一个新的内存硬上限，触发时进程会被直接杀掉，来不及执行收尾动作。如果你的系统依赖"关闭时写日志、刷缓存"这类操作，必须专门测一遍。  
### 三、如果今晚确实升不了  
  
官方原话是"这些措施减少特定攻击路径的暴露，但不会让安装变安全"。只能算拖时间，按性价比排序：  
  
**第一优先：关掉上传进度显示功能。**  
  
配置项是session.upload_progress.enabled。这是已知最容易被利用的入口，而且从2012年起默认就是开着的，绝大多数人根本不知道它开着。注意：必须写在php.ini、FPM池配置或服务器配置里，在代码里改太晚了；改完重启所有worker。  
  
  
**第二优先：收紧上传和请求体限制。**  
  
不需要上传的功能把上限压到8M，需要上传的压到32M。同时在服务器或代理层独立限制请求大小、字段数量、上传分段数、请求头数量、嵌套深度，而且要覆盖那些不带标准长度标识的请求。  
  
**第三优先：减少暴露面。**  
  
不需要上传文件的接口，直接在前置层拦掉这类请求。调试入口、后台管理页、不必要的对外接口，能关就关。  
  
## 第四部分：现在就能做的三件事  
  
**一、我们的PHP装在哪？**  
  
主机、容器、买来的设备、面板环境，一个都不能漏。前面那几个扫描数字差了三倍多，说明这份清单本身就不可靠。  
  
**二、它跑在哪个版本？**  
  
不是命令行显示的那个，是真正在接待访客的那个。  
  
**三、谁能保证它今晚就打上补丁，并且把所有PHP进程都重启一遍？**  
  
这三个问题答不上来，漏洞是9.2分还是10分，其实没有区别。  
  
****  
**本文为公开网安资讯整理，仅供行业交流参考。**  
  
****  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZibWfCgzicQNbU68NXCNH8sw9R1wBYiaT6icvH7moZbnkDB7UPWcP57YnEr5sDNDh6pssbCmuxvzQERZeMhN6Dknw/640?wx_fmt=png "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/gKhkoxLDlEqjUMRPHXVzOpwQDldhvJRQibQYyoKvkzxib2dDXNhaMwwC9pr1P56CG3mmmpYUnC8v5DbxZtZbB7Ultq6KxfvRK6FyKyHC9EeUA/640?wx_fmt=png "")  
  
  天唯科技专注于中大型组织IT基础设施、信息安全、数据资产、AI大模型及应用解决方案的规划、建设与持续运维服务。通过深度融合前沿技术与行业实践，帮助客户构建全方位的IT基础设施及信息安全防护体系，显著提升安全管控水平与安全运营能力；同时，依托AI技术与数据资产的高效利用，赋能企业突破数字化转型瓶颈，强化业务创新与智能决策能力，助力客户在激烈的市场竞争中保持领先优势，实现可持续的高质量发展。  
  
  我们一直秉承“精兵强将，专业专注”的发展理念。先后在江门、深圳、香港成立分公司，在武汉、长沙成立办事处以及成立广州的服务支撑中心。公司已获得高新技术企业认证、已通过IS09001、IS027001、CCRC信息安全集成服务、CCRC信息安全风险评估、CCRC信息安全应急处理等认证。  
  
****  
  
  
