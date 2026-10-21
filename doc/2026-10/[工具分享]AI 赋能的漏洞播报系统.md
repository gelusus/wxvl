#  [工具分享]AI 赋能的漏洞播报系统  
 小猫信安   2026-10-21 00:58  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a4nfIUib3hYJsR4rmtQOUWwC8gDIkaHQBpw3Y0UVia5W0wxoW7ichnTOngicufJuxHc2eH99epXx4XQ8GbJRjftZheaZotEeHE9mBhIwDjDIiaWA/640?wx_fmt=png&from=appmsg "")  
  
项目地址  
```
https://github.com/wr0x00/ai_bobao
```  
  
AI 赋能的漏洞播报系统，AI 根据公开的漏洞思路自动生成 PoC、漏洞简介、漏洞原理、攻击范围，自动落库存储，24 小时监控全网漏洞播报，可设置间隔时间自动抓取，默认每隔10分钟全网抓取一次。  
  
一切漏洞数据均源自公开渠道(论坛，社区等)，不涉及法律风险。  
#### 预览图  
####   
## 特性  
- **多源聚合**  
 — 并发抓取 Seebug、先知、Exploit-DB 的 RSS 与正文，单源失败不影响整体，每个请求有硬性超时。  
  
- **AI 内容判定**  
 — 抓到的每一篇都先交给 LLM 判断性质：漏洞情报才收，赛事 WriteUp、工具教程、行业新闻一律不收。  
  
- **自助接入新社区**  
 — 设置页填一个社区地址，系统自动探测其 feed 并由 AI 判断是否为漏洞社区，确认后落配置即抓。  
  
- **AI 补全**  
 — LLM 将文章转为结构化情报：名称、简介、原理、攻击范围；文章已含 PoC 则原样收录，缺失则按公开思路补写 Python PoC。  
  
- **双向 nuclei 通道**  
 — 收录的漏洞可导出为 nuclei v3 模板；上游模板库的新增模板也可增量同步入库。  
  
- **自动中文化**  
 — 英文情报与模板元数据经 LLM 译为中文，CVE 编号、组件名、函数名保留原文。  
  
- **数据总览**  
 — 组件数量横向条形图、平台分布多色环状图，一眼看清库里有什么。  
  
- **一键更新**  
 — 设置页检测 GitHub 上有没有新版本，点一下快进到最新。  
  
- **文件式存储**  
 — 无数据库依赖，每条漏洞一个目录，换库只改 store.py  
。  
  
- **内置 Web 控制台**  
 — 漏洞列表、详情、总览、模板库、在线设置，定时后台刷新并展示倒计时。  
  
## 技术栈  
  
前后端分离 Python 3.11 · FastAPI · Vue 3 · Vite  
## 快速开始  
```
# 1. 配置
cd backend
cp config.example.yaml config.yaml
# 编辑 config.yaml 填入 LLM 端点与 API Key（不填则 AI 相关功能不可用）
# 2. 后端（默认端口 8100）
pip install fastapi uvicorn pyyaml feedparser
python main.py
# 3. 前端
cd ../frontend
npm install
npm run dev        # http://localhost:5173
```  
```

```  
  
后端http://localhost:8100  
。 打开 http://localhost:5173  
 访问 Web UI控制台。  
## 配置  
  
配置优先级：**内置默认值 < config.yaml < 环境变量**  
。也可在 Web 控制台的「设置」页修改，保存后立即生效。  
<table><tbody><tr><th style="text-align: left;"><p><span leaf="">键</span></p></th><th style="text-align: left;"><p><span leaf="">默认值</span></p></th><th style="text-align: left;"><p><span leaf="">说明</span></p></th></tr><tr><td style="text-align: left;"><p><code><span leaf="">llm.base</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">https://api.openai.com</span></code></p></td><td style="text-align: left;"><p><span leaf="">OpenAI 兼容端点，不含 </span><code><span leaf="">/v1</span></code></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">llm.key</span></code></p></td><td style="text-align: left;"><p><span leaf="">空</span></p></td><td style="text-align: left;"><p><span leaf="">API Key。为空时 AI 功能不可用</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">llm.model</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">gpt-4o-mini</span></code></p></td><td style="text-align: left;"><p><span leaf="">模型名</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">llm.timeout</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">60</span></code></p></td><td style="text-align: left;"><p><span leaf="">请求超时（秒）</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">collect.interval_minutes</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">10</span></code></p></td><td style="text-align: left;"><p><span leaf="">定时抓取间隔（分钟）</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">collect.fetch_limit</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">10</span></code></p></td><td style="text-align: left;"><p><span leaf="">单个源每轮最多取多少条</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">collect.keep_days</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">60</span></code></p></td><td style="text-align: left;"><p><span leaf="">情报保留天数，超期自动清理，</span><code><span leaf="">0</span></code><span leaf=""> 表示不限</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">collect.auto_translate</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">true</span></code></p></td><td style="text-align: left;"><p><span leaf="">入库时自动译为中文</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">nuclei.repo</span></code><span leaf=""> / </span><code><span leaf="">nuclei.ref</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">projectdiscovery/nuclei-templates</span></code><span leaf=""> / </span><code><span leaf="">main</span></code></p></td><td style="text-align: left;"><p><span leaf="">上游模板库</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">nuclei.sync_enabled</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">true</span></code></p></td><td style="text-align: left;"><p><span leaf="">定时任务自动同步模板库增量</span></p></td></tr><tr><td style="text-align: left;"><p><code><span leaf="">nuclei.sync_limit</span></code></p></td><td style="text-align: left;"><p><code><span leaf="">0</span></code></p></td><td style="text-align: left;"><p><span leaf="">单次同步入库上限，</span><code><span leaf="">0</span></code><span leaf=""> 表示不限</span></p></td></tr></tbody></table>  
所有配置项均可通过环境变量覆盖，前缀 AI_BAOBAO_  
（如 AI_BAOBAO_LLM_KEY  
），完整映射见 backend/config.py  
。  
> config.yaml  
 含明文密钥，已列入 .gitignore  
