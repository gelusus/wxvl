#  每日漏洞推送 zip-rs ≤ 9.0.4 双头部脱同步（Zip Cloaking）→ 扫描器放行、解压器落盘任意文件  
原创 nullchen
                        nullchen  富贵学安全   2026-10-27 02:19  
  
# 📡 每日漏洞情报推送 | 2026-10-27  
## 🔴 漏洞一：CVE-2026-115905 — zip-rs ≤ 9.0.4 双头部脱同步（Zip Cloaking）→ 扫描器放行、解压器落盘任意文件  
### 📋 漏洞档案  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n11" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">项目</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n12" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">内容</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n14" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVE编号</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n15" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVE-2026-115905</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n17" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">影响产品</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n18" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Rust </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">zip</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> crate（zip-rs/zip2）</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">≤ 9.0.4</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（含 8.x / 9.0.x 全系）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n20" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞类型</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n21" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">解析器脱同步 / 输入校验缺失（CWE-436、CWE-20）— Zip Cloaking / Dual-Header Smuggling</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n23" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVSS</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n24" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">10.0 (CRITICAL)</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> — CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:L</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（NVD 未提供 v3.1 向量）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n26" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">攻击入口</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n27" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">任何接受 ZIP 上传并流式解压的 Rust 服务：CI 产物解包、插件/主题市场、备份恢复、SaaS 数据导入、文档转换</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n29" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">前置条件</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n30" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">目标使用 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ZipStreamReader</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（流式）落盘，或用 CDFH 做安全检查后再解压</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n32" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">公开时间</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n33" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">2026-10-26（NVD）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n35" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">修复</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n36" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">9.0.5</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> — commit </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">8068ab48</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，GHSA-m665-v7pm-7grj</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="114" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n38" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 69.1875px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">在野利用</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n39" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 839.812px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">暂无公开在野报告；但&#34;上传 ZIP → 网关查 CDH → 后端解 LFH&#34;是极常见的架构，建议按可利用处理</span></span></span></td></tr></tbody></table>### 🔥 紧急程度  
- **CVSS 4.0 = 10.0**  
，是本周唯一满分条目。攻击面不是"某个 Web 应用"，而是  
**所有用 Rust 写解压逻辑的服务**  
；  
  
- 绕过的是  
**安全产品本身**  
：杀毒/沙箱/内容审核基本都读 Central Directory，而   
ZipStreamReader  
 读 Local File Header，两边看到的文件名、大小、CRC 可以完全不同；  
  
- 三种可直接落地的打法：  
  
- **扩展名白名单绕过**  
：CDH 写   
report.pdf  
，LFH 写   
shell.php  
 /   
cron.sh  
 /   
.env  
；  
  
