#  AI自动化IOT漏洞挖掘 | ARTEX-IOT：固件解包 · 二进制逆向 · PoC 生成与复现 · 组件级漏洞挖掘与覆盖度追踪  
buobo
                    buobo  李白你好   2026-11-26 23:45  
  
## ARTEX-IOT  
  
本项目基于 ARTEX-God 二开，感谢原作者的开源；本分支为 **ARTEX-IOT**  
，仓库在 buobo/ARTEX-IOT。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ft6csZH0gNWlvicr0ib3zpWJLicl2NALyd5wSlIyK3sI67nZdG7Lib1C0m5b1cr9NiawZO2patCs4rBFueOgXY2iajw9PSacu8SGzNc0k9zsfvxb4/640?wx_fmt=webp&from=appmsg "")  
  
重心放在 **IoT 固件审计**  
：移除了面向公网资产的探测与扫描渗透，把“资产”换成“组件”——围绕固件里一个个可指认的组件（httpd、busybox、upnpd、MQTT 服务……）去做解包、逆向、漏洞验证与复现。  
  
**支持：固件解包 · 二进制逆向（IDA 无头反编译，Web 端直接读函数反汇编代码）· PoC 生成与复现（可自主录屏）· 组件级漏洞挖掘与覆盖度追踪。**  
  
反编译导出的前提是 **IDA 9.0+（提供无头 idat）与 INP.py。IDA 出于版权考虑请自行安装，只要保证 9.0 及以上；INP 插件来源于开源项目 IDA-NO-MCP，但本项目的 INP.py 有改动，请使用仓库根目录的这一份**  
。  
  
把 INP.py  
 放到 IDA 安装目录的 plugins/  
 下，然后在「系统配置 → 反编译」里填写 IDA 安装目录（或设置环境变量 ARTEX_IDA_DIR  
，也可以把 idat  
 加进 PATH  
）。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNUQyOFnjahx2EmfSTiavjeIch4pGRvMQtOnaKQgicHzCWgDVm5qwr1Tpp3Z3P47BkXHWu1qf5sZ6UX0wLiapmFTDeRSeSVJmuKiapM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNW2r2jP3PJXLhaMMQzyloicK2eT7AXnbNq3WHeicjRibpz1icl65ibMJYk8MgoJom3h2au4k45oPM2sRJ1exianwaZMb0huHWSOqxuQE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ft6csZH0gNWHFbKTTR3DXp9CIicSfJJcaW5eAalDNFbxg4Iib6sMNGeocBzg4Ut348CZN3zib5ZR4HBmNwSUnanFZf0jLbdVgsudhSr7jnicHIg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNXx6IsBB3XjkGhyaluLqYAaGk9abJAuqJnOlvzlVyGwFnJianSk9mqSb3kK1yKWlvhDvAHjHCnZo2rsLnKBeYNbSFia4PGzeLua0/640?wx_fmt=png&from=appmsg "")  
  
## 固件厂商与版本  
  
固件级的「厂商 / 版本」不挂在任务上，而是按**固件内容 sha256**  
 存一份权威值（落在 task_scope  
 中 kind='firmware'  
 的那一行）：同一个固件在所有任务里只有一份值，写入时按 sha 全量同步，不会出现“这行有值、那行空”的分叉。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNUtFmfj9RXoic4KmGbqV9peCzyEtzAmXR8ybm2XW6eEkOY8Bh9LcS4kAxsAy21aE5CNjSKZOmPRJVs623eianFTEiaZqByq2YD4fM/640?wx_fmt=png&from=appmsg "")  
- **两个填写入口**  
：新建任务抽屉里的「固件厂商 / 固件版本」（此刻还没上传固件、没有 sha，先暂存在任务上，上传后自动绑定）；任务详情「测试范围 · 固件」行的「修改」。  
  
- **人填的可以覆盖**  
：改完该 sha256 的所有记录一起变，来源标记 manual  
。  
  
- **Agent 只能补空**  
：已有值不会被覆盖（来源标记 agent  
），避免不同 worker 给同一固件写出多个版本。  
  
- **厂商统一规范化**  
为首字母大写、其余小写（TENDA  
 / tenda  
 → Tenda  
）。  
  
- **与组件版本区分**  
：组件自己的 version / vendor  
 是组件属性（busybox 1.19.2  
、nginx 1.2.2  
、厂商自研 httpd …），与固件版本是两套字段，表格里分别显示为「组件版本」和「固件版本」。  
  
组件库表格：**任务 · 类型 · 组件名 · 组件版本 · 二进制 · 固件 · 固件版本 · 厂商 · 状态**  
。列宽可拖拽、超出列宽省略（悬停看全），支持按任务筛选，以及 DSL 查询（component_name=lighttpd  
、status==pending AND type==web  
、port>=8000  
）。  
## IDA 无头反编译  
  
