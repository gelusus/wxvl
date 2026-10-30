#  破防！PHP2Shell 漏洞首现实战利用，无认证 RCE 拿下服务器权限  
 星盟安全团队   2026-10-30 13:21  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cs6H03FTrIOBUibK3o96dcuiaRD01IXW3mx4ULXqzzZBhM0TApiam5O7egYm5Whib1HxxFp8xryngZJ56VoXTCenndicjgkkS6kn7HicydRoBRQ74/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cs6H03FTrIP3O1CuofqKEPXd4JWjB5ZcKbdWrYsY3giaCjkZblwiawB4j73z6ZVg06eRcWb7iaxz0Q5eicqzNcicjpZe4rg3YFkINriaDfsYiare5M/640?wx_fmt=gif "")  
  
  
**团队招新**  
  
**招新要求**  
  
**·**  
热爱网络安全，喜欢 CTF；  
  
**·**  
拥有 CTF 比赛经验，有较好比赛成绩的；**·**  
乐于奉献、热爱分享，愿意提升自己同时帮助他人；  
  
**·**  
时间允许参加各类赛事，服从战队管理与安排；  
  
**·**  
各类比赛获奖者、能力出众者视情况考量；  
  
**·**  
未参与其他高校联队；  
  
**·**  
 大一同学视情况放宽资历要求。  
  
  
**联系方式**  
  
**·**  
请将个人简历发送至以下邮箱：  
  
简历邮箱：2638726415@qq.com  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOsngh41z1kcdqMmOA3ASLogqzUaFp4SibUkwmN3nSTFaYUznIHP3mLXy1ibsmqcWpcx4ibBGOzxzfAQS7TjG3avj9vwKqp2BDwq4M/640?wx_fmt=png "")  
  
php2shell 实验室复现  
  
  
**php2shell 复现：一个不需要登录的 PHP 8.x RCE**  
  
CVE-2026-109618 / CVE-2026-109619 ｜ 授权靶场全链复现  
  
作者：sqli 安全研究团队 ｜ 2026 年 10 月  
  
  
  
**写在前面**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
如果一个 PHP 页面只是接受用户输入、调用了一次 unserialize()，而服务器又恰好是经典的 Apache prefork + mod_php 部署，攻击者能不能不做登录、不依赖业务上传功能，只靠一串 HTTP 请求就拿到命令执行？  
  
2026 年 10 月披露的 php2shell（CVE-2026-109618 / CVE-2026-109619）给出的答案是：在满足条件时，可以。  
  
我们在隔离靶场完成了两条链的复现：一条需要应用存在可控的 unserialize() Sink，另一条甚至不需要 Sink，只依赖 session.upload_progress 与特定的 PHP 配置。本文记录复现过程、关键证据，以及我们认为更重要的检测与修复建议。  
  
**先说结论：**  
  
****  
 · 受影响版本：PHP 8.x 未修复（官方在 8.4.27 修复；复现时 8.5 分支的修复版本仍需以官方后续公告为准）。  
  
 · 关键部署条件：Apache prefork + mod_php（NTS）；FPM/CGI 不在已复现链路内。  
  
 · 应用条件：存在 unserialize() 输入，且递归反序列化共享 var_hash 的 Serializable 类。  
  
 · 实验室结果：通用链首次全链 1748 请求 / 1503 次崩溃，约 3–6 分钟拿到 www-data；缓存复用 8 请求 / 2 次崩溃。  
  
 · 另一条 OOM 链：目标页只有 22 字节、没有 session_start()，在特定配置下同样拿到了 uid=33(www-data)。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOsGR4DBPiaMH6HaRcDJ78w0nuGU2YRY8U9j6gRwdDQljwPOCPUkibcjk8AdveHCJnyOKKTt9qOLDgiaRBIvFUSdepawLj1jcMIA7E/640?wx_fmt=png "")  
  
图 1：php2shell 的三个前置条件  
  
**一**  
  
**影响范围：三个条件缺一不可**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
php2shell 不是“所有 PHP 8.x 都能直接打”的漏洞。把公开情报和我们的复现结果放在一起，至少要同时看三个条件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOvPp7HNG6lHruOFLaNNnct6Csm0NJvkFx4qIzFiaicKibZ5UnJOG7UeHl5vCUA7damdPTDdhK6Tyd5RiaWWvIibXFQ8bic69OdOonIoA/640?wx_fmt=png "")  
  
