#  Juice Shop 靶场实战：现代全栈应用中的业务逻辑漏洞攻防  
原创 不懂安全的运维
                        不懂安全的运维  运维安全入门   2026-10-12 12:10  
  
前两章我们打的靶场，本质上都是「演示型」漏洞：SQL 拼接、命令拼接、缺少校验。这些漏洞的共同点是——**成因清晰、检测容易、修复路径唯一**  
。  
  
但现实世界里最难防的，恰恰是**业务逻辑漏洞**  
。它们不存在「不安全的函数」，只存在于「设计时的疏忽」：购物车里的价格由前端传参、优惠券可以重复叠加、支付回调没有验签……这类漏洞用任何扫描器都扫不出来。  
  
而 **OWASP Juice Shop**  
 正是为此而生。它被称为「有史以来最现代、最不安全的 Web 应用」——不是技术栈老旧，而是因为它**刻意实现了大量真实世界中的业务逻辑缺陷**  
，并且用现代前端技术（Angular + Express + 完整 REST API）把它们串成一条完整业务链。  
  
**🛡️ 授权声明**  
　Juice Shop 官方提供了自评应用（Self-Assessment），会告诉你每个挑战的解题思路，非常适合自学。但建议第一次挑战时**先不看答案独立完成**  
，否则容易失去靶场训练的意义。  
  
本文目录  
1. Juice Shop 是什么，为什么它比想象中难  
  
1. 技术架构：现代前后端分离是怎么搭的  
  
1. Docker 部署与本地访问  
  
1. 本地部署：Node.js 环境与多语言切换  
  
1. 核心机制：挑战评分与隐藏彩蛋  
  
1. 实战一：SQL 注入拿下登录态  
  
1. 实战二：业务逻辑漏洞——购物车价格篡改  
  
1. 实战三：访问控制缺陷与 JWT 密钥爆破  
  
1. 实战四：XSS 与 DOM 型漏洞实战  
  
1. 实战五：加密货币与支付绕过  
  
1. 常见问题与排错清单  
  
1. 等保视角：业务逻辑漏洞为何最难测  
  
1. 总结  
  
  
  
## 01Juice Shop 是什么，为什么它比想象中难  
  
Juice Shop 由 OWASP 基金会官方维护，是 OWASP 基金会下最受欢迎的教学型靶场之一，GitHub 地址为 juice-shop/juice-shop  
。它的自我定位很直白：「Probably the most modern and sophisticated insecure web application」  
。  
  
很多初学者打 Juice Shop 会觉得挫败——明明是 OWASP 官方出品，却比 DVWA 难得多。原因在于**它的漏洞分布和 DVWA 完全不同**  
：  
  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">维度</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">DVWA</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">Juice Shop</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">漏洞类型</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">经典注入与配置缺陷为主</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">业务逻辑漏洞占比很高</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">难度线索</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">明确标注 Low/High</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">无提示，需自行发现</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">通关标准</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">有明确成功提示</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">触发特定行为才算完成</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">架构形态</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">传统 PHP 单体</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Angular + Express 前后端分离</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">训练目标</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">理解漏洞原理</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">建立业务安全思维</span></section></td></tr></tbody></table>  
  
**💡 提示**  
　**这个差异恰恰是 Juice Shop 的价值所在。**  
如果你只打 DVWA 这样的靶场，遇到真实业务系统时会发现「所有扫描器都报绿灯，但业务被人随便改一下就出事了」的情况。Juice Shop 专门用来训练这种业务视角的安全思考能力  
。  
  
**界面说明**  
 Juice Shop 首页是一套仿电商界面：顶部导航含搜索框、购物车图标与账户入口，中间是六角形商品网格（手机壳、篮球、瑜伽垫等）。右上角有一个不断跳动的挑战进度条，点击后展开Juice Shop Challenges 抽屉，左侧是按星数排序的挑战列表，右侧是解题状态总览——这套挑战清单就是本靶场的完整攻击面地图。  
  
