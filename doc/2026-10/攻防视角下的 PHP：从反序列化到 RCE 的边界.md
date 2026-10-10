#  攻防视角下的 PHP：从反序列化到 RCE 的边界  
原创 Sink
                    Sink  船山信安   2026-10-10 02:00  
  
安全开发 · SECDEV  
  
# 攻防视角下的 PHP：从反序列化到 RCE 的边界  
  
  
     安全开发 · 写给想知道攻击者怎么进来的人     
  
       一个 unserialize 调用，就能让攻击者从读数据一路摸到执行命令。这不是夸张。PHP 里藏着不少这样的函数：单看每一行都平平无奇，串起来却能把一个外部输入推到系统命令的边界。这篇写给做开发、又想知道攻击者从哪儿进来的人。我们不讲怎么打穿一个站点，讲那些危险函数为什么危险，以及怎么在架构上把它们堵住。       
  
01  
反序列化：unserialize 吃不可信数据的代价  
  
反序列化是 PHP 里最容易被低估的危险函数之一。  
  
它的日常用法很正常：把一个对象存进缓存，下次取出来直接能用。问题出在 unserialize() 吃下去的那串东西，如果来自用户，攻击者就能构造一个他想要的类的结构。  
  
PHP 的对象有魔术方法。__wakeup 在反序列化时自动调用，__destruct 在对象销毁时自动调用，__toString 在对象被当成字符串用时触发。单个方法也许干不了什么，但攻击者不需要单个。他找一个类里能读文件的方法，再找一个类里能把它当成字符串输出的方法，把两者串成一条调用链——这叫 POP 链，属性导向编程。  
  
链子一旦串起来，从"把一个字符串反序列化"到"读到一个本不该读的文件"，中间没有任何一道你写过的检查。因为魔术方法是 PHP 自动调用的，它绕过了你所有的业务逻辑。  
  
我见过最让人头疼的情况：项目自己没写几个类，但依赖里某个库的类有现成的 __destruct，里面顺手做了一些文件操作。攻击者不需要知道这个库的细节，只要反序列化的入口在，链子就在。某框架曾因反序列化链被利用，根源就是这个机制，不是某个具体的 bug。  
  
很多团队的反序列化入口藏在缓存读取里——从 Redis 取一串数据，直接 unserialize 还原成对象。缓存内容如果可被污染，比如共用的 Redis 实例、可被写入的键，这条链子连入口都不需要用户直接触碰。所以"不可信"的边界，比"用户请求"要宽得多。  
  
PHP 官方手册在 unserialize 页面上自己写了警告：不要对不可信数据使用它。这不是文档里的客套话，是这个函数从设计上就假设输入来自可信源。把不可信数据喂给它，等于把"构造对象图"的权限交了出去。手册给出的方向也是 json_decode 这类纯数据方案。  
  
// 反例：把用户传来的字符串直接反序列化  
  
$data = unserialize($_POST['payload']);   // 危险：__wakeup/__destruct 可能被串联执行  
  
   
  
// 正例：结构数据用 json_decode，纯数据、不触发魔术方法  
  
$data = json_decode($_POST['payload'], true);  
  
if (json_last_error() !== JSON_ERROR_NONE) {  
  
    http_response_code(400);   // 解析失败直接拒绝，不要 fallback  
  
    exit;  
  
}  
  
防护只有一条硬规矩：不要反序列化不可信数据。用户在请求里给你的任何东西，都不可信。  
  
需要结构数据？用 json_decode。JSON 里只有纯数据，没有类，没有魔术方法，反序列化不出对象，自然串不起链子。拿到之后记得判断 json_last_error，解析失败了就直接拒绝，不要退回到别的解析方式。  
  
如果业务逻辑真的必须用到 unserialize——比如要兼容一份历史存储——那至少要加两道：严格约束允许的类（allowed_classes 选项），以及对数据做签名校验，用只有服务器知道密钥的 HMAC 验一遍，签名对不上就丢弃。把入口从"用户给什么我吃什么"变成"用户给的必须在我允许的范围内、且经过我认可"。  
  
       危险不在某个类的某行代码，在那条"入口到魔术方法"的自动调用链。链子断在入口，后面全是空谈。       
  
