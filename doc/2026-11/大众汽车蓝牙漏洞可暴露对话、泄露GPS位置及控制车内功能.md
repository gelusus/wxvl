#  大众汽车蓝牙漏洞可暴露对话、泄露GPS位置及控制车内功能  
Abinaya
                    Abinaya  代码卫士   2026-11-24 08:14  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**大众汽车存在多个蓝牙漏洞，可被用于暴露私人对话、泄露****GPS****位置，并允许未经授权控制车内功能。**  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
这些研究发现凸显了补丁推送周期漫长，部分车辆在漏洞公开约  
16  
个月后仍处于暴露状态。据媒体  
heise  
报道，匈牙利公司  
PCA Cyber Security  
在布达佩斯针对一辆大众  
ID.4  
演示了该攻击。  
  
据报道，研究人员在车外使用笔记本电脑访问了免提麦克风、监控了位置数据，并操作了加热器、挡风玻璃雨刮器和天窗。该攻击利用的是  
PerfektBlue  
，即  
OpenSynergy  
公司  
BlueSDK  
蓝牙软件中的一组漏洞链。该组件负责车载信息娱乐系统中的无线连接。  
PCA  
的安全公告解释了这些漏洞如何被组合利用，从而在受影响系统上运行攻击者控制的代码。  
  
在研究员演示的大众汽车攻击中，驾驶员必须先批准一个蓝牙配对请求。一旦连接成功，攻击者便可利用存在漏洞的软件，并获取对信息娱乐系统可用功能的控制权。这一前提很重要。该演示并未表明攻击者可以在无交互的情况下静默入侵附近每一辆大众汽车。拒绝陌生的配对请求即可阻断该攻击所需的连接。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfWSv85kf6AsflRE28fyJFE6ibRF6ujzQJCGjOj8Ka0kajibrJia33OAWXh2fmTcCNiaYiay4cPCBicxDD8Sg5crRE45eYcU4xPyPG5mU/640?wx_fmt=gif&from=appmsg "")  
  
**通过蓝牙入侵大众汽车**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWPiaF2DNetCtIXcRwkvw68RA0FmNOWn9iaFHrGNvhtKPA76du4x3PJoyMfrZlIQxv8DJRYAsaDd0evibptm3ib0ocB97Ald2g3R3I/640?wx_fmt=gif&from=appmsg "")  
  
  
  
PerfektBlue  
蓝牙漏洞影响多个汽车品牌所使用的软件。研究人员估计可能有数十万辆大众汽车受到影响。但该数字并非已确认的受入侵车辆数量，且所报道的演示并不能证明存在广泛的恶意利用。  
  
大众汽车表示，较旧的信息娱乐软件需要额外修改和安全检查，导致修复延迟。该公司称已在较新的软件分支中更快地解决了该问题。对于运行  
4.0  
之前版本软件的车辆，大众汽车表示  
3.9.1  
版本可修复该漏洞。符合条件的车主可在车辆停放时通过蜂窝网络或  
Wi-Fi  
安装该更新。其它受影响的软件版本需要到店维修。大众汽车表示，运行  
5.6  
或更高版本的车辆不受影响。车主应向大众汽车确认正确的更新途径，而不是假定每辆车都获得同样的修复。  
  
大众汽车表示，这些漏洞不会提供对转向、制动或加速的访问权限。所演示的访问涉及的是信息娱乐系统和互联座舱功能。尽管如此，麦克风访问可能泄露敏感对话，而  
GPS  
追踪可能暴露行程和常去目的地。即使驾驶控制不在攻击者触及范围内，这些隐私风险仍然重大。  
  
此前的安全研究还发现了影响车辆数据和信息娱乐系统的其它独立弱点，如斯柯达和大众汽车安全漏洞。这些发现不应被视为同一漏洞链。  
  
大众汽车建议车主切勿接受未知的蓝牙配对请求，也不要连接陌生的  
USB  
设备。驾驶员应检查已安装的软件版本，遵循官方更新说明，并在补丁可用性不明确时联系经销商。据  
heise  
报道，大众汽车尚未宣布针对该安全问题的车辆召回。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[黑客通过默认凭据利用 ForgeRock RCE，远程解锁联网汽车](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527714&idx=1&sn=9517d673fdd2b65fcf53df808063a273&scene=21#wechat_redirect)  
  
  
[15年前的雷诺汽车 Linux 导航系统软件被破解 可执行任意 root 命令](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527686&idx=1&sn=f75e5ce428d9d4a5a9d306af91430362&scene=21#wechat_redirect)  
  
  
[XCharge C6电动汽车充电桩存在多个严重漏洞，可导致任意代码执行](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526201&idx=2&sn=94f249f0b82ed4528faa20db71165ca7&scene=21#wechat_redirect)  
  
  
[电动汽车充电网络告警：Everon OCCP 后端系统存在严重漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525349&idx=1&sn=352cbd47ed06c60804138658fa37fece&scene=21#wechat_redirect)  
  
  
[2026 Pwn2Own 东京汽车大赛落下帷幕，Master of Pwn 诞生](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524969&idx=1&sn=fbd3e630ba31d2f0118b90ad0779d309&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/hack-volkswagen-cars-via-bluetooth/  
  
  
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
  
