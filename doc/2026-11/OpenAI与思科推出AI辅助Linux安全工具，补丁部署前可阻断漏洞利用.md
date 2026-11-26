#  OpenAI与思科推出AI辅助Linux安全工具，补丁部署前可阻断漏洞利用  
 FreeBuf   2026-11-25 10:26  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1sRiaeFSpGwbDVyQXrFu12ZqAz3xruvN0q6a7aeYX6dCVUjI60bd0QGMnsa3n6GMYicNVlZTlygKZzoSz1xbO2dugGVxLhGtxmo/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1zKh1XyNYNMWibMDQibdv2T6H59ZLXmsmiaEcIXmcFpImS9afedD81s3oNkLV11zLMicZxwueZ1lePIrTam7wrvnVehoEMA50KLPM/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
OpenAI联合思科发布Shirudo  
  
  
OpenAI与思科联合推出开源Linux运行时安全工具Shirudo（日语意为“盾牌”），尝试让AI Agent在不影响业务连续性的情况下，为系统提供实时防护。  
  
  
OpenAI在X平台介绍，Shirudo能够让Agent自动编写动态eBPF防护规则，在系统层面拦截漏洞利用行为，为安全团队争取测试和部署补丁的时间。这也是OpenAI探索“自防御基础设施”的第一步，即让系统在持续运行过程中自主应对新出现的安全威胁。  
  
  
当前AI正大幅加速漏洞发现速度，已经能在支撑联网服务的基础软件中找到严重漏洞。  
  
  
Shirudo的定位是帮助防御人员测试临时防护措施（即“防护盾”），验证规则是否能拦截触发漏洞的恶意行为，同时放行合法请求。防护规则经运维人员审批后，无需停止服务就可以直接部署。  
  
  
这种设计可以避免一种常见的服务中断场景：由于漏洞利用行为和合法请求可能依赖同一项底层Linux操作，防御方为了阻断攻击路径不得不直接停服。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX395CCF1OXqqq3x4biaFTcfGDS0xz7Vvp4BQsgWy3Sc7Smu0Lh5YeUlFTTvyD6ibPXc7VCaehEFXWRbBTkES4LkD5YXLngiaTRzPk/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
02  
  
基于eBPF与Tetragon构建  
  
支持Agent自动生成防护规则  
  
  
为实现这一能力，Shirudo采用eBPF（扩展伯克利数据包过滤器）技术。该技术支持在内核中运行轻量级沙箱程序，无需修改系统代码，也不需要重启系统。  
  
  
思科旗下Isovalent对eBPF的解释是，它相当于Linux内核中的一套高效沙箱虚拟机。相关程序能够与内核交互，监测系统事件并作出响应，因此也被广泛用于系统监控、网络路由和安全策略执行。  
  
  
Shirudo基于Isovalent于2022年5月开源的eBPF安全平台Tetragon构建。与传统Tetragon相比，Shirudo更侧重Agent的自动化操作，能够借助eBPF为存在漏洞的代码路径注入临时防护措施。  
  
  
![Shirudo架构](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0UialdFTEl4T8SF8gCfv4NL1e9krDSFevqRksD0fFHWfBZ0vEIISQAWUzv3NpvuGyJqwQDAguhokhlkAmBFpaScnN3MCPibicf8o/640?wx_fmt=png "")  
  
  
Shirudo的核心组件是一个守护Agent，可通过eBPF防护盾拦截存在漏洞的代码路径，无需重启系统。安全团队在正式启用防护规则前，还可灵活测试规则对应用的实际影响。所有防护盾加载前，系统都会对其进行密码学签名与验证。  
  
  
整个过程由Agent自动推进：先扫描开源代码、分析厂商发布的漏洞通告，再定位受影响资产和攻击路径，生成对应的防护规则。随后，Agent会验证规则能否阻断漏洞利用，同时确保正常服务不受干扰。只有验证结果符合安全要求，防护盾才会正式启用。待官方补丁发布并完成部署后，临时防护盾便会自动撤除。  
  
  
Part  
03  
  
自动化防护全流程闭环  
  
  
整个防护流程分为以下几步：  
  
- Agent负责发现应用中的新漏洞，或处理外部来源的最新漏洞披露信息  
  
  
- 运维端部署的Shirudo会复现并验证漏洞利用方式，构建由eBPF或应用字节码组成的安全防护盾  
  
  
- 防护盾以经过签名的.bee包形式，通过兼容OCI的镜像仓库分发  
  
  
- 每台目标主机上的Shirudo组件会拉取对应.bee包，通过TPM验证每个包的签名，建立可信安全链路  
  
  
OpenAI还展示了一项验证结果：在对照实验中，Codex生成的防护盾成功拦截了针对运行中Elasticsearch服务的Java4Shell漏洞利用尝试，整个过程无需重启服务。  
  
  
OpenAI与思科希望借助这套机制，将AI的漏洞发现能力转化为机器速度的防御能力。思科安全首席技术官Thomas Graf和Isovalent首席开源官Liz Rice认为，随着AI模型发现、利用漏洞以及串联多个漏洞的能力不断增强，传统防御方式正面临更大压力，企业需要具备同等速度的响应能力。  
  
  
尤其是Linux内核、通用依赖库等基础组件，一旦出现严重漏洞，补丁往往需要数天、数周甚至数月才能覆盖大量部署环境。而如今，漏洞利用时间甚至已经早于补丁发布时间，单纯依靠等待官方修复再逐步升级，显然难以跟上攻击节奏。  
  
  
参考来源：  
  
OpenAI and Cisco Unveil AI-Assisted Linux Security Tool to Block Exploits Before Patching  
  
https://thehackernews.com/2026/11/openai-and-cisco-unveil-ai-assisted.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
