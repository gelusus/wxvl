#  [工具推荐]AI 自主渗透测试平台 · 从一句话目标到可交付漏洞报告-破军 Pojun  
chu0119
                    chu0119  陌笙不太懂安全   2026-11-17 09:39  
  
免责声明  
```
由于传播、利用本公众号所提供的信息而造成
的任何直接或者间接的后果及损失，均由使用
者本人负责，公众号陌笙不太懂安全及作者不
为此承担任何责任，一旦造成后果请自行承担！
如有侵权烦请告知，我们会立即删除并致歉，谢谢！
```  
  
工具描述  
  
Pojun 是一套 LLM 多 Agent 自主渗透测试平台，用户只需给出一个目标，它便能自动完成“目标拆解→侦察→漏洞挖掘→利用→后渗透→报告”全流程，并通过目标闸门、证据闸门、自主续跳和全程留痕等机制，确保 AI 像守纪律的渗透测试工程师一样工作——目标未达成便如实标记“未完成”，高危漏洞必须附可机械复现的证据，确认漏洞后自动推进到数据/资金影响层，所有 HTTP/S 流量逐请求留存、每一步可回放；平台还提供多 Agent 协作、目标管理、攻击链续跳、漏洞去重、写回守卫、漏洞情报运营、红队作战、三层审批拦截、证据链交付与成本模型路由等能力，单机实测在真实授权项目下实现 12 个任务、56,000+ 次工具调用、295 个确认漏洞的无人值守运行。  
  
核心能力  
  
**1. 多 Agent 自主推进**  
  
planner 唯一意图生成者，持跨唤醒共享 todolist 按依赖派发；worker 做完一条即停，写回双图自动唤醒 planner；mainagent 管对话与即时任务。人在环路可纠偏、暂停取消、审批。多层韧性：空转收场 / 超时 / 瞬时故障重跑，免费限流通道也能稳定跑完。  
  
**2. 目标管理：完成判定不可绕过**  
  
set_goals 每条目标可带可核验 criteria，禁止把「发现/验证漏洞」写成最终目标；goal_met 闸门有目标未 met 直接拒收官。诚实终态：全部达成才 done，否则一律 timeout 并如实标注 x/y 目标。  
  
**3. 执行面：原生执行器 + 会话复用 + 后台作业**  
  
http_request 自动过 MITM 留痕；session_login 五形态登录，任务级 cookie jar 与 curl 互通；session=同名 自动带登录态，双身份对照做越权零成本；poc_scaffold 生成可运行验证骨架；后台作业让长扫描/爆破跑完进程继续；任务级共享脚本一次编写全任务复用。  
  
**4. 攻击链续跳：确认漏洞之后继续走**  
  
report_finding 回执带 chain_intent_id 自动派发继任意图：SQL 注入 → 数据外带 → 登录态 → 横向；上传 → 落地验证 → webshell 边界；未授权接口 → 越权对照 → 数据面影响。纪律：确认漏洞 ≠ 完成意图；按 L1 信息 → L2 认证/越权 → L3 控制 → L4 数据/资金 推进；判「不可利用」前须换 ≥2 种手段并留失败输出。  
  
**5. 漏洞去重与合并**  
  
**一份漏洞一份真相**  
多键去重（强键 + 弱键），新上报自动合并，merge_count 计数、来源收进时间线；跨任务同资产同漏洞默认不重复打扰（可关）。  
  
**6. 写回守卫：范围越界的硬防线**  
  
record_fact / report_finding 锚定非本任务范围资产直接拒绝；planner「复核优先」：跨域名归属须双域取证复核，不过即放弃该链，防止误报跑偏整条链。  
  
**7. 漏洞情报运营（持续，不是静态知识库）**  
  
按时间流而非按组件查：最近 7 天爆了什么 → 匹配我的资产 → 排进任务。NVD + CISA KEV + GitHub Advisory + PoC 聚合；高危快通道每 30 分钟拉 CVSS≥9.0 或 KEV；资产指纹→CVE 反向匹配命中挂 hint；版本区间三态保守判定。  
  
