#  黑客利用NoisyTCP 的新漏洞可导致 Linux 服务器内核崩溃  
Rhinoer
                    Rhinoer  犀牛安全   2026-11-22 16:04  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8JCDJGm1VMXBloicD9tJDI86gXgFaMib4w02hItibOekwo6DMbEdfo5gGDe3KSHQQaNrnvs5DYbkAXfAbMTq28xmjg6IjpPGPN3iaw/640?wx_fmt=png&from=appmsg "")  
  
一个名为NoisyTCP的 Linux 内核漏洞已有 11 年历史，几乎影响所有 Linux 发行版，攻击者可以利用该漏洞使运行 Nginx、Apache 和 SSH 服务器且配置为默认配置的系统崩溃。  
  
据发现该漏洞的安全研究员 Hyunwoo Kim 称，这种预认证远程数据损坏漏洞（编号为CVE-2026-99288）源于Linux 内核 TCP 网络子系统中的空指针解引用弱点。  
  
未经身份验证的攻击者可以通过恶意构造的 TCP 数据包远程触发该漏洞，利用 SYN cookie 处理和套接字哈希之间的时间差，使目标服务器上的内核和操作系统崩溃。  
  
然而，虽然 CVE-2026-99288 漏洞利用会导致内核崩溃并使主机瘫痪，但它们并不允许攻击者更改数据或获得任意代码执行权限。  
  
Kim补充说，虽然攻击者可以利用此漏洞攻击任何直接接收TCP数据包并运行Nginx、Apache或SSH服务器的Linux系统，“但竞争窗口很窄，因此对一台裸机服务器触发此漏洞可能需要很长时间。”  
  
红帽安全团队解释说：“由于无锁套接字查找期间存在竞争条件，回收的套接字内存可以在正确初始化和哈希之前被访问。远程攻击者可以通过生成驱动套接字重用的网络流量来利用此问题，从而触发空指针解引用和内核崩溃，最终导致拒绝服务 (DoS) 攻击。 ”  
  
研究人员说：要触发此漏洞，必须在不同的接收上下文中同时进行过期的 ehash 查找以及释放和重新分配现有请求套接字的操作。数据包处理必须通过使用 IPv4 分片和常规 TCP 数据包之间的接收哈希差异来分散，因此可以针对具有两个或多个接收队列的服务器，或者 RPS 将数据包分发到多个 CPU 的服务器触发此漏洞。  
  
虽然 CVE-2026-99288 补丁已经可用，并且修复程序已于两周前合并到 Linux 内核主线中，但研究人员建议那些无法立即应用该补丁或升级的用户禁用 SYN cookie（但这会降低对 SYN 洪水攻击的防护），并将数据序列化到单个 RX 队列，以阻止成功利用该漏洞所需的竞争条件。  
  
今年 5 月，Kim 还披露了Dirty Frag，这是一个 Linux 本地权限提升漏洞，它利用 RxRPC ( CVE-2026-43500 ) 和 xfrm-ESP ( CVE-2026-43284 ) 页面缓存写入漏洞，在包括 Ubuntu、Red Hat Enterprise Linux、CentOS Stream 和 Fedora 在内的主要发行版上获取 root 权限。  
  
最近，在 7 月份，他还披露了Januscape，这是一个存在了 16 年的 Linux 内核漏洞，攻击者可以利用它逃离虚拟机并在主机上运行任意代码。  
  
  
信息来源：BleepingComputer  
  
