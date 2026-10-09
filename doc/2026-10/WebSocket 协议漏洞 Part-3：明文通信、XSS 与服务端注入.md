#  WebSocket 协议漏洞 Part-3：明文通信、XSS 与服务端注入  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-09 00:40  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6OcHGhOrfpkJuuibOn2DOrZuScoOEaiadb85Xb6VTSwoUT7zH0Ms040aLozVk9H9pXrCF6fAcvUWQIq8pk3e4ID8u0k3oJvia1ed4/640?from=appmsg "")  
> **导语**  
：前两篇我们已经把握手协议、IDOR 越权和连接洪泛讲透。第三篇专门拆剩下的五个攻击面——明文传输泄漏、服务端注入、XSS  payload——很多应用在 WebSocket 层完全没有做输入校验，和 HTTP 层共用同一个后端接口，等于把一块原本就脆弱的攻击面重新开了一遍口子。  
  
  
![WebSocket 五大攻击面全景图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NSNaFWiaCohEeNLs5aCWVSZ0XSZ53w2zQGFxv9lb8mgFh6g75T4KZKr1hPIWHq8shbOxg73qKHxTOQLbImwMC2RFHDZcCNjpPo/640?from=appmsg "WebSocket 五大攻击面全景图")  
## 一、未加密连接：WS 明文传输等于裸奔  
  
WebSocket 协议提供两种传输方案：ws://  
（明文）和 wss://  
（加密）。差别就是 HTTP 和 HTTPS 的关系——如果应用在 WS 明文通道里传输登录态、JWT token、聊天内容，中间人攻击（MitM，Man-in-the-Middle Attack）分分钟能拿到全部数据。  
  
测试流程很简单：  
1. 抓包看有没有 wss://  
，如果没有直接存在明文传输问题  
  
1. 拦截 WS 连接请求，看有没有 Sec-WebSocket-Protocol  
 头带敏感信息  
  
1. 如果应用支持 wss://  
，尝试把连接降级回 ws://  
，看服务端会不会拒绝  
  
**红队价值**  
：在公共 Wi-Fi 或已控局域网内，明文 WS 比 HTTPS 还容易抓——因为 WS 帧不经过常规代理，很多工具直接解析原始 TCP 流即可。  
## 二、服务端注入：WebSocket 直达后端接口  
  
很多应用的 WebSocket 连接后直接对接后端数据库或 API。这意味着在 WebSocket 消息里注入 SQL payload，效果和 HTTP 请求里的 SQL 注入完全相同。  
  
测试步骤：  
1. 注册一个账号，在 WebSocket 客户端里正常发消息  
  
1. 在消息体里找 ID、用户名、房间 ID 这些参数  
  
1. 把这些参数换成 SQL 注入 payload，比如 ' OR 1=1--  
  
1. 看服务端返回有没有报错、延迟异常或数据泄露  
  
SQL 注入只是最明显的，服务端注入还包含命令注入（OS Command Injection）、模板注入（Template Injection）、NoSQL 注入——只要后端没有单独对 WS 消息做输入校验，所有 HTTP 层的注入攻击在 WS 层照样有效。  
## 三、跨站脚本（XSS）：WS 消息写哪儿，XSS 就出在哪儿  
  
WebSocket 上的 XSS 分两种场景。第一种是消息被前端回显到界面——比如聊天室、通知面板、仪表盘——攻击者发一条带 <script>  
 的 WebSocket 消息，所有在线用户就会中招。第二种是盲 XSS（Blind XSS），消息被存在后台数据库里，管理员在后台面板查看时才触发。  
  
实战检验步骤：  
1. 向 WebSocket 发送 XSS payload，比如 <img src=x onerror=alert(1)>  
，看能不能回显  
  
1. 在 WebSocket 的 JSON body 字段里找可以注入的字段，比如 {"message":"{{xss}}"}  
 格式  
  
1. 如果有管理后台，攻击者往服务器写一条含 payload 的消息，等管理员后台加载  
  
1. 尝试将载荷改成 XSS Polyglot（多态），绕过各种过滤规则  
  
几个真实案例：GitHub SecurityLab 披露过 WebSocket XSS 导致 RCE 的链子；HackerOne 上有报告 ID #409850 就是 WebSocket 消息被前端渲染后触发 DOM XSS。  
## 四、敏感信息泄漏：WS 流量里的宝藏  
  
WebSocket 是全双工的——不像 HTTP 一次请求一次响应，WS 连接一建立，前后端持续交换数据。测试时把所有 WebSocket 消息里的内容逐条看一遍，重点找：  
- Token、Session ID 是否明文携带  
  
- 用户 ID、房间 ID 等内部标识是否可枚举  
  
- 后端返回的错误堆栈（Stack Trace）是否暴露数据库结构  
  
- WebSocket Sec-WebSocket-Protocol  
 头有没有放 API Key  
  
这个动作和日常 HTTP 渗透里扫响应体的逻辑完全一样，区别在于 WS 是持续流，消息量大、粒度细，容易漏。  
## 五、综合检查清单  
  
把前三篇的攻击面归总，做渗透测试时按顺序走：  
- WS:// 是否明文传输 → 抓包确认，降级测试  
  
- WebSocket 消息里的 ID/Token 能否篡改 → IDOR 测试  
  
- WebSocket 参数有没有做 SQL/命令注入校验 → Fuzz 注入  
  
- 消息是否回显到前端 → XSS payload 测试  
  
- 连接数是否有限流 → 洪泛 DoS 测试  
  
- Sec-WebSocket-Key  
 随机性 → 密钥强度检验  
  
**WebSocket 三部曲到此收尾。下一篇进入原型链污染（Prototype Pollution）——这个漏洞既影响 JavaScript 对象原型，又能通过 JSON 接口从 HTTP 打到 NoSQL，是渗透测试里"一个 payload 穿两层"的经典玩法。**  
## 素材出处  
- 原文：harsh-bothra/learn365 day15  
  
- PortSwigger WebSocket 安全实验室：https://portswigger.net/web-security/websockets  
  
- HackerOne 相关报告：#178990 / #409850 / #395729 / #163464 / #512065 / #1023669 / #86283  
  
- GitHub SecurityLab：#65 WebSocket XSS → RCE 链  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PCxiahJ4HG6Y7NILVJibRrTGe9DDQXa5ic7elw8vlDZNBuuwemJyStrwZ3FnvzficTzbXicd8IuicSwZica9NCicQqSzLeCInrAHZcfNE/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
