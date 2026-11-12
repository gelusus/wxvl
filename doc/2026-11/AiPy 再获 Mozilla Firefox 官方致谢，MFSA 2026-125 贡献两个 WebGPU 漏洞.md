#  AiPy 再获 Mozilla Firefox 官方致谢，MFSA 2026-125 贡献两个 WebGPU 漏洞  
 知道创宇   2026-11-12 11:45  
  
AI智赋未来 · 安全守护信息化  
  
近日，Mozilla 发布安全公告**MFSA 2026-125**  
（2026 年 11 月 10 日），修复了 Firefox 160 中的多个安全漏洞。知道创宇自研开源 AI Agent 产品 AiPy（章鱼哥）在本次公告中贡献了**2 个 CVE**  
，均位于**Graphics: WebGPU**  
组件。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mVOy0n0uJdhFKQYrhxhxgiaKub6eaJFxLcnnYeTJL51cdboC70cJq6gKHrtogm4fzC5jvyGYGauoEBTPU4SaLDWHNa0f9qicQEaCRib43AVCTE/640?wx_fmt=png&from=appmsg "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
  
图 1：Mozilla 官方安全公告 MFSA 2026-125（Firefox 160）  
  
这是 AiPy 在 Firefox 安全研究中获得的又一次官方认可。从 MFSA 2026-74 到本次 MFSA 2026-125，AiPy 在 Gecko 引擎中的研究已覆盖音视频、布局、图形渲染、WebGPU、WebRTC、配置文件备份等多个子系统，其中 WebGPU 已成为反复命中的重点区域。  
  
01 两项漏洞：聚焦 WebGPU 组件  
  
**CVE-2026-124915：**  
WebGPU 组件边界条件不正确  
  
组件：Graphics: WebGPU  
  
影响等级：**moderate**  
  
描述：WebGPU 图形渲染模块中的边界条件校验问题  
  
参考：Bug 2056172  
  
报告者：5up3rh3i  
  
**CVE-2026-124920：**  
WebGPU 组件未初始化内存  
  
组件：Graphics: WebGPU  
  
影响等级：**moderate**  
  
描述：WebGPU 图形渲染模块中的未初始化内存问题  
  
参考：Bug 2062097  
  
报告者：Samuel Erwin  
  
WebGPU 是新一代 Web 图形与计算标准，允许网页直接访问 GPU 能力，进行高性能渲染与通用计算。该模块涉及 GPU 缓冲区分配、绑定组管理、着色器资源绑定与跨进程数据传输，其中大量操作需要精确的边界校验——缓冲区大小、偏移量、访问范围等参数一旦校验不严，就可能导致越界访问或数据错位。该漏洞由 AiPy 独立发现并单独报告。  
  
未初始化内存问题源于内存分配后未正确清零，程序在后续使用中读取到了残留数据。在 WebGPU 场景下，这类问题可能出现在缓冲区映射、纹理上传或队列提交等环节，理论上可能导致敏感数据被意外暴露给网页内容。该问题同样由 AiPy 独立发现，与另一位研究者撞洞。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mVOy0n0uJdg800o1S1jCfuibDwwk45WUQXWlGiavjfbSA6pkOGudib6ZVLenDTQWS9icZBibrArPUjtbkibfgsA2lTE8qicCIebib6Fb2RZxjeo5GdY/640?wx_fmt=png&from=appmsg "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
![]( "")  
  
图 2：MFSA 2026-125 中两项 WebGPU 漏洞的官方条目  
  
02 WebGPU：AiPy 反复命中的重点区域  
  
回顾 AiPy 在 Firefox WebGPU 组件中的研究轨迹：  
  
**MFSA 2026-82：**  
CVE-2026-84130，WebGPU 信息泄露  
  
**MFSA 2026-97：**  
CVE-2026-100799、CVE-2026-100802，两个未初始化内存问题  
  
**MFSA 2026-125：**  
CVE-2026-124915（边界条件）、CVE-2026-124920（未初始化内存）  
  
累计下来，AiPy 已在 WebGPU 组件中贡献**5 个 CVE**  
，涉及信息泄露、未初始化内存、边界条件校验三类问题。  
  
这种在同一组件中的反复命中并非偶然。WebGPU 是浏览器中相对较新的技术栈，其规范仍在演进，实现层面需要同时处理 Web 标准的抽象语义与底层 GPU API 的具体约束，两者之间的映射关系复杂。加之 GPU 内存管理涉及显存分配、映射与同步，跨进程数据传输又引入了额外的生命周期管理问题，这些因素共同构成了较为集中的缺陷来源。  
  
03 从内存安全切入：Firefox 研究的技术纵深  
  
本次两个漏洞分别属于边界条件校验与未初始化内存问题，都是浏览器引擎中较难通过常规功能测试暴露的缺陷类型。  
  
边界条件问题通常出现在参数校验的临界值处理上——比如缓冲区大小的计算、偏移量的累加、访问范围的裁剪。这类缺陷需要针对性地构造边界输入才能触发，正常使用路径往往不会覆盖到。未初始化内存问题则涉及内存分配与初始化的时序，需要分析对象在创建、复用、释放各阶段的状态。  
  
在 Firefox 这样经过多年高强度审计的代码库中持续发现此类缺陷，说明 AiPy 的分析并不停留在代码模式匹配层面，而是能够深入理解组件的**内存模型与状态流转**  
。  
  
04 持续深耕，跨平台战绩持续积累  
  
从 Mozilla Firefox 的多轮致谢，到 F5 NGINX 的堆缓冲区溢出、OpenVPN 的四连击、PostgreSQL 18.6 的六项 CVE、Google Chrome 的高危漏洞、QEMU 的虚拟化缺陷、Apple macOS 的三次官方致谢，以及 Microsoft Windows 内核的 CVE——AiPy 已经横跨浏览器、应用交付、VPN、数据库、虚拟化平台和三大主流操作系统。  
  
每一次致谢背后，都是真实的技术产出。具备安全基因的 AI Agent，正在实战中持续证明自己的**价值**  
。  
  
知道创宇将继续推动**可信 AI 技术**  
与**安全能力**  
的深度融合，以自主可控的 AI 安全方案，为更多关键基础设施组件和底层系统安全提供保障，致力于成为各行业、全场景可信赖的**AI 安全伙伴**  
。  
  
相关链接  
  
https://www.mozilla.org/en-US/security/advisories/mfsa2026-125/（点击“阅读原文”跳转查看）  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mVOy0n0uJdgOPSY3Fa7Wj3d5pqVXwK5eDeiaAnkTOMpBTA6b6pNV6CPiayjSqZ8reThLpX2icBCI0UibClGMk6uibB1WGQPDZO4ibISUAgkwrAYIk/640?wx_fmt=png&from=appmsg "")  
  
  