**8. 红队作战能力**  
  
回连接收器接住反弹 shell / OOB 回连并逐会话留痕，启停走人工审批；C2 出口三形态 direct / SSH 中继托管 / manual，listener_start 返回解析后回连地址；redteam-ops 六阶段方法论 + 证据纪律；66 件后渗透工具「宿主环境」页一键装齐。  
  
**9. 审批与拦截：三层闸门**  
  
deny 明令禁止不调模型直接拒（10 条）；judge 按上下文走模型裁决（12 条）；ask 能力边界变更转人工审批（1 条）。规则级 timeout 真生效；裁决严格 JSON 解析失败走失败策略；裁判输入仅工具名+参数+工作目录；写回范围守卫数据层再兜一道。  
  
**10. 全程证据链与交付**  
  
MITM 逐请求留存全部 HTTP/S 流量；finding 绑定四类流量证据快照版本化（sha256）；截图与文档取样自动转图归档可归属漏洞；专业级 Word（封面·目录·严重度着色·复测记录·流量证据·修复路线图）+ Markdown + 批量「一漏洞一 docx」zip；对抗式自检「确认了才报」。  
  
**11. 成本与模型路由**  
  
LLM 三通道轮询：熔断 + 故障转移 + 用量归因，免费限流通道也能跑完（实测 2,725M token，缓存命中 93%）；按角色配模型：planner 强推理 / worker 中推理 / judge 便宜档；worker 态势瘦身，未测资产按簇排序、http_probe 批量探活替代逐条 curl。  
  
界面速览  
  
控制台登录：首次启动初始化管理员，防爆破锁定  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRe9BkqQjVrTjjH0xbiaPdC5e64Ugc9EAPatovfw1kjfcodZibTzicaWAEngzr2MpWjcXyleyret33eE63WUVnsysCekIooicVMlLU/640?wx_fmt=png&from=appmsg "")  
  
总览：任务 / 发现 / 资产 / Token 全局态势与趋势  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSnCZppQ49woibbag5pPLZwOrN6X6kwWXjtJiaPnf9zhs8Juib7rwVBiaohpFxpOPoic5LIiaIjXiaA7drHfMujgBngpWvSjia6ttKBNQw/640?wx_fmt=png&from=appmsg "")  
  
任务列表：目标进度与漏洞分布一屏可见  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRibC7uxzyIUhPia3RN2UoZu3CxoQgPCA61lcBfQ2kjp9PDKgEaqfajBKMIaUJFFMTu6HZUS5cV1Wwhu0bAJP6Dg4fstKAevz27E/640?wx_fmt=png&from=appmsg "")  
  
任务详情：多 Agent 会话，planner 态势研判实时可见  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTd1w3DDciaHwII3737CsNI5ksMNmFiaGSQrFa3S2xkvGgfynKzZmmFq3KJC14iceQ9HxIDZIJZAH0XiaKrllMT6XibKUmkSb4cVnfI/640?wx_fmt=png&from=appmsg "")  
  
任务总览：目标逐条带验收标准，达成状态诚实呈现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQjicvZ4lZ19MUicfAD5ZlxfO20CD30ibzsl148mbaK5MgB3NJ9wyS8GOyIsBeJw05l8chO64JB8yrlnm5Bqj60fiaVSoeNkKaOD1A/640?wx_fmt=png&from=appmsg "")  
  
目标管理与操作约束：目标逐条勾选达成，allow/deny 约束直接约束 planner 派单  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSTDYvTej75YYcjA4y90UonRPvc6NYBMbZZUCC1FHOKyfTgAWlAval9tYyATzS8h2ufiaYuVTicJIaZA21UxT19FvQnGjfOZ6qfo/640?wx_fmt=png&from=appmsg "")  
  
