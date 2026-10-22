#  PHP「php2shell」高危漏洞复现验证与工控场景风险分析  
原创 攻防实验室
                    攻防实验室  威努特安全网络   2026-10-22 10:30  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/vEkwp3V9UtvBib63wYKkyu30qXmniaEoiatX6yZ1OR362Y0A23BDs3VfKFjMCElB8viaoLEkiakkkJ3ux75IBqRzSGQ/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
  
10月21日，全球广泛使用的服务器端编程语言PHP曝出两项严重安全漏洞（CVE-2026-109618、CVE-2026-109619），社区统称为  
 php2shell  
。两项漏洞的CVSS v4.0评分均为9.2分。攻击者在特定条件下无需身份认证，即可在受影响服务器上远程执行代码，可能造成敏感数据泄露、网站篡改等后果。  
  
  
此次漏洞发现的背后，AI发挥了关键作用。据OpenAI披露，安全研究公司Calif借助GPT-5.6 Sol识别了关键漏洞；Wiz也独立发现并上报了两项漏洞，并在研究过程中使用AI工具开展漏洞发现、利用验证和补丁验证。Wiz指出，相关缺陷已在PHP代码中存在十余年，AI显著降低了分析复杂漏洞、验证其可利用性的门槛。  
  
  
此次事件受到高度关注，与PHP在全球Web生态中的广泛应用密切相关。WordPress、Drupal、Joomla等内容管理系统均以PHP为基础，大量商业与开源软件也依赖PHP运行。由于漏洞位于PHP核心层，即使上层应用代码没有安全缺陷，底层运行环境仍可能成为攻击入口。  
  
  
风险披露后，多方网络安全机构迅速发出警告。英国国家网络安全中心敦促相关组织立即采取行动；欧盟委员会、欧盟网络安全局、CERT-EU等机构发布联合声明，要求优先排查面向互联网的系统并安装安全更新。CERT-EU进一步建议，对无法及时修复的相关应用，考虑暂时与公网隔离，直至修复并验证完成。  
  
  
威努特在隔离实验环境中完成了完整验证：在无凭据、无应用层缺陷的前提下，通过标准HTTP请求实现远程命令执行，连续三次复现成功。  
  
  
鉴于该类漏洞的触发位置位于应用代码执行之前，其影响范围不局限于特定 CMS，对采用 PHP 构建 Web 管理界面的工控设备同样构成现实威胁。现将验证过程与风险研判通报如下。  
  
  
**漏洞概述**  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4IK3icKUnRFHAVwr45wZ4s3VY90x9dVVEOAjhxm8SeZkQPTM6zfCkxmV1zkA5QzvTFWIyevCdplzneqzU98pkuybFFic3JqcCIMfPzQTfxz0w/640?wx_fmt=png&from=appmsg "")  
  
  
该漏洞的核心特征在于触发时点早于业务代码。PHP 在处理请求体解析阶段即可能因内存耗尽而中断，此时应用脚本尚未开始执行。因此，即使 Web 应用本身不存在任何缺陷，也无法规避该风险。  
  
  
  
**影响范围与修复版本**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4IK3icKUnRFEUY1qqnWpQDq0VLjGicK1JJxHicz6Rp4SUcN09yGkcmSo2aFpUDN0xum6T5cDmWclcnPnE2CR5iawKbClSztWn8acNJbuibciaB9NY/640?wx_fmt=png&from=appmsg "")  
  
  
公开研究认为该缺陷可追溯至 PHP 5.4，覆盖近十年发布版本。对于 7.4 / 8.0 / 8.1 等已停止常规支持的版本，官方仅提供最小化源码补丁分支，不具备工程可维护性。  
  
  
**技术成因简析**  
  
  
  
PHP 在请求处理过程中，若发生内存分配失败，会通过 zend_bailout() 的 longjmp 机制中断当前流程。该机制绕过了中断点之后尚未完成的初始化步骤，而请求清理阶段（request shutdown）在 mod_php 部署下仍会执行。由此产生两类独立缺陷：  
  
  
CVE-2026-109618（单请求内）：中断后请求内存可能处于未完成初始化状态，而清理代码将其视为有效对象进行访问，导致内存破坏。  
  
  
CVE-2026-109619（跨请求）：worker 进程的全局状态（会话、错误回溯等）保存了指向当前请求堆的指针。正常流程应先重置全局状态再释放内存；若中途发生致命错误或内存耗尽中断，重置被跳过而内存照常释放，指针随即悬空。同一 worker 处理的下一个请求在初始化完成前访问该指针，构成释放后重用。  
  
  
需要指出的是，该漏洞并非"上传文件即可执行命令"的简单利用模式。其成立依赖请求解析、异常退出、资源清理与 worker 复用等环节的时序配合，因而在不同部署方式下利用难度差异显著。据公开信息，Apache mod_php 部署已被证实可实现完整利用；PHP-FPM 部署的限制条件较多，尚未见到完整利用链。  
  
**复现验证**  
  
  
  
**验证环境**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4IK3icKUnRFFHSKeRLJuBKicUGUq5CgibZUVruMkQxbQqOLd1d9uEE6hiaIpeS4BOP2JTia24DOe1BYtU87dZiaNMibgflIydDu7hLkgViaNia8Zwyd4/640?wx_fmt=png&from=appmsg "")  
  
  
**验证过程**  
  
  
**第一步：验证触发时点**  
  
****  
在发行版自带 mod_php（PHP 8.4.24）上先行验证引擎层行为。构造单个 part 携带大量头部的 multipart 请求，并设置较小内存上限，使解析过程中发生内存耗尽。错误日志记录如下：  
  
  
PHP Fatal error: Allowed memory size of 8388608 bytes exhausted  
  
                (tried to allocate ...) in Unknown on line 0  
  
  
