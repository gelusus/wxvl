#  Java4Shell深度解析：64个"A"就能RCE？Java史上最严重的内存破坏漏洞全揭秘  
原创 calif.io
                    calif.io  潇湘信安   2026-11-18 01:45  
  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">限时免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table>  
现在只对常读和星标公众号才展示大图推送，建议大家把“  
潇湘信安  
”  
设为星标  
，  
否则可能看不到了  
！  
  
## 导语  
> 漏洞出在 OpenJDK 的 Base64 解码器：手写汇编算错了填充掩码，64 个 "A" 加一个等号，就能往堆外写 62 个字节。找到它的是一个 AI——一小时出洞，两小时出可用利用，全程 token 成本不到 200 美元。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tNyBeBKReNKX5OzZXeYGiaPTvj2dia5lA4HicHZESMPSib2qs6a65M0pLGKkwdAOPU06V3wqxCmQg3hVkiblHl2qticfTbNYyugHdkEbNx4GZoW9k/640?wx_fmt=png&from=appmsg "")  
  
图：Java4Shell 官方主视觉，来源 Calif Research  
## 一、漏洞概述：什么是Java4Shell？  
  
2026年11月17日，安全研究公司 Calif 披露了一个代号为 **Java4Shell**  
 的严重漏洞，编号 **CVE-2026-114938**  
。这是一个存在于 OpenJDK Base64 解码器中的内存破坏漏洞，攻击者无需任何认证凭据，只需提交一段特制的畸形 Base64 数据，就能触发堆内存越界写入，进而导致内存泄露或远程代码执行（RCE）。  
  
Java 是全球第三大编程语言，支撑着 90% 的财富 500 强企业。从银行、航空、电信到国家安全系统，几乎所有关键行业都运行在 Java 之上。正因为 Java 应用如此广泛，一个核心组件的漏洞可能产生"Write once, RCE everywhere"（一次编写，处处RCE）的连锁效应——这正是 Java4Shell 名字的由来，也是它最可怕的地方。  
  
Base64 是一种将二进制数据表示为文本的常见编码方式，广泛用于认证协议、文件上传编码和网络消息传递。为了提升 Base64 解码速度，OpenJDK 在 x86-64 平台上使用了专门的 CPU 指令（AVX-512 VBMI）来批量处理数据。研究人员在这段经过高度优化的机器码中发现了一个 bug，使得畸形输入能够覆盖相邻内存。  
  
