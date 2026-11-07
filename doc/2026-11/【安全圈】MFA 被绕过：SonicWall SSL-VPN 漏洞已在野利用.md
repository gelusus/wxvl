#  【安全圈】MFA 被绕过：SonicWall SSL-VPN 漏洞已在野利用  
 安全圈   2026-11-07 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：CVE-2026-19868，CVSS 7.1，影响启用 SAML 认证的 SonicOS SSL-VPN 服务。持有另一种认证方式有效凭据的攻击者，可以绕过 SAML 认证控制，包括由 SAML 强制执行的 MFA。SonicWall 已确认收到在野利用报告，CISA 同步把它拉进 KEV 目录，联邦民事机构整改截止 11 月 20 日。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyFvLseZGn45Rdp8dHUnAVE78V3RAWl8UyLIkJ4PiakL4efphDJEhDlYZrxiar2O3lIJ3EoOsciaicTx42ic0Ddz1JUm189cktUoU97Y/640?wx_fmt=jpeg&from=appmsg "")  
  
  
## 🔍 绕过点：两条认证路径，一条不做检查  
  
SSL-VPN 通常允许用户从多个认证源登入。配置了 SAML 之后，管理员的预期是「走 SAML 就得过 MFA」。这个漏洞把预期打穿了：攻击者拿着另一种认证方式的有效凭据进来，SSL-VPN 没有把他们导回 SAML 流程，MFA 这一步被整段跳过。  
  
注意前提：**攻击者手上已经有一组能用的凭据**  
。这不是无凭据直通。真正麻烦的地方在于，MFA 是这类账号的最后一道锁，锁被绕过之后，一组泄露的旧密码就足够把人送进内网。  
- **受影响范围**  
：仅 SonicOS SSL-VPN 且配置为 SAML 认证的实例。未启用 SAML 的不受影响  
  
- **发现者**  
：The Electronic Office 的 Ryan Harris  
  
- **官方表态**  
：SonicWall PSIRT 已调查利用报告，强烈建议尽快升级。除升级外没有缓解措施  
  
## 📦 版本对照：该升到哪一版  
- **Gen7 受影响**  
：7.3.3-7015 及更早版本  
  
- **Gen7 修复版本**  
：7.3.4-7013  
  
- **Gen8 受影响**  
：8.2.2-8015 及更早版本  
  
- **Gen8 修复版本**  
：8.2.3-8015（同版本号修订）  
  
## ⚠️ 同一次更新里还有六个洞  
  
这次公告修的是 7 个问题，MFA 绕过只是其中之一。  
- **两个管理界面缺陷，评分 8.8**  
：特定条件下可让未认证攻击者执行未授权操作。评分比在野利用的那个还高  
  
- **SSL-VPN 与 TACACS+ 服务的认证前缓冲区溢出**  
：可造成拒绝服务  
  
- **虚拟设备弱口令哈希**  
：属于口令存储强度问题，长期隐患  
  
就算你的 SSL-VPN 没开 SAML，那两个 8.8 的管理界面缺陷也该让你把这个补丁打上。  
## 🎯 这是一条持续了四个月的攻击线  
  
SonicWall 的边缘设备最近几个月一直在被打，节奏相当密集：  
- **7 月**  
：两个 SMA1000 零日被利用，用于在设备上部署恶意软件  
  
- **9 月**  
：两个 SMA1000 零日被串联，达成远程代码执行  
  
- **10 月**  
：最高严重级别的 CVE-2026-102255 出现利用尝试，可让未认证攻击者访问内部功能  
  
- **11 月**  
：轮到 SSL-VPN 的 SAML 绕过，且已确认在野  
  
规律很清楚：攻击者盯的是边界设备。这类设备暴露在公网、常年不重启、固件更新要走变更流程，一旦某个环节拖延，窗口就打开了。  
## 🛡️ 现在该做的四件事  
1. **确认版本并升级**  
：Gen7 升到 7.3.4-7013，Gen8 升到 8.2.3-8015。没有 workaround，只能升  
  
1. **盘点非 SAML 认证源**  
：SSL-VPN 上凡是为「备用登录方式」开着的本地账号、LDAP 直连、其他 IdP，逐个确认是否还需要。关掉不用的，减少可被拿来绕过的入口  
  
1. **翻 SSL-VPN 登录日志**  
：查那些只走了密码、没有 SAML 断言记录的会话。重点看同一账号在短时间内从不同认证路径登入的情况  
  
1. **重置可疑账号凭据**  
：若日志里出现异常登入，把这批账号的口令全部轮换，并检查该账号在 VPN 内网侧的访问记录  
  
另外一个容易被忽略的动作：把 SSL-VPN 的管理入口从公网摘下来，只对可信网段开放。这条减少的不是某个漏洞的风险，而是整类漏洞的可达性。  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】Chainalysis 被打穿：全体用户凭据泄露](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079722&idx=1&sn=cbb0f9bdab2358a4b513dad213c41288&scene=21#wechat_redirect)  
  
  
  
[【安全圈】隐形后门寄生 CWP 面板，访问日志全程沉默](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079722&idx=2&sn=0ffae3851bf87f1c5fcafe01d9a44a92&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Barracuda 网关双漏洞链：免登录直取 root](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079722&idx=3&sn=c41a23a7212e4cf43702e04774a897e0&scene=21#wechat_redirect)  
  
  
  
[【安全圈】白宫专网被卖做代理？总统办公室设备疑遭植入后门](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079711&idx=1&sn=05bdbc8ba75d9805145596493772fbf6&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
  
