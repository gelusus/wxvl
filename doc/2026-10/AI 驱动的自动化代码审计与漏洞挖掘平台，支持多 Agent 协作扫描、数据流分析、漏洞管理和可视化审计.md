#  AI 驱动的自动化代码审计与漏洞挖掘平台，支持多 Agent 协作扫描、数据流分析、漏洞管理和可视化审计  
vs-limits
                    vs-limits  夜组安全   2026-10-09 23:30  
  
免责声明  
  
由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！  
**所有工具安全性自测！！！VX：**  
**NightCTI**  
  
朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把  
**夜组安全**  
“**设为星标**  
”，  
否则可能就看不到了啦！  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icZ1W9s2Jp2WrOMH4AFgkSfEFMOvvFuVKmDYdQjwJ9ekMm4jiasmWhBicHJngFY1USGOZfd3Xg4k3iamUOT5DcodvA/640?wx_fmt=png&from=appmsg "")  
  
## 工具介绍  
  
DefectMine 是一个面向代码安全审计的 AI 辅助漏洞挖掘平台。它通过多个专用 Agent 协同分析项目目录、调用链、跨请求数据流和潜在漏洞，并提供一个本地 Web 控制台，用于管理项目、运行扫描、查看审计结果和维护规则。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WibL3bOeESMKiayYOPVYRRmuQx95RliccfThcIVy1tibQgEVYVbwManw99ic4Iia7U2933MWjtCzpvs7QcRDtIX7QDCjKwT5xP1ysNtjSpSIXHcwI/640?wx_fmt=png&from=appmsg "")  
## 核心能力  
- **多 Agent 协同扫描**  
：TreeScan、CallScan、DataFlowScan 和 Auditor 分阶段完成代码审计。  
  
- **项目与版本管理**  
：按来源、所有者、仓库和版本组织待审计代码。  
  
- **全仓库与局部扫描**  
：支持完整项目扫描，也支持针对指定目录创建 specific run。  
  
- **漏洞审计与复核**  
：查看漏洞详情、证据链、调用链、数据流和审计结论，并支持人工标注。  
  
- **可配置的扫描知识库**  
：在控制台维护 Prompt、Skill 和 Sink 规则。  
  
- **LLM 配置切换**  
：支持维护多个模型配置，按需要切换扫描模型和参数。  
  
- **大文件分页浏览**  
：扫描产物、Markdown、JSONL、日志和对话记录按需加载，降低内存占用。  
  
- **报告导出**  
：支持编写和导出漏洞复现报告。  
  
## 控制台预览  
### 扫描工作台  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WibL3bOeESMKcEfasBA6ZyzCBd0via5BjrMwxjlWyAQD6yQvqZ5Q464sDMbYfc7YS29spljT5PNJFhfQyDich7IxDNBLiaqTfk2CCBwSOICd0hc/640?wx_fmt=webp&from=appmsg "")  
  
控制台提供项目列表、扫描流程、目录树、Agent 产物和审计结果等统一入口。  
### 项目信息与版本  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/WibL3bOeESMIpoicDIcYR1UUroC6PgdNXgBWZv3hhnWcD0KLj1lKH0MySREytrSo6QLJNiaqh6pYqkicq2klZvd9OpM75KPqoia9J304bRkiaBtSg/640?wx_fmt=webp&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WibL3bOeESMJNgbBmIjXEhBQYt9ngoYZHmmNeFjO5FeW2NAXPSoWSsAy17F8sRicJx8fQ1zial79J4t0CedL2xdsibfULpLezSnuU7d9y3m2TdU/640?wx_fmt=webp&from=appmsg "")  
  
项目支持自定义名称、描述、标签、审计状态以及多个代码版本。  
### 漏洞管理与报告  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WibL3bOeESMIjfqiazfqz1bswvUba0cvzfscicu4fJ2FqAyuuXoQJmHmb3nIgrVt8gn7c8tTciaDC9UicW6jRAQUn6LW8AfgiaUibbRuFxZK0b7q3A/640?wx_fmt=webp&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/WibL3bOeESMIBqIfiaExjia0BJ3bPyXYEKrOy4GYW78N4EyVqNicvcy47XyopALcu3u1rOPeibjiaJxu5Kahs4f24R5c4GaemtIcezdJ3brSse91E/640?wx_fmt=webp&from=appmsg "")  
  
可以查看漏洞证据、关联扫描链，并整理复现步骤和修复建议。  
### Skill、Sink 与 LLM 配置  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/WibL3bOeESMK1xfianGVMmTrV7TOicoBl3hX03axkDick9ibocMXpOrGmDq19ZzWGiaezmwT0npmdCiaPuiaUQficujLG3tAdhWd3iah3yZ9sDLSAIFhY/640?wx_fmt=webp&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/WibL3bOeESMLbGtPCeSjs5GY1v25RAZsAHLsgqzsonF65ibTQnkfKgwuX3hRgoFVIJfmWm1994F2LfulZGYKyOaBJVtX4YHOiaP0R3N1LCc2L4/640?wx_fmt=webp&from=appmsg "")  
  
规则和模型配置均可在控制台维护，便于针对不同项目调整扫描策略、效果和效率。  
  
## 工具获取  
  
  
  
点击关注下方名片  
进入公众号  
  
回复关键字【  
261010  
】获取  
下载链接  
  
  
## 往期精彩  
  
  
往期推荐  
  
[一键获取目标网站的源代码信息、网络数据包（HAR）、页面快照与媒体资源，方便进行逆向、二次开发与安全分析](https://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247497818&idx=1&sn=f766d78163f4ecfaa81fa63b9fb38cc8&scene=21#wechat_redirect)  
  
  
[一个 AI 驱动的安全研究工作台，面向 CTF、渗透测试、红队行动、逆向分析、Pwn、IoT/车联网研究和恶意样本分析](https://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247497801&idx=1&sn=ac34ceb80f349b41ad189d04dfbe4dc0&scene=21#wechat_redirect)  
  
  
[应急响应流量分析工具，支持pcap、excel、log等多种数据源，同时结合加解密、反编译等多个工作流](https://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247497796&idx=1&sn=3c7fb90b46b16ea97fc57eeee1c73f48&scene=21#wechat_redirect)  
  
  
[AISentinel · 大模型安全评测与 AI 内容鉴别平台](https://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247497778&idx=1&sn=1663ea7a47eb13f7922cf01ad7298a70&scene=21#wechat_redirect)  
  
  
[企业关键系统失陷狩猎图谱 | 一款完全离线、单 HTML、自包含的调查规划与失陷痕迹狩猎工具](https://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247497777&idx=1&sn=26135255b6013b13f571d882e7b34e63&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/OAmMqjhMehrtxRQaYnbrvafmXHe0AwWLr2mdZxcg9wia7gVTfBbpfT6kR2xkjzsZ6bTTu5YCbytuoshPcddfsNg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&random=0.8399406679299557&tp=webp "")  
  
