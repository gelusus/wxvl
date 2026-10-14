#  Docker修复Docker Hub存续两年高危漏洞，可冒用他人身份推送镜像  
 FreeBuf   2026-10-14 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX33KA0EYufSlp8Zn7EDxCZAKiaDAMgQj4VZDbY4OkTyXXdZHyIcvVjQspibDxYfOE4iaWXLK2SCZZ9vA5wuoFVtGf0qqGnaibxLpMk/640?wx_fmt=gif "")  
  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icBE3OpK1IX3psddKIGNYJ06PHHLPX9xCz8KFHAia37eNevTXr0GxHq95jLDMWvxAHGy9rIS7Uwu6K12bafZ0PBibBRmicOZBjwkKZTDmLicdeMI/640?wx_fmt=jpeg "")  
  
  
Part  
01  
  
Docker严重漏洞可冒用用户身份  
  
  
Docker Hub近日修复了一处存在两年多的严重安全漏洞。攻击者可利用该漏洞冒用其他用户身份，在镜像仓库中执行推送、拉取或删除操作。Docker已于9月30日完成修复，目前尚未发现该漏洞遭到实际利用的证据。  
  
  
根据Docker于10月12日发布的安全公告，问题出在个人账号转换为组织账号的过程中。正常情况下，个人访问令牌（PAT）可代替密码用于镜像仓库身份认证。但存在漏洞时，账号转换操作会将原有PAT的所有权转移给新指定的组织所有者，而令牌创建者仍然可以继续使用该令牌。  
  
  
不过，这类权限仅限于镜像仓库操作，无法访问Docker Hub网页界面，也不能修改账号密码、邮箱地址或双因素认证设置，因此无法直接接管整个账号。  
  
  
该漏洞编号为CVE-2026-105445，被Docker评为严重级别。10月13日公布的CVE记录进一步确认，其CVSS 4.0评分达到9.4。  
  
  
漏洞自2024年8月26日起存在，直到今年9月30日才得到修复，持续时间超过两年。由于Docker已直接在Hub服务端完成修复，用户无需额外安装更新。  
  
  
Docker Hub不支持在两个账号之间直接转移仓库。如果用户希望转移仓库，需要先将个人账号转换为同名组织账号，再指定一个已有用户作为组织所有者。  
  
  
正常情况下，PAT的权限取决于所属账号及创建时设定的权限范围。但在漏洞存在期间，账号转换会修改令牌登记的所有者信息，却不会改变令牌密钥本身。  
  
  
攻击者正是可以利用这一机制，先在自己控制的账号中创建PAT，再将该账号转换为组织，并将其他用户指定为组织所有者。完成转换后，攻击者便能继续使用原有令牌，以被指定用户的身份操作镜像仓库。  
  
  
整个过程中，被指定为组织所有者的用户既不需要确认，也不会收到相关通知。然而，至少从2024年8月起，Docker的账号转换页面就一直提示，新所有者将收到通知邮件。  
  
  
在令牌原有权限范围内，攻击者可以拉取、推送或删除目标用户仓库中的镜像，也能操作目标用户有权访问的其他组织仓库。  
  
  
由于这一权限包含以合法用户身份发布镜像的能力，Docker将漏洞定为严重级别。今年3月至4月，攻击者就曾利用窃取的发布者凭证，向合法发布者自己的Docker Hub仓库推送恶意Trivy和KICS镜像，说明此类权限一旦被滥用，可能直接威胁软件供应链安全。  
  
  
此次漏洞由mptrdev.com的两名研究人员Peter Molnar和Krisztian Hozak发现，并命名为GhostPat。Docker收到报告后，不到4小时便完成了修复部署。  
  
  
在获得授权的测试环境中，两名研究人员曾两次成功复现漏洞。他们利用转移后的令牌，以目标用户身份完成镜像仓库认证，在目标命名空间下创建仓库并推送镜像，向目标用户所属组织发布镜像，同时成功拉取了目标用户私有仓库中的内容。  
  
  
Part  
02  
  
