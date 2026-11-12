#  攻击者利用AhsayCBS漏洞，在PNG图片中隐藏恶意代码并部署隐蔽后门  
 FreeBuf   2026-11-12 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX3iaEdMRuqKV0wCsHbibGHNO71qrfvSmfOFfibHBicnmEtic9LYtZeDIOUmCn9KdpBKlEB9wWaotaEL7NRRlzNH0VyicEugZudfsgdUM/640?wx_fmt=gif "")  
  
  
![AhsayCBS漏洞利用](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX124MYYSC4ICxpvHwhorlQyoLhsnD2a92DVgibXHhxlv3c3O6a5EfOOj09dsicbUBtUOx8tuu9Y3oFMhibeU0OsWvNR7Ry1b9ibwwM/640?wx_fmt=png "")  
  
  
攻击者针对AhsayCBS备份服务器发起漏洞利用攻击，植入多类webshell，将加密代码隐藏在PNG图片内，还部署了仅能在目标主机上运行的Python植入物。  
  
  
调查显示，此次入侵的活动范围不止加密货币挖矿，攻击者还搭建了多条持久化访问通道，可长期控制负责保护企业备份数据的核心系统。  
  
  
此次攻击发生在2026年10月8日至11日，当时厂商尚未发布热修复补丁，共造成某企业的3台AhsayCBS服务器失陷。  
  
  
攻击者利用了两个漏洞：其一是身份绕过漏洞（CVE-2026-105133）；其二是远程代码执行漏洞（CVE-2026-105134），未授权攻击者利用该漏洞可直接获取SYSTEM权限。两个漏洞组合利用，不仅攻破了备份平台本身，还导致平台持有的敏感访问权限完全暴露。  
  
  
Huntress研究人员在调查过程中，共识别出5类webshell、第二个加密货币挖矿程序，以及一个与主机绑定的Python植入物。  
  
  
11月9日发布的技术报告中，研究人员Jonathan Semon介绍了攻击者使用的工具集：这些工具可隐藏执行的命令、支持文件传输，还能主动削弱端点防护能力。  
  
  
不过研究团队尚未确认所有载荷的具体行为，也无法证实攻击者是否窃取了备份数据。  
  
  
Part  
01  
  
植入多类webshell  
  
  
其中最具特殊性的一类webshell伪装成内部图片预览服务。它不会接收明显的命令参数，而是处理用户上传的PNG文件，从像素数据中提取隐藏信息。  
  
  
这种手法与此前其他攻击活动中披露的PNG恶意软件隐写技术相似，但Huntress并未将此次入侵归因于那些活动的运营者。  
  
  
加载器会读取每个像素红、绿、蓝通道值的最低有效位，将这些位拼接重组为字节，先解析出4字节的载荷长度字段，后续即为加密的载荷内容。  
  
  
加载器内置硬编码AES密钥，使用Java默认的AES转换模式解密数据，Huntress确认该模式为ECB模式搭配PKCS5填充。  
  
  
首次收到请求时，自定义Java类加载器会将解密后的类加载到内存中，同时返回一张用于伪装的正常图片。后续请求会将参数传入该类并触发代码执行。  
  
  
执行结果会被加密后嵌入另一张PNG图片返回，让整个交互过程看起来和普通图片流量无异，而非传统的命令控制会话。  
  
  
这种技术的隐蔽性极强：可执行代码完全通过正常请求传递，且全程在内存中运行，不会落地文件。此前Earth Sirrush组织的PNG隐写攻击活动表明，这类图片隐藏技术也被用于间谍行动，但二者仅手法相似，不足以证明存在关联。Huntress仅分析了该加载器本身，未获取到其投递的恶意类，因此目前尚不清楚攻击者通过该webshell执行过哪些命令。  
  
  
![植入物执行流程，经静态分析还原（来源：Huntress）](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0KlaH0Y1dYkNkVNLicwZJmGd9FaibWvBVdWI7Mgk6WnWRXmjqyBHRqgW70JpAibz369FH5JJcfmugUBCSeNPSnIrLZOF29MO1pBw/640?wx_fmt=png "")  
  
  
研究人员还发现了另一款行为类似远程管理API的webshell。它会先校验API令牌、时间戳、随机数以及HMAC-SHA256签名，校验通过后才会接收请求。  
  
  
通过校验后，攻击者可以执行命令、切换目录、分块传输文件，最大支持4GB的文件传输。不过Huntress未找到相关日志，无法证明攻击者曾使用过该webshell的下载功能。  
  
  
Part  
02  
  
