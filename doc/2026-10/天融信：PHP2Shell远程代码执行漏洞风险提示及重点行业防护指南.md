#  天融信：PHP2Shell远程代码执行漏洞风险提示及重点行业防护指南  
 天融信   2026-10-22 09:32  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/nJmicCz2NYxNibMqIOfXMnZxbVBPBGKu3pficMjqFslyVdhUYhSozJ0egjyKoezIaK9qEyYy6ttzMv3T5Kiasiae7icg/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
![]( "")  
  
2026年10月21日，PHP官方发布安全更新，修复CVE-2026-109618与CVE-2026-109619两个严重远程代码执行漏洞。该组漏洞被安全社区命名为PHP2Shell，CVSS v4.0评分9.2。漏洞可能影响PHP 5及之后的所有版本。  
  
  
详情及分析可点击“【漏洞复现】天融信关于PHP远程代码执行漏洞CVE-2026-109618/CVE-2026-109619(PHP2Shell)的风险提示”查看。  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg3MDAzMDQxNw==&mid=2247497082&idx=1&sn=47ea0886e806ca0cc3553a9bcec601ef&scene=21&click_id=1895198866#wechat_redirect)  
  
  
值得关注的是，攻击者利用该漏洞在特定条件下无需身份认证即可远程代码执行。搭载受影响PHP的网站、Web业务存在被入侵、植入木马、数据泄露、网页篡改风险，一旦被利用，直接造成业务停摆、敏感信息失窃。目前，大量CMS、政企与互联网Web系统均基于PHP搭建，或许  
会成为全网批量扫描攻击重点。  
  
  
天融信安全研究团队在漏洞信息公开后迅速启动应急响应，目前天融信下一代防火墙系统、Web应用防火墙系统、  
入侵防御系统、  
脆弱性扫描与管理系统、容器安全检测系统等产品已支持对该漏洞攻击的检测和防护。同时，天融信AI MSS安全托管运营服务的云端平台与各组件规则库同源并已实时同步更新，客户侧无需做任何升级。  
  
  
<table><tbody><tr><td data-colwidth="198"><section style="text-align: center;" data-review-block-id="arb_mvjad9ix_xg2tdm90"><span leaf=""><span textstyle="" style="font-weight: bold;">产品名称</span></span></section></td><td data-colwidth="376"><section style="text-align: center;" data-review-block-id="arb_mvjad9ix_q9dl9eir"><span leaf=""><span textstyle="" style="font-weight: bold;">系统/规则库版本</span></span></section></td></tr><tr><td data-colwidth="198"><section data-review-block-id="arb_mvjad9ix_lcvj2g7g"><span leaf="">天融信下一代防火墙系统</span></section></td><td data-colwidth="376"><p><span leaf="">攻击检测规则库：ips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p><span leaf="">天融信Web应用防火墙系统</span></p></td><td data-colwidth="376"><p><span leaf="">攻击检测规则库：waf-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><section data-review-block-id="arb_mvjad9ix_jrd73z1i"><span leaf="">天融信应用安全网关系统</span></section></td><td data-colwidth="376"><p><span leaf="">攻击检测规则库：ips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_x5oqvrf3"><span leaf="">天融信入侵防御系统</span></p></td><td data-colwidth="376"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_7gdhka6p"><span leaf="">攻击检测规则库：ngips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_l2ubo9zf"><span leaf="">天融信入侵检测系统</span></p></td><td data-colwidth="376"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_qgzrzbge"><span leaf="">攻击检测规则库：ngips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_cnv6o5t5"><span leaf="">天融信APT安全监测系统</span></p></td><td data-colwidth="376"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_juiq7amz"><span leaf="">攻击检测规则库：ngips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_llt18b3q"><span leaf="">天融信僵尸网络木马和蠕虫监测与处置系统</span></p></td><td data-colwidth="376"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_4qvhmxq5"><span leaf="">攻击检测规则库：ngips-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><section data-review-block-id="arb_mvjad9ix_3ne73fgw"><span leaf="">天融信火焰威胁检测系统</span></section></td><td data-colwidth="376"><section style="text-align: left;" data-review-block-id="arb_mvjad9ix_xdpep6gn"><span leaf="">攻击检测规则库：</span><span leaf="">ips-v2026.10.22.001</span></section></td></tr><tr><td data-colwidth="198"><section style="font-size: 15px;color: rgb(89, 87, 87);letter-spacing: 1px;line-height: 1.75;box-sizing: border-box;font-style: normal;font-weight: 400;text-align: justify;text-autospace: normal;" data-review-block-id="arb_mvjad9ix_z1ksmree"><span leaf="">天融信脆弱性扫描与管理系统</span></section></td><td data-colwidth="376"><p><span leaf="">系统漏洞规则库：vas-sys-V1.1-2026.10.22、vas-sys-v1.0-2026.10.22  </span></p><p><span leaf="">web漏洞规则库：vas-web-v1.1-2026.10.22、vas-web-v2026.10.22</span></p></td></tr><tr><td data-colwidth="198"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_7wfqnxxt"><span leaf="">天融信容器安全检测系统</span></p></td><td data-colwidth="376"><p style="white-space: normal;margin: 0px;padding: 0px;box-sizing: border-box;" data-review-block-id="arb_mvjad9ix_0awvt0f3"><span leaf="">漏洞规则库：vuln-v2026.10.21</span></p></td></tr></tbody></table>  
  
  
该组漏洞实际影响范围远超一般Web应用漏洞，以下客户和场景请优先关注  
：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dSWSuPicfjTeo9BYtpPkAVsviavIsOoJA2YaUiabQVsz4ibQkpgH6XG3Jgvia637rDMWUdvf2WmS2uibYO48ibDdibS86DVy0fn6o1u76IM4nK24UPg/640?wx_fmt=png&from=appmsg "")  
  
