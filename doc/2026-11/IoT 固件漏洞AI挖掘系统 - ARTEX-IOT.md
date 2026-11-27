#  IoT 固件漏洞AI挖掘系统 - ARTEX-IOT  
GSDK
                    GSDK  GSDK安全团队   2026-11-27 10:40  
  
01  
项目地址  
  
https://github.com/buobo/ARTEX-IOT/  
  
02  
项目介绍  
  
IoT 安全研究做到固件层面，传统的自动化思路就有点水土不服了：  
  
公网资产扫描那套打法，对路由器/摄像头的固件不管用固件里几十个组件：httpd、busybox、upnpd、MQTT……从哪个下手？IDA 反编译要一个个开界面手动看，效率太低挖到漏洞想复现留证，录屏全靠手动  
  
**ARTEX-IOT**  
 是基于 ARTEX-God 二开的 **LLM 多 Agent 固件漏洞挖掘系统**  
，重心放在 **IoT 固件审计**  
：移除了面向公网资产的探测与扫描渗透，把"资产"换成"组件"——围绕固件里一个个可指认的组件（httpd、busybox、upnpd、MQTT 服务……）做**解包、逆向、漏洞验证与复现**  
。核心能力：**固件解包 · 二进制逆向（IDA 无头反编译，Web 端直接读函数反汇编）· PoC 生成与复现（可自主录屏）· 组件级漏洞挖掘与覆盖度追踪**  
。  
  
**核心理念：不扫公网、只盯固件——把挖掘进度落到每一个组件上，AI 多 Agent 自主走完解包到复现的完整链路。**  
  
![仪表盘：总览与 Token 消耗](https://mmbiz.qpic.cn/sz_mmbiz_png/Wq4VJsQicA19xfZZlx0Z4rz7wZJAcibjWpXqhfOmh3gMOMDVt0F9KFwhZmCErwgwibDGR5xpzZtx49lQLAHWDaPbiaW2Q0LkoyF3BxjckkFUcNQ/640?from=appmsg "")  
  
仪表盘：总览与 Token 消耗  
  
03  
核心亮点  
  
📦 组件库真值管理：一行 = 固件里一个组件（类型/版本/厂商/二进制/端口/   状态），按固件 sha256 全局唯一；厂商版本人工可覆盖、Agent 只补空🔬 IDA 无头反编译：IDA 9.0+ idat 队列化导出，Web 端直接浏览函数列表、   反汇编代码与 callers/callees，不用打开 IDA 界面🎯 PoC 复现与录屏：独立复测 Agent 会话，三态结论（仍可复现/已修复/无法   确认）自动回写漏洞状态——只有复测过的漏洞才产出复现视频🗺️ 双图架构：组件库（全局共享真值）+ 探索图（goal/intent/fact/finding   血缘链），支撑组件测试覆盖度与覆盖图可视化🧠 多 Agent 自主链路：planner 共享 todolist 按依赖派意图 + worker 过程级   信息交换，事件驱动闭环直到目标证明（norma SDK）🚦 工具审批门：guard/intercept 拦截审批，命令、参数、命中规则与放行理由   全程可查，全局审批记录 + 任务内拦截审批双入口  
  
![反编译：Web 端直接读函数反汇编](https://mmbiz.qpic.cn/sz_mmbiz_png/Wq4VJsQicA1icZEwZNcicwLOwqFUibj53eIpgEEAw8WnwIh33kCwS0l7NAEqmvdk0IUP3U5iafXneJ71cOCiaYsqmHeCAB6QhrvshzWiam2ibibURLNk/640?from=appmsg "")  
  
反编译：Web 端直接读函数反汇编  
  
04  
技术栈与部署  
  
Go 单体后端内嵌 Next.js 前端（单二进制）+ PostgreSQL，依赖 LLM（Anthropic 或 OpenAI 兼容接口）。部署四选一：install.sh  
 一键安装（引导填库 → 编译 → 启动）、Releases 预编译二进制（start.sh  
 守护拉起）、源码编译、跨平台 Release 构建。默认端口 8787，首次访问 /setup  
 设管理员密码。IDA 反编译为可选能力：装好 IDA 9.0+ 并把仓库根目录的 INP.py  
 放进 plugins/  
 即启用。开源协议 AGPL-3.0，支持页面一键更新（SHA256 校验 + 失败自动回滚）。  
  
**一句话概括：把 IoT 固件审计变成组件级的流水线——解包、IDA 无头逆向、AI 挖洞、PoC 录屏复现，全在 Web 端一个后台完成。**  
  
免责声明  
  
⚠️ 免责声明  
  
注：工具仅供安全研究与学习之用，若将工具做其他用途，由使用者承担全部法律及连带责任，作者及发布者不承担任何法律及连带责任。信息及工具收集于互联网，真实性及安全性自测！！  
  
