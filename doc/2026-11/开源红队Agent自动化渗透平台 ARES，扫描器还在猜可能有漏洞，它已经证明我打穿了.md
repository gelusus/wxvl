#  开源红队Agent自动化渗透平台 ARES，扫描器还在猜"可能有漏洞"，它已经证明"我打穿了"  
 网安之家-CyberHomestead   2026-11-09 14:25  
  
# 扫描器还在猜"可能有漏洞"，它已经证明"我打穿了"——66 模块开源红队平台 ARES，最短路径直取皇冠资产  
> 操作员主导的红队编排与持续安全验证。证明可利用性。计算通往皇冠资产（Crown Jewels）的最短路径。交付审计就绪的证据。  
  
## ARES Logo  
  
**操作员主导的红队编排与持续安全验证**  
  
证明可利用性。计算通往皇冠资产（Crown Jewels）的最短路径。交付审计就绪的证据。为企业红队、MSSP 和安全运营中心而建——他们需要持续验证防御态势，而不是一年一次。  
  
发布版  
 · Python  
 · FastAPI  
 · React  
 · MITRE ATT&CK  
 · 许可证  
 · 测试  
  
快速开始 · 截图 · 特性 · 文档 · 安全 · 赞助  
## ARES 是什么  
  
一个自托管平台，在故障关闭（fail-closed）的作用域飞地内运行真实的攻击性验证模块，在交互式 DAG 上绘制攻击路径，并生成品牌化报告——让你的 SOC 精确看到攻击者今天能触及什么，而不是扫描器认为可能存在的问题。  
  
**不只是扫描器。**  
 ARES 只报告它在 live 目标上真正证明的东西，包括用于证明它的多阶段攻击链。  
  
为什么不用传统渗透测试或漏洞扫描器？参见市场对比矩阵。![Attack graph](https://mmbiz.qpic.cn/sz_mmbiz_png/l6gHYLEF4ibhYGFhW4dDwEoR8hJfE4zOLFvynteEnTiaiaaxENEgmZgTY9Q0ygvkxoSp1FsjSeNeckIoibvtzb4MAiaezm4icrsGZiaMWXzgVtibNW8/640?wx_fmt=png&from=appmsg "")  
  
## 截图  
  
**ARES 登录 — 龙吉祥物**  
使用 ARES 吉祥物和 SSO 后备方式登录。仅存内存的 JWT 访问令牌，HttpOnly 轮换刷新。  
  
**操作员总览**  
操作员总览 — 进行中的任务、已验证的发现、攻击面、引擎健康度、遥测脉搏。  
  
**模块目录**  
目录 — 横跨 13 个类别的 66 个活跃模块，每个都标注了 MITRE ATT&CK 技术和 OPSEC 噪声级别。  
  
**模块运行后的发现卡片**  
结果 — 发现卡片，含严重性、置信度、修复建议，以及一键跳转到下一个模块。  
  
**攻击图**  
攻击图 — 边界防火墙（火焰动画）、目标、跳转边。每个节点的只读 beacon 遥测。  
  
**任务作用域**  
任务作用域 — 目标任务选择器，恢复 Vault / 干跑计划 / 删除 控件。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/l6gHYLEF4ibgU1eREicONqxeg3YDoMBtNJOAbkSUx6UAtu11VsmPSpiaPAg8EwgVOkAdmGe8mIic46A7OJmibVMO1DVMTm73eLialSiaDNibqVl8l4c/640?wx_fmt=png&from=appmsg "")  
## 特性  
- **故障关闭作用域飞地**  
 — 双层 ScopeGuard（应用预检 + OS/传输层防火墙）保证零攻击性数据包抵达作用域外的主机。→ 安全模型  
  
- **目标导向的攻击规划**  
 — 设定 domain_admin  
 或 cloud_audit  
，决策引擎即合成多阶段执行路径（图启发式，可选的 LLM 代理需显式授权）。→ 引擎设计  
  
- **多向量攻击图（DAG）**  
 — 摄入 BloodHound 或实时映射会话。最短路径、咽喉点、横向跳转可行性。→ 攻击图  
  
- **零信任认证模型**  
 — 仅存内存的短生命周期 JWT、HttpOnly 轮换刷新、HMAC CSRF、AES-256-GCM AEAD 保险库（PBKDF2-HMAC-SHA256 600k）。→ 安全模型  
  
- **横跨 60+ MITRE ATT&CK 技术的 66 个活跃生产模块**  
（AD、云、凭据访问、渗出、横向、侦察、Windows、Linux、EDR）。→ 模块目录  
  
