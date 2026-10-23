#  请求还没开始，代码已经跑完了-php2shell-复现笔记  
原创 niuko
                    niuko  Ncko   2026-10-23 07:35  
  
# 请求还没开始，代码已经跑完了  
  
那天晚上我在反复发一个 332 字节的包。  
  
靶场是我自己搭的，两个容器，一个 PHP 8.4.26 跑 FPM，一个 8.4.27，中间只差一个小版本。我发的是同一条 multipart 请求，字段名 PHP_SESSION_UPLOAD_PROGRESS  
，值 pwn619  
，外加一个 Cookie: PHPSESSID=cve619  
。目标页面的第一行就写死了鉴权，没登录直接 403。  
  
服务器回了我 152 个字节：  
```
DENIED: 鉴权在应用层第一行 
```  
  
干干净净的拒绝。我又去翻日志，同一秒，同一个 pid，多了一行：  
```
[04:34:40] pid=3 uid=www-data cmd={id} out={uid=33(www-data) gid=33(www-data) groups=33(www-data)} 
```  
  
页面告诉我"你没权限"，日志告诉我 id  
 已经跑完了。  
  
两个说法都是真的。因为跑命令的那段代码，在鉴权之前，甚至在"这个请求到底属于哪个脚本"都还没确定之前，就已经结束了。我给这篇文章起了这个标题，就是因为我是先看到日志、才想明白这句话的。  
## 一、一开始我盯错了地方  
  
起点是微信上转来的一篇文章，说 PHP 连爆了两个洞，编号 109618 和 109619，同一个安全公告下面，评分 9.2，都写着 RCE。  
  
我第一反应是照着公开的 PoC 打。仓库里给了测试脚本、php.ini、Nginx 配置，看着挺齐。我把两个版本分别跑起来，一条一条照着投，结果全绿——我的意思是全灭。  
  
109619 那边，脚本跑完说"未命中"。109618 那边更干脆，PHP 回我一句规规矩矩的 OOM fatal，进程不掉，没有 signal 11，没有 core dump，接着发请求照样正常。  
  
按说这时候该收工了，写一句"环境不符"也行。但有个细节让我不舒服：那个仓库的判定脚本把 502 算崩溃，把 000 也算崩溃。000 就是 curl 连不上、超时、一个字节都没收到。这东西也能当证据，那这套测试其实什么都没证明。  
  
我决定不看它的脚本了，自己找差异。  
## 二、先找"能看见的东西"  
  
我的习惯有点笨：在写 payload 之前，先找一个不依赖 payload 的观测点。也就是说，同一份数据、同一个包，打在修复版和没修复版上，必须有一个肉眼能数出来的不同。  
  
没有这个，后面所有"我觉得应该能打"都是猜。  
  
我先做了一件很土的事——写一个页面，只干一件事：把会话里那个对象在被反序列化时记一笔。  
```
classWakeupLogger{     public$cmd = 'id';     publicfunction__wakeup() {         file_put_contents('/tmp/wakeup.log',             '[' . date('H:i:s') . '] pid=' . getmypid()             . ' uid=' . posix_getpwuid(posix_geteuid())['name']             . ' class=WakeupLogger' . "\n", FILE_APPEND);     } } 
```  
  
然后三个页面：一个调 session_start()  
 的正常页，一个**完全不碰 session 任何函数**  
的页面（只有一行 echo 'ok';  
），还有一个第一行就鉴权、不通过直接 die  
 的页面。  
  
我盯的是第二和第三个。如果 __wakeup  
 在它们上面跑了，那就说明反序列化发生在应用代码之前——应用连自己有没有权限都还没判断。  
  
