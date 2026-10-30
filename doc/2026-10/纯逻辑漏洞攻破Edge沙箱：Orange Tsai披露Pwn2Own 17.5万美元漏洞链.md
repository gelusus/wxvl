#  纯逻辑漏洞攻破Edge沙箱：Orange Tsai披露Pwn2Own 17.5万美元漏洞链  
原创 Dr. Clay
                    Dr. Clay  黑白之道   2026-10-30 00:49  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M7ric1JVHicGztrxqFQZfTxUdr9epWcrAlQjicXFoYwfXHfpH0125mfWYpJoL9BYPVlwygdYYIjuKE6Zia6Su1k1iatJdDx7WjQzf0/640?from=appmsg "")  
> **导语**  
：没有内存破坏，没有UAF，也没有堆溢出。DEVCORE研究员Orange Tsai用一条纯逻辑漏洞链，把恶意网页变成了Windows上的本地代码执行。这条链在今年的Pwn2Own Berlin上价值17.5万美元，也拿下了2026年Pwnie的"Epic Achievement"奖。  
  
  
![Edge沙箱被逻辑漏洞链攻破](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MS3gPgiasrOTplxk1Liaako2JJVC3l4ic73YJPxbroCUmq0SvXNnuXn9jicfFz2IiaDSoNLEg11HK8VhiaNpvuVwP8AQT5G660QWUTI/640?from=appmsg "Edge沙箱被逻辑漏洞链攻破")  
## 一、事件概述  
  
2026年10月28日，DEVCORE高级研究员Orange Tsai在博客中公开了完整技术细节。这条链早在今年5月的Pwn2Own Berlin上就已击败微软Edge浏览器团队，拿下175,000美元奖金，如今是首次完整披露。  
  
需要强调的是，这不是一次真实攻击。微软在比赛次日（2026年5月15日）即发布修复，公开CVE记录中 exploitation 字段全部为 none，目前没有任何针对真实用户的利用证据。  
  
Tsai自述整条链不到100行代码，由他手工搭建、耗时约两个月，且全程未使用任何AI辅助。他写道："难的是把它们找出来，找到之后，串联反而是最简单的部分。"  
## 二、为什么值得关注：浏览器沙箱的"非内存"崩溃路径  
  
浏览器沙箱是阻止网页代码访问操作系统的墙，突破沙箱并在本地执行程序，才意味着一个网页漏洞升级为整机控制权。  
  
过去的浏览器逃逸几乎都依赖内存破坏——释放后重用、越界写、类型混淆。这次不一样。Tsai 的整条链只由 Edge 与 Windows 在处理**正常功能**  
时的逻辑错误组成，没有一个字节级的内存bug。这在工程上意味着两件事：  
  
一是"内存安全"投入并不能覆盖全部攻击面；二是单个看似无害的实现细节，只要错了信任归属，就能成为链条的第一环。  
  
Tsai 把最终产物定义为一个**通用型UXSS原语**  
——在任何网站上执行 JavaScript，并且能绕过本该阻止它的 CSP。攻击从浏览器自身进程发起，因此可以触达 google.com 这类普通网页漏洞根本碰不到的站点。  
## 三、漏洞链拆解：从通用UXSS到本地提权  
  
整条链分为两个阶段：先在浏览器自身进程内拿到任意站点执行权，再借助 Edge 内部能力落地为本地代码执行。  
### 第一阶段：四个逻辑缺陷拼出的UXSS  
  
起点是 Edge 面向企业单点登录设计的"条件访问导航节流"（conditional access navigation throttle）。触发该机制后，浏览器**自身进程**  
会发起一次与发起页面无关联的新导航，而 Edge 却把这次导航挂到了当前打开的任意标签页上。请求方与承载方之间的错配，就是这条链的第一个漏洞。  
  
要让整条链无需用户点击即可跑通，还需要再补四个弱点：  
- Edge 会校验请求是否来自可信的微软站点，Tsai 通过**第二个窗口**  
发起请求绕过了这项检查；  
  
- 弹窗拦截本应拦下这个窗口，但 Edge 内部存在一份**硬编码的放行地址列表**  
，其中一项恰好放行；  
  
- 受害者需处于已登录状态，Edge 的自动登录功能在访问微软站点时会主动完成这一动作；  
  
- 链条最后从 assets.msn.com  
 读取受害者邮箱，而该站因**跨域策略配置错误**  
，把邮箱地址返回给了任意网站。  
  
![多阶段攻击链示意](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6O13DXyUodu1GkO0asVicO2Payss3HTrFIrCamSBrJIaRsOePmLuuzdVaP32RiajCibJou7qSUcIRTwHJYOGzP6uQHYup8BaLuaibU/640?from=appmsg "多阶段攻击链示意")  
### 第二阶段：借内部协议跳出沙箱  
  
拿到 JS 执行权仍不等于控制整机。Tsai 转向 Edge 的沉浸式阅读器（Immersive Reader）——它运行在自定义的 read://  
 协议下，而微软**漏掉了把这个地址列入禁止执行脚本的页面名单**  
。  
  
从 read://  
 出发，链条调用 Edge 内部用于打包诊断日志的函数，滥用其写入能力，在磁盘任意位置写出**文件名与内容均由攻击者指定**  
