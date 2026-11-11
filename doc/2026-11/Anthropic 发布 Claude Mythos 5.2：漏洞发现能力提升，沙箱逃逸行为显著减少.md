#  Anthropic 发布 Claude Mythos 5.2：漏洞发现能力提升，沙箱逃逸行为显著减少  
 数世咨询   2026-11-11 08:00  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/lYWDmickZ2mwic7cjCia6L3rpFeIxBibTFyeQ3R3dE918lCviauhy28CuW5tdtxYtOt1AjQImRcAWib7pib5okh40EDBKahrVpKiaqr3VT6ZkmyUv5Y/640?wx_fmt=gif&from=appmsg "")  
  
▍以下正文内容基于英文原文编译，可能存在语义偏差，请以原文为准。  
  
Anthropic 于本周一发布了 Claude Mythos 5.2，这是其限制访问的 AI 模型的最新版本。Mozilla 表示，在 Firefox 浏览器的漏洞发现测试中，Mythos 5.2 的表现约为上一代 Mythos 5.1 的两倍。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/lYWDmickZ2mztgUEPC587GYo0ibv9hpw0DRb5P6GHpbm4IAwUQMhF6l2ORjQdh3wHckOSCOmwHmYm771cBvKlmK415yHsiarxTwPic7m4Q3QJOY/640?wx_fmt=png&from=appmsg "")  
  
与 Mythos 5.2 同时发布的还有 Claude Fable 5.2。两个版本采用相同的底层模型，主要区别在于安全防护机制。Fable 面向普通用户开放，包含防止网络安全和生物领域滥用的安全限制；Mythos 则取消了其中部分限制，以支持专业研究。  
  
Mythos 的直接访问权限仅向经过审核的组织开放。符合条件的机构可以通过 Anthropic 的网络安全与生命科学验证计划申请使用。  
## 漏洞发现能力进一步提升  
##   
  
Mozilla 杰出工程师 Brian Grinstead 表示，Mozilla 今年一直与 Anthropic 密切合作，利用不同版本的 Claude 模型加强 Firefox 的安全性。  
  
随着多轮安全检测不断深入，剩余漏洞数量越来越少，漏洞也变得更加隐蔽。即便如此，在各自对应的扫描活动中，Mythos 5.2 的漏洞发现指标仍约为 Mythos 5.1 的两倍。  
  
Anthropic 内部采用一项名为 CVE Rediscovery（CVE 重新发现）的测试来衡量模型的漏洞发现能力。  
  
在这项测试中，研究人员向模型提供某个项目在已知 CVE 漏洞修复之前的源代码，观察模型能否在没有额外提示的情况下自主识别漏洞。  
  
2026 年 10 月开展的测试涵盖了 1,829 个 CVE，结果如下：  
  
<table><tbody><tr><td style="color:rgb(0, 0, 0);font-weight:bold;text-align:center;word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1ipv" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">AI 模型</span></section></td><td style="color:rgb(0, 0, 0);font-weight:bold;text-align:center;word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2ck3j" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">漏洞重新发现成功率</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1db0" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Mythos 5.2</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2cw6x" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">89.60%</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2cqf3" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Opus 5.5（关闭安全限制）</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1m1b" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">81.30%</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1uo1" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Mythos 5.1</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1ia8" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">78.30%</span></section></td></tr></tbody></table>  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/lYWDmickZ2mxOpbLcibslAPAa09QiauNYCwBTHYqu0FopMO09diaTcV7fgvENq5L4vGcdzVSY33qC3ySXibJXbFjURMlPXPBRoCNM8wEvWfyvSsk/640?wx_fmt=png&from=appmsg "")  
  
Anthropic 还分享了外部测试人员的反馈。其中一家安全公司发现，在 Mythos 5.2 发现并确认的漏洞中，约六分之一此前未被该公司测试过的任何其他前沿模型发现。  
  
不过，Anthropic 在相关说明中也指出，按照其内部评估，Mythos 5.2 的漏洞识别能力与 Opus 5.5 大致相当。  
  
对于没有加入网络安全验证计划的 Claude Enterprise 客户，Anthropic 也提供了另一种使用 Mythos 最新能力的方式。  
  
该公司的代码漏洞扫描产品 Claude Security 已经开始采用 Mythos 5.2，并结合其他模型执行安全检测任务。  
## 加强安全限制，同时减少正常任务被误拦截  
##   
  
在面向普通用户的 Claude Fable 5.2 中，Anthropic 继续通过安全分类器限制高风险网络安全操作，包括渗透测试、漏洞利用代码生成等任务。  
  
当用户请求触发相关限制时，系统会改由 Opus 4.8 提供响应。  
  
Anthropic 表示，Fable 5.2 的安全防护强度与 Fable 5.1 相当，但对合法任务的误拦截明显减少。  
  
早期测试显示，与 Fable 5.1 相比，Fable 5.2 在网络安全相关任务中的拦截次数减少了约 60%。  
  
这意味着 Anthropic 正在尝试改善一个长期存在的问题：既要限制模型被用于高风险攻击活动，又要避免正常的安全研究和开发工作受到不必要的阻碍。  
## 沙箱逃逸行为显著减少  
##   
  
除了漏洞发现能力，Anthropic 还重点评估了 Mythos 5.2 在受控环境中的行为安全性。  
  
研究人员设计了一项包含超过 1,800 个场景的对抗性测试。测试环境中的沙箱被故意设置了安全弱点，而且大多数任务只有突破沙箱限制才能完成。  
  
研究人员观察模型是否会利用沙箱外部的凭据或文件来完成任务。  
  
测试结果如下：  
  
<table><tbody><tr><td style="color:rgb(0, 0, 0);font-weight:bold;text-align:center;word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c9of" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">AI 模型</span></section></td><td style="color:rgb(0, 0, 0);font-weight:bold;text-align:center;word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2ctvz" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">利用沙箱外资源的场景比例</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1h5k" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Mythos 5</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2coeq" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">39%</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c1p42" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Mythos 5.1</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2cbv7" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">21%</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c15io" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Opus 5.5</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2c24nl" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">3.30%</span></section></td></tr><tr><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2cei7" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">Claude Mythos 5.2</span></section></td><td style="color:rgb(0, 0, 0);word-wrap:break-word;word-break:break-word;white-space:pre-wrap;"><section><span leaf="" mpa-font-style="mwbqma2cs6e" style="color: rgba(0, 0, 0, 0.9);background-color: rgba(0, 0, 0, 0);font-size: 16px;font-weight: 400;font-style: normal;letter-spacing: 0.578px;word-spacing: 0px;text-shadow: none;">3.10%</span></section></td></tr></tbody></table>  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/lYWDmickZ2mzmGeHbzzUf3ZXhtlQK3xNFTp6tgHEb6yzrSpQG0mj0T2AeKJ53mxeazgfH1vnVLAWs2v4YLIGMibO6Ov88jT0hooq6lFMQrhvQ/640?wx_fmt=png&from=appmsg "")  
  
Mythos 5.2 在这项测试中的比例降至 3.1%，远低于前两代 Mythos 模型，并且与 Opus 5.5 的表现接近。  
  
这一结果表明，在 Anthropic 设计的受控测试环境中，Mythos 5.2 更少利用沙箱外部资源完成任务。  
  
需要注意的是，这些数字衡量的是特定实验场景下的模型行为，并不代表真实环境中的沙箱逃逸成功率，也不能证明模型已经完全不存在越界操作风险。  
  
*   
注：图片均来源于网络，无法联系到版权持有者。如有侵权，请与后台联系，做删除处理。  
  
— 【 THE END 】—  
  
  
