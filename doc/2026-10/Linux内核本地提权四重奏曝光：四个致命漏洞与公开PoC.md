#  Linux内核本地提权四重奏曝光：四个致命漏洞与公开PoC  
原创 Dr. Clay
                    Dr. Clay  黑白之道   2026-10-09 00:40  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6Pax1wdUdmUMJLWgpQSrjWJXUXpdBIicjPjTLzfRd5lGmMXt2tAwKI96GyzhPX36fFHY10FPUD1icCic0iaQgOof96XKyn6iay9bTGQ/640?from=appmsg "")  
> **导语**  
：2026年9月18日，安全研究员Asim Manizada在个人博客发布了一篇重磅研究文章，公开了四个Linux内核本地提权（LPE）漏洞的完整漏洞分析与工作PoC。这四个漏洞被命名为"本地提权四重奏"（LPE Quartet），分别对应IPv6 AH6、TUN/TAP、PPPoE和SCTP诊断子系统，影响所有未及时更新的Linux系统。从PoC发布那一刻起，**任何本地非特权用户都能在受影响主机上拿到root shell**  
——这场静默的提权风暴正在蔓延。  
  
## 一、事件概述  
### 背景介绍  
  
Linux内核是全球数百万服务器和云主机的底层基石。2026年9月，研究员Asim Manizada在完成与内核团队和发行版厂商的协调 embargo 后，按约定时间公开了四枚"雷弹"的完整利用代码。这四个漏洞分别是：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">漏洞代号</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE编号</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">涉及子系统</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVSS</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">攻击前提</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">DirtyAH6</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-80844</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">IPsec / AH6 (IPv6认证头)</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">7.8+</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">需开启IPv6 + AH6/XFRM支持</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">TUNderflow</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-81000</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">TUN/TAP 虚拟网卡</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">7.8+</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">需unprivileged user namespaces + TUN支持</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">PPPoEject</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-68121</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">PPPoE (点对点协议 over Ethernet)</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">7.8+</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">需unprivileged user namespaces + PPPoE支持</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">DiagSpill</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-74469</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">SCTP诊断接口</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9.8</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">需SCTP协议支持</span></section></td></tr></tbody></table>  
CISA于同月将这批漏洞中的部分纳入已知被利用漏洞（KEV）目录，并要求联邦机构在规定时间内完成修复。  
  
![Linux内核四重奏漏洞攻击面总览](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6PvWPzBsm5MjGXgT2b45722ekhdygkCINqFr854rzekgHQQwntPeFjarRehDhzmKBXCPJBTsC21O9FAGg3pd30exHtC0RUs64U/640?from=appmsg "Linux内核四重奏漏洞攻击面总览")  
## 二、技术分析  
### 2.1 DirtyAH6（CVE-2026-80844）——IPsec AH6元数据破坏  
  
**Root Cause**  
: 漏洞位于Linux内核的IPv6认证头（Authentication Header, AH6）处理路径。当系统处理经过IPsec隧道、带有AH6扩展头的IPv6数据包时，内核在 xfrm_input()  
 回调中对 socket buffer（skb）的元数据进行了不当操作。攻击者构造一个畸形的AH6数据包，可导致skb元数据破坏，随后通过覆写 /etc/pam.d/su  
 文件实现本地提权。  
  
**利用条件**  
：  
- 内核编译时启用 CONFIG_INET6_AH  
 和 CONFIG_XFRM  
（IPsec）  
  
- 非特权用户命名空间可用（sysctl kernel.unprivileged_userns_clone=1  
）  
  
**验证（PoC结构）**  
：  
```
// poc.c 核心片段（来源：github.com/manizada/DirtyAH6）inttrigger_ah6_uaf(int fd) {    structipv6hdr *ip6h;    structauth_hdr  *ah;    char pkt[256] = {0};    ip6h = (void *)pkt;    ah   = (void *)(pkt + sizeof(*ip6h));    ip6h->version = 6;    ip6h->nexthdr = IPPROTO_AH;    ah->nexthdr   = IPPROTO_TCP;    ah->hdrlen    = 0;   // 畸形值，触发元数据破坏    send(fd, pkt, sizeof(pkt), 0);    return0;}
```  
  
