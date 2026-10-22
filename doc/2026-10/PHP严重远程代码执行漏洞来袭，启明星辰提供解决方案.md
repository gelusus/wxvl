#  PHP严重远程代码执行漏洞来袭，启明星辰提供解决方案  
 启明星辰集团   2026-10-22 11:50  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j1hpA7GJeRAk1wI3cuSM30icSE6Vb81oH9eGapxyqgu6TzpTmgwIXuvH2kJ0JA1ajX5CywF8t3TKVr2iajEnty5uTvlSX6AaGkPt53tw1PZ9k/640?wx_fmt=gif&from=appmsg "")  
  
  
  
PHP是一种广泛使用的开源服务器端脚本语言，主要用于Web应用与动态网站开发，通常以PHP-FPM、CGI或Apache模块方式部署，是内容管理系统、电商、论坛、控制面板及各类业务系统的常见运行环境。  
  
  
10月21日，  
启明星辰监控到PHP官方披露了PHP核心中存在的两项远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）。  
  
- CVE-2026-109618与内存不足时的异常中断有关：  
部分请求数据尚未完成初始化，执行流程即被中断，后续清理可能错误处理无效内存状态。  
  
- CVE-2026-109619涉及跨请求残留的全局状态：  
一个请求结束后，工作进程仍可能保存指向已释放内存的引用，后续请求在正常初始化之前读取该状态，从而触发释放后使用。  
  
两个漏洞均为CVSS4.0，评分均为9.2，漏洞等级为  
高危。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRDw14icKYxsGc34m2LGHt7x9bOl7OrLyG9icCiaZLpt6q2U6zFQySicZkCfSBbuOhsJM8lqN2VNMyejwapicicsRanpRwzNMwHtWIgyQ/640?wx_fmt=png&from=appmsg "")  
  
  
【漏洞复现】  
  
  
本次复现使用自建Docker靶场，环境采用php8.4.26官方镜像， php:8.4.26-apache的mod_php prefork模式，memory_limit设为8M留出OOM触发窗口，session.upload_progress开启。  
  
  
组合利用的核心路径是：先以携带PHP_SESSION_UPLOAD_PROGRESS及大体积字段的multipart请求，在请求启动阶段、业务代码执行之前打爆内存，触发longjmp跳转；此时部分请求级结构尚未完成初始化，却已在后续请求关闭阶段被当作有效数据消费，形成UAF型内存破坏。在此基础上，通过多轮地址爆破叠加堆布局命中，逐步将脏结构引导至可控状态，最终拿下RCE。  
  
  
概率问题贯穿两层：地址命中本身具有随机性，且即便地址对上，堆布局还须恰好落入可利用区间——htdata偏移受分配器状态影响，同一地址在不同轮次可能对应不同布局，需要多轮尝试才能命中地址和堆布局。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBJucBgBNia9tDyx9fiaczribC9joVOxmTdcAOSXhDXGEkrHwCADZmAXRvQSKaKScyGHZK1iciaVDM2T5yC9LlvconKVxMnPqg5tGCk/640?wx_fmt=png&from=appmsg "")  
  
  
【在野利用】  
  
  
VenusEye威胁情报中心的僵尸网络诱捕系统成功捕获并监测到多起针对PHP核心高危远程代码执行漏洞（CVE-2026-109618 与 CVE-2026-109619）的在野利用活动。  
  
  
在剔除海量的常规环境探测与版本扫描后，系统精准甄别出100余起针对性的高危利用尝试。分析表明，攻击者使用了高度特征化的自动化工具（如携带php2shell-scan指纹），通过向目标发送包含深层嵌套数组（如f0[k0]...[k59]）的异常HTTP请求，蓄意触发CVE-2026-109618导致PHP进程内存耗尽（OOM）崩溃。  
  
  
进阶的载荷分析显示，部分攻击源已开始尝试构建完整的RCE利用链。在触发OOM导致内存状态异常后，相关报文中出现了针对CVE-2026-109619 (UAF漏洞)的利用特征。攻击者试图通过堆喷射及伪造底层zval数据结构，以期劫持进程执行流。从捕获的二进制结构片段中可提取出对应的恶意Shell指令（如尝试利用wget拉取并运行名为nfs的远端木马文件），这表明该僵尸网络正致力于将此漏洞组合武器化，意图在受影响节点上实现代码执行与后续的感染扩张。  
  
  
OOM进阶利用尝试报文（堆喷射带传播shell）：  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRDp7CV3ZCtq3x11EQGD4Akumzqqz5nTHuuZ2RS9MskkOq0Eib97a5EicevHGxLLz4AlNp1mC8iaOzd8PdRgiayGggtLcWsKPHIOiafw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBVkVlMLARVBnDKtHs69u5YicPccqp6m0AkbicrvibjLdUr1ibeWTLEQ4ZhgCclTcLDy2Gjj5u2vcibMRPNtPmqzdNvjH17E75myrEo/640?wx_fmt=png&from=appmsg "")  
  
  
OOM基础尝试报文：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRAsibJbDibf6YPn0Qk6vUdfvnicrgyQ8aQE3BUYdUmYMxZZTnoib2rIdjQbTaHV7REA5iaYtKuORLYsxkFHBRKsGroCwLx9JDtRyrs0/640?wx_fmt=png&from=appmsg "")  
  
  
  
