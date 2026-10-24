#  一套面向 CNVD 通用型未授权漏洞挖掘的开源便携工具包  
 黑白之道   2026-10-24 01:29  
  
## 这是什么  
  
一套面向 **CNVD 通用型未授权漏洞**  
 的批量挖掘工作流工具包：  
- **Skill 本体**  
（skills/cnvd/SKILL.md  
）——完整流程 / 原则 / 判据 / 合规红线，不用 Claude Code 也能当操作手册人肉照着走  
  
- **9 个 Python 脚本**  
——纯标准库、零 pip 依赖，覆盖 **测绘 → 定级检测 → 查重抽验**  
 全链路  
  
- **8 文件 28 条端点字典**  
——来自实战验证的未授权端点，跨厂商可迁移（本包核心资产）  
  
- **脱敏示例 + 一键自检**  
——示例产出/  
 展示全部中间产物格式，校验.bat  
 双击完成环境自检与冒烟  
  
核心思路：**不做固件级深挖，做端点级的横向覆盖**  
。挑一个冷门产品线，用预置字典批量验证同类设备，命中 ≥10 台即够一个 CNVD 通用型漏洞的案例量。  
  
**运行语义（2.3）**  
：**无产出不停**  
 —— 唯一合法停机条件 = 已挖到 ≥1 个可提交级洞并落盘（命中 ≥10 + 查重无重复 + 复验通过 + 厂商资质达标 + 报告完成）。在此之前，换线 ≠ 收工：没出货就换下一条产品线，多条线重复就继续换，直到出货或用户喊停。合规红线在连续作业下执行强度不变。  
  
**厂商资质门槛（2.3）**  
：CNVD 发证硬门槛 = 通用型 + 厂商**注册资本实缴 > 5000 万人民币**  
 + 优先国内。甜点区 = **注册资本过线但知名度低的中型/小厂**  
——大厂是重复重灾区（白干），小厂不够发证资格。准备阶段须核验厂商工商信息（§1.1a），报告须含「厂商信息」段。  
  
**复现规范（2.3 · 反虚构铁律）**  
：复现步骤以最简单形式呈现——**优先 curl / 浏览器直接访问**  
；curl 无法表达时（特殊头部/POST/WebSocket）贴**真实数据包**  
。**禁止任何虚构**  
：所有请求/响应必须是实操原文，所有复现方式必须亲自跑过且能复现，没跑过的步骤不写进报告（详见 SKILL.md §1.4a）。  
## 工作流  
```
准备(10min) → 测绘(recon) → 批量验证(check) → 查重+抽验(agent) → 报告 → 下一条线
```  
  
<table><thead><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">阶段</span></section></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">工具</span></section></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">产出</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">0 测绘</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;tab-size: 4;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf="">cnvd_recon.py</span></code></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">目标存量 CSV（FOFA 多通道拉取）</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">1 验证</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;tab-size: 4;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf="">cnvd_check.py</span></code></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;tab-size: 4;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf="">findings.csv</span></code><section style="text-align: left;"><span leaf="">（三重防误报）</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">1.5 查重+抽验</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;tab-size: 4;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf="">verify_*.py</span></code><section style="text-align: left;"><span leaf=""> + 派发模板</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><code style="box-sizing: border-box;font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;tab-size: 4;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;margin: 0px;padding: 0.2em 0.4em;"><span leaf="">verify_report.json</span></code></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">2 报告</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">SKILL.md §1.4 流程</span></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">提交版 + 复现版报告 + 全量案例 IP</span></section></td></tr></tbody></table>  
## 特性  
- **三类协议只读探针**  
：HTTP（GET/POST 双差分）、RTSP（严格 SDP 判据）、FTP（匿名 230 探测）  
  
- **双差分防误报**  
：HTTP 命中后自动二次无 cookie 请求，结果不可复现即丢弃  
  
- **蜜罐排除**  
：内置 ip-api 归属查询，云厂商段自动标 cloud_suspect  
 供人工复核  
  
- **严格 RTSP 判据**  
：必须 DESCRIBE 200  
 且 SDP 含 v=0  
 + m=video/audio  
；OPTIONS-only 一律丢弃（实战返工教训固化）  
  
- **FOFA 多通道**  
：支持任意数量通道（官方格式 / 中转格式均可），失败自动切下一个  
  
- **断点续跑**  
：--resume  
 跳过已完成目标，中断不丢进度  
  
- **合规内置**  
：单 IP 严格串行 + 间隔 ≥1s（代码级固化），全链路只读  
  
- **零依赖**  
：只需 Python 3.8+，不装任何第三方包  
  