## 02技术架构：现代前后端分离是怎么搭的  
  
理解架构，才能理解为什么某些攻击在前端「看起来不可能」。Juice Shop 的技术栈如下：  
  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">层次</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">技术</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">说明</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">前端</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Angular</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">单页应用，路由与状态在客户端</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">后端</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Node.js + Express</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">提供 REST API</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">数据库</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">SQLite / PostgreSQL</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">业务数据存储</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">认证</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">JWT（jsonwebtoken）</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">无状态令牌鉴权</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">支付</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">第三方支付模拟</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">含可绕过的回调逻辑</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">加密货币</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">内置虚拟货币模块</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">可被用于搭建后门通道</span></section></td></tr></tbody></table>  
  
这里有个关键点必须讲清楚：**Angular 是纯前端框架，所有页面渲染都在浏览器完成**  
。这意味着源码里看到的「安全校验」大多只是 UI 层的假象——真正的权限判断必须在服务端 API 里。Juice Shop 有大量挑战就是利用了这个「前后端信任错位」：前端以为删掉了按钮，后端却照样接受请求。  
  
**⚠️ 注意**  
　这也是一个重要的防御原则：**任何客户端的校验都只能改善体验，不能作为安全控制**  
。前端隐藏了「删除」按钮不代表用户无法删除；前端限制了价格输入不代表后端不会接受这个价格。所有关键校验必须在服务端重复执行。  
  
## 03Docker 部署与本地访问  
  
官方推荐 Docker 方式，跨平台且无需配置依赖。  
  
首次启动会进行数据库迁移，可能需要 30-60 秒  
  
```
```  
  
### 推荐的 docker compose 方式  
  
NODE_ENV=ctf 是官方推荐配置，会显示挑战与评分信息  
  
```
```  
  
  
**💡 提示**  
　首次访问时数据库需要初始化，**页面可能短暂无响应**  
，属正常现象。可用 docker compose logs -f juice-shop  
 观察进度，看到 Server listening on port 3000  
 即表示就绪。  
  
常用管理命令：  
  
重置练习进度用 down -v  
  
```
```  
  
  
**界面说明**  
 首次 docker compose up 时日志会刷屏，重点盯三行：数据库迁移完成、Express 监听端口（默认 3000，若被占用会自动顺延到 3001）、以及 Server listening on port。日志停在数据库 seed 阶段不动，通常是磁盘空间不足或端口冲突；看到 listening 字样再访问浏览器，过早访问会拿到连接被重置的错误页。  
  
## 04本地部署：Node.js 环境与多语言切换  
  
如果你需要修改源码、研究漏洞原理，或想在多语言环境下练习，可以走本地部署。  
### 环境要求  
  
npm start 会自动完成数据库初始化  
  
```
```  
  
### 常用环境变量  
  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">变量</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">作用</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">示例</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">PORT</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">服务监听端口</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">PORT=3000</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">NODE_ENV</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">运行模式（ctf 开启挑战提示）</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">NODE_ENV=ctf</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CTF_KEY</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">加密挑战的固定密钥</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CTF_KEY=your-key</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CUSTOM_CTF_KEY</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">自定义 CTF 密钥</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">CUSTOM_CTF_KEY=xxx</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">HOST</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">绑定地址</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">HOST=127.0.0.1</span></section></td></tr></tbody></table>  
  
**⚠️ 注意**  
　把 HOST  
 显式设为 127.0.0.1  
 是个好习惯。Juice Shop 在某些启动模式下会监听 0.0.0.0  
，意味着同一局域网内其他设备可能访问到这个满是漏洞的服务。  
### 多语言练习  
  
Juice Shop 支持多语言切换，实际是很好的**安全工程实践演练**  
——同一套逻辑在处理多语言输入时，往往会出现过滤遗漏。  
  
