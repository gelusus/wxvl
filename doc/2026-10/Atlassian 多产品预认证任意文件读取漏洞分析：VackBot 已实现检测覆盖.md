#  Atlassian 多产品预认证任意文件读取漏洞分析：VackBot 已实现检测覆盖  
 墨云安全   2026-10-14 08:10  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/7lCiaSMMkhia4WIkRNZHTwq8jJicy27jdbWa7ED26252RGmSPRE0rmHQsgZ6ZoichVyFNlvhLelZS09a194B9dyoAQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
点击↑  
**蓝字**  
  
关注  
**墨云安全**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC64Jegp0lgLDGafrdvU8MV80FwNQPMbnsrL7uZZicQcmtDkwNu5wflcYicoJ1OSoKLFpcH2wc15vJxw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1 "")  
  
Atlassian 多款 Data Center 产品存在一个严重预认证任意文件读取漏洞（CVE-2026-21589）。该漏洞源于共享库   
atlassian-plugins-webresource  
 的 Web 资源路径处理逻辑缺陷，攻击者利用   
::  
 双冒号序列绕过路径规范化，无需任何认证即可读取 Web 应用根目录下的敏感文件。  
  
针对该高危漏洞，**墨云 VackBot虚拟黑客机器人已完成该漏洞检测能力的支****持**  
。VackBot   
以黑客攻  
击视角实现漏  
洞的真实  
性验证，可对企业内暴露的 Atlassian 多款 Data Center产品  
开  
展批量检测，识别是否受远程代码执行漏洞影响，输出风险判定结果与处置建议，帮助用户提前发现隐患，验证防护策略有效性，支撑安全团队快速完成资产风险排查工作。  
  
**漏洞概述**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NcM654OfmXIZKaZWjEfsp3hicX5uYFxDtKI1qTNk5zMYJUvV65TWGxQGzCFLjw1zPQM8qeoibticTE87qiaBQLy8xkAA8cmVQUC3hAQgMrq1yok/640?wx_fmt=png&from=appmsg "")  
  
**漏洞原理**  
  
双冒号路径遍历：非常规分隔符的"隐形通道"  
  
路径遍历漏洞中，../、;、. 等字符是常见的利用载体，但本次漏洞的特殊之处在于——双冒号（::）也能被识别为路径分隔符。  
  
漏洞位于 Atlassian 的 Web 资源处理逻辑中。  
  
当程序处理类似以下路径时：  
  
..::..::..::..::WEB-INF::web.xml  
  
程序会将其转换为：  
  
../../../../WEB-INF/web.xml  
  
从而穿越目录层级，访问本无权读取的文件。  
  
该漏洞的触发点位于共享库   
atlassian-plugins-webresource  
，影响所有引用该库的 Atlassian Data Center 产品。  
  
**漏洞复现**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/NcM654OfmXLYWgicl0RHx4IPydl6ibWb5iaoyicFtbLxJ3bp9picnOL1x5icyiaHpT561s1FtQlCKLGwgnJNjt2AzDpCpqQn6C5iaFfO5EQXAqPuuH0/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/NcM654OfmXJzNytw9Bym4b2c5XP37rjTdungszsia5hStE8vmYCx21rj3B0ecB8ehwQ1cicxDmTDw9T2icBStUdNWoZEVKWPOaP42rosQzq23E/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/NcM654OfmXKDuscMEdPTCA5xw4PdeVODHuPVHbPwUoCaK1hshFSsBbGf3ZkucK7r1A0ETfKd0OsS1nbuQytrFdibGSYNHWpiaegfT4KGS2VoU/640?wx_fmt=jpeg&from=appmsg "")  
  
**破坏性分析**  
  
该漏洞的 CVSS 评分估算为   
9.3  
，属于  
严重  
级别  
，其破坏性体现在以下几个维度  
：  
  
零门槛利用  
：  
无需任何认证，单条 HTTP 请求即可完成利用，攻击门槛极低。  
  
**敏感信息全面泄露**  
：  
可读取 WEB-INF  
 目录下的配置文件、JDBC 连接串、API Token、私钥、Crowd 凭据等敏感数据。  
  
**管理员权限接管**  
：  
在 Jira + Crowd 集成场景下，攻击者可通过读取 crowd.properties  
 获取凭据，创建管理员账号，完全接管 Jira 实例。  
  
**软件供应链攻击跳板**  
：  
Bitbucket Data Center 受影响，攻击者可通过读取配置文件获取 CI/CD 凭据，进一步入侵构建流水线，污染下游所有依赖该平台的软件交付环节。  
  
**在野利用态势**  
  
PoC 公开后约 2 小时内即出现活跃利用，攻击者已迅速将矛头指向未修复的公网实例。  
  
**修复建议**  
  
官方已发布  
安全补丁，请及时  
更新至最新版本：  
  
Jira Software DC：  
升级至   
9.12.40 / 10.3.26 / 11.3.12。  
  
Jira Service Management DC：  
升级至  
 5.12.40 / 10.3.26 / 11.3.12。  
  
Confluence DC：  
升级至   
9.2.26 / 10.2.19。  
  
Bitbucket DC：  
升级至  
 9.4.26 / 10.2.8 / 10.5.1。  
  
Bamboo DC：  
升级至  
 10.2.24 / 12.1.12。  
  
Crowd DC：  
升级至   
6.3.7 / 7.0.3 / 7.1.7 / 7.2.4。  
  
Crucible / Fisheye：  
升级至   
4.9.15。  
  
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
  
  
