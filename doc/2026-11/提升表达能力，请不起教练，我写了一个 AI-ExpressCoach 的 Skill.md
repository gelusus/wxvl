#  提升表达能力，请不起教练，我"写"了一个 AI-ExpressCoach 的 Skill  
原创 xiejava
                    xiejava  fullbug   2026-11-17 09:30  
  
很多人都有提升表达能力的需求，一套"AI 表达教练"的完整设计：解决什么问题、怎么做的、两个半月跑出了什么效果。  
  
过去两个多月，我每天干一件事：录 10 分钟音，交给 AI 点评。  
  
前面几篇讲的都是故事：同一份讲话稿录五遍、两个半月的数据总账。今天这篇讲**系统**  
——这套"AI 表达教练"是怎么被设计出来的，踩了哪些坑，效果怎么验证。  
  
想自己练表达、或者想把任何"自我训练"工程化的朋友，这篇可以直接抄作业。  
  
**练表达最难的不是坚持，是三件事：没有反馈、感觉不可靠、无法积累。**  
  
我这套系统把每天 10 分钟录音，做成了一个软件项目——机器管测量（stats.py  
 数汉字数／语速／口头禅／卡顿，永远不数错），AI 管判断（按五场景标尺双评打分），三份文件沉淀（进步账本只追加、规律库、标尺独立存放）。  
## 一、它解决什么问题  
  
练表达这件事，普通人有三个死穴：  
  
**死穴一：没有反馈。**  
 说完就过，下一秒就忘。你不知道自己口头禅多严重，不知道哪里绕了，不知道那个"嗯——“停了多久。没有反馈的练习，只是重复。  
  
**死穴二：感觉不可靠。**  
 我的数据里最扎心的一条：自我感觉"没提升"的那天，AI 打出了两个半月的历史最高分 4.5；感觉"越录越差"的那一版，硬指标全是五版里最好的。**感觉专门在你进步的时候骗你。**  
  
**死穴三：无法积累。**  
 今天发现的问题，明天换个场景又犯，每次都从零开始。三年前的你和今天的你，犯的是同一类错——因为从来没有一个账本。  
  
那请个人类教练呢？我被两个现实劝退：贵还在其次，更要命的是**他不可能每天等我 10 分钟，等两个半月**  
。  
## 二、那直接问 AI 不就行了？不行  
  
你可能想说：现在 AI 那么强，把录音转写扔给它点评，不就完了？  
  
我一开始就是这么干的。三个星期，撞了三堵墙：  
  
**第一，AI 天生嘴甜。**  
 你问它"我讲得怎么样”，它第一句永远是"整体表达清晰流畅"——废话浓度极高。你要的是教练，不是捧哏。  
  
**第二，AI 数数不可靠。**  
 我那段录音里"这个"到底说了 16 次还是 12 次？它一会儿报一个数。**测量数据错一次，信任就没了**  
——后面它说的所有话你都会打问号。  
  
**第三，每次对话都是新教练。**  
 今天这个 AI 觉得 3 分钟超时没事，明天的觉得超时要扣分。**标尺漂移的点评，等于没有标尺。**  
  
这三堵墙指向同一个结论：**光有 AI 不够，得把"教练"这个角色本身工程化。**  
## 三、核心设计：把练表达当成一个软件项目  
  
我干了二十多年软件，最熟的就是这套逻辑：**可测量才可优化，可积累才复利。**  
  
于是我把每天 10 分钟的录音点评，做成了一个小系统：  
  
![AI-ExpressCoach：一天一条录音的闭环](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6EuqF9cSY2m8KzO5FVHsYASVJeIVz7ZibICkPV2t5ZWYWephm6YGl93ibckcS64qRhsGsaWNRK2fnj1Z8PDehMeicr5YAicF4eCiaZYibn1F7ruVU/640?wx_fmt=jpeg&from=appmsg "")  
  
就这么四层，但每个环节都有讲究。  
  
**环节一：机器管测量，AI 管判断。**  
  
数数这种活，一律交给脚本——语速、字数、口头禅频次、重复卡顿，一个 Python 脚本输出，永远不数错。AI 只干它擅长的：结构分析、亮点识别、打分、开处方。  
  
![第一条铁律：机器管测量，AI 管判断](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6EuqF9cSY2nfeoh73324xibTNfQAoV7abdXSHunu1VuXvdZ8mbljKOPlWS9hhu3vFJ29rzWx3R8MrutKZY7AEEuicted6YMIibV9K7OsPKaYR4/640?wx_fmt=jpeg&from=appmsg "")  
  
