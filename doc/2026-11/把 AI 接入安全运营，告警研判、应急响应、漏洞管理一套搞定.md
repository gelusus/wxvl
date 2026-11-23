#  把 AI 接入安全运营，告警研判、应急响应、漏洞管理一套搞定  
原创 菜狗
                    菜狗  只会看监控的实习生   2026-11-23 08:30  
  
做安全运营的应该都遇到过类似的问题：  
  
告警在一个平台，资产信息在另一个平台，漏洞台账又是单独维护的。遇到安全事件时，还得手动查日志、找证据、整理报告，最后再把整改情况录入系统。  
  
如果能把这些工作整合到一个平台里，再结合 AI 辅助分析，日常运营效率会不会更高？  
  
今天分享一个开源项目：**御枢（AegisOps）**  
。  
  
它是一套 AI 安全运营工作台，将大模型、智能体、工具和工作流结合起来，主要用于告警研判、应急响应、漏洞整改以及威胁情报管理。  
  
**项目地址：**  
 https://github.com/knot07/AegisOps  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Pled5HYvsFG7Tn4BUmg3iafIfOER3z1JCwLWib9ibygI4GsZtQicicnUC8gtgic5mKgt2LNnYYHyG688GUSoianW0sUfVPLBvDRHuua3WPZFtnxCn4/640?wx_fmt=webp&from=appmsg "")  
## 一、项目能做什么？  
  
AegisOps 并不是简单套了一层 AI 对话界面，而是围绕安全运营中的几个实际工作场景设计了不同的功能模块。  
  
目前主要包括四个方向：  
- AI 告警研判  
  
- 应急响应与取证  
  
- 漏洞运营管理  
  
- 威胁情报采集与资产核查  
  
下面分别看看。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Pled5HYvsFGfGeMUjAxkAfxMeZuD63EO1bIHN5icWZs0h8WxPMy66wc5gE8ZsMcZaX6OCYw0VnvFIGLTlLxQXkpX7FH4GSd9lzLxcYAia8sWg/640?wx_fmt=webp&from=appmsg "")  
## 二、核心功能介绍  
### 1. AI 告警研判  
  
安全设备产生告警后，可以进入平台进行进一步研判。  
  
平台保留告警来源设备和原始线索，方便后续追踪。  
  
经过人工确认后，可以将确认的安全事件转入应急响应流程，也可以将发现的漏洞转入漏洞管理模块。  
  
这里有一点值得注意：项目区分了演示环境和正式设备接入。想要接入自己的安全设备，需要根据实际 API 和日志格式进行适配，并不是修改一下设备名称就能直接使用。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Pled5HYvsFGkjz7jZiaQyFFVXxjlr5DicrBzRtzLibibDz1r5McjROfl8rgkP1xibSLgYibVvaeYsYJ0yct6C4L46MqcPsHGhasL1icRDK1o65VUSQ/640?wx_fmt=webp&from=appmsg "")  
### 2. 应急响应与取证  
  
这是比较值得关注的一个模块。  
  
项目将应急响应过程拆分为几个环节：  
1. 接收安全事件  
  
1. 关联来源设备日志  
  
1. 对授权主机进行排查  
  
1. 收集文件及相关证据  
  
1. 分析调查结果  
  
1. 生成证据报告  
  
1. 人工复核  
  
远程主机排查方面：  
- Linux：支持 SSH 连接。  
  
- Windows：支持 WinRM HTTPS。  
  
AI 可以根据已经收集到的证据继续辅助调查，报告中也会引用对应的证据编号。  
  
这样做的好处是，调查结论能够关联实际证据，而不是直接把模型生成的推测当成已经发生的事实。  
  
另外，项目还集成了 Solar 辅助取证能力。  
  
样本分析方面，支持静态初筛，也可以按需使用 REA、Ghidra 进行静态逆向分析，并支持微步、VirusTotal 查询。  
  
需要说明的是：它并不提供恶意样本动态沙箱，也不能把简单的字符串匹配当成完整的逆向分析。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Pled5HYvsFEHydAVL58gQy9eXVAoadCibxBXtZZt6Qym3s3csJNRv0yPF69Mz85LXN2xzb4ibngkXskGcNT8gnPfUTpgVHbj4rDia4zePBtZiag/640?wx_fmt=webp&from=appmsg "")  
### 3. 漏洞运营管理  
  
漏洞管理也是 AegisOps 的一个重要模块。  
  
它提供了统一的资产目录和漏洞台账，可以围绕漏洞从发现到归档进行管理。  
  
主要功能包括：  
- 漏洞信息录入与编辑  
  
- 漏洞与资产关联  
  
- 漏洞整改任务分派  
  
- 整改进度跟踪  
  
- 提交复测  
  
- 历史记录归档  
  
对于甲方安全运营人员来说，这类功能比较实用。  
  
因为漏洞管理并不是扫描出漏洞就结束了，真正重要的是后续有没有人处理、有没有完成整改、复测是否通过，以及整个过程能不能追溯。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Pled5HYvsFEn09bFKWoHY0wKjSrNXfDWnhHH7hT01JjF988IicBkKO2taYDeRAiamLqXMAIBphm7a7YPZlEKVw2kD27KfcZ4pmlIKyKzo7JQE/640?wx_fmt=webp&from=appmsg "")  
### 4. 威胁情报管理  
  
