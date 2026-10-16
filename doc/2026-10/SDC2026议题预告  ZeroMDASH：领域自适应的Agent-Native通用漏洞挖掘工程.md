#  SDC2026议题预告 | ZeroMDASH：领域自适应的Agent-Native通用漏洞挖掘工程  
SDC2026
                    SDC2026  看雪学苑   2026-10-16 10:05  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1Ys9HJ11Nib02RNjPsibvuDOfW8BibTe6S0XGic1OypMEQFkdC1Yx7PaNQ8QlnNF7appUfkxYRhGBSTxeVhLDfZQerlloJicMSLMibM/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0dWt83pTKdCkeOdBBsWiaX72YFPE1zdOpwibFFiagTHjJOYDrTxJ3vs7iaS9qGA0ESoMhINgxy0Pe0IicwGEfibjG4InO5muaJs2hm8/640?wx_fmt=gif&from=appmsg "")  
  
  
**ZeroMDASH：领域自适应的Agent-Native通用漏洞挖掘工程**  
  
  
  
AI Agent已经能够阅读代码、调用工具并自主执行安全分析，但当目标从单一函数扩展到 Linux 内核、虚拟化系统、网络协议栈等复杂软件时，为什么它仍会遗漏真正重要的漏洞？问题往往不只是上下文长度不足，而是模型缺少对目标软件特有安全机制的理解：它不知道哪些对象属于核心资产，哪些权限关系和状态转换必须满足特定的安全约束，也难以判断散落在不同模块中的局部缺陷能否组合成一条真实可利用的攻击链。  
  
  
本议题将介绍 ZeroMDASH，一套面向源码漏洞挖掘的领域自适应 Agent-Native 分析框架。  
它不依赖预设的漏洞类型清单，而是从目标代码出发，自主识别其中的关键对象、权限关系、信任边界、状态转换及其安全约束，并将潜在攻击面转化为可追踪的分析范围和漏洞假设。随后，由调查 Agent 与审查 Agent 围绕源码证据、反例和未解决条件展开多轮对抗式验证。系统还会关联分析分散在不同模块中的风险线索，识别单点分析难以发现的复杂漏洞与攻击路径。  
  
  
当每一个局部行为看起来都“不是漏洞”时，它们组合起来是否可能导致权限提升、信息泄露？当 Agent 声称“没有发现问题”时，我们又如何判断它确实完成了必要的安全分析，而不是因为分析范围不足而提前停止？  
本议题将围绕这些问题，分享 ZeroMDASH 在目标领域安全建模、假设集推理、对抗式证据复核以及跨原语攻击链验证方面的设计与实践，并讨论 AI 漏洞挖掘如何从生成潜在issue，进一步发展为构建可追踪和复核的安全证据。  
  
  
**演讲嘉宾：C0ss4ck**  
  
  
  
安全研究员，https://cossack9989.github.io/about  
  
  
长期从事漏洞挖掘与自动化安全分析，研究方向涵盖物联网（IoT）、音视频协议、模糊测试、二进制分析、Linux 内核及虚拟化安全。目前主要探索 AI for Security，致力于研究Agent-Native的自动化漏洞挖掘方法。曾在 Linux、NVIDIA、ASUS、PJSIP、FreeSWITCH 等项目或产品中发现多项安全问题，并在多个安全论坛（沙龙）上发表研究成果。  
  
  
**演讲嘉宾：Swing**  
  
  
  
安全研究员，https://swing.rs  
  
  
聚焦于漏洞挖掘与利用，研究兴趣涵盖广泛领域，包括但不限于物联网（IoT）、Linux内核、虚拟化以及网络安全应用。目前他正在探索 AI 与安全的交叉领域，包括 AI for Security 和 Security for AI 两个方向。曾多次参加国内外 PWN 赛事并获奖。同时他也是 FlappyPig和 r3kapig 战队的 CTF 选手，并曾在多个安全论坛（沙龙）上发表研究成果。  
  
  
**听众收获**  
  
  
  
1. 剖析 AI Agent 挖掘 Linux 内核、协议栈等大型软件时漏报漏洞的底层根源。  
  
  
2. 学习 ZeroMDASH 多 Agent 对抗验证方案，实现跨模块复杂攻击链的自动挖掘。  
  
  
3. 获取 AI 漏洞挖掘结果的可信校验方法，区分真实安全结论与 Agent 提前终止的误判。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K3DPHKKpVm7PaJmFUGAeOVvTk693f9p2C6dZX7GZX5P7s6f5sPJyicqXE7gAibFTfVic2zdaqAtGicMGZg2xdA7S66mB37ccP4IcTk/640?wx_fmt=jpeg&from=appmsg "")  
  
报名参会  
  
来SDC现场解锁更多议题细节  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K2RIlOibryUb8dYDHhn3QNoIypqyCNTXo9vdCpTom4OKV7ia6Hse5nicGMtoOVwicibZ9jWeZVTqJ2icOkbV8BQMRl0A7N7H6bibe8H74/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K36UQR1aetPSTfIe1mJcJVFPNibibkLjD4Te4riaXTyWmlzNSNvsOS2QA6VgGNZjClPARbaB4ZPWapAvonqPGIoODVuib4huicrDuYM/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0MkxUcniblKFzWIH9dF5Dm69gdjlfO1BMGPrYNetrJ4bg4dnj1Sf5RNx2ybrdV49icd6iaAib36uejAPONgxAx1EWUezLUU9jnibOE/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K2at0p4R2OurQicYSiahRW1IakianEckiaPIZOsszxf5XGpDpiabI7sSYmot9x6XQoqTiavnykRf11ougBapVNia3QMb9Gqqqsw46zEkQ/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0IPe0vB6GcZRGJHBAoibocg1e6Takfuj08jkrHlCTqahM1UZOTGaAM4vr5iazlmYbZgbQ0Kdy8sA7navKbEDsdD2ec9AntM8gkM/640?wx_fmt=gif&from=appmsg "")  
  
点击阅读原文，报名参会  
  
  
