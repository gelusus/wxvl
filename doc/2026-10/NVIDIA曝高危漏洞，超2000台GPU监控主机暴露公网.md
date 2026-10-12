#  NVIDIA曝高危漏洞，超2000台GPU监控主机暴露公网  
 FreeBuf   2026-10-12 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3oFlzv5ULQwsykYfVDNtpurz38DiazTxwth8xicu4EmNOPaccpL0ibuoId3FKVdsednsaJ7ff6oiaOUV97HaQoYyILF6BN91pv8BU/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1umQZ3ojzZricrWyXfvVMUCp45XMjqEyicOAyxHkZZJX3lEhicVW6zX2zdjVpPsLfLGaibya81iaLBwRzfOGbIRoOia70rWmnn6zEBg/640?wx_fmt=png&from=appmsg "")  
  
  
NVIDIA曝出DCGM Exporter中的高危漏洞（CVE-2026-47483）。该漏洞可被未授权远程攻击者利用，耗尽服务器资源、导致GPU监控崩溃，进而中断AI工作负载。  
  
  
Part  
01  
  
漏洞未授权访问可耗尽资源  
  
  
DCGM Exporter是GPU服务器与Kubernetes集群的常用监控工具。它会采集GPU型号、温度、内存占用、功耗、运行错误、工作负载状态等信息，通过HTTP接口输出符合Prometheus格式要求的监控数据。  
  
  
每块GPU都对应唯一的UUID标识。如果监控接口暴露在公网，外部攻击者可以梳理高价值硬件资产、掌握GPU繁忙时段，还能获取相关软件与工作负载的详细信息。  
  
  
该漏洞存在于/debug/pprof路径下的Go性能分析接口中。这类接口原本用于帮助开发者排查CPU与内存占用问题，但存在漏洞的版本未对接口设置登录校验，也没有配置合理的请求频率限制，直接将接口暴露在外。  
  
  
攻击者只要短时间内发送大量性能分析请求，就会导致服务器内存与CPU占用失控，最终使DCGM Exporter运行变慢、停止响应甚至直接崩溃。  
  
  
Part  
02  
  
公网暴露超2000台主机  
  
  
NVIDIA将该漏洞的CVSS评分为8.2（满分10），漏洞类型归为CWE-770（无限制资源分配）。一旦DCGM Exporter崩溃，运维人员将无法实时查看GPU健康状态、运行错误、温度与资源占用情况。如果DCGM Exporter没有配置严格的容器或主机资源限制，这类资源挤占还会拖慢同一环境下运行的AI任务。研究人员仅在受控环境中测试了该资源耗尽攻击的效果，并未对发现的公网暴露系统发起攻击。  
  
  
![Shodan扫描发现2100台暴露的DCGM Exporter主机，通过未授权HTTP接口泄露超12000个GPU UUID（来源：lava.security）](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX03OsDAMwnEufdGH2DHwbqicoHs9LfDFO1UzZgNLRLVCbVSqrjQL3jBoA4p6ZLX1Picwe0BiaU560WsAzcAzMhMxNwLej08QdECCc/640?wx_fmt=jpeg "")  
  
  
Lava Security研究员Michael Katchinskiy在研究公网暴露的DCGM Exporter系统时发现了该漏洞。2026年3月至5月期间，研究团队开展了四次全网扫描，共发现约2100台主机无需认证即可对外提供监控指标数据。  
  
  
这些主机分布在近300家机构中，共泄露超过12000个唯一GPU标识符。其中约四分之一的DCGM主机同时暴露了存在风险的性能分析接口。可被识别的硬件包括H100、H200、B200、Blackwell Ultra B300、RTX 4090、RTX 5090等型号GPU。从地域分布看，美国暴露的GPU数量为5274块，占观测总量的44%。  
  
  
![暴露的AI基础设施同时使用NVIDIA H100/H200加速卡与消费级RTX 4090/5090 GPU（来源：lava.security）](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icBE3OpK1IX2CMJtLlVBnKLC8jQGyIljWBr3RDkLLIcwzJeRHiaAx0aeMc5uTExoBHE0RmfUBoWicFLtvn7AwvVk5wsLGgCcHrmKC2OQY4GsNw/640?wx_fmt=jpeg "")  
  
  
研究人员还在多家GPU云服务商的关联网络中发现了用户自行部署的暴露端点，涉及服务商包括Voltage Park、Lambda、Northern Data、DigitalOcean等。相关服务商收到漏洞通报后，已协助通知受影响客户。  
  
  
这一事件也说明，AI监控端口需要得到与其他管理服务同等的安全重视。Cyber Security News此前曾报道过可导致代码执行的NVIDIA GPU驱动漏洞，还发布过覆盖GPU、固件与管理层风险的AI数据中心安全检查清单。  
  
  
Part  
03  
  
官方推送4.8.2修复版本  
  
  
NVIDIA在安全公告中要求DCGM Exporter用户升级至4.8.2及以上版本。运维人员若非必要性能分析需求，应保持--enable-pprof参数为禁用状态。  
  
  
![近300家机构的超2000台主机存在GPU Exporter暴露问题（来源：lava.security）](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX0DLL2mbtkLdJaN7r1KXovj52sG4raFlgdMeMzj2PvZ4rPC1eVUSE0gnqcJgqdyZnA1ialenk8WicNNkibGOukgurhZfZHQ3yq8q0/640?wx_fmt=jpeg "")  
  
  
运维人员应将Exporter、Prometheus、Node Exporter服务绑定至私有地址或回环接口，仅允许受信任的监控系统访问。访问控制可通过防火墙规则、云安全组或Kubernetes NetworkPolicies实现。  
  
  
安全团队应全面排查面向公网开放的9400端口，禁止公网访问/debug/pprof路径，同时检查日志中是否存在重复或长时间运行的性能分析请求。  
  
  
如果出现内存突增、CPU飙升、Prometheus抓取失败或GPU告警断档等情况，可能意味着系统正被探测或已遭利用。团队需确认所有GPU节点都已部署修复后的容器版本，不能仅在镜像仓库或配置文件中更新版本。  
  
  
参考来源：  
  
NVIDIA DCGM Exporter Bug Exposes GPU Servers to Resource Exhaustion Attacks  
  
https://cybersecuritynews.com/nvidia-dcgm-exporter-bug-exposes/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
