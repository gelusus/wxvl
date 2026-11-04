#  【一天一个nacos漏洞·第四天】【严重】同一个东西有两个名字，一个分隔符谁都没转义  
YNsec
                    YNsec  YNsec安全实验室   2026-11-04 03:53  
  
   
  
# 【一天一个nacos漏洞·第四天】【严重】同一个东西有两个名字，一个分隔符谁都没转义  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个，一个 9.9，一个 9.1。  
  
共同点不在"鉴权对象错位"——那个第二天就写过了。今天的问题更靠底层：**标识符是怎么造出来的**  
。  
  
一个是**同一个东西有俩名字**  
：鉴权认 tenant  
，业务认 namespaceId  
，两边各叫各的，谁也不认谁。  
  
一个是**分隔符没人转义**  
：拿 :  
 把五个字段粘成一个 key，可 :  
 本身就能出现在字段里。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与口令均未写入本文。内容仅用于安全学习与防御加固。  
  
## 一、先说结论：这两份到底在讲什么  
  
<table><thead><tr><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">VE-0023 · 配置接口</span></span></th><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">VE-0737 · Prompt/Skill 存储</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">9.9</span></strong><span style="display: inline;"><span leaf="">（全套最高之一）</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">9.1</span></strong><span style="display: inline;"><span leaf="">（原始 High，实测重算 Critical）</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">命名空间有俩名字，鉴权认一个，业务认另一个</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">存储 key 用 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">:</span></code><span style="display: inline;"><span leaf=""> 拼接，可 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">:</span></code><span style="display: inline;"><span leaf=""> 是合法字符</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">后果</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">有权限碰具名空间的人，能读/写/删/导出 public</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">用冒号构造别名，覆盖、删除他人的 Prompt 和 Skill</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
两条线看起来不搭，其实是同一件事的两面：**一个标识符，要么名字对不上，要么分隔符靠不住。**  
## 二、VE-0023：同一个东西，有两个名字  
### 2.1 命名空间，其实一直有俩别名  
  
Nacos 的配置命名空间，历史上同时叫过两个名字：老一点的接口用 tenant  
，v1 之后改叫 namespaceId  
。为了兼容，鉴权层做了个"回退"——先读 namespaceId  
，读不到就退回去读 tenant  
。  
  
问题出在**业务层没跟着做这个回退**  
。  
### 2.2 一边认 tenant，一边认 namespaceId  
  
```
// 鉴权侧：先 namespaceId，为空回退 tenant
protected String getNamespaceId(HttpServletRequest request) {
    String namespaceId = request.getParameter("namespaceId");
    if (StringUtils.isBlank(namespaceId)) {
        namespaceId = request.getParameter("tenant");   // ← 回退认了 tenant
    }
    return NamespaceUtil.processNamespaceParameter(namespaceId);
}

```  
  
  
```
// 业务侧：ConfigForm 里只有 namespaceId 这一个字段
private String namespaceId = StringUtils.EMPTY;   // 根本没有 tenant 属性

```  
  
  
于是 Spring 绑定参数的时候，tenant  
 是个未知字段，**直接被丢掉**  
，namespaceId  
 保持空串，空串再被转成默认命名空间 public  
。  
### 2.3 结果：鉴权过的不是你要操作的  
  
一句话讲清楚这条链：  
  
```
请求  ?tenant=我的空间&groupName=...&dataId=...
  │
  ├─ 鉴权：namespaceId 为空 → 回退 tenant="我的空间" → 检查"我的空间"权限 → 通过 ✅
  │
  └─ 业务：tenant 被丢弃 → namespaceId="" → 转成 public → 实际操作 public ❌

```  
  
  
**鉴权检查的是"我的空间"，实际动的是 public。**  
 中间没有任何一句把它们对起来。  
### 2.4 决定性的三组对照  
  
<table><thead><tr><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">请求</span></span></th><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">结果</span></span></th><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">说明</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">tenant=我的空间</span></code></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">，返回 public 的明文</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">越权成功</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">namespaceId=public</span></code></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">403</span></strong></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">直接要 public 反而被拒</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">namespaceId=我的空间</span></code></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">200，返回自己空间</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">证明权限本身是好的</span></span></td></tr></tbody></table>  
  
第三行最说明问题：**权限模型没坏，坏的是"名字对不上"。**  
 你正正经经报自己的空间名，能读到自己的东西；换个拼法，就能读到 public。  
### 2.5 不只是读，全都能干  
  
读、写、删、批量删、列表、全文检索、导出（export2  
 直接吐一个含 public 明文配置的 ZIP）、clone、历史查询——**console 和 admin 两个入口，十几个接口，同一个根因**  
