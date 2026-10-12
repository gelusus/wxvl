#  CVSS满分10.0！WordPress Bricksforge插件无认证RCE漏洞，现已被野外利用  
 看雪学苑   2026-10-12 10:07  
  
Patchstack安全监测平台证实，Bricksforge插件曝出严重安全漏洞CVE-2026-85097，该漏洞影响3.1.8.9及更早全部版本，官方已在3.1.8.10版本完成补丁修复（最新稳定版4.0.1）。  
  
  
**插件背景**  
  
Bricksforge被称为Bricks可视化建站工具的“瑞士军刀”，是WordPress生态热门扩展，可实现动画、高级表单、滚动视频等丰富页面功能，大量企业与个人建站网站在使用。  
  
  
**漏洞原理：参数校验缺陷，引发任意代码执行**  
  
漏洞根源在于表单提交环节，程序对` temporaryFileUploads`参数内`url`字段校验不足：  
  
  
上传阶段会校验文件MIME类型；但提交表单时，服务端直接信任客户端提交的文件元数据，不对url扩展名做二次校验。  
  
  
攻击者可分4步完成入侵：  
  
1. 调用公开AJAX接口`bricksforge_regenerate_nonce`，直接获取合法nonce令牌，不需要任何账号登录  
  
2. 上传GIF/PHP多态文件：文件本身是合法GIF图片，内嵌PHP后门代码，顺利通过MIME检测，存入服务器临时上传目录  
  
3. 构造表单请求：服务端读取路径指向已校验的GIF文件，但自定义`url`字段后缀修改为php  
  
4. 服务端信任元数据，将文件内容映射为PHP文件；访问该地址，内嵌恶意代码直接执行  
  
  
**黑客绕过扩展名过滤手段**  
  
为绕过服务器后缀拦截，攻击者使用多种变形手段：  
  
- 备选PHP后缀：`.php5`、`.php7`、`.phtml`  
  
- 大小写混淆：`.PHP`、`.Php`  
  
- URL编码变形：`.%70hp`等编码绕过  
  
  
**攻击现状：自动化批量攻击，多IP集群扫描**  
  
Patchstack在2026年10月7日UTC时间21:47首次捕获到针对该漏洞的攻击流量。  
  
监测数据显示攻击具备明显自动化特征：  
  
1. 第一组：44个IP短时间发送完全相同载荷批量探测  
  
2. 第二组：3个IP对同一个临时文件，测试25种不同PHP后缀与编码变体  
  
  
攻击流量来自63个独立源IP，其中6个IP承担了近46%的全部攻击请求，攻击者大概率使用代理池、分布式基础设施批量扫站；目前无法将攻击行为归属到单一黑客组织。  
  
  
攻击目标接口：  
  
- REST API接口：`/wp-json/bricksforge/v1/form_submit`  
  
- admin-ajax接口，携带`bricksforge_form_submit`动作 + 恶意`temporaryFileUploads`参数  
  
成功利用后果：攻击者在服务器执行任意PHP代码，上传WebShell，窃取网站敏感数据，实现持久化后门，完全接管整个WordPress站点。  
  
  
**✅ 处置建议（网站管理员必做）**  
  
1. 优先升级：将Bricksforge插件更新至4.0.1最新版本，封堵漏洞入口  
  
2. 流量审计：监控`/wp-json/bricksforge/v1/form_submit`接口请求，重点筛查携带`temporaryFileUploads`参数的异常访问，以及高频重复表单提交行为  
  
3. 目录巡检：检查`/wp-content/uploads/bricksforge/tmp/`目录，排查是否存在异常PHP文件  
  
4. 深度防护：可在Web服务配置中禁止上传目录执行PHP代码，作为纵深防御手段  
  
  
**📌 连带漏洞提醒：多款WP插件同步遭受攻击**  
  
同一监测周期内，黑客还在利用另外两个WordPress插件XSS漏洞植入持久化后门：  
  
1. WPC Product Bundles for WooCommerce（CVE-2026-93836，CVSS 7.1），修复版本8.6.7  
  
2. Ninja Forms（CVE-2026-94504，CVSS 7.1），修复版本3.15.4  
  
  
黑客利用XSS向管理员浏览器注入JS脚本`x.js`，在管理员会话下创建隐藏后台管理员账号、部署恶意插件，生成魔法免登链接实现持久驻留。  
  
升级插件只能阻止新攻击，无法清除已经植入的后门。  
  
  
建议管理员：查询数据库排查隐藏管理员账号、检查must-use插件目录、完整审计Web服务器日志。  
  
  
**📈 WordPress 2026 Q2安全态势**  
  
Wordfence发布季度统计：  
  
2026年第二季度，WordPress相关漏洞共计2073个，高危漏洞数量环比上涨15.2%至182个，约57.3万个WP网站在本季度遭到入侵。  
  
  
攻击者重点打击TOP5漏洞：  
  
1. CVE-2024-28000（LiteSpeed Cache，9.8分，未授权提权）  
  
2. CVE-2019-9879（WPGraphQL，9.8分，创建管理员账号）  
  
3. CVE-2016-15042（前端文件管理器，9.8分，任意文件上传）  
  
4. CVE-2023-28121（WooCommerce Payments，9.8分，认证绕过提权）  
  
5. CVE-2025-3102（SureTriggers，8.1分，未授权创建管理员）  
  
  
攻击IP来源地排名：美国第一，其次为德国、捷克、澳大利亚、挪威、法国等。  
  
  
Wordfence建议：防火墙、及时补丁、安全加固、持续漏洞扫描+文件变更告警，是保护WP站点的核心手段。  
  
  
资讯来源：The Hacker News  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K3BzkG5P8uaw83lWp3Wo31ibp3HTmzCuSicw2WbsFPLCcnroQmjniaADLncayI5UUrVEH5k2AskrO9Lx8VzwXjia5UtlseVLUtaGibw/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3Px3buc5FddUru4qdyv78jYNyialzCuxXzsls3yMlg7H0kjhE7vEWU3vAwEHvpEQngmB2x4dS0PsBdqfazCIhh2T1CXgBKbJPc/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3F3iaSVSRFSibDZ2RPqv4PKZjJ01t0nSKTJ7pgXibTn9B5n32IYJGkVBhhdSj6EwicV5cmNPPq7ibBD5nUfrEWAXXoo0qricKOWibIibA/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2agRnLOg5ToMAa5c5rJaeCJGVYMwdhQDxkwtPfZWBqAkmgqtegoLZrwhlejTkjJpO8MZVIwUAvPhrgwSYmZZMrPsLTzjt1uKU/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
