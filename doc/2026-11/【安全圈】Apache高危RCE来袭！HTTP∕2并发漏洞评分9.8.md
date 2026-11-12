#  【安全圈】Apache高危RCE来袭！HTTP/2并发漏洞评分9.8  
安全圈
                    安全圈  安全圈   2026-11-12 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Apache HTTP Server 项目于 11 月 11 日发布 **2.4.71**  
 安全更新，修复其 HTTP/2 模块（mod_http2  
）中的未授权远程代码执行漏洞 CVE-2026-119618  
（Red Hat CVSS 3.1 评分 **9.8**  
）。该漏洞影响 **2.4.55 至 2.4.70**  
 全系启用 HTTP/2 的实例，Wiz 云遥测数据显示 64% 的云环境运行着受影响版本，51% 直接暴露在公网。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyEjUKyT3lkhgOceHjs4dEeoia4sLjrd7NtcjEck25kF97A2I2eN5ROibDiaPh8EkYQAvQTTeCAEfyBt3omiaiaRCPs0l3CG1ZrJ4o2s/640?wx_fmt=other&from=appmsg "")  
## ⚡ 漏洞根因：assign_headers() 并发竞态写穿堆内存  
  
CVE-2026-119618  
 由 Anthropic 旗下自动化工具 OSS Scanner（内部编号 ANT-2026-MAPKT6MZ  
）于 10 月 26 日率先上报，Google Cloud 漏洞研究团队的 Anthony Weems 与 Simon Scannell 随后在 11 月 3 日独立发现了同一缺陷。  
  
HTTP/2 允许客户端在单条 TCP 连接上并发塞入大量请求，Apache 会调度多个工作线程并行处理这些流。问题出在 mod_http2  
 的 assign_headers()  
 函数：由于缺少线程同步保护，工作线程会直接通过共享的无锁内存分配器向连接线程名下的内存块写入数据，造成堆内存损坏并最终转化为以 Web 进程权限执行任意命令的 RCE 漏洞。  
  
只有通过 Protocols  
 指令（无论全局或仅单个 VirtualHost  
）开启了 h2  
 或明文 h2c  
 的节点会受到攻击，纯 HTTP/1.1 服务不受影响。同时需要留意，已在 Apache 2.4.67 中修补的早期 HTTP/2 漏洞（CVE-2026-23918  
）并不能免疫本次竞态缺陷。  
## 🔍 利用现状与版本矩阵：RHEL 因内存隔离降级评级  
  
对于当前威胁紧迫度，Red Hat 产品安全团队在通报中给出高度警惕，称“公开研究已经演示了针对默认 mod_http2  
 配置的稳定漏洞利用（reliable exploit）”，并给出 CVSS:3.1/AV:N/AC:L/PR:N/UI:N  
 满分网络向量；不过 Wiz 表示截至目前尚未发现公开可用的 PoC 代码，也暂未监测到在野攻击事件。  
  
底层的内存分配器策略直接改变了不同发行版的实际利用难度。RHEL 9 与 RHEL 10 随附的 mod_http2  
 为每个 HTTP/2 流划拨独立分配器，大幅抬高了跨流写坏堆内存的门槛，因此 Red Hat 将两者定级为“重要（Important）”而非“严重（Critical）”；RHEL 8 及更早版本因内置模块早于 2.0.8，完全免疫该问题。当前全系受影响与修复进度如下：  
- 📦 **Apache HTTP Server 上游**  
：**2.4.55 – 2.4.70**  
 受影响，**2.4.71**  
（提交 824beab  
）已修复，并同步解决了 mod_md  
 空指针解引用崩溃（CVE-2026-106023  
）及 mod_vhost_alias  
 1 字节栈溢出（CVE-2026-109163  
）。  
  
- ⚠️ **独立 mod_h2 仓库**  
：**2.0.8 – 2.0.42**  
 受影响；由于该 GitHub 独立项目已于 2026 年 6 月 9 日归档停更，仓库内不会再发布单独修复。  
  
- 💻 **Linux 发行版状态**  
：RHEL 9（RHSA-2026:98981  
，mod_http2-2.0.26-7.el9_9.1  
）与 RHEL 10（RHSA-2026:98978  
，mod_http2-2.0.29-5.el10_3.1  
）及 EUS 均已放行；Debian sid  
 已升级 2.4.71-1  
，但 bookworm  
、trixie  
 与 forky  
 以及 Amazon Linux 2/2023 仍在等待更新，Ubuntu 暂未列出独立公告。  
  
## 🛡️ 运维升级与临时停用 HTTP/2 处置  
- ✅ **优先打补丁升级**  
：公网暴露的 Apache HTTP Server 节点与容器镜像应第一时间升级至 **2.4.71**  
 或发行版对应勘误包；正在使用独立编译 mod_h2  
 的环境须切换回 httpd 2.4.71 主线源码。  
  
- ✅ **来不及升级时剥离 h2/h2c**  
：在 httpd.conf  
 及所有 VirtualHost  
 配置中移除 h2  
 与 h2c  
，仅保留 HTTP/1.1 协议后重载服务即可切断攻击路径：  
  
```
Protocols http/1.1
```  
  
  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】微软11月修1259洞！Outlook零日盗取NTLMv2](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079834&idx=1&sn=87414c25c31b4e94e839597e00255ff4&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Railway遭AI代理穿透！10万工作区泄密烧掉74万美元](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079834&idx=2&sn=f9a3ec5c78818d7d7509c38d4e00528b&scene=21#wechat_redirect)  
  
  
  
[【安全圈】iOS零日沙盒逃逸曝光！iOS27.2.1仍可跨应用窃密](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079834&idx=3&sn=9f5233b9da2385dbe4ce4527012d3c10&scene=21#wechat_redirect)  
  
  
  
[【安全圈】云商被黑连累FireworksAI！模型权重与密钥遭窃](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079821&idx=1&sn=4deb7108dc0f96ae2b0f2c738718402e&scene=21#wechat_redirect)  
  
  
  
[【安全圈】SSHole与GPUStack遭在野利用！CISA急列四漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079821&idx=2&sn=38d64841709704b9cc08a0e06921010e&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
