#  紧急预警｜PHP php2shell高危漏洞，无需认证可RCE！  
原创 建哥聊安全
                    建哥聊安全  建哥聊安全   2026-10-29 08:58  
  
一、漏洞简述  
  
近期披露一组PHP  session.upload_progress  组件高危远程代码执行漏洞，代号 php2shell，包含核心漏洞 CVE-2026-109618、CVE-2026-109619，搭配关联配置依赖漏洞 CVE-2026-115560。  
  
漏洞CVSS评分高达 9.2（严重），攻击者无需身份认证，构造恶意请求即可实现远程代码执行，风险等级极高。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trFGxlyJ6WhEYHBJzs4kAxOhMia2xYPkQ9b0h1cAkXLkr8mCgTcBC8FI5fxialic2pl90oHo1nyxvibsZZPb6h2NwXq99nupxlpmlE0/640?wx_fmt=png&from=appmsg "")  
  
二、 漏洞原理  
  
漏洞根源为  php_session_rfc1867_callback  函数内的**堆释放后使用（Heap Use-After-Free）**缺陷：PHP在处理上传进度时，缓存了会话数据的原始内存指针；特定逻辑执行后，对应内存已经释放，但指针没有同步更新。攻击者可构造请求抢占这块空闲内存，最终劫持程序流程，执行任意代码。  
  
触发条件：  
  
请求携带  PHP_SESSION_UPLOAD_PROGRESS  字段，并且请求格式为  multipart/form-data  即可进入漏洞触发路径；  
  
重点提醒：PHP 8.4及以上版本， request_parse_body()  支持 PUT、PATCH 方法解析表单，漏洞不再局限POST请求。  
  
受影响版本  
  
影响范围跨度极大，覆盖   
PHP 5.4.0 ~ 8.5.11，长达十余年的版本均存在风险。  
  
官方修复版本（优先升级到此版及以上）  
  
- PHP 8.5.x → 8.5.12 +  
  
- PHP 8.4.x → 8.4.27 +  
  
- PHP 8.3.x → 8.3.36 +  
  
- PHP 8.2.x → 8.2.35 +  
  
 重要提示：PHP 8.1及更早版本已经停止官方维护（EOL），官方不会提供补丁。这类环境建议联系操作系统厂商确认是否有移植补丁，或直接规划版本升级。  
  
三、处置方案（按优先级执行）  
  
方案一（推荐）：首选｜升级PHP（根治方案）  
  
升级到上面列出的官方修复版本，是彻底消除该漏洞风险的唯一办法，优先在测试环境验证业务兼容性后上线。  
  
方案二：临时应急加固（无法立刻升级时）  
  
核心思路：关闭漏洞入口功能，关闭session.upload_progress 不会影响业务正常文件上传，仅前端无法展示上传进度条  
  
1. 修改  php.ini   
  
session.upload_progress.enabled = Off  
  
2. PHP-FPM 环境，必须额外在池配置内强制覆盖（php_admin_flag优先级更高）  
  
php_admin_flag[session.upload_progress.enabled] = off  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trHHNYuIkrDKlM2nHOF4CoNVbgYZeC8YjDOg0GAia9vPKuW2kkrsZ7wEKeHwwOicMibXBNVcTNkpic2FicDyB7W6fejklQkic2TluVEibk/640?wx_fmt=png&from=appmsg "")  
  
3. 纵深防御补充（推荐同步配置）  
  
session.use_strict_mode = 1  
  
4. 生效操作：修改配置后，重启PHP-FPM / Nginx / Apache  
  
方案三：WAF边界防护（辅助拦截，不能替代上面加固）  
  
WAF/反向代理添加检测规则，拦截恶意请求特征：  
  
- 请求Content-Type包含  multipart/form-data （注意兼容逗号分隔的变种格式）  
  
- 请求体内携带字段  PHP_SESSION_UPLOAD_PROGRESS   
  
- 规则覆盖 POST / PUT / PATCH 多种请求方法  
  
自查清单（运维直接复制核对）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trH4PrytkcQ2mE0rMYRHega0MDz5CiaPyzbReFiay87XpKT6EzHptrC4yL6ncO7lmETHG0WRPwh3qG60jJlBM9KLKOjFryBa3t5iac/640?wx_fmt=png&from=appmsg "")  
  
1.确认服务器PHP版本，核对是否在受影响范围  
  
2.业务是否可以短期下线上传进度条功能  
  
3.php.ini + FPM池配置两处都检查  session.upload_progress.enabled   
  
4.配置修改后确认服务已经重启，配置生效  
  
5.WAF策略添加对应拦截规则，做第二层防护  
  
6.EOL老旧PHP版本，评估升级计划  
  
总结：php2shell 是近期PHP生态危害很大的高危漏洞，无认证RCE、影响周期长，黑客利用门槛低。最优解：升级PHP到官方修复版本应急兜底：关闭  session.upload_progress.enabled ，阻断攻击入口  
  
  
免责声明：本文仅为安全预警与加固指引，加固前请先在测试环境验证业务可用性；防护策略根据自身业务场景调整。  
  
