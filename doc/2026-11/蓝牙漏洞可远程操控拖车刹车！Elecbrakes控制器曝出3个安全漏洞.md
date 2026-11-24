#  蓝牙漏洞可远程操控拖车刹车！Elecbrakes控制器曝出3个安全漏洞  
 看雪学苑   2026-11-24 10:05  
  
安全研究人员披露，Elecbrakes 拖车制动控制器存在多个蓝牙安全漏洞。  
攻击者无需配对、PIN码或驾驶员授权，即可向存在漏洞的设备发送制动指令，远程施加、调整或解除拖车刹车。  
  
  
其中，两个漏洞的 CVSS 评分达到 9.6，可能影响车辆行驶安全。美国网络安全和基础设施安全局（CISA）于2026年11月19日发布相关安全公告。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K39dwxoyuXoIkLx0kNgxBfeia02KiaWHMZ1lgyHP43WuJN948MjYSU0gdH5893Q1ibib7HSl9QK1zBARW7icsHWtibicexpDhXGKlwLqM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2zHO1QPObrUzKOA6o2RGPtzE5az4yBskHDibCK16RwjSTwNmqdNNQuPGGuuObr15ZL4NLgmFThia25k6RO9KhUbCErOtRibQsxu8/640?wx_fmt=png&from=appmsg "")  
  
**01**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K2GE580icd3Hibm3libqPibQsLg3ZTvTmiaYTkMhibXay4icSAZSmSZ120ZebicqUrUzBhfCDpouxNckn1DmicO7d3lxrbLicduRhRASBQ3c/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0otBymLTGzcjwSjQ6aUicDF7hvRqgeic5yiaS76iaXENWIFGkANw5cYPdU0qzdM2yg61NFHnQcZwlDsPoRe2LJdCNwf1XbnxVcrhY/640?wx_fmt=png&from=appmsg "")  
  
  
**三个漏洞影响蓝牙通信及固件更新**  
  
  
此次漏洞由 Element Digital 的研究人员 Patrick Forck 和 Jeremy McCarthy 发现。  
  
  
研究人员最初在使用车载无线遥控器时，注意到设备未经配对提示便连接到了另一辆拖车。随后，他们针对自有设备开展测试，并发现了三个安全问题。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K1hQL5iaxK6FibMugJJFWxK69AgDJodGaI0lmp3wR7ovA76AGTt6Fs5K0ybJBQic2LLpgFnV3oEsyAA19HX2aMnSdlHC9QrpKGHaE/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞详情如下：**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K3uTOPkMECMaBnI0XL5AgAVHWQLiaOVr3lsHvZGED6ukaruRXjjJzaV6WiaibUu3qaD56PHAqMxUTxWLxpS1GpRSYTRMGTLzNcumM/640?wx_fmt=png&from=appmsg "")  
  
  
其中，CVE-2026-96805涉及蓝牙低功耗（BLE）通信。受影响的控制器未能有效验证制动指令是否来自授权设备，攻击者可以在无需配对的情况下发送指令。  
  
  
CVE-2026-125273则涉及重放攻击。设备可能再次接受此前已经发送过的制动指令，未能有效检查指令的新鲜性。  
  
  
第三个漏洞 CVE-2026-124281 涉及无线固件更新通道，安全保护不足可能导致未经授权的固件修改。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K3Klc0RMK0vMqRg9HlRs6t0L1FFUt9m8fu8rKZib7dRGaVKiaQhvQw0YC7aMibZUXO8sst3qdfGibIktDEJQad2fV7cWtmY7Ls0ET8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2Wrp4iczVsuic98eEVTTveRwUVh6XcdF2FYkTsM4HtZs93ClYibMNHib7cJsVOq4ufnGk5IDs55YqXyGoxbrS8pDh3uzPG3ZcHWZo/640?wx_fmt=png&from=appmsg "")  
  
