#  刚刚爆出！Windows 11 0day漏洞， 绕过锁屏，执行代码  
原创 hacking
                    hacking  Hacking黑白红   2026-11-22 05:58  
  
11月19日刚爆出来、 Windows 11 0day漏洞：**FastUnlock，可以在锁屏状态下执行代码。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TVljsu2eAicLdKUiaBnAGgqVtCuDUYbZaZ3XEeWFNAayUy66bqkqJ4QCyMdOtR1uQ8mO5OH8nXNagDTCdrbUCh779MPTqyms99iaevUjY2OEao/640?wx_fmt=png&from=appmsg "")  
  
起因，11 月 19 日，国外安全研究员阿卜杜勒哈米德・纳西里在 GitHub 公开了 FastUnlock 漏洞文档与复现演示视频。  
  
  
独立安全厂商 ThreatLocker 同步完成验证，确认漏洞稳定可复现。  
  
截至发稿，**微软尚未发布补丁、没有官方受影响版本清单**  
。  
  
很多企业和个人都有一个默认认知：  
  
电脑锁屏 = 数据锁住，只要开了 BitLocker 全盘加密，笔记本临时离开工位也相对安全。而这个漏洞，直接推翻了这条物理安全假设。  
  
⚠️ 先划最重要的误区：****  
  
**FastUnlock 不是破解 BitLocker**  
。 BitLocker 本身没有被攻破。在 TPM 自动解锁模式下，开机后磁盘就已经解密，锁屏只是登录身份屏障。  
  
漏洞利用发生在磁盘已经明文可读的阶段，目标是突破锁屏界面，拿到**WsiAccount 锁屏账户上下文执行权限**  
。  
## 漏洞原理：  
##   
## 讲述人 Narrator 的校验缺失  
  
漏洞根源在 Windows 内置辅助功能 —— 讲述人（Narrator）。  
  
 微软近期重构讲述人 UI 界面时，保留了 “系统身份禁止弹窗” 的防护逻辑，但是漏掉了**锁屏上下文校验**  
。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TVljsu2eAicLOFSSKyrZGoiafzJtRtd47U50EUAr4iaPXibPKpibibuDJqhnuKnj4JX0Dr1hqibbcOyZBnQdRKObp5a7yR1XhREYrYMkS73tqgEKkU/640?wx_fmt=png&from=appmsg "")  
  
  
简单说：系统允许讲述人新界面在锁屏环境拉起，后续链路就可以被攻击者一步步操控。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TVljsu2eAicIaGUJRsibk0o2SsW4uP3X8EbxTF8sicbHY70Ck6xa6CYiaPZicnic24HxcfgZBw9kvlQfVic5KbAia0oODrYMGnqVbRIxFmMn7o89oXs/640?wx_fmt=png&from=appmsg "")  
  
  
完整利用流程：  
1. 锁屏界面点击【忘记 PIN / 密码】，进入微软账户恢复流程（前提：设备绑定可恢复的微软在线账户）  
  
  
1. 按下 Win+U  
 调出辅助功能面板，启用讲述人  
  
  
1. 讲述人聚焦后台不可见窗口，攻击者通过 Caps、方向键、回车盲导航操作界面  
  
  
1. 修改系统自动播放策略  
  
  
1. 插入带恶意载荷的 U 盘，借助自动播放拉起资源管理器，实现代码执行  
  
研究员自述，从发现漏洞到完整拿下一台 BitLocker 加密终端，只需要**5 分钟**  
，在测试设备上复现稳定。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TVljsu2eAicIlD1kH6FuVKoiaKW0iaKn6XJyFYmemf3P3KX121jPg756xWrWUfe0fkv6LSGTPrICfzUMhCuOn644qbtPGUBiaicJIl9yqQ5vDPKM/640?wx_fmt=png&from=appmsg "")  
  
> 权限边界：拿到的是 WsiAccount 上下文，不是管理员权限，但足够读取该账户可见数据、投放后门、作为后续横向渗透跳板。 利用前提：**必须物理接触设备，无法远程利用**  
；暂未确认 Windows10 是否受影响。  
  
##   
## 哪些场景风险最高？  
##   
  
这个漏洞不是黑客远程扫端口的武器，是典型**物理攻击面漏洞**  
，下面这些场景一定要警惕：  
  
 ✅ 会议室公用笔记本、前台共享终端   
  
✅ 研发、财务等高敏感岗位笔记本，外出携带、临时离座只锁屏、不关机  
  
 ✅ 休眠 / 睡眠状态的电脑：同样处于磁盘已解密状态，风险和锁屏一致  
  
