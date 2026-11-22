#  Windows 11 锁屏绕过零日漏洞  
原创 Blake Chen
                    Blake Chen  黑白之道   2026-11-22 00:59  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NtNKsAZhFT0bqJdhx3OVMY6ubuvlicTF3kBKwLf9hicmwOmcmPShtUgPYwF1zb1dI16TBchiajqy58eNF4ctl4QvPxwrgc7wzVrE/640?from=appmsg "")  
> **导语**  
：一台处于锁屏状态、磁盘已被 BitLocker 加密的 Windows 11 电脑，攻击者只用手边的键盘就能跑起自己的程序。2026 年 11 月 19 日，前微软研究员公开了这一名为 FastUnlock 的锁屏绕过零日漏洞，微软至今未发布补丁。  
  
  
![Windows 11 锁屏绕过零日漏洞 FastUnlock](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NffcSW9AjxrJUOssq1bkb4rXyZTWfAQoD7d4Ffd1wctDW1tPeKic99kQAJWiaofbRqJa3ibGYaT1egm8aBJncvLpuibyRsj8j6tP4/640?from=appmsg "Windows 11 锁屏绕过零日漏洞 FastUnlock")  
## 事件概述  
  
2026 年 11 月 19 日，以 MSNightmare、Nightmare Eclipse（“噩梦日蚀”）和 Chaotic Eclipse（“混沌日蚀”）等网名活跃的安全研究员阿卜杜勒哈米德·纳西里，在 GitHub 上公开了一份名为 FastUnlock 的 Windows 零日漏洞披露文档，并同步放出演示视频。文档给出的判断相当直接：Windows 11 内置的屏幕阅读器 Narrator（讲述人）在重构界面时，遗漏了对“锁屏用户上下文”的启动校验，导致攻击者可以在锁屏状态下把讲述人的新界面拉起来，并借此取得代码执行权限。  
  
  
  
漏洞的实用价值在于门槛极低。纳西里称自己从发现到完成一台完整 BitLocker 加密机器的突破只花了五分钟，并且在他测试过的每一台机器上都稳定复现。安全厂商 ThreatLocker（威胁锁）在同一天完成了独立验证，确认该方法确实生效。目前尚未发现该漏洞被用于真实攻击的证据，微软也没有发布补丁或受影响版本清单。  
  
值得一提的是，这已经是纳西里自今年 4 月以来公开的又一个 Windows 零日漏洞。据其本人与媒体说法，他与微软存在围绕离职纠纷的长期分歧，因此不再走常规协调流程直接公开；此前披露的同类漏洞包括 ZDI-21-1053（趋势微零日计划编号）。  
  
从防守视角看，这条链路的完整走向如下。  
  
![FastUnlock 锁屏绕过攻击链](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NUYvOqd4biabOW8tSLZVNDUly3WzjvphlHmtXjMic02qJFh8ElleBCpYQtLX34E4aL1ibmC0hUbYZaXZDk8D3NEmCBuwg73CwOGQ/640?from=appmsg "FastUnlock 锁屏绕过攻击链")  
## 漏洞原理与触发条件  
  
整条链路的关键不在于某个越权内核漏洞，而在于一处**上下文校验缺失**  
。微软给讲述人换新界面时，保留了旧代码中“系统身份下禁止弹出窗口”的保护，却没检查当前是否运行在锁屏账户上下文，于是新界面被允许在锁屏状态启动。  
  
具体触发路径并不复杂：攻击者在锁屏界面点击“忘记 PIN/密码”，进入微软账户恢复流程，此时按下 Win+U 呼出辅助功能设置并启用讲述人；讲述人会立即聚焦到一个后台不可见窗口，攻击者按住 Caps 键配合方向键、Enter 键在后台“盲操作”导航，进入系统设置并修改“自动播放”相关选项；随后插入携带恶意程序的 U 盘，系统在自动播放策略下拉起资源管理器，程序即在锁屏账户 WsiAccount 上下文完成执行。  
  
从利用门槛看，有三个硬性前提值得蓝队记下来：  
- **必须物理接触设备**  
，属于典型的物理攻击面，不是远程可利用漏洞；  
  
- **设备上需存在可恢复的微软在线账户**  
，否则“忘记 PIN”流程走不通；  
  
- **最终权限有限**  
，代码以锁屏账户 WsiAccount 运行，并非管理员，但足以读取该账户可见的数据、投放程序并为后续攻击铺路。  
  
研究者本人表示尚未验证 Windows 10 是否同样受影响。  
## 影响面与风险边界  
  
这里有一个最容易被误读的点：FastUnlock **并不破解 BitLocker 加密，也不恢复任何密钥**  
。  
  
在常见的 TPM（可信平台模块）配置下，开机瞬间磁盘已由 TPM 自动解锁，等到锁屏界面出现时，磁盘其实已处于可读明文状态——微软官方文档也描述了这一“无需输入 PIN 即可解锁”的模式。FastUnlock 是在这个已经解锁的阶段动手，因此它的破坏力不在于“破解加密”，而在于**推翻了“设备已上锁即数据安全”的物理安全假设**  
。  
  
真正需要担心的场景是设备离手：共享工位的前台机、会议室笔记本、存放研发机密的移动工作站、已经关机但存在休眠问题的终端。官方缓解建议也指向同一逻辑——为 BitLocker 配置启动 PIN，离开设备时关机或休眠而非仅锁屏，因为启动 PIN 保护的是“尚未启动的机器”，锁屏本身挡不住这条链路。睡眠状态下的设备同样处于已解锁状态，风险等同。  
  