结果出来的时候我愣了一下：  
<table><thead><tr><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">目标页面</span></section></th><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">8.4.4</span></section></th><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">8.4.26</span></section></th><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">8.4.27</span></section></th></tr></thead><tbody><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">第一行鉴权的 403 页</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><code style="color:#e96900 !important;background:#f3f3f3 !important;padding:2px 6px !important;border-radius:4px !important;font-size:0.88em !important;font-family:&#34;SFMono-Regular&#34;,Consolas,Menlo,monospace !important;"><span leaf="">uid=33(www-data)</span></code><section><span leaf=""> ×2</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><code style="color:#e96900 !important;background:#f3f3f3 !important;padding:2px 6px !important;border-radius:4px !important;font-size:0.88em !important;font-family:&#34;SFMono-Regular&#34;,Consolas,Menlo,monospace !important;"><span leaf="">uid=33(www-data)</span></code><section><span leaf=""> ×2</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">空</span></section></td></tr><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">完全没有 session 代码的页</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">×2</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">×2</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">0</span></section></td></tr><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">应用自己调 </span><code style="color:#e96900 !important;background:#f3f3f3 !important;padding:2px 6px !important;border-radius:4px !important;font-size:0.88em !important;font-family:&#34;SFMono-Regular&#34;,Consolas,Menlo,monospace !important;"><span leaf="">session_start()</span></code></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">3</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">3</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><strong style="color:#ef7060 !important;font-weight:700 !important;"><span leaf="">1</span></strong></td></tr></tbody></table>  
第三行那个 1 得说清楚，它不是漏洞。修复版里应用自己调 session_start()  
 仍然会正常反序列化一次，这是它本职工作。前两行才是问题：一个连 session 函数都没写的页面，两次 __wakeup  
 是说不通的。  
  
差出来的那两次，就是我一直在找的"能看见的东西"。  
## 三、那段时间里到底谁在读 POST  
  
这里得把 PHP 一个请求的生命周期摊开看。平时我们说"PHP 处理一个请求"，顺序大概是这样：  
```
php_request_startup()   ← 超全局、会话、模块全部激活 RINIT                   ← 各模块的请求初始化 编译并执行脚本           ← 业务代码在这里 RSHUTDOWN 
```  
  
而 php_request_startup()  
 里面还藏着一个东西：如果 session.upload_progress.enabled  
 是开的（它默认就是开的），PHP 会挂一个 rfc1867 上传进度钩子，专门处理带 PHP_SESSION_UPLOAD_PROGRESS  
 字段的上传请求。  
  
这个钩子要做的事之一，是把**上一次请求写进会话的数据解码出来**  
，好在进度信息里带上会话上下文。解码 = 反序列化。  
  
于是就有了这么一段：从读 POST，到 RINIT，中间这段代码在跑，可 session 的全局状态还是"上一次请求的"。这段窗口里没有路由、没有中间件、没有鉴权、没有业务代码，只有一处老老实实调用的 php_unserialize()  
。  
  
旧版本对这段窗口管得很松：解出来的对象该构造就构造，__wakeup  
 该跑就跑，类不存在还顺手调一下 unserialize_callback_func  
。  
  
修复提交（506e87fa7d  
，标题写着 "Avoid state corruption via bailout"）改的正是这里。它在 rfc1867 更新期改成只解出"残缺类"——不 autoload，不跑 __wakeup  
，不调回调；同时在启动前把 PG(http_globals)  
、PG(modules_activated)  
 这些东西先复位成未定义，免得被上一次请求的残留值带偏。  
  
看懂这两点之后，前面那张表就完全解释得通了。  
## 四、第一条链：一个对象，一个预先加载的类  
  
我这条链能成立，靠两样东西。  
  
第一样，会话里得有一个对象。这在真实业务里不是什么稀罕事——框架把用户对象、购物车、登录态往里丢，落盘的内容长这样：  
```
logger|O:12:"WakeupLogger":2:{s:3:"cmd";s:2:"id";s:4:"data";s:40:"...";} 
```  
  
竖线前面是 key，后面是序列化过的对象。  
  
第二样，这个类必须在 pre-RINIT 的时候就已经存在。因为那个窗口里没有 autoload，类不存在就没法实例化，__wakeup  
 也不会跑。  
  
这就是 opcache.preload  
 的位置。生产环境为了省掉每次请求的编译开销，很多人会预加载一批类，其中完全可能包含带 __wakeup  
 或 __unserialize  
 的业务类。预加载发生在进程启动时，早于任何请求，所以到了那个窗口，类是真存在的。  
  
我用 auto_prepend_file  
 试过替代方案，不行——它要等脚本编译阶段，赶不上，__wakeup  
 一次都没跑。这条路当时就断了，我把这次失败记在心里，因为后面判断"这条链到底有多通用"的时候它很关键。  
  