发现：跨任务漏洞汇总，按严重度统计  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRhcBicTWWO8GHxYjEx9zZUCWP2j3esv1To7ia2E5sVSsVrP7jz0POTHNAfiaY2crKR0RCDrLshLMq3As0l2UcLLSdOCxV421XnAg/640?wx_fmt=png&from=appmsg "")  
  
漏洞详情：摘要 + 复测命令 + 证据 PoC + 状态卡  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSAJzs8iafyNXdIDk28zVVRGjtlDSK0ZfxkQiayibynibwynLlaZcE6WUpPq71NPCGGbulJTcicyYicWtE0q781tiaOsHFLoiamSwGiaicKk/640?wx_fmt=png&from=appmsg "")  
  
资产台账：企业 → 根域名/IP/子域/服务/接口 全量登记  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSqf2C2I85IicGU0d9YRQjQGxQ8Xv44DzQia7KeXQSOlfR0m42lbXXmCVFneEmaiaRKVq5EC8OcdPECicNbmNE4oPrfDuqe8oicldAc/640?wx_fmt=png&from=appmsg "")  
  
资产覆盖图：力导向布局，已测节点高亮  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSYcoicms7yJHVwTGMh7aL8ib3xJ2XPj9XzYdfB9ic2QI6awEibzYRjypBtJf9SkFibUGx42Io8TrjWbWsX4GNPCWZiaKBMSyPib07kwY/640?wx_fmt=png&from=appmsg "")  
  
漏洞情报运营：42 万 CVE / KEV 在野利用 / PoC 仓库持续同步  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQdialStkaLMrhlEew41ABYAN7eO9IeS9Tic5Utia729DW2FdzC3pVtJ5s88JicEBVEc9UMCFWah67icDdMUVsZ2o0fOBYjEeDybGwo/640?wx_fmt=png&from=appmsg "")  
  
LLM 配置：多通道轮询 / 熔断 / 故障转移  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSbrp5l6KiavH0389aHe6qoPIbXjeOJmNGLb3WSVNiaNchmEpsFnFjOLmfIo7xYozTv74Tlm4nYCaiapXGXyjJEweY2E0NrGgPYBQ/640?wx_fmt=png&from=appmsg "")  
  
对话工作台：跨 Agent 对话与工具触发回放  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSuyS1SG8e3pcBxAwtjQf7Ty20YxAib045NrrcibkzK8JdrqX3pfO8HibwxZcSxXicxCjHarmTccpSHF8vsEEiaP9gb19KnEscKLAhw/640?wx_fmt=png&from=appmsg "")  
  
领域工具：agent 可调用的平台工具清单与开关  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRW6G61z1kLkZ6cNvMQYBcOkAT4pQB4Sa38DN5OOt85DlJCf172Rw270BEFCkAboRziacvWWcVTVmibBeGDYCCHpavFCibW4w8bJk/640?wx_fmt=png&from=appmsg "")  
  
工具使用  
  
**Linux（Debian / Ubuntu / RHEL / Kali 等，root 执行，全自动交互安装）：**  
```
curl -fsSL https://github.com/chu0119/Pojun/releases/latest/download/install.sh | sudo bash
```  
  
**macOS 13+（Apple Silicon / Intel，一条命令，不需要 sudo）：**  
```
curl -fsSL https://github.com/chu0119/Pojun/releases/latest/download/install.sh | bash
```  
  
自动完成：Homebrew 检测（没有就装）→ PostgreSQL 16 安装与服务化 → 下载并 校验 darwin 包 → 生成 config.json → 注册 launchd 用户级服务（开机自启 + 崩溃自动拉起）→ 健康检查。渗透工具与 LibreOffice 等 brew install 按需装 （缺失工具在「宿主环境」页如实显示），详见  
安装详解  
。  
  
