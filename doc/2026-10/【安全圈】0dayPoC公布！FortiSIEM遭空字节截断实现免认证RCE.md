#  【安全圈】0dayPoC公布！FortiSIEM遭空字节截断实现免认证RCE  
 安全圈   2026-10-21 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事件快报**  
：安全研究团队 Horizon3.ai 正式公开发布了针对 Fortinet 核心安全审计产品 FortiSIEM 的完整武器化利用 PoC（概念验证代码）。该漏洞编号为 CVE-2026-106173，CVSS 严重程度评分为 9.4。未经身份验证的外部远程攻击者，只需向 FortiSIEM 开放的 gRPC 服务发送精心构造的数据包，便可触发路径穿越与空字节截断，向服务器任意路径覆盖写入文件，进而将其转化为高特权远程代码执行（RCE）。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyEUxOX7ZR3IvwHpOokwAaCdibsGr1eoonBmWpw3GoO8muygwiaQQcJu7YADzUvOIS9fLxew8r2uEnHCyMiaSBLJiaXtNJNlQbHCBzc/640?wx_fmt=other&from=appmsg "")  
## 🔍 漏洞剖析：gRPC元数据校验缺失与空字节截断利用链  
  
根据 Horizon3.ai 释出的技术白皮书，FortiSIEM 的攻破链条极为紧凑且致命，主要发生在系统默认监听的 TCP 27918 端口上：  
- **第一步：无认证直连 gRPC 端口**  
：FortiSIEM 在 TCP 27918 端口运行的 gRPC 模块未做强制身份凭据校验，允许外部客户端直接建立 HTTP/2 会话并调用 QueryResultReq  
 接口。  
  
- **第二步：不安全的路径拼接**  
：处理函数 SendQueryResult()  
 接收客户端传入的 query-id  
 元数据参数时，未对目录穿越字符进行有效过滤，攻击者可输入 ../../  
 逃逸出受限的临时文件目录。  
  
- **第三步：巧妙规避强制后缀限制**  
：原本系统在拼接路径后会自动追加 result-<源IP>-0-0-<id>.dat  
 的固定文件名，导致攻击者无法自由指定落盘文件名。  
  
- **第四步：HPACK 空字节截断破局**  
：研究员发现在 gRPC 的 HTTP/2 编码与 HPACK 压缩层中，可以成功注入空字节（\0  
 / NUL Byte）。当底层 C/C++ 文件系统调用解析该路径时，遇 \0  
 立即视为字符串终止符，从而丢弃服务端强加的 .dat  
 后缀。  
  
## ⚡ 从任意文件覆盖到 Root RCE 闭环  
  
一旦突破了路径穿越与文件名限制，攻击者即可利用任意文件写入（Arbitrary File Write）原语，向目标操作系统核心目录投递后门：  
- 覆盖 Linux 系统的定时任务（/etc/cron.d/  
）或系统环境变量启动脚本。  
  
- 篡改 Web 控制台目录下的静态资源或后端处理逻辑，植入交互式 Webshell。  
  
- 由于 FortiSIEM gRPC 守护进程通常以高权限用户运行，攻击者借此直接斩获宿主机的 root 权限反弹 Shell。  
  
## ⚠️ 资产现状：PoC 已公开，在野扫描正在加速  
  
虽然 Fortinet 官方在近期释出了维护版本补丁，但随着 Horizon3.ai 将高可用度、低利用门槛的 PoC 代码在 GitHub 及公开安全社区放开，针对企业边界 FortiSIEM 资产的在野自动化嗅探正呈现指数级增长。  
  
作为企业核心的日志分析与威胁监控（SIEM）平台，FortiSIEM 一旦被黑客击穿，整网资产拓扑、安全告警记录以及内网凭据将尽收攻击者眼底。  
## 🛡️ 应急加固与处置防护建议  
  
各机构安全运维团队必须立即核查 FortiSIEM 的公网暴露面并落实下列修复措施：  
- **立即封堵 TCP 27918 端口**  
：在网络边界防火墙及路由器上，强制阻断来自未经授权的外网 IP 对 TCP 27918（gRPC）端口的访问，仅允许受信任的内网 Agent 节点通信。  
  
- **全面升级官方维护补丁**  
：核验系统版本，立即升级至 Fortinet 官方发布的对应修复固件版本。  
  
- **巡检关键系统目录异常文件**  
：排查 FortiSIEM 节点的 /etc/cron*  
、/tmp  
 以及 Web 部署路径，核验最近 72 小时内是否有非官方变更的异常文件落盘。  
  
 END    
  
  
阅读推荐  
  
  
[【安全圈】双11前紧急叫停！京东JSRC全面暂停外部漏洞测试](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=1&sn=64985dae9e27b64b33a95e031a7e8784&scene=21#wechat_redirect)  
  
  
  
[【安全圈】克隆代码就中招！94个GitHub开源库遭假字体投毒](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=2&sn=5347b52ee72fe6b67a3af750295fed29&scene=21#wechat_redirect)  
  
  
  
[【安全圈】四漏洞连环击穿！Plex遭武器化利用植入挖矿木马](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079414&idx=3&sn=c59fa44ef0a5791429f5ba4c7b36e7e4&scene=21#wechat_redirect)  
  
  
  
[【安全圈】连密码都不用！博通WiFi曝0day隔空秒拿ROOT](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079403&idx=1&sn=21c4f345cc81f0721be1f3a465c9026c&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
