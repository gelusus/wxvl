#  CVSS 10.0！特定条件下 GitLab 远程代码执行漏洞  
原创 微步情报局
                    微步情报局  微步在线研究响应中心   2026-10-14 09:14  
  
漏洞概况  
  
  
GitLab CE/EE 是  
 GitLab 公司开发的一体化 DevOps 平台(社区版/企业版),提供 Git 仓库托管、CI/CD 流水线、代码审查与项目管理等能力，广泛用于企业源代码协作与持续集成。  
  
近日，GitLab CE/EE官方发布通告，修复了GitLab CE/EE 远程代码执行漏洞（CVE-2026-97229）。  
由于正则表达式校验不足，  
满足特定条件时，  
未经身份验证的用户可通过 CI/CD 配置中的特制正则表达式，  
在 GitLab 服务器上执行任意代码。  
微步情报局已成功复现该漏洞。  
  
利用  
前置条件：  
  
1.   
需要事先在仓库植入恶意 `.gitlab-ci.yml`：仓库  
允许  
新用户自  
行注册并创建项目  
，或攻击者具备仓库写权限。  
  
2. 攻击者可访问CI Lint 接口：公开仓库或  
攻击者具备仓库读权限  
  
经分析，GitLab 使用的 Ruby 正则引擎 Onigmo 在编译带重复次数的正则时，用有符号 32 位整数计算 ”子程序长度 × 重复次数“，乘积超  
过上  
限会发生整数溢出。回绕后的数据被写进 Onigmo 字节码的跳转偏移，可造成控制流劫持。  
  
已证实的RCE利  
用链需要事先在仓库  
植入恶意 `.gitlab-ci.yml`；对于公开仓库，攻击者可通过匿名请求 `GET /api/v4/projects/:id/ci/lint?content_ref=<该分支>` 触发 CI Lint 检查 `.gitlab-ci.yml`。因正则校验不足，CI Lint 按 `content_ref` 读取已入库文件并调用 Onigmo 编译其中的正则，触发整数溢出，  
并在后续利用中以 `git` 用户权限实现远程代码执行。  
  
（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-76904）  
  
漏洞处置优先级(VPT)  
  
  
**综合处置优先级：**  
高风险  
<table><tbody><tr><td rowspan="3" style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">基本信息</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;"><section><span leaf="">微步编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">XVE-2026-76904</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE编号</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">CVE-2026-97229</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程代码执行(整数溢出)</span></section></td></tr><tr><td rowspan="5" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用条件评估</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的网络条件</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">远程</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要绕过安全机制</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">对被攻击系统的要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">无特殊要求</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">利用漏洞的权限要求</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">完整利用需具备仓库写权限</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否需要受害者配合</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">否</span></section></td></tr><tr><td rowspan="2" style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-weight:bold;background-color:#f8f9fa;"><section><span leaf="">利用情报</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">是否有POC</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><span style="color:#d93025;font-weight:bold;"><span leaf="">是</span></span></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">已知利用行为</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">暂无</span></section></td></tr></tbody></table>  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">产品名称</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">GitLab CE/EE</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">19.2&lt;=version&lt;19.2.8、19.3&lt;=version&lt;19.3.5、19.4&lt;=version&lt;19.4.2</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
漏洞复现  
  
  
执行POC，植入恶意CI/CD配置后，  
攻击者发送请求  
触发 CI Lint 检查，触发整数溢出，实现任意代码执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEMFl4kSXU4m760uO8iaJ5sFvmXXL9MvnW3pYFrmfvqkKqlKE5FXjze7VnWkvxqYGTzvDnR52iacrZdYYq9n5d1TFVMX5fxnZ9wEI/640?wx_fmt=png&from=appmsg "")  
  
修复方案  
  
### 官方修复方案  
  
官方已发布修复方案，请访问链接下载：  
  
https://docs.gitlab.com/releases/patches/patch-release-gitlab-19-4-2-released/  
### 临时缓解措施  
  
1.   
禁止不可信账号新建公开项目，并禁止其向公开项目推送。  
  
2.   
停止向未登录用户开放公开项目。  
  
微步产品支撑  
  
  
微步漏洞情报于  
2026-10-13  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
  
