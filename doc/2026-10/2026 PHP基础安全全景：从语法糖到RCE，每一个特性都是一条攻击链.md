#  2026 PHP基础安全全景：从语法糖到RCE，每一个特性都是一条攻击链  
原创 KLSEC
                    KLSEC  昆仑AI安全实验室   2026-10-24 16:48  
  
2026年10月23日，PHP 8.4.27和8.5.12紧急发布  
。修复的是一个叫PHP2Shell的漏洞链，CVSS 9.2，无需认证即可远程代码执行。补丁发布时PoC已经公开，有云厂商在泄露前十小时就监测到数千次攻击尝试  
。  
  
漏洞出在session.upload_progress  
。rfc1867上传处理器缓存了指向session HashTable的裸指针。如果在两个文件数据块之间制造一个超过1秒的停顿，session会重新初始化，HashTable被释放，但缓存指针仍然指着那块已释放的内存  
。后续的文件数据事件通过悬空指针写入，Use-After-Free变成代码执行。PoC只需要一个multipart POST，加上一个刻意制造的停顿。  
  
这个漏洞的教学意义不在“又一个RCE”。它说明了一件事：**PHP的安全边界，从来不只在你写的代码里。session.upload_progress这个你几乎从不显式调用的特性，本身就是一个攻击面。**  
 你审完了include  
、审完了file_get_contents  
、修完了SQL注入，运行时的一个“便利特性”把整条防线绕过去了。  
  
这篇文章按PHP的基础脉络走一遍——声明、作用域、字符串、数组、函数、类、数据库、文件、网络、Session——但每一节都带着同一个问题：**这个特性的信任边界在哪里，攻击者会从哪里切进去。**  
  
**一、声明、作用域与字符串：数据进入代码的第一道缝**  
  
PHP的变量声明极其宽松。$a = "1"  
，$a = 1  
，在弱比较的世界里它们可能相等。2026年仍然有大量代码在拿==  
做安全判断。  
```
if ($_GET['role'] == "admin") { /* 管理员逻辑 */ }
```  
  
传?role=0  
，"0" == "admin"  
在PHP 8之前可能为真。PHP 8修正了字符串与数字的比较逻辑，但**弱比较的思维惯性没有消失**  
。更隐蔽的是in_array()  
不传第三个参数true  
，类型转换的漏洞在2026年的代码库里仍然随处可见。  
  
字符串函数是另一个被低估的攻击面。addslashes()  
在2026年仍然被大量使用，但PHP 8移除了get_magic_quotes_gpc()  
之后，依赖这个函数做“自动转义”的历史代码失去了原本的保护层。更根本的问题是：addslashes()  
不是为SQL上下文设计的，它不处理字符集，不处理NO_BACKSLASH_ESCAPES  
模式。2026年4月披露的OTRS漏洞（CVE-2026-48188）就是这个模式的典型案例——应用层用\'  
转义单引号，数据库启用了NO_BACKSLASH_ESCAPES  
，反斜杠失去特殊含义，\'  
被解析为两个普通字符，注入成立。  
  
**字符串的安全边界不是“有没有过滤”，是“过滤依赖的语义假设在当前环境下是否成立”。**  
  
**二、数组与自定义函数：沙箱的裂缝在类型转换里**  
  
PHP数组的灵活性是一把双刃剑。array_merge()  
在2026年5月被披露存在堆缓冲区溢出（CVE-2025-14178），影响从PHP 7.1开始的多个版本。urldecode()  
配合ctype  
函数在默认signed char的平台上可能导致负偏移越界访问（CVE-2026-7258）  
。  
  
但这些运行时漏洞之外，数组在应用层的安全风险更值得关注。Twig模板引擎在2026年连续披露了两个沙箱绕过（CVE-2026-48806/48807），根因都是**动态映射键的类型转换**  
。当数组的键是一个Stringable  
对象时，PHP会自动调用__toString()  
。Twig的沙箱本应在调用前检查“这个对象是否允许被字符串化”，但在ArrayExpression  
和join  
/replace  
过滤器里，这个检查被遗漏了。攻击者可以通过控制模板输入，让沙箱在不知情的情况下执行任意__toString()  
方法。  
  
这和你自己写自定义函数有什么关系？关系在于：**任何接受数组或对象作为参数的函数，只要内部触发了隐式类型转换，就继承了PHP引擎的类型转换语义，而不是你的安全策略。**  
 你写了一个render($data)  
