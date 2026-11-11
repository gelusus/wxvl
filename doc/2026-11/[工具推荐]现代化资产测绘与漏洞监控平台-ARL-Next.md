#  [工具推荐]现代化资产测绘与漏洞监控平台-ARL-Next  
 华盟信安   2026-11-11 03:05  
  
免责声明  
```
由于传播、利用本公众号所提供的信息而造成
的任何直接或者间接的后果及损失，均由使用
者本人负责，公众号陌笙不太懂安全及作者不
为此承担任何责任，一旦造成后果请自行承担！
如有侵权烦请告知，我们会立即删除并致歉，谢谢！
```  
  
工具介绍  
  
ARL-Next 是基于经典开源项目 ARL (资产侦察灯塔) 进行架构重构与持续演进的资产测绘平台。 针对大规模扫描场景下的任务队列阻塞、长时间运行内存增长以及历史依赖过时等工程瓶颈，系统实施了微服务拆分与异步任务解耦，全面升级现代运行时环境与测绘工具链，并集成了 Model Context Protocol (MCP) 标准协议，支持自动化资产透视与 AI 协同分析。  
### 核心特性矩阵  
  
🤖   
**MCP 标准协议集成**  
：提供基于 Python 标准库的原生 MCP Server，仅注册单一工具 get_asset_profile，支持外部 Agent 通过标准协议检索单标的多维资产画像（任务下发与策略调度能力持续接入中）。（👉   
MCP 配置指南  
）  
  
⚙️   
**异步解耦并发调度**  
：微服务拆分与独立任务队列分流，结合批量数据落库优化，保障高吞吐扫描平稳运行。  
  
🌐   
**多级资产组与深度融合**  
：企业资产查询深度融入资产组管理架构，支持集团多级架构管理、多类型 ICP（网站 / APP / 小程序 / 快应用）与生态资产自动化关联及网络暴露面统一测绘。  
  
🔍   
**现代资产测绘基准**  
：基于 Nmap 7.95 稳定版；内置原生 Python 敏感凭据与信息流式提取引擎。  
  
🛡️   
**漏洞与代码泄露监控**  
：持续同步权威 CVE 漏洞情报，监听 GitHub 关联代码与凭据泄露，支持基于原子更新的去重机制。  
  
⚡   
**高可用运维与容错机制**  
：集成容器健康探活、就绪轮询与异常自动重启机制，支持管理后台在线平滑升级。  
  
📦   
**预构建镜像加速**  
：提供阿里云国内镜像源预编译构建制品，规避跨国网络波动导致的镜像拉取与依赖安装失败。  
## 界面预览  
  
**全局仪表盘 (Dashboard)**  
：实时呈现资产暴露面态势、今日侦察动向、漏洞风险矩阵与威胁情报；集成系统资源占用（CPU/内存/磁盘）与扫描调度队列监控、风险演进趋势分析、全景安全调度日志流、Web 组件指纹 TOP 5 分布及实时侦察流水线追踪。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboS4KUCLDQ3bR15Zw3SENCUYwzDAElUNuRup22Xk7iafc0DrHdD9XkYE1JwEicRUPnF1bWoyNxMyicKVFCColAftyJndunBE2AtWG8/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=0 "")  
  
**多级资产组与全景资产画像**  
：原企业资产查询现已深度整合至资产组统一架构。支持集团多级组织架构管理、企业生态资产关联分析 (OSINT)、网络暴露面测绘 (ASM) 与全景资产画像下钻（画像抽屉聚合 13 个维度；「资产搜索」页则为 12 个页签维度，不含 Nuclei 结果）。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQsOCTVMatLvvfXhLVG9m0FNgr7oRG6g8ng0PIqiaszBTGT2ibxHFP4qtScOctW0j7biaRSNkfTia37fSKAEFNa6KPyERYfU9ySjdc/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=1 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRvCgzVzItqn5pOdGA6mQ6T8q0mJT1RK5ttFZSGm52QrJFmmib7yfveFvrWMM7jQ4oG2J9iaUETr49sP8XGno5LK7ic290mUQhib5g/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=2 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTtKnF7qC7rDKNtiaFnTr1MZIrcR4ggibDicWNEgvicK4VYoomLxFP4t0TmpA9VF2YUy3jyMRZswJyyI7icOphgWGJYn1FuVOmicsj0Q/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=3 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQda216U6HDwo4CeibpGZ5SwImu6FhTmrx65NcU7rwt1AWsMCsmIKdSxeU6kLmHDI4sLoo5PIicluXO759jDP2WMcJjCkPhZlxcU/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=4 "")  
  
