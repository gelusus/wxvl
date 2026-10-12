#  GraphQL 漏洞第一讲：从内省查询到 IDOR 全攻击面  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-12 00:40  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6N0GB52llNvDfpydt8qIAzJ7yRYoN0b2PC2ibH1DicdibcqeVqmXQMic9tKYMQN6j2icJrOFQqeC8fHxdOIftcQ8icGg9g8v5HNroEZY/640?from=appmsg "")  
> **导语**  
：GraphQL（一种由 Facebook 开源的 API 查询语言）用一个端点替代 REST 一整套路由，查询、写入、订阅都塞进同一条 POST。这把双刃剑的另一面是：内省开着等于把数据模型直接送给攻击者，再加上参数未过滤、批处理滥用、CSRF 防护缺失，整条攻击链在数小时内就能跑完。  
  
## 一、GraphQL 是什么：单端点的双刃剑  
  
传统 REST API 每个资源都对应一个 URL，查询、写入、删除各走一条路径。GraphQL 不一样，它把整个后端的数据模型抽象成一张"图"，客户端只用一个端点（通常是 /graphql  
），发一条结构化查询就能精确取回想要的字段，不多取也不漏取。  
  
前端不用再为每个新字段等后端出接口，坏处也在这里：同一个端点承载了所有读写逻辑，攻击者只要摸到一个 URL 就拿到了整个系统的入口。  
  
GraphQL 文档里有三类操作：**query**  
（只读，类似 GET）、**mutation**  
（写入并返回，类似 POST/PUT/DELETE）、**subscription**  
（长连接推送）。攻击者偏爱 mutation，因为绕过认证就能直接改数据。  
  
数据模型用 Schema Definition Language（SDL，模式定义语言）定义，规定每个类型的字段、类型、是否可空。SDL 看着像配置文件，实际上就是后端的真值来源——前端、后端、文档、IDE 自动补全都靠这一份定义。  
## 二、内省查询：把数据模型直接送人  
  
内省（Introspection）是 GraphQL 内置的元数据查询能力。任何 GraphQL 端点都暴露三个特殊字段：  
- __schema  
：返回整个 schema 的结构  
  
- __type  
：返回某个具体类型的字段定义  
  
- __typename  
：返回被查询对象的类型名（字符串）  
  
这套机制本来是给客户端做自动补全和文档生成用的，但攻击者拿它来绘制整个后端地图。  
### 2.1 探测端点是否存在  
  
先用一个最小查询确认目标是不是 GraphQL：  
```
{ __typename }
```  
  
所有 GraphQL 端点都会回 {"data": {"__typename": "Query"}}  
。如果对方回 404，说明不是 GraphQL；如果回别的 JSON 结构，说明端点协议不对。  
  
常见端点路径：/graphql  
、/api  
、/api/graphql  
、/graphql/api  
、/graphql/graphql  
。PortSwigger 文档建议五条挨个试，再叠上 /v1  
 后缀。  
  
内省开着的话，下面的查询能把整个 schema 拖出来：  
```
{  __schema {    queryType { fields { name description } }    mutationType { fields { name description } }    types { name description }  }}
```  
  
返回里包含每个 query、mutation、subscription 的字段名和注释。实战中很多团队把字段描述写成"删除用户""重置密码"这种直白文字，相当于把攻击指南直接挂在 API 上。Hasura 教程给了更精简版本：只想找 mutation 就只查 mutationType  
。  
  
HackerOne 报告 #291531 就是典型路径：访问 /graphql  
，运行内省，拿到包含 PII（个人可识别信息）字段的完整 schema。报告 #1132803 中，running.com/en-in/graphql  
 一次拉出所有查询、类型、字段、可用操作。HackerOne 官方博客 2024 年 7 月还专门写过第三方子域的 GraphQL 端点开了内省，主站促销横幅系统被拖出全部能力列表——攻击者顺势拿到 mutation 入口，绕过身份验证。  
## 三、八大攻击面与四步探测法  
### 3.1 攻击面全景  
  
![GraphQL 攻击面全景](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6ONWkr43UEk0dxDJkghBphavic1q5MibSBjJAypK3x1qSz3IoFZibSS62OSZWhg0ibz733txtWPcLQ5hSXFBoYKxH07XazLPx1povM/640?from=appmsg "GraphQL 攻击面全景")  
  
day20 原文列了八类 GraphQL 攻击，可以归到三个维度：  
  
