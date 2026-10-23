#  SonicWall SMA1000 严重性最高的漏洞现已被用于不法攻击  
Rhinoer
                    Rhinoer  犀牛安全   2026-10-23 16:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8J3tGHf5M7bO2O6y3WSEMTbB0rr5LuYP10sqF7DePYOqakUfrruD5OkQS00PXtat3iaozqmV2OT3FtNObfibR39jpWte5vmya644/640?wx_fmt=png&from=appmsg "")  
  
攻击者正在利用 SonicWall SMA1000 设备中的一个严重性  
最高的  
漏洞（CVE-2026-102255），该漏洞已于三天前的周二进行了修补。  
  
该漏洞编号为CVE-2026-102255，会影响 SMA1000 6210、7210 和 8200v 型号上的 Appliance WorkPlace 接口，但不会影响 SMA 100 系列产品线或在 SonicWall 防火墙上运行的 SSL-VPN。  
  
SonicWall解释说： “通过滥用此路径，未经身份验证的远程攻击者可能会利用此漏洞，指示设备代表他们发出请求，从而访问内部功能并执行未经授权的操作。”  
  
虽然 SonicWall 尚未在其周二的公告中将此漏洞标记为已被积极利用，但 Previdian 创始人兼安全研究员 Ryan Dewhurst 周五告诉 BleepingComputer，该公司的蜜罐网络已检测到与 CVE-2026-102255 漏洞一致的利用尝试。  
  
Dewhurst 告诉 BleepingComputer：  
“这些请求的目标是 WorkPlace Extraweb 界面，攻击者使用精心构造的 OPTIONS 请求来访问设备内部的 CouchDB 服务，地址为 127.0.0.1:5984。有效载荷试图访问 CouchDB 设计文档并调用其 _rewrite 函数，同时提供了一个包含凭据 admin:admin 的 HTTP 基本授权标头”。  
  
“它影响的是与 2026 年 7 月和 9 月披露的 SSRF 漏洞相同的 WorkPlace 界面。但是，10 月份的漏洞使用了不同的利用技术。  
  
德赫斯特还补充说，虽然这种活动与积极的攻击尝试相符，但 Previdian 尚未确定“这些尝试是否已成功入侵任何系统”。  
  
Previdian 的报告也得到了威胁情报公司 Defused 的证实，该公司表示：“从昨天开始，我们一直在观察针对我们的 SonicWall SMA 诱饵的未经身份验证的 SSRF 流量。”  
  
虽然互联网威胁监督机构 Shadowserver 目前追踪到400 多台暴露在网上的 SMA1000 设备，但尚无信息表明其中有多少是蜜罐，或者有多少已经针对 CVE-2026-102255 攻击进行了修补。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/vO1zY1O9p8JetBd0FFY5A64ayw3KNZHgEtvtKPCr9PRQrYQicGK9xyhYuabmcMTXQoL1ArzcKj0icuibiaNkbD6smmlTicOuXa0t5LAibjhIjNnnM/640?wx_fmt=png&from=appmsg "")  
  
SMA1000 企业级安全远程访问网关经常成为攻击目标，因为托管服务提供商 (MSSP)、许多大型企业和政府机构使用它们进行 VPN 访问以访问内部应用程序和企业网络。  
  
例如，在 7 月份，攻击者利用两个 SMA1000 零日漏洞（CVE-2026-15409 和 CVE-2026-15410）持续数周，在易受攻击的 VPN 设备上安装定制的 Sou5、OrangeTail 和 RootRun 恶意软件。  
  
美国网络安全和基础设施安全局 (CISA) 后来将其中一些攻击与勒索软件团伙联系起来。  
  
上个月，SonicWall 还警告客户，攻击者正在利用两个新的零日漏洞（CVE-2026-83548 和 CVE-2026-83549）在易受攻击的 SMA1000 网关上执行远程代码。  
  
在过去的四年里，CISA 已将 19 个 SonicWall 漏洞添加到其活跃利用的漏洞目录中，并标记了其中 13 个漏洞已被勒索软件团伙利用。  
  
  
信息来源：  
BleepingComputer  
  
