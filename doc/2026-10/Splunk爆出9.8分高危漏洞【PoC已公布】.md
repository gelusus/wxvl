#  Splunk爆出9.8分高危漏洞【PoC已公布】  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-15 02:22  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
#   
  
## CVE-2026-76268：通过Patroni API实现未授权远程代码执行  
  
**重大安全警报**  
：Splunk已于2026年10月7日紧急修复了编号为CVE-2026-76268的严重安全漏洞。攻击者只需通过未经验证的网络连接访问搜索头集群成员上的**Patroni REST API**  
，便可直接执行任意操作系统命令。该漏洞CVSS 3.1评分高达**9.8分**  
(属于"严重"级别，由官方Splunk安全团队评估)。修复版本为**Splunk Enterprise 10.2.7**  
和**10.4.3**  
。目前，第三方漏洞研究人员已声称开发出概念验证代码(PoC)。不过，美国网络安全和基础设施安全局(CISA)暂时未将其列入已知被利用漏洞(KEV)目录。  
  
漏洞PoC代码见文末链接  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZiaDdA5rXQRrr4EDYn1iaXemLCj9081XcP95PQYTSU2WsLEH5Iza16wgQlLaZiaH3TMZgUYnTItHINd25jfRt6BFkw6uIiatSsYFNE/640?wx_fmt=png&from=appmsg "")  
  
## 事态概述  
  
Splunk官方于**2026年10月7日**  
发布了编号为SVD-2026-1001的安全公告，一次性修复了Splunk Enterprise平台中的多个安全缺陷。其中最严重的当属**CVE-2026-76268**  
，其安全等级被评定为"严重"级别。  
  
该漏洞的根本问题在于**Patroni REST API**  
的身份验证机制缺失。Patroni是负责管理搜索头集群成员中PostgreSQL辅助进程(sidecar)的组件。令人震惊的是，这个API在关键配置操作上**竟然完全不要求任何身份验证**  
，这无异于为攻击者敞开了后门。  
  
据官方披露，漏洞是由Splunk公司内部的安全研究员**Gabriel Nitu**  
发现的，系**内部安全审计**  
的成果。  
## 影响范围详解  
  
本次漏洞波及以下Splunk Enterprise版本：  
- **10.2.0至10.2.6系列**  
的全部版本  
  
- **10.4.0至10.4.2系列**  
的全部版本  
  
值得注意的是，Splunk Enterprise 10.0.x和9.4.x这两个较早的版本线**不受**  
此漏洞影响。  
  
安全公告明确提到，受攻击风险集中在**搜索头集群成员**  
。要利用此漏洞，攻击者需要首先获得对**Patroni API的网络访问权限**  
。令人担忧的是，官方并未公布具体有多少实际生产环境将此API暴露在网络上。因此，安全建议是：**在确认安全状态前，将所有可被访问的集群成员都视为已暴露的攻击面。**  
## 已核实的技术细节  
  
**漏洞评分与特征：**  
- Splunk官方评估的CVSS 3.1评分：**9.8分(严重级别)**  
  
- 国家漏洞数据库(NVD)确认相同评分，弱点分类为**CWE-306(关键功能缺少身份验证)**  
  
- 核心危害：**未经身份验证的攻击者可以直接在宿主机上执行任意操作系统命令**  
  
**官方修复方案：**  
- 彻底修复版本：**Splunk Enterprise 10.2.7**  
和**10.4.3**  
  
- 临时缓解措施：如果您的部署环境中**未使用Edge Processor、OpAmp或SPL2管道**  
，可**禁用PostgreSQL sidecar组件**  
来规避风险。应用此措施后，请重启Splunk服务。  
  
**漏洞状态追踪：**  
- 该漏洞**尚未**  
被CISA列入"已知被利用漏洞(KEV)"目录  
  
- CISA通过安全漏洞分类(SSVC)系统向NVD报告的漏洞利用情况为"**未发现野外利用**  
"  
  
- 漏洞扫描厂商Tenable已在其扫描器插件中更新该漏洞的检测方法，并标记为"**存在可用利用程序**  
"  
  
**第三方安全研究：**  
- GitHub平台上用户名为"**murrez**  
"的账户已于2026年10月13日发布一个公开仓库，声称拥有该漏洞的**完整概念验证代码(PoC)**  
。该仓库不仅包含**漏洞验证检测工具**  
，甚至还有**大规模自动化利用工具**  
。需要特别强调的是：ThreatWire并未实际运行或审核这些代码，且该GitHub账户**并非**  
Splunk官方认可的漏洞发现者。  
  
## 尚未确认的关键信息  
  
**安全监控与攻击态势：**  
- 截至目前，**没有任何可靠来源报告在真实攻击环境**  
中监测到此漏洞的利用活动  
  
- ThreatWire**无法独立验证**  
第三方PoC的实际功能和有效性  
  
- **无法统计**  
当前互联网上究竟有多少Splunk集群将Patroni API暴露在非受信网络中  
  
**媒体报道与企业透明度：**  
- 截至本文撰写时，ThreatWire**尚未发现**  
主流安全媒体对此漏洞进行详细报道  
  
- Splunk官方公告中**未明确说明**  
通过此漏洞执行的命令**具体的权限级别**  
（如root权限、普通用户权限等），这可能影响企业风险评估的策略制定。  
  
## 紧急防护建议  
  
**根本性解决方案：**  
- **立即升级**  
Splunk Enterprise至10.2.7或10.4.3版本  
  
**临时应急措施：**  
- 如无法立即升级，请遵循官方建议，在确保**不影响业务管道(Edge Processor/OpAmp/SPL2)**的前提下，**启用PostgreSQL sidecar组件禁用方案**  
  
**网络安全加固：**  
- **严格限制**  
搜索头集群成员的**网络访问权限**  
，仅对受信主机开放  
  
- **全面审查**  
受影响的宿主机是否存在**异常的配置变更**  
或**可疑的进程活动**  
  
- 特别提醒：随着公开PoC的流传，**互联网扫描探测活动将显著增加**  
。请优先保护那些**可能从管理网络外部被访问**  
的集群环境！  
  
**信息引用来源：**  
 Splunk安全公告SVD-2026-1001、国家漏洞数据库(NVD)、CVE.org官方记录、Splunk技术文档以及Tenable漏洞情报平台。  
  
原文：  
https://www.threatwire.tech/news/splunk-enterprise-patroni-flaw-cve-2026-76268-allows-unauthenticated-rce  
  
**PoC：https://github.com/murrez/CVE-2026-76268**  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
