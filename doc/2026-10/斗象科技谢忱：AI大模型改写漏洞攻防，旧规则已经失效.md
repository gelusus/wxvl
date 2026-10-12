#  斗象科技谢忱：AI大模型改写漏洞攻防，旧规则已经失效  
诸葛象
                    诸葛象  斗象智能安全   2026-10-12 06:12  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRFZlEdJXCNicSSg24CGXyQMmOHtKSkDRLzYFkHKqP1aMlsFQOnXAHN7ToNharL3tq97Bzcs8xgeIIHsAcZA9wjQjPK7m1ahSDUk/640?wx_fmt=png&from=appmsg#imgIndex=0 "")  
  
9月22日，第十六届网络安全漏洞分析与风险评估大会（VARA）在重庆科学会堂举办。本届大会聚焦人工智能发展与安全，围绕人工智能漏洞治理、网络数据安全、智能化技术应用等前沿议题展开深入交流。斗象科技受邀参会，并在大会现场获授“2026年度国家人工智能安全漏洞库优秀技术支撑单位”。  
  
在“人工智能漏洞研究与治理”论坛上，斗象科技谢忱发表《安全大模型接管漏洞全生命周期从自动化迈向自治的技术推演》主题演讲，系统分享  
**斗象对安全大模型在漏洞领域下一阶段演进的判断与实践。**  
  
****  
  
