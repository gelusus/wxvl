#  【一天一个nacos漏洞·第十九天】【高危】传输层认证缺陷：TLS 证书校验失效  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-26 06:35  
  
   
  
# 【一天一个nacos漏洞·第十九天】【高危】传输层认证缺陷：TLS 证书校验失效  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个都在**客户端连服务端**  
的那条链路上，类型是 **CWE-295/297：证书校验不当 / 主机名校验不当**  
。  
  
一句话：**TLS 起来了，握手成功了，锁头图标有了——但对面是谁，没验。**  
- 一个：**只要是 IP，就无条件放行**  
，还把这条规则设成了整个 JVM 的默认。  
- 一个：gRPC 那条路的 trustAll  
，**缺省值是 true**  
。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与口令均未写入本文。内容仅用于学习与防御加固。  
  
  
01  
  
一、TLS 其实承诺了两件事  
  
大家提 TLS，第一反应是"加密了"。但 TLS 同时承诺两件事：  
  
```
① 加密（Confidentiality）：中间人看不懂内容
② 认证（Authentication）：对面真的是它，不是冒充的

```  
  
  
**第 ② 件靠的就是证书校验。**  
  
如果只做 ① 不做 ②，会发生什么？——中间人可以这样：  
  
```
客户端 → [中间人] → 服务端
         ↑ 用自签证书跟客户端握手（客户端不验，接受）
         ↑ 再用真证书跟服务端握手
         ↑ 两边都"加密"了，但中间人全看得见

```  
  
  
**这叫"合法的中间人"，因为客户端自愿把证书校验关掉了。**  
  
所以证书校验不是 TLS 的"可选配件"，它是 TLS 这个词的一部分。今天两个洞，就是把这部分拆掉了。  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><section><span leaf=""><br/></span></section></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">VE-0168 · HTTP 侧</span></span></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">VE-0169 · gRPC 侧</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">7.4</span></strong></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">7.4</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">类型</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">CWE-297 主机名校验不当</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">CWE-295 证书校验缺失</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">手法</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">IP 一律放行</span></strong><span style="display: inline;"><span leaf=""> + 全局安装</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><strong style="font-weight: 600;color: #2B2B2B;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">trustAll</span></code><span leaf=""> 缺省 true</span></strong></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">影响面</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">单次连接 + </span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">整个 JVM</span></strong></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">单条 gRPC 链路</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0168：只要是 IP，就不验  
### 2.1 那个 verifier 的三行逻辑  
  
Nacos 客户端启用 TLS 时，会构造一个自己的主机名校验器：  
  
```
@Override
public boolean verify(String hostname, SSLSession session) {
    if (LOCALHOST_HOSTNAME[0].equalsIgnoreCase(hostname)     // "localhost"
        || LOCALHOST_HOSTNAME[1].equals(hostname)) {         // 127.0.0.1
        return true;
    }
    if (isIp(hostname)) {                                    // ← 任意 IP 字面量
        return true;
    }
    return hv.verify(hostname, session);                      // 只有域名才委托 JDK
}

```  
  
  
**localhost 和任意 IP，直接 return true。**  
 它连 session.getPeerCertificates()  
 都没读。  
  
而 isIp()  
 只是个字符串语法判断——“这串字符长得像不像 IP”，和对面拿的是哪张证书**毫无关系**  
。  
  
于是：  
  
```
访问 10.10.10.10（IP）    → return true（不管证书上写的是谁）
访问 nacos.corp.com（域名）→ 委托 JDK 正常校验

```  
  
### 2.2 这甚至不是"手滑"  
  
报告里有一句很关键的话：  
> 仓库单测 SelfHostnameVerifierTest  
 亦断言 verify("10.10.10.10")==true  
，说明这是**既定实现**  
而非偶发缺陷。  
  
  
**有人写了这个行为，还写了单测把它锁住。**  
  
推测动机不难理解：早期很多证书没有 IP SAN，用 IP 直连时 JDK 会报"证书与主机名不匹配"，用户嫌麻烦——于是"IP 就不校验吧"。  
  
