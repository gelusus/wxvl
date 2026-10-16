#  Next.js 曝出严重漏洞：可导致未授权远程代码执行 (RCE)  
 网安百色   2026-10-16 10:42  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJnsWlIWMcV7g59aKJqakBF7cjrBmkeJeU48TZnnt8hmMBRRHOQaVdCfzxAsS5jVfCS8yYuqPiaJCYfOHNL1HmVr7EKiaib9GRAzTgk/640?wx_fmt=png&from=appmsg "")  
  
Next.js 发布了安全更新，修复了两个严重漏洞，攻击者可借此在未授权（无需登录）的情况下远程执行恶意代码。第三个高危漏洞可能导致受影响的应用程序拒绝服务（无法响应请求）。  
  
这些问题涉及缓存函数（cached functions）、SVG 图像处理以及 Server Actions（服务端动作），相关修复已包含在 Next.js 16.4.1 和 15.5.28 版本中。官方于 2026 年 10 月 14 日发布的安全公告详细说明了这些具有不同触发条件的独立攻击链路。  
  
这两个远程代码执行（RCE）漏洞的 CVSS 评分均为 9.5，而拒绝服务（DoS）漏洞的评分为 8.2。由于实际暴露面取决于应用程序的具体配置和启用的功能。  
  
第一个漏洞（编号：GHSA-673q-mjm6-r5xr）影响使用了 Cache Components（缓存组件）的 App Router 应用。当 use cache  
 函数返回多个类型化数组（typed arrays）时，应用即处于易受攻击状态。在此条件下，攻击者无需任何身份验证或用户交互，即可实现远程代码执行。  
  
Next.js 16.0.0 至 16.4.0 版本均受此漏洞影响，修复版本为 16.4.1。由于目前尚无临时缓解措施（workaround），且该漏洞暂未分配 CVE 编号，开发者应立即执行升级。  
### Next.js 严重漏洞详情  
  
第二个漏洞（CVE-2026-96889）涉及 Image Optimization API 在处理 SVG 文件时所依赖的上游库中存在的内存安全缺陷。该漏洞可能导致运行在基于 glibc 的 Linux 环境下的 Next.js 部署遭受远程代码执行。公告指出，利用此漏洞无需任何特权或用户交互。  
  
官方图像优化安全公告指出，受影响的版本范围为 10.0.0 至 15.5.27，以及 16.0.0 至 16.4.0。  
  
开发者可通过确保 Next.js 使用 sharp  
 0.35.5 或更高版本来修复此问题。直接安装已打补丁的 Next.js 版本，可确保包管理器自动拉取并绑定修复后的依赖项。  
  
第三个漏洞（编号：GHSA-p3m5-59mp-jv5r）影响包含至少一个 Server Action 的 App Router 应用。攻击者发送的精心构造的请求会导致 CPU 资源被过度消耗，从而阻止同一进程处理后续请求（即拒绝服务）。使用 Pages Router 或未启用 Server Actions 的应用不受此漏洞影响。  
  
受影响的版本包括 13.0.0 至 15.5.27，以及 16.0.0 至 16.4.0。除升级外，目前暂无其他临时缓解措施。尽管此前关于 Next.js 安全补丁的报道曾提及过类似的功能级风险，但那些早期的修复并不能替代本次更新。  
  
开发者应根据实际情况，将 Next.js 升级至 15.5.28 或 16.4.1 版本。请注意，仅更新 sharp  
 依赖只能修复图像处理漏洞，无法解决独立的缓存函数或 Server Actions 漏洞。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
