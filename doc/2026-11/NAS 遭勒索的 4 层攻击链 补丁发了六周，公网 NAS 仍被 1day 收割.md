#  NAS 遭勒索的 4 层攻击链 补丁发了六周，公网 NAS 仍被 1day 收割  
宝十八
                    宝十八  网络安全老宋   2026-11-04 04:00  
  
**导语：**  
 你好，我是网络安全老宋。  
安全攻防干货准时送达！  
  
  
网络安全  
老宋  
// 漏洞预警 · 存储安全  
  
// 漏洞预警 · 存储安全  
# NAS 遭勒索的 4 层攻击链补丁发了六周，公网 NAS 仍被 1day 收割  
  
360 反勒索证实群晖 NAS 被大面积加密，溯源指向 Cerber 变种 C3RB3R——利用 9 月 18 日已修复的 CVE-2026-13639/13684，从 1day 空窗到自动收单全程无人值守。  
  
CVE-2026-13639  
C3RB3R  
群晖 NAS  
  
●一句话精华  
NAS 不只存文件，它还是你的备份终点。公网暴露加上补丁滞后，生产数据和备份会在同一刻一起归零。  
  
10 月以来，群晖（Synology）NAS 被大面积勒索的消息在国内外社区接连刷屏：文件被加上  
 .L0CK3D  
   
后缀，每个目录躺着一份  
 read-me3.txt  
。10 月 29 日 360 反勒索发预警，10 月 31 日群晖官方证实并催促升级，11 月 1 日晚 360 社区挂出带样本逆向细节的分析贴——一周之内用户吐槽、官方公告、逆向报告节奏衔接得严丝合缝。  
  
大家讨论的多半是"群晖又中招了"。真正该问的是：一台扔在角落的存储设备，凭什么能在一夜之间同时打掉生产数据和备份数据？据 No More Ransom 梳理，群晖在中小企业、设计、影视、医疗影像团队里往往同时干四份活——文件共享、虚拟机存储、监控录像、备份目标。一台被加密，四份业务同时失灵，这才是真正的杀伤半径。  
  
那么问题来了，攻击到底是怎么打进来的？下面先拆几个最贵的误会，再走一遍四层攻击链。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flib04KsUQg4XNtkv4U0J7F5Y5yyRliarn4YFyrgZRGkY218esfVf3gLfepNTq5gSGg4sKmeeHJwNiaKbtvvCprgcsvJukhOcTD0F4/640?wx_fmt=png&from=appmsg "")  
  
▲ 封面：公网直连的 NAS 被公网箭头找上门，盘里的文件被锁链封死，备份也跟着够不着  
  
## 00先拆掉三个最贵的误会  
  
  
第一个误会：RAID 就是备份。  
不少人的第一反应是把 RAID 盘拔下来插电脑读镜像。没用，勒索是在文件系统层直接改写文件内容，RAID 阵列不但救不了你，反而会在毫秒级内把同一份乱码同步进每一块镜像盘——你以为多了份拷贝，实际是多了份一样的锁。  
  
第二个误会：开了快照就万事大吉。  
快照得分类型，普通快照在拿到 root 权限的人手里是可以被删的；DSM 7.2 起的"不可变快照（Immutable Snapshots，基于 WORM 一写多读）"才在锁定期内哪怕 root 也删不掉、改不了。有受害者反馈 C3RB3R 没去动 Btrfs 快照，数据就这么捞了回来，但这是运气不是设计。  
  
第三个误会：同步工具能当备份用。  
一位国内用户半夜听见风扇狂转，起来发现 Cloud Sync 正高速运转，把刚生成的  
 read-me3.txt  
   
和  
 .L0CK3D  
   
加密文件同步上多个 OneDrive 网盘，云端干净的旧版本被挨个覆盖成乱码。同步是复制，不是备份。  
  
还有个坑得单独说：  
别为了防 PHP2Shell 手欠去卸载 PHP 7.4。比利时安全机构 nxtbit 实测发现，群晖底层依赖里 Hyper Backup 强制绑定 PHP 7.4，你强行卸载它，群晖会连带把 Hyper Backup 一起卸掉，定时备份当场停摆。而 PHP 7.4 只在本地跑小脚本、不接收公网请求，只要别用它在 Web Station 对外挂站点，留在机器里是安全的。  
  