但代价是：**Nacos 部署里最常用、最自然的连接方式（填 IP），恰好是唯一不校验的方式。**  
### 2.3 更麻烦的是：它把这个规则变成了全局的  
  
```
public void setSslContext(SSLContext sslContext) {
    HttpsURLConnection.setDefaultSSLSocketFactory(sslContext.getSocketFactory());   // 进程级
}

public void replaceSslHostnameVerifier(HostnameVerifier hostnameVerifier) {
    HttpsURLConnection.setDefaultHostnameVerifier(hostnameVerifier);                // 进程级
}

```  
  
  
setDefault*  
 是 **JVM 级别**  
的设置。  
  
也就是说：**只要你的进程里初始化了 Nacos 客户端，整个 JVM 后续所有 HttpsURLConnection 都会用这套"IP 放行 + trustAll"的规则。**  
  
报告的实测证明了这一点：  
  
```
初始化 Nacos 客户端后
→ 用一个与 Nacos 完全无关的普通 HttpsURLConnection 访问不匹配证书的服务端
→ GLOBAL_AFTER... HTTP 200

```  
  
  
**一个组件的 TLS 配置，改变了整个进程的信任决策。**  
  
这个副作用比"连错服务器"严重得多——因为**别的组件根本不知道自己被降级了**  
。  
### 2.4 TrustManager 也有一个 fail-open  
  
```
public static TrustManager[] trustManager(boolean needAuth, String trustCertPath) {
    if (needAuth) {
        try {
            return trustCertPath == null ? null : buildSecureTrustManager(trustCertPath);
        } catch (SSLException e) {
            LOGGER.warn("degrade trust manager as build failed, will trust all certs.");
            return trustAll;              // ← 构建失败？那就信任所有
        }
    } else {
        return trustAll;                  // ← 默认 authServer=false
    }
}

```  
  
  
两处放行：  
- 默认 tls.client.authServer=false  
 → 直接 trustAll  
；  
- 即使开了 authServer=true  
，**只要证书库构建失败，就降级成信任所有**  
——一个 warn 日志，然后照常放行。  
**"失败就放行"这个模式，我们在第八天见过一次（鉴权 fail-open）。这次它出现在 TLS 上。**  
### 2.5 那"严格模式"能不能救？  
  
测试专门跑了一组严格配置：authServer=true  
 + 正确配置 CA 文件。  
  
```
UNIT trustManager(true, ca.pem) => X509TrustManagerImpl acceptedIssuers=count=1   ← 证书链校验真的生效了
CONTROL_JDK_DEFAULT_REJECTED SSLHandshakeException: No subject alternative names matching IP address 127.0.0.1
FAKE_SERVER_REQ POST /nacos/v3/auth/user/login?username=viewer body=password=...
STOLEN_CREDENTIAL username=viewer password=...

```  
  
  
**证书链校验确实生效了，JDK 默认的校验器也确实拒绝了——但 Nacos 的客户端握手成功了，并把口令发了出去。**  
  
因为那张证书是一个"受信 CA 签发给别的域名"的证书：**链是可信的，只是不是这个 IP 的**  
。而主机名分支对 IP 直接放行。  
> 📌 所以严格模式挡不住。**它能防"证书链不可信"，防不了"证书不是发给这个地址的"。**  
  
### 2.6 顺手说一句实测出来的细节  
  
客户端自动登录时，凭据是这样发的：  
  
```
POST /nacos/v3/auth/user/login?username=<用户名>   body=password=<密码>

```  
  
  
**用户名在查询串里，密码在表单体里，都是明文**  
——反正 TLS 会保护，对吧？  
  
但在上面这套配置下，TLS 只保护了"中间人看不懂"，没保护"对面是不是真的"。所以中间人拿到的是完整的 username  
 和 password  
。  
  
实测截获到了三类账号的口令，包括管理员的。  
  
03  
  
三、VE-0169：gRPC 那条路，缺省就是"信"  
### 3.1 工厂把安全缺省覆盖了  
  
配置类本身是安全的：  
  