**Windows / 已有 Docker（工具链全在镜像里，宿主机零安装）：**  
```
git clone --depth 1 https://github.com/chu0119/Pojun.git pojun && cd pojun
cp .env.example .env    # Windows 用 copy；至少改 POSTGRES_PASSWORD
# Windows（Docker Desktop，WSL2 后端）：
docker compose -f docker-compose.yml -f docker-compose.windows.yml up -d
# macOS（Docker Desktop / OrbStack）：
docker compose -f docker-compose.yml -f docker-compose.macos.yml up -d
# Linux / 其他：
docker compose up -d
# → http://localhost:8787
```  
  
**装好之后（3 分钟上手）：**  
1. 浏览器打开 http://  
<host>  
:8787 → 初始化管理员密码（≥8 位）  
  
1. 「LLM 配置」页配模型通道（≥2 个通道 + 开启轮询 = 实战推荐）；Anthropic/OpenAI/ DeepSeek 等 OpenAI 兼容端点直接填 base_url + key + model（如 DeepSeek：   
https://api.deepseek.com  
 + deepseek-chat/deepseek-reasoner）  
  
1. 「宿主环境」页一键装齐工具（Docker 镜像已内置全部 66 件，可跳过）  
  
1. 新建任务，给一句话目标  
  
工具链接  
```
https://github.com/chu0119/Pojun
```  
  
  
  
**后台回复加群加入交流群**  
  
****  
**广告：********cisp pte/pts &nisp1级2级低价报考**  
  
  
**陌笙安全纷传圈子+陌笙src挖掘知识库+陌笙安全漏洞库+陌笙安全面试题库**  
**简单介绍****（**  
**加入纷传圈子**  
**送****知识库+漏洞库+面试题库****）**  
                          
  
如果觉得合适可以加入,圈子目前价格  
39.9元，价格只会根据圈子内容和圈子人数进行上调，不会下跌。。。    
  
  
**圈子福利**  
   
  
**edu漏洞挖掘1v1指导出洞**  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKQWHxLsRrPqpqdiceX76d7yExQIyOqFmmJAfHQh7qzKvPc2V5z6iaa0RY6Ib8AsGvgS5MKkAk5aaHnJBaSnI10LDKQYMLcQMmg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR8pnPeapLBK4Jsa4ufCvFoGL66t7PKeZyA3AjNxsObjtnCibN2gzGX7NMS7Wo5sj3YYL2iboeRuQDcWqiapc8xuo5fticoBG4DsyY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRKIBNIQIVicRWJLbyGRmg92vPzc8375PJpcYVvfywzwqnaeBicZuEbfvuic9KRdjwkahSDic5VqrH2Mb4NkqtkADl5HLIh8gPex60/640?wx_fmt=png&from=appmsg "")  
  
**skill+grok辅助挖掘某企业sr**  
**c****实战效果，能出但是重复多，agent独立挖掘也可以，见仁见智，看个人习惯，好的模型是最重要的。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQwyn779TTwY7vZkePQCL8k3K8dYxNdyzfgADL4dJcNUvpmodLeDVCZ6xDC4RJXEBmO2tcWqgUNdTVicKTW0jpdsg7X6xwP5gIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQnIqagDL2A4BUIXrib9YVmATWuaIDqETqYd9ToHib52mDyoMyqc6Wzh733FRnbsDsGgey7B8s8jr72UtkPY6ich58niaPJqoItKcE/640?wx_fmt=png&from=appmsg "")  
  
****  
**企业src边缘&核心资产实战效果&&有重复但是证明好模型+AI确实够用**  
  
**（图片仅供参考，我出不等于你出，见识到ai神力即可，多去用AI!!!）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboT1YNI9U6r6NMO4UMUBROoWeC4XQC4Dge94ODZ7tXY6tbxqb3IJoghve0u1SfkygE5UJ5HTdUBvLZKrn5ps13F71piax5dlHnOE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTGjdxKy4nllaLXIznTvRqicichITccuB8psYFRYakw6ViauCk6iccziahfPw4fnrqhyCp7Zkq7lRI0DOicyrZlNyicYibTVibfGFZVaG0c/640?wx_fmt=png&from=appmsg "")  
  
