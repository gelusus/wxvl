#  超长密码反而能免密登录？OpenSSH + musl 组合漏洞，把 Alpine 容器的门焊开了 免密登录  
宝十八
                    宝十八  网络安全老宋   2026-11-09 04:00  
  
**导语：**  
 你好，我是网络安全老宋。  
安全攻防干货准时送达！  
  
  
网络安全  
老宋  
// 漏洞 · 认证绕过  
  
// 漏洞 · 认证绕过  
# 超长密码反而能免密登录？OpenSSH + musl 组合漏洞，把 Alpine 容器的门焊开了免密登录  
  
一个 256 字符以上的密码，竟能在不开密码认证的情况下登进服务器。CVE-2026-123158，CVSS 8.1，专挑 Alpine、Docker、云主机下手。  
  
CVE-2026-123158  
认证绕过  
Alpine  
  
●一句话精华  
密码不是越长越安全——在 musl 的世界里，长过头等于没有密码。  
  
你肯定听过一句话：密码越长越安全。我们做等保、做基线检查，第一件事就是要求口令拉到 12 位、16 位，越长越好。  
  
可是这两天冒出来的一个漏洞，把这条常识直接反将了一军。OpenSSH 在搭配 musl libc 的 Linux 上，出现了一个认证绕过：攻击者只要敲一个 256 到 1024 字符的超长“密码”，就能在根本没开密码认证的环境里登进服务器。  
  
这个漏洞编号 CVE-2026-123158，NVD 在 11 月 5 日收录，CVSS 8.1，评级 High。OpenSSH 官方自己把它叫 critical security workaround——说白了就是“这事儿很要命，赶紧改”。它最阴的地方在于目标画像：Alpine Linux、Docker 容器、云主机，恰恰大量使用 musl。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/yJLbez93flibkstfy4HnJodDn3JNgyz5hmyey6aCJutO248OkJdwqxe9qVEdwAj6DysqQJUpksmdibdVg4o3ibDMLOpHd4ib19CjsmVCIAXBGsk/640?wx_fmt=jpeg "")  
  
▲ ▲ 超长密码像一把钥匙，拧开了容器的大门（示意图）  
  
## 00漏洞到底怎么回事：musl 把“超长密码”翻译成了星号  
  
<table><tbody><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom: 2px solid rgb(217, 210, 199);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">触发条件</span></section></td><td style="font-size: 14.5px;line-height: 26px;font-weight: 700;color: rgb(26, 23, 20);border-bottom-width: 2px;border-bottom-style: solid;border-bottom-color: rgb(217, 210, 199);padding: 8px 0px;text-align: left;"><section><span leaf="">说明</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">① musl libc 系统</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(44, 44, 44);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">典型为 Alpine Linux 与 Docker 镜像，容器场景极常见</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">② sshd 未启用 PAM</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(44, 44, 44);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">musl crypt 绕过的正是非 PAM 路径，Alpine 默认如此</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">③ 密码认证为 yes</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(44, 44, 44);border-bottom-width: 1px;border-bottom-style: solid;border-bottom-color: rgb(236, 232, 224);padding: 8px 0px;text-align: left;"><section><span leaf="">OpenSSH 默认即 yes，多数未改配置中招</span></section></td></tr><tr style="text-align: left;"><td data-colwidth="172" style="font-size: 14.5px;line-height: 26px;color: rgb(94, 93, 93);padding: 8px 10px 8px 0px;text-align: left;"><section><span leaf="">④ 星号账户 + 哈希账户</span></section></td><td style="font-size: 14.5px;line-height: 26px;color: rgb(44, 44, 44);padding: 8px 0px;text-align: left;"><section><span leaf="">存在“*”字段账户，且另有 $5$/$6$ 哈希账户</span></section></td></tr></tbody></table>  
四要素需同时满足，缺一项即不成立  
  
先说结论，这个漏洞的根子不在 OpenSSH，而在 musl libc 的  
 crypt()  
   
实现上。正常的密码校验里，sshd 拿到你输入的密码，交给 crypt() 做一次哈希，再和 /etc/shadow 里存的哈希比对，比对上了才放行。  
  
问题出在 musl 对“超长输入”的处理：当密码长度超过 256 字节时，musl 的 crypt() 会返回一个固定的字符串  
   
“*”  
，而这个“*”恰好是 /etc/shadow 里一种特殊账户的密码字段——代表“该账户被锁定、没有可用密码、只能走其他认证方式（比如 SSH 密钥）”。于是攻击链路闭环了：攻击者拿超长密码去登录，musl 算出来是“*”，和那个“星号账户”的字段一模一样，sshd 一看比对通过，直接放行。CWE 把它归为 CWE-394（意外的状态或条件），非常贴切。  
  
