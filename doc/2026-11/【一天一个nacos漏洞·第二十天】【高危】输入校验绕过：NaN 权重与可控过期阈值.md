#  【一天一个nacos漏洞·第二十天】【高危】输入校验绕过：NaN 权重与可控过期阈值  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-27 03:11  
  
   
  
# 【一天一个nacos漏洞·第二十天】【高危】输入校验绕过：NaN 权重与可控过期阈值  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个都是**输入校验绕过**  
，但踩的是两个不同的坑：  
  
一个用 if (x > 10000 || x < 0) throw  
 写范围判断——**传 NaN 就过了**  
，因为 NaN  
 和任何数比较都是 false。  
  
一个更隐蔽：**校验时用一个函数解析，执行时用另一个函数解析**  
，同一个字符串 "-1"  
，一处当成 30000（合法），一处当成 -1（立即删除）。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址与账号口令均未写入本文。内容仅用于学习与防御加固。  
  
  
01  
  
一、两个常见的校验坑  
  
写"值必须在某个范围内"的时候，有两个坑特别容易踩：  
  
**坑一：用比较符号表达范围**  
  
```
if (x > MAX || x < MIN) throw ...;      // 看起来无懈可击

```  
  
  
这段代码对正常数字是对的。但 NaN  
 不是"正常数字"——**它和任何值比较都返回 false**  
，包括它自己（NaN == NaN  
 也是 false）。  
  
所以 NaN > MAX  
 是 false，NaN < MIN  
 也是 false，**两个条件都不成立，直接放行**  
。  
  
**坑二：同一个值，两处解析**  
  
```
// 校验的时候
if (isNumber(value)) { use Long.parseLong(value); } else { use DEFAULT; }

// 执行的时候
long timeout = ConvertUtils.toLong(value);

```  
  
  
两个函数对同一个字符串可能有**完全不同的结论**  
。校验看到的是"默认值，合法"，执行看到的是"-1，立刻生效"。  
  
今天两个洞，正好各踩一个。  
  
<table><thead><tr><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">VE-0295 · NaN 权重</span></span></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">VE-0304 · 可控阈值</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><strong style="font-weight: 700;color: #1a1a1a;"><span leaf="">7.1</span></strong></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><strong style="font-weight: 700;color: #1a1a1a;"><span leaf="">8.1</span></strong></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">类型</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">输入校验绕过（数值语义）</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">安全阈值客户端可控</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">踩的坑</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">比较符号漏了 NaN</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">校验与执行解析不一致</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">后果</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">客户端负载均衡崩</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">实例被永久保留 / 被强杀</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0295：NaN 让两个比较都变成 false  
### 2.1 那行看起来很严的校验  
  
实例的权重必须在一定范围内，代码是这么做的：  
  
```
public static void checkWeight(Double weight) throws NacosException {
    if (weight > Constants.MAX_WEIGHT_VALUE      // 10000.0
        || weight < Constants.MIN_WEIGHT_VALUE) { // 0.0
        throw new NacosApiException(...);
    }
}

```  
  
  
**> 和 < 都写了，边界也写了，看着很完整。**  
  
但 weight = NaN  
 时：NaN > 10000  
 是 false，NaN < 0  
 也是 false → **整个条件为 false → 不抛异常**  
。  
  
而 Spring 的参数绑定会把表单里的 weight=NaN  
 直接绑成 Double.NaN  
（Double.valueOf("NaN")  
 是合法调用）。  
  
**一个字符串 NaN，就这么穿过了区间校验。**  
### 2.2 差分的证明  
  
光说"校验通过了"不够，得证明"是它被漏了，而不是别的环节拦住了"。报告做了个很干净的差分：  
  
```
weight=10001  → 400 weight error（区间校验生效）
weight=NaN    → 通过校验，进入后续业务逻辑

```  
  
  
**同一个入口、同一个参数，只有值不同。**  
 10001 被拦，NaN 被放。差异明确指向那个 if  
。  
### 2.3 一条很长的因果链  
  
这个洞真正有意思的地方，是 NaN 穿过校验之后还能走多远：  
  
