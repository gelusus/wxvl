#  Broadcom WPS 未修复漏洞细节公开:攻击者无需密码,可在路由器以 root 身份运行代码  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-20 01:08  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NGuJLHQZup2PlLF18fPHia0pKiaEibKcF15Mjven4r5g8LoO894KAt1hIgNY45jVX0C0SiaJQ7JxxuMg9jHzlWMLibyicpj2T2VDcYY/640?from=appmsg "")  
> **导语**  
:10 月 16 日公开 Broadcom WPS 分片重组漏洞——无线范围内发畸形分片帧无需密码,就能在路由器拿到 root,无 CVE、无补丁可用,旧 eCos 芯片被研究者称作「永久 0day」。  
  
## 一、漏洞概况  
  
Kaiser 在 GitHub 释出**指纹识别脚本**  
和**YARA 规则**  
的同时,在 The Hacker News 等渠道同步披露漏洞细节,核心要点如下:  
- 漏洞出在 Broadcom 自家版本 WPS 的**分片重组代码**  
,对畸形 WPS setup 消息处理不当,可导致路由器内存破坏;  
  
- 在此基础上,他构造了两类攻击载荷:**读取设备内存**  
(例如直接拿到 Wi-Fi 密码)和**在路由器上以 root 身份运行代码**  
;  
  
- 攻击条件非常轻:不需要 Wi-Fi 密码,不需要任何配对步骤,只要在无线覆盖范围内即可,稍远距离也可行;  
  
- 受影响的 Broadcom WPS 代码被嵌在几十家厂商的路由器和 AP 产品里,Kaiser 估计全球**数百万台**  
设备受影响;  
  
- 漏洞代码可追溯到 2010 年,Broadcom 在 2018 年给过一个不完整修复,并私下建议客户切到 hostapd,但对老一代 eCos 芯片**从未提供有效补丁**  
--这也是 Kaiser 将其称作 "forever day"(永远不会被打补丁)的原因。  
  
CERT 协调中心(CERT/CC)受理后,因为 Broadcom 自己掌握自家产品的 CVE 分配权、又拒绝分配,案件在当天被关闭。截至 10 月 19 日仍查不到该漏洞的 CVE 编号。  
## 二、攻击原理与已验证场景  
  
Kaiser 围绕 WPS 的分片重组路径做了两种 PoC:  
  
**读取 Wi-Fi 密码(Orange Belgium - Askey TCG3000)**  
:攻击者伪造分片触发内存破坏,直接把存储在路由器里的 PSK(Wi-Fi 密码)从内存里读出来,再以无线方式回传--设备因此被"破门",后续可正常加入网络。  
  
**空中开 shell(Proximus - Sagemcom BBOX3V+)**  
:借助 Wi-Fi 数据帧反弹远程 shell,整个过程不需要任何外部服务器,纯靠无线电通信完成命令执行。  
  
值得注意的是,这两类攻击**都不要求认证**  
,且对距离不敏感。Kaiser 表示尚未观察到真实环境中的利用行为,公开的利用工具都来自他自己。  
  
![基于 Broadcom WPS 的攻击场景演示(Linux 终端 + Wireshark 抓取 802.11 Beacon 与 WPS 行为)](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6Od0B4zXGkia1oefFlHT1fxgkvTPGcTlU1haibu93o0USJMMydqhGXMS7TuoqfFTiaJ0RvTnhbxkn2Oiamt3XUSJ6h36LIMa9al8hc/640?from=appmsg "基于 Broadcom WPS 的攻击场景演示(Linux 终端 + Wireshark 抓取 802.11 Beacon 与 WPS 行为)")  
## 三、受影响芯片与典型 OEM 型号  
  
Broadcom 受影响芯片分两族,治理思路也不同:  
- **eCos 旧芯片(无可用补丁)**  
:BCM3380  
、BCM3383  
、BCM33834  
,Kaiser 评估这类机型只能整机更换;  
  
- **Linux 系新芯片(可修补)**  
:BCM3390  
、BCM65xx  
、BCM68xx  
,前提是 OEM 切换到 hostapd 实现。  
  
Kaiser 在比利时实测到 Proximus、Orange Belgium、VOO 三家 ISP 的网关均受漏洞影响(第四家 Telenet 因缺乏样机未测试),每家受影响的网关粗估在数十万台量级。  
  
