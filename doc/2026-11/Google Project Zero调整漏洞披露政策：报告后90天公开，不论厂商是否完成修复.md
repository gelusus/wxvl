#  Google Project Zero调整漏洞披露政策：报告后90天公开，不论厂商是否完成修复  
 看雪学苑   2026-11-16 10:00  
  
# Google旗下安全研究团队 Project Zero 宣布调整漏洞披露政策。根据新规则，团队向软件厂商提交漏洞报告后，将在规定期限届满时公开漏洞技术细节，不再以厂商是否完成修复作为公开条件。  
  
  
#   
  
新政策名为  
Full X，其中 X 代表漏洞报告提交后到公开披露之间的天数。披露期限将分阶段缩短：目前为90天，自2027年2月2日起缩短至60天，自2028年1月3日起进一步缩短至30天。  
  
## 取消原有的“90+30”披露机制  
  
自2021年以来，Project Zero一直采用“90+30”漏洞披露规则。厂商在收到漏洞报告后，通常有90天时间完成修复，随后还有30天的额外缓冲期，用于让安全补丁逐步推送至用户。  
  
  
此次政策调整取消了额外的30天缓冲期。按照新规则，即使厂商未能在规定期限内完成漏洞修复，Project Zero仍会按计划公开漏洞技术细节。  
  
## LLM技术发展推动披露期限调整  
  
Project Zero研究人员Natalie Silvanovich在政策说明中介绍，此次调整主要基于三个方面的变化。  
  
1. LLM降低了漏洞发现和利用门槛  
  
大型语言模型（LLM）的发展，使攻击者能够更快地发现漏洞并构建利用方式。同时，在漏洞尚未公开的保密期内，其他研究人员或恶意攻击者发现相同漏洞的可能性也有所增加。  
  
Project Zero认为，与过去相比，新发现漏洞在报告提交后可能面临更高的即时安全风险。  
  
2. 补丁分析和漏洞利用速度加快  
  
LLM也缩短了攻击者从安全补丁中分析漏洞成因、构造利用程序所需的时间。  
  
Project Zero指出，补丁本身可能已经暴露漏洞所在的位置。攻击者可以通过分析补丁前后的代码差异，定位漏洞并尝试构建利用程序。  
  
因此，团队在制定新政策时，将攻击者能够在补丁发布后立即开展漏洞利用，作为一种合理但并不完全准确的风险模型。  
  
Project Zero同时提出，软件厂商需要缩短补丁发布与用户实际获得安全更新之间的时间差，也就是 Patch Gap。厂商可以通过加快补丁推送，或者在正式修复完成前采取其他临时防护措施，降低用户面临的风险。  
  
3. 漏洞发现数量明显增加  
  
Project Zero表示，目前团队每周都会在部分研究目标中发现多个漏洞，而过去同类目标可能一年才发现少量漏洞。  
  
随着漏洞报告数量增加，团队需要与厂商反复沟通漏洞是否已经修复、是否适用延期等问题，原有披露机制的执行成本也随之上升。  
##   
  
  
## 新政策保留三种例外情况  
  
  
##   
  
在取消大部分特殊处理规则后，Project Zero仍保留三种例外：  
- 已在野外遭到利用的漏洞： 此类漏洞按照Google应用安全漏洞披露政策处理。  
  
- 漏洞信息已经被其他渠道公开： 如果其他研究人员或机构已经公开漏洞细节，Project Zero可以自行决定是否提前发布相关技术信息。  
  
- 厂商主动提出公开请求： 厂商可以要求Project Zero公开此前提交给该厂商的漏洞技术细节。  
  
对于硬件漏洞，Project Zero仍可能根据具体情况给予额外时间。此前，Spectre和Meltdown等漏洞就曾获得特殊披露安排。  
  
  
此外，Project Zero要求厂商提前制定严重漏洞即将公开时的应对计划。相关措施不局限于发布软件补丁，也可以包括其他能够保护用户的技术方案。  
  
## 漏洞严重程度不再影响披露日期  
  
