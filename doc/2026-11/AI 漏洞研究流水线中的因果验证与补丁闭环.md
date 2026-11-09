#  AI 漏洞研究流水线中的因果验证与补丁闭环  
原创 ITPAPA
                        ITPAPA  ITPAPA   2026-11-09 05:12  
  
拿到 Root 也可能验证错了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/sSvWYibwznrONkE0nfFvhTcXnlIZ6ROIqQic9QnxbGjdounf7oP3KibkWgpJeC0AVyI2ECtpQpBibntAG7XkZNGqe1lSF5RMk93Hyx5ibyM9iasnk/640?wx_fmt=png "")  
  
一条利用链最终获得 uid=0，只能证明测试环境里存在某条可行路径。要证明 AI 找到的是它声称的漏洞，还必须对路径做干预、差分和回退验证。  
# 一次成功利用为什么会是错误答案  
  
2026 年 11 月 4 日，OpenAI 公布了一套面向 Linux 内核的自主漏洞研究原型。报告称，这套流水线生成了 2,308 个候选发现，其中 1,724 个可以动态复现，948 个形成了可工作的本地提权利用；同一批工作还产生了 291 个信息泄露 PoC。数字很大，但真正值得安全工程师停下来看的，是报告中一个更小的数字。  
  
在早期一次包含 200 条“成功提权”的运行中，人工复核发现有 20 条并没有通过分配给它们的漏洞拿到 root，而是碰巧利用了测试内核中的另一个零日。朴素验证器只看最终是否获得高权限，因此把错误的因果归因当成了正确结果。团队随后收紧任务约束，并加入独立检查，验证每条利用是否真正使用指定漏洞。  
  
核心结论  
    
自主漏洞研究的瓶颈已经从“能否生成 PoC”转向“能否证明发现、利用和补丁围绕同一个根因闭环”。终点状态是结果证据，不能替代因果证据。  
# 把漏洞流水线拆成三类证据  
  
