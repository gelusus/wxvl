#  DVWA 靶场实战：四级安全等级下的 Web 漏洞与防御演进  
原创 不懂安全的运维
                        不懂安全的运维  运维安全入门   2026-10-10 10:30  
  
上一章我们用 SQLi-Labs 把 SQL 注入彻底拆开看了。但现实中，一个 Web 系统的漏洞从来不是孤立存在的——它面前有登录校验、有输入过滤、有 WAF，身后还有 CSRF 令牌、文件上传、会话管理。  
  
这一章介绍 **DVWA（Damn Vulnerable Web Application）**  
。它最大的特色不是漏洞多，而是**提供了低、中、高、Impossible 四档安全等级**  
——这正是它区别于普通靶场的地方：同一个漏洞，你可以同时看到「裸奔」和「加固后」两种形态，把防御机制的作用量化在眼前。  
  
**🛡️ 授权声明**  
　本文所有演示均在本地 Docker 环境中完成。对生产系统进行安全测试，必须先取得书面授权并约定测试窗口。  
  
本文目录  
1. DVWA 是什么，四档安全等级才是灵魂  
  
1. 靶场架构与模块清单  
  
1. Docker 部署：从 0 到可访问  
  
1. 源码部署与 MariaDB 初始化  
  
1. 核心机制：难度等级是如何生效的  
  
1. 实战一：Brute Force 暴力破解与真实加固  
  
1. 实战二：SQL Injection 与 CSRF Token 的攻防  
  
1. 实战三：File Inclusion 本地/远程包含  
  
1. 实战四：File Upload 文件上传绕过  
  
1. 实战五：Weak Session 与 JWT 会话安全  
  
1. 常见错误与排错清单  
  
1. 等保视角：DVWA 各模块的合规映射  
  
1. 总结  
  
## 01DVWA 是什么，四档安全等级才是灵魂  
  
DVWA 由 Ethical Hackers 社区维护，在 GitHub 上是 digininja/DVWA  
。官方定位是：「一个故意设计为脆弱的 PHP/MySQL Web 应用，用于练习和演示 Web 应用安全课程」  
。  
  
和其他靶场相比，DVWA 有一个决定性的设计：**安全等级（Security Level）**  
。它位于页面左下角的 DVWA Security  
 菜单，包含四档：  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">等级</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">名称</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">防护状态</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">适合阶段</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">低</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">几乎无防护，直接拼接 SQL/系统命令</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">理解漏洞原理</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Medium</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">中</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">增加部分参数处理与简单过滤</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">学习基础绕过</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">High</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">高</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">严格过滤 / 类型约束 / 命令白名单</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">理解有效防御</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Impossible</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">不可能</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">参数化查询、Prepared Statement</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">理解根治方案</span></section></td></tr></tbody></table>  
**💡 提示****为什么这个设计如此重要？**  
因为大多数教程只教你「怎么打」，不告诉你「为什么打不动了」。在 DVWA 里你可以亲手验证：把等级切到 High，同样一个注入 payload 会直接失效；切到 Impossible，源码里会明确写出 prepare()  
 和参数化写法。**这才是完整的知识闭环。**  
  
**界面说明**  
 DVWA 左侧栏底部是 Security 等级下拉菜单，四个档位分别是 Low、Medium、High、Impossible，可随时切换。建议按 Low → Medium → Impossible 的顺序把同一道题做三遍，直观感受过滤规则如何逐级收紧、防护代码如何从「完全不校验」演进到「参数化查询」，这是理解等保 2.0 中输入验证与代码审计要求的最快路径。  
## 02靶场架构与模块清单  
  
DVWA 的代码结构非常清晰，每个漏洞模块是一个独立目录，每个难度等级对应一个独立源码文件。例如 vulns/  
 目录下：  
  
同一漏洞的四个等级各有一份源码，可直接对照阅读  
```
```  
  
