#  微软 Defender 0day：80% Windows 可提权到 SYSTEM、还能关杀软  
红队安全圈
                    红队安全圈  红队安全圈   2026-11-13 12:00  
  
30 秒看懂  
  
影响范围 ·   
Windows 10 / 11 / Server，凡 CLFS 认证机制未进入 enforcement 的机器（注册表 Mode 为 0 或 1），发布者估计占全球装量八成  
  
利用状态 ·   
公开可用 PoC，本地提权拿到 SYSTEM；暂无在野利用报告，但同一套链可指向任意杀软与 EDR  
  
补丁状态 ·   
未修  
，无 CVE、微软未发通告；先自查 Mode 值，别用关闭 mitigation 当解法  
  
这可能是今年最讽刺的一个 Windows 提权洞：微软专门为堵 CLFS 内存破坏加上的签名机制，本身成了提权入口，被用来开路的还是 Defender。  
  
## 01  事情是什么  
  
11 月 10 日，一个专发 Windows 0day 的研究者（ID 用过 MSNightmare、Nightmare-Eclipse、Chaotic Eclipse，真名 Abdelhamid Naceri）在 GitHub 放出了完整 PoC，起名 SignDrop。  
  
洞在   
clfs.sys  
，也就是 Common Log File System 的内核驱动里。本地标准用户跑一下 PoC，就拿到 SYSTEM shell，实测平台是 Windows 11 26H2、26H2 Experimental 和 Windows Server 2025 Experimental。  
  
没有 CVE 编号，微软也没发通告。这是他自己公开的，不是走完披露流程的那一类。  
  
## 02  为什么危险  
  
CLFS 这些年一直是内核提权的温床：它在内核态解析结构复杂的日志文件，解析出错就是内存破坏，直接换权限。微软从 2025 年 10 月 28 日起在正式通道推 CLFS authentication mitigation，分两段走——前 90 天 learning mode，之后进入 enforcement，要求每个日志文件都带有效认证码，认证码存在同名的   
.cnpf  
 文件里。  
  
漏洞就出在这个过渡设计上。  
  
创建和打开容器文件、连带生成认证文件，都在   
CclfsContainer::Create  
 里做。到这一步微软写得没问题：新建对象时指定了   
OBJ_FORCE_ACCESS_CHECK  
，文件建在调用进程自己的上下文里，权限检查一步不少。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DdHS2WVTUoEZ1VknA2GclENHJWSXPZvpllZiaE2DzXOUq9EyS3F8r2icibAqXvJloa0QmCrCeX2k0cicnmsQXyodl7fH5pQypPH9zibctC45pRrs/640?wx_fmt=jpeg "")  
  
问题在关联容器。一个日志文件可以再挂一个关联容器（  
AddLogContainer  
）。打开原始   
.blf  
 时，关联文件会被一起打开；如果关联文件是空的，CLFS 不管认证文件存不存在，都会给它签一次。  
  
而打开日志文件不需要任何特殊权限，只要路径写对。写成   
\CLFS\??\C:\test\t  
 就能打开   
t.blf  
，连带打开它关联的那个文件。  
  
于是攻击面就出来了：标准用户先把现场布置好——自己建一个   
.blf  
，关联一个空容器，DACL 设成自己可写——然后骗一个高权限进程去打开它。  
  
被挑中的是 Defender。调用   
ServerMpUpdateEngineSignature  
，配合篡改 Object Manager 里的符号链接，让 Defender 在自己的反恶意软件引擎上下文里打开这个 CLFS 容器。它顺着原始容器记录的路径去找关联文件，发现没签名，就用 SYSTEM 权限签一个。  
  
关键一步在最后：CLFS 创建这个   
.cnpf  
 时，用的是原始文件的 DACL。原始文件是攻击者建的，权限自然也是攻击者说了算。  
  
拿到一个 SYSTEM 权限创建、但攻击者可写的认证文件，接下来就是污染 CLFS 的日志解析，走内核内存破坏换权限。PoC 直接弹 SYSTEM shell。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/DdHS2WVTUoFOCSkaTwSxnxfnOgGBIuVPPbCT3oCBQjjn1dRYSqYp61FCkiay9eeBPZ3C3DcEtb3veoZz2gUiatSiaTHTDPxnRDOib1w4FACYxjk/640?wx_fmt=jpeg "")  
  
更麻烦的是这套链不挑产品。文件创建请求是从杀软自己的进程发出来的，在它内核驱动看来这就是自己人在干活，自保护、tamper protection 全部放行。所以同一套链也能指向任何杀软或 EDR，把它们关掉。  
  
这一点让 SignDrop 和普通提权洞彻底分开了：它不只是提权，它还能顺手拆掉防线。  
  
## 03  谁受影响  
  
凡是 CLFS 认证机制还没进入 enforcement 的 Windows 10 / 11 / Server 都在范围内，判断只看注册表   
Mode  
 值：0 或 1 受影响，2 或 3 不受影响。  
  
learning mode 有 90 天窗口，所以新装机、刚升级、镜像回滚、曾经手动关过这个机制的机器全在里面——八成这个估算就是从这来的。  
  
前置条件是攻击者已经能在机器上执行代码。这是本地提权，不是远程打进来，别把它当成"边界被打穿"级别的新闻，但它在内网横向的每一跳上都有用。  
  
## 04  怎么修  
  
先查 Mode 值，三行 PowerShell 就够：  
  
$p = "HKLM:\SYSTEM"  
  
$p += "\CurrentControlSet\Services"  
  
$p += "\CLFS\Authentication"  
  
(Get-ItemProperty $p).Mode  
  
一、结果是 0 或 1，按业务窗口评估把认证机制切到 enforcement，切完重启再验一次值。  
  
二、不要把关闭 mitigation 当修复方案。关掉能消掉 SignDrop 这条路，但 CLFS 老的内存破坏洞会全部复活，那是更大的坑。  
  
三、微软补丁出来之前，重点收本地代码执行面：钓鱼附件、宏、脚本宿主、能落盘的下载器。  
  
四、检测盯三样：用户目录或临时目录里成对出现的 .blf 与 .cnpf；Defender 进程在非系统路径创建 .cnpf；Object Manager 里指向   
\CLFS  
 的异常符号链接。  
  
## 05  一句判断  
  
本地提权洞单看不算致命，但它出现在内网横向的每一跳上。一个能顺手关掉 EDR 的本地提权，等于把对手的初始访问直接升级成无人看守。手上 Windows 资产多的，今晚先把 Mode 值查一遍，不费时间。  
  
## 06  POC 链接  
  
https://github.com/davincee/signdrop  
  
仓库里带 PoC 源码、注册表自查方法（Mode 0/1 受影响，2/3 不受影响），以及原作者的完整技术说明。仅用于自查与复现自己的环境。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
