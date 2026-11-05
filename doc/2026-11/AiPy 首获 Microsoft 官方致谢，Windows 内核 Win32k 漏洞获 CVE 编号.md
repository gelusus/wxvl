#  AiPy 首获 Microsoft 官方致谢，Windows 内核 Win32k 漏洞获 CVE 编号  
 知道创宇   2026-11-05 08:53  
  
   
  
近日，Microsoft 发布 2026 年 10 月安全更新，知道创宇自研开源 AI Agent 产品 AiPy（章鱼哥）在**Windows Win32 Kernel  Subsystem**  
中发现并报告了一个信息泄露漏洞，获 Microsoft 官方致谢并分配 CVE 编号 **CVE-2026-70295**  
。  
  
这是 AiPy 首次在 Microsoft Windows 生态中获得官方 CVE 认可。此前 AiPy 的研究成果已覆盖 Apple macOS、Mozilla Firefox、Google Chrome、PostgreSQL、QEMU、OpenVPN、FreeBSD 等多个国际主流平台，本次进入 Windows 内核领域，意味着其安全研究版图已覆盖全球三大主流操作系统。  
  
  
01 CVE-2026-70295：Win32k 内核子系统越界读取  
  
根据 Microsoft 安全更新指南（MSRC）的公开信息：  
  
**CVE-2026-70295：Win32k 信息泄露漏洞**  
  
组件：Windows Win32 Kernel Subsystem  
  
漏洞类型：CWE-125 越界读取（Out-of-bounds Read）  
  
严重性评级：  
Important 重要  
  
CVSS 3.1 评分：**5.5**  
  
CVSS 向量：AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N  
  
影响：本地授权攻击者可借此泄露信息  
  
发布日期：2026 年 10 月 13 日  
  
描述：Windows Win32 内核子系统中的越界读取问题，允许已授权的攻击者在本地泄露信息。  
  
Win32k 是 Windows 内核中最核心的子系统之一，负责图形设备接口（GDI）、窗口管理、用户输入处理等基础功能。该子系统长期运行于内核态，直接处理来自用户态程序的系统调用请求，历来是 Windows 提权与信息泄露漏洞的**高发区域**  
。  
  
从 CVSS 向量可以看出该漏洞的特征：攻击向量为本地（AV:L），需要低权限（PR:L），不需要用户交互（UI:N），机密性影响为高（C:H），但完整性与可用性不受影响（I:N/A:N）。这意味着攻击者需要先在本机获得一个低权限账户，之后便可利用该缺陷读取内核内存中的敏感数据——这类信息泄露往往是为后续提权攻击做铺垫，攻击者可借此绕过内核地址空间布局随机化（**KASLR**  
）等防护机制。  
  
Microsoft 将该漏洞的利用可能性评估为"**Exploitation Less Likely**  
"（较不可能被利用），且该漏洞未被公开披露、未被实际利用。  
  
  
02 从用户态到内核态：Windows 研究的门槛  
  
Windows 作为闭源商业操作系统，其安全研究难度体现在多个层面。  
  
第一：代码不可见  
  
研究者无法像分析 Linux 内核那样直接阅读源码，只能通过逆向工程还原 Win32k 的函数逻辑与数据结构。Win32k 子系统经过多年演进，代码规模庞大且包含大量历史遗留路径，逆向分析的复杂度很高。  
  
第二：内核态调试受限  
  
Windows 内核调试需要双机调试环境或虚拟机配置，且受 PatchGuard（内核补丁保护）等机制约束，动态分析的灵活度远低于用户态程序。  
  
第三：漏洞利用缓解机制完善  
  
Windows 在内核层面部署了 KASLR、SMEP、SMAP、CFG、ACG 等多层防护，即便发现了内存缺陷，也需要绕过这些机制才能形成有效攻击链。  
  
在这样的环境中发现一个可被 Microsoft 确认并分配 CVE 的内核缺陷，说明 AiPy 的分析能力已经能够穿透闭源系统的重重屏障，深入到操作系统**最核心的代码层**  
。  
  
  
03 三大操作系统全覆盖  
  
本次 Windows 漏洞的确认，补齐了 AiPy 在主流操作系统领域的最后一块拼图：  
  
**Apple macOS：**  
三次官方安全公告致谢，累计 17 个 CVE（macOS Tahoe、Golden Gate 27、Tahoe 26.7.2）  
  
**Linux 生态：**  
FreeBSD 源码提交致谢，QEMU 虚拟化漏洞 CVE-2026-66900  
  
**Microsoft Windows：**  
CVE-2026-70295（Win32k 内核子系统）  
  
从开源到闭源，从 Unix-like 到 Windows NT 架构，从用户态服务到内核子系统——AiPy 的安全研究已经完成了对主流操作系统内核层的**全面覆盖**  
。  
  
  
04 持续深耕，跨平台战绩持续积累  
  
从 Mozilla Firefox 的多轮致谢，到 F5 NGINX 的堆缓冲区溢出、OpenVPN 的四连击、PostgreSQL 18.6 的六项 CVE、Google Chrome 的高危漏洞、QEMU 的虚拟化缺陷、Apple macOS 的三次官方致谢，再到如今 Microsoft Windows 内核的 CVE——AiPy 已经横跨浏览器、应用交付、VPN、数据库、虚拟化平台和三大主流操作系统。  
  
每一次致谢背后，都是真实的技术产出。具备安全基因的 AI Agent，正在实战中持续证明自  
己的**价值**  
。  
  
知道创宇将继续推动**可信 AI 技术**  
与**安全能力**  
的深度融合，以自主可控的 AI 安全方案，为更多关键基础设施组件和底层系统安全提供保障，致力于成为各行业、全场景可信赖的**AI 安全伙伴**  
。  
  
  
参考资料  
  
https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-70295（点击“阅读原文”可以查看）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mVOy0n0uJdg3TRmeNIo4d6CKsocTpbAWawZCuTsQyVcHUTJicsfVTSjP0jyI1oyngrVdF9CASjq9ptZfMicpPKCywTfDaMEUlnax3bjvHbhI0/640?wx_fmt=png&from=appmsg "")  
![]( "")  
![]( "")  
![]( "")  
  
  
