#  公开漏洞狩猎提示词包:14 阶段 recon 到自去重,deepseek MVP,MIT 可下载  
原创 Violet Walker
                    Violet Walker  黑白之道   2026-10-20 01:08  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6N6fqqPt6JTTaaX6XaUGUaXu8QibcTquUBdn0eWic7AiaVTib47hBAFPgHU7BrCN50bjsVeCIPFnpia8TwiaL7jQj1L5yuJIOcMxibhOE/640?from=appmsg "")  
> **导语**  
:漏洞猎人 mdp_sec 公开 14 阶段私有狩猎工作流提示词包,从 recon 到自去重全覆盖,deepseek 作 MVP 模型,MIT 协议可下载。  
  
## 一、推文与仓库事件  
  
10 月中旬,Marius du Preez(@mdp_sec)在 X 上发了一条面向所有私信的统一回复:  
- 翻他之前的推文(自动化狩猎要的东西都在里头);  
  
- 卡住了请直接问具体问题;  
  
- MVP 模型用 deepseek;  
  
- 新人 starter workflow 给到 mdpsec/bug-bounty-hunting-prompts  
。  
  
同一条推文还配了两张图:一张是他自己的赏金平台 dashboard,页面顶部显眼的 **"payments paused"**  
 标签,下方列出 $1,012,925 / $882,125 / $0 / $130,800  
 等历史结算金额——这是他愿意把工作流开源出来的现实背景;另一张是 Burp Suite 的实拍请求/响应,左侧是 GET //////////////////../../../../../../etc/passwd  
,右侧响应里 root:x:0:0:root:/root:/bin/bash  
 等行直接读出——证明这套工作流在真实目标上拿下过路径穿越类漏洞。  
  
![推文配图 1:mdp_sec 自己的赏金 dashboard,payments paused 与历史结算金额](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6OCVe2ZjzSdDjFEWjsla3kr6aibEympOrR6bf3jAyEHNlKxK1icicLkFIFLSThXTiaicQ8TibtgnGypBBzvCOUTGXnNgqdtBpGL47rBc/640?from=appmsg "推文配图 1:mdp_sec 自己的赏金 dashboard,payments paused 与历史结算金额")  
  
回复里另外有一条被同好 @SteelyDu 追问 "GLM 5.3 怎么样",他只回了 https://blackboxarena.ai  
 一个链接——这是个给模型做攻防对抗测评的网站,他自己对 GLM 5.3 的评价言简意赅:"自己去跑。"  
## 二、仓库与下载方式  
  
mdpsec/bug-bounty-hunting-prompts  
 仓库的元数据:  
- **作者**  
:mdpsec(Marius du Preez)  
  
- **协议**  
:MIT  
  
- **首发**  
:2026-09-30,最后一次 push 2026-10-03  
  
- **规模**  
:182 KB,12 个 repo topics(bug-bounty、llm-prompts、penetration-testing、web-security、responsible-disclosure 等)  
  
- **热度**  
:636 Star / 131 Fork  
  
- **下载**  
:GitHub 主分支(无 release tarball,直接 git clone  
 即可)  
  
下载命令:  
```
git clone https://github.com/mdpsec/bug-bounty-hunting-prompts.git
```  
  
需要强调的是:仓库里**只有提示词文件**  
,不是可以直接跑起来的应用。README 写得很直白——"这些提示词文件不是应用也不是调度引擎",并附 ADAPTATION.md  
 列出所有 YOUR_*  
 占位符的替换说明,例如 $YOUR_WORKSPACE_ROOT  
、$YOUR_HELPERS_ROOT  
、$YOUR_MITM_HOST  
 等。私有 helper、浏览器舰队、代理、邮件服务、dashboard、PoC 运行时、凭证、参考语料都**不包含**  
,用户必须用自己的环境去适配。  
## 三、14 阶段提示词序列  
  
仓库的 14 个阶段文件全部在 phases/  
 目录下,按漏洞狩猎的实际顺序排列:  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">阶段</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">文件</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">用途</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">0</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-00-recon.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">轻量目标表面积绘制</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-01-access.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">账号准备与认证访问</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-02-sweep.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">端点与漏洞类别的广扫</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">3</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-03-closeout.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">校验 sweep 交接并释放资源</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">4a</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-04a-scope.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">仅范围上下文</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">4b</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-04b-browser-walk.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">认证态浏览器巡查</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5a</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-05a-scope.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">关键狩猎前的范围上下文</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">5b</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-05b-critical.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">高影响漏洞狩猎</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">6</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-06-triage.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">初筛与去重</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">7</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-07-validate-escalate.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">验证与升级</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-08-verify-escalate.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">独立验证与升级</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8a</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-08a-independent-escalation.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">一轮只读预闸升级</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">9</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-09-triage-gate.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">独立线上分诊闸</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">10</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">phase-10-self-duplicate-check.md</span></code></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">同一项目内重复收敛</span></section></td></tr></tbody></table>  
