#  Elecbrakes蓝牙存在严重漏洞，可用于控制行驶中的拖车刹车制动力  
 代码卫士   2026-11-25 06:27  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
作者  
：  
Tushar Subhra Dutta  
  
编译：代码卫士  
  
**Elecbrakes****拖车制动控制器中存在一个严重的蓝牙漏洞（****CVE-2026-96805****，****CVSS****评分****9.6****），可导致附近人员在拖车被牵引时施加、调节或释放拖车刹车，且无需配对、****PIN****码或驾驶员采取任何操作。**  
  
美国网络安全和基础设施安全局 (CISA) 于2026年11月19日披露了该问题，同时披露的还有两个相关漏洞。EB2和VM1已有固件修复程序，但车主必须在Elecbrakes应用中自行启动更新。较旧的ELBC 2000也存在该漏洞，且在披露时没有修复程序。这一区别很重要，因为拖车制动控制器管理的是拖车施加多少制动力，而不是牵引车自身刹车如何运作。  
  
Element Digital公司的研究人员Patrick Forck和Jeremy McCarthy在Forck注意到他的车载遥控器在没有配对提示的情况下连接到了别人的拖车后，发现了这些漏洞。他们仅在私人场地或封闭环境中测试了自己拥有的设备。他们的报告描述的是产品漏洞，而非恶意软件活动。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfVfRcf75lJTX2UwLDs4FDcQCpySCKoJqFZrIvUTiaAuxgSEfCdqtic6fS37OARuuWWr5lX8yKybAhia1Y0QOVkkoS0WicbgMQket3E/640?wx_fmt=gif&from=appmsg "")  
  
**严重的蓝牙漏洞**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUcpRWKDmHljQqQYo05Vcdic79PDDGufpniboDGtjr81QibG3O0Joy88PR4wDTDU1IfswtVuVMaicPzf69y33OJ3UcItUibVZCc561A/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该控制器使用低功耗蓝牙与Elecbrakes应用和无线遥控器通信。在存在漏洞的固件上，控制器未验证制动指令是否来自授权设备。  
  
与影响信息娱乐软件的PerfektBlue汽车蓝牙漏洞不同，Elecbrakes漏洞直接暴露了控制制动力的功能。在受控驾驶测试中，相同速度下更强的未经授权制动产生了明显更大的减速度。在公共道路上，意外的拖车制动可能破坏牵引车和拖车的平衡。此前关于无线列车制动漏洞的报道描述的是另一项技术，但两者强调了相同的安全原则：制动控制器必须验证是谁在向其发出指令。  
  
第二个严重漏洞CVE-2026-125273涉及重放：控制器可能再次接受先前发出的指令，而不检查其是否为新发指令。该漏洞同样评分为9.6。这与大众蓝牙黑客的演示不同，在后者中研究人员利用信息娱乐软件访问座舱功能和数据，而非直接控制拖车刹车。第三个问题CVE-2026-124281评分为6.1，影响无线固件更新通道。CISA警告称，保护不足可能允许未经授权的固件更改。研究人员并未在运行中的控制器上安装修改过的固件，因此不应将持久更改的可能性与他们在受控测试中演示的制动激活混为一谈。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUYv2iaqfMFBBc8Jum2ibuDZXhVd9QSYvbh5GtwDdBkibpA0bnib9FDI74715jrVeiaDXpvFNQepaoWwI4nYARic9NiaAD9IIvxOz58hc/640?wx_fmt=gif&from=appmsg "")  
  
**固件修复及 Elecbrakes 车主应采取的措施**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfXiaGobS7LFkErUytkQmb8ibXYkuDrh47OkAUE1fWibWMTSWLo7unkbOsg4KmrMGHexG4oMLtGycAicl3qZU4H1xdbNYflPuvCvMdA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
Elecbrakes于5月18日收到报告，并于7月6日发布修复程序。CISA工业控制公告指示EB2车主安装3.0.4.4.3或更高版本固件，VM1车主安装1.3.4.2.1或更高版本。  
  