函数，以为$data  
是数组就安全了，但数组里的某个值可能是一个对象，对象的__toString()  
可能来自一个你不信任的类。  
  
**三、类的访问控制与继承：private不是安全边界**  
  
private  
方法不能被外部直接调用，这是PHP的访问控制规则。但**访问控制不是安全控制**  
。  
```
class User {
    private function getSecret() { return $this->secret; }
}
```  
  
如果攻击者能触发反序列化，private  
方法可以通过魔术方法被间接调用。如果攻击者能控制__destruct  
或__wakeup  
里的逻辑，private  
属性在对象内部是可读的。**private的语义是“这个类的外部不能直接调用”，不是“这个数据不能被攻击者拿到”。**  
  
2026年8月披露的MaxSite CMS漏洞（CVE-2026-70554）就是这个逻辑的翻版。maxsite_comuser  
 cookie被直接传给unserialize()  
，没有类白名单检查，攻击者构造恶意序列化对象，在对象图重建过程中触发魔术方法，实现POP攻击或利用SoapClient  
/Imagick  
扩展的gadget chain完成RCE  
。类的访问控制修饰符在反序列化面前没有提供任何屏障。  
  
静态方法和静态变量同理。静态属性在请求之间不共享（PHP的请求隔离模型），但在同一个请求的整个生命周期内是全局可写的。一个静态的“配置”变量，如果在某个代码路径上被用户输入覆盖，所有读取它的代码都会受影响。**访问控制管的是“语法上能不能调用”，不是“运行时数据是否可信”。**  
  
**四、数据库与预处理：最危险的谎言是“我用了prepare”**  
  
预处理语句是防SQL注入的正确方案。但2026年的现实是：**“用了prepare”和“防住了注入”之间，隔着一整套实现细节。**  
  
PDO的ATTR_EMULATE_PREPARES  
是第一个陷阱。PHP默认开启模拟预处理，此时prepare()  
只是在客户端做字符串替换，数据库根本没有收到预编译指令  
。攻击者仍然可以利用宽字节注入——连接字符集是GBK时，%df'  
的前半部分和转义用的反斜杠\  
（0x5c）结合成一个合法的GBK字符，后半部分的单引号逃逸出来  
。**“关了模拟预处理”是硬门槛，不是可选项**  
。  
```
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```  
  
但这还不够。连接字符串里必须显式指定charset=utf8mb4  
，否则宽字节的土壤仍然存在  
。更隐蔽的是“部分预处理”——AVideo的SQL注入（CVE-2026-33767）中，users_id  
用了占位符，videos_id  
直接拼接：  
```
$sql = "SELECT * FROM likes WHERE users_id = ? AND videos_id = ".$this->videos_id." LIMIT 1;";
```  
  
**部分预处理比完全不用预处理更危险**  
，因为它给了代码审查者一种虚假的安全感，让人默认“这个文件已经安全了”，从而跳过了逐条SQL的追溯。  
  
2026年Drupal核心的SQL注入（CVE-2026-9082）则走了另一条路：重复前缀PDO模拟预处理绕过  
。攻击者构造一个以多个前缀字符开头的Payload，利用PDO模拟预处理在客户端拼接时的边界处理缺陷，让拼接后的SQL在数据库端被解析为注入语句。**预处理的保护强度，取决于它是否真正使用了数据库原生的预处理协议，而不是PDO在客户端做了一次字符串拼接。**  
  
**五、文件操作：file_get_contents的两副面孔与路径的“最后一步”**  
  
file_get_contents()  
是PHP里最方便的函数之一。也是2026年SSRF漏洞最常见的入口。  
  
AVideo的两个端点调用isSSRFSafeURL()  
验证了用户提供的URL，然后直接用file_get_contents()  
去拉取  
。问题在于：isSSRFSafeURL()  
只验证初始URL，PHP的file_get_contents()  
默认跟随302重定向  
。攻击者提供一个指向自己服务器的URL，服务器返回302跳转到http://169.254.169.254/latest/meta-data/  
，SSRF防护被完全绕过  
。  
  