的文件。这一步之后是两次跳转：  
```
read:// 沉浸式阅读器（未被列入禁脚本名单）   └─> 调用 Edge 内部"打包诊断日志"函数          └─> 任意路径 + 任意内容写文件                 └─> 覆写"协议处理器授权"配置文件                        └─> 重启 Edge 使配置生效                               └─> 触发 telnet:// 链接                                      └─> Windows 默认不带 telnet，                                          按 PATH 搜索并运行低权限用户                                          可写目录中的攻击者投放副本 → RCE
```  
  
最终落地的思路很巧妙：Windows 默认不预装 telnet 客户端，当系统按 telnet://  
 协议处理程序时，会在 PATH 中搜索可执行文件并运行它——而低权限用户可写的目录中，早已被前一步写入了一份攻击者伪造的副本。  
  
需要说明的是，其中"重启浏览器"这一步 Tsai 指出是为比赛舞台效果做的便利处理，真实攻击并不需要。  
## 四、修复情况与风险评估  
  
微软在2026年5月15日发布 Edge Stable 148.0.3967.70，修复了三项 Edge 专属漏洞。  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">CVE 编号</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">性质</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">影响版本</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">评分</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-45492</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">安全功能绕过</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Edge &lt; 148.0.3967.70</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5.4</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-45494</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">欺骗/仿冒</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Edge &lt; 148.0.3967.70</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5.4</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-45495</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">同批修复</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Edge &lt; 148.0.3967.70</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">—</span></section></td></tr></tbody></table>  
这里的评估落差值得警惕：微软记录中前两项仅评为 5.4 分（中危），DEVCORE 也承认，这个评分反映的是**单个缺陷孤立存在时的价值**  
，而不是串联之后达成代码执行的完整链。对于习惯按 CVSS 数字排定处置优先级的团队，这直接构成一个盲区。  
  
DEVCORE 称修复在披露后 24 小时内完成。  
## 五、验证方法：如何确认自己是否受影响  
  
版本检查是最直接的手段。在浏览器地址栏输入 edge://version  
，或在终端执行 microsoft-edge --version  
。  
- 版本 **≥ 148.0.3967.70**  
：已修复，**无需任何额外操作**  
，自动更新早已覆盖；  
  
- 版本 **< 148.0.3967.70**  
：立即更新至最新 Stable；  
  
- 排查项：检查用户可写目录下是否存在异常的 telnet.exe  
、tlnt.exe  
 副本；复核企业策略中的 Edge 自动登录行为。  
  
![防御与加固示意](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M1pPR1eR2oSNyHW1Qiaxfuyg1AhnIQMF3RdV5OfGQlSuoZ8Z5ewGSUtEFCMqwhXXmt40OafdjS3XO90OS1EqEkyxmnqniaic7aDg/640?from=appmsg "防御与加固示意")  
## 六、未来趋势预测  
  
**逻辑链将成为主流。**  
 现代浏览器中的内存破坏漏洞数量被长期压制，而逻辑缺陷——配置错误、协议实现、信任模型——成本极低且难以靠内存安全投入消除。这类"平淡无奇"的实现细节将持续成为攻击链的第一环。  
  
**评分机制滞后于真实攻击链。**  
 单点评级驱动处置的流程会越来越不可靠，CVSS 5.4 的中危漏洞串成 RCE 的案例只会更多，安全团队需要转向以攻击链可达性为核心的评估方式。  
  
**内部协议 scheme 需要被审计。**read://  
 这类浏览器内部协议正在变成新攻击面，传统只审计 http/https  
 的思路需要扩展到全部可注册处理器的内部入口。  
  
**UXSS 应作为一等防御对象。**  
 CSP 不再是万金油，需要配合浏览器侧针对内部 scheme 的脚本执行管控，才能真正收敛跨站脚本的影响半径。  
## 七、总结  
  
这条链的核心启示并非某个具体漏洞，而是一个方法论层面的提醒：现代浏览器的攻击面，已经从内存深处迁移到了信任逻辑的表层。Tsai 用时两个月手工完成，没有任何内存破坏，全程没有 AI 参与——这恰恰说明，在防御体系高度工程化的今天，**发现逻辑缺陷仍然是人类研究员最难被替代的能力**  
。  
  
而微软给出的 5.4 分，也提醒我们：漏洞的价值从来不取决于它的编号，而取决于它能被谁、在什么条件下、接到一起。  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OqYnCqUmKq5reCSFiaPdQibfiaiaEkmSic2Lrlt6HjTJEdY0rdsMmNGnZqfEniag0PicP9tGKOQMJTCqTkTmhicug6COhtOodzvgolvqQ/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6M3amSRCtNHmnwll47ibiaVib7f72fOQibgktKZ9UuTEzOeDJLIyGZodzaUQeriaYLT9RxqSEiaN5gzjpmbF557yqIL77MyPTPIoQ7Mk/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6O3qhCckiaApkUy3RrZBrgKKhXykBld433HpIYCJbY5WQ3BFtsn8cJzlYlZSmChXherwib0YBNwm7T0qgC0YWBnibTEhaeBeVE8kw/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6M6ib3Lc69FhtrpjjicztHXsJks5UvKT4LZAku14Xy0u3Z4vlsGSZcvFORWJDnVwgEVgu0WyfKYvffnc9swR3n6ViaFeoWPNnSvLM/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
