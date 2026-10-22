#  讲解一下这次PHP核弹级别漏洞的原理  
 取证与溯源   2026-10-22 15:08  
  
这几天PHP圈很热闹，AI发现了一个核弹级别，隐藏了十几年的PHP的RCE漏洞，连日本首相都发X提醒大家注意。哈哈哈  
  
本来昨天就想写一个分析的，不过考虑到可能被攻击者利用，就等到Patch出来以后，今天再发吧，本文以已经公开的攻击脚本POC为例，来解释，具体链接我就不放了。  
  
我第一次看到这个漏洞的报告的时候，我就心想，这能是人干的事？也幸亏是AI，超强的综合能力，才能发现这个漏洞，并且利用。上午我还跟同同事说，AI一旦开始研究攻击，那就不仅仅是PHP了，所有的系统都可能会被攻破，毕竟之前的代码都是人写的，而人，一定会犯错。果然，下午的时候，消息就出来了NodeJS也核爆了。  
  
为什么说这个漏洞隐藏的深呢，因为这个漏洞的利用，综合了多个条件，并且需要对PHP的实现有较深的理解, 包括内存管理，Session的实现，反序列化的机制，OOM的处理机制。而要利用这个漏洞就需要更加丰富的系统知识，所以在AI之前，能综合具备这些知识的人，还是比较少的，而就算有这样的知识，也可能没有这样的精力。但，AI来了..  
  
本次我核心介绍危害最大的  
CVE-2026-109619，它能实现RCE，不过它和CVE-2026-109618共享一个前提，所以讲清楚一个，另外一个也就清楚了。  
  
大部分场景下，PHP都是用的多进程模式，早些年Apache流行的时候，是mod_php, php提供了一个模块，编译进Apache。后来Nginx流行了以后是php-fpm, PHP自己是一个多进程服务，通过sockets接受Nginx的请求转交。  
  
这个模式下，一个PHP进程会常驻，然后来一个请求，处理一个请求。为了保证请求之间不互相影响，PHP会在请求到来之前执行Request INIT(RINIT)，结束之后执行Request Shutdown(RSHUTDOWN)。  
  
PHP的内存是由PHP自己的内存管理器Zend MM管理的，每次请求结束后会重置这个状态。  
  
我们以PHP8.5为例(main/main.c):  
```
/* php_request_startup() */
zend_activate();           /* 重置 Zend 引擎核心状态 */
sapi_activate();
...
php_hash_environment();    /* 建立 $_POST / $_FILES —— 解析 HTTP body，
                              multipart 到达 rfc1867 回调（upload_progress 在此触发） */
zend_activate_modules();   /* 各扩展执行 RINIT，重置扩展状态 */
PG(modules_activated) = 1;
/* ... 执行应用脚本 ... */
/* php_request_shutdown()，按相反顺序拆除 */
zend_deactivate_modules(); /* 各扩展 RSHUTDOWN —— 受 PG(modules_activated) 门控 */
zend_deactivate();         /* Zend 引擎收尾：销毁符号表 */
shutdown_memory_manager(); /* 请求堆整体释放，还给 Zend MM（无条件执行） */
```  
  
一般情况下这个工作机制是没有任何问题的，但是永远都有二般：  
  
在出现致命的情况下，比如内存OOM，比如代码BUG导致内存写花，PHP会调用zend_bailout() ，而这个函数内部调用了C的longjmp，跳过了所有的清理代码，这样做也是对的，毕竟如果内存花了，那中间状态都是不可信的，如果做清理，那就可能会读到花的数据，导致不可预期的结果。  
  
而CVE-2026-109618就是利用这一点。  
  
在一般的情况下呢，zend_bailout之后，虽然留了一堆烂摊子，但是在下一个请求来临的时候，PHP的RINIT机制会恢复现场，初始化好所有的环境，所以不会有问题，但还是一样，永远都有二般，真还有那么一个时机，是在PHP RINIT之前被执行的，那就是:  
```
php_hash_environment()
```  
  
这里面会去构建PHP的超级变量，此时各个扩展的RINIT还没有被调用，而我们此次漏洞的关键的扩展，ext/session，就要提一提了。  
  
Session的RINIT(php_rinit_session_globals)中会初始化一些Session扩展使用的全局变量，但是在Session RINIT还没调用的时候，它还保留者上一个请求遗留的值。  
  
你可能会问，RINIT还没调用，Session怎么会跑到php_hash_environment()里去？  
  