in Unknown on line 0 表明致命错误发生于请求启动阶段，早于任何脚本代码执行；同时该 worker 进程存活并继续响应后续请求，证实跨请求状态残留的载体客观存在。经测试，8M 限制下的触发阈值约为 1.6 万至 3.2 万条请求头。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4IK3icKUnRFGtZ4IdOK2FFf5Pg8ZeNtLoePGO90icS5jxC3eGXQogIEvnDvjWlkMAbJwNgjyeRuVuYj2mhoHicIJgicibpNwJTLdQU6mNvzvqnZM/640?wx_fmt=png&from=appmsg "")  
  
  
**第二步：完整利用链验证**  
  
****  
在官方 8.4.26 镜像上运行公开研究的三阶段利用链，全程仅使用 HTTP 请求：  
  
  
1. worker 轮换：以大字段 POST 触发中断，使子进程异常退出，由 prefork 派生的新 worker 继承相同内存映射；  
  
  
2. 堆地址泄露：借助输入变量数量告警使响应头提前发出，再在会话初始化路径中触发内存耗尽，遗留悬空指针；后续请求重占该内存区域，通过 URL 编码越界读取，将内存管理指针写入响应 Cookie；  
  
  
3. 模块基址泄露：由析构函数指针反推 libphp 加载基址，完成地址随机化绕过；  
  
  
4. 命令执行：将析构函数指针改写为系统命令函数，沿对象析构路径触发，全程仅依赖 libphp 内部函数，不引入 libc 依赖。  
  
  
其中，偏移标定需从目标容器内 libphp.so 的 ELF 符号表读取。首次验证时，因标定脚本输出目录权限异常而静默失败，利用链退回内置参考值，导致基址泄露环节连续三次未命中——现象上表现为"利用不稳定"，实为标定参数错误。修正后一次命中。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4IK3icKUnRFHcCZr6b4cWFB9rwJadEhkaDiayibxwsxSic4kgZb31zibQ8w0XiaaMC0D6u16V97X0XjWI6kfbdfZDicE7gpZTlkae8qEngddmB5LvI/640?wx_fmt=png&from=appmsg "")  
  
  
  
**工控场景风险分析**  
  
  
  
**PHP 在工控 Web 管理面具有实际存量**  
  
  
网关、DTU、串口服务器、远程运维通道、组态软件 Web 端及部分 HMI 配置界面，普遍采用 PHP 实现管理功能。此类界面多为设备厂商自研，缺乏与通用 CMS 相当的漏洞响应机制与版本跟踪能力。  
  
**补丁可达性差，暴露周期长**  
  
  
工控设备升级通常涉及停机窗口、现场配合与厂商排期，实际生命周期内升级次数有限。叠加 PHP 官方对 8.1 及更早版本仅提供源码级 backport 分支，设备厂商普遍不具备跟进条件，导致受影响资产长期处于暴露状态。  
  
  
**绕过应用层安全假设**  
  
  
漏洞位于引擎路径，厂商对自有管理程序代码的审计与加固无法规避风险，除非升级 PHP 运行时本身。这对"应用代码安全即整体安全"的传统认知构成直接挑战。  
  
**可用性风险与 RCE 风险并存**  
  
  
利用过程需制造内存耗尽并导致 worker 中止重建。在单 worker 部署下，表现为 Web 服务进程反复异常退出。对于连续性生产场景，即便未达成代码执行，亦可造成管理面不可用。  
  
  
**处置建议**  
  
  
  
（一）资产排查  
  
  
梳理全部具备 Web 管理界面的工控资产，确认其 PHP 版本与 SAPI 类型。对可获取固件的设备，建议直接解包定位 libphp.so 或 php-cgi，读取版本字符串与编译参数——该方式较远程指纹识别更为准确，亦是工控安全团队相对传统 IT 团队的检测优势。  
  
  
（二）临时缓解  
  
- 关闭会话上传进度功能（session.upload_progress.enabled=0），可切断已验证的中途会话生命周期路径；  
  
- 对 memory_limit=-1 的使用需审慎评估：该配置虽可阻断触发条件，但会移除内存限制，在内存资源紧张的嵌入式设备上可能引入新的拒绝服务风险；  
  
- 在设备前端部署反向代理或 WAF，限制 multipart 分段数、请求头数量与请求体大小；  
  
- 条件允许时设置 pm.max_requests=1，使 worker 处理完毕即退出，切断跨请求状态复用；  
  
- 管理界面收敛至运维网段或 VPN，禁止直接暴露于互联网。  
  
（三）根本修复  
  
  
升级至 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 或更高版本。官方明确表示，仅升级可完整消除风险，其余措施均为缓解手段。  
  
  
为避免您的业务受影响，威努特建议您立即排查业务系统及相关软件组件是否使用PHP，如在受影响范围，请您及时进行更新修复，避免被外部攻击者入侵。  
  
  
  
**参考资料**  
  
  
- PHP 安全公告：  
https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
；  
https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
  
- PHP 安全版本发布：  
https://www.php.net/archive/2026.php  
  
- 披露邮件：  
https://www.openwall.com/lists/oss-security/2026/10/21/6  
  
- 公开技术研究：  
https://www.wiz.io/blog/php2shell-php-rce  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/vEkwp3V9UttO1byVSbuod03z4vTwBZa0QPXxjXLFBAUUpkYOYz78KpuM3Lic13XvZSLwMRqwPWx2RcI41KF0fcw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=18 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4IK3icKUnRFEJ3tYTuTkgeQT2enboicAQ6togVxGn42DE4ZuHuUoYdsnicz7zvyZiaCEjicmBbrFxbHRLCxbibW23dQ0258gLOVicVpXA3YsiaTGlCY/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
