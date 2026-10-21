#  EvilWAF：传输层绕过 WAF，透明代理加深度漏洞扫描  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-21 01:07  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6N7gg0KNWZQnBKHwiahFlWGhw6IDL5d6ribmqEh1x9EOQomS0jvmyPqP9QhYf1lP6EVwGlAdXXa1E8KMPKlg2nVvh100D7KUFeEM/640?from=appmsg "")  
> **导语**  
：WAF 这几年进化得很快，早年那套改改 payload、换换请求头的把戏，现在基本走不通了。EvilWAF 换了个思路——它不碰你的攻击载荷，直接从传输层下手：轮换 TCP 配置、伪造 TLS 指纹，必要时还能帮你挖出 WAF 后面的真实源站 IP。顺便它还能反过来当 WAF 扫描器用，一次探十层防御。  
  
  
![EvilWAF 绕过 WAF](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6OpkcgL3kLM0y7fHgicx4icsMeS0LMHia8to8fn3rhOWa2XVdOS5icCH2TeLMhkoEsFu6tyUSsQeDTKiaKA57iaYCqZhxoOcZl4vOFDw/640?from=appmsg "EvilWAF 绕过 WAF")  
## 一、EvilWAF 是什么  
  
上一篇文章我们聊过 WAF 是什么、手工怎么绕。现在的问题是，防护侧已经不满足于简单的特征匹配了，开始做行为分析。结果就是，你精心藏起来的 payload 往往还没摸到应用逻辑就被拦下来。  
  
EvilWAF 是为了对付这个局面来的。它是一个开源渗透测试工具，工作方式是透明中间人（MITM）代理加 WAF 漏洞扫描器。  
  
关键区别在这：sqlmap 那类工具是改 payload、改 header；EvilWAF 一个字都不动你的攻击数据，它在传输层做文章。这样它能绕开 WAF 的行为检测，同时顺手检查防火墙本身的弱点。  
  
它有两种工作模式：代理模式，以及独立的 WAF 扫描器。  
  
**代理模式**  
下，你在本地跑起 EvilWAF，让你的攻击工具把流量发过来。这条链路上没有任何载荷篡改——sqlmap 发出的 SQL 注入语句、HTTP body、应用头，EvilWAF 原样放行。sqlmap 负责找应用层漏洞，EvilWAF 负责处理连接：轮换 TCP 配置、TLS 指纹（比如 JA3、JA4），让 WAF 既不拦你也不给你限速。  
  
更狠的是 auto-hunt 选项。开了它，EvilWAF 会想办法找出 WAF 背后的真实后端 IP，然后把 sqlmap 的流量直接送到源站——WAF 干脆被绕过去了。  
  
**独立扫描模式**  
下它也能单干。加上 scan-only 参数启动，目标就变成防火墙本身：横跨十个防御层做一轮完整体检，规则引擎有没有缺口、限流配得对不对、已知的规避手法能不能钻空子、HTTP 方法行为是否正常、TLS 设置有没有松的地方，全给它查一遍。  
## 二、安装  
  
在 Kali Linux 上装，用 Python 虚拟环境就行：  
```
kali> git clone https://github.com/matrixleons/evilwaf.gitkali> cd evilwafkali> python3 -m venv myenvkali> source myenv/bin/activatekali> pip3 install -r requirements.txtkali> python3 evilwaf.py -h
```  
  
![EvilWAF 帮助信息](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OLFvIzFEInvYMub8piaic061C8bqjqaUIiaMfxIZ8HZon9VWbUOMovMXklesx8jHCA2OhyrGDtYTE7B9SMTjFjyCDAXSpKSoKnGc/640?from=appmsg "EvilWAF 帮助信息")  
  
因为 EvilWAF 是个完整的 HTTPS 中间人代理，它得能看到加密流量。第一次运行时会自动生成一套本地 CA 证书。你需要在 Kali 上信任这套证书，否则代理工具用起来会报 SSL 错误。  
  
先生成证书，跑一次就行：  
```
kali> python3 evilwaf.py -t https://example.com
```  
  
然后导入证书并设置为系统级信任：  
```
kali> sudo cp /tmp/evilwaf_ca_*/evilwaf-ca.pem /usr/local/share/ca-certificates/evilwaf-ca.crtkali> sudo update-ca-certificates
```  
  
![导入并信任证书](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6P2ddZ5V4KYStjickW90QoBw3m19QwVryPZgjT3ia0M9S7N7GIKdibHtKW3rg7NiavIiaa2laIwu9MkPnErkhassVOwiaTXVo7Isr3qk/640?from=appmsg "导入并信任证书")  
## 三、跑起来  
### 3.1 代理模式  
  
指定审计目标启动，默认会开一个实时终端界面，监听 127.0.0.1:8080：  
```
kali> python3 evilwaf.py -t https://target.com
```  
  
EvilWAF 跑着之后，让你的攻击工具把请求走这个代理。比如 sqlmap：  
```
kali> sqlmap -u "https://target.com/page?id=1" --proxy=http://127.0.0.1:8080
```  
  
这时候 EvilWAF 的界面就会动起来。  
  
