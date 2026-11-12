#  ARL-Next二开版本 | 自动化资产侦察与漏洞监控平台  
buddingworld
                    buddingworld  HACK之道   2026-11-12 02:29  
  
   
  
### 什么是 ARL-Next？  
  
  
   
## ARL-Next 是基于经典开源项目 ARL (资产侦察灯塔) 进行架构重构与持续演进的资产测绘平台。 针对大规模扫描场景下的任务队列阻塞、长时间运行内存增长以及历史依赖过时等工程瓶颈，系统实施了微服务拆分与异步任务解耦，全面升级现代运行时环境与测绘工具链，并集成了 Model Context Protocol (MCP) 标准协议，支持自动化资产透视与 AI 协同分析。  
## 自动化资产侦察与漏洞监控平台 (ARL-Next)。重构自 ARL 经典版，升级为 Puppeteer + Nuclei 引擎，打通“天眼查/ICP ➔ 资产发现 ➔ 漏洞扫描➔ 威胁情报追踪”全链路。Vue3 + Flask + Celery 架构，Docker │ 极简部署，AI 二开友好。  
### 架构演进与技术方案基准对比  
  
<table><thead><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><th align="left" style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh1s7z" style="font-size: 16px;">评估维度</span></section></th><th align="left" style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh6tx" style="font-size: 16px;">ARL 原始架构与实现</span></section></th><th align="left" style="box-sizing: border-box;padding: 6px 13px;font-weight: 600;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh12q2" style="font-size: 16px;">ARL-Next 重构方案</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1t23"><span leaf="">并发与调度架构</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nhrwl" style="font-size: 16px;">任务执行共用单 Celery 队列，复杂任务高并发时易发生队列阻塞与假死</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1qks"><span leaf="">拆分 OSINT 与截图为独立微服务</span></strong><section><span leaf="" mpa-font-style="mwbql1nhel5" style="font-size: 16px;">，引入轻重分离队列与批量落库，显著收敛队列阻塞与并发瓶颈</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nhhae"><span leaf="">自动化协议集成</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh414" style="font-size: 16px;">无标准化外部调度接口，仅支持 Web 控制台人工交互</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nhr03"><span leaf="">支持原生 MCP Server</span></strong><section><span leaf="" mpa-font-style="mwbql1nh1n9g" style="font-size: 16px;">，提供通过标准协议进行多维资产画像检索的能力</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1yrt"><span leaf="">企业资产拓扑</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh1178" style="font-size: 16px;">无关联企业与全类型数字资产下钻链路</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh24de"><span leaf="">资产组深度融合</span></strong><section><span leaf="" mpa-font-style="mwbql1nh1im9" style="font-size: 16px;">：支持企业股权关联与多类型 ICP 检索（网站 / APP / 小程序 / 快应用），构建多级集团拓扑与暴露面画像</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nhh6u"><span leaf="">容器构建与部署</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh15sx" style="font-size: 16px;">依赖陈旧且缺乏镜像加速，构建拉取易受外部网络波动影响</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1so9"><span leaf="">多阶段容器构建与阿里云国内镜像源预编译</span></strong><section><span leaf="" mpa-font-style="mwbql1nh1nyt" style="font-size: 16px;">，提升构建速度与部署稳定性</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1ws1"><span leaf="">底层技术栈基线</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh1bey" style="font-size: 16px;">Python 3.6 / Vue 2 / MongoDB 3.x / Nmap 7.70</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh187m"><span leaf="">Python 3.13 / Vue 3.5 / MongoDB 7.0 / Nmap 7.95 稳定版</span></strong><section><span leaf="" mpa-font-style="mwbql1nh1czf" style="font-size: 16px;"> 现代运行环境基线基准对齐</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1m3p"><span leaf="">敏感信息提取</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh1a80" style="font-size: 16px;">基于外部 Go 二进制工具，跨平台存在依赖与编译维护成本</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh3rh"><span leaf="">纯 Python 原生提取引擎</span></strong><section><span leaf="" mpa-font-style="mwbql1nh1d9n" style="font-size: 16px;">，敏感信息分析模块无需外部二进制依赖，支持流式提取</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nhlhw"><span leaf="">Web 交互体验</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nhbsr" style="font-size: 16px;">大数据长列表无固定表头，分页状态未持久化，字典与策略配置分散于多级子菜单</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nha5h"><span leaf="">表头与操作栏 Sticky 悬浮吸附</span></strong><section><span leaf="" mpa-font-style="mwbql1nhaj3" style="font-size: 16px;">、分页状态本地持久化、三栏式字典管理器</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 0.666667px solid rgba(209, 217, 224, 0.7);"><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1hnu"><span leaf="">威胁情报协同</span></strong></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><section><span leaf="" mpa-font-style="mwbql1nh14m7" style="font-size: 16px;">缺乏持续情报同步机制，时间戳处理不统一易产生冗余告警</span></section></td><td align="left" style="box-sizing: border-box;padding: 6px 13px;border: 0.666667px solid rgb(209, 217, 224);"><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;font-size: 16px;" mpa-font-style="mwbql1nh1h5g"><span leaf="">CVE 与 GitHub 代码泄露监听</span></strong><section><span leaf="" mpa-font-style="mwbql1nh15im" style="font-size: 16px;">、基于数据库原子更新锁去重、严格时区归一化</span></section></td></tr></tbody></table>  
### 核心特性矩阵  
- MCP 标准协议集成  
：提供基于 Python 标准库的原生 MCP Server，仅注册单一工具 get_asset_profile  
，支持外部 Agent 通过标准协议检索单标的多维资产画像（任务下发与策略调度能力持续接入中）。（👉 MCP 配置指南）  
  
