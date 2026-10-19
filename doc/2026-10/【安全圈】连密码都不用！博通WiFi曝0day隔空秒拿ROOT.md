#  【安全圈】连密码都不用！博通WiFi曝0day隔空秒拿ROOT  
 安全圈   2026-10-19 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事件快报**  
：安全研究员 Quentin Kaiser 公开披露了博通（Broadcom）Wi-Fi 设置软件中存在长达十余年的严重内存破坏缺陷。攻击者只要处于路由器的无线射频覆盖范围内，无需知道 Wi-Fi 密码，无需任何配对交互，就能通过特制无线帧向设备发起攻击，直接在路由器上以最高 root 权限执行任意代码或回传 Wi-Fi 密钥。该漏洞迄今未分配 CVE 编号，涉及全球数以百万计的路由器网关，且多款老旧芯片将面临永久无补丁可打的困境。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/sbq02iadgfyGbobNABV7wUZv7lUopoBOEUH0eGfTDSakk8vhZ4wfJiag7iaCYwUibWYbZicXbmU09YSqKJawaNYYs5ibphhduDag3MHKBl179W1Rk/640?wx_fmt=gif&from=appmsg "")  
## 🔍 漏洞原理：WPS 协议分片重组失控触发内存破坏  
  
问题的症结位于博通官方实现的 Wi-Fi 保护设置（WPS，Wi-Fi Protected Setup）功能模块中。WPS 原本是为了让用户省去繁琐密码输入、一键快速联网设计的标准协议，但博通的代码在处理经过分片的 WPS 配置协商报文重组时，缺乏对边界和有效载荷长度的严谨校验。  
  
攻击者在 Wi-Fi 覆盖范围内发送特制分片报文，就能破坏路由器的内存堆栈状态：  
- **远距离凭空窃取 Wi-Fi 预共享密钥（PSK）**  
：在对 Orange Belgium 的 Askey TCG3000 网关测试中，利用内存越界泄露缺陷，直接从路由器内部存储区读出明文密码，并通过无线信道发回给攻击者，攻击机随即自动握手连入局域网。  
  
- **无线帧承载反弹 Shell**  
：在对 Proximus 旗下的 Sagemcom BBOX3V+ 网关实测中，攻击者通过纯 Wi-Fi 802.11 原始数据帧直接弹出反向 Root 命令行终端，全程完全不依赖任何外部公网 C2 服务器中转。  
  
- **零门槛非接触渗透**  
：受害设备不需要开启任何管理端口，只要无线广播处于开启状态，攻击者在临近马路、楼道或隔壁房间就能静默打穿网络边界。  
  
## ⚡ 受影响范围：涉及数百万设备与永久 0day 困局  
  
博通的 Wi-Fi 代码方案被思科、华为、ARRIS、Sagemcom 等数十家全球网络设备大厂广泛集成于家用网关与商业 AP 中。受影响的芯片硬件横跨两大阵营：  
- **老旧 eCos 实时系统芯片（永久 Forever Day）**  
：包括 BCM3380、BCM3383 和 BCM33834 等型号。这些芯片在十几年前就已投产，厂商早已停止维护，博通从未针对这套微内核推出可用补丁，导致这些在网设备沦为终身无法修复的“活靶子”。  
  
- **较新的 Linux 固件芯片**  
：涵盖 BCM3390、BCM65xx 与 BCM68xx 系列。这类芯片虽然具备修补基础，但博通曾建议厂商直接抛弃旧 WPS 栈并改用开源 hostapd 替代方案，很多 OEM 硬件厂商并未跟进迁移。  
  
更耐人寻味的是，博通早在 2018 年就已知晓该缺陷，但始终未公开发布安全公告，也没有在 CVE 官方字典中注册编号。卡内基梅隆大学 CERT/CC 介入后，因博通本身是其产品线的主管 CVE 编号授予机构（CNA）且拒绝登记，导致协调披露当天便被归档关闭，直接造成大量下游运营商与用户多年蒙在鼓里。  
## 🛡️ 应急处置与近场无线防御建议  
  
针对近场无线“贴脸”攻击风险，企业与家庭网络管理员应立即采取以下防御措施：  
- **立即彻底禁用路由器的 WPS 功能**  
：登录网关管理后台，找到 WPS（Wi-Fi Protected Setup / 快速连接）选项并直接关闭。关闭 WPS 能直接阻断脆弱代码模块接收外部重组报文。  
  
- **利用 YARA 规则排查设备固件**  
：研究员已公开了针对脆弱重组逻辑的 YARA 静态检测规则与空口监听探测脚本，安全团队可用于对现网存量无线设备进行排查摸底。  
  
- **加速淘汰停更的老旧硬件资产**  
：对于搭载 BCM3380/BCM3383 等老旧架构、运营商无法推送固件更新的遗留光猫与网关，唯一的彻底根治办法是淘汰替换为支持现代安全标准的更新设备。  
  
 END    
  
  
阅读推荐  
  
  
[【安全圈】Skyticket 旅行预订网站遭黑客攻击，6140 名平台用户信用卡信息外泄](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079392&idx=1&sn=21e75e487b508bd20850baafe3114343&scene=21#wechat_redirect)  
  
  
  
[【安全圈】OpenCode披露安全漏洞，控制台用户第三方API密钥或遭泄露](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079392&idx=2&sn=065ddfcf11b031a8c154f04a0a06866e&scene=21#wechat_redirect)  
  
  
  
[【安全圈】伪造的 Arch Linux 软件包以 root 权限推送恶意软件](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079392&idx=3&sn=dc161ca3b5c81ea4e70e4d51d84b02c0&scene=21#wechat_redirect)  
  
  
  
[【安全圈】程序员为女友抢号搞瘫医院挂号系统](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079381&idx=1&sn=62452dec7ffbb0c77a86ee1e051297e4&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
