#  赏金视角下的缓存欺骗与投毒：补丁比漏洞更危险？Cloudflare 绕过与 DoS 翻车复盘  
原创 升斗安全XiuXiu
                    升斗安全XiuXiu  升斗安全   2026-10-24 04:23  
  
**【文章说明】**  
- **目的**  
：本文内容仅为网络安全**技术研究与教育**  
目的而创作。  
  
- **红线**  
：严禁将本文知识用于任何**未授权**  
的非法活动。使用者必须遵守《网络安全法》等相关法律。  
  
- **责任**  
：任何对本文技术的滥用所引发的**后果自负**  
，与本公众号及作者无关。  
  
- **免责**  
：内容仅供参考，作者不对其准确性、完整性作任何担保。  
  
**阅读即代表您同意以上条款。**  
  
****> 【内容描述】 关于缓存投毒漏洞的相关资料一出，各家厂商连夜补漏洞。本文复盘这场"防御 vs 绕过"的攻防：Cloudflare 的两道防线看似严密，却栽在一个"头匹配 Host 就跳过缓存键"的小优化上；另一些公司的补丁更是自己造出了拒绝服务。最后附上 Param Miner 的重要更新提醒。看完你会明白——临时打补丁，有时候比漏洞本身还危险。  
  
  
关于 Web 缓存投毒的相关资料发出来之后，好几家公司连夜部署了防御，想把这招按下。  
  
这本来是好事。但作为研究的一方，我自然想看看：这些防御，到底经不经得起敲。  
  
结果嘛……有的扎实，有的却......怎么说呢，像极了考试前熬夜突击、专背原题却没理解原理的同学。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qg1MKHx3jGHLXqVoIiaW10AXz83Ksuib7ljpCjyicA4xPsNa7XZLict0OZJRI3mwDVV5icworGulibRfXGh4LL8DXyjFticMzjdwzj6Owr9pkjjUSw/640?wx_fmt=png&from=appmsg "")  
## 一、三家厂商的"答卷"  
  
先说整体反应，挺有意思的:  
- Akamai 发了一份简短回应，令人费解地引用了一堆针对"Web 缓存欺骗（Web Cache Deception）"的缓解措施。问题是，缓存欺骗和缓存投毒是两码事，那些措施对防投毒基本没用。这波属于……答非所问。  
  
- Fastly 发了一份安全公告，给了不少关于缓解的详细建议，态度是端正的。  
  
- Cloudflare 走得更远，直接上了  
全局缓解措施，还写了篇博客《Cloudflare 如何保护客户免受缓存投毒》专门讲这事。  
  
咱们重点扒一扒 Cloudflare 那两道防线，因为它们最有代表性，也最有"翻车"的戏剧性。  
## 二、Cloudflare 的两道防线  
  
防线一：在 WAF 里加规则，拦掉某些头里的"XSS 友好字符"。  
  
比如 X-Forwarded-Host 里要是出现 < 这种符号，直接 403:  
```
GET / HTTP/1.1
Host: wafproxy.net
X-Forwarded-Host: xss<
HTTP/1.1 403 Forbidden
Attention Required!  
```  
  
这招确实让"用这些头直接打 XSS"变难了。但 Cloudflare 自己也承认：这拦不住所有——因为很多利用根本不需要 < 这种字符。  
  
防线二：把这类头加进默认缓存键，理论上就再也没法用它们投毒了。  
  
看效果:  
```
GET / HTTP/1.1
Host: wafproxy.net
X-Forwarded-Host: evil.net
HTTP/1.1 200 OK
<a href=”https://evil.net/”
缓解前的缓存键：https://wafproxy.net/
缓解后的缓存键：https://wafproxy.net/|evil.net  
```  
  
加了 |evil.net 之后，每个不同的头值都会生成不同的缓存键，投毒自然就失效了。这条，看着是真的稳。  
## 三、栽在一个"看起来很聪明"的小优化上  
  
可惜，两道防线都埋了同一个实现缺陷，能被彻底绕过。  
  
根子出在一个小优化上：  
如果X-Forwarded-Host的值跟Host头一样，Cloudflare 就不把它加进缓存键。听起来很合理——自己指向自己，有啥可记录的？  
```
GET / HTTP/1.1
Host: wafproxy.net
X-Forwarded-Host: wafproxy.net
缓解后的缓存键：https://wafproxy.net/  
```  
  
致命的地方在于：  
Cloudflare 只看了每个头的"第一个实例"。  
  
攻击者于是可以发一个"重复头"——第一个值无害（跟 Host 一样），第二个值塞满载荷。后端服务器处理这种重复头时，通常会用逗号把两个值拼起来:  
```
GET / HTTP/1.1
Host: wafproxy.net
X-Forwarded-Host: wafproxy.net
X-Forwarded-Host: evil.net”/><script...
缓解后的缓存键：https://wafproxy.net/
```  
  
