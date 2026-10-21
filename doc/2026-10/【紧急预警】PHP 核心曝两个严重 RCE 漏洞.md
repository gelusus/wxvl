#  【紧急预警】PHP 核心曝两个严重 RCE 漏洞  
泼猴
                    泼猴  表哥带我   2026-10-21 08:57  
  
![](https://mmbiz.qpic.cn/mmbiz_png/UA4ABKCY6OzcOrUMTzLdbbfn592QWFZ1iaFw4wQVnugEOPibZ7C0HUNCdMiaey20WnArfLqCC8XJuJWPzAVH8HDgpaoNrp38eJxPTxrccjchSU/640?wx_fmt=png&from=appmsg "")  
  
## 内容验真  
  
**核心事实与来源**  
：  
- 漏洞真实性与编号、CVSS v4.0 9.2、补丁排期 21 日 14:00 UTC：PHP 官方公告（php.net）、CERT-EU Security Advisory 2026-018  
  
- 影响 PHP 5.4+ 几乎所有版本（报道口径，官方最终范围待公告）：watchTowr FAQ、Beazley Security Labs BSL-A1233、澳大利亚 ACSC 预警、国内安全媒体整理  
  
- 利用工具在补丁发布前泄露、演示视频流传：社交平台公开讨论（郑昀微博 13:08），属公开报道  
  
- 漏洞与 multipart 上传处理相关：第三方研究（DeXpose）描述，**未获官方确认**  
，文中已标注  
  
- 内存类缺陷不必然每个构建可稳定利用：DeXpose 研究提示，文中已作为未知项处理  
  
- 国内媒体 "补丁已发布" 快讯：DoNews 14:25 发布，与官网状态不符，判定为预发稿，**已不采信并在文中说明**  
  
**信息时点**  
：2026-10-21 17:00（补丁发布前）。官方技术细节、确切受影响版本、修复版本号以今晚 PHP 官方公告为准  
  
  
