#  【安全圈】四漏洞连环击穿！Plex遭武器化利用植入挖矿木马  
 安全圈   2026-10-20 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事件快报**  
：开源多媒体服务器巨头 Plex Media Server 曝出名为“Plex2Shell”的高危漏洞利用链。安全研究员公开技术细节及概念验证（PoC）后不足 24 小时，在野攻击者已将其完全武器化，对公网暴露的未打补丁服务器展开大规模全自动扫描与远程代码执行（RCE）。黑客攻陷服务器后，立即植入 XMRig 门罗币挖矿程序与隐蔽 Linux 代理后门，将受害者主机接入僵尸网络。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyHGAqgDoMAVPWuLEKMcxdevFf9HerkghtGSVatxJWc0Vb7ltjicbiaFmASYSDfnAxypWSgKcGXoHObTGJvOsCxkgtd1rhhZGPaZ4/640?wx_fmt=other&from=appmsg "")  
## 🔍 连环漏洞机理：四重弱点串联实现无认证 RCE  
  
根据披露报告，Plex2Shell 利用链并非单一漏洞，而是由安全研究员 Zach Main 提交的 4 个中高危缺陷环环相扣构建而成。攻击者完全无需身份认证，仅通过外部 HTTP 请求即可一步步提权并最终在宿主系统落地执行任意二进制指令：  
- **第一环 · 无认证 SSRF（CVE-2026-96652，CVSS 5.3）**  
：服务器 /player/timeline  
 接口对传入的 protocol  
 参数缺乏合法性校验，将其直接拼接进系统播放器请求 URL，使外部未授权攻击者能够诱导 Plex 服务器向内网或指定目标发起任意 HTTP POST 请求。  
  
- **第二环 · 参数注入与管理员令牌失窃（CVE-2026-96654，CVSS 6.9）**  
：/searchOne  
 接口存在 URL 过滤中和缺陷。攻击者借此调用内部插件未公开函数并注入自选参数，强行提取出服务器属主的完整超级管理员凭证（X-Plex-Token）。  
  
- **第三环 · 认证后转码通道 SSRF（CVE-2026-96655，CVSS 5.3）**  
：获取属主令牌后，攻击者向 /video/:/transcode/universal  
 路径发起调用，驱使 Plex 服务端拉取攻击者预先架设的载体流媒体，该媒体文件中暗藏了恶意的 Linux ELF 动态链接库。  
  
- **第四环 · 路径穿越与目录任意写入（CVE-2026-96656，CVSS 8.6）**  
：Plex 内部对编解码器（Codecs）的下载存储路径校验失效。攻击者将恶意动态链接库（.so）强行落盘至 Codecs 目录并触发动态加载，瞬间完成宿主机层面的非授权代码执行。  
  
## ⚡ 在野武器化分析：挖矿与流量中继双重压榨  
  
网络威胁监测机构 boredhackerblog 与多地蜜罐日志显示，攻击者在攻陷宿主机后会迅速通过 curl 或 wget 下载名为 x12345  
 的 Shell 初始化脚本，并同时投放两个关键恶意进程：  
- **XMRig 门罗币挖矿程序**  
：进程被重命名为 mylovelyminer  
，连接至外部矿池节点 ota.proxystream[.]st  
，强行吃满宿主机的 CPU 核心与 GPU 计算算力。  
  
- **隐蔽 Linux 代理后门**  
：恶意载荷 myawesomeproxy  
 在后台静默运行。VirusTotal 样本分析确认其为流量穿透与反向代理代理件，运行时直接打印提示免责声明，将失陷主机的家庭或机房公网出口转变为商业代理流量池。  
  
- **关联僵尸网络**  
：Bitsight 威胁情报确认，回连域名 global.relay.proxystream[.]st  
 与此前活跃的 AISURU / Kimwolf 僵尸网络直接同源，表明该批失陷设备已被整体收编入分布式代理网络。  
  
## 🛡️ 受害范围与紧急阻断处置方案  
  
Plex 官方已在 **Plex Media Server 1.43.3.10861**  
 及以上版本中彻底修复了上述漏洞链。但由于媒体服务器通常部署在家庭 NAS、自建 VPS 或私有云上，自动更新率相对较低，公网仍有大量资产裸奔。建议管理员立即排查：  
- **核对版本并立即升级**  
：检查当前 Plex 服务端版本，凡低于 1.43.3.10861 的实例必须立即停机升级；升级完成前切勿将 32400 端口直连公网。  
  
- **排查可疑进程与计划任务**  
：在 Linux 控制台运行 ps aux | grep -E "x12345|mylovelyminer|myawesomeproxy"  
，发现立即强行杀除；检查 /etc/crontab  
 及用户定时任务目录。  
  
- **清理 Codecs 缓存目录**  
：定位 Plex 数据根目录下的 Codecs  
 文件夹（如 /var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Codecs/  
），检查近期新生成的未知 .so  
 文件。  
  
- **封禁攻击者 IP 与域名**  
：在防火墙上对以下已知攻击源与 C2 节点配置双向拦截：  
  
- 217.60.194[.]116  
（恶意脚本分发站）  
  
- 91.92.40[.]202  
、176.65.139[.]228  
、45.194.92[.]222  
（在野扫描利用源）  
  
- ota.proxystream[.]st  
、global.relay.proxystream[.]st  
（C2 代理与矿池）  
  
- **重置 Plex 账户密钥与访问令牌**  
：登录 Plex 官网，在账户安全中选择“注销所有设备”（Sign out connected devices），强制失效旧版令牌以防凭据被持续复用。  
  
 END    
  
  
阅读推荐  
  
  
[【安全圈】连密码都不用！博通WiFi曝0day隔空秒拿ROOT](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=1&sn=21c4f345cc81f0721be1f3a465c9026c&scene=21#wechat_redirect)  
  
  
  
[【安全圈】密码MFA全成摆设！黑客借云备份打穿数百企业M365](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=2&sn=cb2f9ac966ca8ee1464bccf38c83517c&scene=21#wechat_redirect)  
  
  
  
[【安全圈】一夜被盗刷上千刀！OpenCode失陷大模型Key全泄](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=3&sn=f20efa338f57605e69383fafbd726697&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Skyticket 旅行预订网站遭黑客攻击，6140 名平台用户信用卡信息外泄](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079392&idx=1&sn=21e75e487b508bd20850baafe3114343&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
