#  AI时代下的深度漏洞研究  
原创 关注AI与安全
                    关注AI与安全  知白守黑1024   2026-11-26 22:39  
  
挖洞这件事，正在从手艺变成流水线。  
  
以前一个 0day，背后是几个顶级研究员几个月的手工活；现在一套大模型加上算力，普通工程师也能把它做成标准产出。攻击者不必等补丁，防守方赖以生存的 “补丁窗口” 正在被一点点吃掉。  
  
安恒信息金权这份《AI 时代下的深度漏洞研究》，把这条变化从头到尾讲了一遍：先摆出 AI 时代防守方的三个挑战，再拆解 “怎么设计一个会自己思考的漏洞挖掘智能体”，然后用两个真实案例落地 ——LangGraph 反序列化（CVE-2026-27794）、Linux 内核 rxrpc 整数溢出（CVE-2026-31641，与国际团队几乎同时挖到同一个点），最后用 1day 自动化产线收尾：78 分钟，从监测到交付带防护规则的报告。  
  
20 页，几乎没有多余页面。看图时不必一口气划到底，带着一个问题会更值：这些能力里，哪一环是你现在就能用上的。  
  
下面直接上原版 PPT。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1iaXgfdlib7WBYPrc7KEYtGtEh0e3ic9BhLS7EIfXZaTYfCPXzeWqARt6WL2ZicLq4b0ESl9n4G9636Dw6qSO7xpZmxQYFlzwETUUQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1jO7OgnnF1Nqtk2k6mYsrvuSmSSiaIzxia7KiaYiaicNR2nY7EctQerMnAtKEiaib9HFUlydmsBeAJr56gy83xPpwSMJ6w8y0fyqsmw3w/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1hCwibs3tKeSWibGjMiaBCbstaCq6Uic032yzuMDvElzUnweDvBNsUrO4lDbdPJicNToY4gpQ3hZicmByUFiabzU4hXm7ZcLdnnuibicraI/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1iapEtTCBZ1rnbAHcMjV2dn0TOPwibOW1LTd0A1X1LaOuckrZ0L5BueHCRGn6UjRQOyDSqbfdfjKCicw87tZ71O5sib08JJBfzyc24/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1iaHx7ueicEU4VwXsjy8t6Jh8w0qqLjRbh8l9oQWiacwXFkSVAjpToq5hHh1JeHicQiau9vMxic4Q899RJwUfVFLEiaib4e1O5xOgaW1z4/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1jqq6AuW2wRCzeQhX9MN3ibJn4H8BF88b0LL1ZgmBo2xNJ6nyGC0iaJD3w1jicd520yHhSvTXK55bjX0IG6HHGtRe0XohjPh5HblI/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1jRJnicxOkv0T08MYzibrhBh73csF5dmGJfUc34hWd2p65BiaMETzyBUn3hFo7aYptSvDmrib6zialRuu9AYx7GTwdI0Nemb4Y2C0Rg/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1iaseAc9WncfyeIVCdcQnZyaJVJdDlyLnXXf9JjSd54xdC5ragnua0XbkNSbVZFMopib5eLEpmctZvibkYGHZCRtnkjTFCiaaYbGqw/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1j0vagWZoicictKu53AHloecicAWVvibrF7OeGclKRG0TtNEhSJOsOwXGuXCezoe03LxsNkkjcwfS4P9hEgJsmTkPFoxfHxehUibe20/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1gPHfK07RVBzsCSQ4XruGvUPoF3jwXgCzxicGIcgiaZ2sHwRSdpNSQJZsVkIhLuh7JlSwaTKcoWDibU2kHYvuqhHAuYQNVcT2CSCA/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1gFmWsib86tcJtyxR6WMu1OZCcmE0QXHYlkOgSmibD9MmxHnH0OBBVCbrw1gia18e7QKJGXuFaLicyQyG10E6AULPyICT27f2vFdia8/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1jNOdmfKDHhp3zoRGDuMDr4M0vz3sp1NfHVQ7B4RecVp9Fn8b1GEvaMicYJzdzPYCfc0jV5abelGBWVOCZOr3tcfwalvGvh6KYY/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1gYhvYxm8sDDgjeG8XVh96orY2o1EooU4iba6M6EgpjsrKMjRI21Lric3S1vnEsmhWFYgVM38vlB8U8oDl1l9995Iar0tGNJ2vms/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1jc8PXAADsM0QsFhnNrzI60FGV1BzIn4xx1qXmEJS3AMDTgMkNoic8ofTzk5JoiaKKictRKFico7d7NPKEWqicHe1DhkEX9SQQyBfDM/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1jY5vhYygtjHDXjfRYCNHah0760FNj75DzDpYDpR2oiafbZ26u70zNkHF0zNHP6tz31IM6vmqPjf6tY8zU40tEbhETQqZ7OTrdg/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1jK8MyYt9Zt3lMYicGCgB5B2xqftml7H6LljRKAZHk9vJ7r08Qg8HaWnSibH1VtlrjIf0JiaRBafWWOhjcIF0RX3sMVZPfBrdiapI8/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/j7ZnQr1RD1iaHd2L9LyRPUxdAORbW6GDdWofzdea9g84ibEqZa6yjeWKtd3c8lmuiaDybnV1fSHmVZIfX9v5eicpx3VwzFtEjwPlKGNfpia5VZdw/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1iaSRJF8QJzQYdCAnnicjNKOH0RicXwJ5m51fI2RMGDvMQoRo6NGc8pKpk9yibz1flLnCnxXtXFjTO5qSBHElSGcdOuHriatpkDu0HE/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1iaLpjMK6nEyzOzf0l7z8lkS4PRnGhkibEYI0ZfGsyBwn1CqwMkE6A3fZZWA2SkwIIN5Fxbwy0lAEaPOceibR4WG30iaicWKocuTLGg/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j7ZnQr1RD1jlUSnv4pKPkmibagoFE48iaia0rT6M1uXKBuia1qVWwI9e85Wia8BchZicfD2b4esnN5ZGpgQf9ePGcYRu7e4rt8VLyVZePPCXE34wo/640?wx_fmt=jpeg&from=appmsg "")  
  
  
