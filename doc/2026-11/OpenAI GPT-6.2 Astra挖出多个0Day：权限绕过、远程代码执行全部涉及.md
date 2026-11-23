#  OpenAI GPT-6.2 Astra挖出多个0Day：权限绕过、远程代码执行全部涉及  
 看雪学苑   2026-11-23 10:04  
  
OpenAI 最新发布的系统卡披露，GPT-6.2 Astra 在安全测试中成功发现并利用了多个零日漏洞，涉及移动操作系统权限绕过和网络服务远程代码执行（RCE）。这一进展再次引发了业界对 AI 自动化漏洞挖掘与攻防能力的关注。  
  
  
值得注意的是，此次漏洞发现并非来自真实攻击事件，而是由安全公司 Irregular 在受控测试过程中完成。相关漏洞目前正通过负责任披露流程处理。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0LanQuBROkgyCAibyb1lfTGDcV75gtDwbkjSSmbp3I65EMUX4XvSlTdo1PbDcABLq5uR6wNGIa6TmRQjXcwQvmrKqXRWI51SxQ/640?wx_fmt=png&from=appmsg "")  
  
**一、发现两类零日漏洞**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2EUnibXdHr1hZTibiaiarmjyx9HPfkXDrPwmu8j02OicaM2ptXLqGvezznYCSPicFEiabNJQOfmiaDs6VYtia2Hhp7zZZoItlwP9rZCal0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2fGAf2EzLeRojQptTic8nmPpofJHnsu2zk1RChqmNHaJj04cgZ4MHDxntRokMSFSKvhrkA5J1c2Nt4CZeyfK0e9ibP5Mibd08jiaY/640?wx_fmt=png&from=appmsg "")  
  
  
  
根据 OpenAI 11 月 17 日发布的系统卡，GPT-6.2 Astra 在安全评估中发现了两类不同的安全问题：  
  
1. 移动操作系统权限绕过  
  
研究人员发现，恶意应用即使没有获得相应系统权限，仍能够访问原本受到保护的私人数据。这意味着操作系统的访问控制机制存在缺陷，应用可以绕过正常的权限限制。  
  
2. 网络服务远程代码执行  
  
另一项发现涉及某网络服务的 RCE 漏洞。GPT-6.2 Astra 能够利用低权限角色实现代码执行，表明攻击者不一定需要较高权限，就可能触及系统中的代码执行能力。  
  
  
目前，OpenAI 并未公开受影响产品名称、具体版本、CVE 编号及补丁发布时间。因此，暂时无法据此判断具体产品的受影响范围。  
  
  
需要强调的是，这两项漏洞属于独立发现，并非一条完整的移动设备攻击链，也没有证据表明相关漏洞已经遭到野外攻击。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0jSLia5dEeewTtHDRCI1bTtYoUHeL0uD5t0NgCSYooVE9Spjo6ibmnsQzUAjcUf46f790e3wqjbNyR68cGjPzDkEtX0AMQoEmE4/640?wx_fmt=png&from=appmsg "")  
  
**二、实战评测：发现漏洞不等于全面攻破目标**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0HQh7oxnvg4gWjWVWfYDrrh85u07kCXUXc432xFib1Q1Zqes8RmVE8tvfpf0LSbJgtkSVm1afoAJcFD7wyUtJX8Pk3SlOl89Sc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2KLteibRsF82MLXox99YlqKXY1krXySNXAvAm9XwCAZibREPsvda75SH9uKBSFAYsNgBU0q2k4pYM0nGsIzZJXAaa0DyTtWLVIQ/640?wx_fmt=png&from=appmsg "")  
  
  
  
除了漏洞发现，Irregular 还通过 FrontierCyber 和 CyScenarioBench 两套测试体系，对模型的安全攻防能力进行了评估。  
  
  
在 FrontierCyber 测试中，GPT-6.2 Astra 成功完成了 217 项挑战中的 68 项，前代 GPT-6 Astra 则完成了 77 项。与此同时，难度最高的 15 项挑战，两款模型均未能成功完成。  
  
  
在 CyScenarioBench 测试中，两款模型均至少成功完成了 11 项挑战中的 9 项，GPT-6.2 Astra 的成功率为 63%，略高于前代的 61%。  
  
  
这些数据表明，Astra 在部分安全任务上有所进步，但并不意味着其已经具备突破完整安全防护体系的能力。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0HMo9CoLQwtCFLfXyQ42qEib1l9VXiacXribCdiaqLs8eIaBJkA7tDCeKIuCnk5K5QyOaSEWc9KFlUl7RTkO5CvhfoC0eAloibb43I/640?wx_fmt=png&from=appmsg "")  
  
