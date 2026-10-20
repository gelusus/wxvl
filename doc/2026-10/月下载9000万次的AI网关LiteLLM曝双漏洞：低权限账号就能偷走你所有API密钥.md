#  月下载9000万次的AI网关LiteLLM曝双漏洞：低权限账号就能偷走你所有API密钥  
原创 欢迎关注→
                    欢迎关注→  安全客   2026-10-20 06:53  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ribStUdgfRibSMjwdLibCDXjHJ9fYhm4Tqy3qqUa1KOLMOh4Uy5N26uicz79QlnwyptRs9usBhJibc7X84VAGIxLln0nUUrTB7R7ah0ibytgbgfj0/640?from=appmsg "")  
  
Check Point披露LiteLLM两大漏洞CVE-2026-35029和CVE-2026-59823，低权限用户可接管AI网关、读取服务器文件、窃取企业存储的AI厂商API密钥，1.83.9以下版本全受影响。  
  
一家互联网公司的AI平台工程师最近遇到了件怪事：他们内部平台上，一个刚入职的实习生，居然调通了GPT-4的接口，用的还是公司统一采购的企业级API密钥。查了一圈才发现，问题不在权限配置，而在他们部署的LiteLLM网关——这个组件里藏着一个漏洞，任何拥有最低权限账号的人，都能把整个网关的配置改得面目全非。  
  
这不是孤例。Check Point Research（CPR）最近披露了LiteLLM的两个漏洞，一个月下载量超过9000万次的AI网关，在1.83.9版本之前，几乎是门户大开的状态。更要命的是，LiteLLM不是某个小众工具，它是连接企业应用和OpenAI、Anthropic等140家AI厂商的标准接口，GitHub星标超过61000，被嵌入了无数内部平台和AI应用里。  
## 一、LiteLLM是什么：AI时代的"总开关"，也是"总钥匙"  
  
先搞清楚LiteLLM是干嘛的。简单说，它是一个开源的AI网关，作用是把企业的应用和各家大模型厂商接到同一个接口上。开发者不用关心OpenAI的API怎么调、Anthropic的密钥怎么管，LiteLLM统一处理路由、负载均衡、密钥管理、用量监控和成本控制。  
  
正因为它是"统一入口"，所以LiteLLM手里握着的东西极其敏感：企业存储在里面的，是OpenAI、Anthropic、Azure OpenAI等厂商的真实API密钥。这些密钥一旦泄露，攻击者不仅能白嫖企业的AI算力，还能通过API调用记录反推企业的业务逻辑、用户数据甚至商业机密。  
  
Check Point发现的两个漏洞，恰恰就砸在了这个"总钥匙"上。  
## 二、CVE-2026-35029：低权限用户，一键改配置  
  
第一个漏洞是权限控制缺陷，编号CVE-2026-35029，严重性评分8.7/10（高危）。  
  
问题出在LiteLLM的/config/update  
端点。这个接口本来是用来动态修改网关运行时配置的，但代码里压根没检查调用者的角色。这意味着什么？任何已经通过身份验证的用户——哪怕只是一个普通员工、一个实习生、一个被钓鱼攻击拿到的低权限账号——都可以向这个端点发送请求，修改网关的任意配置。  
  
改配置能干什么？攻击者可以：  
- 读取服务器上的任意文件，包括存储API密钥的配置文件  
  
- 把请求重定向到自己控制的服务器，截获所有经过网关的AI调用  
  
- 直接接管管理员账户，把整个网关变成自己的傀儡  
  
- 在网关上植入恶意代码，实现远程代码执行（RCE）  
  
CPR的研究员说得一针见血："配置的权限"和"实际执行的权限"之间的差距，正是这类漏洞造成最大破坏的地方。很多企业在管理后台里设置了"只读用户""普通用户""管理员"的分级，但代码里没做校验，一切形同虚设。  
## 三、CVE-2026-59823：SSRF漏洞，内网穿透的敲门砖  
  
第二个漏洞是服务端请求伪造（SSRF），编号CVE-2026-59823，评分5.3/10（中危）。虽然评分不高，但配合第一个漏洞使用，威力翻倍。  
  
SSRF的本质，是让服务器代替攻击者去访问内部资源。LiteLLM作为AI网关，天然需要访问外部API，但如果不对目标地址做严格校验，攻击者就能构造恶意请求，让网关去访问内网的数据库、元数据服务、或者其他不该暴露的系统。  
  
在云上部署的场景里，SSRF往往是获取云服务器临时凭证、进而横向移动的第一步。一个中危的SSRF，加上一个高危的配置修改，攻击者就能从"低权限用户"一路打到"内网核心数据"。  
## 四、影响范围：1.83.9以下全中招，而且藏得很深  
  
