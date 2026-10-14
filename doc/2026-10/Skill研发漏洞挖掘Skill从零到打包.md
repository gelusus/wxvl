#  Skill研发:漏洞挖掘Skill从零到打包  
原创 Z0安全
                        Z0安全  Z0安全   2026-10-14 07:09  
  
   
  
## 一、为什么要写skill  
  
用AI审代码这件事,我烦躁了挺久。每次开新会话,都得重新教一遍:先看入口点,再追污点传播,反序列化和反射调用优先,PoC最小化验证,别乱发挥。几百字的prompt,窗口一关全没了。AI知道什么是SQL注入,但它不知道我的习惯。我惯用的检索套路、我踩过的坑、哪些洞值得提交哪些不值,这些东西没地方放。  
  
后来发现Skill就是干这个的。说白了,把你沉淀的流程写成一个目录,里面有份机器可读的文档加几个脚本,AI在合适的时机自己加载。你不用每次重新教,它像是个看过你笔记的同事。  
  
比塞prompt强的地方在加载机制。Skill分三层:元数据(name加description)常驻上下文,就百来个词;SKILL.md正文,触发时才加载;脚本和参考文档按需取用。脚本还有个特性,可以不读进上下文直接执行。我写了个三百行的侦察脚本,AI不需要读懂它,跑就完了,回来的只有输出。token效率完全不是一个量级。  
  
对搞安全的来说还有一层好处:写出来的Skill本身能分发。团队人人装上,等于全组用同一套挖掘标准。做成zip就是作品。  
  
下面记录我把漏洞挖掘流程做成Skill的完整过程,踩了7个坑,都写进去了。  
## 二、Skill长什么样  
  
一个Skill就是一个目录:  
```
vuln-hunter/├── SKILL.md          # 必须,元数据+指令正文├── scripts/          # 可选,可执行脚本├── references/       # 可选,按需加载的参考文档└── assets/           # 可选,输出用的资源
```  
  
SKILL.md开头是YAML frontmatter。我第一版在这里踩了两个坑,先看修好的样子:  
```
---name: vuln-hunterdescription: >  开源第三方组件漏洞挖掘工具。系统化执行目标侦察、攻击面分析、  代码审计、PoC验证全流程。适用于挖掘开源组件漏洞、代码审计找洞、  漏洞赏金、安全研究场景。触发词:挖掘漏洞、代码审计、找0day、  漏洞挖掘、PoC开发、漏洞赏金。agent_created: true---
```  
  
坑一在这里。我最早把description写成"强大的漏洞挖掘工具,支持代码审计和PoC开发"。听着挺唬人,但AI根本不知道什么时候该用它,因为这句话没回答用户在说什么话的时候用它。结果就是明明在挖洞,Skill死活不触发。后来改成场景和触发词都写进去,命中率才正常。判断标准很土:把description念给同事听,他说不出什么情况下需要它,就重写。  
  
坑二是agent_created: true这个字段我漏了。它不影响触发和运行,平时看不出来,但管理工具会因此识别不了这个Skill,改删都做不了。本地自用无所谓,要分发或长期维护,记得带上。这个坑隐蔽,翻了好久文档才定位到。  
  
name是目录名,小写连字符,别用中文和空格。存放位置两种选择:放~/.workbuddy/skills/  
(Windows下是C:\Users\<你>\skills\  
)全项目可用,放.workbuddy/skills/  
随仓库共享。个人工具用前者,团队规范用后者。  
## 三、动手:先想场景再定形态  
  
写之前我先列了几句自己真实会说的话:"帮我看看这个GitHub项目有没有可挖的洞"、"审计一下这个反序列化库"、"给这个PoC做个最小化验证"、"这个漏洞值得提交吗"。四句话,对应选靶、审计、验证、提交决策,加上中间必然要做的攻击面分析,五个阶段的工作流就出来了。这步别省,工作流覆盖哪几段全靠这些例句定。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibuSfucPryHXGicTB4xFbVnNL7OkDvz7BhvfzXqHfNdcDyx48oAOLJK2T9K637G82sicONxH8IbxItLsWpjA7OK2P1WicMlEMSZQjytnMJ1QCX0/640?wx_fmt=png&from=appmsg "")  
  
  
然后判断什么内容做成什么形态。我的判断方式土但有效:这段代码是不是每次都重写?是就进scripts。这份资料是不是查了又查?是就进references。这个文件是不是用在产出里?是就进assets。都不满足的,留在SKILL.md正文当流程指令。  
  
