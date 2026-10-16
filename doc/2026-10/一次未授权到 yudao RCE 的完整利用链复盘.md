#  一次未授权到 yudao RCE 的完整利用链复盘  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-10-16 00:30  
  
       这是一台在攻防实战中遇到的服务器：6 个 Web 应用、3 类框架堆在一起，而它们几乎共享同一个毛病——鉴权形同虚设。从整段无鉴权的接口、明文回显的平台私钥，到跨实例复用的共享口令，最后一条“任意文件写 → cron 提权”的链子把三套实例全部打到 root——没有零日，全是“本该拦住却没拦住”的老问题。  
  
       本次目标是某市Axx的“某平台”（xx.xx.xx.xx:20110）及其同主机应用（22119 某接口服务中间件、28090 某商户支付渠道管理后台、xx.xx.xx.xx:28007/28009/28019 某授权平台），共发现未授权访问、任意文件读写、默认与共享口令、RCE（root）等 11 项高危漏洞，服务器被完全控制；平台数据与个人信息可被读取，支付通道配置可被篡改。  
  
**命名缩写：A=28007 实例、B=28009 实例、C=28019 实例**  
  
https://xx.xx.xx.xx:20110/#/login  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqa1uUZr1OticpicbA8pvnIKOd7cGMic4txKwrfwvoamxuWFAGq8zm6HWdTNkdyYEG0gBW2CBn178dm5pjuRWjdW0qoCDSBUEyUfU/640?wx_fmt=png&from=appmsg "")  
### 1.1 系统架构  
  
       主目标为某市Axx「某平台」，前端为 BladeX/Saber 单页应用（:20110）。经端口探测与响应特征分析，同一主机 xx.xx.xx.xx 上并存 6 个独立 Web 应用（分属 3 类框架），拓扑如下（302 = 需登录、500 = 被网关拦截）：  
  
yudao--主项目 ruoyi-vue-pro（从若依 RuoYi 深度重构而来，现已改名 yudao系）。  
```
```  
### 1.2 漏洞清单  
  
