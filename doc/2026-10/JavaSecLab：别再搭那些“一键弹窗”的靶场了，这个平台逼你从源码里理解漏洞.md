#  JavaSecLab：别再搭那些“一键弹窗”的靶场了，这个平台逼你从源码里理解漏洞  
原创 KLSEC
                    KLSEC  昆仑AI安全实验室   2026-10-28 16:00  
  
你有没有过这种感觉：XSS靶场玩了一下午，弹窗弹了上百次，关掉靶场回到真实项目，看到一段innerHTML  
赋值，还是不知道它到底能不能被利用、利用链长什么样、修复时该改哪一行。  
  
问题不在你，在靶场。**大部分靶场教你“输入什么能弹窗”，JavaSecLab教你“为什么这段代码会弹窗，以及在真实系统里它长什么样”**  
 。  
  
JavaSecLab（whgojp/JavaSecLab）不是那种“输入<script>alert(1)</script>  
就通关”的靶场。它的定位是**源码到Sink的完整映射**  
——每一个漏洞场景都提供缺陷代码、修复代码、审计Sink点、安全编码规范，覆盖从SQL注入、SSRF、RCE、反序列化到SpEL注入、SSTI、Spring Boot生态暴露的完整攻击面。GitHub上的中文文档把它描述为“综合型Java漏洞平台”，但更准确的说法是：**它是一本可以运行起来的代码审计教科书**  
。  
  
**一、选对部署路径：本地IDEA还是Docker，取决于你想“看”还是想“用”**  
  
JavaSecLab提供两种部署方式：本地IDEA编译和Docker Compose一键部署。两者的选择标准很简单：**你想读代码，走本地；你想快速验证某个Payload，走Docker。**  
  
**本地部署的核心环境要求**  
很明确：JDK 1.8、Maven、MySQL 5.7+或8.0+  
。项目是标准的Maven工程，git clone  
之后，mvn clean package -DskipTests  
就能打出target/JavaSecLab.jar  
，java -jar  
启动，浏览器访问localhost:8080  
就是登录界面  
。  
  
但这里有一个**新手必踩的坑**  
：application-dev.yml  
里的数据库连接参数是硬编码的。默认active  
是dev  
，spring.datasource.url  
指向localhost:13306  
，password  
是QWE123qwe  
。如果你本地MySQL跑在3306，或者密码不是这个，应用启动时会直接抛数据库连接异常，而且错误信息不会告诉你“去改配置文件”。**第一次启动前，先打开src/main/resources/application-dev.yml，把username、password、url三行改成你本地MySQL的实际值**  
，这是整个本地部署里唯一不能跳过的步骤。  
  
数据库初始化同样不能跳。先在MySQL里CREATE DATABASE JavaSecLab CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci  
，然后执行sql/JavaSecLab.sql  
。不导这个SQL，登录页面能打开，但登录会失败，因为用户表是空的。  
  
Docker路径简单得多。项目自带了docker-compose.yml  
，docker-compose -p javaseclab up -d  
一条命令拉起MySQL 8.0.33和JavaSecLab容器  
。如果宿主机的3306被占用了，compose文件里MySQL的外部映射端口默认是3308:3306  
，不会冲突。**Docker路径适合“我只想测这个Payload能不能打通”，不适合“我想看这个漏洞的代码到底怎么写的”**  
 ——容器里的代码是编译后的class，不是源码。  
  
**二、SpEL注入：Spring表达式里藏着的RCE，比SQL注入更隐蔽**  
  
SpEL（Spring Expression Language）注入是JavaSecLab里最值得先练的模块之一，因为它在真实Spring项目里的暴露面比大多数人想象的大——@Value  
注解、SpelExpressionParser  
、以及各种接受表达式字符串的接口，都是潜在入口。  
  
JavaSecLab的SpEL场景给了一段典型的缺陷代码：一个接口接受用户输入的“表达式模板”，然后通过SpelExpressionParser  
解析执行。**缺陷不在于“用了SpEL”，而在于“用了SpEL但没有限制表达式的语法树”**  
 。SpEL的T()  
操作符允许直接引用类，T(java.lang.Runtime).getRuntime().exec(...)  
就是一条RCE链。  
  
