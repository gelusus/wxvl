#  【一天一个nacos漏洞·第九天】【高危】身份复用和重放攻击  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-11 03:53  
  
   
  
# 【一天一个nacos漏洞·第九天】【高危】一个请求头，跳过了所有检查  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个，是同一个机制的两半：**一个请求头，命中之后，身份校验和权限校验一起不跑了。**  
  
一个讲它怎么泄出去——**Nacos 自己会把这把钥匙送给"成员列表里的任意地址"**  
。  
  
一个讲它为什么一直有效——**这个值是静态的，不轮换、不过期、可以无限重放**  
。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号、口令与共享密钥均未写入本文。内容仅用于安全学习与防御加固。  
  
  
01  
  
一、先说清楚这个头是干什么的  
  
Nacos 集群里，节点之间要频繁互调：上报成员信息、同步配置、发 Distro 数据。要是每次调用都走一遍用户认证，开销和复杂度都不划算。  
  
所以它有一套"内部互信"机制：配一个**全集群共享的密钥**  
  
```
nacos.core.auth.server.identity.key=serverIdentity
nacos.core.auth.server.identity.value=<全集群共用的那个值>

```  
  
  
节点之间发请求时带上 serverIdentity: <这个值>  
，服务端一比，相等就认你是"自己人"。  
  
设计上说得通。问题在下一步——**"认你是自己人"之后，它做了什么：**  
  
```
case MATCHED:
    identityContext.setParameter(Constants.Identity.SERVER_IDENTITY, Boolean.TRUE);
    chain.doFilter(request, response);   // 直接放行
    return;

```  
  
  
**它没有"通过认证然后去查权限"，它直接 return 了。**  
 后面的 validateIdentity  
（你是谁）和 validateAuthority  
（你能干什么）根本没有执行。  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><section><span leaf=""><br/></span></section></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">VE-0705 · 钥匙怎么泄的</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">VE-0793 · 钥匙为什么不换</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">7.4</span></strong></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">8.1</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">成员上报把凭据发给任意成员地址</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">静态值 + 纯 equals，可永久重放</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">关键前提</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">集群模式 + 可控成员地址</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">拿到值 + 8848 可达</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">最低权限</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">无需任何账号</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">无需任何账号</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0705：Nacos 会亲手把钥匙送出去  
### 2.1 上报任务：把凭据写进请求头  
  
集群里有个定时任务，每轮挑一个"非自身成员"，把本节点的信息上报过去：  
  
```
List&lt;Member&gt; members = ServerMemberManager.this.allMembersWithoutSelf();
Member target = members.get(cursor);
if (target.…isGrpcReportEnabled()) {
    reportByGrpc(target);
} else {
    reportByHttp(target);        // 目标不支持 gRPC 时走明文 HTTP
}

```  
  
  
而在真正发出去之前，它会做这么一件事：  
  
```
AuthHeaderUtil.addIdentityToHeader(header, NacosAuthConfigHolder.getInstance()
        .getNacosAuthConfigByScope(NacosServerAuthConfig.NACOS_SERVER_AUTH_SCOPE));

```  
  
  
**把那个全集群共享的密钥，塞进请求头。**  
  
于是每一轮上报，都是一次"密钥投递"。  
### 2.2 而"发给谁"，不由 Nacos 决定  
  
关键问题来了：**这个 target 是谁？**  
  
成员列表来自两个地方，**都不可信**  
：  
- **地址服务器**  
：syncFromAddressUrl()  
 用普通 HTTP 拉一个纯文本列表，没有 TLS、没有签名、没有来源校验，直接解析成成员；  
- **cluster.conf**  
：本地文件，能写就能加。  
也就是说：**谁能让 Nacos 把某个地址当成"集群成员"，谁就能收到 Nacos 主动发来的密钥。**  
### 2.3 实测：把自己变成成员，等钥匙上门  
  
测试时在目标机上起了一个伪成员（只记录请求头，不落任何数据），然后分别用两条路径把它塞进成员列表——地址服务器、cluster.conf  
。  
  
结果，捕获日志里：  
  
```
[08:01:03] GRPC unary type=MemberReportRequest
    headers={serverIdentity=<共享密钥>}

[08:01:05] HTTP POST /nacos/v1/core/cluster/report
    H Serveridentity: <共享密钥>

```  
  
  
**gRPC 通道和明文 HTTP 通道，两条都带。**  
 整个测试期间，HTTP 上报里出现了 **33 次**  
 同一把密钥。  
  
还有个小细节：如果 gRPC 那边返回 NO_HANDLER  
（302），代码会把 grpcReportEnabled  
 置 false，**下一轮（2 秒后）改走明文 HTTP**  