最值得仔细读的细节有三点:  
1. **阶段 8a 是"独立验证"环节**  
:README 把它称作 independent-escalation  
——和阶段 8 的验证不是同一组上下文,目的是防止模型把"已验证漏洞"自我强化;  
  
1. **阶段 10 是 self-duplicate-check**  
:在最后一道闸上把同一项目内早前可能报过的相似发现收敛掉,避免重复打扰平台;  
  
1. **4b 是可选**  
:README 明确说"浏览器巡查阶段作为可选的浏览器专项阶段保留",不需要浏览器 fleet 的项目可以整段跳过。  
  
公开版本里**不包含**  
重复的"第二轮关键分支"和"第二轮独立升级分支",这部分逻辑被有意保留在私有版本中。  
  
![mdp_sec 推文配图 2:Burp Suite 实拍路径穿越 PoC,响应中直接读出 /etc/passwd 内容](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NtfeBUKHZGvsCjdDfO9ettHFzqvguUZuoyRzX7m3BmeIKvJiaSfJGJHoc5jCsJxf0sIuf9yvSNbOoRy54NTYpqpanQ8TlyaZKY/640?from=appmsg "mdp_sec 推文配图 2:Burp Suite 实拍路径穿越 PoC,响应中直接读出 /etc/passwd 内容")  
## 四、典型用法与模型选择  
  
mdp_sec 在回复里给的"入门三步"实际上对应这套提示词的标准用法:  
1. **挑阶段**  
:从 README 表里挑与目标工作流匹配的阶段;  
  
1. **替换变量**  
:把 ADAPTATION.md  
 列出的 $YOUR_*  
 与 {{...}}  
 替换为环境里的真实值;  
  
1. **逐阶段喂给模型**  
:一次只喂一个阶段,保留模型要求的 handoff 工件(报告、证据、原始抓包)。  
  
MVP 模型选择上他直接点名 **deepseek**  
,从回复语境看,这是他私信里对"我该用哪个模型"问题的一贯答案——可能与成本/中文支持/工具调用稳定性有关,但他没有展开说,推文评论区对 GLM 5.3 的追问他也没直接评价,只丢了个 https://blackboxarena.ai  
 让提问者自己跑测试。  
## 五、安全与披露  
  
SECURITY.md  
 写得很短,核心要求是"只对授权目标使用提示词,不要自动批量提交、不要采集用户数据、不要留下持久状态变更"。ADAPTATION.md  
 也再次提醒——不要把凭证、Cookie、Token、API Key 或私有基础设施细节粘进任何提示词文件。仓库不提供集成支持,README 明确写"我们不会重现私有环境或回答个别实施问题"。  
## 六、上手路径建议  
  
对于第一次接触这类"提示词流水线"的读者,推荐路径是:  
- 先读 README + ADAPTATION.md  
,把占位符全部映射到自己的环境;  
  
- 从阶段 0(recon)和阶段 2(sweep)开始跑,因为它们不需要认证态,门槛最低;  
  
- 跑通两个阶段后,再去读阶段 6、7、9、10 这一串"分诊/验证/去重"逻辑,这是整套提示词和普通 prompt 包差异最大的部分——也是 mdp_sec 私有方法论里**真正公开出来的部分**  
;  
  
- 把它当作"参考蓝图",而不是"开箱即用的工具"。README 自己写得很清楚:这是一份 reference prompt pack,会随不同代理、工具和项目规则而调整。  
  
仓库:https://github.com/mdpsec/bug-bounty-hunting-prompts  
(MIT,636 Star) 下载命令:git clone https://github.com/mdpsec/bug-bounty-hunting-prompts.git  
 作者推特:@mdp_sec  
  
**版权声明**  
:本文由华盟网原创发布,保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NvudVQffHc38pWvM4EpiaoyWD5DDy2FiazhE0Z9uEMp3CgPc2ZozSRtiaIwBWSV4AwjZuw7oDyjOVxUQWUYbkluOTicsXbtdBHuJ4/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