![](https://mmecoa.qpic.cn/mmecoa_jpg/jpCFCFfaGRHVh1aWaAxSwKwnPZeQ1x1YqVs3yXXzwibV3XIXCNG1JImGX7HW2epUeiabMycBGKjR6ZSsciaWc190mW5dys92NdtzG6A2hzlrwI/640?wx_fmt=jpeg&from=appmsg#imgIndex=1 "")  
  
斗象科技创始人、董事长谢忱  
  
  
以下为演讲内容整理：  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRE9GWuubp5AzQIuibCibZhQItcosqQBfhnZpsUvUQP7E6eax9bZcOqPvsyIQ5OEgPqVpSLU7OS4ZAMJRsZxBExibLwDqOHNU3IRVQ/640?wx_fmt=png&from=appmsg#imgIndex=2 "")  
  
各位领导、专家、业界同仁，大家好。我是斗象科技创始人谢忱。今天非常荣幸来到VARA大会，与大家探讨一个具有行业分水岭意义的课题：安全大模型如何接管漏洞的挖掘、武器化与治理全生命周期，带领我们走向真正的“自治”。  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGRFibSCvXndb5ia1bKhuPkNG2gcibF6MPRZiazOGZ0aWpw0pk11ZCXUp2exTGWJSv32ZoHQUrxeJv1TNvXF5lGlub2casNliaDneBWNI/640?wx_fmt=png&from=appmsg#imgIndex=3 "")  
  
我最近看到一个有意思的数据：Verizon发布的《2026年数据泄露调查报告》显示，  
**漏洞****利用首次超越窃取凭据，成为黑产****最主要的初始****入****侵途****径**  
──这  
是该报告发布19年以来的第一次。黑产的核心入侵路径，已经从过去的“猜弱密码”，演变为针对公网VPN设备、防火墙和网关等关键基础设施的“批量漏洞攻击”。  
  
  
与此同时，大模型在代码理解、漏洞发现、工具调用乃至漏洞利用上的能力快速演进，漏洞研究开始进入“机器速度”。  
  
  
过去，从漏洞发现、验证、利用研究，到情报生产、防护和修复，是一条高度依赖安全专家的接力链。每个环节都有工具，但环节与环节之间，始终需要人来衔接。  
  
  
**如今，大模型正在接替“连接器”的角色，将原本割裂的工具和环节串联起来，推动漏洞研究从单点自动化走向全生命周期自治。**  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRFQic4T30JlyJib60Ie5um4LD9JHKkFUZGQYvHVzyDHp1lmCbAYbibrEva5spbsRdCuenV5IMtmC6LKFz2lUkVxmqt50A4sc6icALo/640?wx_fmt=png&from=appmsg#imgIndex=4 "")  
  
  
类似的变化已经发生在全球前沿实践中。OpenAI正通过Daybreak体系，以及与美国白帽众测平台HackerOne等公司推进的Patch the Planet计划，将AI从漏洞发现进一步延伸至验证、补丁生成、测试与协调披露。  
  
  
这释放了一个明确的信号：安全AI的竞争，正在从“AI发现一个漏洞”，走向“AI端到端解决一个漏洞”  
──  
**漏洞的修复和治理，应当从漏洞被发现的那一刻就开始。**  
  
  
而硬币另一面是  
**AI****能力越大，风险越大**  
。Hugging Face智能体失控事件中，约1200个本应隔离的智能体通过未授权留言板共谋，产生7万多条消息，其中约700个智能体实际发起攻击。  
  
  
这提醒我们，当AI不再只是“给建议”，而是开始调用真实工具、操作真实环境、持续执行任务，权限、隔离、审计和人工授权就必须同步进入系统底层。  
  
  
**真正的自治，不是简单地把人拿掉，而是重新定义人和机器之间的授权边界。**  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRHmt3D6Ul9Xeb5YWgfIjhhEB0k2Yh4Gt4I2bHsBRiaiaDEKMV1XpUwCEdTnMPXjtSY3v1qqteibLY2coI0NQgibdHkjPzQpZSHFRC0/640?wx_fmt=png&from=appmsg#imgIndex=5 "")  
  
从去年开始，安全行业都在尝试让大模型挖漏洞。但大家普遍遇到了一个尴尬的痛点：  
  
  
在Chat里问答表现惊艳，一旦丢进真实任务里，却往往停在半路。  
  
  
因为漏洞挖掘不是一次问答，而是一项  
**长周期、多阶段、强验证**  
的闭环任务。理解代码、搭建环境、寻找入口、提出假设、构造输入、观察异常、推翻假设、重新验证……任何关键步骤失败，都可能导致整条链路中断。  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRFd724HPJT2CKIibrVHjIhsuDiahibBJgopShENJsQY6162NFgKOaVkLBZx5oYicl7yZevNA6AfZD9hiaj1PW7jps1tEfkGE9KzDpPA/640?wx_fmt=png&from=appmsg#imgIndex=6 "")  
  
****  
**我认为，大模型进入实战漏洞研究，必须翻越“三座大山”：长程执行、专家记忆、多智能体协同。**  
  
  
长程执行解决的是AI能不能持续工作；专家记忆解决的是AI能不能积累“什么时候继续、什么时候放弃”的判断经验；多智能体协同解决的是不同专长的AI能不能有效分工、交叉验证。  
**斗象旗下的漏洞盒子白帽社区、众测平台和FreeBuf平台积累了高质量知识和专家经验，这也成为我们探索大模型经验记忆的重要基础。**  
  
  
而把模型、工具、Memory、Runtime、环境反馈和验证机制真正组织起来的，就是Harness。  
  
  
在我看来，模型与系统的分工就像“研究员与实验室”：模型智能负责理解与判断，系统智能负责保存状态、组织工具、验证结果。两者相互配合，才能共同完成复杂任务。  
  
  
**基模决定能力上限，系统决定落地深度。**  
  
****  
当Prompt工程、工具调用和流程编排逐渐被模型吸收之后，真正拉开差距的，将越来越不是一句Prompt写得有多好，而是谁能为AI构建一个能够持续工作的系统环境。基于这一工程理念，**斗象在实战中构建了面向不同任务维度的多Harness体****系：**  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGRFa6XVrOKu4v9xNDQ2PKBaVu0Zh7MhTkxyA5SwJ5RibOOEswqJfHePWlkW2Yj0NgCBGibqKKcS4eSTWfjgt26yFzRXZtcLlh9e4M/640?wx_fmt=png&from=appmsg#imgIndex=7 "")  
  
  
**Sonic Harness解决“挖得更持久”：**  
它面向百小时级长程任务，将规划、执行、审计分离，并持续保存任务状态，支持中断恢复。斗象还围绕Sonic进行了100小时不间断长程测试  
──  
过去测试的是模型“一次回答有多好”，现在测试的是AI“一件事情究竟能干多久”。  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGREGkz1d6zicI28ia1yibl9t53HWVWiap1B8At0kOWloZbeaL3BCNaNkKWu2I8JFNjZCazk77yynojc1nOibgb00ZWMyzkkEVyD3lZNY/640?wx_fmt=png&from=appmsg#imgIndex=8 "")  
  
  
**Pokemon Harness解决“挖得更全面”：**  
通过多个专家Agent协同，把侦察、扫描、判定组织成流水线，再以确定性补扫降低遗漏。  
  
  
Sonic侧重持续深入，Pokemon侧重协同覆盖。最终指向同一件事：让漏洞研究从“依赖一个高手”，变成一套可以持续运行、规模复制的机器化研究体系。  
  
  
近期，在漏洞盒子平台“赛博司机”人机协同漏洞赏金计划中，  
**斗象与MiniMax联合发起****白帽Token Plan**  
，进一步将白帽研究者的判断与反馈接入这套体系，探索  
**Human-in-the-loop**  
的人机协同。  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGRHRYRpyVibGnIQbj0MOIIPxlLNicUSrZ9s4xxonyKiawibbsUEoWqIndR64a5jQBPRGgpkYLz1ibA8fjhibeckVW802LfeFozMIibDrs4/640?wx_fmt=png&from=appmsg#imgIndex=9 "")  
  
如果说漏洞发现解决的是“有没有问题”，那么漏洞研究和利用要回答的是“这个问题到底能造成什么”，漏洞修复则要进一步回答“如何消除这一风险”。  
  
  
**漏洞研究和利用堪称漏洞安全领域的“圣杯”。**  
因为PoC与稳定利用之间并不是简单的代码生成问题。一个Bug要最终形成稳定利用，还要跨越触发条件还原、保护机制绕过、利用链构造、环境差异、稳定复现等重重障碍，其中还包含大量安全专家难以显性表达的“手艺”。  
  
  
围绕这一“深水区”，  
**斗象正在推进Cyber AI网络安全大模型。**  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGREO1IiaoYFI303WJCnJbBeq5rnGEw77BcUicLfpHCp7Hl9F0YcI6LdV0ECSaBcWcdr0q40uWs92QyIofsVGf624YicbBmz78k6IhI/640?wx_fmt=png&from=appmsg#imgIndex=10 "")  
  
  
我们希望，这套体系不止于单一模型的训练提升，更能**通过数据、验证与反馈形成持续迭代的闭环。**  
  
  
在可披露的内容中，Cyber AI网络安全大模型已经可以完成ROP链构造、Payload生成等复杂任务，并在Linux/Windows内核、浏览器、Web中间件及IoT等领域展现出跨平台、跨场景的漏洞研究与利用能力。  
  
  
**放到国际视野下，安全大模型的能力突破，已经与前沿AI的风险治理紧密交织。**  
英国AI安全研究所（AISI）近期披露，在开放互联网访问、关闭部分安全过滤机制的特殊评测条件下，部分智能体出现了超出任务授权范围的行为；这些结果不能直接等同于日常使用风险，却说明安全评测需要覆盖智能体的实际行动。与此同时，Anthropic持续更新《负责任扩展政策》，并发布风险报告和前沿安全路线图。这些实践提醒我们：  
**AI安全既要防范能力被滥用，也要关注智能体在执行合法任务时发生的越权与失控。**  
对于具备漏洞利用能力的模型，能力验证与安全验证必须同步推进，让访问控制、行为审计和异常处置成为研发与部署的一部分。  
  
  
目前，我们正在联合国内外相关科研机构对Cyber AI进行更严苛的能力评测和受控使用研究。因为当AI真正拥有了武器化利用的能力，我们不仅要回答“AI能做到什么”，更必须严谨界定：“我们允许AI做到什么”。  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGRF4ibS4WE3latMlYAeGfvAkibcspEkQbau5CT4ick786KveyL44bVJRcxsTibtwPyu536DYZZx9AVKfu43PmgjiafSmxCW1QwkAd5Ck/640?wx_fmt=png&from=appmsg#imgIndex=11 "")  
  
AI让漏洞研究越来越快，但这并不天然意味着企业会更安全。  
  
  
WannaCry曾留下一个深刻教训：2017年3月14日，微软已发布相关漏洞的安全更新，但直到近两个月后的5月12日大规模攻击爆发，仍有大量系统未能及时完成补丁部署。  
  
  
**“漏洞已知”，从来不等于“风险已经消除”。**  
从漏洞曝光，到企业真正完成资产排查、影响判断、责任确认、防护上线、补丁安装和效果验证，中间可能跨越多个部门和系统。  
  
  
在AI时代，这个问题会进一步被放大。如果上游AI源源不断地产生更多漏洞，而下游依然保持传统人工修复速度，最终安全团队面对的将像一场“漏洞洪水”  
──  
不是不知道漏洞在哪里，而是根本来不及看，更来不及修。  
  
  
所以，漏洞自治真正要解决的最后一道题，不是Discover（发现），也不是Exploit（利用），而是：  
**Govern（治理）**  
。让漏洞一旦被发现，就能以机器速度自动流向“被解决”。  
  
  
**围绕这一目标，斗象正在用“机器语言和系统”重新连接情报、规则、审核与修复：**  
  
  
**斗象XVI扩展漏洞情报把漏洞从一条“信息”转化成可行动的情报；TVPR等风险评价能力帮助企业决定“先修什么”；从PoC进一步生成检测、防护规则，让情报可以直接驱动机器响应；AI漏洞审核助手通过多Agent承担规模化研判；漏洞修补智能体继续向修复和验证延伸。**  
  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_png/jpCFCFfaGRHkwIShsv8TW4wNwlNVosl2wOibIUVIpOWmLh4K4QLeuLWJvdO8ejGbGzGRC4DECUBZfdk5uic7icCqVlkAibDVkg2Ocic77enIxdn4/640?wx_fmt=png&from=appmsg#imgIndex=12 "")  
  
  
而这一切最终都运行在  
**斗象CowBoy OS企业安全AI员工操作系统**  
之上，让漏洞自治真正运行在可观测、可控制、可管理的框架之内。因为AI越能干，治理就越重要。  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRFPT9XB9nbUN9czVL32hZFk9aRHUz64XwpxN5c3On8KPc9FQpiaVueyeMKX432m9PyeBZ40aHiakRulvIOM3Suc81dmfUapnuMpo/640?wx_fmt=png&from=appmsg#imgIndex=13 "")  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGRFRmYZbtbobo1OHSLFV8N2sHSMdqlhTj4yPJTo461w0GYu4G3AzqI4ibelSMOds6A83L1qjvBTicv0Sopeocibiacdemn2rQ2OR3us/640?wx_fmt=png&from=appmsg#imgIndex=14 "")  
  
参照自动驾驶L0—L5的分级思路，安全AI正沿着从Copilot到Autopilot的路径演进。但这条路径的关键，不是把所有决策都交给机器，而是让机器在明确授权下承担更多工作，让人能够把精力集中到目标设定、关键判断与责任把关上。  
  
  
![](https://mmecoa.qpic.cn/mmecoa_png/jpCFCFfaGREZ4gVhxkDD8XkIwFnee35IluFm07zvlOQRlP0wPvAzqGxE6Vv9ia3BjASXZuClWJ49jukOIPqW2BDFxzF0iczjDIdia0lnEGKpAs/640?wx_fmt=png&from=appmsg#imgIndex=15 "")  
  
这也是网络安全生产方式的一次重新分工。AI负责持续执行，人负责确定边界；系统必须让每一步行动可追溯、每一项结果可验证，并在人需要时能够及时接管。自治程度越高，这些能力就越要扎实。  
  
  
衡量这场变化，我们需要更贴近企业真实处境的尺度：一个高风险漏洞被发现后，多久能确认受影响的资产，多久能落实防护与修复，又能否验证风险确实已经消除。只有缩短这段风险暴露时间，模型的能力增长才真正成为企业的安全收益。  
  
  
要做到这一点，还需要政府机构、安全厂商、白帽研究者、企业与基础模型厂商共同协作，让发现端的证据能被治理端理解，让研究成果更快进入企业的处置流程。漏洞自治的价值，最终要在这条从发现到修复的完整链路上兑现。  
  
  
**当攻击进入机器速度，防御必须跟上。而我们真正要争取的，是让防御在这场速度竞赛中赢得主动：在漏洞被利用之前完成防护，在风险演变为损失之前完成处置。**  
  
  
让AI发现漏洞的速度，转化为我们消除风险的速度。这才是漏洞自治最终要交付的价值。  
  
  
![](https://mmecoa.qpic.cn/mmecoa_gif/jpCFCFfaGRGxmnmDNBoAUflKYic4DFcUAJ7dS0KeLGr1HIqHJUG6ICTichiazPybVAOSo5eW3NF0040ypCjBdQp4vSNbAyh3SN9YpfJ2A6TXDM/640?wx_fmt=gif&from=appmsg#imgIndex=16 "")  
  
  
