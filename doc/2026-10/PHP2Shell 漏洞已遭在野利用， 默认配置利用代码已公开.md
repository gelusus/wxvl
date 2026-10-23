#  PHP2Shell 漏洞已遭在野利用， 默认配置利用代码已公开  
原创 360漏洞研究院
                    360漏洞研究院  360漏洞研究院   2026-10-22 22:00  
  
PHP 官方发布紧急安全更新，修复两项严重远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619，CVSS 4.0：均为 9.2）。未经身份认证的远程攻击者在满足特定条件时，可利用 PHP 核心内存状态处理缺陷，在目标服务器上执行任意代码，可能造成网站被篡改、敏感信息泄露或服务器失陷。  
  
  
**利用前置条件：**  
  
攻击者能够通过网络访问运行受影响 PHP 版本的 Web 服务  
  
目标 PHP 运行环境满足相应漏洞触发条件，具体取决于 PHP 运行模式、配置及应用处理流程  
  
  
**目前360图龙锋 · 漏洞挖掘智能体已成功复现该漏洞。**  
本文包含完整影响范围、正式修复方案、技术原理与复现细节。  
  
  
<table><tbody><tr style="box-sizing: border-box;"><td colspan="4" data-colwidth="100.0000%" width="100.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;background-color: rgb(100, 130, 228);box-sizing: border-box;padding: 0px;"><section style="text-align: center;color: rgb(255, 255, 255);box-sizing: border-box;"><p style="margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞概述</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞名称</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">PHP 远程代码执行漏洞</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞编号</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-109618</span></p><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-109619</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">公开时间</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">2026-10-21</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">POC状态</span></span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞类型</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">远程代码执行</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">EXP状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">利用可能性</span></span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">高</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(0, 0, 0);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">技术细节状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">CVSS 4.0</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">9.2</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">在野利用状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已发现</span></strong></p></section></td></tr></tbody></table>  
  
  
**01**  
  
**漏洞影响范围**  
  
  
  
受影响版本范围：  
  
PHP 8.2 分支 < 8.2.35  
  
PHP 8.3 分支 < 8.3.36  
  
PHP 8.4 分支 < 8.4.27  
  
PHP 8.5 分支 < 8.5.12  
  
PHP 8.1 及更早版本同样可能受到影响，由于这些分支已停止官方常规安全维护，建议升级至受支持版本。  
  
  
**02**  
  
**修复建议**  
  
  
  
**正式防护方案**  
  
升级至对应分支的 PHP 版本或更高的已修复版本  
  
PHP 8.2 分支 >= 8.2.35  
  
PHP 8.3 分支 >= 8.3.36  
  
PHP 8.4 分支 >= 8.4.27  
  
PHP 8.5 分支 >= 8.5.12  
  
  
**临时防护方案**  
  
立即落实以下措施并重启所有 Worker：  
  
**PHP-FPM 部署：**  
- 设置 session.upload_progress.enabled = 0禁用会话上传进度  
  
- 设置 memory_limit = -1  
  
**Apache mod_php 部署：**  
- 设置 memory_limit = -1  
  
在 Web 服务器/反向代理层限制请求体大小、字段数量、multipart 部分数等。  
  
  
注意：临时措施只能降低部分攻击路径风险，无法完全消除威胁。  
  
  
**03**  
  
**图龙锋专项检测支持**  
  
  
  
针对本次 PHP 远程代码执行漏洞，360图龙锋 · 漏洞挖掘智能体提供专项安全检测支持，助力企业快速开展安全风险排查。  
  
  
专项支持期间，企业用户可通过图龙锋 SaaS 版上传经授权的 PHP 项目源码、相关运行库或软件安装包，借助图龙锋智能安全检测能力，辅助识别 PHP2Shell 相关漏洞风险及其他潜在安全隐患，提升风险发现与排查效率。  
  
  
此外，企业还应结合实际业务部署环境、PHP 运行版本及安全补丁状态开展综合核查，及时完成安全更新与风险处置，进一步筑牢业务系统安全防线。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dZ7ia5iaWFzz8NRLmicQZmpgIquxK9ZAstOd8mxpeMiaKOXIEYJPFnEloojqDOUTYzWJXNt2YZMNs5sgqQLzhBuR6CFyegFxpicWRKrYibnWbW9I0/640?wx_fmt=png&from=appmsg "")  
  
扫描二维码 获取360图龙锋专项支持  
  
  
**04**  
  
**漏洞描述**  
  
  
  
