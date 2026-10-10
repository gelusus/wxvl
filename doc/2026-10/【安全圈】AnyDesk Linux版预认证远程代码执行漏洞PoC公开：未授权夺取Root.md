#  【安全圈】AnyDesk Linux版预认证远程代码执行漏洞PoC公开：未授权夺取Root  
 安全圈   2026-10-10 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心威胁快报**  
：远程桌面运维工具 AnyDesk 曝出致命安全隐患。安全研究团队于 GitHub 公开发布代号为“AnyPwn”的完整工作级 Exploit，针对 Linux 版 AnyDesk 会话协议堆缓冲区溢出缺陷，攻击者无需任何认证凭据、无需目标用户点击确认，即可在建立连接前直接获取服务器 Root 最高系统控制权。  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/sbq02iadgfyFWiaNs6NUXUFhf3FFfqRzytUuHVyImOEgyGPYm3qMO4QdTscamMab3mGVC8HHyhlrH0eP1hiaSfe4icefCUCt0oWZkeibKibKpjfXw/640?wx_fmt=gif&from=appmsg "")  
## 🔍 披露回顾：官方“崩溃修复”实为高危预认证 RCE  
  
该漏洞由 V12 安全团队研究员 Rick de Jager 借助代码审计引擎发现。研究团队于 6 月 22 日向 AnyDesk 提交了漏洞细节，官方在次日确认并在 8.0.3 版本中推送了修复补丁。  
  
值得警惕的是，AnyDesk 在 8.0.3 更新日志中仅以轻描淡写的“修复了可能导致崩溃的错误（fixed a bug that could lead to a crash）”一笔带过，既未申请 CVE 编号，也未发布任何安全通告。在研究团队于 10 月 8 日正式公开 PoC 概念验证视频及利用脚本后，官方下载页面更是直接下架了受影响严重的 8.0.2 版本安装包。  
## ⚡ 溢出机理：32位整数溢出触发堆结构破坏  
  
漏洞根源潜伏在 AnyDesk 会话协议底层处理逻辑中，攻击链条清晰精密：  
- **协议特征**  
：AnyDesk 采用基于模式 5（mode-5）的流式数据包协议。在为该数据包分配内存时，程序将数据包声明的负载长度与 16 字节头部（0x10）直接相加。  
  
- **无检查整数回绕**  
：底层计算采用原生 32 位无符号整型运算，且未作任何溢出边界检查。利用程序故意声明负载长度为 0xFFFFFFF0  
。当加上 0x10 时，计算结果发生 32 位整型回绕，运算值直接归零。  
  
- **错配写入与越界覆盖**  
：内存分配器误判为仅需极小缓冲区，但在对象结构体内部仍然忠实记录了原始声明的巨额长度。当接收哪怕仅仅 1 个字节的数据时，写入指针便彻底突破缓冲区边界。  
  
- **ROP 链稳定提权**  
：越界写入精确破坏相邻堆块的核心控制字段。Exploit 紧接着利用精心构造的面向返回编程（ROP）调用链，劫持控制流并以后台运行的 root  
 权限直接执行任意系统命令。  
  
## 🌐 攻击面分析：端口直连与中继穿透隐患  
  
目前公开的 AnyPwn 利用工具专门针对 AnyDesk Linux 8.0.2 构建，依托 TCP 7070 默认监听端口直连生效。由于属于堆内存概率性利用，若堆布局出现偏差可能会导致服务进程直接崩溃。  
  
尽管 AnyDesk 曾表态该漏洞仅影响直连模式、不波及官方中继（Relay）服务器，且 Windows 和 macOS 客户端不受该具体二进制构建影响；但安全团队通过 Frida 动态插桩测试证实，有缺陷的代码路径在穿越官方中继服务器时同样可以被触发。  
## 🛡️ 应急加固与缓解策略  
- **立即升级客户端**  
：Linux 运维主机应立刻检查 AnyDesk 运行版本，必须全量升级至 8.0.3 或当前最新 8.1.0 版本。  
  
- **关闭外部 TCP 7070 暴露**  
：在主机防火墙（iptables/ufw）及外部网关边界，严格限制或封堵外网对 TCP 7070 端口的访问请求，仅允许内网受信任源访问。  
  
- **生产环境主机审计**  
：全面核查 Linux 生产服务器上的远程桌面后台守护进程，遵循最小权限原则，避免运维服务以 Root 特权长期监听开放端口。  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】SonicWall曝10分满分漏洞：无需凭据直穿内网核心](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079285&idx=1&sn=40d7e7b43c34047213b376da287167d1&scene=21#wechat_redirect)  
  
  
  
[【安全圈】大模型缓存LMCache曝严重0day：无补丁直接RCE](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079285&idx=2&sn=5e9f5b16350c4daf6a1e4984eaf8184b&scene=21#wechat_redirect)  
  
  
  
[【安全圈】黑客劫持三国顶级域名注册局：非法签发Google证书](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079285&idx=3&sn=5856c4cc9c9c560a2b4c9fedec4fb02c&scene=21#wechat_redirect)  
  
  
  
[【安全圈】百余网站遭挂马植入伪Cloudflare：智能合约派发木马](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079274&idx=1&sn=15cf3e01caa2b91119ad90f970c2db20&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Anthropic放开Claude安全限制：实测挖出12.9万漏洞引发争议](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079274&idx=2&sn=2ac9ed707c794a32564ec1e4c3ba5722&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