这意味着你可以**一边打，一边看源码**  
，这是理解「防御到底做了什么」最高效的方式。建议通关顺序：**先在 Low 打完所有模块，再回到每个模块看 Medium/High/Impossible 的源码差异**  
。  
### DVWA 完整模块清单  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">模块</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">漏洞类型</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">OWASP 对应</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">难度分布</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Brute Force</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">弱口令 / 认证缺陷</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A07 身份识别失效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Command Injection</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">OS 命令注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CSRF</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">跨站请求伪造</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A01 访问控制失效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">File Inclusion</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">本地/远程文件包含</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">File Upload</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">任意文件上传</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A04 不安全设计</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Weak Passwords</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">弱口令生成</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A07 身份识别失效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">SQL Injection</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">SQL 注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">SQL Blind</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">SQL 盲注</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Weak Session</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">会话固定/劫持</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A07 身份识别失效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">XSS DOM</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">DOM 型跨站脚本</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">XSS Reflected</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">反射型 XSS</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">XSS Stored</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">存储型 XSS</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A03 代码注入</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">JavaScript</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">客户端校验绕过</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A04 不安全设计</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Open Redirect</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">开放重定向</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A01 访问控制失效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CSP Bypass</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">内容安全策略绕过</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">A05 安全配置错误</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low/Medium/High</span></section></td></tr></tbody></table>  
**💡 提示**  
　注意 Impossible  
 等级并不覆盖所有模块——有些漏洞（如部分 XSS）本身就没有「完全不可能」的实现，DVWA 会在该等级下直接禁用相应功能。这本身也是一种工程现实的表达。  
## 03Docker 部署：从 0 到可访问  
  
DVWA 官方在 2021 年后已把 Docker Compose 作为主要推荐方式，并且**默认端口从 80 改成了 4280**  
。很多老教程还写着 http://localhost/  
，那是过时的。  
### 方式一：官方 docker compose（推荐）  
  
端口 4280 是官方当前默认值，不是 80  
```
```  
### 方式二：指定配置文件启动  
  
DVWA 仓库提供多份 compose 文件：docker-compose.yml  
（常规）、docker-compose.test.yml  
（测试环境）。常规部署还支持通过环境变量初始化数据库口令：  
  
环境变量优于命令行明文传参，避免密码进入 shell 历史  
```
```  
### 启动后必做的初始化  
  
第一次访问 http://localhost:4280  
，页面下方会出现红色的 Setup / Reset Database  
 链接，**必须点击它**  
，DVWA 才会创建数据库表和初始数据。这一步经常被忽略，导致后面所有模块都报「Table doesn't exist」。  
  
**⚠️ 注意**  
　如果点击 Setup 后仍报错，检查日志：docker compose logs -f dvwa  
，常见原因是 MariaDB 尚未完全初始化就执行了建表脚本，这时**等 30 秒后重新点击 Setup**  
即可。  
  
常用运维命令  
```
```  
  
**⚠️ 注意**  
　依旧建议 compose 中把端口写成 "127.0.0.1:4280:80"  
，只监听本机回环地址。**含高危漏洞的服务绝不应暴露在可路由网络**  
——这是靶场使用中最重要的一条安全纪律。  
  
**界面说明**  
 DVWA 首页把所有练习按漏洞类型平铺成导航：Brute Force、Command Injection、CSRF、File Inclusion、File Upload、SQL Injection、Weak Session、Insecure CAPTCHA 等。每个模块内部再按 GET / POST / Session 拆成若干子题，点击条目进入对应关卡页面，右上角有 Submit 提交框用于验证攻击是否成功。  
## 04源码部署与 MariaDB 初始化  
  
如果你要改源码、或者需要长期保存练习进度，走源码部署。  
```
```  
### 配置文件说明  
  
DVWA 的核心配置在 config/config.inc.php.dist  
，首次运行需复制为 config.inc.php  
：  
```
```  
  
**⚠️ 注意**  
　官方明确提示：**不要把 DVWA 部署在有其他真实服务的机器上**  
，也不要用高权限数据库账号。DVWA 的配置文件里数据库口令是明文存储的，任何能读到该文件的人都能拿到数据库权限。  
## 05核心机制：难度等级是如何生效的  
  
这是 DVWA 最值得讲清楚的一个机制。等级切换**不是**  
简单地在前端加个遮罩，而是**服务端根据配置决定加载哪个 PHP 文件**  
。  
  
等级通过 GET 参数 Security 传入，服务端动态 include 对应文件  
```
```  
  
