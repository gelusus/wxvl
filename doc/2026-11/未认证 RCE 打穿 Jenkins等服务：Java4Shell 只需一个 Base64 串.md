#  未认证 RCE 打穿 Jenkins等服务：Java4Shell 只需一个 Base64 串  
红队安全圈
                    红队安全圈  红队安全圈   2026-11-18 04:12  
  
30 秒看懂  
  
影响范围 ·   
OpenJDK 17.0.6 以上的 17 / 21 / 25 / 27 四个系列，且必须跑在支持 AVX-512 VBMI 的 x86-64 服务器上；ARM 与 Java 8 / 11 不受影响  
  
利用状态 ·   
截至 11 月 17 日未见在野利用；发现方已放出只验证触发、不做控制流劫持的 PoC，另有研究者称已构出可用利用  
  
补丁状态 ·   
已修  
11 月 17 日发布 17.0.21.1 / 21.0.13.1 / 25.0.5.1 / 27.0.1.1，先查 CPU 再排修复优先级  
  
JVM 里最普通的 Base64 解码，被一个 65 字节的畸形字符串捅穿了堆。写出去的字节不受控，但落点受控，于是先泄露内存、再拿代码执行。这条链不需要账号。  
  
## 01  事情是什么  
  
OpenJDK 为了把 Base64 解码做快，在支持 AVX-512 VBMI 的 x86-64 处理器上启用了手写汇编的 intrinsic：一次处理 64 字节，比纯 Java 实现快出数倍。问题出在处理尾部那几个字节的代码上，它把输出长度算错了，于是一段 64 字节的向量写操作落到了缓冲区外面。  
  
触发条件简单到不像真的：64 个 A，加一个等号。解码器为这 65 字节算出 50 字节的结果数组，尾部逻辑却把它算成负一，写掩码被全部打开，64 字节整体写出，越界 62 字节。Java 之后才发现输入非法并抛出异常，但越界写已经发生。  
  
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
  
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=  
  
# 共 64 个 A 加一个等号  
  
## 02  为什么危险  
  
越界写进去的内容是固定的，来自解码器内部的查找表，攻击者改不了。所以这个洞一开始看起来只能打崩 JVM。转折点在于落点可控：数组大小决定它被分配到哪块堆、紧挨着谁。  
  
覆盖掉相邻数组的长度字段，固定字节的溢出就升级成有边界的任意值写入；再覆盖另一个数组的长度并让它随响应返回，就有了任意读。到这里，泄露堆里的密钥、令牌已经成立。剩下的工作是把读写原语推到代码缓存和 JVM 钩子，换掉执行流。  
  
output_len = (input_len / 4) * 3  
  
if input[-1] == '=': output_len--  
  
output_mask = bzhiq(-1, output_len)  
  
# 长度变 -1，掩码全 1  
  
工程上的难度全在堆排布。G1 把堆切成等大 region，线程还有私有的分配缓冲，请求线程不同、顺序就乱；要靠喷对象、算大小、留空洞，把解码结果摆到目标数组下面。远程版本还要用分包发送控制分配时机，对往返延迟和抖动敏感，失败最常见的表现就是 JVM 直接崩。  
  
## 03  谁受影响  
  
版本门槛和硬件门槛缺一不可，两条都满足才谈得上暴露。  
  
·   
Java 线：OpenJDK 17.0.6 起，17 / 21 / 25 / 27 全线。下游发行版同样中招，Amazon Corretto、Eclipse Temurin、微软 Build of OpenJDK、Azul Zulu、BellSoft Liberica、SAP SapMachine、Red Hat OpenJDK 都在列。  
  
·   
硬件：Intel Ice Lake、Sapphire Rapids、Emerald Rapids、Granite Rapids 的 Xeon，以及 AMD Zen 4 及更新的 EPYC 与 Ryzen。ARM64 不走这条路径，不受影响。  
  
·   
应用侧：只要把攻击者可控的输入喂给标准 Base64 解码器就算。实测默认配置下可未认证打穿的包括 Keycloak、Jira、Confluence、JFrog Artifactory、Jenkins、PingFederate；需要登录的有 Kafka Broker、Solr、Elasticsearch、Gerrit、Spinnaker；Spring Boot、Struts 这类框架的样例应用也能打。  
  
一个容易漏判的细节：MIME 解码器不走这条路径，基本解码器和 URL 解码器才走。判断某段认证逻辑是否暴露，要看它到底调的是哪个解码器。  
  
## 04  怎么修  
  
1   
升级到 17.0.21.1、21.0.13.1、25.0.5.1、27.0.1.1，或发行版对应的修复构建。  
  
2   
容器镜像要重建基镜像后重新部署，只换宿主机上的 JDK 没有用。  
  
3   
改完重启进程，用 jcmd 确认实际跑的是新版本。  
  
4   
短期缓解加启动参数，优先第一个，它只关掉 Base64 intrinsic，不影响其它向量运算。  
  
-XX:-UseBASE64Intrinsics  
  
另一个选择是 -XX:UseAVX=2，但它只对 x86-64 有效，ARM 上的 JVM 不认这个选项，写进公共启动配置会让服务起不来。  
  
## 05  检测与排查  
  
先确认机器是不是那批 CPU，再看运行的 JVM 到底是什么版本和参数。  
  
grep -o avx512vbmi /proc/cpuinfo  
  
jcmd <PID> VM.version  
  
jcmd <PID> VM.flags -all  
  
再找被触发过的痕迹：JVM 异常崩溃留下的 hs_err_pid 日志、不明原因的进程重启、网关与应用日志里长度不是 4 的倍数的 Base64 入参。这个洞的利用失败率高，崩一次就留一份崩溃日志，蓝队反而好发现。  
  
## 06  红队视角  
  
短期价值不在稳定拿 shell，而在面广。Jenkins、Jira、Confluence、Artifactory、Keycloak 是内网最常见的入口，默认配置就能打，前置条件少。但每打一个应用都要单独做堆排布的 profile，没有通杀 payload；网络稍差就会崩服务，属于会主动暴露自己的动作。  
  
更现实的用法是把它当权限验证的最后一环，而不是第一步：先确认目标 JVM 版本、CPU 标志，以及认证入口是不是真的调了受影响的解码器，再决定要不要动手。防守方要盯的也是这三件事，另外再加一条：给所有 Base64 入参加一道长度校验，长度不是 4 的倍数直接拒。  
  
## 07  一句判断  
  
值得连夜修，但不是因为它已经被打。真正的理由是这条路径埋在所有 Java 服务的认证入口下面，而修复只是换一个补丁版本。等 PoC 变成武器再动手，代价就不只是重启了。  
  
## 08  POC 链接  
  
https://calif.io/java4shell/  
  
https://openjdk.org/groups/vulnerability/advisories/2026-11-17  
  
https://securitylabs.datadoghq.com/articles/java4shell-remote-code-execution-in-openjdk-cve-2026-114938/  
  
第一份是发现方原文，含可安全触发、不做控制流劫持的 PoC 与完整利用链；第二份是 OpenJDK 官方公告，版本与修复构建以此为准。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
