#  【一天一个nacos漏洞·第十一天】【高危】注册覆盖和实例删除  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-13 05:43  
  
   
  
# 【一天一个nacos漏洞·第十一天】【高危】注册覆盖和实例删除  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天讲 Nacos 的心脏——**服务注册表**  
。  
  
一个实例注册上来，Nacos 要给它一个内部标识，叫 clientId  
。这个 ID 是这么算的：  
  
```
clientId = ip + ":" + port + "#" + ephemeral

```  
  
  
**它不是你证明自己是谁换来的，而是你报上来的地址拼出来的。**  
  
于是：谁都能算出你的 clientId，谁都能拿它去改你的注册、删你的实例。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与口令均未写入本文。内容仅用于安全学习与防御加固。  
  
  
01  
PART  
  
一、先说清楚 clientId 是什么  
  
微服务注册到 Nacos 时，报三样东西：服务名、IP、端口。Nacos 把这条记录存进注册表，别的服务来查，就拿到这份地址列表。  
  
内部管理这些记录时，Nacos 用 clientId  
 标识"**这是哪个客户端**  
"：  
  
```
public static String getClientId(String address, boolean ephemeral) {
    return address + ID_DELIMITER + ephemeral;   // ID_DELIMITER = "#"
}
// 例：192.0.2.5:8080#true

```  
  
  
这个设计本身不复杂。问题是——**它被当成了身份**  
。  
  
因为 ip  
、port  
、ephemeral  
 三个值**全部来自请求体**  
，而服务端从头到尾没问过一句：“这个 clientId，真的是你在用吗？”  
  
<table><thead><tr><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">VE-0548 · 注册侧</span></span></th><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">VE-0552 · 注销侧</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">7.1</span></strong></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">7.1</span></strong></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">报别人的 ip:port 注册，直接覆盖他的记录</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">报别人的 ip:port 注销，把他从注册表删掉</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">同一根因</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">clientId = ip:port<a class="wx_topic_link" topic-id="mwejsgpo-i1lyj5" style="color: #576B95 !important;" data-topic="1" data-recommend="">#ephemeral</a></span></code></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">同一行</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">最低权限</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">已认证普通用户 + 目标服务写权限</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">同左</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
02  
PART  
  
二、VE-0548：报上别人的地址，就能改他的注册  
### 2.1 注册那几行  
  
```
NamingUtils.checkInstanceIsLegal(instance);       // 只校验 ip 非空、port 范围、cluster 正则
boolean ephemeral = instance.isEphemeral();
String clientId = IpPortBasedClient.getClientId(instance.toInetAddr(), ephemeral);
createIpPortClientIfAbsent(clientId);             // ← 只判断"存在吗"
Service service = Service.newService(namespaceId, groupName, serviceName, ephemeral);
clientOperationService.registerInstance(service, instance, clientId);

```  
  
  
createIpPortClientIfAbsent  
 这个名字其实已经把答案写出来了：**“如果不存在就创建”，存在就直接复用**  
。  
  
它从不判断"这个已经存在的 client 是不是你的"。  
### 2.2 于是覆盖是必然的  
  
registerInstance  
 最后落到这一行：  
  
```
publishers.put(service, instancePublishInfo);     // 覆盖式写入

```  
  
  
Map.put  
，同 key 直接盖掉。  
  
所以：**你报一个和受害者完全相同的 ip:port，就会命中同一个 clientId，然后他的注册信息被你整条覆盖。**  
  
实测（自建测试服务，ip:port  
 用文档地址表示）：  
  
```
受害者（A 账号）注册：192.0.2.10:7001，weight=1.0，metadata={"owner":"devuser"}
攻击者（B 账号）注册：192.0.2.10:7001，weight=0.2，metadata={"owner":"attacker-tampered"}
→ 200 ok

受害者视角再读：
  weight = 0.2                    ← 被改了
  metadata.owner = attacker-tampered  ← 被改了

```  
  
### 2.3 更要命的是：心跳不能还原  
  
微服务会定期发心跳。直觉上——“下次心跳总该把真实信息刷回来吧？”  
  
不能。因为心跳只更新两样东西：  
  
```
lastHeartBeatTime   ← 会更新
healthy             ← 会更新

```  
  
  
**weight、enabled、metadata 都不会被心跳覆盖回去。**  
  
所以攻击者改一次，就一直是改过的状态。而 weight  
 在客户端负载均衡里意味着什么，做后端的都清楚——**权重调低，流量自然就绕开它了。**  
### 2.4 删也不用商量  
  
同一个 clientId，直接发注销：  
  
```
DELETE /nacos/v3/client/ns/instance?...&ip=192.0.2.10&port=7001
→ 200 ok

```  
  
  
受害者那边再发心跳，收到的是：  
  
