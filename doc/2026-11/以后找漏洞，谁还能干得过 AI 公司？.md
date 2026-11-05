#  以后找漏洞，谁还能干得过 AI 公司？  
原创 🅰🅸
                    🅰🅸  独眼情报   2026-11-05 04:37  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cBGhzWwhSAgwxAibynh7hdzubAjBAkcFGU2PAIBEs8r0EmF5eguxqcv4Gsffz5AGiaVv0Ra8mV65tLlv8m6AlKyzFFpXNCK4wic20jTiaH6Nuoc/640?wx_fmt=webp&from=appmsg "")  
  
**112 小时，21 个 Linux 内核提权 Exploit；10 天，948 个漏洞利用程序。OpenAI 用约 121 万美元的 API 成本，完成了一次传统漏洞研究团队难以想象的批量化漏洞挖掘。更值得警惕的是，AI 不仅能发现漏洞，还能自主研究利用技术、编写 Exploit，甚至生成修复补丁。漏洞研究正在从拼个人技术，变成拼模型和算力。**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAiaHJAhFdAvSLgHpBmCWx0aR8qzI2gJIoAO1bRibHC2bgOzhHkzBoia534Xda3cibH4ibMOy8j0VOg0RWrT3hS4GSelBKOxgBtxVKuo/640?wx_fmt=webp&from=appmsg "")  
  
2026 年 11 月 4 日，OpenAI 发布《A thousand fewer paths to root》，披露了一项利用新一代网络安全模型自动挖掘 Linux 内核漏洞的实验。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAiaM16HpXXnBmU1mm2EseeLTchmFXe07q6B3CEj25qzFumHBMs5SdDo3nqsWmzBz0o1UHPnPQaSKzyjhn8Vbql4oB69hORfwLVs/640?wx_fmt=webp&from=appmsg "")  
  
实验中，一个尚未公开发布的网络安全模型，在约 10 天内扫描 Linux 内核，为一个常用发行版生成了 948 个互不重复的本地权限提升漏洞利用程序，能够让普通用户获得 root 权限。整个过程消耗约 1,213,430 美元的 API 额度，平均每个漏洞约 1,279 美元。  
  
另一组实验则针对谷歌的 kernelCTF 环境。AI 流水线运行 112 小时，生成了 21 个提权 Exploit，成本约 3.3 万美元。  
  
这不是简单的 AI 辅助代码审计，而是一套能够自动完成漏洞发现、验证和利用开发的研究流水线。  
## 一、10 天，948 个内核提权漏洞  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cBGhzWwhSAj1BAAsR6pzbM7rtHHYUhpOtcrdCvvqP1amN3ibKL5gvibae22UiaKVJekuU2aoGRgWFUDE1MKUibW2wGT0LvazjaxvMxXNE8ruphk/640?wx_fmt=webp&from=appmsg "")  
  
OpenAI 将整个研究流程拆成五个阶段。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAgFNXM1cloaDvwXf61e1Ya9K8Dib4w4hgVkYmJ1qkm1zUZbPMcRAzIDYQYwj9NL4eA5rDf9k9D4cTDbu5QPoYadTyTibeHQ7LRDg/640?wx_fmt=webp&from=appmsg "")  
  
首先，侦察代理从 Linux 内核中筛选普通用户能够访问、具有安全研究价值的代码区域，最终选定 5034 个目标，涉及约 280 万行代码，约占内核代码规模的 7%。  
  
随后，分析代理在 12 小时内生成 2308 条去重后的漏洞候选。  
  
第三阶段由分诊代理负责，在隔离虚拟机中生成验证程序，通过内存错误检测等机制反复测试，最终复现了 1724 个漏洞候选，成功率约 74.7%。  
  
接下来，利用代理尝试开发真正可用的 Exploit。在默认安全防护开启的环境中，模型需要证明漏洞不仅存在，而且能够实际完成攻击。  
  
经过利用开发和变体分析，最终得到 948 个互不重复的本地提权漏洞。  
  
值得注意的是，整个流程最后仍然需要人工审核，但大部分重复性研究工作已经交给 AI 自动完成。  
## 二、AI 开始创造新的漏洞利用技术  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAg2Uiby1unhI1hwyMZ0Ph0pzaSyE6Jd3E7G2LXzZqgLQz43icyCYryAPFcwbQfEw5jDYicLY1mECe4FPFD8Ouiaia96XnDr38fFYXa0/640?wx_fmt=webp&from=appmsg "")  
  
比漏洞数量更值得关注的是模型的利用能力。  
  
早期模型并不擅长自主选择内核利用技术。OpenAI 曾整理 103 种已知技术放入模型环境，在 63 个候选漏洞上，将利用成功率从 76.2% 提升至 88.9%。  
  
但新一代模型已经能够自主识别和选择利用技术。撤掉预先整理的技术库后，模型表现反而更好，重新加入技术库也只带来约 1.5 个百分点的提升。  
  