Docker全面排查存续期操作  
  
  
漏洞修复后，Docker对其存续期间的账号转换操作展开了全面排查，并将相关记录与镜像仓库日志、访问令牌数据进行交叉比对，重点检查曾在多个身份下使用的令牌。  
  
  
此次调查共覆盖约30.4万次通过转移令牌完成的镜像推送操作，未发现漏洞遭到实际利用的证据。  
  
  
从排查结果来看，相关推送操作的目标仓库均属于转换后的组织或被指定的组织所有者。绝大多数情况下，涉及的两个账号实际上由同一个人或同一团队控制。  
  
  
对于无法确认账号控制关系的情况，Docker进一步对高风险公开镜像开展了非运行态分析，最终没有发现恶意软件或镜像遭到篡改的迹象。  
  
  
不过，现有日志仍存在一定局限。Docker Hub的仓库日志只会记录令牌使用行为，不会记录令牌是否存在。因此，如果某个转移后的令牌从未被使用，就不会在日志中留下对应记录。  
  
  
基于这一限制，Docker将漏洞存续期间通过账号转换转移的所有令牌都纳入潜在受影响范围，无论这些令牌是否实际使用过。  
  
  
目前公开的排查结果主要针对镜像推送操作，尚未交代通过转移令牌执行的镜像拉取和删除操作是否经过同等范围的检查，其中也包括对私有仓库内容的访问。  
  
  
Part  
03  
  
Docker建议排查轮换访问令牌  
  
  
10月2日，Docker分别向收到转移令牌的用户及相关组织所有者发送了通知邮件。邮件按照令牌描述和创建日期，列出了需要关注的个人访问令牌。  
  
  
此次公告并未披露具体涉及多少用户和令牌，仅说明受影响用户数量有限。未收到通知邮件的用户不在此次漏洞影响范围内，无需采取额外操作。  
  
  
10月8日，Docker进一步停用了已经识别出的转移令牌，但对于所有者明确要求保留的令牌，则没有强制停用。  
  
  
Docker Compose代码检查工具compose-lint的开发者Todd Matens也收到了相关通知。10月2日，他分别以组织所有者和个人用户身份收到两封邮件。随后，他轮换了正在使用的两个持续集成（CI）令牌，并检查了已发布的42个镜像签名，将每个镜像逐一追溯至对应的CI构建记录。  
  
  
对于其他用户，Docker建议重新检查账号中的访问令牌，及时吊销无法识别或不再使用的令牌。组织内部的自动化流程则应优先使用组织访问令牌，而不是依赖个人PAT。  
  
  
用户可通过以下要点完成检查：  
  
- 打开账号设置，进入个人访问令牌页面，列表会显示每个令牌的创建时间和最后使用时间。  
  
  
- 研究人员总结了两类异常痕迹：一是出现并非自己创建的令牌（无论令牌名称是什么，因为令牌创建者可自行设置令牌描述）；二是账号切换器中出现自己不记得加入过的组织。  
  
  
- 组织访问令牌需要Docker Team或Docker Business订阅才能使用，由组织所有者、企业所有者或拥有对应权限的自定义角色管理，无法用于Docker Desktop或Image Access Management。  
  
  
- 持有上述订阅的组织所有者，可以在活动日志中查看历史推送记录，日志会记录推送的标签以及执行推送的令牌。查看日志的Docker Home页面会保留30天内的日志，更早的记录可通过Docker Hub API获取。  
  
  
Part  
04  
  
官方文档曾记录漏洞机制  
  
  
在漏洞存在的约25个月里，有17个月的时间Docker官方文档都明确记载了这种令牌转移行为。2025年4月22日的一次文档更新，还在Docker的账号转换页面添加了注释，说明指定用户为组织所有者“将确保与该用户账号关联的所有PAT都转移给组织所有者”。  
  
  
The Hacker News查阅Docker文档的公开历史记录发现，Docker在9月30日（即其公布的修复时间点后数分钟）就删除了这条注释。  
  
  
Docker表示，目前用户被指定为组织所有者前必须经过本人确认，指定完成后也会向该用户发送通知。Docker还会记录令牌所有权的每一次变更，同时排查其他会修改凭证所有者的功能。  
  
  
参考来源：  
  
Docker Fixes Critical Two-Year-Old Hub Flaw That Let One User Push Images as Another  
  
https://thehackernews.com/2026/10/docker-fixes-critical-two-year-old-hub.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
