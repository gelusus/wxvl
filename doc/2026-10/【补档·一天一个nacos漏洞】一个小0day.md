#  【补档·一天一个nacos漏洞】一个小0day  
原创 YNsec
                        YNsec  YNsec安全实验室   2026-10-29 03:27  
  
   
  
**免责声明：**本文所述测试全部在授权靶场环境中完成，公网 IP、VPC 地址、主机名与口令均已脱敏（IP 使用 RFC5737 文档专用段）。内容仅用于安全学习与防御加固，请勿用于任何未授权测试。  
## 漏洞简介  
  
Nacos 是阿里巴巴开源的**注册中心 + 配置中心**  
，在 Spring Cloud Alibaba 生态里几乎是默认选项，部署量很大。  
  
漏洞位于**客户端库**nacos-client  
：拉取配置后会在本地磁盘存一份“快照”用于降级读取，而快照路径由 tenant  
（namespace）、group  
、dataId  
 三段外部可控字符串**直接拼接**  
，没有任何基于基准目录的包含性校验。  
  
<table><thead><tr><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">影响组件</span></span></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">nacos-client</span></code><span style="display: inline;"><span leaf="">（客户端库）</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">影响版本</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">3.2.4 及更早</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">漏洞类型</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">路径穿越</span></span></td></tr></tbody></table>  
### 利用条件  
  
需要同时满足以下四项，缺一不可：  
  
<table><thead><tr><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">条件</span></span></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">具体内容</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">① 能写入配置</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">具 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">WRITE</span></code><span style="display: inline;"><span leaf=""> 权限的账号，或鉴权被放宽</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">② 服务端放行</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">nacos.core.param.check.enabled=false</span></code><strong style="font-size: inherit;color: #009874;font-weight: bold;"><span leaf="">且</span></strong><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">config_disk_type=rocksdb</span></code><span style="display: inline;"><span leaf="">（均为非默认）</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">③ 客户端 sink 可达</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">调用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">getConfigAndSignListener</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">addListener</span></code><span style="display: inline;"><span leaf="">，或 namespace 可被外部影响</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">④ 能被执行</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">客户端进程权限足够，且落点被特权程序消费</span></span></td></tr></tbody></table>  
## Nacos 详解  
### 服务端与客户端  
  
很多初次接触 Nacos 的人会以为它是单体服务，实际上分两半：  
  
```
┌──────────────┐        ┌──────────────┐
│ 订单服务       │        │ 用户服务       │   ← 业务进程（各自内嵌客户端库）
│ + nacos-client│        │ + nacos-client│
└───────┬───────┘        └───────┬──────┘
        │  8848 / 9848            │
        └────────────┬────────────┘
                     ▼
            ┌─────────────────┐
            │   Nacos Server  │                ← 中心，一个集群
            └─────────────────┘

```  
  
  
<table><thead><tr><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">服务端</span></span></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">客户端</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">形态</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">独立进程（</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">nacos-server.jar</span></code><span style="display: inline;"><span leaf="">）</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><strong style="font-size: inherit;color: #009874;font-weight: bold;"><span leaf="">库</span></strong><span style="display: inline;"><span leaf="">（</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">nacos-client-*.jar</span></code><span style="display: inline;"><span leaf="">）</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">数量</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">一个集群</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">无上限，每个应用一个</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">端口</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">8080 / 8848 / 9848 / 9849 / 7848</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">不监听端口</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">作用</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">存配置、维护实例列表</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">拉配置、注册自己、订阅变更</span></span></td></tr></tbody></table>  
### 为什么要有“快照”  
  
配置中心挂了业务就启动失败，这在生产上不可接受。所以客户端做了本地缓存：  
  
```
第 1 次拉取    服务端返回 → 写快照到本地
服务端宕机
第 2 次拉取    服务端不可达 → 读本地快照 → 服务仍能启动

```  
  
  
设计本身没问题。但只要是「**按外部数据拼路径写本地文件**  
」的设计，就要问一句：拼出来的路径，还在该在的目录里吗？  
### 快照目录怎么拼  
  
```
// LocalConfigInfoProcessor.java
static File getSnapshotFile(String envName, String dataId, String group, String tenant) {
    File snapshotDir = new File(LOCAL_SNAPSHOT_PATH, envName + "_nacos");  // 默认 ${user.home}/nacos/config

    if (StringUtils.isBlank(tenant)) {
        snapshotDir = new File(snapshotDir, "snapshot");
    } else {
        snapshotDir = new File(snapshotDir, "snapshot-tenant");
        snapshotDir = new File(snapshotDir, tenant);          // ← 外部输入 ①
    }

    return new File(new File(snapshotDir, group), dataId);     // ← 外部输入 ②③
}

```  
  
  
三段拼接，**没有 getCanonicalPath()，没有 normalize()，也没有一句“结果是否还在基准目录下”**  
。  
  
写入与删除共用同一路径：  
  
