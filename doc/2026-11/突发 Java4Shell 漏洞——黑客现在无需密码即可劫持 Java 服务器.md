#  突发 Java4Shell 漏洞——黑客现在无需密码即可劫持 Java 服务器  
原创 Atlas Chen
                    Atlas Chen  黑白之道   2026-11-17 22:49  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NicdCXibKOicsAk8iaAgENJOogaic2Jybf7oGwwpveLrAEAvicxa8Lb8QADMibldT0CjHzicC34MqyibHZNuI3Xd8Y23s2V6VdEcmib0rVs/640?from=appmsg "")  
> **导语**  
：漏洞出在 OpenJDK 的 Base64 解码器：手写汇编算错了填充掩码，64 个 "A" 加一个等号，就能往堆外写 62 个字节。找到它的是一个 AI——一小时出洞，两小时出可用利用，全程 token 成本不到 200 美元。  
  
  
![Java4Shell 堆内存越界写示意图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6OKhqVcZs5pVxlibb39V7Oq2PMA6xBUicoVmI4rzCjjK4icwyAVubogLFiaoJ9amgibPibb5Ara1Du0cyg29RvewbD5gatuiaKOMNUDzs/640?from=appmsg "Java4Shell 堆内存越界写示意图")  
## 危害：不是崩溃，是免认证的读和写  
  
结论先摆出来。这个漏洞有两级危害。  
  
**读**  
。越界写破坏了堆上对象的长度字段，攻击者能让应用把堆里的东西原样吐回响应 —— 这就是 Java 版的 Heartbleed，团队直接管它叫 JavaBleed。堆里有什么？会话令牌、私钥、PII，一概能读。  
  
**写**  
。用"改长度"这套经典手法，把固定值写升级成可控值的线性写，再够到 Code Cache 和 JVM 钩子，就是任意代码执行。  
  
实测拿到 RCE 的产品，默认配置：  
  
免认证 RCE：Keycloak、Atlassian Jira、Atlassian Confluence、JFrog Artifactory、Jenkins、PingFederate  
  
需认证 RCE：Apache Kafka Broker、Apache Solr、Elasticsearch、Gerrit、Spinnaker  
  
名单不完整。团队还在 Apache Struts、Spring Boot 这类框架的示例应用上复现了。普通的认证流程、文件上传都可能调用到那个 Base64 解码器，暴露面比这份名单大得多。  
  
更狠的是客户端侧：samczsun 贡献了一个**客户端**  
利用，一个主动连上恶意 TLS 服务器的 Java 程序，自己不提供任何服务也能被打穿。  
  
CVSS 4.0 打 9.2（CRITICAL），OpenJDK 官方公告里用的 CVSS 3.1 是 8.1。  
  
![Java4Shell 官方主视觉](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PuiaicUEJd8jdsx29K35fHMNj45pcxv4Bibicib42ghca3ePHwwriahYqaoibiaOcIAHVZnx7F5DxNteGDMouqzIeNLicdk9NPibFNhxv6M/640?from=appmsg "Java4Shell 官方主视觉")  
  
（图：Java4Shell 官方主视觉，来源 Calif Research）  
## 原因：JIT 里的手写汇编算错了掩码  
  
Java 的卖点之一是内存安全，但 HotSpot 为了性能，把热点函数用**手写汇编**  
实现成 intrinsics。x86-64 的 Base64 intrinsic 就是这么一条路，用 AVX-512 VBMI 指令（VMOVDQU8）一次搬 64 字节。  
  
问题出在处理末尾填充的那几行：  
```
input_mask := bzhiq(-1, input_length)output_len := (input_length / 4) * 3if input[input_length - 1] == '=':    output_len -= 1    input_mask >>= 1    if input[input_length - 2] == '=':        output_len -= 1        input_mask >>= 1output_mask := bzhiq(-1, output_len)
```  
  
当尾巴是 =  
、==  
、X=  
、X==  
、XY=  
、===  
 这几种形态时，output_len  
 先被减成 0，再减就**下溢**  
成 -1 或 -2。而 bzhiq(-1, output_len)  
 在 output_len ≥ 64 时一个位都不改，于是 output_mask  
 变成全 1 的 -1u64。接下来这句：  
```
evmovdqub(Address(dest, dp), k1, tmp, true, Assembler::AVX_512bit);
```  
  
掩码全 1，意味着**无条件写满 64 字节**  
。  
  
对上数字：输入 65 字节（64 个 A 加一个 =）时，Java 侧算出目标缓冲区只有 50 字节，解码过程却从偏移 48 开始写了 64 字节 —— 越界 62 字节。JDK 17 上因为触发快路径的阈值不同，需要 128 个 A 加一个 =。  
  
Java 侧最后会抛 IllegalArgumentException 拒绝这个输入，但堆已经被写坏，拒绝得再快也来不及。  
  
