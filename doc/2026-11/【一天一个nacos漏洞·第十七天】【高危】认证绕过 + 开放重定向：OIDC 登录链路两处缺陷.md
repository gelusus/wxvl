#  【一天一个nacos漏洞·第十七天】【高危】认证绕过 + 开放重定向：OIDC 登录链路两处缺陷  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-24 10:39  
  
   
  
# 【一天一个nacos漏洞·第十七天】【高危】认证绕过 + 开放重定向：OIDC 登录链路两处缺陷  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个都在 **OIDC 登录链路**  
上，类型不同但位置对称：  
  
一个是**认证绕过**  
——email_verified=false  
、sub  
 完全不同的陌生人，只要 email 撞上，在 Nacos 眼里就是同一个人。  
  
一个是**开放重定向**  
——回调地址是用请求里的 Host  
 头拼的，改个头，就能把人送到任意站点。  
  
**一个认错了人，一个送错了门。**  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与真实令牌均未写入本文。内容仅用于学习与防御加固。  
  
  
01  
  
一、OIDC 登录链路长什么样  
  
```
① 用户点"登录"
② Nacos 拼一个回调地址，带上 client_id / state / nonce，把浏览器踢到 IdP
   → 用户在 IdP 完成认证
③ IdP 带着 code 跳回 callback
④ Nacos 拿 code 换 token，验签，读出"你是谁"，建立自己的会话

```  
  
  
这条链上有两个关键判断：  
  
**第 ③ 步：跳到哪个地址**  
（由 Nacos 的 Host  
 决定）**第 ④ 步：这个人是谁**  
（由 IdP 的 token 声明决定）  
  
今天这两个洞，正好一个打第 ③ 步、一个打第 ④ 步。  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><section><span leaf=""><br/></span></section></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">VE-0401 · 主体混淆</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">VE-0762 · Host 头注入</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">8.2</span></strong></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">8.0</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">类型</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">认证绕过 / Principal Confusion</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">开放重定向（CWE-601）/ Host 头注入</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">打哪一步</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">④ 认人</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">③ 跳转</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">最低权限</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">已认证的 IdP 用户</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">未登录</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0401：不同的人，被判成同一个人  
### 2.1 身份是怎么从 token 里抠出来的  
  
OIDC 的 token 里有一堆声明。Nacos 要选一个当"用户名"，它的逻辑是**四级回退**  
：  
  
```
public String extractUsername(JWTClaimsSet claims) {
    // ① 配置指定的 claim（默认 preferred_username）
    Object username = claims.getClaim(usernameClaim);
    if (username != null) return username.toString();

    // ② preferred_username
    String preferredUsername = (String) claims.getClaim("preferred_username");
    if (StringUtils.isNotBlank(preferredUsername)) return preferredUsername;

    // ③ email —— 注意这里
    String email = (String) claims.getClaim("email");
    if (StringUtils.isNotBlank(email)) return email;

    // ④ 最后才回退 sub
    return claims.getSubject();
}

```  
  
  
**sub 排在最后。**  
 而在 OIDC 规范里，sub  
 才是**该 IdP 内唯一且稳定**  
的用户标识——其它字段（email、preferred_username）都是"用户资料"，**可能重名、可能被改、可能没验证**  
。  
### 2.2 第三个分支，少了一个检查  
  
email  
 这个分支，没有读 email_verified  
。  
  
报告做了一件事：**全仓库检索 email_verified**  
。  
  
结果是 **0 处**  
。  
  
也就是说，Nacos 从来没有校验过"这个邮箱到底验证了没有"——一个 email_verified: false  
 的合法签名 token，照样被接受。  
### 2.3 然后 IDENTITY_ID 就这么定下来了  
  
```
// OidcUserMapper
String username = tokenValidator.extractUsername(claims);
user.setUsername(username);
user.setSubject(claims.getSubject());   // ← sub 存下来了，但之后不参与判定

// OidcIdentityProvider
identityContext.setParameter(Constants.Identity.IDENTITY_ID, user.getUsername());   // ← 用 username

```  
  
  
**IDENTITY_ID = username，而 sub 只是存着看的。**  
  
于是产生一个很直接的后果：  
  