**影响范围**  
：Fedora 43、Ubuntu（具体版本见各发行版安全公告）、启用了IPsec AH6支持的所有主流Linux发行版内核 < 6.12.109。  
### 2.2 TUNderflow（CVE-2026-81000）——TUN/TAP接收超限破坏  
  
**Root Cause**  
: TUN/TAP是Linux虚拟网卡驱动，允许用户空间程序接收/发送网络数据包。该漏洞源于TUN/TAP驱动的 tun_get_user()  
 函数在处理来自用户空间的超长数据包时，未正确校验 skb  
（socket buffer）的 truesize  
。攻击者通过发送一个 oversize 的数据包，使 skb->truesize  
 与实际数据大小产生不匹配，进而在接收路径触发内存覆写，最终控制内核执行流。  
  
**利用条件**  
：  
- CONFIG_TUN  
 编译进内核  
  
- 非特权用户命名空间已开启（默认多数发行版开启）  
  
- 存在可访问的TUN/TAP设备路径（如 /dev/net/tun  
）  
  
**攻击链**  
：  
```
open("/dev/net/tun") → create TUN device →  write oversized pkt → skb truesize mismatch →    heap out-of-bounds write → overwrite cred struct → root
```  
### 2.3 PPPoEject（CVE-2026-68121）——PPPoE处理内存溢出  
  
**Root Cause**  
: PPPoE（Point-to-Point Protocol over Ethernet）在内核的 pppoe  
 模块处理阶段存在一个类型混淆漏洞。当PPPoE帧的 tag  
 字段被精心构造时，内核错误地将某个指针解引用为整数进行偏移计算，产生任意内核地址写。攻击者利用这个原语覆写 modprobe_path  
 或 acct  
 指针，下次触发特权程序执行时即获得root shell。  
  
**利用条件**  
：  
- CONFIG_PPPOE  
 或 CONFIG_PPPOATM  
 加载  
  
- 非特权用户命名空间可用  
  
### 2.4 DiagSpill（CVE-2026-74469）——SCTP诊断子系统Use-After-Free  
  
**Root Cause**  
: 这是四重奏中危险系数最高的漏洞，涉及SCTP（Stream Control Transmission Protocol）的诊断接口。SCTP在内核中维护 assoc  
（关联）结构，当处理 SIOCSCTPIOCTL  
 等诊断ioctl时，如果一个已释放（stale cookie）的 assoc  
 被再次引用，就会触发 **Use-After-Free（UAF）**  
。攻击者可利用UAF释放后重分配（free-hook→code-pointer劫持）获得内核态代码执行。  
  
该漏洞与早些时候披露的 **CVE-2026-52924**  
（SCTP outqueue purge UAF）有相似之处，但DiagSpill专注于诊断路径，影响面更直接。  
  
**关键代码路径**  
：  
```
sctp_diag_ioctl() → sctp_assoc_lookup() →  stale cookie check bypassed → use-after-free →    kmem_cache_alloc (realloc) → overwrite with ROP chain
```  
## 三、PoC获取方式  
  