第一条决定了“有没有漏洞”，第二条决定了“能不能按已复现的方式利用”，第三条决定了“远程入口在哪里”。少一条，风险判断就应该降级。  
  
也要避免另一个极端：看到 nginx 或 FPM 就认为完全没事。版本是否修复、是否还有别的 SAPI、是否存在其他内存破坏原语，都需要单独确认；WAF 只能改变利用成本，不能替代版本修复。  
  
  
**二**  
  
**根因：同一张 var_hash，被内外两层同时改写**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
PHPunserialize() var_hash（varHashTable） 跟踪反序列化过程中的对象与引用关系。问题出在“递归”：  
  
**·**  
  外层 unserialize() 正在使用一张 var_hash；  
  
**·**  
 某个实现了 Serializable 的类，在自己的 unserialize() 方法里又调用了一次全局 unserialize()；  
  
**·**  
  内外两层共享同一个 var_hash；  
  
**·**  
当内层属性写入达到第 9 次时，HashTable 从 8 槽扩容到 16 槽，旧的 arData（288 字节）被 efree() 释放；  
  
外层 var_hash 里仍然保存着指向旧 arData 的属性 zval——这就是悬挂引用（UAF）。  
  
一个最小化的 gadget 形态如下（仅用于说明触发路径，不是完整利用代码）：  
  
class CachedData implements Serializable  
  
{  
  
    public function unserialize(string $data): void  
  
    {  
  
        // 内层递归反序列化，与外层共享 var_hash  
  
        unserialize($data)->x = 0; // 第 9 次属性写入触发 8→16 扩容  
  
    }  
  
}  
  
  
释放出来的 288 字节块会落入 ZendMM 的 bin-320。攻击者用 32 条 280 字节的喷洒字符串稳定回收它，悬挂 zval 的 bucket 就被攻击者控制；把伪造成 IS_STRING 的 zval 指向任意地址，靶页再调用 serialize() 时就会把该地址的内容回显出来——这就是任意读的起点。  
  
拿到任意读之后，利用链继续完成信息定位、伪造对象和控制流劫持。官方修复描述是 “Avoid state corruption via bailout”，与我们观察到的 fatal/bailout 路径跳过 var_hash 清理的行为一致。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DC4TgvRKhOvED9jmib6AMhxHMHTzm9gGXw5wmyCiaQuNjx3qG6nf5hvib1U7yyUB79TBPiaWFh9mHiaJueofpFjUxvnZf7WiazqcXgwDTrf5RBPGs/640?wx_fmt=png "")  
  
图 2：var_hash 共享导致 arData UAF 的关键路径  
  
**三**  
  
**复现环境**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
复现全部在隔离的 Docker 靶场完成，资产为自有授权环境。主目标使用 PHP 8.5.11 NTS + Apache 2.4.69，SAPI 为 apache2handler，MPM 为 prefork，加载 php_module（mod_php）。  
  
目标页是标准的 Sink 页 remote_app.php：接收 POST 参数 cook，定义 CachedData implements Serializable，并在递归 unserialize() 后做一次属性写入。为了让实验可核对，靶页主动返回了 X-PHP-Version 与 X-PHP-SAPI 响应头；真实业务不会这样写，但旧页面、phpinfo、错误页或日志可能泄露同类信息。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOsNDodo7aPxVgqGcv0QZRp69YzKP17oLiceaFicPwOWhG6wzyicrnz0Oicu9K0h12Qv0aZNxTQoEibkODmDoibVP8v1dsS1GegIVnHsg/640?wx_fmt=png "")  
  
图 3：复现环境与靶页响应头  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DC4TgvRKhOunOAJI6qnd0JBFuicrgBRHibTEGKgCzvcgN7ag5phJDmLDUveT868DqFeyQIZPMX6lCZtZicj5jVRY1heQR7oh7oGYLjBKgCibrG4/640?wx_fmt=png "")  
  
  
**四**  
  
**利用链：从泄漏到 system()**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
通用链可以拆成 P1–P6 六个阶段。这里只保留思路，不复述可直接武器化的完整载荷。  
  
1. P1 堆泄漏：利用 R:N 反引用，把真实堆指针写进被回收的 bucket，再通过回显读出来。  
  
