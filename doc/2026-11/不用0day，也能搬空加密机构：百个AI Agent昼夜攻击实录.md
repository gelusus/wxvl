#  不用0day，也能搬空加密机构：百个AI Agent昼夜攻击实录  
MarxICB
                    MarxICB  FOFA   2026-11-26 05:00  
  
****  
**▌概述**  
  
据公开报道，近期发生多起加密货币被盗事件。2026年10月26日，Blockdaemon（区块链基础设施服务商）运营的五个 Solana 主网验证者密钥被滥用，约13.5万美元被盗；11月2日，King（Solana 链上运行了多年的套利机器人）的热钱包被清空，损失约78.6万美元。而在公开报道之外，FOFA 旗下威胁狩猎分析引擎 Ferret 近期还捕获了多起利用 AI Agent 攻击加密行业的完整攻击记录，本文披露其中三起典型案例。  
  
三起案件单独看，手法都不算复杂。放在一起，指向的是同一个变化。  
**AI 正在放大攻击能力，把网络犯罪的成本打到地板，最先被冲击的一定是资产最值钱的加密货币行业。**  
1. **AI 放大攻击能力，网络犯罪的成本正在崩塌**  
。这批攻击没有动用一个0 day。入口要么是写进集中日志的云凭证，要么是未授权访问的运维入口，要么是模板自带的老漏洞，工具全部来自公开渠道。攻击的组织形态也变了：上百个 Agent 组成集群昼夜运转，信息收集和漏洞利用被拆成流水线分工，甚至有攻击者自建9.4GB的漏洞情报库辅助研究。四天给2,089台主机建档，59台服务器逐台拿下root，3,971把私钥登记造册。这些过去是一个团伙干数周的活，现在是一个人加几个Agent的日常产出。便宜、持续、可无限复制，网络攻击有了工业化产能。  
  
1. 加密行业，是网络犯罪中成本最小、收益最大的目标。  
收益端，资产就是一把私钥，拿到即转走，转账即时、不可撤回，赃款本身就是钱，连传统黑产最头疼的变现环节都省了。成本端，目标完全公开，谁的钱包在赚钱、赚了多少，第三方榜单免费可查，抓下来就是一份现成的攻击名单，连踩点都省了。风险端，行业团队普遍极小，一支套利团队往往就是一个人一台服务器，几乎不存在安全运维；游走灰色地带的小交易平台被盗，甚至不敢声张。AI把攻击的成本打了下来，加密行业把犯罪的收益顶了上去，这份差价，没有犯罪者忍得住。  
  
1. 这不是个案，是正在成型的趋势。  
我们发现的受害者，一台存有60.7亿条交易记录的ClickHouse数据库里，留有多伙攻击者进出的痕迹。同一批目标，正在被一波又一波AI攻击者反复过筛。这条低成本、高收益的路一旦被走通，进场的就不会是一个攻击者，而是一个生态。  
  
还有一个对比更让人担心。攻击者用AI把目标资产摸得比谁都细，按每笔交易重建一名运营者的盈亏，把59台服务器上的每一把密钥登记造册；大多数防守方对自己资产的掌握，还停留在一份人工维护的台账上。  
**攻防的天平正在被AI单方面压向进攻的一侧。**  
  
