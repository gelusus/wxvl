#  PHP远程代码执行漏洞来袭，启明星辰提供解决方案  
 启明星辰集团   2026-10-23 09:33  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j1hpA7GJeRAk1wI3cuSM30icSE6Vb81oH9eGapxyqgu6TzpTmgwIXuvH2kJ0JA1ajX5CywF8t3TKVr2iajEnty5uTvlSX6AaGkPt53tw1PZ9k/640?wx_fmt=gif&from=appmsg "")  
  
  
  
PHP是一种广泛使用的开源服务器端脚本语言，主要用于Web应用与动态网站开发，通常以PHP-FPM、CGI或Apache模块方式部署，是内容管理系统、电商、论坛、控制面板及各类业务系统的常见运行环境。  
  
  
10月21日，  
启明星辰监控到PHP官方披露了PHP核心中存在的两项远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）。两个漏洞均为CVSS 4.0，评分均为 9.2，漏洞等级为  
高危。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRDw14icKYxsGc34m2LGHt7x9bOl7OrLyG9icCiaZLpt6q2U6zFQySicZkCfSBbuOhsJM8lqN2VNMyejwapicicsRanpRwzNMwHtWIgyQ/640?wx_fmt=png&from=appmsg "")  
  
  
【漏洞复现】  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j1hpA7GJeRBIAF1bOfKhpeonwTSsnPWMp15vgoG350lraRbZHRT438YJ5pBo2jFicbYF6RdKEDjw1FCor5icDlqBC7EXO62Pl1VkLc8UkveOo/640?wx_fmt=jpeg "")  
  
  
【解决方案】  
  
  
一、官方修复方案：  
  
  
请受影响的用户尽快升级版本进行防护。下载导航链接：  
  
https://www.php.net/archive/2026.php  
  
  
二、启明星辰解决方案：  
  
  
1、启明星辰检测与防护类产品方案  
  
事件库下载地址：https://venustech.download.venuscloud.cn/  
  
- 启明星辰“天阗威胁分析一体机（TAR）”升级到20261022版本即可支持检测该漏洞。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRD5YNNqqLcaBiafKZMXKdpG3NibK5ohqyuM83qX2WpOOdMo0eCB9kSQgGSCToDcuvrXXZkYHFh1Ae9fvQXRYActQG082oHgiaPRpE/640?wx_fmt=png&from=appmsg "")  
  
- 启明星辰“天阗超融合检测探针（CSP）”升级到20261022版本即可支持检测该漏洞。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBrhESxPF7k2QhHrLSWgxEYvXYtI8chgS7MEicmnVBggoW1a9gcLltF5JIOXDa3Unj7ZevkIQPKpsKiaPh3ea2buUXpcfAoGgnog/640?wx_fmt=png&from=appmsg "")  
  
  
- 启明星辰“天清入侵防御系统（IPS）”升级到20261022版本即可支持防护该漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCKS7x4ZbD4jrIx1qfOAPEpWs29y2iaaAsZzDETFdialcFLny3JAn6ZaMcPboxpMIxl3DJOftXVdiaT9HOHY2o9EthcOLhv0PduEo/640?wx_fmt=png&from=appmsg "")  
  
  
- 启明星辰“天清WEB安全应用网关（WAF）”升级到20261022版本即可支持防护该漏洞。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCE4GQicolAjwibs9l8sJV64BbVWmHFT7hTbPFnCkM1tTjcA24T3brEdKhFDhzGpPZbT1iafaeOQN3L2qEic6R0IeAE75NmQNcrSlA/640?wx_fmt=png&from=appmsg "")  
  
- 启明星辰“天阗全流量分析取证系统（NFT）”升级到20261022版本即可支持狩猎该漏洞。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRBpe6zJSa0iaBSLib0RcA7u4ictnF4UnZbsMxJJrWaSYB8aqK5emLZKvIiafyHIvcXmbER1UnsmQh49dzN6icf6HJvPJ72hfs7exUm0/640?wx_fmt=png&from=appmsg "")  
  