- **覆盖已有文件**  
：解压目录里覆盖   
templates/*.html  
、  
config/*.yml  
、  
.env  
、前端 bundle → 下游服务重启/热加载即执行；  
  
- **Zombie 条目**  
：条目只存在于 LFH、CDH 里根本没有 → 扫描器  
**完全看不到**  
，解压器照写不误；  
  
- 修复版本  
**默认没有开启校验**  
（见下），"升了 9.0.5 就安全"是错觉。  
  
### 📝 漏洞描述  
  
ZIP 格式天生有  
**两份元数据**  
：  
```
```  
  
规范要求两者一致，但  
**格式本身不强制**  
。9.0.4 及之前的   
src/read/stream.rs  
 里，  
ZipStreamReader  
 读到 LFH 就直接把   
file_name  
 交给 visitor 落盘；而   
ZipArchive  
 是拿 CDFH 建索引的。两条代码路径  
**从不对账**  
：  
```
```  
  
于是攻击者只要让   
**CDFH 和 LFH 说不同的话**  
，就能让"检查的那个组件"和"执行的那个组件"看到两个不同的归档。  
  
9.0.5 的修复引入   
HeaderConsistencyMetadata  
，逐项比对：  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n66" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">比对项</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n67" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n69" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">name</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> / </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">name_raw</span></code></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n70" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">LFH 与 CDFH 文件名必须</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">逐字节相等</span></span></strong></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n72" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Info-ZIP Unicode Path（0x7075）</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n73" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">UPATH 内容与其 CRC32 都要匹配（防止用 extra field 二次改名）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n75" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">压缩方法 / 加密标志</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n76" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">含 AES(99) 外层方法一致性</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n78" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CRC32 / 压缩后大小 / 解压后大小</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n79" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">data descriptor 场景做&#34;待定值&#34;特判</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n81" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">时间戳、AES extra field</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n82" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">防止元数据侧信道混淆</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n84" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">物理偏移</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n85" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CDFH 的 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">header_start</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 必须等于流中真实 LFH 偏移；重名/重叠直接报错</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n87" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 292.275px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">条目数量</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n88" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 616.725px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">LFH 条目数 ≠ CDFH 条目数即中止（打 Zombie ZIP）</span></span></span></td></tr></tbody></table>  
⚠️   
**关键细节（源码确认）**  
：9.0.5 里   
ZipStreamReader::new()  
 的   
enforce_header_consistency  
 默认是   
false  
（注释写明"for backward-compatibility"）；  
ZipArchive  
 侧同样要通过   
ZipReadOptions::enforce_header_consistency(true)  
 或显式调用   
ZipArchive::validate_header_consistency()  
 才会校验。  
**升级只是给了开关，不打开等于没修。**  
### 💻 PoC/EXP  
#### 0️⃣ 影响确认  
```
```  
#### 1️⃣ 构造器：一份归档，两套文件名（实测可用）  
```
```  
  
实测输出（本机已跑通）：  
```
```  
  
同一份文件，  
python3 -m zipfile -l cloak.zip  
 /   
unzip -l  
 只列出 2 个条目（  
readme.txt  
、  
docs.pdf  
），  
unzip -o  
 也只解出这两个——  
**因为 Info-ZIP 和 Python 都信 Central Directory**  
。而 Rust 的   
ZipStreamReader  
 会依次落盘   
readme.txt  
、  
shell.php  
、  
z.php  
。  
#### 2️⃣ 受害者侧：一个"标准写法"的流式解压器  
```
```  
  
运行结果：  
```
```  
#### 3️⃣ 检测脚本：给 CI / 上传网关补一道"对账"  
```
```  
#### 4️⃣ 实战变体  
- **扩展名白名单绕过**  
：  
invoice.pdf  
 ↔   
invoice.php  
、  
logo.png  
 ↔   
run.sh  
（长度不必相等，手工构造 CDFH 时文件名长度字段独立）；  
  
- **配置覆盖**  
：把 LFH 名写成解压目录中  
**已存在**  
的   
config.yaml  
 /   
.env  
 /   
templates/index.html  
，  
File::create  
 直接截断覆盖，服务热加载即生效；  
  
- **Unicode Path 二次改名**  
：带   
0x7075  
 extra field 时，某些读取实现会用 UPATH 覆盖 ASCII 名——9.0.5 一并校验了 UPATH 的 CRC32，说明这是真实利用面；  
  
- **供应链场景**  
：CI 下载第三方 artifact 后流式解包 → 覆盖构建脚本/  
build.rs  
 → 下一次构建即执行。  
**这条比 Web 上传更值钱**  
。  
  
### 🛡️ 检测与防御  
#### 归档层面 IOC  
```
```  
#### 修复建议  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n120" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">方式</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n121" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n123" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">首选</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n124" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">升级到 </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">zip 9.0.5+</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，并</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">显式开启校验</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">：</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ZipStreamReader::new(r).enforce_header_consistency(true)</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">；</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ZipArchive</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 用 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">by_index_with_options(.., ZipReadOptions::new().enforce_header_consistency(true))</span></code></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n126" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">兜底</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n127" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">解压前先 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ZipArchive::new()</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 全量遍历（走 CDH），再用 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">validate_header_consistency()</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">；两者名字集合必须完全一致才允许落盘</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n129" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">架构</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n130" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">解压目录与 Web 根目录、可执行路径</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">物理隔离</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">；解压目录 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">noexec</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">；解压后按内容重命名，不沿用归档文件名</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n132" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">扫描策略</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n133" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">安全检查必须发生在</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">解压后</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（对真实落盘文件做内容检测），不能只信归档元数据</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n135" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">临时缓解</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n136" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">无法升级时，禁用 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ZipStreamReader</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 流式路径，改为&#34;先落临时文件 → 用 ZipArchive 校验 → 再解压&#34;</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="81" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n138" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 32.175px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">取证</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n139" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 876.825px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">排查历史导入记录里是否有&#34;上传 PDF 却出现 .php/.sh/.env&#34;的落盘痕迹；检查解压目录内文件 mtime 与导入任务时间是否吻合</span></span></span></td></tr></tbody></table>### 🔗 参考来源  
- NVD CVE-2026-115905  
  
- GHSA-m665-v7pm-7grj  
  
- 修复 commit 8068ab48  
  
- zip 9.0.5 Release  
  
## 🔴 漏洞二：CVE-2026-112346 — OpenHands Agent SDK < 1.52.0 sandbox 端口绑 0.0.0.0 + 空密钥表 → 未授权 root RCE  
### 📋 漏洞档案  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n155" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">项目</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n156" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">内容</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n158" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVE编号</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n159" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVE-2026-112346</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n161" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">影响产品</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n162" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">OpenHands </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">software-agent-sdk</span></code></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">&lt; 1.52.0</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">openhands-agent-server</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> / </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">openhands-workspace</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 的 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">DockerWorkspace</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">、</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ApptainerWorkspace</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n164" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞类型</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n165" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">缺失认证（CWE-306）+ 不当网络暴露（CWE-1327）→ 未授权 RCE</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n167" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVSS</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n168" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">9.8 (CRITICAL)</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> — CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">CVSS 4.0：</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">9.3</span></span></strong></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n170" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">攻击入口</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n171" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">TCP </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8000</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（agent server：</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">/api/*</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> REST + WebSocket）；</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8001</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 为容器内 VSCode</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n173" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">前置条件</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n174" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">宿主机端口可从攻击者网络访问（默认 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">-p {host_port}:8000</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，未限定绑定地址）；</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">OH_SESSION_API_KEYS_*</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 未设置</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n176" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">公开时间</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n177" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">2026-10-26（NVD / VulnCheck）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n179" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">修复</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n180" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1.52.0</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> — commit </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">7b66584</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，PR <a class="wx_topic_link" topic-id="mvq1wtid-aa7mct" style="color: #576B95 !important;" data-topic="1" data-recommend="">#5209</a>（端口只发布到 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">127.0.0.1</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">，并正确下发 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">OH_SESSION_API_KEYS_0</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">）</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="92" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n182" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 55.8125px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">在野利用</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n183" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 853.188px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">暂无公开在野报告；AI 沙箱/推理服务是当前资产测绘热点，公网 8000 端口建议按&#34;已失陷&#34;排查</span></span></span></td></tr></tbody></table>### 🔥 紧急程度  
- **一个 HTTP 请求就是 root**  
：  
POST /api/bash/execute_bash_command  
 直接   
system()  
 级别执行，无需会话、无需 key；  
  
- **密钥表默认空 = 认证函数直接放行**  
：  
```
```  
  
- 沙箱里通常挂着：  
**workspace 卷**  
（读写宿主工作目录）、  
**LLM API Key**  
、  
/api/settings/secrets/{name}  
 明文返回的 secrets store、git 凭据；  
  
- 更麻烦的是  
**横向**  
：agent server 有   
file  
、  
git  
、  
skills  
、  
mcp  
、  
conversation  
 全套路由，等于给攻击者一个"以 root 操作该项目"的后门；  
  
- 默认   
deferred_init=False  
，不存在 503 dormant 门槛；  
/server_info  
 还是  
**免认证**  
的，扫起来毫无成本。  
  
### 📝 漏洞描述  
  
DockerWorkspace._start_container()  
 用 docker CLI 起 sandbox：  
```
```  
  
容器内 agent server 以   
--host 0.0.0.0  
 监听（这是容器内正常写法，问题在宿主机端口发布），而   
Config.session_api_keys  
 的默认值来自   
OH_SESSION_API_KEYS_*  
，  
**没配就是空列表**  
——  
check_session_api_key()  
 遇到空列表直接通过。  
  
三条叠加结果：  
**任何能连到 host:8000 的人，都是这个 sandbox 的 root**  
（容器以 root 运行 agent server）。  
  
修复做了两件事（commit 只看这两处就够）：  
```
```  
  
注意：  
**绑定 127.0.0.1 是主修复**  
。如果部署方自己在 compose/k8s 里写   
ports: 8000:8000  
 或   
hostNetwork: true  
，升级 SDK 也救不了——必须同时确认端口发布方式。  
### 💻 PoC/EXP  
#### 0️⃣ 指纹识别（免认证）  
```
```  
#### 1️⃣ 认证状态判定（先无损探测，别直接上 payload）  
```
```  
#### 2️⃣ 一条 curl 拿 root 命令执行  
```
```  
  
返回体是   
BashOutput  
，命令输出在   
stdout  
 字段：  
```
```  
#### 3️⃣ 密钥与工作区一把梭  
```
```  
#### 4️⃣ 反弹 shell（目标出网时）  
```
```  
> 出网被封时改用 agent server 的   
file  
 路由做"慢速外带"，或直接   
POST /api/conversations  
 让 agent 自己读文件——它在沙箱里权限更大。  
  
#### 5️⃣ 批量自查脚本（授权范围内使用，串行限速）  
```
```  
#### 6️⃣ 修复后验证  
```
```  
### 🛡️ 检测与防御  
#### 日志 / 流量 IOC  
```
```  
```
```  
#### 修复建议  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n231" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">方式</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n232" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">说明</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n234" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">首选</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n235" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">升级 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">software-agent-sdk</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> / </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">openhands-agent-server</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 到 </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">≥ 1.52.0</span></span></strong></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n237" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">必做</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n238" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">设置 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">OH_SESSION_API_KEYS_0=&lt;32字节随机&gt;</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">（修复版会下发给容器）；</span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">不要</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">只依赖绑定地址</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n240" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">网络</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n241" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8000/8001 </span></span><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">永不</span></span></strong></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">发布到公网；k8s 用 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">ClusterIP</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> + NetworkPolicy，compose 写 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">127.0.0.1:8000:8000</span></code></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n243" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">加固</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n244" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">容器非 root 运行、</span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">read_only</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 根文件系统、workspace 卷最小权限、禁 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">--privileged</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">、不挂 docker.sock</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n246" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">取证</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n247" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">端口若曾暴露：轮换 workspace 内全部凭据、LLM API Key、git token；检查 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">/workspace</span></code></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> 文件改动与容器内新增进程/定时任务</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td data-colwidth="55" style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n249" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 31.25px;min-height: 10px;"><span md-inline="strong" style="box-sizing: border-box;"><strong style="box-sizing: border-box;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">兜底</span></span></strong></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n250" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 877.75px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">前置反向代理做 mTLS/基本认证，并只允许调度器网段访问 </span></span><span md-inline="code" spellcheck="false" style="box-sizing: border-box;"><code style="box-sizing: border-box;font-family: var(--monospace);text-align: left;vertical-align: initial;border: 1px solid rgb(231, 234, 237);background-color: rgb(243, 244, 244);border-radius: 3px;padding: 0px 2px;font-size: 0.9em;"><span leaf="">/api/*</span></code></span></span></td></tr></tbody></table>### 🔗 参考来源  
- NVD CVE-2026-112346  
  
- 修复 commit 7b66584  
  
- PR #5209 — publish Docker ports on loopback  
  
- v1.52.0 Release  
  
- VulnCheck 公告  
  
## 🛡️ 优先行动建议  
```
```  
> **给做渗透的兄弟**  
：  
> zip 这条最好用的场景是"上传→扫描→解压"三段式。先确认扫描器和解压器是不是同一套库：如果目标用 Rust 服务解包、前面挂 ClamAV/自研网关，基本必中。打之前先用 unzip -l 和原始 PK\x03\x04 手工解析各看一遍，两者名字不一致就是洞。OpenHands 这条别一上来就打 /api/bash。先 GET /server_info 拿版本，再发一个 {"command":"true"} 的无害探测——401 说明配了 key，200 再上 id。很多环境是内网可达的 sandbox，价值在于读 /workspace/.env 和 secrets store，比拿个 root shell 更实在。两条都别忘了时间窗：/server_info 和 ZIP 上传日志都会留下痕迹，客户环境里先看日志再动手。  
  
  
⚠️ 警告：提供的 PoC/EXP 仅限授权安全测试使用，未经授权的利用行为可能违反《刑法》第 285 条及相关法律法规。  
  
  
[每日漏洞推送  Dokploy ≤ 0.30.10 跨租户](https://mp.weixin.qq.com/s?__biz=MzY5NjM0MDI4OQ==&mid=2247484349&idx=1&sn=48505f5164fa365b8bee076cb5260f92&scene=21#wechat_redirect)  
  
  
[每日漏洞推送  Dokploy ≤ 0.30.10 跨租户越权](https://mp.weixin.qq.com/s?__biz=MzY5NjM0MDI4OQ==&mid=2247484341&idx=1&sn=e09e898721fca43f020e16afd1a20b77&scene=21#wechat_redirect)  
  
  
[每日漏洞推送  帆软 FineReport 9.0 design_save_svg 未授权任意文件写入 → JSP WebShell RCE](https://mp.weixin.qq.com/s?__biz=MzY5NjM0MDI4OQ==&mid=2247484337&idx=1&sn=b4dfb608a0aea7a5f88ef0452f722a80&scene=21#wechat_redirect)  
  
  