答案是PHP的一个"贴心"功能：Session Upload Progress（session.upload_progress）。这个功能自PHP 5.4引入，默认开启，用途是让前端能在文件上传时显示进度条。它的实现思路是：在PHP解析multipart上传数据的过程中，实时把进度信息写进session文件。  
  
注意这个设计的必须之处：写进度条这种事，等脚本跑起来再写就晚了（上传那个时候就结束了），所以它必须在解析body的时候就写。于是PHP把一个回调挂在了multipart解析器上——第一个带文件的part出现时，这个回调会提前把session初始化了：读取PHPSESSID、  
  
打开session文件、把$_SESSION挂到符号表。而这一切，都发生在RINIT之前。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TanicctQBA09Bb81DHFnq9PnYpa2WopNJ3SRelveBueOLqszicXFk0NkSvJPVXrRnt4OfocLMhU0iamY1725D10NT8zMFHn4icDLR6lM957DtWY/640?wx_fmt=png&from=appmsg "")  
  
现在把拼图凑齐。想象同一个PHP worker进程上的两个请求：  
  
请求A（投毒）：  
  
攻击者发一个精心构造的multipart上传。文件part出现，upload progress回调提前初始化了session——此时Session扩展的全局变量里，PS(id)指向了一个请求堆上的zend_string，PS(http_session_vars)指向了一个请求堆上的$_SESSION数组。注意，这些指针指向的内存，是属于请求A的。  
  
紧接着，攻击者让请求A在解析中途OOM了（怎么做到的后面说）。zend_bailout()一跳，跳过了multipart的END回调——本来那里会做session的清理；又因为此刻`PG(modules_activated)`还是0，请求关闭流程里的Session RSHUTDOWN也被整段跳过。但是shutdown_memory_manager()是无条件执行的——请求A的堆，连同里面那个session id字符串和$_SESSION数组，整个被释放了。  
  
注意此时，PHP worker本身并没有死，但它的全局状态里留下了  
两个悬空指针：指针还在，指着的那块内存已经还给了Zend MM，等着被下一次分配领走。  
  
请求B（引爆）：  
  
攻击者再发一个multipart请求。这个请求不带文件，只带两个普通的表单字段：  
  
PHPSESSID=攻击者选定的id  
  
和  
  
PHP_SESSION_UPLOAD_PROGRESS=任意值。  
  
因为没有文件part，upload progress的初始化路径不会触发，请求B会直接走到multipart的END回调。而END回调里有一个清理函数，它会调用php_session_initialize()——于是PHP顺着那个从未被重置的悬空指针，去读一块已经被释放、并且刚刚被请求B自己的数据重新占据的内存。  
  
这里有个关键认知：请求B的字段名和字段值，每一个都会让PHP emalloc一次，分配的大小由字段长度决定。而PHP为了内存分配的效率，有一个机制是同样大小的内存，基本上会在一个小的list里分配，所以攻击者可以精确设计这些长度，让请求B的某次分配，恰好领回>请求A留下的那块悬空内存。领回来之后，这块内存里装的是什么？是攻击者在multipart里填的字节。  
  
也就是说，PHP拿着"这是一个合法的session结构"的旧记忆，去解读一块内容完全由攻击者控制的内存。这就是经典的UAF(use-after-free)。  
  
那怎么把一个UAF变成RCE？  
  
公开的PoC中给出了完整的答案，整个攻击分四个阶段，全程纯HTTP：  
  
第一步，把OOM变成可控的打断原语。要触发上面说的机制，攻击者得能在session已初始化、清理还没执行的窗口里制造fatal。三个放大器让这件事在默认配置下轻松做到：没有Content-Length的chunked multipart可以绕过post_max_size的预检查；单个part可以塞无限多的header，内存里的表示比线上体积大得多；filename=""的空文件控件不计入max_file_uploads，但六个$_FILES元数据数组照建。于是几MB的请求体，能打爆128MB的memory_limit——OOM发生在攻击者选定的精确位置。  
  
第二步，泄露。悬空的PS(id)被引向一块被回收重填的内存，攻击者在那里伪造了一个zend_string的长度字段。PHP生成Set-Cookie头的时候，把这个"长度"的字节原样吐了出来——响应头里带了4KB的堆内容。攻击端扫一遍，就拿到了堆基址；换个姿势再泄一次，这次泄露的是某个HashTable里的pDestructor字段——那是一个指向libphp代码段的函数指针。ASLR在这一刻宣告失效：已知的函数偏移减去泄露的指针，就是libphp的加载基址。  
  