**认证授权类**  
- 未鉴权访问端点（直接打 /graphiql  
 看能不能进调试 UI）  
  
- IDOR（不安全直接对象引用），参数未过滤导致越权读取  
  
**协议设计类**  
- 内省查询开启（上面讲过）  
  
- Debug 模式开启（堆栈、源码、查询日志外泄）  
  
- 应用层 DoS（嵌套查询、字段别名、批处理把数据库打爆）  
  
**注入与泄漏类**  
- SQL 注入（参数拼到 ORM 里）  
  
- 错误信息数据泄漏（报错里带回 PII）  
  
### 3.2 探测四步法  
  
PortSwigger 推荐的测试流程：  
1. **找端点**  
：试五条常用路径 + GET/POST/x-www-form-urlencoded 三种方法  
  
1. **跑内省**  
：上面那段 __schema  
 查询跑一次  
  
1. **看 Web 流量**  
：如果目标网站本身用 GraphQL，用浏览器开发者工具或 Burp 看真实查询，照着改  
  
1. **捅参数**  
：用 IDOR 思路改 ID、改邮箱、改外键，看返回是否越权  
  
### 3.3 IDOR 实战例子  
  
PortSwigger 文档里有个标准教学场景：电商站用 GraphQL 列出已上架商品，返回 ID 1/2/4（3 缺）。攻击者直接查：  
```
query { product(id: 3) { id name listed } }
```  
  
返回里 product 3 是存在的，只是不上架而已——典型"列表过滤但单查不过滤"的漏洞。这种 BOLA（Broken Object Level Authorization，破坏对象级授权）漏洞因为有内省辅助，枚举所有带 id  
 参数的类型非常快。  
## 四、批处理与限速绕过  
  
GraphQL 支持用一个请求发多个查询（batch / alias），攻击者拿这个绕过限速。直接用别名让同一个 mutation 跑一百次：  
```
mutation { a: login(usr:"admin",pwd:"a") { token } b: login(usr:"admin",pwd:"b") { token } ...}
```  
  
WAF 只数 HTTP 请求数不看里面的别名，一百次登录尝试就是一个 POST。Directus 在 CVE-2024-39895 里栽过这个跟头，CVSS（通用漏洞评分系统）6.5，普通认证用户能把数据库负载线性放大。  
## 五、防御五要点  
1. **生产环境关掉内省**  
：Apollo、Hasura、graphql-java 都支持 introspection: false  
，线上必须关  
  
1. **强制 POST + application/json**  
：拒绝 GET 和 x-www-form-urlencoded，从源头掐掉 CSRF（跨站请求伪造）  
  
1. **字段级授权**  
：用 graphql-shield 等库做 resolver（解析函数）级鉴权，不要只看认证  
  
1. **深度/复杂度限制**  
：限制嵌套层数和字段数，挡住单查询 DoS；别名查询单独计数  
  
1. **错误信息脱敏**  
：生产环境不回 stack trace 和原始 SQL，统一改成"内部错误"  
  
## 六、思考题  
1. 你手上的目标如果是 GraphQL，先用哪一条 query 确认它在线？  
  
1. 内省关掉之后，还有什么办法可以推断 schema 结构？  
  
1. mutation 列表里如果有 deleteUser  
 但没有角色校验，这属于哪一类攻击面？  
  
## 素材出处  
1. learn365 day20 原文（GraphQL Series Part-1，About/Introspection/Attacks/Practice/References 五节框架）  
  
1. graphql.org Learn 教程（Schemas/Typed system/Introspection 章节）  
  
1. hasura.io GraphQL Introspection 教程（__schema  
 / __type  
 / __typename  
 三类查询示例）  
  
1. PortSwigger Web Security Academy GraphQL API Vulnerabilities（探测端点、IDOR、内省、批处理绕过）  
  
1. HackerOne 披露报告 #291531、#1132803  
  
1. HackerOne 博客 2024-07《How a GraphQL Bug Resulted in Authentication Bypass》  
  
1. Stingrai 2026 GraphQL API Vulnerabilities 综述  
  
1. dev.to 2026-09《GraphQL as Attack Surface: Introspection, Batching, and Schema Enumeration》  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6NAfcYz5VNc0DiapBZAKE2zqerGlZ9yvmAuRS5OAYnmicGxzGicWps7SuGTVIVv2D9hKDRMHqLkQWgEEnPeewxqQic4HpPwILibfyM4/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
