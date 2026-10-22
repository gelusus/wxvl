#  PHP 严重远程代码执行漏洞来袭，网御星云提供优选解决方案  
 网御星云   2026-10-22 12:20  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/nT4hKuibpowwDeTWjMSRX7LsmOjoeqYqtyNzVJ57p3vqIsYCSzTsfNo0vCnFmOBrCRKTJ8uibpmzJp2J0boLMcVXxoBmp8ibibFFVIK6v9JfkHQ/640?wx_fmt=gif "")  
  
  
PHP 是一种广泛使用的开源服务器端脚本语言，主要用于 Web 应用与动态网站开发，通常以 PHP-FPM、CGI 或 Apache 模块方式部署，是内容管理系统、电商、论坛、控制面板及各类业务系统的常见运行环境。  
  
  
2026年10月，  
网御星云监控到PHP 官方披露了 PHP 核心中存在的两项远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619）。  
  
  
• CVE-2026-109618与内存不足时的异常中断有关：部分请求数据尚未完成初始化，执行流程即被中断，后续清理可能错误处理无效内存状态。  
  
  
• CVE-2026-109619涉及跨请求残留的全局状态：一个请求结束后，工作进程仍可能保存指向已释放内存的引用，后续请求在正常初始化之前读取该状态，从而触发释放后使用。  
  
  
两个漏洞均CVSS 4.0，评分均为 9.2，漏洞等级为  
高危。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nT4hKuibpowxN3unmRokIkNdZXhXicHLhfdStvTMhAnViasvFLBWazpte6dreYBb8yiauicX1t1OdhCP5GiafhtibVadpuT2rHzmV5ia0pHGN2CmicJ8/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
  
**漏洞复现截图**  
  
  
****  
本次复现使用自建Docker靶场，环境采用PHP8.4.26官方镜像， PHP:8.4.26-apache 的 mod_php prefork 模式，memory_limit 设为 8M 留出 OOM 触发窗口，session.upload_progress 开启。  
  
  
组合利用的核心路径是：先以携带 PHP_SESSION_UPLOAD_PROGRESS 及大体积字段的 multipart 请求，在请求启动阶段、业务代码执行之前打爆内存，触发 longjmp 跳转；此时部分请求级结构尚未完成初始化，却已在后续请求关闭阶段被当作有效数据消费，形成 UAF 型内存破坏。在此基础上，通过多轮地址爆破叠加堆布局命中，逐步将脏结构引导至可控状态，最终拿下 RCE。  
  
  
概率问题贯穿两层：地址命中本身具有随机性，且即便地址对上，堆布局还须恰好落入可利用区间——htdata 偏移受分配器状态影响，同一地址在不同轮次可能对应不同布局，需要多轮尝试才能命中地址和堆布局。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowwmBlsbYGibvkoj65RNCWpEQTTScrjHkMEt7gXWuVz9jbicSiaymicuWh8B4pcH41layV7ubx5Lv5QXtHfLPMLbK9pSn4r6mlV8GjA/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
**修复建议**  
  
  
  
**一、官方修复方案**  
  
  
请受影响的用户尽快升级版本进行防护，下载导航链接：  
  
https://www.php.net/archive/2026.php  
  
  
**二、网御星云方案**  
  
  
**1.网御星云检测类产品方案**  
  
  
（1）“网御威胁分析一体机（TAR）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowyJQn3fRa2kXcDLeiapWjJoTKgpal228xBjUk7rAUb2Yo8Jvac0b3L91yr5dUYPKWwj17ib26960BicqqhJhScfqk5Y7QWpmQveibc/640?wx_fmt=png&from=appmsg "")  
  
  
（2）“网御超融合检测探针（CSP）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowxnvDCxwicEnCohgFeZrn2V3GUmDWJicX39nBIdzCP9qnjfCvSibDhmicMGLoIh9QoTRXtrs2L9XF7drvnEVuDETvQtxDp339Ic5s0/640?wx_fmt=png&from=appmsg "")  
  
  
（3）“网御入侵防御系统（IPS）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowxbgdcYRWaPmzb783xhbxMJVVHtXd7DJ3PvbaMOfxibqvr2JdgpXewKocrO6MpdAJhfPXCkXO2PMCqYA6yiaQ0oJAFwcxicVqBrPM/640?wx_fmt=png&from=appmsg "")  
  
  
（4）“网御WEB安全应用网关（WAF）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nT4hKuibpowwGSlbzmvgxpibg2icEnOicalnRNMllozdhicTC0TxvPicf9N3sOImjxEPZ6icttKj8H35mER0RLX02qI9QeTGSQdWicIc3BVib3BHQcibQ/640?wx_fmt=png&from=appmsg "")  
  
  
（5）“网御全流量分析取证系统（NFT）”升级到20261022版本即可支持狩猎该漏洞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nT4hKuibpowyKYkQACgz2GibWYTXT4md9knsgmYSicEhsyviaU7SKia9xkqCanlh7EibVQXw1gHvjociaicTEZdBxLv0zb5bZlJGQHUXqMia2xlcUvFo/640?wx_fmt=png&from=appmsg "")  
  
  
（6）“网御入侵检测系统（IDS）”升级到20261022版本即可支持检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowwphicEqdOKA7dSAibnm0lWqTHAqxREjHGn2t7ESoTib45sSaZ63sRicibvApzjaFbhibBEIepiaw1yCMVsPJv6BxDpuadfmCRpIxXRWU/640?wx_fmt=png&from=appmsg "")  
  
  
事件库下载地址：  
  
