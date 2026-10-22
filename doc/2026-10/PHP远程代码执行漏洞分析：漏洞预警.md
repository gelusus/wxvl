#  PHP远程代码执行漏洞分析：漏洞预警  
 墨云安全   2026-10-22 09:13  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/7lCiaSMMkhia4WIkRNZHTwq8jJicy27jdbWa7ED26252RGmSPRE0rmHQsgZ6ZoichVyFNlvhLelZS09a194B9dyoAQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
点击↑  
**蓝字**  
  
关注  
**墨云安全**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC64Jegp0lgLDGafrdvU8MV80FwNQPMbnsrL7uZZicQcmtDkwNu5wflcYicoJ1OSoKLFpcH2wc15vJxw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1 "")  
  
PHP 内核中存在两处可导致远程代码执行的严重漏洞  
（  
CVE-2026-109618  
/CVE-2026-109619  
）。  
漏洞位于 PHP 解释器内核，攻击者无需任何认证，仅发送一条特制 HTTP 请求即可在服务器上执行任意代码  
，  
CVSSv4.0 评分均为 9.2。  
补丁发布前漏洞细节已提前泄露，Cloudflare 在补丁发布后 2 小时内监测到数十万次利用尝试，CISA 同日发出紧急告警。  
  
需要特别说明的是：**“未认证”指的是触发条件无需凭据，不等于“任意 PHP 部署均可无限制 RCE”。**  
 完整代码执行已在 Apache mod_php 环境下公开验证，但依赖特定 SAPI、堆布局控制与 ASLR 处理；PHP-FPM 场景下尚无公开的完整 RCE 演示，实际利用难度因部署形态差异显著。  
  
**漏洞概述**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NcM654OfmXLsfo5yhKRY1adqjza1bhJicQxutAj9BJQ67ibQuVMDN2x1wcKicTxkDbCicaLdnhccDrM34iaJFQu6IWnGxhicz1Lzs3VPrU21PPUN8/640?wx_fmt=png&from=appmsg "")  
  
**影响范围**  
  
PHP 8.5	< 8.5.12	  
  
PHP 8.4	< 8.4.27	  
  
PHP 8.3	< 8.3.36	  
  
PHP 8.2	< 8.2.35	  
  
PHP 8.1 及更早的全部版本  
  
**漏洞原理**  
  
CVE-2026-109618：OOM 处理中的内存状态破坏  
  
PHP 在处理内存耗尽（OOM）错误时，会通过 longjmp()  
 直接退出当前上下文。问题在于：在某些上下文中，请求相关的内存在完全初始化之前就已处于可访问状态。若此时发生内存分配失败，PHP 未能正确处理提前退出，导致请求内存处于无效状态。  
  
攻击者需构造包含大量请求头、嵌套字段名或空文件控件的 multipart 请求，在**请求体大小不超过 post_max_size**  
 的前提下，使解析阶段的内存消耗超过 memory_limit  
，从而触发 OOM。这里的关键前提是**解析开销与请求体大小之间的不对称**  
：请求体本身可能只有几 MB，但解析其头部结构可消耗数百 MB 内存。  
  
该路径的完整 RCE 已在 **Apache mod_php + prefork MPM**  
 环境下被公开验证。在 mod_php 下，Apache 会无条件调用请求关闭流程，关闭过程会处理 OOM 留下的无效状态，最终导致内存破坏和代码执行。**PHP-FPM 从源码分析可能受同一缺陷影响，但公开研究中尚未演示完整的 FPM RCE 链**  
；LSAPI 是“可能但未验证”。此外，成功利用通常需要处理 ASLR 或获取与目标构建匹配的偏移量，公开 PoC 多针对特定 PHP 版本和构建环境。CyCognito 确认，该漏洞的完整 RCE 验证针对的是 mod_php + prefork + 默认内存限制，不是所有部署形态。  
### CVE-2026-109619：残留全局状态导致释放后重用  
  
PHP 工作进程会保留指向上一个请求内存的全局状态（如错误、会话、对象和扩展状态），但在请求内存释放前未将这些状态完全重置。同一工作进程处理下一个请求时，若部分输入解析、配置、会话或错误处理逻辑在请求正常初始化之前运行，就会访问这些残留的陈旧状态，造成释放后使用（Use-After-Free）。  
  
利用此漏洞需要**两个请求由同一个 worker 先后处理**  
，分别充当“生产者”与“消费者”。已确认的其中一条路径依赖  
 session.upload_progress  
：第一个 multipart 请求触发 OOM 或异常退出，留下指向已释放内存的全局状态；第二个请求在 RINIT 重置该状态之前，由上传进度回调再次访问该指针。因此，**session.upload_progress.enabled = On（PHP 默认值）是这条路径的前提**  
。另一条路径依赖 fatal_error_backtraces  
 开启。官方已确认该路径，同时还有与 Fatal Error 等相关的其他路径在研究中。在 FPM 场景下，PHP 官方明确指出利用“取决于合适的应用操作”（depends on a suitable application operation）。  
  
需要说明的是，有一个公开仓库声称对 session.upload_progress  
   
UAF 机制进行了源码审计和 ASAN 复现，结论是该 UAF 在已发布的 PHP 中**不存在**  
，Z_TRY_ADDREF  
 使 progress 结构体持有自己的引用，track_init()  
 释放的是反序列化副本而非活跃数组。这个结论与官方通告和多数 PoC 仓库矛盾，但至少说明该漏洞的利用细节在社区中仍有争议，不能简单写成“确定可利用”。  
  
