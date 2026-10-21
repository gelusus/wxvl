#  【漏洞预警】PHP 远程代码执行漏洞  
 数字人才创研院   2026-10-21 03:49  
  
点击上方  
蓝字  
关注我们  
  
 漏洞预警  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/5d6O0HSxyXkoJlmfe0w3ScXV6VhNKNLP8JvkZzicOziaviaSxjKtia1pzPiaT7RYjdTUm8TYGhpY365mIXEzVGucoSA/640?wx_fmt=png&from=appmsg "")  
  
01  
  
漏洞基本概述  
  
Vulnerability Overview  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4kcMpxcoP324FB41iaOFjLovYzq48jiayABVZozfrfjNU4hTK3iaTAS71icN9rSUtVqAavoMiagZ3BVyyJlLgxTCiarQ/640?wx_fmt=png "")  
  
01  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qiaou2SWwgPxZsPuGUBqvfztY349JahH5gTicMQHkTaziasQOt0sYZP0QicweflsMyu4g1lUGPOrIpDlBwCYibwNjYw/640?wx_fmt=png "")  
  
    
PHP 是一种广泛使用的开源服务器端脚本语言  
。  
  
**【风险等级】**  
**高 危**  
  
**【CVE编号】**  
**CVE-2026-109618&109619**  
  
  2026年10月20日，禾盾安全应急响应中心监测到该漏洞  
，  
经分析，  
攻击者  
可能远程代码执行，攻击成功后，可能造成网站被篡改、敏感信息泄露或服务器失陷。  
建议受影响的用户尽快修复，  
与此同时，请做好资产自查以及预防工作，以免遭受黑客攻击。  
  
02  
  
漏洞影响范围  
  
Vulnerability Impact  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4kcMpxcoP324FB41iaOFjLovYzq48jiayABVZozfrfjNU4hTK3iaTAS71icN9rSUtVqAavoMiagZ3BVyyJlLgxTCiarQ/640?wx_fmt=png "")  
  
01  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qiaou2SWwgPxZsPuGUBqvfztY349JahH5gTicMQHkTaziasQOt0sYZP0QicweflsMyu4g1lUGPOrIpDlBwCYibwNjYw/640?wx_fmt=png "")  
  
      
PHP 5 至 PHP 8.5  
  
03  
  
漏洞修复方案  
  
Vulnerability Fixes  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4kcMpxcoP324FB41iaOFjLovYzq48jiayABVZozfrfjNU4hTK3iaTAS71icN9rSUtVqAavoMiagZ3BVyyJlLgxTCiarQ/640?wx_fmt=png "")  
  
01  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qiaou2SWwgPxZsPuGUBqvfztY349JahH5gTicMQHkTaziasQOt0sYZP0QicweflsMyu4g1lUGPOrIpDlBwCYibwNjYw/640?wx_fmt=png "")  
  
  目前官方暂未发布修复版本，  
PHP 官方计划于 2026年10月21日22:00（北京时间）发布安全更新。  
  
    
**注：其它建议**  
  
      
1.  
清点对外提供服务的 PHP 主机与容器实例，建立版本台账，优先处置直接暴露于互联网的 Web 服务  
。  
  
    2.  
最小权限原则：确保用户权限最小化，避免高权限账户暴露于不可信环境。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/0wuPBaVgoOYjNSv7cAjMMYg0mhKAeyIsH7bLKhSWIBYfTGr6QJg3rzuBib1j14CktH8V69yCqMLstHpQdibcqibXzulNryI5KlYCgLO9wcibEnQ/640?wx_fmt=png&from=appmsg "")  
  
  
04  
  
漏洞参考链接  
  
Vulnerability Fixes Link  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4kcMpxcoP324FB41iaOFjLovYzq48jiayABVZozfrfjNU4hTK3iaTAS71icN9rSUtVqAavoMiagZ3BVyyJlLgxTCiarQ/640?wx_fmt=png "")  
  