```
```  
  
  
## 05核心机制：挑战评分与隐藏彩蛋  
  
Juice Shop 的挑战系统是它的灵魂，也是最容易被误解的部分。  
### 评分与进度追踪  
  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">元素</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">作用</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">获取方式</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Score Board</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">显示已解决/待解决挑战</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">右上角菜单进入 /#/score-board</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Challenges（面包屑）</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">顶部黄条，逐个完成时出现</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">完成挑战后弹出</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">API 文档</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">完整接口列表与数据结构</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">访问 /ftp/api-docs 或 /restful-api-spec</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">Privacy &amp; Security 页面</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">部分挑战的提示线索</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">/#/privacy-security</span></section></td></tr></tbody></table>  
  
**💡 提示**  
　**REST API 文档是 Juice Shop 最大的宝藏。**  
访问 http://127.0.0.1:3000/ftp/api-docs  
，你会看到 Swagger 格式的完整接口文档，包含每个接口的请求参数与响应结构。很多挑战的答案就藏在其中——尤其是那些前端界面里根本没有对应入口的接口。  
### 隐藏彩蛋  
  
Juice Shop 藏有多个彩蛋页面，通常通过非标准路由访问。找到彩蛋本身也是挑战的一部分，奖励是提示或额外分数。  
  
这些路径在 robots.txt 与源码注释中常能找到线索  
  
```
```  
  
  
**💡 提示**  
　下载 /ftp/package.json  
 并分析依赖版本是很好的习惯——既能发现已知 CVE，也能帮助你理解后端的整体技术结构。这正是真实渗透测试中「资产梳理」阶段会做的事。  
  
**界面说明**  
 访问 /ftp/ 会看到一个目录列表，暴露了 juicy_malware.jar、preferences.json、endorsement_ledger.csv 以及多个 .js.map 文件。这一层是典型的「静态目录未做访问控制」：jar 包可反编译出后端完整逻辑，js.map 能还原前端源码注释，preferences.json 直接给出管理员初始口令。它也是全站最容易被 nmap + gobuster 命中的入口。  
  
## 06实战一：SQL 注入拿下登录态  
  
即使 Juice Shop 以业务逻辑漏洞著称，传统注入依然存在。登录接口就是一个典型的注入点。  
  
登录接口是 NoSQL 场景下 $ne 操作符注入的典型入口  
  
```
```  
  
  
Juice Shop 登录接口在不同版本中可能使用 SQL 或 NoSQL 存储。对于 NoSQL（MongoDB）场景，注入的原理完全不同——利用的是**查询操作符本身**  
而非 SQL 语法：  
  
典型 NoSQL 注入：绕过密码比较，命中第一个用户  
  
```
```  
  
  
**💡 提示**  
　这个案例的教学意义在于：**「不拼 SQL」不等于「就安全」**  
。NoSQL 时代攻击面从 SQL 语法转移到了操作符注入  
，但根因完全一致——**用户输入直接参与了查询结构的构造**  
。无论 SQL 还是 NoSQL，防御原则都是同一个：输入必须是纯粹的数据，不能影响查询结构。  
  
## 07实战二：业务逻辑漏洞——购物车价格篡改  
  
这是 Juice Shop 的招牌挑战，也是最值得反复琢磨的一个。它的难度不在于技术手段，而在于**你必须意识到「价格不该由客户端决定」**  
。  
  
正常流程是：用户把商品加入购物车 → 前端显示价格 → 下单时后端计算总价 → 支付。但在 Juice Shop 里，购物车接口直接接受客户端传来的价格。  
### 分析思路  
  
接口直接接受 totalPrice 字段，这是漏洞所在  
  
```
```  
  
  
把 totalPrice 改为 0，服务端未重新计算，直接入库  
  
```
```  
  
  
利用负数总额抵消，实现实际意义上的「负价购买」  
  
