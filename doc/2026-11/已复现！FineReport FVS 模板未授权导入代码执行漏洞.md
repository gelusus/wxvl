#  已复现！FineReport FVS 模板未授权导入代码执行漏洞  
 微步在线研究响应中心   2026-11-23 12:10  
  
## 漏洞概况  
  
FineReport是企业级Web报表软件，基于Java架构，广泛应用于报表设计与数据可视化场景。  
  
近日，FineReport报表软件官方发布更新，修复了 FineReport FVS 模板未授权导入代码执行漏洞。**微步情报局已成功复现。**  
经分析，该系统 FineVis 模板复用与导入相关接口存在鉴权与输入校验缺陷，未登录攻击者可向服务端提交特制模板包，将可控内容投递至应用的类加载路径，最终经无需认证的文件访问接口完成实例化执行，以应用服务进程权限运行任意系统命令。（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-61443）  
  
此漏洞**无需任何权限**  
，攻击者成功利用此漏洞**可实现前台远程代码执行，**  
建议受影响用户**尽快修复。**  
## 漏洞处置优先级(VPT)  
  
**风险等级：高风险**  
**风险分数：9.8**  
<table><tbody><tr><td rowspan="3" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">基本信息</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">微步编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">XVE-2026-61443</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">CVE编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">-</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">漏洞类型</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">代码注入</span></p></td></tr><tr><td rowspan="5" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用条件评估</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的网络条件</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">可远程利用</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要绕过安全机制</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">对被攻击系统的要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">默认配置</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的权限要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">无需任何权限</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要受害者配合</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td rowspan="2" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用情报</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">POC是否公开</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">否</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">已知利用行为</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">已捕获PoC，极有可能被利用</span></p></td></tr></tbody></table>## 漏洞影响范围  
<table><tbody><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">产品名称</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">帆软软件有限公司 | FineReport报表软件</span></p></td></tr><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">受影响版本</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">FineReport报表软件: version&lt;11.5.16</span><span leaf=""><br/></span><span leaf="">FineVis 插件 &lt; 4.15.0</span></p></td></tr></tbody></table>## 漏洞复现  
  
1. 产品首页  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEPkVKIzJ0jrcJxczpOQbNS0UKFQPaR6BIicOMwvYhgMjBDIiaruFfHplg1ia4avkznibaISQdSJTJtrzSjGEY5lNoZwNHxq6hj6pIA/640?from=appmsg "")  
  
2. 在11.5.15版本测试，执行命令  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/T4OSm0sXdEM56p9mibRWcVEp6mOyqDicwD2VzQ1kPZ7wS954opAkB6arQv28hkkxZTVPpLHEKtiafPtqAqDrCYhCicPwqz9jlbqu7zo6HlZNDN0/640?from=appmsg "")  
## 修复方案  
### 官方修复方案  
  
官方已发布修复版本：https://help.fanruan.com/finereport/doc-view-4699.html  
## 微步产品支撑  
  
微步漏洞情报于**2026年9月10日**  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报社区已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