老宋提醒：  
被登录上的，是那个“密码字段为 *”的账户本身，而不是凭空创造一个新身份。所以真正危险的组合是——你的系统里恰好有一个被锁密码、只靠密钥登录的账户（比如 Alpine 默认从 3.16 起的 root），再加上另一个存了 $5$/$6$ 哈希的账户存在。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93fl86D6cwcGqugQ2uSE4CicBWwkIZ1a8Fic8WWynGOcibLFL7icRnpcXBQP04qrB9iaIP8dHjbG4DPVibIYLDu4QXVJbEAcNhjLwOOEXXU/640?wx_fmt=png&from=appmsg "")  
  
▲ ▲ 触发链路：四要素同时满足 → 认证绕过  
  
## 01谁中招、谁能幸免、今天该做什么  
  
  
  
中招画像已经很清楚了：Alpine 3.16 及之后版本自带的 openssh-server 包、Docker 里那些  
 FROM alpine  
   
的镜像，以及大量云厂商提供的轻量 Alpine 主机。这些环境天然满足“musl + 默认无 PAM + 默认开密码认证 + 默认 root 为星号账户”的组合，几乎是为这个漏洞量身定做。  
  
修复动作也很明确。OpenSSH 已经在 10.6p2 版本里修掉了这个问题（11 月 5 日发布），Alpine 从 3.16 起的所有受支持分支都回溯打了补丁。但要注意——  
升级 openssh 包之后，必须重启 sshd 才能让修复生效  
，很多同学升完就以为完事了，结果 sshd 进程还是旧代码在跑。  
  
如果你暂时没法马上升级，最有效的缓解措施只有一句话：  
显式关掉密码认证，只用密钥  
。在 sshd_config 里写上  
 PasswordAuthentication no  
，然后校验配置并重启 sshd。这一步做完，无论底层 libc 怎么算，攻击者都没法用密码这条路径进门了。  
  
关于在野利用，老宋必须给你踩个刹车：  
截至发稿，还没有权威来源确认这个漏洞已经在野外被大规模利用  
。部分中文快讯里写的“已遭在野利用”，目前缺乏 NVD、厂商公告或可靠研究员的佐证，本文标记为“待验证”。这不是说你可以不修——CVSS 8.1 加上 Alpine 的普及度，早晚会有人拿来用；而是说，别被标题党带节奏，按下面的清单踏实补就行。  
- 第一步  
确认 libc——ldd --version  
   
看是不是 musl，或查 /etc/os-release 是不是 Alpine。  
  
- 第二步  
查 sshd 是否走 PAM——grep -i usePAM /etc/ssh/sshd_config  
，Alpine 默认是 no。  
  
- 第三步  
查密码认证开关——看到 PasswordAuthentication yes 就要警觉。  
  
- 第四步  
看 shadow 里有没有星号账户——grep ':\*:' /etc/shadow  
，再确认是否有 $5$/$6$ 哈希账户。  
  
- 第五步  
立即加固——升级 openssh 到 10.6p2+ 并重启 sshd；升级不了的，直接 PasswordAuthentication no + 密钥登录。  
  
- 第六步  
排查历史入侵——翻 auth.log 里有没有超长密码的登录成功记录（重点看 256 字符以上）。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flicicujSgpsw9uADGwCrqPAXFVicAUa2ciaTCIKD7CibS5pQOAbThuwUqQUEFweX2lfS9TjVOkKdbVHxicqIYF0mQZc61eiaJ1nPOMrfc/640?wx_fmt=png&from=appmsg "")  
  
▲ ▲ 影响面与止血决策  
  
## 结尾：别只看单点，要看组合  
  
这个漏洞给所有人的提醒其实就一句话：安全不是堆参数，而是看组合。单独看，musl 超长密码返回星号是个边缘行为；单独看，Alpine 默认 root 锁密码是个好习惯；可一旦 OpenSSH 把它们串起来，好习惯反而成了攻击入口。  
  
所以老宋一直跟团队强调：做暴露面收敛，不能只看单点配置，要看“libc × sshd × 账户模型”这层叠加关系。今天这篇你只要记住三件事——确认你是不是 musl/Alpine、把密码认证关掉只用密钥、升级后记得重启 sshd。剩下的，交给那份清单。  
  
## 读推荐阅读  
  
1. 多地爆发 “银狐” 病毒，办公场景成重灾区，这些防范要点必看  
  
2. 紧急预警 | Windows远程桌面高危漏洞爆发，千万服务器面临沦陷风险！  
  
3. 渗透测试从业者的新工具：938 Star、号称漏洞利用成功率90%（鸾鸟LuaN1ao）  
  
4. 甲方运维应急排查： 银狐木马进内网，3步定位失陷主机  
  
5. 渗透测试从业者遇到过这种情况吧： 扫出 37 个高危，能打进去的不到 10 个  
网络安全老宋 · 转载请注明出处  
  
防御，不是在演练期间发现攻击，而是在演练开始前就把攻击面收敛到最小。  
  
end  
  
  
  
不想错过文章内容？读完请点一下**“在看**  
**”**  
，加个**“****关注”**  
，您的支持  
是我创作的动力  
  
期待您的一键三连支持（点赞、在看、分享~）  
  