同一个代码库里，url_get_contents()  
函数**正确实现了**follow_location=0  
加上逐跳重新验证：  
```
$opts = ['http' => ['follow_location' => 0]];
$context = stream_context_create($opts);
for ($redirectCount = 0; $redirectCount <= 5; $redirectCount++) {
    $fetched = file_get_contents($currentUrl, false, $context);
    // ... parse Location header ...
    if ($redirectTarget) {
        if (!isSSRFSafeURL($redirectTarget)) { return false; } // 逐跳重新验证
        $currentUrl = $redirectTarget; continue;
    }
}
```  
  
但那两个端点**没有调用这个安全函数**  
，而是直接用了裸的file_get_contents()  
。根因不是“没有防护”，是**防护在正确的函数里实现了，但没有传播到所有调用点**  
。  
  
Admidio的SSO元数据端点更直接。FILTER_VALIDATE_URL  
只做格式校验，不做协议白名单。file:///etc/passwd  
、http://127.0.0.1/  
、data://text/plain,payload  
全部通过校验。file://  
协议甚至不依赖allow_url_fopen  
，它是原生文件系统处理器，永远可用。  
  
**网络请求的安全边界不在“有没有校验URL”，在于“校验之后，执行请求的函数是否继承了校验时的假设”。**  
  
文件写入侧，AVideo的locale/save.php  
把“最后一步”的杀伤力展示得更清楚：  
```
$file = $dir . $_POST['flag'] . ".php";
fwrite(fopen($file, "w"), $_POST['code']);
```  
  
flag  
被直接拼进路径，没有basename()  
，没有../  
过滤  
。传flag=../../webshell  
，路径逃出locale/  
目录，落在Web根目录。code  
原样写入，攻击者写入<?php system($_GET['c']);?>  
，WebShell到手  
。  
  
这个漏洞还有一层放大器：**端点的CSRF保护是零**  
。SameSite=None  
让跨站POST自动携带管理员会话，攻击者不需要拿到管理员密码，只需要让管理员访问一个恶意页面，表单自动提交，WebShell就写进去了  
。**“写文件”本身不一定是RCE，但当写入路径可控、写入内容可控、写入位置在Web目录内，RCE就是必然结果。**  
  
**六、文件包含与伪协议：读源码和RCE之间的那条线**  
  
文件包含漏洞的入门Payload是?file=../../etc/passwd  
，但2026年真正的杀伤力在伪协议。  
  
php://filter/read=convert.base64-encode/resource=index.php  
，把PHP文件以Base64输出，**绕过PHP引擎的执行**  
，直接读源码  
。读源码的目的不是“看代码”，是找数据库密码、找硬编码密钥、找下一个可利用的入口。  
  
一个典型的Resort Reservation System漏洞代码是：  
```
$page = $_GET['page'] ?? 'home';
include($page . ".php");
```  
  
代码强制拼接了.php  
后缀，看起来限制了包含的文件类型。但php://filter  
的resource=  
参数会覆盖这个后缀——?page=php://filter/convert.base64-encode/resource=DBConnection  
，include()  
收到的完整路径是php://filter/.../resource=DBConnection.php  
，.php  
后缀被追加在resource路径的末尾，**反而正好补全了目标文件的扩展名**  
。攻击者读到了DBConnection.php  
的Base64编码内容，其中包含数据库连接凭证和管理员的bcrypt密码哈希。  
  
PHP RFC在2026年5月提出限制php://filter  
中链式过滤器的数量，目标就是缓解这类“LFI升级为RCE”的攻击  
。但RFC的推进速度远慢于攻击者的利用速度。**文件包含的防御难点在于：你可以过滤../，但过滤不了php://；你可以禁用allow_url_include，但php://filter是本地协议，不受这个开关影响。**  
  
**七、Session与Cookie：在代码运行之前就要锁死的东西**  
  
PHP2Shell的教训是：session.upload_progress  
这个特性本身可以成为RCE入口。但这不是说你应该关掉session，而是说 **session的安全配置必须在应用代码运行之前就锁死**  
。  
  
2026年的安全基线很明确：session.cookie_httponly = 1  
阻断XSS窃取，session.cookie_secure = 1  
强制HTTPS，session.cookie_samesite = Strict  
或Lax  
限制跨站发送，session.use_strict_mode = 1  
拒绝攻击者提供的未初始化Session ID，session.use_only_cookies = 1  
不接受URL里的Session ID。  
  
