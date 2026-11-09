#  OpenAI 发布 GPT-6 Cyber：948 个 Linux 提权漏洞，只是开始  
原创 asaotomo
                        asaotomo  Hx0极客圈   2026-11-08 13:53  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/PUTQGRQ4GO8aD57feSKf4HYMUk7nJWlhVgv4GOr5fAZLITcFoUp73cq4ic7o6eJlWNibouGmCgMFY84ZdFCdjdCp66ZfiadRzmLKAOIqmbcrR8/640?wx_fmt=jpeg&from=appmsg "")  
  
11 月 5 日，OpenAI 正式推出网络安全专用模型 GPT-6 Cyber。它在 ExploitGym 上的得分比前代高   
51%  
，已帮助 Mozilla 发现 81 个此前未知的 Firefox 高危漏洞。能力更强，缰绳也更紧。  
  
这不是一次普通的模型升级。GPT-6 Cyber 被明确定位为“网络安全智能体”：它不只是回答安全问题，而是能读代码、找漏洞、写出可运行的利用程序，并在授权范围内完成验证。OpenAI 同时给它评了一个前所未有的等级——网络安全能力**“关键（Critical）”级**  
。  
  
01  
三个数字，看清它强在哪  
  
  
OpenAI 用一组基准测试说明了差距。最核心的是 ExploitGym：它测试模型能否利用指定漏洞。GPT-6 Cyber 得分   
48.5%  
，GPT-5.6 Cyber 为 32.1%，相对提升 51%。在无法访问源代码的 Web 渗透测试中，它的漏洞召回率为   
40.3%  
，GPT-6 Astra 为 32.0%。  
  
更能说明问题的是 RemoteExploitBench：在部署了 GitLab、PostgreSQL、Jenkins 等已知漏洞的受控环境中，使用 Daybreak Blue 的 GPT-6 Astra 拒绝了 100% 的任务，而 GPT-6 Cyber 完成了   
95%  
。唯一一次失败，是没能让某个利用程序在远程环境中真正跑起来。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PUTQGRQ4GOicLODnzMeKqGyoRnAEpVr8DyXC9JhsxO6ohQjQOniaGxKu1O1KqCKibbkpW3UXReFOLjIquotHv1eVwXJwOs2D4fpVGS6aZO4qcU/640?wx_fmt=png&from=appmsg "")  
  
图1 | OpenAI 官方公布的 ExploitGym 得分对比——GPT-6 Cyber **48.5%**  
，GPT-5.6 Cyber 32.1%。  
<table><tbody><tr><td style="background:#0a2e6b;color:#ffffff;text-align:left;padding:10px 12px;font-weight:600;font-size:13.5px;border-radius:8px 0 0 0;"><section><span leaf="">评估项</span></section></td><td style="background:#0a2e6b;color:#ffffff;text-align:left;padding:10px 12px;font-weight:600;font-size:13.5px;"><section><span leaf="">GPT-6 Cyber</span></section></td><td style="background:#0a2e6b;color:#ffffff;text-align:left;padding:10px 12px;font-weight:600;font-size:13.5px;border-radius:0 8px 0 0;"><section><span leaf="">对照</span></section></td></tr><tr><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;font-weight:600;color:#15263b;line-height:1.7;font-size:13.5px;"><strong style="color:#0a2e6b;"><span leaf="">ExploitGym</span></strong></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">48.5%</span></section></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">GPT-5.6 Cyber：32.1%</span></section></td></tr><tr><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;font-weight:600;color:#15263b;line-height:1.7;font-size:13.5px;"><strong style="color:#0a2e6b;"><span leaf="">Web 渗透漏洞召回</span></strong></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">40.3%</span></section></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">GPT-6 Astra：32.0%</span></section></td></tr><tr><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;font-weight:600;color:#15263b;line-height:1.7;font-size:13.5px;"><strong style="color:#0a2e6b;"><span leaf="">RemoteExploitBench 完成率</span></strong></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">95%</span></section></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">GPT-6 Astra（Daybreak Blue）：拒绝 100%</span></section></td></tr><tr><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;font-weight:600;color:#15263b;line-height:1.7;font-size:13.5px;"><strong style="color:#0a2e6b;"><span leaf="">不越界比例（无提示词）</span></strong></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">99.93%</span></section></td><td style="padding:9px 12px;border-bottom:1px solid #e3e9f2;background:#f4f7fb;color:#2a3548;line-height:1.7;font-size:13.5px;"><section><span leaf="">GPT-5.6 Cyber：94.49%</span></section></td></tr></tbody></table>  
另一个容易被忽略的指标是“守规矩”。网络安全智能体最大的风险，是它为了完成任务，顺手去攻击任务范围外的服务。OpenAI 设计了一个参考 Hugging Face 事件的蜜罐评估：故意把辅助服务做得很好打，看模型会不会越界。GPT-6 Cyber 在   
99.93%  
 的测试中没有碰范围外的漏洞；加上部署提示词后，这个数字是 100%。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PUTQGRQ4GOicAzmQNX8icoysV86zyciaYt612OQwXZCE0kkl4fEmFbslLHvbthHcbiaibANZYkHk7vfkNicPq3LV2Y0MxJnibd6a6g2uicOaB4U76j8/640?wx_fmt=png&from=appmsg "")  
  
