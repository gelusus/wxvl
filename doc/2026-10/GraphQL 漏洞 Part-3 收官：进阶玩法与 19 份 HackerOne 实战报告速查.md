#  GraphQL 漏洞 Part-3 收官：进阶玩法与 19 份 HackerOne 实战报告速查  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-14 00:58  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NxuxgggaWibgBDALd1NQO86onrcYUKBS6Nzl67eOeZnj9Q7icSJhSPA3nsibbeiaKerFd8XicnoNo7RXkBMnsGldJ4amD0yCxfJ29o/640?from=appmsg "")  
> **导语**  
：GraphQL 三部曲前两篇把内省、DoS、IDOR、错误泄漏打透了，这篇是收官。讲那些藏在字段权限、订阅长连接、限流逻辑里的"进阶玩法"，再把 19 份 HackerOne 实战报告和 6 个深度资源按攻击类型重新归类，红队下次遇到 GraphQL 端点时直接照表打。  
  
## 一、三部曲一句话回顾  
  
第一讲拆的是"端点长什么样"：单端点架构、内省三件套（__schema  
 / __type  
 / __typename  
）、批处理别名绕过。第二讲拆的是"如何打挂和打穿"：深度递归 DoS、字段 IDOR、错误消息泄漏 ORM 内部 SQL。  
  
第三讲不重复基础，直接进入**权限边界模糊地带**  
——这些点防守方经常写一半就放过，攻击者拿分却最稳。  
  
![GraphQL 收官蓝图：四类进阶手法收口到一个端点](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6P45icDJZaZK0hH3d4RkeEMibPrhMvtd9EdTHqvABsGb2SH4XZpbpLt2od4ZpPCfAGW5VI4bmQFAEMhvWgyQU0Clf0icwGCdTPF0o/640?from=appmsg "GraphQL 收官蓝图：四类进阶手法收口到一个端点")  
## 二、四个最容易拿分的进阶手法  
### 2.1 内省被关后的字段枚举  
  
团队把生产环境内省关掉就以为安全了，其实关内省只是关了一扇门，攻击者照样能进。  
- **错误消息字段名泄漏**  
：发一个故意缺字段名的 query，服务端在 Cannot query field "xxx"  
 里**主动建议**  
正确字段名（Did you mean user / users / me?  
）  
  
- **GET 参数污染**  
：很多端点同时支持 GET query string，POST 关了内省 GET 开着  
  
- **Apollo Federation 多 schema 合并**  
：联邦模式下查询 _service { sdl }  
 直接拿全部子图 SDL  
  
- **客户端 JS 源码**  
：前端打包文件里往往能挖到完整 schema.graphql 一并传  
  
Bishop Fox 专门写过一篇文章《Design Considerations for Secure GraphQL APIs》，把"内省关掉就安全"列为一等误区，文章里给出完整的 schema 探测清单。  
### 2.2 订阅（Subscription）长连接滥用  
  
Query 和 Mutation 在 HTTP 短连接里跑，Subscription（基于 WebSocket）是个完全独立的长连接通道。很多团队的安全策略只覆盖了 HTTP 端点，WebSocket 这条暗道没人管。  
  
实战中最常被忽视的三类：  
- **订阅里的权限校验缺失**  
：用户 A 订阅了用户 B 的事件流，过滤逻辑只在前端加，服务端没复检  
  
- **订阅事件注入**  
：mutation 触发的字段没消毒，直接推给所有订阅者，等于把 XSS 投到所有在线客户端  
  
- **心跳与断线检测**  
：没设心跳的订阅会被反向用来占满 WebSocket 槽位，DoS 成本比 query 端低一个数量级  
  
第一讲里讲过 WebSocket 八大致命风险，订阅把这八条全继承了一遍。418883  
 这份 H1 报告就是订阅端权限绕过拿到后台数据流的案例。  
### 2.3 别名击穿限流器  
  
419883  
、614355  
、633001  
 三份报告里有同一个共同点：限流器按 HTTP 请求数计，而不是按 GraphQL 操作数计。  
  
攻击者把 200 个查询塞进一条 HTTP 请求（每个查询起一个别名），限流器算 1 个请求，resolver 跑了 200 次。结果：登录爆破、优惠码枚举、批量 IDOR 全部畅通无阻。  
```
query {a:login(user:"admin", pwd:"p1"){ token }b:login(user:"admin", pwd:"p2"){ token }c:login(user:"admin", pwd:"p3"){ token }# ...197 more}
```  
  
PortSwigger Academy 在 GraphQL 安全章节里专门写了别名绕过限流的检测手法，burp 插件inQL  
可以直接批量生成。  
### 2.4 GraphQL CSRF  
  
