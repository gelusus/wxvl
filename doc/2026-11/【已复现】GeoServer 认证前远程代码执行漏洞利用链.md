#  【已复现】GeoServer 认证前远程代码执行漏洞利用链  
原创 360漏洞研究院
                    360漏洞研究院  360漏洞研究院   2026-11-24 17:20  
  
GeoServer 是开源的地理空间数据服务器，广泛部署于政务、测绘、自然资源、应急等行业的内网与外网 GIS 服务中，通常承载 WMS/WFS/WCS 等 OGC 服务与后台管理能力。  
  
  
GeoServer 官方确认，公开研究中的认证前远程代码执行链涉及 5 个 CVE。在受影响的默认部署中，攻击者无需预先持有账号，也无需用户交互，即可能通过网络获取 GeoServer 管理员权限，最终以 GeoServer 进程的操作系统权限执行命令。研究人员对整条链的 CVSS 3.1 评分为 9.8。   
  
  
**利用前置条件：**  
  
无需任何账号，仅需网络可达 GeoServer 应用路径（如 http://host:port/geoserver）。  
  
  
**目前图龙锋已成功复现该漏洞。**  
本文包含完整影响范围、正式修复方案、技术原理与复现细节。  
  
  
<table><tbody><tr style="box-sizing: border-box;"><td colspan="4" data-colwidth="100.0000%" width="100.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;background-color: rgb(100, 130, 228);box-sizing: border-box;padding: 0px;"><section style="text-align: center;color: rgb(255, 255, 255);box-sizing: border-box;"><p style="margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞概述</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞名称</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">GeoServer 认证前远程代码执行漏洞</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞编号</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-129244</span></p><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-129243</span></p><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-120705</span></p><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-129245</span></p><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-129242（核心）</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">公开时间</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">2026-11-24</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">POC状态</span></span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞类型</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">远程代码执行</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">EXP状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">利用可能性</span></span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">高</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(0, 0, 0);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">技术细节状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">CVSS 3.1</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="word-break: break-all;white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">9.8</span><span style="box-sizing: border-box;"><span leaf="">（整链评估）</span></span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">在野利用状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">未发现</span></p></section></td></tr></tbody></table>  
  
说明：5 条 CVE 中，CVE-2026-120705 的 8.6 为厂商给出的 CVSS 3.1 评分；其余 4 条为厂商 CVSS 4.0 评分（6.9 / 8.8 / 9.2 / 8.6）。表中 9.8 为本文对“认证前整链 RCE”的评估值，非厂商官方评分。  
  
  
**01**  
  
**漏洞影响范围**  
  
  
  
受影响的软件版本：  
- 3.0.0 <= GeoServer <= 3.0.1  
  
- 2.28.0 <= GeoServer <= 2.28.5  
  
- GeoServer <= 2.27.6  
  
  
  
  
**02**  
  
**修复建议**  
  
  
  
**正式防护方案**  
  
GeoServer  
- 3.0.x 受影响版本：建议升级至3.0.2或更高版本；  
  
- 2.28.x 受影响版本：建议升级至2.28.6或更高版本；  
  
- 2.27.6及此前受影响版本：建议升级至2.27.7或更高版本。  
  
  
  
  
**03**  
  
**漏洞描述**  
  
  
  
该漏洞并非单点缺陷，而是 5 个缺陷叠加形成的完整攻击链：  
  
  
**1. 数据目录路径泄露（CVE-2026-129244）**  
  
WFS 在处理超长 storedQuery_id 时，文件查找失败并把尝试打开的绝对路径写入公开的异常信息，返回形如 <数据目录>/wfs/query/<id>.xml，从而得到数据目录位置。  
  
**2. 操作安全规则绕过（CVE-2026-129243）**  
  
请求使用 service=x/WFS 时，仍会进入真实 WFS 服务，但无法匹配针对 wfs.CreateStoredQuery 的操作安全规则，导致**匿名用户可以持久化存储查询**  
。  
  
**3. XInclude 本地文件读取（CVE-2026-120705）**  
  
存储查询文本中经过转义的 xi:include 元素在保存时被反转义并重新解析；DescribeStoredQueries 以启用 XInclude、空实体解析器的方式重新加载文件，从而把 security/config.xml（remember-me 密钥）与 security/usergroup/default/users.xml（管理员口令摘要）内联进响应。  
  
**4. remember-me 会话伪造（CVE-2026-129245）**  
  
Spring Security 的 remember-me 签名是 SHA-256(user:expiry:password-digest:key)。密钥与口令摘要既已泄露，攻击者无需破解口令即可计算合法签名，构造 remember-me Cookie，并由 remember-me 过滤器换取管理员 JSESSIONID。  
  
**5. JDBC 反序列化命令执行（CVE-2026-129242）**  
  
以管理员会话调用 REST，将精心构造的 HSQLDB 数据库文件写入数据目录，并注册为 JDBC 用户组服务（jdbc_geoserver_poc）。当查询该服务下的用户属性时，OTHER 列中的序列化对象被反序列化：PriorityQueue → BeanComparator → TemplatesImpl 触发 XSLTC translet 加载，执行 /bin/sh -c <命令>。  
  
  
整链**不需要任何凭据、不需要用户交互、不需要可选扩展**  
，默认配置即可完成命令执行。  
  
  
**04**  
  
**漏洞复现**  
  
  
  
360漏洞研究院依托图龙锋已成功复现 GeoServer 认证前远程代码执行链。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/dZ7ia5iaWFzz8h1YKdqHooYctZHIhHchuNrnfbBKBouY1GCvmy4zfhB53YlU11SSTnBtdfKd6r6CdKxhyDlGnnDHtbYFnAsFicQr4mfVwfGF7o/640?wx_fmt=png&from=appmsg "")  
  
GeoServer 认证前远程代码执行漏洞复现  
  
  
**05**  
  
**产品侧支持情况**  
  
  
  
**360安全智能体：**  
支持该漏洞攻击的智能分析**。**  
  
**360测绘云 Quake**  
：默认支持该产品的指纹识别。  
  
**360高级持续性威胁预警系统**  
：预计 2026年11月28日发布规则更新包，支持该漏洞利用行为的检测。  
  
**360资产与漏洞检测管理系统**  
：预计 2026年11月29日发布规则更新包，支持该漏洞利用行为的检测。  
**本地安全大脑**  
：默认支持该漏洞的PoC检测。  
  
  
**06**  
  
**时间线**  
  
  
  
2026年11月25日，360漏洞研究院发布本安全风险通告。  
  
  
**07**  
  
参考链接  
  
  
  
- 技术分析：https://cryptocat.me/blog/research/analysis/geoserver_preauth_rce/  
  
- GHSA-9wf2-fcgq-vxhq（CVE-2026-129242）：https://github.com/geoserver/geoserver/security/advisories/GHSA-9wf2-fcgq-vxhq  
  
- GHSA-hgm3-wq4h-8vvq（CVE-2026-129245）：https://github.com/geoserver/geoserver/security/advisories/GHSA-hgm3-wq4h-8vvq  
  
- GHSA-39mw-5chv-jh47（CVE-2026-120705）：https://github.com/geoserver/geoserver/security/advisories/GHSA-39mw-5chv-jh47  
  
- GHSA-rv5g-g2r9-58rj（CVE-2026-129243）：https://github.com/geoserver/geoserver/security/advisories/GHSA-rv5g-g2r9-58rj  
  
- GHSA-4v26-c537-jvxc（CVE-2026-129244）：https://github.com/geoserver/geoserver/security/advisories/GHSA-4v26-c537-jvxc  
  
  
  
  
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
  
  