**重点行业**  
  
金融、电信、政务等行业的在线业务系统大量采用PHP技术栈，公网服务暴露面广，对业务连续性和数据安全的要求极高。一旦被利用，可能导致核心业务中断和敏感信息外泄。建议相关行业客户立即排查PHP资产并尽快完成修复，在补丁部署前通过安全设备提供过渡防护。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/dSWSuPicfjTeo9BYtpPkAVsviavIsOoJA2YaUiabQVsz4ibQkpgH6XG3Jgvia637rDMWUdvf2WmS2uibYO48ibDdibS86DVy0fn6o1u76IM4nK24UPg/640?wx_fmt=png&from=appmsg "")  
  
**重点场景**  
  
1.  
 **公网直接暴露的PHP服务：**  
面向互联网开放的Web站点、API接口、管理后台等，攻击者可直接触达，是最高优先级处置对象；  
  
  
2. **基于PHP的CMS系统：**  
WordPress、Drupal、Joomla等内容管理平台部署量庞大，且常因插件众多导致版本更新滞后，易成为批量攻击目标；  
  
  
3. **容器化部署环境：**  
Docker、K8s环境中的PHP-FPM镜像可能存在版本滞后，需通过实际执行php-v确认版本，不能仅依赖镜像标签判断；  
  
  
4. **多租户与共享主机：**  
PHP运行版本由服务商统一管理，租户无法自行升级，需推动服务商及时更新；  
  
  
5. **已停止维护的旧版本：**  
PHP 8.1及更早版本不再获得官方安全更新，存在该漏洞但无法通过补丁修复，应尽快规划迁移至受支持版本。  
  
  
此次PHP系列漏洞事件，折射出AI深度重塑网络安全攻防格局的趋势。随着AI挖掘漏洞的能力持续增强，软件安全防御必须向开发、验证、运行等更早、更深的环节延伸，以AI对抗AI。在漏洞发现门槛持续降低的当下，漏洞修复的效率更需要同步跟上。  
  
  
天融信7*24小时技术支持  
  
  
天融信拥有成熟的安全漏洞应急响应体系和丰富的安全防护实践经验，持续跟踪国内外重大安全威胁。  
如您在漏洞排查、版本升级或安全防护方面需要技术支持，可随时拨打  
天融信7*24小时客服热线：400-777-0777，我们将为您提供专业的安全咨询与处置协助。  
  
  
声明：天融信拥有对此公告的修改和解释权，转载须保证公告完整性。因传播或利用此公告内容造成的任何后果，由使用者自行承担，天融信不承担相关责任。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/dSWSuPicfjTd3dJib33e8bzOdEjDEbiakJ43Mn69glY2VHicImmZLgbKTxkhK0oM9aQzXhzU5sJVjyFUYa69xwnntCTlF8Xjnrkfwtf8ysrELb4/640?wx_fmt=png&from=appmsg "")  
  
  
  
近期热点  
  
[天融信李雪莹：“行动力”是风险的关键变量，智能体安全须构建体系化能力](https://mp.weixin.qq.com/s?__biz=MzA3OTMxNTcxNA==&mid=2650995414&idx=1&sn=eacd65201d6c8513df4e3bf836fcdc30&scene=21#wechat_redirect)  
  
  
[大模型如何从“能用”走向“好用”？光明网专访李雪莹：须筑牢安全防护体系](https://mp.weixin.qq.com/s?__biz=MzA3OTMxNTcxNA==&mid=2650996047&idx=1&sn=b183ec16a168d82917335a7dec306f73&scene=21#wechat_redirect)  
  
  
[打破算力黑盒，定义智算边界！天融信AI网关正式发布](https://mp.weixin.qq.com/s?__biz=MzA3OTMxNTcxNA==&mid=2650996873&idx=1&sn=492a90bb5f5d22f6398eb7cccbda4fa7&scene=21#wechat_redirect)  
  
  
[天融信发布AI Agent安全防护方案！](https://mp.weixin.qq.com/s?__biz=MzA3OTMxNTcxNA==&mid=2650996620&idx=2&sn=df675f69ecaa2114beea3b2f02feded1&scene=21#wechat_redirect)  
  
  
[体检+守护+运营，天融信发布教育行业智能体安全方案](https://mp.weixin.qq.com/s?__biz=MzA3OTMxNTcxNA==&mid=2650996873&idx=2&sn=6f1414400fe881ceec3150e658b8c1f6&scene=21#wechat_redirect)  
  
  
  
**关注天融信了解更多信息**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dSWSuPicfjTfMTWicc8vjaOqbhhrKehADyT8bNDMKcj3HRo43hWskicsVDx0v5AIybW780yVwklGZIgsI8d7rU2QveGyUnE08BFQ6YHTiaI2DEs/640?wx_fmt=gif&from=appmsg#imgIndex=2 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/dSWSuPicfjTch4xDfDkudNeE2ejFmFFlP2aXic9unoe8RnpINgNzfOoczYWndia6D5nLW8jAFAXsXYs9jVVqNozeYE4JshxcpyjlVk1jY6GE84/640?wx_fmt=gif&from=appmsg#imgIndex=3 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dSWSuPicfjTd2lEcLMGdLD9jnKfdcKH98RiaxZhnQHEHVn3ES0B7975o0vlNOV7iaicXMIq8AsBV8QW6zUnBhtEOJdIhmgPicnISd9yHTPjN504o/640?wx_fmt=gif&from=appmsg#imgIndex=4 "")  
  
  
  
  
  
  
  