研究员Asim Manizada已在GitHub上公开了所有四个漏洞的完整PoC代码仓库，以下为直接下载链接：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">漏洞</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">GitHub仓库</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">关键文件</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">DirtyAH6</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">https://github.com/manizada/DirtyAH6</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exploit/poc.c</span></code><section><span leaf=""> + </span><code><span leaf="">Makefile</span></code></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">TUNderflow</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">https://github.com/manizada/TUNderflow</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exploit/*.c</span></code><section><span leaf=""> + </span><code><span leaf="">Makefile</span></code></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">PPPoEject</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">https://github.com/manizada/PPPoEject</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exploit/*.c</span></code><section><span leaf=""> + </span><code><span leaf="">Makefile</span></code></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">DiagSpill</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">https://github.com/manizada/DiagSpill</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">exploit/*.c</span></code><section><span leaf=""> + </span><code><span leaf="">Makefile</span></code></section></td></tr></tbody></table>  
**快速验证方法**  
（以DirtyAH6为例）：  
```
# 克隆仓库git clone https://github.com/manizada/DirtyAH6cd DirtyAH6# 编译PoC（需要 kernel-headers 和 gcc）make# 以普通用户运行（目标系统需满足利用条件）./exploit/dirtyah6_poc# 验证提权id    # 确认 uid=0cat /etc/shadow  # 读取shadow文件证明root权限
```  
## 四、影响范围与修复方案  
### 影响范围  
  
以下Linux发行版默认内核配置会受到影响：  
- **Ubuntu**  
 22.04/24.04 LTS（内核 < 6.12.109）  
  
- **Fedora**  
 43（内核 < 6.12.109）  
  
- **Debian**  
（部分内核分支）  
  
- **CentOS / Rocky Linux / AlmaLinux**  
 8.x 和 9.x（见各发行版SA）  
  
- **所有使用容器共享宿主机内核的多租户环境**  
（K8s、VM共宿等场景风险极高）  
  
### 修复方案  
  
**方案一（首选）：升级内核**  
```
# Ubuntu/Debiansudo apt update && sudo apt install linux-image-$(uname -r)# Fedorasudo dnf update kernel# 检查当前内核版本uname -r
```  
  
**方案二（缓解）：禁用非特权用户命名空间**  
 如果暂无法升级内核，可通过以下方式切断 DirtyAH6、TUNderflow、PPPoEject 的利用路径：  
```
# 临时缓解（重启失效）sysctl -w kernel.unprivileged_userns_clone=0sysctl -w net.ipv6.conf.all.disable_ipv6=1   # 禁用IPv6（阻断DirtyAH6）# 永久缓解echo 'kernel.unprivileged_userns_clone=0' >> /etc/sysctl.d/99-lpe-quartet.conf
```  
  
**方案三（针对DiagSpill）：禁用SCTP（若无业务需求）**  
```
# 确认是否加载sctp模块lsmod | grep sctp# 如无业务需求，列入黑名单echo "blacklist sctp" | sudo tee /etc/modprobe.d/sctp-blacklist.confsudo modprobe -r sctp 2>/dev/null || true
```  
## 五、漏洞验证与排查  
  
管理员可通过以下命令确认系统是否受影响：  
```
# 1. 检查内核版本是否低于 6.12.109uname -r# 2. 检查是否启用了非特权用户命名空间cat /proc/sys/kernel/unprivileged_userns_clone# 3. 检查IPv6和IPsec/AH6是否启用ip6tables -L -n 2>/dev/null || echo "IPv6 not active"# 4. 检查SCTP是否加载lsmod | grep sctp# 5. 检查TUN设备是否可用ls -la /dev/net/tun
```  
## 六、总结与趋势预测  
  
**为什么这四个漏洞同时爆发？**  
 表面上是独立漏洞，背后是Linux内核在网络子系统长期积累的技术债务。IPsec AH6、TUN/TAP、PPPoE、SCTP都是相对"老旧"的组件，代码维护不活跃，边界情况容易被忽视。本次四重奏同时公开PoC，标志着**漏洞武器化门槛已降至"下载-编译-运行"的三步**  
。  
  
**未来趋势：**  
1. **容器逃逸+本地提权组合**  
将成为云环境横向移动的标准路径。本地root≈宿主机root，配合容器逃逸漏洞（如runC CVE）可实现一键控制整个节点。  
  
1. **内核安全研究方向转向**  
。随着内核直接漏洞披露频率加快，研究者注意力正在向eBPF supervisor调用、io_uring等新型子系统转移。  
  
1. **发行版内核碎片化加剧**  
。Ubuntu、Fedora等维护各自内核分支，补丁推送时间差将产生大量"灰色地带"主机——这是攻击者的狩猎场。  
  
**给管理员的一句话**  
：不要等，重启之前先拿shell。  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OTxxtaiackJXjicN1jlbbxjia4lKk4zEhBX9mLLLHF2B5n4R9pyE8UnrElHeS9mhdrtibXb6VATf8DArCnIERYLLM2IyPYQ1O5UJY/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