。请勿提交。  
  
## 项目结构  
```
backend/
  main.py         FastAPI 入口与路由、定时调度
  config.py       配置加载 / 校验 / 原子写回
  crawler.py      漏洞文章爬虫：RSS 聚合 + 正文抓取 + PoC 提取
  sources.py      数据源聚合：文章源 + nuclei 模板源
  source_discovery.py  新社区接入：feed 地址探测 + AI 判活
  analyzer.py     AI 生成层：情报补全、PoC 生成、模板同步编排
  content_judge.py  内容性质判定：让 LLM 分清漏洞情报与技术文章
  translate.py    情报中文化与批量回填
  nuclei_tpl.py   PoC → nuclei 模板生成（结构化 JSON + YAML 组装）
  nuclei_sync.py  nuclei-templates git 增量同步
  store.py        文件存储层
  llm.py          OpenAI 兼容客户端
  security.py     出站安全网关（SSRF 防护、XML 解析加固、限流）
  updater.py      自更新：查 GitHub 新版本 + 快进合并
  data/           运行时数据（已忽略）
frontend/
  src/
    ListView.vue      漏洞列表
    DetailView.vue    漏洞详情
    OverviewView.vue  数据总览（条形图 + 环状图）
    TemplatesView.vue 模板库
    SettingsView.vue  在线设置
```  
```

```  
## 工作流程  
```
RSS 源 ──► crawler ──► content_judge ──► analyzer ──► store ──► translate
             │              │                │                      │
             │              │                └──► nuclei_tpl ──► data/nuclei_tpl
             │              └──► LLM识别→非漏洞文章 → 直接丢弃
             └──► nuclei_sync ◄── nuclei-templates
```  
```

```  
  
每轮任务开始时先同步上游模板库增量，再跑文章流水线，最后按 keep_days  
 清理过期条目。抓取与模板同步各自加锁，避免手动触发与定时任务撞车。  
## nuclei 双向通道  
  
**导出**  
 — 把收录漏洞的原理、影响与 PoC 转成 nuclei v3 模板。详情页可单条重新生成，也可通过接口触发；默认只产出只读探测模板。  
```
curl -X POST localhost:8100/api/vulns/<ID>/nuclei \
  -H 'Content-Type: application/json' -d '{"allow_write": false}'
```  
```

```  
  
**导入**  
 — 定时增量同步上游 nuclei-templates，也可手动触发或按条件拉取入库。模板汇总在 data/nuclei_tpl/  
，可直接 nuclei -l  
 执行。  
```
curl localhost:8100/api/nuclei/sync                                  # 同步状态
curl -X POST localhost:8100/api/nuclei/sync                          # 手动触发增量同步
curl 'localhost:8100/api/nuclei/remote?limit=20&severity=critical'  # 预览，不落盘
curl -X POST localhost:8100/api/nuclei/pull -d '{"limit": 20}'      # 拉取入库
```  
```

```  
## 接入新漏洞源  
##   
  
设置页填一个社区首页即可，系统自动完成探测与判活：先按常见路径（/feed  
、/rss  
、/rss.xml  
、/atom.xml  
 …）逐个试，解析得出条目的即为 feed；再把最新几条标题交给 AI 判断这是不是漏洞技术社区。确认后落入 sources.custom  
，下一轮抓取即生效，不必重启。  
```
# 探测（不写配置，先看结果）
curl -X POST localhost:8100/api/sources/discover \
  -H 'Content-Type: application/json' -d '{"url": "https://xz.aliyun.com"}'
# 接入（name 唯一，重复即视为更新）
curl -X POST localhost:8100/api/sources \
  -H 'Content-Type: application/json' \
  -d '{"name": "先知", "url": "https://xz.aliyun.com/feed", "enabled": true}'
# 查看已接入的源 / 移除
curl localhost:8100/api/sources
curl -X DELETE localhost:8100/api/sources/先知
```  
```

```  
  