看懂这段代码，你就理解了 DVWA 的设计精髓：**同一个业务逻辑，四份完全不同的实现**  
。攻击者的每一步尝试，都可以立刻在源码里找到对应。  
### Impossible 等级的源码才是精华  
  
以 SQL 注入模块为例，Impossible 版本是这样的：  
  
Impossible = 参数化查询 + CSRF Token + POST 方法限制，三重防护  
```
```  
  
**💡 提示**  
　请注意最后一行：$data->execute([$user, $pass])  
。参数化查询的精髓在于——**SQL 语句的结构在执行前就已被数据库解析并锁定**  
，用户输入只是作为纯数据填进去，永远不可能改变语法结构  
。这就是为什么它对所有注入手法（包括你还没学到的那些）都天然免疫。  
  
**💡 提示**  
　CSRF Token 的作用也要讲清楚：Security=Impossible  
 时，页面会生成一个随机会话令牌藏在表单里，提交时必须原样带回。攻击者即使能构造出合法请求，**也无法预测这个令牌**  
，因为它与受害者会话绑定且一次一换。  
## 06实战一：Brute Force 暴力破解与真实加固  
  
Brute Force 模块是一个没有验证码、没有锁定、没有限速的登录框。这在现实中早就被禁用了——但正是这种「裸奔」状态，让你看清了防护机制各自在防什么。  
### Low 等级：完全裸奔  
  
无验证码、无锁定、无延迟、用户名字段可注入  
```
```  
  
**⚠️ 注意**  
　Low 级的用户名参数同样可注入。一个 3 行的脚本就能判断用户是否存在：admin' AND SLEEP(3)--+  
，这说明**「用户名枚举」在没有任何防护时是零成本的**  
。  
### Medium 等级：加了锁定与延迟  
  
Medium 的防护：会话级锁定 + 服务端强制延迟  
```
```  
### High 等级：换了一套思路  
  
High：无论成功与否都延迟，消除「响应时间差异」这一侧信道  
```
```  
  
**⚠️ 注意****Low/Medium/High 的一个本质缺陷是：**  
锁定和延迟都只作用于当前会话  
。攻击者只要**每次换一个会话（比如清除 Cookie）就能无限重试**  
。真正的防护必须做到「跨会话、以账号为维度」的锁定——这正是 Impossible 级采用 CSRF Token 的原因：令牌与服务端会话绑定，攻击者无法自行生成。  
  
**界面说明**  
 DVWA 不做自动判题，每道题都要你自行取证。最稳妥的凭证有三种：提交框里回显出预期结果（如 SQLi 里拿到数据库版本号）、页面出现 flag 字符串、或弹窗 / 跳转出现预期页面。建议每关把终端命令与页面结果对照记录，形成可复查的通关台账，这也是等保测评里「漏洞验证需留存证据」的直接对应。  
## 07实战二：SQL Injection 与 CSRF Token 的攻防  
  
SQL 注入模块的设计很巧妙：**登录成功后服务端会告诉你注入是否成功**  
，这省去了手工判断回显的麻烦。  
### Low 等级：经典字符型注入  
  
注意 AND/OR 的运算优先级，这是 SQL 注入的必考点  
```
```  
  
**💡 提示**  
　很多教程让你写 admin'--   
，但那样在部分版本会因密码字段校验而失败。理解优先级比死记 payload 重要：**OR 的结合性来自「左到右」的隐式优先级，实际应显式加括号避免歧义**  
。  
### Medium 等级：数值型 + 换行绕过  
  
Medium 把 id 改为数值型，且对若干关键字做了简单过滤  
```
```  
### High 等级：换一个注入点  
  
High 级别的 payload 需在正确参数位置构造，括号可能来自前端表单  
```
```  
### Impossible 等级：攻击彻底失效  
  
当你把等级切到 Impossible，**无论你输入什么，注入都不可能成立**  
——因为语句结构在编译期就固定了。此时建议你做一件事：**逐行读一遍 index.impossible.php，把这套写法当成你未来代码评审的模板**  
。  
  
**💡 提示**  
　对照记忆 DVWA 四级 SQL 注入的防御演进：Low 无防御 → Medium 简单过滤 → High 类型约束 + 复杂语法 → Impossible 参数化查询。**只有 Impossible 是结构性的，前三级都是概率性的。**  
## 08实战三：File Inclusion 本地/远程包含  
  