![Calif Research 官方披露页](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OLibImSOTb4s2DfGI0pVF9uibMpvOKUlSWicjBXamA4Vry2ClcLchicZrSf0wT8EGnpq8HIuLpmXJSMVrRk72UAxfJeg17WAdZCGs/640?from=appmsg "Calif Research 官方披露页")  
  
（图：Calif Research 官方披露页，来源 calif.io）  
## 利用：先把堆摆成想要的样子，再动手  
  
  
从"能越界写"到"能跑代码"，Calif 走了完整的一条链。  
  
**第一步，固定写升级成可控写。**  
 越界写的那 64 字节里，只有第 0 字节完全可控，第 1 字节可控 4 bit，其余不可控，字节 48 到 63 固定是 0x9a。团队用的是 JavaScript 引擎里那套经典手法：覆盖一个数组的长度字段，长度一旦被改大，它就会继续接受塞进去的值 —— 这就是 writer。再把另一个数组的长度改坏、让它的内容随响应返回，就得到 reader，也就是堆泄漏原语 JavaBleed。  
  
**第二步，G1 堆的内存布局。**  
 G1 把堆切成固定大小的 region，而 HotSpot 给每个线程分一块私有 TLAB，不同请求线程的分配顺序完全不可控。做法是 spray and pray：对象大小和 padding 把偏移钉死在 region 内，喷多组 writer/reader 对，给解码器的输出准备多个可能的邻居；谁的 region 刚好落在 writer 下面，谁就中奖，落空就重试。  
  
**第三步，远程控制分配时机。**  
 请求分两段发：第一段发到能触发关键分配的地方，然后**等 TCP ACK**  
，确保数据已经进了目标机器的缓冲区，再把剩下的发出去。这样就控制了分配发生的时机。要用小包分片，因为小包比大包时序更稳。  
  
**第四步，整条链跑起来。**  
- **WARM**  
：用合法 Base64 反复解码，把 JIT 逼进那条快路径，此时还没有内存破坏  
  
- **REHEARSE**  
：可选的一轮安全校准  
  
- **TRIGGER**  
：发畸形尾巴，对象位置对了，固定溢出就撑大了 writer  
  
- **DISCLOSE**  
：继续写，把选定的字节灌进 reader，从响应里收堆数据  
  
- **REPAIR**  
：用泄漏到的原始数据把临时改坏的表头和邻居恢复回去，保证 JVM 不崩  
  
- **FINISH**  
：做应用层面的动作，用一个真实请求验证  
  
限制也得说清楚：**没有万能 payload**  
。每条链都需要解码路径、活着的 writer、能回传的 reader、堆布局和应用动作，每个目标都得单独做一版。Calif 自己讲，这"不是一把指向哪打哪的枪"。  
  
还有一个容易被误当成缓解手段的点：**关掉 nginx 的 request buffering 不算通用缓解**  
。Elasticsearch 是一条完整请求就跑完，挡不住；WebSocket 升级之后的帧走隧道，请求体缓冲那套也不适用。  
## 影响范围：CPU 也得对  
  
受影响的是 **OpenJDK 17.0.6 及以上**  
。因为这些发行版都基于 OpenJDK，下面这些一并中招：Amazon Corretto、Eclipse Temurin、Microsoft Build of OpenJDK、Azul Zulu、BellSoft Liberica、SAP SapMachine、Red Hat OpenJDK、Oracle OpenJDK、Oracle Java SE、Oracle GraalVM for JDK。  
  
Oracle 在 11 月 17 日的 CPU 里给出的修复版本是：Java SE 8u511b33 / 8u511b33-perf、11.0.33.0.3、17.0.21.0.3、21.0.13.0.3、25.0.5.0.3、27.0.1.0.3 —— 8 和 11 分支同样在这次 CPU 里打了补丁。  
  
**硬件门槛**  
：这段代码是绑硬件的，要求 x86-64 且支持 AVX-512 VBMI，ARM64 不受影响。主流能踩到的 CPU：Intel Xeon 的 Ice Lake / Sapphire Rapids / Emerald Rapids / Granite Rapids，AMD EPYC 与 Ryzen Zen 4 及以后。  
  
Linux 上查自己有没有中招：  
```
grep -m1 '^flags' /proc/cpuinfo | tr ' ' '\n' | grep -E '^(avx512f|avx512bw|avx512vbmi|bmi2)$'
```  
  
四个 flag 必须齐全，缺一个就说明 AVX-512 VBMI 那条快路径走不到。  
## 修复：两个启动参数能挡  
  
OpenJDK 官方通告和 Oracle 补丁都在 **2026 年 11 月 17 日美西时间 13:00**  
 发布，能升就升。  
  
升不了的话，OpenJDK 启动时加下面两个参数之一：  
```
-XX:-UseBASE64Intrinsics
```  
  
