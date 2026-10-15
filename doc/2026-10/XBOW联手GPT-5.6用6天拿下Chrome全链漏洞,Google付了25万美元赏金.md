#  XBOW联手GPT-5.6用6天拿下Chrome全链漏洞,Google付了25万美元赏金  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-15 00:43  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6PGr0AzoFepq6h0cFpkDXjX1ibWMwpCxoAw9yFialVOr3IiaUBHy0ib9ibhAfbV7HWFeldOkibrEOlvlF5F4xiaoLzymwTwwDUeEAx4Pc/640?from=appmsg "")  
> **导语**  
:XBOW团队借助OpenAI的GPT-5.6 Sol模型,6天时间、花费约4万美元算力,完成了一条影响Windows版Chrome 10亿用户的全链漏洞利用。Google据此向XBOW颁发了25万美元奖金——这是Chrome漏洞赏金历史上第二次颁发该级别奖励。  
  
  
![XBOW收到的Chrome漏洞奖励邮件](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MvLHDZKRBibcz25fUkLQFgPZIXdjribJHSU2DhOyE4r3x7efPdic9tkH8aEw7JRL2cqa2Qwn64s9avcj7AEca2O7Oo3RJTlo5Vhc/640?from=appmsg "XBOW收到的Chrome漏洞奖励邮件")  
图:XBOW创始人Oege de Moor在推文中公布的Chrome VRP通知邮件,确认25万美元全链漏洞奖励。图源:XBOW创始人Oege de Moor (@oegerikus) 在 X 上的公开推文。  
## 一、事件:Chrome历史上第二次25万美元全链奖励  
  
2026年Black Hat与DEF CON大会期间,Google向安全研究公司XBOW颁发了25万美元的全链漏洞(full-chain exploit)奖励。这是Chrome漏洞赏金计划历史上第二次支付该级别金额,距离上一次已经过去多年。  
  
所谓"全链漏洞",指的是一条从浏览器渲染进程远程代码执行,到沙盒逃逸、再到系统权限提升的完整利用链。攻击者拿到这条链后,只需让受害者访问一个恶意网页,就能在用户毫无感知的情况下读取所有邮件、密码与银行登录信息——无需下载文件,无需点击钓鱼链接。  
  
XBOW创始人Oege de Moor在公开推文中写道:"一个情报机构的老兵告诉我,他所在的团队以前要用6个月才能完成这类漏洞。"  
  
![Chrome浏览器与AI神经网络的安全研究概念图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6NcwwibDJEgllIb0cc0uTuxyFytlDqfJPjhYQqE40n5INpRQTnOImqlGyibsp6OC55SjC5OhOGiczib9s1gs501xicXIXoHR9lIQfY4/640?from=appmsg "Chrome浏览器与AI神经网络的安全研究概念图")  
## 二、团队构成:四名人类研究员,一名AI队友  
  
承担这次挑战的是XBOW的五人小组——四名人类研究员加一名AI:OpenAI的GPT-5.6 Sol模型。  
  
人类成员包括研究主管@moyix、Andres Luksenberg、Baul Lee与Willem Pinckaers。GPT-5.6 Sol被团队视为"第五位队友",参与漏洞分析与利用构造的多个环节。值得注意的是,GPT-5.6并非新近发布的前沿模型,而是2026年初的版本——也就是说,这一成果使用的是已经公开的模型权重,并非任何"独家"工具。  
  
Oege在2026年初于马耳他召开的全员会议上立下了这个目标:在当年8月Black Hat大会前,完成一次Chrome全链漏洞利用。这既是对XBOW能力的检验——其早期业务集中在Web应用安全——也是对模型在桌面软件、服务器与嵌入式设备上是否同样有效的测试。Chrome作为地球上防御最严密的软件之一,被全球超过一半的人口使用,拿下它意味着验证AI已经走出Web应用安全的舒适区。  
  
![6天AI辅助漏洞构造工作流概念图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6OVPrRNBf3Pcmticl8nBvrQvFChjb54jdY9cic20y9T5EBJQxCVWia8hx1hmZTzxvRWSSOSER3uhyrLEXdkD24UzNyeGOzGyahk0I/640?from=appmsg "6天AI辅助漏洞构造工作流概念图")  
## 三、6天与4万美元算力:一次临阵换方案的过程  
  
在Black Hat开幕前,团队提交了第一版利用链,但Google反馈其中一个漏洞他们已经知晓。这条路被堵死了。  
  
此时团队已经身在拉斯维加斯,准备参加Black Hat与DEF CON。Brendan与剩余成员决定就地重做,在6天时间内构造出一条新的全链漏洞,然后再次提交。  
  
最终成本:6天时间,约4万美元的算力开销。XBOW据此拿到了2026年新规下的第一笔25万美元全额奖励。多个研究者在评论中提到,通常此类全链漏洞在地下市场的报价在数百万美元级别——Google以25万美元拿下这份情报,实质上获得了相当大的折扣。  
## 四、行业意义:漏洞发现成本正在崩塌  
  
Oege在推文结尾给出了一个观察:发现真正严重漏洞的成本,对攻击者和防御者同时在下降。限制强大模型并不是出路——GPT-5.6 Sol的权重早已公开,任何人都可以调用。  
  
但防御者有一项攻击者永远不具备的优势:**可以先测试自己的代码**  
。  
  
他的建议直白而沉重:假设你的软件是可被利用的。谁在找你漏洞的人,手里握着和你相同的工具。这不是引发恐慌的理由,但是采取行动的理由——缩短补丁周期、加大内部主动测试、把AI纳入漏洞管理流水线,都变得比以往更紧迫。  
## 五、XBOW的下一步:从Web扩展到桌面、服务器、嵌入式  
  
XBOW同时宣布,公司正在开发一款用于测试编译软件(桌面应用、服务器、嵌入式设备)的自主产品,并与全球顶级机构合作设计这一产品。Oege在推文中邀请有意向的团队与XBOW取得联系。评论区有研究者指出,本次利用链中包含对Chrome沙盒逃逸组件Crashpad的利用——一个原本用于在Chrome崩溃后回收数据的组件,反倒成了突破Chrome防线的出口。  
  
推文最后一行是一句面向所有Chrome用户的提醒:**只要看到更新按钮,就重启Chrome**  
。  
  
**版权声明**  
:本文由华盟网原创发布,保留所有权利。推文截图来源于Oege de Moor (@oegerikus) 的公开X账号,经合理引用,版权归原作者所有。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6OxSJOX74NlGpHoa19a6RaibgkVaV9BjbxLQWFFxSmxr6La48kqp3GlTPxnCHiatgERjDJDQzvq9u2Jxck6uCFHOytermWqJ3wxo/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