文件包含漏洞的本质是：**程序把用户输入直接当作文件路径交给了 include**  
。攻击者由此获得两类能力：读本地文件（LFI）、或让服务器请求一个外部地址（RFI）。  
  
遍历目录跳到系统敏感文件，证明任意文件读取  
```
```  
  
在 DVWA 中更实用的用法是读取**自身源码**  
，从而绕开黑盒分析的局限：  
  
php://filter 是读 PHP 源码的经典技巧  
```
```  
  
**⚠️ 注意**  
　这个技巧在实战中非常有用：**当目标是黑盒、无法看到源码时，LFI + php://filter 相当于一次源码泄露**  
。如果企业系统里还存在 LFI，应视为高危漏洞优先处置。  
### Medium：加了路径前缀过滤  
  
字符串过滤的经典脆弱点：过滤不完整导致可绕过  
```
```  
### Impossible：用白名单而不是黑名单  
  
白名单校验：从「禁止什么」转为「只允许什么」，这是正确的防御范式  
```
```  
  
**⚠️ 注意**  
　这里有一个必须强调的工程观点：**DVWA Medium 的过滤方式（str_replace 去掉 '../'）看起来能挡，但实际是能被绕过的**  
——典型的「黑名单思维陷阱」。安全评审中如果看到 str_replace  
 出现在路径处理里，基本可以直接判定为高风险写法。  
## 09实战四：File Upload 文件上传绕过  
  
文件上传漏洞是等保检查中的高频失分项——因为它可能直接导致**远程代码执行**  
。DVWA 把四级防御讲得很清楚。  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">等级</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">防护手段</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">可能的绕过思路</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">风险评估</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Low</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">无任何校验</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">直接上传 .php 木马</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">RCE</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Medium</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">限制扩展名白名单</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">尝试大小写/双扩展/绕过过滤</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">RCE</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">High</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">后缀白名单 + 图片二次验证</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">针对验证逻辑的对抗</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">RCE 可能</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Impossible</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">随机文件名 + 白名单 + 图片校验</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">结构上不可绕过</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">安全</span></section></td></tr></tbody></table>  
**💡 提示**  
　核心认知：**文件上传的防御不能只看扩展名**  
。只看 in_array($ext, ['jpg','png'])  
 是远远不够的，因为攻击者可能通过双扩展名、大小写、NULL 字节、或服务器解析规则的差异达成绕过。DVWA 的 Impossible 级综合了多重校验，这才接近生产可用的水平。  
## 10实战五：Weak Session 与 JWT 会话安全  
  
Weak Session 模块演示的是**会话固定（Session Fixation）**  
：攻击者能指定会话 ID，之后受害者登录时沿用了这个 ID，攻击者即可用同一 ID 访问已登录页面。  
  
根因：登录成功后未调用 session_regenerate_id()  
```
```  
  
**💡 提示**  
　修复方式只有一行，但必须做到：**在身份验证成功的瞬间，重新生成会话 ID**  
（session_regenerate_id(true)  
）。这条规则在等保的身份鉴别要求中，属于必须落实的会话管理措施。  
  
DVWA 的 High 等级还引入了 **JWT（JSON Web Token）**  
 的概念，让你直观看到令牌与 Cookie 的差异。JWT 的关键特征是**自包含且可验证**  
，但也正因如此，一旦密钥泄露或算法配置不当（如允许 alg: none  
），就会出现严重问题。  
  