对着vuln-hunter过一遍:仓库侦察每次都要拼GitHub API命令,进脚本。Java和Python的常见漏洞模式,SINK点清单、修复绕过技巧,查了又查的资料,做参考文档。PoC报告模板进assets。  
  
坑三也出在这步的初版。我最早把侦察流程写成正文里的文字指令,让AI"先查star数,再查issue数",一步一步描述。AI每次执行都重新拼命令,偶尔还拼错。步骤固定、输入输出明确的东西,必须固化成脚本,AI直接调用,确定性立刻拉满。教训就一句:告诉AI跑什么,别告诉AI怎么拼命令。  
  
初始化用官方skill-creator自带的脚本:  
```
scripts/init_skill.py vuln-hunter
```  
  
脚本在skill-creator的安装目录里(WorkBuddy内置插件,路径形如resources/plugins/workbuddy-builtin/skills/skill-creator/scripts/  
)。调不到就手动建目录,frontmatter模板上面已经给全了,问题不大。不指定路径默认生成到用户级目录,自带三个资源目录的示例文件,生成后删掉不要的就行。  
## 四、SKILL.md正文,给完整可抄的版本  
```
---name: vuln-hunterdescription: >  开源第三方组件漏洞挖掘工具。系统化执行目标侦察、攻击面分析、  代码审计、PoC验证全流程。适用于挖掘开源组件漏洞、代码审计找洞、  漏洞赏金、安全研究场景。触发词:挖掘漏洞、代码审计、找0day、  漏洞挖掘、PoC开发、漏洞赏金。agent_created: true---# Vuln Hunter — 开源组件漏洞挖掘系统化挖掘开源第三方组件漏洞,产出可验证的PoC。## 触发条件- 挖掘开源组件漏洞,或对第三方库做代码审计- 开发/最小化验证漏洞PoC- 漏洞赏金与安全研究## 工作流```目标侦察 → 攻击面分析 → 代码审计 → PoC验证 → 提交决策```### Phase 1: 目标侦察执行选靶评估时,运行 scripts/target_recon.py 获取仓库活跃度、CVE历史与依赖面。选靶标准:| 维度 | 高价值信号 ||---|---|| 知名度 | Stars > 1000 但安全维护不足 || 攻击面 | 处理不可信输入:解析器/反序列化/协议库 || 历史活跃 | 有CVE历史,说明攻击面真实存在 || 影响面 | 依赖该组件的下游项目多 |### Phase 2: 攻击面分析按 references/vuln-patterns.md 的SINK清单圈定重点文件,优先处理反射调用、反序列化入口、动态加载点。### Phase 3: 代码审计从入口点做污点传播追踪,可疑点直接对照references/vuln-patterns.md 中的对应模式编号。### Phase 4: PoC验证验证脚本一律在隔离环境执行,最小化构造:只证明漏洞存在,不做额外利用。产出写入PoC报告模板。### Phase 5: 提交决策按 CVSS 评级与下游影响面判断是否值得申请CVE,参照 references/vuln-patterns.md 附录的提交材料清单。
```  
  
几个设计点说一下。工作流用流程图加分阶段表格组织,相当于给AI的执行控制器。每个Phase都指向具体资源,脚本跑哪个、参考查哪份,写死,不留"酌情处理"的空间。Phase 4里"隔离环境、最小化构造"这条是刻意写死的,后面坑七会讲到为什么。  
  
坑四是嵌套代码块,这个坑最阴。注意上面外层围栏用的是四个反引号。我第一版用三个,而SKILL.md内部的工作流图自带三个反引号,发布到博客后渲染直接碎掉,示例代码从中间断开,后半截全裸奔。Markdown的围栏是从外往里配对的,内层有三反引号,外层必须升级成四反引号。写带代码示例的技术文档,这个细节翻车率极高。  
  
坑五是正文句式。早期我写的是"你应该先跑侦察脚本",AI执行起来偶尔会自由发挥。改成祈使句,"执行目标侦察时先运行脚本",指令感强多了。description用第三人称,正文用祈使句,这是官方指南的原话,照做就行。  
  
还有个体会:SKILL.md别贪长。我第一版把漏洞模式细节全塞进正文,万字长文,AI每次触发都整篇加载,又慢又贵。后来正文只留流程骨架,细节全下沉到references,控制在五千词以内。信息只活一份,别两边重复。  
## 五、脚本和参考文档  
  