三起事件的时间线（UTC）如下：  
<table><tbody><tr><td data-colwidth="124" style="background-color:#ffffff;"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;color: rgb(0, 0, 0);">时间</span></span></span></b></section></td><td data-colwidth="142" style="background-color:#ffffff;"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;color: rgb(0, 0, 0);">主体</span></span></span></b></section></td><td data-colwidth="304" style="background-color:#ffffff;"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;color: rgb(0, 0, 0);">事件</span></span></span></b></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">6–9 月</span></span></span></section></td><td data-colwidth="142"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对灰产交易平台的攻击</span></span></span></section></td><td data-colwidth="304"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">攻击者批量入侵同模板的小型交易平台，篡改提现与充值地址劫持资金</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;background-color: rgb(255, 255, 255);color: rgb(0, 0, 0);">10 月 26 日</span></span></span></section></td><td data-colwidth="142"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;background-color: rgb(255, 255, 255);color: rgb(0, 0, 0);">公开披露</span></span></span></section></td><td data-colwidth="304"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;background-color: rgb(255, 255, 255);color: rgb(0, 0, 0);">Blockdaemon 五个 Solana 验证者密钥被滥用，约 13.5 万美元被盗（https://www.blockdaemon.com/incident-reports/operational-impact-on-solana-validators）</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 2 日</span></span></span></section></td><td data-colwidth="142"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">公开披露</span></span></span></section></td><td data-colwidth="304"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">套利机器人</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> King </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">热钱包被清空，损失约</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> 78.6 </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">万美元（</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">https://www.binance.com/en/square/post/384008770763529</span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">）</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 3 日</span></span></span></section></td><td data-colwidth="142"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对个人运营者的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">首笔战果：转走一个验证者节点的</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> 118.128 SOL</span></span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 5 日</span></span></span></section></td><td data-colwidth="142"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对个人运营者的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">彻底入侵一家跟单交易运营方，转走</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> 94.257 SOL</span></span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 7–10 日</span></span></span></section></td><td data-colwidth="142"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对个人运营者的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">目标台账累积至</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> 2,089 </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">台主机，其中</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> 122 </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">台被攻陷</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 9–12 日</span></span></span></section></td><td data-colwidth="142"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对加密机构的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">同一集群还入侵了 Jito、Doctorisk、Flashblock、Endorphine Stake</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 9 日 23:20</span></span></span></section></td><td data-colwidth="142"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对加密机构的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">管理员的一条</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> sudo </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">命令把云凭证写进集中日志</span></span></span></section></td></tr><tr><td data-colwidth="124"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">11 月 10 日 03:06</span></span></span></section></td><td data-colwidth="142"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">对加密机构的攻击</span></span></span></section></td><td data-colwidth="304"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">59 </span></span></span><span style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">台生产服务器全部被拿到</span></span><span lang="EN-US"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;"> root </span></span></span><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">权限</span></span></span></section></td></tr></tbody></table>  
**▌被盯上**  
的加  
密行业  
  
加密行业的资产就是一把私钥，  
**拿到私钥就等于拿到资产，转账即时、不可撤回。而验证者需要持续签名，做市和套利则以毫秒计，这类私钥放不进冷钱包，只能留在联网的机器上。**  
  
行业团队普遍很小，一支套利团队常常就是一个人一台服务器，不具备长期运维能力。更特殊的是，攻击目标是公开的。谁的钱包在赚钱、赚了多少，第三方榜单全部可查。攻击者把榜单抓下来，就是一份现成的攻击名单。  
  
▌加密机构攻击事件细节  
  
一个由上百个 Agent 组成的集群，先后入侵了 Blockdaemon、Jito 等五家加密机构。在 Blockdaemon，攻击者拿下 59 台生产服务器，提取 3,971 把以太坊验证者签名私钥，对应约十万枚 ETH 的质押量，折合数亿美元。  
  
这起事件的工作机是 109.94.97[.]113（德国法兰克福，Latitude.sh），机器上保留着针对五家机构的完整攻击记录：作战手册、派发清单、战报和战利品目录。攻击者用上百个 Agent 组成集群，由 Kimi Code 驱动，操作者代号 nono。集群使用的模型是 kimi-k3.1-pro 的两个“去审查”变体（abliterated 和 uncensored），通过第三方 API 运营商调用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m7oSwkxuQWfNq9dXcl05XWwgxEeuxAJtSjdNWsiaWJ9eMtJEYUiaYOMv2eaM61lGsrE6TetEdsnHBDfxwmsugAY7LNHm1ibMjtanVzVxTXtCMA/640?wx_fmt=png&from=appmsg "")  
  
