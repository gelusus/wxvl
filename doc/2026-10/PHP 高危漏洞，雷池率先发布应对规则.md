#  PHP 高危漏洞，雷池率先发布应对规则  
 雷池SafeLine   2026-10-23 03:28  
  
近日，PHP Core 被披露存在 CVE-2026-109618、CVE-2026-109619 两项高危安全漏洞。面向 PHP 网站、业务后台、CMS、商城与 API 服务，建议尽快完成版本核查与安全排查。  
  
在漏洞风险扩散后，**雷池团队率先完成已知利用链和 Web 攻击特征分析，并于 10 月 21 日发布专项规则**  
，为用户提供可直接落地的攻击观察与防护方案。  
## 一、漏洞风险概览  
  
本次漏洞位于 PHP Core，影响的是 PHP 运行时本身，而非某一个 CMS、框架或插件。对于仍在运行 PHP 服务的团队，尤其需要关注文件上传、表单提交、登录与 Session 处理等常见 Web 场景。  
### CVE-2026-109618  
  
该漏洞与 multipart/form-data  
 请求解析及内存状态处理有关。攻击者可能围绕异常上传请求进行探测，导致服务资源消耗或运行时异常。  
### CVE-2026-109619  
  
该漏洞与 PHP Worker 的全局状态管理有关。在特定条件下，异常状态可能跨请求残留，带来内存安全风险。  
  
对多站点、多环境或多个 PHP 版本并行运行的服务器，建议逐一确认每个业务实际运行的 PHP 版本，避免出现仅升级部分服务的遗漏。  
## 二、雷池率先完成分析并发布专项应对方案  
  
在本轮 PHP 高危漏洞公开后，**雷池团队率先完成已知利用链及 HTTP 攻击特征分析，并于 10 月 21 日发布 PHP Core RCE CVE-2026-109618 / 109619 专项规则。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/o2rSpTXlXYmCictCjBdnKMfHGyUeWKX77PlqiaS5ria9nJ3WMT5pY7QxehJDoRjzBFcibaaadcxsXPqYref0Mk5SsO5IdLmibZictpheTrcSVKKqg/640?wx_fmt=png&from=appmsg "")  
  
图 1　雷池规则广场公告页。页面显示该专项规则“发布于 2 天前”，并于发布后持续更新规则说明。  
  
规则聚焦可在 Web 请求侧识别的已知攻击特征，包括异常的文件上传请求、PHP_SESSION_UPLOAD_PROGRESS  
 参数、符合特征的 PHPSESSID  
，以及请求体中异常的表单字段组合。  
  
用户可在雷池中将规则配置为黑名单，并启用**观察模式**  
。系统发现疑似攻击后，将记录攻击事件、攻击来源和相关请求信息，便于先进行业务确认与安全排查。  
### 社区反馈：规则已启用并持续产生命中  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o2rSpTXlXYkgSvGrhfa8xIjj1hjyM7O85AnX7QDL9y2lGdXcN6nd3lDBrLG3gJeXyZXckhqzMam6xI2aoKTv0m0SByv2egJWicATs0HIwAmM/640?wx_fmt=png&from=appmsg "")  
  
图 2　社区用户已启用 PHP Core RCE 专项规则，界面显示当日命中情况。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o2rSpTXlXYkUZ6NzWBNMuZGHN6K4Zeic5dYUux1RQiciakIyalsXcCPHhibJpfKencIJ8U5ic3lNIRibAQWqianQszs1adX5MoHh99ne2snrQGIw9I/640?wx_fmt=png&from=appmsg "")  
  
图 3　社区反馈显示，该规则已作为“雷池官方缓解措施”启用，命中数量持续更新。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o2rSpTXlXYlKstEVOuYpWGic5KXNp8Lmsria96xlzxfPSibvI0gnWyyIGgezR3uALbbnKUicqae3b8C6xRqskTkiciaSqw2ibKfmXWcowvDb97wk84/640?wx_fmt=png&from=appmsg "")  
  
图 4　规则创建记录：2026 年 10 月 22 日 18 时 00 分 49 秒。  
### 内部验证：规则成功识别并拦截模拟攻击请求  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o2rSpTXlXYk4luiaY77uf1XkMMXVqtUvr3LbeFq6L0DNoV3qvIIchZyNYlhynJwiaft1r7stlVkprwkAZXJwqKCHTfyDzDojabMcTZr1wuSibA/640?wx_fmt=png&from=appmsg "")  
  
图 5　内部模拟验证中，雷池命中 PHP Core RCE 专项规则并阻断攻击请求。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/o2rSpTXlXYk7k8h4ObiakVBEib6zen6khGusfxIPIGTqMUdOFtGlzxsCf3j6VqdcTOfvV4F1B7UNXgPzt7gPH3FTINHRsjSkibSU1OzGMOm0XQ/640?wx_fmt=png&from=appmsg "")  
  
图 6　内部连续测试记录显示，多个模拟请求均被专项规则拦截。  
  
**规则详情与配置入口**  
https://rivers.chaitin.cn/discussion/dbc7qkj1s5rlia466a30[]()  
#规则库说明  
## 三、建议立即完成的安全检查  
1. **核对实际运行版本。**  
不要只查看服务器已安装的 PHP 版本。请确认每个站点、每个 PHP-FPM 服务实际使用的版本，并建立已升级、待升级与无法升级事项清单。  
  
1. **及时完成补丁升级和业务验证。**  
升级前备份网站文件、数据库与关键配置；升级后重启对应服务，并验证首页访问、登录、上传、表单提交等核心业务功能，同时查看 PHP 错误日志。  
  
1. **在雷池中启用专项规则。**  
建议先使用观察模式，重点关注上传、登录、表单接口是否存在异常高频访问，是否出现可疑 multipart 上传请求，或同一来源持续变换请求内容进行探测。  
  
1. **排查源站绕过入口。**  
确认业务流量均经过雷池防护，并检查源站 IP 是否暴露、管理后台与测试环境是否意外开放，以及安全组和防火墙是否仅放行必要来源。  
  
1. **发现异常后扩大排查范围。**  
如发现规则命中或异常访问，应同步检查 Web 访问日志、PHP 错误日志、网站目录与上传目录中的异常文件、可疑计划任务、异常进程和外连行为。  
  
## 及时响应，让风险止步于攻击入口  
  
面对高危漏洞，越早完成分析与处置，越能降低业务暴露时间。雷池已率先针对本次 PHP 漏洞发布专项应对规则，建议相关用户尽快完成版本核查、规则配置和业务排查。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o2rSpTXlXYm9kVfftSMw9YrGFgNakjgqPzCe30dfcibmfHgZUEfUYIpXBdJrEq12bP4rSVxpvooOBadQtVG8w9gYIvDxCicZBnRY33Ke5KKVU/640?wx_fmt=png&from=appmsg "")  
  
扫码加入雷池社区版交流群  
  
申请雷池试用，请点击文末「阅读原文」。  
  
  
  