本次实战共发现漏洞 11 个，其中高危漏洞 11 个，漏洞情况如下表：  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n14" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">序号</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n15" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统名称</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n16" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞名称</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n17" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞数量</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n18" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞影响</span></span></span></th><th style="box-sizing: border-box;padding: 6px 13px;font-weight: bold;border-width: 1px 1px 0px;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: rgb(223, 226, 229);border-right-color: rgb(223, 226, 229);border-left-color: rgb(223, 226, 229);border-image: initial;border-bottom-style: initial;border-bottom-color: initial;margin: 0px;"><span cid="n19" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">漏洞级别</span></span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n21" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n22" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/v1/api/expose/payment/paymentPage</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某平台——某接口服务中间件</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n23" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权访问</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n24" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n25" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取 xx 条支付记录（xx账号 / 单位 / 金额 / 支付链接）。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n26" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n28" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">2</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n29" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/v1/api/bank/payment/page</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某平台——某接口服务中间件</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n30" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权访问</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n31" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n32" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取 xx 条支付记录，businessParam 明文含个人姓名与文书字号。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n33" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n35" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">3</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n36" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/v1/api/expose/payment/requestLogPage</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某平台——某接口服务中间件</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n37" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权访问</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n38" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n39" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取xx交互日志，明文含一笔退款交易的完整调用参数（含 BANK_API_KEY）与 XML 报文。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n40" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n42" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">4</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n43" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/v1/api/expose/payment/fileDownloadStepPage</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某平台——某接口服务中间件</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n44" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取和下载</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n45" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n46" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取 xx 条对账文件记录，对应 PDF 可未携带任何凭证直接下载。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n47" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n49" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">5</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n50" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/swagger-resources</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> → </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:22119/v2/api-docs?group=xx</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">接口服务模块</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某平台——某接口服务中间件</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n51" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">Swagger接口未授权</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n52" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n53" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权获取某接口中间件 435 个接口定义（含转账 / 支付等xx接口完整入参），为xx接口滥用提供直接入口。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n54" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n56" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">6</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n57" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:28090/merchantUser/getList</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某商户支付渠道管理后台</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n58" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权访问</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n59" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n60" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取 2 个商户的 RSA 公钥与 md5 校验密钥，破坏支付通道身份可信性。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n61" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n63" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">7</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n64" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:28090/merchant/merchantPayChannel/getListPage</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某商户支付渠道管理后台</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n65" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权访问</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n66" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n67" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权读取平台 RSA 私钥明文、通道令牌、xx账号与xx内网地址（xx.xx.xx.xx:8181、xx.xx.xx.xx）。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n68" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n70" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">8</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n71" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">① </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:28090/merchantUser/saveData</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">② </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:28090/merchant/merchantPayChannel/saveData</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某商户支付渠道管理后台</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n72" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">未授权写入</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n73" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n74" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">可持久化篡改支付网关配置（xx IP/FTP 等），重定向支付流量、窃取通道凭据或中断支付业务。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n75" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n77" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">9</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n78" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">http://xx.xx.xx.xx:28090/report/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某商户支付渠道管理后台</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n79" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">默认弱口令 admin/admin</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n80" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n81" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">默认口令未改导致报表管理面暴露，与未授权写接口组合构成支付渠道配置读写通道。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n82" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;background-color: rgb(248, 248, 248);"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n84" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">10</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n85" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">① </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28007/api/admin-api/infra/file-config/create</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> → file/upload</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">② </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28009/api/admin-api/system/auth/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> → file-config/create → file/upload</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">③ </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28019/api/admin-api/system/auth/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf=""> → file-config/create → file/upload</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某授权平台（yudao，A/B/C 三实例）</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n86" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">同链 root RCE（三实例）</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n87" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n88" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">同主机 A/B/C 三实例服务器均被 root 完全接管，平台数据可被读取、篡改、删除，并成为直读主站业务库的枢纽。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n89" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border: 1px solid rgb(223, 226, 229);margin: 0px;padding: 0px;"><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n91" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 18.8px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">11</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n92" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 242px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">① </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28007/api/admin-api/system/auth/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">② </span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28009/api/admin-api/system/auth/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">③</span></span><span md-inline="url" spellcheck="false" style="box-sizing: border-box;word-break: break-all;"><span leaf="">https://xx.xx.xx.xx:28019/api/admin-api/system/auth/login</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">系统：某授权平台（yudao，同批三实例）</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n93" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 101.2px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">同批超管</span></span><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">共享口令 xxxxxxxx</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n94" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">1</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n95" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 186px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">三套授权平台超管权限（userId=1）被掌控，可访问用户/租户/OAuth2/数据源/文件配置等全部管理能力。</span></span></span></td><td style="box-sizing: border-box;padding: 6px 13px;border: 1px solid rgb(223, 226, 229);margin: 0px;min-width: 32px;"><span cid="n96" mdtype="table_cell" style="box-sizing: border-box;display: inline-block;min-width: 1ch;width: 16.4px;min-height: 10px;"><span md-inline="plain" style="box-sizing: border-box;"><span leaf="">高危</span></span></span></td></tr></tbody></table>## 2 漏洞详情  
### 2.1 未授权读支付记录接口  
#### 2.1.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:22119/  
（某接口服务中间件 v1.0.3.RELEASE）  
  
**系统名称：**  
某平台——某接口服务中间件  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:22119/v1/api/expose/payment/paymentPage  
（未授权；GET 返回 405）  
  
**漏洞说明：**  
22119 某接口中间件独立暴露公网且整段无鉴权（129 个读类接口 0 个鉴权拦截），本接口在不携带任何凭证的情况下直接返回支付记录全量数据（xx 条）。  
  
**危害：**  
xx 条支付记录泄露：xx账号、收款单位（涉某市Bxx等）、xx交易备注、金额、有效期与支付链接（可直接继续支付流程）。  
  
**复现过程（完整利用链）：**  
  