图1：工作机里的模型压测脚本，两家付费 API 服务商、两个“去审查”模型变体  
  
Blockdaemon 为机构客户托管运行节点，客户中包括两家全球清算机构。10 月 26 日它刚被披露过另一起密钥事件，但是本次追踪11 月 9 日的这起攻击入口完全不同，大概率为两个团伙。  
  
整条路径的起点，是测试集群的容器网络端口（VXLAN 8472端口，容器组网的隧道协议）暴露在公网上。攻击者由此进入集群内网，经一个已公开的无认证远程执行漏洞（CVE-2025-1974）拿到服务账号令牌，拖出包含部署记录在内的整批集群 secrets。日志系统的查询凭据就写在部署记录里，读写不分，本应只负责写入的凭据在查询口上同样有效。这条路径全程不需要任何初始凭据。  
  
11 月 9 日 23:20（UTC），Blockdaemon 一名管理员在节点上用 sudo 执行补数据命令，他的 shell 里挂着活跃的 AWS SSO（单点登录）会话，sudo 把完整的环境变量写进 auth.log，AWS 临时凭证三件套随之进入集中日志，攻击者随即将这组云凭证读走。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m7oSwkxuQWfEMH5aCtbPMibOyYedr7Z6ywmpjyZcjfQlticicG5o4I8tglXo9nVicicGzQNPdlKAD6SCDqJ9UWTDMd33ws1asMOHG3BtrXL9lQbA/640?wx_fmt=png&from=appmsg "")  
  
图2：工作机复盘文档，一条 sudo 命令把 AWS 三件套写进日志  
  
拿到云凭证后，攻击者通过 SSM（AWS 的服务器批量管理服务）直接在 59 台生产服务器上执行命令，11 月 10 日 03:06（UTC），59 台服务器全部被拿到 root 权限，随即逐台植入 SSH 公钥做持久化。当天08:46（UTC）SSO 会话过期，13:25（UTC）Blockdaemon 收紧防线，把 root 移出 SSH 允许名单并启用登录失败自动封禁，21:41（UTC）攻击者留下的 root 全部失效。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m7oSwkxuQWeTazEZ3GnZ00EJUaQwzp7PaCfXp8FnibLFV4HuQjoWVS3ubHhBsMrTMdibSE8VLQ5XAibDr08BicE3Fpp5KAalpcQic0iaUE7W8ibib8k/640?wx_fmt=png&from=appmsg "")  
  
图3：攻击者战报，59/59 台机器全部拿到 root 权限（uid=0）  
  
攻击者记录的提取清单：3,971 把以太坊验证者签名私钥（其中 3,298 把在链上活跃，对应约十万枚 ETH 的质押量，折合数亿美元）；一条稳定币结算链全部三个验证者的共识私钥；比特币跨链桥 8-of-11 门限签名中的 1 份；两套集群的 83 个 secrets。  
**但是这里攻击者并没能把钱转走。这些验证者的提现凭据全部是 0x01 类型，提现权在质押时就交给了预设地址，签名私钥只能干活、动不了钱**  
**。**  
11月 10 日工作机自己的盘点印证了这点，对应十万枚 ETH 的签名私钥都在手里，一把也提不了款。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m7oSwkxuQWfe64BnNwKcbKgXvSkBQeOmUXbAo3S5PagLibI1BqIdBxPWVvEeDqU4TaxpNLjb3JyTFEVsX3ZDvjYtShfhQSZ3vCZQ4Cicv9Rsg/640?wx_fmt=png&from=appmsg "")  
  
图4：工作机的提现权盘点，可动用的与动不了的  
  