**💡 提示**  
　JWT 相关的安全检查要点（等保测评常见项）：**① 强制使用强算法（HS256/RS256）并拒绝 none；② 校验签名；③ 设置合理的 exp 过期时间；④ 密钥存放于密钥管理服务而非代码仓库。**  
## 11常见错误与排错清单  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">现象</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">可能原因</span></section></th><th style="border: 1px solid rgb(223, 228, 236);background: rgb(240, 245, 253);padding: 9px 8px;font-size: 13.5px;color: rgb(26, 115, 232);text-align: left;font-weight: bold;"><section><span leaf="">解决方案</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">访问 80 端口打不开</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">使用了旧教程的端口号</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">官方当前默认端口为 4280</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">页面提示 Table doesn&#39;t exist</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">未点击 Setup 初始化</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">点击页面下方 Setup/Reset Database</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Setup 后仍报错</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">数据库未就绪</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">等待 30 秒后重新点击 Setup</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CSRF Token 提示缺失</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">直接构造了 GET 请求</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Impossible 级必须从页面表单发起</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">上传文件后打不开</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">上传目录未配置执行权限</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">确认 upload 目录的 Web 服务器配置</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">中文显示乱码</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">数据库字符集不匹配</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">导库时指定 utf8mb4</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">等级切换后模块消失</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">该等级未实现此模块</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">切换回 Low 继续练习</span></section></td></tr></tbody></table>  
**💡 提示**  
　DVWA 官方要求：**每道题都需要提交攻击成功后的页面截图作为通关凭证**  
，截图里必须能看到成功提示。这是它作为「课程靶场」的设计初衷——强制你完成完整链路，而不是只看 payload。  
## 12等保视角：DVWA 各模块的合规映射  
  
DVWA 的模块设计其实是一份现成的等保整改对照表。下面把每个漏洞模块映射到等保 2.0 的控制项与落地动作。  
### 一、开发侧：必须从架构上根治  
- **代码注入（SQLi / Command Injection / File Inclusion）→ A03**  
：SQL 使用参数化查询；系统命令使用固定白名单 + 转义函数，严禁拼接执行；文件包含只允许白名单文件名。  
  
- **上传与解析（A04 不安全设计）**  
：上传目录与业务目录物理隔离，关闭上传目录的脚本执行权限，文件类型采用白名单 + 内容二次校验。  
  
- **身份与会话（A07）**  
：口令加盐哈希（如 Argon2id/bcrypt），登录后重置会话 ID，令牌设置短有效期。  
  
### 二、运维侧：纵深防御与可检测性  
- **Web 层防护**  
：部署 WAF 并开启 SQL 注入、XSS、文件上传规则库；但要清楚 WAF 是**补充**  
而非替代，参数化查询做不到的事 WAF 也做不到。  
  
- **进程最小权限**  
：Web 服务以非 root 运行，上传目录不可写、上传文件不可执行，遵循最小权限原则。  
  
- **安全审计**  
：对登录失败、异常上传、大量 4xx 请求建立告警。DVWA 的暴力破解模块可用作检测规则的验证样本。  
  
### 三、测评关注点：容易被忽略的三个细节  
- **上传目录的执行权限**  
：这是能否直接 RCE 的分水岭，测评时务必实测上传一个 webshell 验证。  
  
- **错误信息泄露**  
：生产环境不应回显数据库报错，应统一错误页，避免泄露表名、路径、框架版本。  
  
- **JWT 密钥管理**  
：密钥不得硬编码在代码或配置库中，应使用密钥管理服务，并定期轮换。  
  
**💡 提示**  
　把 DVWA 当成企业内网的一台「靶标机器」来用，是性价比最高的合规自查方式：让内部安全团队在授权范围内对其发起渗透测试，用真实的攻击路径验证现有防护是否有效，而不是只对着配置文件打勾。  
## 13总结  
  
DVWA 的价值在于它展示了一个安全领域的重要事实：**同一个漏洞，在不同防护等级下的攻击难度可以相差数个数量级，而真正的分水岭只有一个——代码是「拼接」还是「绑定」。**  
  
Low 到 Medium 的跨越靠的是补丁式的过滤，Medium 到 Impossible 的跨越靠的是架构重写。这个规律不止适用于 SQL 注入——**几乎所有注入类漏洞都是同一个道理：代码与数据有没有分离**  
。带着这个视角去看后面的靶场，你会发现防御手段千变万化，但本质高度一致。  
  
**💡 提示**  
　下一章我们进入 Juice Shop——如果说 DVWA 教你「漏洞怎么防」，那 Juice Shop 就是要告诉你：**在一个真实的前后端分离架构里，业务逻辑漏洞能有多刁钻**  
。  
> 免责声明：  
> 本文所述靶场均部署于本地或已获授权的测试环境，所有操作仅用于安全学习、漏洞原理验证与合规性自查。严禁在未获书面授权的任何生产系统或公网资产上实施相关操作，否则将承担法律责任。  
  
  