**任务下发与策略调度**  
：深度联动 Nuclei、基于 Python 原生引擎的多类型插件池（PoC 验证 / 弱口令爆破 / 服务识别等）及 8800+ 条 Web 指纹库，支持任务下发、状态轮转与全流程追踪。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTP52YlMcb6fQnRITZCE09rv6G1ia0SOh49icKibnMVlrFGYY6tKV0b7eicEDc3bAt2Wkd3qYYJPZOL2Ra87K6NZiaOPmBGuSBXxIwY/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=5 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSz2VDznib619srGNxZl1gH84S0aXvKJgibIRFV7TIfCiciarMTI6NfTqbRDQ101My1VM4bgDw7JssTbh6Qq7vXpNujxNFo3tdGMDc/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=6 "")  
  
**威胁情报与代码泄露雷达**  
：实时追踪全球最新 CVE 漏洞，自动化监听 GitHub 敏感代码与凭证泄露。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboR02bXIib4ibicM8lM8GjARZdYj53aBuQH6KFqZDkhaUgWOv3xBiaXxXF4FiaAvgrhQTmOgAJNbPBR3jfGeUbuGZUNoObsovd7j598A/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=7 "")  
  
**系统设置与现代字典管理**  
：全新现代三栏式字典管理（支持在线预览/呼吸灯提示）、队列并发热扩缩容及告警通道。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRo2m97YXWGO7FLxrQuJaVDjVBH1rW3EBc9NyjJVEWIXOO47hZd7RUeNG8z8qJd8rcSDXQS3Pfia1YXVicrsETdxOUhOlUT69pN4/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&watermark=1#imgIndex=8 "")  
## 部署指南  
### 生产部署 (推荐)  
  
**适用环境**  
：公网云服务器 (Ubuntu / Debian / CentOS / 统信等)、企业内网服务器。  
**推荐配置**  
：4 核 4G 及以上（保证多任务并发与大资产巡航稳定性）。  
**预估耗时**  
：自带 Docker 环境通常在数分钟内完成；裸机需额外安装 Docker 与配置 Swap，耗时视宿主机与网络状况而定。  
  
**核心机制与优势**  
：  
  
⚡   
**国内镜像加速**  
：直连阿里云国内镜像私库，规避跨国网络超时与拉取阻断。  
  
📦   
**开箱即用**  
：预构建交付，无需在宿主机配置复杂编译链与工具库。  
  
🛡️   
**双层安全防护**  
：内置 SSL 证书，  
**Basic Auth 前置网关防护默认关闭**  
（可在顶部导航栏开启），核心组件默认私有网络隔离。  
  
🔄   
**后台平滑热更**  
：可在管理后台触发在线升级流程，自动同步编排脚本与最新镜像。  
  
🩺   
**服务健康探活**  
：内置 API 就绪轮询机制与内存防护配置，服务就绪再放行，保障启动稳定性。  
#### 🚀 部署方式选择  
  
**绝大多数场景直接执行下方「方式一」即可**  
，无需预先判断系统类型或网络环境——脚本会自动探测系统、安装缺失依赖并调优内核。  
##### 方式一：国内镜像快速部署（推荐）  
  
在服务器终端（需 root 权限）粘贴执行下方命令。命令从阿里云国内镜像提取部署编排，  
start-prod.sh  
 会自动配置 Swap、安装/补齐 Docker 与 Compose 插件并拉起服务：  