同一集群还入侵了 Jito（Solana 上最大的 MEV 基础设施，MEV 即链上抢跑、套利收益）。入口是一处未授权访问的容器编排接口（Kurtosis），攻击者经它摸到 docker.sock，再用特权容器把宿主机挂载进来，拿到 root。战利品包括一名 searcher（链上抢跑者）的钱包私钥、773 个以太坊验证者的 keystore（密钥文件），以及一份标注为   
"Robinhood treasury"   
的助记词，标注来自攻击者自己的战利品清单，真伪未核。另外几起入侵的对象分别是安全评估平台 Doctorisk（未授权访问的 GraphQL 接口，提取 35 组密钥）、Flashblock 的 Kibana（已公开漏洞 CVE-2024-37285，提取 67 个在用 API key）和Endorphine Stake 的 IOTA 验证者账号（拿到能控制其操作权的 keystore）。  
  
  
图5：Jito 线的战利品清单  
  
▌个人运营者攻击事件细节  
  
第二起事件针对链上交易运营者，是一条流水线式的产线。四天建起两千多台主机的台账，攻陷 122 台，战果约 212 个 SOL（4 万美元左右）。最深的一次入侵，把一名运营者的生意、数据和钱包一起搬空。  
  
第二台工作机是 91.92.40[.]140（荷兰，TechTies），由 Claude Code 驱动，两个子 Agent 分工明确：hunter 负责探测；exploiter 负责突破，行动原则只有一句提示词：  
"A root shell without keys is ACCESS, not OWNED"  
（  
拿到 root 只是访问，不算拿下  
）。另有一份写给第二个模型（GLM）的简报，把任务包装成   
"a live, authorized red-team engagement"  
（  
一场正在进行的授权红队评估  
）来绕过安全道德围栏。  
  
  
图6：hunter 与 exploiter两个角色定义文件  
  
11 月 7 日到 10 日，四天里目标资产台账累积 2,378 条记录、2,089 台主机，其中 1,542 条记录确认存在未授权访问入口（服务类型分布见图 7），122 台服务器被攻陷。目标名单先来自公开的 MEV 套利榜单，11 月 5 日又从一台未授权访问的 ClickHouse 数据库（存着 60.7 亿行逐笔交易记录）里提取出 30 天利润排名前 400 的 Bot 钱包，逐个验余额、按钱排序。  
  
  
图7：目标台账的服务类型分布  
  
对 397 台接受 root 密码登录的验证者节点，攻击者做了 310 万次字典尝试，零命中；唯一拿下的一台，口令是  
“  
**软件名+年份”式弱口令，**  
靠针对性规则而非通用字典猜中  
，行动第一天（11 月 3 日）即得手，转走118.128 枚 SOL 币。  
  
做得最深的一次，是对一家做跟单交易（自动复制交易策略）的运营方的入侵，目标来自之前那台 ClickHouse 数据库，里面存着别家 MEV 研究平台的 60.7 亿行逐笔交易和 7,700 万行钱包盈亏，攻击者按 30 天利润筛出头部钱包名单，再从钱包反查到背后的机器。  
  
进去靠的不是漏洞，  
**这名运营者把 VPN 密码、2FA 密钥和钱包口令规律留在了自己的 AI 工具的 359MB 聊天记录里**  
**，**  
攻击者读完记录便直接登录他的 Windows 电脑，打包拖走 753MB（Telegram 数据、385个浏览器密码、钱包保险库、全部 SSH 私钥），两个钱包保险库当天被破解，97 个地址同日被清空。  
  
11 月 5 日，攻击者又在他的 Bot 服务器上拿到 root，从服务器钱包转走 94.257 SOL（约 1.8 万美元），随后改写恢复环境、抹掉电脑数据，断开连接（灭迹过程只有工作机交接文档的单方记录）。  
  
那台 ClickHouse 数据库里留着多伙攻击者进出的痕迹。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m7oSwkxuQWfsriaRmU5xJCrUQspKibgfR1ZSbMtqOfNr36xibKZskEBfgqMRf7vPxicYPJBYRFHFGu8icMtgW6uZyIibvl4qhkq68gp6DP7ickB4QE/640?wx_fmt=png&from=appmsg "")  
  
图8：攻击者发现数据库存在多伙攻击者痕迹  
  