关掉硬件加速的 Base64 编解码；或者在 x86-64 上加：  
```
-XX:UseAVX=2
```  
  
把 AVX-512 整体关掉。**注意 ARM 上的 OpenJDK 不认第二个参数，会直接起不来。**  
## POC：官方自带的独立触发器  
  
Calif 在文章里给了自包含 PoC：先把 Base64 解码器热身到走那条 vulnerable intrinsic，再把 64 个 A 加一个等号的畸形输入解码进一个 4096 字节的数组。受影响 JDK 上会打印 REPRODUCED —— 解码器拒绝了输入，但输出缓冲区之后 62 个字节（偏移 50 到 111）已经被覆盖。所有写入都落在真实数组里，跑起来是安全的。  
```
import java.nio.charset.StandardCharsets;import java.util.Arrays;import java.util.Base64;publicfinalclassBase64Trigger {    privatestaticvolatileint sink;    publicstaticvoidmain(String[] args) {        Base64.Decoderdecoder= Base64.getDecoder();        byte[] valid = "A".repeat(64).getBytes(StandardCharsets.US_ASCII);        byte[] malformed = ("A".repeat(64) + "=").getBytes(StandardCharsets.US_ASCII);        byte[] target = newbyte[4096];        for (inti=0; i < 200_000; i++) {            sink += decoder.decode(valid, target);        }        Arrays.fill(target, (byte) 0x5a);        booleanrejected=false;        try {            decoder.decode(malformed, target);        } catch (IllegalArgumentException expected) {            rejected = true;        }        intchanged=0, first = -1, last = -1;        for (inti=50; i < target.length; i++) {            if (target[i] != (byte) 0x5a) {                changed++;                if (first < 0) first = i;                last = i;            }        }        System.out.println("runtime=" + System.getProperty("java.runtime.version"));        System.out.println("vm=" + System.getProperty("java.vm.name"));        System.out.printf("rejected=%s changed=%d first=%d last=%d%n",                rejected, changed, first, last);        if (!rejected || changed != 62 || first != 50 || last != 111) {            System.err.println("NOT REPRODUCED");            System.exit(1);        }        System.out.println("REPRODUCED");    }}
```  
  
编译运行：  
```
javac Base64Trigger.javajava Base64Trigger
```  
  
官方**暂未公开**  
各具体产品的武器化利用链，理由是给所有人留升级时间，团队说之后会公开。  
  
原文与下载地址：  
- 完整技术分析 + PoC：https://calif.io/java4shell/  
  
- OpenJDK 官方安全通告：https://openjdk.org/groups/vulnerability/advisories/2026-11-17  
  
- Oracle Critical Patch Update：https://www.oracle.com/security-alerts/cspunov2026.html  
  
- 纽约时报报道：https://www.nytimes.com/2026/11/17/us/politics/ai-security-bug-java.html  
  
## 找到它的，是一个 AI  
  
这条链子里最值得记的不是 64 个 A，是研究员 Quang Luong 的说法：他把 OpenJDK HotSpot 拉下来，让 Codex 像 2026 年的正经安全研究员那样去审 JIT 编译器、intrinsics、常用 API 和密码学代码。头 20 分钟 Codex 给的全是不可达、利用条件不成立的垃圾结论，他继续提示，直到 Codex 报出 Base64 解码器里的内存破坏。  
  
发现漏洞花了 1 小时，写出第一个可用利用又花了 2 小时，全程 token 成本不到 200 美元。团队把这套人机协作叫"半人马黑客"。  
## 参考来源  
- Calif Research：Java4Shell — Write once, RCE everywhere  
  
- OpenJDK Vulnerability Advisory 2026-11-17  
  
- Oracle Critical Patch Update — November 2026  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。文中图片来源于 Calif Research。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PiaUpMH0U761WWJrSfUGlB1qAribmEDXQodkskx6QU3dnicU6grv3wePWGoKd5yeLGWpo649FV74nO8IEFmvQR5iaHMurTRqZ0m9I/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6N9PibkuiaRqpribbJiapVicZyOzHZqtXKRffV0JhmkwkLIHcefpe3TnFXsMzAuD3YmVEFRI3hXtRw8j6raicBicrL0ve86E04b0OPdiaE/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PsBibkLkwBrVkoIO65rlDWib6Rlo9EDFCHvKLpuCxe8r1dN2PbWA3ibhUBNicXnJDBjPcPWM8fEL3mtyh3tSYl6zWicickvoDq4q5T8/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6Mp53K72MHdePxibvSvJXzuY7I62dUEFm2ibnO5ZunGrOSKDppUCaibTOiaoJYmp6ma6ghH1A23k6LSaWcSajcaicodvYRwermcoBibs/640?from=appmsg "")  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
