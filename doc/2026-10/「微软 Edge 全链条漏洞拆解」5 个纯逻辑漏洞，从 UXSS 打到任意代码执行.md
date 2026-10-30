#  「微软 Edge 全链条漏洞拆解」5 个纯逻辑漏洞，从 UXSS 打到任意代码执行  
原创 句芒安全实验室
                    句芒安全实验室  句芒安全实验室   2026-10-30 01:57  
  
2026 年的 Pwn2Own Berlin，Orange Tsai 用五个漏洞加两个 Windows 系统特性，把 Microsoft Edge 从"打开一个网页"打到了"在受害者机器上执行任意程序"。赏金 17.5 万美元，年底拿了 Pwnie Award 的史诗级成就奖。  
  
这套链有几个数字值得先摆出来：全程两个月，零 AI 辅助，利用代码不到 100 行，没有一个内存破坏漏洞。  
  
最后这条最关键。过去十年浏览器攻防的主战场是内存破坏：UAF、类型混淆、JIT 优化错误，攻防双方都在堆沙箱、堆 CFI、堆分区分配器。而这套链从头到尾没碰过一块被释放的内存，它打的全部是"谁有权决定这次导航"这类逻辑判断。  
  
真正让人后背发凉的是第一个漏洞的成因——它不是某个程序员写错了，而是 Chromium 的默认值本身就是一个陷阱，连 Chromium 自己的代码注释都把这个默认行为写在了那里。  
  
