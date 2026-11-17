#  NoisyTCP：潜伏 11 年之久的 Linux 内核漏洞，可致未授权远程拒绝服务（DoS）  
 网安百色   2026-11-16 10:05  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WibvcdjxgJntVnTRibIKiakHgeh64GKVROYoApIEPH33cLmicuFXqUTnjzUJibb9Xm3SibDydcjt3eI6JmArpPlQeP5KfrnENneCRyhyQibIdicZcTw/640?wx_fmt=png&from=appmsg "")  
  
NoisyTCP 是一个潜伏了 11 年之久的 Linux 内核漏洞。它允许远程攻击者在无需身份验证（无需登录）的情况下使目标服务器崩溃。该漏洞编号为 CVE-2026-99288，由研究员 Hyunwoo Kim (@v4bel) 发现并提交修复。在 Linux 发行版的保密期（embargo）结束后，他在 GitHub 上正式发布了 NoisyTCP 的安全公告。  
  
该缺陷位于内核的 TCP 代码中，而该代码负责处理 Linux 系统上几乎所有的网络流量。Kim 表示，该漏洞可针对普通的 TCP 监听器（plain TCP listeners）触发。  
  
目标服务器无需任何特殊配置、防火墙规则或特定的 Socket 设置。只要调用了 listen()  
 函数并接收来自攻击者的数据包，即可触发漏洞。这意味着采用默认配置的 Nginx、Apache、SSH 服务器以及各类代理服务器均面临风险。  
  
攻击的前提是，目标服务器需具备两个或以上的接收队列（receive queues），或者启用了接收包转向（Receive Packet Steering, RPS）功能。这两种情况在现代硬件中极为常见。  
  
攻击者会发送 IPv4 分片和普通的 TCP 数据包，使其被调度到不同的 CPU 核心上处理。这会引发竞态条件（race condition），导致内核的两个不同部分在同一时刻访问同一块内存区域。  
### NoisyTCP Linux 内核漏洞详情  
  
在第一条触发路径中，一个 CPU 核心查找到了一条现有的半开连接（half-open connection）记录。与此同时，另一个 CPU 核心释放了该记录，并迅速将同一块内存重新分配给具有相同连接详情的新 SYN Cookie 请求。  
  
第一个 CPU 核心获取了这个新对象并通过了相关检查。然而，这个新对象并未绑定任何监听器（listener）。随后，内核在尝试读取该监听器时遇到空指针（NULL listener），从而触发内核恐慌（kernel panic）。  
  
最终结果是导致远程拒绝服务（Remote DoS）。如果容器化的 Web 服务器因此崩溃，将会导致整个宿主机宕机，而不仅仅是该应用本身。第二条触发路径可能导致远程释放后使用（Use-After-Free, UAF），但这需要连续多次竞态条件同时成功，利用难度极大。  
  
在此过程中，时序（Timing）至关重要。在 QEMU/KVM 等虚拟机环境中，测试耗时约为 5 秒至 2 分钟。而在拥有 8 个接收队列的闲置裸金属服务器上，经过调优的概念验证（PoC）代码大约需要 14 小时才能触发。因此，云环境和虚拟化服务器面临的暴露风险最高。  
  
这并非一个远程代码执行（RCE）漏洞。目前仅针对 VM 服务器构建了有限的、特定于实验室环境的利用代码（exploit），仍在研究通用的利用方法。尽管如此，对于任何面向互联网的服务而言，能够引发稳定崩溃的漏洞依然极其严重。  
  
该漏洞最早引入于 2015 年 10 月 3 日的提交（commit 079096f103fa  
）。修复补丁（commit 4e2b3ef19e0e  
）则于 2026 年 11 月 3 日合入主线。这意味着几乎所有的 Linux 发行版都携带此缺陷长达约 11 年之久。管理员应在厂商发布更新内核后尽快安装并重启系统。  
  
如果暂时无法打补丁，Kim 提出了两种临时缓解措施（stopgaps）。第一种是通过 sudo sysctl -w net.ipv4.tcp_syncookies=0  
 命令关闭 SYN Cookies。该设置按网络命名空间（network namespace）生效，因此每个拥有独立命名空间的容器也都需要单独应用此配置。  
  
但这也会削弱对 SYN 泛洪（SYN floods）攻击的防护能力，而这是导致服务器宕机的常见攻击手段。第二种是将网络接口的接收队列数量减少至 1 个，并禁用 RPS 和 RFS（Receive Flow Steering），从而从根本上消除竞态条件。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