第三步，伪造。攻击者在multipart字段里写一个精心布局的167字节结构：一个伪造的HashTable（析构函数指针pDestructor被写成了zif_system的地址）、一个伪造的execute_data（参数个数为1）、一个伪造的zend_string（内容是待执行的命令）。再让一次分配把这个结构恰好搬到悬空指针指向的地址上，把PS(http_session_vars)指向的位置覆盖成一个假的引用，指向这个伪造的HashTable。  
  
第四步，引爆。请求B走到END回调，PHP的清理代码顺着悬空指针开始析构它以为的session数据：zval_ptr_dtor → zend_reference_destroy → zend_array_destroy → zend_hash_destroy——最后这一步，HashTable的析构会为每个元素调用pDestructor。而这个pDestructor，是攻击者写的zif_system。命令执行了，输出重定向到web目录，攻击者再发一个普通的GET请求把结果取回来。  
  
整个过程HTTP返回200，worker不崩，应用日志里干干净净。目标的PHP页面哪怕只有一行  
```
<?php echo "OK"; ?>
```  
  
也照打不误——因为从始至终，攻击发生在应用代码开始执行之前。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TanicctQBA08icLESYpxHTxjKuld8ftWIqLdNcdt635aMJD0GMGMocPrjyMjKSTMHRa1ER0muyaMqdCoC3jIQ6bEaicAbB0ymU3d8jQsZN9AUw/640?wx_fmt=png&from=appmsg "")  
  
还有一个更"优雅"的变体值得说一句：甚至不需要内存破坏。攻击者想办法把恶意的序列化对象写进session文件,比如：  
```
O:3:"CMD":1:{s:3:"cmd";s:2:"id";}
```  
  
upload progress在RINIT前读session时会把它反序列化成真实对象，对象的__destruct()被PHP"合法地"调用——如果目标应用里恰好存在一串魔术方法能接力到system()的gadget类（WordPress、Laravel这类生态里从来不缺），RCE就成了。这条路完全不需要碰内存布局，稳定得可怕。  
  
现在你也能理解为什么这个漏洞难防了：  
  
它  
不需要认证——攻击请求就是普通的HTTP POST；  
  
它  
早于应用代码——路由、中间件、WAF规则、ini_set()统统来不及跑；它的入口  
默认开启——upload_progress从PHP 5.4起就是On，不需要应用调用session_start()；session.use_only_cookies=1也挡不住，因为攻击者可以把PHPSESSID放在表单字段里。  
  
  
最后回到AI这个话题  
  
这个漏洞最让我感慨的，不是漏洞本身——写PHP内核这么多年，"请求边界上的状态清理"这种坑，我知道它一直有风险——而是发现它的方式。发现它的安全研究者自己说：没有用什么特殊harness，没有私有模型，就是研究者带着二十年的PHP漏洞史认知（Stefan Esser在2004年就发现过第一代memory limit漏洞），明确地引导AI去做变体分析。AI像是一个不知疲倦、读过所有历史漏洞的审计员，把"publish state → fallible work → commit"这个模式在百万行代码里扫了一遍又一遍，而人负责提出正确的问题。  
  
所以"AI会不会取代安全研究员"这个问题问错了。AI发现不了你不知道该问什么的问题，但它能把你知道该问的问题，问到人力永远达不到的深度和广度。知识在人的脑子里沉淀成方向感，AI把方向感展开成地毯式搜索。  
  
这次是PHP，下午是NodeJS。底层的C库、Python的解释器、JVM……所有"人写的、跑了几十年的"代码，都在同一架显微镜下面了。基础设施软件没有"安全默认"，只有"默认暴露"——这句话送给所有还在跑PHP 7.x的同学。  
  
升级吧，今天。  
  
当然，也有一个办法就关闭PHP的上传进度反馈，这个从根本上可以杜绝RCE，但是不能解决OOM可能带来的DDOS攻击:  
```
session.upload_progress.enabled=Off
```  
  
记得这个需要设置在php.ini中，不能通过ini_set，原因你也懂。  
  
当然，聪明的你想，既然要跨请求才会出现，那就还有一个极端的办法，就是让每个Worker只处理一个请求，fpm设置:  
```
pm.max_requests = 1
```  
  
Apache设置:  
```
MaxRequestsPerChild 1
```  
  
但，这个方法过于凶残，属于宁为玉碎，不为瓦全，各位自行斟酌吧.  
  