这是整套系统的第一铁律。**机器的归机器，智能的归 AI。**  
  
具体到实现上，这个分工比"嘴上说说"严格得多。我给脚本划死了边界：**能数出来的，一个字都不许 AI 猜。**  
- 汉字数只数 [一-鿿]  
，不含标点数字字母；  
  
- 语速必须由 --duration  
 参数喂进去，**没有时长就不报语速**  
——宁可少一个指标，也不能用猜的语速污染数据；  
  
- 口头禅密度 = top3 合计次数 ÷ 分钟；  
  
- 重复卡顿靠正则 ([一-鿿]{2,4})\1  
 抓"他们的他们的"；  
  
- 开头 150 字只摘出来**不做自动判断**  
——结论句识别不可靠，这活交给 AI 读。  
  
反过来说，AI 该干的也一样不许外包：**结构通不通、有没有个人增量、入题几秒、这句是不是金句**  
——脚本做不了，只能读。  
  
**环节二：AI 必须按固定标尺判，不许自由发挥。**  
  
我给五个训练场景各写了一把尺子——  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">场景</span></span></section></th><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">课表</span></span></section></th><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">练什么</span></span></section></th><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">过关线</span></span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">即兴（弹性三档）</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">周一/四</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">限时压缩、快速入题</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">三档中稳定达 2 档</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">金字塔 3 分钟</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">周二/五</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">结论先行、层次支撑</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">3min ±30s</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">复述 2 分钟</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">周三/六</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">转述保真 + 个人增量</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">2min ±30s 且必须有增量</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">回放对比</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">周日</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">自我观察、证据意识</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">至少 2 条具体差异</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">自由复盘</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">机动</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">思维产品化</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">有一个可带走的结论</span></span></section></td></tr></tbody></table>  
每把尺子都有明确的过关线，比如金字塔场景：30 秒内必须出现结论，超时即扣。**AI 点评前先读尺子，照尺子打分**  
——这样无论哪天、哪次对话，标尺都是同一把。  
  
有个细节值得单说：**即兴场景是弹性三档**  
——30 秒、60 秒、3 分钟。现实里的即兴时长本来就不固定，60 秒只是入门档。真正要练的是**可伸缩性**  
：同一个素材，能讲 30 秒、能讲 60 秒、能讲 3 分钟，且每档都收得住。  
  
**环节三：三份文件，三个角色。**  
- **进步账本**  
（progress.md  
，每条录音一行：场景、时长、口头禅、分数、处方）——相当于 git log，**只追加，不删除**  
。进步不可篡改，退步也赖不掉；  
  
- **规律库**  
（glossary.md  
，两个半月攒了 36 条已验证规律）——相当于 bug 库。同类问题出现第二次，就从"随口提醒"升级为"专项训练"；  
  
- **标尺文件**  
（rubrics.md  
）——相当于验收标准，独立存放。**数据和标尺分开**  
，升级方法不动数据。  
  
账本的表头长这样，一行就是一条完整的进步证据：  
```
| 日期 | 编号 | 场景 | 主题 | 时长 | 语速 | 口头禅(次) | 内容 | 执行 | 综合 | 关键发现/备注 |
```  
  
这里有个我特别在意的规矩：**没数据的格子填 ?，不许编数。**  
 一个诚实的问号，比一个看起来很完整的假数字有用得多。  
## 四、几个反直觉的关键设计  
  
**1. 打分打两个，不打一个。**  
  
内容分（观点和结构）与执行分（流畅度和时长控制）分开打。我的账本立刻显形：内容分均值 4.22，执行分均值 3.56——**想得清楚，说得不够流畅**  
。混成一个分，这两种完全不同的能力会互相打掩护，你的训练就找不到靶子。  
  
而且两个分数的锚点也不一样：**内容分的核心是"个人增量"**  
——有增量 4.0 起，纯转述封顶 3.5，增量加验证闭环才能到 4.5 以上；**执行分的核心是"场景任务完成度"**  
——时长、结构、口头禅三者达标 4.0 起。  
  
**2. 处方永远只开一条。**  
  
问题可以列一箩筐，处方只给一条、当天可执行。贪多的处方等于没有处方——人的注意力就那么多，一次改一个才改得动。  
  
对比一下就明白：好的处方长这样——「今晚把这个素材压成 3 分钟金字塔版」；差的处方长这样——「以后注意结构」。  
  
**3. 标尺可以被"复议"。**  
  
