#  CI/CD工具—TeamCity被曝出两个严重漏洞   
 安全客   2024-03-05 16:17  
  
JetBrains 修复了两个影响 TeamCity On-Premises 的严重安全漏洞（CVE-2024-27198、CVE-2024-27199），并敦促客户立即修补它们。  
  
“Rapid7 最初发现并向我们报告了这些漏洞，并选择严格遵守自己的漏洞披露政策。这意味着他们的团队将在收到通知后 24 小时内发布这些漏洞的完整技术细节及其复制步骤。”该公司今天表示。  
  
这也意味着概念验证和充分利用可能会迅速浮出水面并得到利用。  
  
**关于漏洞（CVE-2024-27198、CVE-2024-27199）**JetBrains 的 TeamCity 是一款持续集成和持续交付 (CI/CD) 服务器，其中的漏洞最近被俄罗斯和朝鲜国家支持的攻击者利用。  
  
CVE-2024-27198 可能允许未经身份验证的远程攻击者危害易受攻击的 TeamCity 服务器，并通过创建新的管理员用户或生成新的管理员访问令牌来控制与服务器关联的所有项目、构建、代理和工件。这使得该漏洞成为发起供应链攻击的理想选择。  
  
第二个漏洞 CVE-2024-27199 允许有限数量的信息泄露和有限数量的系统修改，包括未经身份验证的攻击者能够使用攻击者的证书替换易受攻击的 TeamCity 服务器中的 HTTPS 证书。攻击者可以通过将 HTTPS 端口号更改为客户端不期望的值，或者上传无法通过客户端验证的证书，对 TeamCity 服务器执行拒绝服务。或者，如果攻击者上传的证书（并拥有私钥）受到信任，则在网络上拥有适当位置的攻击者可能能够对客户端连接执行窃听或中间人攻击。  
  
它们影响 2023.11.3 之前的所有 TeamCity On-Premises 版本，已在版本 2023.11.4 中修复。  
  
**更新、修补或让服务器脱离互联网**如果无法将服务器升级到 v2023，建议升级到固定版本（手动或使用解决方案中的自动更新选项）或应用安全补丁插件（与所有 TeamCity 版本兼容）。  
  
“JetBrains 的政策通常涉及在发布后较长一段时间内隐藏漏洞的技术细节，以确保彻底缓解；然而，这种加速的时间表需要立即升级服务器或打补丁以防止利用，”该公司补充道。“如果您的服务器可通过互联网公开访问，并且您无法立即执行下述缓解步骤之一，我们强烈建议您在缓解操作完成之前将您的服务器设置为不可访问。”  
  
  
**来**  
  
**领**  
  
**资**  
  
**料**  
  
**【免费领】**  
**网络安全专业入门与进阶学习资料，轻松掌握网络安全技能！**  
  
****![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ok4fxxCpBb59ibIezbic1Dob2DsGBgT7WkA3sJgtXriaUGWIocjCgU8JQth19dEFvC8lSOwlp1ALOVnZltOicA1RkA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
  
