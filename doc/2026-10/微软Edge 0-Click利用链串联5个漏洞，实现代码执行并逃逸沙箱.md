#  微软Edge 0-Click利用链串联5个漏洞，实现代码执行并逃逸沙箱  
 FreeBuf   2026-10-30 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX1ibsMeIrg5NRJtsSKjVu5ZTBuSBrezxqHDR3NFN7tVhiaol3Psw07RxVd4ne5uYhEIncx5JbdwepxAH7vCMnvTs9ibialnbDYSRBE/640?wx_fmt=gif "")  
  
  
![Edge 0-Click利用链](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX2aW8sJWB7K5wibcLwCicy5iacuDzroCW8Mfd9dtdvKnULiaXiaJJrsPET02GPSTYYAJHrRPgq2SGAgiabesxbZQ77hrgm9I6slPvqLo/640?wx_fmt=png "")  
  
  
Part  
01  
  
Edge导航缺陷导致跨站脚本执行  
  
  
研究人员披露了一条全新的微软Edge漏洞利用链，该利用链组合5个漏洞与两项Windows原生功能，可突破浏览器沙箱限制在外部执行代码。  
  
  
安全研究员Orange Tsai于2026年10月28日发布相关研究，展示了攻击者如何利用浏览器逻辑缺陷，无需内存破坏漏洞，仅通过恶意网页即可在Windows系统上执行代码。  
  
  
Orange Tsai是DEVCORE团队成员，他为参加2026年柏林Pwn2Own大赛构建了这条利用链。根据其发布的技术文档，整个研究耗时两个月，全程未使用AI工具，最终成型的漏洞利用代码不到100行。  
  
  
这条利用链的起点是Edge的条件访问导航处理模块，该模块的作用是在用户访问微软服务时，帮助切换至对应的浏览器配置文件。当用户提交的邮箱与当前配置文件匹配时，该处理模块会根据提交的目标地址发起新的导航请求。  
  
  
该模块存在一个关键缺陷：没有将导航请求与原始标签页绑定。Chromium内核会直接调用当前活动标签页处理请求，导致原本仅应在特定上下文运行的JavaScript可以在其他网站上执行，形成通用跨站脚本（UXSS），打破了不同网站之间原本的隔离边界。  
  
  
Part  
02  
  
多重漏洞组合实现0-Click攻击  
  
  
由于这类导航请求由浏览器进程发起，还可绕过内容安全策略（CSP）防护。不过要实现完全自动化的攻击，仍需绕过剩余的几道检查机制。  
  
  
第一道检查是Edge会验证导航请求是否来自受信任的微软站点，但没有考虑到其他窗口控制该导航的场景，研究者通过跨窗口流程绕过了这一检查。此外，代码中存在硬编码的弹窗例外规则，无需用户常规Click即可打开新窗口。  
  
  
该利用链还需要获取已登录的Edge配置文件及其对应的邮箱地址。在测试环境中，当用户访问微软网站时，微软的OneAuth功能会调用已有的微软账户自动创建并登录配置文件。  
  
  
而assets.msn.com存在的跨域资源共享（CORS）配置错误，会将已登录账户的邮箱地址泄露给第三方网站。上述步骤组合后，用户加载攻击者控制的内容后，无需进行任何额外Clicks即可完成利用。需要说明的是，这里的zero-click指整个利用链自动化执行，并非指用户完全不需要接触恶意网页内容即可触发攻击。  
  
  
![利用链技术细节](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1gkPPq2ns1ZgC6QHoobChsCYg7gZj5mGyYWVXavmI76UtVSh8qN84DqXibBZicqs9ZxM4Wfez89CIz2LB5YUy2vqYiaouDqxOK5s/640?wx_fmt=png "")  
  
  
Part  
03  
  
利用Windows原生功能  
  
突破浏览器沙箱  
  
  
仅实现跨网站执行JavaScript，还不足以在Windows系统上执行代码。研究者接下来将目标对准Edge的沉浸式阅读器功能，但浏览器特权页面的JavaScript URL拦截列表并未收录该功能使用的read://地址协议。  
  
  
这一缺陷暴露了一个用于打包诊断日志的原生反馈功能。研究者借助路径遍历与空字节处理缺陷，首先突破了路径限制，可以将文件写入指定临时文件夹之外的位置。同时他还绕过了系统强制的JSON文件扩展名校验，实现任意文件写入。Cybersecurity News此前曾报道过相关的Edge反馈日志代码执行漏洞。  
  
  
随后研究者修改了Edge的配置文件protocols.json——该文件负责控制哪些网站可以启动外部协议处理程序。修改后，网页即可调用Windows的telnet处理程序。研究者将一个小型可执行文件放入Windows会搜索的可写目录中，最终实现了在Edge沙箱之外执行代码。  
  
  
与此前单独披露的Edge内存破坏漏洞不同，这条利用链无需利用内存破坏问题，完全依靠逻辑缺陷与受信任功能的组合完成利用。  
  
  
这一案例表明，浏览器安全审计不能仅单独验证每个功能的安全性，必须覆盖导航逻辑、身份认证、特权页面、Windows系统集成等各模块的交互场景。用户应及时安装微软发布的Edge安全更新，不要将浏览器沙箱隔离视为能够抵御所有攻击的完备防护。  
  
  
参考来源：  
  
Microsoft Edge Zero-Click Exploit Chains Five Flaws to Execute Code and Escape Sandbox  
  
https://cybersecuritynews.com/microsoft-edge-zero-click-exploit-chain/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