https://leadsec.download.venuscloud.cn/  
  
  
**2.网御星云漏扫产品方案**  
  
****  
“网御漏洞扫描系统”升级到“608000124-608000125.wvs”版本即可有效检测该漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowxMP0vKxolKYLn4jw1BW4tbun8B3ZpqsWQJAlI62T6H2UwtKhzicKJTuicYUo4SiaKHEkezjWvLm7cS0LSgYeicibpXQePwyEW0Tpc0/640?wx_fmt=png&from=appmsg "")  
  
  
漏洞升级插件包下载地址：  
  
https://leadsec.download.venuscloud.cn/  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowzVD0KZC2fcZ8kCj6MoOXRR16tYrFwibiaYmtiays4ib6eShGH8sNP7aQXe2prib2TibmTib3tsibYVrXokazaIS6jOg7sG8HjCFYciaJoI/640?wx_fmt=png&from=appmsg "")  
  
  
**3.网御星云安全管理和态势感知平台产品方案**  
  
  
（1）用户可以依托部署的网御安全管理和态势感知平台，通过关联分析策略配置，结合实际环境中采集的主机系统日志和安全设备告警日志进行持续监控，从而发现“PHP远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”的漏洞利用攻击行为。  
  
  
• 通过平台脆弱性发现功能，针对“PHP 远程代码执行漏洞(CVE-2026-109618，CVE-2026-109619)”建立漏洞扫描任务，排查管理网络中受此漏洞影响的重要资产。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nT4hKuibpowwsOeWicicVSyVO246Chozsj1CpecwxXKaRTuwmMLQoyyOpn75SgBAp4amQUnrnw55UicrYhdSln2bGicZDwqkttQ1UgvS0g5BstXM/640?wx_fmt=png&from=appmsg "")  
  
  
• 平台“关联分析”模块中，添加“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，通过网御星云检测设备、目标主机系统等设备的告警日志，发现外部攻击行为。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nT4hKuibpowxHz8mVxY6HkAaiaIbiaIsibqMclwE2JErHGRyx2G86472aQdxLh8wvibrOcFVfDaiccSLm51HCnP1eglvMWGvWQR2BABQh1ql8B4nw/640?wx_fmt=png&from=appmsg "")  
  
  
同时，通过分析规则自动将"L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)"漏洞利用的可疑行为源地址添加到观察列表“高风险连接”中，作为内部情报数据使用。  
  
  
• 添加“L3_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，条件日志名称等于或包含“L2_PHP远程代码执行漏洞(CVE-2026-109618|109619)”，攻击结果等于或属于“攻击成功”，目的地址引用资产漏洞或源地址匹配威胁情报，从而提升关联规则的置信度。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/nT4hKuibpowz62IRtric4lGFchlM820LS3C8saBWe3VnTzBbMkYpBxIhdrqrLmSBuyrM0gd599bEZJEOTg570RGiaLtQMa2j6HKbRUWlDAsqJA/640?wx_fmt=png&from=appmsg "")  
  
  
（2）基于ATT&CK攻击链分析与SOAR处置建议  
  
  
根据对 PHP 远程代码执行漏洞 (CVE-2026-109618、CVE-2026-109619) 的攻击利用过程进行分析，攻击链涉及多个 ATT&CK 战术和技术阶段，覆盖的 TTP 包括：   
  
  
TA0001 - 初始访问：T1190 利用面向公众的应用程序   
  
TA0002 - 执行：T1203 利用代码执行漏洞   
  
TA0004 - 权限提升：T1055 进程注入   
  
TA0009 - 数据收集：T1005 从本地系统收集数据  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nT4hKuibpowy40WnHvpK820g2VrsCQoRvfN45Qia5fmFuFr4UmPqr3dkkJibpE6ZEwicQk7hgTmTNoduIsMkQMfhRbhsZicPL4dksibUwuXwPic3cc/640?wx_fmt=jpeg&from=appmsg "")  
  
  
通过平台内置 SOAR 自动化、半自动化编排联动响应处置能力，针对该漏洞利用的告警事件定制响应剧本，实现标准化自动化应急处置。  
  
  
**4.网御星云终端安全方案**  
  
  
网御端点威胁检测产品（EDR）已复现该漏洞，提供自定义poc，根据资产模块、进程httpd、进程php-fpm获取PHP组件版本信息，可从服务端下发poc进行全网同步验证，匹配漏洞资产，预防漏洞攻击风险。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nT4hKuibpoww2waIBfxNHNUmmibcsNhDqqxP9PP949afHbOwuNEWhtHRhPuksDHB4XcI1jb4O5w3BiaykzjamZQbdtZOglkyhDRp4aCfkoUV0s/640?wx_fmt=jpeg "")  
  
  
