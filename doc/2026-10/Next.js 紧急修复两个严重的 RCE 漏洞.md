#  Next.js 紧急修复两个严重的 RCE 漏洞  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-10-16 09:16  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**10****月****14****日，****Next.js****紧急修复了两个可导致未认证远程代码执行的严重漏洞以及一个高危拒绝服务漏洞。**  
  
  
  
其中  
RCE  
漏洞存在于  
 React Server Components   
中，另一个位于  
 librsvg   
中，后者是  
 Next.js   
通过  
 sharp   
包使用的开源库，用于处理  
 SVG   
图像。修复版本为  
 Next.js 16.4.1   
和  
 15.5.28  
，分别是当前的活跃和维护长期支持版本。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUSuZSuWUzoiabaHw3nQGsKcLCIsmaeEicXZMduYicsY5OESia3eVfiaHc0vSttOCJK09UN3ogZ03UX8hrZWaXse3d7yePDKWZ9EkkU/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞概述**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUd1MMv7KAL3h8I6pNH9sIXsvdicS3Ortg2fBB8os0TpcU5wO3aWSo1nuJRTqic1KIJfy2Qlmow7x24ccEwh6szZE2QEibKD4rvVE/640?wx_fmt=gif&from=appmsg "")  
  
  
本次更新涵盖三个独立漏洞。  
Next.js   
的开发公司  
 Vercel   
将两个代码执行漏洞都评为“严重”级别，  
CVSS   
评分均为  
 9.5  
。其中两个漏洞目前无  
CVE  
编号。  
  
（1）  
通过  
 use cache   
的代码执行（  
GHSA-673q-mjm6-r5xr  
）漏洞，影响版本  
16.0.0   
至  
 16.4.0  
，已在  
16.4.1   
版本中修复，影响启用了  
 Cache Components   
的  
 App Router   
应用，其中  
use cache   
函数返回多个类型化数组。  
  
（2）  
通过  
 SVG   
图像的代码执行漏洞（  
CVE-2026-96889  
，  
GHSA-2464-fhr4-w9xw  
），受影响版本为  
10.0.0   
至  
 15.5.27  
，以及  
 16.0.0   
至  
 16.4.0  
，修复版本为  
16.4.1  
，或  
 15.5.28   
搭配  
 sharp 0.35.5   
或更高版本。影响在基于  
 glibc   
的  
 Linux   
上将攻击者提供的  
 SVG   
文件传给  
 sharp   
的应用。  
  
（3）  
通过  
 Server Actions   
的拒绝服务漏洞（  
GHSA-p3m5-59mp-jv5r  
），影响版本为  
13.0.0   
至  
 15.5.27  
，以及  
 16.0.0   
至  
 16.4.0  
，修复版本为  
15.5.28  
、  
16.4.1  
，影响至少定义了一个  
 Server Action   
的  
 App Router   
应用。  
  
具体而言，第一个代码执行漏洞存在于  
 React Server Components   
中，这是  
 React   
在服务器上运行用户界面组件的功能。  
React   
的公告涵盖三个包：  
react-server-dom-webpack  
、  
react-server-dom-parcel   
和  
 react-server-dom-turbopack  
，版本为  
 19.0.0   
至  
 19.3.0  
。公告将该漏洞描述为影响  
“  
响应传输多个类型化数组且存在攻击者控制值的极其罕见情况  
”  
。  
  
在  
 Next.js   
应用中，触发条件是  
 use cache   
函数返回多个类型化数组，因此只有启用了  
 Cache Components  
（  
use cache   
背后的功能）的版本  
 16   
应用会受到影响。  
Vercel   
表示，即使用户的应用没有这样的函数，也应升级。  
  
第二个漏洞是位于  
 librsvg   
中的内存缺陷，该库用于渲染  
 SVG   
图像，  
Next.js   
通过捆绑的  
 sharp   
包使用它。公告提到，当  
 Next.js   
运行在基于  
 glibc   
的  
 Linux   
上时，可能发生代码执行。默认情况下，  
Next.js   
不会优化  
 SVG   
文件。通过图像优化器提供这些文件需要开启名为  
 dangerouslyAllowSVG   
的设置。当  
 next/og  
（用于生成社交预览图像的功能）在  
 Node.js   
运行时上运行时，同一个库也会为其渲染图像。  
  
第三个漏洞是拒绝服务问题。在至少定义了一个  
 Server Action   
的  
 App Router   
应用中，精心构造的请求可以使一个服务器进程的  
 CPU   
占用率达到满负荷，并使它停止处理其他请求。使用  
 Pages Router   
或没有  
 Server Actions   
的应用不受影响。  
  
不同主机的暴露情况可能不同。  
Netlify   
表示其平台不运行图像优化器代码路径，这些漏洞对其设置影响有限，但仍建议升级。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVCKkicy08wRWhYyWQj2fxMiaqrhrukFnnl6EbTRGb93XCmZKicXTllB7fwican6pjxdVJvIjWJOZ03iabvBzktJrHQAhcib8h8BebNU/640?wx_fmt=gif&from=appmsg "")  
  
**应对措施**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVRpCPiaQCwicgvfqRic0aUzFDUOMVK2WovnP9dPOUh2ckl4W6BYsURfNNSm4TWCYa7coOK8TXOuZmBIic0Pq9jLYibLwz1qH0EFic1s/640?wx_fmt=gif&from=appmsg "")  
  
  
  
如果用户运行的是版本  
 16  
，应升级到  
 Next.js 16.4.1  
；如果运行的是版本  
 15  
，应升级到  
 15.5.28  
，然后重新构建并重新部署。  
  
Vercel   
在公告中表示，升级  
 Next.js   
后包管理器会自动提供修复后的  
 sharp  
