#  F5 BIG-IP 堆栈缓冲区溢出漏洞可导致远程代码执行和拒绝服务攻击  
原创 ZM
                    ZM  暗镜   2026-11-25 22:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zdwoicOrrJb1FiayGGgUNhTAMib7Wu0nXUjqxI4XWmqbXwEqEmDkk0VZ8FHXZXpURjZMJ7sDTHBbplichoFLmAExGKp6WzD92AxMTooEh6QGPT0/640?wx_fmt=png&from=appmsg "")  
  
2026 年 11 月 18 日，F5 披露了 BIG-IP Access Policy Manager (APM) 中一个严重的基于堆栈的缓冲区溢出漏洞。该漏洞编号为 CVE-2026-123637，允许远程未经身份验证的攻击者终止流量管理微内核 (TMM)，从而在进程重新启动时中断流量。  
受影响的配置需要 APM 配置和具有按请求策略的虚拟服务器，该策略包括客户端信息代理和端点安全代理。  
F5 表示，该内存损坏漏洞可能还会导致远程代码执行 (RCE)。该漏洞在 CVSS v3.1 中的评级为 9.8，在 CVSS v4.0 中的评级为 9.3。  
  
该漏洞存在于处理应用程序流量的数据平面，而非控制平面。限制管理接口访问并不能消除受影响虚拟服务器上的风险。策略前提条件将此问题与编号为 CVE-2026-94127 的独立 BIG-IP APM 远程代码执行漏洞区分开来。F5将该缺陷归类为 CWE-121，即对堆栈分配缓冲区进行越界写入，并为其分配了内部跟踪 ID 2567577。  
  
TMM崩溃是造成拒绝服务攻击的原因，但仅凭重启本身并不能证明攻击者执行了代码。F5并未披露触发该攻击的具体请求。另一个BIG-IP HTTP/2 拒绝服务漏洞也影响了数据平面，但涉及不同的流量处理组件。F5 将受影响的 BIG-IP APM 版本列为 21.1.0、17.5.0 至 17.5.1 以及 17.1.0 至 17.1.3。其他 BIG-IP 模块，如 BIG-IP CNF、BIG-IP Next for Kubernetes、BIG-IQ Centralized Management、F5 Insight、F5OS 和 NGINX，则不受影响。  
  
F5 未对已停止技术支持的版本进行评估。针对 BIG-IP APM 的 PoisonedRefresh 攻击活动涉及 CVE-2025-53521 漏洞，但并未证实该漏洞已被实际利用。F5 的热修复指南 K000164239将每个工程版本映射到当前已安装的基础版本或工程热修复版本。每个 17.x 版本系列列出的两个版本不可互换；管理员在选择软件包之前应验证完整的已安装版本字符串。F5 没有提供任何解决方法，并建议客户从 MyF5 Downloads 获取相应的软件包。管理员应清点受影响策略配置的虚拟服务器，并优先处理那些可以从不受信任的网络访问的服务器。  
  