```
```  
  
  
**⚠️ 注意**  
　**这类漏洞的本质是「信任边界划错了」。**  
开发者认为「价格是系统内部数据，用户改不了」，于是把它当成可信输入直接使用。但只要这个值经过客户端，它就是不可信输入——无论它是不是业务数据。  
  
**💡 提示**  
　修复思路也很明确：**服务端必须基于商品 ID 重新查询价格并计算总价，完全忽略客户端提交的任何金额字段**  
。在代码评审中，凡是看到「从前端接收金额/价格/余额」的地方，都应当高度警惕。  
  
**界面说明**  
 把购物车里商品的 totalPrice 用 Burp 改成 0 或负数再放行，结账流程照样走完。根因是服务端信任前端传来的价格字段，没有从商品库重新取价——这正是业务逻辑漏洞与传统注入的本质差异：没有 SQL 语法可注入，但「谁能改、改什么、改完谁负责算钱」这条信任链被绕开了。对应到等保 2.0，就是应用数据完整性校验与权限分离的落地要求。  
  
## 08实战三：访问控制缺陷与 JWT 密钥爆破  
  
Juice Shop 的另一类重要漏洞是**访问控制（Broken Access Control）**  
，这是 OWASP 2021 版 Top 10 的第一位漏洞，现实中发生频率也最高。  
### 越权访问的基本形态  
  
水平越权（改 ID 访问他人）与垂直越权（普通用户访问管理端）  
  
```
```  
  
  
**⚠️ 注意**  
　这里的核心是 **Insecure Direct Object Reference（IDOR，不安全的直接对象引用）**  
。它的成因极其简单：服务端在处理请求时**只用 URL 里的 ID 定位资源，却没有验证「当前登录用户是否有权访问该 ID 对应的资源」**  
。  
  
防御方法只有一条，但必须严格执行：**任何涉及资源访问的接口，都必须从会话/令牌中提取当前用户身份，并与请求的资源归属做服务端校验**  
。前端隐藏按钮不算防御。  
### JWT 密钥爆破  
  
Juice Shop 默认配置下签发 JWT 使用的密钥是**硬编码在源码里**  
的。任何人都能拿到，从而可以伪造任意用户的合法令牌。  
  
JWT 载荷可被任意人读取，这是设计使然  
  
```
```  
  
  
**⚠️ 注意**  
　这里必须建立一个安全认知：**JWT 载荷是 Base64 编码，不是加密**  
。任何拿到令牌的人都能看到里面的所有信息。因此**严禁在 JWT 中放置密码、身份证号等敏感数据**  
——这是很多系统在设计令牌时犯的低级错误。  
  
**💡 提示**  
　更严重的是**硬编码密钥**  
。一旦攻击者知道密钥，就能伪造任意用户的合法令牌，直接绕过所有认证。密钥管理必须使用密钥管理服务（KMS），并纳入轮换机制。  
  
**界面说明**  
 把 token 粘到 jwt.io，payload 段是 Base64 明文，直接暴露 userId、email、role、lastLoginAt 等字段。JWT 的签名只保证「不可篡改」，不保证「不可读」——把敏感数据塞进 payload 等于公开发布。对应的防御是只放不可推断的标识符，角色与权限以服务端查询为准，并在等保测评中作为「传输数据保密性」的关注点。  
  
## 09实战四：XSS 与 DOM 型漏洞实战  
  
Juice Shop 的 XSS 系列挑战覆盖了存储型、反射型和 DOM 型三种形态，其中 DOM 型是最值得学的——因为**它完全发生在客户端，服务端防护完全失效**  
。  
### 反射型与存储型：服务端可过滤  
  
利用存储型 XSS 可窃取其他访问者会话  
  
```
```  
  
### DOM 型：纯客户端，服务端管不着  
  
innerHTML 赋值是 DOM XSS 的温床  
  