【解决方案】  
  
  
一、官方修复方案：  
  
  
请受影响的用户尽快升级版本进行防护。下载导航链接：  
  
https://www.php.net/archive/2026.php  
  
  
二、启明星辰解决方案：  
  
  
1、启明星辰检测与防护类产品方案  
  
事件库下载地址：https://venustech.download.venuscloud.cn/  
  
  
（1）启明星辰“天阗威胁分析一体机（TAR）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRD5YNNqqLcaBiafKZMXKdpG3NibK5ohqyuM83qX2WpOOdMo0eCB9kSQgGSCToDcuvrXXZkYHFh1Ae9fvQXRYActQG082oHgiaPRpE/640?wx_fmt=png&from=appmsg "")  
  
  
（2）启明星辰“天阗超融合检测探针（CSP）” 升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBrhESxPF7k2QhHrLSWgxEYvXYtI8chgS7MEicmnVBggoW1a9gcLltF5JIOXDa3Unj7ZevkIQPKpsKiaPh3ea2buUXpcfAoGgnog/640?wx_fmt=png&from=appmsg "")  
  
  
  
（3）启明星辰“天清入侵防御系统（IPS）”升级到20261022版本即可支持防护该漏洞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCKS7x4ZbD4jrIx1qfOAPEpWs29y2iaaAsZzDETFdialcFLny3JAn6ZaMcPboxpMIxl3DJOftXVdiaT9HOHY2o9EthcOLhv0PduEo/640?wx_fmt=png&from=appmsg "")  
  
  
  
（4）启明星辰“天清WEB安全应用网关（WAF）”，升级到20261022版本即可支持防护该漏洞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCE4GQicolAjwibs9l8sJV64BbVWmHFT7hTbPFnCkM1tTjcA24T3brEdKhFDhzGpPZbT1iafaeOQN3L2qEic6R0IeAE75NmQNcrSlA/640?wx_fmt=png&from=appmsg "")  
  
  
（5）启明星辰“天阗全流量分析取证系统（NFT）”，升级到20261022版本即可支持狩猎该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBpe6zJSa0iaBSLib0RcA7u4ictnF4UnZbsMxJJrWaSYB8aqK5emLZKvIiafyHIvcXmbER1UnsmQh49dzN6icf6HJvPJ72hfs7exUm0/640?wx_fmt=png&from=appmsg "")  
  
  
（6）启明星辰“天阗入侵检测与管理系统（IDS）”，升级到20261022版本即可支持防护该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRCoNfqRS0fULzHKch3XDSGgSsHUBWBCvEsb9ib4sxE4a08qM6geSicSNO3W7qweRHFs3dw3V1BLrLiaVjytOXk9ibJraX1EIZP5pkI/640?wx_fmt=png&from=appmsg "")  
  
  
2、启明星辰“天镜脆弱性扫描与管理系统”，升级到“608000124-608000125.wvs”版本即可支持检测该漏洞的风险。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRAc2PeYwkiaVzQeiaZLicRxicyySoXe9EzwvOrcmDGdjWyuJxnfib6uO691nRtyX008HhtmOaqZ8j6jBEibT8SAudbs8gonnEuw1qIK8/640?wx_fmt=png&from=appmsg "")  
  
  
漏洞升级插件包下载地址：  
  
