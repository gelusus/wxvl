#  AI 驱动的自动化代码审计与漏洞挖掘平台  
vs-limits
                    vs-limits  乌雲安全   2026-10-15 01:22  
  
介绍  
  
DefectMine 是一个面向代码安全审计的 AI 辅助漏洞挖掘平台。它通过多个专用 Agent 协同分析项目目录、调用链、跨请求数据流和潜在漏洞，并提供一个本地 Web 控制台，用于管理项目、运行扫描、查看审计结果和维护规则。  
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
  
![DefectMine 扫描工作台](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscQOeSgCLZ8SUIURicYMHxf3WFibp9ibFzFVGXRnWvLUiaQnic2EItRx0RCpnt3gAQTafGgeHVCLMibicZSTEETc4ShGRauYzwTtAp5Vv8/640?wx_fmt=png&from=appmsg "")  
  
控制台提供项目列表、扫描流程、目录树、Agent 产物和审计结果等统一入口。  
### 项目信息与版本  
  
![自定义项目信息](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscSg8bI5mRzBVDlTHSia2A1a84UgdUUByic1buqhX2ksYO96g7aDwXYb7OvDCI8GNFibUWuTyedeOUZIdTib5oagwlktv8ITpAMESpU/640?wx_fmt=png&from=appmsg "")  
  
![项目版本管理](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscTzFbzQdQm1kQHIclcGIfpOBtaw5T8aic3QOgWxw9uUM83k6KiaDQ2R3KicUk8OiaZGDMxzOcP5bJTrB5t1U3iawcJzWh9Mh679re5g/640?wx_fmt=png&from=appmsg "")  
  
项目支持自定义名称、描述、标签、审计状态以及多个代码版本。  
### 漏洞管理与报告  
  
![漏洞信息管理](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscTicRpWvgCqz4Et7SXiaSnXliahhGdWKNBnefZlhaq4TX6lMUf3ZYVAQk5g7dASR6syq9FuSjlatzCqhIpq7TWQ9K2ia7RC1SXAfok/640?wx_fmt=png&from=appmsg "")  
  
![漏洞复现报告](https://mmbiz.qpic.cn/sz_mmbiz_png/5yYXmGfnscSicsjSsAd5RQXTwiaQwTemsbcZiaMmk1khL9mrbELzEE1Sr93TC3S9qoF7ib9hS4jREIjFAF9MicCxY8TMwDNxOG4cbibDAtbibMLzwc/640?wx_fmt=png&from=appmsg "")  
  
可以查看漏洞证据、关联扫描链，并整理复现步骤和修复建议。  
### Skill、Sink 与 LLM 配置  
  
![Skill 与 Sink 规则管理](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscRLxQe5S3VnGaLuORNSr2czibYs914lekNDly3jnic3ykRr77bzN4MB9Oy0vmtgtwIag0DNcZVMc1zZxCGgtUoiawzc2ib3biberIso/640?wx_fmt=png&from=appmsg "")  
  
![LLM 配置切换](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscSpPNz1HGPKvRvk6DPXPotXWZvxBgDKBicGlGRPlxkFt5VeoO2SyNBQxva6HlFRs3L9MIcscqxQVGIeiaRhwSQbUnBPM85JBOLFQ/640?wx_fmt=png&from=appmsg "")  
  
规则和模型配置均可在控制台维护，便于针对不同项目调整扫描策略、效果和效率。  
## 环境要求  
- Python 3.10+  
  
- Node.js 18+  
  
- npm  
  
- 可访问的 OpenAI 兼容或项目支持的 LLM 服务（运行实际 AI 扫描时需要）  
  
## 安装与启动  
### 后端  
```
cd backend
python -m pip install -r requirements.txt
python -m uvicorn app.server.app:app --host 127.0.0.1 --port 8000 --reload
```  
  
后端 API 文档：http://127.0.0.1:8000/docs  
### 前端  
  
新开一个终端：  
```
cd frontend
npm install
npm run dev -- --host 127.0.0.1 --port 5173
```  
  
前端地址：http://127.0.0.1:5173  
  
Vite 会将 /api  
 请求代理到 http://127.0.0.1:8000  
。如果 5173 端口已被占用，请同步修改 frontend/vite.config.ts  
 中的后端代理地址，或使用一个单独的演示副本运行。  
  
项目地址  
  
https://github.com/vs-limits/ZeroHunter  
  
  