。在版本  
 16   
上确实如此，因为  
 16.4.1   
要求  
 sharp 0.35.5  
。但在版本  
 15   
上可能不会。  
15.5.28   
仍然接受较旧的、受影响的  
 sharp 0.34.x  
，在现有安装上升级可能会保留该版本。使用  
 npm ls sharp   
检查，如果低于  
 0.35.5  
，则更新  
 sharp   
到  
 0.35.5   
或更高版本。  
  
如果无法立即升级  
 Next.js  
，针对该漏洞有一个临时方案：确保  
 sharp   
为  
 0.35.5   
或更高版本，或者按照  
 sharp   
公告所述，使用  
 sharp.block({ operation: ["VipsForeignLoadSvg"] })   
阻止  
 sharp   
中的  
 SVG   
渲染。  
use cache   
和  
 Server Actions   
漏洞没有变通方法，只有升级才能修复。  
  
截至  
 10   
月  
 15   
日，  
use cache   
和  
 Server Actions   
公告没有  
 CVE   
编号，也未出现在  
 npm audit   
使用的  
 npm   
公告源中，因此依赖扫描器可能不会提醒这些问题的存在。图像漏洞则会通过  
 sharp   
自己的公告显示出来。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfX2pnwAVTaefo8TypVZSUko4fMVlrcb3wVqjibh0SfqNSB5z1tbEJ1TnK6fGMfuxJn47c4sFlqELAnXYibJu5RQic9AcJ5jhnnqL4/640?wx_fmt=gif&from=appmsg "")  
  
**为什么图像漏洞反复出现**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWWpJGc3ickeDo6EGibFkWgMgvG03EnxnQlicqsjeQWhTaS8YZX2qfibyEeN05ls2QiafY96DdMZbjTDsoOCuWvehGG4FbEmp7AL9yY/640?wx_fmt=gif&from=appmsg "")  
  
  
  
这是自  
 8   
月以来  
 Next.js   
中的图像处理第三次出现代码执行漏洞。  
8   
月，通过  
 sharp   
触及的  
 libheif   
中的一个缺陷允许在优化器处理精心构造的  
 AVIF   
图像时执行代码。  
9   
月，  
next/og   
中的一个缺陷让攻击者控制的文本可以更改其生成的  
 SVG  
。  
  
Next.js   
本次没有更改自己的图像代码。  
16.4.1   
搭载的优化器与  
 16.4.0   
相同，修复图像漏洞的方式是将捆绑的  
 sharp   
提升到  
 0.35.5  
，该版本包含已修复的  
 librsvg   
版本（  
2.63.2  
）。  
  
librsvg   
漏洞  
 CVE-2026-96889   
于  
 9   
月  
 23   
日公开，并于  
 9   
月  
 27   
日在  
 sharp   
中修复。  
Red Hat   
将其评为  
 7.8   
分，并表示利用需要用户交互，例如打开恶意文件。  
Vercel   
将  
 Next.js   
的暴露评为  
 9.5   
分，并可通过网络触达。报告  
 librsvg   
漏洞的研究员  
 Rafael B. Dias   
表示，这是  
 9   
月  
 next/og   
漏洞背后的弱点。针对那个较早漏洞的一个可用漏洞利用程序，专为某种特定服务器设置构建，此后已被公开。  
  
sharp   
的维护者还提醒说，该漏洞最终导致代码执行还是仅导致崩溃，可能取决于服务器的  
 Node.js   
构建，因为官方  
 Node.js   
二进制文件没有针对这类内存攻击进行加固。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUmarRU93mH5eesiaJ9tCgKhickd5RhDLHYjHaOzaqvP8iaynm6VyWICibxLLlcBx881tOBt93LCb7bicm0aCFozuiaQZpDNq8xMpCm0/640?wx_fmt=gif&from=appmsg "")  
  
**与 React2Shell 不同**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfWNbTOiaDM0WzakpOR3crM0qzdsX7NZsS5KaGib7LnZ6RQScf9YGasx8TLR3jmiajO7NicJhicqyW8nTVia9zNsQ9bjzmpdSV3lIfwic0/640?wx_fmt=gif&from=appmsg "")  
  
  
  
React Server Components   
漏洞与  
 React2Shell   
位于相同的三个包中，后者是  
 2025   
年  
 12   
月披露的最高严重性漏洞，曾遭攻击者大规模利用，而这个漏洞的范围要窄得多。  
React   
将其描述为影响  
“  
响应传输多个类型化数组且存在攻击者控制值的极其罕见情况  
”  
。  
  
截至  
 10   
月  
 15   
日，无公开报告表明  
 10   
月  
 14   
日修复的这三个漏洞遭攻击利用。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[速修复！React满分漏洞同时影响 Next.js，可导致未认证RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524584&idx=2&sn=d601040ac12c82888e8edba633d2fc02&scene=21#wechat_redirect)  
  
  
[速修复Next.js中严重的授权绕过漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522568&idx=1&sn=6c208cae9bd5b7c538ed45a6a6cd2308&scene=21#wechat_redirect)  
  
  
[Palo Alto：PAN-OS RCE 漏洞已遭利用](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527392&idx=1&sn=44cddd7e186a3bcdca81b7ae6690583f&scene=21#wechat_redirect)  
  
  
[六天，研究员利用AI模型构建 Chrome 完整利用链，获赏$25万](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527392&idx=2&sn=324df0a8612a62b14cb09f554273fef6&scene=21#wechat_redirect)  
  
  
[Citrix：这个严重的 NetScaler SAML 漏洞可导致 RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527286&idx=2&sn=3083d12c9537dcc1cdaf2f329457dc89&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/nextjs-ships-out-of-band-patch-for-two.html  
  
  
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
  
