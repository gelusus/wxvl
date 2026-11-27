#  强推一个AI 安全运营工作台（御枢 AegisOps） - 告警研判、应急响应、漏洞整改！  
 华盟信安   2026-11-27 02:44  
  
介绍  
  
**御枢 · AegisOps**  
 是一个把模型、智能体、工具和业务场景连接起来的 AI 安全运营工作台：Atlas 负责对话与能力调度，工作流约束重复步骤，业务场景承接实际处置与记录——从告警研判，到应急取证，再到漏洞整改，一条链路走完。内置**告警研判、漏洞运营、应急响应**  
三个可切换工作空间，可从演示平台熟悉界面再接入真实环境；新部署不携带任何预置账号或业务数据，没有默认口令。  
  
![AegisOps 工作台首页](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscRwYc9rwm2AdDoqJo6icqlmcfHJ9eqATvwkFJpxUS5mOrwicBRiaiaiclianJ3lKuLX0bTxJYGBwFziaBBRWpByibfWx8sCTpJT8cvvL7E/640?wx_fmt=png&from=appmsg&wxfrom=13&tp=wxpic&watermark=1#imgIndex=0 "")  
  
更新  
## 2026.11.19：统一主题，完善日常操作  
  
新增 **海盐蓝、青岚、暮紫、暖砂**  
 四款皮肤，连同经典森绿均支持浅色、深色与跟随系统。统一工作台与业务场景的卡片、按钮、日期筛选和文字层级，同时保留厂商标识与风险状态的含义。  
本次更新实际变化五套主题皮肤侧栏调色盘或偏好设置随时切换，自动保存，业务场景同步生效深色界面重做修复配置抽屉、模型管理、工具详情与弹窗中的白底和低对比度模型配置完善品牌化编辑界面、上下文刻度、完整模型名称和当前思考强度研判图表保留只更换汇总卡和趋势区背景，坐标及图表配色保持原样运行问题修复工具刷新不再重复注册场景路由，发行包移除旧默认定时任务  
![海盐蓝深色工作台](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscSek2kibbuXEkkqicsOd5EzXv6nLVyWiazNrwsTqGFQItDAjTk0xgmW5DwBn0FkZibH6DRJr6W1tSrMlb6JibKmickRPJvrUnwicHH7nE/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=2 "")  
![主题皮肤选择](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscSxH5ULCCEhj7iaMuYMibobC9pHiajQ9sw2chTGBgRyxQc5HYbSSelpicWIpnQFjOokglLNyeafmYFh0SXwCYFRN6xVMHo658xPicxs/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=3 "")  
  
此前版本已包含威胁情报官方直采、Atlas 资产核查、Solar 辅助取证及按需安装 REA，详见上一版更新。  
## 四个工作空间，贯通日常运营  
### 态势感知 AI 研判  
  
从设备告警进入研判，保留来源设备与原始线索。确认攻击后，可将事件转入应急响应，或将漏洞转入漏洞库。演示环境与正式接入分别保留；非预置设备需按真实 API 和日志样本适配，不能仅修改名称后视为接入完成。  
  
![态势感知 AI 研判](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscRNoO8m0xzDuOar7DHO3h70gicnz8LBxCnOFuhqqvdhlicMHW8WOibiawbP5AXKTj7WHTicaep6jpNvb8fq7rQokiaKIWhndadnXpwsI/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=5 "")  
### 漏洞运营  
  
统一资产目录与漏洞台账，支持搜索、编辑、删除、资产关联、整改分派、提交复测和历史归档。管理员同样可以完成复测。资产拓扑与趋势帮助查看业务关联，Atlas 可协作整理录入资料。  
  
![漏洞运营总览](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscQT8GYTbvoeibCRphHGQs4yHtEZibjkRemgCDoibSrcUV5vR0ZSg9icYFoo3BKAicSicib6yiay0ou7kk7N2a5L1Eu9koLkDauTicwhNstY/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=6 "")  
![资产目录与拓扑](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscSvu34nHFH8DZdGUrVySZwViaWic6NsqbbKXyXSB9jMice6Lm3ZX10SsZJDUCTpaicZGfg47C4sh5UHkibRNdYEoAt0HxWxFicQknaGs/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=7 "")  
![漏洞台账与整改](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscTY924B6ib1icjibJiatw1Nd4Un4E4IsRgFibHbBwKOuMXgMlFc0OTLadMIkELJEhicxwA1t619SbrsxaoJTpiaFlVWSZjm7eLXo1g1wM/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=8 "")  
### 应急响应  
  
事件接收 → 来源设备日志关联 → 授权主机排查 → 文件取证与分析 → 证据报告 → 人工复核。Linux 支持 SSH，Windows 支持 WinRM HTTPS。AI 根据实际证据继续追查，报告引用证据编号，不将建议写成已执行动作。  
  
Solar 作为调查背后的辅助工具使用。Linux 采集受主机登记、指纹、权限、超时及尺寸限制约束；失败会留下记录并继续使用已有证据。Windows 使用固定 WinRM 模板，当前不宣称能后台执行 Solar 的图形工具。  
  
![应急响应事件工作台](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscR7l5ibwPiaN8rqicRcickaYy5yV3fbwINFbNW9m0ZYicxSRQ3IkR7Mk37cd4HOZJkJH3CZjlP8LKOGESfLZDv68MFFQfsuBKicO2pSo/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=10 "")  
  
样本支持静态初筛、可选 REA/Ghidra 静态逆向、微步与 VirusTotal 查询或明确确认后的上传。REA 环境按需安装，使用 Linux/WSL2 工作环境；不提供运行恶意样本的动态沙箱，也不把字符串匹配称为深度逆向。  
### 威胁情报  
  
首页仅展示七个来源入口：微步、阿里云、斗象、长亭、NVD、Tenable、CISA。进入来源后查看具体公告，按日期、等级或关键词筛选，保留原文链接和模型辅助中文译文。采集遵循公开接口或官方订阅，来源限制会如实显示，不伪造更新成功。  
  
![七个情报来源](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscQ3QBVUYWfrFS07thQmTfnfd1lmqZtue1SxURpPLbjIm5Js3eASJVKAQsn7FOyBPOpkkYelon6SmEuiceMPS5DGhwgOO9eiceEts/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=11 "")  
  
Atlas 的这个入口专门用于**核查已有资产是否受情报漏洞影响**  
，不再默认打开资产创建引导。它可只读获取资产产品、版本、公告与候选关联，结合证据区分已确认、可能受影响、已排除及资料不足。产品名命中只能产生线索；人工核实后才进入漏洞整改。  
  
![Atlas 资产情报核查](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscTFR48TbQu3JTuDWsibS58X1KRFj8HPuiaSt8jkNsoxXbMD8471xTHdq4Fdj4FOXVgVEbRRgcJItb3SJ25K5KoGJwER7byfBPFEw/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=12 "")  
![每日情报工作流](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscTKuUjtJV7M3AnqwvjaxxvtefLBHpYFAvPor92lveg4Ltibv8GkjOEB18H5qX9SLBq7A2MoSbKrzCu8RV8he7xQrAuKsDRBuHI0/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=13 "")  
  
项目地址  
  
https://github.com/knot07/AegisOps  
  
文章来源：乌雲安全  
  
