#  EDK II曝16年高危漏洞，攻击者可在Secure Boot前执行代码  
 FreeBuf   2026-10-30 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX2fRGzBFV5LiaJbLztLm4ozCqKqBUbwwXnnIicnPicB9p57nvr2xrvPvCNKR1oo2x6F960BjkOb5MYicXgiaA1xEvZ59ROniccEvnRI4/640?wx_fmt=gif "")  
  
![EDK II UEFI漏洞](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0GGmVLdM4atm057IrB4ImSDgKLLUicwIsKWw1L2AZgDCPdibcoOwhicctZgHqib4cIIZRp8ibHUBR67rFZicY5LvOoQ9SicrrkrOCCFU/640?wx_fmt=png "")  
  
  
EDK II是绝大多数硬件厂商构建UEFI固件时使用的开源代码，其中存在一个潜伏16年的高危漏洞。攻击者可在操作系统加载前在目标设备上执行代码，但利用场景仅限设备通过网络启动时。  
  
  
UEFI固件是计算机开机后、操作系统启动前最先运行的底层软件。在网络启动过程中，攻击者如果能向目标设备发送构造的网络报文，即可导致启动流程挂起；若针对目标设备的具体固件版本定制利用代码，还能在Secure Boot及其他启动防护机制加载前执行自有代码。  
  
  
攻击者不需要密码，也无需提前获得目标设备的访问权限，但必须在设备网络启动的短暂时间窗口内通过网络触达目标。  
  
  
包含受影响代码的红帽（Red Hat）和SUSE均评估，攻击者必须处于目标的邻接网络位置才能利用该漏洞，无法通过公网远程发起攻击。  
  
  
EDK II背后的TianoCore项目于10月22日发布安全公告，将该漏洞（CVE-2026-97631）的严重程度评为9.5分（满分10分）。目前已有对应的代码修复方案，但尚未有EDK II稳定版本集成该修复，公告中也未列出任何已打补丁的版本，绝大多数用户当前没有可直接安装的更新。  
  
  
Part  
01  
  
漏洞可触发启动挂起  
  
  
该漏洞位于EDK II的PXE网络启动代码中，属于缓冲区溢出问题。PXE启动模式下，设备不会从本地磁盘加载启动镜像，而是从网络获取启动所需文件。  
  
  
PXE启动过程中，PXE驱动会监听ICMP错误报文以及网络设备返回的小型状态报文，将所有收到的报文复制到内存中一块大小固定为504字节的区域。  
  
  
IPv4和IPv6版本的代码中，复制操作的长度校验均存在错误，长度超过504字节的报文会溢出该内存区域，用攻击者构造的数据覆盖邻近内存。  
  
  
攻击者触发漏洞的门槛极低，只需要在设备网络启动过程中向目标地址发送一个构造的错误报文即可，无需密码或提前访问设备。  
  
  
漏洞触发后最常见的后果是设备崩溃，启动流程挂起，设备无法正常开机。要实现代码执行则难度高得多，且可靠性极差。公告显示，攻击者只有掌握目标设备的确切固件版本，才能植入"伪造事件"诱导固件运行攻击者的代码，因此利用代码必须针对特定固件版本单独构建。  
  
  
一旦攻击者实现代码执行，代码将运行在固件层，此时Secure Boot和其他启动校验机制尚未完成任何验证，这种执行时机正是该漏洞的核心风险所在。  
  
  
通过这种方式植入的代码仅会运行一次，本身无法在设备重启后留存。但仅这一次执行，就足以安装其他可在每次开机时自动启动的恶意代码。  
  
  
Part  
02  
  
漏洞潜伏16年  
  
  
该漏洞影响EDK II所有历史版本，截至8月21日发布的最新稳定版edk2-stable202608仍未修复。EDK II是几乎所有UEFI固件厂商的参考代码，用户涵盖独立BIOS厂商AMI、Insyde、Phoenix，以及微软的加固固件分支Project Mu。  
  
  
The Hacker News追溯项目提交历史发现，存在缺陷的复制循环代码早在2010年11月的一次代码提交中就已引入，至今已潜伏16年。  
  
  
EDK II的标准构建版本默认开启PXE网络启动功能。仅从本地磁盘启动的家庭和办公用户不受该漏洞影响，受波及的主要是采用网络启动设备的机构，尤其是服务器和数据中心设备。  
  
  
目前尚不清楚有多少在售产品集成了存在漏洞的代码。英特尔向Ars Technica表示，其"当前处于安全支持周期内的商用产品"均未发现受影响，且其"参考代码中未包含受影响的EDK2功能"，但OEM主板厂商或设备制造商可能自行添加该功能。  
  
  
Arm表示该漏洞"不直接影响Arm的产品组合"。英特尔和Arm的自有固件均基于EDK II开发。  
  
  
多个Linux发行版也包含受影响代码。红帽称Red Hat Enterprise Linux 8、9、10中的edk2包均受影响且尚未修复，SUSE和Debian也将其对应软件包标记为相同状态。  
  
  
截至报道发布前，AMD、Phoenix Technologies、AMI、微软、亚马逊均未回应该科技媒体的提问，TianoCore也未作出回复，因此这些厂商的产品受影响状态尚不明确。  
  
  
Part  
03  
  
