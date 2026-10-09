#  WordPress 严重漏洞可引发跨站脚本 (XSS)、SQL 注入及信息泄露攻击  
 网安百色   2026-10-09 10:12  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJntCclQv6uuV9H0UVv7oT3JVhOyzqs8KkZ1SibBib9ic9ErhJeib8XdqCIH20alGgAjJ7Eaha0UMycoNAY7EUxDZG7SBicrNlyia4MMGQ/640?wx_fmt=png&from=appmsg "")  
  
2026年10月6日，WordPress 发布了 7.1.3 版本，修复了涉及跨站脚本（XSS）、SQL 注入、信息泄露及其他安全缺陷的多个漏洞。  
  
官方强烈建议用户立即更新，受影响的旧版本分支也已同步提供修复补丁。目前，WordPress 仅对最新版本提供持续的安全维护与支持。  
  
尽管发布文档的引言仅提及“一项安全修复”，但实际共列出了七个安全问题。公告中未提供 CVE 编号、CVSS 严重程度评分，也未披露任何在野利用（in-the-wild exploitation）的细节。因此，不应将标题中的警示性措辞直接等同于官方对每个漏洞的严重程度定级。  
  
其中一个漏洞存在于评论管理页面，属于存储型跨站脚本（Stored XSS）攻击。待审核的评论构成了主要的攻击面，管理员在审查用户提交的内容时极易触发该漏洞。  
  
该问题由 Trail of Bits 的 Thomas Chauchefoin 报告。目前的披露信息并未详细说明恶意载荷（payload）的具体构造，也未明确成功利用所需的前置条件。  
  
另一个独立的跨站脚本漏洞影响 Imgur 媒体嵌入功能。WordPress 官方对此向报告者 Zhengyu Liu、Jingcheng Yang 和 Gavin Zhong 表达了致谢。  
  
这两项发现涉及不同的内容处理链路：评论审核与外部媒体嵌入。对于任何处理用户生成内容（UGC）或展示外部来源材料的网站而言，此次更新尤为关键。  
### WordPress 漏洞详情  
  
Anthropic 报告了 WordPress WXR 导出功能中存在二阶 SQL 注入（Second-order SQL Injection）漏洞。发布说明中指出了受影响的导出组件，但未阐明具体的攻击链路、所需权限或对数据库的潜在影响。  
  
WordPress 安全团队的 Alex Concha 报告了一处参数伪造漏洞：传递至 {status}_{type}  
 钩子（hook）的参数可被伪造，进而引发操作名称冲突（action name collisions）。公告未具体说明该问题在更深层次利用场景下的潜在后果。  
  
一处拒绝服务（DoS）漏洞影响了 WP_Http::make_absolute_url()  
 方法。该发现同样由 Anthropic 报告，但 WordPress 在发布说明中未提供具体的请求特征、资源消耗指标或详细的攻击前置条件。  
  
管理员可通过后台“仪表盘 > 更新”安装 WordPress 7.1.3，或从官方发布页面获取该版本。鉴于这是一次安全更新，WordPress 明确建议立即对站点进行升级。受影响的旧版本分支虽会获得这些修复，但这并不会改变该项目现行的版本支持策略。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