有天我嫌某条过关线不合理，提出异议。AI 没有硬顶，也没顺着我，而是**拿数据判断异议成不成立**  
：成立，就改标尺文件、重算当期评分；不成立，就拿我的原文数据反驳。同时还立了规矩：**改标尺不能抵消与标尺无关的硬伤**  
——比如时长标准放宽了，“入题 31 秒"照样扣分。  
  
标尺不是圣旨，是活的。但它只能被证据推动，不能被情绪推动。  
  
**4. 转写错误必须"二分类”。**  
  
录音转文字会出错，但错和错不一样：把"创发"转成"创办"是**同音漂移**  
——机器的错，不算我的；把"王亚"转成"王耀"是**音不同的错**  
——我的错，必须纠正。  
  
判据很朴素：把转写词念出来，跟我的实际发音同音，就归为噪声。**不冤枉我，也不放过我**  
——人机协作要长期跑下去，这种公平感比什么都重要。  
  
同样的道理还用在断句上：“大量句号断句"到底是 ASR 的切分噪声，还是我真的停顿了？也要分开算。  
## 五、效果怎么样：两个半月的账  
  
**分数本身几乎没动**  
：9 月 8 条平均 4.00，11 月 24 条平均 3.98。但账本显示标尺在这期间变严了三次（综合分拆成双评、又上分场景基线）——**考卷变难分没掉，这本身就是进步**  
。这个账前面写过专篇，不重复。  
  
更有说服力的是三件事：  
  
**第一，反馈当天闭环。**  
 口头禅"这个"从单条 16 次到清零，用时三天；讲话稿语速 312（背稿冲刺）压到 160（有呼吸的讲述），用时一晚五遍。问题从"隐隐约约感觉有"变成"白纸黑字在下降”。  
  
**第二，标尺搬进了脑子。**  
 前几天的回放自评，我先自己诊断"停顿词多是因为脑子在停顿"，再对照 AI 从数据推出的结论——**完全一致**  
。教练的最高境界是失业：被点评者长出了自我诊断能力。  
  
这里其实藏着一个我一开始没想明白的机制：**感觉不可靠，往往不是你没进步，是你的标尺在偷偷变严。**后来我把"感觉没提升"这类问题定成了固定处理动作——先拉全表看硬指标趋势，如果硬指标在动而分数没动，那不是停滞，是感知阈值上移；如果硬指标也没动，那才叫停滞。**这两种结论完全不同，混为一谈最伤人。**  
  
**第三，实战没掉链子。**  
 一场 31 分钟的真实客户方案会，讲方案结构清晰，被质疑没硬撑、当场给对比方案。这条实战录音计入账本，**平均分没被拉低**  
——练的东西是能用的。  
  
还有一个意外收获：**这套系统顺带变成了一条内容生产线**  
。复述训练逼我发现"哪些东西我以为懂了、其实讲不出来"；每天的训练记录攒成了文章素材——这个系列的前几篇，全是这套系统的副产品。  
## 六、它是怎么变成一个 Skill 的  
  
跑通流程之后，我干了一件程序员的老事：**把跑通的东西固化成别人也能装的包。**  
  
