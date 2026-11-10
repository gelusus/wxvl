#  D-Link：注意缓解DGS-3130系列交换机中的多个0day  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-11-09 05:51  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**D-Link****分享了影响****DGS-3130****系列可管理千兆堆叠交换机的多个未修复漏洞的缓解措施。**  
  
这些交换机型号为企业提供多千兆上行链路，使企业网络管理员能够消除数据瓶颈，主要面向中小型企业和城域以太网部署。虽然  
 D-Link   
未分享这些漏洞的细节，也尚未公开披露便于追踪的  
 CVE   
编号，但该公司表示  
 Spectrum Community Solutions   
公司的网络工程师  
 Giresh Kumar Enugu   
在  
 D-Link   
发现协议  
 (DDP)   
身份验证、访问控制和信息处理中发现了这些漏洞。成功利用这些漏洞可能允许攻击者获取敏感设备数据、进行未经授权的凭据  
/  
配置更改、泄露  
 SNMP   
团体字符串，或导致服务中断和设备重置。  
  
这些  
 0day   
漏洞影响多个  
 DGS-3130-30TS   
固件版本，该公司仍在调查对其它  
 D-Link   
交换平台和固件安全机制（包括  
 SSH   
服务安全配置、底层操作系统凭据的保护措施，以及固件加密  
/  
嵌入式加密密钥的处理）的影响。该公司表示，“所报告的漏洞涉及  
 D-Link DDP  
以及设备固件安全架构的某些组件。根据安全研究人员的说法，成功利用某些漏洞可能允许能够访问受影响网络的攻击者获取敏感设备信息、执行未经授权的管理操作、修改设备配置或中断网络服务。”  
  
在发布新固件解决这些漏洞之前，  
D-Link   
要求管理员禁用  
 DDP   
以消除主要已知攻击途径，移除不必要或默认的  
 SNMP   
团体字符串，并将  
 SSH  
、  
Web UI   
和  
 SNMP   
的可访问性仅限制在隔离的管理网络和指定的管理员主机。  
D-Link   
还建议管理员维护离线交换机配置备份，并跟踪设备日志以发现意外配置更改、无故重启或恢复出厂设置。  
  
9   
月，  
D-Link   
提醒客户注意  
CVSS  
满分漏洞  
CVE-2026-86296  
，该漏洞已有公开的概念验证利用代码且无补丁，影响旧版  
 DIR-822A   
双频  
 Wi-Fi   
路由器。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[D-Link：遗留 DSL 路由器中存在已遭利用漏洞，速修复](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524812&idx=1&sn=f525d9edab2101534877ebaa069472f2&scene=21#wechat_redirect)  
  
  
[D-Link：注意 DIR-878 路由器中的多个RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524471&idx=1&sn=2c5a11dba130eea68bd2080f69e5410e&scene=21#wechat_redirect)  
  
  
[D-Link 决定不修复这个严重漏洞，影响6万多台 NAS 设备](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247521440&idx=2&sn=1ddad70ced916d3fad493805c8d04d62&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/d-link-warns-of-multiple-zero-days-in-dgs-3130-series-switches/  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