对照安全框架：  
- NIST SP 800-53：AC-11 设备锁定、SC-28 静态数据保护控制失效  
  
- MITRE ATT&CK 链路：T1200 硬件添加（U 盘）+ T1078 有效账户（WsiAccount）+ T1562.001 削弱防御（篡改自动播放）  
  
很多蓝队容易忽略物理侧的零日，这种漏洞一旦在内部被社工、内鬼利用，EDR 如果没有对应检测规则，很容易漏掉告警。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TVljsu2eAicId4jcTQ0YFKWGkPqq4iaXy7as6Jb1Tyv9dibiaHqMIN9l8DwzibYUgias6WnqC7mgvKh7geHMZPm8WTKHJCPRwafuWCO343nx0k90k/640?wx_fmt=png&from=appmsg "")  
## 补丁空白期，蓝队 & 运维可落地缓解方案  
##   
  
没有补丁的时候，防守思路就三件事：切断链路、增加屏障、留存审计痕迹。  
  
### ✅ 事前基线加固（优先做）  
###   
1. 组策略强制禁用可移动介质自动播放、自动运行，直接切断 U 盘载荷执行链路，这是最核心的缓解手段  
  
  
1. 高价值终端，给 BitLocker 开启**预启动 PIN 码**  
。这样哪怕休眠唤醒被绕过，重启之后必须输入启动 PIN，磁盘不会自动解密  
  
  
1. 规范员工习惯：离开高密设备，**优先关机，而不是只锁屏**  
；睡眠模式同样有风险  
  
  
1. 按需通过策略限制讲述人等辅助功能在锁屏界面启用（注意兼顾残障员工使用场景，不要一刀切）  
  
  
1. 物理管控：笔记本电脑锁、机房 / 研发区准入、工位摄像头、固定终端防盗  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TVljsu2eAicKcgWS2eKGOFIJ09xravdnoWI9icJyFVpwXr9yzicwIKAibBOnvf6Wl4B3AJ0iccl37P1xutRibaNBhVnM8doDbfY1GhPydwssV20AA/640?wx_fmt=png&from=appmsg "")  
### ✅ 事中检测规则（EDR/SIEM）  
###   
  
重点监控这几类告警：  
1. 进程创建事件（4688、Sysmon EventID 1），进程账户为 WsiAccount  
  
1. explorer.exe 作为父进程，程序路径来自 U 盘、移动介质  
  
1. 锁屏时段，Narrator 讲述人进程、系统设置窗口异常拉起  
  
1. 自动播放相关注册表项、AutoplayDevices 配置被篡改  
  
1. 非登记 USB 介质在非工作时段接入终端，和资产台账联动审计  
  
  
### ✅ 事后与持续跟踪  
###   
1. 回溯历史日志，排查过往是否存在 WsiAccount 账户下异常进程创建行为  
  
1. 统一复核全公司 BitLocker 策略，重点排查笔记本是否全部配置预启动 PIN  
  
1. 把 FastUnlock 加入漏洞跟踪台账，微软发布补丁第一时间测试、批量推送  
  
## 关于 PoC 与研究链接  
  
研究员公开仓库只放出漏洞文档、说明与演示素材，**没有直接一键利用的 EXP**  
。 所有验证、复现，仅限**自有、授权的测试靶机**  
，严禁在生产环境、他人设备尝试，遵守网络安全相关法律法规。  
- GitHub 披露仓库：https://github.com/MSNightmare/FastUnlock  
  
- 打包下载：https://github.com/MSNightmare/FastUnlock/archive/refs/heads/main.zip  
  
- 演示视频：https://www.youtube.com/watch?v=6BKsdUtc6bg  
  
- 原始推文：https://x.com/0x0SojalSec/status/2123964360762798137  
  
##   
## 写在最后  
##   
  
终端安全里，大家总盯着远程漏洞、内网横向，常常低估物理接触攻击面。 FastUnlock 给我们的提醒很直白：****  
  
**锁屏是身份校验屏障，不是数据保险箱；TPM 自动解锁的 BitLocker，不等于离座安全**  
。  
  
在零日补丁窗口期，不靠神奇的应急工具，而是回归基础：管住 U 盘自动播放、开启 BitLocker 启动 PIN、养成关机习惯、补齐进程审计，就是当下最有效的防线。  
  
参考文章  
```
https://mp.weixin.qq.com/s/hD6AaJELH_w6Sba00Y6ADA
https://mp.weixin.qq.com/s/ClklkGnmja-Q71EG2O61ww

```  
  
  