```
// TlsConfig.java
private Boolean trustAll = false;        // ← 字段缺省：false（对）

```  
  
  
但工厂在构造时把它覆盖了：  
  
```
// RpcClientTlsConfigFactory
tlsConfig.setTrustAll(getBooleanProperty(properties, "...trustAll", true));   // ← 缺省 true

```  
  
  
getBooleanProperty  
 的逻辑是：**配置里有这个键就用配置值，没有就返回传进来的默认值**  
。  
  
而发行包的示例配置里，**根本没有 trustAll 这一项**  
（全仓库检索过）。  
  
于是：  
  
```
运维写了 tls.enableTls=true  →  TLS 开启
没写 trustAll                →  取缺省 true

```  
  
  
**结果：TLS 开了，但证书不验。**  
### 3.2 那个 if 只有两个分支  
  
```
if (tlsConfig.getTrustAll()) {
    builder.trustManager(InsecureTrustManagerFactory.INSTANCE);   // ← 接受任意证书
} else {
    if (StringUtils.isBlank(tlsConfig.getTrustCollectionCertFile())) {
        throw new IllegalArgumentException("trustCollectionCertFile must be not null");
    }
    ... // 加载 CA 校验
}

```  
  
  
注意 else  
 分支：**当 trustAll=false 却没配 CA 文件时，它直接抛异常**  
——宁可起不来。  
  
对比一下：trustAll=true  
 时什么检查都没有。**  
  
**"不校验"是畅通无阻的默认路径；"要校验"反而需要额外配置，配漏了还起不来。**  
### 3.3 服务端也是同一个缺省  
  
```
// RpcServerTlsConfigFactory
tlsConfig.setTrustAll(getBooleanProperty(properties, prefix + TLS_TRUST_ALL, true));

```  
  
  
服务端在 mutualAuthEnable=true  
 时也走同一个 InsecureTrustManagerFactory  
——**双向认证里，客户端证书也不验。**  
### 3.4 一个必须说清的前提  
  
报告的诚实之处在这里：  
> enableTls  
 的缺省值仍是 false  
。靶机当前未启用 TLS，现网是明文通道，**默认部署本身并不存在可被 MITM 的 TLS 会话**  
。  
  
  
也就是说，这不是"装完就能打"。  
  
但换个角度看这句话，它反而更刺人：  
> **很多团队开 TLS，是因为合规要求"必须使用加密传输"。**  
于是他们打开 enableTls=true  
，验收通过，报告上写着"已启用 TLS"。**而通道其实只加密、不认证。**  
  
  
**为了合规而做的加固，做成了一个签字齐全的空壳。**  
  
04  
  
四、两条链路，同一个结论  
  
```
VE-0168：HTTP 侧  →  IP 就放行，还把规则装成了 JVM 默认
VE-0169：gRPC 侧  →  trustAll 缺省 true，一行 if 就接受任意证书

```  
  
  
一个在"**怎么判断主机名**  
"上放弃，一个在"**要不要判断**  
"上放弃。  
  
而两条链路加起来，正好覆盖了 Nacos 客户端跟服务端通信的全部方式：  
  
```
配置拉取 / 服务发现（HTTP）  →  0168
长连接推送 / 集群同步（gRPC） →  0169

```  
  
  
**也就是说：一个开了 TLS 的 Nacos 客户端，两条路都没有在验对面是谁。**  
  
05  
  
五、边界  
  
**0168**  
：需要 -Dtls.enable=true  
、且 serverAddr  
 用 **IP 字面量**  
（域名不触发，因为会委托 JDK）。攻击者需要网络中间人位置。  
  
**0169**  
：需要**启用 TLS 但没写 trustAll=false**  
。默认 enableTls=false  
 时是明文，反而没有 TLS 会话可劫持。  
> 📐 两份都是 AC:H，因为都需要"网络中间人位置"这个前提。 但这一点值得多想一层：**中间人位置在内网里并不稀有**  
——一台被控的跳板机、一个 ARP 欺骗、一个恶意的 Wi-Fi、一个配置错的代理，都算。  
  
  
06  
  