很多人以为"非简单请求要预检就安全"，实际上 GraphQL 端点接收 application/json 时确实会触发 CORS 预检，**但服务端接受 application/x-www-form-urlencoded 时，预检就跳过了**  
。攻击者写一段 form 自动提交，浏览器不带 cookie 也要带 cookie——只要用户的 session cookie 没设 SameSite=Lax/Strict，照样被打。  
  
417170  
 这份报告里就有一段经典 PoC：<form action="https://target/graphql" method="POST" enctype="text/plain">  
 把整段 query 塞进 input value，CSRF 一次投递完成。  
## 三、19 份 HackerOne 报告按攻击类型速查  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">攻击类型</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">代表报告</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">内省泄漏</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">885539</span></code><code><span leaf="">645299</span></code><code><span leaf="">707433</span></code><code><span leaf="">924914</span></code><code><span leaf="">978143</span></code><code><span leaf="">715192</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">字段/对象 IDOR</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">489146</span></code><code><span leaf="">608656</span></code><code><span leaf="">800231</span></code><code><span leaf="">898528</span></code><code><span leaf="">924914</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">限流击穿（别名/批处理）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">419883</span></code><code><span leaf="">614355</span></code><code><span leaf="">633001</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">错误消息泄漏</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">423388</span></code><code><span leaf="">707433</span></code><code><span leaf="">924914</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">订阅/实时流越权</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">419883</span></code><code><span leaf="">633001</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CSRF / 跨域滥用</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">417170</span></code><code><span leaf="">608656</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">嵌套查询 DoS</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">885539</span></code><code><span leaf="">978143</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">注入（SQLi/SSRF/SSTI）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">645299</span></code><code><span leaf="">489146</span></code><code><span leaf="">898528</span></code></td></tr></tbody></table>  
这 15 个 ID 是原文 day22 给的清单，剩下 800231898528924914978143715192  
 五个按上下文归到多个类别，说明它们往往是组合拳。原始报告全部需要登录 HackerOne 才能看完整 PoC，但每个标题都标注了漏洞类型，照着去找就行。  
## 四、6 份深度资源速读  
- **Security-Simplified《GraphQL Penetration Testing》**  
：印度安全研究员写的实战 blog，覆盖从端点发现到利用的完整流程，适合入门第一篇  
  
- **YouTube Wb0BO8J7024**  
：用动画演示 batch + alias + introspection 怎么组合打，红队培训常用  
  
- **Pen Test Partners《Pwning WordPress GraphQL》**  
：WPGraphQL 0.2.3 多 CVE 实战（CVE-2019-9879/9880/9881），演示一个未鉴权 mutation 如何创建管理员账号、再用主题编辑拿 shell  
  
- **ThreatNinja《GraphQL Injection Tutorial》**  
：把 SQLi / NoSQLi / SSRF / SSTI 套到 GraphQL 字段上做演示的工具集  
  
- **Bishop Fox《Design Considerations for Secure GraphQL APIs》**  
：从架构层面讲 SDL 切片、Resolver 隔离、Persisted Query，可作为企业部署参考  
  
- **PayloadsAllTheThings/GraphQL Injection**  
：GitHub 仓库，渗透测试 cheat sheet 的标配，复制即用  
  
## 五、红队收官 Checklist  
  
照下面这张表走，10 分钟内出第一个 lead：  
- /graphql  
 端点发现（OPTIONS、错误页路径扫、JS 提取）  
  
-  内省是否开着（{__schema{types{name}}}  
 一发命中）  
  
-  内省关了用错误消息字段建议挖 schema  
  
-  批处理 + 别名跑限流击穿  
  
-  深度递归 fragment 测 DoS 阈值  
  
-  mutation 字段遍历做 IDOR 测试  
  
-  subscription 走 WebSocket 单独测越权  
  
-  content-type 切 text/plain  
 / application/x-www-form-urlencoded  
 测 CSRF  
  
-  错误响应里看 ORM 内部 SQL（信息泄漏）  
  
-  拿到第一个 lead 后跑 inQL  
 / graphqlmap  
 / clairvoyance  
 做深度  
  
工具栈：inQL  
（Burp 插件）+ graphqlmap  
（自动注入）+ clairvoyance  
（schema 恢复）+ graphql-playground  
（手动调试）。三个加起来 30 分钟出报告。  
## 六、写在最后  
  
GraphQL 不是"新东西"，但它把"后端的事"搬到客户端来做的设计让权限、深度、错误处理这些传统安全控制变得格外脆弱。learn365 day20-22 三篇合起来是一张完整的攻击地图：第一篇定位、第二篇打穿、第三篇收口。  
  
下回开新坑，比如 OAuth 那一档的 day23 Password Reset Token。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6MwxLLT4oKxSUl8dT0xqP0ZzDwMOKqr7vT67EpBWTkLibVeQUfUsjn35bpKv3bykfgticbd34YNxJl5iblZnOrUTsxyVserQFvFbc/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
