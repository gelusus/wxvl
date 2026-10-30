#  接个电话就被黑？Telegram安卓漏洞潜伏四年，一通来电即可远程执行代码  
原创 网空闲话
                    网空闲话  网空闲话plus   2026-10-29 23:05  
  
2026年10月29日，安全初创公司NebuSec披露Telegram for Android中的一个高危漏洞：攻击者仅需一个普通Telegram账号，在受害者接听语音通话后即可在其手机上运行代码。该漏洞自2022年3月起潜伏四年，影响8.6.2至12.10.3之间的所有安卓版本。Telegram已于9月底发布的12.10.4版本中完成修复。  
  
# 一、漏洞概况：四年潜伏，一个电话即可触发  
  
据NebuSec联合创始人兼首席执行官向The Hacker News提供的细节，可触发漏洞的版本范围为8.6.2至12.10.3，这些版本发布于2022年3月至2026年9月之间。攻击者不需要受害者密码、不需要钓鱼链接，也不需要任何特殊权限——一个普通Telegram账号即可发起。NebuSec在基于ARM64的手机上通过私人通话确认了该漏洞，目前尚无证据表明它直接影响Telegram的iOS、桌面版或群组通话。截至10月29日，该漏洞没有CVE编号。  
  
Cybernews将这一漏洞概括为“四年未被发现”，并指出它存在于Telegram管理语音和视频通话的代码中。用户应确保安卓版Telegram升级至12.10.4或更高版本，版本号可在应用“设置”页面底部查看。  
# 二、触发机理：MediaState消息处理缺陷导致越界访问  
  
通话过程中，两个客户端互相发送短消息以建立连接并报告状态。其中一类消息名为MediaState，用于报告麦克风是否静音、摄像头是否开启以及视频如何旋转。  
  
NebuSec指出，漏洞出在读取MediaState消息的代码中：该代码未能正确处理缺失字段，访问了不应访问的内存区域。The Hacker News对Telegram 12.10.3公开发布源代码的审查证实了这一描述——当MediaState消息包含旋转值时，代码会检查该值，随后读取另一字段“视频状态”，即使消息中根本没有该字段。12.10.4版本则从正确的字段读取旋转值。  
  
Telegram的安卓、iOS和桌面应用共用同一个通话库tgcalls。2026年4月21日，tgcalls中一项标注为“Fix camera rotation”的更改做出了同样的修正。iOS源代码从5月发布的12.7版本起包含该更改，桌面版从7月发布的7.0.1版本起包含，而安卓版直到9月的12.10.4才获得修复。至于更早的代码在iOS和桌面版中是否可被利用，目前尚未得到证明。  
# 三、触发条件与证据局限：接听即中招，但利用路径未完全公开  
  
触发条件极为简单：受害者接听一通Telegram语音通话。NebuSec发布的演示视频显示，两部手机分别标记为攻击者和受害者，受害者接听来电约30秒后，手机切换至主屏幕，应用图标向下坠落，随后出现“HACKED BY NEBUSEC”标志。  
  
然而，证据链存在明显局限。The Hacker News指出，视频未显示受害者手机运行的Telegram或安卓版本；截至10月29日，NebuSec网站上没有该漏洞的技术分析报告，因此“从漏洞到运行代码”这一步目前依赖于视频和该公司单方面陈述。Cybernews同样指出，研究人员尚未公开记录该具体漏洞的精确利用路径，尽管内存处理错误有时可被利用来执行代码。The Hacker News已就“漏洞是否被用于真实攻击”及“攻击者代码在手机上能触及什么范围”向NebuSec和Telegram求证，截至报道发布相关答复尚未公开。  
  
