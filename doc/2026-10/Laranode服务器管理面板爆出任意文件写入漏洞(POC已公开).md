#  Laranode服务器管理面板爆出任意文件写入漏洞(POC已公开)  
YGnight
                    YGnight  night安全   2026-10-11 02:08  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctVS8Ps0NsFTqMiasz8uDibcvoib0spt17ORFYGT7Lk8y0JElHpWukiczXboIicO8mrOUut0DwfE4PvMwpuReeibS0yQHqN8Tictdm4Z4/640?wx_fmt=png&from=appmsg "")  
  
## 一、漏洞描述  
  
Laranode 是装在服务器上的虚拟主机管理面板，Laravel 写的，一套面板管着好几个租户的站点和数据库。  
  
面板里带了个文件管理器，租户登录后能在自己的家目录里传文件、改内容。  
  
1.2 及更早版本里，上传接口把文件写到哪个目录，是由请求里的一个字段定的。  
  
这个字段没有被清理。  
  
里面带 ../ 的话，会原样进到最后那次字符串拼接。  
  
于是普通租户账号能把文件落到别的租户家目录下面，连文件内容一起。  
  
编号 CVE-2026-100520，厂商公告 GHSA-34h2-2696-vfvr，归类 CWE-22。  
  
CVSS 3.1 打分 8.8，向量 AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H，三项影响都记 High。  
  
CVSS 4.0 打分 8.7，向量 AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H。  
  
两份都由 VulnCheck 给出。NVD 那条记录状态是 Deferred，没自己评分。  
  
受影响区间从 0 到 1.2，1.2.1 修掉。  
  
触发要一个能登录面板的普通租户账号。管理员权限不需要，用户交互也不需要。  
  
能写成功还靠一层部署状态。安装脚本会把面板进程加进每个租户的组，租户家目录对它是组可读写的。  
  
当时挡在租户与租户之间的，就是应用层那段路径处理。  
  
文件落到别人的站点根目录之后，剩下交给 Web 服务器。请求到那个文件就会被解析运行，进程权限沿用的是那个租户的系统用户。  
  
公开的验证脚本已经放出，一个 poc.py。  
  
时间上，修复代码 2026 年 9 月 21 日合并，CVE 记录同年 9 月 26 日发布，9 月 30 日最后一次更新。  
  
公开情报里没有观测到在野利用记录。EPSS 对这个编号没有返回数据。  
<table><tbody><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVE 编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">CVE-2026-100520</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">公告编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">GHSA-34h2-2696-vfvr</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">漏洞类型</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">路径穿越转任意文件写入，CWE-22</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">影响组件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">Laranode 文件管理器的上传动作 app/Actions/Filemanager/UploadFileAction.php，接口 POST /filemanager/upload-file</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">受影响版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(220, 38, 38);font-weight: bold;"><section><span leaf="">0 到 1.2，含 1.2</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">修复版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(146, 64, 14);font-weight: bold;"><section><span leaf="">1.2.1</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">触发前提</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">持有任意一个能登录面板的普通租户账号</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">利用条件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">无需管理员权限，无需用户交互，目标目录对面板进程可写</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">危害面</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">跨租户写文件，写进对方站点根目录后可被请求执行，以该租户系统用户身份运行</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVSS</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">8.8 CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H，8.7 CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H，均由 VulnCheck 给出</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">在野利用</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">公开情报中暂无已观测到的在野利用记录</span></section></td></tr></tbody></table>## 二、漏洞原理  
  
入口挂在 POST /filemanager/upload-file 上，中间件只有 auth 一条。控制器那一层做的是转发，中间没有别的动作。  
  
// routes/web.php v1.2  
Route::post('/filemanager/upload-file', [FilemanagerController::class, 'uploadFile'])  
    ->middleware(['auth'])->name('filemanager.uploadFile');  
// app/Http/Controllers/FilemanagerController.php v1.2  
public function uploadFile(UploadFileAction $uploadFile, Request $r)  
{  
    return $uploadFile->execute($r);  
}  
  
处理全落在 execute 里，开头是表单校验。  
  
// app/Actions/Filemanager/UploadFileAction.php v1.2  
$r->validate([  
    'file' => 'required|file',  
    'chunkIndex' => 'required|integer|min:0',  
    'totalChunks' => 'required|integer|min:1',  
    'originalName' => 'required|string|max:255',  
    'path' => 'required|string',  
]);  
  
path 这一项挂了两条规则，required 管有没有送过来，string 管是不是字符串。  
  
字符内容一概不管，斜杠、点、长度都没设限。  
  
校验过了就拼路径。  
  
$file = $r->file('file');  
$path = '/home/' . $r->user()->systemUsername . '/' . $r->path;  
File::append($path . '/' . $r->originalName, $file->get());  
  
