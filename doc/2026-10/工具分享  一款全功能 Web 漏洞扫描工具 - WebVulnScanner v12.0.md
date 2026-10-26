#  工具分享 | 一款全功能 Web 漏洞扫描工具 - WebVulnScanner v12.0  
huocai250
                    huocai250  乌雲安全   2026-10-26 01:22  
  
介绍  
  
WebVulnScanner 是一款**检测 / 评估型**  
安全扫描器（与 OWASP ZAP、Nikto、Nuclei 同类）： 它探测目标是否存在常见漏洞、梳理攻击面并给出整改建议，**不包含利用、提权、 数据窃取或持久化功能**  
（详见「关于漏洞利用」）。请仅在已获得书面授权的目标上使用。  
  
**v12 亮点**  
：63 个检测模块 + **PoC 自动化验证引擎**  
（兼容 nuclei 模板语法），内置 **8600+ PoC 模板**  
（含 1700+ 高危/严重 CVE 验证）；默认执行内置应急集（Log4Shell / Spring4Shell / Shiro / Struts2 / ThinkPHP / Fastjson / WebLogic / Confluence 等）+ 指纹匹配的库模板，可用 --poc-all  
 全库扫描。v11 的 62 个模块与 1500+ 检测规则全部保留。  
## 检测模块（63 个）  
  
**信息收集与指纹**  
：爬虫、信息收集、指纹识别（90+ 规则）、版本漏洞提示、robots/sitemap/well-known 情报、WebSocket/实时端点发现、子域名枚举、子域名接管指纹。  
  
**配置与传输**  
：安全头、**安全头策略深度**  
（Referrer/Permissions/COOP/CORP）、SSL/TLS、CSP 深度评估、Cookie 深度分析、配置错误/点击劫持、前端安全、HTTP 方法、**动词篡改/WebDAV**  
、Host 头注入、CORS、**CORS 高级绕过**  
、CSRF、OAuth/OIDC 配置、**会话固定**  
、**敏感页缓存**  
。  
  
**信息泄露**  
：敏感信息、**全站 PII/密钥深扫**  
、JS 密钥/端点、敏感路径暴露（740+ 路径）、源码泄露、调试端点、反序列化指示、模板签名库（YAML，可扩展）、API 文档发现、CMS 专项。  
  
**注入与漏洞（覆盖完整）**  
：SQL 注入、NoSQL 注入、LDAP 注入、XPath 注入、XSS/SSTI、**DOM XSS 静态分析**  
、SSI 注入、EL/OGNL 表达式注入、LFI/命令注入、**PHP 包装器/LFI 向量**  
、路径穿越、XXE、SSRF、CRLF 注入、邮件头注入、CSV 公式注入、**HTTP 参数污染 (HPP)**  
、原型链污染、开放重定向、缓存投毒、缓存欺骗、Log4Shell、JWT 安全、GraphQL（含深度/DoS 面）。  
  
**主动扫描**  
：端口扫描（68 高危端口）、目录枚举。  
  
**v12 新增**  
：**PoC 自动化验证**  
 modules/poc.py  
 —— 8,600+ nuclei 兼容 PoC 模板 （cves / exposed-panels / misconfiguration / vulnerabilities / exposures / takeovers 等类别，已剔除 default-logins、token-spray、credential-stuffing 等口令爆破类）。  
> 重型模块请求量较大，可用 --fast  
、--skip <模块>  
 或 --max-requests N  
（请求预算）控制开销。  
  
> 启动时会显示实际加载的**内置检测规则/签名总量**  
（模板 + 敏感路径 + 指纹 + 载荷 + 端口 + 字典 + PoC），当前 **10000+**  
，且可通过模板持续扩展。  
  
## PoC 自动化验证（v12 核心）  
  
v12 内置 **nuclei 兼容的 PoC 引擎**  
（core/poc_engine.py  
）：  
- 支持 http:  
 模板：method / path / raw / headers / body / matchers / extractors  
  
- 多请求串联、{{BaseURL}}  
 等变量与 helper（{{randstr}}  
、{{base64()}}  
、 {{hex_decode()}}  
 等）  
  
- DSL 匹配器（contains()  
、len()  
、regex()  
、to_lower()  
 等，白名单求值）  
  
- payload 模糊测试（batteringram / pitchfork / clusterbomb，单模板上限 500 组合）  
  
- internal extractor 提取变量并跨请求引用、req-condition、stop-at-first-match  
  
- OOB 探测：{{interactsh-url}}  
 自动映射到 --canary  
 域名（需自建 collaborator 观测回连）  
  
- 预构建索引 pocs/index.json  
（8,600+ 模板秒级加载），tools/build_poc_index.py  
 可重建  
  
### 用法  
```
# 默认：内置应急集 + 指纹匹配的库模板（推荐，请求量小）
python main.py https://example.com

# 只做 PoC 扫描
python main.py https://example.com --poc-only

# 全库扫描（8,600+ 模板，请求量大，请配合 --rate / --max-requests）
python main.py https://example.com --poc-all --rate 20 --max-requests 5000

# 按标签/严重级/指定 id 筛选
python main.py https://example.com --poc-tags cve,rce --poc-severity critical,high
python main.py https://example.com --poc wvs-log4shell-cve-2021-44228

# 查看模板库清单
python main.py --list-pocs
python main.py --list-pocs --poc-tags thinkphp

# 追加自定义 PoC 模板（nuclei 兼容 YAML）
python main.py https://example.com --pocs my_pocs
```  
## 基础设施  
- Web UI：  
--web 启动 Flask 界面，SSE 实时进度、在线查看结果与报告  
  
- 插件系统：--plugins DIR 从目录热加载自定义 BaseScanner 子类  
  
- 批量扫描：-f targets.txt 用 asyncio 并发扫描多目标（--concurrency N）  
  
- 配置文件：--config scanner.cfg 集中管理默认参数  
  
- 多格式报告：JSON / HTML / Markdown / CSV + 风险评分 + 修复建议  
  
- 负责任扫描护栏：作用域锁定、限速（令牌桶）、礼貌延迟、被动模式  
  
## 安装与用法  
```
pip install -r requirements.txt
pip install flask        # 仅使用 Web UI 时需要

# 基本扫描（启动时显示已加载规则总量）
python main.py https://example.com

# 快速模式（跳过端口/目录/暴露/子域名等重型模块）
python main.py https://example.com --fast --html report.html

# 被动模式（仅非侵入式检测）
python main.py https://example.com --passive

# 追加自定义模板 + 插件
python main.py https://example.com --templates my_templates --plugins plugins

# 带外探测（Log4Shell/SSRF）指定你自己的 collaborator
python main.py https://example.com --canary your.collaborator.net

# 批量扫描
python main.py -f targets.txt --concurrency 5 --html out.html

# Web UI
python main.py --web        # http://127.0.0.1:5000

# 作为模块运行
python -m webscanner https://example.com
```  
  
完整参数见 python main.py -h  
。重型模块（exposure  
 / ports  
 / dirbust  
 / subdomain  
） 请求量较大，可用 --fast  
 或 --skip <模块>  
 控制。  
## 免责声明  
  
本工具仅供**授权**  
的安全测试与教育研究。使用者须确保已获得目标系统所有者的 **书面授权**  
。任何未经授权的扫描或攻击均属违法，开发者不对滥用行为负责。  
  
项目地址  
  
https://github.com/huocai250/Webscanner  
  
  