- **自动化交付物**  
 — 通过无头浏览器一键生成品牌化 PDF/HTML/Markdown/JSON 报告（Windows/Linux 上无 GTK 依赖）。→ 仪表盘指南  
  
- **MCP 网关**  
 — Cursor/Claude Desktop/Windsurf/Cline 可在起飞前 ScopeGuard、抗提示注入污点隔离和一次性 HMAC 确认令牌下安全驱动 ARES。→ MCP 规范  
  
- **类型安全的 Python SDK**  
 — 用 BaseModule[P, R]  
 + @ares_module  
 编写自定义模块，在 ModuleTestHarness  
 中模拟，用 AresClient  
 自动化。→ SDK 文档 · → 模块开发指南  
  
## 快速开始  
### 前置条件  
- Python 3.11 或 3.12（Windows 上推荐 3.12）  
  
- Node.js v18+（在 v20 LTS 上测试）  
  
- 操作系统 Windows 11/10、Linux（Ubuntu/Debian/Kali）或 macOS  
  
### 60 秒（Windows 上的 PowerShell）  
```
git clone https://github.com/Mafifrizi/ARES.git
Set-Location .\ARES

py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -U pip
.\.venv\Scripts\python.exe -m pip install -e ".[dev,pdf]"

Set-Location frontend ; & "C:\Program Files\nodejs\npm.cmd" ci ; Set-Location ..

$env:ARES_PDF_BROWSER = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
.\.venv\Scripts\ares.exe doctor --pdf-smoke

.\.venv\Scripts\ares.exe dashboard dev --no-reload

```  
  
打开 http://127.0.0.1:5173/dashboard/。初始操作员：admin  
。密码：在 .env  
 中设置 ARES_DEFAULT_ADMIN_PASSWORD  
（必填，最少 12 个字符，无硬编码默认值）。  
### 测试  
```
pytest tests/unit -n auto -q                    # 5,074 个测试，并行
pytest tests/integration/test_postgres_*.py -q  # Postgres 集成

```  
### 原地升级（无需 git clone / 重建）  
```
ares update              # 仅添加新模块
ares upgrade             # 全系统升级（核心 + 数据库 + 模块 + UI）
ares upgrade --check     # 预览，磁盘上不做任何更改

```  
  
用户数据库、凭据、配置和插件在数学上被列入修改黑名单 — 参见原地升级保证。  
## 文档  
  
所有深入内容都在 docs/  
 中。主要入口：  
- 快速开始任务指南  
  
- 文档门户与子系统索引  
  
- 仪表盘逐界面手册  
  
- 模块目录与 schema · 模块开发 · SDK 规范  
  
- API 参考 · MCP 网关 · 企业 SSO（SAML/OIDC）  
  
- 安全与威胁模型 · 执行策略 · 验证实验室  
  
- 架构 · 引擎设计 · Worker 架构 · 数据库迁移  
  