侦察脚本要的是确定性,这是我用的版本,放scripts/target_recon.py  
:  
```
#!/usr/bin/env python3"""仓库选靶侦察:活跃度 + CVE历史线索"""import json, sys, urllib.requestdefgh(path):    req = urllib.request.Request(        f"https://api.github.com{path}",        headers={"Accept": "application/vnd.github+json"})    with urllib.request.urlopen(req, timeout=15) as r:        return json.load(r)defmain(repo):    info = gh(f"/repos/{repo}")    print(f"Stars:        {info['stargazers_count']}")    print(f"Open issues:  {info['open_issues_count']}")    print(f"Last push:    {info['pushed_at']}")    lang = gh(f"/repos/{repo}/languages")    print(f"Languages:    {', '.join(f'{k}{v}'for k, v inlist(lang.items())[:3])}")    print(f"\nCVE/漏洞历史查询: https://osv.dev/list?q={repo.split('/')[-1]}")if __name__ == "__main__":    iflen(sys.argv) != 2or"/"notin sys.argv[1]:        sys.exit("usage: target_recon.py owner/repo")    try:        main(sys.argv[1])    except urllib.error.HTTPError as e:        sys.exit(f"GitHub API {e.code}: 检查仓库名是否为 owner/repo 格式")
```  
  
坑六在这脚本末尾。最初我写的是exit("usage: ...")  
塞在三元表达式里,能跑,但exit是site模块注入的,不是规范用法。更要命的是没有错误处理,有次把仓库名拼错,404的traceback直接糊了一屏。给脚本写报错提示这事,读者第一次跑就撞上,体验差太多。现在这版参数校验和HTTPError都有,拼错会得到一句人话。  
  
references/vuln-patterns.md  
按模式编号组织,方便AI对照引用:  
```
# 漏洞模式速查表## VP-01 反序列化RCE- SINK: ObjectInputStream.readObject / pickle.loads / yaml.load- 审计入口: 所有接收外部字节的解码函数- 常见修复绕过: 修复方上黑名单时,换用名单外的gadget链## VP-02 反射调用注入- SINK: Class.forName(用户可控字符串).getMethod(...).invoke(...)- 审计入口: 参数拼接进反射调用名的代码路径## VP-03 路径穿越- SINK: new File(用户可控路径) / open()未规范化- ...
```  
  
SKILL.md里只放"什么时候查模式表",模式表本身放references。这份文件超过一万词的话,还要在SKILL.md里给检索模式,比如按VP编号查阅,让AI精准定位而不是全文通读。  
## 六、打包  
  
写完用官方打包脚本做校验和分发:  
```
scripts/package_skill.py ~/.workbuddy/skills/vuln-hunter
```  
  
先验证后打包,验证会查frontmatter格式、目录命名、description质量、资源引用关系,任何一项不过就拒绝打包并报具体错误,改完重跑。过了会产出vuln-hunter.zip  
,目录结构原样保留,发给别人就是完整交付。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibuSfucPryHXDT3REDQbZjCjdacY4VqavLHpEYeibMEr3H1tib6YSzNrLO8JIG2RDf6F3FsJcgpefl6Xbq8jgOU4ydDa9v1sElsLMXAycF1XnU/640?wx_fmt=png&from=appmsg "")  
  
## 七、最后一个坑,也是最重要的一个  
  
坑七:PoC验证纪律必须明写。这条是安全圈特有的。AI默认会热心帮你把PoC"做得完整",加上利用链、批量验证,一套全活。你不明写"隔离环境、最小化构造、只证存在性",它真能给你生成一个能批量打目标的脚本。这不是开玩笑,我亲眼见过它自作主张往PoC里加回连逻辑。Phase 4里那句写死的纪律,就是防这个。  
  
Skill写完不等于结束。真拿它跑几单挖掘任务,短板自己会浮出来:AI在哪个Phase开始自由发挥,说明那里指令不够硬。哪份参考文档被反复整篇加载,说明要拆分加索引。哪段命令每次重新拼,说明该升级成脚本。改完再跑,三轮下来,好不好用自己心里有数。我的验收标准就一条:跑一单完整挖掘,AI全程没偏离我预设的流程一次。做到了,经验才算真固化进去。  
  
  
点个关注吧~  
  
   
  
  
  
