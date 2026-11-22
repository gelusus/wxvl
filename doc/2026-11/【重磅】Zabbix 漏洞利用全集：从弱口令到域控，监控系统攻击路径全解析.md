#  【重磅】Zabbix 漏洞利用全集：从弱口令到域控，监控系统攻击路径全解析  
原创 CabinQ
                        CabinQ  逆熵寻生   2026-11-22 01:27  
  
【重磅】Zabbix 漏洞利用全集：从弱口令到域控，监控系统攻击路径全解析  
  
【深度长文】全文约 20000 字 | 含 12 个完整利用链 | 覆盖 28 个 CVE（2018—2026 全时间线，含 2026 年 10 月最新公告）| 含完整工具资源包  
  
核心结论  
  
Zabbix 是全球部署量最大的开源监控平台之一，被金融、电信、政府、能源、互联网等行业广泛采用，是整个 IT 基础设施的"监控中枢"。它通过 SNMP、IPMI、JMX、Agent 等多种协议接入并采集数据，覆盖从物理服务器到云原生容器的几乎所有资源类型——这也意味着，一旦 Zabbix 被攻破，攻击者获得的不再是单点权限，而是所有被监控主机的"上帝视角"：运行状态、网络拓扑、资产清单，甚至操作系统登录凭据全部暴露。  
  
本文系统梳理 Zabbix 四层攻击面（Web 前端 / Server / Agent / Proxy），从资产发现到认证绕过、从 SQL 注入到未授权 RCE、从 Agent 劫持到 DLL 提权，拆解 28 个真实 CVE 的完整利用链，覆盖 2018—2026 全时间线关键漏洞（含 2026 年 8 月/10 月两批官方公告中的会话密钥硬编码、sortfield 时间盲注等新漏洞），并附上 HTB Watcher、Vulnlab Watcher、HTB Unrested 三个实战靶场的完整复盘。  
  
读完本文，你将掌握从公网 Zabbix 面板识别到内网域控权限的完整攻击路径，能够独立完成 Zabbix 渗透测试项目。  
  
CVE 速览  
  
🔹 CVE-2020-11800：Zabbix Server Trapper 未授权命令注入 RCE，CVSS 9.8，无需认证直击 10051 端口  
  
🔹 CVE-2022-23131：SAML SSO 认证绕过，CVSS 9.1，CISA KEV 收录（2022-02-22），EPSS 约 95%，伪造 sessionid 直接伪造管理员登录  
  
🔹 CVE-2022-23134：setup.php 未授权配置泄露，Zabbix 官方口径 3.7 / 聚合库口径 5.3，CISA KEV 收录，数据库凭据招手即得  
  
🔹 CVE-2023-29450：JavaScript 预处理器任意文件读取，CVSS 8.5，凭据收割利器  
  
🔹 CVE-2024-22116：受限管理员脚本参数注入 RCE，CVSS 9.9（官方 ZBX-25016 口径），面板内直接弹 shell  
  
🔹 CVE-2024-22120：审计日志时间盲注到管理员会话伪造 RCE，CVSS 9.1（官方公告口径），HTB Watcher 核心链  
  
🔹 CVE-2024-42327：非管理员 API SQL 注入提取 Admin Token，CVSS 9.9，HTB Unrested 立足点  
  
🔹 CVE-2025-27233：Agent 2 smartctl 插件 NTLMv2 哈希泄露，CVSS 5.7，域环境凭据收割  
  
🔹 CVE-2025-27234：Agent 2 smartctl 插件参数注入 RCE，CVSS 7.3，Zabbix 5.0 专属直击 root  
  
🔹 CVE-2025-27237：Agent Windows 版本 DLL 注入本地提权至 SYSTEM，CVSS 7.3  
  
🔹 CVE-2026-23933：前端会话签名密钥硬编码写入数据库 seed（2026-08 公告，CVSS 7.7），可伪造任意用户会话，7.4.0-7.4.10 全线中招  
  
🔹 CVE-2026-23921：API sortfield 时间盲注 SQL 注入（2026-03 批次修复），低权限用户再次打通"注入 → 会话窃取 → RCE"老路  
  
🔹 CVE-2026-23920：主机/事件动作脚本的多行正则校验绕过，换行符即可注入 shell 命令（2026-03 批次修复）  
  
🔹 CVE-2026-23924：Agent 2 Docker 插件参数注入，可读取运行中容器内的任意文件（2026-03 批次修复）  
  
🔹 CVE-2026-1199：API/前端登录锁定存在竞态条件（CVSS 6.9），账号锁定形同虚设，爆破助攻神器  
  
🔹 CVE-2026-23937：Zabbix API 可提取主机 PSK（CVSS 6），Agent 通信密钥直接易主  
  
收益清单  
  
1. 掌握 Zabbix 资产识别全手法（端口指纹 + 组件识别 + 版本匹配 + Shodan/Fofa 语法），快速定位目标网络中的 Zabbix 组件  
  
2. 掌握 Web 前端认证绕过：CVE-2022-23131 SAML 会话伪造、CVE-2022-23134 setup.php 配置泄露、CVE-2026-23933 会话密钥硬编码的完整利用步骤  
  
3. 掌握未授权直击 Server：CVE-2020-11800 通过 10051 端口 Trapper 协议实现无需认证的 RCE  
  
4. 掌握三条 SQL 注入到 RCE 的完整攻击链：CVE-2024-22120 时间盲注伪造管理员会话、CVE-2024-42327 API 注入直接提取 Admin Token、CVE-2026-23921 sortfield 时间盲注  
  
5. 掌握 Agent 层攻击：CVE-2025-27234 smartctl 参数注入 RCE、CVE-2025-27233 NTLM 哈希泄露、CVE-2025-27237 DLL 提权至 SYSTEM、CVE-2026-23924 Docker 容器文件读取  
  
6. 掌握后渗透路径：从 zabbix 用户 shell 收割数据库凭据、绘制内网拓扑、通过 system.run 批量控制被监控主机  
  
7. 获得十余个真实开源利用工具与源码，覆盖扫描、爆破、注入、RCE 全链路，开箱即用  
  
8. 通过 HTB Watcher / Vulnlab Watcher / HTB Unrested 三个实战靶场复盘，理解漏洞在真实环境的落地方式  
  
**适用读者**  
：渗透测试工程师、红队攻击队员、安全研究员、CTF 选手、SRC 漏洞挖掘者。  
  
**免责声明**  
：本文所有技术内容仅面向授权安全测试与教育研究，任何未经授权的渗透测试行为均违反中华人民共和国网络安全法、刑法第二百八十五条等相关法律，滥用造成的后果由使用者自行承担。  
  
**下载说明**  
：文末资源包包含完整的工具集，覆盖 CVE-2020-11800、CVE-2022-23131、CVE-2024-22120、CVE-2024-42327 的完整 PoC 与 exploit 源码，以及 zabbixguiscanner 批量扫描器、GenZabbixSQLi 通用注入工具、zabbixservertimebasedblind_sqli 盲注脚本等开源工具。  
  
一、Zabbix 攻击面总览：为什么监控系统是红队的终极跳板  
  
1.1 Zabbix 架构与四层攻击面  
  
