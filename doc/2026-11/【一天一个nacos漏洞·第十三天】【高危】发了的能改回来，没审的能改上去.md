#  【一天一个nacos漏洞·第十三天】【高危】发了的能改回来，没审的能改上去  
原创 YNsec
                    YNsec  YNsec安全实验室   2026-11-17 04:10  
  
   
  
# 【一天一个nacos漏洞·第十三天】【高危】发了的能改回来，没审的能改上去  
> "  
> 安全研究 · 漏洞利用链拆解 · 靶场实录  
  
  
**摘要**  
  
软件交付里有个基本约定：**同一个版本号，永远是同一份内容。**  
  
1.0.0  
 发布出去之后，它就该是 1.0.0  
 那一份。哪怕有 bug，也是发 1.0.1  
，而不是回头改 1.0.0  
。  
  
今天这两个洞，把这个约定从两头都破了：  
  
一个能**把已发布的改回去**  
——online  
 变 draft  
，内容悄悄换掉。  
  
一个能**把没审核的推上去**  
——draft  
 直接 online  
，还进了客户端的默认清单。  
> **免责声明**  
：测试全部在授权靶场完成，主机地址与账号口令均未写入本文。内容仅用于安全学习与防御加固。  
  
  
01  
  
一、"版本"这两个字，值多少钱  
  
先想一个问题：为什么大家都这么在意版本号？  
  
因为整条供应链的信任，都挂在它上面。  
  
```
你的 pom.xml 里写着 foo:1.2.3
→ Maven 去仓库拉 1.2.3
→ 拉到的必须是"当初审计过的那个 1.2.3"

Docker 镜像用 digest 固定
→ 同一个 digest 必须永远是同一层文件

```  
  
  
**依赖锁定、SBOM、签名验签、漏洞库比对——全部建立在"版本号 → 内容"是固定映射这个前提上。**  
  
一旦这个映射可以变，上面所有机制一起失效：你锁了版本号，等于没锁。  
  
今天这两个洞，就是把这个映射从两边撬开。  
  
<table><thead><tr><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><section><span leaf=""><br/></span></section></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">VE-0064 · 往下改</span></span></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">VE-0042 · 往上改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">分数</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><strong style="font-weight: 700;color: #1a1a1a;"><span leaf="">7.1</span></strong></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><strong style="font-weight: 700;color: #1a1a1a;"><span leaf="">7.7</span></strong></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">一句话</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">已发布版本被改回 draft，内容被替换</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">draft 直接置 online，跳过审核</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">方向</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">online → draft</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">draft → online</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">最低权限</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">普通开发者（需资源写权限）</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">普通开发者（需资源写权限）</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">命令执行</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">否</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">否</span></span></td></tr></tbody></table>  
  
02  
  
二、VE-0064：发过的版本，还能被改回去  
### 2.1 一行 update，代替了一次冲突报错  
  
正常逻辑下，往数据库插一条"版本行"，如果这个版本已经存在，应该报冲突——因为**版本号是唯一键**  
。  
  
但代码是这么写的：  
  
```
public void insertVersionRow(..., String status, String version, ...) {
    AiResourceVersion existing = persistService.find(namespaceId, name, type, version);
    if (existing != null) {
        updateExistingVersionRow(...);      // ← 已存在？那就不报错，直接覆盖
        return;
    }
    try {
        persistService.insert(row);
    } catch (DuplicateKeyException e) {
        updateExistingVersionRow(...);      // ← 唯一键冲突？还是覆盖
    }
}

```  
  
  
**两条路都通向"覆盖"。**  
 版本号是唯一键，但唯一键冲突被当成了"更新指令"。  
  
而那个覆盖方法做的事是：  
  
```
private void updateExistingVersionRow(...) {
    persistService.updateStorageAndDesc(..., storageJson, description);   // 换内容
    persistService.updateStatus(..., status);                            // 换状态
}

```  
  
  
updateStatus  
 的 SQL 条件只有四个字段：  
  
```
UPDATE ai_resource_version SET status=?, gmt_modified=NOW()
WHERE namespace_id=? AND name=? AND type=? AND version=?

```  
  
  
**没有 AND status <> 'online'，没有任何状态前置条件。**  
 一个 online  
 版本，可以被直接改成 draft  
。  
### 2.2 三条绕过路径  
  
