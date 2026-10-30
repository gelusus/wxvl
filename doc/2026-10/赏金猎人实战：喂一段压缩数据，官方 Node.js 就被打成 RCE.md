#  赏金猎人实战：喂一段压缩数据，官方 Node.js 就被打成 RCE  
原创 升斗安全XiuXiu
                    升斗安全XiuXiu  升斗安全   2026-10-30 10:38  
  
**【文章说明】**  
- **目的**  
：本文内容仅为网络安全**技术研究与教育**  
目的而创作。  
  
- **红线**  
：严禁将本文知识用于任何**未授权**  
的非法活动。使用者必须遵守《网络安全法》等相关法律。  
  
- **责任**  
：任何对本文技术的滥用所引发的**后果自负**  
，与本公众号及作者无关。  
  
- **免责**  
：内容仅供参考，作者不对其准确性、完整性作任何担保。  
  
**阅读即代表您同意以上条款。**  
> 【内容描述】  
CVE-2026-111497，藏在 Google Brotli 解码器里的一处"越界写"，被 Imperva 红队在官方原版 Node.js 上打成了实打实的远程代码执行（他们管这叫 Node2Shell）。这不是某一个软件的 Bug，而是"一个底层库出事，Node.js、Python、Bun、Envoy 全跟着躺枪"的供应链老剧本。本文用一线老兵的视角，把这漏洞的来龙去脉、谁中招、怎么排查升级、以及为什么现在就得动，一次讲透。面向赏金猎人和安全爱好者，重实战、轻恐慌。  
  
  
干这行久了，有个错觉特别害人：觉得"压缩库嘛，解压个文件而已，能有多大风浪"。  
  
直到 2026 年 10 月 21 号，  
Node.js 一把甩出十个安全修复，其中最扎眼的一个  
CVE-2026-111497，就藏在它内置的   
Google Brotli 解码器里。性质是"越界写"（  
Out-of-Bounds Write）。说出来你可能不信：攻击者只要往你的服务里塞一段精心构造的   
Brotli 压缩数据，解码器处理时就会越过缓冲区边界去写内存，堆坏了，进程权限就成了攻击者的。  
  
更骚的是，这玩意儿不是理论风险。安全公司   
Imperva 的红队，在官方原版的   
Node.js 24.22.0（Linux arm64）镜像上，真把代码跑起来了。他们给这利用起了个名：  
Node2Shell。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qg1MKHx3jGGkP34B4YBV1HQ9TsiahPvmibnRAyDcDCq5ibHtznKQheVfE9Hz4InnE6T51Jia2SsIkDRdVsnWcuxDeOGibQE9CaRszmqxZqnOiaTTc/640?wx_fmt=png&from=appmsg "")  
## 一、Brotli 是什么，凭什么一个库能掀翻半个互联网  
  
先给不熟的同学补个课。  
Brotli 是   
Google 搞的压缩格式，今天   
Web 上你看到的网页、图片、API 响应，很大一部分都是它压完再传的，省带宽、加载快。Nginx、Apache、Cloudflare、CloudFront，底层用的全是同一份 Google 代码。  
  
关键就在这儿：  
Node.js 自己内置了一份   
Brotli 解码器（在 zlib 模块里），Python 的 brotli / brotlicffi 包绑了一份，Bun 内置了一份，Envoy 也内置了一份。换句话说，  
这不是"一个软件有洞"，而是"一份代码被大家各自打包了一份，于是大家集体中招"。  
  
漏洞类型前面说了，越界写。攻击者构造畸形 Brotli 数据 → 解码器越界写 → 三种后果按严重度排：程序崩溃（DoS）、内存泄漏（敏感信息泄露）、执行攻击者代码（RCE）。最后一种，最要命。  
## 二、Node2Shell：在标准版 Node.js 上打出的 RCE  
  
讲真，漏洞披露当天就有   
PoC 了，但早期那个只能在"故意削弱的测试构建"上跑通——说服力有限。  
Imperva 这次不一样：他们用的是  
官方发布的正式   
Node.js 24.22.0，没改一行代码、没挂调试器，就是原汁原味的生产镜像，在全新容器里喂入构造好的   
Brotli 数据，目标进程里凭空多出个文件，说明携带的代码真跑了。  
  
而且他们在修复版   
24.22.1 上用同样的数据再打，直接被拒，文件没生成。这说明补丁是真有效，也反过来印证：这洞是可利用的，不是纸面风险。  
  
不过咱得把话说全，别吓过头：  
Imperva 自己承认，在他们的   
arm64 环境里  
也不是一次就中，得试几次；而且只测了这一个架构和这个应用配置，x64 和其他版本线（22.x、26.x）他们没逐一验证。换句话说，"能打"是真能打，但"一打一个准、全平台通杀"还到不了。  
  
根因我替你们扒了一下，挺典型的：Brotli 解压有个"块类型切换"环节。按格式，如果流只声明了 1 种块类型，那压根就不该出现"切换块类型"的指令。攻击者偏就构造一个"只声明 1 种、却发了切换指令"的畸形流。这本该立刻报格式错停下，但 Node.js 打包的这版解码器，上层几个返回类型是 void 的封装函数，把失败信号  
吞了——解码器带着一个没被正经赋值的块类型、和不一致的内部状态继续往下走，用越界的索引去查 Huffman 表和上下文映射，于是堆上就写了不该写的地方。一颗越界写的种子，就这么埋下。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qg1MKHx3jGFq8rduiaQYpIZ6loKFfnD1LpuJh2xPxSnyBiaibGgD4h9TrNPd6ficz8VTvt9lmSaaFyEVYGcbRv3xYhibm5OLDLZNQA4BNTNJQv5Q/640?wx_fmt=png&from=appmsg "")  
## 三、影响范围：Node.js 只是冰山一角  
  
