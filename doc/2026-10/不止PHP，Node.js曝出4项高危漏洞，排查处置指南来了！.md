#  不止PHP，Node.js曝出4项高危漏洞，排查处置指南来了！  
攻防实验室
                    攻防实验室  威努特安全网络   2026-10-23 11:58  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/vEkwp3V9UtvBib63wYKkyu30qXmniaEoiatX6yZ1OR362Y0A23BDs3VfKFjMCElB8viaoLEkiakkkJ3ux75IBqRzSGQ/640?wx_fmt=gif&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
  
10月21日，  
PHP与Node.js同日发布安全更新，引发广泛关注。  
  
  
PHP此次修复两项可导致未经身份认证远程代码执行（RCE）的严重漏洞，官方CVSS v4.0评分均为9.2分。部分相关缺陷已存在十余年，AI辅助研究进一步揭示了其实际可利用性。  
  
  
Node.js安全公告列出10项漏洞，其中4项高危、3项中危和3项低危，涉及Brotli解码、TLS证书主机名校验及DNS解析等功能。  
  
  
PHP与Node.js广泛用于Web业务、管理后台和数据服务。  
威努特建议，用户应尽快确认实际运行版本，结合生产连续性要求落实升级加固。  
  
  
**PHP漏洞分析与处置建议**  
  
  
**漏洞概述与成因**  
  
  
本次涉及CVE-2026-109618和CVE-2026-109619。前者与内存耗尽时的异常处理有关，后者与跨请求状态清理不完整有关，均可能引发内存安全问题。  
  
  
提供Web请求服务的PHP部署均应排查，实际可利用性受部署方式、配置、扩展及应用行为影响。已验证的部分Apache mod_php路径可在业务脚本运行前触发，PHP-FPM也需纳入排查和修复范围。  
  
  
**影响范围与修复版本**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4IK3icKUnRFEWuqjN32aSRt2ED3e5hlaSJBwmcu3ickLq8G9nAibBzicvUOt37qUGJvqibibUJB9Pyhic4yiaNXcW2eNzeMmGIO3yLpribKGeXnLbqVw/640?wx_fmt=png&from=appmsg "")  
  
  
PHP8.1及更早分支已结束官方维护，应核查供应商补丁并规划迁移。PHP官方为7.4、8.0、8.1提供了最小化源码回移补丁，供特定维护场景使用；这不代表恢复常规支持。发行商构建的修复状态应以其公告为准。  
  
  
**复现验证**  
  
威努特已完成PHP2Shell漏洞复现。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/4IK3icKUnRFEs31YX6CicSxshckXtQbMx9HVqCtMGynHTu0rwV992mH5ZSBkUtHxKq9UY7IMjfeHJ4hRiavKsBXVwgTZkdW0ZpHtjg2G2V3QbQ/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**处置建议**  
  
  
（一）资产排查  
  
  
梳理使用PHP的Web管理和数据服务资产，结合运行信息、软件清单及供应商公告，核对实际版本、SAPI类型、配置与补丁状态。  
  
  
（二）临时缓解  
  
  
以下措施仅针对部分攻击路径，不能替代安全更新；调整PHP配置后须重启全部相关工作进程并核验生效。  
  
  
跨请求状态问题：在请求解析前生效的服务配置中设置session.upload_progress.enabled=0；PHP 8.5可另设fatal_error_backtraces=0。PHP-FPM可评估pm.max_requests=1，同时关注性能影响。  
  
  
内存耗尽问题：审慎评估memory_limit=-1。该配置取消PHP请求级内存限制，仍无法覆盖底层分配失败，且可能增加资源耗尽问题；采用前须评估资源余量，由管理配置强制设置并确认未被覆盖。  
  
  
请求与访问限制：在前置服务器或代理限制解码后的请求体大小、字段、multipart分段及分段内头部数量，覆盖缺少有效Content-Length的请求，关闭非必要上传入口并收紧管理面访问。  
  
  
（三）根本修复  
  
  
部署包含完整修复的PHP版本或供应商修复包，并重启相关工作进程。涉及生产业务的更新，应先完成兼容性验证、配置备份和回退准备，在维护窗口实施；更新后确认实际服务进程已使用修复版本。  
  
  
**Node.js漏洞分析与处置建议**  
  
  
**漏洞概述及成因**  
  
  
本次公告中的4项高危漏洞如下，受影响分支均指尚未应用相应修复的相关版本。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4IK3icKUnRFGibN0Sibez5qmhnhGOASgQmJibGhtyAE5UD2d4HI2aIAQ7AIvXvoDy0yXHs0U0sUg9cBS6diaX4hXsxPTnG6oYicicOzbQIJ69IYbQ4/640?wx_fmt=png&from=appmsg "")  
  
  
Brotli漏洞涉及压缩数据解码过程中的内存安全缺陷，实际影响取决于应用是否处理来自不可信来源的Brotli数据。两项TLS漏洞涉及地址识别与证书主机名校验的处理差异；DNS漏洞则可能导致应用校验的名称与实际解析名称不一致。具体影响需结合实际调用路径判断。  
  
  
**影响范围与修复版本**  
  
  
本次安全更新的修复基线如下。实际更新应选择相应受支持分支中包含这些修复、并经兼容性验证的版本。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/4IK3icKUnRFGxAwTf6f8aM5ZvktRnW3QClozuibCVia7ZwClsfywwSicVa0VlA8xMfbElWiaJCc6RGq8cJ7OLpP6OLJlZCRc10w5j9PTYibqc0bWE/640?wx_fmt=png&from=appmsg "")  
  
  
各漏洞的受影响分支以官方公告为准。采用发行商回补的软件包，应结合安全公告与包修订号判断修复状态；已结束官方维护的分支仍需排查并规划迁移。  
  
  
**处置建议**  
  
  
核查实际版本。盘点服务、容器及软件内置的Node.js运行环境，结合进程路径核对版本与补丁状态，避免仅凭系统命令行版本号或npm audit结果判断。  
  
  
按部署形态更新。独立服务更新至修复版本并重启；容器应用更新镜像并重新部署；Node-RED先核对兼容性，必要时重建依赖；Electron及设备内嵌组件跟进供应商修复包。  
  
  
收紧数据与访问入口。核查Brotli解码、外部响应及DNS处理路径，在解码前拒绝非必要压缩输入，限制出站访问目标，并保持TLS证书和主机名校验开启。  
  
  
验证修复效果。生产环境先完成兼容性验证、配置备份和回退准备，在维护窗口实施更新；确认各实例已使用修复版本、旧进程已退出，并按业务实际验证接口、TLS连接及关键功能。  
  
  
PHP与Node.js服务于企业不同的业务组件。威努特建议建立覆盖后端运行环境、容器及设备内嵌组件的资产台账，持续开展版本核查与修复验证，减少遗漏，为业务稳定运行提供保障。  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/vEkwp3V9UttO1byVSbuod03z4vTwBZa0QPXxjXLFBAUUpkYOYz78KpuM3Lic13XvZSLwMRqwPWx2RcI41KF0fcw/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=18 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4IK3icKUnRFEJ3tYTuTkgeQT2enboicAQ6togVxGn42DE4ZuHuUoYdsnicz7zvyZiaCEjicmBbrFxbHRLCxbibW23dQ0258gLOVicVpXA3YsiaTGlCY/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=8 "")  
  
  