OpenAI 还抽样分析了 166 个 Exploit，认为其中涉及至少 10 种此前研究团队没有掌握的新技术，并交由第三方内核利用专家审查。  
  
这意味着 AI 不再只是重复人类已经掌握的漏洞利用方法，而是开始具备自主探索新利用路径的能力。  
## 三、漏洞研究开始按算力计价  
  
OpenAI 公布了两代模型的实验对比。  
  
2026 年 7 月，GPT-5.6 Sol 找到并利用了 166 个不重复的提权漏洞，成本约 51.37 万美元，平均每个约 3094 美元。  
  
10 月，新模型完成 948 个，成本约 121.34 万美元，平均每个约 1279 美元。  
  
三个月时间，漏洞数量增长超过五倍，单个漏洞成本下降约 59%。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAgcdnsp5T7K5luw22UC9WmVXREuKIrib1eojkcx47JGUibw2EgD43IPw6K7yLutV8aKdfXHclpoC4qU5yP4WotkrGOicW2SVZ8UWA/640?wx_fmt=webp&from=appmsg "")  
  
OpenAI 还按照历史漏洞赏金标准进行了估算。如果每个漏洞对应 2 万至 7 万美元奖励，948 个漏洞的理论赏金价值约为 1890 万至 6630 万美元，相当于实验成本的 16 至 55 倍。  
  
当然，这只是理论估值，并不意味着所有漏洞都能获得赏金。重复漏洞、披露规则和奖励上限都会影响实际收益。  
  
但它至少说明了一个趋势：当漏洞研究能够自动验证，模型和算力就可以直接转化为研究产量。  
## 四、个人研究员还能竞争吗？  
  
过去，漏洞研究依赖个人长期积累的代码审计、逆向分析、调试和 Exploit 开发经验。  
  
现在，AI 公司拥有另一套竞争优势：前沿模型、充足算力、并行 Agent，以及能够持续运行的自动化研究流水线。  
  
一个人可能需要数周分析一个复杂漏洞，而 AI 公司可以同时启动大量任务，对数百万行代码进行批量审计。  
  
当然，AI 也有局限。实验中，部分漏洞因为复杂竞态、特殊触发条件和验证耗时过长而未能完成利用。早期实验甚至出现过 Exploit 成功获取权限，却实际上依赖另一个漏洞的情况。  
  
这说明漏洞真实性、利用有效性和补丁质量仍然需要人工审核。  
  
但个人研究员面临的竞争压力已经发生变化。单纯依靠手工阅读代码、寻找常见漏洞和编写 PoC 建立的技术优势，正在被模型能力和自动化体系不断压缩。  
## 五、找漏洞越来越快，修漏洞却未必跟得上  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAgcpZKEpIibHWUQib5La8jBPSBibBep57ibmGm7GbmuGibSaDZLVpGCHxEiczLwfjH9nPuIuAfwhkeJFb1VBy5ctHIuicdFaFeNl08xks/640?wx_fmt=webp&from=appmsg "")  
  
扫描期间，Linux 内核上游已有 61 个相关提权漏洞完成修复，43 个获得暂存补丁，另有 299 个处于公开报告和讨论过程中。这些工作来自其他贡献者，并非全部由 OpenAI 完成。  
  
OpenAI 也尝试让 AI 自动生成补丁，但最初的补丁经常存在代码冗余、性能取舍和遗漏漏洞变种等问题，需要经过多轮迭代才能改善。  
  
这暴露了未来网络安全的另一个问题：漏洞发现和利用开发可能首先实现规模化自动化，而补丁审核、代码合入和大规模部署仍然需要人类参与。  
  
攻击者可能在漏洞公开后迅速生成 Exploit，防守方却还需要时间完成修复和部署。  
  
安全窗口可能因此进一步缩短。  
## 六、真正的竞争才刚刚开始  
  
但核心趋势已经很明确：在源码公开、运行环境可控、结果能够自动验证的目标上，AI 已经具备规模化漏洞研究能力。  
  
未来，复杂漏洞根因分析、硬件安全、闭源目标研究、攻击链设计和补丁质量评估，仍然需要专业人员参与。  
  
但传统依靠个人经验和时间成本建立的漏洞研究壁垒，正在被迅速削弱。  
  
以前，一个漏洞研究员需要和另一个研究员竞争。现在，他可能需要和一家拥有顶级模型、数百个 AI Agent 和充足算力预算的公司竞争。  
  
**以后找漏洞，谁还能干得过 AI 公司？**  
  
答案或许是：拥有同等级模型、算力和自动化研究体系的人。  
  
而对防守方来说，更现实的问题是：当 AI 找漏洞的速度远远超过人类修复漏洞的速度，我们还能不能守住自己的系统？  
> 参考： https://openai.com/index/a-thousand-fewer-paths-to-root/  
  
  
  
