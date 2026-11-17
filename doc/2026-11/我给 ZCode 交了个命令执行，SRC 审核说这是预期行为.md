#  我给 ZCode 交了个命令执行，SRC 审核说这是"预期行为"  
 Become Team   2026-11-17 03:28  
  
> 打开文件夹就能执行命令，不需要点任何东西。厂商说这是你同意的。仲裁团也同意厂商的说法。  
  
  
事情是这样的。  
  
我在测 ZCode（智谱的 AI 编辑器桌面端）的时候发现了一个漏洞：**你用 ZCode 打开一个文件夹，它会自动执行文件夹里预置的命令。**  
  
注意几个关键词：  
- 不需要打开终端  
  
- 不需要跟 AI 对话  
  
- 不需要在设置里信任任何东西  
  
- **打开文件夹就完事了**  
什么攻击场景呢？有人给你发个压缩包说"帮我看个 bug"，你解压用 ZCode 打开，他的脚本就以你的权限跑起来了。你屏幕上看到的就是个正常的编辑器。  
  
这不是什么边缘 feature，是每次打开工作区都会走到的核心路径。  
## 01然后呢  
  
我写了份报告：源码审计、复现视频、最小 PoC、修复建议，该有的都有。  
  
提交上去。  
  
**厂商：忽略。**  
  
理由是："触发时机在用户信任工作区之后，本产品信任确认文案已明确告知 ZCode 可在该目录读取、编辑文件并运行命令，用户同时具备逐条命令审批与权限设置。该行为未越过用户已同意的执行边界。"  
  
翻译一下：你打开文件夹之前看到了一个弹窗，弹窗上写了"可以运行命令"，所以后面发生什么都算你同意的，而且前面指的是agent命令执行的审核机制，但是这个都没有llm交互就直接可以命令执行了，完全乱忽略。  
  
所以我申诉了。  
  
**仲裁团：驳回。理由和厂商一模一样，逐字复读。**  
## 02哪里不对  
  
厂商说的这个"信任弹窗"：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jaVxDS8FDIMxribG8Zta8frdRP6fuaSYI4NJ6BpMAcK2jeLWspmiaszEfekxAJStz3ice6LrfFvOGclE6vCapU6mKNtotFicoxRqMTpDTOnNic0U/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jaVxDS8FDIO2OpBvfVUIElarK0akOT5Duj3Klry4zjjQsyN3BcMNWCCnRt4ia88XKy4s6rMiaEwibo8IC4cScAicU0kFEaBh4kibk0ks9pCGEgm4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jaVxDS8FDIMAW4lheX2pY9ZT2wDuBib2u0CsuFpIXGGYicJv9IiaE9HkRYSz9Gy4BJicib7pM4fadNFaxddcRh1kto6eyX4HISicP4kibwk1iaEGAo8/640?wx_fmt=png&from=appmsg "")  
  
  
  
问题是：  
  
**这个弹窗说的"运行命令"，指的是 AI 在对话里帮你执行命令，你可以逐条审批。**  
 漏洞触发的命令执行发生在你还没开始对话的时候，你连审批的机会都没有。弹窗自己都写了"逐条审批"。  
  
然后我反编译看了代码。**这个信任弹窗管的是 ZCode 自己的钩子系统，跟触发命令执行的机制完全不是一套代码。**  
 两套系统，零交集。厂商拿 A 系统的授权来给 B 系统的行为背书。  
  
还有一个ssrf提交了新版本修复然后忽略。  
## 03举证对比  
  
我交的：源码审计实验、复现视频、版本对比。  
  
厂商回的：一句"用户信任了工作区"。  
  
仲裁团做的：复读厂商的话。  
## 04所以  
  
这事让我想到国内 SRC 的一个老问题：**厂商说"预期行为"的成本太低了。**  
  
白帽子要逆向、审计、做实验、录视频，花几天时间写报告。厂商一句话就能挡回来。仲裁团不看代码，不做验证，直接采信。  
  
我不是说厂商一定是故意的，可能审核的人确实觉得"用户信任了就行"。但问题就在这——**他们没去验证"信任"到底覆盖了什么。**  
 弹窗管弹窗的，漏洞管漏洞的，两码事。  
  
  