![七步工作流：一条录音怎么变成一条进步证据](https://mmbiz.qpic.cn/mmbiz_jpg/6EuqF9cSY2l13j1lyKdwlULYhQxSs3rZicVIicNuKC9SheYwz0qF1OWBNR3M9iaJaSdjFs7HricBnBtRzia4ZAePg2icarSicEcSz96iacyZjZOEv1E/640?wx_fmt=jpeg&from=appmsg "")  
  
整个包只有四个文件，纯 Markdown 加一个 Python 脚本，**只用标准库，零第三方依赖**  
：  
```
AI-ExpressCoach/├── SKILL.md            # 教练手册：三条铁律 + 七步工作流 + 输出模板├── README.md           # 部署指南├── references/│   ├── rubrics.md      # 五场景评分标尺（含过关线）│   └── metrics.md      # 指标定义与测量口径└── scripts/    └── stats.py        # 硬指标统计：字数 / 语速 / 口头禅 / 重复卡顿
```  
  
七个步骤拆开看，每一步的  
产物都很明确：  
1. **获取转写**  
——贴文本 / 给路径 / ima note_id 三种入口；  
  
1. **场景识别**  
——按周课表定场景，编号规则 YYYYMMDDNN  
；  
  
1. **机器测量**  
——跑 stats.py  
，同时做转写错误二分类；  
  
1. **场景标尺评分**  
——读 rubrics.md  
，双评打分；  
  
1. **结构分析**  
——逻辑链、个人增量、金句识别；  
  
1. **进步对比**  
——取近 5 条**同场景**  
记录比趋势；  
  
1. **处方与记录**  
——开一条处方，追加一行账本。  
  
**安装方式有四种**  
，覆盖从零到满配：  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">环境</span></span></section></th><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">怎么装</span></span></section></th><th style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);text-align: start;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">适配成本</span></span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><strong style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 15px;">ima</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（腾讯智能工作台）</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">目录丢进工作区，</span></span><code style="box-sizing: border-box;direction: ltr;padding: 0.2rem 0.3rem;font-size: 0.78em;line-height: 1.5;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(245, 245, 245);border-radius: 0.2rem;margin: auto 0px;"><span leaf=""><span textstyle="" style="font-size: 15px;">ima_skill_create -d /path/to/AI-ExpressCoach</span></span></code></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">零适配</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><strong style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 15px;">Claude / Claude Code</span></span></strong></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">放进 </span></span><code style="box-sizing: border-box;direction: ltr;padding: 0.2rem 0.3rem;font-size: 0.78em;line-height: 1.5;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(245, 245, 245);border-radius: 0.2rem;margin: auto 0px;"><span leaf=""><span textstyle="" style="font-size: 15px;">skills/</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;"> 目录即可（目录结构与 Agent Skills 规范同构）</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">零适配</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><strong style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 15px;">Cursor</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;"> 等带 rules 的编辑器</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">SKILL.md 正文粘进 rules，references/scripts 放进项目</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">约 10 分钟</span></span></section></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><strong style="box-sizing: border-box;"><span leaf=""><span textstyle="" style="font-size: 15px;">网页版 AI</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（无代码执行）</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">SKILL.md + rubrics.md 全文贴进系统指令，</span></span><code style="box-sizing: border-box;direction: ltr;padding: 0.2rem 0.3rem;font-size: 0.78em;line-height: 1.5;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(245, 245, 245);border-radius: 0.2rem;margin: auto 0px;"><span leaf=""><span textstyle="" style="font-size: 15px;">stats.py</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;"> 在本地跑完把输出贴给 AI</span></span></section></td><td style="box-sizing: border-box;min-width: 80px;padding: 8px 5px;line-height: 1.5;border-bottom: 1px solid rgb(238, 238, 238);"><section><span leaf=""><span textstyle="" style="font-size: 15px;">降级可用</span></span></section></td></tr></tbody></table>  
最后一种我特意标注了"降级可用"：**不建议让网页版 AI 直接"估"数字**  
——它本来就是这条铁律的靶子。让它评可以，让它数不行。  
  
还有一个设计我特别看重：**数据与方法分离。**  
- DATA_DIR  
 存你的成长档案（progress.md  
 / glossary.md  
 / connections.md  
），**只存这里，不随 skill 分发**  
；  
  
- skill 目录存方法（标尺、脚本、说明书），**升级 skill 不动你的数据**  
。  
  
所以这个包**不含任何个人数据**  
——分享出去零隐私泄露，备份迁移只动一个目录，升级方法不动积累。这三条我一开始没想到，是被"数据跟方法缠在一起，升级一次就丢一次记录"坑出来的。  
## 七、你想练，马上就可以开始  
  
不想折腾安装？把这个地址丢给你的 AI（ima、workbuddy 或其他任何 AI agent），让它装上这个 Skill 就行：  
```
https://github.com/xiejava1018/myopenclaw-skills/tree/master/AI-expresscoach/AI-ExpressCoach
```  
  
装上之后，三步就能跑：  
  
**第一步：用手机录几分钟，转成文字。**  
 手机录音 + 随便一个转写工具，0 门槛。  
  
**第二步：把这段话发给你的 AI，说一句：**  
> 你是我的表达教练，请点评这段录音  
  
  
**第三步：看它给你的点评。****记录比点评更重要**  
——单次点评只是反馈，连续记录才是复利。  
### 一段真实的点评长什么样  
  
下面这是我一段录音拿到的真实点评（2026111002  
，即兴 60 秒场景）：  
  
