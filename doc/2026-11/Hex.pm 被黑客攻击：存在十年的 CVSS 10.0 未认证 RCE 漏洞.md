#  Hex.pm 被黑客攻击：存在十年的 CVSS 10.0 未认证 RCE 漏洞  
原创 Gale Swift
                    Gale Swift  黑白之道   2026-11-26 00:58  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6Ot1Dwaqtqs8vYhRZPujRUlpiasgx9h8qHTgLVPGyILBgicWABxW30GiaajUkGbJOBBVqhLsG9j5VkX73bM7PvNicpbL0IT83gNqLQ/640?from=appmsg "")  
> **导语**  
：全球最大的 Elixir/Erlang 包注册中心 Hex.pm 确认已被黑客攻击利用。攻击者利用 CVE-2026-129514——一个 CVSS 满分 10.0 的未认证远程代码执行漏洞，在 6 台生产服务器上以服务账户身份执行了系统命令。该缺陷自 2016 年 2 月起就藏在请求体解析器里，官方在首个恶意请求后约 15 分钟完成全网修复。  
  
## 漏洞速览  
  
**风险等级：红（严重，必须立即处置）**  
  
CVE-2026-129514（GHSA-9pmm-hf4r-27pg）是 Elixir/Erlang 生态包注册中心 hex.pm 的未认证远程代码执行漏洞，CVSS v4 评分为 **10.0**  
，向量字符串为 CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:A  
。厂商 ERLEF CNA 已登记该编号，OSV 侧同步为 EEF-CVE-2026-129514。  
  
**影响实体**  
- hex.pm 包：受影响区间 >= 2016-02-23, < 2026-11-23  
，修复版本 **2026-11-23**  
  
- hexpm/hexpm 源码：引入缺陷的提交 ab1bbce  
，修复提交 379b0a2  
  
- 攻击面在服务端注册表代码，不在 Hex 客户端。所有自建 hex.pm 私有注册表（企业内部制品库）的组织同样受影响，直到部署补丁为止  
  
漏洞类型为反序列化不可信数据（Deserialization of Untrusted Data），根因是 Erlang 外部项格式解析器对带有 __struct__  
 键的映射缺少校验。  
## 利用链拆解  
### 解析器跑在鉴权之前  
  
HexpmWeb.Endpoint  
 使用 Plug.Parsers  
 且 HexpmWeb.PlugParser  
 排在路由器之前，因此任何路由的请求体都会先经过它解析——**路由和认证插件此刻都还没执行**  
。当 Content-Type 为 application/vnd.hex+erlang  
 时，请求体被交给 HexpmWeb.ErlangFormat.decode/1  
，内部调用 Plug.Crypto.non_executable_binary_to_term/2  
 并传入 [:safe]  
。  
  
这个 "safe" 模式确实拦住了函数和新原子，但**对映射没有任何限制**  
，一个带 __struct__  
 键的 map 可以直接通过。安全校验恰恰是在 2016 年 2 月 23 日被移除的，这就是漏洞"十年高龄"的来源。  
### 从 struct 到 apply/3  
  
请求体本身不会自动执行命令，它只是引发了一连串连锁反应：  
1. Plug.MethodOverride  
 在每个 POST 上求值 body_params["_method"]  
，对 struct 而言 Access.get/3  
 会调用目标模块的 fetch/2  
，而 Ecto.Query.DynamicExpr  
 未定义该函数，抛出 UndefinedFunctionError  
，栈帧里就带着攻击者控制的 struct。  
  
1. 错误上报环节（Sentry、Bandit 错误日志、JSON 日志格式化）对请求数据与栈帧参数调用 inspect/1  
，选中 Inspect.Ecto.Query.DynamicExpr  
，它对 binding 字段执行 Enum.map/2  
，却不检查该字段是否为列表。  
  
1. binding 字段里塞着 DBConnection.Stream  
 struct，其 Enumerable  
 实现调用 DBConnection.reduce/3  
，进而走到 declare/4  
。  
  
1. declare/4  
 从 opts 中读取 :log  
 选项，查询编码在真正取出数据库连接之前就失败了，但错误路径依然会调用这个日志钩子。钩子对 {module, function, args}  
 元组执行 apply(mod, fun, [entry | args])  
——**三个值全部来自请求体**  
，最终以 BEAM 进程所属用户身份运行操作系统命令。  
  
整个过程最终返回一个普通的 500 响应，不会有任何异常迹象。  
  
![CVE-2026-129514 利用链示意](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MNl5uFgENSRM6ia8XtKCn2TL6rMV7WicF7VZ9eW3NedBBJufP0YR0ZgBTDQMOkjJ5O6egibpAblvvevnJNOQYPb8JmoELZhjHuo8/640?from=appmsg "CVE-2026-129514 利用链示意")  
## 真实攻击复盘  
  
2026 年 11 月 23 日（周一），攻击开始。负载均衡器日志记录到 **69 次利用请求，集中在 18:16 至 18:29 UTC 之间，全部来自两个 IP 地址**  
。利用生成的文件出现在 **6 台 web 服务器**  
上，其中一台在请求后仅 3 毫秒就出现文件；而这 6 台中有 3 台是在攻击进行到一半时因机器重启拉起、仍运行旧构建而"主动"加入战场的。到 **18:31 UTC，全部服务器已运行修复版本**  
，从首个恶意请求算起约 15 分钟。  
  
攻击者的两条来源 IP 在此前一周末就有活动，官方判断对方搭建了 hex.pm 的本地副本进行演练。一份描述同一缺陷的独立安全报告在 **19:07 UTC**  
 通过邮件送达——此时攻击已经结束，说明对方并不急于"抢首报"。  
  