```
{"code":21003,"message":"instance not found"}

```  
  
  
**他的实例已经不在注册表里了。**  
### 2.5 顺带两个副作用  
  
**一是失败也留垃圾。**  
 注册在"创建客户端"之后才做服务类型校验，所以校验失败时，客户端已经建好了、且不会被回收：  
  
```
POST 注册 persistent 实例到一个 ephemeral 服务
→ 400 Current service ... is ephemeral service, can't register persistent instance.
→ 但客户端列表里已经多了 192.0.2.40:7101#false

```  
  
  
而 PersistentIpPortClientManager**没有过期清理器**  
（临时实例那边有），健康检查任务还会在 finally  
 里无条件重排自己。  
  
**结果是：一个永远不会过期、又没法通过 API 删掉的客户端，外加一个自循环的检查任务，一直挂到进程重启。**  
  
**二是抢名字。**  
 先用 ephemeral=false  
 注册一个服务名，之后任何人再用正常方式注册这个服务，都会被拒：  
  
```
Current service DEFAULT_GROUP@@xxx is persistent service, can't register ephemeral instance.

```  
  
  
因为 Service.equals/hashCode  
 不包含 ephemeral，持久服务单例会永久占住这个名字。  
  
03  
PART  
  
三、VE-0552：同一个推导，删也不用证明  
### 3.1 注销那几行  
  
```
boolean ephemeral = instance.isEphemeral();
String clientId = IpPortBasedClient.getClientId(instance.toInetAddr(), ephemeral);
if (!clientManager.contains(clientId)) {          // ← 同样只判断"存在吗"
    Loggers.SRV_LOG.warn("remove instance from non-exist client: {}", clientId);
    return;
}
Service service = Service.newService(namespaceId, groupName, serviceName, ephemeral);
clientOperationService.deregisterInstance(service, instance, clientId);

```  
  
  
**同一个 clientId，同一句 contains。**  
 它问的是"这个客户端在不在"，不是"这个客户端是不是你的"。  
  
而上层的 @Secured  
 只做**服务级**  
鉴权（namespace:group:service  
 的写权限），**根本不存在实例级的归属校验**  
。  
### 3.2 实测：A 注册，B 注销  
  
```
【管理员注册】192.0.2.30:18091  → 200
【实例列表】  [192.0.2.30:18091, ephemeral=true]
【普通用户注销】DELETE ...&ip=192.0.2.30&port=18091  → 200 ok
【2 秒后列表】[]                                    ← 没了

```  
  
  
服务端日志记录得很清楚：  
  
```
Client connection 192.0.2.30:18091#true connect
Client remove for service Service{...}, 192.0.2.30:18091#true

```  
  
  
**两个不同的人，一个注册、一个删除，注册表只认那个字符串。**  
### 3.3 但这份报告最值得学的，是它证伪了什么  
  
原始清单给了四五种利用方式，报告逐条去试，然后**大部分自己否掉了**  
：  
  
<table><thead><tr><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">原始主张</span></span></th><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">实测结果</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">ephemeral=false</span></code><span style="display: inline;"><span leaf=""> 能绕过 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">contains</span></code><span style="display: inline;"><span leaf=""> 守卫</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">不成立</span></strong><span style="display: inline;"><span leaf="">：返回 200 但实例还在，日志说 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">non-exist client</span></code></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">能跨服务任意注销</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">不成立</span></strong><span style="display: inline;"><span leaf="">：鉴权按请求的服务校验，改 serviceName 不删原实例</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">普通 SDK（gRPC）的实例也能被注销</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">不成立</span></strong><span style="display: inline;"><span leaf="">：gRPC 的 connectionId 不含 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">#</span></code><span style="display: inline;"><span leaf="">，永远命中不了 IpPortBasedClient</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">守卫失败仍会发布删除事件</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">不成立</span></strong><span style="display: inline;"><span leaf="">：只有真删掉了才发事件</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">服务级写权限用户可删该服务内任意实例</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><strong style="font-weight: 700;color: #059669;"><span leaf="">成立</span></strong><span style="display: inline;"><span leaf="">（这才是核心）</span></span></td></tr></tbody></table>  
  
**四条里否掉四条，只留一条真的。**  
 这种报告比"五个利用方式全部验证成功"的更可信——因为它证明作者真的去打了，而不是照着猜的往下写。  
  
而且它还留了一个残余风险没说死：group@@service  
 这种写法，鉴权解析器会拆成 svc  
，业务却用原始字符串——**校验的和生效的不是一个**  
。但作者如实写了"本靶机构不成实际越权，仅作为设计风险记录"。**没把猜测升格成结论。**  
  
04  
PART  
  
四、两个加起来，意味着什么  
  
