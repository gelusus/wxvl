#  12 年、1 条连接、61 字节越界写：Linux NFS满级漏洞可root  
原创 安妮
                    安妮  黑白之道   2026-11-27 00:59  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MtXt498G68F23SkwrfOD8hibr2hmTYEGibLue8kRYnVoGwuB4GAibS5zl6JdBmJ0SyXy52jcCx06IjgljAoicq68JpTvkNDNhic3w8/640?from=appmsg "")  
> **导语**  
：Linux 内核里的 NFS 服务器 nfsd，藏着一条能预认证拿 root 的堆溢出——CVE-2026-99362，CVSS 9.8 满分级。只要你的主机开着 NFSv4.1 或 4.2，任何能摸到 tcp/2049 的人，不用登录、不用账号、不用点任何东西，一条 TCP 连接就够写穿内核堆，把 nfsd  
 变成自己的 root shell。而这个洞，从 2014 年的 Linux 3.16 就埋下了。  
  
  
![NFS2ROOT 标题图：Linux 企鹅身后是 NFS 服务器的 root 之门](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PQjCOIeYa2icZvmpQJ4FBTl4OHKGiahEXhvBdnGKfYdA3nbcSDuniaw2Z45zXmF4icyd9NjzUbaIaBCWlNfmoNqTCVGSdTrj3gM00/640?from=appmsg "NFS2ROOT 标题图：Linux 企鹅身后是 NFS 服务器的 root 之门")  
## 一个连接，不需要登录  
  
NFS 服务器跑在内核里，它在搞清楚「你是谁」之前，就已经在解析网络输入了。CVE-2026-99362 正好卡在这个空档里：客户端可以把自己的 reply 缓存协商成 0 字节，然后让服务器把一整条 reply 拷进去。  
  
这次拷贝落在内核堆上，而且**拷贝内容里的大部分字节，是客户端自己挑的**  
。61 字节被写到堆对象边界之外，其中 40 字节由攻击者控制。一条未经认证的 TCP 连接，就能把这种越界写串成一次完整的 root 提权。  
  
![NFS2ROOT 攻击路径：攻击者经 2049 端口直达文件服务器、NAS、存储节点](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6Py5icytcBUAPDDTTjuDoz5rOeFHCJvfYyibrRCyLJapvS3hHkia2JoVbJgbNgw80L9bDwgibxcD0iaJHWq9HRVQ3e4FLyaWribawmuk/640?from=appmsg "NFS2ROOT 攻击路径：攻击者经 2049 端口直达文件服务器、NAS、存储节点")  
## 十二年的几行 C  
  
这条链的时间线摊开看，触目惊心：  
- **2014 年 3 月**  
：commit 47ee52986472  
 在处理 SEQUENCE 时收窄了 reply 缓冲区，随 Linux 3.16 合入主线。  
  
- **2014–2026，整整 12 年**  
：从 LTS、发行版到各类设备内核，所有 3.16 之后的内核都带着它。  
  
- **2026 年 8 月 17 日**  
：ANSSI 的 Jérémy Jean 带着一份 KASAN 报告，把修复补丁发到上游。  
  
- **2026 年 9 月 21 日**  
：OpenSec 的 Antara 标注了该漏洞，打了 9.8 分，并独立把它做成 root shell。  
  
- **2026 年 11 月 20 日**  
：CVE-2026-99362 随修复一同公开。  
  
## 根因：reply 缓存越界写  
  
每个 NFSv4.1 会话槽（slot）会缓存自己的上一条 reply，这样重试的请求能被恰好应答一次。**缓存多大，由客户端说了算。**  
  
把 ca_maxresponsesize_cached  
 压到最小值 NFSD_MIN_HDR_SEQ_SZ  
，nfsd4_alloc_slot()  
 就给每个槽分配一个**零长度的 sl_data[]**  
。接着 nfsd4_sequence()  
 在接收槽之前，先把 reply 缓冲区收窄到这个上限。再用 COMPOUND tag 填满缓冲区——填到刚好塞下 SEQUENCE 操作码、却塞不下后面的 status 字。  
  
于是 nfsd4_encode_operation()  
 直接返回，没跑 nfsd4_encode_sequence()  
，op->status  
 仍是 nfs_ok  
，服务器误以为 SEQUENCE 成功了。nfsd4_store_cache_entry()  
 本应拒绝缓存出错的操作，但它读到的状态是「成功」，就把整条 reply——RPC 头、COMPOUND 头、tag 全算上——拷进了零长度的 sl_data[]  
。  
  
上游修复附带的 KASAN 报告写得很直白：  
```
BUG: KASAN: slab-out-of-bounds in read_bytes_from_xdr_buf+0x1bc/0x390Write of size 80 at addr ffff888003a549cd __asan_memcpy+0x38/0x60 read_bytes_from_xdr_buf+0x1bc/0x390 nfsd4_sequence_done+0x5b0/0x810 nfs4svc_encode_compoundres+0x1bf/0x240
```  
  
80 字节被拷出，其中 **61 字节越过堆对象边界，40 字节由客户端控制**  
。  
  
![根因示意：reply 被拷进一个没有空间的槽缓冲区](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6M1cU0SyU5xdAXj74Dn9uVLm1NbwtmFFToKC1p5UeYEHT0SC5iaHf5WqQuUCkwIYAcx9p2pTFnbUKUbhgj6TS0UkKs8pWHydjKw/640?from=appmsg "根因示意：reply 被拷进一个没有空间的槽缓冲区")  
## 从溢出到 root  
  