02  
命令与代码注入：把字符串交给系统的瞬间  
  
命令执行和代码执行，是危险里最直白的两种。  
  
exec、system、passthru、shell_exec 这一组，把一段字符串交给操作系统的 shell 去跑。eval 更狠，把一段字符串当成 PHP 代码本身来执行。它们的共同点：参数是"字符串"，而字符串是可以被拼接的。  
  
拼接就是裂缝。你写 $cmd = 'ls ' . $dir，本意是列某个目录。攻击者把 $dir 换成分号加另一条命令，shell 照单全收。因为对 shell 来说，你拼进去的分号和管道符，跟它自己写的命令没有任何区别。  
  
eval 那边更没有余地。用户传来的内容一旦进了 eval，攻击者发来的就是服务端正在执行的代码。这已经不是"能否提权"的问题，是攻击者此刻就站在你的进程里。  
  
// 反例：用户内容被拼进要执行的命令字符串  
  
$out = shell_exec('ping -c 1 ' . $_GET['host']);   // 危险：等于把命令行交给用户  
  
   
  
// 正例：用参数数组 + escapeshellarg，不让用户输入改变命令结构  
  
$cmd = ['ping', '-c', '1', escapeshellarg($_GET['host'])];  
  
$out = shell_exec(implode(' ', $cmd));  
  
我理解有些场景确实想用。比如按用户选的操作系统执行不同的清理命令。这恰恰是白名单该上场的地方：允许的操作就那么几种，列出来，用户给的如果在表里就用，不在就拒绝。不要让用户输入成为命令的一部分，让用户输入成为"从已知安全选项里挑一个"的索引。  
  
参数必须进命令时，用 escapeshellarg 把它包起来。这个函数会确保用户输入永远只是一个参数、一段带引号的内容，没法变成"下一个命令"。它不改变命令结构，只约束输入的角色。能传参数数组的接口就传数组，比拼字符串稳得多。  
  
还有一句话得说清楚：eval 不是用白名单能救回来的东西。它吃掉的是代码语义，白名单无从列起。我的建议很绝对——业务代码里不要有 eval。遇到"动态执行"的诉求，多半是设计问题，该换的是架构，不是加一层过滤。  
  
顺带一个误区：有人用 preg_replace 的修饰符做"动态替换"，那个写法本质上就是 eval 的另一种形式，PHP 后来把它删了，因为太容易出问题。看到老代码里有它，删掉重写，别想着打补丁。  
  
有人会问，用 escapeshellcmd 不也一样吗？不一样。escapeshellcmd 只转义整条命令里那些可能改变语义的字符，但它不保证每个参数边界清晰；escapeshellarg 才是把单个参数整个包住、让它永远只能是一个值。混用或只用前者，仍可能留口子。要区分这两个函数的职责，别图省事。  
  
03  
SSRF：借你服务器的位置和身份出门  
  
SSRF 的全称是服务端请求伪造，名字拗口，机理却很生活。  
  
你用 file_get_contents($_GET['url']) 去抓用户给的外链，本意是做个预览。攻击者把 url 换成那串云元数据地址，你的服务器就会替他去问云平台的元数据服务，把临时凭证、角色信息一股脑交出来。换成内网地址加管理路径，他就能借你的身份去碰内网里那台没做鉴权的设备。  
  
关键点在于：发起请求的是服务器，不是用户浏览器。服务器的网络位置、它能访问的内网、它挂在身上的云凭证，用户浏览器都没有。SSRF 就是让攻击者"借用"这台服务器的位置和身份。  
  
curl 那一族函数同理。只要 URL 来自用户，且你没拦，他就能把目标指向任意地址、任意协议。  
  
协议这一步最容易被忽略。很多人只校验了"是不是 http"，忘了 file:// 能读本地文件、某些协议能打内网的服务。一个 file:// 加本地路径进来，file_get_contents 会老老实实把文件内容吐回去。  
  
// 示意：URL 校验骨架，禁止私有地址与危险协议  
  