****  
**不是P图,单洞1.2w记录**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQQpQdR2Ttwqxibyr75Is0kBG2N2tLYQIaau7SS278oyQ4RDpNScviaMt4wtlfgDCibE05WgoMhE5kZUrP8ciaYIdnxA594wsmoAAs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSRn2EsfFkA5mG6dcn7JLQMroc2dy3EQb3ueY2Cspd0WYgicXEnSF68UD43nNd4plkxmkTpEOh2kkQMEWZZIjE0ibA8r1q4IfiaxI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSexLYepQ2HCU6jVEiaS4UhZ4TItms5nzVGeTTIAgAzXe84dMZl36LsFIib3HxZdvtkNzdCKaS0RRJbczCQTtO1U9LG9NDwLxrvQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙src挖掘知识库介绍（内容持续更新中!!!)**  
```
信息收集(主域名信息收集,子域名信息收集等&会永久提供fofa-key助力)
弱口令漏洞&未授权访问漏洞挖掘
任意文件读取&删除&下载&上传漏洞
sql注入漏洞
url重定向漏洞
csrf&ssrf漏洞挖掘
XSS&XXE漏洞挖掘等等常见漏洞
cors&目录遍历&越权漏洞挖掘
EDUSRC(证书站挖掘案例分享&edusrc挖掘技巧分享)
CNVD挖掘技巧分享&实战案例报告编写
公益漏洞挖掘（公益src挖掘漏洞分享&提供补天1权重资产）
SRC挖掘实战(针对各种常见功能总结的常见测试思路等快速提升)
经典常见Nday漏洞(常见中间件&以及各种常见框架)复现
云安全相关漏洞挖掘（云key扫盲&云存储桶&快速识别云环境&云攻防）
AI相关学习（AI基础&AI代码审计实战测试&webLLM攻击等）
APP&小程序漏洞挖掘
等各模块不在一一介绍
```  
  
信息收集  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTu9DGyTubluhYicFynwVBKa4V06sDfEVKOyk5Q4ghZzLMDAuLb1M1oR4RJumGWrADPapFjTrOjpksKQ8q0YYCnl3ZWLof8Knzg/640?wx_fmt=png&from=appmsg "")  
  
src挖掘基础  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboR45bibbJEb28a1gS5yth3r5HyOsgPiaOUHHYriahZyIyrk0LMOsHW4VoDibyBRibTNzptGiaLWX62UwykicwvbxCJPopvklqiaxML8lS8/640?wx_fmt=png&from=appmsg "")  
  
src挖掘实战  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTKWnTsN6CXf3djhXIlMKNRjVmJn3g5b23ur9E6Cx3O68f0hXVjCiaj8J4RYeTGBecqf1k99phG0ice2wtd5lKgR46OeqeLQfMpk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRbZ1HYm7R7YEiaxRVQibGWyricx9l7HpGjS4ZfWRdlft8iacwkpzYyZfmYEkWdJgYRORPkNFR6dADR5MyE524tWX6cAwN8MmrCZu0/640?wx_fmt=png&from=appmsg "")  
  
  
edusrc  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQVVlTXhibjR8UiakZBQicXRZrQ7hdoOz5G8MQrcuDBGbqJdO0kIz6R9IU4ObAeOiabT8pr6lc7jibdIkKoTjiaXNHPLAwAB3BV2UvLM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSCrvarBbzP4L9kS6P0LVH9JMdmcbFDKiaicHqMFgTxq3x4iatjDJQicmc7NPC14C9Fk3icFjrouSgNVaN8Byuf0C0Iq9O6D1XPvFvY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRALwXmgZ4mh2LW0RdicrKjBCP7P1iaF14G0Eq2v3KRnTJORpwXZlF58WEz6QicxLJpyJaA5iah5CF2rHjBz4JzOELFRaZTAKOQ2tQ/640?wx_fmt=png&from=appmsg "")  
  