在靶场里，这个漏洞的利用方式很直接：向表达式参数传入T(java.lang.Runtime).getRuntime().exec("whoami")  
，观察命令执行结果。但**JavaSecLab的价值不在于让你看到whoami的回显，在于它同时提供了修复代码**  
。修复方案不是“过滤Runtime  
关键字”——那会被T(java.lang.Run  
 + time)  
这样的拼接绕过——而是**使用SimpleEvaluationContext替代StandardEvaluationContext**  
 ，禁用T()  
操作符和类型引用，把表达式的能力限制在“属性访问和方法调用”的安全子集内。  
  
这个修复逻辑在真实代码审计里同样适用。你看到一个SpelExpressionParser.parseExpression(userInput)  
，不要只搜“有没有过滤exec  
”，去看它用的是StandardEvaluationContext  
还是SimpleEvaluationContext  
。前者默认开启类型引用，后者默认关闭。**上下文的选择，决定了这个接口是“表达式计算”还是“远程代码执行”**  
。  
  
**三、反序列化：从readObject到Gadget Chain，JavaSecLab让你亲手拼一条链**  
  
反序列化是Java安全里最“深”的领域，也是大多数靶场一笔带过的部分。JavaSecLab的反序列化模块覆盖了**CommonsCollections、Fastjson、Jackson、XStream、Log4j2、Shiro、SnakeYAML、XMLDecoder**  
等多个生态的案例，并且每一个案例都指向具体的触发点和Gadget路径。  
  
以一个典型的ObjectInputStream.readObject()  
场景为例。靶场提供了一个接受Base64序列化数据的接口，缺陷代码直接new ObjectInputStream(input).readObject()  
，没有任何类白名单。攻击者的利用路径是：用ysoserial  
生成一个CommonsCollections6  
链的Payload，Base64编码后提交，触发命令执行。  
  
但**JavaSecLab的练习目标不是“你会不会用ysoserial”**  
 。它的漏洞场景页面同时展示了**Source→Sink的映射**  
：Source是request.getParameter("data")  
，Sink是ObjectInputStream.readObject()  
，中间的传播路径被标注出来。这意味着你练习的是 **“在一个陌生的代码库里，如何从Sink反向找到Source”**  
 ，而不是“记住一个Payload”。  
  
修复侧的案例同样具体。针对readObject  
，正确的修复不是“过滤rO0AB  
（Java序列化魔数）”——攻击者可以换用JSON、XML等其他序列化格式——而是**使用ObjectInputFilter（JEP 290）设置类白名单**  
，或者干脆抛弃Java原生序列化，改用JSON+显式类型映射。靶场的修复代码演示了ObjectInputFilter.Config.createFilter("top.whgojp.**;!*")  
的写法，只允许业务包下的类被反序列化。  
  
**四、Spring Boot生态暴露：Actuator、Swagger、Druid，每一个都是真实世界的入口**  
  
JavaSecLab单独把Spring Boot生态的暴露面作为一个模块，覆盖Swagger、Actuator、Druid、MySQL JDBC反序列化等场景。这不是凑数——**在2026年的真实渗透测试里，/actuator/env和/actuator/heapdump的命中率，比很多传统Web漏洞都高**  
。  
  
靶场里有一个Actuator未授权访问的完整场景。缺陷配置是management.endpoints.web.exposure.include=*  
，且没有引入Spring Security做端点保护。利用路径很清晰：访问/actuator/env  
读取环境变量（可能包含数据库密码、云AK/SK），访问/actuator/heapdump  
下载JVM内存镜像，再用JDumpSpider  
之类的工具从堆转储里提取明文凭据。  
  