- **异步解耦并发调度**  
：微服务拆分与独立任务队列分流，结合批量数据落库优化，保障高吞吐扫描平稳运行。  
  
- **多级资产组与深度融合**  
：企业资产查询深度融入资产组管理架构，支持集团多级架构管理、多类型 ICP（网站 / APP / 小程序 / 快应用）与生态资产自动化关联及网络暴露面统一测绘。  
  
- **现代资产测绘基准**  
：基于 Nmap 7.95 稳定版；内置原生 Python 敏感凭据与信息流式提取引擎。  
  
- **漏洞与代码泄露监控**  
：持续同步权威 CVE 漏洞情报，监听 GitHub 关联代码与凭据泄露，支持基于原子更新的去重机制。  
  
- **高可用运维与容错机制**  
：集成容器健康探活、就绪轮询与异常自动重启机制，支持管理后台在线平滑升级。  
  
- **预构建镜像加速**  
：提供阿里云国内镜像源预编译构建制品，规避跨国网络波动导致的镜像拉取与依赖安装失败。  
  
## 界面预览  
  
**全****局仪表盘 (Dashboard)**  
：实时呈现资产暴露面态势、今日侦察动向、漏洞风险矩阵与威胁情报；集成系统资源占用（CPU/内存/磁盘）与扫描调度队列监控、风险演进趋势分析、全景安全调度日志流、Web 组件指纹 TOP 5 分布及实时侦察流水线追踪。  
  
![ARL-Next 全局仪表盘](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic68LSzBBkSAZbh2mnmxvbPiaMG1ZVVxswk6ITu4I3lPf1tuGP0Z7zcrVYiadu9CqOlhmOZEmUAUnRD051mch6xJga4F2sQoibQsApk/640?wx_fmt=png&from=appmsg "")  
  
**多级资产组与全景资产画像**  
：原企业资产查询现已深度整合至资产组统一架构。支持集团多级组织架构管理、企业生态资产关联分析 (OSINT)、网络暴露面测绘 (ASM) 与全景资产画像下钻（画像抽屉聚合 13 个维度；「资产搜索」页则为 12 个页签维度，不含 Nuclei 结果）。  
  
![网络暴露面 (ASM) 测绘](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69q4y9H8pAUUJmZbIM5AEAC988SlvS8cT7SQAs5pDRP3rydHQm1rboZFPAsXz2PS5vXELeOYR9P4vzd5Y0YBQuXMUbNcibLayOo/640?wx_fmt=png&from=appmsg "")  
  
![13 维全景资产画像抽屉](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6ibLiaYOf4xTWe0uMdQ5QkOdajLeaib9tAqvZiaEYR8ziczNhcHEQV9icibX6owPpzzODE7L1llgbuKGqFjHMbibPMPwDmj1K84icPjlnLI/640?wx_fmt=png&from=appmsg "")  
  
![网络暴露面 (ASM) 测绘](https://mmbiz.qpic.cn/sz_mmbiz_png/HqolA1dQic6ibRzK8Xc4nY90mPPRZiafyJxm8VYI0F0sNLiaKaEAxYZFwMylaNkEytRMUm1rTiasXCl9xvlnHX4TBfFYQnsN3DHKDLujBco8rMhY/640?wx_fmt=png&from=appmsg "")  
  
![13 维全景资产画像抽屉](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6icPozibtyW8YQr83YFIWHQrz01FhngQV22OM54oRJ3tgBPHiavOp9QY2h2rEicb0GiajibERwRMFEbUdAOMib4mBgbibiboJvU3Ffib4ibC4/640?wx_fmt=png&from=appmsg "")  
  
**任务下发与策略调度**  
：深度联动 Nuclei、基于 Python 原生引擎的多类型插件池（PoC 验证 / 弱口令爆破 / 服务识别等）及 8800+ 条 Web 指纹库，支持任务下发、状态轮转与全流程追踪。  
  
![任务新建与策略选择](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic6icF8orpaR8IjT8sGKUZdkvG2WToricMsoJTN6ib4bASNWMnXib4icGCwO6bB0PSSYY8hGn79ibiakO5bJfr5aHDkEfJIkcjvQ1H0EfjQ/640?wx_fmt=png&from=appmsg "")  
  
![任务全生命周期管理](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69pjaC3px2lkmRx6njqmu7icVCAJf3A80uV7TNZVGMsibpbS23v8Ad63EyEINdk13BbQa0rzic2HoTyE5bL0w5cxlZfCKEYSYJ6TQ/640?wx_fmt=png&from=appmsg "")  
  
**威胁情报与代码泄露雷达**  
：实时追踪全球最新 CVE 漏洞，自动化监听 GitHub 敏感代码与凭证泄露。  
  
![威胁情报雷达](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic68ia5VZBLygCFe9sSyNJn1jM5dpLWbOTxHoIko9LWTmfZiadbxLZHSNwgHInP9q2oib6Kp5yf7JiaVnp0TNcfbicK0ZEXmVU16AvjZ4/640?wx_fmt=png&from=appmsg "")  
  
**系统设置与现代字典管理**  
：全新现代三栏式字典管理（支持在线预览/呼吸灯提示）、队列并发热扩缩容及告警通道。  
  
![系统设置与字典管理](https://mmbiz.qpic.cn/mmbiz_png/HqolA1dQic69q1icpyaM0EuWCXbMiahZpvneiavqTSoib2IuuXZnPNZib0M3rrkq8rDF8cxGAN9KvHhbVOYu4far9I1ficZVZFT3pFB3qnLYx5dPrk/640?wx_fmt=png&from=appmsg "")  
  
项目地址  
  
https://github.com/buddingworld/ARL-Next2  
  
  