![AI-ExpressCoach 的真实点评输出：数据 / 亮点 / 问题 / 进步对比 / 双评分 / 一条处方](https://mmbiz.qpic.cn/mmbiz_jpg/6EuqF9cSY2ktQM7N9ic7gITtxR283SmSf1SwUzAGHic2Uwsicf1KflF2LRrtrRPBBWpIKWqDQpJG0K5XKGjQib1sEXyz8OINNsfJZGxlrjicXZQg/640?wx_fmt=jpeg&from=appmsg "")  
  
对着这张图看，前面讲的每一条设计都落地了：  
- **数据区是机器算的**  
——时长 6:20、语速 143 字/分、口头禅频次、重复卡顿，一个数都没让 AI 猜；  
  
- **亮点和问题都引用原文**  
——「“这个"清零了——16 → 0」「素材是即兴的真正本事」「结尾"所以湖南这种天气确实还是值得吐槽一下啊"是感叹，不是观点收束」，没有一句空泛的"整体不错”；  
  
- **进步对比是拿历史记录算的**  
——「口头禅密度 3.82 → 2.68、语速 143 稳定在 140–150 区间、卡顿 1 处」，还有一句很扎心的诊断：「素材层面在涨，剥去层面原地不动——这就是分数卡在 3.5 的原因」；  
  
- **评分拆成两个**  
——内容 3.8 / 执行 2.9 / 综合 3.5，并注明「内容给到 3.8 是因为例子维度优秀；执行 2.9 是时长和入题双超标拖的」；  
  
- **处方只有一条**  
——「今晚把长沙天气压成 60 秒版：只留一个观点、第一句话直接给判断、手机开计时器，60 秒到了必须用自己一句话收尾」。  
  
最后那句「已存 progress.md（11-11 行）+ glossary 新增 2 条」——**这才是这套系统的目的**  
：每一条录音都变成可积累的证据，而不是一次性的爽。  
## 写在最后  
1. **没有反馈的练习只是重复。**  
 练任何东西，先解决反馈，再谈坚持。  
  
1. **感觉会说谎，账本不会。**  
 你缺的不是毅力，是证据。  
  
1. **最好的教练手册，由你自己的 bug 构成。**  
 我这套尺子上的每一条刻度，背后都是我犯过的错——照搬我的没用，攒你自己的才有用。  
  
这套东西我先用文档跑通了流程，后来固化成了一个可以直接安装的 Skill 包——换台电脑、换个 AI 助手，装上就能跑，账本继续攒。  
  
**代码已经开源在 GitHub 上**  
（标尺 + 脚本 + 说明书都在里面，想玩的自取）：  
  
![AI-ExpressCoach 已发布在 GitHub：SKILL.md 含 name / description / 部署配置 / 三条铁律 / 七步工作流](https://mmbiz.qpic.cn/mmbiz_jpg/6EuqF9cSY2mXQVeibSkzjScxYusK1ia3IXjDOz3tib7UaoAnbes8gozDibrracAP2RH8Z7ic1Voq0ClhlHwTYnlv2SwrOCrGfzfmyrY0oU1yqas0/640?wx_fmt=jpeg&from=appmsg "")  
  
仓库地址：github.com/xiejava1018/myopenclaw-skills —— 路径 AI-expresscoach/AI-ExpressCoach  
。  
  
**评论区聊聊：你练过表达吗？卡在哪一步——是不敢开口，还是开了口没人告诉你真实水平？**  
  
延伸阅读：  
1. [AI 什么都知道，你的脑子还剩什么用？](https://mp.weixin.qq.com/s?__biz=MjM5NDMwMjEwMg==&mid=2451853152&idx=1&sn=306bc6ea71db1811f4b464972e72ad6c&scene=21#wechat_redirect)  
  
 —— 同系列：把大脑当缓存，AI 当硬盘  
  
1. [AI 时代，别再靠脑子记了：知识库不是仓库，是你的第二大脑](https://mp.weixin.qq.com/s?__biz=MjM5NDMwMjEwMg==&mid=2451853169&idx=1&sn=3c9588f29482b683fd749017a1c7d781&scene=21#wechat_redirect)  
  
 —— 知识怎么被组织起来  
  
1. [人人都有 AI 之后，你的"目标函数"是什么？](https://mp.weixin.qq.com/s?__biz=MjM5NDMwMjEwMg==&mid=2451853159&idx=1&sn=0258b65e86020aab80a1946119e9cb85&scene=21#wechat_redirect)  
  
 —— 工具平权之后，差距出在哪  
  
1. [《一个人的 SOC》系列导航（18 篇）](https://mp.weixin.qq.com/s?__biz=MjM5NDMwMjEwMg==&mid=2451853175&idx=1&sn=1baeb962d6b5f754067a0d0fd2334f10&scene=21#wechat_redirect)  
  
 —— 另一个"一人项目"跑出来的完整作品  
  
作者博客：http://xiejava.ishareread.com/  
  
  