。  
  
尤其导出这条：GET /v3/console/cs/config/export2?tenant=我的空间&ids=5  
 会返回一个 ZIP，里面装着 public 的配置明文。生产里 public 命名空间往往堆着核心配置和密钥。  
> 🔍 **边界得说清楚**  
：因为业务侧 namespaceId  
 恒等于空、恒落 public，所以**只能打到默认命名空间 public，不能任意指定别的具名空间**  
。这是它的上限，但也是它最疼的地方——public 恰恰是多数部署里最值钱的那个命名空间。  
  
## 三、VE-0737：分隔符没人转义  
### 3.1 key 是用冒号粘出来的  
  
AI 资源（Prompt、Skill）存在 Nacos Config 里，存储键是这么拼的：  
  
```
// Prompt / typed 键（5 段）
namespaceId + ":" + resourceType + ":" + name + ":" + version + ":" + filePath

// Skill legacy 键（4 段）
namespaceId + ":" + skillName + ":" + version + ":" + filePath

```  
  
  
读回来的时候再 split(":", 5)  
 反解。  
  
问题只有一个字：**: 是 Nacos Config 参数的合法字符**  
（白名单 {'_','-','.',':'}  
 里有它）。字段里能出现 :  
，而 :  
 又用来切分字段——那切分的位置就会漂。  
### 3.2 两个身份，落到同一个物理位置  
  
```
逻辑 A：(promptKey="ve737v:b",  version="1.0.0")
逻辑 B：(promptKey="ve737v",    version="b:1.0.0")

两者拼出来都是：
  public:prompt:ve737v:b:1.0.0:content.json

反解后都成了：
  name="ve737v", version="b", dataId="1.0.0:content.json"

```  
  
  
**两个毫不相干的身份，指向同一个物理 Config。**  
 后写的人，静默覆盖先写的人。这和第三天那个"名字+"-"版本"是一回事，只不过分隔符从 -  
 换成了 :  
，而且这次跨了 Prompt 和 Skill 两个模块。  
### 3.3 三步走：写别名，覆盖，删  
  
**第一步，写一个带冒号的"自己的"资源。**  
 新建分支不做写权限检查，只要求 promptKey 非空、version 是 x.y.z  
：  
  
```
POST /v3/console/ai/prompt/draft
namespaceId=public&promptKey=ve737v:b&targetVersion=1.0.0&template=ATTACKER...
→ {"code":0,"data":"1.0.0"}

```  
  
  
**第二步，受害者按自己的身份读，读到的是你的内容：**  
  
```
GET /nacos/v3/client/ai/prompt?promptKey=ve737v&version=b:1.0.0
→ {"promptKey":"ve737v","version":"b:1.0.0","template":"ATTACKER..."}

```  
  
  
注意返回里的 promptKey  
 / version  
 被**强制改写成了请求值**  
——所以受害者看到的身份没错，内容却已经换了。**审计行里的 author、status、storage 全都原样没动。**  
  
**第三步，删。**  
 删自己的草稿，把别人 online 的物理存储也删掉：  
  
```
DELETE /v3/console/ai/prompt/draft  promptKey=ve737d:d
→ ok
受害者侧再读：resource not found，物理行计数 = 0

```  
  
  
DB 里那条记录还显示 online  
，客户端却已经 404 了——**故障隐蔽，很难排查。**  
### 3.4 同一个根因，Skill 那边也炸了  
  
冒号那套在 Skill 上一样成立，而且**不需要重启**  
（Prompt 的迁移入口要重启触发，Skill 不用）：  
1. 受害者有 skill v1@0.0.1  
；攻击者建一个 skill@v1  
，资源文件名故意写成 0.0.1:SKILL.md  
——4 段的 legacy 键被误判成 5 段的 typed 键，命中了受害者的   
SKILL.md  
；  
  
1. 或者，直接写 skillName="skill:otherskill"  
 会被格式校验拦下（只许小写字母数字连字符），但换成 frontmatter 里的 name: skill:otherskill  
，归一化阶段就把名字换成了带冒号的版本，绕过校验覆盖 otherskill  
。  
两条路，落点都是**受害者的 SKILL.md 被替换**  
。  
> 💡 这里有个细节值得单独拎出来：**Prompt 的 version 校验其实是对的**  
——发布/草稿接口强制 x.y.z  
。冒号版本能进来，靠的是"迁移"那条默认开启、且**不做版本校验**  
的旁路。所以这不是"没写校验"，是"校验了主门，没管侧门"。  
  
## 四、为什么一个 9.9，一个 9.1  
### VE-0023 的 9.9  
  
