#  WordPress升级流程曝免登录RCE漏洞，24小时遭超6万次攻击  
 FreeBuf   2026-11-02 10:04  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX1meUuuhGE4Ddpd9K2DFSibkiaJU4h4BT3QFhDPdTpPVHfTb835icJTD5eKRH2KU4d6ckic3WtQwoKVvGpkFcEUqibrUcCQSGEdJOEo/640?wx_fmt=gif "")  
  
  
![WordPress RCE漏洞遭大规模攻击](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3GEoSU7Re2EyrKTDYjhm36Nmue4sMo7GiciaE6kuwQLfib2U9puGNBCRe3eouD0Ye2zC2YNCM6Ut6WubQQs21Kj8xtUyrICw3DFQ/640?wx_fmt=png "")  
  
  
攻击者正瞄准WordPress核心的一处远程代码执行（RCE）漏洞，站点在软件更新过程中可能因此被入侵。根据Wordfence提供的威胁情报快照，相关防护规则24小时内共拦截60471次攻击尝试。  
  
  
上述数据仅统计被拦截的攻击尝试，不代表已确认的站点入侵事件，也无法反映实际遭受攻击的独立站点总数。  
  
  
WordPress于2026年10月28日发布7.1.4版本，共修复22个安全漏洞，除本次的代码执行漏洞外，还覆盖存储型跨站脚本、信息泄露、权限配置不当等多类风险。  
  
  
WordPress官方公告敦促管理员立即完成更新，并向报告该升级流程漏洞的研究员Umut Bayram致谢。  
  
  
Part  
01  
  
漏洞源于更新流程竞态条件  
  
  
该漏洞属于竞态条件类漏洞，攻击者必须精准命中临时安装流程的特定时间窗口才能利用成功。与需要窃取密码或持有站点账号才能利用的漏洞不同，攻击者无需登录就可以发起攻击。不过，漏洞成功利用的前提是目标站点正处于完整的WordPress核心升级过程中。升级过程中，WordPress会将更新文件解压到临时暂存目录。这一阶段，由于临时文件夹名称可被预测、安装文件可公开访问，会短暂暴露一个独立的WordPress安装程序。  
  
  
攻击者一旦访问到该安装程序，就可以将其连接到自己控制的数据库。完成这一步后，攻击者就能在服务器上执行任意代码。  
  
  
据Cybersecuritynews.com此前对WordPress 7.1.4安全更新的报道，本次补丁主要做了三处调整：为临时文件夹名称添加16位随机字符、恢复index.php的防护机制、禁止从工作目录复制wp-config.php文件。上述修复针对的是核心升级暂存流程的暴露问题，并非第三方插件漏洞。  
  
  
这类漏洞的攻击窗口通常仅持续数秒，利用难度较高但并非完全无法成功。攻击者可以通过发送大量重复请求，提升命中站点更新文件暴露窗口的概率。Wordfence已监测到补丁发布后出现多起漏洞利用尝试，说明尽管有严格的时间条件限制，攻击者仍在积极测试该漏洞的可行性。Cybersecuritynews.com此前也报道过这批针对WordPress的活跃利用尝试。需要明确区分防火墙拦截的攻击请求与成功的代码执行事件：高请求量仅说明攻击活动活跃，并不代表攻击者已经成功入侵了同等数量的网站。  
  
  
根据Wordfence发布的公告，其Premium、Care、Response订阅客户在10月28日（即版本发布约3小时后）就已收到对应的防火墙防护规则。免费用户则按照标准的30天延迟规则，将于11月27日收到该规则。管理员不应等待防护规则推送，应优先安装官方补丁。  
  
  
Part  
02  
  
及时安装官方安全补丁  
  
  
站点所有者应登录后台，进入更新页面，确认站点已安装WordPress 7.1.4版本。使用旧分支版本的站点，需安装对应分支的安全更新包。官方版本文档列出了所有已修复该漏洞的版本，包括7.0.8、6.9.11，向后移植补丁的覆盖范围最低至4.7.39。WordPress 4.6及更早版本已停止接收安全更新。  
  
  
符合支持条件的站点会通过后台自动更新推送对应安全版本，但管理员不能默认站点已受保护，必须手动确认更新完成。官方文档明确说明，只有最新版WordPress会获得持续主动支持。旧分支的修复补丁仅提供过渡更新路径，不能作为推迟版本维护的理由。  
  
  
管理员还可以限制公众对wp-content/upgrade/目录的访问权限，作为额外防护手段。仅靠插件防火墙可能无法覆盖解压后的安装程序，因为该程序的加载流程独立于主站点。安装核心安全更新，仍是修复该升级流程漏洞的最核心措施。  
  
  
参考来源：  
  
Over 60,000 Attacks Target WordPress RCE Vulnerability Within 24 Hours  
  
https://cybersecuritynews.com/60000-attacks-targeting-wordpress/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