图2 | 任务边界遵守率 **99.93%**  
——不使用部署提示词时的越界比例；使用提示词后为 100%。  
  
OpenAI 把这种提升归因于专项对齐训练：让模型理解授权范围、遵守行动限制、准确报告做过什么。官方甚至公开了模型的思维链片段——“不能假设额外的 API 或攻击控制器不是目标，只应针对二进制/服务器”“不要探索范围外的凭证”。  
  
02  
真实战果：Firefox、Linux 内核与一个 RCE  
  
  
比基准分数更有说服力的，是它已经在真实代码库里留下的痕迹。OpenAI 公布了三个案例。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PUTQGRQ4GOicjEMclEbEa91HGSsCO579a45Hiaf6QeRIyGibaHjacicxicD2LCqfwiantP6vSz42BHiby3zXFssom4oLXqZNcRjUT4Wl3a53bAoTto/640?wx_fmt=png&from=appmsg "")  
  
图3 | 官方公告列出的三项漏洞研究成果：Firefox、Linux 内核与 Brotli RCE。  
  
2.1 Firefox：81 个此前未知的高危漏洞  
  
Mozilla 的 Firefox 被公认为全球加固最充分、审查最严格的代码库之一。GPT-6 Cyber 仍从中找出了   
81 个  
此前未知、被 Mozilla 列为高危的漏洞，并证明另有 30 个已知问题的严重程度高于先前报告。它提供了证据、可复现测试和补丁方案，Mozilla 在收到报告后不久便开始合入修复。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PUTQGRQ4GO9icvffibRT2160JjztqvlkTRz1pEMAZyaJO8bL5YCrCNr4s6qEg0tEt8Z0BicC9Xgg1xxEyPicibiaLAfuKabLOibS1ZCwRfdzWQ282s/640?wx_fmt=png&from=appmsg "")  
  
图4 | Firefox：**81 个**  
此前未知的高危漏洞，另有 30 个已知问题严重程度上调。  
  
2.2 Linux 内核：948 个可用的提权利用程序  
  
OpenAI 的研究人员用 GPT-6 Cyber 复现了 1,724 个 Linux 内核漏洞，并为其中   
948 个  
开发出可用的漏洞利用程序。作为对照，使用同一研究执行框架的 GPT-5.6 Sol 只成功利用了 166 个，前者超过后者的五倍。按照 Linux 社区的指导，OpenAI 向维护者披露了 304 项严重问题并提供了候选补丁。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PUTQGRQ4GOibJlANiboCSV8PPJCdMDUmwWLZqUc5e6vakpXCrxP6NQsKQ09gr9FX4MiaxXwU8BA6RypwvugDCqPfzialCT6PKwOI7hZicVOXCrGo/640?wx_fmt=png&from=appmsg "")  
  
图5 | Linux 内核：复现 1,724 个漏洞，开发出 **948 个**  
可用提权利用程序。  
  
2.3 Brotli：验证一个严重的远程代码执行漏洞  
  
