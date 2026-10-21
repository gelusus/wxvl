#  VPN漏洞遭勒索软件大规模利用，Palo Alto警告多家客户被入侵  
安全内参编译
                    安全内参编译  安全内参   2026-10-21 10:04  
  
**关注我们**  
  
  
**带你读懂网络安全**  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/FzZb53e8g7sCJ9An5FeDcUKJA1xL2QdPJVK970AnWEKrIdG8dHcGVicLOBLzrSsIHpxZ6rWB8N1cV3ibV4iauyvsw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
  
多家安全公司监测发现，多个威胁行为者正利用Palo Alto公司VPN产品的一个高危漏洞发起攻击，未来数月这一情况可能更加猖獗；  
  
  
Palo Alto威胁情报团队也表示，发现小部分VPN产品遭到未授权连接，教育、金融、医卫等11个行业的组织受到影响，有攻击者进入内网后开始攻击虚拟化设施，官方建议尽快排查修复。  
  
  
前情回顾·**安全产品漏洞利用态势**  
- [警惕！Fortinet防火墙漏洞遭勒索软件利用，多家企业被黑](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247514014&idx=1&sn=940df55ce8c872acd2dc9639c8eef32a&scene=21#wechat_redirect)  
  
  
- [Palo Alto防火墙又被黑：最新漏洞披露后第二天就遭利用](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247513752&idx=1&sn=39cc6a0e87b2d70d65cdc8731d42f1e2&scene=21#wechat_redirect)  
  
  
- [警惕！近期Fortinet防火墙频遭攻击，疑似零日漏洞被利用](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247513504&idx=1&sn=838af406a4e2aec5e99b37399b5f1a2e&scene=21#wechat_redirect)  
  
  
- [2024年至少75个零日漏洞遭滥用，网络与安全产品占比近三成](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247514283&idx=1&sn=681c71445b5cfa5f0ab2fde668d4fef8&scene=21#wechat_redirect)  
  
  
  
  
安全内参10月21日消息，网安巨头派拓网络（Palo Alto Networks）的研究人员警告，旗下GlobalProtect产品的漏洞披露数月后，勒索软件团伙正在加大利用力度。  
  
该漏洞编号为CVE-2026-0257，属于身份验证绕过漏洞。攻击者利用这一漏洞，就能在未经授权的情况下与该远程访问产品建立VPN连接。该漏洞的严重性评分为7.8分（满分10分）。  
  
  
多个勒索软件团伙持续利用该漏洞，  
  
  
逐步成为标配手段  
  
  
  
10月9日，网络安全公司ReliaQuest的研究人员在X平台发文称，他们发现多个威胁团伙利用该漏洞发动攻击，其中包括Qilin和Settra勒索软件团伙。攻击者借此建立未经授权的VPN会话，进入目标组织的内部网络，获得初始访问权限。  
  
ReliaQuest副总裁Brandon Tirado表示：“漏洞披露约五个月后，它已经成为勒索软件团伙入侵网络的一条可靠路径。未来三个月，这种情况很可能会持续下去，因为会有更多团伙和初始访问权限经纪人开始利用这一漏洞。”  
  
ReliaQuest还警告称，相关攻击来自内部VPN连接，行为可能与合法的远程访问会话难以区分，因此容易逃过检测。在安全团队发现异常之前，攻击者可能已经窃取凭证、横向移动、窃取数据，甚至部署勒索软件。  
  
研究人员还在相关攻击中发现了多种工具，包括Cobalt Strike、BloodHound和SharpHound。  
  
网络安全公司Arctic Wolf在一篇博客文章中披露，其研究人员调查了今年6月发生的多起与Qilin勒索软件有关的攻击。这些攻击方式各不相同，有的快速加密受害者数据，有的采用双重勒索手段。这表明，相关活动可能涉及多个Qilin附属团伙，它们在勒索软件即服务（RaaS）模式下运作。  
  
Arctic Wolf对手行动部门总监Justin Moore表示：“从攻击时间线来看，攻击者利用漏洞获得访问权限后，可以迅速把这种访问转化为持久的远程访问能力，并直接操控受害系统，最终为部署勒索软件创造条件。”  
  
  
官方公布真实攻击案例，  
  
  
建议相关组织尽快排查  
  
  
  
派拓网络旗下负责威胁情报与事件响应的Unit 42团队发布报告称，研究人员发现，Qilin和INC勒索软件团伙均在利用这一漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/wT9KAyOic0NCUlT3NDdvFLELibYmpOiaVGPMQXic8Hiax1z3tfCsSZfWeGyoEzib0GlMYaHgY32LicNEN1ExVlSwwPUtqiaNIFKY3sr6mB8OEsWFaR4/640?wx_fmt=png&from=appmsg "")  
  
Unit 42的研究人员自今年6月以来一直在调查相关威胁活动。受影响的目标涉及多个行业，包括教育、金融服务、医疗保健、IT、制造业和政府部门。  
  
Unit 42研究人员指出，这些攻击的一个显著特点是，攻击者成功利用漏洞后，会进一步侵入虚拟化基础设施层。在一起案例中，攻击者从已遭入侵的VPN转向VMware vCenter管理设备。  
  
研究人员还发现，攻击者关闭了ESXi虚拟机监控程序上的安全控制措施，并使用定制工具，对域控制器的虚拟磁盘进行离线凭证转储，窃取身份验证信息。  
  
Unit 42表示，这种手法能帮助攻击者绕过端点检测与响应（EDR）机制，让防御方更难发现和阻止攻击。  
  
Unit 42建议，相关组织尽快排查相关资产和攻击指标情报，并处置修复。  
  
  
**参考资料：bleepingcomputer.com**  
  
  
**推荐阅读**  
- [网安智库平台长期招聘兼职研究员](http://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247499450&idx=2&sn=2da3ca2e0b4d4f9f56ea7f7579afc378&chksm=ebfab99adc8d308c3ba6e7a74bd41beadf39f1b0e38a39f7235db4c305c06caa49ff63a0cc1d&scene=21#wechat_redirect)  
  
  
- [欢迎加入“安全内参热点讨论群”](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247501251&idx=1&sn=8b6ebecbe80c1c72317948494f87b489&chksm=ebfa82e3dc8d0bf595d039e75b446e14ab96bf63cf8ffc5d553b58248dde3424fb18e6947440&token=525430415&lang=zh_CN&scene=21#wechat_redirect)  
  
  
  
  
  
  
  
点击下方卡片关注我们，  
  
带你一起读懂网络安全 ↓  
  
  
  
  