新政策明确，Project Zero不会根据漏洞严重程度调整披露时间。  
  
  
团队表示，其提交的漏洞报告通常需要具备实质性的安全影响，并能够证明漏洞的实际安全风险。不过，漏洞分析和影响评估也可能存在一定误差。  
  
  
Project Zero认为，漏洞严重程度主要影响漏洞对用户造成的潜在损害，并不直接决定攻击者发现漏洞的可能性。  
  
  
因此，即使厂商认为某个漏洞风险较低，也不能据此要求调整披露日期。例如，对于无法被实际利用的空指针崩溃问题，厂商可以根据自身产品情况判断是否接受相关风险，但不会因此改变Project Zero设定的公开时间。  
  
  
这一政策同时要求漏洞报告满足可复现性、可读性、可达性和实际影响等基本标准。对于不符合这些要求的低质量报告，厂商可以采用不同的处理方式。  
  
## 披露期限分阶段缩短至30天  
  
在正式确定新政策之前，Project Zero曾向经常接收其漏洞报告的软件厂商征求意见。  
  
厂商主要提出了三方面顾虑：  
- 如果突然缩短披露期限，可能导致尚未修复的漏洞提前公开，而厂商自身的漏洞修复流程还没有完成调整。  
  
- 更严格的披露期限可能促使厂商优先处理Project Zero提交的漏洞，而忽略其他风险更高的漏洞报告。  
  
- 如果厂商无法满足新的期限要求，可能降低其参与漏洞修复工作的积极性。  
  
针对这些反馈，Project Zero决定采取分阶段调整的方式，为厂商留出适应时间。  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">生效时间</span></section></th><th style="text-align: left;"><section><span leaf="">漏洞报告后公开期限</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">当前政策</span></section></td><td style="text-align: left;"><section><span leaf="">90天</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">2027年2月2日</span></section></td><td style="text-align: left;"><section><span leaf="">60天</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">2028年1月3日</span></section></td><td style="text-align: left;"><section><span leaf="">30天</span></section></td></tr></tbody></table>  
  
以上日期均为漏洞报告提交后的披露期限，不包含额外的补丁推送缓冲期。  
  
## 是否完成修复，不再作为披露决策条件  
  
Project Zero过去在判断漏洞是否已经修复时，经常需要与厂商进行多轮沟通。尤其是对于没有直接交付给最终用户，或者无法公开访问的代码，判断修复状态存在一定困难。  
  
  
随着软件厂商开始采用更多非传统修复方式，例如临时禁用存在风险的代码路径，而不是直接发布补丁，漏洞修复状态的判定也变得更加复杂。  
  
  
针对这一问题，Project Zero表示，团队仍会跟踪漏洞修复情况，并向外界说明漏洞是否已经得到修复，但这一判断不再影响漏洞披露时间。  
  
  
按照新政策，Project Zero可以在公开漏洞技术细节时，同时说明厂商采取了哪些措施保护用户，而不需要先对漏洞是否修复作出二元判断。  
  
  
从2028年1月3日起，Project Zero提交的漏洞报告将在30天后公开，无论厂商是否已经完成修复，也不再额外给予补丁推送缓冲期。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K0DiaAGh6hvmxRqfUpVibCBPsibhWEHGcGq2nya9Fdic26xX7OHQw9IYews5GviamZOHruNqtqwtvicD5VmuDQibRShssFicdOZXwgjGs0/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K1DiaZWR5upCSYJkVBEbOPqCjicPlhoeBd1r42Kibetp6t8goaBO7ZlGibGptxmKQ7AJqQqD3zDvDYrvFR4LNbE8s05LhMia6Z2WMic4/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0BS3CnlGyX54xvo7eTEACZKKib8UsPrbJ89n6icTYj3D6vpnXAcfE9FDeRjVsR44CfUWe1omkqnSicJwKeEsMNsU8eqsSuiaFXDZk/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K22A7ic9VJqSQpTcLdka5YaJbiaN94bibC3Tq3SzY5MIHcdS3iaGUFQxDuBna4pta3ico0Q7CMFFy3y8lMO9JJ5o8Kibr71WoS7cIyzk/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
