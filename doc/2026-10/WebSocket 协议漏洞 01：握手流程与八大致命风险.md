#  WebSocket 协议漏洞 01：握手流程与八大致命风险  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-05 01:12  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6N6DKjBps4tcGiaribuI4l14LGGT0xp7yFW5snzu9e1MHJThmwF92LukwtE6vbISJCfBU6Ficr6Kxia3WKk8kcJn8KnalnEwibWzVBU/640?from=appmsg "")  
> **导语**  
：聊天、股票行情、协同文档这些"实时"场景，几乎都是 WebSocket 在背后撑。它在 HTTP 之外开了一条全双工通道，却把 HTTP 那一套鉴权、输入校验、访问控制全抛给了开发者。今天先把协议基础和八大致命风险捋清楚，下一篇直接上 IDOR 与 DoS 实战。  
  
## 一、协议方案：ws 和 wss 不是同一种东西  
  
WebSocket 在浏览器端暴露成两个 URI 方案：  
- **ws://**  
 — 类比 HTTP，明文通信。和 HTTP 一样，握手包和后续消息帧全裸奔在网络里，咖啡厅 WiFi 抓包就能看到聊天内容。  
  
- **wss://**  
 — 类比 HTTPS，底层套 TLS（传输层安全协议）。握手包带证书校验，消息帧走加密通道，机场 WiFi 抓到的也是密文。  
  
光看 URL 头一个字母差，性质差一档：ws 等同于 HTTP，wss 等同于 HTTPS。渗透测试里看到业务跑 ws://，可以直接把 TLS 缺失列为高危。  
  
![WebSocket 握手流程](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6Nb2S1tnbjLELuAJvlaAXUdM7IibFictpibiaeZR1xxkC9oadk65brhAic3o5a5lZxUicjLewrAQE3LGqzZuEicw7B7dkpSDqBmZVZEAU/640?from=appmsg "WebSocket 握手流程")  
## 二、握手过程：HTTP 借壳上线，再也不回头  
  
WebSocket 没有自己发明握手，而是借 HTTP 升级机制开场。客户端发一个长得像 HTTP 的请求，带两个关键头：  
```
GET /chat HTTP/1.1Host: target.exampleUpgrade: websocketConnection: UpgradeSec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==Sec-WebSocket-Version: 13
```  
  
服务端同意就回 101 Switching Protocols  
，从此这条 TCP（传输控制协议）连接升级为 WebSocket 通道，后续全是 WebSocket 帧，不再有 HTTP 请求行。攻击者最爱的就是这个"借壳"动作——可以借助 HTTP 基础设施（CDN、反向代理、负载均衡）做隐蔽跳板，因为前几个包看起来人畜无害。  
  
要拆解握手，看 Sec-WebSocket-Key 和响应里的 Sec-WebSocket-Accept 是不是按 RFC 6455（WebSocket 协议标准）规则算的；不一致的话基本能判定是自研 WebSocket，要么是有洞，要么是 bug。  
## 三、消息交互：双向帧不止是文本  
  
握手完成后双方互发 WebSocket 帧，每帧有自己的操作码（Opcode，标识帧类型）和 Payload（载荷）。关键几类帧：  
- **0x1 文本帧**  
 — UTF-8 编码的字符串，最常见  
  
- **0x2 二进制帧**  
 — 字节流，音视频、协议转发常用  
  
- **0x8 关闭帧**  
 — 任意一方发，对端回一个就拆连接  
  
- **0x9 Ping / 0xA Pong**  
 — 心跳保活  
  
测的时候两个工具必备：  
- **Burp Suite**  
 — Proxy 直接拦 WebSocket 消息，每条都能改、重放、批量发，配合 Match & Replace 还能做注入  
  
- **Simple Web Socket 浏览器扩展**  
 — 不用启 Burp，直接在 DevTools（浏览器开发者工具）里当客户端连，配合历史 Payload 重放特别顺手  
  
![WebSocket 双向消息流](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NGRljz3B4dLqMcwQ9VicExylJr8PKCkREiaWlvda0pmYKBPXd7N8asLUXn8HAmtryQykiaiaDnhP6Fvo7TB4EhiayRaFP5OqdSX2kE/640?from=appmsg "WebSocket 双向消息流")  
## 四、八大致命风险清单  
  
按从底向上被砍次数排：  
1. **拒绝服务（DoS）**  
 — 协议不限制单 IP（互联网协议地址）连接数，循环握手+慢帧就能塞满服务端 fd（文件描述符）池。最阴的是"长连接挂空"，客户端连上不发心跳，服务端一直占着连接。  
  
1. **明文通信**  
 — 用 ws:// 跑敏感业务，TCP 重置攻击 + 中间人抓包就能把聊天记录、Token（身份令牌）一锅端。  
  
1. **访问控制缺失**  
 — HTTP 路由上有鉴权，WebSocket 升级包经常被反向代理当成普通流量放过去，业务层只校验登录态、不校验 WS 帧里的参数。  
  
1. **输入校验缺失**  
 — 服务端信任帧内容直接拼 SQL（结构化查询语言）/Shell/HTML，等于把 HTTP 那套注入面在 WS 通道上重演一遍。  
  
1. **认证缺失或不当**  
 — 升级包不带 Token、握手完成后才校验身份，会出现"先连接、再认证"的竞态，恶意脚本可在这中间发一帧拿到未授权数据。  
  
1. **隧道滥用**  
 — WebSocket 帧体是二进制，可以塞任意协议（DNS、SSH、ICMP 即互联网控制报文协议）。内网里被攻陷的 WebSocket 端点能直接成为出网跳板，绕过防火墙 egress（出向流量）规则。  
  
1. **跨站 WebSocket 劫持（CSWSH）**  
 — 类比 CSRF（跨站请求伪造），恶意页面诱导用户浏览器对目标域发起 WebSocket 握手，复用用户 Cookie 过鉴权。比 CSRF 更阴，因为连接是常驻的。  
  
1. **服务端 OOB（带外）问题**  
 — 协议解析器本身的内存越界、整数溢出，常出现在自研/魔改实现上。CVE-2023-38545 这种就是典型代表。  
  
## 五、下一篇预告  
  
光知道有这些风险还不够，下一篇 day14 直接上两个最常砍下来的：  
- **IDOR 横向越权**  
 — 聊天协议里 sid  
（会话标识）、uid  
（用户标识）这些参数不校验发起人，改一个值就能替别人发消息  
  
- **应用层 DoS**  
 — 循环建连 + 长帧体 + 慢消息三种打法，把服务端资源耗干  
  
中间夹一份可直接复用的 Burp 拦截模板。  
## 六、素材出处  
- Learn365 Day 13 原文：https://github.com/harsh-bothra/learn365/blob/main/days/day13.md  
  
- WebSocket Top 7 漏洞（Neuralegion）：https://www.neuralegion.com/blog/websocket-security-top-vulnerabilities/  
  
- RFC 6455：WebSocket 协议标准  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6PmibVggMZJ3dH2Mic0vjmGD6lSX0kAvicF3SgqqKGb4GpWccKwCO20XAqibCeHbJnib1XeaL1FwibYhtOacteeDVW8GI1MHsGOG1Ejg/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
