#  Upload2Shell漏洞曝光，WordPress、邮件及打印服务器均受影响  
 FreeBuf   2026-11-22 10:48  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX0JzhE9uo6lIaAabVqHXM3UlVy6AsIqO2ERfK6otRtkmhibQ4F9icyLY6rJawQKNiciaJKK5ohgCnNvg1DlYHfbQ5iapmCQnKgdtr1Y/640?wx_fmt=gif "")  
  
  
![Upload2Shell漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX0ZuWicibvSGeKAgFzMI1dld4TFWrbwicBajRcZricbClS8HPNUldict4mY9CF1rwYZeRaicgkv7fFLPKUyzJLJytbwyGIniaicPO9FulM/640?wx_fmt=png "")  
  
  
Ghostscript存在一处安全漏洞，研究人员可通过常规文件处理流程，在WordPress、Roundcube网页邮件、Linux打印服务器上执行命令。该漏洞名为Upload2Shell，编号CVE-2026-125396，影响10.08.0版本之前的上游Ghostscript发行版。  
  
  
漏洞位于Ghostscript的Type 1字体加载器中，但受影响服务器无需直接对外暴露Ghostscript服务。服务器自带的预览、缩略图生成、打印功能，会自动将待处理文件传递给Ghostscript。  
  
  
CVE公开记录显示，该漏洞CVSS 4.0评分为8.5，标注的攻击向量为本地。这一评分针对的是解释器层面的漏洞本身，并未覆盖文档传递到解释器的完整路径。  
  
  
安全研究人员Paulos Yibelo与pwn.ai团队在11月17日发布的研究成果中证实，面向网络开放的服务可将远程提交的文件传递给存在漏洞的代码段。本次报告仅描述研究场景下的演示验证，尚未确认存在在野攻击。  
  
  
Part  
01  
  
Upload2Shell利用内存漏洞  
  
篡改校验逻辑  
  
  
Ghostscript主要用于渲染PDF与PostScript文档。PostScript是专门用于描述页面内容的编程语言，因此渲染文件的过程本质上是执行文件内携带的指令。  
  
  
![Ghostscript渲染流程](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX3E0mlwwzwDMFMLT07ibniciaVID5o2cDmiaoUormgVzqn0m0dnPZVhfhEz12PkHaD9ictmrvfeIceg52stT6Pb3y1a9hzKHPopgpR0/640?wx_fmt=png "")  
  
  
Ghostscript的-dSAFER模式原本用于限制文件指令的访问范围。Upload2Shell通过一处内存释放后重用漏洞突破了这些限制——这类漏洞的成因是软件释放某块内存后，仍保留了指向该内存区域数据的引用。  
  
  
该字体加载器在执行可能触发错误的操作前，未校验接收数据的类型。这一缺陷会暴露受保护的写入权限，而文档内的代码本不应拥有该权限的控制权。  
  
  
研究人员利用这一内存漏洞，在-dSAFER模式仍启用的状态下篡改了Ghostscript的安全校验逻辑。测试过程中，执行的命令仅拥有渲染器的对应权限，不会自动获得root权限。Cyber Security News此前报道另一款Ghostscript PDF漏洞时曾提到，这类权限范围决定了攻击者获得代码执行能力后，可访问的资源边界。  
  
  
Part  
02  
  
三类主流服务受漏洞波及  
  
  
在WordPress场景中，研究人员通过使用Imagick与ImageMagick的站点媒体预览流程，将恶意文件传递到Ghostscript。WordPress安全公告将核心上传路径问题单独编号为CVE-2026-65640。利用该路径需要媒体上传权限，默认情况下作者及更高权限的角色拥有该权限。  
  
  
此前WordPress Imagick漏洞相关报道曾提到，WordPress已在7.0.4版本修复该攻击路径，并将补丁向后移植到包括6.9.7在内的旧版本。研究人员还发现，在测试配置下，Forminator、Elementor Essential Addons中的部分公开上传功能，无需登录即可将文件传递给渲染器。  
  
  
在1.7.2版本的Roundcube中，发件人无需拥有网页邮件账号即可发起攻击。收件人只需查看邮件，无需Click或下载附件即可触发漏洞。Roundcube服务器会自动生成邮件缩略图，通过ImageMagick将附件传递给Ghostscript处理。该测试场景依赖允许上述传递流程的ImageMagick安全策略。命令以web服务账号www-data的权限执行，不会运行在收件人本地设备上。  
  
  
打印服务器场景下无需收件人执行任何操作。在Debian 13系统中，共享CUPS打印队列无需凭据即可接收网络打印任务，并将测试文档发送给Ghostscript做格式转换。研究人员通过以打印服务账号lp的身份写入标记文件，证实了命令执行的可行性。  
  
  
测试显示，开启身份认证可阻止匿名提交打印任务，但无法修复底层漏洞。上述三个场景只是触达同一存在漏洞的渲染器的不同路径，并非Ghostscript存在三个独立漏洞。  
  
  
Artifex已在Ghostscript 10.08.0版本中修复了底层漏洞。Linux发行版可将同一补丁应用到旧版本，因此仅通过上游版本号无法判断系统是否存在风险。例如Debian 13就已在10.05.1~dfsg-1+deb13u2版本中首次推送了该修复补丁。  
  
  
管理员应核查厂商针对CVE-2026-125396的软件包更新状态，逐一确认每个服务调用的Ghostscript版本。更新主机系统不会替换现有容器或应用程序包内内置的Ghostscript副本。  
  
  
WordPress运营方应同时应用核心程序与渲染器的更新，核查所有会将上传文件送入媒体库的公开表单。如果暂时无法安装补丁，应在可行范围内暂停不可信文件的自动预览功能。  
  
  
针对打印服务器，应为共享打印队列开启身份认证，限制队列的网络访问范围。保持渲染器进程隔离，仅为其服务账号分配必要的访问权限。  
  
  
最关键的核查节点在文件被接收之后。研究人员的测试显示，仅存储文件不会启动Ghostscript，只有执行渲染操作时才会触发漏洞。管理员需要追踪实际生成预览、缩略图或打印输出的工作进程，确认该进程调用的是已打补丁的软件包。  
  
  
参考来源：  
  
Hackers Could Execute Commands on WordPress, Email and Print Servers Through Upload2Shell Flaw  
  
https://cybersecuritynews.com/upload2shell-flaw/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
