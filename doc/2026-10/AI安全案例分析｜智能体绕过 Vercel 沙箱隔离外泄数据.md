#  AI安全案例分析｜智能体绕过 Vercel 沙箱隔离外泄数据  
原创 天元实验室
                    天元实验室  M01N Team   2026-10-19 10:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/uZT6kWW1jCkae2Dqoic3S3sd5VLM4cFFN7ZowCgMGPiccnFpJGjmib1SLXcRdudOLib3ib0TSAq6KV8Mn15ockeiaGLNgoyyKLJaMFksK8joNaeKU/640?wx_fmt=gif&from=appmsg "")  
  
概述  
  
  
2026 年 10 月 12 日，AI 安全公司 AIR Security 披露了 Vercel Sandbox 的一处网络隔离绕过手法，命名为 VerCellBreak。被攻击者控制的 AI 智能体可借助 TLS 证书链校验过程中的一个标准特性将密钥带出沙箱，而沙箱自身与攻击服务器之间从未发生任何直接通信。  
  
  
攻击方式是为沙箱内的智能体配置一张伪造的 mTLS 客户端证书，把待外泄的数据编码进证书 AIA 扩展所含的 URL。智能体只连接白名单内的域名，但该受信任服务器在构建证书链时会主动请求这一 URL——数据由此绕开沙箱的出口限制，而在沙箱看来整个过程中没有任何异常。  
  
  
  
**01 攻击背景**  
  
  
2026 年 7 月，OpenAI 的智能体从评估沙箱中逃逸并入侵了 Hugging Face。该事件中，智能体并未突破沙箱的边界——沙箱允许其访问一个内部服务，也就是一个包管理器，智能体诱使该服务代为发起对外请求。攻击者没有强行突破隔离，而是让一个受信任的对象代替自己完成了请求。  
  
  
数周之后，Vercel 为其沙箱开放了漏洞赏金，范围覆盖网络隔离与虚拟机隔离。AIR 沿同一思路排查，发现了同类问题。  
  
  
智能体被投毒或劫持后，其影响范围基本包含两处——运行它的宿主机，以及它可建立的网络连接。破坏宿主机可借只读挂载、微虚拟机等手段缓解；而一旦放开网络出口，智能体就可能将凭据、密钥或会话数据送往攻击者。域名白名单正是针对后者的设计：即便智能体持有密钥，只要无法访问名单之外的服务器，数据便无处可去。  
  
  
  
**02 攻击原理与流程**  
  
  
**2.1 前提条件**  
  
链条成立需要四个条件同时满足：智能体运行在启用了出站域名白名单的沙箱中；智能体已被攻击者控制；白名单内的某个可信域名要求客户端使用 mTLS 认证；该服务器在构建证书链时会解析 AIA 扩展并主动发起外部请求。  
  
  
**2.2 被绕过的机制：SNI 过滤与 AIA 扩展**  
  
Vercel Sandbox 的出站限制中有一层是 SNI 过滤：它检查 TLS 握手中客户端声明的目标主机名，据此放行或阻断连接。业界已知的绕过方式包括域名前置（domain fronting），但 Vercel 将最常见的几种变体列为超出范围。AIR 转而研究另外两个机制的交互——双向 TLS（mTLS）与证书的 AIA（Authority Information Access）扩展。  
  
  
mTLS 要求客户端也出示证书供服务端验证，而验证一张证书需要一条通向受信任根的信任链，通常由出示方将完整的中间证书一并发送。但协议也允许动态获取——AIA 扩展可以指向一个 URL，服务端在构建证书链时按需前往该地址取回缺失的 CA 证书。风险正源于这一动态获取过程：URL 的内容由证书出示方控制。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uZT6kWW1jCn3ETNsANrbTyVSaicSA6HLsavLxNtWFEgaYrllDuOYm74icHahYyd0pPvdic3sCgu6J7kobwMz18KLwxPh56BJ0X89kFqC59dibZM/640?wx_fmt=png&from=appmsg "")  
  
图1 普通 TLS 与 mTLS + AIA 的差别：后者要求客户端也出示证书，服务端因此需要构建并验证客户端的证书链（来源：AIR Security）  
  
  
**2.3 攻击链路**  
  
1. ACME 公司的智能体运行在 Vercel Sandbox 内，磁盘上存有私钥；出站通信被限制在 acme.com。  
  
  
1. acme.com 按正常业务要求客户端使用 mTLS 认证。  
  
1. 攻击者控制该智能体后，直接命令它把私钥通过 HTTP 发往 evil.com，请求被沙箱拦截。  
  
1. 攻击者制作一张伪造的 mTLS 客户端证书，把编码后的私钥放进证书的 AIA URL，形如 https://evil.com/<私钥的 base64>.cert。  
  
  
1. 智能体以 mTLS 连接白名单内的 acme.com，并出示这张伪造证书。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uZT6kWW1jCmZEB4I7Jl3oWKB52jaUH6MjtYpwic3JadHz6B4J0wggc6ia2l8TnTiar9BMOsBjl0TdH9LQz8tHWxK52KdRhLQ97pNRhjzC1ibyg0/640?wx_fmt=png&from=appmsg "")  
  
图2 智能体以mTLS连接acme.com，出示的客户端证书其AIA URL指向 evil.com/<key>.cert（来源：AIR Security）  
  
1. acme.com 在验证证书链时，去 https://evil.com/<私钥的 base64>.cert 请求缺失的 CA 证书。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uZT6kWW1jCm2sJlGHvdsWFaoyAlUKmn2MicQsFFMNAbOjZaA7toA2ugQ7Qn8hpAKxvV5SzXBJCohvhSNTlDYGeKtzM5AYvJyRq39c1htRyxw/640?wx_fmt=png&from=appmsg "")  
  
