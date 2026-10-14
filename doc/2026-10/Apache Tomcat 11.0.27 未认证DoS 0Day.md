#  Apache Tomcat 11.0.27 未认证DoS 0Day  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-14 00:58  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6OoOMDN7BJuvnGPsoGvFn7PqLicdibAzshsQFWcsoBCTCNC5ibgibaiaHOZR04SmPyNcwIhskQboJhgAc5SLMrnDgdMGCZpbBicHMwpk/640?from=appmsg "")  
> **导语**  
：安全研究员 abraxas 在 X 上公开了一个针对 Apache Tomcat 11.0.27 的未认证远程 DoS 0Day——一个普通的 GET 请求就能把 HTTP connector 线程钉死，循环里没有 round cap，乘以 maxThreads 就能把整台服务器堵到无法响应。漏洞尚未分配 CVE，官方 11.0.27 release 没有 security advisory，目前只能靠 main 分支两个 commit 止血。本文是技术要点编译报道。  
  
  
![Tomcat RewriteValve DoS 0Day 攻击概念图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6PYbsKnBJn7pBKknphfbvFkG83g47QlcVDlT52xu5PFBIYVP8J9OyRdPKjU4BPpU7icyPjIiaLSASIib7Xm7T38IShNJyJCW37UdY/640?from=appmsg "Tomcat RewriteValve DoS 0Day 攻击概念图")  
## 一、漏洞一句话  
  
Apache Tomcat 11.0.27 的 RewriteValve 在解析规则 flag 时用的是 **prefix-match**  
 而不是 exact-match——NC  
 或 NE  
 之后，任何以 "N" 开头的剩余 flag 都被当成 "next"。从 httpd 抄来的 [NS]  
（nosubreq）和 [NV]  
（novary）被简化成 [N]  
。配合 substitution 用 -  
 留 null、i = -1  
 没有 round cap，一条未授权 GET 请求就能把 worker 线程锁死到客户端断开或进程重启。  
  
PoC 形态（abraxas 在 127.0.0.1:18910 一键 docker compose 复现，rewrite.config  
 五条规则）：  
```
RewriteRule ^/loop$ - [NS]   # 4.002s hangRewriteRule ^/next$  - [N]   # 4.003s hangRewriteRule ^/skip$  - [S=-1]  # 4.002s hangRewriteRule ^/ok$    - [L]   # 13ms 404RewriteRule ^/safe$  - [L,NS]  # 5ms 404（安全）
```  
  
后三条是同一条 sink 的不同入口，单独一条就够打死一个 worker，发 maxThreads 条（默认 200）就把整个 connector 池填满。其他 sibling 请求还能响应，直到线程耗尽。  
## 二、技术原理——为什么 [NS] 会被错认成 [N]  
  
漏洞的核心出在 RewriteValve.parseRuleFlag  
。Apache httpd 的 mod_rewrite 里有几个看起来 "N" 开头的 flag：N  
（next，下一条规则）、NS  
（nosubreq，不要处理子请求）、NV  
（novary，不要传给子请求的查询串）。Tomcat 想抄 httpd 的语义，但对 NC  
/NE  
 处理完之后，对剩余 flag 用的是 startsWith("N")  
 这条前缀判断：  
```
// RewriteValve.parseRuleFlag 简化伪代码（abraxas 提取自 11.0.27）if (flag.startsWith("NC") || flag.startsWith("NE")) {    // NC/NE 处理完，前缀消化到 'N'，接下来的 startsWith 判断会命中}if (flag.startsWith("N")) {    newtest = rule;          // 关键：保留当前 rule 作为下一轮的测试    nextFlag = true;          // isNext() 返回 true}
```  
  
Tomcat 本意是支持 [N]  
，但错误把 [NS]  
/[NV]  
 也囊括进来了——这是 flag 解析逻辑上的 classic 误用：把"枚举集合"当成"前缀树"处理。  
  
然后在 RewriteRule.evaluate  
 里，substitution 是 -  
 时按 mod_rewrite 规范"不替换"，但代码让 substitution  
 字段保持 null。evaluate  
 拿 null substitution 走特殊路径——**返回原始 URL**  
。i = -1  
 被 isNext()  
 设回去时，for 循环下一次增量到 0——又从当前 rule 开始。结合 newtest  
 是 rule 自己的浅拷贝，只要这条规则对自己成立（即被请求的 URL 匹配），rewrite 就自己喂自己，永远不退出。  
  
这里没有 round cap。Tomcat 11.0 的循环退出条件本来靠 [L]  
（last）或 round 计数，但 i = -1  
 直接绕过两者。截至 11.0.27 都没有保护。  
  
CVE-2026-65927（11.0.25 修复）已经把 [N]  
 重启点的 bug 改对了——但**它没加 round cap**  
。[NS]  
/[NV]  
 这条新路径仍然能进入同一个无限循环，只是入口藏在了"看起来无害"的 flag 里。  
## 三、攻击路径四种姿势  
  