▌灰产交易平台攻击事件细节  
  
**第三起事件规模更小，手法也更简单：一名攻击者按同一套模板的指纹批量圈定 3,582 个小型交易平台，入侵后篡改提现地址劫持资金。链上可见约 2,459 USDT、3.87 ETH，以及两笔合计约 2,164 美元的 USDC 与 USDT，整体规模一万多美元。**  
  
攻击者自建了一个 9.4GB 的 AI 漏洞情报库 RAG（17,565 篇漏洞文章，带向量检索、可按语义搜索）辅助研究：  
  
攻击者的目标是一批用同一套交易模板搭建的小型加密货币交易平台，这类平台自身多游走在灰色地带，攻击它们无异于一场“黑吃黑”。攻击按流水线展开：先按模板指纹在测绘平台上批量圈定 3,582 个目标；利用 SQL 注入、任意文件读取等漏洞拿下数据库权限；随后自研的监听器每 10 秒轮询交易网站的待审核的提现订单，把受害者收款地址替换成与用户原地址首尾各四位一致的相似地址，肉眼对账很难发现，而这样的替换地址攻击者备了 17,000 个，充值通道的地址同样可以改成自己的，此后新充值的钱直接进账。服务器权限靠 webshell 和一款集成 50 多个内核提权漏洞的工具维持。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m7oSwkxuQWes2WkWHeCwJ8uRgPwBXePDvytq7wI1MXdnh3tYiactMHcJpqnZ8cjVxagPLIIaRRYhGg7G1qBbQw7ULklJuPw8s65PL6AMlBiaQ/640?wx_fmt=png&from=appmsg "")  
  
图9: wd_watch.py的替换链，上为首尾匹配的优先级矩阵，下为轮询待审核提现订单并改写收款地址  
  
链上可见其收款地址快进快出，TRX 地址两天内过账约 2,459 USDT，ETH 侧先后收到 3.87 ETH 和两笔合计约 2,164 美元的稳定币（1,000 USDC、1,163.84 USDT），进账后很快转出，整体规模在一万多美元。9 月，工作机（167.17.69[.]200，加拿大，GTHost）的开放目录暴露，这套完整武器库随其工作记录一起被捕获。  
  
▌收缩暴露面  
  
**几起事件的入口都不复杂。第一起的入口是一套可被读取的生产日志系统，云凭证随 sudo 环境变量被写进日志；第二起的入口是 1,542 个未授权访问的运维入口；第三起的入口是模板自带的老漏洞，SQL 注入、任意文件读取。没有未公开漏洞，攻击所用的漏洞和工具都来自公开渠道。**  
  
暴露面集中在三类：  
1. 运维面板。  
Grafana、Prometheus、Portainer、Kibana 等能看环境变量、能跑查询、能起容器的界面，逐个确认是否有鉴权、是否仅限管理网可达。  
  
1. 数据库与缓存。  
MySQL、MariaDB、Redis、ClickHouse、MongoDB 直接暴露在公网的，确认是否存在弱口令，确认监听地址是否收敛到内网。  
  
1. 日志与监控里的凭证。  
sudo 会把环境变量写进 auth.log，很多服务的启动参数里带着令牌，日志系统的读权限又经常收不紧。一次配置失误，等于把云账号的临时票据挂在了网上。  
  
要收掉不该对外的服务，前提是知道哪些真的对外。一家机构的域名、IP 段和云账号下面挂了多少  
端口、多少服务，靠人工台账对不上实际数：新开的测试环境、临时起的管理面板、忘了下线的旧节点，都不会自己写进台账。这件事只能靠资产测绘持续做，把自己名下的地址空间反复过一遍，再和台账逐条核对差异。  
  