更长的型号清单里包含一长串读者可能熟悉的名字:Cisco SMB 系列的 RV110W/RV130W/RV215W/RV340W 与 WAP121/125/150/361/371/571/571E/581、ARRIS SURFboard AC1200、Arcadyan VRV9517(KPN 与 SPARK 在用)、Askey TCG3000 / RTF3505VW、华为 B310/B315/B593/WS880、NetComm NF4V/NF10W/NF10WV/NF8AC/NF17ACV/NF18ACV/NF18MESH、TP-Link Archer C3150/C5400/VR600/VR900/VR2800、ZTE F680 v4 / 368N 等。同时清单一并列出了**已验证未受影响**  
的型号(Sagemcom F5598T、F380B 等)--这些机型虽基于 Broadcom,但运行的是打过补丁的 hostapd,不在攻击面内。  
  
![受影响的 Broadcom 芯片及典型 OEM 设备示意](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NIZy6nkibW6H36S21RAibCY5G31TtmPQhiciancAfEoMmk6UB05taazLChBYVJx82R8ceiaw0zkoa1RscEKuaxuZj04zgqGpicSxaV0/640?from=appmsg "受影响的 Broadcom 芯片及典型 OEM 设备示意")  
## 四、检测与临时缓解  
  
在没有 CVE、没有补丁的情况下,Kaiser 把重心放在了**如何确认设备是否受影响**  
:  
- **YARA 规则**broadcom_wps_monitor.yar  
:在提取出的固件目录里扫描 ARM/MIPS ELF32 二进制,识别易受攻击的 WPS Monitor 代码路径。三条规则覆盖通用 WPS Monitor、HAL 配置事件变体以及"分片重组"路径,适合静态审计;  
  
- **在线指纹脚本**broadcom_fingerprint.py  
:把无线网卡切到 monitor 模式扫描信道,通过 Broadcom 私有 IE 加"发送零长度分片后是否回 NACK"两个信号判定目标 AP 是否在跑有缺陷的 wps_monitor  
。两条同时命中,可较有信心给出"可能易受攻击"的结论。  
  
README 也明确写了硬件坑点:Intel Dual Band Wireless-AC 7265(iwlwifi)虽然能切 monitor 模式,但**抓不到任何帧**  
,脚本会一直空转;真正可用的是 MediaTek MT7921AU 这类 USB 网卡,以及 mt76、ath9k_htc、rtl8812au 等已知能抓帧的芯片。  
  
临时缓解措施上,**最直接且无需等待厂商的做法是关掉 WPS**  
--多数运营商网关默认开启,一旦关闭,这条攻击路径就完全切断。其余可执行的步骤包括:升级到使用 hostapd 的新固件(若 OEM 提供)、对不会收到补丁的旧机型直接整机更换,以及部署无线侧流量监控,留意异常的 WPS 分片帧。  
  
Kaiser 特别提到,这种"隔着一条街就能动手"的攻击模式,与俄罗斯背景攻击组织曾使用过的"邻近 Wi-Fi 渗透"思路契合--攻击者并不需要触及目标的内网或互联网入口,只要走到物理信号覆盖范围内就能开始。  
## 五、写在最后  
  
这起事件是一份典型的"厂商消极、影响深远"的研究样本:Broadcom 早在 2018 年就已知晓,Kaiser 在 2026 年公开后,CERT/CC 因 Broadcom 拒绝分配 CVE 而当天关单。对个人用户而言,可立刻执行的动作只有"关 WPS"这一项;对运维团队而言,核心命题是"在无法等到补丁前,先把这一条攻击路径从暴露面里移出去"--尤其是仍在用运营商 eCos 老网关的家庭和分支机构,Kaiser 的清单与指纹脚本恰好是当前唯一可用的"分诊"工具。  
  
工具和检测脚本的开源地址:GitHub qkaiser/broadcom-wps  
(MIT 协议,114 Stars)。  
  
**版权声明**  
:本文由华盟网原创发布,保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6ObfjkJ89ibKU6vH6wj4EHcBhPcvLhDGdkUTUiby1Z9jCVysuwAFic5iayjJxz5Wjsawu5XZJbvDfbuHYOZxsR45Xr8kpq2EN7wLbQ/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