剩下就是发包。一条 multipart，332 字节，字段名必须是 PHP_SESSION_UPLOAD_PROGRESS  
（我一开始写成 PHP_SESSION_PROGRESS  
，怎么打都不触发，白耗了半小时），Cookie 和表单里都给同一个 SID。  
```
POST /guard.php HTTP/1.1 Host: 127.0.0.1:19026 Cookie: PHPSESSID=cve619 Content-Type: multipart/form-data; boundary=----wsc  ------wsc Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"  pwn619 ------wsc Content-Disposition: form-data; name="PHPSESSID"  cve619 ------wsc Content-Disposition: form-data; name="file"; filename="a.txt" Content-Type: text/plain  x ------wsc-- 
```  
  
打完之后，页面还是那句 DENIED  
，日志里 uid=33(www-data)  
 出来了，两次。  
  
两次是有讲究的：一次发生在 pre-RINIT 的进度钩子里，另一次是应用自己 session_start()  
 的时候顺手又解了一遍。修复版只剩后一次，所以它显示 1；而那个不碰 session 的页面，修复版是 0。  
  
一进一出，链的边界就很清楚了。  
## 五、第二条链：把类名当命令  
  
第一条链有个前提——得有个类被预加载。我又试了另一种玩法，它连预加载都不需要，只要一行 ini：  
```
unserialize_callback_func = system 
```  
  
这个配置本来是给老代码做兼容的：反序列化时遇到不存在的类，就把类名交给这个函数处理。函数名叫 system  
 的时候，事情就变得很直接——它会把你写的类名当命令执行。  
  
所以会话内容写成这样：  
```
x|O:2:"id":0:{} 
```  
  
id  
 是类名。类不存在，回调被叫起来，system("id")  
。  
  
8.4.26 上实测，包是 345 字节，uid=33(www-data)  
 出来了两次，PHP 还老老实实打了一条告警：  
```
Warning: PHP Request Startup: Function system() hasn't defined the class it was called for 
```  
  
PHP Request Startup  
 这个前缀是 PHP 自己给的，它等于承认了执行位置在请求启动阶段。同一份内容打 8.4.27，响应只剩 11 个字节的 nostart-ok  
。  
  
这条链的短板也明显：命令只能是"一个合法的类名"，带不了参数。id  
、whoami  
、hostname  
 这种单 token 能跑，cat /etc/passwd  
 就不行。而且是 ini 级别的前置条件，不是每个站点都有。它更适合当第一条链的放大器，而不是主力。  
  
顺带说个我踩过的坑，免得有人跟着走弯路：我曾经想"一条包全搞定"，也就是不依赖应用存对象，直接靠上传进度那个字段把 对象序列化内容  
 写进会话文件。试了一晚上，不行。值里带 |  
 落盘是 0 字节，用 filename 注入被锁在字符串上下文里出不来。**"单包免写原语"这个想法，我自己的实验把它证伪了**  
，所以文章里不会出现"一条包通杀"这种话。  
## 六、另一条 CVE，我没打穿  
  
109618 到现在我也没复现成功。这件事得单独写清楚，因为同一个安全公告下的两个编号，可信度其实不一样。  
  
一开始我照公开 PoC 逐字投：mod_php 三个版本、FPM 单 worker、带析构的 100 个对象环链、shutdown 里再分配、关闭期二次 OOM 的各种尺寸（4 到 16 MB）。结果全阴性——一行干净的 OOM fatal，worker 不掉，没有 signal 11，后续请求健康。  
  
后来我想明白自己错在哪了。补丁复位的是 PG(http_globals)  
、EG(delayed_oom)  
，这些是**请求启动阶段**  
的状态；而我一直把 OOM 打在脚本执行期，压根不在同一个窗口。  
  
于是我把客户端改成 FastCGI keep-alive，一个连接连发三包：正常 GET、撑爆内存的 POST、再一个正常 GET。先确认首尾两包是同一个 worker PID，再去找脏值。memory_limit  
 压到 8M，max_input_vars  
 提到十万，四种打法轮着上：  