把三起事件放在一起看，AI 的攻击力并不来自高深手段。拿下 59 台服务器靠的是一条写进日志的云凭证，进入 122 台主机靠的是敞开的入口，圈定 3,582 个交易平台目标靠的是一个现成的模板指纹。它来自 AI 昼夜不停的信息收集与漏洞挖掘，四天给 2,089 台主机建档，把 Blockdaemon 59 台机  
器上的 3,971 把钥匙列成清单，把一个运营者的生意按每笔交易重建出盈亏。这种速度与细度，大多数防守方对自己资产的掌握都没有。  
  
攻击者已经在用 AI 梳理你的资产，防守方的台账却还靠人工维护。要防住这样的攻击，最有效的办法是抢先用同样的方式梳理自己的资产，用 AI 持续盘点名下的地址空间、端口、服务和暴露的凭证，在下一轮扫描到来之前，把不该对外的收掉。  
  
▌  
IOC  
<table><tbody><tr><td data-colwidth="97" style="background-color:#ffffff;"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">类型</span></span></span></b></section></td><td data-colwidth="333" style="background-color:#ffffff;"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">IOC</span></span></span></b></section></td><td data-colwidth="141" style="background-color:#ffffff;"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><b style="" data-pm-slice="0 0 []"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">说明</span></span></span></b></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">IP</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">109.94.97[.]113</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例一工作机</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">IP</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">91.92.40[.]140</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例二工作机</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">9c8ozBMAomvSGtPmxmyFTdy3pYzKkue6qMU5ZuNcy7Wp</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例二攻击者钱包</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">HgM8EA5LXrmFgNiHjzB2Ergz89ehTLzhxnSZUratqKab</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例二攻击者钱包</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">2dpSKAf3Nnvc5SthdVS4HhUsowk2eEfaZaDvsSndUz6N</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例二攻击者钱包</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">IP</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">167.17.69[.]200</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例三工作机</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">bc1q4asxd73xclkyg757m9kzzgqch0t4u9mmun7nxv</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例三攻击者钱包</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">0xeDF6db038ADbf309A9019a78eC37ff55BF15b0eD</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例三攻击者钱包</span></span></span></section></td></tr><tr><td data-colwidth="97"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">钱包地址</span></span></span></section></td><td data-colwidth="333"><section style="text-align: justify;margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">TWChvSnDMaBVBdmhW3PLcKs6Ap3VHaZaK4</span></span></span></section></td><td data-colwidth="141"><section style="margin-bottom: 8px;line-height: 1.6em;text-indent: 0px;text-align: justify;"><span lang="EN-US" style="font-size: 10.5pt;font-family: 宋体;" data-pm-slice="0 0 []"><span leaf=""><span textstyle="" style="font-size: 13px;letter-spacing: normal;">案例三攻击者钱包</span></span></span></section></td></tr></tbody></table>  
Ferret 依托于 FOFA 在资产测绘领域的能力，持续监控攻击者的资产变更，通过 AI 摘要快速标定高价值内容，并将提取的 IOC 纳入关联分析流程，让公开目录从一次性发现变为可持续运营的情报来源。如有感兴趣的研究者欢迎联系交流。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m7oSwkxuQWeAo3QLEVD72Nq5ibDvSsd9ojChXMyyiaty7mbaUUr2NCvS0VeoOc3yLiaLM8k8TvVa7aMJrO1u52jLkxkvTIzJm4VYia5hKJKU0iaE/640?wx_fmt=png&from=appmsg "")  
  
图10：Ferret 详情页，AI 摘要自动标定高价值内容  
  
▌End  
  
欢迎各位白帽师傅们加入我们的社区大家庭，一起交流技术、生活趣事、奇闻八卦，结交无数白帽好友。  
  
也  
欢  
迎投稿  
到 FOFA  
，  
审核通过后可获得F点奖励，快来加入微信群体验吧~~~  
  
微信群：  
扫描下方二维码，加入 FOFA 社群！  
获取更多一手信息！  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6RgNHf9xEICZ9XbhPBz0GwJ4JJZsjiby26SsTibKgpMySib8ibChqtItmLFBP47vkhBknJLWia1Rz0D5hXVicUxMTicibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=25 "")  
  
