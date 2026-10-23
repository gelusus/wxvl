#  Linux KSMBD曝两大9.0分RCE漏洞，主线内核尚未修复  
 FreeBuf   2026-10-23 10:08  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1Z32ibpb3LyTz7uibdSicdSexZwDdfrF3ItEP695J3IP1EgZjwopW78sQ8ElDpbFWP3Jdyvg3ZE7YhHBNOhGPGoosricicHyJ9947s/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX3LasK239COG6fFMicsm7WFgTThayhoGloH2IueIWia9T5PEeziaAh22VmaZOMFibLodl1E8eyEMKlftKmbBw5bPDbnx3pgeFcMbyY/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
两个KSMBD RCE漏洞CVSS 9.0  
  
  
KSMBD是Linux内核内置的SMB服务器。只要Linux服务器运行该服务，就会面临两个已公开披露的远程代码执行漏洞风险。趋势科技旗下ZDI为这两个漏洞给出的CVSS评分均为9.0，攻击者可利用漏洞远程执行任意代码。  
  
  
两个漏洞分别编号为ZDI-CAN-33774和ZDI-CAN-33549，利用门槛存在明显差异。攻击者无需任何网络身份验证就可以触发ZDI-CAN-33774；利用ZDI-CAN-33549则需要启用SMB Direct传输协议，且设备配备支持RDMA的网卡。  
  
  
ZDI-CAN-33549源于KSMBD的SMB Direct路径存在整数下溢问题。攻击者可构造恶意RDMA缓冲区描述符，使首个描述符长度大于声明的载荷长度。这会导致服务端的大小计数器回绕，后续执行内存拷贝操作时，数据就会超出已分配缓冲区的边界。利用该漏洞需要同时满足三个条件：网卡支持RDMA（硬件支持或加载rxe软件驱动）、KSMBD共享目录开启写入权限、客户端协商使用SMB Direct协议。  
  
  
ZDI-CAN-33774的危害范围更广，攻击者无需身份验证就可以利用，所有在445端口开放KSMBD服务的服务器都会受影响。该漏洞源于KSMBD分配处理程序表时存在竞态条件：攻击者在同一次TCP写入中先发送大量短SMB1数据包，再发送一个SMB2数据包，会导致工作进程用SMB2处理程序解析SMB1数据，读取长度超出较短缓冲区的边界。攻击者获取共享目录访问权限后，还可通过该路径实现越界写入。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0fE99dWAyibUIcBibLic8JRAdEX9S6VfHzuD1muZg4QibicBL4cdJeXqmtJbIoX9MO7oC4bSicl5X32ics7UE6MCYdnxub3qia8nsSwRM/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
主线内核暂无修复补丁  
  
  
截至周四，主线内核和用于合入新代码的linux-next集成分支中，都没有针对这两个漏洞的正式修复补丁。Linux科技站点Phoronix表示，SMB Direct维护者Stefan Metzmacher已提交针对ZDI-CAN-33549的修复补丁，目标合入下一个内核开发周期，但其他渠道尚未证实该消息。  
  
  
按照内核社区的常规流程，主线内核的补丁需要数周时间才能进入稳定分支，之后再推送至各Linux发行版的官方内核中。这两个漏洞目前都没有分配CVE编号，因为内核项目规定，只有稳定分支的修复补丁正式可用后，才会为对应漏洞分配CVE编号。  
  
  
管理员可运行systemctl status ksmbd或lsmod | grep ksmbd命令，确认KSMBD模块是否处于活动状态。红帽官方支持文档显示，红帽企业版Linux（RHEL）默认不包含KSMBD模块。  
  
  
Part  
03  
  
漏洞经公开邮件列表披露  
  
  
ZDI的漏洞报告显示，这两个KSMBD漏洞由趋势科技的AI扫描器TrendAI发现。2026年5月，Linux内核社区根据Linus Torvalds的指导意见更新了安全文档，明确AI工具发现的漏洞默认属于已公开信息，应当直接发送给对应子系统的开发者邮件列表。  
  
  
ZDI的漏洞通报流程似乎遵循了这一规则，在向内核私有安全邮箱发送报告的同时，也抄送了公开邮件列表，且所有报告都未附带补丁。  
  
  
稳定内核维护者Greg Kroah-Hartman在收到五份同批内核漏洞报告后的数分钟内，就对其中三份作出了回复。针对VMware GPU驱动的漏洞报告，他写道："不知出于什么原因，你们把这份报告发到了公开列表，也没有附带补丁，我们没法处理。"17秒后，他针对NVMe驱动的漏洞报告回复道："又是这样，没有补丁还发到公开列表？请修正你们的提交流程。"  
  
  
本次ZDI共公开的五份内核漏洞报告中，ZDI-CAN-33549属于KSMBD SMB Direct子系统的远程代码执行漏洞，需启用SMB Direct且支持RDMA；ZDI-CAN-33774属于KSMBD协议分发子系统的远程代码执行漏洞，所有KSMBD部署环境均受影响，两者CVSS评分均为9.0。  
  
  
其余三份与SMB服务无关：ZDI-CAN-33367是网络栈skb_zerocopy的本地权限提升漏洞，CVSS 7.8；ZDI-CAN-34068是VMware GPU驱动vmwgfx的本地权限提升漏洞，CVSS 8.8；ZDI-CAN-34076是NVMe目标驱动nvmet的信息泄露漏洞，CVSS 8.6。  
  
  
ZDI-CAN-33367由研究人员p1r0x发现，是内核网络栈中的本地权限提升漏洞，攻击者需要拥有本地访问权限才能利用。ZDI-CAN-34068是VMware虚拟GPU驱动中的本地权限提升漏洞，ZDI-CAN-34076是NVMe目标驱动中的信息泄露漏洞，这两个漏洞都不会影响KSMBD服务器。  
  
  
除上述五份内核漏洞报告外，ZDI还公开了第六份漏洞报告ZDI-CAN-32005。该漏洞存在于独立于Linux内核的网络守护进程ConnMan中，类型为缓冲区溢出。  
  
  
参考来源：  
  
Two CVSS 9.0 Linux KSMBD RCE Flaws Disclosed on Public Mailing Lists Before Kernel Fixes  
  
https://www.bleepingcomputer.com/news/security/two-cvss-90-linux-ksmbd-rce-flaws-disclosed-on-public-mailing-lists/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349044&idx=1&sn=d77f4721bae4ab99397417570be69c4f&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
