#  博通Wi-Fi漏洞已存在16年，攻击者无需密码即可接管路由器  
 FreeBuf   2026-10-20 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3mNoQbjPDYDEK5gicTumAz4YO0EAdKibgweMJo0zRlPLPQYh95VDPqzj8nFApbCYsjJiaZm4RCPJuWh8hAia1gsvnMDQTD9Oia9vUM/640?wx_fmt=gif "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icBE3OpK1IX2YgcSfHxUEv63eMM94gIp6EjN8bJS14ldicPicPY4sFKPbRMzfTQe6ur7GXjibzg1tOWVtjOv5H2ZTFVJe55EbllDkGCp32R7hA4/640?wx_fmt=jpeg&from=appmsg "")  
  
  
一个最早可追溯至2010年的博通Wi-Fi漏洞，可让处于Wi-Fi信号覆盖范围内的攻击者无需认证，就在受影响路由器上执行代码。  
  
  
Part  
01  
  
漏洞潜伏博通SDK超16年  
  
  
研究人员Quentin Kaiser在巴黎Hexacon大会上发表演讲，披露了博通私有802.11协议实现中的一个漏洞。该漏洞至少从2010年起就存在于博通SDK中，攻击者无需知晓Wi-Fi密码，也无需完成正常WPS配置流程，就能在受影响路由器上实现未授权远程代码执行。  
  
  
截至2026年10月15日，比利时境内所有受影响设备均未收到补丁。  
  
  
Kaiser指出："博通私有802.11协议实现中的这一漏洞，至少从2010年起就存在于其SDK中。博通曾在2018年悄悄发布过不完整补丁，随后建议用户迁移至hostapd替代原有方案。这意味着基于eCos的平台（BCM3380、BCM3383、BCM33834）上的漏洞将彻底成为永久无补丁漏洞，基于Linux的平台（BCM3390、BCM65xx、BCM68xx）上的漏洞也可能面临同样情况，除非受影响的原始设备制造商（OEM）愿意完成软件栈迁移。攻击者无需知晓网络密钥，也无需完成正常WPS注册流程，就能通过Wi-Fi远程触发漏洞。成功利用后可实现可控的信息泄露和任意代码执行。"  
  
  
该漏洞存在于WPS EAP重组代码中，这段代码负责处理多数消费级路由器使用的无线配置流程。只要处于Wi-Fi信号覆盖范围内，攻击者无需知晓Wi-Fi密码、无需完成配置流程，也不需要用户执行任何操作，就能利用该漏洞。攻击可泄露敏感信息，允许攻击者在路由器上运行代码。在演示中，Kaiser从路由器内存中提取出Wi-Fi密码，通过无线连接将密码传出，随后使用该密码接入网络。  
  
  
Part  
02  
  
实测验证攻击链完整  
  
  
这些攻击并非理论层面的推演。Kaiser针对两款设备完成了漏洞利用验证，分别是Orange Belgium的ASKEY TCG3000，以及Proximus的Sagemcom F@st/BBOX3V+。两款设备均允许攻击者在未认证的情况下执行代码。在其中一次演示中，攻击者仅通过Wi-Fi帧就建立了反向shell，不需要可通过互联网访问的命令服务器。  
  
  
攻击支持远距离实施。视频中天线近距离摆放只是实验室测试环境的配置，并非攻击的必要条件。  
  
  
在第一段演示视频中，Kaiser利用Orange Belgium的ASKEY TCG3000路由器发起攻击，成功在未认证的情况下在设备上执行代码。代码从路由器NVRAM中读取Wi-Fi密码（PSK），通过Wi-Fi回传给攻击者。他随后使用窃取到的密码接入网络，验证了漏洞利用有效、密码准确。  
  
  
不要被视频中天线的近距离摆放误导，该攻击支持更远的作用距离。  
  
  
在第二段演示视频中，Kaiser针对Proximus的BBOX3V+（即Sagemcom F@st）路由器发起攻击，同样实现了未认证代码执行。他通过Wi-Fi帧建立反向shell，不需要部署在云端的命令与控制（C2）服务器。  
  
  
Part  
03  
  
厂商未提供完整修复  
  
  
博通在2018年悄悄发布了不完整修复，随后没有为自有代码推出完整补丁，反而要求客户完全迁移到其他软件栈。对于搭载BCM3380、BCM3383、BCM33834等芯片的eCos平台，这意味着该漏洞从设计层面就永久无法修复。对于基于Linux的平台（BCM3390、BCM65xx、BCM68xx），漏洞能否修复完全取决于各设备厂商是否愿意采取行动。  
  
  
Kaiser于2026年7月向博通上报了该漏洞。博通称已有补丁可用，让他告知受影响设备厂商直接与博通对接。博通拒绝为该漏洞分配CVE编号，也不愿分享其通过客户门户下发给客户的通知内容。CERT/CC曾就该漏洞启动协调流程，但在确认博通有权自行分配CVE编号且无意为该漏洞分配后，当天就关闭了案件。整个漏洞披露流程完全不合规，最终走入死胡同，仅留下纸面流程记录。  
  
  
Kaiser结合比利时电信市场数据粗略估算，Proximus约有85万台受影响设备，VOO的旧设备群约有50万台，Orange Belgium约有21万台，仅比利时这一个小国，潜在受影响的网关设备就超过150万台。Telenet的Sagemcom设备也可能受影响，但Kaiser未能拿到设备样本，无法确认具体情况。  
  
  
受影响厂商的销售范围远不止比利时。相关芯片被全球数十家厂商的数百款产品采用。目前没有人能完整掌握受影响设备的全球部署范围，因此Kaiser将发布专门的指纹识别脚本和YARA规则，帮助排查所有受影响的设备部署。  
  
  
Part  
04  
  
AI辅助一周完成漏洞挖掘  
  
  
这一2010年引入、2018年仅部分修复的漏洞，至今仍影响大量在用路由器。AI辅助分析帮Kaiser在不到一周内就找到了漏洞，换做纯人工分析则需要数月时间。这种能力既能帮助防御方更快发现漏洞，也可能被攻击者用于同类活动。  
  
  
在攻击者处于Wi-Fi覆盖范围内的定向攻击场景中，风险最高，比如针对酒店网络的攻击、对周边目标的长期监控。被入侵的路由器很难被安全监控覆盖，攻击者可以悄悄拦截、收集全网流量。多数用户和安全团队很少检查运营商提供的路由器的固件版本。  
  
  
Kaiser总结道："如果完全靠人工开展这项研究，我需要花数月时间才能完成。借助前沿AI模型，整个过程耗时不到一周。这是件好事，因为我们只需要极低的成本，就能开始排查技术基础设施中更深层次的安全问题。"  
  
  
参考来源：  
  
16-Year-Old Broadcom Wi-Fi Bug Lets Attackers Take Over Routers Without a Password  
  
https://securityaffairs.com/201268/iot/16-year-old-broadcom-wi-fi-bug-lets-attackers-take-over-routers-without-a-password.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