```
static void saveSnapshot(..., String config) {
    File file = getSnapshotFile(envName, dataId, group, tenant);

    if (null == config) {
        IoUtils.delete(file);                              // 目录则递归删除
    } else {
        file.getParentFile().mkdirs();                     // 逐段创建
        IoUtils.writeStringToFile(file, config, "UTF-8");
    }
}

```  
  
  
因此同一处缺陷同时构成**任意文件写**  
与**任意目录删除**  
。  
## 漏洞复现 & 分析  
### 第一步：找到能传进恶意参数的入口  
  
```
// NacosConfigService.java
private String getConfigInner(...) {
    ParamUtils.checkKeyParam(dataId, group);        // 有校验，含 '/' 抛异常
}

// :103 —— 没调用
public String getConfigAndSignListener(String dataId, String group, long timeoutMs, Listener listener) {
    ConfigResponse configResponse = worker.getAgent()
        .queryConfig(dataId, group, worker.getAgent().getTenant(), timeoutMs, false);
}

// :125 —— 也没调用
public void addListener(String dataId, String group, Listener listener) {
    worker.addTenantListeners(dataId, group, Collections.singletonList(listener));
}

```  
  
  
**第一个疑问**  
：同一个类、同一个校验函数，为什么有的方法调了有的没调？  
  
因为**校验分散在各个调用方，而非收敛在公共入口**  
。新增 API 时“忘了调”，就多一个洞。更糟的是：**tenant 在所有入口都没被校验**  
——哪怕走 getConfig  
，穿越序列塞进 tenant 照样能过。  
### 第二步：确认服务端拦不拦  
  
先发布一条 namespace 带穿越序列的配置：  
  
```
curl -X POST "http://<server>:8080/v3/console/cs/config" \
  -H "accessToken: $TOKEN" \
  --data-urlencode "dataId=<任意>" \
  --data-urlencode "groupName=." \
  --data-urlencode "namespaceId=<含 ../ 的 namespace>" \
  --data-urlencode "content=<任意>" \
  --data-urlencode "type=text"

```  
  
  
返回：  
  
```
{
  "code": 20002,
  "message": "parameter validate error",
  "data": "Param 'namespaceId/tenant' is illegal, illegal characters should not appear in the param."
}

```  
  
  
被拦了。需要服务器关掉参数校验并重启：  
  
```
echo 'nacos.core.param.check.enabled=false' >> $NACOS_HOME/conf/application.properties
systemctl restart nacos

```  
  
  
关掉后响应应为 {"code":0,"message":"success","data":true}  
。  
### 第三步：客户端拉取  
  
用**不做 key 校验的那个 API**  
 消费：  
  
```
Properties p = new Properties();
p.put("serverAddr", "127.0.0.1:8848");
p.put("namespace", NS);      // 外部可控
p.put("username", USER);
p.put("password", PWD);

ConfigService cs = NacosFactory.createConfigService(p);
String content = cs.getConfigAndSignListener(dataId, group, 3000, listener);

```  
  
  
结果：  
  
```
ErrCode:500, ErrMsg:http error, code=500,  msg=errCode: -400, errMsg: parameter is invalid.

```  
  
  
**第二个疑问**  
：参数校验明明关了，为什么还有 -400  
？  
  
堆栈指向 ConfigRawDiskService.targetFile()  
：  
  
```
at ConfigRawDiskService.targetFile(ConfigRawDiskService.java:69)at ConfigRawDiskService.getContent(ConfigRawDiskService.java:166)at FormalHandler.handle(FormalHandler.java:52)at ConfigQueryRequestHandler.handle(ConfigQueryRequestHandler.java:94)

```  
  
  
反编译该方法：  
  
```
static java.io.File targetFile(String, String, String);
   3: invokestatic  ParamUtils.checkParam(String, String, String)   ← 无条件调用！
  14: sipush  -400
  17: ldc     "parameter is invalid."
  22: athrow

```  
  
  
targetFile()  
 里的 checkParam**完全不受 param.check 开关控制**  
——那个开关只管 ParamCheckerFilter  
。  
  
这正是磁盘后端语义差异造成的：  
  
```
// ❌ rawdisk：namespace 是【文件路径的一段】
File file = targetFile(dataId, group, tenant);   // 内部无条件 checkParam

// ✅ rocksdb：namespace 是【KV key 的一个字段】
return getContentInner(BASE_DIR, dataId, group, tenant);  // 无 checkParam

```  
  
  
切换后端：  
  
```
# systemd 单元里追加 JVM 参数
Environment="CUSTOM_NACOS_MEMORY=... -Dconfig_disk_type=rocksdb"
systemctl daemon-reload && systemctl restart nacos
ls $NACOS_HOME/rocksdata/config-data/    # 确认生效

```  
  
  
<table><thead><tr><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">后端</span></span></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">客户端读取结果</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">rawdisk</span></code><span style="display: inline;"><span leaf="">（默认）</span></span></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">❌ </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">-400 parameter is invalid.</span></code></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">rocksdb</span></code></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">✅ 正常返回，进入本地写盘</span></span></td></tr></tbody></table>  
### 第四步：算要退几级  
  
