#  Node2Shell：Node.js 修复严重的 RCE 漏洞  
Swati Khandelwal
                    Swati Khandelwal  代码卫士   2026-10-23 09:11  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**Imperva****公司的安全研究人员已演示证明，恶意****Brotli****数据可让运行官方****Node.js****构建的应用程序执行攻击者提供的代码。**  
  
他们的“  
Node2Shell”  
概念验证在官方  
 Node.js 24.22.0   
的  
 Linux arm64   
镜像上运行了代码，没有使用调试器或自定义构建，而是通过向捆绑的  
 Brotli   
解码器（  
CVE-2026-111497  
）输入特殊构造的数据实现。  
Node.js   
于  
10  
月  
21  
日披露该漏洞并发布修复，解码不受信任  
 Brotli   
数据的应用程序应进行更新。  
  
Imperva   
的测试并非每次都能成功，而是在  
 Linux arm64   
环境中执行代码尝试了多次。该结果只覆盖该构建和应用程序配置，其它架构以及  
 22.x   
和  
 26.x   
系列未进行测试。在修复版本  
 24.22.1   
上，相同数据被拒绝，没有代码运行。  
  
此次演示比首个公开概念验证更完善，后者仅在刻意削弱的测试构建上实现了代码执行。在标准构建上，该概念验证的作者报告的是崩溃和对进程内存的有限读取，而非代码执行，并表示未能在正常构建上实现可用攻击。  
  
截至  
10  
月  
22  
日，尚不存在该漏洞遭在野利用的报告。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfUs0HtSYTtG3WXibX6f7mt9iaNulANDSsxaFKO9nL67z2DwcqlmhSibTk3fwD6iayXYw9AGYKG0sAftWAlyspY5flZrmaf7GBY5lnQ/640?wx_fmt=gif&from=appmsg "")  
  
**受影响应用**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWxnmYbKlS4zYWY34SufZYEX8JibVEicKQUuZozThVFA0QTf1icZG7aC2WCjy2uKiart4CAtw8S6dOsnpibBxHiaX62CxUXkiaibNqWh6A/640?wx_fmt=gif&from=appmsg "")  
  
  
  
该漏洞存在于  
 Node.js   
捆绑的  
 Brotli   
解码器中，因此影响  
 22.x  
、  
24.x   
和  
 26.x   
系列上的应用程序，而不只是  
 Imperva   
测试的构建。服务可在  
 HTTP   
交换的两端到达该解码器：当它通过  
 Node.js   
内置的  
 fetch()   
读取  
 Brotli   
压缩响应时，或当中间件解码  
 Brotli   
压缩的请求体时。  
  
Express 5   
的  
 body   
解析器，例如  
 express.json()  
、  
express.urlencoded()  
、  
express.text()   
和  
 express.raw()   
应用于路由时会解码  
 Brotli   
请求体，与  
 gzip   
和  
 deflate   
并列。普通的  
 Node.js HTTP   
服务器不会仅因收到  
 Brotli   
请求体就对其进行解码，因此解码需要该中间件或显式调用。  
  
Imperva   
公司通过请求体和响应两条路径复现了代码执行，测试了固定版本的  
 Express 5.0.0   
至  
 5.3.0   
以及  
 body-parser 2.0.0   
至  
 2.3.0  
，并未测试它们之间的每个版本。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUHGnEqf2m0vHaoiaibGabftULdenRwKJ7cVtJkkgnZsUhMqhINKDjaFletH3Fmes3GyHJPaq8Jq03GQjG9qLYdSfNRlk216NDY4/640?wx_fmt=gif&from=appmsg "")  
  
**该漏洞如何运作**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfWicbLdoNUJ1jb0gjP6gptg1xh27XegP66gthGhMia0ytYfFh3t6qg6EFWprtfKeWthwQ3aMPGSZibooygIbU6chUtJ3dfmicibZgMA/640?wx_fmt=gif&from=appmsg "")  
  
  
  
Imperva   
将原因追溯到解码器如何处理  
 Brotli   
流中一个不可能出现的点：流在解码过程中会声明其将使用的块类型，而测试输入到达了一个流自身结构所排除的块变更。存在漏洞的解码器将其理解为请求更多输入，并在无效状态下继续运行。  
  
在一次使用内存清理器的单独测试中，从该状态继续运行写越过了预期缓冲区，这正是该漏洞背后的越界写入。  
Node.js   
通过导入上游  
 Brotli   
补丁修复了该漏洞，该补丁使解码器在该点以格式错误停止，而不是继续运行。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/t5z0xV2OYfXTgn739hBy6LPUzXGtzhunpttcKln2zqbIQtC2HBkkDPWsdrjjAnQBk2ZukIYT5666bXBNmomShKa3VJmktzmDmXjSSV24P7E/640?wx_fmt=gif&from=appmsg "")  
  
**应对措施**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfVWENOSiaayjrsfooxJ23rhpibOHoMqoeYAUUXdxE5mNuqz0YtdRPwcfpOD07icKNPazNHtcOfMDssqOWfrIKovSvgcvsBy5uHkws/640?wx_fmt=gif&from=appmsg "")  
  
  
  
迁移到已修复版本就是修复方案。更新应覆盖捆绑  
 Node   
二进制的容器镜像和其它工件，而软件包清单可能会遗漏这些内容。已结束生命周期的  
 20.x   
及更早系列不会获得修复，应迁移到受支持系列。  
  
该公告未列出  
 Node.js   
内部的任何变通方法，因此无法立即更新的应用在运行时中没有后备方案。  
Imperva   
公司的研究人员建议在可行的情况下限制受影响路径上不受信任的  
 Brotli   
流量，并表示只有更新才能消除该漏洞。该公司还在其自有  
 Web   
应用防火墙中添加了规则，并针对其复现的攻击进行了测试。  
  
负责为客户运行  
 Node.js   
的  
 Vercel   
公司表示，已于  
10  
月  
21  
日添加规则，在入站请求中阻止该攻击。这些规则未覆盖应用程序从其它服务器获取的  
 Brotli   
响应，也不能替代更新。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[MongoDB库中存在多个漏洞，可用于在Node.js服务器上实现RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247522334&idx=2&sn=474bb8a99d7a850a95b1ba847ff41044&scene=21#wechat_redirect)  
  
  
[Node.js 修复多个漏洞，可导致RCE和HTTP请求走私](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247512799&idx=1&sn=4c911268f2a7aa2fa486878c0d088a2b&scene=21#wechat_redirect)  
  
  
[GitLab 存在满分未认证RCE漏洞，影响12年来发布的所有版本](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527484&idx=1&sn=7537f27f49b677186d5e7c04cff03f8b&scene=21#wechat_redirect)  
  
  
[PHP 提醒注意两个未修复的严重 0day 漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527478&idx=1&sn=62774f44ed727a037c21a199384396a3&scene=21#wechat_redirect)  
  
  
[XSS2Shell：WordPress Core 漏洞利用预认证登录XSS实现 PHP RCE](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526848&idx=1&sn=b29b6dc57efb2687f969776b26ce49fe&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
https://thehackernews.com/2026/10/researchers-demonstrate-node2shell-rce.html  
  
  
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
  