- 启明星辰“天阗入侵检测与管理系统（IDS）”升级到20261022版本即可支持防护该漏洞。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRCoNfqRS0fULzHKch3XDSGgSsHUBWBCvEsb9ib4sxE4a08qM6geSicSNO3W7qweRHFs3dw3V1BLrLiaVjytOXk9ibJraX1EIZP5pkI/640?wx_fmt=png&from=appmsg "")  
  
  
2、启明星辰“天镜脆弱性扫描与管理系统”，升级到“608000124-608000125.wvs”版本即可支持检测该漏洞的风险。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRAc2PeYwkiaVzQeiaZLicRxicyySoXe9EzwvOrcmDGdjWyuJxnfib6uO691nRtyX008HhtmOaqZ8j6jBEibT8SAudbs8gonnEuw1qIK8/640?wx_fmt=png&from=appmsg "")  
  
  
漏洞升级插件包下载地址：  
  
https://venustech.download.venuscloud.cn/  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRBYicOWaUiaib08FS6tibMwwF939BnfN5jxhSXDnxhmq4kgK2zgLoB212saH8h4LXicz3oBTu63AB0rKm5xtgI1rgPhaxgIt7f3cl0g/640?wx_fmt=png&from=appmsg "")  
  
  
3、启明星辰安全管理和态势感知平台产品方案  
  
  
1）用户可以依托部署的泰合安全管理和态势感知平台，通过关联分析策略配置，结合实际环境中采集的主机系统日志和安全设备告警日志进行持续监控，从而发现“PHP远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”的漏洞利用攻击行为。  
  
  
• 通过平台脆弱性发现功能，针对“PHP远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”建立漏洞扫描任务，排查管理网络中受此漏洞影响的重要资产；  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRCudEd9zKiahSAibv3DONrOQibEz4FLJyyEJ86Zl5micZPERkdAeIFfm1BD2Sz43RymW8bNY7tMVgNzhtJhBAx99EAzniaJgNiclYFPc/640?wx_fmt=png&from=appmsg "")  
  
  
• 平台“关联分析”模块中，添加“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，通过启明星辰检测设备、目标主机系统等设备的告警日志，发现外部攻击行为：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRAAYdicNcEjVhcibnA4OHJc3R02jjNu6FQKia7CMSmP9uGjoW0SYllnGXnmL8B8jUZkmwYq9dZSZgxJibd51vC8zlJkMIEwmCVTdIk/640?wx_fmt=png&from=appmsg "")  
  
  
同时，通过分析规则自动将“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)  
”  
漏洞利用的可疑行为源地址添加到观察列表“高风险连接”中，作为内部情报数据使用；  
  
  
• 添加“L3_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，条件日志名称等于或包含“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，攻击结果等于或属于“攻击成功”，目的地址引用资产漏洞或源地址匹配威胁情报，从而提升关联规则的置信度。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/j1hpA7GJeRCicS9mFjhpk17Il7icpU2Fy1DpDaBw9ocL44yia8Pu3M5wvBhECVYxicDRd7OoibwYFTPSiaXia8JdezzsAHLZQ7rJIeKrfaNzILHI9k/640?wx_fmt=png&from=appmsg "")  
  
  
2）基于ATT&CK攻击链分析与SOAR处置建议  
  
  
根据对PHP远程代码执行漏洞(CVE-2026-109618、CVE-2026-109619)的攻击利用过程进行分析，攻击链涉及多个ATT&CK战术和技术阶段，覆盖的TTP包括：  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/j1hpA7GJeRC9d1Yl0CqeVYDsFeDJDlibGeoE5iaCKyOzQXTTMgENoxB1CYOSA34Xrn8p4oHKJC14ZSYE6yG2vvgDicFheOCZIWpAVReAcibjRC0/640?wx_fmt=png&from=appmsg "")  
  
  
通过平台内置SOAR自动化、半自动化编排联动响应处置能力，针对该漏洞利用的告警事件定制响应剧本，实现标准化自动化应急处置。  
  
  
4、启明星辰终端安全方案  
  
  
天珣终端安全一体化（EDR）已复现该漏洞，提供自定义poc，根据资产模块、进程httpd、进程php-fpm获取PHP组件版本信息，可从服务端下发poc进行全网同步验证，匹配漏洞资产，预防漏洞攻击风险。  
  
  
  
  
•  
  
END  
  
•  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzA3NDQ0MzkzMA==&mid=2651688529&idx=1&sn=15ae6574a6aa36aa6b5b871b081a5da6&scene=21#wechat_redirect)  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/BwR7Xg3aXhZeib4948EIKnKMjz0waPZd0pSaTDkeicciaKbyYLCFV9Xk5RmB049bU37614KRYF7OGrDb520hwEibkg/640?wx_fmt=gif "")  
  
  