落点由三段字符串拼出来，中间不做任何处理。  
  
第一段是 /home/ 加自己的系统用户名，后两段是这次请求里的 path 和 originalName。  
  
后两段之间没有 basename 比对，没有过滤，拼好的结果也没拿回去跟家目录对一次。  
  
拼好的字符串直接进了 File::append。Laravel 这个门面最后落到 PHP 的 file_put_contents，路径里那两个点由内核处理，先退一层，再往下走。  
  
本机用 PHP 7.0.9 复算了这次拼接，下面是从点开始的字符串处理结果。  
  
$path = '/home/' . 'alice' . '/' . '../bob/public_html';  
$dest = $path . '/' . 'pwn_12345.php';  
// 本机实际输出  
// $dest /home/alice/../bob/public_html/pwn_12345.php  
// basename($dest) pwn_12345.php  
// dirname($dest) /home/alice/../bob/public_html  
  
PHP 这一层一个字符都没动，两个点完整保留。真正的目录回溯发生在打开文件时，由内核按路径规则处理。  
  
应答里的 toDestination 就是拼好的那个字符串，此时里面已经是别人的家目录。  
  
同一条 path 还被送去了另一处。  
  
上传完要给新文件修属主和权限。面板进程改不了属主，这一步调 sudo 跑 laranode-file-permissions.sh。  
  
// app/Actions/Filemanager/UploadFileAction.php v1.2 写入之后紧跟的一步  
(new EnsurePermissionsAction)->execute($r->path . '/' . $r->originalName, $r->user()->systemUsername);  
// app/Actions/Filemanager/EnsurePermissionsAction.php v1.2  
Process::run([  
    'sudo',  
    config('laranode.laranode_bin_path') . '/laranode-file-permissions.sh',  
    $path,  
    $systemUser,  
]);  
  
送过去的还是那个带 ../ 的值。越界的就不止写文件这一件事。  
  
那一步脚本是 root 执行的。属主能被改到任意一个写得进去的目标上，权限也一样。  
  
文件写完，后半段就跟文件管理器没关系了。落进 public_html 的 PHP 文件，被 HTTP 请求到会被 Web 服务器交给 PHP 解析，进程权限沿用那个租户的系统用户。  
  
厂商公告把这里的问题归成两类。另一类是其他操作会跟着符号链接走，编号 CWE-59，1.2.1 一并处理。本篇只跟着上传这一条。  
  
1.2.1 的改法没停在加一层过滤。文件名先被单独约束住了。  
  
// app/Actions/Filemanager/UploadFileAction.php v1.2.1  
$originalName = str_replace('\\', '/', $r->string('originalName')->toString());  
if (basename($originalName) !== $originalName || in_array($originalName, ['.', '..'], true)) {  
    return response()->json(['error' => 'Invalid upload file name'], 422);  
}  
  
反斜杠先换成斜杠，再拿 basename 比一遍。两边不一致说明带了路径成分，直接 422 挡回去。单独的点与两个点也在黑名单里。  
  
相对的那一段也会先过一遍规整，/home 前缀不再由这一步自己拼。  
  
$relativePath = (new WhitespacePathNormalizer)  
    ->normalizePath($r->path . '/' . $originalName);  
SafeFileProcess::run(  
    'append',  
    $homedir,  
    [$relativePath, $r->integer('chunkIndex') === 0 ? 'reset' : 'continue'],  
    $file->get(),  
    $r->user()->systemUsername,  
    config('laranode.laranode_bin_path'),  
);  
  
真正兜住边界的是里面那个辅助脚本，一个 Python 文件 app/Filesystem/safe_file.py。  
  
def split_path(path):  
    if not path or path.startswith('/') or '\x00' in path:  
        fail('Invalid relative path')  
    parts = path.split('/')  
    if any(part in ('', '.', '..') for part in parts):  
        fail('Invalid relative path')  
    return parts  
def open_parent(root_fd, path, create, owner):  
    parts = split_path(path)  
    current = os.dup(root_fd)  
    for component in parts[:-1]:  
        following = os.open(component, os.O_RDONLY | os.O_DIRECTORY | os.O_NOFOLLOW, dir_fd=current)  
        ...  
    return current, parts[-1]  
  
它从家目录取一个描述符，往下的每一级目录都用 O_NOFOLLOW 打开，锚点始终是那个描述符。  
  
绝对路径、空段、单独一个点、两个点，在 split_path 这一段就结束了。中间某一级被换成符号链接，os.open 会带上 O_NOFOLLOW 直接报错。  
  
写入也不再对着目标路径直接打开。  
  