基准路径 = ${user.home}/nacos/config/<env>_nacos/snapshot-tenant  
。user.home=/root  
 时逐级回退：  
  
```
起点  /root/nacos/config/<env>_nacos/snapshot-tenant
 1..4 → /root
 5,6  → /       （到顶后不再变化）

```  
  
  
深度由 user.home  
 决定：  
  
<table><thead><tr><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">user.home</span></code></th><th style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(0,152,116,0.08);"><span style="display: inline;"><span leaf="">到 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">/</span></code><span style="display: inline;"><span leaf=""> 所需 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">..</span></code></th></tr></thead><tbody><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">/root</span></code></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">6</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">/home/appuser</span></code></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">7</span></span></td></tr><tr><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">容器内 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #009874;background: rgba(0,152,116,0.08);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">/app</span></code></td><td style="border: 1px solid #dfdfdf;padding: 0.4em 0.8em;word-break: keep-all;color: #3f3f3f;"><span style="display: inline;"><span leaf="">3~4</span></span></td></tr></tbody></table>  
  
深度错了落点完全偏离，**而且不会报错**  
。  
### 第五步：为什么这条链“写两次”  
  
快照环境目录名由 namespace 派生：/  
 和 :  
 都替换成 _  
，超长截断加 md5。于是两个不同的 namespace 可以映射到**同一个目录**  
：  
  
```
NS1 = public:../.._.._.._.._.._.._<目标目录扁平化>   ← 全字面量，无真正的 /
NS2 = public:../../../../../../<目标目录>            ← 真正的穿越

清洗后两者 → Config-fixed-<清洗后的 NS><env 哈希>_nacos
              └──────── 完全相同 ────────┘

```  
  
  
**第一次写**  
（阶段一）：提交 NS1，本身不穿越，落盘结构为  
  
```
${user.home}/nacos/config/<env>_nacos/snapshot-tenant/
    public:..                                  ← 一个真实的目录名，不是退格
        .._.._.._.._.._.._<扁平化的目标目录>
            <dataId>

```  
  
  
它写的文件仍在基准目录内，看起来毫无攻击性。唯一价值是副产品：留下一个名叫 public:..  
 的真实目录。  
> public:..  
 是**完整的目录名**  
，不是 public:  
 + ..  
。Linux 允许文件名带冒号和点。  
  
  
**第二次写**  
（阶段二）：提交 NS2，此时 ..  
 才能真正生效，落点逃逸到 ${user.home}  
 之上的目标目录。  
### 第六步：从落地目录到 root  
  
系统计划任务目录的特性：落地即生效（crond 每分钟扫描）、格式极简（一行）、文件名只要不带点。  
  
叠加上“客户端以 root 运行”，写进去的文件属主就是 root，crond 也就以 root 执行。  
  
/etc/cron.d  
 下是标准 cron 格式：五段时间字段 + 运行用户 + 命令。**这里不给出具体命令行内容**  
——格式本身是公开的，关键结论是：只要写进去的格式合法，crond 就会以文件属主的身份执行它；而属主是 root，是因为写文件的进程以 root 运行。  
## 链路  
  
```
[8080] 发布 namespace 含 ../ 的配置（需 WRITE 权限）
   ↓  服务端要求：param.check=false  且  config_disk_type=rocksdb
[8848/9848] 业务应用中的 nacos-client 拉取
   ↓  getConfigAndSignListener 不校验 key
[本地] saveSnapshot() 路径拼接，无 canonical 校验
   ↓
[业务主机] 落地cron.d

```  
  
## 写在最后  
  
这条链最值得学的不是穿越技巧，而是它的**失败方式很有代表性**  
：  
  
四个位置都有校验，单独看每一处都能说得通，甚至注释里就写着“防止路径穿越”。可它们分别管着 dataId、管着正则、管着另一个 API 的参数、管着“我以为这里不需要”。  
  
安全的边界从来不由“有没有写校验”决定，而由「**输入变坏的那一刻，最靠近落盘的那个函数有没有再问一句**  
」决定。  
  
至于 payload——先写一个名叫 ..  
 的目录，再让 ..  
 真的往上退。它利用的不是“点号特殊”，而是“点号在某个语境里不特殊”：同一串字符，在目录名里是普通字符，在路径解析时是退格符。**两个子系统对同一个字符串的理解不一致，缝隙就在那里。**  
  
这条路走完，这一篇就先到这。**下一篇文章见。**  
  
📚 系列回顾 · 【一天一个nacos漏洞】  
  
本文是该系列第一篇，讲的是"客户端快照路径穿越"这条链。后续会继续按**单洞 → 成链 → 穷举**  
的顺序拆，一个洞一篇，尽量把每一处断点都写清楚。  
  
文中所有测试均在授权靶场完成，IP、主机名、口令均已脱敏。若你在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
— 完 —如果这篇对你有帮助，欢迎**在看 / 转发**  
给做安全的朋友  
  
   
  
  