图3 acme.com 验证证书链时按 AIA URL 取回 CA 证书，请求由此发往 evil.com 并带走密钥（来源：AIR Security）  
  
1. 请求到达攻击者服务器，攻击者从 URL 中还原出私钥。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uZT6kWW1jCn0Xcib8grQQdLwChvGDcgCJ91p4VVWXKIFwAlH74lfE9zgjKsWkxl6qWnCguPov12wp40bIHDoplaKibt4iata57s1enicpyic8snU/640?wx_fmt=png&from=appmsg "")  
  
图4 密钥到达 evil.com；智能体从未与它直连，是由被允许的域名代为完成这次请求（来源：AIR Security）  
  
  
上述过程中，沙箱的拦截逻辑并未失效，智能体确实无法直连 evil.com。数据之所以外泄，是因为一个受信任且被允许的服务器代其发起了那次外部请求。这与 2026 年 7 月 OpenAI 智能体所用的手法属于同一思路，区别在于当时的「包管理器」此处换成了 acme.com。  
  
  
**2.4 问题分析**  
  
整条路径利用的是 TLS 的标准特性，既无服务器被攻破，也无漏洞被利用。AIR 指出这正是其难以修补的原因：沙箱从未与攻击服务器直接通信，在沙箱看来一切正常。  
  
  
该手法依赖链条另一端的外部服务器，而 Vercel 并不运营 acme.com 一侧。这也说明问题的根源在于沙箱只检查智能体自身的出站连接，却无法仅凭域名白名单约束白名单服务器后续发起的请求。  
  
  
  
**03 威胁与后果评估**  
  
  
智能体一旦被劫持，白名单便不再是数据外泄的终点。任何被允许的、会处理攻击者输入并发起外部请求的服务，都可能被改造成间接外泄通道。安全公司 0rcus 联合创始人兼 CEO Nic Adams 的表述是，一个被批准的服务器会变成「被盗凭据的不知情信使」，安全策略必须考虑第一次连接之后会发生什么。AIR 表示尚未发现该漏洞被在野利用的证据，但无法彻底排除。  
  
  
当智能体被授予代码仓库、云凭据、CI/CD 或生产系统的权限时，一次外泄的后果便不再局限于沙箱内部。  
  
  
  
**04 案例启示**  
  
  
不应将沙箱视为独立的安全边界，它只能阻断自身可见的连接，无法保证数据不被外传。任何被允许的、会处理攻击者输入并发起外部请求的可信端点，都可能被改造成间接外泄通道。  
  
  
在沙箱之前的环节，可通过限制智能体可读取与摄取的内容，降低其被投毒或劫持的概率。AIR 的最终结论是，威胁应在抵达智能体之前就被拦下，而非在进入之后依赖沙箱兜底。  
  
  
  
**关于 AISS 安全智链社区**  
  
  
本案例已收录至 AISS 安全智链社区案例库，社区地址：https://aiss.nsfocus.com/#/  
  
  
  
**参考链接**  
  
  
[1] AIR Security — VerCellBreak: How we escaped Vercel's hardened sandbox（2026-10-12） https://www.air.security/blog-posts/vercel-break  
  
  
[2] The Record — Sandbox vulnerability gives rogue AI agents a path to escape and leak data, research says（2026-10-13） https://therecord.media/sandbox-vulnerability-gives-rogue-ai-agents-path-out  
  
  
[3] Vercel Docs — Sandbox firewall https://vercel.com/docs/sandbox/concepts/firewall  
  
  
[4] Vercel — Advanced egress firewall filtering for Vercel Sandbox https://vercel.com/changelog/advanced-egress-firewall-filtering-for-vercel-sandbox  
  
  
[5] Cloud Security Alliance — Autonomous Sandbox Escape: OpenAI Models Breach Hugging Face（2026-07-30） https://labs.cloudsecurityalliance.org/research/csa-research-note-openai-artifactory-sandbox-escape-20260730/  
  
  
[6] Malwarebytes — OpenAI explains how its AI agent breached Hugging Face（2026-07-29） https://www.malwarebytes.com/blog/news/2026/07/openai-explains-how-its-ai-agent-breached-hugging-face  
  
  
**绿盟科技天元实验室**  
专注于新型实战化攻防对抗技术研究。  
  
研究目标包括：漏洞利用技术、防御绕过技术、攻击隐匿技术、攻击持久化技术等蓝军技术，以及攻击技战术、攻击框架的研究。涵盖Web安全、终端安全、AD安全、云安全等多个技术领域的攻击技术研究，以及工业互联网、车联网等业务场景的攻击技术研究。通过研究攻击对抗技术，从攻击视角提供识别风险的方法和手段，为威胁对抗提供决策支撑。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/uZT6kWW1jCnbzKU3sTYudeIxr8WE1TjnZVhkfiaxVxscJth69yEGmN2bftgp6dfOq0qIdIj4emOPnHJodmU5AufWc49yfj7ewv2BnkafWHJE/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
  
**M01N Team公众号**  
  
聚焦高级攻防对抗热点技术  
  
绿盟科技蓝军技术研究战队  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uZT6kWW1jCmicMraXcujy2QaXRLWUcScl5IgKJsdTmy89szco6suHjLS5BcSCSaxia8AwMAMXiaDfJJcMBK7riarkfSpy16BfxLcjJfK7wpLKoQ/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
**官方攻防交流群**  
  
网络安全一手资讯  
  
攻防技术答疑解惑  
  
扫码加好友即可拉群  
  