MISP在2026年9月披露的Session固定漏洞（CVE-2026-85238）展示了缺少这些配置的后果：__customAuthentication()  
把认证用户写入了**已存在的**  
CakePHP session，而Session->renew()  
的调用在此前被禁用了  
。攻击者预先设定一个Session ID，诱导受害者使用这个ID登录，登录后Session没有被重新生成，攻击者手里的旧ID直接变成了已认证会话。  
  
session_regenerate_id(true)  
的调用时机比参数更重要。**登录成功后必须立即调用**  
，这是防Session固定的底线。权限变更时也应该重新生成，并设置周期性轮换（比如每300秒一次），限制令牌被窃取后的有效窗口。PHP官方在PHP2Shell的公告里给出了一个更具体的配置建议：如果没有运行时切换session.serialize_handler  
的需求，保持session.upload_progress.enabled=1  
是可以的；但如果应用在运行时用ini_set()  
把序列化处理器从php_binary  
切到php  
，就必须关闭上传进度功能  
。**安全配置不是“一刀切”，是“理解你的应用在运行时做了什么之后，再决定开什么、关什么”。**  
  
**八、登录系统与防注入：把上面所有东西串起来**  
  
一个安全的登录系统，不是“一个防注入的SQL查询”那么简单。它是一条链：  
  
**输入到达**  
 → 参数化查询（ATTR_EMULATE_PREPARES=false  
 + charset=utf8mb4  
 + 完整占位符）验证凭证 → 凭证正确 → session_regenerate_id(true)  
 → 写入最小化的Session数据 → 后续每个请求从Session读取用户ID，而不是从请求参数读取。  
  
这条链上任何一环的“部分正确”都会瓦解整条链。你关了模拟预处理，但videos_id  
拼进去了，注入成立。你用了Session，但没调session_regenerate_id  
，固定攻击成立。你从Session读了用户ID，但另一个接口从$_GET['user_id']  
读，越权成立。  
  
2026年的代码审计，不应该再从“有没有危险函数”开始。应该从 **“数据从哪个入口进来、经过了哪些转换、在哪个上下文里被当成代码执行”**  
 开始。addslashes  
在SQL上下文里是错的，在HTML上下文里也是错的（应该用htmlspecialchars  
），但它在一个纯文本日志上下文里可能是对的。**函数本身没有安全属性，上下文才有。**  
  
**写在最后**  
  
PHP2Shell在2026年10月的爆发，不是PHP“不安全”的证据。它是一个提醒：**一个为Web而生的语言，它的每一个便利特性都是为了让事情“能跑起来”而设计的，安全不是默认值。**  
  
session.upload_progress  
是为了让上传进度条能显示，它的UAF是运行时缺陷。file_get_contents  
是为了让读文件/读URL只有一种写法，它的SSRF是使用者的信任错位。prepare  
是为了把SQL结构和数据分开，但“部分prepare”让开发者以为分开了，实际上没有。private  
是为了封装，但反序列化让封装在对象图重建面前失效。  
  
**基础语法和基础命令的价值，不在于背下来，在于知道它们的边界在哪里，以及越过边界之后会发生什么。**  
 你写的每一行include  
、每一个file_get_contents  
、每一次prepare  
，都是在划定一条信任边界。攻击者不会去破你的墙，他会去找你划边界时留下的那条缝。  
  
**严正声明**  
  
本文所述所有PHP特性分析、漏洞案例及安全配置，仅用于**已获得明确书面授权**  
的安全测试、代码审计、红队演练和教育研究场景。PHP2Shell（CVE-2026-109618/109619）已在PHP 8.4.27、8.3.36、8.2.35、8.5.12中修复，请立即更新运行时，并评估session.upload_progress.enabled  
的配置必要性  
。AVideo（CVE-2026-43884、CVE-2026-33767）、MaxSite（CVE-2026-70554）、MISP（CVE-2026-85238）等应用层漏洞的验证必须在授权范围内进行，并及时将受影响组件更新至修复版本。未授权扫描、测试、攻击行为违反《中华人民共和国网络安全法》第27条及《刑法》第285条，将承担法律责任。技术无罪，边界是法律。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/3oR6eMARh6x3qdwzDiaIItODAKv2w1v63ibhkibWUIfco5bvDW6vqICxgQ7m8jVE9xS1EpQiaFegYxJEjJGd1tELXm5TF4ZwlS0leMJ14TtjVibY/640?wx_fmt=jpeg&from=appmsg "")  
  
  
