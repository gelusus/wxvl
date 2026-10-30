#  黑客利用思科SD-WAN严重0Day漏洞，部署KeyDrift窃取加密密钥  
 FreeBuf   2026-10-30 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3vp4liaT1zr4IXaGBohAW3Z0RGXeIQ6dD380eibb7j5flo6cbsjEmD6f9AYyutCia2CNYe9criaZhoQtm4qRibJ4NYAm8RaUnicug3w/640?wx_fmt=gif "")  
  
![思科SD-WAN 0Day与KeyDrift](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1am2o2qjicAhEs07FJzlp61Zib9tKOWGnosAicYKroQfFial2snhGxvJAAmUiaknzU2afrzDPwbPUNbAV5q5TlVcnpZa0iaosKbHiaqM/640?wx_fmt=png "")  
  
  
某有国家背景的黑客组织利用思科SD-WAN最高严重等级0Day漏洞入侵网络控制器，植入定制化恶意软件框架KeyDrift。  
  
  
该漏洞编号为（CVE-2026-20127），CVSS评分10.0，远程攻击者无需密码即可绕过认证，直接获取管理员权限。  
  
  
KeyDrift会长期潜伏在思科SD-WAN网络的控制系统中，既可以复制企业站点间的流量加密密钥，也具备后门功能。  
  
  
目前已确认的受害目标包括航天卫星服务机构、国防航空航天企业以及政策研究机构。思科Talos研究人员发现了该恶意软件，并于2026年10月28日发布了相关分析报告。  
  
  
Talos将该攻击组织的追踪编号定为UAT-11473，认定其为技术高度成熟的APT组织。相关攻击活动为历史事件，但Talos未披露具体入侵时间，也未确认被盗密钥是否已被外传。  
  
  
Part  
01  
  
0Day无需认证接管控制器  
  
  
该漏洞存在于Catalyst SD-WAN控制器、管理器和验证器的对等认证环节。攻击者发送构造的请求，就能登录为高权限内部用户，获得NETCONF操作权限，篡改网络配置。  
  
  
思科表示该漏洞没有临时规避方案，安装补丁是唯一修复手段。后续有研究人员公开了该漏洞的PoC利用代码，进一步提升了未打补丁系统的风险。  
  
  
入侵得手后，攻击者会修改runit服务路径下的启动脚本，让恶意软件随系统正常服务启动。名为conf_sshd的脚本会查找运行中的vdaemon进程，再通过rsshd工具将主模块注入每个vdaemon进程。  
  
  
![conf_sshd控制流逻辑（来源：思科Talos）](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX3EJpt8zds9zwdIGZesAPgiamfDqic0d7mtQoSH2uCm3X0iba5gQrGHlf9cruesgw81k7ibkSyfib0PWQOsot392DViaNKpFFib02xgWE/640?wx_fmt=png "")  
  
  
注入的主模块名为PolyFlame，仅存在于内存中，属于后门程序，支持执行命令、读写文件操作，还可以向其他受感染机器转发指令。其中一个变种会伪装成合法的aaamgr服务，通过反调试检测隐藏自身。  
  
  
第二个模块名为LittleFilch，专门负责窃取密钥。它会Hook vdaemon进程内的SSL_read函数，在控制消息完成解密后读取内容。  
  
  
针对预设列表中的边缘路由器，该模块会抓取IP地址，以及一段32字节的字段——Talos判断这是IPsec流量使用的AES-256密钥材料。  
  
  
LittleFilch会用RSA加密窃取到的所有数据，存储到本地文件中，还可通过UDP协议将数据发送到配置文件指定的地址，读取完配置后会立即将其删除。  
  
  
Part  
02  
  
UAT-11473关联Volt Typhoon  
  
  
Talos高置信度判断，UAT-11473使用的基础设施与"Silent Train"高度重合，后者是与Volt Typhoon关联的秘密访问网络。  
  
  
Volt Typhoon此前就有劫持路由器的攻击记录，早前针对该组织瞄准思科路由器的活动分析显示，其惯用手法是将边缘设备转化为隐蔽中继点。  
  
  
思科早在2026年2月就修复了CVE-2026-20127漏洞，企业首先要确认所有控制器都运行在已打补丁的版本上。对应不同版本分支，首个修复版本号分别为20.9.8.2、20.12.6.1、20.15.4.2、20.18.2.1。  
  
  
![conf_sshd执行逻辑（来源：思科Talos）](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1AwqPkotrxibOrlJvmyEsRkUMBZXwNKxAEwKGl7dhW7UbHdd0TZicaZecAbyaiaBrj92Qo2ibvLcO4NkbEyqSbZCycHiaicCYD0cFnc/640?wx_fmt=png "")  
  
  
运行20.9以下版本的设备必须升级到仍受支持的版本。在完成升级前，需要将22、830端口的访问权限限制为已知控制器地址，但要注意这种方式无法清除已经植入的恶意软件。  
  
  
如果此前设备曾暴露在漏洞风险下，仅打补丁不足以消除风险。需要检查/var/log/auth.log中是否存在来自未知IP地址的"Accepted publickey for vmanage-admin"日志条目，同时排查下文列出的恶意文件。  
  
  
在提交思科TAC工单前，需要在每个控制组件上执行"request admin-tech"命令收集信息。防御方还需警惕新近披露的SD-WAN控制器漏洞（CVE-2026-20182），该漏洞同样针对vdaemon服务，且已出现在野利用。完整防御指引可参考思科官方安全公告。  
  
  
Part  
03  
  
入侵指标清单公布  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1u5gV973Phiat7Fu31wPARVLjwyFkNsAA6roBCibP6icDnDI7W5bNAUbnU8Gkib57MfAAzqK8ic0TucHpJvUJMbSadX9ibZ68ZMSXn4/640?wx_fmt=png&from=appmsg "")  
  
  
注：文中涉及的IP地址和域名均已做去活化处理（例如将.替换为[.]），防止意外解析或触发超链接。仅可在MISP、VirusTotal或内部SIEM等受控威胁情报平台内恢复原始指标。  
  
  
参考来源：  
  
Hackers Exploit Critical Cisco SD-WAN 0-Day Flaw to Deploy KeyDrift Malware and Steal Encryption Keys  
  
https://thehackernews.com/2026/10/hackers-exploit-critical-cisco-sd-wan-0.html  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
