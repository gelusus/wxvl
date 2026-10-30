#  Claude研究模型生成Linux内核漏洞利用链，实验环境可远程获取Root权限  
 看雪学苑   2026-10-30 09:59  
  
Anthropic前沿红队团队于10月27日公开一项安全研究：一款尚未对外发布的Claude研究模型，自主生成了针对Linux内核高危漏洞 CVE-2026-98432 的可用EXP。  
  
  
在实验室环境中，该利用链可以实现远程攻击，无需登录、无需用户交互，直接在目标服务器上以root权限执行系统命令。  
  
  
研究明确标注：本次是受控测试，不是真实互联网上发生的入侵事件。  
  
  
**漏洞原理与攻击流程**  
  
  
漏洞根源是Linux内核网络协议栈内的竞争条件，最终引发释放后重用（Use-After-Free）内存缺陷。  
  
  
攻击目标为开启多路径TCP（MPTCP）监听服务的Linux服务器，整套利用分为两步：  
  
1. 利用竞争条件诱导内核返回额外数据，从中读取内核指针，绕过KASLR地址随机化防护，获取内核内存布局；  
  
2. 借助同一个漏洞覆盖回调函数指针，控制内核执行预先布置的载荷，最终拿到root权限。  
  
  
竞争条件的触发存在概率性。如果竞争失败，只会造成TCP握手失败，服务器不会崩溃，攻击者可以反复重试。  
  
  
本次POC测试中，泄露内核指针耗时约48分钟，后续执行root命令还需要额外5分钟。  
  
同一份研究里，还包含Chrome、Safari、Android、Windows平台的同类AI辅助攻击链验证。  
  
  
**一处需要留意的信息差异**  
  
  
该漏洞CVSS评分9.8，属于严重级别。  
  
  
Ubuntu漏洞台账在10月15日录入该CVE信息、10月28日更新记录。台账描述漏洞为IPv4 TCP处理缺陷：  
  
`tcp_v4_syn_recv_sock()`直接传递IP选项至子套接字，没有做拷贝；当收到重复SYN包时，其他CPU线程可能提前释放内存，触发`ip_options_build()`里的UAF问题。  
  
  
该原始漏洞记录没有提到MPTCP。  
  
  
MPTCP和这条攻击链路的关联，仅来自Anthropic的研究描述。  
  
  
包括Aviatrix在内的安全厂商都提示：在其他团队完整复现整条攻击链之前，该攻击路径仅作为厂商单方面研究结论参考。IPv6协议栈不受该漏洞影响。  
  
  
**📌 补丁与发行版修复进度**  
  
  
  
上游内核主线已经完成漏洞修复，补丁向后移植到所有仍在维护的稳定内核分支：  
  
5.10.272、5.15.223、6.1.190、6.6.159、6.12.113、6.18.56  
  
版本号请以 kernel.org 官方为准  
  
  
各发行版推送节奏并不统一：  
  
✅ Oracle Linux 已发布修复包  
- Ubuntu 22.04 / 24.04 / 26.04 内核标记为受影响，补丁仍在制作；20.04、18.04旧版同样存在风险；  
  
- AWS、Azure、GCP大量云内核处于受影响状态  
  
公网直接暴露的业务服务器，应优先安排补丁升级。  
  
  
**🧾 临时缓解方案（无法立刻升级内核时）**  
  
  
- 内核补丁必须重启服务器才能生效，更新后务必重启；  
  
- 评估业务依赖，关闭服务器上不必要的MPTCP监听服务；  
  
- 通过防火墙策略，缩小公网服务暴露面；  
  
- 持续监控监听端口上异常TCP握手重置、频繁握手失败日志。  
  
  
**📈 背后趋势：大模型改变漏洞攻防节奏**  
  
  
这项研究的重点不在于单个漏洞本身，而在于能力变化：**大模型已经可以在少量人工引导下，独立寻找漏洞、组装成完整可利用的攻击链。这类工作过去，只有顶尖的专业攻防团队可以完成。**  
  
  
AI工具是双向的：既可以被用来构造EXP，也能支撑防御侧工作。  
  
  
Anthropic配套推出面向合规安全从业人员的 Cyber Verification Program；  
  
  
Linux社区也有相关案例，Linus Torvalds曾提到 Claude Code 辅助编写过Linux内核补丁，同一类工具同时服务于漏洞挖掘与漏洞修复。  
  
  
安全团队也需要审视自身环境。此前Claude Cowork沙箱也曾爆出漏洞，攻击者可突破沙箱限制拿到Linux沙箱内root权限。  
  
  
AI降低了漏洞武器化门槛，漏洞发现到可用EXP成型的时间窗口持续缩短，对持续扫描、快速响应能力提出更高要求。  
  
  
资讯来源：Cybersecuritynews  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K1UEhWUicWztmZOBO8coXFRUWS5FkdVLO4OKxXp0lzTr1WTqVu4yOYNkeqJiasicC2vbS3yiaJy3tN2PwAPrFicfkriaJlhrBdKVfXbk/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K38HUCib273VicrjS91rhXqIvpWP4A9n0HcAibRAbI5rBNVbNm1jSDedxh3SHz1Z2e2s6zxJvD1JoibONiaiaYdiaLf6qSInB1gwmEWxI/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0GkoBGv8FnNM2LjLAibIgfAZxNRtKMP75NmNa8mMzoLxu4LVfLpMUQ3DlWjD0M1L2iap6Z6Sy7ZA2OmHkqxiauWRza5qY5SiaOrXQ/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K2kOTfl9fpY72K4E0D6hCetNADlCibCWxypSzYn7Uar8OBWXMOsUbmjk09lcibJBEUOia0LyCmrTUBwQopsGsn8I7pFtK1K71amjc/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
