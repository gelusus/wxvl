#  【一天一个nacos漏洞·第五天】【严重】鉴权配置缺陷  
原创 YNsec
                        YNsec  YNsec安全实验室   2026-11-05 07:23  
  
   
  
# 【一天一个nacos漏洞·第五天】【严重】一道门认错了钥匙，一道门压根没装  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
今天这两个，都是 9.6 分。  
注：VE-XXXX是零隐网络安全实验室的内部漏洞编号  
  
一个在**配置接口**  
：鉴权那道门认的是 URL 里的 dataId/groupName  
，可真正要操作的对象藏在 ids  
 参数和 JSON 请求体里——**门认错了钥匙**  
。  
  
一个在 **AI 资源**  
：Prompt、Skill、AgentSpec 明明有 PRIVATE 可见性，可正文被存成了普通 Config，而 Config 通道根本不认可见性——**正门有锁，侧门没装**  
。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址、账号与口令均未写入本文。内容仅用于安全学习与防御加固。  
  
## 一、先说结论：两道门的两种坏法  
  
<table><thead><tr><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">VE-0489 · 配置接口</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">VE-0739 · AI 资源</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">9.6</span></strong></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">9.6</span></strong></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">鉴权认 URL 参数，操作对象在 ids/body 里</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">AI 资源存成 Config，Config 通道不认可见性</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">门的问题</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">认错了钥匙</span></strong></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">侧门没装锁</span></strong></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">最低权限</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">对任意一个配置有读/写权限</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">只要有 Config 的读权限（只读账号即可）</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
两条线，一个是"门在，但看错了东西"，一个是"有锁，但只在其中一扇门上"。  
## 二、VE-0489：门认的是 URL，东西却在别处  
### 2.1 鉴权资源，是从 URL 参数拼出来的  
  
Nacos 的鉴权过滤器在 Controller 执行之前，会用 ConfigHttpResourceParser  
 从**原始 HTTP 请求参数**  
里取 namespaceId / groupName / dataId  
，拼成一个资源串做权限判定：  
  
```
protected String getResourceName(HttpServletRequest request) {
    String dataId = request.getParameter("dataId");   // 只认 URL 参数
    return StringUtils.isBlank(dataId) ? StringUtils.EMPTY : dataId;
}

```  
  
  
问题在于：**不是每个接口要操作的对象都来自这三个 URL 参数。**  
 有些接口，真正的目标是 ids  
 参数，或者是 JSON 请求体。  
### 2.2 三个接口，各自藏着第二来源  
  
```
export / export2   —— 鉴权用 dataId/groupName，实际导出的是 ids 指定的配置
clone              —— 鉴权用 URL 的 dataId，实际写入目标是请求体的 targetDataId/targetGroupName
batchDelete/batch  —— 鉴权用 URL 的 dataId，实际删除的是 ids 指定的配置

```  
  
  
关键的一行在持久层：ids  
 非空的时候，**SQL 里只拼 WHERE id IN (ids)，dataId/group 被直接丢掉**  
。也就是说，你 URL 里填的 dataId  
 只对鉴权有意义，对"实际导出/删除哪条记录"毫无影响。  
### 2.3 结果：鉴权过的，和操作的，是两条记录  
  
```
请求  GET /v3/console/cs/config/export2
      ?namespaceId=public&groupName=group1&dataId=config_attacker&ids=4
        │                                ↑ 鉴权看这个（我自己的，有读权限）
        │                                └── 实际导出 ids=4（受害者的配置）
        │
        ├─ 鉴权：dataId=config_attacker → 有权限 → 通过 ✅
        └─ 业务：ids=4 → 导出受害者 protected_secret ❌

```  
  
  
**门检查的是"config_attacker"，放行的却是"protected_secret"。**  
### 2.4 决定性的对照  
  
<table><thead><tr><th data-colwidth="277" style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">请求</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">结果</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">说明</span></span></th></tr></thead><tbody><tr><td data-colwidth="277" style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">dataId=config_attacker&amp;ids=4</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">200</span></strong><span style="display: inline;"><span leaf="">，导出受害者内容</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">越权成功</span></span></td></tr><tr><td data-colwidth="277" style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">dataId=protected_secret&amp;ids=4</span></code></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">403</span></strong></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">把 dataId 换成真实受害者名，反而被拒</span></span></td></tr></tbody></table>  
  
第二行最有杀伤力：**同一个 ids，只是把 URL 里的 dataId 从"自己的"换成"受害者的"，就从成功变成 403**  
。这说明鉴权完全被那个 dataId 参数牵着走，跟 ids 指向的真实对象没关系。  
### 2.5 读、写、删全都通，而且 ids 能枚举  
- **读**  
：export2  
 把同命名空间任意配置导成 ZIP，含解密后的明文；  