**完整路径：POST http://xx.xx.xx.xx:22119/v1/api/expose/payment/paymentPage（未授权；GET 返回 405）**  
```
```  
```
```  
  
**复现命令（curl）：**  
```
```  
### 2.2 未授权读支付记录接口（含个人姓名）  
#### 2.2.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:22119/  
（某接口服务中间件 v1.0.3.RELEASE）  
  
**系统名称：**  
某平台——某接口服务中间件  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:22119/v1/api/bank/payment/page  
（未授权）  
  
**漏洞说明：**  
同 22119 整段无鉴权，本接口未授权返回支付记录，且 businessParam 字段以 JSON 明文回显个人姓名与文书字号。  
  
**危害：**  
xx 条支付记录泄露，且 businessParam 明文含个人姓名（PII）与文书字号（如 xxxxxxxx立字〔2026〕xxxxxxxx号），与xx流水一一对应，属 PII + 敏感信息双重泄露。  
  
**复现过程（完整利用链）：**  
  
**完整路径：POST http://xx.xx.xx.xx:22119/v1/api/bank/payment/page（未授权）**  
```
```  
```
```  
  
**复现命令（curl）：**  
```
```  
### 2.3 未授权读xx交互日志接口  
#### 2.3.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:22119/  
（某接口服务中间件 v1.0.3.RELEASE）  
  
**系统名称：**  
某平台——某接口服务中间件  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:22119/v1/api/expose/payment/requestLogPage  
（未授权）  
  
**漏洞说明：**  
同 22119 整段无鉴权；本接口未授权返回xx交互日志，requestParam 字段明文含一笔退款交易的完整调用参数（付款/收款账号、收款人姓名、金额、xx行号、BANK_API_KEY），requestMessage 字段含与xx交互的完整 XML 报文。  
  
**危害：**  
攻击者无需枚举参数即可直接复制 requestParam 参数集调用对应xx接口发起交易，xx账户存在被直接操纵的风险。  
  
**复现过程（完整利用链）：**  
  
**完整路径：POST http://xx.xx.xx.xx:22119/v1/api/expose/payment/requestLogPage（未授权）**  
```
```  
```
```  
  
**复现命令（curl）：**  
```
```  
### 2.4 未授权读对账文件读取和下载  
#### 2.4.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:22119/  
（某接口服务中间件 v1.0.3.RELEASE）  
  
**系统名称：**  
某平台——某接口服务中间件  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:22119/v1/api/expose/payment/fileDownloadStepPage  
（未授权）  
GET   
http://xx.xx.xx.xx:22119/file/static/  
**（对账 PDF 未授权直接下载）  
  