handle = os.open(  
    temporary,  
    os.O_WRONLY | os.O_CREAT | os.O_EXCL | os.O_NOFOLLOW,  
    0o600,  
    dir_fd=parent,  
)  
write_input(handle, source)  
os.fchmod(handle, mode)  
os.fchown(handle, uid_gid[0], uid_gid[1])  
os.fsync(handle)  
os.rename(temporary, name, src_dir_fd=parent, dst_dir_fd=parent)  
  
内容先写在随机名的临时文件上，权限和属主改完，再由 rename 挪过去，整条路径都挂在家目录那个描述符下面。  
  
绕这一圈的用意是没有间隔。先判断路径、再打开文件，两步之间是有时间差的，那段空档里目录树可以被改掉。描述符锚定加逐级不跟链接，是把越界变成做不到，而不是检查出来。  
  
这一版还动了两处跟权限有关的东西。sudo 规则从目录通配改成了脚本白名单，面板自己的目录树不再对管理进程组可写。  
## 三、修复建议  
  
升到 1.2.1。  
## 近期值得重点关注的漏洞预警  
  
[【紧急漏洞预警】nacos爆出权限绕过漏洞，可以创建管理员账户，经过分析有点出处（已复现，exp脚本已编写）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486343&idx=1&sn=b67c4eb02c03af9c3b206b1241ff7e9c&scene=21#wechat_redirect)  
  
  
[PostgreSQL爆出远程代码执行漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486480&idx=1&sn=31416416494b366dc85cf4d20e0ab3ee&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Atlassian八款产品爆任意文件读取漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486475&idx=1&sn=3c669c88388236c2af996ceb9782e928&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Joomla视频画廊扩展存在SQL注入漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486469&idx=1&sn=6146142b1b0095532a2d87dcbcf738df&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Roundcube网页邮件客户端存在SQL注入漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486464&idx=1&sn=93e0df392b292c72408f2c33cfb868ca&scene=21#wechat_redirect)  
  
  
[WordPress页面模板存在文件包含漏洞(POC已公开)CVE-2026-87902](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486459&idx=1&sn=5dc658a83a772526773f3e87b1ac8ad0&scene=21#wechat_redirect)  
  
  
[WordPress插件Forminator未认证短代码执行漏洞(poc已公开)CVE-2026-92229](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486454&idx=1&sn=64894c978fbfac86c914945580147bd9&scene=21#wechat_redirect)  
  
  
[WordPress插件Authorizer存在权限提升漏洞(POC已公开)CVE-2026-81294](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486449&idx=1&sn=20a4f9a399ae34e07a27c7b6d2049e45&scene=21#wechat_redirect)  
  
  
[WordPress插件MStoreAPI存在认证绕过漏洞(POC已公开)CVE-2026-13447](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486444&idx=1&sn=a22077edeab35852d6791825ebba8b10&scene=21#wechat_redirect)  
  
  
[CakePHP爆出SQL注入漏洞(POC已公开)CVE-2026-77635](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486439&idx=1&sn=5030655c1d40e179369ab972ddd5dfc9&scene=21#wechat_redirect)  
  
  
[Node.js模块proxy-addr爆出严重漏洞CVE-2026-90711](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486429&idx=1&sn=fc1772f8703bdd4128ac829c1ae8b371&scene=21#wechat_redirect)  
  
  
[Langflow爆出代码执行漏洞CVE-2026-17633  (POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486424&idx=1&sn=0fec55e718cb9fb60eb91ca24307fd1a&scene=21#wechat_redirect)  
  
  
[Apache CVE-2026-42536，补它只用一行，洞龄十四年](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486419&idx=1&sn=393b6baeea4f624ffa8440058c0d324e&scene=21#wechat_redirect)  
  
  
[【紧急】搜狗输入法爆出远程代码执行漏洞CVE-2026-51990  ，已被在野利用](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486413&idx=1&sn=0b2c6a470a165f288873fa6bace1d77b&scene=21#wechat_redirect)  
  
  
[【紧急】无需登录就能获取 GitLab 的任意文件数据(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486408&idx=1&sn=06f47231a649e03866f9e9524b5469a4&scene=21#wechat_redirect)  
  
  
[【紧急】JumpServer堡垒机被公布Access Key泄露漏洞，可以实现提权](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486403&idx=1&sn=8e4dc6a684e95b9dc5871b1aca487a57&scene=21#wechat_redirect)  
  
  
[Spring Cloud Azure公布严重漏洞 无需任何权限即可取得登录会话](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486398&idx=1&sn=c64f6f82f75da584d032b99509ffc726&scene=21#wechat_redirect)  
  
  
[【漏洞POC已公开】Linux RDS 爆出提权漏洞 一块内存放两次，root就到手了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486393&idx=1&sn=629bab1cc81fbee25e8e6dd558ee155b&scene=21#wechat_redirect)  
  
  
[Tenda光猫爆出严重漏洞，目前无补丁(POC已公布](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486388&idx=1&sn=eb210feb27b85439b53b10e595f7667e&scene=21#wechat_redirect)  
  
  
[13 行补丁，堵住Linux能拿 root 的内核洞(EXP已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486383&idx=1&sn=a87059fbbca283a7bd64ad78280ed00a&scene=21#wechat_redirect)  
  
  
[48 字节，换走一台 Windows 的 SYSTEM(CVE-2026-62735漏洞POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486378&idx=1&sn=f9e17fe32b6a3c67249fb4f037a3f1c6&scene=21#wechat_redirect)  
  
  
[【漏洞预警】linux删个网桥就能提权root？目前没有补丁(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486373&idx=1&sn=5f8bb9b0ff7c9fad75a8071c026b9f93&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Linux三年前补上的洞，又一次被挖开了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486368&idx=1&sn=337dc5fa148d0b24e7dbd352a71e184f&scene=21#wechat_redirect)  
  
  
[【情报】支付宝8.2亿数据泄露？给同事说一声，别自己吓自己](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486362&idx=1&sn=927fdf3d81c79a2ff8871086f0e78a8b&scene=21#wechat_redirect)  
  
  
[【漏洞预警】自己注册个号，就能搬空别人的 Magento 订单和支付卡（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486363&idx=1&sn=344a1f2e1a64b3b3be1509610edd2a0d&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Next.js 爆出新漏洞，不用登录实现命令执行(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486348&idx=1&sn=350cb257c3bfc3372e2dd50f2eb13738&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过预警】Redis CVE-2026-23479变种高危漏洞，EXP已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486299&idx=1&sn=7d1d8a95dad56cb021c2aa4ff549ee62&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过】超 30 万 WordPress 站点受影响，Forminator 文件上传漏洞（附POC）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486304&idx=1&sn=a22d6632366218b72fd68b3a5cb91557&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Exchange爆出RCE漏洞，利用Exchange 帮另一台 Exchange 开门（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486323&idx=1&sn=124dd54f863f64371f18c668d52e1fab&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用登录，一个请求就能让 GeoServer 乖乖执行命令（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486318&idx=1&sn=b478f775998b89579c53d4e0fbdc8c6e&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Keycloak不登录就能接管任意账户，双重验证也拦不住](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486309&idx=1&sn=d5c1a812f00448d397d3e3a8e78cdd45&scene=21#wechat_redirect)  
  
  
[【漏洞预警】 8月WebLogic公布了9个漏洞，有一个需要重点关注](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486294&idx=1&sn=eb26bc5f8fcafc9ab402b77acdb1b2a4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】近期MongoDB 公布了5个漏洞，最严重的一个可以不认证、远程就能拿下你电脑](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486289&idx=1&sn=10023bdd570d9f011eef242a23f83bb6&scene=21#wechat_redirect)  
  
  
[【0DAY漏洞】 全球5G手机芯片厂商之一‌的紫光展锐SOC存在RCE漏洞，影响数十家手机厂商，还未发布补丁](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486279&idx=1&sn=5e1452f6984bf3d4b4d35a2240676df5&scene=21#wechat_redirect)  
  
  
[【漏洞告警】安卓 14 到 16 全中招，通讯录 SQL 注入漏洞正在抄家（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486274&idx=1&sn=4bacf56f2045fca36b7b3de4be3ae591&scene=21#wechat_redirect)  
  
  
[【漏洞预警】紧急排查，SMB服务再曝新漏洞，利用脚本已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486251&idx=1&sn=633ed2b7d916bcf98404a7b3a9a8c4b4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用输密码，对方已经接管你的Mac：官方补丁公布macOS存在未鉴权RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486246&idx=1&sn=eaf0c51686e92e89826098fcf51fb959&scene=21#wechat_redirect)  
  
  
[【0day漏洞预警】MariaDB 13 爆 9.8 分 RCE 链：最低 USAGE 权限就能远程执行命令](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486225&idx=1&sn=bd13754fd9378e01a088cf996fa5b557&scene=21#wechat_redirect)  
  
  
[VMware 爆 5 个洞，两个 9.8 不用密码就能打穿 vCente](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486204&idx=1&sn=2590e283b0981541da74647ddc817282&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctTWicwicwovsVtMLAjYPfEB9lMRmJJiaIozWIuicwrbneXZ3pGly4lRNAd1WrP6AKYA925Iz4c1EbnMLCOqWzkCmdCLXDQibsG4VQw/640?wx_fmt=png&from=appmsg "")  
  