Node.js 这边的修复版本记死：22.x 升到  
22.23.4、24.x 升到  
24.22.1、26.x 升到  
26.12.1。注意，18.x、20.x、25.x 这些已经 EOL 的线  
拿不到官方补丁，要么迁到受支持线，要么等发行版/LTS 服务商的单子。  
  
但 Node.js 真只是开胃菜。底层那份 Brotli C 库，Google 自己在 10 月 21 号也发了修复版：  
1.0.10 / 1.1.1 / 1.2.1。凡是静态打包了这份代码的，都得跟上，包括但不限于：  
- Python：brotli / brotlicffi 包，取决于它绑定的底层 Brotli 库版本，跟着升到上面那几个修复版；  
  
- Bun：内置 Brotli，修复版本   
1.4.4；  
  
- Envoy：配置了 Brotli 解压过滤器的版本；  
  
- 再往外，.NET、Istio、Java 的 brotli4j、libbrotli、甚至 curl/libcurl 的 Brotli 依赖，名单还在拉长。  
  
## 四、你中招了吗？怎么查、怎么修  
  
先泼盆冷水：裸 Node.js 的 HTTP 服务器，  
不会一收到请求就自动解压 Brotli 请求体。中招的前提是你的应用真的把"不可信的 Brotli 数据"喂给了解码器。Imperva 点出了两条真实路径：  
1. 出站侧：你用 Node.js 内置 fetch()  
 去拉一个外部响应，对方返回 Brotli 压缩体，Node 会自动解压——恶意响应就这么打进来；  
  
1. 入站侧：Express 5 / body-parser 2（2.0.0 起）在配置了 Brotli 解压时，会在  
鉴权之前就把 br  
 请求体解压——恶意请求体就这么打进来。  
  
所以自查清单很清晰：  
- Node.js：node -v  
 看版本，对照上面三条线；  
  
- Python：pip show brotli  
 和 pip show brotlicffi  
 看版本；  
  
- Bun：bun -v  
；  
  
- 代码里搜 brotliDecompress  
 / fetch  
 / body-parser 的 compress 配置。  
  
修法就一句：  
升运行时，别只升依赖。Node 的修复在运行时本身，光 npm update 不够；用容器的，得重新打镜像、重启进程，确认跑着的二进制是真新版。Python/Bun/Envoy 同理，等各自官方修复版，临时缓解就是先关掉不可信来源的 Brotli 解压、改用 gzip，或者像 Envoy 那样禁用解压过滤器。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qg1MKHx3jGHq2U45ykC3vOTEEu8eb4drGxCT5So15B7Gfj25C8k2ibcCB3lzicwibEjyYRuibCgm6QJ6ghCecoIkzI1RDhnSftbiaga3zq5DJb2Q/640?wx_fmt=png&from=appmsg "")  
## 五、供应链老剧本，和已经响起的警报  
  
看到这儿你大概品出来了：这又是 Log4Shell（2021）、xz-utils 后门（2024）同一个味儿——  
一个底层库被广泛依赖，它一咳嗽，半个生态跟着感冒。SBOM（软件物料清单）、依赖自动更新（Dependabot/Renovate）、最小化依赖、应用层输入校验，这些老生常谈的纵深防御，每次漏洞出来都被拎出来说一遍，但真落到自己项目上的人还是少。  
  
最后给你加个紧迫感，不是吓唬：Zscaler 的 ThreatLabz 已经通过诱捕网络，  
观测到针对这个漏洞的侦察和探测流量了。也就是说，扫描器已经在扫、攻击者在试了——虽然截至披露时还没见到完整武器化的在野利用，但"攻击者只是在等"这句话，现在得改成"攻击者已经开始敲门了"。  
  
评分上也别被单一口径带偏：编号机构 Google 给的是 CVSS 4.0  
9.2（Critical），而 Node.js 官方自己定的等级是  
High。两种口径并存，但结论不矛盾——这是个高价值、高利用潜力的洞，值得你优先处理。  
  
补丁是现成的，路径是清楚的。别等服务器真被打成肉鸡才动：现在就去查你的 Node / Python / Bun / Envoy 版本，该升的升，该打镜像的打镜像。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qg1MKHx3jGGoXNKvbMDrNE9mDHdaGfbWTypcyD9GMQwKZsYibZqVrSNOKF9t0NNrPSpMoCuL364sXgfveepOcGs0s2ZZgQoSLlicbVLSaNkUM/640?wx_fmt=png&from=appmsg "")  
  
最后啰嗦一句底线：  
只在你明确授权的范围内测试这个漏洞。本文讲的是防御视角，怎么查、怎么修、怎么不被打。拿去打别人没授权的系统，那是另一回事，咱不干那活。  
  
如果你觉得这篇把 CVE-2026-111497 从"是什么"到"怎么修"讲清楚了。  
  
👍  
点个赞，让更多跑 Node 的兄弟看见；  
  
💬  
关注我，后面接着扒那些"藏在运行时底下"的内存安全雷；  
  
🔁  
顺手转发给还在用老版本 Node / 没打镜像的运维和开发队友；  
  
📣  
推荐给做安全的朋友，供应链这堂课，早点补比事后救强。  
  
咱们下篇接着聊。守住边界，保持敏锐。  
  
