#  VPN漏洞遭勒索软件大规模利用，Palo Alto警告多家客户被入侵  
e安在线
                    e安在线  e安在线   2026-10-22 01:50  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Hxdb7gjfn9nJMibZJ7J4WFt2X5OvZL80jp6AyT9D4icIIHqOaUpiaS4RemY5QGecPQDOL3WCgYWhZMfzvgHyTYe5u1Yt6IKnprWibnLGcfS4P5c/640?wx_fmt=jpeg&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9nbkDuoiayWgM8xf9Vzuiad7ckInXNibIqibuJmJnzI7HCf2ThNdux6riaiadpy8uGkEPzMLeBd1jufb6BXBNKohMe1XfDFFNeYRJ6nA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9m0ezrOIpQvksVD8mnvW8MVHPRLzK3eWYjicwiad8ibQV24Cyr6BwRHH7Fia6ibibynJInXWyVh3klV5enSLibpEFuPXl4agricYRibGyfw/640?wx_fmt=png&from=appmsg "")  
  
  
**多家安全公司监测发现，多个威胁行为者正利用Palo Alto公司VPN产品的一个高危漏洞发起攻击，未来数月这一情况可能更加猖獗；**  
  
  
**Palo Alto威胁情报团队也表示，发现小部分VPN产品遭到未授权连接，教育、金融、医卫等11个行业的组织受到影响，有攻击者进入内网后开始攻击虚拟化设施，官方建议尽快排查修复。**  
  
  
  
网安巨头派拓网络（Palo Alto Networks）的研究人员警告，旗下GlobalProtect产品的漏洞披露数月后，勒索软件团伙正在加大利用力度。  
  
  
该漏洞编号为CVE-2026-0257，属于身份验证绕过漏洞。攻击者利用这一漏洞，就能在未经授权的情况下与该远程访问产品建立VPN连接。该漏洞的严重性评分为7.8分（满分10分）。  
  
  
**多个勒索软件团伙持续利用该漏洞逐步成为标配手段**  
  
  
10月9日，网络安全公司ReliaQuest的研究人员在X平台发文称，他们发现多个威胁团伙利用该漏洞发动攻击，其中包括Qilin和Settra勒索软件团伙。攻击者借此建立未经授权的VPN会话，进入目标组织的内部网络，获得初始访问权限。  
  
  
ReliaQuest副总裁Brandon Tirado表示：“漏洞披露约五个月后，它已经成为勒索软件团伙入侵网络的一条可靠路径。未来三个月，这种情况很可能会持续下去，因为会有更多团伙和初始访问权限经纪人开始利用这一漏洞。”  
  
  
ReliaQuest还警告称，相关攻击来自内部VPN连接，行为可能与合法的远程访问会话难以区分，因此容易逃过检测。在安全团队发现异常之前，攻击者可能已经窃取凭证、横向移动、窃取数据，甚至部署勒索软件。  
  
  
研究人员还在相关攻击中发现了多种工具，包括Cobalt Strike、BloodHound和SharpHound。  
  
  
网络安全公司Arctic Wolf在一篇博客文章中披露，其研究人员调查了今年6月发生的多起与Qilin勒索软件有关的攻击。这些攻击方式各不相同，有的快速加密受害者数据，有的采用双重勒索手段。这表明，相关活动可能涉及多个Qilin附属团伙，它们在勒索软件即服务（RaaS）模式下运作。  
  
  
Arctic Wolf对手行动部门总监Justin Moore表示：“从攻击时间线来看，攻击者利用漏洞获得访问权限后，可以迅速把这种访问转化为持久的远程访问能力，并直接操控受害系统，最终为部署勒索软件创造条件。”  
  
  
**官方公布真实攻击案例建议相关组织尽快排查**  
  
  
派拓网络旗下负责威胁情报与事件响应的Unit 42团队发布报告称，研究人员发现，Qilin和INC勒索软件团伙均在利用这一漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9mX2iaMiaziacgeNhJvtF1TAz7BSpnsuj1FGsgFPgic0aFVCTRq3LJHzxG5DBlr5YCicFFE2PdbdIVcWpJCK56beibdgGaJfbwiaJweuM/640?wx_fmt=png&from=appmsg "")  
  
  
Unit 42的研究人员自今年6月以来一直在调查相关威胁活动。受影响的目标涉及多个行业，包括教育、金融服务、医疗保健、IT、制造业和政府部门。  
  
  
Unit 42研究人员指出，这些攻击的一个显著特点是，攻击者成功利用漏洞后，会进一步侵入虚拟化基础设施层。在一起案例中，攻击者从已遭入侵的VPN转向VMware vCenter管理设备。  
  
  
研究人员还发现，攻击者关闭了ESXi虚拟机监控程序上的安全控制措施，并使用定制工具，对域控制器的虚拟磁盘进行离线凭证转储，窃取身份验证信息。  
  
  
Unit 42表示，这种手法能帮助攻击者绕过端点检测与响应（EDR）机制，让防御方更难发现和阻止攻击。  
  
  
Unit 42建议，相关组织尽快排查相关资产和攻击指标情报，并处置修复。  
  
  
  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9mdiazS6kj8yRJ5Z4Ud2ibK0O72gROzMm0AIINXIW5OIsA5FvVk4D7Wt52bNXpAsOALWWiaYFIicHWuiaSWCKPcyKwN2FBzib19jU07Y/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9l4Os8icdlRkJpbaDYdCSI4wSet9wichps53w3ASJerf86L2QmBJFIJH7k7CuDVtOOTs4z9OnAUqY5hVHB4qibicchDCpmLUBB0AiaA/640?wx_fmt=png&from=appmsg "")  
  
  
声明：除发布的文章无法追溯到作者并获得授权外，我们均会注明作者和文章来源。如涉及版权问题请及时联系我们，我们会在第一时间删改，谢谢！文章来源：安全内参  
  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9m8siczpPOiajUicKHMFznDCMkiaG4x3C28IanmcYa6QmSbGs7pydwNLzmEjas6MJOSHwAx55xdQEa7Fyt2Otia1E8U9ia0AEv8Tc4UQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9mtgFQlia6Ca5LM6icEfofJOl4qwIBEZu038W8Q2vgxFjfTPWjAT1xuZvN2jiaFq3TAoelFNknWFic6kVcJIj0v6NB2TGNxNgdJrib4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9mfNibh3FZGLMFlTB4cEVarhKefrNTpulstonFvP40KJPHJ9Ba5MDMAnO8dIVXKShz4L3aMN9P306AvljcXmrxezlMAkGO4NjEw/640?wx_fmt=png&from=appmsg "")  
  
  
  
