#  论坛卖家叫卖VMware vCenter 0Day利用工具，售价3万美元未经验证  
 FreeBuf   2026-11-23 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX2ayMdmn6tvrhvdwtfJAXfU40k2pLYrwcyuXjWRDsicMyptDmaDPAy29QuXUm2fwQpOGwNQ5ejMS90pIbaJIOqftpuqLibX5O0no/640?wx_fmt=gif "")  
  
  
![VMware vCenter 0Day利用工具](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX3mV7rltM7elw8632MT9b804C9CUkBtxXpeav5ayppoyUPkUYOslAb4hw6fibBmfIZNI0DRuPIo9uKx2vyakdUvic5zlPwYicOotY/640?wx_fmt=png "")  
  
  
网名为Matilda的论坛卖家正以3万美元的价格，出售一款针对VMware vCenter Server严重漏洞CVE-2026-123890的漏洞利用工具。  
  
  
Dark Web Informer分享的截图显示，卖家声称该工具可以新建vCenter管理员账号。该售卖帖发布于2026年11月19日博通发布漏洞补丁之后。目前这款工具宣称的功能和兼容性尚未得到独立验证。  
  
  
![卖家售卖帖截图](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0hic9fMjL7UicicAibcBRpytNwEpIvsuqZRmYaOAylNeBmMJ5V7BLMUZzSW1yic4f4OC3j2V7t9amjpHt6enGDjPvZjDQnG8ib0kIr8/640?wx_fmt=png "")  
  
  
卖家介绍，这款工具是Python编写的程序，分别包含漏洞检测、漏洞利用两个独立功能模块。  
  
  
售卖帖称，工具支持vCenter 7.0到9.1的所有受影响版本，还可为意向买家提供演示。不过，发布售卖帖不代表工具已经通过实测，目前也没有证据表明卖家曾使用该工具攻击过任何机构。  
  
  
Part  
01  
  
补丁发布前已现vCenter在野攻击  
  
  
博通发布的VMSA-2026-0009公告将该漏洞评为严重级，CVSS评分9.8（满分10）。Cyber Security News此前报道这一VMware vCenter认证绕过漏洞时提到，漏洞位于VMware目录服务中，该组件为vCenter的单点登录系统提供支持。  
  
  
攻击者只要能通过网络访问受影响的服务器，无需有效凭证即可绕过登录验证。vCenter是集中管理VMware主机与虚拟机的核心平台，因此这一漏洞的风险极高。  
  
  
谷歌威胁情报组（Google Threat Intelligence Group）与Mandiant的溯源显示，早在2026年9月初补丁尚未发布时，就已有攻击者利用该漏洞发起攻击。Cyber Security News此前报道这起vCenter 0Day攻击时提到，疑似与中国有关联的攻击组织ARMORY CASTLE曾使用另一款名为BONESAW的工具发起攻击。  
  
  
这些已被记录的在野攻击，正是该漏洞被定性为0Day的原因。而本次论坛上售卖的漏洞利用工具，针对的是已经公开披露并发布补丁的漏洞。目前没有证据表明卖家Matilda与ARMORY CASTLE组织存在关联。  
  
  
Part  
02  
  
博通发布漏洞修复补丁  
  
  
博通已在vCenter 9.1.1.0200、9.0.2.0200、8.0 Update 3m版本中修复该漏洞。使用VMware Cloud Foundation 5.x版本的用户，需按照厂商指引升级至5.2.5版本。  
  
  
公告显示，vCenter 7.0版本暂无公开修复补丁，签订了扩展支持合同的用户可联系博通技术支持获取修复。管理员需参照厂商发布的响应矩阵，确认自身部署环境对应的正确更新版本。  
  
  
美国国家漏洞数据库（NVD）的条目显示，CISA已于11月20日将该漏洞纳入已知被利用漏洞目录。博通未公布任何临时规避方案，因此安全团队无需等待本次论坛售卖信息的核实结果，应尽快部署对应更新。  
  
  
管理员还应排查vCenter登录记录与管理员账号变更情况，查看是否存在异常活动，重点检查补丁安装前可从非信任网络访问的服务器。更新补丁可以封堵漏洞，但无法确认补丁安装前是否已有攻击者利用该漏洞入侵过系统。  
  
  
参考来源：  
  
Hackers Allegedly Claiming Exploit for VMware vCenter 0-day Vulnerability  
  
https://cybersecuritynews.com/alleged-vmware-vcenter-0-day-exploit/  
  
###   
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