```
① 校验通过
② buildMetadata.setWeight(NaN)            ← 无条件写入
③ Hessian 序列化                          ← 无损保存 NaN（实测往返 isNaN=true）
④ CP 写入 → 下发
⑤ InstanceUtil.updateInstanceMetadata     ← instance.setWeight(metadata.getWeight())，NaN 原样覆盖
⑥ 客户端 Chooser.Ref.refresh

```  
  
  
到第 ⑥ 步，事情变得很精彩。  
### 2.4 最漂亮的一处：两行之间  
  
客户端的加权随机选择器，逻辑大致是：  
  
```
// 第一遍：求和
for (Pair&lt;T&gt; item : itemsWithWeight) {
    double weight = item.weight();
    if (weight <= 0) { continue; }              // ← NaN 比较为 false，不被过滤
    if (Double.isNaN(weight)) { weight = 1.0D; } // ← 求和时，NaN 被顶替成 1.0
    originWeightSum += weight;
}

// 第二遍：算累计区间
for (Pair&lt;T&gt; item : itemsWithWeight) {
    double singleWeight = item.weight();
    if (singleWeight <= 0) { continue; }         // ← 这里拿的是**原始 NaN**
    exactWeight = singleWeight / originWeightSum;
    weights[index] = randomRange + exactWeight;  // ← 累计数组变成 NaN
    randomRange = weights[index++];
}

// 最后校验
if (Math.abs(weights[index - 1] - 1) < precision) return;
throw new IllegalStateException("Cumulative Weight calculate wrong ...");

```  
  
  
**第一遍求和时，代码特意把 NaN 当成 1.0；第二遍分配时，又拿原始的 NaN 去算。**  
  
于是：  
- 总和是"1 + 1 = 2"（NaN 被当成 1）  
- 但累计区间里出现了 NaN  
- 最终校验 |NaN - 1| < delta  
 → false → **抛异常**  
**“求和的时候把它当 1，分配的时候又用原始的它”——两处只隔了十几行，但结论完全矛盾。**  
  
结果就是：**只要这个服务里有任何一个实例的权重是 NaN，所有用加权选例的消费者，对这个服务都会持续抛 IllegalStateException。**  
  
selectOneHealthyInstance  
 直接不可用。  
### 2.5 一个意外：注册路径反而没事  
  
报告还发现一个有意思的细节：NaN 走**注册**  
接口时，反而不落库。  
  
```
if (Math.abs(DEFAULT_WEIGHT - instance.getWeight()) >= EPSILON) {
    extendDatum.put(PUBLISH_INSTANCE_WEIGHT, instance.getWeight());
}

```  
  
  
Math.abs(1.0 - NaN)  
 是 NaN，NaN >= EPSILON  
 是 **false**  
 → 不写入 → 回读是默认 1.0。  
  
**同一个 NaN 语义缺陷，在这里变成了"意外保护"。**  
 所以真正的落库点是 update  
 / partial  
 分支——那里是 setWeight()  
 无条件写入。  
> 📌 这也说明：**"NaN 比较返回 false"这件事，在不同地方会带来完全相反的后果**  
——有时候让坏事发生（绕过校验），有时候让坏事不发生（跳过写入）。**不能指望它当防线。**  
  
  
03  
  
三、VE-0304：校验看到 30000，执行看到 -1  
### 3.1 阈值是客户端给的  
  
服务端判断"临时实例多久没心跳就删掉"，用的是一个可配置的阈值。而这个阈值，来自**被判定实例自己提交的 metadata**  
：  
  
```
private long getTimeout(Service service, InstancePublishInfo instance) {
    Optional&lt;Object&gt; timeout = getTimeoutFromMetadata(service, instance);
    if (!timeout.isPresent()) {
        timeout = Optional.ofNullable(
            instance.getExtendDatum().get(PreservedMetadataKeys.IP_DELETE_TIMEOUT));  // ← 客户端值
    }
    return timeout.map(ConvertUtils::toLong).orElse(Constants.DEFAULT_IP_DELETE_TIMEOUT);
}

```  
  
  
**“你多久算过期”，由被判定的人自己说。**  
  
这句话本身就已经很奇怪了。但真正致命的是下面的解析差异。  
### 3.2 同一个字符串，两个函数，两个结论  
  
**执行期**  
（上面那段）用的是：  
  
```
ConvertUtils.toLong("-1")   → -1
ConvertUtils.toLong("abc")  → 0        // 解析失败返回默认 0

```  
  
  
**校验期**  
（注册时的合法性检查）用的是另一个：  
  
