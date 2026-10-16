#  Linux vhost高危漏洞PoC公开，可致root提权与虚拟机逃逸  
 FreeBuf   2026-10-15 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX185z2mibWrsL7RyKPEBU9VjxzfbNsx50sTp3gFJy48fvGiaosTzPn9OGia1TBibrzJuBRVGacpALliafgzFDsiaib8iafWfv7zjt8XwEk/640?wx_fmt=gif "")  
  
  
![配图](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX3LQJ7lqI80kIBNwKia9c80SIhhia6c7JRtt0MbXAZtXX7IgOW7M8VDxxvbgMVedRl2qyR3fiavVxpy6J9JExicsNBnj0VWmXHzicIk/640?wx_fmt=jpeg "")  
  
  
Part  
01  
  
vhost缓存校验存在缺陷  
  
  
Linux vhost高危漏洞（CVE-2026-74580）代号为Vghost，目前已有公开PoC。验证结果显示，本地攻击者可利用该漏洞获得root权限；在特定虚拟机配置场景下，还能突破客户机隔离，影响宿主机运行。该漏洞CVSS v3.1评分为8.8，Linux内核新版本已推送修复补丁。  
  
  
vhost是Linux内核中的一套子系统，可提升虚拟机处理网络及其他设备流量的效率。它通过名为virtqueue的共享虚拟队列，在客户机与宿主机之间传输设备数据。此次漏洞出现在vhost缓存IOTLB映射的逻辑中，系统正是依靠这些映射在内存中定位virtqueue。  
  
  
漏洞触发场景位于虚拟环重配置流程中。内核会更新虚拟环的地址或大小，但不会清除环上描述符、可用区、已用数据对应的缓存映射。  
  
  
后续若命中该缓存，系统会跳过应有的地址校验流程，直接用旧映射转换新地址。最终生成的指针可能携带无效目标地址，传入内核拷贝例程。  
  
  
Seclists邮件列表上的研究人员溯源发现，该漏洞源于此前两次内核变更：一次引入了元数据缓存机制，另一次推迟了客户机I/O地址的校验流程。  
  
  
Part  
02  
  
PoC验证两类利用路径  
  
  
针对本地提权场景，攻击者首先需要拥有打开/dev/vhost-net、/dev/vhost-vsock等vhost设备的权限。PoC的利用流程为：先预热缓存，在不清除对应缓存条目的前提下修改环地址，随后触发队列运行。利用该漏洞可实现内核内存的读写。  
  
  
已公开的利用链首先通过内核内存读取获取地址随机化偏移，随后修改内核的modprobe_path配置。触发内核模块请求后，就能以root权限运行攻击者控制的辅助程序。  
  
  
目前该利用链已在Ubuntu 26.04、RHEL 9.8、Amazon Linux 2023、Azure Linux 3.0、Oracle Linux 10.2上成功复现。不同系统的设备权限配置存在差异：测试用Ubuntu系统仅允许kvm用户组访问vhost设备，其余测试系统则向普通用户开放该设备权限。  
  
  
这一差异意味着设备权限是重要的暴露面检查项，但不能替代补丁修复。在系统未打补丁期间，应将kvm或其他拥有vhost设备访问权限的用户组成员视为特权账户。  
  
  
第二条利用路径的起点在虚拟机内部。该路径需要满足几个配置条件：vhost设备使用虚拟IOMMU、开启iommu_platform参数、启用virtio队列重置特性VIRTIO_F_RING_RESET。攻击者在客户机内重置并重新启用队列，会诱使宿主机重配置虚拟环，此时过期的元数据仍残留在缓存中。  
  
  
客户机触发队列后，过期映射会导致已用环的写操作越出客户机内存范围，写入宿主机上的QEMU进程堆。这种内存破坏可导致QEMU崩溃，或篡改其控制流。需要注意的是，该虚拟机逃逸风险仅存在于特定配置场景，并非所有虚拟机都能通过该漏洞实现逃逸。  
  
  
Part  
03  
  
官方已发布内核修复补丁  
  
  
上游内核的修复方案是，在虚拟环（vring）地址或大小变更时清除元数据缓存，确保新配置必须通过IOTLB校验。  
  
  
目前受影响内核分支的首个上游稳定修复版本已发布，分别为5.10.265、5.15.216、6.1.183、6.6.152、6.12.104、6.18.45、7.1.9、7.2。管理员应安装官方修复版内核或厂商提供的向后移植补丁，重启系统后生效。  
  
  
在完成补丁安装前，应限制所有vhost设备节点的访问权限，包括/dev/vhost-net、/dev/vhost-vsock、/dev/vhost-scsi、/dev/vhost-vdpa-*，不要将这些设备透传给不可信容器。  
  
  
针对未打补丁的虚拟化宿主机，不要将虚拟IOMMU后开启iommu_platform的vhost设备暴露给不可信客户机。需要注意的是，权限修改不会回收已经打开的文件描述符，也无法阻断虚拟机内部触发的利用路径。  
  
  
参考来源：  
  
PoC Released for Linux Vhost Flaw Enabling Root Privilege Escalation and VM Escape  
  
https://cybersecuritynews.com/poc-released-for-linux-vhost-flaw/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