经典nday复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboRu8Gf849iaCkSBxLL8IlzJTRs185QicEe9l5UGI1dEVKISt2IGGveZynXBW9tIUsxNsz4adSTib7rib50uSJdjNfTvVRFrbPJhzL4/640?wx_fmt=png&from=appmsg "")  
  
  
云安全&AI安全  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQ9qiavETNjaaX162czpNCqpw3uJqVpicbI15AXzhf5x8icmHxBdTGOgRgzNPGF3Aw2gglT4Fx09JGXYibQC6U7CQKVmoH08l3meia4/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQgcsjiaZ4S26TWowHfpBkhSeHf2pjrcDyicJuia3uqvRBauLEOicibibEMqibnBMtjopFL8No7UXNibbURvzeJ3dQHTibvGxRQGnorb4co/640?wx_fmt=png&from=appmsg "")  
  
**陌笙安全漏洞库介绍**  
```
最新漏洞查看
1day&0day分享
EDU学校相关漏洞
Web应用漏洞
CMS漏洞
OA产品漏洞
中间件漏洞
云安全漏洞
人工智能漏洞
其他漏洞
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTFBRMa8XwYxfcZMyXicx94xSKxawPcqFia2rJKOL7fSLYXiccwHc868XxNGIQ5z7ibiaI1MNAGRrK7U6wXJTsZOCAu2I5XV1boTAL4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSBzdakI9XI33ReAm2dxO8vgzw3JicQmUuWCb5ayBlKR1PoQHEHFETteBnicyupwU0mXvXibfrDoyg8nSWBGoK1p2YXY3ElhcvOQ0/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSajCclDhuRpaLic9Ld915CHU7RqSC1LCrPGfNZiavdPEVeDedDWOPBhtMLCicTp3RNd1lT0Pmfo3mx5B0hUxbQg3ic6Via90NMtZVk/640?wx_fmt=png&from=appmsg "")  
  
****  
****  
**陌笙安全面试库**  
```
渗透测试基本问题一汇总
渗透测试基本问题二汇总
渗透测试基本问题三汇总
微步护网面试题目
长亭科技面试
深信服护网面试
启明星辰渗透测试面试题目
安恒面试题目
绿盟笔试题目
360面试
奇安信护网面试
运维面试题目
运维面试题库
网安面试相关文档大全
相关面试文章推荐
等等
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSoLqEzH0a3A4LQrvTIkGx81Sh5pf6fCoEQJhYg715vrJicSkfBuCoAmV2Kp4uOMe5jcUZutPwicibFibtJ1ZmyiaAibCg0XicWnsNcicE/640?wx_fmt=png&from=appmsg "")  
  
****  
**POC库****&&更新适配afrog&&nuclei&&dddd的POC&1day/Nday等&&**  
**dddd二开****工具[助力渗透测试&&红蓝攻防]**  
  
**工具截图**  
  
****  
**实战效果**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboSKqLXNcOPE07xOwOUCjRGuFphopPumW9RaticmNCuEUXu52GtdTTfpTUicrBj80kMcZzJsnps3abyvXIvLHEIhvMoXUApOqZCe4/640?wx_fmt=png&from=appmsg "")  
  
