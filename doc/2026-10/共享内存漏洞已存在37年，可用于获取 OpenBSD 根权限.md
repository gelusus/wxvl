#  共享内存漏洞已存在37年，可用于获取 OpenBSD 根权限  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-10-19 11:59  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**网络安全公司****Calif****发现，组合利用已存在****37****年之久的一个整数溢出漏洞与另一个独立的内核漏洞，可实现提权，在****OpenBSD 7.9****上获得****root****权限，绕过****OpenBSD****的****KARL****内核地址随机化及其****SMAP****和****SMEP****硬件保护。****OpenBSD****在****9****月份予以修复。**  
  
shmat(4)  
附加函数中的整数溢出漏洞已在勘误  
018  
中修复，并于  
9  
月  
14  
日作为可靠性修复发布。完成攻击链的引用计数释放后使用问题则单独在勘误  
026  
中修复并于  
9  
月  
30  
日作为安全修复发布。两者均可通过  
syspatch  
获取，并覆盖所有架构。  
OpenBSD 7.8  
也获得了整数溢出修复方案。该整数溢出是完整攻击链的入口点，被归类为可靠性修复，而不是安全修复。  
  
截至发布时，两个漏洞均未分配  
CVE  
。未有证据表明这些漏洞已被用于攻击。  
  
共享内存附加计数器  
shm_nattch  
在可追溯到  
1988  
年的原始  
BSD  
源码中被定义为有符号  
16  
位  
short  
，最大值为  
32767  
。  
OpenBSD  
继承了该类型，且从未将其加宽。  
  
该漏洞利用映射  
127  
个共享内存段并派生  
259  
个进程，使计数器超过最大值并回绕到  
−32643  
。随后竞争条件触发释放后使用：被破坏的引用计数导致内核释放其它进程仍在使用的内存。  
  
另一个越界写覆盖内核中的多播成员计数器。当受影响的套接字关闭时，由此产生的递减会修改页表项，将用户内存映射切换为监管者权限可访问，从而绕过  
SMAP  
和  
SMEP  
。然后该利用覆盖可预测内核地址处的函数指针，运行自己的代码。  
  
研究员  
Andrew Griffiths  
写道，“尽管  
OpenBSD  
的  
KARL  
会重新链接内核二进制以随机化函数和数据地址，但内核内存分配器仍是确定性的。”启动时分配器和最佳适配分配器都缺少基址随机化，因此早期启动结构，包括  
lo0  
和  
enc0  
网络接口处理程序中的函数指针，在同一硬件上多次重启后落在相同地址。该利用无需信息泄露漏洞便可实施。  
  
除两个勘误外，  
Calif  
还提出四类加固措施：内核链表的安全解除链接（  
FreeBSD  
和  
NetBSD  
已使用）、页表自引用映射随机化、内核分配器随机化以及基于编译器的边界检查。  
  
Calif  
表示其  
“  
今年早些时候  
”  
向  
OpenBSD  
报告了这些漏洞，并在  
GitHub  
平台发布了  
 PoC   
利用代码。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[OpenBSD 修复已存在27年的 PPP 协议栈认证绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526319&idx=2&sn=dd48e102a2751f99b4ea33b644ef79a5&scene=21#wechat_redirect)  
  
  
[开源操作系统 OpenBSD 被曝四个严重的认证绕过和提权漏洞（详情）](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247491821&idx=1&sn=99481c872b8bb5bb526699225619cab3&scene=21#wechat_redirect)  
  
  
[DirtyClone: Linux 内核新漏洞，通过克隆的数据包获得根权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526443&idx=2&sn=c847b004b20a9d1f991dad8dcd81c7b4&scene=21#wechat_redirect)  
  
  
[Pedit COW：通过投毒缓存二进制，获得根权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526443&idx=1&sn=0248c2f40eb99f883bab4a8b30dba2bf&scene=21#wechat_redirect)  
  
  
[Ubunntu 高危漏洞可导致攻击者获得根权限](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525508&idx=1&sn=2833d5727c1faf94104890b325bb6d75&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/researchers-demonstrate-openbsd-root.html  
  
  
题图：Pixa  
b  
ay Licens  
e  
  
  
**本文由奇安信编译，不代表奇安信观点。转载请注明“转自奇安信代码卫士 https://codesafe.qianxin.com”。**  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSf7nNLWrJL6dkJp7RB8Kl4zxU9ibnQjuvo4VoZ5ic9Q91K3WshWzqEybcroVEOQpgYfx1uYgwJhlFQ/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/oBANLWYScMSN5sfviaCuvYQccJZlrr64sRlvcbdWjDic9mPQ8mBBFDCKP6VibiaNE1kDVuoIOiaIVRoTjSsSftGC8gw/640?wx_fmt=jpeg "")  
  
**奇安信代码卫士 (codesafe)**  
  
国内首个专注于软件开发安全的产品线。  
  
   ![](https://mmbiz.qpic.cn/mmbiz_gif/oBANLWYScMQ5iciaeKS21icDIWSVd0M9zEhicFK0rbCJOrgpc09iaH6nvqvsIdckDfxH2K4tu9CvPJgSf7XhGHJwVyQ/640?wx_fmt=gif "")  
![]( "")  
![]( "")  
  
   
觉得不错，就点个 “  
在看  
” 或 "  
赞  
”   
  
