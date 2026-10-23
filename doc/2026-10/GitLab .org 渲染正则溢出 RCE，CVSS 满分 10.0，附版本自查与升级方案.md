#  GitLab .org 渲染正则溢出 RCE，CVSS 满分 10.0，附版本自查与升级方案  
原创 安全值班室
                    安全值班室  安全值班室   2026-10-23 01:00  
  
![GitLab CVE-2026-110264 CVSS 10.0 封面](https://mmbiz.qpic.cn/sz_mmbiz_png/ZNdOU6xPdJfnq67zNWOgbtAwPDJnomeWgeP2gm0Xo8tMB4ENla1QeSBvqb87GUemjzlgV3vrXiaIgrkUkdtlcDbFKIptRS51oaicJtaVpONDw/640?wx_fmt=png&from=appmsg "")  
  
10 月 21 日，GitLab 发了一次计划外的补丁。  
  
7 个版本，同一天：19.5.1、19.4.3、19.3.6、19.2.9、19.1.9、19.0.10、18.11.13。  
  
**CVSS 10.0。不需要登录，不需要点链接，不需要用户交互。**  
  
只要你的 GitLab 会渲染一个 .org 文件。  
  
这不是插件洞。是 GitLab 从 2014 年的 7.0 起，一路带过来的正则引擎问题。  
  
12 年  
7.0 起全部版本  
7 个  
分支同日修复  
0 登录  
0 次点击  
  
01  
  
满分是怎么算出来的  
  
CVSS 3.1 的向量是 AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H。  
  
翻成人话：网络可达，复杂度低，不需要任何权限，不需要用户点任何东西；影响范围越过 GitLab 本身，机密性、完整性、可用性全部拉满。  
  
这些项叠满，最后就是 10.0。  
  
10.0 不是没见过。少见的是，它出现在一个自建代码托管平台上，而且一打就是 12 年。  
  
GitLab.com 已经修复。GitLab Dedicated 在 19.3.5-internal0 / 19.4.2-internal0 及以上也不用动。**需要动手的，只有自建实例。**  
  
![GitLab 各分支受影响范围与修复版本对照](https://mmbiz.qpic.cn/sz_mmbiz_png/ZNdOU6xPdJcibUdM5OWRM9sROut9FlicvFpibeyIjASzEtVFbqUnlQhK6WGGagzK5VSzO4G3icWe5P57swic7htkHBA098uIh5OEoHgAmMYkZFIY/640?wx_fmt=png&from=appmsg "")  
  
受影响的范围是：7.0 起，到各分支修复版之前。19.5 到 19.3 是正常支持范围，18.11、19.0、19.1、19.2 是官方额外回补的四条线。  
  
GitLab 平时只维护当前版本和前两个月度版本。这次一次回补四条线，原因不复杂：漏洞太深，老实例太多。  
  
02  
  
一个 .org 文件，怎么变成命令  
  
Org-mode 是 Emacs 的纯文本笔记格式。GitLab 用 org-ruby 把 .org 文件渲染成 HTML。  
  
.org 里可以写一行 #+TODO:，定义自己的待办关键词。渲染时，org-ruby 会把这些关键词拼进正则表达式。  
  
问题出在第二次拼接：关键词没有转义，直接进了 Regexp.new。  
  
#+TODO: TODO REPEAT{1,1000000000}  
  
Regexp.new("^#{@keyword}\\s*")  
  
文件里写什么，GitLab 就编译什么。一个能放 .org 文件进仓库的人，就能决定这段正则长什么样。  
  
Onigmo 是 Ruby 自带的正则引擎，GitLab 把它打包在自己的安装包里。它用 16 位有符号数存 repeat ID。repeat 数量超过 32767，ID 就翻负；匹配时再拿这个 ID 去选栈位置，写到了分配区外面。  
  
GitLab 的公告只写了一句：编译特制正则时整数溢出，未认证用户可执行任意代码。它没有公布前置条件。上面这条路径，来自 GitLab 同步开源的检测规则，以及独立分析对 org-ruby 代码的核对。  
  
**不需要登录。一个公开项目，放一个 .org 文件，就够。**  
  
还有第二个入口：编辑器的预览。[#preview]()  
 用请求里的 file_path 选渲染器，file_path 以 .org 结尾，就算你正在编辑 .md，也会走 Org 渲染。  
  
这不是「打开一个恶意链接」。是 GitLab 自己把文件内容编译成了正则，再在匹配时写坏了自己的内存。  
  
![从 .org 文件渲染到命令执行的链路](https://mmbiz.qpic.cn/sz_mmbiz_png/ZNdOU6xPdJcgvaqgvbR8kSlJk8Zg8PRCYMvZrqOmnEuzs7StSnChozWx18rtLfHsibAzBwhQVd8oxeovJr9l0jToqX6deFicYnmyYnETGibeaM/640?wx_fmt=png&from=appmsg "")  
  
成功之后，命令以 GitLab 的 git 用户执行。不是 root，但已经够了：代码仓库、CI/CD 变量、Runner 注册令牌、镜像仓库凭据，都在这个用户的脚下。  
  
GitLab 里的一个 RCE，不只是服务器。是代码、流水线、云凭证和签名钥匙。  
  
03  
  
为什么这次不能等  
  
截至 10 月 22 日，GitLab 说没有发现这个漏洞被实际利用，CISA 的已知利用目录里也没有它。  
  
这不代表可以等。  
  
第一，10 月 12 日刚修过一个同样 10.0 的正则 RCE（CVE-2026-97229）。**如果你上次已经升到 19.4.2 / 19.3.5 / 19.2.8，这次还在影响范围里。**  
新的修复版就压在它们上面一个补丁位。  
  
第二，官方补丁页没有给临时缓解。除了把实例从公网收回，没有等价的配置规避。  
  
第三，这次修复打在 GitLab 打包的 Ruby 上，不是把 org-ruby 升上去。独立分析核对 19.5.1 的依赖清单，里面仍然是 org-ruby 0.9.12。从源码安装、用系统 Ruby 的实例，升级 GitLab 源码不等于给系统 Ruby 打补丁。  
  
升级有代价。19.4.3 和 19.3.6 带数据库变更，单机升级要停机等它跑完；多节点可以走官方的零停机流程。  
  
但和「一个未认证的 10.0 挂在公网」比，停机是便宜的那一边。  
  
04  
  
今晚的检查顺序  
  
先查版本。三种方式：  
  
# Omnibus  
  
sudo gitlab-rake gitlab:env:info | grep -A1 '^GitLab information'  
  
  
# 通用（Omnibus / Docker）  
  
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION  
  
  
# Docker（容器名 gitlab）  
  
docker exec gitlab cat /opt/gitlab/embedded/service/gitlab-rails/VERSION  
  
管理员登录后打开 /help，也能看到版本。  
  
然后按分支升，别跨分支跳。7.0 到 18.11 升 18.11.13；19.0 升 19.0.10；19.1 升 19.1.9；19.2 升 19.2.9；19.3 升 19.3.6；19.4 升 19.4.3；19.5 升 19.5.1。  
  
Omnibus 按官方包名固定到修复版，升完执行 sudo gitlab-ctl reconfigure；Docker 把镜像 tag 固定到对应修复版，重建容器。升级前照常备份。  
  
升不了，就先收网。把实例从公网移到 VPN 或白名单后面，关掉不必要的公开项目和公开注册。这不是修复，只是把暴露面缩小。  
  
再查日志。GitLab 的 TLDR 仓库给了两条 Sigma 规则：Onigmo native worker crash、Org mode render error burst。  
  
前者盯原生 worker 崩溃；后者盯 .org 渲染报错突增。检测规则里写得很具体：同一客户端 30 分钟内 5 次以上 .org 渲染 Timeout::Error；或者 [#preview]()  
 请求的 file_path 以 .org 结尾，但 URL 路径指向另一个文件。  
  
注意，GitLab 自己说了，成功利用可能是静默的，不一定崩溃。**没告警，不等于没中。**  
  
如果实例曾经暴露公网，且版本在影响范围，升级后按这个顺序轮换：CI/CD variables、Runner 注册令牌、部署密钥、镜像仓库凭据、Personal Access Token、OAuth 应用密钥。再查审计事件里的新管理员、新 Runner、异常流水线。  
  
05  
  
一个月，第四个正则洞  
  
这不是孤立的一次。  
  
同一个研究者 joaxcar，一个月内在 GitLab 的正则引擎里报了四个代码执行漏洞：9 月 23 日的 CVE-2026-93577 和 CVE-2026-89078，CVSS 9.9，需要登录；10 月 12 日的 CVE-2026-97229，10.0，不需要登录；10 月 21 日的 CVE-2026-110264，10.0，不需要登录。  
  
前三个只打到 19.2。这一个，一路回到 7.0。  
  
同一个组件，一个月四次。不是运气差。是正则一直被当成「文本匹配的小工具」，没人愿意把它当攻击面。  
  
但正则不是小工具。它是编译器，是虚拟机，是会吃内存、会越界、会被写成 exploit 的原生代码。  
  
自建 GitLab 的补丁债，平时看不出来。一个 10.0 出来，全部到期。  
  
今晚的 GitLab，只有两个状态：已升级，和待升级。  
  
  
**MORE**  
  
往期回顾  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJd2VWNzn1zwES9wep9zXMxpd4pzZEX5A2auqIDHXxKQu4mTJjniatK47ib7967dK4mN09zS0mUGlx3yVJdDA9Oty2xsADqJnUGT4/640?wx_fmt=gif&from=appmsg "")  
  
[CTF新手速成](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4616106042689814529#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[靶场实战：从漏洞基础到红队综合](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4694391088487563265#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/ZNdOU6xPdJdeDc9RodEDdrwONAFutCzMLCRuQ9OArXOzQhRTMl9ALSeICh3S5pR88veNF5iaQ5MJKhSPibx156OibdfRWEls16qEiajUDZE3juE/640?wx_fmt=gif&from=appmsg "")  
  
[内网渗透学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4572157479522107394#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJfj1YSdkgqeicpsicCDlBHcAC5Q36YsAtcicZdr9WXT2gh8WDicPUbcHBoibYr5ph4LIFMJ1hibHZ8IfBSTjDkE6PfEmg4eXLnggFeQU/640?wx_fmt=gif&from=appmsg "")  
  
[云安全攻防实战连载](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4544879677747986438#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/ZNdOU6xPdJeAFe1tsVmCstoS3GUZ4ZjVQ4jz1Diawnx3CwB8yTlRmDGpK87xGlKVgticyM802YDmck0NdeAZb8u5IdHW2NMK3DgzSyapgiaHhg/640?wx_fmt=gif&from=appmsg "")  
  
[Web安全学习路线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3MzczMDc0OQ==&action=getalbum&album_id=4534863005871931394#wechat_redirect)  
  
  