2. P2 模块扫描：双向扫描 2MB 地址空间，按 ELF 头与特征早停，定位 libphp 映射。  
  
3. P3 libphp / EG：解析 .gnu_hash，定位 executor_globals 与 .dynamic。  
  
4. P4 libc / system：以 GOT 为锚点拉平外部指针，定位 libc 并解析 system()。  
  
5. P5 stdClass 类入口：遍历 EG 类表，用 DJBX33A 哈希找到 stdClass 的真实 ce。  
  
6. P6 喷洒槽位：定位可控的喷洒槽位，伪造 stdClass 对象与 handlers 伪表，把 get_properties_for 槽指向 system()，完成控制流劫持。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DC4TgvRKhOu7Wyfo4IxGicPW6icmxZLcy1jicC57oS8IziaU6oB0C1QKpq6BSopD7r9OoQq0eickRyibrLR71gBmssPghGicJCUqtAAo8vXbiaqxsnw/640?wx_fmt=png "")  
  
图 4：通用利用链 P1–P6  
  
在缓存命中、地址部分失效的情况下，工具会自动补算 heap/ce/spray，然后执行命令并经 HTTP 回读输出。下面这次运行只用了 8 个请求、2 次崩溃：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOtzmdib4Z6IL3qoczajvSTgpgFUrt2XvicT2M1jIiaLOPqd5rE5IlQK1IDWxibS2sdWb0Jkc8TD9sw5QRf00a1JiafNAkJ4Er6KKBkY/640?wx_fmt=png "")  
  
图 5：缓存复用下的命令执行回显  
  
**需要强调两点：**  
  
·输出里的地址每次实验都会变化，不能照抄；  
  
·崩溃不是“打失败了”。探测性坏读会杀掉一个 prefork worker，Apache 自动重 fork；这是原语的固有代价。首次全链 1748 请求、1503 次崩溃，主要成本就来自这里。  
  
**五**  
  
**影响证明与崩溃代价**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
拿到命令执行后，我们部署了一个最小的验证 webshell，并回读 id。这不是“利用的终点”，只是影响证明：mod_php 子进程以 www-data 运行，说明代码执行身份就是 Web 服务身份。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DC4TgvRKhOtZbrzke6ybKvZ29ia2HMFfh9I7rYKq41pRkicTEAefyQNMHg96T1jV7ooq9YmaL0ibtB7knLq1t1AzYiaHP5ed60ReWGGgYNLdBYY/640?wx_fmt=png "")  
  
图 6：webshell 回显与 Apache 段错误日志  
  
同一张图的下半部分是 Apache 的 error.log。child pid ... exit signal Segmentation fault (11) 短时高频出现，是这条链在探测阶段的典型特征，也是防守方最有价值的检测信号之一。  
  
也正因为会造成大量 worker 崩溃和 core dump，生产环境不要直接复现：它可能造成业务抖动、连接中断，甚至把磁盘打满。实验室复现和生产验证是两件事。  
  
**六**  
  
**另一条链：连 Sink 都不需要**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
通用链要求应用存在可控的 unserialize() 入口。我们进一步验证了另一条 OOM 链：目标页只有 22 字节，没有 session_start()，没有任何业务逻辑，也没有 Sink。  
  
<?php echo "OK\n"; ?>  
  
  
利用流量全部是普通的 multipart/form-data POST。session.upload_progress 在解析 multipart 时触发 OOM，longjmp/bailout 留下 stale session 指针；随后通过 Set-Cookie 泄漏 Zend MM chunk 基址，再泄漏 libphp 基址，最后伪造 HashTable / execute_data，把 pDestructor 指向 zif_system。  
  
实验室在 PHP 8.4.26（8M，单 worker）与 PHP 8.5.11（8M / 128M）上均拿到了 proof；下图是 8.4.26 容器的 proof 文件回读：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DC4TgvRKhOuIFL1u9ibicDdC9BTlMDPKnPkIaiaH0Ric79adSPa2g1zuN9HbWVr49cZSFRmTovcxgNzw9ia9qjWff2pfyeXQSiaVja0kFA1MXNSf8/640?wx_fmt=png "")  
  
图 7：无 Sink 的 OOM 链与 proof  
  