光有"覆盖"还不够——得先让系统认为"这个版本不存在"，才会走到插入逻辑。报告找到了三条路：  
  
**路径一：500 行窗口。**  
  
判断"版本是否存在"，用的是：  
  
```
persistService.list(namespaceId, name, type, null, 1, 500);   // 只取最近 500 行

```  
  
  
**只看最近 500 条。**  
 版本数超过 500 时，排在第 501 位之后的版本就"查不到"了——哪怕它明明还在库里、还挂着 online  
。  
  
而攻击者可以自己把版本堆到 501 个（用同一个授权接口），然后把老版本"看不见"地覆盖掉。  
  
**路径二：大小写。**  
  
```
库里已有：v1（online）
判断存在性：listExistingVersions().contains("V1")  → false   （Java 大小写敏感）
落库查询：find(version='V1')                        → 命中 v1 （MySQL utf8mb4_unicode_ci 不敏感）

```  
  
  
**判断说"不存在"，执行却打中了。**  
 一条请求，不需要任何前置条件。  
  
**路径三：孤儿行。**  
  
如果 ai_resource  
 的 meta 行丢了（异常残留、部分删除），代码走 meta == null  
 分支——**跳过存在性检查和属主校验**  
，直接用固定的 0.0.1  
 去写。而这个 0.0.1  
 如果还有遗留版本行，就被接管了。  
### 2.3 后果：状态和内容一起变，而且没人发现  
  
实测（管理员视角，自建资源）：  
  
```
覆盖前：0.0.1  status=online   content="VE0064 marker base"
请求：  POST /draft  targetVersion=0.0.1
→ 200 {"code":0,"data":"0.0.1"}

覆盖后：0.0.1  status=draft    content="VE0064-HIJACK-MARKER-1729"
meta：  labels={'latest':'0.0.502','stable':'0.0.1'}   ← 标签还指着这个版本
        onlineCnt=502                                   ← 计数没有回退

```  
  
  
**版本从线上消失了，但标签还指着它，计数还是 502。**  
  
更细的一处：不同客户端的反应不一样——  
  
```
Skill 客户端      不校验版本行状态 → 按 label=stable 照样返回 200，下发被替换的 SKILL.md
AgentSpec 客户端   校验 status==online → 返回 404 "version not online"

```  
  
  
**一个把恶意内容发出去了，一个把已发布制品搞成不可用。**  
 前者是投毒，后者是破坏。  
  
03  
  
三、VE-0042：没审核的版本，能被直接推上去  
### 3.1 正规发布：draft 会被拦住  
  
正常的发布接口 doPublish  
 有两道门：  
  
```
// 第一道：状态必须是对的
if (!REVIEWING.equals(v.getStatus()) && !REVIEWED.equals(v.getStatus()) && !ONLINE.equals(v.getStatus())) {
    throw ... "Only reviewing version can be published";
}
// 第二道：如果走过流水线，必须 APPROVED
if (execution.getStatus() != PipelineExecutionStatus.APPROVED) {
    throw ... "Pipeline not approved, cannot publish";
}

```  
  
  
实测也是拦住的：  
  
```
POST /publish (draft 版本)
→ 400 {"code":20002,"data":"Only reviewing version can be published: 1.0.0"}

```  
  
  
**这道门是真的。**  
### 3.2 但 /online 那道门，不看状态  
  
同一个模块里，还有个"上架/下架"开关：  
  
```
public AiResourceVersion toggleVersionOnlineStatus(..., String version, boolean online) {
    AiResourceVersion v = persistService.find(namespaceId, name, type, version);
    if (v == null) { throw ... NOT_FOUND ...; }

    String targetStatus = online ? ONLINE : OFFLINE;
    if (targetStatus.equalsIgnoreCase(v.getStatus())) {
        return null;                                   // ← 唯一的判断："是不是已经是目标状态"
    }
    persistService.updateStatus(..., targetStatus);     // 直接改
    info.setOnlineCnt(online ? cnt + 1 : Math.max(0, cnt - 1));
}

```  
  
  
**它只问"是不是已经 online 了"，不问"你凭什么 online"。**  
  
没有 reviewing/reviewed 前置、没有 pipeline 查询、没有 APPROVED 校验。  
  
于是：  
  
```
POST /draft   → 200，创建 draft 1.0.0
POST /publish → 400，拒绝（状态不对）
POST /online  → 200 ✓ 直接上线

```  
  
  
而且上线时还顺手写进了客户端清单：  
  