NebuSec是一家获得Y Combinator支持的初创公司（YC S26）。该公司表示，其Code Security产品运行在OpenAI的GPT-5.4模型上，发现了这一漏洞。NebuSec在X平台上的推文以调侃口吻写道：“以前诈骗电话还只问你要密码，日子多好。现在接错一个电话就可能被黑。”推文同时确认修复版本为Telegram Android 12.10.4（7099），并戏称“我们把一个恐龙模型变成了网络安全 powerhouse”。今年7月，该公司曾描述一条安卓利用链，从Firefox漏洞出发，最终抵达运行Android 17的ARM64设备内核。  
# 四、修复节奏：安卓迟到数月，iOS与桌面版先行  
  
从修复时间线看，安卓用户承受了最长的暴露窗口。tgcalls的修正早在4月21日已完成，iOS和桌面版分别在5月和7月跟进，安卓版却等到9月底的12.10.4才获得修复。NebuSec于10月29日公开披露，此时距修复发布已约一个月。Telegram方面未就漏洞是否遭在野利用作出回应。  
# 五、关联风险：通话漏洞之外，贴纸库亦有隐患  
  
与通话漏洞并行的是另一条风险线。2026年10月15日，零日计划（ZDI）发布公告，指出Telegram应用用于绘制动态贴纸的库rlottie存在未修补漏洞，编号CVE-2026-111105，CVSS评分7.0。ZDI称，用户打开恶意文件或页面时，攻击者可借此运行代码。Telegram曾在3月告知ZDI其服务器端检查已缓解该漏洞，4月又要求ZDI提供完整的远程代码执行链。但研究员Saifeldeen Aziz在东京OffensiveCon演讲摘要中表示，这些检查可以被绕过。其研究利用AI引导的模糊测试，在Telegram iOS、安卓和桌面应用的rlottie中发现内存破坏漏洞，并将这些源于贴纸投递与检查缺陷的漏洞串联，在Telegram for iOS上实现了代码执行。  
  
根据Telegram公开发布的源代码，安卓应用在6月的12.8.1至8月的12.10.0之间用Rust编写的tlottie替换了rlottie；桌面版于9月完成同样替换；iOS代码在10月的13.0版本中加入tlottie，但同时仍包含rlottie。  
  
两起事件叠加，勾勒出Telegram客户端攻击面的一个共同特征：通话和贴纸这类高频、自动处理的功能模块，长期承载着内存安全风险，而修复在不同平台间的落地节奏并不一致。  
# 六、用户建议  
  
对安卓用户而言，最直接的行动是检查版本号并升级至12.10.4或更高。NebuSec的披露虽缺乏完整技术细节，但The Hacker News对源代码的独立审查为漏洞描述提供了佐证，修复版本的存在本身也构成有力旁证。  
  
更深层的问题在于证据透明度。一个被描述为“接听电话即可运行代码”的漏洞，其从内存错误到代码执行的完整链条尚未公开，真实攻击中是否被利用也无从确认。NebuSec以视频演示和推文宣传为主，技术细节留待后续；Telegram则保持沉默。在厂商和发现者都未提供完整图景的情况下，用户能做的是尽快升级，并对陌生来电保持基本警惕——即便在加密通讯应用内部，接听一通电话也并非毫无代价。  
  
参考文献  
1. Swati Khandelwal.Telegram Android Flaw Lets Attackers Run Code When Users Answer Voice Calls. The Hacker News, October 29, 2026.https://thehackernews.com/2026/10/telegram-android-flaw-lets-attackers.html  
  
1. Paulina Okunytė.Telegram warning: Hackers could run code through Android calls. Cybernews, October 29, 2026.https://cybernews.com/security/telegram-calls-vulnerability-android-breach/  
  
1. NebuSec (@nebusecurity). X post, October 29, 2026.https://x.com/nebusecurity  
  
1. Paulina Okunytė.Received a 4 a.m. call? Your Telegram may be hacked 5 minutes later. Cybernews, October 6, 2026 (Updated: October 13, 2026).  
https://cybernews.com/security/telegram-account-takeover-voicemail-hack/  
  
