#  AI 自主渗透测试系统 多 Worker 协同探索、漏洞复测自动化闭环!  
200210xsb
                    200210xsb  乌雲安全   2026-11-27 01:03  
  
介绍  
  
ARTEX-Full是基于上游 ARTEX 二次开发的 AI 自主渗透测试衍生项目，采用 Go 后端内嵌 Next.js 前端架构，依托多 Agent 与 LLM 驱动自动化探索流程。内置资产图谱、流量录制、OOB 带外监听、MCP 工具接入、漏洞复测等能力，可对接 Nuclei、  
Subfinder  
 等工具，支持 ScopeSentry 资产同步与人在环路审批。提供 Docker、预编译包、源码编译多种部署方案，自带一键更新与回滚机制。项目遵循 AGPL-3.0 协议，仅限安全研究学习，禁止未授权测试。  
## 功能概览  
  
单二进制 4 后端内嵌 Next.js 前端，PostgreSQL 存数据，核心是 LLM 多 agent 驱动的自主渗透测试流程：  
- **任务 / 资产 / 覆盖图**  
：按目标建任务，资产按公司范围归并，覆盖图用节点折叠与「已测高亮」看进度  
  
- **执行过程**  
：会话 + 工具调用逐条留痕，planner 多轮共享 todolist 稳定推进攻击链  
  
- **流量录制**  
：内置代理（:8788  
）旁路抓包，请求/响应可关联到漏洞作为证据  
  
- **人在环路**  
：拦截审批、对话追问，风险操作不自动放行  
  
- **LLM 池 / Agent 管理**  
：多模型按任务分配，Token 消耗在仪表盘可见  
  
- **扩展能力**  
：OOB 带外回连、MCP 工具接入、资产同步（ScopeSentry）  
  
## 资产同步（ScopeSentry）  
  
支持从 ScopeSentry 直接同步资产数据，免去重复收集：  
- 在「**资产同步**  
」页填 ScopeSentry 的地址与 API Key，接入数据源；  
  
- 按**项目**  
或**任务**  
维度选择要同步的目标与资产类型（域名 / 子域 / IP / 端口 / 站点 / 端点…）；  
  
- 一键导入并按公司资产范围归并，直接进入 ARTEX 的资产图供 agent 探索使用。  
  
## 安装  
  
**环境要求**  
：① Docker 方案 —— Docker + Compose v2（docker compose version  
 能跑即可）； ② 本地编译 —— **Go ≥ 1.26.3**  
（go.mod  
 的要求，低了会直接拒绝编译）、**Node ≥ 20**  
（前端静态导出用）。  
  
依赖数据库 **PostgreSQL**  
（Docker 方案自带给容器，本地编译需自备 16+）；探索需配置 **LLM**  
（ANTHROPIC_API_KEY  
 或 OPENAI_API_KEY  
，也可在 UI 里配）。  
  
国内网络提示：Docker 首次构建要拉 node:20  
 / golang:1.26  
 / python:3.12  
 三个基础镜像，拉不动请先给 Docker 配镜像加速器（/etc/docker/daemon.json  
 的 registry-mirrors  
）；npm  
 与 Go 模块已在 Dockerfile 里默认走国内源。  
  
**Docker 首次构建的资源要求**  
：前端静态导出 + Go 静态编译 + Playwright 装 chromium 会同时跑在容器里， 内存建议 **≥ 4 GB**  
（3.6 GB 的小机器实测会被拖死），磁盘预留 8 GB 左右。 机器吃不消就走**方式零（一键部署，直接下预编译包，不编译）**  
。  
### 方式零：一行命令部署（最省事，约 2 分钟）  
```
curl -fsSL https://raw.githubusercontent.com/200210xsb/artex-full/main/deploy.sh | sudo bash
```  
  
全自动、幂等，**重跑即升级**  
（配置与数据都保留）。脚本做这些事：  
- 检查系统、补 curl unzip  
；需要时装 Docker（只用来跑 PostgreSQL，ARTEX 本体是单二进制）  
  
- 从 Releases 下载对应架构的包并校验 SHA256（不编译，14 MB，秒下）  
  
- 装到 /opt/artex  
；已有的 config.json  
 / data/  
 / skills/  
 一律保留  
  
- 起 PostgreSQL（复用已有容器，或新建一个；密码随机生成，宿主机端口冲突自动从 5432 往后找）  
  
- 写 systemd 服务并启动，等 /api/health  
 通过才返回  
  
- 打印访问地址、日志/重启/升级/卸载命令  
  
完成后打开 http://<公网IP>:8787  
，首次进入会跳 /setup  
 设管理员密码。  