项目还提供了威胁情报采集入口。  
  
目前 README 列出的情报来源包括：  
- 微步  
  
- 阿里云  
  
- 斗象  
  
- 长亭  
  
- NVD  
  
- Tenable  
  
- CISA  
  
进入对应来源后，可以根据日期、风险等级和关键词筛选公告，同时保留原始公告链接，并提供模型辅助中文翻译。  
  
除此之外，Atlas 还可以用于已有资产的漏洞影响核查。  
  
例如，拿到一条新的漏洞情报后，可以结合资产产品、版本和公告信息，辅助判断当前资产是否可能受到影响。  
  
平台会区分：  
- 已确认  
  
- 可能受影响  
  
- 已排除  
  
- 资料不足  
  
这里有个细节：产品名称匹配只能作为排查线索，不能直接认定资产存在漏洞，仍然需要结合版本和实际环境进行人工核实。  
## 三、Atlas：把模型、工具和工作流连接起来  
  
除了上面几个业务模块，项目还提供了 Atlas 相关能力。  
  
可以将它理解为平台中的 AI 协作入口。  
  
它能够结合模型、智能体、技能、工具、知识库和工作流完成不同任务。  
  
项目还提供了：  
  
<table><thead><tr><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">功能模块</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">主要用途</span></section></th></tr></thead><tbody><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">原生对话</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">通过模型和智能体进行交互</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">任务中心</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">查看执行记录、工作流任务和定时计划</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">智能体</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">管理智能体、角色及模型</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">技能库</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">按安全场景组织技能</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">工作流</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">将重复操作整理成可复用流程</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">知识库</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">管理知识资料和检索内容</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">工具清单</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">查看 API、设备、取证及逆向等工具</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">设备接入</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">配置安全设备和主机连接</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">模型管理</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">配置模型供应商及默认模型</span></section></td></tr></tbody></table>  
  
对于想研究 AI Agent 和安全运营结合的朋友来说，这部分也值得看看。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Pled5HYvsFGNXCePEoBCT6nnaN8hHdWTdVDMtdfOt8xhzKIlGtibtVkYjwqfL8ic1MjtuxyHURL46biaPHKeezLBXyiaVIFc45gKpE41EFMLWo0/640?wx_fmt=webp&from=appmsg "")  
## 四、项目部署方式  
  
项目提供了 Windows、Linux 和源码三种主要下载方式。  
  
<table><thead><tr><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">版本</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">适用环境</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">使用方式</span></section></th></tr></thead><tbody><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">Windows x64</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">Windows 10/11</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">解压后运行 </span><code><span leaf="">start.cmd</span></code></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">Linux</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">Linux / WSL2 / Docker</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">使用 </span><code><span leaf="">install.sh</span></code><span leaf=""> 或 Docker Compose</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">源码</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">二次开发</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">下载源码自行构建</span></section></td></tr></tbody></table>  
### Linux 部署  
  
克隆项目：  
```
git clone https://github.com/knot07/AegisOps.git

```  
  
进入项目目录：  
```
cd AegisOps

```  
  
执行安装脚本：  
```
bash install.sh

```  
  
也可以参考项目提供的 Docker Compose 部署方式。  
  
注意：以上是项目 README 提供的基础部署入口，实际启动参数、环境依赖及配置要求建议以仓库中的 DEPLOY.md  
 和部署手册为准。  
### 部署时需要注意  
1. 首次部署需要自行设置管理员账号和密码。  
  
1. 模型供应商及 API Key 需要自行配置。  
  
1. 安全设备和主机连接需要根据实际环境配置。  
  
1. Windows、Linux 的部分依赖会在首次启动时安装。  
  
1. 更新已有环境前，建议备份 runtime  
 数据目录。  
  
另外，REA、Ghidra 和 JDK 离线环境不包含在主发行包中，需要根据实际需求单独准备。  
## 五、使用时需要注意什么？  
  
这个项目虽然引入了 AI，但并不意味着可以完全交给模型自动处置安全事件。  
  
有几个地方需要特别注意：  
- AI 的分析结果需要结合原始日志和证据进行复核。  
  
- 威胁情报命中不代表资产一定存在漏洞。  
  
- 远程主机调查应当限定在已经登记并获得授权的资产范围内。  
  
- 样本上传第三方平台前，需要明确确认。  
  
- 设备正式接入需要适配真实 API 和日志格式。  
  
尤其是应急响应场景，AI 可以辅助分析、整理线索和生成报告，但最终的事件定性与处置操作仍然需要人工把关。  
## 六、写在最后  
  
AegisOps 比较有意思的地方，在于它尝试把 AI 能力放进安全运营的实际流程中，而不只是提供一个聊天窗口。  
  
从告警研判、主机排查，到证据整理、漏洞整改和威胁情报核查，基本覆盖了安全运营中几个比较常见的工作环节。  
  
**项目地址：**  
  
https://github.com/knot07/AegisOps  
  
  
