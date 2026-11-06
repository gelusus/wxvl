#  ARTEX-FULL 衍生加强版｜多 Agent 自主渗透系统、新增 MCP 工具链、多 Worker 协同探索、漏洞复测自动化闭环  
200210xsb
                    200210xsb  渗透安全HackTwo   2026-11-06 00:03  
  
0x01 工具介绍  
  
ARTEX-Full 是基于上游 ARTEX 二次开发的 AI 自主渗透测试衍生项目，采用 Go 后端内嵌 Next.js 前端架构，依托多 Agent 与 LLM 驱动自动化探索流程。内置资产图谱、流量录制、OOB 带外监听、MCP 工具接入、漏洞复测等能力，可对接 Nuclei、Subfinder 等工具，支持 ScopeSentry 资产同步与人在环路审批。提供 Docker、预编译包、源码编译多种部署方案，自带一键更新与回滚机制。项目遵循 AGPL-3.0 协议，仅限安全研究学习，禁止未授权测试。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWXibaAF3RMwFX8pibW3GFQyrhXjeTO2OKGducQt5YNia7TicItamzpI7h9KTrzY1V2vT4kcjAHcJANDJqqrcMLPEKHvqcp7vpFribM/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
注意：  
现在只对常读和星标的公众号才展示大图推送，建议大家把  
**渗透安全HackTwo**  
"**设为****星标⭐️**  
"  
**否****则可能就看不到了啦！**  
  
**下载地址在末尾 #渗透安全HackTwo**  
  
0x02   
功能介绍  
  
✨核心特点  
## MCP 工具链（补齐扫描与子域能力）  
  
**ARTEX 本体是单二进制，但扫描、子域枚举、路径探测这些能力由外挂 MCP 子进程提供。 Release 包里的 mcp/ 目录就是这套东西的源码与一键安装入口：**  
<table><thead><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">MCP</span></span></section></th><th data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">能力</span></span></section></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">随 Release 分发</span></span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><td data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf=""><span textstyle="" style="font-size: 12px;">Nuclei</span></span></code></td><td data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">POC 模板库检索、指纹探测、实弹验证、指纹→匹配→验证一条龙</span></span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">✅</span></span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><td data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf=""><span textstyle="" style="font-size: 12px;">Subfinder</span></span></code></td><td data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">被动子域枚举、字典爆破、Hunter 测绘、DNS 解析、HTTP 探活</span></span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">✅</span></span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><td data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf=""><span textstyle="" style="font-size: 12px;">WebPath</span></span></code></td><td data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">站点路径枚举（软 404 基线 + 硬限速）</span></span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">✅</span></span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><td data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf=""><span textstyle="" style="font-size: 12px;">OOB</span></span></code></td><td data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">带外回连会话（需要公网可达端口）</span></span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">✅</span></span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.909091px solid rgba(209, 217, 224, 0.7);"><td data-colwidth="106" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf=""><span textstyle="" style="font-size: 12px;">Webshell</span></span></code></td><td data-colwidth="347" style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">会话登记、命令执行、文件管理</span></span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border: 0.909091px solid rgb(209, 217, 224);"><section style="text-align: left;margin-top: 16px;margin-bottom: 16px;"><span leaf=""><span textstyle="" style="font-size: 12px;">✅ 脚本随包；payload 库需自备</span></span></section></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWquuibpqs2wCUv3YCStOyRiaI9unTLFDgkqW7TYyniad8DcaWTyicgpSFU2o7BeicxAibQz9kTo14l0KibRLbWII8Tl0pV4xOHwYvWYU/640?wx_fmt=png&from=appmsg "")  
## 新增功能  
- **任务 / 资产 / 覆盖图**  
按目标建任务，资产按公司范围归并，覆盖图用节点折叠与「已测高亮」看进度  
  
- **执行过程**  
会话 + 工具调用逐条留痕，planner 多轮共享 todolist 稳定推进攻击链  
  
- **流量录制**  
内置代理（:8788  
）旁路抓包，请求/响应可关联到漏洞作为证据  
  
- **人在环路**  
拦截审批、对话追问，风险操作不自动放行  
  
- **LLM 池 / Agent 管理**  
多模型按任务分配，Token 消耗在仪表盘可见  
  
- **扩展能力**  
OOB 带外回连SSRF、MCP 工具接入、RCE  
  
## 资产同步（ScopeSentry）  
  
支持从 ScopeSentry 直接同步资产数据，免去重复收集：  
- 在「**资产同步**  
」页填 ScopeSentry 的地址与 API Key，接入数据源；  
  
- 按**项目**  
或**任务**  
维度选择要同步的目标与资产类型（域名 / 子域 / IP / 端口 / 站点 / 端点…）；  
  