```
if (online) {
    List&lt;String&gt; files = parseStorageFiles(v.getStorage());
    if (files != null && !files.isEmpty()) {
        SkillIndexManifest manifest = manifestService.loadForUpdate(namespaceId, name);
        manifest.getVersions().put(version, files);    // ← 未审核版本进清单
        manifestService.write(namespaceId, name, manifest);
    }
}

```  
  
  
**客户端按 version=1.0.0 或 label=latest 都能拉到这份没审过的内容。**  
### 3.3 管理员专属的能力，用两个普通接口拼出来  
  
这个项目里其实有一个"跳过审核"的正当入口：  
  
```
@PostMapping("/force-publish")
@Secured(resource = ADMIN_PATH + "/force-publish", action = WRITE, ...)   // ← 绑定管理员专属资源

```  
  
  
普通用户调它：  
  
```
POST /force-publish (devuser)
→ 403 authorization failed

```  
  
  
**这个门也是好的。**  
  
但对照 /online  
 的注解：  
  
```
@PostMapping("/online")
@Secured(action = WRITE, signType = AI, apiType = CONSOLE_API)   // ← 只要求普通写权限

```  
  
  
**没有绑定专属资源。**  
  
所以：  
  
```
devuser 调 /force-publish       → 403（管理员专属）
devuser 调 /online + /publish   → 200 + 200（效果完全一样）

```  
  
  
**两个普通接口，拼出了管理员专属能力。**  
 这在权限模型里叫"等价路径未收敛"——你堵了一条路，旁边开了条一样的。  
### 3.4 一个很硬的证据：字节码  
  
报告做了一件我很欣赏的事——它去看了**实际跑起来的 jar**  
：  
> 从 /opt/nacos/target/nacos-server.jar  
 里提取 nacos-ai-3.2.4.jar  
，反汇编 AiResourceManager.class  
。toggleVersionOnlineStatus  
 的方法体里只有 "online"  
、"offline"  
 两个字符串和 updateStatus  
 调用，**没有任何 Pipeline / APPROVED / reviewing 分支**  
；而 doPublish  
 里有。  
  
  
**源码和部署产物一致，不是"我猜的"。**  
  
这一步很多报告会跳过——直接看 GitHub 源码就写结论。但如果用户跑的是改过的包，或者版本对不上，结论就是空的。**反编译部署产物，是唯一能证明"线上真的是这样"的办法。**  
  
04  
  
四、两个方向，一个结论  
  
```
VE-0064：online → draft（改回去），顺手换内容
VE-0042：draft → online（改上去），顺手进 latest 清单

```  
  
  
一上一下，正好把状态机的两个方向都占了。  
  
而状态机本来是什么？**它是那个"只能沿着既定路径走"的东西**  
：  
  
```
draft → reviewing → reviewed → online
         ↑ 审核在这里                 ↓
         └──────── offline ←──────────┘

```  
  
  
今天这两个洞，一个允许你**跳过中间所有格**  
直接落地，一个允许你**从终点倒着走回起点**  
。  
> 📌 状态机的价值不在"有个 status 字段"，而在"**每次状态转移都有前置条件**  
"。 少了前置条件，status  
 就只是个字符串，谁都能改。  
  
  
05  
  
五、边界  
  
**VE-0064**  
：需要普通开发者账号 **且对目标资源有写权限**  
（自己的资源，或管理员）。跨属主会被 403 拦住——所以它不是跨租户越权，是"**对自己能管的资源，做了超出预期的事**  
"。  
  
**VE-0042**  
：同样需要资源写权限。未登录和只读账号都会被拒。  
> 📐 两份都把这点写清楚了，没有夸大。但要看到：**"对自己有权限的资源"这个限定，在供应链场景下不成立。**  
因为上线的东西是要发给别人的——你的技能会被别的团队、别的 Agent 拉取。**“我能改我自己的包"和"我能改全世界拿到的那个包”，是一回事。**  
  
  
06  
  
六、修复  
  
