#  【已复现】Telegram Desktop 高危漏洞，点击链接可致文件泄露甚至账户被接管  
原创 360漏洞研究院
                    360漏洞研究院  360漏洞研究院   2026-10-10 03:26  
  
Telegram Desktop 曝出高危 IPC 记录分隔符注入漏洞（CVE-2026-107181，CVSS 3.1：8.1）。攻击者可诱导用户打开特制链接，在满足利用条件时触发客户端内部文件发送功能，导致本地敏感文件泄露，甚至造成账户会话被接管。  
  
  
**利用前置条件：**  
- 目标用户 Telegram Desktop 客户端正在运行  
  
- 用户打开攻击者提供的特制链接  
  
**目前360图龙锋 · 漏洞挖掘智能体已成功复现该漏洞。**  
本文包含完整影响范围、正式修复方案、技术原理与复现细节。  
  
  
<table><tbody><tr style="box-sizing: border-box;"><td colspan="4" data-colwidth="100.0000%" width="100.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;background-color: rgb(100, 130, 228);box-sizing: border-box;padding: 0px;"><section style="text-align: center;color: rgb(255, 255, 255);box-sizing: border-box;"><p style="margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞概述</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞名称</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">Telegram Desktop IPC 记录分隔符注入漏洞</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞编号</span></strong></p></section></td><td colspan="3" data-colwidth="76.0000%" width="76.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">CVE-2026-107181</span></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">公开时间</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">2026-10-07</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">POC状态</span></span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">漏洞类型</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">IPC 记录分隔符注入</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">EXP状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span style="color: rgb(0, 0, 0);box-sizing: border-box;"><span leaf="">利用可能性</span></span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">高</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(0, 0, 0);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">技术细节状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;color: rgb(100, 130, 228);box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">已公开</span></strong></p></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="24.0000%" width="24.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">CVSS 3.1</span></strong></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">8.1</span></p></section></td><td data-colwidth="28.0000%" width="28.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;color: rgb(0, 0, 0);padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><strong style="box-sizing: border-box;"><span leaf="">在野利用状态</span></strong></p></section></td><td data-colwidth="20.0000%" width="20.0000%" style="border-width: 1px;border-color: rgb(100, 130, 228);border-style: solid;box-sizing: border-box;padding: 0px;"><section style="font-size: 12px;padding: 0px 8px;box-sizing: border-box;"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;"><span leaf="">未发现</span></p></section></td></tr></tbody></table>  
  
  
**01**  
  
**漏洞影响范围**  
  
  
  
Telegram Desktop < 7.2.9  
  
目前公开研究已在 Windows 环境下完成漏洞验证。  
  
  
**02**  
  
**修复建议**  
  
  
  
**正式防护方案**  
  
升级至以下或更高版本  
  
Telegram Desktop >= 7.2.9  
  
  
**03**  
  
**漏洞描述**  
  
  
  
近日，安全研究人员披露了 Telegram Desktop 中存在的 IPC 记录分隔符注入漏洞（CVE-2026-107181）。当客户端已有实例运行时，新启动进程会通过本地套接字传递外部链接；由于传输内容中的分号未得到正确转义，接收进程可能将单条链接错误拆分为多条内部指令。攻击者可借此触及原本供内部发布流程使用的 interpret: 处理功能。该功能能够根据本地指令文件读取并发送文件，且缺少充分的授权校验。两项缺陷组合后，可能引发敏感文件外传；若会话相关数据泄露且满足恢复条件，则存在账户会话被接管的风险。  
  
  
**04**  
  
**漏洞复现**  
  
  
  
360漏洞研究院依托图龙锋漏洞挖掘智能体已成功复现 CVE-2026-107181。  
  
在 Windows 测试环境中，研究人员创建测试超级群组，并在无需目标用户确认的情况下将其加入群组。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dZ7ia5iaWFzzib9bxs5tIibe1Nibwuh9OmckUX1C9ibicCRBHGLfOs5ubWStAZpEVUmID7CE9eCoa3jqUThY26fw0W92lzib7SxrQsRYNg3HpltMRbk/640?wx_fmt=png&from=appmsg "")  
  
向群组发送包含特定文件处理指令的 TXT 文件，利用 Telegram Desktop 的文件自动下载机制将其保存至本地。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dZ7ia5iaWFzz9wTSXeXpe598tS3vJelibB9niatB4TmPOq5Pibznb8LEKh0y0wMKuTYV7HzAEicv1AFZHiat1F6XA9ZFFNWLyb5yz3af9kZVbNwKdI/640?wx_fmt=png&from=appmsg "")  
  
在测试客户端保持运行的情况下，访问特制 PoC 链接，触发 Telegram Desktop 单实例 IPC 指令解析缺陷，进而调用客户端内部文件发送功能。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dZ7ia5iaWFzzic3doDd03iaz4kBuXo7LMichW1q1fNYzvkWXHCDI3GTY9q2X8ayDH0yuMKxyknuh8lRdU0wibzG1SkcM4MZibiaXicJAMbmGw1WWS4C4/640?wx_fmt=png&from=appmsg "")  
  
测试结果显示，客户端在未经额外文件发送确认的情况下，自动将本地测试文件发送至指定群组，验证了该漏洞可导致本地文件未经授权读取及外传。  
  
  
**05**  
  
**时间线**  
  
  
  
2026年10月10日，360漏洞研究院发布本安全风险通告。  
  
  
**06**  
  
参考链接  
  
  
  
https://beaksec.github.io/posts/telegram-desktop-one-click-account-takeover/  
  
  
07  
  
更多漏洞情报  
  
  
  
“扫描下方二维码，进入公众号粉丝交流群。更多一手网安资讯、漏洞预警、技术干货和技术交流等您参与！”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dZ7ia5iaWFzz8YToicKab1BicPnEdr7jiatvQUVWSMnYTBeG5ibibgxkGAG1rF4pUdpowPcCmokOO5tp4UjjhUsos4Zf4VwE1aM9NTUz3ogfgdwwFw/640?wx_fmt=gif&from=appmsg "")  
  
  
建议您订阅360漏洞情报服务，获取更多漏洞情报详情以及处置建议，让您的企业远离漏洞威胁。  
  
  
邮箱：360VRI@360.cn  
  
网址：https://vi.loudongyun.360.net  
  
  
  
“洞”悉网络威胁，守护数字安全  
  
  
**关于我们**  
  
  
360漏洞研究院，隶属于360安全能力中心。其成员常年入选谷歌、微软、华为等厂商的安全精英排行榜, 并获得谷歌、微软、苹果史上最高漏洞奖励。研究院是中国首个荣膺Pwnie Awards“史诗级成就奖”，并获得多个Pwnie Awards提名的组织。累计发现并协助修复谷歌、苹果、微软、华为、高通等全球顶级厂商CVE漏洞3000多个，收获诸多官方公开致谢。研究院也屡次受邀在BlackHat，Usenix Security，Defcon等极具影响力的工业安全峰会和顶级学术会议上分享研究成果，并多次斩获信创挑战赛、天府杯等顶级黑客大赛总冠军和单项冠军。研究院将凭借其在漏洞挖掘和安全攻防方面的强大技术实力，帮助各大企业厂商不断完善系统安全，为数字安全保驾护航，筑造数字时代的安全堡垒。  
  
  