打开Elecbrakes应用，连接控制器，检查已安装版本。如果版本较旧，请手动启动更新，而不要假设它已自动安装。修复后的控制器将制动和更新模式访问限制为经过身份验证的连接。初始配对需要通过牵引车刹车踏板进行物理确认。这一更改导致较旧的无线遥控器与新配对流程不兼容，因此Elecbrakes为受影响的EB2和VM1车主提供免费更换遥控器。使用遥控器的车主应检查自己的遥控器是否需要更换。  
  
所有版本的ELBC 2000均被列为受影响，披露时没有可用的固件修复程序。正如此前关于遗留雷诺车辆软件的报道所示，设备在其软件老化很久之后仍可能继续使用。ELBC 2000车主应联系Elecbrakes获取针对具体型号的指导，并定期检查应用以获取修复更新。  
  
手动更新还在发布修复程序与保护道路上拖车之间留下了空档。截至10月下旬，Elecbrakes告诉研究人员，约60%的每日活跃的EB2和VM1用户已安装最新固件。这一数字并未计入很少打开应用的车主。车队运营者应逐一核实每个控制器的固件，而不是将该部署比例作为自身设备已打补丁的证据。  
  
**入侵指标（IoC）：**  
<table><tbody><tr><td data-colwidth="277" width="277" valign="top" style="border: 1px solid windowtext;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size:15px;"><strong><span leaf="">指标类型</span></strong></span></p></td><td data-colwidth="277" width="277" valign="top" style="border-top: 1px solid windowtext;border-right: 1px solid windowtext;border-bottom: 1px solid windowtext;border-image: initial;border-left: none;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size:15px;"><strong><span leaf="">来源中的状态</span></strong></span></p></td></tr><tr><td data-colwidth="277" width="277" valign="top" style="border-right: 1px solid windowtext;border-bottom: 1px solid windowtext;border-left: 1px solid windowtext;border-image: initial;border-top: none;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size: 15px;"><span leaf="">SHA-256、SHA-1或MD5哈希</span></span></p></td><td data-colwidth="277" width="277" valign="top" style="border-top: none;border-left: none;border-bottom: 1px solid windowtext;border-right: 1px solid windowtext;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size: 15px;"><span leaf="">未发布</span></span></p></td></tr><tr><td data-colwidth="277" width="277" valign="top" style="border-right: 1px solid windowtext;border-bottom: 1px solid windowtext;border-left: 1px solid windowtext;border-image: initial;border-top: none;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size: 15px;"><span leaf="">恶意文件或载荷哈希</span></span></p></td><td data-colwidth="277" width="277" valign="top" style="border-top: none;border-left: none;border-bottom: 1px solid windowtext;border-right: 1px solid windowtext;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size: 15px;"><span leaf="">未发布</span></span></p></td></tr><tr><td data-colwidth="277" width="277" valign="top" style="border-right: 1px solid windowtext;border-bottom: 1px solid windowtext;border-left: 1px solid windowtext;border-image: initial;border-top: none;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size:15px;"><span leaf="">恶意IP地址、域名或URL</span></span></p></td><td data-colwidth="277" width="277" valign="top" style="border-top: none;border-left: none;border-bottom: 1px solid windowtext;border-right: 1px solid windowtext;padding:5px 10px;"><p style="text-align:left;text-indent: 0em;margin-bottom: 15px;display: block;"><span style="font-size: 15px;"><span leaf="">未发布</span></span></p></td></tr></tbody></table>  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[大众汽车蓝牙漏洞可暴露对话、泄露GPS位置及控制车内功能](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527816&idx=1&sn=fc71ab4b2e1407d56091d3c0684c89c4&scene=21#wechat_redirect)  
  
  
[CISA：这个思科 RCE 漏洞已遭活跃利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527816&idx=2&sn=c384a37aad4ccd136309edfc3da08093&scene=21#wechat_redirect)  
  
  
[蓝牙芯片集多个漏洞可导致攻击者窃听或盗取敏感信息](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247523404&idx=2&sn=3ff2f901d13c30f8048d7a38b567880f&scene=21#wechat_redirect)  
  
  
[苹果修复可导致窃听的 AirPods 蓝牙漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247519901&idx=2&sn=3262641607db182c3d651a403a7e8a1f&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://cybersecuritynews.com/critical-bluetooth-flaw/  
  
  
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
  