**攻击链路分析**  
  
整  
体利用路径可概括为：  
  
构造恶意 multipart POST 请求  
  
      
→ 触发 OOM 或残留全局状态  
  
      
→ PHP 内核内存状态异常（内存破坏 / UAF）  
  
      
→ [前提：特定 SAPI + 堆布局控制 + ASLR 处理]  
  
      
→ 以 Web 工作进程权限执行任意代码  
  
      
→ 植入后门 / 横向移动 / 数据窃取  
  
**针对特定条件下的靶场的漏洞复现**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/NcM654OfmXKKh5WDxJXzfFK0UmWWojE3qKzhPWsyicwJ9rnvBKN9rpreaoSNuTfJX48hrReprjO5iaaofSGezVw72iao7CXCDbiaXAIRzhoG3s8/640?wx_fmt=jpeg&from=appmsg "")  
  
**破坏性分析**  
  
该漏  
洞的 CVSSv4.0 评分  
为  
 9.2  
，属于  
严重  
级别  
，其破坏性体现在以下几个维度  
：  
  
认证门槛为零，但利用门槛不为零：  
无需认证或用户交互即可  
触发内存破坏条件  
，但实现代码执行需要满足部署形态（mod_php 已验证，FPM 未演示完整 RCE）、堆布局控制和 ASLR 处理等条  
件  
。  
在 PHP-FPM 等 SAPI 下，Red Hat 评估认为“远程代码执行在实践中是困难的”。  
  
内核层触发：  
漏洞在 PHP 引擎内部、应用代码运行之前触发，业务层安全机制全部失效  
。  
  
**全生态波及：WordPress、Drupal、Joomla、Laravel、Magento、MediaWiki、phpMyAdmin、Nextcloud 等主流 CMS 和框架均在受影响范围内**  
。  
  
**在野利用活跃度需审慎看待：补丁发布前细节已泄露，Cloudflare 在补丁发布后 2 小时内监测到数十万次利用尝试；但 TrendAI 指出“成功利用在野尚未被独立证实”，CyCognito 称“截至撰写时没有确认的大规模入侵报告”。大量“利用尝试”不等于大量成功 RCE**  
。  
  
**修复建议**  
  
官方安全更新  
  
唯一彻底的修复手段是升级至对应分支的安全版本，并重启所有 PHP 工作进程（仅替换二进制文件不足以消除风险）：  
  
PHP 8.5  
：  
升级至   
8.5.12  
。  
  
PHP 8.4：  
升级至  
 8.4.27  
。  
  
PHP 8.3  
：  
升级至  
 8.3.36  
。****  
  
**PHP 8.2：升级至 8.2.35。**  
  
PHP 8.1 及更早的版本：  
官方已终止支持，建议升级至受支持版本；短期内无法升级的应实施网络隔离，或联系发行版/LTS 厂商获取回溯补丁   
。  
### 临时缓解措施  
### PHP 官方明确表示没有完整的临时缓解方案，以下措施仅能降低风险，不能替代升级。  
  
关闭会话上传进度：  
在 php.ini 或 PHP-FPM 池配置中设置   
session.upload_progress.enabled = Off  
。需特别注意，该配置在 mod_php 下无效，而 mod_php 恰是本次风险最高的部署形态；mod_php 用户不能依赖此项缓解，应优先升级或下线暴露面。此外，该措施仅针对 CVE-2026-109619。  
  
限制请求体大小：  
按业务需要下调   
post_max_size、max_file_uploads  
，并在 Web 服务器层限制请求体上限。  
  
WAF 虚拟补丁：  
拦截请求体含   
PHP_SESSION_UPLOAD_PROGRESS  
 的 multipart 请求，并限制字段名长度与 multipart 分段数。该措施作用于流量层，可拦截已知请求特征，但无法覆盖所有变形 payload，属缓解而非根治。  
  
![图片](https://mmbiz.qpic.cn/mmbiz/cZV2hRpuAPgyGRhyoqbTupN7lM2NSVJqkaFQzA59F6kiblIQsL175lxIVZbSLrFDFicibxXiaXpXmAGkrGNSib76Ylw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=3 "")  
  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/asLg7via5ibAkf1mRkpS4IuZibZE5eeC0t8nibIZBfZEekibOEZVWyf9jHzIVvT2sTzKS1OtZzSBErxJUZXD1AwAAWw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=4 "")  
  
往期回顾  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/icjDF5uGXY5ibE0P0Mtzns3KNb5hsCIKPfMIRultHDbmzgJcDaibI4wNKM6ZloyGRtRovyXtVdv3SuuVOcmA8gn8A/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=5 "")  
  
# AI红客机器人重磅更新，全面覆盖OpenClaw安全检测  
# 墨云科技AI红客机器人亮相香港世界青年科学大会  
# Black Hat Asia 2026 | 墨云AI红客团队闪耀亮相  
  
  
让网络攻防更智能  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/CD1iaLIMEhibPv9rc3gdLj3g6fiaAcCZqIicylIMVKlbvd5ic5usJ2oia9cTgavs6BwQpEEYbfglc82kCJ0Qic3OHMEaw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=6 "")  
  
  
  
点击  
**转发**  
  
分享给小伙伴  
  
  
