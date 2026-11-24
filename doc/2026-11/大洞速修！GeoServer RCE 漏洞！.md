#  大洞速修！GeoServer RCE 漏洞！  
原创 ChinaRan404
                    ChinaRan404  知攻善防实验室   2026-11-24 15:21  
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXGAGEGckzNb1ia76LNIRrOXeeT1vLdSdfcbISbFJpZzShBbmvobJ3Ay2Enpfiag1LYZyJhxSw1joKwa5hqqdC02crAZCZeXyxH3I/640?wx_fmt=gif&from=appmsg "")  
  
前言  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFKVEhCPOicJJqq1rGb4sjkj1niaGLeG7icUWhgPJVPQJjSDaNQgymhH8xShXTsZsWVC8iaxtQAicpl3vjfB3lNFkicW6BNF4cSrMn2I/640?wx_fmt=gif&from=appmsg "")  
  
  
AI 大人还是太强大了，5 个组合拳才能 RCE  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a3etiafIAYXH5Y6I9sQVFzXX7ZtDjmwonXGtaFvJ5U8o1c6gKW97YKttCKDYqlgXjSuYvn5iaG7Bq4q7B8INyaR1HRtzs1UjB2TNXokYo4mAU/640?wx_fmt=png&from=appmsg "")  
  
GeoServer 是一款开源、基于 Java 的地理空间数据服务器，用来发布、共享 GIS 矢量与栅格数据，支持 WMS/WFS 等 OGC 标准服务。   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXGwAXzAicuyKTVpMAwgibqBuYZaejUeibrhE3ATPa3icyE4PcCjhhF2lOKw9A8TIqC4mzKbD3NuYYGXWvROO51DnTGT9SdZ2LmicfNA/640?wx_fmt=gif&from=appmsg "")  
  
影响版本  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXEhJzoZv0iciayAK54SUhu38M0CU4mouH4y8ONFWiadKWc2rLmhibavqofeicmTbvSSBcPLHmFpZricxM6A6QhvTJCVLGqXfmcPb8Edc/640?wx_fmt=gif&from=appmsg "")  
  
  
GeoServer <= 2.27.6GeoServer 2.28.0 - 2.28.5GeoServer 3.0.0 - 3.0.1  
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXHyxvWdpG84x3mc50FmPFPRUM12NdgYurRQ85yRJxL9wguvibicoSBxTO4q2euv1lDlqvxRyg5Gu0Dl7wcq9U1sRYS2puZm59XfQ/640?wx_fmt=gif&from=appmsg "")  
  
攻击条件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXEdfxMCroA3bndrX8icyLz3GXKzgAaQTglywROERNP6kGgPy0omicZOeiacMvvBUCpicQvqe21sjDSXlIXhofjt6ab5beZehzatbTE/640?wx_fmt=gif&from=appmsg "")  
  
  
无需任何凭据  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXH8ROq2JWeoUK5IUuq4jnfCyFzMOia86zbdSn0LauzNsO8ue4Bd1liaDtDPMjnTueu29f0m47hoQBrWCNbNzMr8GlGRLaDRU0ZFE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFrEoQxGtC1pVRIsGibmW9bwstCjKIElVUAmEGIyD3jrwkuShWetJmkEAyVF3fPVWVDxNrc9HEY3Y1HAwfw61EHwibic4AKBuQ0ic8/640?wx_fmt=gif&from=appmsg "")  
  
组合拳利用  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXH5GocexLiav2bUiapZWlM0A40TShkH8J2Iib3TvmG4bmy3sZy0jkNnG1l8wU8x8ZhxiaMia6TBc2tYDjPrlgv7Zr1fS3RNyDuKibD7E/640?wx_fmt=gif&from=appmsg "")  
  
  
这 5 个 CVE 漏洞：  
  
129244→129243 → 120705 → 129245 → 129242   
  
简述一下这几个漏洞的作用：  
  
  
CVE-2026-129244  
  
超长 storedQuery_id 异常回显数据目录->获取路径  
  
CVE-2026-129243  
  
service=x/WFS 绕过 ADMIN 规则，匿名建/删存储查询  
  
CVE-2026-120705  
  
XInclude 读 security/config.xml + users.xml，拿 key 与 admin 口令摘要  
  
CVE-2026-129245  
  
remember-me 固定密钥 geoserver + 摘要 → 伪造管理员会话  
  
CVE-2026-129242  
  
JDBC 用户服务 OTHER 列 getObject() 反序列化 → 代码执行  
  
   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXHsMVun0VajVSJyEQfAsuLgiclOtPzZuuibJC2ksw4DabQqPXcAU71COFibBH3EQKkzzibicT8pUtCANic3QzeL71omic8Buu36QcLVP0/640?wx_fmt=gif&from=appmsg "")  
  
漏洞 POC  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXGnm6U69o68uTZPp2117FyLXFkLKzFAPtYFxacSKhBibpfiahbibR4IXjpxkfG7hfwT3YBuGOrfuuS1QgSYLUncuhclmm4e4H7da4/640?wx_fmt=gif&from=appmsg "")  
  
  
全链哨兵已第一时间复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a3etiafIAYXF014YypCLW2ZEiaZxC5ATVDXd1dp5jCr7K3pCPXw7XLjoNgXwzicVlBXibT9434j18w5ichQp6nz2Y7w3pEyicIfibnAskU2XMFQiaSU/640?wx_fmt=png&from=appmsg "")  
  
关于全链哨兵  
  
[关于网络安全 Agent 相关的一些思考](https://mp.weixin.qq.com/s?__biz=MzkxMTUwOTY1MA==&mid=2247494302&idx=1&sn=9b1f338412d17283776d203e6d066d81&scene=21#wechat_redirect)  
  
  
  
复现脚本  
  
https://github.com/zgsfsec/geoserver_rce_1124  
  