function is_safe_url($url) {  
  
    $parts = parse_url($url);  
  
    $scheme = strtolower($parts['scheme'] ?? '');  
  
    if (!in_array($scheme, ['http', 'https'], true)) {  
  
        return false;   // 禁用 file:// 等协议  
  
    }  
  
    $ip = gethostbyname($parts['host'] ?? '');  
  
    if (ip_in_range($ip, '10.0.0.0/8')  
  
        || ip_in_range($ip, '192.168.0.0/16')  
  
        || ip_in_range($ip, '172.16.0.0/12')  
  
        || $ip === '127.0.0.1'  
  
        || $ip === '169.254.169.254') {  
  
        return false;   // 落私有段或云元数据，拒绝  
  
    }  
  
    return true;  
  
}  
  
// 注：ip_in_range 为示意，请使用经审计的 IP 段判断实现，  
  
//     且务必先解析域名再判段，防 DNS rebinding  
  
防护是分层的。第一层，协议白名单：只接受 http 和 https，别的协议一律拒绝。第二层，URL 解析后取出 host，解析成 IP，判断是不是私有地址段——10.0.0.0/8、192.168.0.0/16、172.16.0.0/12、127.0.0.0/8，还有那个特殊的云元数据地址。落在这些段里，直接拒绝。  
  
有个坑要提醒：只校验用户填的 host 字符串不够。攻击者会填一个域名，让 DNS 解析后再指向内网。所以必须先把域名解析成 IP，再判段。解析这一步本身也要防 DNS rebinding，生产环境建议用支持一次性解析校验的客户端。  
  
第三层，如果业务只访问少数几个固定的外部服务，干脆做目标白名单，连 IP 段判断都省了。能列清单的，就别做判断题。  
  
还有一点：SSRF 的危害不只在"读"。攻击者用你的服务器作跳板，对内网发起请求，可能触发内网里某个有副作用的接口。所以 SSRF 的修复不只是"防止读数据"，是"防止你的服务器替任何人做任何它本不该做的事"。  
  
04  
不要自己造加密：base64 加 md5 不是加密  
  
这条路我踩过，也见过太多团队踩。  
  
需求很常见：把用户的 token 存进 cookie，或者把一段配置落盘。有人图快，base64 一下，再 md5 一次，觉得"别人看不懂就是加密了"。有人更自信，自己写个 XOR，密钥是项目名，觉得"算法只有我自己知道"。  
  
这两种都不叫加密。base64 是编码，不是加密，任何人拿到都能解。md5 是摘要，固定输入固定输出，用来做完整性校验都嫌弱，更别说保密。XOR 对称算法只要密文和明文任一对子被拿到，密钥当场还原。你以为的"别人不知道"，在攻击者眼里是"密文已经在手、就差套个表"。  
  
更危险的是把这种"伪加密"当成安全的依赖。代码里一旦出现"已加密"的注释，后面的人就不会再想防护，数据就这么裸奔在日志、缓存、备份里。  
  
// 对称加密用 libsodium，不要手写算法  
  
$nonce  = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);  
  
$cipher = sodium_crypto_secretbox($plaintext, $nonce, $key);  
  
$packed = base64_encode($nonce . $cipher);   // nonce 随密文一起带过去  
  
// 解密用 sodium_crypto_secretbox_open(...)  
  
// 关键：密钥 $key 来自配置/密钥管理，绝不能硬编码在源码  
  
PHP 自带的东西足够好，不需要你发明。对称加密用 libsodium 的 sodium_crypto_secretbox，或者 openssl 的 AES-256-GCM。它们都是经过长期审查的算法，你只要把"密钥怎么来、怎么存"这件事想清楚。  
  
我必须强调后半句：密钥管理比算法选择更重要。AES-256 再强，密钥硬编码在源码里、提交进版本库，等于没加密。密钥要从配置中心、环境变量、密钥管理服务拿，按环境分离，能轮转。算法选错了还能换，密钥泄露了，所有历史密文一起失效。  
  
还有一类常见错误：用 md5 直接存密码。这是哈希，不是加密，但同样不能裸用——要加盐、要用慢哈希，PHP 的 password_hash 用 bcrypt 或 Argon2。password_hash 和 password_verify 把这件事做对了，直接用，别自己拼。  
  
我也理解团队的历史包袱：老系统用了一套自研"加密"，现在全员依赖，不敢动。这种情况别一次性推翻，先做密钥与算法的盘点，把真正敏感的字段先迁到 password_hash 或 sodium，其余的排期替换。动不了全局，就先守住最高价值的那几处。  
  
