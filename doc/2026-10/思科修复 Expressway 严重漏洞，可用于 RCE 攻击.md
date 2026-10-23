#  思科修复 Expressway 严重漏洞，可用于 RCE 攻击  
Abinaya
                    Abinaya  代码卫士   2026-10-23 09:11  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**思科已发布****Cisco Expressway****中严重漏洞（****CVE-2026-76466****，****CVSS****评分****9.8****）的安全更新，该漏洞可导致远程攻击者以****root****权限运行操作系统命令。**  
  
该漏洞可导致远程攻击者影响配置为集群的易受攻击的  
 Expressway   
设备，影响  
 Cisco Expressway Core  
（  
Expressway-C  
）和  
 Cisco Expressway Edge  
（  
Expressway-E  
）。在启用集群的情况下运行易受攻击软件的虚拟部署和硬件设备均受影响。成功利用该漏洞可使攻击者控制底层操作系统。  
  
思科在安全公告中提到，该弱点位于数据库服务中，涉及持久性  
 cookie  
。思科还发现，当集群启用  
 TLS Verify Mode   
时，客户端证书未得到正确检查。这意味着启用该设置并不能提供预期的证书验证。  
  
拥有所需  
 cookie   
且具备受影响设备网络访问权限的攻击者，可以连接到其数据库服务并调用特定的远程过程调用命令。这些调用可能允许攻击者以  
 root   
权限运行命令。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVC7dZJSeQQ5HnpW9dVPAQuzdf8bLibK8EibC2ve3RoJ1FHca1uYa8j2BxtcAgianm8Ho9WHsmuvWIWeHs5yQcaFAOF0qrjqXZLCQ/640?wx_fmt=gif&from=appmsg "")  
  
**思科 Expressway 漏洞**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfWjan5mKKfUspqHcDNmOfABJBCmqjIMrCUl7FQqcPRX4YTxxqTyhABWfoTrATDYomSIsJkpD8dn9ickKNZoicttMozUNdVfo9rwA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
尽管思科将攻击者描述为未经身份验证，但拥有这些  
 cookie   
仍然是一项重要前提。该公告未说明攻击者将如何获取这些  
 cookie  
。因此，应避免将该发现解读为攻击者可以立即入侵每一台可访问的  
 Expressway   
设备的证据。  
  
思科将  
 Expressway X15.5.3   
及更早版本列为受影响版本，  
X15.5.20   
被确定为首个修复版本。使用  
 X14   
或更早版本的客户必须迁移到修复版本。不能存在可解决该漏洞的应变方法。  
  
该修复方案也包含在思科更广泛的  
 2026   
年  
 10   
月  
 Expressway   
安全加固版本中。该版本还解决了在内部安全审计中发现的其它弱点。与该集群特定漏洞不同，更广泛的加固公告涵盖受影响的任何配置下的  
 Expressway-C   
和  
 Expressway-E   
设备。管理员应检查已安装版本和集群设置，然后按照思科的兼容性指南规划升级。仅检查版本并不能确定暴露情况，因为该特定漏洞还需要集群部署。  
  
Jane Street   
公司的研究员  
 Stefan Aleksic   
发现并报送了该漏洞。截至发布时，其产品安全事件响应团队表示未发现公开公告或恶意利用。鉴于远程命令以  
 root   
权限运行的潜在可能性，该声明并不降低修复受影响集群的必要性。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[思科：注意可导致 Nexus 交换机遭接管的多个严重漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527286&idx=1&sn=eb70fbb49754e65564698f039b45ba32&scene=21#wechat_redirect)  
  
  
[思科：ISE 认证绕过满分 0day 已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527173&idx=1&sn=1ae52540af18bbb2b176b85380e718cc&scene=21#wechat_redirect)  
  
  
[思科：注意这些已达生命周期IP电话中的RCE 0day](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247520401&idx=2&sn=bad61fd4a7dd0064d773564100fa0d93&scene=21#wechat_redirect)  
  
  
[思科提醒注意通信软件中的严重 RCE 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247518760&idx=2&sn=d82d599134c7b2a410f4ccfe05d73d96&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/critical-cisco-expressway-vulnerability/  
  
  
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
  
