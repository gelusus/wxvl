#  一仓库横扫 Java 生态高危漏洞！19 大模块、数百个可运行 PoC 的 Java 安全漏洞复现实验室  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-22 02:39  
  
免责声明： 本项目仅用于安全研究与漏洞复现学习，禁止用于任何非法攻击行为，所有使用后果由使用者自行承担。请在合法的测试环境中使用。
Java 生态的安全问题，长期以来分散在不同的 CVE 公告、私有技术博客和零散的复现仓库中。开发与运维人员想要验证某个依赖版本是否存在风险，往往需要自己拼接载荷、搭建回连服务、调试序列化协议。learnjavabug 把这些分散的工作收敛到同一个 Maven 工程里，以 283 个 Java 源文件、18 个顶层模块的规模，提供了从序列化利用链到认证绕过的完整实验环境。
重点导读项目定位

漏洞复现代码集合
版本条件对照实验
载荷与回连设施
特征检测样本
加固对照实现

这是一个以 Maven 多模块组织的 Java 工程，groupId 为 com.xyh，artifactId 为 learn-java-bug，编译目标为 Java 8。所有模块统一挂载在根 pom.xml 的 modules 列表下，彼此独立编译，便于按需拉取单个漏洞模块进行复现。
重点导读模块清单
PART 01序列化链路
fastjson 模块是整个仓库中体量最大的部分，按攻击类型划分为 rce、ssrf、dns、dos、leak、file 六个子包。仅 rce 包内就有 15 个独立 PoC，覆盖 TemplatesImpl、CommonsProxy、HikariConfig、JndiConverter、IbatisSqlmap、Shiro、Hadoop、IgniteJta 等多条利用路径，并标注了各自的受影响版本上限与 AutoType 前置条件。
jackson 模块结构类似，提供 rce 与 ssrf 两类载荷，涵盖 Anteros、Ehcache、H2、HikariConfig、Logback、Quartz、SpringAop、IbatisSqlmap 等 gadget。
dubbo 模块针对 hessian2 序列化协议，除 Rome、Resin、XBean、SpringAop 四条 gadget 链路外，还提供 java 原生与 commons-collections 系列的反序列化载荷，以及一组基于黑名单的 hessian2 加固对照代码。
common-collections 模块保留了对 commons-collections 各版本 Transformer 链的封装实现。
PART 02认证绕过
shiro 模块按 Shiro 版本拆分出 5 个独立子工程，分别对应 1.4.1、1.5.1、1.5.3、1.7.1、1.8.0。每个子工程是一套完整的 Spring Boot 演示应用，包含自定义 Realm、登录控制器、Shiro 配置类与绕过验证接口，可以直接启动并观察过滤链行为差异。
PART 03容器与框架
tomcat 模块覆盖两类问题：ajp-bug 子工程复现 AJP 协议任意文件读取与 JSP 渲染；sync-session-bug 子工程针对集群会话同步机制，实现了 Jdk7u21 与 Jdk8u20 版本的会话反序列化载荷。
spring 模块包含 spring-boot-actuator、spring-cloud-config-server 的三个 CVE 复现、spring-data-mongodb 的 SpEL 注入、spring-uricomponentsbuilder 的两版差异对照，以及 spring-session-redis-sync 的会话同步利用。
cas 模块按版本线组织，含 4.1.x 至 4.1.6 的默认密钥反序列化、4.1.7 至 4.2.x 的签名密钥反序列化、5.x 子工程，以及一个独立的 CAS4 padding oracle 实验。
PART 04其他组件

nexus：三个 CVE 的复现说明文档
ShardingSphere-UI：YAML 反序列化复现
apache-poi：Excel 解析漏洞复现
rmi：JDK 8u121、8u191 前后 JNDI 查找行为差异与服务端实现
xxe：DOM、SAX、SAXReader、XStream、Transformer 等解析器测试
security-manager：SecurityManager 绕过实验
java-compile：Javassist 与 javac 动态编译、运行时生成类
padding-oracle-cbc：多组与单组密文的 CBC 填充预言机实现
feature：序列化特征样本
common：公共支撑库

重点导读关键技术点
PART 05版本矩阵
每个 PoC 在类注释或 README 中标注受影响版本区间与触发前置条件，例如 fastjson 的 NoNeedAutoTypePoc 覆盖 1.2.48 以下版本且默认配置即可触发，而 CommonsProxyPoc 需要 1.2.61 之前版本并开启 AutoType。版本边界是复现前必须确认的第一道参数。
PART 06载荷构造
com.threedr3am.bug.common.support.ClassFiles 负责生成含恶意方法的字节码，TemplatesUtil 负责将其封装进 TemplatesImpl，Reflections 封装反射调用。这套组合被 fastjson、jackson、dubbo 三个模块共用。
PART 07回连通道
common 模块内置 HTTP、FTP、LDAP 三类本地服务端，配合 JDK 原生 JNDI 与 DNS 解析构成完整回连链路，覆盖 DNSlog、外带请求与 LDAP 注入等多种取证方式。
PART 08特征提取
feature 模块单独抽取各序列化格式的特征生成逻辑，覆盖 Java 原生序列化、Fastjson、Jackson、Hessian、Kryo、XStream、SnakeYAML、Jolokia 请求以及 Java 类字节码。该模块的用途是产出可检测的流量特征。
重点导读工程结构

Maven 多模块聚合
统一 com.threedr3am.bug 包前缀
依赖版本独立管理
JUnit 4.11 测试依赖
编译级别 Java 8

模块之间共用 common 模块的字节码生成、模板封装、反射工具与本地回连服务实现，复用逻辑集中在 support 与 utils 两个子包中。
重点导读适用场景

依赖组件版本风险自查
安全补丁回归验证
漏洞原理复现与教学
流量特征规则调试
加固方案有效性对比

重点导读合规提示
仓库中的每一个模块都会在真实进程中触发远程代码执行、任意文件读取、认证绕过或拒绝服务行为。请确保实验在隔离的本地环境中进行，关闭外网出口，禁止在生产系统或未授权的第三方资产上运行。涉及第三方组件的利用代码，请严格遵守对应软件的许可协议与授权范围。
重点导读开源地址
本公众号非项目作者，仅做技术分享。
本文介绍的项目开源地址如下：
https://github.com/threedr3am/learnjavabug

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