这条链的“门槛”从“应用必须有 Sink”降到了“特定 PHP 配置”，但前提更苛刻：单 worker / KeepAlive Off、display_errors=On、session.upload_progress 开启、session.save_path 可写、session.use_strict_mode=0 等。它会 OOM 和 Abort worker，属于破坏性验证；生产多 worker 集群并不一定满足同一 worker 的布局假设。  
  
**七**  
  
**检测与修复：WAF 不是修复**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
如果只记一件事：先把版本和 SAPI 清点清楚，再谈其他。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DC4TgvRKhOslVJyibJtZiaAibEX4Wlt1tJyA8G1cy7cWOyoZhmcqbh0xBUsg3NLQ4Jpj9NV6yx9oNIuReNJ7dU4EloDwZGL0doia84Axbqv8Xg8/640?wx_fmt=png "")  
  
图 8：修复与检测优先级  
  
  
1. 升级  
  
升级到 PHP 8.4.27 或更高版本；8.5 分支用户关注 GHSA-w8hw-w9fx-j79w / GHSA-6wmf-4rv5-p3rp 的后续合入版本。不要只看“PHP 8.x”这个大版本，补丁级别才是关键。  
  
2. 换 SAPI  
  
用 php-fpm 替代 mod_php，避免 prefork + mod_php 的组合。已复现链路依赖同进程堆与 libphp 映射，FPM/CGI 不满足这个前提；这是架构层的降险手段，但不能替代升级。  
  
3. 收窄应用  
  
不对不可信输入调用 unserialize()，优先使用 JSON 等数据格式；审计代码中 Serializable::unserialize() 内部递归调用全局 unserialize() 的模式。这类 gadget 在真实业务里并不罕见，尤其是缓存、会话、消息队列等“看起来只是反序列化一下”的代码。  
  
4. 补检测  
  
·POST 体中出现 C:10:"CachedData" 类自定义序列化头，或超长嵌套 a:N:{} 载荷；  
  
·Apache error.log 中 child pid ... exit signal Segmentation fault 短时高频出现；  
  
·5xx、worker 重启、core dump 数量异常；  
  
·对 session.upload_progress 相关 multipart 异常流量做基线告警。  
  
WAF 可以挡住已知 payload，但挡不住协议层变形和新的 gadget；它只能买时间，不能替代修复。运维侧还可以设置有限的 MaxRequestsPerChild、限制 core dump、确保 webroot 不可写——这些措施会抬高武器化成本，但同样不是修复。  
  
**八**  
  
**写在最后**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/wvE4hmHqGmBNIwjLlaROSjwL4zphs9p5BMpYEeKPpaGhgdt8JwjicNccSicaeJdicZpc0vmGxz2mL0YIP5JPLJyRr3xyxQNV8UCKIQRiaozZR3s/640?wx_fmt=gif "")  
  
  
  
php2shell 的复现再次说明：内存破坏漏洞的“触发条件”和“利用条件”要分开看。一个 22 字节的 <?php echo "OK\n"; ?> 页面，在特定配置下也可能成为入口；而一个看似复杂的 Sink，在正确的部署形态下只需要几个请求。  
  
本文只记录授权靶场内的研究性复现，不提供完整利用代码，也不建议对生产环境做破坏性验证。截图均来自实验室真实输出，已隐藏公网地址与与本次研究无关的资产信息。  
  
如果要在自己的资产里做排查，建议按这个顺序：  
  
1. 清点 PHP 版本与 SAPI（php -i 的 Server API、Thread Safety，apachectl -M 的 MPM/mod_php）；  
  
2. 找出所有 unserialize() 入口与 Serializable 递归模式；  
  
3. 能升级先升级，不能升级先隔离/降权/加检测；  
  
4. 最后才是在隔离环境里做受控验证。  
  
漏洞利用的终点不是 shell，而是修复和检测。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/DC4TgvRKhOt0fszKzSnbADL9wnHfTUytx6q9t05wuwuF85NrUW4jngicAuenjuJzpJFpw5q4tytVVjJGE1a4Wdm6IibLZ8jrNYSzFGbloo8Xo/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DC4TgvRKhOu1OXIF38mRjfGQbCLuR5x80z03iaAxsfYLE3vmuqjW3ico0Lkic0lJ4JP8aonjy5atTqgG2wwWoOEEn9SUhTulfsX0iaBFldVAE3Q/640?wx_fmt=jpeg "")  
  
  
**-END-**  
  
  
  
  
  
