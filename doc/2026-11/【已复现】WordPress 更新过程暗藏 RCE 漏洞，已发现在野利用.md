#  【已复现】WordPress 更新过程暗藏 RCE 漏洞，已发现在野利用  
原创 360漏洞研究院
                    360漏洞研究院  360漏洞研究院   2026-11-04 06:57  
  
WordPress 核心更新流程存在高危条件竞争漏洞（CVE-2026-119977，CVSS 8.1）。未经身份认证的远程攻击者，在目标执行核心更新、临时升级目录可通过 Web 访问且允许执行 PHP 文件、更新包包含相关安装文件的情况下，可利用短暂的竞争窗口实现远程代码执行。CISA 已将该漏洞纳入已知被利用漏洞目录（KEV）。  
  
  
**利用前置条件：**  
- 攻击者可以访问WP服务器  
  
- WP服务器可以访问外网  
  
- 目标使用存在缺陷的 WordPress 核心更新程序，且会自动更新或管理员手动点击一次更新  
  
**目前360图龙锋 · 漏洞挖掘智能体已成功复现该漏洞。**  
本文包含完整影响范围、正式修复方案、技术原理与复现细节。  
  
  
<table><tbody><tr style="box-sizing: border-box;"><td colspan="4" data-colwidth="100.0000%" width="100.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;background-color: rgb(100, 130, 228);box-sizing: border-box;padding: 0px;"><section style="text-align: center;color: rgb(255, 255, 255);box-sizing: border-box;"><p style="margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞概述</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞名称</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">WordPress 核心更新条件竞争远程代码执行漏洞</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞编号</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-119977</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">公开时间</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">2026-10-31</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">POC状态</span></span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞类型</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">条件竞争</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">EXP状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">未公开</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">利用可能性</span></span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">高</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(0, 0, 0);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">技术细节状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">CVSS 3.1</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">8.1</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">在野利用状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已发现</span></strong></p></section></td></tr></tbody></table>  
  
  
**01**  
  
**漏洞影响范围**  
  
  
  
4.7.0 <= WordPress <= 4.7.38  
  
4.8.0 <= WordPress <= 4.8.33  
  
4.9.0 <= WordPress <= 4.9.34  
  
5.0.0 <= WordPress <= 5.0.30  
  
5.1.0 <= WordPress <= 5.1.27  
  
5.2.0 <= WordPress <= 5.2.29  
  
5.3.0 <= WordPress <= 5.3.26  
  
5.4.0 <= WordPress <= 5.4.24  
  
5.5.0 <= WordPress <= 5.5.23  
  
5.6.0 <= WordPress <= 5.6.22  
  
5.7.0 <= WordPress <= 5.7.20  
  
5.8.0 <= WordPress <= 5.8.18  
  
5.9.0 <= WordPress <= 5.9.19  
  
6.0.0 <= WordPress <= 6.0.17  
  
6.1.0 <= WordPress <= 6.1.15  
  
6.2.0 <= WordPress <= 6.2.14  
  
6.3.0 <= WordPress <= 6.3.13  
  
6.4.0 <= WordPress <= 6.4.13  
  
6.5.0 <= WordPress <= 6.5.13  
  
6.6.0 <= WordPress <= 6.6.10  
  
6.7.0 <= WordPress <= 6.7.10  
  
6.8.0 <= WordPress <= 6.8.11  
  
6.9.0 <= WordPress <= 6.9.10  
  
7.0.0 <= WordPress <= 7.0.7  
  
7.1.0 <= WordPress <= 7.1.3  
  
  
WordPress 2.7 至 4.6 的核心更新程序可能存在相同缺陷，但未提供对应的官方修复版本。  
  
  
**02**  
  
**修复建议**  
  
  
  
**正式防护方案**  
  
受影响产品升级至以下或更高版本  
  
4.7.x系列 >= 4.7.39  
  
4.8.x系列 >= 4.8.34  
  
4.9.x系列 >= 4.9.35  
  
5.0.x系列 >= 5.0.31  
  
5.1.x系列 >= 5.1.28  
  
5.2.x系列 >= 5.2.30  
  
5.3.x系列 >= 5.3.27  
  
5.4.x系列 >= 5.4.25  
  
5.5.x系列 >= 5.5.24  
  
5.6.x系列 >= 5.6.23  
  
5.7.x系列 >= 5.7.21  
  
5.8.x系列 >= 5.8.19  
  
5.9.x系列 >= 5.9.20  
  
6.0.x系列 >= 6.0.18  
  
6.1.x系列 >= 6.1.16  
  
6.2.x系列 >= 6.2.15  
  
6.3.x系列 >= 6.3.14  
  
6.4.x系列 >= 6.4.14  
  
6.5.x系列 >= 6.5.14  
  
6.6.x系列 >= 6.6.11  
  
6.7.x系列 >= 6.7.11  
  
