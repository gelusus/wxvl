#  一个请求头接管GPU集群：GPUStack在野漏洞拆解（附PoC）  
原创 赤霄攻防实验室
                    赤霄攻防实验室  赤霄攻防实验室   2026-11-06 01:20  
  
先说结论：**GPUStack 2.2.0 之前的所有版本，只要管理端口网络可达，任何人发一个请求就能成为内置管理员**  
——不需要密码，不需要漏洞利用链，一个伪造的 HTTP 头就够。这不是理论推演：官方安全通告白纸黑字确认，该漏洞已在真实环境中被利用，一台 v2.1.1 部署的 GPUStack 被攻击者植入自定义后端、跑起了反弹 shell。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8acbAuaTTanbNtDzN4SBGslJgHQRCQwEFTa1X6luLE4qU2a4IcO2Zibq8AdZqDUVLj5FXiauQMndmhLF6CdbdwibOtlM3RR5fNsRnA/640?wx_fmt=png "")  
  
CVE-2026-102895 · CVSS 9.8 · 官方确认在野利用  
## TL;DR 四行速览  
  
**是什么：**  
GPUStack 认证绕过（CVE-2026-102895，CVSS 9.8）。服务端把「请求来源是不是 127.0.0.1」当成「是不是本机管理员」的判断依据，而来源可以被一个 HTTP 头伪造。  
  
**影响谁：**  
所有 gpustack < 2.2.0 的部署，pip 单机装、Docker、Kubernetes 形态全部在列。GPUStack 是开源 GPU 集群管理平台，管的是最值钱的资产——GPU worker 主机与云厂商凭据。  
  
**怎么修：**  
升级到 v2.2.2 及以上（v2.2.0 修本洞，v2.2.2 补上同窗披露的 SAML 绕过 CVE-2026-102894）。临时缓解：设 force_auth_localhost 为 true，并在反向代理层剥掉客户端伪造的 X-GPUStack-Real-IP 头。  
  
**看点：**  
「localhost 即可信」这个设计假设是怎么被一个头打穿、又被同机反代部署全量放大的；以及我们在源码级验证后写出的三态判定 PoC（文末已开源）。  
## 一、漏洞档案  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8afsucqjVSuZ2zwCPb8ophQePRoOib8gPVUaJhbCt6DOdVOKBeJqNFB6Cia4aEY5ZW3TmCb8UExFva3v8YSSP3lJniaM2oVB3POVOg/640?wx_fmt=png "")  
  
CVE-2026-102895 漏洞概要信息卡  
  
GPUStack 这两年随着自建 AI 推理集群的需求火了起来：一条 pip install 或一行 docker run 就能把几台带卡的服务器变成模型服务平台，提供模型部署、GPU 分配调度、用户与 API 密钥管理。越是大而全的管理面，一旦认证失守，塌方就越彻底——这个漏洞正是最典型的例子。  
  
漏洞的 CWE 归类很有意思，NVD 同时挂了三个：CWE-287（不当认证）、CWE-290（伪造绕过）、CWE-346（来源校验错误）。三个标签拼出同一个故事：**代码把网络来源当成了认证凭据**  
。这不是内存破坏，不需要精心构造的字节流，攻击复杂度为低——这正是它 9.8 分的原因：网络可达、无需权限、无需交互、全机失守。  
## 二、根因：把「来源」当「凭据」的两行代码  
  
我们拉取了 v2.1.1 与 v2.2.0 的源码做对比。漏洞出在 gpustack/api/auth.py 的 get_current_user 函数，逻辑直白得近乎朴素：先走正常凭据校验（Basic / Bearer / API Key / Cookie），如果全都没提供、用户为空，就问一句「请求是不是从 127.0.0.1 来的」——如果是，并且配置项 force_auth_localhost 为假（默认就是假），直接把数据库里第一个管理员账号赋给这个请求。本机命令行操作免登录，这个初衷是方便，但方便的前提是「来源地址可信」。  
  