****  
**poc库【后续持续更新】**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboTG9Lyp44aFffUOxQKtHjToGfqFWTjswYft0VtAPINtV5MqmrTTj8GWrVb6yowvHURubPgOqdribmibWEb0Fcj3YdN4iahUwItcxE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRAMJIIvexOOJa5KhrsKmlsx8bkwib9SPoK72Q0OSPWR5qx67yvl8scMQ5bg8caBXZH01kM39RDnKpnWSaTicgobRmLygERGFWls/640?wx_fmt=png&from=appmsg "")  
  
  
**AI赋能-**  
**skill辅助**  
**漏洞挖掘（免责&&慎用）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboR3Dib0RVxVhUOzS6ibC6BvkfulXQAclic0XCXMS35C4EPoqX1b2eMVj2CFiaLCelVs1szGibaHiaAq7WibRdwHUg0IwO8fjDdWxNv6eY/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSXZZVels2NibmHgyxntlCRNIkgoqMPfUPwSM9O43OqniaZLDEJic9QRkW01gNTydFkibdI6yBRkJJ1sDUmfl7iaicoibz1QLp0J2pWE4/640?wx_fmt=png&from=appmsg "")  
  
  
**圈友skill+ai辅助渗透**  
**实战效果**  
**，支持打假！**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboTeoic5S15BA8Ln4kjQfENmf6eDwrJzZntfWa8dT09ShNTQrTVuCA9KrKSluy1BWml6nxbia5tDW5fTaMsprQNDrr4MTBtCC9iaxk/640?wx_fmt=png&from=appmsg "")  
  
  
证书站  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboQibuxKAyHBZicB1t5yGVKyV82Teo8C2MbjKPytKziaXUcjPiao8ylHbD4vicAld8equC9alic3NksvWJ09wArXaPXZD10vjPtfoia4Vg/640?wx_fmt=png&from=appmsg "")  
  
普通站点  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboQ1Xt6gdLgd2d1mf8QURQX4YZjHA2uIw9GdTuxzSafBVOJzrQVmHJlqdhVWdVDj3OQsiaQhYOaoiabXc6EgajvBMvB6xBZwdJkIQ/640?wx_fmt=png&from=appmsg "")  
  
****  
**陌笙**  
**纷传****圈子介**  
**绍**  
```
1、src挖掘思维导图，信息收集思维导图，edusrc挖掘思维导图，以及后续的红队&面试思维导图&自己网安笔记等持续更新
2、2025-2026的edusrc实战报告包含证书站和非证书站以及2025之前的各种优质报思路分享
3、各种src报告思路分享（内部&外部）
4、分享各种src挖掘&edusrc挖掘培训资料&视频
5、不定期分享通杀、0day
6、有圈子群可以技术交流以及不定期抽取证书&免费rank
7.分享各种护网资料各家安全厂商讲解视频&精选实战面试题目
8、各种框架漏洞技巧分享
9、各种源码分享（泛微、正方系统、用友等）
10、漏洞挖掘工具&信息收集工具&内网渗透免杀等网安工具分享
11、各种ctf资料以及题目分享
12、cnvd挖掘技巧&CNVD资产&src资产分享&补天1权重资产分享&fofakey共用
13、免杀、逆向、红队攻内网防渗透等课程分享
14、漏洞库&字典以各种内容不在一一说明
15、cisp-pte/pts&nisp一级&nisp二级&edusrc证书内部价格
15、如果有漏洞挖掘问题或者工具资料需求可以找群主(尽量满足)
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/MSDUaqtwboSY2pbvbP3qGAlW8O43bRvAISCxZm4UDTRsaMVbJKTsjfTMTDlq6qNBcVs4tkl4UzgqGz5ag81baU1rusKE09J9T6cMVliaibibwQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/MSDUaqtwboTrLRQpTicOR7bzyNiajiapVJgyMiaYlEDBVU87YXMnanOFWsCYN3cCVGsKkibzV9dMryvbFXBb4Z3472ib27RJ1Xq1HnKJIp5u49GYQ/640?wx_fmt=jpeg&from=appmsg "")  
  
**目前800多条内容，扫描下方二维码查看详情以及加入圈子，持续更新中。。**  
  
**如果觉得合适可以加入，价格不定期会根据圈子内容和圈子人数进行上调。。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/MSDUaqtwboRngNCK3ae5W1nw179icr078zj1eBrn2aP1FVic1yJHdvbJ1sIKcZzNKlhXsbTfcUYdo0miblHoJpdcfG10cZfp6HbbHQfkfiajU40/640?wx_fmt=png&from=appmsg "")  
  
  