```
```  
  
  
**⚠️ 注意**  
　**服务端转义对 DOM XSS 完全无效**  
，因为数据从未发送到服务端。这类漏洞的防御必须靠：① 前端避免使用 innerHTML  
，改用 textContent  
；② 部署严格的 **CSP（内容安全策略）**  
，禁止 inline  
 脚本，从源头阻断。  
  
**💡 提示**  
　Juice Shop 自带一个 CSP Bypass 挑战，很好地说明了**CSP 不是绝对可靠，但仍是性价比极高的纵深防御手段**  
。正确的策略是「严格 CSP + 安全的编码方式」双管齐下，而不是把全部希望寄托在 CSP 上。  
  
## 10实战五：加密货币与支付绕过  
  
Juice Shop 里有一个「加密货币支付」的挑战，它是全文最精彩的**业务逻辑漏洞**  
案例之一。  
  
正常流程：用户选择加密货币支付 → 前端生成一个钱包地址 → 用户向该地址转账 → 后端通过区块链查询接口确认到账 → 订单完成。  
### 漏洞所在：传输层与业务层的混淆  
  
支付回调是业务逻辑漏洞的高发区  
  
```
```  
  
  
**💡 提示**  
　**支付类漏洞的通用防御原则：**  
① 金额必须由服务端根据商品重新计算，忽略客户端任何金额输入；② 支付回调必须**验签**  
（确认请求确实来自支付网关）；③ 必须做**幂等处理**  
，同一笔订单不能重复发货；④ 订单状态机要清晰，防止「未支付」状态被直接置为「已支付」。  
  
顺带一提，Juice Shop 还提供一个**无头浏览器后门**  
相关的挑战。它的教学意义是：**把浏览器自动化工具暴露在服务端并允许外部触发，本身就是一个高危攻击面**  
——攻击者可以借此用服务端的浏览器访问任意地址，探测内网资源。生产环境中任何调试接口、Webhook 测试工具都不应裸露。  
  