问题在「来源」怎么算。同文件里的 client_ip_getter 给出了答案：当网关为内嵌模式时，返回请求头 X-GPUStack-Real-IP 的值，拿不到才回退到 TCP 对端地址。而 HTTP 请求头是**客户端想写什么就写什么**  
的。两条伪造路径由此而来：  
  
**路径一，直接伪造头。**  
攻击者从任意位置发请求，带上一行 X-GPUStack-Real-IP: 127.0.0.1，client_ip_getter 如实上报回环地址，自动管理员分支应声触发。一个头的成本，换整个集群的管理员。  
  
**路径二，同机反向代理放大。**  
官方通告特别点名了这种「教科书式自托管形态」：GPUStack 躲在本机 nginx 后面，反代配置 proxy_pass 指向 127.0.0.1 加端口。此时**每一个外部请求**  
到达 GPUStack 时 TCP 对端都是 127.0.0.1——不需要任何头，全体远程用户自动变管理员。单个漏洞成为整类部署的默认失陷态，这是本案例最值得记住的工程教训。  
  
安全设计原则一句话：**任何「来源即凭据」的信任决策，都必须先证明来源不可伪造。**  
TCP 对端地址在反代场景必然失真，自定义头则从未可信过。v2.2.0 的修复正是围绕这一点：Real-IP 头只有在请求携带正确的网关令牌（X-GPUStack-Auth-Token）校验通过后才被采信，否则一律回退 TCP 对端；回环自动管理员分支则被整体移除。  
## 三、攻击链：从请求头到 worker 主机失陷  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8afia59JTzlM20KXViceAL2vMdMjicdcQb05XtTg7F5Lk9o1RqmRv1bspPJhoIgsKTGmRWqFKiav8OxBs8RbWSxONdL04DBtNrkHSP8/640?wx_fmt=png "")  
  
五步攻击链 · 与官方在野攻击路径一致  
  
拿到免认证管理员只是开头。管理员权限在这个平台上的变现路径非常顺：第一步是**持久化**  
——调用 API Key 接口给自己签发一个全权限的管理员密钥，此后从任意 IP 使用，不再依赖回环伪造，即使目标日后升级，这把钥匙依然有效（所以升级后务必审计密钥列表）。  
  
第二步是**横向收割**  
。GPUStack 管理面里存着 worker 节点的通信凭据和云厂商密钥（cloud_credentials 接口），拿到它们等于拿到扩权到云资源的跳板。第三步是**落地执行**  
：官方通告明确提到，通过 POST /v2/inference-backends/from-yaml 接口提交一份构造的 YAML 就能在 worker 主机上部署「自定义推理后端」——推理后端本质是一个可执行环境，攻击者用它运行反弹 shell。这正是 v2.1.1 在野攻击现场的手法：攻击者创建的自定义后端跑着反向连接，GPU 算力平台直接变成肉鸡管理面板。  
## 四、同一修复窗的另一半：SAML 回调不验签  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8adicGvYN4M7Libnq2MtmqtnDtJ2biboI4uox3KUzV8icSCIcThCyTXuqD1j23YsE1HPpUELOibsNAgLiccWwrlDe49ZE12PMaE1bdbibo/640?wx_fmt=png "")  
  
CVE-2026-102895 与 CVE-2026-102894 对照  
  
深挖通告时发现这不是孤立事件。同一时间窗，GPUStack 还披露了 CVE-2026-102894（CVSS 9.1）：/auth/saml/callback 这个 SAML 登录回调路由，收到 Base64 编码的 SAMLResponse 后，直接解码 XML、读出用户名、签发会话令牌——**唯独不调用验签函数**  
。签名、签发方、时效，SAML 安全模型的三根支柱全数跳过。  
  
更糟的是触发条件：这个回调在所有部署里默认启用，**哪怕你从未配置过 SAML 单点登录**  
。攻击者只要能访问到这个端点，构造一段指名道姓的 XML（比如 NameID 写成管理员），就能直接拿到那位管理员的会话——完整的账号接管。两个漏洞一个是「来源即凭据」，一个是「收到即信任」，病根相同：该验证的一步被省掉了。修复版 v2.2.2 补上了完整验签与失效时间校验。这也是为什么我们建议直接升到 v2.2.2 以上，而不是只升到修了本洞的 v2.2.0。  
## 五、PoC：三态判定的完整验证  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8aeA8iaYIOw8RE2koCXpEZBKAgiaeKyFqCPdrqtT9tOawxQicoIe0bbZLibNHUkkTzkJOSGoOIkhL4MLLTia0Mn7M0XFf5gSzEnI0a7w/640?wx_fmt=png "")  
  