```
// Instance.getIpDeleteTimeout()
if (NamingUtils.isNumber(value)) { return Long.parseLong(value); }   // 正则 ^\d+$
return defaultValue;                                                  // 30000

```  
  
  
^\d+$  
 只认纯数字：  
  
```
"-1"   → 不匹配 → 当作 30000 → 30000 > 心跳间隔 → 校验通过
"abc"  → 不匹配 → 当作 30000 → 校验通过

```  
  
  
**校验时它看见的是"30000，很合理"；执行时它看见的是"-1，立刻过期"。**  
  
而判定逻辑是：  
  
```
return System.currentTimeMillis() - instance.getLastHeartBeatTime() > deleteTimeout;

```  
  
  
now - lastBeat  
 恒大于 -1  
 → **判定恒为真 → 一定删除**  
。  
### 3.3 两个相反的极端，都成立  
  
**方向一：永不删除**  
  
```
metadata = {"preserved.ip.delete.timeout": "9223372036854775807"}    // Long.MAX_VALUE
校验期：MAX > 5000 → 通过
执行期：now - lastBeat > MAX → 恒为 false → 永不删除

```  
  
  
实测：MAX 实例 96 秒仍在（默认阈值 30 秒，对照组 38 秒就被 [AUTO-DELETE-IP]  
 删了）。  
  
**方向二：立即删除**  
  
```
metadata = {"preserved.ip.delete.timeout": "-1"}
→ 首个 5 秒检查周期就被删

```  
  
  
**而这时候实例还在正常心跳。**  
 日志里写得很清楚：  
  
```
[AUTO-DELETE-IP] ... "healthy":true ... "preserved.ip.delete.timeout":"-1"

```  
  
  
**一个健康、正在心跳的实例，被自己提交的阈值判定为"过期"。**  
### 3.4 还能让它永远"健康"  
  
同一套机制也管着健康判定：  
  
```
metadata = {"preserved.heart.beat.timeout": "9223372036854775807"}
→ 实例从不心跳，却始终 healthy=true

```  
  
  
而列表接口的 healthOnly=false  
，会把这些"永远健康"的黑洞端点持续返回给消费者。  
  
**流量会一直打到一个已经死掉的地址上。**  
### 3.5 连"3 分钟客户端清理"都能绕过  
  
服务端还有个兜底：客户端 3 分钟没更新就整体清理。  
  
报告发现了一个绕过方式：**同一个 ip:port 下注册两个服务**  
，对其中一个正常心跳，就能持续刷新共享 client 的 lastUpdatedTime  
 —— 于是那个设了 MAX 阈值、从不心跳的实例，**超过 252 秒仍然存在**  
。  
  
**一个僵尸实例，配一个假的保活心跳，可以永久驻留。**  
### 3.6 还能强杀别人的健康实例  
  
因为服务端只用 ip:port  
 定位实例（这一点我们在第十一天见过一次），**用同样的 ip:port 注册并带上 -1 阈值**  
，就能覆盖受害者已有的登记项：  
  
```
before:  victim-beat = 200，list = [victim]
attacker 用相同 ip:port 注册 deleteTimeout=-1
t+4s:    victim-beat = 21003 instance not found
t+6s:    list = []

```  
  
  
**受害者一直在正常心跳，却被强制摘除了。**  
  
04  
  
四、两个合起来  
  
```
VE-0295：NaN 绕过了两个比较        →  一个"不合法"的值被当成合法
VE-0304：校验器和执行器各解析一次   →  同一份数据被读成两个意思

```  
  
  
一个在**判断的边界**  
上漏了一类值，一个在**解析的路径**  
上分叉了。  
  
但根子是同一句：  
  
**“校验通过"不等于"值合法”。**  
  
校验只是一组条件成立而已。NaN  
 让所有比较都是 false，于是"一个条件都不成立"——**看起来像"一个毛病都没有"**  
。而 "-1"  
 在一个正则里不是数字，在另一个函数里是负数——**两个都对，只是说的不是一回事**  
。  
  
05  
  
五、边界  
  
**0295**  
：需要实例写权限（未登录、只读都 403）。而且要注意——报告很诚实地写了：靶机 CP 无 Leader，**线上落库被 500 阻断**  
，"写入成功后的下发 DoS"是靠部署包里的**生产类**  
证明的（Hessian 往返保 NaN、下发保 NaN、Chooser  
 必抛异常）。**没有假装端到端跑通。**  
  