- 市场对比 · RFC 001：Linux AD 跨平台技艺![ARES login — dragon mascot](https://mmbiz.qpic.cn/sz_mmbiz_png/l6gHYLEF4ibgPeV4icbGx4FfriaFUwzNASShjMkAd2liaVHON5TfGXIQVp03qZhU0iameSrlgP04bOPmyVE8YHjXaey03OF4T6eS7RUialGutWxlA/640?wx_fmt=png&from=appmsg "")  
  
  
## 支持  
  
如果 ARES 加速了你的评估或保护了你的基础设施，考虑赞助持续的功能开发、威胁研究和实验室基础设施：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">渠道</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">适用地区</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">GitHub Sponsors</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">国际（周期性 / 一次性）</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Saweria</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">印度尼西亚（QRIS、GoPay、OVO、Dana）</span></section></td></tr></tbody></table>  
## 致谢  
- Muhamad D Alvito  
  
- Muhamad D Alvito — UI 与 UX 合作伙伴  
  
## 负责任使用与免责声明  
> **重要**  
  
  
ARES 是一个双用途软件框架，专为授权的网络安全研究、企业内部韧性验证以及拥有显式书面许可的专业红队任务而设计。  
- 未经系统所有者事先书面授权，不得对任何网络、主机或云基础设施执行 ARES。  
  
- 未经授权的系统访问或测试违反国家和国际网络犯罪立法（例如 18 U.S.C. § 1030《计算机欺诈和滥用法》）。  
  
- ARES 的创建者和维护者对本软件造成的滥用、损害或监管违规不承担任何责任。  
  
- 要报告 ARES 本身的安全漏洞，请遵循安全策略。  
  
## 与任何加密货币或代币无关联  
> **注意**  
  
  
本项目是一个开源安全软件平台。它与任何使用"ARES"或类似品牌的加密货币、币、代币、NFT、空投或金融工具没有任何关联、背书、合作或联系。我们不发行、推广或背书任何数字资产。任何此类产品都与本项目无关，且在我们未参与或未批准的情况下运作。不要购买或交易任何此类资产并期望其与本仓库有关联。  
## 许可证  
  
ARES 在开源 MIT 许可证下分发。  
  
**ARES — 现代红队任务与持续安全验证系统。**  
持续安全验证。确定性的作用域收敛。  
## 关于  
  
ARES - 授权红队任务自动化，配备仪表盘、任务作用域、模块编排、OPSEC 控制、加密保险库和报告。  
## 项目树状索引  
### 仓库结构  
```
Mafifrizi/ARES  (Public · v6.0.0)
├── .github/                  # Comprehensive audit: fix real bugs + gate ruff/mypy/bandit in CI (#62)
├── ares/                     # feat(dashboard): enterprise redesign — honest UI, restrained design s…
├── docker/                   # refactor(overview): redesign dashboard, clean redundant data, and rem…
├── docs/                     # feat(dashboard): enterprise redesign — honest UI, restrained design s…
├── example_modules/          # refactor(overview): redesign dashboard, clean redundant data, and rem…
├── frontend/                 # feat(dashboard): enterprise redesign — honest UI, restrained design s…
├── migrations/               # Implement C-LIVE-v1 execution lifecycle
├── scripts/                  # Comprehensive audit: fix real bugs + gate ruff/mypy/bandit in CI (#62)
├── tests/                    # feat(dashboard): enterprise redesign — honest UI, restrained design s…
├── .env.example              # feat(security): implement dual-layer OS scope firewall and standardiz…
├── .gitignore                # fix(ci): fix proc_keys test mock and remove internal audit report fro…
├── .pre-commit-config.yaml   # docs: clean root config and setup guidance
├── .secrets.baseline         # Initial public release of ARES
├── CHANGELOG.md              # docs: clean root config and setup guidance
├── CODE_OF_CONDUCT.md        # docs: add Code of Conduct, Discussion templates, and community guidel…
├── CONTRIBUTING.md           # Comprehensive audit: fix real bugs + gate ruff/mypy/bandit in CI (#62)
├── Dockerfile                # docs: remediate third-party review feedback on test architecture, doc…
├── LICENSE                   # docs: add MIT license
├── QUICKSTART.md             # feat(security): implement dual-layer OS scope firewall and standardiz…
├── README.md                 # feat(dashboard): enterprise redesign — honest UI, restrained design s…
├── SECURITY.md               # feat(security): implement Kernel Scope Wall egress firewall and upgra…
├── alembic.ini               # fix(updater): ignore non-module helper files from attack module catal…
├── ares.bat                  # docs(mcp): remove outdated mcp-console screenshot and restore standal…
├── mcp.bat                   # docs(mcp): remove outdated mcp-console screenshot and restore standal…
├── mcp.ps1                   # docs(mcp): remove outdated mcp-console screenshot and restore standal…
├── mypy-baseline.txt         # Comprehensive audit: fix real bugs + gate ruff/mypy/bandit in CI (#62)
├── pyproject.toml            # Comprehensive audit: fix real bugs + gate ruff/mypy/bandit in CI (#62)
└── requirements-lock.txt     # refactor(deps): replace netaddr with stdlib ipaddress, prune 5 unused…

```  
### 仓库元信息索引  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">索引项</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">值</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">所有者 / 仓库</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Mafifrizi / ARES</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">可见性</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Public</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">最新提交</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Delete AUDIT_FINDINGS.md (</span><code><span leaf="">d092513</span></code><span leaf=""> · 4 days ago)</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Stars</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">946</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Watchers</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">6</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Forks</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">199</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Releases</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">1（v6.0.0 · Latest · 5 months ago）</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Contributors</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">3（@Mafifrizi、@claude、@TifalAbdusyahid）</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Packages</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">No packages published</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">许可证</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">MIT</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">主要语言</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">Python 92.6% · TypeScript 6.6% · CSS 0.8% · Dockerfile 0% · Shell 0% · JavaScript 0%</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">赞助</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">GitHub Sponsors（@Mafifrizi）· Saweria（https://saweria.co/mafifrizi）</span></section></td></tr></tbody></table>  
### 导航入口索引  
- README · Code of conduct · Contributing · MIT license · Security  
  
- Quickstart · Screens · Features · Docs · Security · Sponsor  
  
  
  
  
