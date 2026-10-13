#  GitLab 紧急修复严重漏洞，可在服务器上实现任意代码执行  
Guru Baran
                    Guru Baran  代码卫士   2026-10-13 06:29  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Az5ZsrEic9ot90z9etZLlU7OTaPOdibteeibJMMmbwc29aJlDOmUicibIRoLdcuEQjtHQ2qjVtZBt0M5eVbYoQzlHiaw/640?wx_fmt=gif "")  
    
聚焦源代码安全，网罗国内外最新资讯！  
  
编译：代码卫士  
  
**GitLab****紧急修复了可导致攻击者在无需登录的情况下，在受影响服务器上执行任意代码的一个****CVSS****满分严重漏洞****(CVE-2026-97229)****，同时影响社区版和企业版。该公司已在****10****月****12****日发布补丁。**  
  
该漏洞与GitLab CI/CD 正则表达式编译器中的一个整数溢出漏洞有关。GitLab 表示，正则表达式验证不充分可导致未经身份认证的攻击者通过 CI/CD 配置中的一个特殊构造的表达式执行代码。正则表达式用于匹配或过滤数据的文本模式，在本案例中，处理恶意模式可攻陷 GitLab 服务器本身。  
  
GitLab 在安全公告中提到，在一定条件下该漏洞很可能遭利用。已发布的漏洞说明无法提供完整的工具序列或确切的所需条件。不过CVSS满分说明需要网络访问权限、攻击复杂度低、无需权限、无需用户交互。另外该评分还对机密性、完整性和可用性分配了高风险级别。  
  
该漏洞是一个服务器端的代码执行漏洞，而非在访客浏览器中运行的恶意 JavaScript。这一区别使得该漏洞对于在受影响 GitLab 安装上托管着开发项目的组织机构而言，更为严重。成功利用该漏洞可威胁服务器的数据和运营安全。GitLab 并未提到该漏洞是否已遭真实利用。  
  
该漏洞由安全研究员 joaxcar 通过 HackerOne 漏洞奖励计划报送。GitLab 在今年9月份还修复了多个严重的 CI/CD 正则表达式漏洞。这些漏洞的利用要求进行身份认证，而本文中所提到的新漏洞无需账号即可遭利用。GitLab 并未透露上述老旧漏洞和新漏洞之间的底层根因是否一致。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfU2cQAwDPQyibaLNywdnKfoXWMsd9sNlrQ14T8Za95aAewbe9WQgbV5r8jubeNkYOY0w4vyHkteLBYW5fOfVoTJ9NiaSFAWSBexE/640?wx_fmt=gif&from=appmsg "")  
  
**受影响版本和更新需求**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/t5z0xV2OYfUu1MpGBNyvDjQVYhcCR4bHBSPEMZCSt8ibricVdPWANvdic6vJmFD7S4w3aq0ic7Q0zgl77lQmoGK4NhEKdI8sSKw1PR7KbmMqKrc/640?wx_fmt=gif&from=appmsg "")  
  
  
  
受影响版本为 GitLab 社区版和企业版 19.2.8之前的19.2版本、19.3.4之前的19.3版本以及19.4.2之前的19.4版本。管理员应当立即升级受影响的自管理安装。GitLab.com 已运行已修复版本，GitLab Dedicated 客户无需采取任何措施。  
  
本次更新还修复了其它多个安全弱点，如GitLab Duo Agent 平台上的跨站脚本、授权失败以及与提示相关的漏洞。更早时候GitLab 安全更新还修复了多个 CI/CD 认证漏洞，因此管理员应当查看完整的补丁通知而非仅关注上头条的漏洞。  
  
GitLab 提醒称，19.3.4版本中还存在一个影响合并请求页面的已知漏洞，在启用 Ona 集成后可触发。GitLab 计划在19.3.5版本修复。管理员应当在升级过程中关注该问题且不能忽视本次紧急更新。GitLab 还在实例安全指南中提供了更宽泛的保护建议。  
  
  
代码卫士试用地址：https://sast.qianxin.com/  
  
开源卫士试用地址：https://oss.qianxin.com/  
  
  
  
  
  
  
  
  
  
**推荐阅读**  
  
[GitLab 邮件地址可用于供应链攻击](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527227&idx=2&sn=ebe8eb1c6273dd854494ce61cdcf3e57&scene=21#wechat_redirect)  
  
  
[GitLab：速修复这个满分路径遍历漏洞](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247527116&idx=1&sn=a5bb9e6f5885a23ace296375f350354b&scene=21#wechat_redirect)  
  
  
[GitLab GraphQL 严重漏洞可使未认证攻击者删除公开项目](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247526883&idx=1&sn=80604cef5784651f30751cec579a2172&scene=21#wechat_redirect)  
  
  
[GitLab 公开仓库暴露超过1.7万份机密信息](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524561&idx=2&sn=7ed0ade9d82089d6e02e5c90e36d90a4&scene=21#wechat_redirect)  
  
  
[GitLab 修复多个漏洞，可导致触发 DoS 条件](https://mp.weixin.qq.com/s?__biz=MzI2NTg4OTc5Nw==&mid=2247524251&idx=2&sn=4e764bf2d4ed304bf3f75a38df1f877d&scene=21#wechat_redirect)  
  
  
  
  
  
**原文链接**  
  
**https://cybersecuritynews.com/gitlab-vulnerability-code-execution/**  
  
  
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
  