从合规角度看，这类问题直接冲击静态数据保护与设备锁定两类控制项。对照 NIST SP 800-53，AC-11（设备锁定）与 SC-28（静态信息保护）已不再满足；映射 MITRE ATT&CK，T1200（硬件添加，对应 U 盘载荷）、T1078（有效账户，指以 WsiAccount 身份获得的执行上下文）与 T1562.001（削弱防御，指被改写的自动播放设置）构成了一条可检测的行为链。  
## 蓝队检测与缓解建议  
  
补丁缺位期，能做的就是把物理面收回来、把执行链切断、把痕迹留下来。  
  
![蓝队检测与缓解四道防线](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M5uhwa87nfXASKiciazIthtMq8OeXg748up54yIzTica9o40vlmKLUCa4llaichgRlbcCIGmrKHskjQFNPic4AHIYux0f1BicWbRGLY/640?from=appmsg "蓝队检测与缓解四道防线")  
  
**事前基线。**  
 首要动作是通过组策略禁用可移动介质自动运行与自动打开，切断 U 盘载荷的落地路径；同时在高价值终端上为 BitLocker 配置预启动 PIN，让设备在关机或重启状态下也必须输入口令。物理侧的手段依然有效：设备柜、锁孔、摄像头覆盖、前台机固定摆放，这些是零日无补丁场景下最可靠的补偿控制。软件侧可评估是否通过策略限制讲述人等辅助功能工具的启动。  
  
**事中检测。**  
 建议在 EDR 与 SIEM 中重点关注三类信号：一是进程创建事件（Windows 事件 4688、Sysmon 事件 ID 1）中账户为锁屏账户 WsiAccount 或类似特殊主体的记录；二是父进程为 explorer.exe、命令行或镜像路径指向可移动设备路径的执行行为；三是锁屏状态下讲述人进程或设置窗口被拉起的痕迹，以及自动播放相关注册表项（禁用策略与 AutoplayDevices 键值）发生的变更。此外，把 USB 设备接入事件纳入基线比对，可与设备台账关联，识别“未登记介质在非办公时段的接入”。  
  
**事后加固。**  
 结合 MITRE ATT&CK 做一次回溯，确认历史上是否出现过锁屏账户上下文下的进程创建；复核 BitLocker 策略清单，确保启动 PIN 覆盖到全部笔记本与外设机；把“未授权介质 + 锁屏上下文执行”的组合纳入检测规则。  
  
**持续跟踪。**  
 微软尚未发布官方公告与受影响版本清单，建议把该漏洞纳入补丁跟踪台账，一旦官方发布说明立即验证覆盖率。补丁到位前，不要把“已上锁”当作数据安全的证明。  
## PoC 与原文获取  
  
按研究者的公开披露，仓库中提供的是披露文档与演示素材，可用于安全研究和内部评估：  
- 披露仓库（MIT 许可）：https://github.com/MSNightmare/FastUnlock  
  
- 仓库内容打包下载（ZIP）：https://github.com/MSNightmare/FastUnlock/archive/refs/heads/main.zip  
  
- 演示视频（研究者本人操作录屏）：https://www.youtube.com/watch?v=6BKsdUtc6bg  
  
- 原始推文（漏洞要点与截图）：https://x.com/0x0SojalSec/status/2123964360762798137  
  
截至发稿，仓库内并未附带可直接运行的利用程序，验证需要真实复现流程，建议仅在自有、已授权的测试设备上进行，切勿对生产终端或他人设备验证。  
## 结语  
  
FastUnlock 提醒我们一件事：安全边界不只存在于网络里，也存在于那一块物理屏幕之前。零日无补丁的窗口期里，设备管控、启动 PIN、自动播放策略和进程审计这四件事，是当下最实在的防线。  
  
**参考资料**  
：  
- FastUnlock: Windows Narrator LockScreen Bypass 0day Vulnerability - GitHub 仓库，研究者披露文档，发表于 2026-11-19  
  
- New Unpatched Windows 0-Day FastUnlock Lets Someone Holding a Locked PC Run Programs on It - The Hacker News，发表于 2026-11-20  
  
- Chaotic Eclipse Releases FastUnlock, a New Windows Zero-Day Vulnerability - Security Affairs，发表于 2026-11-20  
  
- FastUnlock 演示视频 - YouTube，研究者本人录屏  
  
**来源/编译**  
：本文编译自《FastUnlock: Windows Narrator LockScreen Bypass 0day Vulnerability》，了解更多可访问原文。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6P0EAOv7HVXpOKiaFQHuicyeOGicELD443VBGHicnSv4Ynp3oeVu1GqsPDwArkRJT1E4RLhpccaTZmkZrHj2sl1VnAQYUWVbnakENI/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PZIBGyFz5nY5ibPppSaib160Rlg4icCI1pc6rn0hzCZXVFzuHNojj3TkcYiajLAkuFT0d5g693ibS3iaoPQLhOz7cWDOt3xCiaHrryrY/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PQjTmARWHN1UrUwknCLKuLG44iafL1tpUrE3HMeYGTDEUDpg3yBCByfxicwHeLfMicq5WyMwDlfIvkjjtJBSUra7qP8C7zzqic9nk/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NR8LWWoM0OuFzgdxBLuDgxyuBkElIaq6ibNFxLYTyuiaOWf6B7gPj0MwzdflFDHe8IaNN6bsv9C2XcQvAQicsKvFbePc8uZl1ps0/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