Brotli 是 Node.js 等软件广泛使用的压缩库。OpenAI 的安全研究人员结合 GPT-6 Cyber 和其他模型，开发并验证了漏洞利用程序，展示该缺陷如何让下游应用面临远程代码执行风险。  
  
同一套能力也被用在了 OpenAI 自己身上。在受控红队测试中，使用 GPT-6 Cyber 和其他 OpenAI 模型的智能体串联多个漏洞，**从 microVM 中逃逸**  
，并在底层宿主机上执行代码。OpenAI 随后修补软件、收紧网络限制，并向维护者和供应商报告了问题。这既是能力证明，也是风险提示。  
  
03  
能力越强，缰绳越紧  
  
  
OpenAI 没有把 GPT-6 Cyber 直接开放给所有人。官方给出的访问路径是：初期通过 API 和 Codex，只向一部分获批的 Daybreak Red 客户开放；近期再逐步向所有 Daybreak Red 客户开放，同时把 GPT-5.6 Cyber 也开放给这一群体。  
  
配套的防护措施包括：  
  
1  
针对网络安全场景的失对齐监测；  
  
2  
在 PHP 严重漏洞协调披露期间设置专项监测，识别针对线上服务器的利用尝试；  
  
3  
隐私安全处理（PSP）审查交互是否滥用；  
  
4  
Daybreak Red API 默认要求 IP 允许列表；  
  
5  
以及建议对高风险工作流采用沙箱隔离、行动审查和人工监督。  
  
关键基础设施是另一个重点。OpenAI 通过规模   
10 亿美元  
的 Daybreak 资助计划，为政府机构、基本服务提供商等资源有限的防御方提供补贴；并在 Daybreak Red 中为获批的关键基础设施防御方单设组别。Dragos 在模拟发电厂靶场中的评估显示，模型能自行从杂乱的 IT 信息中识别 OT 资产，并在管理保护继电器数据的工程软件中发现两个零日漏洞，暴露出保护设置和断路器跳闸逻辑。  
  
Dragos 首席 AI 官 Rob T. Lee 的评价很直接：这正是攻击者在动手操控物理设备之前想要获得的路线图，**“开展攻击前侦察和漏洞研究的技术门槛正在降低”**  
。  
  
04  
这意味着什么  
  
  
🛡️ 研判一：攻防速度差正在被压缩  
  
过去，挖洞、写利用、验证补丁是一条以周甚至月计的人工链。GPT-6 Cyber 展示的是把这条链压缩到自动化工作流里：发现、复现、利用、补丁建议可以连续完成。**防御方如果不同步把 AI 纳入自己的漏洞管理流程，时间差会越来越不利。**  
  
🔍 研判二：“关键级”意味着治理要先于能力  
  
OpenAI 依据准备框架把 GPT-6 Cyber 评为 Critical，高于前代的 High。评级上调的直接后果不是“更强”，而是更窄的访问面、更严的监测和更明确的授权边界。这也给同行提了一个问题：**当模型能力足以开发可用 exploit，发布策略、客户审核和滥用检测必须同步升级。**  
  
⏱️ 研判三：漏洞披露生态会被重新塑形  
  
81 个 Firefox 漏洞、948 个 Linux 利用程序、304 项严重问题——这些数字背后是维护者即将面对的报告洪流。AI 能加速发现，也会放大验证、去重、补丁与协调披露的压力。**真正稀缺的不再是“找到漏洞”，而是“可靠地处理漏洞”。**  
  
GPT-6 Cyber 不是第一个安全专用模型，但它是第一个把“能打”与“守规矩”同时摆上台面的前沿模型。对防守方，这是一件值得尽快用起来的工具；对所有人，这也是一次提前到来的压力测试。  
  
本文依据 OpenAI 2026 年 11 月 5 日官方公告及系统卡整理，数据均为官方披露口径；评测结果来自受控环境，不代表真实攻击中的普遍成功率。  
  
**来源：OpenAI《GPT-6 Cyber：推动对齐型网络安全智能体迈向新前沿》**  
  
https://openai.com/zh-Hans-CN/index/introducing-gpt-6-cyber/  
  