LiteLLM官方已经在1.83.9版本中同时修复了这两个漏洞。所有低于这个版本的实例，都在风险范围内。  
  
但真正的麻烦在于"间接运行"。LiteLLM不是那种需要单独部署、单独运维的大型软件，它经常被嵌入到其他平台里：企业内部的AI中台、SaaS化的AI应用、开发者自己写的内部工具。很多团队可能根本不知道自己的系统里跑着LiteLLM，更谈不上及时升级。  
  
CPR在报告里特别提醒：安全团队需要排查"间接运行"的LiteLLM实例。那些从未单独部署过LiteLLM的环境，可能正因为嵌入了某个AI框架、某个内部平台，而暴露在风险之中。  
## 五、修复与排查：升级只是第一步  
  
**立即升级。**  
把LiteLLM升级到1.83.9或更高版本。如果通过pip安装，执行：  
```
```  
  
**审查访问权限。**  
检查谁有权限访问LiteLLM的管理接口，特别是/config/update  
端点。确保只有proxy_admin  
角色的用户才能调用配置修改接口。  
  
**轮换API密钥。**  
如果发现网关曾被未授权访问，或者无法确认历史访问记录，立即轮换存储在LiteLLM中的所有AI厂商API密钥。OpenAI、Anthropic、Azure OpenAI，一个都不能漏。  
  
**排查隐藏实例。**  
在企业内部搜索所有依赖LiteLLM的项目，包括那些通过SDK、框架间接引用的场景。检查requirements.txt、package.json，搜索"litellm"关键词。  
  
**监控异常调用。**  
在网关上启用详细的访问日志，监控是否存在异常的配置修改记录、异常的API调用模式。  
## 六、写在最后  
  
LiteLLM的漏洞，是AI基础设施安全的一个缩影。当企业蜂拥而上拥抱大模型时，往往把注意力放在了模型能力、Prompt工程、业务场景上，却  
忽略了底层网关、密钥管理、访问控制这些"老问题"。  
  
AI时代的安全，没有新魔法，全是基本功。  
一个低权限账号就能接管整个AI网关，偷走所有API密钥——这样的故事，不该在2026年还在上演。  
  
检查你的LiteLLM版本，现在。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/o0buL62hK7M8RnVz7mqRVDRkqm2sJeT2icM4WyR7kMkHpLVaicR3tJ4gr5kIb4zje9lXgd5PuOw42Z5KtathltcQ/640?from=appmsg "")  
  
**END**  
  
推荐阅读  
  
[3.5万次请求、翻找泄露密钥：失控AI智能体围猎高德地图，运营者却毫不知情](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790587&idx=1&sn=38aa20efb77571be0a0fed789da1789f&scene=21#wechat_redirect)  
  
  
2026-10-19  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790587&idx=1&sn=38aa20efb77571be0a0fed789da1789f&scene=21#wechat_redirect)  
  
  
[51个AI代理，61秒植入32个后门：黑客已经把AI训练成红队主力](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790582&idx=1&sn=1ba627a29325739cbc67be4535160b89&scene=21#wechat_redirect)  
  
  
2026-10-16  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790582&idx=1&sn=1ba627a29325739cbc67be4535160b89&scene=21#wechat_redirect)  
  
  
[黑客把冷柜调到50℃，数百家超市一夜报废：工控安全最扎心的一课](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790577&idx=1&sn=35269abfde7cd79f9ff3410c8de5d269&scene=21#wechat_redirect)  
  
  
2026-10-15  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790577&idx=1&sn=35269abfde7cd79f9ff3410c8de5d269&scene=21#wechat_redirect)  
  
  
[7分钟加密3.6PB，运维7小时没找到勒索信：日本国家级云平台被黑全过程](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790572&idx=1&sn=3630c6de55a1d65ad6888c1b364ab715&scene=21#wechat_redirect)  
  
  
2026-10-13  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790572&idx=1&sn=3630c6de55a1d65ad6888c1b364ab715&scene=21#wechat_redirect)  
  
  
[AI第一次"报假案"：Claude向费城警方提交虚假凶杀线索，两个月后才被发现](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790567&idx=1&sn=b7a15f159a3cc5947da78132d8f2b290&scene=21#wechat_redirect)  
  
  
2026-10-12  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790567&idx=1&sn=b7a15f159a3cc5947da78132d8f2b290&scene=21#wechat_redirect)  
  
  
**更多AI知识，请关注Aiker World社区**  
  
  