## 目录结构  
```
cnvd-skill/
├── README.md                    ← 本文件（仓库首页）
├── README-移植说明.md            ← 便携包移植说明（换机部署看这篇）
├── README_EN.md                 ← English overview
├── CHANGELOG.md                 ← 版本记录
├── LICENSE                      ← MIT
├── 校验.bat                     ← 双击运行：环境自检 + 冒烟测试
├── api_keys.json                ← FOFA 通道 key 模板（占位符，用前必须填自己的 key）
├── skills/
│   └── cnvd/SKILL.md            ← Skill 本体（流程 / 原则 / 判据 / 合规红线）
├── tools/
│   ├── scripts/                 ← 9 个 Python 脚本（纯标准库，零 pip 依赖）
│   │   ├── cnvd_recon.py        ← 阶段0 测绘：FOFA 多通道拉存量
│   │   ├── cnvd_check.py        ← 阶段1 验证：HTTP/RTSP/FTP 三类探针 + 双差分判据
│   │   ├── verify_newfe_A.py    ← 新前端线A（密码找回链）抽验
│   │   ├── verify_newfe_B.py    ← 新前端线B（泄露面）抽验
│   │   ├── probe_newfe_A_bodies.py ← 结构探查：重取原始 body 头部
│   │   ├── verify_rtsp2.py      ← RTSP 批量补验（OPTIONS+DESCRIBE）
│   │   ├── rtsp_probe.py        ← RTSP 单点探测（命令行直用）
│   │   ├── rtsp_live_verify.py  ← RTSP 真拉流四步确认（DESCRIBE→SETUP→PLAY→RTP）
│   │   └── bing_search.py       ← Bing 检索（厂商查重辅助）
│   └── rules/                   ← 端点字典库（8 文件 28 条规则，核心资产）
│       ├── rules_boa.json       ← NVR（Boa 固件线）
│       ├── rules_newfe.json     ← 新前端 NVR/IPC
│       ├── rules_access_menjin.json ← 门禁（门禁专家）
│       ├── rules_access_drid.json   ← 门禁（Dr.ID）
│       ├── rules_access_secom.json  ← 门禁（SECOM）
│       ├── rules_ews_printer.json   ← EWS 打印机
│       ├── rules_superpw_only.json  ← 超级密码单点（拆出复跑用）
│       └── rules_generic.json   ← 通用起步模板（新线用）
├── docs/
│   ├── 使用教程.md               ← 从零跑通一条产品线
│   ├── 规则编写指南.md            ← 自定义端点字典的方法
│   └── FAQ.md                   ← 常见问题
└── 示例产出/                    ← 格式样例（已脱敏，RFC 5737 保留段）
    ├── progress.md              ← 状态机写法样例
    ├── intel.md                 ← 情报台账写法样例
    ├── verify_prompt_template.md ← 抽验 agent 派发模板
    ├── targets_样例-脱敏.csv     ← check 的输入格式（ip,port 两列）
    ├── findings_样例-脱敏.csv    ← check 的输出格式
    └── 全部案例IP-格式说明.txt    ← 报告附件格式说明
```  
## 快速开始  
### 环境要求  
- **Python 3.8+（推荐 3.12）——唯一硬性依赖，脚本零第三方包**  
- Windows / Linux / macOS 均可（校验.bat  
 仅 Windows）  
  
### 三步跑起来  
  
**第 1 步：装 Python**  
（已有可跳过）  
  
到 https://python.org 下载安装，装时勾选 "Add python.exe to PATH"。验证：命令行敲 python --version  
 有输出即成功。  
  
**第 2 步：填自己的 FOFA key（必做）**  
  
包内 api_keys.json  
 是**占位模板**  
（所有 key 都是 REPLACE_ME  
）。编辑 channels  
 数组配置你的查询通道：  
```
{
  "channels": [
    {
      "name": "primary",
      "enabled": true,
      "base_url": "https://你的中转站地址",
      "search_path": "/api/v1/search/all",
      "key_param": "key",
      "query_param": "qbase64",
      "key": "你的key"
    }
  ]
}
```  
- query_param  
：官方格式用qbase64  
（查询语句 base64 编码），中转明文格式用 query  
  
- key_param  
：一般是 key  
，个别站是 api_key  
  
- 不同中转站格式不一样，对照你的站文档配置；配错会 404 或返回空  
  
- 支持任意数量通道：自动按顺序尝试，失败切下一个。加备用站复制一项改 name  
 / base_url  
 即可  
  
**第 3 步：自检**  
  
双击 校验.bat  
。全 ✅ 即可开工。会检查：① Python 可用 ② 各脚本能跑 ③ key 读得到 ④ FOFA 通道实测发一条小查询。  
### 最短命令  
```
cd tools\scripts

:: 1) 测绘：拉某类产品线存量（preset 可选 nvr/printer/industrial/network/access/nas/conference）
python cnvd_recon.py --preset nvr --out stage0_nvr.csv

:: 2) 整理 targets.csv（表头 ip,port 两列）后批量验证
python cnvd_check.py --targets targets.csv --rules ..\rules\rules_boa.json --geo --out findings.csv

:: 3) 命中 ≥10 → 按 SKILL.md §1.3 派 agent 查重 + 抽验
```  
⚠️ 输入格式别搞反：cnvd_check.py --targets 要的是 targets 格式（只有 ip,port 两列）；findings_样例-脱敏.csv 是 check 的输出格式，四列版（ip,port,rule_id,url）才是给抽验脚本的输入。## 合规红线（务必遵守）  
```
只做：只读 GET/OPTIONS/DESCRIBE/LIST ｜ 单 IP 间隔 ≥1s ｜ 完整操作日志 ｜ 仅用于 CNVD 提交
禁止：改配置/删数据 ｜ 下载敏感文件 ｜ 暴力破解/横向 ｜ 提交前公开细节 ｜ 无授权扫政府/军事/关基
```  
- 本工具**只做未授权访问验证**  
（证明"能读到什么"），不提供任何写入 / 利用 / 持久化能力  
  
- 使用前确认你对目标资产**有测试授权**  
，或目标为公开致谢范围  
  
- 漏洞细节在平台披露前不得公开  
  
## 免责声明  
  
本工具仅供**授权范围内的安全测试**  
与 **CNVD 等平台的正规漏洞提交**  
使用。使用者应确保其行为符合《网络安全法》等相关法律法规，并自行承担因不当使用产生的一切责任。作者不对任何滥用行为负责。  
  
项目地址  
  
https://github.com/sywinksvg/cnvd-skill  
  
文章来源：乌雲安全  
  