再多说一句为什么 XOR 不行。异或运算有个性质：密文异或明文等于密钥，而且是逐字节循环的密钥。只要攻击者拿到任意一对"明文加密文"，密钥当场被还原出来，之后所有密文都能解。你觉得"算法保密"，但保密的前提是攻击者永远拿不到任何一对已知明文——这在真实系统里几乎不可能，因为很多字段的格式是固定的，比如 token 开头、JSON 骨架。  
  
05  
架构层面的纵深防御：单点严防没有意义  
  
前面讲的都是单个函数。现在退一步，看整个系统。  
  
一个函数写对了，不代表系统安全。攻击者从来不打最硬的那面墙，他找链条上最弱的一环。你把 unserialize 堵死了，他在 SSRF 进来；SSRF 堵死了，他在某个忘记鉴权的回调里进。单点严防没有意义，要的是纵深。  
<table><tbody><tr><td style="background:#eef4ff;padding:9px 10px;color:#2563eb;font-weight:bold;border-bottom:1px solid #d4e3ff;font-size:12px;"><section><span leaf="">层</span></section></td><td style="background:#eef4ff;padding:9px 10px;color:#2563eb;font-weight:bold;border-bottom:1px solid #d4e3ff;font-size:12px;"><section><span leaf="">做什么</span></section></td><td style="background:#eef4ff;padding:9px 10px;color:#2563eb;font-weight:bold;border-bottom:1px solid #d4e3ff;font-size:12px;"><section><span leaf="">关键动作</span></section></td></tr><tr><td style="padding:9px 10px;color:#3f3f3f;border-bottom:1px solid #f0f0f0;"><strong><span leaf="">最小权限</span></strong></td><td style="padding:9px 10px;color:#6b7280;border-bottom:1px solid #f0f0f0;"><section><span leaf="">进程与账号只给必要权限</span></section></td><td style="padding:9px 10px;color:#6b7280;border-bottom:1px solid #f0f0f0;"><section><span leaf="">普通用户运行、数据库最小授权</span></section></td></tr><tr><td style="padding:9px 10px;color:#3f3f3f;background:#f7faff;border-bottom:1px solid #f0f0f0;"><strong><span leaf="">禁用危险函数</span></strong></td><td style="padding:9px 10px;color:#6b7280;background:#f7faff;border-bottom:1px solid #f0f0f0;"><section><span leaf="">语言层面砍掉 exec 等</span></section></td><td style="padding:9px 10px;color:#6b7280;background:#f7faff;border-bottom:1px solid #f0f0f0;"><section><span leaf="">php.ini disable_functions</span></section></td></tr><tr><td style="padding:9px 10px;color:#3f3f3f;border-bottom:1px solid #f0f0f0;"><strong><span leaf="">WAF</span></strong></td><td style="padding:9px 10px;color:#6b7280;border-bottom:1px solid #f0f0f0;"><section><span leaf="">请求入口的粗筛</span></section></td><td style="padding:9px 10px;color:#6b7280;border-bottom:1px solid #f0f0f0;"><section><span leaf="">当兜底，不当唯一防线</span></section></td></tr><tr><td style="padding:9px 10px;color:#3f3f3f;background:#f7faff;border-bottom:1px solid #f0f0f0;"><strong><span leaf="">威胁建模</span></strong></td><td style="padding:9px 10px;color:#6b7280;background:#f7faff;border-bottom:1px solid #f0f0f0;"><section><span leaf="">画数据流、标信任边界</span></section></td><td style="padding:9px 10px;color:#6b7280;background:#f7faff;border-bottom:1px solid #f0f0f0;"><section><span leaf="">每个边界写清该有的校验</span></section></td></tr></tbody></table>  
第一层：最小权限。Web 进程不要跑在 root 下，给它一个普通用户，只能写它该写的目录。数据库账号不要给 DROP、FILE 权限，只给这个应用真正用到的那几张表的增删改查。这样就算某处真的被 RCE，攻击者拿到的也是一个被削掉牙的进程，能做的有限。  
  
; php.ini：按业务需要禁用能执行命令、读文件的函数  
  
disable_functions = exec,system,passthru,shell_exec,proc_open,popen  
  
