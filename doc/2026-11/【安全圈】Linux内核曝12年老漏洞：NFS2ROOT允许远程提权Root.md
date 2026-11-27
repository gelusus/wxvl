#  【安全圈】Linux内核曝12年老漏洞：NFS2ROOT允许远程提权Root  
安全圈
                    安全圈  安全圈   2026-11-27 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Linux 内核网络文件系统服务端（nfsd）被爆出潜伏长达 12 年的超高危漏洞，官方追踪编号为 CVE-2026-99362，代号“NFS2ROOT”。远程未经身份认证的攻击者只需面向开放的 TCP 2049 端口发送特制 RPC 请求，即可破坏内核内存，绕过 KASLR 内存保护机制，直接接管整个服务器并获取最高 Root 执行权限。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyGWVDZxCEibGgDYk4y7z1YyALn6F635ibLqBfiaCjD2wHld9afzu9076pKlvp2kPmUWy915XEqsvKvB8EKwk23wNKj8ARoWbkHgRU/640?wx_fmt=jpeg&from=appmsg "")  
  
该漏洞由法国国家网络安全局（ANSSI）安全研究员 Jérémy Jean 首先捕捉，网络安全机构 OpenSec 随后完成了完整的端到端远程提权 Exploit 验证。漏洞代码追溯至 2014 年 3 月发布的 Linux 内核 3.16 版本，在生产环境中隐藏超过十年。  
## 🔍 漏洞机理：nfsd 堆缓冲区溢出与 KASLR 绕过  
  
NFS2ROOT 的核心缺陷归属于 Linux 内核 NFSv4.1 及 NFSv4.2 协议处理实现：  
- **边界校验缺失引发堆溢出**  
：攻击者向服务端发送畸形结构请求，诱使 nfsd 模块在向预先分配的内存缓冲区写入响应数据时发生越界覆盖，引发内核堆缓冲区溢出（Heap Buffer Overflow）。  
  
- **内存泄露击穿 KASLR**  
：通过越界读取与受控覆写，攻击者能够稳定泄露内核基地址（Base Address），精准定位内核数据结构与关键函数指针，使现代 Linux 核心防护 KASLR 完全失效。  
  
- **回调注入与无交互 Root 触发**  
：攻击者在释放的内核内存槽位中植入回调结构体。当攻击端主动发起四次挥手断开 NFS TCP 会话时，内核工作线程自动执行嵌入的回调指令，直接派生出拥有 uid=0 的特权 Root Shell。  
  
## ⚠️ 敞口测绘：NAS 存储与 Kubernetes 容器集群重灾区  
  
不同于需要本地普通用户凭据的常见提权漏洞，NFS2ROOT 属于完全无需认证、无需交互的纯远程攻击向量：  
  
**企业基础设施高危暴露**  
：在大量企业内网、混合云甚至公网环境中，TCP 2049 端口被长期开放用于 NAS 存储设备、自动化构建农场、持久化备份归档以及 Kubernetes 容器集群的存储卷挂载（CSI）。攻击者一旦触碰该端口，即可将文件共享节点转化为跳板机。  
## 🛡️ 受影响内核版本与应急补救策略  
  
只要系统加载并运行了 nfsd 服务，凡低于以下版本的内核均处于易受攻击区间：  
- **官方补丁升级**  
：Linux 内核维护团队已发布修复提交，管理员应立即升级至 6.12.117、6.18.61、7.3.2、7.4-rc1 或各 Linux 发行版官方推送的最新安全内核，并重启使新内核生效。  
  
- **网络层紧急隔离**  
：严禁 TCP 2049 端口直接对公网暴露；在边界防火墙与云安全组中严格配置访问控制列表（ACL），仅允许指定受信任的内网 IP 访问。  
  
- **停用未授权服务**  
：若业务节点不需要提供 NFS 服务，应在 /etc/nfs.conf  
 或通过 systemctl stop nfs-server  
 直接禁用该服务。  
  
- **内网威胁捕获特征**  
：重点监控系统日志中来自 nfsd4_check_resp_size  
 的异常 Warning，排查文件服务器上由内核工作线程非正常衍生的 Shell 进程。  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】1Password客户数据被盗](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=1&sn=c793b7ec19968c2a4193647ef032b172&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Drupal紧急修复高危在野漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=2&sn=d0a76361f98bec89bc0c6880d05cc226&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】假面试题暗藏恶意NuGet包](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=3&sn=e2707d02974793f7ea6a669aaa8a65c5&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】美中情局确认遭网络入侵：开源情报分析系统失守](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=1&sn=51b9ba7239b6c1d7b4895d8f712c93c9&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Mware曝9.8分高危越权：vCenter目录服务再遭突破](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=2&sn=3241348bb1eaf2254cc0a28b11f7c3a1&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