——等于说，你哪怕不拦 gRPC，它也会自己降级成更好截的明文。  
### 2.4 拿到之后：一个头，全部 200  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">请求（8848 主端口）</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">无头</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">错值</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">捕获到的值</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">集群节点列表</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">用户列表</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">（含 bcrypt 哈希）</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">OPEN_API 实例列表</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">发布配置</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">读取配置 / 删除配置</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">403</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><strong style="font-weight: 700;color: #27272A;"><span leaf="">200 / 200</span></strong></td></tr></tbody></table>  
  
**没有带任何账号。**  
 一个 serverIdentity  
 头，等于管理员。  
  
03  
  
三、VE-0793：这把钥匙永远不会换  
### 3.1 校验只有一句 equals  
  
拿到值之后为什么能一直用？因为服务端就这一句：  
  
```
public ServerIdentityResult check(ServerIdentity serverIdentity, Secured secured) {
    if (authConfig.getServerIdentityValue().equals(serverIdentity.getIdentityValue())) {
        return ServerIdentityResult.success();
    }
    return ServerIdentityResult.noMatched();
}

```  
  
  
**没有来源 IP、没有签名、没有时间戳、没有 nonce、没有次数限制。**  
  
而这个值本身来自部署配置——**全集群一个、不轮换、不过期**  
，只有人工去改才会变。  
  
所以它具备了一个"完美凭据"的全部特征：**一次拿到，永久可用，且与你是谁完全无关。**  
### 3.2 实测：未登录读全部用户，还能改任意人的密码  
  
```
GET /nacos/v3/auth/user/list        无头        → 403
GET /nacos/v3/auth/user/list        serverIdentity: <值> → 200
  → {"pageItems":[{"username":"devuser","password":"$2a$10$..."},
                  {"username":"nacos","password":"$2a$10$..."}, ...]}

同一头连续重放 3 次 → 200 / 200 / 200

```  
  
  
更狠的是改密：  
  
```
PUT /nacos/v3/auth/user?username=<任意用户>&newPassword=<新密码>
serverIdentity: <值>          （不带任何 token）
→ 200 {"code":0,"data":"update user ok!"}

```  
  
  
实测用这个方式改了一个临时账号的密码，**然后用新密码成功登录了**  
。  
  
用不存在的用户做对照更能说明问题：无头 → 403 access denied  
；带头 → 500 user not exist  
。**500 说明权限门已经过了，只是人不存在。**  
### 3.3 第二道门，是同一个头  
  
改密接口里还有一层自己的权限检查：  
  
```
private boolean hasPermission(String username, HttpServletRequest request) {
    ...
    if (isFromServerIdentity(request)) {   // 又一次 equals 请求头
        return true;
    }
    ...
}

```  
  
  
**同一个头，第二次放行。**  
 这也解释了为什么"未登录改任意用户密码"能成立——不是绕过了一道门，是两道门都认同一个东西。  
### 3.4 一个测试者没忘的边界  
  
报告特意测了 console：NacosConsoleAuthFilter.checkServerIdentity**恒返回 noMatched**  
，所以 8080 控制台带这个头照样 403。  
  
也就是说这个洞打的是 **8848 主服务端口**  
（ADMIN_API / OPEN_API），不是控制台。这个边界写清楚很重要——不然读者会以为"加个头就能进控制台"。  
  
04  
  
四、为什么这不是"认证通过"，是"全部跳过"  
  
把这两份里最关键的那几行放在一起看：  
  
```
正常用户请求：
  validateIdentity   → 你是谁？（验 token）
  validateAuthority  → 你能干什么？（查 RBAC）

带 serverIdentity 头的请求：
  checkServerIdentity → MATCHED
  → chain.doFilter   → 直接进业务
  → return           → validateIdentity 和 validateAuthority 都没跑

```  
  
  
**这不是"这个身份有权限"，而是"这次请求不需要权限"。**  
  
差别很微妙但很致命：前者还能被最小权限原则约束，后者**完全在权限体系之外**  
。所以它绕过的不是一个接口，是**所有标了 @Secured 的接口**  
——ADMIN_API、OPEN_API 全都算。  
  
05  
  
五、边界：两个都是 AC:H，得说清楚  
  
**VE-0705**  
 需要：目标跑在**集群模式**  
（standalone 不调度上报任务）、攻击者能控制地址服务器响应或写 cluster.conf  
。  
  
**VE-0793**  
 需要：先拿到那个值（读日志、或观测节点间明文流量）、且 8848 可达（很多部署只对外开 8080）。  
  