审计方面，团队将每张数据库表与首个利用请求前两分钟生成的快照做了逐行比对：攻击期间发布的 23 个版本全部与审计日志吻合，没有旧版本被修改，当晚写入的包文件与注册表文件也与数据库记录一致。**未发现被篡改的包**  
。但数据库读取、shell 命令、环境变量与文件存储直读均未落日志，因此官方明确表示"无法排除攻击者访问过数据库数据"，其中包含邮箱地址、密码哈希、API key 哈希、两步验证密钥与会话数据，私有包也可能被读取而无痕迹。审计日志中未发现残留 shell、新增账号或新增云密钥。  
  
事件当晚，hex.pm 向瑞典数据保护机构、美国 CISA 与欧盟 ENISA 进行了通报。  
## 处置方案  
  
![CVE-2026-129514 事件响应处置清单](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NQSkqlwf67YygdtnEk44dsQKBObZficW4HCpm8ia5BKgKeRakb3SEzYIv1JXIKcaDK9ab3riap3qiblRP69dfhEKXIXJsRV1GbDyU/640?from=appmsg "CVE-2026-129514 事件响应处置清单")  
### 私有注册表运维方  
  
修复方式很直接：解码器在发现解码结果中任何 map 含 __struct__  
 键时直接返回错误，解析器随之返回 400。私有部署方需升级到 2026-11-23 版本，或移植修复提交 379b0a2  
。  
  
⚠️ **补丁的业务中断风险**  
：hexpm 是自建 Elixir 应用，升级需要重新构建并发布 BEAM release，**必须重启服务**  
，期间私有制品库不可用；若同时升级依赖或执行数据库迁移，需要预留回归测试窗口。建议做法是先把补丁提交 cherry-pick 到当前发布分支做最小改动验证，再安排发布窗口。  
  
在官方新客户端发布前，**只从 repo.hex.pm 拉取包，避免使用任何第三方镜像**  
。注册表签名密钥尚未轮换，而它正是攻击者可能读到的凭据之一。  
### 依赖方账号处置  
  
官方已更换会话与访问令牌的签名密钥，所有 hex.pm 账号被强制登出。账号持有者需要完成：  
1. 修改 hex.pm 密码（密码使用 bcrypt 哈希，弱密码仍需修改）  
  
1. 重置两步验证，重新绑定验证器并妥善保存新的恢复码  
  
1. 用 mix hex.user auth  
（Erlang 项目用 rebar3 hex user auth  
）重新登录客户端  
  
1. 升级 Hex 客户端到数日内发布的新版本，该版本引入了新的注册表签名机制  
  
1. 警惕钓鱼：已曝光的邮箱可能被用于伪造邮件，任何涉及 Hex 的链接都必须指向 hex.pm 域名  
  
**API key 无需更换**  
：它们是 128 位随机值，服务器只保存 HMAC-SHA256 哈希，暴力破解不可行。只拉取公共包、从未注册账号的用户无需做任何处置。  
### 同批披露的三个库  
  
事件期间，三个常用 Elixir 库也各自披露了"由不可信数据构造 struct"相关缺陷，均已发布修复版本，属于独立披露而非针对 hex.pm 的二次攻击：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">库</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">受影响版本</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">ash</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-124283</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">3.31.3 至 &lt; 3.38.0</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">3.38.0</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">db_connection</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-133555</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">1.1.0 至 &lt; 2.10.3</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2.10.3</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">flop</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-130411</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">0.8.0 起各分支修复前</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">0.26.7 / 0.27.3 / 0.28.1 / 0.29.2</span></section></td></tr></tbody></table>  
ash 的触发条件较苛刻：需要 keyset 分页配合客户端提供的游标、特定排序字段类型，且签名密钥仍保持未设置的默认值。db_connection 官方明确说明"单独不可利用"，必须先有其他组件把不可信数据反序列化成 struct。flop 的触发条件同样集中在分页游标上。  
## 总结  
  
CVE-2026-129514 的教训并不复杂：一个"安全模式"的反序列化解析器，只要少校验一种结构体标记，就能在十年后变成满分 RCE。修复虽然只有一行判定，但它守住了整个 Elixir 供应链的信任根。  
  
对运维侧的即时提醒只有三条：**自建 hex.pm 的立即打补丁；注册过账号的立刻改密码并重置 2FA；在此之前只从 repo.hex.pm 拉包。**  
  
**来源**  
：本文编译自 The Hacker News《Hackers Exploited CVSS 10.0 Hex.pm RCE Flaw to Compromise 6 Package Registry Servers》、hex.pm 官方安全公告 GHSA-9pmm-hf4r-27pg 及 ERLEF CNA 漏洞记录 CVE-2026-129514。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OMXOxxWrFBvtO4LnNSokJdr4Msp3sDh29v4HzC4KaemcElVvZ2VURm8U4wT9MA9AEnXxuuicvZPY6ziaJAJEw5g96THBERLNyiaI/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PvjRjk24cvlwz535BNB7mdLIeKLYduoSEgvZmAibPMybfb3lVuNHoo43NO027JGqvW7RWruBiav7F3HicLsp2Pmf98BTGW1NK2Ok/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PiaQqoAgISdoVOAF8IsSsCPstibG9LnibInU4EicaMtE9If1xlN813H8gFjtPTh9jmG5KF4fad8K47QFjNicuPKRWqaoLt7WBbzGib8/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NKh7MQGxMBKmib1TZDv4vMA2J53r9yKkrFIYys8ibh5WXefMspTzMv6rgP4TGgneR0toM1W1mUvgGeBjONUics4U0AlTAiaoeAicA0/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