![一条链的九个环节：五个漏洞 + 两个 Windows 特性 + 一个服务端配置错误](https://mmbiz.qpic.cn/mmbiz_png/J2hBCjr4LftWm0Q45SqXwbrlShw2UCgSQRYK3WwDFnFk9Uz7ribCRFfhbKonzOqFr0eYe5a4NE0qXEr3SWVMcO0H7TpSfbuiaWYVkWBCwvxXQ/640?wx_fmt=png "一条链的九个环节：五个漏洞 + 两个 Windows 特性 + 一个服务端配置错误")  
## 一、为什么从一个没人看的组件开始  
  
Orange 的切入点不是"找 Edge 的漏洞"，是"找浏览器里被产品定制过、但没人审过的地方"。  
  
他先看的是 Chromium 的导航机制。浏览器里所有"跳转"——地址栏输入、点链接、location.href  
、window.open  
、重定向——最终都要经过浏览器进程里的一层组件，叫导航节流器（Navigation Throttles）。这层东西驻留在浏览器进程，能拦下渲染器发起的每一次导航，也能改写、取消、重新发起。  
  
一个数字说明了这个角落有多冷：过去五年 Chrome 修了 3000 多个 CVE，和导航节流器相关的不到 20 个。也就是说，修复量的 0.6% 落在这个组件上，而它经手的是 100% 的导航。  
  
然后他做了一件很"研究"的事：不去审 Chromium 原生的节流器，去审 Edge 在这层上加的东西。Edge 为了支持企业级的 Entra ID 条件访问（Conditional Access），自己实现了一个 EdgeCAGuidanceNavigationThrottle  
。这个节流器的功能很具体：当 Edge 发现用户在微软站点用工作账户登录时，引导用户切到正确的配置文件。  
  
触发方式是一个特定格式的 URL：  
```
https://guidance-ca-error/?email={目标邮箱}&switch_profile={跳转URL}
```  
  
节流器会拿 email  
 参数和当前配置文件的邮箱比对。不匹配就弹窗引导切换；匹配就从 switch_profile  
 里取出目标 URL，用 Chromium 的内部导航 API 发起一次新导航：  
```
NavigateParams params(browser, switch_profile_url, ui::PAGE_TRANSITION_LINK);params.disposition = WindowOpenDisposition::CURRENT_TAB;params.window_action = NavigateParams::SHOW_WINDOW;Navigate(&params);
```  
  
注意这里的三件事：URL 来自攻击者可控的查询参数；导航由浏览器进程发起；disposition  
 是"当前标签页"。  
  
Orange 一开始没把这当回事。他在原文里写得很坦白：这充其量是个强化版的开放重定向，虽然能跳到 edge://settings  
、devtools://  
 甚至 javascript:  
 伪协议，但触发点必须在 login.live.com  
 上，JS 执行还是困在同一个上下文里，最多算自 XSS。这个漏洞在他笔记里躺了好几个星期。  
## 二、第一个洞：无源导航  
  
转折点是一个很朴素的问题：节流器在浏览器进程里处理，新导航也是浏览器发起的，但触发它的页面在渲染器进程里。代码里没有任何一处把"这次导航"和"那个渲染器"绑在一起。那么 CURRENT_TAB  
 指的到底是哪个标签页？  
  
他设计了一个八秒延迟的实验：触发节流器之前先 setTimeout  
 八秒，在这八秒里手动切到另一个标签页（比如 google.com），然后看会发生什么。  
  
结果是：**在 google.com 上执行了攻击者的 JavaScript。**  
  
原因在 Chromium 的 Navigate()  
 里，三段代码连起来看：  
```
// 1. 没指定源 WebContents，就取当前活动标签页if (!params->source_contents && params->browser) {  params->source_contents = params->browser      ->GetBrowserForMigrationOnly()      ->tab_strip_model()      ->GetActiveWebContents();   // ← 谁在前台就用谁}// 2. CURRENT_TAB 的情况下，目标 = 源if (!contents_to_navigate_or_insert) {  if (params->disposition != WindowOpenDisposition::CURRENT_TAB) {    // ...  } else {    DCHECK(params->source_contents);    contents_to_navigate_or_insert = params->source_contents;  }}// 3. 用这个目标发起导航navigation_handle = contents_to_navigate_or_insert->GetController()    .LoadURLWithParams(load_url_params);
```  
  
source_contents  
 为空、disposition  
 是 CURRENT_TAB  
——这两个条件正好都是默认值。于是"谁发起的"这个问题被回答成了"谁现在在前台"。  
  
这为什么危险？因为这次导航是在浏览器进程里发起的，不经过渲染器的 CSP 检查。Orange 在原文里说得很直接：这个 UXSS 由浏览器进程直接发起，可以无视任何 CSP 限制，能在 google.com、facebook.com、icloud.com 或任何站点上执行 JavaScript。  
  
他自己的判断是：**这应该被视为一个全新的 UXSS 攻击面。**  
  
![无源导航的执行路径：渲染器触发 → 浏览器进程 Navigate() → 默认回填当前活动标签页](https://mmbiz.qpic.cn/mmbiz_png/J2hBCjr4LfsjBehbEVEXZQIfLiacRNN9hETQNzc9BeKqVLicwpLfNIsTJw4fAevhnsTeQNhjeM6WKPq1fATjzAne036b359vZSztsxgF7vQIU/640?wx_fmt=png "无源导航的执行路径：渲染器触发 → 浏览器进程 Navigate() → 默认回填当前活动标签页")  
## 三、四个挑战，每一个都是"产品设计让的路"  
  
有了 UXSS 原语还不够。真实攻击要满足四个条件，Orange 是一个一个拆的，拆法很值得看。  
  
**挑战一：节流器要求从微软自有域名触发。**  
  
他去看源验证怎么写的：  
```
content::WebContents* web_contents = navigation_handle()->GetWebContents();const GURL last_committed_url = web_contents->GetLastCommittedURL();if (!IsSupportedLoginUrl(last_committed_url))  return content::NavigationThrottle::CANCEL_AND_IGNORE;
```  
  
GetLastCommittedURL()  
 拿的是"这个 WebContents 当前的 URL"。看起来是标准写法，但它只覆盖"当前窗口里的页面发起的导航"，完全没考虑跨窗口。所以任意站点上执行这两行就绕过了：  
```
popup = window.open('https://login.live.com/', '_blank');popup.location = `https://guidance-ca-error/?email=...&switch_profile=...`;
```  
  
这就是 Orange 反复强调的那句话：验证用的坐标系，和实际被验证的对象，不是同一个。  
  
**挑战二：弹窗拦截。**  
  
现代浏览器默认拦 window.open  
。但 Edge 为了内部使用，硬编码了一个弹窗白名单。于是：  
```
window.open("https://malicious.tld", "_blank");        // 被拦window.open("about:blank#quickAuthPopup", "_blank");   // 放行// 拿到句柄之后再导航popup.location = "https://malicious.tld";
```  
  
一个 about:blank#quickAuthPopup  
 字符串，把整层弹窗防护绕过去了。  
  
**挑战三：目标浏览器没有登录的配置文件。**  
  
节流器要求 Edge 里有一个已登录的用户配置文件，且 email  
 参数和它匹配。攻击者不能假设受害者已经登录过浏览器。  
  
Edge 生态里有两个东西把这条路铺平了：OneAuth 允许任何 Edge WebView 应用在访问微软站点时自动用当前 MSA 凭据登录；而 Windows 11 25H2 开始，系统初始设置（OOBE）强制要求 MSA 账户登录。两件事叠起来的结果是：**即使用户从没在浏览器里建过配置文件，只要访问过 microsoft.com，Edge 就会自动建配置文件并完成登录。**  
  
**挑战四：怎么拿到受害者的邮箱。**  
  
这一步是纯 Web 漏洞挖掘。他在微软生态里翻了几个小时，找到 assets.msn.com  
（内嵌在 edge://new-tab-page-online  
 里）会返回当前登录用户的邮箱，而且这个站点的 CORS 配置是错的：  
```
const resp = await fetch(  `https://assets.msn.com/service/community/users/me?profile=social&it=edgeid`,  {    method: 'GET',    mode: 'cors',    credentials: 'include',      // 带凭据    referrerPolicy: 'no-referrer',    headers: { 'Accept': 'application/json' },  });const body = await resp.json();console.log(body.value[0].email);   // 拿到邮箱
```  
  
到这里，四个条件齐了：任意站点能触发、弹窗能过、受害者自动登录、邮箱能读到。零点击 UXSS 成型。  
## 四、从 UXSS 到任意代码执行  
  
拿到 UXSS 之后，第一反应是"去特权 WebUI 页面上执行 JS"。这条路 Chromium 早就堵了。渲染器的 Mojo 处理层有一份黑名单：  
```
void LocalFrame::LoadJavaScriptURL(const KURL& url) {  if (SchemeRegistry::ShouldTreatURLSchemeAsNotAllowingJavascriptURLs(          GetSecurityContext()->GetSecurityOrigin()              ->GetOriginOrPrecursorOriginIfOpaque()->Protocol())) {    return;   // 特权协议直接不执行  }}
```  
  
黑名单里是 edge  
、chrome-extension  
、chrome  
、chrome-untrusted  
、chrome-distiller  
、chrome-error  
、chrome-native  
、chrome-search  
、devtools  
 这一串。基本把能想到的特权页面都列进去了。  
  
**突破口是 Edge 的沉浸式阅读器。**  
  
Edge 的 Immersive Reader 是从 Chromium 早期版本分叉出来、由微软大量定制开发的组件。它没有用 Chrome 的 chrome-distiller://  
，也没有用被拉黑的 chrome-untrusted://  
，而是用了自己的 read://  
 协议。而微软忘了把 read://  
 加进那份 JavaScript 黑名单。  
  
于是 UXSS 在这个特权 WebUI 上能跑。跑起来之后能碰到原生 C++ 对象，其中一个 API 是：  
```
chrome.edgeFeedbackPrivate.zipDiagnosticLogFiles(  'LOL.ZIP',  '{"logFiles": {"ORANGE": "AAAAAAAA"}, "deviceInfo": {1337: 1337}}');
```  
  
这个 API 的行为是：建临时目录 → 把内容写进 ORANGE.json  
 → 打包成 ZIP → 上传到微软服务器 → 清理临时目录。  
  
他试了 ../  
 路径穿越——可以，能跳出临时目录。麻烦的是 .json  
 后缀会被自动加上。然后他用了空字节：**后缀被截断，任意路径任意文件写成立。**  
  
最后一步是把文件写变成代码执行。他先排除了 DLL 侧加载：写入内容必须是合法 UTF-8 字符串，而 64 位 PE 头没法在 UTF-8 约束下构造出来。  
  
替代方案是 Edge 启动时加载的一个组件：AutoLaunchProtocolsComponent  
。它会从一个**用户可写目录**  
读 protocols.json  
，这个文件定义"哪些源有权启动哪些协议处理器"：  
```
AppData\Local\Microsoft\Edge\User Data\AutoLaunchProtocolsComponent\1.0.0.10\protocols.json
```  
  
覆盖它，就能从任意源启动任意协议处理器。重新加载的方式也现成：用前面拿到的"浏览器进程发起导航"能力，跳到 edge://restart  
。  
  
最后一个协议选的是 telnet://  
。Windows 的 telnet 处理器会去 PATH 里找 telnet.exe  
，而这个可执行文件默认没装。PATH 里有一项是当前用户可写的：  
```
C:\Users\lowpriv\AppData\Local\Microsoft\WindowsApps
```  
  
放一个 32 位微型 exe 进去，telnet://  
 一触发，代码执行完成。整条链的利用代码不到 100 行。  
## 五、哪些是漏洞，哪些不是  
  
这一节是本文和大部分"Edge 被打穿"的标题党最大的区别。九个环节里，性质并不一样。  
<table><thead><tr><th style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top; font-weight: bold; background-color: rgba(122, 234, 240, 0.094); color: rgb(8, 155, 163);"><section><span leaf="">环节</span></section></th><th style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top; font-weight: bold; background-color: rgba(122, 234, 240, 0.094); color: rgb(8, 155, 163);"><section><span leaf="">性质</span></section></th><th style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top; font-weight: bold; background-color: rgba(122, 234, 240, 0.094); color: rgb(8, 155, 163);"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">Navigate()</span></code><section><span leaf=""> 默认取活动标签页</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Chromium 代码缺陷</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">默认值即陷阱，</span><code><span leaf="">CURRENT_TAB</span></code><span leaf=""> + 空 </span><code><span leaf="">source_contents</span></code><span leaf=""> 正好是默认组合</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">GetLastCommittedURL()</span></code><section><span leaf=""> 只认当前窗口</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Edge 逻辑缺陷</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">验证坐标系错位，跨窗口即失效</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">about:blank#quickAuthPopup</span></code><section><span leaf=""> 白名单</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Edge 硬编码设计缺陷</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">为内部功能开的口子，被外部复用</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">OneAuth + OOBE 强制 MSA</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Windows 产品设计</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">不是漏洞，但它把&#34;必须已登录&#34;这个前提消掉了</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">assets.msn.com</span></code><section><span leaf=""> CORS 配置错误</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">服务端配置问题</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">任何浏览器都中招，严格说不算浏览器漏洞</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">read://</span></code><section><span leaf=""> 未进 JS 黑名单</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Edge 自定义协议遗漏</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">分叉组件没跟上原生黑名单</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">zipDiagnosticLogFiles</span></code><section><span leaf=""> 路径穿越</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Edge 特权 API 缺陷</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">../</span></code><section><span leaf=""> 加空字节，任意路径任意文件写</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">protocols.json</span></code><section><span leaf=""> 用户可写</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Edge 组件加载设计缺陷</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">配置文件放在可写目录，且无完整性校验</span></section></td></tr><tr><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><code><span leaf="">telnet.exe</span></code><section><span leaf=""> 从 WindowsApps 加载</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">Windows 特性</span></section></td><td style="font-size: 10px; padding: 9px 12px; line-height: 22px; border: 1px solid rgb(8, 155, 163); vertical-align: top;"><section><span leaf="">PATH 里有用户可写目录，不是漏洞</span></section></td></tr></tbody></table>  
这张表值得单独看一会儿。九项里，只有三项是"代码写错了"（无源导航、跨窗口验证、路径穿越），两项是"产品设计给攻击者让了路"（OneAuth/OOBE、telnet PATH），一项是"别人的配置错误"（CORS），三项是"自定义组件没跟上原生防护"（read://、protocols.json、弹窗白名单）。  
  
也就是说，这套链能成立，靠的不是九个 bug，而是**三类东西的叠加**  
：代码默认值、产品设计取舍、以及分叉组件与原生防护之间的时间差。  
  
这一点在安全研究里经常被忽略。大家习惯数 CVE，但真正决定一条链能不能打通的，往往是那些"不算漏洞"的部分。  
  
![九个环节的三种性质：只有三项是代码写错了](https://mmbiz.qpic.cn/sz_mmbiz_png/J2hBCjr4Lfuyc8JyBAGQhdudLEnlmVq3fsct5WYDyTclakV9YIJBiciaJwsJefc4RHNcdbhgfjmUzXjXEVfRhqsnG4lGCVezEqzFcVXTsbmek/640?wx_fmt=png "九个环节的三种性质：只有三项是代码写错了")  
## 六、方法论：三条可以搬走的东西  
  
Orange 在原文里没有把这套方法写成教条，但三条东西是可以直接拿去用的。  
  
**第一条：找被产品定制过的组件。**  
  
原生 Chromium 的节流器被审了很多年，但 Edge 为了 Entra ID 加的那个不是。它不在 Chromium 的代码库里，不在 Chrome 的 CVE 列表里，也不在大多数研究者的审计范围里。被定制过的组件 = 新的攻击面，这个规律在任何"基于开源做产品"的软件上都成立。  
  
**第二条：看它是否在高权限进程里做决策，但输入来自低权限侧。**  
  
节流器在浏览器进程里跑，输入是渲染器里的一个 URL。这个组合本身不是错，错的是它没有把"这个 URL 是谁给的"一起带过来。凡是"高权限侧做决策、低权限侧提供参数"的地方，都值得问一句：这个参数的身份，跟着一起过来了吗？  
  
**第三条：看它的验证用的是什么坐标系。**  
  
GetLastCommittedURL()  
 是"当前窗口"的坐标。跨窗口、跨标签页、跨 iframe、跨进程之后，这个坐标还成立吗？Orange 的总结很精确：不是验证逻辑写错了，是验证的对象和实际被验证的对象不是同一个。  
## 七、冷静判断：这套链在真实世界里有多难  
  
必须说清楚几件事，否则这篇文章就变成了标题党。  
  
**第一，它不是零点击的。**  
 挑战一和挑战二需要受害者的浏览器打开攻击者页面，挑战三需要受害者已经登录过微软账户，第一个洞的 UXSS 在演示里还需要"手动切到另一个标签页"这个动作。Orange 自己在原文里说，八秒延迟是为了给用户切换标签页的时间。真实攻击里这个窗口要靠社工或者其他手段撑开。  
  
**第二，它的价值不在"能打穿"，在"证明了纯逻辑链条可行"。**  
 内存破坏类漏洞最难的是利用构造，纯逻辑漏洞最难的是发现。Orange 的原话是：一旦找齐所有必要漏洞，把它们串联起来反而相对容易。这套链真正证明的是：一个浏览器可以被五个逻辑错误打穿，不需要一行 shellcode。  
  
**第三，Pwn2Own 的演示环境有额外条件。**  
 重启步骤是为了满足比赛要求，实际攻击不需要 edge://restart  
。反过来说，比赛环境也给了攻击者一些便利（比如可控的配置文件目录），真实环境要重新评估。  
  
**第四，Orange 自己在往前推。**  
 他提到已经在把同样的思路往 Chrome 上移植（接近完成），也在做"从 calc.exe 反向劫持 Edge"这类 Windows RCE。也就是说，这套方法不是一次性成果，是一条还在延伸的线。  
## 八、和上一篇的关系  
  
上一篇我们拆了 PwnEscape：Linux 内核的校验和延迟元数据被 guest 反向利用，32 字节泄露打穿 KVM。那篇的公式是"低信任侧提供元数据 + 宿主延迟写回 + 中间包被重解释"。  
  
这一篇的公式是另一条：**低信任侧提供参数 + 高信任侧做决策 + 验证坐标错位**  
。  
  
两条公式的共同点是同一个：信任在传递过程中被换了主体。PwnEscape 换的是"这个包还是不是当初那个包"，Edge 这套换的是"这次导航还是不是当初那个人发起的"。  
  
这也是为什么两篇文章都值得看：它们打的都不是内存，是"谁说的算"。  
## 最后  
  
这套链最值得记住的不是 17.5 万美元，也不是 Pwnie Award，是那句代码：  
```
if (!params->source_contents && params->browser) {  params->source_contents = params->browser      ->tab_strip_model()->GetActiveWebContents();}
```  
  
一个默认值，把"谁发起的导航"这个问题回答成了"谁现在在前台"。而这个默认值不是某个人偷偷加的，是 Chromium 一直以来的写法，注释里还写着"这必须在 GetBrowserAndTabForDisposition() 有机会替换 browser 之前发生"。  
  
安全研究里最贵的东西往往不是最复杂的漏洞，是那些"所有人都看过的代码里，所有人都默认对的那一行"。  
  
如果你也在做浏览器或者客户端的逻辑审计，欢迎把这篇转给那个正在审 Chromium 的同事。关注「句芒安全实验室」，后面继续拆这类"不需要内存破坏也能打穿边界"的东西。  
  
