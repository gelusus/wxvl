#  KVM逃逸漏洞暴露Linux网络栈更广攻击面，影响多类虚拟化场景  
 FreeBuf   2026-10-21 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX1RInVUJUhMSR6ZTZSZUP2SoxDFNXkEx9yzpgiaQ98giaOIlTysibP2dgKoibFTibrvPFF0IibFk1tlwYXFHKrmFFn3ibBM73icac7uxR4/640?wx_fmt=gif "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX2EPf2vTic0wicweZvFY3dlmQzlXzNg6e9oib9HibJp6YNpFQ1CyDMlyMzcV1GFdh7ostJds58ZkvPrqSm1pQEEWCEEibmKmCLiaxnhA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
Part  
01  
  
PwnEscape漏洞引发KVM逃逸风险  
  
  
安全研究人员在Linux内核网络栈中发现一个严重漏洞，攻击者可借此逃逸KVM虚拟机，在主机系统上执行代码。该漏洞还可能波及其他基于Linux的虚拟化与沙箱平台。  
  
  
该漏洞追踪编号为CVE-2026-99999，命名为PwnEscape，由AI驱动的渗透测试初创公司Pwn.ai的研究人员发现。发现场景是一场奖金100万美元的Vercel沙箱平台破解挑战，Vercel的沙箱基于KVM架构的Firecracker微虚拟机搭建。  
  
  
此次攻击并未利用KVM hypervisor本身的漏洞，而是瞄准Linux系统中负责处理虚拟机内部流量的网络代码。攻击者会构造特殊数据包，在内核完成IPv4头校验后篡改报文内容。这种篡改会让内核的后续处理流程，信任攻击者恶意修改过的数据包数据。  
  
  
研究人员在Vercel生产环境沙箱中成功演示了主机内存泄露，又在自主可控的测试系统上实现了主机崩溃与root权限代码执行。Vercel已确认漏洞存在，但表示目前没有证据显示攻击者访问过客户文件、密钥或应用数据。  
  
  
目前Linux主线内核已合入漏洞修复代码，多个Linux发行版已提供修复后的内核版本。管理员必须在主机系统上部署这些内核更新，仅升级客户机系统无法修复该漏洞。  
  
  
Part  
02  
  
校验和处理逻辑存在缺陷  
  
  
漏洞涉及校验和卸载机制，这是Linux系统中的一项性能优化功能，可以延后计算数据包的校验和。内核会随数据包携带元数据，指定校验和计算的起始位置与结果写入地址，但在虚拟网络场景下，这些指令可能来自不可信的客户机系统。  
  
  
存在漏洞的实现逻辑没有做防护，无法保证校验和计算完成后，已通过校验的IPv4报文头不会被改动。这意味着客户机可控的元数据可以直接写入IPv4报文头，在数据包被分片传输前，修改报头中声明的数据包长度。  
  
  
在演示过程中，研究人员将报文头声明的长度从20字节修改为60字节，但当时数据包缓冲区中仅存储了28字节的有效数据。后续分片流程会将有效数据边界外的32字节内容复制到出站数据包中，造成主机内存信息泄露。  
  
  
通过重复触发该流程，研究人员可以观测内存状态、控制内存分配，最终实现主机内核内存写入。借助该能力，攻击者可以root权限运行辅助程序。目前从内存泄露到代码执行的完整利用步骤尚未公开，研究人员计划单独发布技术分析文章。  
  
  
研究人员表示："涉及分配器敏感的写入链、对象生命周期控制以及其他hypervisor特定适配的内容体量较大，需要单独成文。我们认为安全行业会借助本次公开的内容排查其他受影响的hypervisor，预计弱配置场景下的利用代码几周内就会出现，默认hypervisor配置下的利用代码会在一个月内出现。"  
  
  
Part  
03  
  
漏洞攻击面超出KVM范围  
  
  
PwnEscape与以往针对hypervisor虚拟CPU或内存处理逻辑的虚拟机逃逸攻击不同。比如今年早些时候披露的Januscape漏洞，影响的是KVM的内存管理代码，但本次漏洞的问题代码位于Linux主机的IPv4网络子系统。  
  
  
研究人员警告，网络栈中只要存在"字段已通过校验、但客户机可控元数据仍能修改数据包字段"的场景，就可能存在同类漏洞。  
  
  
本次合入主线的补丁由红帽（Red Hat）开发者Michael S. Tsirkin编写，在5个校验和完成节点新增校验逻辑。这些校验点覆盖IPv4分片、网桥连接跟踪、数据包过滤等场景。补丁提交说明提到："后续会在数据包输入阶段实现更完善的校验，目前相关工作还在推进中。"  
  
  
该漏洞对其他hypervisor和容器技术的影响目前尚不明确。红帽已将OpenShift Container Platform列为受影响产品，Docker、Xen等其他虚拟化技术目前尚未发布相关安全公告。  
  
  
这种从客户机到主机的攻击，需要攻击者控制的工作负载能通过虚拟网络发送畸形流量。因此，允许在虚拟机内运行不可信代码的环境管理员，需要尽快采取修复措施。  
  
  
容器安全厂商Edera首席技术官兼联合创始人Alex Zenla在接受CSO采访时表示："KVM本身运行在Linux内核内部，因此这类虚拟机逃逸本质上也属于内核漏洞，影响要比普通内核漏洞严重得多。攻击者从客户机突破后，并不会进入一个边界清晰的独立hypervisor层，而是直接落到操作系统的最底层，掌控主机的所有进程、驱动和网络栈。这类漏洞风险极高，和那种特意设计了极小特权层的hypervisor的安全模型完全不同。"  
  
  
今年以来，KVM及其他内核态虚拟化组件已经出现多个严重漏洞，其中包括PwnEscape在内的部分漏洞是借助LLM发现的。同时测试中也观测到AI Agent尝试突破沙箱的行为，这对虚拟化环境的工作负载隔离假设提出了挑战。  
  
  
Zenla表示："安全架构设计必须假设系统总有被攻破的可能，要确保单点失陷不会导致整个主机被完全控制。"  
  
  
AWS托管的EC2、Lambda、Fargate服务不受CVE-2026-99999影响。如果用户在存在漏洞的主机上自行部署使用virtio-net网络的KVM虚拟机，需要安装对应的内核更新。管理员应遵循所用发行版的安全公告，因为补丁可能被反向移植到旧版本内核中。  
  
  
参考来源：  
  
KVM escape exposes broader attack surface in Linux networking  
  
https://www.csoonline.com/article/4004609/kvm-escape-exposes-broader-attack-surface-in-linux-networking/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
