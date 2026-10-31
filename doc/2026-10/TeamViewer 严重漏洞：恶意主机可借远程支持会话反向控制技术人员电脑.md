#  TeamViewer 严重漏洞：恶意主机可借远程支持会话反向控制技术人员电脑  
 网安百色   2026-10-31 11:13  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WibvcdjxgJnt39ppiasltc4k2ftB2swvUuGPTyl7CUMwUHXa3WVaAtFbibAhvbDBibzB0b0H8m0Xbb8uPKAAws2XXOqIk56uBLbXybrzFJDafpk/640?wx_fmt=png&from=appmsg "")  
  
最新披露的 TeamViewer 漏洞可能使攻击者将常规的远程技术支持会话“反客为主”，直接威胁提供帮助的技术人员。该漏洞编号为 CVE-2026-111585，允许恶意的受控远程主机导致查看端（Viewer）应用程序崩溃，并有可能在技术人员的机器上执行任意代码。TeamViewer 将其严重等级评定为“高危（High）”，CVSS 评分为 8.8。  
  
Remedio 安全研究员 Omri Dar 于 10 月 29 日公布了该技术细节（官方修复补丁已于 10 月 27 日发布）。根据 Remedio 的说明，该攻击的前提是受害者（技术人员）需与攻击者控制或已沦陷的主机建立远程会话。一旦连接建立，攻击者无需受害者进行任何额外点击，也无需窃取密码即可触发漏洞。  
### TeamViewer 安全漏洞详情  
  
TeamViewer 的会话架构分为共享屏幕的“主机端（Host）”和显示屏幕的“查看端（Viewer）”。在此次攻击中，主机端通过原本用于技术支持的同一条连接，向查看端发送精心构造的屏幕数据。这种攻击模式颠覆了传统的风险预期，使原本处于信任地位的技术人员计算机反而陷入危险。  
  
攻击过程中，主机端会发送包含屏幕宽度、高度和色彩深度的 DisplayParams  
 消息。查看端会根据这些值，在一个名为 CBitmapBuffer  
 的对象中为即将传入的像素数据预留内存。恶意主机可以通过拒绝使用 AV1 和 H.264 视频编码选项，强制查看端回退到使用位图（bitmap）更新的处理路径。  
  
此次发现进一步扩展了此前关于 TeamViewer 远程桌面漏洞的报告，并精准定位了该攻击背后的计算逻辑缺陷。缓冲区大小的计算使用了 32 位算术运算。当屏幕尺寸参数过大时，该计算会发生整数溢出回绕（wrap around），导致实际分配的内存远小于查看端预期的容量。  
  
在 Remedio 的测试用例中，一个分辨率为 32,768 x 32,768 像素、每像素 32 位的屏幕，在经过内存对齐（padding）后，理论上需要超过 42.9 亿字节的内存。然而，由于整数溢出，计算结果仅分配了约 4 MB 的内存。但查看端程序仍错误地认为该小缓冲区足以容纳所声明的屏幕数据。  
  
随后，解码器在验证传入的图像矩形（image rectangles）时，仅比对了声明的屏幕尺寸，而未校验实际的内存分配大小。因此，一个超大的矩形数据可以通过验证，但其像素数据在写入时会溢出缓冲区边界。这些越界写入操作会破坏相邻的堆内存（heap memory），不仅会导致可复现的程序崩溃，还为后续的代码执行铺平了道路。  
  
Remedio 报告称，其在 Windows 环境下成功执行了指定代码，并绕过了控制流防护（CFG, Control Flow Guard）和数据执行保护（DEP, Data Execution Prevention）。然而，需要指出的是，其概念验证（PoC）演示使用了主机端的 Hook 技术来注入破坏性字节，以此替代了真实的网络载荷（network payload）和堆内存布局准备（heap preparation）。在评估该攻击链的完整性和实际野外利用难度时，这一细节至关重要。  
### 受影响版本与修复方案  
  
TeamViewer 的安全公告 TV-2026-1011 指出，受影响的版本包括 Windows、Linux 和 macOS 平台上版本低于 15.83 的完整客户端（Full Client），以及 Windows 便携版（Portable）和 Android 远程控制端。代码执行的最终效果取决于具体平台，且通常仅以查看器进程的权限运行，这并不意味着攻击者能自动获取 SYSTEM 或 root 级别的最高权限。  
  
管理员应立即安装 15.83 版本或最新的可用发行版。对于遗留部署环境，需应用公告中列出的对应修复版维护构建（maintenance build）。鉴于此次更新是 TeamViewer 近期一系列安全补丁的延续，运维人员必须全面核查技术支持工作站、便携版副本以及可能脱离常规自动更新流程的老旧安装实例。  
  
TeamViewer 在发布公告时表示，目前尚未发现该漏洞在野外被主动利用（in-the-wild exploitation）的迹象。Remedio 已于 8 月 14 日负责任地披露了此问题。对于技术支持团队而言，当务之急是在连接到不受信任的计算机之前，务必先完成查看端软件的更新，而不能仅仅依赖会话建立时的授权确认机制。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
