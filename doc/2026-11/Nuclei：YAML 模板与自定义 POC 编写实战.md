#  Nuclei：YAML 模板与自定义 POC 编写实战  
原创 不懂安全的运维
                        不懂安全的运维  运维安全入门   2026-11-15 11:29  
  
# Nuclei：YAML 模板与自定义 POC 编写实战  
  
在漏洞扫描这个领域，工具的分化已经非常明显。Nmap 解决的是"有什么"，ZAP/Burp 解决的是"Web 应用层有什么问题"，而 Nuclei 解决的是一个更尖锐的问题：**已知漏洞的检测逻辑，能否被快速复用和规模化执行**  
。  
  
Nuclei 的答案是把"检测逻辑"从代码里剥离出来，变成 YAML 模板。一个模板就是一个知识单元，可以被社区共享、被版本管理、被批量调度。这正是它和传统扫描器的根本差异。  
  
   
## 01模板引擎的底层模型  
  
Nuclei 的执行模型可以拆成三层。  
### 1. 协议层（Protocol）  
  
模板必须声明自己通过什么协议发包。常见的有 http  
、network  
（原始 TCP/UDP）、dns  
、websocket  
、ssl  
、headless  
。协议层决定了请求如何构造、响应如何接收。  
### 2. 请求构造（Requests）  
  
以 HTTP 为例，模板可以自定义 method、path、headers、body、payloads。payloads  
 是 Nuclei 的核心抽象，它允许把同一份请求用不同的参数值反复发送，用于参数注入类检测。  
```
```  
  
attack  
 支持 single  
（单值替换）、pitchfork  
（笛卡尔积，一次性组合）、clusterbomb  
（笛卡尔积，逐一组合）。这个差异在爆破类模板中直接影响请求量级。  
### 3. 匹配器（Matchers）  
  
这是模板表达力的核心。Matchers 决定了"什么算命中"。  
```
```  
  
关键在 condition  
：  
- and  
：所有 matcher 必须同时命中  
  
- or  
：任一命中即可  
  
- and + or  
 的组合条件还支持条件表达式  
  
实践中最容易踩的坑是 part  
 字段。Nuclei 可以在多个位置做匹配：  
<table><tbody><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#f4f6f8;font-weight:600;color:#1f2329;word-break:break-word;"><section><span leaf="">part</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#f4f6f8;font-weight:600;color:#1f2329;word-break:break-word;"><section><span leaf="">匹配位置</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><code style="font-family:&#39;SFMono-Regular&#39;, Consolas, &#39;Liberation Mono&#39;, Menlo, &#39;Courier New&#39;, monospace;font-size:13px;padding:1px 5px;background:#f0f2f5;color:#c7254e;border-radius:3px;"><span leaf="">body</span></code></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">响应体</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><code style="font-family:&#39;SFMono-Regular&#39;, Consolas, &#39;Liberation Mono&#39;, Menlo, &#39;Courier New&#39;, monospace;font-size:13px;padding:1px 5px;background:#f0f2f5;color:#c7254e;border-radius:3px;"><span leaf="">header</span></code></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">响应头</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><code style="font-family:&#39;SFMono-Regular&#39;, Consolas, &#39;Liberation Mono&#39;, Menlo, &#39;Courier New&#39;, monospace;font-size:13px;padding:1px 5px;background:#f0f2f5;color:#c7254e;border-radius:3px;"><span leaf="">all</span></code></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">响应体 + 响应头</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><code style="font-family:&#39;SFMono-Regular&#39;, Consolas, &#39;Liberation Mono&#39;, Menlo, &#39;Courier New&#39;, monospace;font-size:13px;padding:1px 5px;background:#f0f2f5;color:#c7254e;border-radius:3px;"><span leaf="">data</span></code></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">提取出的数据</span></section></td></tr></tbody></table>  
默认是 body  
。很多"检测版本号"的模板把匹配写在了 body 上，但版本信息实际在响应头里，导致漏报——这是自写模板时最常见的错误。  
  
   
## 02条件与 DSL：把逻辑写进模板  
  
高阶模板会用 dsl  
（Domain Specific Language）做多步判断。DSL 允许在模板内部定义变量并进行条件分支。  
```
```  
  
DSL 的价值在于**提取与复用**  
：先提取出版本号，再把它作为后续 matcher 的输入。这种链式能力是纯 matchers  
 做不到的。  
  
需要注意的是 DSL 的执行是有顺序的，前一个步骤的输出才能被后一个引用。顺序写错会得到静默的漏报，而不是报错——这类问题只能靠人工审模板发现。  
  
   
## 03自写模板：从 CVE 到检测规则  
  
下面完整演示如何把一个已知 CVE 写成可复用模板。以一个"某框架 debug 模式未关闭导致源码泄露"的典型场景为例。  
```
```  
  