**漏洞说明：**  
同 22119 整段无鉴权；本接口未授权返回 xx 条对账文件记录（含文件路径），且其 url 指向的 /file/static/*.pdf 路由未携带任何凭证即可直接下载对账 PDF。  
  
**危害：**  
对账文件（xx流水对账材料，含账户与交易明细）被未授权批量获取与下载，破坏xx数据保密性，并为xx核对与伪造提供素材。  
  
**复现过程（完整利用链）：**  
  
**完整路径：POST http://xx.xx.xx.xx:22119/v1/api/expose/payment/fileDownloadStepPage（未授权）GET http://xx.xx.xx.xx:22119/file/static/**  
（对账 PDF 未授权直接下载）**  
```
```  
```
```  
```
```  
```
```  
  
**复现命令（curl）：**  
```
```  
### 2.5 Swagger接口未授权  
#### 2.5.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:22119/  
（某接口服务中间件 v1.0.3.RELEASE）  
  
**系统名称：**  
某平台——某接口服务中间件  
  
**漏洞地址：**  
GET   
http://xx.xx.xx.xx:22119/swagger-resources  
（未授权）  
GET   
http://xx.xx.xx.xx:22119/v2/api-docs?group=xx  
接口服务模块（未授权）  
  
**漏洞说明：**  
22119 中间件生产环境未关闭 Swagger（另含 /doc.html UI 可未授权访问）；/swagger-resources 未授权返回全部接口分组，/v2/api-docs 未授权返回「xx接口服务模块」全部 435 个接口定义（含转账 / 支付 / 账户 / 限额 / 文件下载等xx接口的完整路径与入参结构）。  
  
**危害：**  
未授权获取某接口中间件完整接口契约（435 个接口），攻击者无需猜测即可枚举全部读类与写类xx接口，为未授权读取xx数据、xx操作接口利用提供直接入口，显著降低攻击成本。  
  
**复现过程（完整利用链）：**  
  
**完整路径：GET http://xx.xx.xx.xx:22119/swagger-resources → GET http://xx.xx.xx.xx:22119/v2/api-docs?group=xx接口服务模块（435 接口）**  
```
```  
```
```  
  
**复现命令（curl）：**  
```
```  
### 2.6 未授权读商户接口（RSA 密钥泄露）  
#### 2.6.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:28090  
  
**系统名称：**  
某商户支付渠道管理后台（xx.xx.xx.xx:28090）  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:28090/merchantUser/getList  
（未授权）  
  
**漏洞说明：**  
Spring Security 未将该管理接口纳入鉴权，任何人可直接调用；接口直接回显商户密钥字段。  
  
**危害：**  
2 个商户的 RSA 公钥与 md5 校验密钥——某市Axx（appId=2021xxxxxxxx0335）、某市Bxx（appId=2023xxxxxxxx0028）。  
  
① /merchantUser/getList —— 商户 RSA 公钥/密钥  
  
请求方法：POST（Content-Type: application/json）；接口来自后台前端路由 /merchantUserInfo。  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
### 2.7 未授权读渠道接口（RSA 私钥/xx账号/内网地址）  
#### 2.7.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:28090  
  
**系统名称：**  
某商户支付渠道管理后台（xx.xx.xx.xx:28090）  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:28090/merchant/merchantPayChannel/getListPage  
（未授权）  
  
**漏洞说明：同 2.7 节，管理接口未鉴权，且返回平台私钥明文与xx网络信息。**  
  
**危害：**  
2 条生产支付渠道配置（A、B）——平台 RSA 私钥明文（两条记录同一密钥）、通道令牌、xx账号、xx内网地址（xx.xx.xx.xx:8181、xx.xx.xx.xx）；单页码 50 条可取全。  
  
② /merchant/merchantPayChannel/getListPage —— 平台 RSA 私钥明文  
  
请求方法：POST（JSON，分页）；接口来自后台路由 /merchantPayChannel。  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
### 2.8 未授权写入 / 支付网关配置投毒  
#### 2.8.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:28090  
  
**系统名称：**  
某商户支付渠道管理后台（xx.xx.xx.xx:28090）  
  
**漏洞地址：① POST http://xx.xx.xx.xx:28090/merchantUser/saveData（未授权）**  
  
② POST   
http://xx.xx.xx.xx:28090/merchant/merchantPayChannel/saveData  
（未授权）  
  
**漏洞说明：**  
写接口未纳入鉴权，且未对网络端点字段做白名单校验，可持久化任意值。  
  
**危害：**  
支付渠道配置表（可写 bankIpAddress/txnIttIpAdr/ncIp/ftp  
/sftp  
；生产值 xx.xx.xx.xx:8181、xx.xx.xx.xx）。  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
### 2.9 报表登录默认口令 admin/admin  
#### 2.9.1 漏洞描述  
  
**资产详情：**  
http://xx.xx.xx.xx:28090  
  
**系统名称：**  
某商户支付渠道管理后台（xx.xx.xx.xx:28090）  
  
**漏洞地址：**  
POST   
http://xx.xx.xx.xx:28090/report/login  
（未授权 + 默认口令）  
  
GET   
http://xx.xx.xx.xx:28090/report/index/  
（尾斜杠绕过）  
  
**漏洞说明：**  
报表系统后台 admin 使用默认口令 admin/admin；且登录接口本身未纳入鉴权；Spring Security URL 匹配存在尾斜杠绕过。  
  
**危害：**  
报表系统管理会话（admin），可访问 /report/exportExcel、/report/printReport 及商户支付渠道后台前端。  
  
⑥ /report/login —— 默认口令签发 token  
  
请求方法：POST（JSON）。  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
```
```  
```
```  
### 2.10 三实例同链 root RCE（yudao 任意文件写 → cron 提权）  
#### 2.10.1 漏洞描述  
  
**资产详情：https://xx.xx.xx.xx:28007、https://xx.xx.xx.xx:28009、https://xx.xx.xx.xx:28019（同一主机，A/B/C 三实例）**  
  
**系统名称：**  
某授权平台（yudao）—— A、B、C 三个实例（同一主机，见 1.2 拓扑）  
  
**漏洞地址：① POST https://xx.xx.xx.xx:28007/api/admin-api/infra/file-config/create → file/upload（超管会话）**  
  
② POST   
https://xx.xx.xx.xx:28009/api/admin-api/system/auth/login  
 → file-config/create → file/upload  
  
③ POST   
https://xx.xx.xx.xx:28019/api/admin-api/system/auth/login  
 → file-config/create → file/upload  
  
**漏洞说明：yudao 框架的「文件配置」允许创建 storage=Local 且 basePath 完全可控（可设为 /）的存储；「文件上传」的 path 参数未做归一化/根目录限制，直接作为落盘绝对路径。二者组合即成为任意文件写，且应用以 root 身份运行，写入 /etc/cron.d/ 即触发 root 命令执行。三套实例同模板部署、同框架缺陷，超管会话来自跨实例复用的共享口令（见 2.11 节）——同一条链在三个实例上均成立。**  
  
**危害：同主机 A/B/C 三实例服务器均被 root 完全接管，三实例平台数据可被读取、篡改、删除；该主机同时承载主站 20110 的业务数据库。**  
  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
  
**B / C 实例同链复现（与 A 完全同源，仅如下差异）：**  
```
```  
### 2.11 同批超管共享口令 xxxxxxxx  
#### 2.11.1 漏洞描述  
  
**资产详情：**  
https://xx.xx.xx.xx:28007  
、  
https://xx.xx.xx.xx:28009  
、  
https://xx.xx.xx.xx:28019  
  
**系统名称：某授权平台（yudao）—— A、B、C 三个实例（同一主机，见 1.2 拓扑）**  
  
漏洞地址（三个实例分别登录，共享弱口令 admin/xxxxxxxx）：  
① POST   
https://xx.xx.xx.xx:28007/api/admin-api/system/auth/login  
② POST   
https://xx.xx.xx.xx:28009/api/admin-api/system/auth/login  
③ POST   
https://xx.xx.xx.xx:28019/api/admin-api/system/auth/login  
  
**漏洞说明：三套实例的超管账号 admin 使用与整个生态相同的共享口令 xxxxxxxx，跨实例复用导致一处失陷、全域失守。**  
  
**危害：**  
同批三套授权平台超管权限（userId=1），可访问用户/租户/OAuth2/数据源/文件配置/定时任务等全部管理能力。  
  
**复现过程（完整利用链）：**  
```
```  
```
```  
```
```  
```
```  
```
```  
  
       写在最后：复盘整条链，22119 端口整段无鉴权是入口，明文回显的密钥和口令让后续几乎没有成本，file-config 与 file-upload 的组合则是最后那一击。11 个高危里没有一个是需要“高技术”才能挖到的——生产环境的 Swagger 开着、私钥明文回显、应用以 root 运行、口令跨实例复用，每一条都是老生常谈。  
  
       防守视角其实就三句话：接口要鉴权（尤其是内网服务，别只靠网络隔离）、敏感字段不回显、应用降权运行。这条链子只要断掉其中任何一环，故事就讲不下去。  
  
       写这篇文章不是教怎么“打”，而是想说明一件事：一堆看似无关的小问题长在同一台机器上时，它们从来就不是“小问题”。  
  