![EvilWAF 终端界面](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NSAtpegOa3tTXuJFI4znHVInaEiczEj1OukgEkFSFooXF2E8HUumKp6s7yz1CPr6feu8q2ibe1QniaZkTic6pnnvYwPL7ZVtibIiavY/640?from=appmsg "EvilWAF 终端界面")  
  
你可以实时看流量，更关键的是它会显示：哪些传输层操纵技术生效了、用了什么指纹配置、往出站请求里注入了哪些头。这块信息在调绕过策略时非常有用。  
  
auto-hunt 是另一个值钱的功能。它并行跑 10 个扫描器来找源站 IP，手段包括 DNS 历史记录、SSL 证书分析、子域名枚举等等：  
```
kali> python3 evilwaf.py -t https://target.com --auto-hunt
```  
  
找到高置信度的 IP 之后，它会问你要不要把流量直接路由到真实服务器。  
  
![auto-hunt 溯源结果](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6MIiagrzczicLL6Yx777D1ibYVmQugXqMk2F4KFfeK1NBcCoQR6Xv99FgKcDPImaIbrJeThYjqVqBEmLv39uyQ23mJH39q1WicoUvE/640?from=appmsg "auto-hunt 溯源结果")  
  
如果你已经知道源站 IP，也可以手动指定：  
```
kali> python3 evilwaf.py -t https://target.com --server-ip 1.2.3.4
```  
### 3.2 独立 WAF 扫描模式  
  
不跑任何第三方工具，EvilWAF 自己就能给防火墙做配置体检：  
```
kali> python3 evilwaf.py -t https://target.com --scan-only
```  
  
十个防御层同时开探。  
  
![WAF 扫描结果第一页](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6MtseZB8wXfnbicQrFym1EaDMmt2iahibOtqZ3zJWricMHh5RoUXSibJcuEhjhgdGkUQrSx0icUosXLMicL0U2TNvPHaibee0FcCiaIhfFo/640?from=appmsg "WAF 扫描结果第一页")  
  
![WAF 扫描结果第二页](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6MHXIdQwVABzhbQpJjt8X9FzZMHIJiaLe30djCJSWRjXxnFprxD5kAw0rxxLnhPXPR1OxQ7nicvIEVatZGh8lYabCW6uymjtrEqI/640?from=appmsg "WAF 扫描结果第二页")  
## 四、说说 Cloudflare  
  
得单独拎出来讲一句：Cloudflare。  
  
今天的 Cloudflare 算得上一线 WAF，它不是一堆规则，而是一整套多层防御体系。全球几百个数据中心协同，让它能在所有受保护的站点上快速识别并缓解新出现的攻击模式。  
  
它判断一个请求的信号很多：TCP/IP 与 TLS 指纹、HTTP/2 结构、行为模式，然后把这些因素关联起来，给每个请求打一个风险分。跟那种依赖已知攻击特征的传统 WAF 不同，Cloudflare 用的是在海量真实攻击流量上训练出来的机器学习模型，专门识别与正常浏览器流量的偏离。  
  
这就让它很难打。规则型 WAF 上好用的招，落到它身上基本失效。它的行为指纹甚至在检查载荷之前就先给你拦了——光凭传输层特征就能判定你是个工具。限流也做得细，它会追踪会话模式并逐级加挑战，阈值很难被有效利用。  
  
所以 EvilWAF 只是"绕过 WAF 可能长什么样"的一个例子，绝对不是穿甲弹。  
## 五、小结  
  
EvilWAF 算是防火墙绕过进入新阶段的一个标志。载荷混淆、简单编码这些东西，在现代 WAF 面前基本没用了。真正需要关注的是 EvilWAF 这类在传输层做文章的思路。  
  
![作者 AirCorridor](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OgRc8fcFCuTYcxaeT9syOCOWhAR9AmnYwBaLasChGTn0j36AWQp7b4icSNDnpq0z7FugGJJibur7uPbnAE2vqrw4vQ0HGLZ0WjU/640?from=appmsg "作者 AirCorridor")  
  
原文作者 AirCorridor 是一名资深网络安全专家，拥有信息技术与系统方向的网络安全硕士学位，做过政府平台的加固、Linux 高级运维和专业的网络攻防，同时也在 Hackers-Arise 平台上供稿。  
  
对漏洞赏金、红队或者 AI 安全感兴趣的话，可以看看他们的培训中心的课程。  
## 素材出处  
  
本文为编译报道，内容整理自以下原始资料：  
- 原文标题：Bypass Web Application Firewall (WAF) with EvilWAF  
  
- 原文地址：https://hackers-arise.com/bypass-web-application-firewall-waf-with-evilwaf/  
  
- 工具仓库：https://github.com/matrixleons/evilwaf  
  
- 延伸阅读：WAF 介绍与手工绕过方法  
  
- 原文作者：AirCorridor  
  
- 文中截图版权归原作者所有，本文仅作技术学习交流使用  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6PW4A4hiaC7Xk11me9ia2rtd2RjsNoWmUazsqKqMykoXSRxnBSxuw5flY5sSVibXVbfbPZCGgqwjxR6t9MicK0gC5tJqI70xqEib8KQ/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