abraxas 在本地 loopback 复现里给出四条触发路径，对应同一个 sink：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">入口</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">规则</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">攻击姿势</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">延迟</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">/loop</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">[NS]</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">抄自 httpd 的&#34;不要处理子请求&#34;修饰符，最像正常配置</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">4.002s hang</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">/next</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">[N]</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">显式 next 循环，旧 CVE 没修干净的死角</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">4.003s hang</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">/skip</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">[S=-1]</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">skip 写成负数，i 直接退回 -1</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">4.002s hang</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">/safe</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">[L,NS]</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">加 L 让 isLast() 先 break，</span><strong><span leaf="">安全</span></strong></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5ms 404</span></section></td></tr></tbody></table>  
所有路径都需要 RewriteValve 在 Host 或 Context 上启用，并且 Host 里有匹配 URL 的 RewriteRule。这正是默认配置：Tomcat 11 的 Host 默认会加载 conf/Catalina/localhost/rewrite.config  
，任何匹配规则都会触发。  
  
最危险的是：只用一条 HTTP 请求 + 一个常见 flag（[NS]  
）。零认证、零交互、零 payload。**这种 flag 在生产环境很可能被写进去**  
——httpd 文档里 NS 是合法修饰符，写 Tomcat 的人很容易无脑照搬。  
  
到了对接的 chain 阶段，攻击者只需要对着一台对外开放 Tomcat 11.0.x 的服务器，反复发 maxThreads 个匹配请求（默认 200），整个 connector 池就会被填满——其他用户的请求全部被 queue 排队或直接拒绝。要恢复就只能重启 JVM。  
## 四、当前状态——还没发 CVE 公告  
  
abraxas 报告时（2026-10-14），情况是这样的：  
- **没有 CVE**  
：abraxas 表示 "no CVE yet"，CVE 分配机构还没给号  
  
- **没有 security advisory**  
：11.0.27 是当前的稳定 release，但 Apache Tomcat 团队没在 security-11.html  
 里加条目  
  
- **main 分支已经悄悄修了一部分**  
：11.0.27 之后不久，main 分支加了 MAX_REWRITE_ROUNDS=1000  
（commit 2da9fddb  
，循环上限 1000 次后强制退出）和"精确 flag 匹配"（commit 8992dbdd  
，改成 equals  
 而不是 startsWith  
），同时拒绝 skip < 0  
  
- **但没出新版 release**  
：main 分支的修复没有打包成 release tag，所以生产上跑 11.0.27 的服务器完全暴露  
  
- **相邻 CVE 提醒**  
：CVE-2026-78383（AJP 线程钉死同类）、CVE-2026-77791（WebSocket close 线程钉死同类），都是同一类 Important 级别漏洞——Tomcat 最近这堆 CVE 都聚在"线程钉死"上，攻击面比想象中大  
  
- **作者身份**  
：abraxas 在 abraxaslabs.tech 注册了研究主页，GitHub 上 @abraxas 账号活跃，用 AGPL v3.0 发布 PoC（强 copyleft，意味着任何拿这段代码做产品的人都得开源整个产品）  
  
## 五、临时缓解——三条路  
  
在官方 release 出来之前，可以选其中一条：  
  
**1. 升级到 main 分支**  
  
直接拉 main 分支编译，commit 2da9fddb  
 + 8992dbdd  
 已经修。但这是测试代码、生产稳定性自负。  
  
**2. 关闭 RewriteValve**  
  
从 conf/server.xml  
 或 Host 配置里移除 RewriteValve，rewrite.config 不被加载，自然不可能触发。这是损失最小、风险最低的临时方案——如果你没用到 URL 重写的话。  
  
**3. 审计 rewrite.config**  
  
把所有 [NS]  
 / [NV]  
 / 显式 [N]  
 改成 [L,NS]  
（加 L 让 isLast() 先 break），并禁止 skip < 0  
。验证完成后所有 URL 必须 5ms 内 404 才算安全——abraxas 的 Lab 里 /safe  
 就是这个标准。  
## 六、附录——原文链接与 PoC 仓库  
  
按用户要求提供下载（即公开仓库地址）：  
  
**Write-up（技术分析原文）：**  
- https://abraxaslabs.tech/research/tomcat-rewrite-ns-loop  
  
**PoC 仓库（含 lab 一键复现）：**  
- https://github.com/abraxas/tomcat-rewrite-ns-loop  
  
复现方法（仓库 README）：  
```
git clone https://github.com/abraxas/tomcat-rewrite-ns-loopcd tomcat-rewrite-ns-loop/lab./run.sh
```  
  
loopback 监听 127.0.0.1:18910  
，对应容器内 8080  
。Lab 镜像 tag tomcat-rewrite-ns-loop:11.0.27  
，基于官方 apache-tomcat-11.0.27.tar.gz  
（SHA512 已记录）。  
  
**素材出处**  
：  
- 推文：abraxas @abraxas_null - https://x.com/abraxas_null/status/2109790728826310710  
  
- PoC：https://github.com/abraxas/tomcat-rewrite-ns-loop（GNU AGPL v3.0）  
  
- Write-up：https://abraxaslabs.tech/research/tomcat-rewrite-ns-loop  
  
- 推文配图（已下载到本地 images/）：HUd7EdnaQAAwyxJ  
 1408x1408  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6NTiamSDJBEsPxhNlMiaKNUkyjn74JhSicbarr5KZxhwtBIUbWuDYSU9bmUEOs6dqjr1vYiaShh4V4JusLGDicj0gat6entyXb3HDlM/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