漏洞利用本身保持私有，但每一步都是普通的 NFS 请求：  
1. **溢出**  
——越界写破坏相邻槽的记账数据。  
  
1. **泄露**  
——重放请求返回 3.8 KB 内核堆，顺带拿到 KASLR 偏移。  
  
1. **释放**  
——伪造指针，让内核释放一个由攻击者指定的对象。  
  
1. **回收**  
——用 NFS 客户端名重新填充那块内存，塞进一个假的回调。  
  
1. **触发**  
——挂断连接，nfsd 调用假回调，内核以 root 执行 /bin/sh  
。  
  
在演示录像里，那条命令是 whoami; id; poweroff -f  
：受害机先打印 root  
，再打印 uid=0(root)  
，然后自己关机。  
  
![实战演示：受害者控制台打印 root 与 uid=0(root) 后关机](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6NY9LawuSwOkEz49Lsgib2WWgslCQaAIAIlyAqcu9FYqBiapGK0X1rbAPeTdNricT2gPh7PAQsnRH5llljIDB6augQU6YPfdGtGz4/640?from=appmsg "实战演示：受害者控制台打印 root 与 uid=0(root) 后关机")  
## 机器自己找到的  
  
这条链的发现者不是人类研究员，而是 OpenSec 的白盒 harness **Antara**  
。它标记了 nfsd 的 reply 缓存路径，打了 9.8 分，然后**自己**  
完成了泄露、任意释放、root shell 的全过程——端到端，没有人类插手。  
  
需要强调的是：内核加固（KASLR、KASAN 等）并不能消除这个溢出本身，只是让利用更费劲。**每台主机都得打补丁。**  
## 怎么判断自己中招了  
  
对着下面四条自查，四条全中，今天你就暴露着：  
- 内核版本早于 6.12.117 / 6.18.61 / 7.3.2 / 7.4-rc1，且没有发行版 backport；  
  
- nfsd 已加载（lsmod | grep -w nfsd  
）；  
  
- 开着 NFSv4.1 或 4.2（cat /proc/fs/nfsd/versions  
 列表里有 +4.1  
 或 +4.2  
）；  
  
- 2049 端口监听在非回环地址上（ss -ltn 'sport = :2049'  
）。  
  
## 怎么办  
  
**打补丁并重启。**  
 修复版本：6.12 LTS → 6.12.117，6.18 → 6.18.61，7.3 → 7.3.2，主线 → 7.4-rc1。发行版内核则安装列有 CVE-2026-99362 的更新包。  
  
今天打不了补丁？先封死 2049，只放行存储网络：  
```
nft add rule inet filter input tcp dport 2049 ip saddr != 10.20.0.0/16 drop
```  
  
或者直接关掉 NFSv4.1 和 4.2，在 /etc/nfs.conf  
 里写：  
```
[nfsd]vers4.1=nvers4.2=n
```  
  
然后重启 nfs-server  
。代价是失去 pNFS 和部分 4.2 特性，但能立刻止血。  
## 排查：攻击本身是合法 NFS  
  
由于这条链走的是合法 NFS 流量，**网络层面没有特征签名**  
。要盯的是主机侧线索：  
- 每次尝试（无论成败）都会留下 nfsd4_check_resp_size  
 的内核告警；  
  
- 带 nfsd4_*  
、svc_*  
 或 call_usermodehelper*  
 帧的 oops；  
  
- 文件服务器上由内核 worker 拉起的 shell；  
  
- 来自同一源地址的短促 NFSv4.1 会话突发。  
  
## 披露与致谢  
  
OpenSec 于 2026 年 9 月 22 日向 NFSD 维护者与 security@kernel.org  
 提交了完整远程 root 漏洞利用，并在同日发送补丁；CVE-2026-99362 于 11 月 20 日随修复公开。  
- **OpenSec Intelligence**  
：用 Antara 发现漏洞，自主将其升级为预认证 root，并连同完整利用与补丁一起上报。  
  
- **Jérémy Jean（ANSSI）**  
：独立发现并编写上游修复 5d5a72e965c8  
 与 241848cc774e  
。  
  
- **Chuck Lever、Jeff Layton**  
：NFSD 维护者。  
  
**版权声明**  
：本文为转载翻译，原文版权归 OpenSec Intelligence 所有，原文地址：https://opensec.in/blog/nfs2root-cve-2026-99362 。图片均为原作者所有，按转载形式保留原文配图，特此致谢。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6P04AskLXLsn3BFUMCvE2Sibj5mvbNftfq4ibiczTvyLLv6f96Zw8iarBicIKzLhOPxReNv0w7NXWNs1WL8Do0NAp2LcFnbCr0Sdy6s/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6N9EUVWd8icz4LgHTQgV03WA6VLgoH8qBKSYVhLNjWtAiaOLn7luk1VDgEPYjY07aSzSa8dRbEJb6GmcZ16zl2LbREuAmI2B01GU/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6Nk8eDMIy4hNqKDsLso6iajg5DbiaMbc5P36fan5ZcibDmsFAqawCSCNqIjwXIIeRhMIBUia2boq8B4m2cL9eFr25OC9P7BteGt9T8/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6Oho3PIjmwOQtHFta0ELicUyDaFX7M1FwDoEF33YNrGqtyhwNTh0RLVajF6PNqh0PhfCE3uibvQVTE2zc0pJXvhmSzibkyPfIlVJc/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