部署主机绑定后门  
  
  
另一款独立的Python植入物采用了多层隐藏机制。其包装程序结合了Base64编码、LZMA压缩以及序列化的Python代码对象。  
  
  
运行前，它会根据主机的MachineGuid计算SHA-256值，并检查主机上是否存在匹配的标记文件。如果没有该文件，脚本就会直接退出，以此避免在分析人员的沙箱环境中运行。  
  
  
如果标记文件存在，代码会将一段673KB的二进制块复制到可执行内存中并启动线程。随后反射式加载器会映射程序内置的x64 DLL文件。  
  
  
静态分析发现了C2通信信息以及符合DNS-over-HTTPS特征的字符串，但Huntress既未观测到该植入物实际运行，也无法确定其所属的恶意软件家族。  
  
  
攻击者还部署了XMRig挖矿程序，创建了以SYSTEM权限运行的计划任务，每分钟检查一次程序状态，必要时会自动重启挖矿进程。  
  
  
攻击者还投放了隧道工具、远程访问软件，并添加了大范围的Microsoft Defender排除项，既拓展了操作渠道，也大幅降低了失陷服务器上的安全可见性。  
  
  
在一台关联的Veeam服务器上，调查人员发现了与NetExec凭证窃取模块特征一致的命令。该命令会查询本地存储的备份凭证，并调用DPAPI解密获取凭证明文。  
  
  
此前已有多篇报告披露Veeam备份凭证泄露风险，解释了这类环境为何成为高价值攻击目标，但本次入侵的初始入口并非该Veeam漏洞。  
  
  
Ahsay已针对10版本发布热修复补丁10.3.4.45，针对7至9版本发布热修复补丁9.15.2.70。Huntress验证确认，10版本的补丁可有效阻断此次攻击中观测到的入侵入口。  
  
  
需要注意的是，安装补丁无法清除服务器上已存在的后门。受影响企业应全面排查服务器上的可疑活动、未授权的防护排除项以及各类持久化机制。如果确认服务器已失陷，Huntress建议从可信备份重建系统，并轮换所有可被失陷服务器访问的凭证。  
  
  
Part  
03  
  
公开攻击失陷指标  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1uVVrnH9t5F9rfYmZb5VsxMibib47VM6qv59byWRWauT6iaicFCdx8dniakMj5ciaHdiaqQ5YPIXTdTBoY6zXwH2IicR3y7aovlG16hE8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1JudPSmNy1BphH8bOkmiaoROsGEmSNAuaIicz5m2Lr9kialHjd80C8qE6JkElQW0bzOP9TYnEVttB2KOBNibOoZW8YaRYgUfHKvGY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX16hPZfTfEic2KBibjBmA03Fjz5rrj6unQMvIQ2CBgePC8qKH7xdRibuVVgcPgIoRM4kyFUiaMmFn8P0qDjGnzPwZdlEiaRLRE41YKw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX22mqgStibKNyEewlcYYhm3dlvjiaiaR6V5Mgroq9gjwlQvj9iaibWNpDiczxvm8ibnquBZ6ldgm5UeKTqgibQfkQaVKCty8icLPoJ4RT6M/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3ClvreChOKiaW0rMN8siboggQZibbqJ3ueGyv5lRgAVtWxwTeI84MsyGXjx9lODXaeJ93Asiat1HW5cL65ABN6EkFkBua2y3dASHE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX1LZCclnyADIGMpxElSib0hzQrc63e2pxyXD7ib4ibOgrFhuiaRTD49iaibNR7TX9icJPdUz1p9ThwOtCR4ibyx1upLrraFialkyKVLaicWw/640?wx_fmt=png&from=appmsg "")  
  
  
注：文中IP地址和域名均已做去激活处理（如使用[.]替换原有点号），防止意外解析或触发超链接。请仅在MISP、VirusTotal或内部SIEM等受控威胁情报平台中还原地址后使用。  
  
  
参考来源：  
  
Hackers Exploit AhsayCBS Vulnerabilities to Hide Malware in PNG Images and Deploy Stealthy Backdoors  
  
https://cybersecuritynews.com/hackers-exploit-ahsaycbs-vulnerabilities/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
  