两个都不是"公网随手一打"。但换个角度：  
> 📐 **这两个洞真正说明的问题，不是"能打进去"，而是"内部互信这套设计的信任边界，比想象中宽得多"。**  
一个静态字符串，既是"成员身份"，又是"管理员通行证"，还会被主动发给"任何自称成员的人"。 而且第二个洞的泄露路径，报告里写得很直白——**它和第六天那个"配置打进日志"是同一行代码**  
（toString()  
 没脱敏）。一个泄露点，喂了两个洞。  
  
  
06  
  
六、修复：让"内部信任"不再等于"全部放行"  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;font-weight: 700;background: #FAFAFA;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">MATCHED 即全放行</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">收窄作用域：</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">serverIdentity</span></code><strong style="font-weight: 700;color: #27272A;"><span leaf="">只对 INNER_API 生效</span></strong><span style="display: inline;"><span leaf="">，不得对 ADMIN_API / OPEN_API 放行</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">静态共享密钥</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">改成</span></span><strong style="font-weight: 700;color: #27272A;"><span leaf="">每节点独立的短期凭据</span></strong><span style="display: inline;"><span leaf="">（含 nonce、时间戳、目标绑定），支持自动轮换</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">纯 equals 校验</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">加来源校验（集群网段 / mTLS 节点证书）+ 防重放缓存</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">主动外发凭据</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">成员上报不要带共享密钥；改用 mTLS 或按成员签名的会话密钥</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">成员地址不可信</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">地址服务器响应要签名或走 TLS；</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">cluster.conf</span></code><span style="display: inline;"><span leaf=""> 变更需可信来源；成员地址白名单</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">二次校验也是同一个头</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">删掉 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">hasPermission</span></code><span style="display: inline;"><span leaf=""> 里的 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">isFromServerIdentity</span></code><span style="display: inline;"><span leaf=""> 直接放行；改密这类操作应校验旧密码或用户凭据</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">明文日志</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #27272A;background: #F4F4F5;padding: 2px 6px;border-radius: 2px;white-space: nowrap;"><span leaf="">toString()</span></code><span style="display: inline;"><span leaf=""> 对 identity 值脱敏（同第六天）</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">端口暴露</span></span></td><td style="border: none;border-bottom: 1px solid #E4E4E7;padding: 10px 12px;word-break: keep-all;color: #52525B;"><span style="display: inline;"><span leaf="">8848/9848/9849 只监听内网，对外只留 8080</span></span></td></tr></tbody></table>  
> 💡 一句可以写进架构评审的话：**"内部"不是一个安全等级，是一个网络位置。**  
任何"因为它在集群里所以可信"的设计，都要回答两个问题——**怎么证明它真在集群里？它拿到这个身份之后能干什么？**  
  
  
07  
  
写在最后  
  
第九天，聊了一个和前八天都不太一样的东西：**不是校验写错了，而是有一个东西能跳过校验。**  
  
前面八天，问题都出在"某道关卡"。今天的问题出在"**有一条通道绕过了所有关卡**  
"——而且这条通道的设计初衷是好的（让集群内部通信更轻），只是它给的东西太多了。  
  
回头看这九天，其实一直在讲同一件事的不同侧面：  
- 有的洞是**名字对不上**  
（第二、三、四天）  
- 有的洞是**门装错位置**  
（第五天）  
- 有的洞是**执行和输出没管住**  
（第六天）  
- 有的洞是**信任建立得不对**  
（第七天）  
- 有的洞是**失败时选了放行**  
（第八天）  
- 今天：**有一个后门，本来只该给内部用，结果给了全部权限**  
九天下来的体会，一句话：**安全设计里最难的不是"加校验"，而是"划边界"**  
——什么东西能绕过校验、绕到什么程度、凭什么证明它有资格绕。这三问答错了，加再多校验也只是给正门加了锁，侧门还开着。  
  
下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
：MCP 与 A2A，名字和版本拼成的 key 不唯一。**第四天**  
：配置接口的别名错配（9.9），Prompt/Skill 的冒号分隔符碰撞（9.1）。**第五天**  
：配置接口认错参数（9.6），AI 资源可见性被 Config 通道绕过（9.6）。**第六天**  
：MySQL 健康检查的 SQL 注入（7.7），鉴权密钥进日志（7.8）。**第七天**  
：OIDC 信任链两端——不验来源（8.1），不认撤销（7.4）。**第八天**  
：两处 fail-open——插件缺失即全放行（7.2），空白主体即成管理员（8.1）。**第九天**  
（本篇）：服务间共享身份——凭据被主动投递（7.4），静态值可永久重放（8.1）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