```
# 换端口，装完顺手铺 MCP 工具链（nuclei / subfinder / 模板）curl -fsSL .../deploy.sh | sudo bash -s -- --port 9000 --with-mcp

# 用已有的 PostgreSQL，不起容器curl -fsSL .../deploy.sh | sudo bash -s -- --dsn 'postgres://user:pass@host:5432/artex?sslmode=disable'

# 内网 / 离线：用本地包sudo bash deploy.sh --version v0.4.0 --zip /path/artex-0.4.0-linux-amd64.zip

# 卸载（加 --purge 连安装目录、容器、数据卷一起删）sudo bash deploy.sh --uninstall --purge
```  
参数说明默认--dir安装目录/opt/artex--port对外端口（代理端口自动取 +1）8787--version版本号latest--dsn用已有 PostgreSQL自动起容器--zip本地 / 内网镜像的包从 GitHub 下--servicesystemd 服务名artex--with-mcp装完执行 mcp/install-mcp.sh不装--no-start只铺文件，不注册服务—--uninstall / --purge卸载 / 连数据一起删—> 只支持 Linux，需要 root。没有 systemd 的环境（容器 / WSL）会自动退化成 nohup  
 后台启动。  
  
### 方式一：交互式安装脚本  
```
git c
```  
```
git clone https://github.com/200210xsb/artex-full.gitcd artex-full./install.sh
```  
  
测 / 自动安装 Docker → 让你选 **① 全部 Docker**  
 或 **② 本地编译运行**  
：  
- **① 全部 Docker**  
：填一个 Postgres 密码（可回车随机）→ 自动写 .env  
 → docker compose up -d  
。  
  
- **② 本地运行**  
：选数据库（连已有 / 用 Docker 起一个）→ 生成 config.json  
 → go  
 编译内嵌单二进制 → 启动。  
  
装好后打开 **http://localhost:8787**（首次进入/setup  
 设置管理员密码）。  
### 方式二：Docker Compose（手动）  
```
git
```  
```
git clone https://github.com/200210xsb/artex-full.gitcd artex-fullcp .env.example .env          # 填 POSTGRES_PASSWORD、可选 ANTHROPIC_API_KEYdocker compose build          # 本地构建镜像，不依赖任何外部镜像源docker compose up -d          # 启动 artex + postgres# → http://localhost:8787
```  
> 镜像由本仓库源码就地构建（docker compose build  
），**不拉取任何第三方预编译镜像**  
， 因此不存在上游镜像被删导致拉取失败的问题。 Dockerfile 为多阶段构建（前端静态导出 → Go 静态编译 → 运行镜像），默认走国内友好依赖源，可加 --build-arg GOPROXY=... / --build-arg NPM_REGISTRY=...  
 覆盖。  
  
  
镜像已含常用工具（ripgrep/curl/vim/npm/nmap…）；./skills  
 与 ./data  
 以绑定挂载持久化。  
  
远程 MCP 可在系统设置中选择 http  
（Streamable HTTP）或 sse  
（旧版 SSE）。 旧版 SSE 服务通常使用 GET /sse  
 建立事件流，再通过服务返回的 /message?sessionId=...  
 接收 JSON-RPC 请求；配置时将 URL 填为 /sse  
，请求头按 Authorization=Bearer <token>  
 填写。  
### 方式三：下载预编译二进制（Releases，最快，约 10 分钟）  
  
包里有：artex  
、start.sh  
（Windows 为 start.bat  
）、skills/  
、config.example.json  
、mcp/  
（外挂 MCP 工具链）。  
  
**第 1 步 · 准备 PostgreSQL 16+**  
（ARTEX 不内嵌数据库）  
```
# 已有 P
```  
```
# 已有 PG 就跳过。没有的话用 Docker 起一个：docker run -d --name artex-pg --restart unless-stopped \  -e POSTGRES_USER=artex -e POSTGRES_PASSWORD='换成你的密码' -e POSTGRES_DB=artex \  -p 127.0.0.1:5432:5432 -v artex-pgdata:/var/lib/postgresql/data postgres:16-alpine
```  
  
**· 下载并解压**  
  
到 Releases 下载对应平台的 zip：  
```
unzip 
```  
```
unzip artex-0.4.0-linux-amd64.zip && cd artex-0.4.0-linux-amd64chmod +x artex start.sh
```  
### · 写 config.json  
```
cp
```  
```
cp config.example.json config.json
```  
  
database  
 里的 host / port / user / password / dbname  
 改成第 1 步的库；也可以删掉分字段、只留 database.dsn  
 一个完整连接串。LLM 的 key 不用写这里，启动后在页面里配。  
  
**第 4 步 · 启动**  
```
./start.sh                            # 前台运行；后台常驻：nohup ./start.sh >artex.log 2>&1 &
```  
  
  
打开 http://localhost:8787  
，首次进入 /setup  
 设置管理员密码。  
  
