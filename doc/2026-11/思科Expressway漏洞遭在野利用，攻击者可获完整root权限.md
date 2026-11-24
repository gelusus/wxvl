#  思科Expressway漏洞遭在野利用，攻击者可获完整root权限  
 FreeBuf   2026-11-24 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX2gCnmcYKCFoqW1OwL0lGzb0vG3rjJmpxwQeJyQzpzdhz2ViaeY3PWLia07icqhO5E8wTOl109wRIAfLBE6103w9tIaPNfSSVAlHY/640?wx_fmt=gif "")  
  
  
![思科Expressway漏洞遭在野利用](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3THH5bcpJxkiaibhBHsxTQU1YoDS7oLM1yFUFsbQMy6PxNz6uxndB3L9PQb5b6dOSft4SrxmvtQTtribYvBqQJFe1reyD82PykXU/640?wx_fmt=png "")  
  
  
攻击者正活跃利用CVE-2026-76466漏洞。该漏洞是思科Expressway的严重漏洞，远程攻击者可借此以root权限执行操作系统命令。  
  
  
2026年11月20日，美国网络安全和基础设施安全局（CISA）在确认该漏洞存在活跃利用证据后，将其列入已知被利用漏洞目录。思科表示，目前该漏洞没有临时缓解方案，敦促用户尽快升级。  
  
  
思科最早于10月21日披露该漏洞，给出的严重程度评分为9.8/10。据Cyber Security News此前对该漏洞的报道，在CISA确认漏洞被利用前，思科就已发布修复版本。美国联邦机构的补丁修复截止日期为11月23日。CISA在漏洞目录条目中要求，各联邦机构需按规定检查受影响设备，排查是否存在失陷痕迹。  
  
  
Part  
01  
  
利用漏洞可获取root权限  
  
仅集群部署设备受影响  
  
  
该漏洞存在于Expressway集群设备使用的数据库服务中。根据思科安全公告，漏洞成因有两点：一是持久化Cookie的校验逻辑存在缺陷，二是集群开启TLS验证的情况下，设备仍未对客户端证书进行检查。  
  
  
攻击者只要获取到所需的Cookie，且能通过网络访问到受影响设备，无需正常登录即可执行命令。这些命令以root身份运行，拥有底层操作系统的最高访问权限。  
  
  
思科Expressway Core（Expressway-C）和Expressway Edge（Expressway-E）均受影响，无论部署形式是虚拟机还是物理设备。需要注意的是，该漏洞的触发有明确前提：设备必须运行受影响版本的软件，且已加入集群。这一点在资产排查时非常重要：仅靠产品名称和软件版本，无法判定设备是否存在暴露风险。  
  
  
风险不止于边界暴露的设备。据Cyber Security News报道的一起案例，F6的研究人员发现，某机构两台面向互联网的Expressway设备被攻击者攻陷后，攻击者以其中一台为跳板，进一步渗透到两台内网集群节点。因此，安全团队需要对受影响集群的所有内网节点进行排查，即便这些节点没有直接暴露在互联网上。  
  
  
Part  
02  
  
官方已发布修复版本  
  
  
思科明确，运行X15.5.3及更早X15版本的用户，首个修复版本为X15.5.20。运行X14及更早版本的用户，必须迁移至已修复的版本。本次更新还包含思科10月发布的Expressway安全加固补丁，可修复其他多个安全漏洞。思科对Jane Street的Stefan Aleksic上报该root权限漏洞表示致谢。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0uXvkAWrACKhwib3LG1ib1ibibxzIgGGd3hwdm7NoncsuXzHb3sgWq2998NJgt6lEHTSuORQW3smhB2oCuJvxGbI6QvocUS9ZpRdg/640?wx_fmt=png&from=appmsg "")  
  
  
管理员完成升级后，需要逐一验证所有集群节点的已安装版本。如果受漏洞影响的设备曾暴露在互联网上，需要妥善留存现有日志，排查暴露设备及其内网对等节点的历史活动。补丁仅能修复漏洞本身，无法确认设备此前是否已遭攻击者利用。  
  
  
参考来源：  
  
Hackers Actively Exploiting Cisco Expressway Vulnerability to Gain Full Root Access  
  
https://cybersecuritynews.com/cisco-expressway-vulnerability-exploited/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