老宋数据：  
一次免认证远程代码执行，只需要发送一个请求。所以"输错 5 次自动封 IP"这道防线在它面前等于不存在。你设的十六位密码和二次验证，是往门后面加锁；可这扇门压根没走锁那道流程。  
  
## 01四层攻击链，比你想的短  
  
  
第一层：免认证 RCE，两分钟拿下 root。  
Rapid7 研究员 Ryan Emmons 在 DEF CON 33 公开了名为 Disguise Delimit 的利用技术（对应  
 CVE-2024-10441  
，CVSS 9.8）：群晖登录框把用户输入打包成内部表单，用  
 PLUGIN_VALUE_END  
   
做字段分隔符，再交给以 root 运行的守护进程 synotade 处理，旧版系统压根不检查你在"用户名"框里塞了什么。攻击者直接在用户名里伪造一个结束分隔符，后面拼上  
 LD_PRELOAD  
   
和  
 LD_DEBUG_OUTPUT  
   
这两条动态链接器环境变量指令，于是 glibc 加载器在校验账号之前，就以 root 身份往  
 /etc/cron.d/  
   
写进一行恶意脚本。  
  
本轮主入口是另一组。  
9 月 18 日群晖发布 Synology-SA-26:13，一次修了八个漏洞，其中两个 CVSS 9.8：CVE-2026-13639  
   
是 DSM 登录逻辑熵值不足（CWE-331），令牌可被预测；CVE-2026-13684  
   
是 SCGI 组件输出转义不当（CWE-116）。两者都允许未授权远程读写任意文件，而该公告的 Mitigation 一栏写的是 None——没有临时缓解，只有升级。从补丁发布到大规模爆发，中间隔了 41 天。  
  
第二层：驻留与横向。  
原生套件这条路走 PHP2Shell。PHP 官方 10 月 21 日紧急发版修复  
 CVE-2026-109618  
   
和  
 CVE-2026-109619  
，两个洞都在解释器内核里，不是某个 CMS 插件；10 月 26 日 CISA 把它们收进 KEV，联邦机构修复死线 10 月 29 日。群晖 SA-26:14 确认 Web Station、Synology Calendar、Mail Station 等依赖 PHP 的对外套件全部暴露在风险下。容器这条路更贴近企业现场：Container Manager 里自建 GitLab、Confluence 并把端口映射到公网，GitLab 近期接连爆出未授权 RCE（CVE-2026-110264  
、CVE-2026-97229  
），攻击者利用高权限容器挂载或宿主机逃逸把拉取脚本写进 crond 静默潜伏，NAS 变成跳板。  
  
第三层：分级加密，这才是真杀伤。  
C3RB3R 是 2016 年就成名的老牌勒索家族 Cerber 的变种（360 反勒索的定性口径，另有研究者认为它基于泄露的 Conti 源码，归属仍有分歧）。它硬编码跳过 27 个群晖核心系统目录（/boot  
、/etc  
、/usr/syno  
   
等），理由很简单——把系统锁死了你就看不到勒索信了；更狠的是分级加密，大于 5MB 的普通文件每隔 5MB 只加密头部 1MB，几秒钟就能废掉一部几十 GB 的原盘，但内置的一份 187 个扩展名优先名单（.sql、.sqlite、.vmdk、.qcow2、.iso 等）命中就一律 100% 全文件加密，不计成本。算法是 ChaCha8 加 RSA-2048 公钥封装，每个文件独立生成 256 位密钥和 64 位 nonce，没有私钥在数学上无法暴力破解。  
  
第四层：全自动收单。  
每个目录生成一份带 6 位设备识别码的  
 read-me3.txt  
，暗网站点为每台设备单独生成比特币地址和二维码，赎金通常在 0.03696 到 0.085 BTC 之间；页面提供"免费解密 1 个文件"的上传框当场证明捏着私钥，链上交易完成 6 次区块确认后解密器下载链接自动刷出。  
  