```
mkdir -p ~/ARL-Next && cd ~/ARL-Next && \
docker pull crpi-laul1izptqrf0tkf.cn-beijing.personal.cr.aliyuncs.com/owl234-arl-prod/arl-web:latest && \
docker rm -f arl-temp 2>/dev/null || true && \
docker create --name arl-temp crpi-laul1izptqrf0tkf.cn-beijing.personal.cr.aliyuncs.com/owl234-arl-prod/arl-web:latest && \
docker cp arl-temp:/code/start-prod.sh ./ && \
docker cp arl-temp:/code/docker-compose.prod.yml ./ && \
docker cp arl-temp:/code/updater ./ && \
docker cp arl-temp:/code/version.txt ./ && \
docker cp arl-temp:/code/CHANGELOG.md ./ && \
mkdir -p ./frontend && docker cp arl-temp:/code/frontend/default.conf.prod ./frontend/ && \
docker rm -f arl-temp && \
chmod +x start-prod.sh && bash start-prod.sh
```  
  
**方式二：完整命令部署（按系统类型手动执行）**  
  
下面两条命令的差异  
**仅在第一段的基础工具安装方式**  
（apt-get 与 yum 二选一），第二段完全一致。  
  
**第一步：安装基础工具并创建目录**  
  
Ubuntu / Debian / 统信：  
```
apt-get update && apt-get install -y docker.io openssl curl && \
(systemctl enable --now docker 2>/dev/null || service docker start 2>/dev/null || true)
```  
  
CentOS / RHEL / Alibaba Cloud Linux / TencentOS：  
```
yum install -y openssl curl && \
(command -v docker &>/dev/null || curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun) && \
systemctl enable --now docker 2>/dev/null || true
```  
  
第二步：拉取部署编排并启动（所有系统一致）  
```
mkdir -p ~/ARL-Next && cd ~/ARL-Next && \
docker pull crpi-laul1izptqrf0tkf.cn-beijing.personal.cr.aliyuncs.com/owl234-arl-prod/arl-web:latest && \
docker rm -f arl-temp 2>/dev/null || true && \
docker create --name arl-temp crpi-laul1izptqrf0tkf.cn-beijing.personal.cr.aliyuncs.com/owl234-arl-prod/arl-web:latest && \
docker cp arl-temp:/code/start-prod.sh ./ && \
docker cp arl-temp:/code/docker-compose.prod.yml ./ && \
docker cp arl-temp:/code/updater ./ && \
docker cp arl-temp:/code/version.txt ./ && \
docker cp arl-temp:/code/CHANGELOG.md ./ && \
mkdir -p ./frontend && docker cp arl-temp:/code/frontend/default.conf.prod ./frontend/ && \
docker rm -f arl-temp && \
chmod +x start-prod.sh && bash start-prod.sh
```  
#### 🔑 访问与初始凭据  
  
部署完成后，在浏览器中访问：https://<你的服务器IP>:5173（首次自签名证书请无视浏览器不安全提示）。  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n149" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 74.2375px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">验证层级</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n150" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 49.8625px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">默认账号</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n151" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 60.1375px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">默认密码</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n152" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 133.163px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n154" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 74.2375px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">第一层：Nginx Basic Auth 网关</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n155" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 49.8625px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">admin</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n156" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 60.1375px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">arl_next</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n157" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 133.163px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">前置防扫描器爆破（</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">默认关闭</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，可在顶部导航栏安全开关处开启/关闭；修改密码需先关闭再重新开启）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n159" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 74.2375px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">第二层：ARL-Next 系统登录</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n160" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 49.8625px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">admin</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n161" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 60.1375px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">arlpass</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n162" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 133.163px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">平台主账号，首次登录后建议立即修改</span></span></span></td></tr></tbody></table>  
工具链接  
```
https://github.com/buddingworld/ARL-Next2
```  
  
文章来源：陌笙不太懂安全  
  
