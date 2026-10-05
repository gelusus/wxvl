#  俄罗斯黑客利用 Zimbra 0-Day 漏洞，零点击邮箱秒没  
原创 hacking
                    hacking  Hacking黑白红   2026-10-03 13:37  
  
**兄弟们，今天聊个真能吓出冷汗的。别再以为“不点链接、不下附件”就高枕无忧了。**  
  
  
最近NSA、CISA联手曝光：  
  
  
一个俄罗斯顶级黑客组织（Laundry Bear），用Zimbra邮箱的0day漏洞，搞出了真正的“零点击”攻击——你啥也没干，就只是看了一眼邮件，邮箱就被彻底搬空了。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicJ861ibtj9DNoia9xQZTe7le4IicL6GDnIBRGUCuicvTw2zCWibzMbcDuhIiaPyncvgPjuiaRaKXjqwwHI1nAKzqATahcVy5roibfmnAwM/640?wx_fmt=jpeg "")  
  
  
**黑客的“碎片魔术”：邮件咋自己炸的？**  
  
****  
****  
这帮黑客玩了个贼溜的手法，叫“标签切分”。  
  
  
他们把恶意代码打碎，塞进HTML邮件里。  
  
Zimbra的清洗器以为自己在清理垃圾，结果一通操作，反而把碎片拼成了完整的攻击标签。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicKJJYUVianrjjcyoiakl3mrgicwH6MaMPL19hXPRE3Y6CyibtM5nyV1Kn8dcIdOyuy8z3eqXx0pegN3aYAo2ibnbGRXZuO1eYyCx1as/640?wx_fmt=jpeg "")  
  
  
  
邮件一渲染，恶意脚本直接在你的登录会话里跑起来了。这哪是钓鱼，这简直是“看一眼就中招”！  
  
  
**ZimReaper：黑客的“收割机”有多狠？**  
  
****  
跑起来的攻击脚本叫ZimbraReaper，干的活儿堪称“一条龙服务”：  
  
  
偷CSRF令牌、浏览器存的密码、2FA救援码；翻出整个公司的通讯录；把你近90天的邮件打包外传。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TVljsu2eAicJesMbNsOrehxCV5aCnDvgjCiaZdYD391TFb71xD34cmBcibUzmGYv0yKicnuVlLFHhx7C7fIgX2UgGWoWcswdIjSgzQrIibOE1anE/640?wx_fmt=jpeg "")  
  
  
  
最阴的是，它还能偷偷建一个叫“ZimbraWeb”的应用专用密码，绕过2FA，就算你改了主密码，黑客照样能用IMAP静默登录读邮件，根本踢不走！  
  
  
**紧急应对：别光打补丁，账户得“洗澡”**  
  
****  
Zimbra虽然发了补丁（建议直接升到10.1.20），但补丁只是堵洞。  
  
  
已经中招的账户，必须彻底清洗：  
  
重置密码、清会话、重发2FA码、删掉那个阴险的应用专用密码，再查查IMAP开关有没有被偷偷打开。  
  
  
自托管邮件的兄弟，这波操作给所有Webmail都敲了警钟——渲染层有漏洞，看邮件就等于给黑客开门。  
  
  
  
-- END --  
  
*本文作者：hacking  
。  
文章内容或图片来自网络，如果存有侵权，请留言，第一进行处理。  
  
  