01  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qiaou2SWwgPxZsPuGUBqvfztY349JahH5gTicMQHkTaziasQOt0sYZP0QicweflsMyu4g1lUGPOrIpDlBwCYibwNjYw/640?wx_fmt=png "")  
  
```
https://www.php.net/archive/2026.php#2026-10-20-1
https://www.bleepingcomputer.com/news/security/php-warns-admins-of-two-critical-flaws-emergency-patches-coming-tomorrow/
```  
  
  
05  
  
漏洞时间滚轴  
  
Vulnerability Time  
  
[#发现时间]()  
#  
  
2026年10月20日  
  
[#验证时间]()  
#  
  
2026年10月21日  
  
[#通告时间]()  
#  
  
2026年10月21日  
  
HD  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibwov0bgAkBeVIILoZgYNTn6rrKD6sSkPVG8nZcwn2QWHibG6AsEyuy9n3XGMuoZIyGCEmvv1ZRnaibXbYkrIP4Hg/640?wx_fmt=png "")  
  
禾盾安全应急响应中心  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/f1Ap1TId9Sp8P5te00xOHDe8oCZN6GRlKk24gBLXpsI9gV0pyabsYgkrAhygibQL1VzhABHrUstCyoMHysrBY5g/640?wx_fmt=png "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XCey8HnLC70G2J8ft1vNlFXiaURP3NcPfiaB4KASRaibFGRX73tiaiaKFFxVbDv0UiaGpvxGxAvdr147CIwPrVkw8Qvw/640?wx_fmt=png "")  
  
    
**以技术为驱动，以安全专家为核心，以诚信为本、以专业为先、以坚持为恒，围绕漏洞生态体系打造集漏洞监测、漏洞收集、漏洞挖掘、漏洞分析、漏洞管理、专家响应、漏洞预警、安全服务定制化于一体的漏洞安全一站式服务，帮助客户防患于未然，在降低资产风险的同时，大幅提升客户对漏洞感知、预警、分析等响应能力，为国家、政企客户、用户抢占风险预警处置先机，提升网络安全主动防护能力。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ry30yBl2sr8jsWibpdia0ku8mG2eibhcjnIADPzCMXXlsdUXs3KzibJ8Q6q6VCw33WRPxx0xNpI8vyepKOONH1Ij2A/640?wx_fmt=png "")  
  
  
HD  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibwov0bgAkBeVIILoZgYNTn6rrKD6sSkPVG8nZcwn2QWHibG6AsEyuy9n3XGMuoZIyGCEmvv1ZRnaibXbYkrIP4Hg/640?wx_fmt=png "")  
  
获取更多最新情报  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/f1Ap1TId9Sp8P5te00xOHDe8oCZN6GRlKk24gBLXpsI9gV0pyabsYgkrAhygibQL1VzhABHrUstCyoMHysrBY5g/640?wx_fmt=png "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XCey8HnLC70G2J8ft1vNlFXiaURP3NcPfiaB4KASRaibFGRX73tiaiaKFFxVbDv0UiaGpvxGxAvdr147CIwPrVkw8Qvw/640?wx_fmt=png "")  
  
**建议您订阅「禾盾安全-漏洞情报」服务，及时获取更多漏洞情报详情以及处置建议，让您的单位真正远离漏洞威胁。**  
  
**电话：177-128-77993**  
  
**邮箱：src@hedun.com.cn**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ry30yBl2sr8jsWibpdia0ku8mG2eibhcjnIADPzCMXXlsdUXs3KzibJ8Q6q6VCw33WRPxx0xNpI8vyepKOONH1Ij2A/640?wx_fmt=png "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5d6O0HSxyXkoJlmfe0w3ScXV6VhNKNLPmYBiaYmk4QgObS8Iicl7vJ6Q685oE8vTNLHFfHefk58InmILibRUJDzGw/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/SkPgzibRbCL5X8OKYMB024f1gWFpTuBibMDe5YTnPU2uAuEBUoiaXD3OXnTBfPfGyew6VpxO5dBgFoic9TaRhHouFw/640?wx_fmt=jpeg&from=appmsg "")  
  