**三、已知漏洞利用能力提升明显**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K32QiaczjWObJkU8CgrgyyFKKhjACbzicjHJwvrmF5EGDXNaXnlhlpglf0EhsWms9EgqUuAmyaAX9uuUXuwP7s8tNzohW6rJkYfA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2q9LnMeN7vDzySqxwDhrBQXnccKwH1shd3t1y4RNLuhgJ9s7ibQwofghTAlzZibl9HZL1TicYleT8PYoYRnxMLJeJXEIsRQmVlHk/640?wx_fmt=png&from=appmsg "")  
  
  
  
在针对已公开漏洞的利用测试中，GPT-6.2 Astra 的表现则更加突出。  
  
  
OpenAI 内部 ExploitBench 测试数据显示，Astra 在 Arbitrary Code Execution（任意代码执行）测试中的平均成功率达到 53.5%，而 GPT-6 Astra 为 31.5%。  
  
  
不过，这项测试使用的是 2026 年 6 月至 8 月披露的 V8 和 Chrome 漏洞，评估的是针对已知漏洞开发利用代码的能力，并不等同于自主发现零日漏洞的成绩。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3jByTtdPcgmTMgFHhGowL5EibibxMoljl0MTFialn9Ka4DOVXeefeY4iaHoKtzDyUicL6M5mibYC8Czw6XLOJEcaZJjUoxBiablweTHQ/640?wx_fmt=png&from=appmsg "")  
  
**四、AI 自动化漏洞研究进入新阶段**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2eLra1nvSOLicXV6SUSCW6CdwfK4lDNYzxicc7eLzclFSHYKickpUeEPOrqAKSibnR5oPGRxgngkEoE6E5B6Afn6jBiawxw3JRIC70/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0t9ZJ2x3GBfNn0ZUlUuNlOic34OetgyoCyzo7R1as4Oicy8IwUMQ5bLHR6NHIVdSBiajRWEWVwicpygbmWqpq1kWiacgwl1XlAZxzI/640?wx_fmt=png&from=appmsg "")  
  
  
  
OpenAI 表示，GPT-6.2 Astra 与 Sol 均已达到其定义的关键网络安全能力门槛。针对高风险能力，公司也设置了更严格的使用限制、监控机制和额外审批要求。  
  
  
从此次披露来看，AI 在漏洞发现、漏洞分析和利用代码开发等环节的能力正在持续增强。但漏洞发现能力与实际攻击能力仍然存在区别，安全防护、运行环境隔离和权限控制同样不可忽视。  
  
  
随着 AI 模型逐步参与真实安全研究，未来漏洞挖掘与攻防技术的工作方式也可能迎来新的变化。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K2A9XfK1MasZLEeZH48dhUcrGu58eKpQuxJ5qZyOMlBMwRKPLVKf6uiaBhK6zjk5q9x9Z48ib5EFzFMMLU0iaanHHyQqR6wBfkPiaQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2zgLyZevliaOrzgeynuYUJwpic3qd3WqtdibFpIibbqHGpAZlxthZUjmuicxZbCjBptQOlN6Jbxgd0NAs8ILlOCCwrwrb2KHcD3hWI/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2bXCWe0v548SfhcD1sIPv6znicnbRWtMmficH1Gglg1QxxmT0U8TwS1agR0Tm3PZGNNacPQkxCKR4SDhiavnEMLOSbAMkaja7oLE/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0bAZbiagn2weHTOPxj8PovSmHXkpiaRYnIuuRgACzTtAKenPAlX76gpLBGbnPPa3Fx0WNhObx55jibV6soicfias1KTLbRFvuqgicfA/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
  
