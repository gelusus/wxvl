#  腾讯重磅开源：业界首个项目级漏洞猎杀基准平台 VulnGym 震撼发布  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-05 02:38  
  
免责声明：本文仅做技术分享，使用本项目进行安全研究时请严格遵守适用法律。

重点导读简介
VulnGym 是腾讯悟空安全团队联合多所顶尖高校推出的项目级白盒漏洞发现基准平台，专注于评估 AI Agent 在真实工程场景下的漏洞检测能力。该基准平台覆盖 38 个开源项目、184 条安全公告、408 个可达入口点，为漏洞检测工具提供可验证、可量化的评估标准。
VulnGym Logo

重点导读核心设计
PART 01真实项目粒度
传统基准测试多以函数或代码片段为评估单元，无法真实反映 AI Agent 在完整多文件、多模块工程项目中定位漏洞的能力。VulnGym 每个样本均绑定特定仓库的漏洞 commit，模拟真实代码审计场景。
PART 02全面漏洞覆盖
平台漏洞类型涵盖两大类别：

业务逻辑漏洞（占比 71.2%）：授权绕过、认证失效、权限提升、AI Agent 能力边界突破等，需要跨模块代码语义推理
传统安全漏洞（占比 28.8%）：代码注入、路径遍历、命令注入、XSS、SSRF、代码执行等

PART 03可验证路径
每个样本均提供人工审核的可达入口点（entry_point）、关键操作点（critical_operation）以及跨模块推理链（trace），确保评估结果可复现、可解释。

重点导读数据规模



指标
数值




安全公告
184 条


可达入口点
408 个


覆盖项目
38 个


覆盖仓库
23 个


人工审核条目
393 / 408（96.3%）




重点导读漏洞分类体系
PART 04业务逻辑漏洞（131 / 184）



子类别
公告数
占比




BL-AUTHZ-BROKEN 授权失效
31
23.7%


BL-AUTHZ-MISSING 缺失授权
23
17.6%


BL-AGENT-CAPABILITY Agent 能力边界突破
20
15.3%


BL-PRIV-ESC 权限提升
13
9.9%


BL-AUTH-BYPASS 认证绕过
11
8.4%



PART 05传统漏洞（53 / 184）



类别
公告数
占比




代码注入
12
22.6%


路径遍历 / 文件操作
9
17.0%


命令注入
8
15.1%


XSS
5
9.4%


沙箱逃逸
5
9.4%




重点导读数据格式
平台提供两份 JSONL 数据文件：

data/reports.jsonl：安全公告级聚合记录（184 条）
data/entries.jsonl：可达入口点级标注记录（408 条）

每条记录包含仓库 URL、commit SHA、入口点路径、行号、关键操作位置、污染传播链等完整信息，支持直接 checkout 到对应漏洞版本进行复现。

重点导读快速上手
bashgit clone https://github.com/Tencent/VulnGym.git
cd VulnGym
python3 examples/load_dataset.py

Python 加载示例：
pythonimport pandas as pd
entries = pd.read_json("data/entries.jsonl", lines=True)
verified = entries[entries["verify"] == 1]
print(f"人工审核条目：{len(verified)}")


重点导读评估工具
将工具发现结果写入 JSONL 文件，运行评估脚本：
bashpython3 examples/evaluate.py path/to/your_findings.jsonl -v

评估指标：

公告级召回率（主要）：覆盖公告数 / 可用公告数
入口点级召回率（次要）：匹配条目数 / 可用条目数

默认匹配策略：路径精确匹配，行号容差 ±5 行。

重点导读仓库结构
VulnGym/
├── data/
│   ├── reports.jsonl    # 安全公告级数据
│   └── entries.jsonl    # 入口点级数据
├── examples/
│   ├── load_dataset.py  # 数据加载脚本
│   ├── evaluate.py      # 评估脚本
│   └── example_result.jsonl  # 结果示例
├── img/
│   └── wukong_logo.png  # 项目 Logo
├── SCHEMA.md            # 数据格式定义
├── CHANGELOG.md         # 变更日志
└── LICENSE              # CC-BY-4.0 许可证


重点导读应用场景

AI Agent 能力评估：量化评估大模型在代码安全分析任务上的表现
漏洞检测工具基准：为 SAST、IAST 等工具提供标准化测试集
安全研究：基于真实漏洞样本进行深入分析
教育培训：提供可复现的漏洞案例用于教学


重点导读项目开放性
VulnGym 采用 CC-BY-4.0 许可证，可用于商业和学术场景，需注明来源。平台支持社区贡献，包括新漏洞提交、标注纠正、评估脚本改进等。

本公众号非项目作者，仅做技术分享。
本文介绍的项目开源地址如下：
https://github.com/Tencent/VulnGym

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