![Juice Shop 加密货币支付流程：钱包地址生成与到账确认的接口调用时序](https://mmbiz.qpic.cn/sz_mmbiz_png/0l1UjZtX9twRuz8B3dvNFQiaoMn7TMLB5DQbsMradw4jXsP3rM3sCv5PBEfINsEZIQY2cdcLH9Sia1MeVz0YGfsCsDWOo59jzDrrGibNdDarXU/640?from=appmsg "")  
  
图：Juice Shop 加密货币支付流程：钱包地址生成与到账确认的接口调用时序  
  
## 11常见问题与排错清单  
  
<table><tbody><tr><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">现象</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">可能原因</span></section></th><th style="border: 1px solid rgb(223, 228, 236); background-color: rgb(240, 245, 253); padding: 9px 8px; font-size: 13.5px; color: rgb(26, 115, 232); text-align: left; font-weight: bold;"><section><span leaf="">解决方案</span></section></th></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">首次启动很慢/无响应</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">数据库迁移与依赖安装</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">等待 1-2 分钟，查看容器日志</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">端口 3000 被占用</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">本地已有服务占用</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">改用 -p 127.0.0.1:3100:3000</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">找不到挑战提示</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">未启用 ctf 模式</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">设置 NODE_ENV=ctf 环境变量</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">注册功能报 500</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">邮箱格式/密码强度校验</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">使用符合规则的邮箱与 10 位密码</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">API 文档打不开</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">路径已变更</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">尝试 /ftp/api-docs 或 /restful-api-spec</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">挑战进度丢失</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">容器数据卷被清理</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">使用 docker compose down（不带 -v）</span></section></td></tr><tr><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">修改后不生效</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">本地 npm 缓存问题</span></section></td><td style="border:1px solid #dfe4ec;padding:9px 8px;font-size:13.5px;color:#4a4a4a;line-height:1.65;word-break:break-all;"><section><span leaf="">删除 node_modules 后重装并清理缓存</span></section></td></tr></tbody></table>  
  
**💡 提示**  
　关于账号：Juice Shop 支持**自由注册**  
，但如果某个挑战提示需要特定账号（如管理员），可以通过前面的 IDOR 漏洞直接查看用户列表获得。不要试图猜密码——这个靶场的设计思路就是「绕过而非硬攻」。  
  
## 12等保视角：业务逻辑漏洞为何最难测  
  
这是本章最想传达给运维安全同行的一点。技术类漏洞有明确的检测特征，而业务逻辑漏洞**没有通用检测方法，只能靠理解业务**  
。  
### 一、为什么扫描器扫不出业务逻辑漏洞  
  
扫描器的工作模式是：发送预设的恶意特征，检测异常响应。但「把商品价格改成 0」这个请求，在协议层面**完全合法**  
——它是格式正确的 JSON，包含服务端认识的字段，只是值不合法。扫描器没有任何理由报错。  
  
换句话说：**业务逻辑漏洞的攻击流量，在协议层面与正常流量无法区分**  
。这决定了它的发现只能依赖「懂业务的人手动构造异常场景」。  
### 二、检测思路：把业务规则写成断言  
  
既然无法用特征检测，那就**用业务规则反推测试用例**  
。对每个涉及金额、权限、状态的接口，问三个问题：  
1. **数值问题**  
：这个值是客户端提交的吗？如果是，服务端校验了吗？能否传负数、零、超大值、NaN、科学计数法？  
  
1. **身份问题**  
：这个操作校验了「操作者是否有权操作该资源」吗？换个用户 ID / 组织 ID 还成立吗？  
  
1. **时序问题**  
：这个流程有状态机约束吗？能否跳过前置步骤（先支付后下单）、重复执行（同一优惠用多次）、并发执行（同时提交两个请求）？  
  
### 三、源码审查的高危信号（等保：应用安全审计）  
- **信任客户端的价格/金额/库存/积分字段**  
——最经典的逻辑漏洞源头。  
  
- **仅凭 URL 中的 ID 取数据，未校验归属**  
——IDOR 的标准特征。  
  
- **订单/审批状态可直接由前端指定**  
——状态机缺失。  
  
- **优惠、折扣、积分无幂等与叠加校验**  
——可被批量套利。  
  
- **硬编码密钥、密钥与代码同仓**  
——直接可伪造令牌。  
  
### 四、把业务安全纳入等保测评范围  
- **访谈与走查**  
：安全测评阶段，要求业务部门提供关键流程说明，并由测评人员手工构造边界场景，而非只跑工具。  
  
- **专项测试**  
：对涉及资金、积分、权限的接口，执行越权、参数篡改、重放、并发四类用例。  
  
- **代码审计**  
：将「金额/权限/状态由客户端决定」列为必检项，在 CI 流水线中固化检查规则。  
  
**💡 提示**  
　一个非常实用的建议：**把 Juice Shop 部署成企业内网的「业务安全靶标」**  
，安排研发、测试、安全三方共同参与一次演练。让业务人员亲自尝试「改价格」，他们才会真正理解「为什么一个字段就能造成资损」——这种认知比任何培训都深刻。  
  
## 13总结  
  
DVWA 告诉你「漏洞有标准形态，防御有标准答案」；Juice Shop 告诉你「真实世界里，最贵的漏洞往往没有标准答案」。  
  
它逼着你去思考：如果你是开发，你会在哪里犯下这个错误？如果你是运维，你该如何在没有告警的情况下发现它？**这种换位思考，正是安全从「工具驱动」走向「业务理解」的分水岭。**  
  
**💡 提示**  
　下一章我们进入 WebGoat——同样是 OWASP 官方出品，但它走的是完全不同的路线：**把每个知识点包装成一节课，配上原理讲解和引导式提示**  
，像一门可以动手的自学课程。  
> 免责声明：  
>   
  
  