几点说明：  
- **内置源与自定义源叠加抓取**  
，自定义源每轮现读配置，改完不用重启。  
  
- **AI 判活不可用时**  
自动退化为标题关键词匹配，宁可放过也不误杀——把正常社区挡在门外的代价远高于放进一个无关站点。  
  
- **JS 渲染的站点**  
探测不到 feed（页面由脚本生成），可先在浏览器里找到它的 RSS 地址再直接填进上面的 POST /api/sources  
。  
  
- **平台型 API**  
（OSV / GitHub Advisory / NVD 这类 JSON 接口）暂未实现，扩展点见 source_discovery.py  
 末尾的 _API_ADAPTERS  
。  
  
## API  
<table><tbody><tr><th style="text-align: left;"><p><span leaf="">方法</span></p></th><th style="text-align: left;"><p><span leaf="">路径</span></p></th><th style="text-align: left;"><p><span leaf="">说明</span></p></th></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns</span></code></p></td><td style="text-align: left;"><p><span leaf="">漏洞列表（分页）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/{id}</span></code></p></td><td style="text-align: left;"><p><span leaf="">漏洞详情（含 PoC 与模板全文）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/{id}/poc</span></code></p></td><td style="text-align: left;"><p><span leaf="">生成 / 重新生成 Python PoC</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/{id}/translate</span></code></p></td><td style="text-align: left;"><p><span leaf="">单条情报译为中文</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/{id}/nuclei</span></code></p></td><td style="text-align: left;"><p><span leaf="">生成 nuclei 模板</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/purge</span></code></p></td><td style="text-align: left;"><p><span leaf="">按 </span><code><span leaf="">keep_days</span></code><span leaf=""> 清理过期条目</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/vulns/recheck</span></code></p></td><td style="text-align: left;"><p><span leaf="">存量内容复检：非漏洞条目删除、兜底条目重写</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/stats</span></code></p></td><td style="text-align: left;"><p><span leaf="">库内统计（总数 / 近 7 天 / 带模板 / 已中文化）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/translate/backfill</span></code></p></td><td style="text-align: left;"><p><span leaf="">批量翻译（</span><code><span leaf="">ids</span></code><span leaf=""> 可指定范围）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/collect</span></code></p></td><td style="text-align: left;"><p><span leaf="">手动触发一轮抓取</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/collect/status</span></code></p></td><td style="text-align: left;"><p><span leaf="">抓取状态与下次执行时间</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/update/check</span></code></p></td><td style="text-align: left;"><p><span leaf="">检查 GitHub 上有没有新版本</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/update/apply</span></code></p></td><td style="text-align: left;"><p><span leaf="">拉取最新版本（只快进，本地有改动会拒绝）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET/POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/nuclei/sync</span></code></p></td><td style="text-align: left;"><p><span leaf="">模板库同步状态 / 触发同步</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/nuclei/templates</span></code></p></td><td style="text-align: left;"><p><span leaf="">本地模板库列表</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/nuclei/remote</span></code></p></td><td style="text-align: left;"><p><span leaf="">预览上游模板库</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/nuclei/pull</span></code></p></td><td style="text-align: left;"><p><span leaf="">从上游拉取入库</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET/POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/settings</span></code></p></td><td style="text-align: left;"><p><span leaf="">读取 / 保存配置（Key 脱敏）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/settings/test</span></code></p></td><td style="text-align: left;"><p><span leaf="">用当前配置测试 LLM 连通性</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">GET</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/sources</span></code></p></td><td style="text-align: left;"><p><span leaf="">已接入的源（内置 + 自定义）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/sources/discover</span></code></p></td><td style="text-align: left;"><p><span leaf="">探测社区 feed 地址并 AI 判活（不落盘）</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">POST</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/sources</span></code></p></td><td style="text-align: left;"><p><span leaf="">接入 / 更新一个源</span></p></td></tr><tr><td style="text-align: left;"><p><span leaf="">DELETE</span></p></td><td style="text-align: left;"><p><code><span leaf="">/api/sources/{name}</span></code></p></td><td style="text-align: left;"><p><span leaf="">移除一个源</span></p></td></tr></tbody></table>## 数据存储  
```
backend/data/
├── vulns/{ID}/
│   ├── meta.json       名称、来源、简介、原理、攻击范围、参考链接
│   ├── poc.py          PoC 文本
│   └── nuclei.yaml     对应模板（如有）
├── nuclei_tpl/         全部模板汇总目录，可直接 nuclei -l 执行
├── nuclei_sync.json    模板库同步状态
└── nuclei_cache/       远程模板库索引缓存
```  
```

```  
## 安全声明  
  
本项目仅用于防御性安全研究与授权测试。生成的 PoC 依据公开披露的漏洞思路编写，使用前请确认已获得目标系统所有者授权。  
  
