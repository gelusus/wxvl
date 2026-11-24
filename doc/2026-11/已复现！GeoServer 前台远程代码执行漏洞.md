#  已复现！GeoServer 前台远程代码执行漏洞  
 微步在线研究响应中心   2026-11-24 15:53  
  
## 漏洞概况  
  
GeoServer 是一个基于 Java 的开源地理信息服务器，用来发布和共享地理空间数据。它能把 PostGIS、Shapefile 等数据发布为符合 WMS、WFS、WCS 等 OGC 标准的服务，供网页地图和 GIS 客户端使用。  
  
今日，GeoServer官方发布安全通告，披露了5个安全漏洞。这些漏洞被组合利用时，可能使未登录的攻击者以 GeoServer 服务进程的权限执行代码。**微步情报局已成功复现。**  
经分析，5个漏洞涉及 WFS 请求处理、获取管理会话和 JDBC 反序列化漏洞。这些漏洞被组合利用时，可能使未登录的攻击者实现无需认证的远程命令执行。  
  
此漏洞**无需任何权限**  
，攻击者成功利用此漏洞可**攻击者可实现无需认证的远程命令执行，**  
建议受影响用户**尽快修复。**  
## 漏洞处置优先级(VPT)  
  
**风险等级：高风险**  
**风险分数：9.8**  
<table><tbody><tr><td rowspan="3" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">基本信息</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">微步编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">XVE-2026-103268</span></p><p style="margin: 0px;"><span leaf="">XVE-2026-103269</span></p><p style="margin: 0px;"><span leaf="">XVE-2026-103302</span></p><p style="margin: 0px;"><span leaf="">XVE-2026-103304</span></p><p style="margin: 0px;"><span leaf="">XVE-2026-103305</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">CVE编号</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">CVE-2026-129242</span></p><p style="margin: 0px;"><span leaf="">CVE-2026-129243</span></p><p style="margin: 0px;"><span leaf="">CVE-2026-129244</span></p><p style="margin: 0px;"><span leaf="">CVE-2026-129245</span></p><p style="margin: 0px;"><span leaf="">CVE-2026-120705</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">漏洞类型</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">远程代码执行</span></p></td></tr><tr><td rowspan="5" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用条件评估</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的网络条件</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">可远程利用</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要绕过安全机制</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">对被攻击系统的要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">默认配置</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">利用漏洞的权限要求</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">无需任何权限</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">是否需要受害者配合</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">不需要</span></p></td></tr><tr><td rowspan="2" style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">利用情报</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">POC是否公开</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">否</span></p></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">已知利用行为</span></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">否</span></p></td></tr></tbody></table>## 漏洞影响范围  
<table><tbody><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">产品名称</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">GeoServer | GeoServer</span></p></td></tr><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">受影响版本</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">GeoServer &lt;= 2.27.6</span></p><p style="margin: 0px;"><span leaf="">GeoServer 2.28.0 - 2.28.5</span></p><p style="margin: 0px;"><span leaf="">GeoServer 3.0.0 - 3.0.1</span></p></td></tr><tr><td style="background-color: rgb(248, 249, 250);border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><strong style="font-weight: 700;"><span leaf="">有无修复补丁</span></strong></p></td><td style="border: 1px solid rgb(221, 221, 221);padding: 8px 10px;vertical-align: top;"><p style="margin: 0px;"><span leaf="">有</span></p></td></tr></tbody></table>## 漏洞复现  
  
![168677692a4fbb71cb9e3235bedc7448.png](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEO82icaKialFxmqRc01KE46IWNVlRibVia5L3UCN5f1DbpKHiccjZNyreoVXfrWpb38JADvSlQCMNGabOOFKL5icJrG9AJydQmTF3oxM/640?from=appmsg "")  
  
## 修复方案  
### 官方修复方案  
  
https://github.com/geoserver/geoserver/releases  
### 官方临时缓解建议  
  
防护设备添加规则，拦截 /geoserver/wfs 请求携带 ID 参数含 xi:include、fes:、file:/// 等XInclude特征  
## 微步产品支撑  
  
微步漏洞情报于**2026年11月24日**  
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP，高级威胁情报平台ATI及X情报中心已于漏洞公开时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