```
AV:N / AC:L / PR:L / UI:N / S:C / C:H / I:H / A:H  →  9.9

```  
  
  
三个 H 全占：能读 public 全部配置明文（含解密后内容）、能覆盖、能批量删。加上 S:C  
（越过了命名空间隔离边界）。报告里还特意做了个稳健性说明：**即便把可用性 A 从 H 降到 L，还是 9.9**  
——因为机密性和完整性两个 H 已经够了。  
### VE-0737 的 9.1  
  
```
AV:N / AC:L / PR:L / UI:N / S:C / C:L / I:H / A:L  →  9.1

```  
  
  
机密性和可用性只给了 L，原因很克制：**跨资源读取的"越权读"这次没有真正打破可见性**  
——普通用户正常读别人的 Prompt 会被拦，别名读只造成了"两个自己的资源串味"，所以 C 取 L；删除也只影响单个目标资源，服务整体还活着，A 取 L。  
### 差在哪  
  
9.9 和 9.1 差的不是"一个更危险"，而是**影响面**  
：0023 打的是 public 的**全量**  
配置，0737 是**单个资源**  
的篡改和删除。方向都对，剂量不同。  
> 📐 两份报告都写着 is_command_exec = false  
，而且都解释清楚了为什么：一个写的是配置库记录、一个写的是固定 group 的 JSON，都不是"路径可控 + 内容可控的任意文件写"。 二阶风险照旧存在——被篡改的配置和   
SKILL.md  
 最后是被下游客户端拿去消费的。**把"我没执行"和"它会被执行"分开写，比标一个可 RCE 诚实。**  
  
## 五、修复：让标识符可靠  
  
<table><thead><tr><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(234, 88, 12, 0.07);"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">别名错配</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">鉴权和业务</span></span><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">用同一个解析方法、同一个字段</span></strong><span style="display: inline;"><span leaf="">；v3 接口对 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">tenant</span></code><span style="display: inline;"><span leaf=""> 直接 400，别留歧义</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">鉴权-执行不一致</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">Service 层用最终生效的 namespaceId 再校验一次（defense in depth），不一致就拒</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">分隔符拼接</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><strong style="font-size: inherit;color: #ea580c;font-weight: bold;"><span leaf="">别用可能出现在字段里的字符当分隔符</span></strong><span style="display: inline;"><span leaf="">。要么结构化传 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">KeyParts</span></code><span style="display: inline;"><span leaf="">，要么长度前缀 / 整体 JSON 或 Base64URL 编码</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">version 旁路</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">迁移得到的 version 同样强制 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">x.y.z</span></code><span style="display: inline;"><span leaf="">，冲突时告警跳过，而不是静默覆盖</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">别名键读写删</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">读回/删除前核对存储内容里的 promptKey/version 是否等于请求，把 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #ea580c;background: rgba(234, 88, 12, 0.07);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">checkWritableResource</span></code><span style="display: inline;"><span leaf=""> 扩到新建分支</span></span></td></tr><tr><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">审计错位</span></span></td><td style="border: 1px solid #f0dcc8;padding: 0.4em 0.8em;word-break: keep-all;color: #3b3028;"><span style="display: inline;"><span leaf="">记账用解析后的真实对象，别用请求里自报的身份</span></span></td></tr></tbody></table>  
> 💡 一句可以套用的判据，跟前几天一脉相承：**凡是用字符串表达"结构"，先问两件事——分隔符会不会出现在内容里？同一个东西会不会有两个名字？**  
两问只要有一个是"是"，这段代码就埋了雷。  
  
## 写在最后  
  
第四天了，回头看看这四天其实是一条很清楚的线：  
  
**第一天**  
：把几段拼成路径，拼完没人确认它还在不在原地。  
  
**第二天**  
：两个字段各管一段，中间没人问是不是同一个。  
  
**第三天**  
：把两个值拼成一个 key，而 key 从数学上不唯一。  
  
**第四天**  
：一个东西有俩名字、一个分隔符没转义——**标识符本身的不可靠**  
。  
  
四天的共同点，一句话：**系统里所有安全判断，最终都压在一个"字符串"上；而字符串，恰恰是最不可信的东西。**  
  
该是名字的地方，别给它两个名字；该是分隔符的地方，别用内容里能出现的字符；该是结构的地方，别压平成字符串。  
  
下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
：MCP 与 A2A，名字和版本拼成的 key 不唯一。**第四天**  
（本篇）：配置接口的 tenant/namespaceId 别名错配（9.9），Prompt/Skill 存储的冒号分隔符碰撞（9.1）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
