#  芯片底层漏洞无法 OTA 修复！用旧 iPhone 存助记词要当心  
 看雪学苑   2026-11-03 09:59  
  
不少币圈用户习惯把助记词存在旧iPhone里，关机、清数据、开飞行模式，以为离线就绝对安全。OneKey创始人Yishi近期提醒：这个想法有风险。  
  
  
**哪些机型中招**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K02mRgVl9gicEj6RzaNerlgkuibrib3wylx62lrQWvYWctVJh69abHBcJyaxRMgNicnM3ZQX74lhf3ZPSJW3j8vmInQ0XyGRrlyhnc/640?wx_fmt=png&from=appmsg "")  
  
- checkm8：iPhone X及更早机型  
  
- usbliter8（2026年6月公开）：A12/A13芯片，XS、XR、11全系列、第二代SE  
  
漏洞在BootROM，是芯片出厂就固化的底层启动代码，iOS升级永远修不掉。  
  
  
**攻击方式**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2JMonP1JicMhc5XbX76HibP68hZmTpYnQShmSjxy9yVNy6nzMCeawKWiaS7tyqxUhUEd7Bf7d5oo0xiafY9Yqg8BxxPnZVRPSxZKQ/640?wx_fmt=png&from=appmsg "")  
  
  
攻击者拿到手机物理控制权，进入DFU模式，搭配简易USB硬件（树莓派Pico 2即可，成本很低），发送特制USB信号造成内存异常，在iOS系统启动前跑自定义代码，绕过苹果启动校验。  
  
  
这个漏洞不会直接攻破安全隔区（Secure Enclave），没法一键读出加密助记词。想要盗币，还需要其他条件配合，不是拿到设备就一定能转走资产。但它打通了底层启动链，风险真实存在。  
  
  
**额外风险**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0XtVwTibdZvmjDUOpUntB5gzX3FYNZPgN1b5pic0dwzibPgyLmULFjlWwbyIMHGISlpGiaDY8MSp9CkiafuhBU0fZ8Z9ZCPM6U6znA/640?wx_fmt=png&from=appmsg "")  
  
  
除BootROM漏洞外，恶意充电设备的ChoiceJacking攻击，也能在离线状态下篡改连接授权。离线 ≠ 安全。  
  
  
**实用建议**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2ibzcBKazwBubNez4Dwafp5ia8Cqye0faTdicrA6GYylRm31PFVYibQXFfJBTsl0PvN0sWt6hzjpHdMKCX0GlSnD8fzO6icV8iapYog/640?wx_fmt=png&from=appmsg "")  
  
1. 不要把旧iPhone当作冷钱包存放助记词；  
  
1. 强锁屏密码、钱包密码是必要防护；  
  
1. 若怀疑助记词已经暴露：在可信设备生成全新助记词，资金全部迁移；只从旧助记词新建地址没用，风险依旧存在。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K0ZKaY9XA86xZokKovxMkI5ow0tyUhq1qFcRQXViafJDzNhugBXwcP1bHwNgEACJDpicLqghSxdU5xiaicMYWKtcpgn1VlWsElIhPY/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0r0W2ByCa5L1yemhBjAelJxgyCh2UjjLUQjdlYM8icBMn1vEm3tzwXhpnPPr6u75cCJQZo8Z4fibsA8wicRd3aXT1phmlMMN8OKM/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K2aGy1SToniaOZibXSSCX9p0SGXw2a159fj3NdPHvMV1m3netFqPuLQL52bUCGUdE1dGDAfDvKlmuncWichExiaDvUjUZTBDdys9rY/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K13Pvk2HEZUpFWbTTrW4YUafNw5xH2QKficTu1gSrdBic6ia0WxL7ZJAiaBBpp3xVhrrFlvV7z85qh11QpPAGfIvTWshuuEb9cOJHw/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