- **写**  
：clone  
 用请求体的 targetDataId/targetGroupName  
 覆盖或新建任意配置；  
- **删**  
：batchDelete  
 伪造一个 dataId  
 参数骗过鉴权，实际删掉 ids  
 指定的配置。  
而且 ids  
 是自增主键，可以从 1 开始一个个试（不存在的 id 被忽略）——**你甚至不需要提前知道目标 id 列表**  
。  
> 🔍 特别要强调删这条：报告还做了个负向对照——**同样的 ids，不带伪造的 dataId/groupName 就是 403**  
。这个对照把根因钉死了：绕过点就是那个"鉴权会信、业务却不用"的 URL 参数。  
  
## 三、VE-0739：正门有锁，侧门没装  
### 3.1 AI 资源，其实存成了普通 Config  
  
Nacos 3.x 的 Prompt、Skill、AgentSpec 都有可见性（PUBLIC / PRIVATE，默认 PRIVATE）。走 AI 接口（/v3/console/ai/*  
）的时候，会做可见性校验——只读账号想看别人的 PRIVATE 资源，返回 404。  
  
但**这些资源的正文，底层是存成普通 Nacos Config 的**  
：  
  
```
prompt 版本存储    group=prompt__enc.〈hex〉__enc.〈hex〉   dataId=content.json
prompt legacy 镜像 group=nacos-ai-prompt                dataId=〈key〉.json
skill 版本存储    group=skill_enc.〈hex〉__enc.〈hex〉      dataId=SKILL.md
skill manifest    group=skill_〈name〉                    dataId=skill_index.json
agentspec 版本    group=agentspec__enc.〈hex〉__enc.〈hex〉 dataId=manifest.json

```  
  
  
而**存储键里只有 namespaceId / resourceType / name / version / filePath**  
——没有 owner，没有 scope，没有任何可见性信息。  
### 3.2 于是有了两套互不相认的权限体系  
  
```
AI 接口     → 校验 AI visibility（owner / scope / @@visibility 权限）   → 有锁 ✅
Config 接口 → 只校验 SignType.CONFIG 的 READ/WRITE                     → 不认 visibility ❌

```  
  
  
关键就在这句：**Config 模块里没有任何一处调用 VisibilityService。**  
 AI 的可见性，和 Config 的 ACL，是两套彼此独立的授权域。同一份数据，走 AI 门被拦，走 Config 门畅通无阻。  
### 3.3 决定性的对照：同一份数据，两扇门两种结果  
  
```
只读账号 viewer 去读别人的 PRIVATE prompt：

  GET /v3/console/ai/prompt/version?promptKey=ve0739p1...
  → 404 {"code":20004,"resource not found"}          ← 正门：被拦

  GET /v3/console/cs/config?dataId=ve0739p1.json&groupName=nacos-ai-prompt&namespaceId=public
  → 200，返回 PRIVATE prompt 的 template 全文          ← 侧门：畅通

```  
  
  
**同一个资源，AI 接口 404，Config 接口 200。**  
 这不是权限没写，是"权限只写在了其中一扇门上"。  
### 3.4 不只能读，还能改  
  
读只是机密性。更狠的是写通道：  
  
```
devuser 想改管理员的 PRIVATE prompt：
  POST /v3/console/ai/prompt/...  → 403  No permission   ← AI 侧防护有效

  POST /v3/console/cs/config
    dataId=content.json&groupName=prompt__enc.〈hex〉__enc.〈hex〉&content={篡改内容}
  → 200，覆盖版本存储                                        ← Config 侧放行

约 3 秒缓存刷新后，管理员的 AI API 返回的，已经是攻击者篡改的内容。

```  
  
  
**正门拦得住，侧门拦不住。**  
 攻击者甚至不用懂 AI 接口，会用普通 Config API 就够了。  
> 💡 有个细节值得单独说：这份报告对原始发现的"MCP 跨 namespace 泄露"子结论做了复核，结论是**未证实**  
——空 namespace 会被归一到 public，未知 namespace 返回 403，没有跨 ns 分支。**诚实地区分"证实了"和"没证实"，比把每个子项都往严重里凑更值钱。**  
  
## 四、为什么两个都是 9.6  
### VE-0489  
  
```
AV:N / AC:L / PR:L / UI:N / S:C / C:H / I:H / A:N  →  9.6

```  
  
  
机密性 H（单配置读权限能导出整个命名空间的明文配置）、完整性 H（能覆盖、新建、删除任意配置），可用性给 N（不直接让 Nacos 宕机，删配置对下游的影响算间接）。  
### VE-0739  
  
```
AV:N / AC:L / PR:L / UI:N / S:C / C:H / I:H / A:N  →  9.6

```  
  
  
一模一样：机密性 H（PRIVATE 资源正文全泄露）、完整性 H（能覆盖他人版本存储）、可用性 N。  
### 共同的 S:C  
  
两份都把"作用域"判成 Changed，理由一致：**攻击者的授权域（CONFIG ACL / 单一配置）和实际被影响的数据（AI visibility 保护的他人资源 / 同命名空间其他配置）不在同一个安全边界里。**  
> 📐 两份都做了稳健性说明：如果按"整个 Nacos 进程算同一安全域"（S:U），同样的实测结果是 **8.1 High**  
。**9.6 和 8.1 差的不是危害，是"边界画在哪"。**  
 这个口径在评审里永远会被吵，先讲清楚比争分数有用。  
  
## 五、修复：把门装对地方  
  
<table><thead><tr><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;font-weight: bold;background: rgba(37, 99, 235, 0.06);"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">鉴权认错来源</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">别用原始请求参数构造鉴权资源；Controller/Service 解析出</span></span><strong style="font-size: inherit;color: #2563eb;font-weight: bold;"><span leaf="">实际要操作的每个对象</span></strong><span style="display: inline;"><span leaf="">后，对目标逐一做权限校验</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">ids/body 绕过</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">export 对 ids 查出的每条配置、clone 对 body 的每个 target、batchDelete 对 ids 逐条做 WRITE/READ 校验</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">未声明参数被信</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">ConfigHttpResourceParser</span></code><span style="display: inline;"><span leaf=""> 只读 handler 明确声明的参数，或改由 Controller 显式传 Resource</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">两套 ACL 分叉</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">Config 与 AI 资源共享同一 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">VisibilityService</span></code><span style="display: inline;"><span leaf="">/资源标识，别让两套授权语义各管各的</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">AI 内部键裸露</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">存储键引入不可预测随机段，或对 name/version 做带密钥的 HMAC，别让公开名字能直接推导物理键</span></span></td></tr><tr><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">Config API 摸到 AI 域</span></span></td><td style="border: 1px solid #d0daf0;padding: 0.4em 0.8em;word-break: keep-all;color: #1a1a2e;"><span style="display: inline;"><span leaf="">对 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">nacos-ai-prompt</span></code><span style="display: inline;"><span leaf="">、</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">prompt__*</span></code><span style="display: inline;"><span leaf="">、</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">skill_*</span></code><span style="display: inline;"><span leaf="">、</span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #2563eb;background: rgba(37, 99, 235, 0.06);padding: 3px 5px;border-radius: 4px;white-space: nowrap;"><span leaf="">agentspec__*</span></code><span style="display: inline;"><span leaf=""> 等内部 group 的普通 Config CRUD/list/export 增加拦截</span></span></td></tr></tbody></table>  
> 💡 一句话收束这一篇：**鉴权这道门，一要认对"到底要动谁"，二要在"每一条能通到数据的路上"都装。**  
认错对象、或者只装一扇门，权限体系再完整，也只是个摆设。  
  
## 写在最后  
  
第五天了。回头把这几天排一排，其实都在讲同一件事的五个切面：  
  
**第一天**  
：路径拼完没人确认它还在原地。  
  
**第二天**  
：两个字段各管一段，中间没人对账。  
  
**第三天**  
：拼出来的 key 从数学上就不唯一。  
  
**第四天**  
：一个东西俩名字、一个分隔符没转义。  
  
**第五天**  
：门认错了钥匙，或者只在正门上了锁。  
  
五天的共同点，最后落到一句很朴素的话：**安全不是一个"有没有写校验"的问题，而是一个"校验是不是作用在你真正要保护的那个东西上"的问题。**  
  
认错对象、拼错名字、漏掉侧门——这三种错，代码里一个 if  
 都看不出来，得靠想清楚"这个请求最终会碰到什么"。  
  
下一天见。  
> 📚 **系列回顾 · 【一天一个nacos漏洞】**  
> **第一天**  
：客户端快照路径穿越，一路退到 /etc/cron.d  
。**第二天**  
：MCP 与 Skill，鉴权看 A、干活看 B。**第三天**  
：MCP 与 A2A，名字和版本拼成的 key 不唯一。**第四天**  
：配置接口的别名错配（9.9），Prompt/Skill 的冒号分隔符碰撞（9.1）。**第五天**  
（本篇）：配置接口认错参数（9.6），AI 资源的可见性被 Config 通道绕过（9.6）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