; 注意：eval 是语言构造，disable_functions 管不到它，  
  
;       只能从代码层面杜绝把用户输入交给它  
  
第二层：在语言层面砍掉危险函数。php.ini 里的 disable_functions 可以把 exec、system、shell_exec 等按业务需要禁用。注意 eval 是语言构造，disable_functions 管不到它，只能从代码层面杜绝。禁用是兜底，不是防线——别因为它关了几个函数就放松别的。  
  
第三层：WAF。很多人把 WAF 当成安全银弹，这是误会。WAF 在请求入口做规则匹配，能拦掉一批明显的攻击流量，但它看不懂你的业务逻辑，绕过的手法一直都有。把它当第一道粗筛可以，当唯一防线不行。  
  
第四层，也是我最想说的：威胁建模。不需要复杂的工具，一张纸就能做。把系统的数据流画出来——请求从哪进、数据去哪、调了哪些外部服务、写了哪些文件。在图上标出信任边界：哪些是外部不可信的，哪些是内部可信的。然后在每一条边界上写清楚：这里该有什么校验。  
  
做完这张图你会发现，很多"危险函数"根本不该出现在那段数据流里。危险不是函数本身，是它被放在了错误的信任边界上。威胁建模的价值，就是让你在写第一行代码之前，先把"数据会经过哪些边界"想明白。  
  
补充一个常被忽略的边界：错误信息和日志。RCE 之前的侦察阶段，攻击者大量依赖报错暴露的路径、版本、表结构。把生产环境的 display_errors 关掉，错误进日志不进响应，能砍掉一大块侦察信息。纵深防御也包括"别替攻击者把地图画好"。  
  
这四层不是选一个，是叠在一起。任何一层被突破，下面还有一层。攻击者要成功，得一口气打穿所有层；而你要做的，是让任何一层都够硬，硬到他放弃。  
  
我见过把四层都布了、却因为"内网可信"这个假设翻车的案例。他们把管理后台放在内网，认为"外面的攻击者进不来"，于是后台的接口连基础鉴权都省了。结果一个 SSRF 把内网打通，后台形同裸奔。信任边界一旦画错，纵深再深也是纸糊的。所以威胁建模里最该反复问的是：我凭什么相信这一段是安全的？  
  
06  
收尾：精通拼的不是函数，是想清楚的那一下  
  
安全开发到精通这一层，拼的不是你会不会用 escapeshellarg，会不会调 sodium_crypto_secretbox。这些查手册就会。  
  
真正拉开差距的，是动笔之前那一下：数据会去哪、谁能碰它、出了事谁兜底。  
  
我见过写得一手好防御代码、却把敏感数据写进前端日志的人。也见过禁用了一堆函数、却让 Web 进程跑在 root 下的部署。单点都对，系统却处处是缝。  
  
攻击者不挑你的强项打，他挑缝钻。而缝，往往长在你从没想过"这里也要防"的边界上。  
  
所以这篇文章我反复在讲机理，不肯给你一条能直接用的链子。因为给了链子，你记住的是"怎么打"；讲清机理，你才可能长出"怎么防"的直觉。  
  
我承认这里有个纠结：讲得太细，怕被拿去当教科书；讲得太浅，又怕你只记住几个函数名。权衡之下我选了中间那条——把每个危险点的机理和边界说清楚，把能直接抄的安全写法给你，但绝不把完整利用链摆出来。你能照着把代码写对，就够了。  
  
**ONE LINE**  
  
       能拦住攻击者的，从来不是某个聪明的函数，是你在架构里为每个信任边界留好的那道校验。       
  
参考来源  
  
· OWASP Top 10（2021）：A08 软件与数据完整性失效、A10 服务端请求伪造（SSRF）两项与本文主题直接相关  
  
· OWASP Deserialization Cheat Sheet（OWASP 官方反序列化防护速查）：关于不反序列化不可信数据、使用类型约束与签名校验的指引  
  
· OWASP SSRF Prevention Cheat Sheet（OWASP 官方 SSRF 防护速查）：URL 解析、协议白名单、私有地址段过滤的推荐做法  
  
· PHP 官方手册：unserialize 函数页（allowed_classes 选项）、sodium 扩展页（sodium_crypto_secretbox 用法）  
  
  
