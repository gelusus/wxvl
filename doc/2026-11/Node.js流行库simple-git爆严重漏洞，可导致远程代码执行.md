#  Node.js流行库simple-git爆严重漏洞，可导致远程代码执行  
 FreeBuf   2026-11-24 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3WiatdeXjSll60ABoWIwZxgFCJoZBKK63C3XWGGtWNibN1oSwekMfkg9rVWcy9uMx2Ud2Zh8Gqsu5vOpZOLlISu67A39XJoVag8/640?wx_fmt=gif "")  
  
  
![simple-git漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX2v6rIm71D0hqlE6YsicFj3uVmjPDG8OI2QBgaeN4R6S53sfB3AicVAD76CGIP72icnIGHoJG2kT6d0XQVIlsicAJkYO8DKepxfucs/640?wx_fmt=png "")  
  
  
simple-git是Node.js生态中广泛用于执行Git命令的流行库，近期被曝出存在一处严重漏洞，攻击者可借此在服务器上执行任意命令。该漏洞编号为CVE-2026-134815，所有将不可信输入传入Git参数的应用均会受到影响。  
  
  
美国国家漏洞数据库（NVD）记录显示，4.1.0之前版本的simple-git存在参数注入缺陷，对应CWE-88。简单来说，该库无法正确区分用户传入的文本与发送给Git的命令选项。为该漏洞分配CVE编号的VulnCheck，在CVSS 4.0评分体系中将其评为9.2分（严重级）。  
  
  
该库原本设有名为allowUnsafePack的安全防护机制，作用是拦截会导致Git执行外部程序的危险选项。在本次漏洞修复前，该防护仅对push操作拦截--exec选项，攻击者只需在其他Git操作中加入--exec参数即可绕过防护。  
  
  
Part  
01  
  
未校验输入传入函数  
  
可触发远程代码执行  
  
  
当应用允许外部输入传入listRemote()或raw()函数时，就会产生风险。攻击者一旦控制这些参数，就可以在ls-remote、archive、fetch-pack、send-pack等Git操作中插入--exec参数。Git会将该参数值识别为远程程序名称，直接在主机上运行。最终结果就是远程代码执行，攻击者的命令将拥有与Node.js进程相同的运行权限。  
  
  
根据CVSS向量描述，该漏洞利用无需登录权限，也不需要用户交互。但漏洞利用复杂度较高，且存在特定前置条件：只有存在漏洞的应用以特定方式配置时，攻击者才能成功利用。  
  
  
即便存在利用门槛，该漏洞对数据、系统完整性和可用性的影响评级均为高危。Upwind发布的研究报告，也描述了同样的allowUnsafePack防护绕过路径。  
  
  
Part  
02  
  
官方已发布修复补丁  
  
  
steveukx/git-js项目旗下共有两个包受漏洞影响，分别是4.1.0版本以下的simple-git，以及3.0.0版本以下的@simple-git/argv-parser。后者负责实现危险参数检测逻辑，因此两个包必须同步更新。  
  
  
维护者已于11月16日发布simple-git 4.1.0版本修复漏洞。根据版本说明，更新后的allowUnsafePack机制将对除rebase外的所有命令拦截--exec选项，同时新增allowConfig与strict配置项。本次更新还新增了git.cmd辅助方法，会将所有传入值视为不可信数据，禁止其添加命令选项或修改配置。维护者对18位提交CVE提案、参与修复方案设计的贡献者表示感谢，NVD则于11月22日正式公开该漏洞记录。  
  
  
这并非simple-git首次在参数处理逻辑上出现问题。2022年披露的CVE-2022-24433显示，攻击者可向fetch()函数注入Git选项实现命令执行。今年早些时候披露的CVE-2026-28292，则利用正则表达式缺少大小写不敏感匹配标记的缺陷，绕过了此前的修复方案。另一则安全公告CVE-2026-6951，还披露过一次不完整修复：当时的修复仅拦截了-c参数，却未覆盖--config参数。  
  
  
这类问题的规律十分明显：基于黑名单拦截危险选项的机制很难做到全覆盖，因为Git支持多种写法来表达同一个选项。目前simple-git在npm上的周下载量超过1200万次，覆盖大量开发项目。  
  
  
Part  
03  
  
及时升级并规范开发  
  
  
用户需将simple-git升级至4.1.0及以上版本，@simple-git/argv-parser升级至3.0.0及以上版本。升级完成后，应执行npm ls simple-git命令，排查其他依赖包间接引入的simple-git副本——很多项目会通过第三方依赖间接使用该库。  
  
  
开发者还应避免将原始用户输入直接传入listRemote()或raw()函数。更安全的做法是建立固定的允许列表，仅允许输入匹配列表内的仓库URL和分支名称。以低权限用户身份运行应用，也能额外增加一层防护。  
  
  
可执行系统命令的Node.js库一直是严重安全问题的高发区。此前systeminformation包就曾出现同类命令注入漏洞，成因同样是未校验的输入直接传入命令执行环境。  
  
  
开发团队还应对项目依赖保持严格管控。此前Keyv包被攻陷的事件就波及了数百万周下载量的项目，足以说明依赖风险的强传导性。  
  
  
开发者应使用锁文件固定依赖版本，在构建系统中执行npm ci命令，正式发布前仔细审核所有依赖更新，从流程上降低供应链攻击风险。  
  
  
参考来源：  
  
Critical Node.js simple-git Library Vulnerability Enables Remote Code Execution Attacks  
  
https://cybersecuritynews.com/node-js-simple-git-library-flaw/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
