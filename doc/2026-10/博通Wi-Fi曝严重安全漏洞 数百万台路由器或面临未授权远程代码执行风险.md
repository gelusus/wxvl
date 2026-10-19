#  博通Wi-Fi曝严重安全漏洞: 数百万台路由器或面临未授权远程代码执行风险  
tuto
                    tuto  杂杂咱谈   2026-10-18 16:06  
  
## 一、漏洞概述  
  
安全研究人员Quentin Kaiser披露，博通专有的Wi-Fi保护设置实现中存在两个内存安全漏洞，涉及其对无线网络协议数据的解析与重组处理。  
  
相关缺陷至少可以追溯至2010年，研究人员发现，虽然博通在2018年曾发布过不完整的修复方案，但部分设备至今仍可能受到影响。由于漏洞存在于被多家设备制造商采用的底层SDK实现中，其影响范围不仅限于单一品牌，而是可能波及大量使用博通芯片及相关软件组件的路由器、网关和电缆调制解调器。  
  
根据研究人员公布的技术分析，攻击者在目标设备的Wi-Fi无线通信范围内，无需知道无线网络密码，也无需完成正常的WPS配网流程，即可尝试触发漏洞。成功利用后，攻击者可能实现内存信息泄露和远程代码执行，进而获取设备的高权限控制能力。  
  
但这并不意味着所有博通芯片设备都必然存在漏洞。 实际影响取决于芯片平台、固件实现、厂商是否采用相关代码，以及是否部署了有效修复。  
## 二、漏洞基本信息  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">详细信息</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">漏洞名称</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Broadcom Wi-Fi WPS解析与重组漏洞</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">涉及厂商</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Broadcom (博通)及采用相关SDK的设备制造商</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">主要问题</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">WPS EAP分片重组过程中的内存溢出等内存安全缺陷</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">影响平台</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">eCos、Linux相关实现</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">攻击方式</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">无线网络范围内的远程攻击</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">身份认证</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员展示的利用不要求事先完成正常 WPS 配网</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">潜在影响</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">信息泄露、任意代码执行、设备高权限控制</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">漏洞编号</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">暂无CVE编号</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">修复状态</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">截至2026年10月15日，研究人员测试的受影响设备仍未修复</span></section></td></tr></tbody></table>## 三、漏洞技术原理  
### 1. WPS 协议处理过程存在缺陷  
  
WPS(Wi-Fi Protected Setup)是一种用于简化无线网络设备配对和接入配置的机制。相关实现需要处理 EAP(可扩展认证协议)消息，并在必要时对分片数据进行重组。  
  
博通专有的802.11/WPS实现中存在与EAP分片重组有关的内存溢出问题。攻击者可以通过构造特定无线协议数据，触发存在缺陷的解析和重组逻辑。  
  
当程序未能正确校验数据长度、分片信息或缓冲区边界时，异常输入就可能破坏内存中的其他数据，进而形成信息泄露或代码执行等安全风险。  
### 2. 两类平台的漏洞影响  
- eCos平台: 涉及BCM3380、BCM3383和BCM33834等芯片系列。  
  
- Linux平台: 涉及BCM3390、BCM65xx和BCM68xx等芯片系列。  
  
研究人员指出，2018年的修复并未彻底解决所有相关问题。博通后来建议相关厂商迁移至hostapd实现，这使得仍依赖旧有专有实现的部分设备面临长期无法获得有效修复的风险。  
### 3. 从无线攻击到设备控制  
  
研究人员公布的演示展示了从无线协议攻击到设备控制的完整利用链。  
  
步骤一: 发送特制无线协议数据  
  
攻击者位于目标设备的Wi-Fi无线通信范围内  
  
步骤二: 触发WPS/EAP解析缺陷  
  
异常分片重组引发内存安全问题  
  
步骤三: 利用内存破坏获取执行能力  
  
具体效果取决于目标平台及利用条件  
  
步骤四: 泄露敏感信息或执行任意代码  
  
可能进一步导致路由器或网关被完全控制  
  
在针对Orange Belgium提供的ASKEY TCG3000网关的演示中，研究人员展示了未认证远程代码执行，并通过执行载荷读取设备NVRAM中保存的无线网络预共享密钥，再通过802.11无线帧将密钥传回攻击端。  
  
  
针对Proximus的BBOX3V+设备，研究人员也展示了未认证代码执行，并利用无线帧建立反向Shell。上述演示说明，攻击者不一定需要先攻入局域网，也不一定需要依赖传统互联网命令与控制服务器。  
  
  
以上为研究人员针对特定设备的演示结果，不代表所有受影响型号都能以相同方式利用。  
## 四、受影响设备及潜在范围  
  
研究人员在比利时的多家互联网服务提供商网络中验证了漏洞，并公布了部分已确认或待确认的设备型号。  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">运营商</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">涉及设备型号</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究情况</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Proximus</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">BBOX 3、BBOX3V+</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员确认相关设备存在漏洞并可利用</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">VOO</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Netgear CG3100、CG3700、Technicolor TC7210、CGA4233</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员确认相关设备存在漏洞并可利用</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Orange Belgium</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">ASKEY TCG3000</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员确认相关设备存在漏洞并可利用</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Telenet</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Sagemcom F@ST3896LG、F@ST5685LGB</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">待确认</span></section></td></tr></tbody></table>### 研究人员估算的受影响设备数量  
  
以下为研究人员根据公开订户数据及运营商历史设备部署情况进行的估算，并非厂商公布的实际受影响数量。  
  
比利时部分运营商受影响网关估算  
  