<table><thead><tr><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">打法</span></section></th><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">body</span></section></th><th style="background:#fff7f5 !important;padding:9px 13px !important;font-weight:600 !important;border:1px solid #f3d9d4 !important;color:#c0392b !important;"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">单值 12MB</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">12 MB</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">解析期 OOM，worker 当场退出（</span><code style="color:#e96900 !important;background:#f3f3f3 !important;padding:2px 6px !important;border-radius:4px !important;font-size:0.88em !important;font-family:&#34;SFMono-Regular&#34;,Consolas,Menlo,monospace !important;"><span leaf="">exited with code 70</span></code><span leaf="">），连接 RST</span></section></td></tr><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">嵌套数组 3000 组</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">27 KB</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">被 </span><code style="color:#e96900 !important;background:#f3f3f3 !important;padding:2px 6px !important;border-radius:4px !important;font-size:0.88em !important;font-family:&#34;SFMono-Regular&#34;,Consolas,Menlo,monospace !important;"><span leaf="">max_input_vars</span></code><span leaf=""> 截断，没 OOM</span></section></td></tr><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">30 万个变量</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">2.8 MB</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">解析期 OOM，worker 退出</span></section></td></tr><tr><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">深嵌套 300 层、multipart 3000 字段</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">&lt; 30 KB</span></section></td><td style="padding:9px 13px !important;border:1px solid #f3d9d4 !important;"><section><span leaf="">没吃满，页面正常执行</span></section></td></tr></tbody></table>  
修复前后的 OOM 报文一模一样，连分配失败的位置都写着 Unknown on line 0  
——这至少印证了 OOM 确实落在脚本之前，但也就到此为止。  
  
问题卡在一个组合上：脚本执行期 OOM，worker 活着，可状态是干净的；启动/解析期 OOM，状态确实被打断了，可 worker 直接退出。**"进程还活着 + 状态是脏的"，我一次都没凑出来**  
，而 618 要的正是这一格。  
  
所以我不会写"两个 CVE 都被我打通了"。机制描述和补丁证据是可信的，但黑盒远程侧没打穿，就是不宣称可利用。  
## 七、防这一手，别踩那两个坑  
  
写到这里顺手说防守，因为我自己在靶场上验了两件事，其中一件推翻了流传很广的说法。  
  
**session.use_strict_mode 挡不住它。**  
 我把 8.4.26 的严格模式开到 1，然后发同一个包，__wakeup  
 还是两次。原因不复杂：严格模式管的是"这个会话 ID 我发过没有"，而攻击者用的往往正是应用自己发出来的那个 SID——你登录过、加过购物车，这个 ID 本来就合法。它只挡得住攻击者凭空捏一个从没存在过的 ID，而那种情况本来也没有会话数据可以解。  
  
**session.upload_progress.enabled = Off 是管用的。**  
 同一个靶场、同一个包：2 次变 0 次；改回 On，立刻又是 2 次。钩子不注册，窗口就不存在。麻烦的是它默认开着，很多人根本不知道自己有这么个开关。  
  
版本上，8.2.35  
、8.3.36  
、8.4.27  
、8.5.12  
 以下都在受影响区间，升级是唯一彻底的解法。  
  
如果还想自查，方法很土但有效：准备一个带 __wakeup  
 的类并预加载，让应用把它存进会话，然后对一个**没有 session 代码的页面**  
发那个 multipart 包，看 __wakeup  
 有没有跑。跑了就是没修。别拿调了 session_start()  
 的页面去测，那里修复版也会跑一次，很容易把正常行为当成漏洞。  
## 八、最后  
  
回头看，这篇文章里最值钱的部分可能不是那条链，而是找差异的那一步。  
  
我一开始拿着公开 PoC 打不出东西，最省事的做法是换个靶场、换个版本，或者干脆跟着宽松的判定脚本宣布"命中"。但那样写出来的东西，别人拿去打真实站点，是要出丑的。  
  
真正让我往前走的，是老老实实承认"我看不见差异"。然后去数 __wakeup  
 跑了几次——一个连指针都不用碰、纯 PHP 计数就能做的观测点。有了它，pre-RINIT 这个说法才不是概念，而是表里那个 2 和 0。  
  
至于 618，它现在还躺在我靶场的记录里，标注是"未打通"。我不打算给它涂一层好看的颜色。两个编号，一个通了，一个没通，这就是我这一周的实际进度。![微信图片_20261023152302_320_19](https://mmbiz.qpic.cn/mmbiz_png/5LNgXJargeUWI57ONznv1zLkNwNssMvmc96eWLhGtTVMiaO0k9TYN5v5oRvELeUBclJfWwUH4P5zBicujB8ic3UU56gq8DKjDrCib2hTkkia7ziaM/640?from=appmsg "")  
  
  