六、修复  
  
<table><thead><tr><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;font-weight: 600;background: #EEF3F0;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">IP 主机名恒放行</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">删掉 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">isIp()</span></code><span style="display: inline;"><span leaf=""> 那条捷径，</span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">始终委托标准 verifier</span></strong><span style="display: inline;"><span leaf="">；JDK 本来就支持 IP SAN 的 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">iPAddress</span></code><span style="display: inline;"><span leaf=""> 匹配</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">确实需要兼容旧证书</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">至少读 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">session.getPeerCertificates()</span></code><span style="display: inline;"><span leaf=""> 的 SAN </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">iPAddress</span></code><span style="display: inline;"><span leaf=""> 做绑定校验，别&#34;看字符串像 IP 就放过&#34;</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">全局安装</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">别用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">setDefaultHostnameVerifier</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">setDefaultSSLSocketFactory</span></code><span style="display: inline;"><span leaf="">；改为在</span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">单个连接实例</span></strong><span style="display: inline;"><span leaf="">上设置</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">TrustManager fail-open</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">构建失败必须 </span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">fail-closed</span></strong><span style="display: inline;"><span leaf="">（抛异常中止），不能降级成 trustAll</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">gRPC trustAll 缺省</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">把工厂里的缺省改成 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">false</span></code><span style="display: inline;"><span leaf="">，与 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">TlsConfig</span></code><span style="display: inline;"><span leaf=""> 字段对齐；服务端同步改</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">示例配置缺项</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #4A5D52;background: #EEF3F0;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">trustAll=false</span></code><span style="display: inline;"><span leaf=""> 写进发行版示例，并加部署自检</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">无感知降级</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">启动日志输出&#34;实际生效的 TLS 校验策略&#34;与 serverAddr 形式，便于审计</span></span></td></tr><tr><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">应急缓解</span></span></td><td style="border: none;border-bottom: 1px solid #E8E8E8;padding: 10px 12px;word-break: keep-all;color: #525252;"><span style="display: inline;"><span leaf="">修复前用</span></span><strong style="font-weight: 600;color: #2B2B2B;"><span leaf="">域名</span></strong><span style="display: inline;"><span leaf="">（且证书包含该域名）配置 serverAddr，别用 IP</span></span></td></tr></tbody></table>  
> 💡 一句可以写进接入规范的话：**"开了 TLS"不是一句结论，是一个需要被验证的断言。**  
验证方式是：拿一张**故意不对的证书**  
去连，看它会不会拒绝。**不拒绝，就等于没开。**  
  
  
07  
  
七、写在最后  
  
第十九天，第一次专门讲**传输层**  
。  
  
前面十八天，问题大多在"业务逻辑"和"鉴权"这一层。今天的两个更靠下——**在 TCP 之上的那一层**  
。  
  
而这一层有个特点：**它特别容易被"看起来对"糊弄过去。**  
- 握手成功了；  
- 浏览器/客户端没有报错；  
- 抓包看到的是密文；  
- 合规扫描器显示"已启用 TLS"。  
**每一项都对，但对面是谁，没人问过。**  
  
这也是为什么 0168 那个 return true  
 特别值得琢磨：它不是疏忽，是**有人权衡之后决定"IP 就不校验了，省事"**  
。省掉的那点麻烦，换来的是"凡是填 IP 的部署，TLS 都只加密不认证"。  
  
所以这一篇最后想说的是：  
> **“加密"回答的是"别人能不能看懂”，“认证"回答的是"对面是不是它”。**  
这两件事经常被合并成一个词——“安全连接”。但它们可以只做一半，而且**做了一半的时候，看起来和全做了完全一样。**  
  
  
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
：OIDC 登录链路——主体混淆（8.2），Host 注入开放重定向（8.0）。**第十八天**  
：路径穿越——服务端缓存任意文件写/删（8.3），导出包 Zip Slip（8.0）。**第十九天**  
（本篇）：传输层认证缺陷——HTTP 侧 IP 主机名绕过（7.4），gRPC 侧 trustAll 缺省 true（7.4）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