**第 5 步 · 配 LLM**  
  
页面「系统设置」里填 ANTHROPIC_API_KEY  
 或 OPENAI_API_KEY  
（也可以用环境变量给进程）。 没有 key 能登录、能建任务，但探索跑不动。  
  
**第 6 步 ·（强烈建议）装 MCP 工具链**  
```
cd mcpsudo ./install-mcp.sh
```  
  
  
它会：铺 5 个 MCP 脚本 → 下载 nuclei / subfinder / nuclei-templates → 生成 provider-config 模板 → 注册进数据库并授权全部 agent。详见「MCP 工具链」。  
  
**第 7 步 · 验证**  
```

```  
```
export DSN='postgres://artex:你的密码@127.0.0.1:5432/artex?sslmode=disable'psql "$DSN" -c 'select key,builtin from agents order by id'          # 内置 agent（含提示词）psql "$DSN" -c 'select nkey,name from task_templates order by id'    # 新库应看到 6 条内置模版psql "$DSN" -c 'select name,enabled from mcp_servers order by name'  # 装了 MCP 才有这几行psql "$DSN" -c 'select s.name,count(*) from mcp_tools_cache c                join mcp_servers s on s.id=c.server_id group by 1'   # agent 领到意图后才有数据
```  
> 请用 start.sh  
 / start.bat  
 启动，不要直接跑 ./artex  
。它是守护脚本：进程退出后按退出码 决定是否重新拉起，**页面上的一键更新靠它完成换装**  
；直接跑 ./artex  
 更新完就不会再被拉起了。  
  
> **关于 Webshell**  
：MCP 服务端脚本随包（mcp/webshell/  
），页面与 MCP 的会话管理功能开箱可用； 但它的 payload 库（可直接落地的马）**不入库、也不随包分发**  
——payload_list  
 初始是空的。 把你的 payload 放进数据目录的 payloads/webshell/<php|jsp|aspx>/  
 即可，详见 webshell/README.md  
。  
  
### 方式四：从源码编译单二进制  
```
# 
```  
```
# 1) 前端静态导出cd web && npm ci && npm run build:static && cd ..# 2) 拷进内嵌目录cp -r web/out server/webui/dist# 3) 编译（-tags embedui 才内嵌前端）CGO_ENABLED=0 go build -tags embedui -o artex ./cmd/artex./start.sh
```  
### ：构建跨平台 Release 压缩包  
### build.sh 会先构建并嵌入前端，再使用 Go linker 去除调试信息，并将发布文件压缩为 zip。Release 模式默认生成 Linux amd64/arm64、macOS amd64/arm64 和 Windows amd64 的 zip 包：  
```
./build.s
```  
```
./build.sh --release# 产物：dist/artex-0.3.3-*.zip
```  
### 进制可能与部分 Linux 内核、虚拟化环境或安全策略不兼容，因此默认不启用。可用 ARTEX_TARGETS 自定义目标；确认目标运行环境兼容时，可显式传入 --upx 进一步缩小二进制：  
```
ARTE
```  
```
ARTEX_TARGETS=linux/amd64,windows/amd64 ./build.sh --release./build.sh --target linux/amd64 --upx
```  
## 具链（补齐扫描与子域能力）  
  
ARTEX 本体是单二进制，**但扫描、子域枚举、路径探测这些能力由外挂 MCP 子进程提供**  
。 Release 包里的 mcp/  
 目录就是这套东西的源码与一键安装入口：  
MCP能力随 Release 分发NucleiPOC 模板库检索、指纹探测、实弹验证、指纹→匹配→验证一条龙✅Subfinder被动子域枚举、字典爆破、Hunter 测绘、DNS 解析、HTTP 探活✅WebPath站点路径枚举（软 404 基线 + 硬限速）✅OOB带外回连会话（需要公网可达端口）✅Webshell会话登记、命令执行、文件管理✅ 脚本随包；payload 库需自备  
安装步骤：  
```
c
```  
```
cd mcpsudo ./install-mcp.sh              # 默认装 Nuclei/Subfinder/WebPath/OOB，并注册授权./install-mcp.sh --no-download     # 离线：只铺脚本 + 注册（扫描器自备）./install-mcp.sh --only nuclei     # 只装其中一个./install-mcp.sh --dry-run         # 只打印计划，不落盘
```  
  
内拉不动 GitHub 时换基址：./install-mcp.sh --github-base https://ghfast.top/https://github.com  
。  
  
装完到页面「系统 → MCP」应能看到对应服务且已启用。不装这套的效果：平台能开、能建任务、 能对话，但 agent 手里没有扫描工具。细节与排错见 mcp/README.md。  
## 项目地址  
  
https://github.com/200210xsb/artex-full  
  
  
