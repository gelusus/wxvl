#  黑客利用思杰NetScaler 0Day漏洞，部署Platypus远控工具  
 FreeBuf   2026-10-14 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX30lE3Z81ILfZgxPibj6aY5rmlcAIDpNpWZtRicic6msicicEeCYB1x3INg60dXib2dldCOAp5ZTc68iaOMtDwnTm4vESOtRIWbZ9CwYI/640?wx_fmt=gif "")  
  
  
![配图](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX0fibzg7iaiaCVyL5XwibPic9rxXMXB5Z7qyadzfBEBRRq866ETGKQhe4S3e8GTOicBIYj6OQz33XdzAibo3BDUqxYXFR8vdwQpcErQRU/640?wx_fmt=jpeg "")  
  
  
威胁行为者正利用思杰（Citrix）NetScaler的两个高危0Day漏洞，在暴露在公网的ADC和网关设备上部署修改版Platypus远程访问工具。  
  
  
这一攻击活动表明，攻击者从获取初始权限到持久控制边缘设备的速度极快。这类设备通常部署在企业内网与公网的边界位置。  
  
  
本次攻击针对CVE-2026-88771和CVE-2026-88772两个思杰NetScaler ADC与网关的在野利用漏洞。两个漏洞的CVSS评分均为9.5，无需有效用户账户即可实现远程代码执行。  
  
  
思杰已发布受影响产品的紧急补丁，CISA也已将这两个漏洞纳入其已知被利用漏洞目录。  
  
  
Binary Defense的研究人员在协助客户升级设备时，对两台出现被利用迹象的NetScaler设备开展了事件调查。  
  
  
其中一台设备存在漏洞利用尝试痕迹，另一台则有证据显示，攻击者已投递并大概率执行了名为Platypus的定制化远程管理植入物。  
  
  
Platypus是一款Go语言编写的开源远程管理框架。据研究人员分析，攻击者针对NetScaler设备运行的FreeBSD操作系统修改并编译了该工具。他们随后将恶意软件伪装成Perl相关文件，混入设备的正常文件中。  
  
  
研究人员观测到，攻击者使用的安装器通过curl命令管道传输至shell。该命令会直接拉取恶意工具并启动运行。  
  
  
恶意载荷存储在NetScaler管理员常见的路径下，包括/var/tmp/、/netscaler.local/以及/var/core/。其中一个样本使用文件名为.ns_09343.pl，但该文件实际是ELF二进制文件，并非合法的Perl脚本。  
  
  
Part  
01  
  
补丁无法清除留存后门  
  
漏洞延迟执行加大检测难度  
  
  
这种部署方式的风险在于，即使企业为存在漏洞的设备打上补丁，攻击者获取的立足点仍可能留存。  
  
  
补丁只能阻止漏洞被进一步利用，但无法自动清除设备上已存在的Webshell、后门、窃取的凭证、被篡改的配置文件或远程访问植入物。  
  
  
此前已有报告显示，攻击者利用该命令注入漏洞安装Webshell、收集NetScaler配置数据。  
  
  
CVE-2026-88771是输入验证不当漏洞，未授权攻击者可在存在漏洞的NetScaler设备上执行命令。  
  
  
![攻击链（来源：Binary Defense）](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX1TXtbN7XEhgfTclPiaichYDZVNic0V5mhcPmuduRbN3fzeeAcpXYwYj6V2XH7meBTCE38ptGPl24rKVSJbeLHKQ6QvX4mw3Dia2cI/640?wx_fmt=jpeg "")  
  
  
Binary Defense研究人员发现，攻击者可在用户名等认证相关的构造输入中嵌入Shell命令。设备会将恶意输入写入NetScaler日志，存在漏洞的错误处理进程后续会执行注入的命令。  
  
  
第二个漏洞CVE-2026-88772是DTLS内存溢出漏洞，影响开启DTLS功能的NetScaler VPN虚拟服务器，而多数部署场景中DTLS为默认开启配置。  
  
  
攻击者成功利用该漏洞后，可在设备上执行代码。该漏洞也可能导致受影响服务崩溃，触发拒绝服务状态。  
  
  
派拓网络（Palo Alto Networks）Unit 42证实，攻击者正在野利用NetScaler ADC和网关系统中的这两个漏洞。  
  
  
这个命令注入漏洞尤其值得警惕，因为注入的命令不一定会立即执行。延迟触发的维护进程可能在后续运行恶意命令，导致防御方很难将可疑登录请求与后续攻击活动关联起来。  
  
  
Part  
02  
  
定制版Platypus功能完备  
  
支持横向探测与持久化  
  
  
研究人员回收的Platypus样本并非公开项目的标准版本。据悉，这个定制版本新增了提权、持久化、痕迹清理、时间戳篡改等模块。  
  
  
![DTLS重组（来源：Binary Defense）](https://mmbiz.qpic.cn/mmbiz_jpg/icBE3OpK1IX1EfIP7cfaiaAQb5Jc8CSq9BueCXFfGMP2U7xhXY0bmiahX07JZN1x8GNHc0icKNqcGG6dqjFIHgPK0LrpBXAYo1MYLIOg0sLXIPI/640?wx_fmt=jpeg "")  
  
  
该恶意软件功能完备，支持通过远程终端执行命令、读写文件、自我更新、加载插件。同时，它还能通过双向TLS与C2服务器通信。  
  
  
恶意软件还内置本地对等节点发现功能，通过_platypus-mesh_tcp mDNS服务实现。该功能可帮助攻击者识别同一网络内的其他失陷系统，或对多台失陷系统进行关联。  
  
  
研究人员将本次攻击活动与域名entretiensol[.]com关联，该域名被用作载荷投递与控制服务器。另有多个域名与本次活动共享基础设施或证书特征，包括white-guard[.]pro、gurerasfalt[.]com、garyvard[.]com以及hickoryusedauto[.]com。  
  
  
Part  
03  
  
需立即升级修复漏洞  
  
  
企业需立即将存在漏洞的NetScaler ADC和网关系统升级至修复版本。思杰已公布两个0Day漏洞的修复版本为14.1-73.37和13.1-64.23，包括对应的FIPS和NDcPP版本。  
  
  
安全团队不能将打补丁视为唯一处置步骤。需要检查ns.log、sh.log、bash.log、httpaccess.log、httperror.log等日志，排查可疑认证字符串、异常Shell命令、curl | sh执行痕迹、扩展名为.pl但实际为二进制的文件，以及NetScaler设备访问未知互联网主机的出站流量。  
  
  
所有补丁修复前暴露在公网的设备，在管理员完成取证检查、轮换所有可访问凭证、吊销活跃会话、检查高可用部署中的双节点之前，都应视为存在失陷风险。  
  
  
参考来源：  
  
Hackers Exploit Citrix NetScaler 0-Days to Deploy Platypus Remote Access Tool  
  
https://cybersecuritynews.com/hackers-exploit-citrix-netscaler-0-days/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
