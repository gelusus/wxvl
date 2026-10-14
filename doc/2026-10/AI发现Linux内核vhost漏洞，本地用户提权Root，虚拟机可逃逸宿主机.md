#  AI发现Linux内核vhost漏洞，本地用户提权Root，虚拟机可逃逸宿主机  
 看雪学苑   2026-10-14 10:06  
  
腾讯朱雀实验室借助AI多智能体系统Corvus AI，发现一处潜伏Linux内核近9年的高危漏洞，漏洞位于内核`vhost`子系统，可实现本地权限提升至Root，特定虚拟化场景下，恶意虚拟机还能突破隔离，篡改宿主机内存实现宿主机逃逸。  
  
  
  
**漏洞基本概况**  
  
  
  
漏洞编号：CVE-2026-74580  
  
`vhost`是Linux内核组件，作用是加速虚拟机、容器的网络IO性能，用户通过`/dev/vhost-net`、`/dev/vhost-vsock`设备节点访问该能力。  
  
  
该漏洞存在两大攻击链：  
  
✅ 本地提权（最通用）  
  
本地非特权用户可利用漏洞，任意读写内核内存，绕过KASLR保护，稳定获取root权限。攻击无需复杂内存布局构造、不依赖运气时序，利用成功率高。  
  
  
✅ 虚拟机逃逸宿主机（特定场景）  
  
当虚机virtio设备挂载虚拟IOMMU、开启`iommu_platform`，vhost工作在device-IOTLB模式时，不受信任的客户机可触发该漏洞，造成宿主机内核越界写入，篡改宿主机QEMU进程函数指针，控制宿主机进程或直接造成系统崩溃。  
  
  
  
**漏洞成因**  
  
  
  
漏洞的形成由两次内核代码变更共同造成：  
  
1. 2017年（Linux 4.11版本）新增内存映射缓存机制，用缓存命中判定内存区域合法性；  
  
2. 2020年（Linux 5.9版本）将地址校验逻辑后置，移除了原本可拦截非法地址的检查。  
  
  
缓存机制存在设计缺陷：当队列处于运行状态，`VHOST_SET_VRING_ADDR`、`VHOST_SET_VRING_NUM`指令修改虚拟环地址/大小的时候，不会清空旧缓存。  
  
系统会复用缓存里过期的映射信息，不再执行地址合法性校验，最终触发越界读写。这个缺陷在代码中潜伏了约9年。  
  
  
  
**受影响版本与修复版本**  
  
  
  
受影响内核范围：Linux 4.11 ~ 7.2-rc4  
  
  
上游稳定分支修复版本一览：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3mZW1x5zlmRD0Q5Uz7Lb0fP1ibkKkFGcjzAkmqHWk4nUqFyyTWd1qSdyL2V2ZVpkibDxH028WfYzN5hjpshG7PJzESkzkiaAKUeI/640?wx_fmt=png&from=appmsg "")  
  
说明：部分发行厂商会单独回补补丁，内核版本号不一定变化，请以厂商公告为准。  
  
  
测试验证：腾讯团队在Ubuntu 26.04、RHEL 9.8、Amazon Linux 2023、Azure Linux 3.0、Oracle Linux 10.2复现完整攻击链。  
  
- Ubuntu：vhost设备节点归属kvm用户组，仅该组成员可利用；  
  
- 其余4个系统：设备节点权限默认为0666，**任意本地用户均可访问**。  
  
  
  
**处置建议**  
  
  
  
1. 优先方案：升级内核至修复版本并重启主机  
  
上游修复逻辑：在虚拟环配置变更时，同步清空内存缓存，杜绝过期映射复用。  
  
厂商进度：红帽、亚马逊Linux、Oracle Linux已发布修复包；Ubuntu 22.04已修复，24.04、26.04补丁仍在推进（截至10月13日）。  
  
  
2. 临时缓解（仅针对本地提权路径，无法防御虚机逃逸）  
  
收紧`/dev/vhost-net`、`/dev/vhost-vsock`权限，取消全局可读权限，限定仅可信虚拟化服务账号所属用户组访问；将该用户组视作高权限组管控。  
  
虚拟机逃逸场景没有临时缓解方案，必须打补丁。  
  
  
  
**漏洞评级与发现背景**  
  
  
  
不同机构CVSS评分存在差异：  
  
- kernel.org CVE团队：8.8分（高危，范围变更）  
  
- Red Hat：7.3分  
  
- Amazon：7.8分  
  
- Oracle：7.3分  
  
漏洞由腾讯朱雀实验室自研多智能体内核漏洞挖掘系统Corvus AI辅助发现，属于机器辅助安全研究，并非AI自主发起攻击。该团队此前在8月披露的SCTP内核漏洞，同样依靠这套流水线挖掘。目前暂无公开在野利用报告。  
  
  
资讯来源：The Hacker News、腾讯朱雀实验室  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K2oiazibYgPKvV2RwrqIvLIkCwnibguvWRuAxeficnaP9F6UujI6Wl149mPG5ftqRVwV8qb0u4OTNibHd1ic5VJ3iaicHzRtkS9AKEbKYA/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3rIgawCrib9r9WPyKarZzFTtGXHffsia5qlZAZtJxLdkofQNv1lSPB3cW62hePlobwQWlVWF67gXZBkrp5qaqRBFDRH8AvibWN0w/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0JLricibl3dPicbuKbVP7rlkjDrPRCnPgygTkTpPByic1t0xvsA1gJ17XcVdicMtIicHPfQFHFmWg3cEoWPibe57orRg8Q8y8icQoEoA8/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2rIlrjEf7A7xITxR0zz6RIZu732BxgrNVb5IL5IszOCKWymSwKwoTRe9Yt4vxgXB9optlkjZ1RfPyxd4ibgkGKb6um0AXxnQiag/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
  