```
注册：报 ip:port → 算出 clientId → 存在就复用 → 覆盖他的记录
注销：报 ip:port → 算出 clientId → 存在就复用 → 删掉他的记录

```  
  
  
**注册和注销，用的是同一个"身份"，而这个身份是算出来的。**  
  
所以真正的授权边界退化成了一句话：**谁能写这个服务，谁就能对服务里任何实例做任何事。**  
  
而"服务级写权限"在微服务团队里通常是很宽松的——很多部署直接给 public:*:*  
。  
  
05  
PART  
  
五、边界：不是零权限就能打  
  
得说清楚这两份的门槛：  
- **需要已认证**  
：未登录 401/403，只读账号 403；  
- **需要目标服务的写权限**  
：不能跨命名空间（非 public 会被 403）；  
- **控制台不适用**  
：8080 的 console 接口明确禁止删临时实例（返回 400，Console only supports deregistering persistent instances  
）；  
- **8848 通常只内网可达**  
：很多部署只对外开 8080，这两条 API 都在 8848。  
> 📐 还有一点报告自己承认得很干脆：**单次删除是"可自愈"的**  
——受害者客户端心跳失败后会重新注册，所以可用性影响只给 L。 但反过来想：**攻击者也可以反复删。**  
 一个循环跑着，对方的实例就一直处于"注册—被删—重注册"的抖动里。这种攻击不需要多高的技术，只需要一个 for 循环。  
  
  
06  
PART  
  
六、修复：把"标识"和"身份"分开  
  
<table><thead><tr><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #BBF7D0;padding: 8px 10px;word-break: keep-all;font-weight: 700;background: #ECFDF5;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">clientId 由请求参数推导</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">引入真实调用者身份：</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">username@appName</span></code><span style="display: inline;"><span leaf="">、服务端会话 ID，或注册时</span></span><strong style="font-weight: 700;color: #059669;"><span leaf="">颁发一个不可猜测的 client token</span></strong></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">contains</span></code><span style="display: inline;"><span leaf=""> 即复用</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">注册/注销/心跳都要校验&#34;这个 client 是不是调用者的&#34;</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">只有服务级鉴权</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">补实例级归属校验；至少&#34;注销他人实例&#34;要求管理级权限</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">失败路径不回收</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">createIpPortClientIfAbsent</span></code><span style="display: inline;"><span leaf=""> 之后若校验失败，异常路径要回收刚建的客户端</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">持久客户端不过期</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">PersistentIpPortClientManager</span></code><span style="display: inline;"><span leaf=""> 补上过期清理，和临时实例那边对齐</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">服务类型抢占</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">创建服务单例前先判定类型冲突，别让持久单例永久占名</span></span></td></tr><tr><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #059669;background: #ECFDF5;padding: 2px 6px;border-radius: 4px;white-space: nowrap;"><span leaf="">group@@service</span></code><span style="display: inline;"><span leaf=""> 解析不一</span></span></td><td style="border: 1px solid #E5E7EB;padding: 8px 10px;word-break: keep-all;color: #374151;"><span style="display: inline;"><span leaf="">鉴权解析器和业务代码用同一个规范化函数</span></span></td></tr></tbody></table>  
> 💡 一句话原则：**标识（identifier）可以重名，身份（identity）必须可验证。**  
把一个"从参数算出来的字符串"当身份用，等于把门禁卡做成了"你报个门牌号我就让你进"。  
  
  
07  
PART  
  
写在最后  
  
第十一天了。回头看，前面十天的洞多少都还有"某个地方没写校验"的影子，今天这个更彻底一点：  
  
**它压根没打算校验，因为它以为 ip:port 就是身份。**  
  
在单机、可信内网的年代，这个假设也许能将就——反正能连上 8848 的都是自己人。但一旦有多个团队共用一套 Nacos、一旦任何人只要有服务写权限，这个假设就塌了：**地址是可以随便填的，而身份不可以。**  
  
这也是为什么"注册中心"这类组件特别值得盯：  
- 它不存业务数据，但它**决定流量去哪**  
；  
- 它的数据看起来"只是地址列表"，但改一条 weight，就能让一台机器慢慢饿死；  
- 它的写入接口通常被当成"内部接口"，权限给得很松。  
**改一条注册记录，不需要拿到那台机器，就能让它的流量归零。**  
 这比很多"高危 RCE"更安静。  
  
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
：服务间共享身份——凭据被主动投递（7.4），静态值可永久重放（8.1）。**第十天**  
：发布审核被架空——审核后换内容（8.1），审核漏扫文件（7.7）。**第十一天**  
（本篇）：注册表身份由 ip:port 推导——可覆盖他人注册（7.1），可删除他人实例（7.1）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
