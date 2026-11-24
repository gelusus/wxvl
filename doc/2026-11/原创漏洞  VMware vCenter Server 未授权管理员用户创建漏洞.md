#  原创漏洞 | VMware vCenter Server 未授权管理员用户创建漏洞  
 微步在线研究响应中心   2026-11-24 10:12  
  
## 漏洞概况  
  
11月24日，vCenter Server官方发布通告，修复了VMware vCenter Server 未授权管理员用户创建漏洞。**该漏洞由微步XGPT发现并报送VMware官方。**  
经分析，VMware vCenter Server 是 VMware 推出的虚拟化管理平台，其内置 vmdir 目录服务承载 SSO 身份目录。该服务在特定认证流程中存在访问控制缺陷，攻击者能访问 389 端口情况下，可能绕过身份验证，创建管理员账号并接管 vCenter。（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-62902）  
  
此漏洞**无需任何权限**  
，攻击者成功利用此漏洞可**创建管理员账号并接管 vCenter，跟****11月19日VMware官方发布CVE-2026-123890不是同一个漏洞，**  
建议受影响用户**尽快修复。**  
## 漏洞处置优先级(VPT)  
  
**风险等级：高风险**  
**风险分数：9.8**  
<table><tbody><tr><td rowspan="3" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">基本信息</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">微步编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">XVE-2026-62902</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">CVE编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">CVE-2026-123903</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">漏洞类型</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">身份鉴别错误</span></p></td></tr><tr><td rowspan="5" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用条件评估</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的网络条件</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">可远程利用</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要绕过安全机制</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">对被攻击系统的要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">默认配置</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的权限要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">无需任何权限</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要受害者配合</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td rowspan="2" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用情报</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">POC是否公开</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">否</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">已知利用行为</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">否</span></p></td></tr></tbody></table>## 漏洞影响范围  
<table><tbody><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">产品名称</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">Vmware vCenter Server</span></p><p style="margin: 0px;"><span leaf="">VMware Cloud Foundation</span><span leaf=""><br/></span><span leaf="">Vmware Telco Cloud Platform</span><span leaf=""><br/></span><span leaf="">Vmware Telco Cloud Infrastructure</span></p></td></tr><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">受影响版本</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">vCenter Server:</span></p><p style="margin: 0px;"><span leaf=""> 9.1.x.x&lt;=version&lt;9.1.2</span><span leaf=""><br/></span><span leaf="">vCenter Server:</span></p><p style="margin: 0px;"><span leaf=""> 9.0.x.x&lt;=version&lt;9.0.2.0300</span><span leaf=""><br/></span><span leaf="">vCenter Server:</span></p><p style="margin: 0px;"><span leaf=""> 8.0&lt;=version&lt;8.0U3m</span></p><p style="margin: 0px;"><span leaf="">vCenter Server: 7.x 未提供修复版本，联系官方咨询</span></p><p style="margin: 0px;"><span leaf="">VMware Cloud Foundation: 5.x&lt;=version&lt;5.2.5</span></p><p style="margin: 0px;"><span leaf="">Telco Cloud Platform: 3.0, 4.x, 5.0.x, 5.1.x, 5.2</span></p><p style="margin: 0px;"><span leaf="">Telco Cloud Infrastructure: 3.0</span></p></td></tr><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">有无修复补丁</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39626</span></p></td></tr></tbody></table>## 漏洞复现  
  
1. 首页  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEOYcicict6I3TyILQRvEKF9DoHjsaScRQjkQGeBtnv6ibhSscULibzJWdWrbSgMY0cWh9Skrf4ytLR4GiawlVwo07sKncpylFqbe8ec/640?from=appmsg "")  
  
2. 在8.0.3.01000版本执行poc，创建新用户  
  
![image.png](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEPThhoIYMUwVySD1XEyM7Etn32HchNsZQUOpI9EjTmHxvTOhq3NApF21Ya8YNAEDQLRlVVlAVTQzF0RWYrgGUH0bwicicc9H9mFk/640?from=appmsg "")  
  
3. 登录后台  
  
![image.png](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEMAcibQ8nW7fpc8OH0o9H5ycSgzZ29WZGxhXbibJhyC9kaNIia3cxpnZPTrIqk0OD4Cmasg2mYbcDicwuoRzDQw61ETzNe5HjC2osc/640?from=appmsg "")  
## 修复方案  
### 官方修复方案  
  
https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39626  
  
## 微步产品支撑  
  
微步漏洞情报于**2026年9月14日**  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP，高级威胁情报平台ATI及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