触发这个漏洞的输入简单到令人发指——只需 64 个 "A" 字符后跟一个等号 "="：  
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
```  
  
在较旧的 OpenJDK 17 版本中，触发输入则是 128 个 "A" 后  
跟一个等号。  
## 二、危害等级：CVSS 9.2，CRITICAL级别的威胁  
  
Java4Shell 的危害绝不仅仅是让程序崩溃那么简单。这个漏洞具备两级杀伤力：  
### 2.1 读：Java版Heartbleed（JavaBleed）  
  
越界写入会破坏堆上对象的长度字段，攻击者能够让应用程序将堆内存中的数据原样吐回响应——这就是 Java 版的 Heartbleed，研究团队直接称之为 **JavaBleed**  
。  
  
堆内存里可能有什么？会话令牌、私钥、个人身份信息（PII）……任何敏感数据都可能被泄露。  
### 2.2 写：远程代码执行（RCE）  
  
通过"改写数组长度"这种经典手法，攻击者可以将固定值越界写升级为可控值的线性写入，进而触及 Code Cache 和 JVM 钩子，最终实现**任意代码执行**  
。  
  
在 CVSS 4.0 评分标准下，该漏洞得分高达 **9.2 分（CRITICAL 严重）**  
。Oracle 在官方公告中使用 CVSS 3.1 评分给出的是 8.1 分（High 高危）。无论采用哪种标准，这都是一个必须立即重视的严重威胁。  
## 三、受影响产品：从Jenkins到Elasticsearch，名单触目惊心  
  
Calif 的研究团队在默认配置下对多款广泛部署的 Java 应用进行了测试，成功实现了远程代码执行。  
### 3.1 免认证RCE（Pre-auth RCE）  
  
无需登录，直接远程执行代码：  
- **Keycloak**  
 — 开源身份认证与访问管理系统  
  
- **Atlassian Jira**  
 — 全球最流行的项目管理工具之一  
  
- **Atlassian Confluence**  
 — 企业级知识管理与协作平台  
  
- **JFrog Artifactory**  
 — 通用制品仓库管理平台  
  
- **Jenkins**  
 — 最广泛使用的持续集成/持续部署工具  
  
- **PingFederate**  
 — 企业级身份联邦与单点登录解决方案  
  
### 3.2 需认证RCE（Post-auth RCE）  
  
登录后可触发远程代码执行：  
- **Apache Kafka Broker**  
 — 分布式消息队列  
  
- **Apache Solr**  
 — 企业级搜索平台  
  
- **Elasticsearch**  
 — 分布式搜索引擎与数据分析引擎  
  
- **Gerrit**  
 — 代码评审工具  
  
- **Spinnaker**  
 — 持续交付平台  
  
### 3.3 远不止于此  
  
这份名单远非全部。研究团队还在 Apache Struts、Spring Boot 等主流框架的示例应用上成功复现了漏洞。普通的认证流程、文件上传功能都可能调用到那个脆弱的 Base64 解码器，实际暴露面比这份测试名单大得多。  
  
更令人担忧的是**客户端侧**  
的利用。安全研究员 samczsun 贡献了一个客户端利用链——一个主动连接到恶意 TLS 服务器的 Java 程序，即使它本身不对外提供任何服务，也可能被攻破。  
## 四、技术原理：手写汇编中的致命bug  
  
Java 的核心卖点之一就是**内存安全**  
——Java 替开发者管理内存，检查内存访问，保护应用免受内存破坏漏洞的困扰。然而，Java 虚拟机本身（尤其是 HotSpot JIT 编译器）大量使用 C、C++ 甚至汇编语言编写，在这些底层代码中，内存安全问题依然存在。  
### 4.1 为什么会有手写汇编？  
  
为了追求极致性能，HotSpot 将热点函数以**手写汇编**  
的形式实现为 intrinsic（内建函数）。x86-64 平台的 Base64 intrinsic 就是其中之一，它使用 AVX-512 VBMI 指令（VMOVDQU8）一次可以搬移 64 字节数据，比普通实现快得多。  
  
问题恰恰出在这段手写汇编中——处理末尾填充的那几行代码。  
### 4.2 核心bug：掩码计算下溢  
  
Base64 intrinsic 的工作方式是：以 256 字节为块处理输入，如果不足 256 字节则以 64 字节为块处理。当输入少于 64 字节时，使用 evmovdqub  
 指令，该指令需要一个 64 位掩码（mask）来决定哪些字节被读取或写入。  
  
以下是处理末尾填充的伪代码：  
```
input_mask := bzhiq(-1, input_length)output_len := (input_length / 4) * 3// 处理填充if input[input_length - 1] == '=':    output_len -= 1    input_mask >>= 1    if input[input_length - 2] == '=':        output_len -= 1        input_mask >>= 1output_mask := bzhiq(-1, output_len)
```  
  
**问题出在哪里？**  
  
当输入末尾是 =  
、==  
、X=  
、X==  
、XY=  
、===  
 这几种形态时，output_len  
 先被减到 0，继续减就会**整数下溢**  
变成 -1 或 -2。而 bzhiq(-1, output_len)  
 在 output_len >= 64  
 时不会改变任何位，于是 output_mask  
 变成了全 1 的 -1u64  
。  
  
接下来这句：  
```
evmovdqub(Address(dest, dp), k1, tmp, true, Assembler::AVX_512bit);
```  
  
掩码全 1，意味着**无条件写满 64 字节**  
。  
### 4.3 越界写入是如何发生的？  
  
我们来算一笔账：  
- 输入：65 字节（64 个 "A" + 1 个 "="）  
  
- Java 侧计算的目标缓冲区大小：50 字节  
  
- 实际写入：从偏移 48 开始，写了 64 字节  
  
- **越界写入量：62 字节**  
  
Java 侧最终会抛出 IllegalArgumentException  
 拒绝这个非法输入，但此时堆内存已经被写坏了——拒绝得再快也来不及。  
  
在 OpenJDK 17 上，由于触发快路径的阈值不同，需要 128 个 "A" 加一个等号才能触发。  
## 五、利用链详解：从越界写入到远程代码执行  
  
从"能越界写"到"能跑代码"，中间隔着一条完整的利用链。Calif 团队展示了如何将一个看似有限的内存破坏升级为完整的远程代码执行。  
  
  
视频：Java4Shell 漏洞利用演示，来源 Calif Research  
### 5.1 第一步：固定写升级为可控写  
  
越界写入的那 64 字节中，可控程度非常有限：  
- 第 0 字节：完全可控  
  
- 第 1 字节：可控 4 bit  
  
- 其余字节：不可控，字节 48 到 63 固定为 0x9a  
  
研究团队使用了 JavaScript 引擎漏洞利用中的经典技术：**覆盖数组长度字段**  
。  
  
他们把被改写长度的数组称为 **writer**  
——一旦长度被撑大，它就会继续接受写入的数据。如果在它后面放置另一个数组或对象，就能用可控值破坏那个邻居，从而将固定值越界写升级为**可控值的线性写入**  
。  
  
接着，再破坏另一个数组的长度，让它的内容随响应返回，就得到了 **reader**  
——堆内存泄露原语，也就是前面提到的 JavaBleed。  
### 5.2 第二步：堆风水（Heap Feng Shui）  
  
有了 writer 和 reader 还不够，你需要让它们在内存中"恰到好处"地相邻。G1 垃圾回收器将堆划分为固定大小的 region，而 HotSpot 为每个线程分配私有 TLAB（线程本地分配缓冲区），不同请求线程的分配顺序完全不可控。  
  
研究团队采用了经典的 **Spray and Pray**  
 策略：  
- 通过对象大小和 padding 将偏移固定在 region 内  
  
- 喷洒多组 writer/reader 对，为解码器的输出准备多个可能的邻居  
  
- 当解码器输出的 region 恰好落在某个 writer 下方时，reader 会产生一个更大的响应  
  
- 落空就重试，直到成功  
  
为了帮助 AI 理解堆内存状态，Calif 还编写了一个小型堆追踪器和可视化工具。  
### 5.3 第三步：远程控制分配时机  
  
远程利用的一大挑战是如何控制目标机器上的分配时机。团队的做法是将请求分两段发送：  
1. 第一段发送到触发关键分配的位置  
  
1. **等待 TCP ACK**  
，确保数据已经进入目标机器的缓冲区  
  
1. 再发送剩余部分  
  
这样就能精确控制分配发生的时机。使用小包分片效果更好，因为小包的时序比大包更稳定。  
### 5.4 第四步：完整利用流程  
  
整条利用链分为以下几个阶段：  
- **WARM（预热）**  
：用合法的 Base64 数据反复解码，将 JIT 逼进那条脆弱的 intrinsic 快路径，此时还没有内存破坏  
  
- **REHEARSE（预演）**  
：可选的一轮安全校准  
  
- **TRIGGER（触发）**  
：发送带有畸形尾巴的输入，如果对象位置对了，固定溢出就会撑大 writer  
  
- **DISCLOSE（泄露）**  
：继续写入，将选定的字节灌入 reader，从响应中收集堆数据  
  
- **REPAIR（修复）**  
：用泄露到的原始数据把临时改坏的对象头和邻居恢复回去，保证 JVM 不崩溃  
  
- **FINISH（完成）**  
：执行应用层面的动作，用一个真实请求验证代码执行成功  
  
### 5.5 限制与注意事项  
  
需要明确的是：**没有万能 payload**  
。每条利用链都需要找到合适的解码路径、存活的 writer、能回传数据的 reader、合适的堆布局以及应用层面的动作——每个目标都得单独定制。Calif 自己也说，这"不是一把指哪打哪的枪"。  
  
另外，关闭 nginx 的 request buffering 并不能作为通用缓解措施。例如 Elasticsearch 是一条完整请求就跑完整个流程，缓冲机制挡不住；WebSocket 升级后的帧走隧道，请求体缓冲那套也不适用。  
## 六、影响范围：版本与硬件门槛  
### 6.1 受影响的Java版本  
  
受影响的是 **OpenJDK 17.0.6 及以上版本**  
。由于以下发行版都基于 OpenJDK 构建，它们也一并受影响：  
- Oracle Java SE  
  
- Oracle GraalVM for JDK  
  
- Amazon Corretto  
  
- Eclipse Temurin  
  
- Microsoft Build of OpenJDK  
  
- Azul Zulu  
  
- BellSoft Liberica  
  
- SAP SapMachine  
  
- Red Hat OpenJDK  
  
Oracle 在 2026 年 11 月 17 日的 Critical Patch Update 中给出的修复版本包括：  
  
<table><thead><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><th style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 2px;border-style: none none solid;border-color: currentcolor currentcolor rgb(4, 120, 87);border-image: none;font-size: 15px;line-height: 21px;text-align: center;background: transparent;color: rgb(19, 64, 52);font-weight: 700;letter-spacing: 0.3px;min-width: 100px;white-space: nowrap;"><section><span leaf="">产品线</span></section></th><th style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 2px;border-style: none none solid;border-color: currentcolor currentcolor rgb(4, 120, 87);border-image: none;font-size: 15px;line-height: 21px;text-align: center;background: transparent;color: rgb(19, 64, 52);font-weight: 700;letter-spacing: 0.3px;min-width: 100px;white-space: nowrap;"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 8</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">8u511b33 / 8u511b33-perf</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 11</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">11.0.33.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 17</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">17.0.21.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 21</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">21.0.13.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 25</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">25.0.5.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">Java SE 27</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">27.0.1.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">GraalVM for JDK 17</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">17.0.21.0.3</span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">GraalVM for JDK 21</span></section></td><td style="margin: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">21.0.13.0.3</span></section></td></tr></tbody></table>  
  
值得注意的是，Java 8 和 11 分支也在这次更新中打上了补丁。  
### 6.2 硬件门槛  
  
这段有问题的代码是绑定硬件的，要求 **x86-64 架构且支持 AVX-512 VBMI 指令集**  
。ARM64 平台不受影响。  
  
主流受影响的 CPU 包括：  
- **Intel Xeon**  
：Ice Lake、Sapphire Rapids、Emerald Rapids、Granite Rapids  
  
- **AMD**  
：EPYC 与 Ryzen Zen 4 及以后的架构  
  
Linux 上可以用以下命令检查 CPU 是否受影响：  
```
grep -m1 '^flags' /proc/cpuinfo | tr ' ' '\n' | grep -E '^(avx512f|avx512bw|avx512vbmi|bmi2)$'
```  
  
四个 flag 必须齐全，缺一个就说明 AVX-512 VBMI 那条快路径走不到，也就不受该漏洞影响。  
## 七、修复与缓解措施  
### 7.1 官方修复方案  
  
OpenJDK 官方安全通告和 Oracle 补丁均于 **2026 年 11 月 17 日美西时间 13:00**  
 发布。**能升级就升级**  
，这是最彻底的解决方案。  
- Oracle 安全公告：  
https://www.oracle.com/security-alerts/cspunov2026.html  
  
- OpenJDK 安全通告：  
https://openjdk.org/groups/vulnerability/advisories/2026-11-17  
  
### 7.2 临时缓解措施  
  
如果暂时无法升级，可以通过启动参数禁用相关功能来规避漏洞：  
  
**方案一：禁用 Base64 intrinsic**  
```
-XX:-UseBASE64Intrinsics
```  
  
关闭硬件加速的 Base64 编解码功能。这是最直接的缓解方式。  
  
**方案二：禁用 AVX-512**  
```
-XX:UseAVX=2
```  
  
将 AVX 级别限制为 2，从而整体禁用 AVX-512。  
> ⚠️ **注意**  
：ARM 平台上的 OpenJDK 不支持 -XX:UseAVX=2  
 参数，会导致启动失败。ARM 用户请使用第一种方案。  
  
  
这些都只是临时缓解措施，所有用户最终都应该升级到最新的安全版本。  
## 八、POC验证：64个"A"加一个等号  
  
Calif 在披露文章中提供了一个自包含的 PoC 验证程序。它的原理是：先把 Base64 解码器"热身"到走那条脆弱的 intrinsic 快路径，再把 64 个 "A" 加一个等号的畸形输入解码进一个 4096 字节的数组。  
  
在受影响的 JDK 上，程序会打印 REPRODUCED  
——解码器虽然拒绝了输入，但输出缓冲区之后的 62 个字节（偏移 50 到 111）已经被覆盖了。所有写入都落在真实分配的数组内，运行是安全的。  
```
import java.nio.charset.StandardCharsets;import java.util.Arrays;import java.util.Base64;public final class Base64Trigger {    private static volatile int sink;    public static void main(String[] args) {        Base64.Decoder decoder = Base64.getDecoder();        byte[] valid = "A".repeat(64).getBytes(StandardCharsets.US_ASCII);        byte[] malformed = ("A".repeat(64) + "=").getBytes(StandardCharsets.US_ASCII);        byte[] target = new byte[4096];        for (int i = 0; i < 200_000; i++) {            sink += decoder.decode(valid, target);        }        Arrays.fill(target, (byte) 0x5a);        boolean rejected = false;        try {            decoder.decode(malformed, target);        } catch (IllegalArgumentException expected) {            rejected = true;        }        int changed = 0, first = -1, last = -1;        for (int i = 50; i < target.length; i++) {            if (target[i] != (byte) 0x5a) {                changed++;                if (first < 0) first = i;                last = i;            }        }        System.out.println("runtime=" + System.getProperty("java.runtime.version"));        System.out.println("vm=" + System.getProperty("java.vm.name"));        System.out.printf("rejected=%s changed=%d first=%d last=%d%n",                rejected, changed, first, last);        if (!rejected || changed != 62 || first != 50 || last != 111) {            System.err.println("NOT REPRODUCED");            System.exit(1);        }        System.out.println("REPRODUCED");    }}
```  
  
编译运行：  
```
javac Base64Trigger.javajava Base64Trigger
```  
  
Calif 官方**暂未公开**  
各具体产品的武器化利用链，理由是给所有人留出升级时间。团队表示之后会公开相  
## 九、发现者的故事：AI与"半人马黑客"  
  
这个漏洞的发现过程本身就值得大书特书——它是 AI 辅助漏洞挖掘的又一个里程碑。  
### 9.1 一小时发现，两小时利用  
  
Calif 的研究员 Quang Luong 拉下来 OpenJDK HotSpot 的源码，然后让 **OpenAI Codex**  
 AI 编码助手像 2026 年的正经安全研究员那样去审计 JIT 编译器、intrinsics、常用 API 和密码学相关代码。  
  
一开始的 20 分钟，Codex 给出的全是不可达或者利用条件不成立的"垃圾结论"。但 Quang 没有放弃，继续给 AI 更精确的提示，直到 Codex 报出了 Base64 解码器中的内存破坏问题。  
- **发现漏洞**  
：约 1 小时  
  
- **写出第一个可用利用**  
：约 2 小时  
  
- **全程 token 成本**  
：不到 200 美元  
  
### 9.2 半人马黑客（Centaur Hacker）  
  
Calif 团队将这种人机协作模式称为 **"半人马黑客"**  
——最好的黑客既不是纯 AI，也不是纯人类，而是人与 AI 的结合体。AI 提供算力和广度，人类研究员提供上下文和深度判断。  
  
关于这个漏洞还有一点值得注意：不存在通用的利用方式。根据 Base64 的使用方式和应用程序的不同，利用步骤也会有所不同。Calif 通过先构建流水线和策略，再指导 AI 代理使用这些策略进行初始利用，大大加速了利用链的开发。之后，大多数后续的利用链几乎都是由 AI 自主构建的。  
## 十、深度思考：公地悲剧与Vibe-Uncoding  
  
Java4Shell 不仅仅是一个技术漏洞，它还引发了更深层次的思考。  
### 10.1 公地悲剧（The Tragedy of the Commons）  
  
Java 是我们所有人都依赖的共享软件基础设施之一。多年来，它不断膨胀以满足各种不同的需求。每个功能请求单独看可能都是合理的，但所有人都在继承由此增加的复杂性——需要某个功能的人得到了好处，而其他所有人都在承担这些代码及其带来的风险，即使他们从来不用。  
  
Java4Shell 就是一个活生生的例子。Base64 硬件加速优化对那些需要解码海量数据的应用很有帮助，但这段代码却被分发给了所有人，包括那些从中获益甚微的应用。在受影响的系统上，一个几乎没人需要的优化变成了严重的安全风险。  
  
功能可以很有用，大多数 bug 也可以被修复。但问题是——我们为什么要为那些从一开始就不需要的代码担惊受怕？  
### 10.2 Vibe-Uncoding：删除的艺术  
  
如果你不需要某个东西，就没有理由让它继续在你的系统里运行。你的应用需要那个手写汇编的 Base64 解码器来处理区区几百字节吗？大概率不需要。  
  
今年早些时候，Calif 还发现了 Squidbleed——Heartbleed 的一个"远房表亲"，它从 1997 年起就藏在 Squid 代理服务器里。这个 bug 出在 Squid 的 FTP 模块里。但现在还有谁在用 FTP？这是一个早就该被移除的僵尸协议。  
  
删除代码说起来容易做起来难。一个看似未使用的功能可能有隐藏的依赖。需要有人搞清楚什么可以删，做出修改，然后测试剩下的部分。  
  
这正是 AI 可以大显身手的地方——它让这项工作在以前无法承受的规模下变得切实可行。  
  
发现了 PHP2Shell 和 Java4Shell 的"半人马黑客"，同样可以梳理大型代码库，标记出死代码、臃肿的功能和不必要的优化。Calif 称之为 **vibe-uncoding**  
：识别系统真正需要的部分，然后删掉其他一切来缩小攻击面。  
  
所有人都在用 AI 问"我们能造什么？"但对于安全来说，也许正确的问题是——**"我们能删什么？"**  
## 参考来源  
- Calif Research：  
Java4Shell — Write once, RCE everywhere  
  
- OpenJDK Vulnerability Advisory 2026-11-17：  
https://openjdk.org/groups/vulnerability/advisories/2026-11-17  
  
- Oracle Critical Patch Update — November 2026：  
https://www.oracle.com/security-alerts/cspunov2026.html  
  
- BleepingComputer：  
Critical Java4Shell flaw allows RCE attacks on major Java applications  
  
**版权声明**  
：本文整合自公开安全披露与技术分析，图片来源于 Calif Research。仅供安全研究与防护参考，请遵守相关法律法规。  
  
  
**关注我们**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
 还在等什么？赶紧点击下方名片开始学习吧   
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
  
  
**知 识 星 球**  
  
  
  
星球已过800人，暂不再发放优惠券，如还有需要的师傅可加我  
VX  
：**S_3had0w，等你一起来学习...！**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XOPdGZ2MYOe8h8vDq3IN1GhNuNnkbWYgcibYRicgo0SRy1febylE4kBDdibgIJV1ia1QNSBYiaZAZWCzAjESoYfxNjw/640?wx_fmt=jpeg&from=appmsg "")  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">限时免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table><table><tbody><tr><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安协同知识库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/mmbiz_jpg/zqnzfCSS1kB81aIibHjJDKmPS8BOiacfOmDUsm8ibwibG3bvo3EsfbkqO70BJNIo7rE4LqOia3xKRITWTZSOHKLwOO2ribkueicxLrLeEsV1FfDicus/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_jpg%2FzqnzfCSS1kDw29J3ia1mGnlrZzHxa4HsTezib66icQ99G6vicWPvurHOicRL3wMv4PUyOibicDeN00EnibyP8feJsmyKUYnRHNRP8CA8Ja2HnPB6Ed8%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安学习资料库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/sz_mmbiz_jpg/zqnzfCSS1kC6ohbbn1EEAjmjkMbLqKC0TevicIR9DOX2IiafOBL5J7ygLr5khHY1XvTibQjqsA412O6HicaL1eyHvANjXdfI5DkCJWficVQibIzia0/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2FzqnzfCSS1kABnOfCKmD6QCfNgmCk1eSkyoGEqZHw5Z54qG2khyAQ32PcGWx9Fz4ESFE2wjVOyRuNLtBhnl3jRbnY7clNeC2uppGr6jXArJ0%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td></tr></tbody></table>  
  
**推 荐 阅 读**  
  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247499188&idx=1&sn=9ce15a0e66b2595285e544aaa0c49c24&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247496043&idx=1&sn=4daa27ade9915de6021fea1c2a21d7bc&scene=21#wechat_redirect)  
  
[](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdAPjIVeN2ZahG9ibP0Y3wlfg6BO1WO7MZfo1JeW7zDWcLSTQ5Ek8zXAia5w1nMnogpbpXP6OxXXOicA/640?wx_fmt=png "")  
  
