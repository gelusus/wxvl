#  微软发布3个Active Directory漏洞预警，支持远程绕过安全防护  
 看雪学苑   2026-10-15 10:06  
  
微软官方披露了3个Active Directory身份服务漏洞，编号分别为：CVE-2026-69982、CVE-2026-96252、CVE-2026-108131。漏洞属于安全特性绕过类缺陷，远程攻击者可利用漏洞绕过Windows域环境的核心安全管控能力。  
  
  
**漏洞背景说明**  
  
  
  
  
  
Active Directory（AD）是绝大多数企业内网的身份中枢，统一管理域内账号、终端、权限以及业务系统访问权限。域控承载整个域的身份认证与信任关系，一旦安全控制被绕过，风险会快速扩散至整个内网。  
  
✅ 漏洞类型解读：  
  
安全特性绕过漏洞不会直接让攻击者获取服务器完全控制权，但能够绕开原本用于访问校验、拦截非法请求、阻断恶意行为的安全防护机制。  
  
  
攻击者拿到内网立足点后，常以域控为最终目标，尝试横向移动拿下身份基础设施；这类绕过漏洞，将直接帮助攻击者规避检测、突破安全策略。  
  
  
**企业安全团队处置要点**  
  
  
  
  
  
本次漏洞属于高优先级补丁事件，建议企业尽快启动修复流程。  
  
1. 全面盘点资产  
  
梳理全部域控制器，包含老旧域控、备份域控、分支机构远程机房的域控主机，不要遗漏冷门节点。  
  
  
2. 分阶段上线补丁  
  
优先在测试环境验证更新包，验证无业务异常后，快速批量部署至生产域控与相关Windows服务器。  
  
  
3. 核验补丁安装状态  
  
不要仅凭“推送成功”判断，通过终端管理平台、Windows更新报表确认补丁实际安装生效。  
  
  
4. 加固网络边界  
  
域控服务严禁直接暴露公网；收紧访问白名单，限制可访问目录服务的主机范围；管理网与业务网隔离，高权限账号强制强认证。  
  
  
5. 加强日志监控  
  
持续监测域控相关异常行为：认证失败突增、权限异常变更、新增可疑服务账号等。  
  
  
**风险提示**  
  
  
  
  
  
微软公开资料暂未发现这3个漏洞存在在野利用的证据。但安全团队不可等出现真实攻击案例再修复。  
  
  
安全特性绕过漏洞具备很高的组合攻击价值，可搭配凭证窃取、钓鱼攻击、横向移动或其他漏洞串联使用，组合攻击后极易造成企业域环境沦陷。  
  
  
企业管理员建议查阅微软安全更新指南，核对受影响产品、补丁包、严重等级与缓解方案，相关信息会随官方更新持续调整，以微软MSRC公告为准。  
  
近期身份与远程接入类漏洞频发，此前Citrix NetScaler、Gateway等远程网关漏洞也给多家企业带来安全威胁，身份基础设施的补丁优先级，应当和对外业务系统保持一致。  
  
  
资讯来源：Cyber Security News  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K2F0bLeiaicKuXoCSHkYLz2lTcTVgKtW8G6ggctN7ah5BL9MvHIC15NLGIfekCDV5z3icDiaSQh46qTntHtyulibkvH2h14ZspZ7H7Q/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K35pnfNFDS66o2NI3NicEWpOgcgYrLWeHceYkjKI2KCoohSmMwBiaxbNb3LichHnwqsj2UAanqyNM9vgVxgicexWXpJc4lAGTGibdm4/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3vxqZlo4kaF9s7s9w8XvLzNrH722icQztWtzNgsng0Bv09mh96lCFLLJ8XDy3dUSKmo2KKnUodHpN6qWNtG8IVHt8nhQtPjwfg/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0VOjTLVCoIGibmKAbgZTCfib4b8TABpQwFZ9nD1kvIl3uwGoicZiclqZiaIsh2QtaVISiahEOwfibbGDkkAR1exONEWLRyPTQYyvcev8/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
