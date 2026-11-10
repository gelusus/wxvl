#  【安全圈】CISA紧急通报！SonicWall防火墙在野绕过MFA漏洞被破  
安全圈
                    安全圈  安全圈   2026-11-09 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：2026 年 11 月 9 日最新通报，网络边界安全巨头 SonicWall 正式发布紧急固件补丁，修复其 SonicOS 防火墙系统中的 7 处安全漏洞。其中包含一处已在野遭到真实攻击利用的高危缺陷（CVE-2026-19868，CVSS 7.1）。拥有备用认证凭据的黑客可直接穿透 SAML 身份验证链路，绕过多因素认证（MFA）强行接入企业内网。美国网络安全与基础设施安全局（CISA）已将其列入“已知被利用漏洞”（KEV）目录，下令联邦机构限期强制升级。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyHSNdXQa0a8h615DGo7RSibibhEyQTCibVExMELsxo7eykfxDeGMHiaxUvel8E4xOMtM1rnk5PdgewHF2Tst2vc0ibUuicxmkQIA1sTQ/640?wx_fmt=jpeg&from=appmsg "")  
  
  
## 🔍 漏洞机理：SAML 多因素认证验证流失陷  
  
本次在野攻击的核心目标是企业广泛使用的 SonicOS SSL-VPN 网关服务：  
- **攻击路径拆解（CVE-2026-19868）**  
：当防火墙 SSL-VPN 开启 SAML 单点登录与 MFA 双因子验证时，若系统同时配置了备用认证机制，攻击者只要掌握该备用方式的有效凭证，即可触发状态机校验分支缺陷，使网关在未完成 SAML 强认证的情况下误判通过，直接建立内部 VPN 隧道。  
  
- **双重管理端高危缺陷（CVSS 8.8）**  
：本次更新同步修复了两处位于 Web 管理界面的权限绕过漏洞。未经身份验证的远程攻击者在特定网络条件下，可直接向底层系统投递越权操作指令，篡改网关策略或导出配置文件。  
  
- **预认证缓冲区溢出隐患**  
：更新还封堵了 SSL-VPN 服务和 TACACS+ 协议处理模块中的多处缓冲区溢出缺陷，攻击者构造恶意载荷可在无需登录状态下直接造成服务崩溃（DoS）与守护进程死锁。  
  
## ⚡ 影响范围与受灾版本矩阵  
  
此次安全漏洞横跨 SonicWall 当前主流的第七代与第八代硬件平台：  
- **受波及版本**  
：  
- SonicOS Gen7 平台：7.3.3-7015  
 及所有更早版本；  
- SonicOS Gen8 平台：8.2.2-8015  
 及所有更早版本。  
  
- **官方安全修复版本**  
：  
- SonicOS Gen7 升级至：7.3.4-7013  
；  
- SonicOS Gen8 升级至：8.2.3-8015  
。  
  
- **厂商明确无临时替代方案**  
：SonicWall PSIRT 官方在通告中强调，针对 CVE-2026-19868 没有可行的软件层临时规避方案（No Workarounds），升级固件是阻断在野攻击的唯一有效手段。  
  
## 🛡️ 企业网络管理员应急响应清单  
  
网络边界作为企业第一道防线，各运维及安全团队应迅速执行排查：  
- **紧急升级 SonicOS 固件**  
：立即核对控制台防火墙系统固件，将 Gen7 与 Gen8 设备更新到目标修复版本，重启生效。  
  
- **审查 VPN 异常认证日志**  
：重点调取近 30 天内 SSL-VPN 认证日志，过滤是否存在从非常用公网 IP 发起的 SAML 登录跳过记录、以及短时间内备用凭据频繁验证的异常行为。  
  
- **彻底关闭公网管理端口**  
：关闭 WAN 侧的 HTTP/HTTPS 设备管理接口，仅允许通过经过认证的专属内网或专用堡垒机访问网关后台。  
  
- **轮换管理员与服务凭据**  
：对防火墙本地管理员密码、虚拟应用设备连接密钥执行一键轮换，防范攻击者已利用历史信息完成横向立足。  
  
  
  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】GPT-6 Cyber发布！漏洞挖掘成本降到千美元级别](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079766&idx=1&sn=a136a27f787a740a8706522b59e326c9&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Gnosis Pay用户数据泄露！卡号CVV钱包地址全曝光](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079766&idx=2&sn=cd94ac373cb548a747530b88b9d59ddb&scene=21#wechat_redirect)  
  
  
  
[【安全圈】ARTEX渗透框架闭源！微步在线声明版权收购](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079766&idx=3&sn=987af32e93ea7154cfe5fbf903e7e9dd&scene=21#wechat_redirect)  
  
  
  
[【安全圈】点了「别再问我」，Claude Code 把密钥写进了公开仓库](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079734&idx=1&sn=ea38ca74c4cda25bed3333c763aeea6b&scene=21#wechat_redirect)  
  
  
  
[【安全圈】补丁打完才安全？WordPress 更新窗口正被抢跑](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079734&idx=2&sn=71e1d2808ab34462f66cf9ae5f2aad67&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