看清楚没？响应里，evil.net 和那段  都好好地回显出来了，说明后端老老实实吃了第二份头的载荷。但 Cloudflare 因为只检查了第一个头（它等于 Host），就认定"这头不用进键"——  
缓存键还是干干净净的https://wafproxy.net/。  
  
于是，一个带着 XSS 的毒响应，被按"正常页面"的键缓存了，发给之后每一个访客。两道防线，一起被绕。  
## 四、已修复，但客观说句公道话  
  
我把这问题报给了 Cloudflare，缓存键那个绕过现在已经修好了。  
  
不过我得还人家一个公道：在所有厂商里，Cloudflare 是  
唯一一个真去搞"技术缓解"的。Akamai 答非所问，Fastly 只给建议，只有 Cloudflare 动真格上了全局防御，而且在我报绕过之后迅速修。现在这版，我认为它是默认配置最安全的。  
  
但有个清醒认知必须说：  
这套缓解，永远不可能让托管在 Cloudflare 上的站点对缓存投毒"免疫"。它顶多挡住用那几个最流行头的攻击。头那么多，漏网的永远有。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qg1MKHx3jGFGhjoFYdU9dvSH13DRL7WLlkFy4mCabYmYmmo9wF10Q4hJ5LEFn8PbnwCziab1RGbXQfibdY7gXlu1Cibu9SaIDTp0tP3dapM4vI/640?wx_fmt=png&from=appmsg "")  
## 五、个别公司的"补丁翻车"更精彩  
  
厂商之外，一些公司自己动手修补时也闹了笑话，而且笑话的代价是——拒绝服务。  
  
翻车一：用"可缓存的拦截响应"挡攻击，结果造出 DoS。  
  
这毛病 WAF 也会犯。比如www.tesla.com用的某个 WAF，会拦掉任何请求头里带 burpcollaborator.net 字符串的请求:  
```
GET /en_GB/roadster HTTP/1.1
Host: www.tesla.com
Any-Header: burpcollaborator.net
HTTP/1.1 403 Forbidden
Access Denied. Please contact waf@tesla.com
```  
  
本来是想防侦察对吧？可如果这个 403 响应被缓存了，之后就变成这样:  
```
GET /en_GB/roadster HTTP/1.1
Host: www.tesla.com
HTTP/1.1 403 Forbidden
Access Denied. Please contact waf@tesla.com 
```  
  
任何一个正常访客想进这个页面，全被挡在门外。  
攻击者一次请求，把整页打成了全员 403。这哪是防御，这是帮攻击者搞定了 DoS。  
  
翻车二：白名单只管了 host，忘了 port 也被同一个头控制。  
  
有个目标，把 request.host 变量（由 X-Forwarded-Host 填充）做了白名单，只允许自家域名。思路没错，但没注意到：  
这个头同时也能填充request.port。  
  
于是:  
```
GET / HTTP/1.1
Host: redacted.com
X-Forwarded-Host: redacted.com:123
HTTP/1.1 301 Moved Permanently
Location: https://redacted.com:123/ 
```  
  
一个 :123 后缀，就让站点永久重定向到一个根本没在监听的端口。这响应一旦被缓存，全站访客都被导到死胡同——  
持久性 DoS，而且比上一个还难发现。  
## 六、治本，终究得靠框架作者  
  
说到底，临时打补丁就是这么别扭。最该出手解决"最常见那类投毒"的，是 Web 框架的作者们。  
  
好消息是有人行动了：  
Django 和 Flask 这些年已经禁用了对这些头的支持。坏消息是，  
Ruby on Rails 被人反复提醒，却一直拖到最近才开始动手修。  
  
这也给所有开发者提个醒：别等被点了名才动，你用的框架支不支持这些头，自己心里得有数。  
## 七、Param Miner 更新提醒（重要）  
  
最后是个工具提醒。  
Param Miner有一波大更新，到burpsuite的插件市场下载更新就行。  
  
这个更新里面最关键的一条：  
默认关掉了那个静态的fcbz缓存破坏器——因为它在某些站点上会搞坏功能。  
  
这就意味着：以后你用浏览器或 Repeater 去试缓存投毒时，  
得自己手动加一个缓存破坏器参数，不然你的请求可能正好命中真实访客的缓存键，一不小心就把毒投给了别人。  
  
具体怎么加，更新说明里会写，记得去看。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qg1MKHx3jGHDWmUPB9ibicws8D29PW1puiaUmVsYfypVovOlkEm9vfu8Xiaq3nsbIHzico6ztlHXLaGr9bS3k9jPEpoAgNYHlAUniaNNajibdibjCds/640?wx_fmt=png&from=appmsg "")  
  
📌  
这篇"防御绕过实录"要是让你后背一凉（或会心一笑），不妨:  
  
👍  
点个赞—— 让更多同行看到，补丁也可能变成新坑  
  
⭐  
关注我—— 缓存投毒系列持续输出，绕过与防御一线复盘  
  
🔁  
转给做运维 / 安全的朋友—— 那两个 DoS 翻车，他们说不定正踩着  
  
祝大家好运，保持安全。🛡️  
  