近日，PHP 官方披露了 PHP 核心中存在的两项远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）。前者与内存不足时的异常中断有关：部分请求数据尚未完成初始化，执行流程即被中断，后续清理可能错误处理无效内存状态。后者涉及跨请求残留的全局状态：一个请求结束后，工作进程仍可能保存指向已释放内存的引用，后续请求在正常初始化之前读取该状态，从而触发释放后使用。研究人员已验证，在不同的部署和应用条件下，这些缺陷可能造成敏感内存暴露或以 Web 工作进程权限执行代码。  
  
  
**05**  
  
**漏洞复现**  
  
  
  
360漏洞研究院依托图龙锋漏洞挖掘智能体已成功复现 CVE-2026-109618、CVE-2026-109619。官方 php:8.4.26-apache 镜像（Apache mod_php、默认 ini、display_errors=Off、  
单worker  
）：未认证纯 HTTP 完成 chunked OOM 后释放的会话状态被同 worker 的下一个请求使用 → 800KB 堆越界读（通过Set-Cookie 泄露）→ libphp/堆基址双泄露绕过 ASLR → 伪造 HashTable（pDestructor=zif_system）→ 以 www-data 权限执行命令。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/dZ7ia5iaWFzzibbNQFg9FSmv1J0gWwSDhK2EhubycWCrPFFOEbqD63pHnK1ERTsUYQt6ORicFQBexCwttupBHo44gu9nVUnZkTaRy6bEGNCIFW0/640?wx_fmt=png&from=appmsg "")  
  
CVE-2026-109618 CVE-2026-109619  
  
PHP 远程代码执行漏洞复现  
  
  
**06**  
  
**产品侧支持情况**  
  
  
  
**360安全智能体：**  
支持该漏洞攻击的智能分析**。**  
  
**360测绘云 Quake**  
：默认支持该产品的指纹识别。  
  
**360高级持续性威胁预警系统**  
：预计 2026年10月23日发布规则更新包，支持该漏洞利用行为的检测。  
  
**360资产与漏洞检测管理系统**  
：预计 2026年10月23日发布规则更新包，支持该漏洞利用行为的检测。  
**本地安全大脑**  
：默认支持该漏洞的PoC检测。  
  
  
**07**  
  
**时间线**  
  
  
  
2026年10月21日，360漏洞研究院发布首次安全风险预警。  
  
2026年10月22日，360漏洞研究院更新二次安全风险通告。  
  
2026年10月23日，360漏洞研究院更新三次安全风险通告。  
  
  
**08**  
  
参考链接  
  
  
  
https://aws.amazon.com/cn/blogs/security/amazon-threat-intelligence-identifies-php2shell-exploitation-prior-to-public-disclosure-in-madpot/  
  
https://watchtowr.com/intelligence/php-cve-2026-109618-faq/  
  
https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
  
https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
  
09  
  
更多漏洞情报  
  
  
  
“扫描下方二维码，进入公众号粉丝交流群。更多一手网安资讯、漏洞预警、技术干货和技术交流等您参与！”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dZ7ia5iaWFzz8YToicKab1BicPnEdr7jiatvQUVWSMnYTBeG5ibibgxkGAG1rF4pUdpowPcCmokOO5tp4UjjhUsos4Zf4VwE1aM9NTUz3ogfgdwwFw/640?wx_fmt=gif&from=appmsg "")  
  
  
建议您订阅360漏洞情报服务，获取更多漏洞情报详情以及处置建议，让您的企业远离漏洞威胁。  
  
  
邮箱：360VRI@360.cn  
  
网址：https://vi.loudongyun.360.net  
  
  
  
“洞”悉网络威胁，守护数字安全  
  
  
**关于我们**  
  
  
360漏洞研究院，隶属于360安全能力中心。其成员常年入选谷歌、微软、华为等厂商的安全精英排行榜, 并获得谷歌、微软、苹果史上最高漏洞奖励。研究院是中国首个荣膺Pwnie Awards“史诗级成就奖”，并获得多个Pwnie Awards提名的组织。累计发现并协助修复谷歌、苹果、微软、华为、高通等全球顶级厂商CVE漏洞3000多个，收获诸多官方公开致谢。研究院也屡次受邀在BlackHat，Usenix Security，Defcon等极具影响力的工业安全峰会和顶级学术会议上分享研究成果，并多次斩获信创挑战赛、天府杯等顶级黑客大赛总冠军和单项冠军。研究院将凭借其在漏洞挖掘和安全攻防方面的强大技术实力，帮助各大企业厂商不断完善系统安全，为数字安全保驾护航，筑造数字时代的安全堡垒。  
  
