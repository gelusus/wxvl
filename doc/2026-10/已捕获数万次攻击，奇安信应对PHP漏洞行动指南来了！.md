#  已捕获数万次攻击，奇安信应对PHP漏洞行动指南来了！  
 奇安信集团   2026-10-22 10:57  
  
10月22日，奇安信Xlab大网威胁分析平台捕获来自全球多个国家近千个源IP，尝试利用PHP远程代码执行漏洞对外发起数万次攻击。  
奇安信第一时间发布应对PHP漏洞行动指南,帮助政企机构构筑从检测、渗透测试到防护的体系化支撑，快速发现和阻断针对该漏洞的攻击。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/P5ichcO4unquqpwMwxwicw1ujI2Eb2vrPGWicy307hfcWmkU3e1FqdH60U4TwSesyZYTlUGViaCUcyrWiasuudJ886Q/640?from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mSqhDgeKrPDVS5ia7rFbCibRqMBcmNtzptB0QYMt5eYT8ppNg3jspPFZtQhWocdg8qCALuMD0SaKT4MLCYYK5jwkKLYuknFF4vSZ3lnEESicrY/640?wx_fmt=png&from=appmsg "")  
  
  
01  
  
PHP高危漏洞已发现在野利用，客户亟需自查及防护  
  
PHP是全球广泛使用的开源服务器端脚本语言，支持Apache mod_php、  
PHP-FPM等多种部署方式，W3Techs统计显示全球约四分之三的网站以其为服务器端语言，WordPress、Drupal、Laravel等主流应用均依赖PHP运行。  
  
2026年10月21日，PHP远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）公开，漏洞披露后，  
多国政府及网络安全机构迅速采取应急措施。美国联邦调查局（FBI）、美国国家安全局（NSA）、欧盟计算机应急响应团队（CERT-EU）等机构相继发出安全警告。日本首相高市早苗公开呼吁相关机构尽快部署安全补丁，必要时暂停网站服务。据媒体报道，荷兰、芬兰等国家的部分政府网站已采取预防性下线措施，日本部分政府部门及企业也紧急开展系统维护，凸显此次事件的严重性与紧迫性。  
  
奇安信CERT已成功复现该漏洞，并将其评级为高危，CVSS 3.1分数为8.1。  
奇安信XLab全球鹰测绘数据初步统计显示，全网约有2,600万网络资产可能涉及该漏洞的潜在影响范围。  
奇安信建议，鉴于该漏洞已发现在野利用，建议客户尽快做好自查及防护  
。  
  
漏洞信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mSqhDgeKrPA9IMU8yialB6SibvKk7CHBbia23RCTvzEvjteiaev51BPl2wibPnKibaXFhvfa1tbnvkxfiaUtPsS1ZxoDhlml4W7G8okN28FszOrC3Q/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/mSqhDgeKrPAMAGUVMmkmUmHsxF1xuUwmm79vDGT3X61pWA02VmznnkbI9Iy5GQHIG2qPSWmQ9oEogibuv6ChznwNxWwoQwaE09vF4Xn9Rc34/640?wx_fmt=png&from=appmsg "")  
  
在威胁方面，奇安信Xlab大网威胁分析平台捕获近千个源IP，来自全球多个国家，尝试利用漏洞对外发起数万次攻击。当前已经捕获的恶意行为包括：  
后门植入、恶意命令执行、挖矿程序部署、远程控制连接、恶意载荷下载、服务器凭据/数据窃取器。  
  
02  
  
奇安信全产品线应对：从检测到防护的立体化支撑  
  
为帮助政企机构快速发现和阻断针对该漏洞的攻击，奇安信多款产品已同步更新检测与防护能力：  
  
奇安信天眼检测方案。  
奇安信天眼新一代安全感知系统已能够有效检测针对该漏洞的攻击，请将规则版本升级到3.0.1021.15155或以上版本，规则ID为0x10022A87。流量探针规则升级方法为：系统配置->设备升级->规则升级，选择“网络升级”或“本地升级”。临时检测方案为：分析中心->日志检索->网络日志->Web访问日志，过滤语句  
data:("PHP_SESSION_UPLOAD_PROGRESS")  
，如请求体中包含大量垃圾字符或PHP反序列化数据，说明受到该漏洞攻击。  
  
奇安信网神云锁服务器安全管理系统（椒图）检测方案。  
椒图内置的RCE模块规则可直接检测该漏洞利用行为，无需更新规则库，默认安装Agent后RCE模块即自动启用。管理员可先通过【资产梳理】-【资产采集】新建采集任务，在【进程、软件应用、web框架、web应用】资产中检索PHP关键字，排查服务器上的PHP应用及版本；攻击发生时，椒图可针对Nginx加载PHP服务、Apache内嵌PHP等场景产生告警，并在告警详情中展示敏感命令与完整进程链，帮助安全人员快速定位和处置。  
   
  
奇安信开源卫士。  
   
开源卫士20261022.1527版本已支持对PHP远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）的检测。  
  
奇安信网站应用安全云防护系统。  
已全局更新所有云端防护节点的防护规则，支持对该漏洞的防护。  
  
奇安信网神网络数据传感器系统。  
NDS5000/7000/9000系列产品已具备该漏洞的检测能力，规则ID为8259、8260，建议用户尽快升级检测规则库至2610221013或以上。  
  
奇安信自动化渗透测试系统  
（加特林）  
。  
已能够有效检测针对该漏洞的攻击，请将插件版本和指纹版本升  
级到  
202610220000  
以上  
版本。  
规则名称：PHP   
远程代码执行漏洞(CVE-2026-109618、CVE-2026-109619)。  
规则升级方法为：系统管理->升级管理->插件升级（指纹升级），选择“网络升级”或“本地升级”。  
   
  
奇安信SaaS天戟AI智能渗透测试平台。  
该平台支持对对外暴露的业务系统网站进行自动化扫描，可快速精准识别PHP远程代码执行等漏洞。企业注册默认赠送积分，支持扫描1-2个站点。使用流程为  
：登录  
https://ai.qianxin.com  
注册企业账号，提交站点归属授权函，授权通过后即可  
在应用中心  
使用  
天戟  
对授权网站  
发起渗透测试任务  
。  
   
   
  
结语  
  
![](https://mmbiz.qpic.cn/mmbiz_png/mflphQBJic3STjs5EXib7E8mUTyW8OOibkBv0A2RCNVdryhNX6icoDx1nzX2h4hcMVPMctYfUcqLiaquTwUnRqlJSbg/640 "")  
  
  
PHP远程代码执行漏洞已不只是一次普通的补丁更新，而是对国内政企机构基础软件供应链安全的一次集中压力测试。面对国内  
百万级  
受影  
响资产的严峻形势，  
政企机构应尽快按照奇安信  
的建议完成版本核查、升级重启和临时缓解，并借助奇安信全产品线  
解决  
方  
案  
，  
实现从  
检测、渗透测试再到防护  
的立体化  
支撑  
。奇安信CERT将持续关注该漏洞进展，并第一时间为用户更新漏洞信息与处置建议。  
  