PoC 三态判定逻辑与实测结果  
  
针对这类认证绕过，很多网上的「PoC」其实只做了指纹识别——看服务是不是 GPUStack、版本号在不在影响区间，就宣布漏洞存在。这不够：指纹只能说明「可能是」，不能说明「真的能绕过」。我们按完整验证版标准写了 PoC，核心是找一个**可判定的响应分叉**  
：选定 /v2/users 这个管理员端点作为探针，不带任何凭据访问它。  
  
判定逻辑三态：伪造回环头后返回 200 且响应体是用户列表 JSON——漏洞通道真实触达、管理员权限落地，结论 VULNERABLE；返回 401——伪造头被拒，说明已修复或已加固，结论 PATCHED；目标不可达或响应不符合预期——结论 UNKNOWN，退出码区分，绝不武断。整个探测只读不写，不创建任何资源，对目标零副作用。  
  
为验证判定逻辑的正确性，我们按 v2.1.1 源码语义搭建了模拟靶机（复刻 client_ip_getter 信任 Real-IP 头与回环自动管理员的完整分支），同时按 v2.2.0 语义搭了修复版靶机。实测结果：漏洞版靶机上，未认证请求加伪造头拿到 200 与含 admin 的用户列表，PoC 判定 VULNERABLE；修复版靶机上同样请求拿到 401，判定 PATCHED；指向不存在端口时判定 UNKNOWN。三态全部按预期分叉，判定依据是真实 HTTP 响应而非版本猜测。PoC 已开源（见文末参考），仅限授权测试使用。  
## 六、处置清单  
  
**如果你在跑 GPUStack：**  
第一步，升级到 v2.2.2 及以上，一次补齐两个认证绕过。第二步，不能立刻升级就上临时缓解——配置 force_auth_localhost 为 true，反向代理层剥掉客户端传入的 X-GPUStack-Real-IP 头，管理面收进内网或 VPN 之后。第三步，**已经升级的也别掉以轻心**  
：回查 API Key 列表里有没有陌生密钥（在野攻击会留持久化钥匙），回查 inference-backends 里有没有不是自己部署的「自定义后端」，回查云凭据是否泄露轮换。管理端口暴露在公网的，按已失陷处置。  
  
**如果你是安全工程师：**  
这个案例值得收进检测规则库。审计自家代码里所有「来源即凭据」的判断：IP 白名单、回环信任、内网直通，逐一问一句「这个来源字段攻击者能不能伪造」。反代部署形态尤其要重点审——那是一个把「外部请求」系统性变成「本机请求」的放大器，任何基于对端地址的信任决策在那儿都是裸奔。  
  
**参考链接：**  
  
 GitHub 安全通告（回环绕过）：github.com/gpustack/gpustack/security/advisories/GHSA-gj9m-f8ph-pmhr  
  
 GitHub 安全通告（SAML 不验签）：github.com/gpustack/gpustack/security/advisories/GHSA-62m9-f875-rqw6  
  
 NVD 条目：nvd.nist.gov/vuln/detail/CVE-2026-102895  
  
 修复提交：github.com/gpustack/gpustack/commit/53f42a6a  
  
 本篇 PoC（完整验证版）：github.com/Rubby2001/CVE-POCS/tree/main/CVE-2026-102895  
  
免责声明：本文仅用于防御性安全研究与授权测试参考。未经书面授权对他人系统进行测试属违法行为。PoC 判定输出基于目标真实响应，使用前请确认你拥有目标系统的测试授权。  
  
—— 赤霄攻防实验室，每晚精选一个高价值漏洞做深度拆解。觉得有用请点个在看，转发给你管 GPU 集群的同事。  
  