对防守方来说，这四层里最值钱的不是解密方法，而是检测点  
。文件被改名是事后，IoC 才是事前。  
<table><tbody><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom: 2px solid rgb(217, 210, 199);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">检查类别</span></section></td><td style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(217, 210, 199);padding: 8px 0px;text-align: left;"><section><span leaf="">具体特征</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">加密主程序</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">/bin/qnetd、/bin/libgld</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">防重运行标记</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">/bin/l0ck.enc、/tmp/log.0</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">持久化驻留</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">/etc/cron.d/synot*、synotade.&lt;PID&gt;</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">日志关键字</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">PLUGIN_VALUE_END、LD_PRELOAD、LD_DEBUG_OUTPUT、synotade</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="111" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">下载源</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);padding: 8px 0px;text-align: left;"><section><span leaf="">46.8.78.122（xdy5.sh）、.SYNOACCOUNTDB.corrupt 报错</span></section></td></tr></tbody></table>  
指标整理自 360 反勒索《Cerber 勒索攻击群晖 NAS 事件预警》与 No More Ransom 处置指引。Reddit 受害者 u/NakamuraHwang 从计划任务解出的 Base64 指令指向 46.8.78.122，而国内一位白群晖用户在 V2EX 求助帖里说，加密开始前 8 分钟同一个 IP 发起了"失败登录"。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flicyF89YdjamKJFGOdXNdW2rrlfdgSjq0ggXyuUianHHUVhAydgZOQanxtgTaVW1DeZj4rkZ8WG2HxcZqiagBYp6rcU0zRh76etJs/640?wx_fmt=png&from=appmsg "")  
  
▲ 图1：免认证 RCE 打穿入口，写入为计划任务驻留，再以分级加密完成加密，最后自动收单  
  
## 02中招后的第一小时，顺序错了越救越死  
  
  
先做一件事：分清是哪种形态，No More Ransom 的分类框架很实用。  
  
形态一，内网 SMB 扩散。  
一台中招的 Windows 终端挂着 NAS 共享，加密器顺着写权限把共享目录里的文件逐个锁死，NAS 系统本身没被入侵，DSM、快照、Hyper Backup 配置通常完好。这种情况最好处理，Btrfs 快照往往能直接把数据救回来。  
  
形态二，NAS 自身被攻破。  
DSM 或 SSH 暴露在公网，口令薄弱或漏洞被利用，攻击者拿到管理员权限后会先删快照、停掉备份任务再开始加密，恢复难度陡增。判断方法不复杂：登录页有没有被替换、SSH 端口是不是莫名开着、有没有新建或被改名的管理员账号、快照列表是不是空了一大截、日志中心里成片的境外 IP 登录失败记录，都是明确信号。  
  
四个"千万别做"：  
- ① 别重启、别重装、别点"重置"，也别跑 fsck。  
这些动作会破坏残留日志、快照和未被覆盖的文件块，等于把唯一的取证线索一起擦了。  
  
- ② 别以为拔了网线就完事。  
外接 USB 盘要拔掉，云同步要断开授权，否则乱码会一路同步上云，覆盖掉你干净的旧版本。  
  
- ③ 别急着找解密工具。  
目前没有公开的免费解密工具，把加密文件和勒索信封存到离线介质上等就行。  
  
- ④ 别删现场。  
/var/log、rm.log、计划任务残留都是判断"NAS 到底有没有被攻破"的直接证据。  
  
两个"赶紧做"：  
一是查 Btrfs 快照还在不在，用感染前的时间点在离线环境下还原；二是找另一份离线副本（外接冷备、异地 NAS、云端都行）重建恢复。如果你身边就有运维同事还在用端口映射的方式远程访问 NAS，这段先转给他看一眼，方法比命令更值钱。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flicFT3u8Yv9oY29M2LMsBCIHmcferNibCOX9bg44V5aMqyFAdWFsLWfLyjuYxREn2TXOO7xcnVgjJCxURydRnC7sJV5PAeKlArV8/640?wx_fmt=png&from=appmsg "")  
  