修复代码已合入主线  
  
  
目前修复代码已经合入EDK II源码仓库。修复方案重写了存在缺陷的复制逻辑，超长报文会被截断在504字节，不会再越界覆盖邻近内存，该修复已于10月15日进入主开发分支。  
  
  
当前还没有任何EDK II稳定版本集成该修复。The Hacker News在10月28日核查项目发布标签时发现，最新版本edk2-stable202608的发布时间早于修复合入时间，公告中仍将已修复版本状态标记为"当前未修复"。微软的Project Mu已将修复代码拉取到自身发布分支，并于10月26日发布了带标签的构建版本。  
  
  
即使修复代码已经公开，服务器运维人员也需要等待设备固件厂商发布基于该修复构建的BIOS或UEFI更新，才能安装修复。固件安全公司Eclypsium首席研究员Jesse Michael告诉Ars Technica，这一更新流程相对简单。和BootHole、BlackLotus这类Secure Boot绕过漏洞不同，该漏洞"仅需基础代码修复即可解决"，不需要吊销证书或密钥。  
  
  
在官方更新发布前，公告未给出任何临时缓解方案。由于漏洞仅能在网络启动过程中被触发，不需要使用PXE功能的运维人员可以禁用该功能，缩小攻击面。Ars Technica同时建议监控启动阶段的网络流量和日志，及时发现攻击尝试迹象。  
  
  
Part  
04  
  
漏洞因PR提前公开  
  
  
公告显示，该漏洞由Anthropic的Alex Matrosov和谷歌的Dan Holmes-Mitra共同发现，原计划于2027年1月开展协同披露。  
  
  
9月28日，一份由AMI工程师签署的EDK II项目PR公开提及了该漏洞的CVE编号，并附带了修复代码，导致原披露计划被打乱。漏洞发现者注意到这一公开内容后，TianoCore于10月22日正式发布安全公告，比原计划提前了数周。  
  
  
Ars Technica指出，补丁公开后，技术能力较强的攻击者可以通过研究补丁掌握漏洞触发方式，构建可实际使用的利用代码。  
  
  
目前没有迹象表明该漏洞已被在野利用。截至10月27日CISA更新已知被利用漏洞目录，该漏洞仍未被纳入清单，也没有研究人员报告发现相关攻击活动。  
  
  
Ars Technica咨询多位固件研究人员后表示，该漏洞本身无法作为攻击者进入企业网络的初始入口。现实场景中，利用该漏洞的攻击者通常是内部人员，或是已经在本地网络获得低级访问权限、试图进一步渗透的攻击者。  
  
  
Dragos首席研究员、Metasploit创建者HD Moore向Ars Technica表示："想要攻陷邻近终端或服务器的攻击者，可以利用这个漏洞，在启动进程仍能访问TPM等敏感组件的阶段劫持程序流程。"  
  
  
如果攻击者拥有短期网络访问权限，就可以通过这种方式破解磁盘加密；在设备被盗的场景下，利用难度还会更低。  
  
  
这并不是EDK II网络启动代码中首次发现严重漏洞。2024年1月，研究人员曾披露PixieFail漏洞集，包含EDK II同一模块中的9个漏洞，同样可在PXE启动过程中触发。Ars Technica报道称，PixieFail事件后业界启动了针对该模块的模糊测试项目，旨在挖掘更多潜在漏洞，但并未发现本次这个潜伏16年的遗留漏洞。  
  
  
参考来源：  
  
Critical 16-Year-Old UEFI Flaw Can Run Code Before Secure Boot on Network-Booting Servers  
  
https://www.bleepingcomputer.com/news/security/critical-16-year-old-uefi-flaw-can-run-code-before-secure-boot/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