https://venustech.download.venuscloud.cn/  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRBYicOWaUiaib08FS6tibMwwF939BnfN5jxhSXDnxhmq4kgK2zgLoB212saH8h4LXicz3oBTu63AB0rKm5xtgI1rgPhaxgIt7f3cl0g/640?wx_fmt=png&from=appmsg "")  
  
  
3、启明星辰安全管理和态势感知平台产品方案  
  
  
1）用户可以依托部署的泰合安全管理和态势感知平台，通过关联分析策略配置，结合实际环境中采集的主机系统日志和安全设备告警日志进行持续监控，从而发现“PHP远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”的漏洞利用攻击行为。  
  
  
• 通过平台脆弱性发现功能，针对“PHP 远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”建立漏洞扫描任务，排查管理网络中受此漏洞影响的重要资产；  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCudEd9zKiahSAibv3DONrOQibEz4FLJyyEJ86Zl5micZPERkdAeIFfm1BD2Sz43RymW8bNY7tMVgNzhtJhBAx99EAzniaJgNiclYFPc/640?wx_fmt=png&from=appmsg "")  
  
  
• 平台“关联分析”模块中，添加“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，通过启明星辰检测设备、目标主机系统等设备的告警日志，发现外部攻击行为：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRAAYdicNcEjVhcibnA4OHJc3R02jjNu6FQKia7CMSmP9uGjoW0SYllnGXnmL8B8jUZkmwYq9dZSZgxJibd51vC8zlJkMIEwmCVTdIk/640?wx_fmt=png&from=appmsg "")  
  
  
同时，通过分析规则自动将"L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)"漏洞利用的可疑行为源地址添加到观察列表“高风险连接”中，作为内部情报数据使用；  
  
  
• 添加“L3_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，条件日志名称等于或包含“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，攻击结果等于或属于“攻击成功”，目的地址引用资产漏洞或源地址匹配威胁情报，从而提升关联规则的置信度。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRCicS9mFjhpk17Il7icpU2Fy1DpDaBw9ocL44yia8Pu3M5wvBhECVYxicDRd7OoibwYFTPSiaXia8JdezzsAHLZQ7rJIeKrfaNzILHI9k/640?wx_fmt=png&from=appmsg "")  
  
  
2）基于ATT&CK攻击链分析与SOAR处置建议  
  
  
根据对 PHP 远程代码执行漏洞 (CVE-2026-109618、CVE-2026-109619) 的攻击利用过程进行分析，攻击链涉及多个 ATT&CK 战术和技术阶段，覆盖的 TTP 包括：  
  
  
TA0001 - 初始访问：    T1190 利用面向公众的应用程序  
  
TA0002 - 执行：        T1203 利用代码执行漏洞  
  
TA0004 - 权限提升：    T1055 进程注入  
  
TA0009 - 数据收集：    T1005 从本地系统收集数据  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRC9d1Yl0CqeVYDsFeDJDlibGeoE5iaCKyOzQXTTMgENoxB1CYOSA34Xrn8p4oHKJC14ZSYE6yG2vvgDicFheOCZIWpAVReAcibjRC0/640?wx_fmt=png&from=appmsg "")  
  
  
通过平台内置 SOAR 自动化、半自动化编排联动响应处置能力，针对该漏洞利用的告警事件定制响应剧本，实现标准化自动化应急处置。  
  
  
4、启明星辰终端安全方案  
  
  
天珣终端安全一体化（EDR）已复现该漏洞，提供自定义poc，根据资产模块、进程httpd、进程php-fpm获取PHP组件版本信息，可从服务端下发poc进行全网同步验证，匹配漏洞资产，预防漏洞攻击风险。  
  
  
  
  
  
•  
  
END  
  
•  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NDQ0MzkzMA==&mid=2651688529&idx=1&sn=15ae6574a6aa36aa6b5b871b081a5da6&scene=21#wechat_redirect)  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/BwR7Xg3aXhZeib4948EIKnKMjz0waPZd0pSaTDkeicciaKbyYLCFV9Xk5RmB049bU37614KRYF7OGrDb520hwEibkg/640?wx_fmt=gif "")  
  
  