▲ 图2：深夜里 NAS 红色告警灯闪烁，一只手果断拔出电源线，旁边木桌上摆着一块未连接的移动硬盘  
  
## 03从一台 NAS，看到一类影子资产  
  
  
真正该带走的不是"群晖怎么修"，而是"我手里还剩几台这样的设备"。按角色分三张清单，基本能覆盖九成现场。  
  
第一张，暴露面清单。  
谁把 5000、5001 或自定义管理端口映射到公网了？谁在用路由器的 UPnP 自动映射？谁在用 QuickConnect？谁明面上说"我们走的是 VPN 中转"，实际后台还开着一个自己的端口？注意区分两种远程访问——厂商的 VPN 与中转服务，和自己家里路由器上的端口转发，是完全不同的两件事。  
<table><tbody><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom: 2px solid rgb(217, 210, 199);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">DSM 分支</span></section></td><td style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(217, 210, 199);padding: 8px 0px;text-align: left;"><section><span leaf="">最低安全版本（Synology-SA-26:13）</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">DSM 7.4</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">7.4-90075</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">DSM 7.3.2</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">7.3.2-86009-4</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">DSM 7.2.2</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">7.2.2-72806-9</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">DSM 7.2.1</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(181, 39, 15);padding: 8px 0px;text-align: left;"><section><span leaf="">7.2.1-69057-12</span></section></td></tr></tbody></table>  
版本基线整理自群晖安全公告 Synology-SA-26:13 与 360 反勒索通报；套件中心的 PHP 系套件、Synology Calendar、Mail Station、Web Station 也要全部更新到最新修复版。  
  
第三张，备份可达性清单。  
有没有一份副本，是这台 NAS 根本写不到的？NCSC 的口径是在 3-2-1 之外补了一条：任何时候至少有一份备份处于离线状态，云端那份在不用的时候还得在数字层面断开连接。理由很硬——勒索软件不只加密原始数据，NCSC 明确记录过它顺手把 USB 盘、网络存储里的备份一起锁死，连已被入侵的云端备份位置也照打不误。  
  
再补一条供应链视角。  
这是 PHP2Shell 最扎心的地方：它危险不是因为 9.8 分，而是因为它根本不一定装在你看得见的地方。CISA 的原始提醒是"PHP 往往是别的软件的一部分"，所以"我装了 PHP 的服务器"这个清单天然是漏的。NAS 里的 Web Station、Calendar、Mail Station，备份一体机的管理控制台，远程接入网关，楼控平台——这些里面藏的 PHP 你自己升级不了，只能找厂商要补丁。补丁优先级也别按 CVSS 从高往低排：KEV 目录只收录 CISA 确认"已在野被利用"的漏洞，把它当第一优先级，比按分数排靠谱得多。  
  
## 04把"自查"压缩到 15 分钟  
  
  
先说个本周的新信号。10 月 29 日发布这次勒索预警的是 360 反勒索团队，11 月 2 日同一家公司在乌镇世界互联网大会展出了 AI 安全攻防系统"图龙锋"，累计挖掘漏洞超过 1.5 万个，500 余家企业接入做专项测试。同一周，AI agent 厂商 Causal Security 的 Cipher 在 3 小时 49 分钟内，从从没接触过的 EPICS Base（全球约 90 个粒子加速器和聚变实验站点在用的控制软件）里挖出 4 个未知漏洞，其中一个 9.8 分可远程执行代码。  
  
这不是说 AI 能替你把 NAS 修好。恰恰相反，这两件事指向同一个结论：AI 最擅长的事，是把"你根本不知道自己有什么"变成一张清单。版本基线、暴露面、IoC、日志异常，这些活儿枯燥、跨设备、容易漏，人做起来三天，AI 做起来十几分钟。所以我建议的做法是，把下面这段提示词里的资产信息换成你自己的，扔给手边的 AI 工具（Claude Code、Codex、WorkBuddy 都行），让它走一遍只读体检。三个原则不能破：只读、不改、留痕；查完立刻关掉 SSH；报告出来先人工复核再动手。  
  