**02**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2hxWBEj0MWeD2ZRsRI5Gorokibk9ENzBcMLYjvZzd14718KgaQ4LQCvKTBN3j32DxjzkemCTNhtW5XMuA82iaUOEU6t9gEGQtE0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1ia1ibnNXH54JB9GkBPKvCkhDFO7Z3ryty1tL7IPGcRicpIMDtyn7nhwIMuDWkVEqK5KvKIzah1CFJ4jXFHojicic5LNkltwbib2XdI/640?wx_fmt=png&from=appmsg "")  
  
  
**攻击可能直接影响拖车制动**  
  
  
Elecbrakes 控制器通过蓝牙与手机应用及无线遥控器通信，负责调节拖车的制动力。  
  
  
在受控驾驶测试中，研究人员发现，未经授权的制动指令可以改变拖车的制动强度。同一车速下，较强的制动指令会产生明显高于轻度制动的减速度。  
  
  
如果车辆在道路行驶过程中遭遇异常制动，可能影响牵引车辆与拖车之间的稳定性。  
  
  
研究人员表示，此次测试仅使用其自有设备，在私人场地或封闭环境中进行，并未涉及恶意软件攻击活动。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1Fhjic8EWhCnt8bIKbMv3leJGFWhq8TXdQXicSYkFyQI3mib7jC3ADkMhfVQjePPwodcbXnWGRXRZfcq84TqvbRjoUTvqmaXBLx4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K0SR7mU1OIibM8OykRcvxWG0ib23cHcE8ROI2XcB3jE38QJhEQDO3tJXrnbvpmO6D9SHNPzBB9aIsWSXNT3x1Ik4GxUNm3FHBZ2s/640?wx_fmt=png&from=appmsg "")  
  
**03**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2vJeQj4zicIMbgF110u0XlCiaIEqPR23BUibMVo36RZNgWib5xPT7U3Q6aDnFrCqWJp7ZSvCwV6nZXqCgmFJrRib8xmZRE25wRydW8/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K1UNmic6uTYqnW3dqkdsHMyn38MViaC7AteQ0HBuAiaWfLalL5BETZKsvjZQ6dUwZgMJGnpQcjk9tiadaBhchjmicficz6pEhDqK0zus/640?wx_fmt=png&from=appmsg "")  
  
  
**受影响型号及修复措施**  
  
  
Elecbrakes 于2026年5月18日收到漏洞报告，并于7月6日发布修复版本。  
  
  
根据 CISA 公告，目前涉及以下型号：  
- EB2： 升级至固件 3.0.4.4.3 或更高版本。  
  
- VM1： 升级至固件 1.3.4.2.1 或更高版本。  
  
- ELBC 2000： 所有版本均受到影响，漏洞披露时尚无可用固件修复方案。  
  
EB2 和 VM1 用户需要通过 Elecbrakes 手机应用连接控制器，并手动检查及启动固件更新，不能假设设备已经自动完成升级。  
  
  
修复后的控制器要求通过经过身份验证的连接才能执行制动控制及进入固件更新模式。首次配对还需要通过牵引车辆的制动踏板进行物理确认。  
  
  
由于新的配对机制导致旧款无线遥控器无法继续兼容，Elecbrakes 为受影响的 EB2 和 VM1 用户提供免费更换遥控器服务。  
  
  
对于暂时无法通过固件修复的 ELBC 2000，用户应联系 Elecbrakes 获取针对具体型号的处理建议。  
  
  
截至2026年10月下旬，Elecbrakes 向研究人员表示，EB2 和 VM1 日活跃用户中，约60%已经安装最新固件。  
  
  
参考资料：  
- CISA 安全公告  
  
- Element Digital 漏洞研究报  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K1qlMAkXeFzhu1kca40yibgdg6csiataHXUbmCSb3QQiabnQib6GD4In9jicXhgmRR5512JEvibQfSOhicUSTEtBOic8MYnXeaQ2Dsv0lY/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K0YfKmNlhbn1aiboYQZ6j3VibMl50Df4ZFyzcWAntQ7HjvDAK1VGbdhG90AB0uhPoibzskapics1kATTkev8n5XYTaAQMEdCKWRGibs/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K01P55LLrv4a0q66lFHiaY8l71uYReJfmKTiaX3Ww2MCvyz0NIlrPW7fnhUAGFZjNVf5RKx7ELeQVceOQm4ibGyrskmQ6f3xazODA/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K2qMzUsSOymRPTsS1P6hrbnOMrXgVxgK3mRKBg4FqjzMDA9EN1fOjdDJXnCedZcnxdN638z5pk7lWQ5ia0Hia6C9bl4wXaibSUIyM/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