```
用户 A：sub=aaa-111，email=alice@corp.example，email_verified=true
用户 B：sub=bbb-222，email=alice@corp.example，email_verified=false

在 Nacos 眼里：这两个是同一个人（IDENTITY_ID 都是 alice@corp.example）

```  
  
  
**两个 sub 完全不同的人，只要邮箱撞上，就是同一个身份。**  
 而邮箱在多数 IdP 里——尤其允许用户自助注册的那些——**是可以自己填的**  
。  
### 2.4 一个非常精确的对照实验  
  
光证明"能读到别人的资源"还不够，得排除"是不是可见性配得太宽"。  
  
报告搭了个最小 IdP，策略刻意设计成：  
  
```
粗粒度资源（控制器 @Secured 层面）  → 允许
@@visibility 开头的细粒度资源        → 拒绝

```  
  
  
然后测同一个受害者资源：  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">谁去读</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">结果</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">原因</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">受害者本人</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">200</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">owner</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">普通用户 bob</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">404</span></strong></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">外部 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">@@visibility</span></code><span style="display: inline;"><span leaf=""> 授权拒绝</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">碰撞者（同 email，不同 sub）</span></strong></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200</span></strong></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">isOwner</span></code><span style="display: inline;"><span leaf=""> 字符串相等，</span></span><strong style="font-weight: 700;color: #27272A;"><span leaf="">直接短路，根本没调外部授权</span></strong></td></tr></tbody></table>  
  
日志也对得上：**只有 bob 出现 denied access to @@visibility/...，碰撞者一条 DENY 都没有。**  
  
看那段判定逻辑：  
  
```
if (isOwner(currentUser, candidate)) {    // 纯字符串相等
    return true;                          // ← 直接返回，后面不执行了
}
...
return checkResourcePermission(candidate, action);   // 外部 IdP 细粒度授权在这里

```  
  
  
**owner 判定在外部授权之前，而且是纯字符串比较。**  
 身份一旦撞上，外部那一层策略根本没机会生效。  
> 📌 这个对照的意义在于：它证明 200 不是"权限给宽了"，而是"**你变成了 owner，所以后面所有检查都不用跑了**  
"。  
  
### 2.5 还能干什么：篡改 + 伪造审计  
  
碰撞之后不只是"能读"：  
  
```
读受害者 PRIVATE 资源        → 200，完整内容
覆盖受害者 Prompt            → 200，受害者读回是 TAMPERED-BY-ATTACKER
以受害者身份创建资源          → 200，DB 里 owner 落成受害者
发布配置                      → 200，config_info.src_user 落成受害者

```  
  
  
最后一条最麻烦：**审计日志里，这次操作记在了受害者名下。**  
  
事后溯源，你会看到"alice@corp.example 修改了配置"——而 alice 什么也没干。  
  
03  
  
三、VE-0762：回调地址是拼出来的  
### 3.1 绝对 URL 从请求元数据来  
  
```
private String buildBaseUrl(HttpServletRequest request) {
    String scheme     = request.getScheme();
    String serverName = request.getServerName();    // ← 就是 Host 头
    int serverPort    = request.getServerPort();
    String contextPath = request.getContextPath();
    return scheme + "://" + serverName + ... + contextPath;
}

```  
  
  
getServerName()  
 在 Tomcat 里直接取自客户端的 Host  
 请求头。  
  
**没有白名单、没有可信 base-url 配置、没有与 IdP 注册值做一致性校验。**  
  
而这个 baseUrl  
 被用在了四个地方：  
  
```
// ① callback 错误分支
response.sendRedirect(buildBaseUrl(request) + "/#/login?error=" + errorMsg);

// ② callback 成功分支
response.sendRedirect(buildBaseUrl(request) + "/#/");

// ③ login：拼成 redirect_uri 发给 IdP
String callbackUrl = buildBaseUrl(request) + "/v1/auth/oidc/callback";

// ④ logout：拼成 post_logout_redirect_uri
String postLogoutUri = buildBaseUrl(request);

```  
  
  
**一条请求头，四处影响。**  
### 3.2 三个端点都不需要登录  
  
/v1/auth/oidc/login  
、/callback  
、/logout  
 三个端点**都没有 @Secured 注解**  
，而鉴权过滤器对无注解的方法直接放行。  
  
所以：**未登录就能打。**  
  
实测（只改一个 Host  
 头）：  
  
