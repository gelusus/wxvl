#  Ivanti Connect Secure 未认证 RCE 与提权漏洞链分析：VackBot 已实现检测覆盖  
 墨云安全   2026-10-16 08:14  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/7lCiaSMMkhia4WIkRNZHTwq8jJicy27jdbWa7ED26252RGmSPRE0rmHQsgZ6ZoichVyFNlvhLelZS09a194B9dyoAQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
点击↑  
**蓝字**  
  
关注  
**墨云安全**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC64Jegp0lgLDGafrdvU8MV80FwNQPMbnsrL7uZZicQcmtDkwNu5wflcYicoJ1OSoKLFpcH2wc15vJxw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1 "")  
  
Ivanti Connect Secure（ICS）存在一个严重未认证远程代码执行漏洞（CVE-2026-104935）。该漏洞允许未经身份验证的远程攻击者在目标系统上执行任意命令，CVSS 评分高达 8.6。Ivanti 已在 2026 年 7 月发布的 25.1.3.0 版本中首次修复该漏洞，但在将补丁移植到其他分支版本的过程中，漏洞细节被攻击者通过补丁比对（Patch Diffing）分析获取，导致在野利用激增。CISA 联合多国网络安全机构已发布联合咨询，警告该漏洞已被活跃利用。  
  
针对该高危漏洞，**墨云 VackBot虚拟黑客机器人已完成该漏洞检测能力的支****持**  
。VackBot   
以黑客攻  
击视角实现漏  
洞的真实  
性验证，可对企业内暴露的  
Ivanti Connect Secure（ICS）  
产品  
开  
展批量检测，识别是否受远程代码执行漏洞影响，输出风险判定结果与处置建议，帮助用户提前发现隐患，验证防护策略有效性，支撑安全团队快速完成资产风险排查工作。  
  
**漏洞概述**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NcM654OfmXKpowBciad0lOtKudCtYX87Lf2Yav4iaawzxIdLHpJ4T0xibAgztl42o2MgM8Gy34tKQJBKqoaLhkTkjIQlyviaFuTLHuXw5BMGnIA/640?wx_fmt=png&from=appmsg "")  
  
**漏洞原理**  
  
CVE-2026-104935 实际上是一条由认证绕过、远程代码执行和权限提升三个环节组成的漏洞利用链。  
  
补丁比对引发的"二次泄露"前端路由校验不足导致认证绕过  
  
Ivanti Connect Secure 在处理前端请求与后端脚本处理程序之间的路由关系时，存在校验不充分的问题。攻击者可以构造特殊请求，使请求最终到达的处理程序与系统允许该请求访问的处理程序不一致，从而绕过身份认证，访问本应要求登录的后端脚本处理逻辑。  
### 上传处理时序缺陷实现 RCE  
  
在获得认证绕过能力后，攻击者可进一步利用内部上传处理逻辑中**文件写入、校验与清理之间的时序缺陷**  
，在设备中写入脚本文件，从而以 Web 服务进程权限执行任意代码。该认证绕过与远程代码执行缺陷共同由 CVE-2026-104935 跟踪。  
### 共享配置数据处理不当导致提权  
  
攻击者获得代码执行能力后，还可继续利用设备上以 root 权限运行的后台维护服务对共享配置数据处理不当的缺陷提升权限，最终获得 root 权限。  
  
**漏洞复现**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/NcM654OfmXKzbaPDBADWdibicABRibiaPCKqVH8ibypxImf122mbqW3LRwYUz6O3erPt0tD8pjx2hMLJ8AjXBP7sbHKgqc6sicVOaH1nDHDgFjJgE/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NcM654OfmXKwLibF6l5wZk5tfEpkjsnrbqvH5uCHK8sxgZIfYLSmTl1u38h7iaX1Ld7ibfsHuHE2wId4oMg8QRDqxmv9TQJuJS6Vu3m4RTrLYU/640?wx_fmt=png&from=appmsg "")  
  
**破坏性分析**  
  
该漏  
洞的 CVSS 评分  
估算为  
 8.6  
，属于  
严重  
级别  
，其破坏性体现在以下几个维度  
：  
  
零门槛利用  
：  
无需任何凭据即可触发，攻击门槛极低  
。  
  
**边界设备失陷**  
：  
ICS 通常部署在网络边界，一旦失陷，攻击者可直接进入内网，进行横向移动、凭据窃取和数据外泄  
。  
  
**IoC 极少**  
：  
Ivanti 官方承认可用于检测的入侵指标非常有限，传统安全设备难以有效拦截  
。  
  
**补丁比对加速利用：攻击者通过 Patch Diffing 快速获取利用方法，在补丁全面覆盖前发动攻击，防御窗口极短**  
。  
  
**修复建议**  
  
建  
议升级至  
对应分支的安全  
版本或更高版本：  
  
**22.8 分支**  
：  
升级至 **22.8R2.5**  
2  
。  
  
25.1  
.1   
分支  
：  
升级至  
 **25.1.1.2****。**  
  
2  
5.1.3 分支  
：  
升级至 **25.1.3.0**  
 或 **25.1.3.1。**  
  
升级前，Ivanti 强烈建议运行**系统完整性检查工具（ICT）**  
，确认设备未被篡改。若 ICT 扫描发现异常，需先执行  
出厂重置  
，再升级至修复版本   
。  
  
![图片](https://mmbiz.qpic.cn/mmbiz/cZV2hRpuAPgyGRhyoqbTupN7lM2NSVJqkaFQzA59F6kiblIQsL175lxIVZbSLrFDFicibxXiaXpXmAGkrGNSib76Ylw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=3 "")  
  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/asLg7via5ibAkf1mRkpS4IuZibZE5eeC0t8nibIZBfZEekibOEZVWyf9jHzIVvT2sTzKS1OtZzSBErxJUZXD1AwAAWw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=4 "")  
  
往期回顾  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/icjDF5uGXY5ibE0P0Mtzns3KNb5hsCIKPfMIRultHDbmzgJcDaibI4wNKM6ZloyGRtRovyXtVdv3SuuVOcmA8gn8A/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=5 "")  
  
# AI红客机器人重磅更新，全面覆盖OpenClaw安全检测  
# 墨云科技AI红客机器人亮相香港世界青年科学大会  
# Black Hat Asia 2026 | 墨云AI红客团队闪耀亮相  
  
  
让网络攻防更智能  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/CD1iaLIMEhibPv9rc3gdLj3g6fiaAcCZqIicylIMVKlbvd5ic5usJ2oia9cTgavs6BwQpEEYbfglc82kCJ0Qic3OHMEaw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=6 "")  
  
  
  
点击  
**转发**  
  
分享给小伙伴  
  
  