<table><thead><tr><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">问题</span></span></th><th style="border: 1px solid #1a1a1a;padding: 8px 10px;word-break: keep-all;font-weight: 800;background: #F0FDF4;"><span style="display: inline;"><span leaf="">怎么改</span></span></th></tr></thead><tbody><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">版本行已存在就覆盖</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">find()</span></code><span style="display: inline;"><span leaf=""> 命中既有行时</span></span><strong style="font-weight: 700;color: #1a1a1a;"><span leaf="">抛版本冲突</span></strong><span style="display: inline;"><span leaf="">，不要静默 update；只有明确的&#34;覆盖上传&#34;语义才允许改</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">状态能被无条件改</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">updateStatus</span></code><span style="display: inline;"><span leaf=""> 加 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">AND status NOT IN (&#39;online&#39;,&#39;reviewing&#39;)</span></code><span style="display: inline;"><span leaf="">，或做期望状态 CAS</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">500 行窗口判断存在性</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">别用固定分页当唯一性依据；改为按 version 精确查（或直接依赖 DB 唯一键）</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">大小写不一致</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">Java 侧与 DB 侧统一比较规则（统一小写，或改 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">utf8mb4_bin</span></code><span style="display: inline;"><span leaf="">）</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">孤儿行被接管</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">meta == null</span></code><span style="display: inline;"><span leaf=""> 分支也要校验版本行是否存在、属主是谁</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">/online</span></code><span style="display: inline;"><span leaf=""> 无状态校验</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">online 方向只允许 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">reviewing/reviewed</span></code><span style="display: inline;"><span leaf="">（或 pipeline APPROVED）；与 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">doPublish</span></code><span style="display: inline;"><span leaf=""> 同一套规则</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">等价路径未收敛</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">/online</span></code><span style="display: inline;"><span leaf=""> 绑定管理员专属资源，或与 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">/force-publish</span></code><span style="display: inline;"><span leaf=""> 合并；admin 入口显式要求全局管理员</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">客户端不校验状态</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">Skill 客户端对齐 AgentSpec，只下发 </span></span><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">online</span></code><span style="display: inline;"><span leaf=""> 版本</span></span></td></tr><tr><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><span style="display: inline;"><span leaf="">检查与写入非原子</span></span></td><td style="border: 1px solid #D1D5DB;padding: 8px 10px;word-break: keep-all;color: #555555;"><code style="font-family: Menlo, Consolas, Monaco, monospace;font-size: 90%;color: #1F2937;background: #F3F4F6;padding: 2px 5px;border-radius: 2px;border: 1px solid #E5E7EB;white-space: nowrap;"><span leaf="">ai</span></code><span style="display: inline;"><span leaf=""> 服务层加事务或分布式锁，&#34;判断存在 + 写入&#34;必须在一个原子单元里</span></span></td></tr></tbody></table>  
> 💡 一句可以写进发布系统 checklist 的话：**发布过的版本，应该变成只读的。**  
想改？发新版本。这不是流程洁癖——**这是让"版本号"这三个字还值点钱。**  
  
  
07  
  
写在最后  
  
第十三天。今天这两个洞加起来，其实在说一件很朴素的事：  
  
**“发布"这个动作，如果没有把东西锁死，那它就不叫发布，叫"暂时放在那儿”。**  
  
而这件事的分量，取决于你把它放在什么位置：  
- 如果只是一个内部配置，改来改去问题不大；  
- 但如果它是**被别人依赖的制品**  
——技能包、镜像、依赖库、Agent 的指令集——那么"能改"就等于"**任何人都可以悄悄替换你正在用的东西**  
"。  
这也是为什么现代供应链安全花了那么多力气在"不可变"上：digest 固定、内容寻址、签名、透明日志。**所有这些机制，本质上都是在防同一件事——防止"同一个名字"底下换了一份东西。**  
  
今天这两个洞提醒的是：**不可变性不是加密算法给的，是那段 if (existing != null) throw 给的。**  
  
而它今天没写。  
  
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
：注册表身份由 ip:port 推导——可覆盖他人注册（7.1），可删除他人实例（7.1）。**第十二天**  
：集群内部基础设施——锁接口无鉴权（8.2），同步接口退让鉴权（7.3）。**第十三天**  
（本篇）：版本不可变性被破坏——已发布的能改回去（7.1），未审核的能推上去（7.7）。  
> 文中所有测试均在授权靶场完成，主机与凭据未写入。若在生产环境发现同类问题，请第一时间联系厂商并遵守披露约定。  
  
  
— 完 —  
  
如果这篇对你有帮助，欢迎在看 / 转发给做安全的朋友  
  
   
  
  