```
GET /v1/auth/oidc/callback?error=x       Host: evil.example
→ 302  Location: http://evil.example/#/login?error=x

GET /v1/auth/oidc/callback               Host: evil.example
→ 302  Location: http://evil.example/#/login?error=Missing+authorization+code

GET /v1/auth/oidc/login                  Host: evil.example
→ 302  Location: http://〈idp〉/authorize?...&redirect_uri=http://evil.example/v1/auth/oidc/callback

GET /v1/auth/oidc/logout?redirect=true   Host: evil.example
→ 302  Location: http://〈idp〉/logout?post_logout_redirect_uri=http://evil.example

```  
  
  
第三条最关键：**发往 IdP 的 redirect_uri 里，域名变成了攻击者的。**  
### 3.3 一个能说明"测试到位"的对照  
  
报告特意试了另一种注入方式：  
  
```
只伪造 X-Forwarded-Host: evil.example（Host 保持正常）
→ 不生效，Location 还是真实主机

```  
  
  
因为这套部署没有启用 forwarded-headers 处理。**这个对照排除了"随便换个头都行"的误判，确认了真正的入口就是 Host。**  
  
顺带一提，Host: evil.example:4444  
 也生效——端口同样取自 Host。  
### 3.4 上限在哪：配合宽松的 IdP 能偷会话  
  
单独看，开放重定向的危害是"钓鱼"。但这套链子能接得更长：  
  
```
① 攻击者用恶意 Host 调 /login，拿到服务端签名的 state / nonce
② 把带 redirect_uri=攻击者域 的授权链接发给受害者
③ 受害者登录后，IdP 把 code 送到攻击者域
④ 攻击者拿 code 调 /callback，Nacos 用 client_secret 换 token
⑤ Nacos 把 accessToken / username 写进 Cookie，并跳到攻击者域

```  
  
  
实测的成功分支：  
  
```
HTTP/1.1 302
Set-Cookie: accessToken=〈token〉; Max-Age=60; Path=/
Set-Cookie: username=〈user〉;    Max-Age=60; Path=/
Location: http://evil.example/#/

```  
  
  
**会话 cookie 下发到了攻击者的域下。**  
> 🔍 这里有个细节值得单独说：state  
 / nonce  
 由服务端签名，**但和 Host 无关**  
。 也就是说攻击者可以先用恶意 Host 自己发起一次 /login  
，拿到合法的 state/nonce，再拿去构造钓鱼链接——**门槛又被降低了一档。**  
  
  
04  
  
四、两个合起来  
  
```
VE-0401：token 说"我是 alice@corp.example"  →  但它可能没验证、可能重名
VE-0762：请求头说"我是 evil.example"        →  它就直接把回调地址写成了这个

```  
  
  
**一个是把不可信的声明当成了身份，一个是把不可信的请求头当成了自己的地址。**  
  
而这两件事在 OIDC 里都有**标准答案**  
：  
- 身份要用 (iss, sub)  
——sub  
 才是 IdP 内唯一稳定的；  
- 自己的地址要用**配置**  
，不能用请求头反推。  
05  
  
五、边界  
  
**0401**  
：要求目标启用 OIDC（默认是 nacos  
 鉴权），且外部 IdP 允许用户控制未验证/重复的 email 或 preferred_username。AC:H 就是这个意思。最低权限是"已认证的 IdP 用户"，匿名 403。  
  
**0762**  
：同样要求 OIDC 模式。开放重定向部分**未登录即可**  
；但要做到"偷会话"，还需要 IdP 没有严格绑定 redirect_uri，以及攻击者域名能正常服务。所以 UI:R、AC:H。  
> 📐 两份都写清了前置条件，没有夸大。但 0401 的那个前提要特别提醒一句：**“IdP 允许用户自己填邮箱”——这在允许自助注册的企业 SSO 里，不是什么罕见配置。**  
  
  
06  
  