6.8.x系列 >= 6.8.12  
  
6.9.x系列 >= 6.9.11  
  
7.0.x系列 >= 7.0.8  
  
7.1.x系列 >= 7.1.4  
  
  
从未修复版本首次升级至安全版本时，若旧版更新程序使用包含相关安装文件的更新包，仍可能在可预测的临时目录中暴露攻击窗口。建议升级前限制外部对 wp-content/upgrade/ 目录的访问，禁止执行其中的 PHP 文件。  
  
  
**03**  
  
**漏洞描述**  
  
  
  
近日，WordPress 官方披露了核心更新流程中存在的条件竞争漏洞（CVE-2026-119977）。该漏洞源于旧版更新程序将核心更新包解压至名称可预测、可通过 Web 访问的临时目录。攻击者可在更新过程中访问临时副本中的 setup-config.php，提交由其控制的数据库配置，使临时目录生成 wp-config.php。旧版更新程序随后重新读取临时目录并复制文件，可能导致正式站点配置被覆盖，使攻击者通过预置管理员账户登录并利用插件功能执行 PHP 代码。值得注意的是，首次升级至修复版本时，旧版解压程序仍可能暴露临时安装文件，攻击者在满足特定条件时仍可实现远程代码执行。  
  
  
**04**  
  
**漏洞复现**  
  
  
  
360漏洞研究院依托图龙锋漏洞挖掘智能体已成功复现 CVE-2026-119977 漏洞。在受影响的 WordPress 测试环境中，通过核心更新过程中的临时目录访问缺陷，成功触发条件竞争，并验证目标服务器上的 PHP 代码执行，证实该漏洞可在满足特定条件时造成远程代码执行。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dZ7ia5iaWFzz8sklcw9SQtrvqNT8ibHY6rvLiaD3LVQg4lzFXyL4uUaJFaSUon6W7t6B0NGCibZn271GBtX3Tm7Q4RIouJdzagLn95PEic9BeLMDo/640?wx_fmt=png&from=appmsg "")  
  
CVE-2026-119977  
  
WordPress 核心更新条件竞争远程代码执行漏洞复现  
  
  
**05**  
  
**产品侧支持情况**  
  
  
  
**360安全智能体：**  
支持该漏洞攻击的智能分析**。**  
  
**360测绘云 Quake**  
：默认支持该产品的指纹识别。  
  
**360高级持续性威胁预警系统**  
：预计 2026年11月05日发布规则更新包，支持该漏洞利用行为的检测。  
  
**360资产与漏洞检测管理系统**  
：预计 2026年11月05日发布规则更新包，支持该漏洞利用行为的检测。  
**本地安全大脑**  
：默认支持该漏洞的PoC检测。  
  
  
  
**06**  
  
**时间线**  
  
  
  
2026年11月04日，360漏洞研究院发布本安全风险通告。  
  
  
**07**  
  
参考链接  
  
  
  
https://xint.io/blog/wordpress-core-update-attack  
  
https://nvd.nist.gov/vuln/detail/cve-2026-119977  
  
  
08  
  
更多漏洞情报  
  
  
  
“扫描下方二维码，进入公众号粉丝交流群。更多一手网安资讯、漏洞预警、技术干货和技术交流等您参与！”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dZ7ia5iaWFzz8YToicKab1BicPnEdr7jiatvQUVWSMnYTBeG5ibibgxkGAG1rF4pUdpowPcCmokOO5tp4UjjhUsos4Zf4VwE1aM9NTUz3ogfgdwwFw/640?wx_fmt=gif&from=appmsg "")  
  
  
建议您订阅360漏洞情报服务，获取更多漏洞情报详情以及处置建议，让您的企业远离漏洞威胁。  
  
  
邮箱：360VRI@360.cn  
  
网址：https://vi.loudongyun.360.net  
  
  
  
“洞”悉网络威胁，守护数字安全  
  
  
**关于我们**  
  
  
360漏洞研究院，隶属于360安全能力中心。其成员常年入选谷歌、微软、华为等厂商的安全精英排行榜, 并获得谷歌、微软、苹果史上最高漏洞奖励。研究院是中国首个荣膺Pwnie Awards“史诗级成就奖”，并获得多个Pwnie Awards提名的组织。累计发现并协助修复谷歌、苹果、微软、华为、高通等全球顶级厂商CVE漏洞3000多个，收获诸多官方公开致谢。研究院也屡次受邀在BlackHat，Usenix Security，Defcon等极具影响力的工业安全峰会和顶级学术会议上分享研究成果，并多次斩获信创挑战赛、天府杯等顶级黑客大赛总冠军和单项冠军。研究院将凭借其在漏洞挖掘和安全攻防方面的强大技术实力，帮助各大企业厂商不断完善系统安全，为数字安全保驾护航，筑造数字时代的安全堡垒。  
  
  
