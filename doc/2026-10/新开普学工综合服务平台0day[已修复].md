#  新开普学工综合服务平台0day[已修复]  
原创 Mystery
                    Mystery  小M安全   2026-10-10 06:24  
  
```
声明：本文仅供技术研究与交流，任何未经授权的安全测试行为均违反法律法规，请严格遵守网络安全法规。
```  
  
通过  
SSO换票伪造JWT  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2eEhf6SKuiaVcywiauEkTgVMDdIjJneyR6EVe7b2ib95bfibLxLIpLpFVxHpOmkQ8ffZS93E7cicxVyVqbz9l7lMgLL1bgibM5O8l7D0/640?wx_fmt=png&from=appmsg "")  
  
查看当前平台账号及教职工  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2dEPbnKicwbyYAdMSMCxibFswOiaCkqszJveQsaDOLSI9frrS5UC5MyLAq2rvfP4vmEDJlJSBPxPiagxh4nic60ib33rsELAPIHx7Qibw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2fueS2Hgnr3r5sCVQ3S0uzkQibg21Wqn0U5t3gibTg1L1GXFNrlPnCm8YZAD213MZcfZx2sQIvcqkUJNpeTPbJ4G3g616CDTEJias/640?wx_fmt=png&from=appmsg "")  
  
查看当前数据库身份  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uFfXaaQWh2cKr04AFW42LpC9ZKP210wNIfhmiaIc3DnfZPia8MQynB8nTOvCrQsVybdOwxzEETOwVd1Vytz0nMtTUSjRY4QrKHfcsalU5Qnxg/640?wx_fmt=png&from=appmsg "")  
  
角色DBA权限  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2d2XwkY5OBSibYsxWVicXdGMDxEwicE4e2appmt79ywZPAUWdFzg8GTDM74QzS5iaxOofghpaKgGZE1cH3psVflaxoJ10At4bFg9MI/640?wx_fmt=png&from=appmsg "")  
  
之后就是建立RCE的条件  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2dBKdVjk7j89X9pFnicpqjbEH2usEKZYhzQ5kiatxlN0lNqg0p9v5SGFy6qMtd7XddTFMeu1wr4LadWibgFABwcPJFzxmYBpZs5lw/640?wx_fmt=png&from=appmsg "")  
  
剩下就是实现RCE了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/uFfXaaQWh2dWucmMvib9icFHF0TXJI4Y2RpdeSDDo2CGicsC3JzT0AIyPU5x4saSjtEzOaw7xn6tqRIykSZZN7yOUJloXZ4LCiafQY71cSzafibI/640?wx_fmt=png&from=appmsg "")  
  