但**这个场景的教学价值在“配置修复”这一侧**  
。JavaSecLab给出的修复方案不是“关掉所有Actuator端点”——那会影响健康检查——而是**精确控制暴露范围**  
：management.endpoints.web.exposure.include=health,info  
，同时通过Spring Security对/actuator/**  
做角色校验。/actuator/env  
和/actuator/heapdump  
在生产环境里永远不应该对未认证请求开放，但如果确实需要远程访问，必须绑定到内部管理端口，而不是与业务流量共用8080。  
  
Druid的案例同理。靶场模拟了Druid监控页面未授权访问的场景，/druid/index.html  
可以直接打开，SQL监控页面暴露了正在执行的查询语句和参数。修复方案包括：设置druid.stat-view-servlet.login-username  
和login-password  
，或者在Spring Security层面对/druid/**  
做IP白名单。**这些配置在真实的Spring Boot项目里经常被遗忘，因为Druid是“引入依赖就自动生效”的，开发者往往不知道它暴露了一个带登录页的监控面板**  
。  
  
**五、业务逻辑与组件安全：那些“扫描器永远扫不到”的东西**  
  
JavaSecLab的漏洞模块里，有一部分是传统扫描器很难覆盖的：**IDOR、验证码安全、支付安全、并发安全、请求签名绕过、JWT凭证安全**  
。  
  
以**并发安全**  
为例。靶场里有一个优惠券领取接口，逻辑是“检查用户是否已领取→发放优惠券”。缺陷在于检查和发放之间没有原子性，攻击者用Turbo Intruder或并发脚本同时发送多个请求，多个请求同时通过“是否已领取”的检查，最终发出多张券。JavaSecLab的修复代码演示了数据库唯一索引和Redis分布式锁两种方案，并且**对比了它们在高并发下的表现差异**  
——唯一索引在极端并发下可能抛出DuplicateKeyException  
，需要在应用层捕获并返回友好的错误信息；分布式锁的粒度需要精确到userId + couponId  
，否则会误伤正常用户的并发请求。  
  
**请求签名绕过**  
是另一个值得单独练的场景。靶场的接口使用sign = md5(params + secret)  
做签名校验，但缺陷在于**secret硬编码在前端JS里**  
，而且签名算法没有绑定timestamp  
和nonce  
。攻击者从JS文件里提取secret  
，就能伪造任意请求。修复方案包括：把签名逻辑移到后端、使用HMAC-SHA256替代MD5、在签名中加入timestamp  
和nonce  
并做服务端去重。**这个场景直接对应了真实SRC里“前端硬编码密钥”那一类高频漏洞**  
。  
  
**六、本地IDEA部署的避坑清单：三个会让新手卡住半天的问题**  
  
**第一个：Maven依赖拉取超时。**  
 项目依赖Spring Boot、MyBatis、Thymeleaf、Layui等组件，首次mvn clean package  
会从中央仓库拉取大量依赖。不配国内镜像，大概率卡在Downloading from central  
。在settings.xml  
的<mirrors>  
里加阿里云镜像，<mirrorOf>*</mirrorOf>  
覆盖所有仓库，包括插件仓库。  
  
**第二个：数据库连接池初始化失败。**  
 即使application-dev.yml  
里的username  
和password  
改对了，如果MySQL的root  
用户不允许从localhost  
以外的地址连接，或者MySQL 8.0的认证插件是caching_sha2_password  
而JDBC驱动版本较旧，仍然会报Access denied  
或Public Key Retrieval is not allowed  
。在JDBC URL里加上allowPublicKeyRetrieval=true&useSSL=false  
，或者把MySQL用户的认证插件改为mysql_native_password  
。  
  
**第三个：端口冲突。**  
 JavaSecLab默认监听8080。如果你本地跑着Tomcat、Burp Suite的代理、或者其他Spring Boot应用，8080可能被占用。启动时如果看到Port 8080 was already in use  
，在application.yml  
里加一行server.port=8081  
，或者杀掉占用8080的进程。  
  
**写在最后**  
  
JavaSecLab不是那种“玩一遍就通关”的靶场。它的每一个漏洞场景，都是一次**从代码出发、回到代码**  
的审计练习。你在SpEL注入里学到的SimpleEvaluationContext  
，会直接用在真实项目的代码审计里；你在反序列化模块里练的ObjectInputFilter  
白名单，会直接用在修复方案里；你在Actuator场景里配的exposure.include  
，会直接用在生产环境的加固清单里。  
  
**靶场的价值不在于“弹出了计算器”，在于“你知道为什么弹出计算器，以及怎么让它弹不出来”。**  
 JavaSecLab把这两件事都给了你。  
  
**严正声明**  
  
JavaSecLab包含故意设计的安全漏洞，**仅允许在隔离的本地环境或授权测试环境中运行**  
。项目README明确警告：不要将靶场直接暴露在公网，不要将靶场中的漏洞代码用于未授权的系统。本文所述所有漏洞场景和利用方式仅用于安全学习、代码审计培训和授权范围内的安全测试。未授权扫描、测试、攻击行为违反《中华人民共和国网络安全法》第27条及《刑法》第285条，将承担法律责任。技术无罪，边界是法律。  
  
  
