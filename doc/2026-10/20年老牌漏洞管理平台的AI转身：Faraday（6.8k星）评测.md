#  20年老牌漏洞管理平台的AI转身：Faraday（6.8k星）评测  
原创 赤霄攻防实验室
                    赤霄攻防实验室  赤霄攻防实验室   2026-10-19 02:09  
  
![封面｜武器库评测（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8aet2TWZtCicvDVBmdUPFrWWSQasnLXicwXgXGvokygHxqicUiahn6jx2LiawsdCCUE2jBlTHZgVvJFvTfnmrFSq3mAxgNYZ8ddFI8Zs/640?wx_fmt=png "")  
  
▲ 封面｜武器库评测（制图：赤霄攻防实验室）  
  
TL;DR  
  
是什么  
：Faraday——2008 年起迭代至今的开源漏洞管理平台（infobyte 出品），把 Burp/Nmap/OpenVAS 等 80+ 工具的扫描结果聚合成统一视图  
  
能干嘛  
：自动去重合并、多用户协作分配、仪表盘可视化、报告一键导出——让团队从「各自扫描」变成「共享发现」  
  
怎么用  
：Docker 全栈部署，数据自托管不出域，MIT 协议免费商用  
  
看点  
：安全团队的痛点从来不是「发现漏洞」而是「管住发现」——这是本号首篇防御向武器库评测  
  
![工具档案卡（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8ac2O30O1weGkb5JBnDorKE7WdTDMQQiaQ2EcsUycG0nImxD0QneORZ60hJS7S3rGQS3qUia1RmyvFicKibsDBicPx2r4Dz2BYvia9FfI/640?wx_fmt=png "")  
  
▲ 工具档案卡（制图：赤霄攻防实验室）  
## 它是什么：安全团队的「漏洞台账」  
  
渗透测试和安全评估有个长期痛点：扫描工具遍地都是，但发现的漏洞散落在 Burp 的项目文件、Nmap 的输出、OpenVAS 的报告里——格式各异、重复率高、没人跟进。Faraday 解决的就是这「最后一公里」：把 80+ 安全工具的输出聚合到一个平台，统一格式、自动去重、分配跟进，最终生成报告。  
  
项目由阿根廷安全公司 infobyte 于 2008 年开源，18 年持续迭代至今（6.8k★、MIT 协议），从早期的协作平台逐步演进为完整的漏洞管理系统——支持多用户、角色权限、资产关联、漏洞生命周期跟踪，以及仪表盘和趋势分析。对安全团队来说，这是一个「不上 SaaS 也能管漏洞」的务实选项。  
  
![仓库主页（来源：GitHub）](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8aerp787Jm58VialBGeP4n4cDmKNX9ztAU6e9wEoCcic83ZtPdsB5XRYlug5bJPp69PW1zMfwVia0zfMjZVfmzlEFlv0ia1zLwgggKM/640?wx_fmt=png "")  
  
▲ 仓库主页（来源：GitHub）  
## 架构与部署：Docker 全栈，数据不出域  
  
技术栈是 Python + PostgreSQL，官方提供 docker-compose 一键部署。数据完全自托管——对金融、政企等有数据合规要求的团队，这是选型时最关键的优势：漏洞数据和资产信息不经过第三方 SaaS。  
  
多源接入是核心竞争力：Burp Suite、Nmap、OpenVAS、Nessus、Qualys 等主流工具的扫描结果都有对应的导入插件，也可以通过 REST API 自定义对接。导入后 Faraday 自动按 CVE/主机/端口三维去重合并，避免不同工具报告同一漏洞时的重复计数。  
  
![工作流：从扫描到报告的聚合管线（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8afwicic9UKu6Sa9qCu6MYXIFcqn90hGicQPiaEDau8sHKBRPJTrYAR3XHM0aWXiajhtb50f8VICeNImVUDUIaX3MOQhWjxo2BakricAU/640?wx_fmt=png "")  
  
▲ 工作流：从扫描到报告的聚合管线（制图：赤霄攻防实验室）  
## 典型工作流：从渗透测试到交付报告  
  
一次典型的团队协作流程是这样的：红队成员各自跑完 Burp 和 Nmap 后，通过命令行客户端或 Web 界面把结果导入 Faraday；平台自动按主机和 CVE 去重合并，去掉重复发现；安全负责人在仪表盘上看到全量漏洞，按严重程度分配给对应的修复负责人；修复完成后更新状态，月底一键导出给管理层的 PDF 报告就搞定了。  
  
这套流程的核心价值是「发现不丢失」——每个漏洞都有生命周期状态（Opened → Risk assessed → Remediated → Closed），任何时刻都能回答「上周扫出的高危漏洞修了几个」这个灵魂拷问。  
## 谁在用它：从红队到合规的中间地带  
  
Faraday 的用户画像是「有持续安全测试需求的团队」：红队用它管理历次测试的发现，安全运营用它跟踪修复进度，合规团队用它生成审计报告。与商业平台（如 Kenna/Brinqa）相比，Faraday 的优势是免费和灵活——但缺少商业产品的风险评分算法和 SLA 报告。  
  
值得注意的是项目正在引入 AI 能力：辅助分析扫描结果、优先级排序。虽然尚在演进期，但方向正确——漏洞管理的瓶颈从来不是「收集」而是「排序」，AI 恰好能补上这块。  
  
![同类对比：漏洞管理平台怎么选（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8acZodDI0vdfjBibcbp1prbFl2ceTQ4XbSRHvSbY5grXDKPb1VQbyGBeEF7BwlD1hlFibCTtszLZKmvgwaAVVmgooxKFfuhEWp2iaE/640?wx_fmt=png "")  
  
▲ 同类对比：漏洞管理平台怎么选（制图：赤霄攻防实验室）  
## 选型建议  
  
如果你是初创安全团队或独立顾问：Faraday 的开源版足够起步，MIT 协议允许商用。如果你是中大型企业：Faraday 可以作为自托管底座，但需要投入运维资源。如果你更在意 AI 自动化和厂商支持：商业平台更省心，但预算和合规是门槛。  
  
一句话总结：Faraday 是漏洞管理领域的「Debian」——不是最炫的，但是最可靠的底座之一。  
## 参考来源  
  
1. 仓库：https://github.com/infobyte/faraday（MIT）  
  
2. 官方文档：docs.faradaysec.com  
  
3. 发布日志：RELEASE.md  
  
免责声明：本文仅用于安全研究与防御交流；工具选型请结合团队实际情况评估。  
  
