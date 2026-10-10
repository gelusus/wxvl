#  Anthropic启动网络使命计划，聚焦漏洞修复助力安全防御  
 FreeBuf   2026-10-10 10:10  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX2beB3lax8xBWtias4UKKEYQBaE3qWstibASCMCPbUQd4ewGPEFDKD9VxGfseFeHZtfpCFjxRdK0u07eWCfsb3psYSWLdeDic0eTE/640?wx_fmt=gif "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX21uAsBnpczhqvdL1pZHKIjKq7wjHIAJp1hibf4IZN9O7VyYAne5HyZv2w7IdjpxlmJoh39w9BR9gcv3hucnceBFYCB0Xsr2apk/640?wx_fmt=png&from=appmsg "")  
  
  
Anthropic于2026年10月8日正式启动网络使命计划（Cyber Mission），帮助安全团队防护关键基础设施与开源软件。该项目整合了先进的Claude模型、工程师团队、威胁研究成果与配套资金。项目核心方向是推动漏洞修复，而非仅输出更多漏洞报告。  
  
  
本次发布同步推出关键基础设施防御计划以及免费开源软件扫描服务（OSS Scanner），面向主动接入的开源项目开放。  
  
  
此次网络使命计划是在“玻璃之翼计划”（Project Glasswing）基础上升级而来。此前“玻璃之翼计划”提升了漏洞发现速度，却暴露出一个难题：安全人员核验漏洞、确定修复优先级、部署补丁仍然需要大量时间与专业人力。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0VQNTYp3XibHl6ib12fU3312dg5CIPcjNxia8yiaUXdialmVic8wSt7LI2Q8IAVBfs94qjnGoD5YDVSlHDdSvhx28SchugI0u2VHHgk/640?wx_fmt=png&from=appmsg "")  
  
  
Part  
01  
  
关键基础设施防御计划启动  
  
  
关键基础设施防御计划将Claude模型、驻场工程师、威胁研究能力输出给可信服务商，覆盖电网、水务、工厂、交通网络与政务系统场景。目前项目共有11家创始合作伙伴，包括埃森哲、博思艾伦、CrowdStrike、德勤、Dragos、日立、Insane Cyber、Nozomi Networks、派拓网络、普华永道、罗克韦尔自动化。  
  
  
这些服务商主要支撑运营技术（OT）系统运行，覆盖工业控制器、控制软件以及操控物理设备的网络。很多OT系统已经连续运行数十年，无法停机开展常规更新。未经充分测试的变更可能中断生产或核心服务，因此这类环境的补丁部署难度远高于普通商业网络。  
  
  
目前，已有多家合作伙伴利用Claude协助客户修复漏洞、处理安全问题。首批参与机构将进一步测试相关方案在生产环境中的安全性和可行性。不过，人工智能并不能解决所有基础设施安全问题，尤其是在设备条件、维护周期和生产安全要求限制修复方案的情况下，仍需要结合实际环境制定防护措施。  
  
  
此外，自2026年6月以来，Anthropic面向政府的防御计划已向美国超过半数的州提供Claude模型和技术支持，涉及代码扫描、漏洞修复、安全事件响应及其他公共部门网络安全工作。  
  
  
Part  
02  
  
免费开源扫描工具上线  
  
  
新推出的开源软件扫描工具将调用Anthropic目前能力最强的模型，为主动申请接入的开源项目提供定期安全扫描。项目维护者收到的报告将包含漏洞原理、漏洞利用概念验证（PoC）以及可直接使用的补丁方案。与Anthropic现有的漏洞披露流程不同，这类报告输出前不经过人工审核。  
  
  
这种模式加快了报告交付速度，但需要维护者自行核验报告的准确性与漏洞影响范围。Anthropic预计报告的真阳性率高于90%，但漏洞严重等级评级可能存在偏差。没有足够人力核验原始扫描结果的项目，仍可通过Anthropic的协同漏洞披露流程接收人工核验后的报告。  
  
  
此前的扫描数据也反映出人工审核面临的压力。Anthropic公布的数据显示，半年扫描周期内共发现超过2.9万个候选漏洞，但其中仅约6000个经过人工审核与分诊。维护者不能将这些候选结果直接认定为已确认漏洞，也不代表对应漏洞已经完成修复。  
  
  
Part  
03  
  
整合原有项目与资源  
  
聚焦实战化防护落地  
  
  
此前，Anthropic正在扩大“玻璃之翼计划”的应用范围，尝试从漏洞检测延伸到补丁修复等其他防御任务。本次网络使命计划延续了这一方向，在开放模型能力的同时，为项目维护者提供工程支持与配套资源。  
  
  
目前“玻璃之翼计划”已并入升级后的网络安全验证计划。该计划设置了三个访问权限等级，分别面向安全防御、授权渗透测试以及安全关键系统的受限测试。申请者需要根据实际工作类型完成相应的资质审核，并满足相关安全管理要求。  
  
  
防御者优势基金（Defender Advantage Fund）将为试点项目提供支持，同时保障开源软件扫描工具永久免费。后续，Anthropic计划进一步拓展关键基础设施领域的合作伙伴关系，持续优化自动化分诊与补丁修复能力。除此之外，团队还将开展更安全软件设计方向的研究。  
  
  
整个计划的目标，是将人工智能的漏洞发现能力转化为实际防护成效。衡量成果的重点也将落在可利用漏洞数量是否减少、关键服务能否保持稳定，以及遭受攻击后能否更快恢复运行。  
  
  
参考来源：  
  
Anthropic Cyber Mission to Support Defenders with Tools, Research, and Resources  
  
https://cybersecuritynews.com/anthropic-cyber-mission/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651348384&idx=1&sn=b0d043fbf6f272983df964e3f249232e&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