- 一键导入并按公司资产范围归并，直接进入 ARTEX 的资产图供 agent 探索使用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXGNEcQCLOapu9Ft16aRXiaiciaYORNjRqh3OuPFOo5amBhqeibGmtH5P55pIdAmyX5cbuNRkMd9jQ12ddZeseBbS4ic8jGKotcIlFY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVzq24FFqhxBdAUdLqBaoJ4w4NPRicRNluldel4lqfm3s30JM1OwjMLaEbjeL4o5bXHNkPOicEVibxnfK8Fgp2ubpbgS9gym9x1K4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXFXiawZ85iaXGygoJzLiahjibLM6yeKvUHKCib6JZxN0NDcgz6W8UJMsbVwO8HeTEqZqpCEEJDf8epoZRzcxeA5wJ9UxdK8u1LdjMQ/640?wx_fmt=png&from=appmsg "")  
  
**0x03 更新介绍**  
  
```
AI新增 mcp/：Nuclei（9 工具）/ Subfinder（5）/ Webshell / WebPath / OOB，各带 register-mcp.sh 与 README
mcp/install-mcp.sh 一键安装：--no-download、--only、--bin-dir、--github-base、--dry-run
五个 MCP 路径可移植（环境变量 > 脚本同级 > /opt/artex > PATH），日志/索引在只读挂载下自动退到临时目录
任务模版内置 6 条，首启播种，ON CONFLICT(nkey) DO NOTHING + 一次性标记，不覆盖用户改动
```  
  
  
0x04 使用介绍  
  
📦安装与使用指南  
> 升级只换程序、不动数据：Postgres 数据卷 pgdata、./data（jwt.key / SQLite 等）、./skills 都会保留。数据库迁移无需手动执行——artex 每次启动会幂等重跑 schema.sql（含 ADD COLUMN / CREATE INDEX IF NOT EXISTS），即“重启即迁移”。升级前仍建议先备份 ./data 与数据库。  
  
### 方式一：页面一键更新（推荐）  
  
在   
**系统配置**  
 页（侧边栏「系统配置」→   
/system/settings  
）的  
**版本与更新**  
卡片里，可以直接检查并安装新版本，无需登录服务器。  
  
点「更新」后：下载当前平台的发布包 → 比对 Release 的   
SHA256SUMS  
 → 用   
-h  
 冒烟测试新二进制 → 暂存为   
artex.new  
 → 程序退出，由   
start.sh  
 /   
start.bat  
 重新拉起并完成换装。页面会自动等到新版本上线后刷新。  
- **失败不会留下坏程序**  
：校验或冒烟不通过就丢弃暂存件、继续跑当前版本；换装后的新版若连续 3 次启动失败，会自动回滚到   
artex.old  
（失败的那个留作   
artex.failed  
 供排查）。  
  
- **随时可回退**  
：上一版本保留为   
artex.old  
，卡片上有「回滚到上一版本」。注意数据库结构不会回退。  
  
- **更新会中断正在运行的任务**  
——更新即重启，请在空闲时进行。  
  
- **开发构建不给更新**  
：版本号是   
dev  
 或   
git describe  
 带后缀时禁用，避免正式版覆盖掉本地调试的二进制。  
  
- **Docker 下升级要重建镜像**  
：镜像从源码构建，  
docker compose build artex && docker compose up -d artex  
 会把镜像里的 playwright / nmap 等工具链一起更新。页面上的「一键更新」只换容器内的二进制，重建容器后会退回镜像自带的版本——Docker 部署请优先用   
./update.sh  
。  
  
- 访问 GitHub 需要代理时，在同一页面配置  
**全局代理**  
即可，更新链路会走它。更新只从 GitHub 域名下载并强制 HTTPS。  
  
### 方式二：一键更新脚本  
```
cd artex-full
./update.sh
```  
  
脚本先可选   
git pull  
 拉取最新代码，再让你选   
**① Docker 更新**  
 或   
**② 本地编译更新**  
（与   
install.sh  
 对应）：  
- **① Docker**  
：  
git pull  
 取到新代码后重新构建镜像 → 重建容器（重启即自动迁移 schema）。镜像从源码构建，没有远程镜像可拉，所以这一步是 build 而不是 pull，耗时几分钟。  
  
- **② 本地**  
：重建前端静态产物 → 重新编译   
./artex  
（完成后重启进程生效）。  
  
### 方式三：Docker Compose（手动）  
```
cd artex-full
git pull                       # 取最新代码（compose / 源码 / 脚本）
docker compose build artex     # 重新构建镜像（源码就地构建，不拉远程镜像）
docker compose up -d artex     # 重建容器重启 → 自动迁移 schema
docker image prune -f          # 清理旧镜像层（可选）
```  
### 方式四：预编译二进制（Releases）  
  
到   
Releases  
 下载新版本 zip，停掉旧进程后覆盖   
artex  
 与   
skills/  
（保留你的   
config.json  
 与   
data/  
），重启即可：  
```
cp -r <解压目录>/skills ./ && cp <解压目录>/artex ./
./start.sh
```  
### 方式五：从源码编译  
  
```
git pull
cd web && npm ci && npm run build:static && cd ..
cp -r web/out server/webui/dist
CGO_ENABLED=0 go build -tags embedui -o artex ./cmd/artex
# 重启 ./start.sh
```  
  
## 配置  
  
**数据库**  
（  
config.json  
，或用环境变量   
ARTEX_PG_DSN  
 覆盖）：  