![](https://mmbiz.qpic.cn/mmbiz_png/sSvWYibwznrMPAYqKcp23avXPPA41GyLmeSYZylH4moEyh0Zb228KyJMucYw484195ibeH7PLNJ0lf52hDfvhXktPsfHIyRhGXrtLzqcSiajOs/640?wx_fmt=png "")  
  
图 1  
    
发现、利用与修复需要不同的判定器，不能共用一个“成功”布尔值。  
  
一套可靠流水线至少要同时维护发现证据、利用证据和修复证据。发现证据回答“目标提交上是否存在可重复触发的缺陷”；利用证据回答“这个缺陷是否能稳定地产生指定安全影响”；修复证据回答“候选补丁是否删除根因，同时没有引入新问题”。三个问题相关，却不能互相替代。  
<table><thead><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">阶段</span></span></p></td><td data-colwidth="178" width="178" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">输入与固定条件</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">最小证据</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">常见假阳性</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">门禁</span></span></p></td></tr></thead><tbody><tr><td data-colwidth="101" width="101" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">候选发现</span></span></p></td><td data-colwidth="178" width="178" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">目标提交、配置、编译器与静态线索</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">可定位的根因假设和触发前置条件</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">把不可达代码、已修复代码或重复问题当新发现</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">固定 commit 与 config；去重并查上游状态。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">动态复现</span></span></p></td><td data-colwidth="178" width="178" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">最小触发器、内核、超时与随机种子</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">稳定崩溃、告警或安全不变量破坏</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">环境噪声、无关告警、非确定性超时</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">多次复现；栈、地址归一化与输入哈希。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">可利用性</span></span></p></td><td data-colwidth="178" width="178" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">分配的漏洞与能力目标</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">获得 uid=0、读原语或写原语</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">命中同一内核中的其他漏洞</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">独立因果验证器，不接受终点状态单独过关。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">补丁</span></span></p></td><td data-colwidth="178" width="178" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">根因、触发器、影响版本与上游风格</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">PoC 失败、根因消失、变体受控</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">只屏蔽样例、扩大锁范围、带来性能或兼容退化</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">A/B、回退、构建、回归、风格与变体检查。</span></span></p></td></tr></tbody></table>  
   
  
OpenAI 报告中的 948 条 LPE 是原型环境里可工作的利用，不等同于 948 个已被上游接受的新 CVE，也不代表每条都对应互不重复的根因。报告同时说明，10 天扫描期间已有 61 条 LPE 被上游修复、43 条已有暂存补丁、299 条仍在讨论。扫描速度越快，去重、版本重放和维护者分流越容易成为新的瓶颈。  
# 终点 Oracle 为什么不够  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/sSvWYibwznrONkE0nfFvhTcXnlIZ6ROIqQic9QnxbGjdounf7oP3KibkWgpJeC0AVyI2ECtpQpBibntAG7XkZNGqe1lSF5RMk93Hyx5ibyM9iasnk/640?wx_fmt=png "")  
  
图 2  
    
在同时存在漏洞 A 与 B 的测试内核中，uid=0 无法告诉验证器实际经过哪条路径。  
  
漏洞利用实验经常使用清晰的终点 Oracle：进程是否获得 uid=0、是否读取内核地址、是否控制指令指针。它适合判断能力目标，却天然丢失路径信息。同一个测试内核可能包含多个未修复缺陷；堆喷、对象重用、竞态和未定义行为也可能把原本针对漏洞 A 的利用导向漏洞 B。  
  
这类问题不是简单的“模型幻觉”。即使利用代码完全可编译、重复执行也稳定成功，它仍可能在回答错误的问题。安全验证需要把“相关性”改造成“干预”：主动改变指定漏洞的存在状态，再观察利用是否随之改变。  
<table><thead><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">验证方法</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">操作</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">通过条件</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">能排除什么</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">局限</span></span></p></td></tr></thead><tbody><tr><td data-colwidth="101" width="101" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">补丁 A/B</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">同一配置分别运行漏洞版本和仅应用指定修复的版本</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">旧版本成功，修复版本失败</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">利用实际依赖其他未修复漏洞</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">补丁本身可能改变堆布局或时序。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">回退验证</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">在修复版本回退指定补丁后重跑</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">行为恢复，轨迹重新指向同一根因</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">一般环境变化和偶然失败</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">回退可能与后续提交冲突。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">触发点观测</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">对关键对象、检查点或 fault site 加 KASAN/KCSAN/tracepoint</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">崩溃、原语和终点可关联</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">仅凭 root 结果的黑盒归因</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">插桩会扰动竞态与时序。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">原语中和</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">关闭声称依赖的越界、UAF 或竞态窗口</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">利用失效且无替代路径</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">利用链偷换原语</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">需要准确的根因模型。</span></span></p></td></tr><tr><td data-colwidth="101" width="101" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">独立重建</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">验证器从 finding manifest 构建干净环境</span></span></p></td><td data-colwidth="144" width="144" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">哈希、提交、配置与步骤一致</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">生成器污染环境或选择性报告</span></span></p></td><td data-colwidth="168" width="168" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">计算成本和周转时间更高。</span></span></p></td></tr></tbody></table>  
   
# 一个可执行的因果验证协议  
  
下面的协议适合嵌入自动化平台。它不要求验证器理解完整利用代码，但要求生成器交付可重建的 manifest，并把“利用执行”和“结果判定”拆成两个权限域。  
## 第一步  固定实验身份  
  
每个 finding 必须绑定 target_commit、kernel_config、toolchain_digest、assigned_bug、trigger_hash 和预期能力。验证器从干净镜像构建，拒绝使用生成器留下的内核、模块或磁盘状态。对于竞态类问题，还要固定 CPU 数、调度参数、超时和重复次数。  
  
finding.json  
    
演示字段，所有哈希与提交均需由真实构建生成  
  
{  
  
    
"finding_id":  
   
"demo-uaf-042",  
  
    
"target_commit":  
   
"<40-char commit>",  
  
    
"kernel_config_sha256":  
   
"",  
  
    
"toolchain_digest":  
   
"sha256:",  
  
    
"assigned_bug":  
   
"object-X use-after-free at function-Y",  
  
    
"trigger_sha256":  
   
"",  
  
    
"oracle":  
   
{"type": "uid", "expected": 0},  
  
    
"runs":  
   
20,  
  
    
"required_checks":  
   
["baseline", "patched", "revert", "trace"]  
  
}  
  
   
## 第二步  同时记录终点和路径  
  
终点记录 uid、退出码和目标能力；路径记录触发栈、关键系统调用序列、KASAN/KCSAN 报告、对象分配与释放事件，以及利用原语出现的位置。地址受 ASLR 影响，比较时应转换为模块加符号加偏移，不能直接比较原始地址。  
## 第三步  运行四象限  
<table><thead><tr><td data-colwidth="110" width="110" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">实验</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">指定漏洞状态</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">利用结果</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">路径证据</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">判定</span></span></p></td></tr></thead><tbody><tr><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">基线</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">存在</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">成功</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">命中指定触发点</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">继续。</span></span></p></td></tr><tr><td data-colwidth="110" width="110" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">修复后</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">已应用最小修复</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">失败</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">指定触发点消失</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">支持因果关系。</span></span></p></td></tr><tr><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">修复后仍成功</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">已应用最小修复</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">成功</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">命中其他 fault site</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">拒绝归因，拆分新 finding。</span></span></p></td></tr><tr><td data-colwidth="110" width="110" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">回退后</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">重新存在</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">恢复成功</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">回到指定触发点</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">进一步支持因果关系。</span></span></p></td></tr><tr><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">修复后失败但轨迹缺失</span></span></p></td><td data-colwidth="139" width="139" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">已应用修复</span></span></p></td><td data-colwidth="110" width="110" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">失败</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">无足够路径信息</span></span></p></td><td data-colwidth="187" width="187" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">证据不足，不能自动通过。</span></span></p></td></tr></tbody></table>  
   
## 第四步  设置独立门禁  
  
验证器伪代码  
    
不执行真实提权，只说明门禁逻辑  
  
def causal_gate(result):  
  
      
assert  
   
result.baseline.oracle is True  
  
      
assert  
   
result.baseline.trace.matches_assigned_bug  
  
   
  
      
if  
   
result.patched.oracle is True:  
  
          
return  
   
"REJECT_WRONG_PATH"  
  
      
if  
   
result.patched.trace.matches_assigned_bug:  
  
          
return  
   
"REJECT_PATCH_INCOMPLETE"  
  
      
if  
   
result.reverted.oracle is not True:  
  
          
return  
   
"INCONCLUSIVE_NO_RESTORATION"  
  
      
if  
   
not result.reverted.trace.matches_assigned_bug:  
  
          
return  
   
"INCONCLUSIVE_TRACE_MISMATCH"  
  
      
return  
   
"PASS_CAUSAL_ATTRIBUTION"  
  
   
  
真实系统还应把统计稳定性纳入判定。例如基线 20 次成功 18 次、修复后 20 次成功 0 次，比单次 A/B 更有说服力；但不能把重复次数机械地当成因果证明。若补丁显著改变调度或内存布局，必须用更窄的插桩或原语中和实验排除混杂因素。  
# 防御性实验  两个缺陷同时存在时如何识别走错路径  
  
下面的 Python 例子只模拟状态机，不包含内核利用代码。环境同时包含 BUG_A 和 BUG_B；研究任务分配的是 BUG_A，但“利用”实际调用 BUG_B。朴素验证器只检查 uid，因此错误通过；因果验证器应用 fix_A 后重新执行，发现仍能获得 uid=0，于是拒绝归因。所有数据均为演示值，读者可在普通 Python 环境复核控制流。  
  
toy_causal_gate.py  
    
防御性教学示例，不对应真实漏洞  
  
from dataclasses import dataclass  
  
   
  
@dataclass  
  
class Lab:  
  
      
bug_a:  
   
bool = True  
  
      
bug_b:  
   
bool = True  
  
      
uid:  
   
int = 1000  
  
   
  
def claimed_exploit(lab: Lab):  
  
      
#  
   
任务声称验证 A，实际错误地依赖 B  
  
      
if  
   
lab.bug_b:  
  
          
lab.uid  
   
= 0  
  
      
return  
   
{"uid": lab.uid, "path": "BUG_B" if lab.uid == 0 else "NONE"}  
  
   
  
def naive_gate():  
  
      
return  
   
claimed_exploit(Lab())["uid"] == 0  
  
   
  
def causal_gate():  
  
      
baseline  
   
= claimed_exploit(Lab())  
  
      
patched_a  
   
= claimed_exploit(Lab(bug_a=False, bug_b=True))  
  
      
return  
   
baseline["uid"] == 0 and patched_a["uid"] != 0  
  
   
  
print(naive_gate())  
     
# True：错误通过  
  
print(causal_gate())  
    
# False：修复 A 后仍成功，归因被拒绝  
  
   
  
复核时再把 claimed_exploit 改成依赖 bug_a：修复 A 后 uid 应保持 1000，causal_gate 返回 True。随后增加回退实验，确认重新启用 A 后结果恢复。这个小实验说明了流水线设计的关键：验证器必须有能力改变被声称的原因，而不只是观察结果。  
# 补丁闭环  从“PoC 不再工作”走到可上游  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/sSvWYibwznrOvZMNzch4HriaB4dzPsP9cC4Kg5biaSNGTSH6jibhFicK9dWjmnKRAN9YpbjrP16h2zgOeeDWKjFicFOZz6NPF6siaTzAXjZnibAVcmM/640?wx_fmt=png "")  
  
图 3  
    
候选补丁要同时通过安全语义、工程质量和责任链检查。  
  
OpenAI 报告称，早期 AI 补丁经常能够阻止给定 PoC，却可能改动过大、带来性能折中、偏离内核风格，或遗漏同类变体。这正是“为测试而修”与“为根因而修”的区别。补丁生成器知道哪一个输入必须失败，维护者关心的是内核不变量是否恢复。  
## 先写不变量，再写补丁  
  
对 UAF，要说明对象生命周期和引用计数不变量；对越界访问，要说明长度来源、整数域和边界关系；对竞态，要说明锁、RCU、原子状态或 happens-before 约束。没有不变量说明的补丁，很难判断新增条件分支是在修根因，还是把当前 PoC 挡在门外。  
## 做近邻变体分析  
  
修复同一个函数后，应搜索同类对象的分配、释放和错误路径，检查是否还有相同状态迁移；对长度检查，应追踪所有调用者是否共享不可信长度；对并发缺陷，应检查其他读写点是否遵守同一同步协议。变体测试既可以从补丁 diff 生成，也可以从根因语义生成，两者都比仅对原 PoC 做回归更可靠。  
## 把内核工程流程作为发布门禁  
  
Linux 内核官方的 AI Coding Assistants 指南要求：报告要固定 commit，非简单问题提供 reproducer；修复后重新构建并验证，不得新增警告，并运行 checkpatch.pl；提交说明要包含充分的根因分析和 Fixes 标签。AI 工具不能代替人添加 Signed-off-by，最终仍由复核者承担责任。  
<table><thead><tr><td data-colwidth="125" width="125" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">检查项</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">最低完成标准</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">失败时动作</span></span></p></td></tr></thead><tbody><tr><td data-colwidth="125" width="125" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">安全语义</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">原 PoC 失败，根因不变量恢复，指定 fault site 消失</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">退回补丁生成，不允许只修改测试。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">差分与回退</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">目标提交、修复提交、回退提交的结果与轨迹一致</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">标记为证据不足或错误归因。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">变体</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">同源调用点与对象状态完成搜索并记录覆盖</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">拆分后续 finding，不宣称完整修复。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">构建与警告</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">目标架构构建成功，无新增 warning</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">不得进入维护者评审。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">风格与元数据</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">checkpatch、Fixes、维护者列表、影响版本齐全</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">修正提交信息与代码风格。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">性能与并发</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">热点路径有针对性基准，锁与时序变化被审查</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">缩小补丁或增加基准和并发验证。</span></span></p></td></tr><tr><td data-colwidth="125" width="125" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">责任链</span></span></p></td><td data-colwidth="398" width="398" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">人工复核并签署，披露未完成验证</span></span></p></td><td data-colwidth="221" width="221" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:8.0pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">不自动发送上游。</span></span></p></td></tr></tbody></table>  
   
# 检测与审计字段  让结果可重放  
  
平台不能只保存 exploit.c 和一个“PASS”。至少要能回答：在什么提交、配置和工具链上，哪一个触发器产生了什么轨迹；哪个补丁改变了什么；谁复核了因果关系。建议把一次 finding 的关键记录保存为不可变 manifest，并把大体积 trace 放在内容寻址存储中。  
  
audit.json  
    
演示审计结构，值均为占位符  
  
{  
  
    
"finding_id":  
   
"demo-uaf-042",  
  
    
"target_commit":  
   
"",  
  
    
"config_sha256":  
   
"",  
  
    
"assigned_bug":  
   
"",  
  
    
"reproducer_sha256":  
   
"",  
  
    
"exploit_sha256":  
   
"",  
  
    
"baseline":  
   
{"oracle": true, "trace_sha256": ""},  
  
    
"patch_commit":  
   
"",  
  
    
"patched":  
   
{"oracle": false, "trace_sha256": ""},  
  
    
"reverted":  
   
{"oracle": true, "trace_sha256": ""},  
  
    
"variant_set_sha256":  
   
"",  
  
    
"build_log_sha256":  
   
"",  
  
    
"reviewer":  
   
"",  
  
    
"limitations":  
   
["race-sensitive", "not tested on arm64"]  
  
}  
  
   
  
这些字段也能支持检测工程。若 patched.oracle 仍为 true，应立即生成“错误路径或补丁不完整”告警；若 baseline.trace 与 reverted.trace 的 fault site 不一致，应标记归因不稳定；若结果在不同 config 下消失，则影响范围必须收窄。  
# 规模化之后  人的工作没有消失，只是换了位置  
  
OpenAI 报告称，10 月中旬一次运行消耗了约 121.3 万美元 API credits，平均每条可工作 LPE 约 1,279 美元。这个数字反映原型运行成本，不等于每条漏洞的真实发现成本，更不能直接换算为赏金或修复收益。独立重建、人工去重、维护者沟通、回移和长期回归都在数字之外。  
  
当候选从几十条增长到上千条，安全团队最需要的不是更快地产生报告，而是优先级和证据预算：高影响且高因果置信度的条目先进入人工复核；重复、已修复或归因不稳定的条目自动降级；所有发往维护者的报告都附带最小复现、版本边界、候选补丁和验证限制。  
<table><thead><tr><td data-colwidth="120" width="120" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">队列</span></span></p></td><td data-colwidth="254" width="254" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">进入条件</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">人工动作</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(23, 54, 93);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#FFFFFF;font-weight:bold;"><span leaf="">退出标准</span></span></p></td></tr></thead><tbody><tr><td data-colwidth="120" width="120" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">P0 可披露</span></span></p></td><td data-colwidth="254" width="254" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">稳定复现、明确根因、因果门禁通过、高影响</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">复核代码、影响面、补丁与协调披露</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">维护者确认或给出可审计反馈。</span></span></p></td></tr><tr><td data-colwidth="120" width="120" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">P1 待验证</span></span></p></td><td data-colwidth="254" width="254" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">复现稳定但因果或版本边界不足</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">补做 patch/revert、插桩和 config 矩阵</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">升级 P0 或降级。</span></span></p></td></tr><tr><td data-colwidth="120" width="120" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">P2 重复/在修</span></span></p></td><td data-colwidth="254" width="254" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">与上游补丁或讨论高度相似</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">确认 commit、分支与回移状态</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">关联现有条目，不重复轰炸。</span></span></p></td></tr><tr><td data-colwidth="120" width="120" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;text-align:center;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;mso-fareast-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">P3 不稳定</span></span></p></td><td data-colwidth="254" width="254" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">低复现率、轨迹漂移或只剩终点证据</span></span></p></td><td data-colwidth="211" width="211" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">保存样本，限制计算预算</span></span></p></td><td data-colwidth="163" width="163" valign="middle" style="background: rgb(245, 247, 250);border-width: 1pt;border-style: solid;border-color: rgb(217, 217, 217);padding: 5.25pt 6pt;"><p style="margin-bottom:0.0pt;"><span style="font-size:7.5pt;line-height:122%;font-family:Noto Sans CJK SC;mso-ascii-font-family:Noto Sans CJK SC;font-variant:normal;text-transform:none;color:#222222;"><span leaf="">达到稳定阈值或归档。</span></span></p></td></tr></tbody></table>  
   
# 适用边界与需要保留的怀疑  
  
第一，公开报告没有披露 948 条利用的完整技术细节，也没有提供逐条独立复核材料。关于“至少 10 种新利用技术”的说法来自厂商对抽样结果的判断，应标为厂商主张，不能当作外部已验证结论。  
  
第二，20/200 的错误路径来自早期运行，报告称后续已经通过更严格约束和独立验证修正。它不是当前系统仍保持 10% 错误率的证据，也不能外推到所有漏洞研究 Agent。这个案例证明的是失败模式真实存在，以及验证设计需要因果干预。  
  
第三，补丁 A/B 也不是绝对证明。补丁可能改变堆布局、锁竞争或代码生成，使利用因环境扰动失败。对于敏感竞态，应结合回退、触发点插桩、原语中和和多配置重放，形成多条相互独立的证据。  
  
第四，自动化越强，越需要限制主动利用环境。测试应在授权、隔离、可销毁的内核和基础设施中进行；验证器只接收内容寻址的输入，并禁止把生成器的网络、凭据和持久化状态带入复核环境。  
# 结语  把“成功”改写成可反驳的证据  
  
AI 可以把漏洞发现和利用的吞吐量推到维护流程从未面对过的规模，但 uid=0、弹出 shell 或让 PoC 停止崩溃，都只是局部结果。可靠的系统必须允许验证者提出反事实问题：如果指定漏洞被修复，这条路径是否消失；如果修复被回退，它是否恢复；如果只改变环境，它是否仍然指向同一个根因。  
  
真正可交付的成果不是一段“能跑”的利用，而是一条可重放、可干预、可否证、有人负责的证据链。只有当发现、利用和补丁围绕同一个根因闭环，自动化带来的速度才不会转化成维护者的噪声。  
  