可选能力，依赖本机的 **IDA 9.0+**（idat  
）与仓库根目录的 INP.py  
（放进 IDA 的 plugins/  
）。  
- **入口**  
：任务详情「反编译」页签（只列本任务组件）→ 勾选组件 → 建导出任务；侧边栏「反编译」整页可跨任务查看所有导出任务。  
  
- **配置**  
：系统配置 → 反编译。IDA 安装目录（留空回退 ARTEX_IDA_DIR  
，再回退 PATH 里的 idat  
）、INP 插件路径（默认 <ida>/plugins/INP.py  
）、单任务超时（默认 1800s）、单次勾选组件上限（默认 8）、任务产物配额（默认 2048 MB）。  
  
- **队列**  
：每个二进制一个 job，唯一键 (task_id, binary_sha256)  
；并发固定 1（IDA 许可限制）；失败/取消的 job 可重试；程序退出时会终止正在跑的 idat  
 进程组。  
  
- **产物**  
：data/tasks/<任务 id>/decompile/<sha 前 8 位>/  
，内含 index.json  
（函数名 / 地址 / callers / callees）与每函数一个 .c  
。  
  
- **浏览**  
：函数搜索、callers/callees 芯片、双击函数名跳转并可一键跳回；代码面板只按 index.json  
 取地址，不做路径拼接。  
  
## 开发  
### 手动漏洞复测  
  
任务详情的「复测」页签可分页选择本任务的漏洞、查看历次结论和证据，并手动发起复测。启动后保留当前页签，显示转圈图标和「复测中」；确认修复后同步更新漏洞状态。  
  
在漏洞列表每行操作区点击「复测」，或在漏洞详情的「漏洞复测」区域点击「发起复测」，填写可选的修复版本、测试条件或限制，系统会创建独立的复测 Agent 会话，启动后保留当前页面。列表的平铺、按任务分组和资产视图均支持该入口；复测运行时显示转圈图标和「复测中」，需要查看时点击进入对应会话，结束后恢复「复测」。复测无需重新启动原扫描任务，结论分为「仍可复现」「已修复」「无法确认」，每次的结论、证据和会话链接保存在漏洞详情中。  
  
新版后端首次启动会预置可编辑的「漏洞复测」（retester  
）Agent，可在 Agent 管理中配置提示词、LLM、运行预算和工具。默认使用其绑定的 LLM，未绑定则使用全局激活配置。复测会话成功完成且结论为「已修复」时，系统自动将漏洞处置状态改为「已修复」；执行中、失败、停止或其他结论保留原状态。原始证据和报告始终保留。也可在状态下拉菜单中手动选择「已修复」。同一漏洞正在复测时复用已有会话，停止、失败或服务重启后可重新发起。  
  
本版历史记录通过漏洞详情和会话查看，暂未纳入漏洞报告导出或任务归档包，也未自动关联流量包。演示模式只生成明确标注的模拟记录，不请求真实目标。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNXwGqmNy3jCxplH713ibianDicplbzzh4C6U54iamgHdXcWib1tTRxicCIkPpicrafYUgVIkYib5HcEnps5zurT4qk0P3AiajIfHv6Um3PM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ft6csZH0gNVAtNo06cb7VEYicx9iaSiaHNtpBSibeOuebUjpIm7KQH3a3DDj4pY9a9YLthhKLRvSOvjMOBrXwyaYaqfMSkTxeSicKtaSwvYduncY/640?wx_fmt=png&from=appmsg "")  
  
**只有复测过的漏洞才会产出复现视频**  
### 本地运行与测试  
```
./dev.sh    # 后端(:8787) + 流量代理(:8788) + 前端 next dev(默认 :3000) → http://localhost:3000
```  
- 后端：go run ./cmd/artex  
（不带 -tags embedui  
 则不内嵌前端）  
  
- 前端：cd web && npm run dev  
（/api  
 反代到后端，带热更新）  
  
- 测试：go test ./...  
（**务必指向独立测试库**  
：ARTEX_PG_DSN=postgres://…/artex_test  
，db  
 / server  
 的用例会真写库）  
  
- 反编译：先在「系统配置 → 反编译」填 IDA 目录并点「测试」，它会实际解析 idat  
 与 INP.py  
  
- Mock 预览（无后端）：cd web && NEXT_PUBLIC_MOCK=1 npm run dev  
  
## 工具下载  
```
https://github.com/buobo/ARTEX-IOT
```  
  
**ARTEX 仅供个人学习、代码研究与本地技术验证使用，不得用于对任何线上系统或网站发起实际测试。**  
## 网络安全情报攻防站  
  
**www.libaisec.com**  
  
**综合性的技术交流与资源共享社区**  
  
**专注于红蓝对抗、攻防渗透、威胁情报、安全资讯**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ft6csZH0gNWDuiavTZzTOQIdQJHfNKzmxr9EPcfaGMALkDOmeg94SJ0SWqrzKHXskK7DrKgoBtdM95plzkicPgGBicibXxxRvicoR1puguiad3OzQ/640?wx_fmt=webp&from=appmsg "")  
> 👇 点击阅读原文，访问网络安全情报攻防站  
  
  