```
{
  "database": {
    "host": "127.0.0.1", "port": 5432,
    "user": "artex", "password": "yourpass",
    "dbname": "artex", "sslmode": "disable"
  }
}
```  
  
**LLM**  
：  
export ANTHROPIC_API_KEY=sk-...  
（或   
OPENAI_API_KEY  
），也可在 UI 的「LLM 配置」页填写。 可选：  
ARTEX_LLM_PROVIDER  
 /   
ARTEX_LLM_MODEL  
 /   
ARTEX_LLM_BASE_URL  
 /   
ARTEX_LLM_PROXY  
。  
  
**并发**  
：每个任务的 work agent 数在「系统设置」里配置（默认 3）。  
  
**常用参数**  
：  
./start.sh -addr :8787 -proxy :8788  
（  
-addr  
 前端+API，  
-proxy  
 流量录制代理）。启动脚本会把参数原样透传给   
artex  
。  
### 反向代理部署（HTTPS / 只开放 443）  
  
前端和 API/SSE 都由同一个后端端口（默认   
:8787  
）提供，实时活动流默认走  
**同源**  
地址，因此  
**无需配置 NEXT_PUBLIC_SSE_BASE**  
，公网只开放 443、把 8787 留在内网即可。  
  
SSE 是长连接 + 持续推送，反代  
**必须关闭缓冲**  
，否则浏览器能连上却收不到事件（表现为活动流一直转圈）。Nginx 示例：  
```
server {
    listen 443 ssl;
    server_name your.domain.com;
    # ssl_certificate / ssl_certificate_key ...
    location / {
        proxy_pass http://127.0.0.1:8787;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        # SSE 关键项：关缓冲、长超时、HTTP/1.1
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 3600s;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```  
  
> 仅当 SSE 需要走与页面不同的来源（如独立子域）时，才在  
**构建期**  
设置   
NEXT_PUBLIC_SSE_BASE  
（该变量在   
next build  
 时固化进静态包，容器运行时再设无效）。  
  
  
  
**0x05 内部VIP星球介绍-V1.5（福利）**  
  
          
如果你想学习更多**渗透测试技术/应急溯源/免杀工具/挖洞SRC赚取漏洞赏金/红队打点等**  
欢迎加入我们**内部星球**  
可获得内部工具字典和享受内部资源和  
内部交流群，  
**每天更新1day/0day漏洞刷分上分****(2026POC更新至21922+)**  
**，**  
包含全网一些**付费扫描****工具及内部原创的Burp自动化漏****洞探测插件/漏扫工具等，AI代审工具，最新挖洞技巧、SKILL等**  
。shadon/  
Hunter  
/  
0zone  
/  
Zoomeye  
/Quake/  
Fofa高级会员/AI账号  
/CTFShow等各种账号会员共享。详情点击下方链接了解，觉得价格高的师傅后台回复"   
**星球**  
 "有优惠券名额有限先到先得  
**❗️**  
啥都有  
**❗️**  
全网资源  
最新  
最丰富  
**❗️****（🤙截止目前已有3000+多位师傅选择加入❗️早加入早享受）**  
  
****  
最新漏洞情报分享：  
https://t.zsxq.com/DSAvv  
  
****  
  
**👉****点击了解加入-->>内部VIP知识星球福利介绍V1.5版本-1day/0day漏洞库及内部资源更新**  
  
****  
  
  
结尾  
  
# 免责声明  
  
  
# 获取方法  
  
  
**公众号回复20261106获取下载、回复 加群 获取交流群**  
  
****  
  
# 最后必看-免责声明  
  
  
      
文章中的案例或工具仅面向合法授权的企业安全建设行为，如您需要测试内容的可用性，请自行搭建靶机环境，勿用于非法行为。如  
用于其他用途，由使用者承担全部法律及连带责任，与作者和本公众号无关。  
本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用。  
如您在使用本工具或阅读文章的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具或文章或来源于网络，若有侵权请联系作者删除，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！  
  
  
  
# 往期推荐  
  
  
**1.内部VIP知识星球福利介绍V1.5（AI自动化）**  
  
**2.CS4.8-CobaltStrike4.8汉化+插件版**  
  
**3.全新升级BurpSuite2026.4专业(稳定版)**  
  
**4. 最新xray1.9.11高级版下载**  
  
**5. 最新HCL AppScan Standard**  
  
  
渗透安全HackTwo  
  
微信号：关注公众号获取  
  
后台回复星球加入：  
知识星球  
  
扫码关注 了解更多  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/RjOvISzUFq6qFFAxdkV2tgPPqL76yNTw38UJ9vr5QJQE48ff1I4Gichw7adAcHQx8ePBPmwvouAhs4ArJFVdKkw/640?wx_fmt=png "二维码")  
  
  
  
上一篇文章：  
[Nacos配置文件攻防思路总结|揭秘Nacos被低估的攻击面](https://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247492839&idx=1&sn=b6f091114fbd8e8922153a996c8f4f1c&scene=21#wechat_redirect)  
  
  