不包括尚未确认的Telenet设备；数值为估算台数。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/xoBWaEOhvREtsQ6bCG9K4ptwEGAtvHshrjzaXEntkIv8vRiaREHK6IhBEJvm4re7Y6LMIBAM745SWe6tulQjjBNffKxBIE1SpzR28C17z7ww/640?wx_fmt=png&from=appmsg "")  
- Proximus: 约85万台。  
  
- VOO: 约50万台。  
  
- Orange Belgium: 约21万台。  
  
上述三家运营商合计约156万台潜在受影响网关。该数字仅针对研究人员估算的比利时相关设备，不代表全球受影响设备总量。考虑到相关芯片和软件组件被广泛采用，全球实际影响范围可能更大，但尚无经过全面核实的全球统计数据。  
## 五、漏洞披露时间线  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">日期</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">事件</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2010年或更早</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">相关漏洞已存在于博通SDK中</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2018年</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">博通曾发布不完整修复，并向研究人员表示已有补丁</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2026年7月18日</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员向博通PSIRT提交涉及 BCM3380、BCM3383、BCM33834的报告</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2026年7月27日</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员提交涉及BCM3390、BCM65xx、BCM68xx的另一份报告</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2026年7月29日</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员向相关运营商及比利时网络安全中心报告问题</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2026年8月4日</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员向CERT/CC提交案件，但因博通作为CVE编号机构且不愿登记该漏洞，案件当天关闭</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2026年10月6日</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">研究人员在巴黎Hexacon安全会议公开演讲并发布详细研究文章</span></section></td></tr></tbody></table>  
研究人员表示，博通没有为该漏洞分配CVE编号，也没有公开发布完整的漏洞通告。研究人员截至2026年10月15日仍未获得相关设备已完成修复的确认。  
## 六、漏洞可能带来的安全风险  
  
1. 路由器及网关被远程控制  
  
在具备相应利用条件的设备上，攻击者可能从无线通信范围内触发漏洞，获取代码执行能力，进一步控制网络设备。  
  
2. 无线网络密码泄露  
  
研究人员已在特定设备的演示中展示读取NVRAM内保存的无线网络预共享密钥。若攻击者获得有效密钥，可能进一步尝试接入目标无线网络。  
  
3. 局域网横向攻击  
  
一旦网关或路由器被控制，攻击者可能利用其网络位置，对连接的电脑、手机、智能家居及其他内部设备开展后续攻击。  
  
4. 邻近网络及公共场所风险  
  
这类漏洞特别值得关注的场景包括酒店、办公场所、共享网络及其他存在大量相邻无线设备的环境。攻击者可能在无需先获得网络密码的情况下尝试攻击附近设备。  
  
5. 长期遗留设备难以修复  
  
当漏洞位于芯片厂商提供的底层SDK中时，下游设备厂商可能需要重新适配无线协议实现，而不仅仅是发布常规固件补丁。对于已停止维护的设备，风险可能长期存在。  
## 七、修复建议与安全防护措施  
### 普通用户  
- 确认设备型号及固件版本: 检查路由器、光猫或电缆调制解调器的设备标签与管理界面。  
  
- 联系运营商或设备厂商: 询问是否使用受影响的博通WPS实现，以及是否已有针对该问题的固件更新。  
  
- 及时安装官方固件: 仅使用与具体设备型号和硬件版本匹配的官方固件，不要将其他型号的固件强行刷入设备。  
  
- 关闭不必要的WPS功能: 如果设备支持独立关闭WPS，可在不影响正常使用的前提下关闭该功能。  
  
- 避免将设备管理接口暴露到互联网: 检查远程管理、调试接口和不必要的服务是否已关闭。  
  
### 企业及网络管理员  
- 开展资产盘点: 识别网络中的博通芯片设备、运营商网关及使用相关SDK的嵌入式产品。  
  
- 核实补丁而非只看公告: 向厂商确认实际固件版本、修复组件及修复范围，必要时开展固件分析或授权安全测试。  
  
- 加强无线网络监测: 关注异常WPS/EAP交互、异常无线管理流量及设备重启等现象。  
  
- 隔离关键网络: 对办公终端、服务器、物联网设备和访客网络进行合理分区，降低单一网关被攻陷后的横向移动风险。  
  
- 评估停止维护设备的替换计划: 如果厂商无法提供修复方案，应评估替换设备或迁移至受支持的平台。  
  
### 设备制造商  
- 核查产品是否集成受影响的博通专有802.11/WPS代码。  
  
- 确认旧版修复是否完整覆盖EAP分片重组相关的内存安全问题。  
  
- 评估迁移至受支持的hostapd实现或其他安全替代方案的可行性  
。  
  
- 明确公布受影响型号、修复固件版本、临时缓解措施及停止维护产品的处置建议。  
  
## 八、总结  
  
此次Broadcom Wi-Fi安全研究揭示了一个值得整个网络设备行业警惕的问题: 底层协议解析代码中的内存安全缺陷，可能在多个芯片平台、设备品牌和固件版本之间长期传播。  
  
该漏洞的突出风险在于，研究人员针对部分设备展示了无需事先获得无线网络密钥即可触发的远程攻击，并进一步实现代码执行及敏感信息读取。由于相关代码已存在多年，部分旧设备可能面临修复困难或无法修复的问题。  
  
对普通用户而言，最重要的是确认设备型号、主动向运营商或厂商核实修复情况，并在没有明确补丁时采取合理的临时防护措施。对于企业网络管理员和设备厂商，则应将底层SDK组件审计、供应链漏洞追踪和旧设备生命周期管理纳入常规安全工作。  
  