这个模板具备几个值得注意的工程属性：  
- **幂等**  
：只发 GET，不改变任何服务端状态，可安全用于生产环境的常态化检测。  
  
- **无破坏性**  
：没有注入 payload，不存在把服务打挂的风险。  
  
- **多路径**  
：用列表一次覆盖多个常见端点，减少调度开销。  
  
与之相对的是**有侵入性的模板**  
（SQL 注入、命令注入类）。这类模板必须：  
1. 明确标注 severity: critical  
 并在 tags 中标记 dos  
 或 intrusion  
；  
  
1. 在自动化流水线中显式排除，不能与只读模板混在同一个任务里。  
  
这一条在合规上尤其重要——未经授权的"验证性"攻击动作，可能构成对生产系统的实际干扰。  
  
   
## 04批量调度的参数工程  
  
Nuclei 的参数组合很多，但真正影响生产稳定性的只有以下几个。  
### 限速与并发  
```
```  
- -c 50  
：并发请求数  
  
- -rl 150  
：每秒最多请求数  
  
- -t 20  
：模板并发数  
  
这三个值必须结合目标承受能力确定。对生产网段做批量扫描时，建议先用 -rl 30  
 试跑，观察目标侧响应时间再逐步放宽。  
### 重试与退避  
```
```  
  
网络抖动造成的失败会污染结果，误报比漏报更消耗排查成本。合理重试能显著降低噪声。  
### 标签筛选  
```
```  
  
按标签分批执行是**规模化落地的关键**  
：把只读检测和有侵入性检测彻底分开，既满足合规要求，也让告警结果更干净。  
  
   
## 05从模板到 CI：告警阈值设计  
  
把 Nuclei 接入流水线时，最大的坑不是跑不通，而是**噪声**  
。开发者会迅速对满屏红字脱敏，此时扫描器就形同虚设。  
  
一个可用的阈值设计需要分三级：  
<table><tbody><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#f4f6f8;font-weight:600;color:#1f2329;word-break:break-word;"><section><span leaf="">级别</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#f4f6f8;font-weight:600;color:#1f2329;word-break:break-word;"><section><span leaf="">处理方式</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#f4f6f8;font-weight:600;color:#1f2329;word-break:break-word;"><section><span leaf="">说明</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">critical</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">阻断合并</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">必须在流水线内修复</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">high</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">阻断合并 + 豁免流程</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">允许带风险说明的临时豁免</span></section></td></tr><tr><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">medium / low</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">仅报告</span></section></td><td style="border:1px solid #e5e8ec;padding:9px 11px;vertical-align:top;background:#ffffff;font-weight:400;color:#1f2329;word-break:break-word;"><section><span leaf="">记录到看板，不阻断开发</span></section></td></tr></tbody></table>  
配套还需要两条规则：  
1. **新增模板必须先在测试环境验证**  
，确认不产生误报再纳入生产流水线；  
  
1. **基线机制**  
：首次扫描产生基线，后续只报告新增项。否则每次都在报同一批历史问题，团队会整体忽略。  
  
   
## 06等保映射  
  
从等保 2.0 的要求看，Nuclei 的检测能力主要集中在**入侵防范**  
与**脆弱性识别**  
两类控制项上。  
  
技术要求层面：  
- **入侵防范**  
：应能检测到网络或行为异常。扫描结果作为周期性安全检测的输入，配合 NTA 实现资产暴露面的持续收敛。  
  
- **安全计算环境 / 通信完整性**  
：漏洞扫描结果应能映射到具体的主机与接口，形成整改闭环。  
  
管理要求层面：  
- **变更控制**  
：有侵入性的检测动作必须纳入变更审批，避开生产高峰。  
  
- **测试管理**  
：模板库本身属于安全资产，应做版本管理、评审与定期更新。  
  
需要注意的边界：扫描器给出的结论是"疑似存在"，不能直接等同于"确证存在"。等保测评结论需要人工复核与佐证材料，工具输出只能作为线索。  
  
   
## 07总结  
  
Nuclei 代表了漏洞扫描从"重量级工具"向"知识库驱动"的转变。它的价值边界很清楚：  
- **优势**  
：检测逻辑可复用、可共享、可审计，适合规模化与常态化。  
  
- **风险**  
：模板质量参差，误报率高；模板越界会造成真实攻击动作。  
  
用好它的前提不是命令熟练，而是**模板治理**  
——建立评审、禁用、基线、豁免四套机制，工具才会从"扫描器"变成"资产"。  
  
**免责声明**  
：本文所涉及工具及技术仅用于安全运维自查、合法授权的渗透测试及安全教育，请勿用于非法用途。  
  