六、修复  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">用 username 当身份</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">IDENTITY_ID</span></code><span style="display: inline;"><span leaf=""> 改用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">(iss, sub)</span></code><span style="display: inline;"><span leaf="">；username 只做展示</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">email 分支不验</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">email_verified != true</span></code><span style="display: inline;"><span leaf=""> 直接拒绝，或降级到 sub</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">声明可能重名</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">若必须以 email/preferred_username 映射，要求 IdP 保证唯一且已验证，Nacos 侧做冲突检测</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">owner 存字符串</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">资源 owner 存 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">(iss, sub)</span></code><span style="display: inline;"><span leaf=""> 或稳定 UUID，别存可被用户控制的字符串</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">owner 短路外部授权</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">保留 owner 快路径，但对&#34;刚出现/映射可疑&#34;的主体触发一次外部校验</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">用 Host 拼绝对 URL</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">引入配置项（如 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">base-url</span></code><span style="display: inline;"><span leaf="">），回调/登出地址</span></span><strong style="font-weight: 700;color: #27272A;"><span leaf="">固定由配置生成</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">动态推导无校验</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">若必须推导，Host / X-Forwarded-Host 走</span></span><strong style="font-weight: 700;color: #27272A;"><span leaf="">显式白名单</span></strong><span style="display: inline;"><span leaf="">，禁非法端口与协议</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">redirect_uri 无一致性</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">与 IdP 注册值严格比对，或配置固定 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">redirect-uri</span></code></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">错误跳转回显 Host</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">改成相对路径或可信固定地址</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">网关层</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">统一剥离/校验 Host 与 X-Forwarded-*，管理端口只允许受信源</span></span></td></tr></tbody></table>  
> 💡 一句可以写进 OIDC 接入 checklist 的话：**“身份用 sub，地址用配置。”**  
 这两句能挡掉今天这两个洞，也能挡掉它们的一大票变种。  
  
  
07  
  
写在最后  
  
第十七天，第二次讲 OIDC（第七天讲的是信任链两端：公钥从哪来、撤销认不认）。  
  
今天这两个，问题更靠上一点：**不是密码学坏了，是"声明"被当成了事实。**  
  
OIDC 的本质是"IdP 给你一张签了名的纸条，上面写着你是谁"。所以安全的前提有两个：  
1. **纸条上的字必须可信**  
——email_verified  
 就是在说"这个字我验过了"；  
1. **纸条之外的东西不能当真**  
——Host  
 头不是纸条，它只是送纸条的人自己报的门牌号。  
这两条，今天各破了一个。  
  
而它们共同的可悲之处在于：**校验其实都做了**  
。签名验了、iss  
/aud  
/exp  
 都验了、state/nonce 也签了。**所有密码学该做的事一件没少**  
——只是最后那一步，"把验过的东西翻译成’这是谁’“和"把请求翻译成’我在哪’”，翻错了。  
  
**密码学保证的是"这张纸条没被改过"，不保证"纸条上写的就是事实"。**  
  
下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
：MCP 与 A2A，名字和版本拼成的 key 不唯一。**第四天**  
：配置接口别名错配（9.9），Prompt/Skill 分隔符碰撞（9.1）。**第五天**  
：配置接口认错参数（9.6），AI 资源可见性被 Config 通道绕过（9.6）。**第六天**  
：MySQL 健康检查 SQL 注入（7.7），鉴权密钥进日志（7.8）。**第七天**  
：OIDC 信任链两端——不验来源（8.1），不认撤销（7.4）。**第八天**  
：两处 fail-open——插件缺失即全放行（7.2），空白主体即成管理员（8.1）。**第九天**  
：服务间共享身份——凭据被主动投递（7.4），静态值可重放（8.1）。**第十天**  
：发布审核被架空——审核后换内容（8.1），审核漏扫文件（7.7）。**第十一天**  
：注册表身份由 ip:port 推导——可覆盖注册（7.1），可删除实例（7.1）。**第十二天**  
：集群内部基础设施——锁接口无鉴权（8.2），同步接口退让鉴权（7.3）。**第十三天**  
：版本不可变性被破坏——已发布的能改回（7.1），未审核的能推上（7.7）。**第十四天**  
：凭证撤销失效——删号后令牌仍有效（8.1），改密不验旧口令（7.6）。**第十五天**  
：LDAP 接入边界——空口令匿名绑定（7.4），默认凭据是 password（7.4）。**第十六天**  
：实例元数据投毒——未注册坐标可预置（7.1），partial 整表覆盖（7.1）。**第十七天**  
（本篇）：OIDC 登录链路——主体混淆认证绕过（8.2），Host 注入开放重定向（8.0）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
