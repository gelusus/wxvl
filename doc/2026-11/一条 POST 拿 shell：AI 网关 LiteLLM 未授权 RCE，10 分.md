#  一条 POST 拿 shell：AI 网关 LiteLLM 未授权 RCE，10 分  
原创 红队安全圈
                    红队安全圈  红队安全圈   2026-11-17 12:00  
  
30 秒看懂  
  
影响范围 ·   
自建 LiteLLM 1.74.2–1.83.6，依赖树含 Starlette ≤ 1.0.0 的 ASGI 应用  
  
利用状态 ·   
已在野扫描  
，周末蜜罐持续收到请求，公开 PoC 在线  
  
补丁状态 ·   
已修  
：升 LiteLLM 1.83.7 + Starlette 1.0.1，并轮换 proxy 内的密钥  
  
AI 网关建在 Starlette 上，鉴权也写在 Starlette 的中间件里。这个组合 5 月就被证明能被一个 Host 头绕开，而 LiteLLM 的 MCP 测试端点正好把命令当参数收。  
  
## 01  事情是什么  
  
先看被打的那条请求。这个周末蜜罐持续收到同一个请求，请求体里的 command 写的是 id，Host 头变成了 localhost]、[localhost、127.0.0.1] 这类畸形值：  
  
POST /mcp-rest/test/tools/list  
  
Host: localhost]  
  
Content-Type: application/json  
  
Authorization: Bearer ***  
  
   
  
{"server": {"transport": "stdio",  
  
  "command": "id",  
  
  "args": [], "env": {}}  
  
攻击者要的只有一个结果：让代理把 id 当子进程跑起来。  
  
这是两个漏洞串成的链。CVE-2026-42271 是 LiteLLM MCP 测试端点的命令注入，4 月 20 日披露时定性为需要认证——接口接受完整的 MCP server 配置，包括 command、args、env，stdio 传输下代理照着配置 spawn 子进程，本来有 proxy API key 挡在前面。CVE-2026-48710 代号 BadHost，是 Starlette 1.0.1 之前不校验 Host 头，5 月 26 日公开，PoC 至今挂在网上。  
  
串起来之后，前置条件里那行需要一个有效密钥直接消失。链式评估给到 **CVSS 10.0**  
。  
  
## 02  为什么危险  
  
BadHost 的原理一句话说完：Starlette 把 Host 头拼进请求 URL，路由按真实路径走，中间件却看拼出来的那条 URL。  
  
拼法是   
http://{Host}{path}  
 再重新解析。Host 头里塞进斜杠、问号或井号，路径边界就被推到别的位置：  
  
curl -i -H 'Host: foo' \  
  
  http://target/admin  
  
# 403，被挡住  
  
   
  
curl -i -H 'Host: foo?' \  
  
  http://target/admin  
  
# 200，直接进  
  
第一行的 url.path 还是 /admin，鉴权中间件拦下。第二行重新解析后路径变成 /foo，不匹配 /admin 前缀，中间件放行，而路由照样把请求送到 /admin。  
  
LiteLLM 的 MCP 测试端点本来就靠路径前缀做鉴权，同样被绕。绕过之后，请求体里的 command 原样进 subprocess，执行身份就是 LiteLLM proxy 进程——实际部署里通常是容器内的服务账号。  
  
这个位置的价值不在 shell 本身。proxy 配置里躺着一堆 provider key，OpenAI、Anthropic、Azure 的账单全挂在上面；.env 里通常还有数据库和消息队列口令；网关的内网身份还能直接摸到同网段的 vLLM、向量库和下游 agent。  
  
蜜罐里那几种畸形 Host 还透露了一点：有人在改 PoC。公开利用用的是斜杠、问号、井号，这里出现的是方括号，专门绕照公开 PoC 写的 WAF 规则。只按已知 payload 加入侵检测，基本等于没加。  
  
从红队角度算这笔账也很直白：一个 curl 就够，不需要注册账号、不需要爆破密钥、不需要等管理员点链接。唯一的门槛是目标得能直连到 ASGI 进程。AI 网关恰恰大量部署在测试环境和 POC 环境里，那类环境通常没有反向代理，也没有人盯着。这解释了为什么补丁发了半年还有人在扫。  
  
## 03  谁受影响  
  
· 自建 LiteLLM，版本 1.74.2 到 1.83.6  
  
· 依赖树里含 Starlette 1.0.0 及以下的 ASGI 应用：FastAPI 系的 vLLM、MCP server、agent 框架、Ray Serve  
  
· 直接对公网开放、前面没有反向代理拦畸形 Host 的部署  
  
· 用中间件按 /admin、/metrics、/v1 前缀做鉴权的项目  
  
一条自查，两分钟：  
  
pip show starlette | grep Version  
  
docker exec 容器名 pip show starlette  
  
宿主机 pip 干净不代表镜像干净，LLM 工具链里 bundle 一份旧 Starlette 很常见。版本低于 1.0.1 的，再拿畸形 Host 打一下自己的受限路径，403 和 200 的差别就是答案。  
  
## 04  怎么修  
  
1. Starlette 升到 1.0.1 以上，重建镜像重新部署，容器和虚拟环境逐个查。  
  
2. LiteLLM 升到 1.83.7 以上，这个版本同时补了鉴权控制和依赖。  
  
3. 短期升不了，先把 /mcp-rest/test/connection 和 /mcp-rest/test/tools/list 挡在网关外，这两个端点原本就是调试用的。  
  
4. 轮换 proxy 里存的所有 provider key 和自定义 key，包括已经出现在日志里的那些。  
  
5. 翻日志：异常 Host 头（含方括号、斜杠、问号的值）、打到两个 test 端点的请求、proxy 用户的异常子进程。  
  
要在 WAF 侧拦截，规则别照 payload 写。只允许合法主机名通过：主机名里出现斜杠、问号、井号、方括号的一律拒。按已知利用编码加黑名单，改一个字符就失效。  
  
代码层面的长期改法只有一条：中间件里凡是拿 request.url 或 request.url.path 做安全判断的，全部换成   
request.scope["path"]  
。重新拼出来的 URL 不该当鉴权依据，这个坑后续还会以别的形式出现。  
  
## 05  一句判断  
  
补丁 5 月 21 日就发了，成规模的扫描到 11 月中旬才出现，中间这半年是攻击者在挑存量。CVSS 10 分、零凭据、后面挂着一串 provider key，符合连夜升的标准。  
  
但如果你的 LiteLLM 只在办公网内跑，前面还有一层会拒掉畸形 Host 的 nginx 或 Cloudflare，那今晚可以睡整觉，明早再升。判断依据是暴露面，不是分数。  
  
## 06  POC 链接  
  
https://horizon3.ai/attack-research/vulnerabilities/cve-2026-42271-chained-with-cve-2026-48710/  
  
https://badhost.org  
  
https://www.x41-dsec.de/lab/advisories/x41-2026-002-starlette/  
  
https://github.com/advisories/GHSA-86qp-5c8j-p5mr  
  
链式利用分析、原始 PoC 站点、X41 披露原文与 Starlette 官方通告  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
