#  F5 紧急修复多个 NGINX 高危漏洞  
Sergiu Gatlan
                    Sergiu Gatlan  代码卫士   2026-10-20 08:08  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**F5****已发布带外安全补丁，以修复多个****NGINX Web****服务器漏洞，其中三个是严重级别，可用于在受影响的系统上执行代码。**  
  
这三个严重漏洞分别存在于 ngx_http_v3_module（CVE-2026-15176 和 CVE-2026-97205）以及 NGINX map 指令 (CVE-2026-93479) 中。未获权限的远程攻击者可利用这些漏洞发起拒绝服务攻击，或在采用非默认配置的 NGINX 系统上获得代码执行能力。  
  
成功利用这些漏洞会导致 NGINX 工作进程中出现基于堆的缓冲区溢出或释放后使用，从而引发崩溃或重启。不过，攻击者也可能“在地址空间布局随机化 (ASLR) 被禁用，或攻击者能够绕过 ASLR 的情况下执行代码”。  
  
F5 为受这两个漏洞影响的多个 NGINX 软件产品发布了补丁，包括 NGINX Plus、NGINX 开源版、NGINX Ingress Controller、NGINX Gateway Fabric 和 NGINX Instance Manager。  
  
如安全团队和管理员无法立即安装这些安全补丁，则可以通过不在 location 块中使用 alias 指令或 $request_filename 变量来缓解 CVE-2026-15176，并通过禁用异步文件 I/O (AIO) 或服务器端包含 (SSI) 来缓解 CVE-2026-97205。CVE-2026-93479 也可以通过使用命名捕获而非未命名捕获，并将其使用限制在带有正则匹配的块中来缓解。  
  
尽管 F5 尚未将这些安全问题标记为已遭在野利用，但近年来网络犯罪和国家背景的威胁组织经常利用 F5 产品中的漏洞。例如，攻击者曾利用 F5 安全漏洞入侵企业网络、探测内部服务器、劫持设备、部署数据擦除恶意软件，并从全球受害者处窃取敏感文件。自 2021 年 11 月以来，美国网络安全和基础设施安全局（CISA）已将八个 F5 漏洞标记为正在被积极利用，其中四个还被用于勒索软件攻击。一年前，即 2025 年 10 月，F5 还披露称，国家支持的威胁组织于 2025 年 8 月入侵了其系统，窃取了源代码以及有关未披露的 BIG-IP 安全漏洞的信息。  
  
F5 是一家财富 500 强科技公司，为超过 23000 家客户提供网络安全、应用交付网络 (AND) 等服务，客户包括 80% 的财富全球 500 强企业和 48 家财富50 强企业。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[CISA要求三天内修复这个严重的 F5 BIG-IP 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247525571&idx=1&sn=3596dc720ddc3cfdb3245a4b7597f249&scene=21#wechat_redirect)  
  
  
[F5 BIG-IP 源代码和 0day 漏洞数据遭泄露，补丁已发布](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524194&idx=1&sn=ab0b13f163b9168171ef5effd17379be&scene=21#wechat_redirect)  
  
  
[F5 BIG-IP 修复高危提权漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247521146&idx=1&sn=9082baa037a6e59fedb1cd685658ae1c&scene=21#wechat_redirect)  
  
  
[已存在15年之久的 NGINX 漏洞可导致RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526682&idx=1&sn=7610fee3a9f96ecb3c9e96f04db49188&scene=21#wechat_redirect)  
  
  
[NGINX Open Source 中存在两个可导致RCE的严重漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526334&idx=1&sn=e353058df9e3673f97b4cec3ed0de092&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://www.bleepingcomputer.com/news/security/f5-releases-out-of-band-patches-for-critical-nginx-vulnerabilities/  
  
  
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
  