给 AI 的体检提示词骨架：  
- ① 版本与组件基线。  
读取 /etc/defaults/VERSION，核对 DSM 版本与 buildnumber，判断是否满足 7.4≥90075、7.3.2≥86009-4、7.2.2≥72806-9、7.2.1≥69057-12；列出已安装的 PHP 相关套件与 Web Station、Synology Calendar、Mail Station 版本，对照 SA-26:14 判断是否达标。  
  
- ② IoC 排查。  
查 /bin/qnetd、/bin/libgld、/bin/l0ck.enc、/tmp/log.0；查 /etc/cron.d/、/etc/crontab、/var/spool/cron/crontabs/ 下是否存在 synot 前缀或异常的 Base64 下载指令；检索 /var/log/ 中的 PLUGIN_VALUE_END、LD_PRELOAD、LD_DEBUG_OUTPUT、synotade、46.8.78.122 以及 .SYNOACCOUNTDB.corrupt 报错。  
  
- ③ 暴露面与快照状态。  
查当前监听端口、有无公网 UPnP 映射痕迹，确认 Btrfs 存储空间是否配置了 Snapshot Replication 快照保护、Hyper Backup 任务状态与备份目标是否可写。  
  
- ④ 输出约束。  
全程只读，不得修改或删除任何文件；输出中文排查报告并给出修复优先级建议；结束后提醒关闭 SSH。  
  
边界也得说清楚，AI 给你的是候选线索不是结论。它会把日志里的一堆异常都摆出来，需要人判断哪些是真攻击痕迹；真正动配置的活儿——不可变快照策略、备份拓扑调整、端口映射下线，依然要人拍板执行。顺手插一句我自己的判断：这一轮 NAS 被批量勒索，暴露的不是群晖的代码质量，而是资产清单的缺失。大部分人心里"我有几台 NAS"这个答案是模糊的，而攻击者的扫描是精确的，两者对上就出事。你现在去列一张表，把所有 NAS、备份一体机、录像存储、跑着 GitLab 的那台机器都写上去，比今晚研究怎么解密有用得多。  
  
## 结尾：老宋说  
  
这不是一个新漏洞，而是一类老问题换了层皮：  
把不该暴露在公网的管理界面敞着，让人拿自动化脚本批量收割。  
  
只要"远程访问就得开个端口"这个习惯还在，  
这类事件就会一个季度再来一次，区别只是这次被勒索的是 NAS，下次可能是录像机或者工控网关。  
  
所以别等出事才想起来。  
今晚花十五分钟，把端口映射关掉、把版本基线对一遍、把备份可达性确认一遍。  
  
## 读推荐阅读  
  
1  
[甲方运维注意：.sorry勒索来了信创服务器也中招](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487500&idx=3&sn=7c68dd9ecb13d79d5bbe0b92ad14e3ca&scene=21#wechat_redirect)  
  
  
2  
[勒索软件攻防实战指南：从识别、处置到防御的全链路应对](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247485549&idx=1&sn=bc46a378e59273cca1e85227eed5e2e8&scene=21#wechat_redirect)  
  
  
3  
[甲方运维：主机被远控别拔网线 黄金1小时应急处置清单](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487426&idx=2&sn=0e8520b6ee519cf9f5b79cc1ccc4f025&scene=21#wechat_redirect)  
  
  
4  
[甲方运维应急排查： 银狐木马进内网，3步定位失陷主机](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487594&idx=2&sn=34b32b1f046b12c6638474e73554be56&scene=21#wechat_redirect)  
  
  
5  
[甲方运维：重保128项加固清单，6层防御照着做](https://mp.weixin.qq.com/s?__biz=MzAxMzIxMjM3Ng==&mid=2247487401&idx=3&sn=b0ceb42e1d3eed8e0996edc2f0ba7b12&scene=21#wechat_redirect)  
  
网络安全老宋 · 转载请注明出处  
  
防御，不是在演练期间发现攻击，而是在演练开始前就把攻击面收敛到最小。  
  
end  
  
  
  
不想错过文章内容？读完请点一下**“在看**  
**”**  
，加个**“****关注”**  
，您的支持  
是我创作的动力  
  
期待您的一键三连支持（点赞、在看、分享~）  
  