**0304**  
：同样需要客户端接入端口的写权限。另外它的两个"方向"里，**"立即删除"比"永久保留"更容易验证**  
，因为前者 5 秒就见效。  
> 📐 这两份的诚实程度值得单独说一下： 0295 明确区分了"实测到的"和"用生产类证明的"；0304 明确修正了"裸静默 PoC 只能活 3 分钟"这个先前判断，并补上了保活绕过。**承认边界，比多写一个利用方式更能说明报告可信。**  
  
  
06  
  
六、修复  
  
<table><thead><tr><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">比较式范围校验</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">显式判 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">Double.isNaN(x)</span></code><span style="display: inline;"><span leaf=""> 或 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">Double.isInfinite(x)</span></code><span style="display: inline;"><span leaf=""> 直接拒绝；或改用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">!(x &gt;= MIN &amp;&amp; x &lt;= MAX)</span></code><span style="display: inline;"><span leaf=""> 写法（NaN 会落进&#34;不满足&#34;，被拒）</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">参数绑定</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">InstanceForm</span></code><span style="display: inline;"><span leaf=""> 层增加数字格式与有限性校验，别让 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">NaN</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">Infinity</span></code><span style="display: inline;"><span leaf=""> 进业务层</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">二次防御</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">buildMetadata</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">mergeMetadata</span></code><span style="display: inline;"><span leaf=""> / </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">setWeight</span></code><span style="display: inline;"><span leaf=""> 处再校验一次，防内部调用绕过</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">客户端健壮性</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">Chooser</span></code><span style="display: inline;"><span leaf=""> 把 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">isNaN</span></code><span style="display: inline;"><span leaf=""> 与 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">isInfinite</span></code><span style="display: inline;"><span leaf=""> 同等处理；累计权重校验失败应降级而不是抛异常</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">阈值信任客户端</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">preserved.*</span></code><span style="display: inline;"><span leaf=""> 这些键必须服务端白名单 + 上下界；不能让客户端决定自己被判定过期的时间</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">校验/执行不一致</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">统一解析函数（都用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">ConvertUtils.toLong</span></code><span style="display: inline;"><span leaf="">），或直接拒绝非法/负数/超大值</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">健康判定同理</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">preserved.heart.beat.timeout</span></code><span style="display: inline;"><span leaf=""> 也要设上界，否则黑洞端点永远健康</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">实例定位</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">别只用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">ip:port</span></code><span style="display: inline;"><span leaf=""> 定位登记项，避免&#34;同地址覆盖&#34;导致强杀他人实例</span></span></td></tr></tbody></table>  
> 💡 一句可以加进 code review checklist 的话：**“写范围判断时，除了想 > 和 <，还要想 NaN；写校验时，除了想’拦不拦得住’，还要想’它和执行时读的是不是同一个东西’。”**  
  
  
07  
  
七、写在最后  
  
第二十天。  
  
今天这两个洞的分量都不算最高（7.1 / 8.1），但它们的"技术含量"我很喜欢——因为都是**逻辑层面的巧劲**  
，没有花哨的利用链：  
- 一个靠 NaN  
 的数学性质：**它和谁都不相等，也就和谁都不构成"越界"**  
；  
- 一个靠两处解析的分歧：**同一串字符，一个说"这不是数字"，一个说"这是负一"**  
。  
这两类问题有个共同点：**代码 review 时几乎看不出来。**  
  
if (x > 10000 || x < 0)  
 这一行，谁看都会觉得写对了；isNumber(v)  
 和 toLong(v)  
 分别看，各自也没毛病。**问题只在你把它们放在一起、并且拿一个"畸形值"去跑的时候才出现。**  
  
所以这一篇想留的最后一句是：  
> **校验函数返回 true，不等于值是安全的——它只等于"这个值没有触发我写的那些条件"。**  
而"没触发条件"和"是我要的东西"，中间隔着 NaN  
、-1  
、空串、超长、大小写……一整排你想不到的形态。  
  
  
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
：传输层认证缺陷——HTTP 侧 IP 主机名绕过（7.4），gRPC 侧 trustAll 缺省 true（7.4）。**第二十天**  
（本篇）：输入校验绕过——NaN 权重击穿负载均衡（7.1），客户端可控过期阈值（8.1）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
