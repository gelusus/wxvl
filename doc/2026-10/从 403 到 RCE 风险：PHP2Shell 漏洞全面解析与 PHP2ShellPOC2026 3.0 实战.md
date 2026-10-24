#  从 403 到 RCE 风险：PHP2Shell 漏洞全面解析与 PHP2ShellPOC2026 3.0 实战  
原创 钟智强
                    钟智强  哪吒网络安全   2026-10-24 17:13  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ia7TorzX5Pa3wlehhfQWaF8Z0mXuQWia9GWIjiakNte5WPcNwx6tYFEgI4TzkLaZib6raKM5G7I2tuUpBSCiabfW9VmLfvicATSuNtqYw0KHgKSxs/640?wx_fmt=jpeg "")  
  
  
一次真实评估的可视化——左侧终端里的   
HTTP/1.1 403 FORBIDDEN  
，经证据推导后指向右侧的   
PHP 7.3.33 END-OF-LIFE  
 与   
SEVERITY 10.0/10  
；背景的六边形墙即 WAF/边缘，底部为 MITRE ATT&CK 杀伤链节点。  
  
本文所有命令行输出均来自 Terminal 实测  
（目标已统一脱敏为   
www.example.com  
），未做任何修饰性删改。  
  
测试日期  
：2026-10-24              
  
工具版本  
：PHP2ShellPOC2026 3.0.0              
  
测试对象  
：某授权评估目标（本文统一脱敏为   
https://www.example.com/  
）              
  
扫描结论  
：  
YES  
，PHP 7.3.33，EOL，暴露严重度 10.0/10，置信度低              
  
POC 结论  
：  
RESULT: FAILED  
，堆布局阶段 3/3 失败，未观察到证明文件              
  
重要提醒  
：POC 失败不等于目标安全；本文仅用于授权安全评估与防御研究  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa0t0KrojHzaQaDG6odTYRl41BGFB92eSkmR1Th4Ng1QwnCxj0YyplM4vicibib9TqJqJbWCf0HMEJJPaDG8iav9ayYwt5raiaHpV5iag/640?wx_fmt=png "")  
## 0. 写在前面：授权、脱敏与免责声明  
  
这不是一篇“拿到域名就能打”的文章，而是一篇  
授权评估报告解读 + 漏洞方法论 + 防御手册  
。  
  
本文涉及的工具会发送真实 HTTP 请求，POC 还会发送大体积 multipart 请求、触发内存耗尽和工作进程轮换。它只适用于以下场景：  
  
·  
你拥有目标系统，或持有明确的书面授权；  
  
·  
授权范围覆盖目标域名、端口、路径和测试时间窗；  
  
·  
你已阅读并遵守项目范围、交战规则（RoE）和速率限制；  
  
·  
你理解 POC 可能中断目标服务，并愿意承担相应后果。  
  
不要  
对军事、政府、关键基础设施，或任何未授权的系统使用本文提到的工具和 POC。未经授权访问计算机系统在大多数司法辖区都是违法行为。  
  
本文中的目标域名、命令和输出均已脱敏为   
www.example.com  
。请不要把示例域名替换成真实目标后直接执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa0vT6XxYDxn2tyB1I8f980tQM9mAq7bRIfVdFQabIHzxAibDuFqHiaJFKAqxbN6ryGLKTOZjbUWYuW1SicN0lfelAe64Yg52poyhM/640?wx_fmt=png "")  
## 1. 一分钟结论  
  
先给结论，再展开细节。  
<table><thead><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">维度</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">本次结果</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">说明</span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描判定</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">YES / HACKABLE</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">目标存在 PHP2Shell 暴露面</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">漏洞状态</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">END-OF-LIFE</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 7.3 分支已停止维护，没有上游修复</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 版本</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">7.3.33</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">从拦截响应头 </span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style=""><span leaf="">X-Powered-By</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> 观察到</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">Web 服务器</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">LiteSpeed</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">SAPI 也识别为 </span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style=""><span leaf="">litespeed</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">访问状态</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">BLOCKED（HTTP 403）</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">根路径和随机控制路径全部被边缘拒绝</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">置信度</span></span></span></span><o:page></o:page></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">低</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">版本来自 403 拦截页，不是正常应用响应</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">暴露评分</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">10.0/10（CRITICAL）</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">这是暴露严重度，不是完整 CVSS 向量</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">利用阻力</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">高</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">WAF/边缘拦截 + 无法确认应用层行为</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">证据比率</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">1.00</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">正向信号 8，负向信号 0，中性信号 0</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">POC 结果</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">FAILED</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">堆泄露 3/3 失败，未进入库基址和命令投递阶段</span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">建议动作</span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">升级 PHP + 隔离 + 验证 + 监测</span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:一分钟结论;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">不要因为 POC 失败就关闭问题</span></span></span></span></p></td></tr></tbody></table>  
一句话版本：  
  
扫描器看到的是一个“被边缘挡住、但仍泄露了 EOL PHP 版本”的高风险目标；POC 因为版本/构建不匹配且应用不可达而失败。正确结论是“高风险且未验证”，不是“安全”。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa3lHLicoicqNEFTNGYhGWaib9M9fXR6urCJlZXtZnKubz2y2H0H0DickVQkUuhF4RR2G1JykmVeIzzhgBYDiaTJ4xD9JEK4wVG44QgY/640?wx_fmt=png "")  
## 2. 为什么 403 不是“安全”  
  
这是整篇文章最重要的认知前提。  
  
很多扫描器会把“所有请求都返回 403”直接归类为“不可利用”或“安全”。这是错误的。403 只说明  
扫描器没有通过边缘控制  
，它没有说明：  
  
·  
应用是否存在；  
  
·  
PHP 是否存在；  
  
·  
PHP 版本是否落后；  
  
·  
攻击者从另一个出口是否能够到达应用；  
  
·  
边缘规则是否覆盖了真正的攻击请求。  
### 2.1 2.x 的失败模式：把“没有证据”当成“没有漏洞”  
  
PHP2ShellPOC2026 2.x 曾经有一个危险的失败模式：  
  
根路径 403  
  
随机路径 403  
  
→ 报告：不可被入侵，证据比率 0.00  
  
这里的   
0.00  
 暗示“有负面证据”，但实际上  
一条有效证据都没有  
。这会把一个被 WAF 挡住的目标，误判成“已修复”。  
  
3.0 改变了这个问题：  
没有证据时，比率是   
null  
，结论是   
INCONCLUSIVE  
，而不是   
NOT EXPOSED  
。  
### 2.2 控制探针：先判断“能不能看见应用”  
  
扫描器不再只请求根路径，而是额外请求随机、几乎不可能存在的路径：  
  
/php2shell-<8位随机十六进制>.txt  
  
/php2shell-<8位随机十六进制>.php  
  
然后比较根路径和控制路径的响应：  
<table><thead><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">根路径</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">控制路径</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">分类</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">含义</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">403</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">403</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">BLOCKED</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">边缘在应用之前拒绝了扫描器</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">403</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">404</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">RESTRICTED</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">路径级访问控制，不是全站拦截</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">200</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">200</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">，正文几乎相同</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">SOFT 404</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">全捕获路由，路径状态码不可信</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">302</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> 到登录页</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">任意</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">AUTH WALL</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 可能存在，但被认证挡住</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">429</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">任意</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">RATE LIMITED</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">限速，不是“不存在”</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">5xx</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">任意</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">SERVER ERROR</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">应用不健康，不是补丁结论</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">连接错误</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">连接错误</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">UNREACHABLE</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">没有可用应用响应</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">200</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style=""><span leaf="">404</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;font-weight:bold;mso-bidi-font-weight:bold;"><span leaf="">OPEN</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:为什么-403-不是安全;"><span style="mso-bookmark:控制探针先判断能不能看见应用;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">路径状态码可以正常解读</span></span></span></span></span></p></td></tr></tbody></table>  
本次测试的结果就是第一行：  
  
/  
                           
403  
  
/php2shell-xxxx.txt  
         
403  
  
/php2shell-xxxx.php  
         
403  
  
三个响应大小都约为 30.3 KB，耗时约 900 ms，服务器都是 LiteSpeed。控制路径没有返回 404，而是和根路径一样返回 403，这说明  
边缘在做全站拦截  
，而不是“某个路径不存在”。  
### 2.3 用一张图看懂访问分类  
  
flowchart TD  
  
      
A[GET 根路径 /] --> B[GET 随机控制路径                  
  
.txt 和 .php]  
  
      
B --> C{根路径与控制路径的状态}  
  
      
C -->|都是 401/403/406/451| D[BLOCKED                  
  
边缘拒绝，应用不可见]  
  
      
C -->|根路径被拒，控制路径 404| E[RESTRICTED                  
  
路径级访问控制]  
  
      
C -->|控制路径 2xx/3xx 且正文近似| F[SOFT 404                  
  
全捕获路由]  
  
      
C -->|根路径跳转登录页| G[AUTH WALL                  
  
认证墙]  
  
      
C -->|429| H[RATE LIMITED                  
  
限速]  
  
      
C -->|5xx| I[SERVER ERROR                  
  
服务不健康]  
  
      
C -->|连接失败| J[UNREACHABLE                  
  
不可达]  
  
      
C -->|根路径 2xx，控制路径 404| K[OPEN                  
  
可以继续探测]  
  
      
D --> L[自适应停止                  
  
报告访问证据]  
  
      
E --> L  
  
      
F --> L  
  
      
G --> L  
  
      
H --> L  
  
      
I --> L  
  
      
J --> L  
  
      
K --> M[继续探测 PHP 路径                  
  
提取版本并做结论]  
### 2.4 拦截页也是证据  
  
3.0 还会识别常见拦截页：  
  
·  
OVHcloud  
  
·  
Cloudflare  
  
·  
Akamai  
  
·  
Sucuri  
  
·  
Imperva  
  
·  
AWS WAF  
  
·  
F5  
  
·  
Wordfence  
  
·  
ModSecurity  
  
本次报告里的   
Access Denied  
 和   
LiteSpeed  
 就是边缘证据。它不能证明 PHP 已修复，但能解释“为什么应用层看不到”。  
  
如果拦截很可能与 User-Agent 有关，报告会提示可以显式使用   
--browser-ua  
 重试。但这个选项  
永远不会自动开启  
，因为它可能跨越机器人控制边界，必须由操作者确认授权。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa3RkzaFFFiakjVQ5oX1zZN4ibbgt9lGJUb10qmy6sDJKgkvAmBcicdNAyuoCTI0guR2UJznq8Y9qQdXlQHbLDKib436RK5RQ4ZCvnc/640?wx_fmt=png "")  
## 3. PHP2Shell 漏洞全景  
### 3.1 两个 CVE，两个可观测的利用面  
  
PHP2Shell 这个漏洞类别在本项目中对应两个 CVE：  
  
·  
CVE-2026-109618  
  
·  
CVE-2026-109619  
  
从防御和检测角度，可以把它们拆成两个可观测的利用面：  
<table><thead><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">CVE</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">利用面</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">关键特征</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">防御重点</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">CVE-2026-109618</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">请求启动阶段的内存耗尽 / OOM 触发面</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">不需要特殊请求字段，普通可达的 PHP 端点即可成为入口</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">限制解码后请求体大小、multipart 部件数、字段数、头部数量；监测 “Allowed memory size exhausted”</span></span></span></span></span></p></td></tr><tr><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">CVE-2026-109619</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">upload-progress / multipart 路径</span></span></span></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">请求体包含 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style=""><span leaf="">PHP_SESSION_UPLOAD_PROGRESS</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">，并携带 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style=""><span leaf="">PHPSESSID</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"></span></span></p></td><td data-colwidth="148" width="148" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">WAF 检查 multipart body；限制上传进度功能；关闭不必要的 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:两个-cve两个可观测的利用面;"><span style=""><span leaf="">session.upload_progress.enabled</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"></span></span></p></td></tr></tbody></table>  
两者组合后，攻击链可以达到：  
  
·  
以 Web 工作进程权限执行命令（  
T1059.004  
）；  
  
·  
投放 Web Shell 并持久化（  
T1505.003  
）；  
  
·  
通过内存泄露读取环境变量、密钥、会话材料（  
T1005  
）；  
  
·  
通过正常 HTTPS 出站流量进行 C2 和外传（  
T1071.001  
）。  
  
注意：本文不会提供可直接复用的利用代码。POC 仅作为授权验证流程的一部分，用于确认“代码执行是否发生”，而不是用于扩大访问、持久化或数据收集。  
### 3.2 受影响版本与修复下限  
  
本项目使用的修复下限如下：  
<table><thead><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">分支</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">首个修复版本</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 8.2</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.2.35</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 8.3</span></span></span></span></span><o:page></o:page></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.3.36</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 8.4</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.4.27</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 8.5</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shell-漏洞全景;"><span style="mso-bookmark:受影响版本与修复下限;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.5.12</span></span></span></span></span></p></td></tr></tbody></table>  
以下分支被视为   
END-OF-LIFE  
：  
  
·  
PHP 5.x  
  
·  
PHP 7.x  
  
·  
PHP 8.0  
  
·  
PHP 8.1  
  
原因很直接：这些分支没有包含该漏洞修复的上游补丁。对于 7.3.33 这样的版本，  
不存在“再等一个小版本”这种选项  
，只能迁移、使用发行版 backport，或者用补偿控制拖时间。  
  
比修复表更新的分支会被报告为   
unknown / newer_than_table  
，工具不会猜测。  
### 3.3 为什么 PHP 7.3.33 是“确定的高风险”  
  
PHP 7.3 已经 EOL。EOL 的含义不是“最后一个版本有漏洞”，而是：  
  
·  
上游不再为该分支发布安全修复；  
  
·  
新发现的漏洞不会得到官方补丁；  
  
·  
发行版 backport 可能滞后、缺失，或者只覆盖部分 CVE；  
  
·  
资产清单里会持续出现“无法通过常规升级修复”的例外。  
  
因此扫描器把 7.3.33 判为   
END-OF-LIFE  
，并给出   
10.0/10  
 的暴露严重度。这个分数衡量的是“解释器暴露有多严重”，不是“攻击者一定能打进来”。  
### 3.4 攻击链与 MITRE ATT&CK 映射  
  
报告把“评估路径”和“杀伤链”分开：  
  
·  
评估路径  
：本次扫描做了哪些低风险侦察（  
T1595  
、  
T1592  
）；  
  
·  
杀伤链  
：只有在确认解释器暴露时才打印（  
T1190  
、  
T1059.004  
、  
T1505.003  
、  
T1005  
、  
T1082  
、  
T1071.001  
）。  
  
这样做的原因是：给一个未确认的发现打印完整杀伤链，会训练读者忽略它。  
  
flowchart LR  
  
      
A[T1595                  
  
主动扫描] --> B[T1592                  
  
主机信息收集]  
  
      
B --> C[T1190                  
  
利用公网应用]  
  
      
C --> D[T1059.004                  
  
Unix Shell 执行]  
  
      
D --> E[T1505.003                  
  
Web Shell 持久化]  
  
      
C --> F[T1005                  
  
本地数据收集]  
  
      
D --> G[T1082                  
  
系统信息发现]  
  
      
E --> H[T1071.001                  
  
Web 协议 C2]  
  
逐项解释：  
  
·  
T1595 / T1592  
：普通 GET 请求、响应头、TLS 证书、服务器指纹，都是被动或低交互侦察。扫描本身会出现在 Web 日志里。  
  
·  
T1190  
：一个精心构造的 HTTP 请求在应用代码之前到达 PHP 解析器。应用层认证、IAM、SSO 都挡不住解析器层的问题。  
  
·  
T1059.004  
：确认可利用时，攻击者可以以   
www-data  
、  
apache  
、  
nginx  
 等工作进程身份执行命令。  
  
·  
T1505.003  
：如果工作进程长生命周期（如 mod_php/FPM），Web Shell 可以驻留，且不一定导致进程崩溃。  
  
·  
T1005  
：内存泄露变体可以读取进程内存中的环境变量、云密钥、会话材料，不需要先拿到命令执行。  
  
·  
T1082  
：被劫持的解释器做侦察时，流量看起来像普通应用请求，EDR 不一定能区分。  
  
·  
T1071.001  
：C2 和外传混在 Web 层的正常 HTTPS 出站流量里。  
### 3.5 WAF 不等于补丁  
  
这是另一个常见误区：  
  
“我们有 WAF，所以 PHP 版本旧一点没关系。”  
  
WAF 改变的是  
利用阻力和可靠性  
，不是  
解释器是否暴露  
。一个未打补丁的 PHP 解析器，即使前面有 WAF，仍然是未打补丁的解析器。攻击者可以：  
  
·  
寻找 WAF 规则覆盖不到的分段、编码、字段名；  
  
·  
从源站 IP 绕过 CDN/WAF；  
  
·  
利用 WAF 只检查部分请求方法的缺陷；  
  
·  
等待规则更新前的窗口期。  
  
所以报告把两件事分开：  
  
·  
risk.score  
：暴露严重度（解释器是否脆弱）；  
  
·  
risk.friction  
：利用阻力（WAF、CDN、FPM 亲和性、认证墙、限速、边缘拦截、软 404）。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa2Y2iaoVK4BfIbwOUsuzKeEUVZObrRYADTeHB9siajkUUd2MV9I3Hq4tMQ4Yibzheia15qlxuJeKuXkGlK7GH7eObNwP2jlzoYbCBk/640?wx_fmt=png "")  
## 4. PHP2ShellPOC2026 3.0：它到底是什么  
### 4.1 定位  
  
PHP2ShellPOC2026 不是传统的“漏洞利用框架”，而是一个  
暴露面排查与决策工具  
：  
  
1.  
实际响应的是什么？（PHP 版本、SAPI、边缘、CDN、WAF、框架）  
  
2.  
解释器是否暴露？（低于修复下限、EOL、已修复、未验证）  
  
3.  
部署形态如何改变结论？（暴露严重度 vs 利用阻力）  
  
4.  
下一步做什么？（ATT&CK 映射、检测建议、修复建议、可复制命令）  
  
默认文本报告刻意保持简短，先回答“  
这台主机能不能被 PHP2Shell 入侵：YES / NO / UNKNOWN  
”，再展开证据和动作。  
### 4.2 安装  
  
RubyGems：  
  
gem  
 install php2shell_poc2026  
  
php2shell  
  
--version  
  
php2shell-poc2026  
  
--version  
  
从源码安装：  
  
git  
 clone https://github.com/ctkqiang/PHP2ShellPOC2026.git  
  
cd  
 PHP2ShellPOC2026  
  
./install.sh  
  
--user  
  
Docker：  
  
docker  
 build   
-t  
 php2shell-poc2026 .  
  
docker  
 run   
--rm  
 php2shell-poc2026   
--version  
  
docker  
 run   
--rm  
 php2shell-poc2026   
--dry-run  
 https://www.example.com/  
  
运行要求：  
  
·  
Ruby >= 2.7；  
  
·  
无运行时 gem 依赖；  
  
·  
POC 只使用 Ruby 标准库（  
Socket  
、  
OpenSSL  
），不调用 Python，不依赖外部利用脚本。  
### 4.3 三种扫描配置档  
<table><thead><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">配置档</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">典型请求数</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">做什么</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">passive</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">2</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">根路径 + 文本探针；不探测 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">.php</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">standard</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">4</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">根路径 + 文本/PHP 探针 + </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">/index.php</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">；检测到框架时才探测框架路径</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">deep</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">最多 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">--max-requests</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">常见 PHP 路径、框架路径、</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">robots.txt</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style=""><span leaf="">sitemap.xml</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> 发现</span></span></span></span></span></p></td></tr></tbody></table>  
深度模式不是目录爆破。它只读取站点自己发布的少量路径，并受   
--max-requests  
 限制。  
### 4.4 本次命令的每个参数  
  
./bin/php2shell  
  
-d  
  
-r  
  
-c  
  
-v  
  
"https://www.example.com/"  
  
-en  
<table><thead><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">参数</span></span></span></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">作用</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">-d</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">--profile deep</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">，深度扫描</span></span></span></span></span></p></td></tr><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">-r</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">攻击者视角笔记（Red Team notes），用于防守方理解攻击者会怎么看</span></span></span></span></span></p></td></tr><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">-c</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">相关工具链建议</span></span></span></span></span></p></td></tr><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">-v</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">技术细节、页面检查、攻击路径、防御检查、完整请求跟踪</span></span></span></span></span></p></td></tr><tr><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style=""><span leaf="">-en</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"></span></span></p></td><td data-colwidth="295" width="295" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:php2shellpoc2026-3.0它到底是什么;"><span style="mso-bookmark:本次命令的每个参数;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">强制英文文本输出，便于截图、报告和跨团队沟通</span></span></span></span></span></p></td></tr></tbody></table>  
这里的   
-r  
  
不是  
“运行 exploit”，也不是“自动攻击”。它只是报告中的 ATT&CK 视角注释。真正的 POC 只在扫描结束后、交互式确认   
y  
 才会运行。  
  
文本输出跟随设备语言；  
-en  
 /   
-zh  
 可以强制英文/中文。机器可读格式始终保留英文消息，以保持 JSON/SARIF/CSV 契约稳定。  
### 4.5 安全属性  
  
·  
扫描阶段只发送   
GET  
；  
  
·  
不发送 POST、载荷、模糊测试或改变状态的请求；  
  
·  
--scope  
 /   
--require-scope  
 在打开 socket 之前强制执行；  
  
·  
默认 User-Agent 明确标识工具；  
  
·  
--browser-ua  
 必须显式开启，且报告中会标注；  
  
·  
机器可读格式（JSON/SARIF/CSV/NDJSON）、  
--quiet  
、  
--output  
、非交互会话永远不会运行 POC；  
  
·  
POC 有独立的警告、确认和失败关闭逻辑。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa2fSXnMfexYzrYqLEDeoKRd2JgZom2SysTnaXyam2sC1yfAuN34IAMHzFnPAWCVMpt7xY8lY7GqaZkpQm76RVmbN1oLOg7fxX4/640?wx_fmt=png "")  
## 5. 实战：扫描命令与整体结果  
### 5.1 命令  
  
./bin/php2shell  
  
-d  
  
-r  
  
-c  
  
-v  
  
"https://www.example.com/"  
  
-en  
### 5.2 顶部摘要  
  
来源：Terminal 实测输出（  
-en  
 英文文本模式，已脱敏）。  
  
php2shell-poc2026 3.0.0 — PHP2Shell vulnerability check  
  
scan depth deep · 1 target · 3 requests · 2.83 s · 2026-10-24T15:59:06Z  
  
Authorized assessments only. GET requests only; no payloads, no state-changing verbs.  
  
    
Bottom line: YES — 1 target hackable via PHP2Shell; patch or upgrade now.  
  
    
Next:  
          
patch or upgrade PHP on the exposed host; see HOW TO FIX IT.  
  
这段摘要给出了四个关键信息：  
  
1.  
YES  
：目标被判定为可被 PHP2Shell 入侵；  
  
2.  
1 target  
：只扫描了一个目标；  
  
3.  
3 requests  
：深度模式本来会发更多请求，但自适应逻辑在拦截后提前停止；  
  
4.  
2.83 s  
：扫描耗时很短，说明没有做暴力枚举。  
### 5.3 发现表  
  
    
FINDINGS  
  
    
TARGET  
                      
HACKABLE?  
    
PHP  
       
NOTE  
  
    
------------------------  
    
---------  
    
------  
    
------------------  
  
    
https://www.example.com/  
    
YES  
          
7.3.33  
    
end-of-life branch  
  
    
Legend: YES = hackable · NO = not hackable on the pages tested · UNKNOWN = could not be checked  
  
            
UNKNOWN is never the same as "NO".  
  
这张表只有一行，但信息量很大：  
  
·  
YES  
 不是“已经拿到 shell”，而是“按照当前证据，必须按可被入侵处理”；  
  
·  
7.3.33  
 是版本证据；  
  
·  
end-of-life branch  
 是原因；  
  
·  
UNKNOWN is never the same as "NO"  
 是整份报告的哲学。  
### 5.4 决策卡：先给行动，再给细节  
  
    
DECISION:  
   
YES — this host can be hacked by PHP2Shell.  
  
              
A remote attacker could exploit this host without logging in.  
  
              
Blocked by edge: Access Denied  
  
              
Caveat: the version came from a blocked response; confirm it from an approved connection before  
  
              
acting on this decision.  
  
    
ACTION:  
     
upgrade PHP now; isolate the host until it is fixed  
  
这段是本报告最值得学习的地方：它同时给出  
结论  
和  
限制条件  
。  
  
·  
结论是   
YES  
，因为 PHP 7.3.33 是 EOL；  
  
·  
限制条件是“版本来自被拦截的响应，置信度低”；  
  
·  
动作是“立即升级；在修复前隔离”。  
  
一个成熟的决策者不会只看   
YES  
，也不会因为   
Caveat  
 就忽略它。正确的读法是：  
  
先按高风险处理，同时用受控方式补充证据。  
### 5.5 WHAT THIS MEANS：把技术翻译成业务语言  
  
    
WHAT  
   
THIS MEANS  
  
    
Result:  
        
HACKABLE  
  
    
Why:  
           
The PHP branch is no longer supported, so no fix is coming.  
  
    
Confidence:  
    
Low — the version came from a blocked response.  
  
    
Evidence:  
      
0 of 3 pages responded normally  
  
    
PHP:  
           
7.3.33 — from a response header  
  
    
Blocked by:  
    
Access Denied  
  
    
Fix:  
           
Upgrade to a supported PHP branch (8.2.35, 8.3.36, 8.4.27, 8.5.12) or your distro's fixed  
  
                 
package.  
  
这里有一个容易被误读的点：  
  
0 of 3 pages responded normally  
 和   
HACKABLE  
 并不矛盾。  
  
因为“可被入侵”是一个  
风险决策  
，不是“已成功利用”的概率。扫描器看到了：  
  
·  
EOL 分支（正向信号，权重 +5）；  
  
·  
PHP 行为（正向信号，+1）；  
  
·  
解析器在认证之前运行（正向信号，+2）；  
  
·  
边缘拦截（中性信号，权重 0）；  
  
·  
WAF（中性信号，权重 0）。  
  
正向 8，负向 0，中性 0，所以证据比率是   
1.00  
，评级是   
EXPOSED  
。但置信度仍然是   
low  
，因为版本来自 403 响应，且没有正常应用页面。  
### 5.6 TECHNICAL DETAILS：技术细节  
  
    
TECHNICAL  
   
DETAILS  
  
    
FIELD  
                    
VALUE  
  
    
---------------------  
    
--------------------------  
  
    
Technical verdict  
        
END-OF-LIFE  
  
    
Access path  
              
blocked (HTTP 403)  
  
    
Severity (CVSS-like)  
     
10.0/10 (CRITICAL)  
  
    
Confidence  
               
low  
  
    
Version  
                  
7.3.33  
  
    
Pages usable  
             
0/3 pages usable  
  
    
Evidence rating  
          
YES — EXPOSED (ratio 1.00)  
  
    
Control page  
             
403 (similarity 1.00)  
  
    
Server  
                   
LiteSpeed  
  
    
Edge  
                     
litespeed  
  
    
Edge protection (WAF)  
    
unidentified edge control  
  
    
Block page  
               
Access Denied  
  
    
PHP runtime (SAPI)  
       
litespeed  
  
    
Exploit difficulty  
       
high (waf, edge block)  
  
逐项解释：  
  
·  
Technical verdict = END-OF-LIFE  
：不是“低于修复下限”，而是“整个分支没有上游修复”。  
  
·  
Access path = blocked (HTTP 403)  
：边缘拒绝了所有探测，应用层不可见。  
  
·  
Severity = 10.0/10  
：这是暴露严重度，不是完整 CVSS 向量；不要把它当作“已确认可利用”的 CVSS 分数。  
  
·  
Confidence = low  
：版本来自拦截响应，缺少正常应用证据。  
  
·  
Pages usable = 0/3  
：没有一页可用于正常应用层判断。  
  
·  
Evidence rating = EXPOSED (ratio 1.00)  
：正向证据占主导；中性证据不参与分母。  
  
·  
Control page = 403 (similarity 1.00)  
：控制页和根路径几乎完全一样，说明是统一拦截页。  
  
·  
Server / Edge / SAPI = LiteSpeed  
：这是部署指纹，不是漏洞结论。  
  
·  
WAF = unidentified edge control  
：识别到有边缘控制，但未能确定具体厂商。  
  
·  
Exploit difficulty = high  
：WAF/边缘拦截增加了利用阻力，但不改变暴露严重度。  
### 5.7 PAGES CHECKED：三页，三种信号  
  
    
PAGES  
   
CHECKED  
  
      
PATH  
                               
METHOD  
    
STATUS  
    
TIME  
      
SIZE  
       
SIGNAL  
  
      
-------------------------------  
    
------  
    
------  
    
------  
    
-------  
    
-------  
  
      
/  
                                  
GET  
       
403  
       
912 ms  
    
30.3 KB  
    
denied  
  
      
/php2shell-fd26384daa5374c2.txt  
    
GET  
       
403  
       
944 ms  
    
30.3 KB  
    
control  
  
      
/php2shell-fd26384daa5374c2.php  
    
GET  
       
403  
       
898 ms  
    
30.3 KB  
    
control  
  
三个路径都返回 403，大小完全一致，耗时接近。这说明：  
  
·  
不是某个路径不存在；  
  
·  
不是软 404；  
  
·  
是边缘统一返回的拦截页；  
  
·  
版本信息是从这个拦截页的响应头泄露出来的。  
### 5.8 ATTACK PATH：评估路径 vs 杀伤链  
  
报告把 ATT&CK 分为两层：  
  
评估路径（本次扫描实际做的）：  
  
·  
T1595  
 Active Scanning；  
  
·  
T1592  
 Gather Victim Host Information。  
  
杀伤链（在确认暴露时才打印）：  
  
·  
T1190  
 Exploit Public-Facing Application；  
  
·  
T1059.004  
 Unix Shell；  
  
·  
T1505.003  
 Web Shell；  
  
·  
T1005  
 Data from Local System；  
  
·  
T1082  
 System Information Discovery；  
  
·  
T1071.001  
 Web Protocols。  
  
这个分层很重要：不要因为报告里出现了完整杀伤链，就以为扫描器已经完成了利用。它只是说明“如果解释器确实可利用，攻击者可能怎么走”。  
### 5.9 DEFENDER CHECKS：蓝队可以直接拿去用的检查  
  
报告给出了七类防御检查，基本覆盖了从边缘到主机的完整链路。  
#### 5.9.1 WAF：阻断 upload-progress 利用面（CVE-2026-109619）  
  
·  
AWS WAF：自定义规则匹配 POST body 中包含   
PHP_SESSION_UPLOAD_PROGRESS  
 和   
PHPSESSID  
 的请求；  
  
·  
AWS WAF PHP Managed Rule Group v2.3+ 已覆盖该模式；  
  
·  
ModSecurity：检查 multipart body，尤其是正常上传表单之外的   
PHP_SESSION_UPLOAD_PROGRESS  
。  
#### 5.9.2 WAF/Proxy：缩小 OOM 触发面（CVE-2026-109618）  
  
·  
对  
解码后  
的请求体实施大小限制；  
  
·  
限制 multipart 部件数、字段数、头部数量、嵌套深度；  
  
·  
覆盖没有可用   
Content-Length  
 的请求；  
  
·  
如果业务不需要上传，AWS WAF   
SizeRestrictions_BODY  
 可以设得很低（例如 8 KB）。  
#### 5.9.3 PHP 日志：捕捉请求启动阶段的内存耗尽  
  
在 FPM/Apache 错误日志中搜索：  
  
Allowed memory size of ... exhausted  
  
Cannot allocate memory  
  
关键不是“有没有 fatal error”，而是  
fatal error 出现在脚本名之前  
。这更符合“请求启动阶段就出问题”的特征。  
#### 5.9.4 Web 层：工作进程健康异常  
  
·  
mod_php：Apache 子进程 segfault、重启、core dump；  
  
·  
FPM：工作进程异常退出、快速 respawn 风暴；  
  
·  
但要记住：  
利用也可能不崩溃  
，没有 segfault 不等于干净。  
#### 5.9.5 EDR：Web 工作进程 UID 派生的 shell  
  
重点监控：  
  
www-data / apache / nginx / litespeed  
  
    
→ /bin/sh  
  
    
→ bash  
  
    
→ curl  
  
    
→ wget  
  
    
→ python  
  
尤其是在一个入站 HTTP 请求之后紧接着出现 shell 派生。这是最直接的后利用信号之一。  
#### 5.9.6 应用层：内存泄露的响应异常  
  
·  
返回异常大的、非 HTML 的响应体；  
  
·  
会话 Cookie 中反射出非预期的内存内容；  
  
·  
本应短小的接口返回大量二进制或随机字节。  
#### 5.9.7 配置审计：确认 PHP 加固真的生效  
<table><thead><tr><td data-colwidth="207" width="207" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">配置</span></span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">建议</span></span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">说明</span></span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="207" width="207" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">session.upload_progress.enabled</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">Off</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">关闭不需要的上传进度功能</span></span></span></span></span></span></p></td></tr><tr><td data-colwidth="207" width="207" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">memory_limit</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">合理限制</span></span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">用 </span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">php_admin_value</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">，不要在运行时被覆盖</span></span></span></span></span></span></p></td></tr><tr><td data-colwidth="207" width="207" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">fatal_error_backtraces</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">0</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">（8.5）</span></span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">避免错误信息泄露</span></span></span></span></span></span></p></td></tr><tr><td data-colwidth="207" width="207" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style=""><span leaf="">post_max_size</span></span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">尽量低</span></span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style=""><span style="mso-bookmark:配置审计确认-php-加固真的生效;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">缩小大体积请求的接受面</span></span></span></span></span></span></p></td></tr></tbody></table>  
注意：这些配置必须  
从真实 Web 请求验证  
（例如受控的   
phpinfo()  
），不能只看 CLI 的   
php -i  
。修改后要重启所有工作进程。  
### 5.10 TECHNICAL SIGNALS：为什么比率是 1.00  
  
    
evidence  
   
rating: EXPOSED ratio 1.00 confidence low coverage 0/3  
  
    
evidence:  
      
positive 8 / negative 0 / neutral 0  
  
      
signals:  
  
         
+5  
    
branch 7.3 is end-of-life — no upstream fix exists  
  
         
+1  
    
PHP behaviour observed  
  
         
+2  
    
fires before authentication — the parser runs before application code  
  
          
0  
    
WAF (unidentified edge control) in front  
  
          
0  
    
edge denied every probe (HTTP 403)  
  
计算公式可以简化为：  
  
ratio = positive / (positive + negative)  
  
        
= 8 / (8 + 0)  
  
        
= 1.00  
  
当   
positive + negative == 0  
 时，比率是   
null  
，评级是   
INCONCLUSIVE  
。这就是 3.0 与 2.x 最大的区别：  
不会用   
0.00  
 伪造“安全”。  
### 5.11 ATTACKER NOTES：从攻击者视角看这份报告  
  
报告里的 attacker notes 不是“攻击教程”，而是让防守方理解优先级：  
  
·  
解析器在应用代码之前运行，所以每个   
.php  
 路由都可能是入口；  
  
·  
mod_php 风险最高，因为一个请求一个进程，没有 worker 回收；  
  
·  
FPM 会降低成功率（两个请求必须命中同一个 worker），但不会消除风险；  
  
·  
EOL PHP 没有上游修复，攻击者会优先关注；  
  
·  
WAF 指纹不明，说明边缘控制存在但未知，找到源站通常是更便宜的路径；  
  
·  
内存泄露变体不会崩溃，所以只看可用性监控发现不了探测阶段；  
  
·  
利用流量可能看起来像普通的大表单提交。  
### 5.12 TOOLCHAIN：工具链建议  
  
报告按阶段列出了工具：  
<table><thead><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">阶段</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">工具</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">用途</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">侦察</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">crt.sh / certificate transparency</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">查证书透明度日志，找主机名和源站证书</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">侦察</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">subfinder / Amass</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">被动子域名枚举</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">WhatWeb / Wappalyzer</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">技术指纹</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">curl</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">最小化、可脚本化的 HTTP 控制</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">httpx</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">批量探活和指纹</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">wafw00f</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">WAF 识别</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">ffuf / Feroxbuster</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">目录和文件发现（授权范围内）</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">扫描与枚举</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">openssl s_client / dig</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">TLS 证书链和 DNS</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">漏洞评估</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">Nuclei（tech-detect + 自定义模板）</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">模板化技术/暴露检查</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">漏洞评估</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">Nuclei PHP2Shell 模板</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">批量检查解释器版本和 upload-progress 面</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">漏洞评估</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">Burp Suite / OWASP ZAP</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">拦截、重放、字节级对比</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">验证</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">浏览器 DevTools / 受控出口的 curl</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:实战扫描命令与整体结果;"><span style="mso-bookmark:toolchain工具链建议;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">区分“扫描器被拦”与“应用真实响应”</span></span></span></span></span></p></td></tr></tbody></table>  
攻击和后利用条目默认不显示；只有显式传入   
--authorized  
 才会出现。这是为了避免把扫描报告变成攻击手册。  
### 5.13 NEXT STEPS：报告给出的 7 个动作  
  
1.  
从自己的出口验证响应  
  
   
curl  
  
-sSI  
 https://www.example.com/  
  
   
目的：用非扫描器 User-Agent 确认状态行和响应头。  
  
2.  
升级后复扫  
  
   
php2shell-poc2026  
 https://www.example.com/  
  
   
目的：确认暴露主机现在报告的是修复后的 PHP 版本。  
  
3.  
生成 CI 可用的 SARIF  
  
   
php2shell-poc2026  
  
--format  
 sarif   
--output  
 php2shell.sarif https://www.example.com/  
  
   
目的：把发现变成机器可读的安全待办。  
  
4.  
盘点同分支的兄弟主机（需授权）  
  
   
nuclei  
  
-u  
 https://www.example.com/   
-t  
 tech-detect  
  
   
目的：在关闭问题前，确认同一分支的其他主机。  
  
5.  
检查 TLS 证书  
  
   
openssl  
 s_client   
-connect  
 www.example.com:443   
-servername  
 www.example.com   
<  
/dev/null   
2  
>  
/dev/null   
|  
  
    
openssl  
 x509   
-noout  
  
-subject  
  
-issuer  
  
-dates  
  
-ext  
 subjectAltName  
  
   
目的：证书 SAN 和签发者是 passive origin discovery 信号。  
  
6.  
解析 DNS  
  
   
dig  
 +short www.example.com  
  
   
目的：确认主机指向，以及是否存在 CDN/源站分离。  
  
7.  
下次运行时强制执行范围  
  
   
php2shell-poc2026  
  
--scope  
 scope.txt   
--require-scope  
 https://www.example.com/  
  
   
目的：目标不在授权列表内时失败关闭（退出码 4）。  
### 5.14 HOW TO FIX IT：修复建议  
  
报告给出的修复路径很明确：  
  
·  
PHP 7.3.33（7.3 分支）没有修复版本，应规划迁移到受支持分支（8.4 或 8.5）；  
  
·  
如果暂时不能迁移：使用发行版/厂商 backport，并记录它跟踪的安全公告和过期时间；  
  
·  
在边缘做补偿控制：WAF 规则 + 源站锁定，防止直接 IP 绕过；  
  
·  
EOL 解释器常出现在被遗忘的主机、子域、vhost、staging 环境，关闭问题前要全量盘点。  
### 5.15 REQUEST TRACE：三请求的完整证据  
  
    
REQUEST  
   
TRACE  
  
      
GET https://www.example.com/  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=912ms  
  
      
GET https://www.example.com/php2shell-fd26384daa5374c2.txt  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=944ms  
  
      
GET https://www.example.com/php2shell-fd26384daa5374c2.php  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=898ms  
  
      
access: blocked — every probe, including a random control path, returned HTTP 403 — from LiteSpeed — the …  
  
      
probes: 3 content, 3 requests, version 7.3.33, verdict eol  
  
这段证据非常干净：  
  
·  
三个请求都是 GET；  
  
·  
三个响应都是 403；  
  
·  
三个响应大小一致；  
  
·  
版本信息来自   
X-Powered-By  
；  
  
·  
结论是   
blocked  
，不是   
open  
。  
### 5.16 RUN SUMMARY：运行摘要  
  
    
RUN  
   
SUMMARY  
  
    
targets:  
      
1 (exposed 1, patched 0, no PHP 0)  
  
    
unverified: 0  
  
    
errors:  
       
0  
  
    
coverage:  
     
0/3 probes usable (0.0%) · 1/1 targets assessable  
  
    
requests:  
     
3 in 2.83 s  
  
注意   
coverage: 0/3 probes usable (0.0%)  
。这看起来很低，但扫描器没有把它隐藏起来。  
低覆盖率 + 高暴露判定 + 低置信度  
，正是一份诚实的报告应该呈现的样子。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa062V1EZcvuq1icWY20WEibF1A8YylHmyhaRzBHcuSpQXfrVyltjvmQ3oviaTIDYPyOkde6JFmnbkRFkkoJIwbXTwg7IQW4bDYXH0/640?wx_fmt=png "")  
## 6. POC 阶段：从 y/N 到 RESULT  
### 6.1 为什么没有 --poc 参数  
  
这个设计是刻意的：  
  
POC 不应该是一个可以随手加上的命令行开关。  
  
如果存在   
--poc  
，它很容易被放进脚本、CI、批量任务或别人的命令历史里，然后在没有人工确认的情况下对大量目标发送大体积 POST 请求。  
  
所以 3.0 只提供一种入口：  
  
·  
文本模式扫描；  
  
·  
判定为   
vulnerable  
 或   
eol  
；  
  
·  
stdout 是 TTY；  
  
·  
用户明确回答   
y  
；  
  
·  
先打印授权警告，再运行 POC。  
  
以下情况永远不会运行 POC：  
  
·  
JSON / Markdown / SARIF / CSV / NDJSON；  
  
·  
--quiet  
；  
  
·  
--output  
；  
  
·  
非交互会话。  
### 6.2 提示与警告  
  
本次输出：  
  
    
AUTHORIZED  
   
PENETRATION TEST  
  
        
· Detected PHP 7.3.33. The lab POC is calibrated for PHP 8.4.26 (bookworm).  
  
    
? This host is exposed. Proceed with the authorized penetration test? [y/N] y  
  
这里已经给出了一个非常重要的信息：  
  
检测到的是 PHP 7.3.33，而 POC 校准于 PHP 8.4.26（bookworm）。  
  
在这种情况下，更稳妥的选择通常是回答   
n  
。原因见下一节。  
  
扫描器不会替操作者判断“构建是否匹配”。它只根据 verdict 决定是否询问，并把  
检测到的版本  
和  
POC 校准版本  
同时显示出来，让操作者自己决定。这是为了避免工具在未经人工确认的情况下，对不匹配的目标运行高风险 POC。  
  
回答   
y  
 后，工具会打印完整警告：  
  
    
AUTHORIZED  
   
USE ONLY  
  
    
----------------------------------------------------------------------------------------------------------  
  
    
Do not use this scanner or POC against military, government, critical infrastructure, or any system you do  
  
    
not own or have explicit written permission to test. Unauthorized access is illegal.  
  
    
This POC performs memory exhaustion and worker rotation. It may disrupt the target service. It is  
  
    
calibrated for the bookworm PHP 8.4.26 lab build and may fail against other builds.  
  
    
The authors and contributors are not responsible for any damage, misuse, or legal consequences. By  
  
    
proceeding, you confirm that you have authorization and accept full responsibility for your actions.  
  
这段警告不是装饰。它明确说明了：  
  
·  
授权边界；  
  
·  
可能中断服务；  
  
·  
构建不匹配可能失败；  
  
·  
作者不承担责任；  
  
·  
继续即表示你确认授权并承担全部责任。  
### 6.3 POC 的三个阶段  
  
POC 不是“发一个包就结束”，而是三个可观测阶段：  
  
sequenceDiagram  
  
      
participant CLI as CLI  
  
      
participant Runner as Poc::Runner  
  
      
participant Target as 目标 PHP 工作进程  
  
      
participant Proof as 本地 out/p1  
  
      
CLI->>CLI: 扫描判定 YES + TTY  
  
      
CLI->>Runner: 用户回答 y，打印警告  
  
      
Runner->>Target: 阶段 1：重置/预热 + multipart  
  
      
Target-->>Runner: PHPSESSID Cookie  
  
      
Runner->>Runner: 计算 Zend 堆锚点  
  
      
alt 堆锚点无效  
  
          
Runner-->>CLI: FAILED / HEAP_LEAK_FAILED  
  
      
else 堆锚点有效  
  
          
Runner->>Target: 阶段 2：数组布局 + 析构指针  
  
          
Target-->>Runner: libphp 映射基址  
  
          
Runner->>Target: 阶段 3：伪造载体 + 证明命令  
  
          
Runner->>Proof: 读取 out/p1  
  
          
Proof-->>Runner: 证明内容 / 空  
  
          
Runner-->>CLI: SUCCESS / FAILED  
  
      
end  
#### 阶段 1：堆布局（Heap layout）  
  
·  
预热工作进程；  
  
·  
发送分片的 multipart body；  
  
·  
从   
PHPSESSID  
 Cookie 中推导 2 MB 对齐的 Zend 堆锚点。  
#### 阶段 2：库基址（Library base）  
  
·  
发送精心构造的数组布局；  
  
·  
从 Cookie 中的   
pDestructor  
 指针推导 libphp 映射基址。  
#### 阶段 3：命令投递（Command delivery）  
  
·  
构造 HashTable / execute_data / zend_string 载体；  
  
·  
发送最终引用；  
  
·  
目标执行   
id>/out/p1  
；  
  
·  
本地读取   
out/p1  
，确认代码执行。  
### 6.4 本次 POC 为什么失败  
  
本次 POC 的输出：  
  
来源：Terminal 实测输出（交互式 POC 会话，已脱敏）。  
  
    
[1/3]  
   
Heap layout  
  
        
· Measuring the Zend heap from the session cookie.  
  
        
→ Heap attempt 1/3.  
  
        
→ Sending the heap-shaping warm-up (about 12 MB)  
  
        
← Response received in 4.1 s.  
  
        
→ Sending the heap-leak request  
  
        
← Response received in 1.4 s.  
  
        
✗ The heap leak did not return a usable anchor.  
  
        
...  
  
        
✗ POC failed: The heap leak did not return a usable anchor.  
  
    
PENETRATION TEST — FAILED  
  
        
✗ RESULT: FAILED — no proof was observed.  
  
        
· Reason: The heap leak did not return a usable anchor.  
  
        
· Attempts: heap 3/3 · library 0/3 · command 0/5  
  
        
· Duration: 42.46 s  
  
        
· Technical details:  
  
        
· Status: failed  
  
        
· Error code: HEAP_LEAK_FAILED  
  
        
· Heap anchor: unavailable  
  
        
· Library base: unavailable  
  
        
· Carrier address: unavailable  
  
        
· system() address: unavailable  
  
失败点在  
阶段 1  
：三次堆泄露尝试都没有拿到可用锚点，因此没有进入阶段 2 和阶段 3。  
  
可能原因有三类：  
  
1.  
构建不匹配  
  
o  
目标是 PHP 7.3.33 + LiteSpeed；  
  
o  
POC 校准于 bookworm PHP 8.4.26；  
  
o  
堆布局、分配器行为、libphp 偏移、SAPI 生命周期都可能不同；  
  
o  
默认偏移   
0x43a870  
 /   
0x613c50  
 /   
0x690d8  
 只对特定构建有意义。  
  
2.  
边缘拦截  
  
o  
扫描阶段所有 GET 都返回 403；  
  
o  
POC 的 POST/multipart 请求很可能也在边缘被拦截，没有到达 PHP；  
  
o  
注意：  
← Response received  
 只说明 TCP/HTTP 有响应，  
不代表应用执行了请求  
；边缘拦截页也会快速返回；  
  
o  
如果请求没有进入解释器，堆布局就不可能成功。  
  
3.  
应用不可见  
  
o  
0/3 pages usable  
；  
  
o  
没有正常应用响应，无法确认上传进度、会话行为或工作进程模型；  
  
o  
在这种情况下，POC 的可靠性天然很低。  
### 6.5 失败不等于安全  
  
这是全文最需要强调的结论之一：  
  
POC 失败只能说明“这次没有确认代码执行”，不能说明“目标已修复”或“目标不可利用”。  
  
可能的情况包括：  
  
·  
目标构建不匹配；  
  
·  
请求被 WAF/边缘拦截；  
  
·  
工作进程模型不同；  
  
·  
偏移不对；  
  
·  
证明文件路径没有共享；  
  
·  
命令执行了但没有写出证明；  
  
·  
目标在测试期间重启或切换了工作进程。  
  
其中“证明文件路径没有共享”经常被忽略：POC 的命令是   
id>/out/p1  
，目标写入   
/out/p1  
，而扫描器在本地读取   
./out/p1  
。只有实验环境把这两个目录映射到一起，POC 才能看到证明。如果命令执行了但目录没有共享，POC 仍然会报告   
FAILED  
。  
  
所以正确的记录方式是：  
  
扫描：YES / EOL / 低置信度  
  
POC：FAILED / HEAP_LEAK_FAILED / 未确认代码执行  
  
结论：高风险未验证，仍需按暴露处理  
  
而不是：  
  
POC 失败 → 目标安全  
### 6.6 如果 POC 成功，输出应该是什么样  
  
成功时，报告会给出一张明确的成功卡：  
  
    
PENETRATION  
   
TEST — CONFIRMED  
  
        
✓ RESULT: SUCCESS — the proof command ran on the target.  
  
        
· Proof: uid=0(root) gid=0(root) groups=0(root)  
  
        
· Attempts: heap 1/3 · library 1/3 · command 2/5  
  
        
· Duration: 12.34 s  
  
-v  
 还会附加：  
  
·  
Heap anchor；  
  
·  
Library base；  
  
·  
Carrier address；  
  
·  
system()  
 address；  
  
·  
完整输出日志。  
  
成功只证明“该构建上代码执行发生了”，  
不授权  
继续做持久化、横向移动或数据收集。  
### 6.7 POC 的边界  
  
·  
不转发   
--proxy  
、  
--basic-auth  
、  
--bearer  
、  
--cookie  
；  
  
·  
使用直连 socket；  
  
·  
默认 12 MB OOM 字段；  
  
·  
30 秒请求超时（预热请求 90 秒）；  
  
·  
堆/库各 3 次重试，命令 5 次；  
  
·  
每个阶段前和失败后轮换工作进程；  
  
·  
证明命令固定为   
id>/out/p1  
；  
  
·  
证明文件固定为   
out/p1  
；  
  
·  
不改变扫描退出码。  
  
退出码仍然是：  
<table><thead><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">退出码</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">含义</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">0</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">安全（已修复 / 无 PHP），或未验证且未指定 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">--fail-on</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">1</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">用法、配置或内部错误</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">2</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">暴露：存在漏洞或 EOL 解释器</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">3</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">无法判定，且 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">--fail-on inconclusive</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> 或 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">any</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">4</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">范围违规（</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style=""><span leaf="">--require-scope</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:poc-阶段从-yn-到-result;"><span style="mso-bookmark:poc-的边界;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">）</span></span></span></span></span></p></td></tr></tbody></table>  
即使 POC 失败，暴露主机仍然退出   
2  
。扫描发现是契约，POC 是额外的证据步骤。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa2Cf0p4TJqm5d0bdrqn5gvMNKakUPylBNdib855ZW8ZLlj42RwlBVrlC32XvD5n8JoGYu3Iu5rskfunbyxGpwUnTiarJibOBjmlzQ/640?wx_fmt=png "")  
## 7. 防守方手册：从告警到修复  
### 7.1 检测优先级  
<table><thead><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">优先级</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">检测点</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">说明</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P0</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">WAF 命中 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style=""><span leaf="">PHP_SESSION_UPLOAD_PROGRESS</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> + </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style=""><span leaf="">PHPSESSID</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">直接对应 upload-progress 利用面</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P0</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">PHP 日志中脚本名之前的 OOM fatal</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">对应请求启动阶段内存耗尽</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P1</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">Web 工作进程 UID 派生 shell</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">最直接的后利用信号</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P1</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">工作进程异常退出/快速 respawn</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">可能是内存耗尽或崩溃利用</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P1</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">异常大的非 HTML 响应 / Cookie 反射内存</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">内存泄露变体</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P2</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">未识别的边缘控制 / 源站暴露</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">攻击者可能绕过 WAF</span></span></span></span></span></p></td></tr><tr><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">P2</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">同分支 EOL 主机的横向盘点</span></span></span></span></span></p></td><td data-colwidth="197" width="197" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:防守方手册从告警到修复;"><span style=""><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">关闭问题前的资产治理</span></span></span></span></span></p></td></tr></tbody></table>### 7.2 响应流程  
  
flowchart TD  
  
      
A[告警：PHP2Shell 相关信号] --> B{是否确认暴露？}  
  
      
B -->|是| C[隔离 / 虚拟补丁]  
  
      
C --> D[升级 PHP 到修复版本]  
  
      
D --> E[重启所有工作进程]  
  
      
E --> F[从受控出口复扫]  
  
      
F --> G{版本是否达到修复下限？}  
  
      
G -->|是| H[关闭问题，保留证据]  
  
      
G -->|否| D  
  
      
B -->|未知| I[主机侧读取版本                  
  
受控出口验证]  
  
      
I --> B  
  
      
B -->|否| J[记录例外，定期复扫]  
### 7.3 0–24 小时、24–72 小时、长期  
  
0–24 小时：  
  
·  
确认资产归属和授权；  
  
·  
在边缘启用 upload-progress 和大体积请求规则；  
  
·  
检查 PHP/FPM 日志中的 OOM 和异常 worker 退出；  
  
·  
如果目标可达且业务允许，隔离或下线；  
  
·  
保留报告、请求日志和版本证据。  
  
24–72 小时：  
  
·  
升级 PHP 到修复版本；  
  
·  
如果无法升级，申请厂商 backport 或虚拟补丁；  
  
·  
锁定源站，防止直接 IP 绕过 WAF；  
  
·  
复扫并验证版本；  
  
·  
检查同分支的兄弟主机。  
  
长期：  
  
·  
把 PHP 版本纳入资产清单和 CI；  
  
·  
对 EOL 分支设置强制迁移期限；  
  
·  
定期演练 WAF 规则和日志检测；  
  
·  
对新暴露面做   
--dry-run  
 和范围校验。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa0wakQueklfx1bVxuNKWCTFjFEn07mzjbgibl1rVJiczZ8mUIPCXcvHibvpU0TpoWYBh0Wic1R2TLoxWicB0e6I5Jqd4icBWvFiawCqjg/640?wx_fmt=png "")  
## 8. 修复与验证  
### 8.1 升级矩阵  
<table><thead><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">当前分支</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">动作</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.5.x</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">升级到 &gt;= 8.5.12</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.4.x</span></span></span></span></span><o:page></o:page></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">升级到 &gt;= 8.4.27</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.3.x</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">升级到 &gt;= 8.3.36</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.2.x</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">升级到 &gt;= 8.2.35</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">8.1 / 8.0 / 7.x / 5.x</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">迁移到受支持分支；没有上游修复</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">新于修复表</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">报告为 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style=""><span leaf="">unknown / newer_than_table</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:修复与验证;"><span style="mso-bookmark:升级矩阵;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">，不要猜测</span></span></span></span></span></p></td></tr></tbody></table>  
发行版 backport 可能使用不同的版本号。以厂商安全公告为准，不要只看   
php -v  
。  
### 8.2 无法立即升级时的补偿控制  
  
1.  
WAF 规则  
  
o  
阻断包含   
PHP_SESSION_UPLOAD_PROGRESS  
 的可疑 multipart；  
  
o  
限制解码后请求体大小；  
  
o  
限制部件数、字段数、头部数量；  
  
o  
覆盖没有   
Content-Length  
 的请求。  
  
2.  
PHP 配置  
  
o  
session.upload_progress.enabled=Off  
；  
  
o  
合理设置   
memory_limit  
，使用   
php_admin_value  
；  
  
o  
降低   
post_max_size  
；  
  
o  
8.5 上设置   
fatal_error_backtraces=0  
；  
  
o  
修改后重启所有 worker。  
  
3.  
网络与源站  
  
o  
锁定源站，只允许 CDN/WAF 回源；  
  
o  
不要把源站 IP 暴露在证书、DNS 或错误页里；  
  
o  
对管理端口和 staging 环境做额外隔离。  
  
4.  
监控  
  
o  
OOM fatal；  
  
o  
worker 重启/segfault；  
  
o  
Web UID 派生 shell；  
  
o  
异常大响应和 Cookie 异常。  
### 8.3 验证命令  
  
# 1. 从受控出口看响应头  
  
curl  
  
-sSI  
 https://www.example.com/   
|  
  
grep  
  
-i  
  
'x-powered-by\|^server'  
  
# 2. 复扫  
  
php2shell-poc2026  
 https://www.example.com/  
  
# 3. 生成 SARIF  
  
php2shell-poc2026  
  
--format  
 sarif   
--output  
 php2shell.sarif https://www.example.com/  
  
如果响应头不再泄露版本，不代表已经修复；它只代表版本不可见。此时需要主机侧确认。  
### 8.4 CI/CD 集成  
  
-  
  
name  
:  
 PHP2Shell triage  
  
    
run  
:   
|  
  
      
php2shell --scope scope.txt --require-scope \  
  
        
--fail-on inconclusive --format sarif \  
  
        
--output php2shell.sarif https://www.example.com/  
  
-  
  
uses  
:  
 github/codeql-action/upload-sarif@v3  
  
    
with  
:  
  
      
sarif_file  
:  
 php2shell.sarif  
  
CI 永远不会运行 POC：提示需要交互式终端。发现暴露主机的 CI 运行仍会以退出码   
2  
 失败并上传 SARIF。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa15ibYBsKjvfKwDicibx7zeudxGzpZCPwqPlwLe0EwaTHSGZAfmOLqQY1esAFyRv1sZfmdo08mhXGo8sGUrBEFiagO1xs2c57tJQxQ/640?wx_fmt=png "")  
## 9. 方法论：为什么 3.0 更“诚实”  
### 9.1 访问证据优先于路径状态码  
  
3.0 的顺序是：  
  
范围 → 根路径 → 控制探针 → 访问分类 → 版本证据 → 结论 → 报告  
  
而不是：  
  
根路径 403 → 不可利用  
### 9.2 版本证据分级  
<table><thead><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">证据来源</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">置信度</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">说明</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">正常响应头 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style=""><span leaf="">X-Powered-By</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">高</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">精确、可直接比较修复下限</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style=""><span leaf="">phpinfo()</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">高</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">权威但少见</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">框架元数据</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">仅上下文</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">不能推断 PHP 补丁级别</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">拦截/错误响应头</span></span></span></span></span><o:page></o:page></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">低</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">有价值，但需要 caveat</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">仅分支级（如 </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style=""><span leaf="">8.2</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">）</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">未验证</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">不提升为补丁级结论</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">EOL 分支的仅分支级</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">EOL</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:版本证据分级;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">该分支没有含修复的补丁级别</span></span></span></span></span></p></td></tr></tbody></table>### 9.3 暴露 vs 阻力  
<table><thead><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">维度</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">回答的问题</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">本次结果</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">暴露严重度</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">如果解释器脆弱，后果多严重？</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">10.0 / CRITICAL</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">利用阻力</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">攻击者要克服什么？</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">高：WAF、边缘拦截、应用不可见</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">置信度</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">证据有多可靠？</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:暴露-vs-阻力;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">低：版本来自 403</span></span></span></span></span></p></td></tr></tbody></table>  
WAF 不会把   
10.0  
 变成   
0.0  
；它只会改变利用的可能性和可靠性。  
### 9.4 2.x vs 3.0  
<table><thead><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">场景</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">2.x</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">3.0</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">全站 403</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">不可利用 / 0.00</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">BLOCKED / INCONCLUSIVE</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">软 404</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">可能误判路径</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">识别为 SOFT 404</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">登录跳转</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">可能当作已修复</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">AUTH WALL / 未验证</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">429</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">可能当作不存在</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">RATE LIMITED</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">拦截页泄露版本</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">忽略或误用</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">低置信度报告 + caveat</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">暴露与阻力</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">混在一个分数</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">分开报告</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">无证据</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style=""><span leaf="">0.00</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style=""><span leaf="">null</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / INCONCLUSIVE</span></span></span></span></span></p></td></tr><tr><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">POC</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">无</span></span></span></span></span></p></td><td data-colwidth="176" width="176" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style=""><span style="mso-bookmark:x-vs-3.0;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">交互式 y/N + 警告 + 结果卡</span></span></span></span></span></p></td></tr></tbody></table>### 9.5 局限性  
  
这份报告不是认证：  
  
·  
patched  
 只覆盖它看到的版本证据；  
  
·  
no_php  
 只覆盖它测试过的路由；  
  
·  
inconclusive  
 就是字面意思；  
  
·  
YES  
 不代表已经拿到 shell；  
  
·  
POC 失败不代表安全；  
  
·  
WAF 存在不代表已修复；  
  
·  
版本隐藏不代表没有漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa3F4VdTu0bJx1TSHSLXeldV7RW4bic4sVkWIBkzjgvYmOzLGVkX5codPc72wklZQpgHrxxWH2B1a5Xsc0zWeiaib2pqJn0l7PUa7I/640?wx_fmt=png "")  
## 10. FAQ  
### Q1：403 是不是说明目标安全？  
  
不是。403 说明边缘拒绝了扫描器，应用层不可见。它可能是 WAF、IP 封禁、地域限制、认证墙或全站拦截。必须从受控出口或主机侧补充证据。  
### Q2：有 WAF 是不是就不用升级 PHP？  
  
不是。WAF 改变利用阻力，不改变解释器暴露。未打补丁的解析器仍然是未打补丁的解析器。  
### Q3：PHP 7.3.33 一定能被利用吗？  
  
不能这么说。EOL 表示没有上游修复，必须按高风险处理；但“可利用”还取决于构建、SAPI、WAF、网络路径和具体利用条件。本次 POC 没有确认代码执行。  
### Q4：为什么扫描器在 0/3 可用页面的情况下还给 YES？  
  
因为   
YES  
 是风险决策，不是利用成功率。EOL 分支 + PHP 行为 + 预认证解析器是强正向信号；边缘拦截是中性信号，不是“已修复”的负向证据。  
### Q5：为什么置信度是 low？  
  
版本来自 403 拦截响应，不是正常应用响应；没有一页可用于正常应用层判断。低置信度不等于低风险。  
### Q6：POC 失败了，是不是可以关闭问题？  
  
不可以。失败可能是构建不匹配、边缘拦截、偏移不对、证明路径未共享等原因。失败只说明“这次没有确认代码执行”。  
### Q7：为什么 POC 没有命令行开关？  
  
因为 POC 会发送大体积 POST、触发内存耗尽和 worker 轮换。把它做成开关，容易被脚本、CI 或批量任务误用。3.0 只允许交互式文本扫描后的明确   
y  
。  
### Q8：为什么 JSON/SARIF 不运行 POC？  
  
机器格式没有终端，无法展示警告和回答 y/N。它们永远不会运行 POC。  
### Q9：POC 成功是不是就可以继续深入？  
  
不是。POC 只证明代码执行发生，不授权持久化、横向移动或数据收集。后续测试需要单独的授权范围。  
### Q10：退出码会被 POC 改变吗？  
  
不会。暴露主机仍然退出   
2  
，无论用户回答   
n  
、POC 失败，还是 POC 确认执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa2ozxE7QN2cpOEY2yypS7FuasnAGicSZtW12Ce7J8Uv0t2LTW419pDXF73kQYZgEZ5WIc4UzLcOekdlcrkO06357rvyibxNH6XvA/640?wx_fmt=png "")  
## 11. 授权评估检查清单  
### 11.1 扫描前  
  
·  
☐ 书面授权覆盖目标域名、IP、端口、路径和时间窗  
  
·  
☐ 阅读项目范围、RoE、速率限制和禁止项  
  
·  
☐ 准备   
scope.txt  
，使用   
--require-scope  
  
·  
☐ 先用   
--dry-run  
 检查请求计划  
  
·  
☐ 选择目标能承受的   
--delay  
、  
--threads  
、  
--max-requests  
  
·  
☐ 确认报告存储和数据处理方式  
### 11.2 扫描中  
  
·  
☐ 只发送计划内的 GET  
  
·  
☐ 不绕过 403、不切换 IP、不伪造 Host  
  
·  
☐   
--browser-ua  
 仅在明确授权时使用  
  
·  
☐ 发现异常立即停止并记录  
### 11.3 扫描后  
  
·  
☐ 保存报告、请求日志、时间戳和证据  
  
·  
☐ 区分   
YES  
 /   
UNKNOWN  
 /   
NO  
  
·  
☐ 对   
YES  
 目标优先升级/隔离  
  
·  
☐ 对   
UNKNOWN  
 补充受控出口或主机侧证据  
  
·  
☐ POC 只在授权、构建匹配、可中断的窗口内运行  
  
·  
☐ 不把 POC 失败当作关闭问题的理由  
  
·  
☐ 按项目流程披露，不公开目标敏感信息  
### 11.4 范围文件示例  
  
# 精确主机及其子域名  
  
example.com  
  
# 仅子域名  
  
*.example.com  
  
# 单个 IP、CIDR 或 URL  
  
192.0.2.10  
  
192.0.2.0/24  
  
https://app.example.com  
  
php2shell  
  
--scope  
 scope.txt   
--require-scope  
 https://www.example.com/  
  
# 目标不在列表中时退出 4  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa3x3TKtcbjaibaSeh3E5QthH42QvXBGdwfkIxDcxuuCqR8AJEaG6Rf5licjmmMjOVdMhajib7Y3icKIojgA7m9lOH5GIkdno6QCEicE/640?wx_fmt=png "")  
## 12. 总结  
  
这次测试的价值不在于“POC 有没有打成功”，而在于它展示了一个成熟的漏洞评估工具应该如何表达不确定性：  
  
1.  
403 不是安全  
，只是应用不可见；  
  
2.  
WAF 不是补丁  
，只改变利用阻力；  
  
3.  
EOL 是确定的高风险  
，因为不会有上游修复；  
  
4.  
低置信度不等于低风险  
，而是“需要补充证据”；  
  
5.  
POC 失败不等于安全  
，只说明这次没有确认代码执行；  
  
6.  
报告应该先给决策，再给证据和动作  
；  
  
7.  
扫描和利用必须分开  
，POC 只能在明确授权和人工确认后运行。  
  
对于这台目标，正确的行动是：  
  
1. 按 YES / EOL 处理，不要因为 POC 失败而降级；  
  
2. 从受控出口确认版本和访问路径；  
  
3. 升级 PHP 到 8.4.27+ 或 8.5.12+，或迁移到受支持分支；  
  
4. 在边缘启用 upload-progress 和大体积请求规则；  
  
5. 检查 PHP/FPM 日志、worker 健康和 Web UID 派生 shell；  
  
6. 锁定源站，避免 WAF 被绕过；  
  
7. 复扫并保留证据，关闭问题。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa3P6qQJZqicibF5Su3RVlZ51hpsdK2T8c4euQsuYzpaHfNqFQAyGG3RCEU7iaDDhBhgFufAjiacOwvI5I360FD62e8vHibuXhTELcWM/640?wx_fmt=png "")  
## 附录 A：完整扫描输出（已脱敏）  
  
以下输出中的目标域名已统一替换为   
www.example.com  
。来源：Terminal 实测（一次完整的深度扫描 + 交互式 POC 会话），未做删节。输出包含一次实际 POC 尝试；请勿在未授权目标上复现。  
  
$ ./bin/php2shell -d -r -c -v "https://www.example.com/" -en  
  
php2shell-poc2026 3.0.0 — PHP2Shell vulnerability check  
  
scan depth deep · 1 target · 3 requests · 2.83 s · 2026-10-24T15:59:06Z  
  
Authorized assessments only. GET requests only; no payloads, no state-changing verbs.  
  
    
Bottom line: YES — 1 target hackable via PHP2Shell; patch or upgrade now.  
  
    
Next:  
          
patch or upgrade PHP on the exposed host; see HOW TO FIX IT.  
  
    
FINDINGS  
  
    
TARGET  
                      
HACKABLE?  
    
PHP  
       
NOTE  
  
    
------------------------  
    
---------  
    
------  
    
------------------  
  
    
https://www.example.com/  
    
YES  
          
7.3.33  
    
end-of-life branch  
  
    
Legend: YES = hackable · NO = not hackable on the pages tested · UNKNOWN = could not be checked  
  
            
UNKNOWN is never the same as "NO".  
  
    
https://www.example.com/  
  
    
----------------------------------------------------------------------------------------------------------  
  
    
DECISION: YES — this host can be hacked by PHP2Shell.  
  
              
A remote attacker could exploit this host without logging in.  
  
              
Blocked by edge: Access Denied  
  
              
Caveat: the version came from a blocked response; confirm it from an approved connection before  
  
              
acting on this decision.  
  
    
ACTION:  
     
upgrade PHP now; isolate the host until it is fixed  
  
    
WHAT THIS MEANS  
  
    
Result:  
        
HACKABLE  
  
    
Why:  
           
The PHP branch is no longer supported, so no fix is coming.  
  
    
Confidence:  
    
Low — the version came from a blocked response.  
  
    
Evidence:  
      
0 of 3 pages responded normally  
  
    
PHP:  
           
7.3.33 — from a response header  
  
    
Blocked by:  
    
Access Denied  
  
    
Fix:  
           
Upgrade to a supported PHP branch (8.2.35, 8.3.36, 8.4.27, 8.5.12) or your distro's fixed  
  
                 
package.  
  
    
TECHNICAL DETAILS  
  
    
FIELD  
                    
VALUE  
  
    
---------------------  
    
--------------------------  
  
    
Technical verdict  
        
END-OF-LIFE  
  
    
Access path  
              
blocked (HTTP 403)  
  
    
Severity (CVSS-like)  
     
10.0/10 (CRITICAL)  
  
    
Confidence  
               
low  
  
    
Version  
                  
7.3.33  
  
    
Pages usable  
             
0/3 pages usable  
  
    
Evidence rating  
          
YES — EXPOSED (ratio 1.00)  
  
    
Control page  
             
403 (similarity 1.00)  
  
    
Server  
                   
LiteSpeed  
  
    
Edge  
                     
litespeed  
  
    
Edge protection (WAF)  
    
unidentified edge control  
  
    
Block page  
               
Access Denied  
  
    
PHP runtime (SAPI)  
       
litespeed  
  
    
Exploit difficulty  
       
high (waf, edge block)  
  
    
Full reason: PHP 7.3.33 (branch 7.3) is end-of-life — no upstream fix exists; only distro backports or an  
  
                 
upgrade will clear this — observed on an error response; every probe, including a random  
  
                 
control path, returned HTTP 403 — from LiteSpeed — the edge denied the scanner before the  
  
                 
application answered  
  
    
Observed PHP features: PHP yes · phpinfo no · live .php no · upload no · errors no  
  
    
PHP runtime note: LiteSpeed SAPI — affected per the PHP advisory  
  
    
PAGES CHECKED  
  
      
PATH  
                               
METHOD  
    
STATUS  
    
TIME  
      
SIZE  
       
SIGNAL  
  
      
-------------------------------  
    
------  
    
------  
    
------  
    
-------  
    
-------  
  
      
/  
                                  
GET  
       
403  
       
912 ms  
    
30.3 KB  
    
denied  
  
      
/php2shell-fd26384daa5374c2.txt  
    
GET  
       
403  
       
944 ms  
    
30.3 KB  
    
control  
  
      
/php2shell-fd26384daa5374c2.php  
    
GET  
       
403  
       
898 ms  
    
30.3 KB  
    
control  
  
    
ATTACK PATH (MITRE ATT&CK)  
  
      
assessment: T1595 Active Scanning · T1592 Gather Victim Host Information  
  
      
kill chain: T1190 Exploit Public-Facing Application → T1059.004 Command and Scripting Interpreter: Unix  
  
                  
Shell → T1505.003 Server Software Component: Web Shell → T1005 Data from Local System → T1082  
  
                  
System Information Discovery → T1071.001 Application Layer Protocol: Web Protocols  
  
      
Assessment detail:  
  
        
T1595  
       
Reconnaissance — Active Scanning  
  
                  
3 ordinary GET requests (deep profile); no payloads, no state-changing verbs, no exploitation.  
  
                  
This scan is visible in web logs.  
  
        
T1592  
       
Reconnaissance — Gather Victim Host Information  
  
                  
Response headers, TLS certificate, server/edge fingerprint, cookies and framework markers were  
  
                  
read passively to identify the deployment.  
  
      
Kill-chain detail:  
  
        
T1190  
       
Initial Access — Exploit Public-Facing Application  
  
                  
One crafted HTTP request reaches the PHP parser before any application code runs; app-layer  
  
                  
authentication, IAM or SSO cannot block it.  
  
        
T1059.004 Execution — Command and Scripting Interpreter: Unix Shell  
  
                  
Confirmed exploit paths execute attacker commands as the web worker (www-data/apache/nginx).  
  
        
T1505.003 Persistence — Server Software Component: Web Shell  
  
                  
Landing inside a long-lived worker (mod_php/FPM) makes a persistent webshell drop trivial;  
  
                  
successful execution need not crash or restart the worker, so it stays quiet.  
  
        
T1005  
       
Collection — Data from Local System  
  
                  
The disclosure variant leaks process memory (Heartbleed-style): env vars, cloud keys, session  
  
                  
material — reachable without achieving code execution.  
  
        
T1082  
       
Discovery — System Information Discovery  
  
                  
Recon through the hijacked interpreter looks like ordinary app traffic without EDR.  
  
        
T1071.001 Command and Control — Application Layer Protocol: Web Protocols  
  
                  
C2 and exfil blend into normal HTTPS egress from the web tier.  
  
    
DEFENDER CHECKS  
  
      
[WAF] Block the upload-progress arm (CVE-2026-109619)  
  
          
AWS WAF: custom rule matching POST where body contains PHP_SESSION_UPLOAD_PROGRESS and PHPSESSID (body  
  
          
or cookie); the AWS WAF PHP Managed Rule Group v2.3+ covers this. ModSecurity: inspect multipart  
  
          
bodies for PHP_SESSION_UPLOAD_PROGRESS outside expected upload forms.  
  
      
[WAF/Proxy] Shrink the OOM trigger surface (CVE-2026-109618)  
  
          
Enforce body-size limits on *decoded* requests (AWS WAF SizeRestrictions_BODY, 8 KB, if uploads aren't  
  
          
needed), multipart part count, field count, header count and nesting depth — including requests  
  
          
without a usable Content-Length.  
  
      
[PHP logs] Hunt memory-exhaustion during request startup  
  
          
Grep FPM/Apache error logs for "Allowed memory size of ... exhausted", "Cannot allocate memory" and  
  
          
fatal errors *before* the script name appears — that ordering is the CVE-2026-109618 signature.  
  
      
[Web tier] Worker health anomalies  
  
          
mod_php: Apache child segfaults/restarts, core dumps. FPM: abnormal worker exits and rapid respawn  
  
          
storms. Exploitation can also succeed without any crash — absence of segfaults is not clean.  
  
      
[EDR] Shell spawned by web worker UID  
  
          
Alert on /bin/sh, bash, curl, wget or python spawned by www-data/apache immediately after an inbound  
  
          
HTTP request; this is the post-exploitation pivot regardless of which CVE landed.  
  
      
[Application] Memory-disclosure response anomalies  
  
          
Endpoints returning abnormally large or non-HTML bodies, or session cookies reflecting unexpected  
  
          
memory contents — the info-leak half of the bug class.  
  
      
[Config audit] Verify effective PHP hardening (host-side)  
  
          
session.upload_progress.enabled=Off, memory_limit=-1 (php_admin_value, not overridable),  
  
          
fatal_error_backtraces=0 on 8.5, low post_max_size. Verify from a live web request (phpinfo), not the  
  
          
CLI — and restart every worker after changes.  
  
    
TECHNICAL SIGNALS  
  
    
evidence rating: EXPOSED ratio 1.00 confidence low coverage 0/3  
  
    
evidence:  
      
positive 8 / negative 0 / neutral 0  
  
      
signals:  
  
         
+5  
    
branch 7.3 is end-of-life — no upstream fix exists  
  
         
+1  
    
PHP behaviour observed  
  
         
+2  
    
fires before authentication — the parser runs before application code  
  
          
0  
    
WAF (unidentified edge control) in front  
  
          
0  
    
edge denied every probe (HTTP 403)  
  
    
exploit difficulty: high — unidentified edge control may block crafted requests; edge denied every probe; no  
  
                  
application-level evidence  
  
      
what an attacker needs:  
  
        
- TCP reachability to the web port — no credentials, session or user interaction  
  
        
- upload-progress acceptance unconfirmed — the CVE-2026-109618 (OOM) arm needs no special request fields  
  
        
- any live .php route — no specific path, permission or session required  
  
    
reliability: undetermined — SAPI not fingerprintable from outside; measure worker behaviour before trusting  
  
                 
either model  
  
    
impact:  
        
code execution as the web worker UID (T1059.004) with webshell persistence (T1505.003); or  
  
                 
silent memory disclosure of env vars, keys and session material (T1005) with no crash  
  
    
ATTACKER NOTES  
  
      
- The request parser runs before application code, so every .php route is an entry point — no credentials,  
  
        
session or user interaction required.  
  
      
- mod_php is the highest-risk posture per the PHP project: one request, one process, no worker recycle  
  
        
between victims. FPM lowers odds (both requests must hit the same worker) but does not remove them.  
  
      
- EOL PHP (7.3.33, branch 7.3): no upstream fix exists. Defenders can only rely on backports, config  
  
        
mitigations or network isolation — expect this class of target to be prioritised by attackers.  
  
      
- WAF fingerprint: unidentified edge control. That changes exploit reliability, not exposure — the  
  
        
interpreter is still unpatched behind it. Finding the origin is usually the cheaper path.  
  
      
- Opsec: the memory-disclosure variant leaks data without crashing the worker, so availability monitoring  
  
        
alone will not reveal the probing phase. Exploitation attempts may look like ordinary large form  
  
        
submissions.  
  
    
TOOLCHAIN (relevant)  
  
      
Reconnaissance  
  
        
crt.sh / certificate transparency — Search public certificate logs for hostnames and origin  
  
                  
certificates.  
  
        
subfinder / Amass — Passive subdomain enumeration with source deduplication and resolution checks.  
  
      
Scanning & enumeration  
  
        
WhatWeb / Wappalyzer — Technology fingerprinting from responses: CMS, framework, server stack, version  
  
                  
hints.  
  
        
curl — Minimal, scriptable HTTP with full control of method, headers and body.  
  
        
httpx — Probe a host list and return only live endpoints with status, title and technology fingerprint.  
  
        
wafw00f — Probe how the edge reacts to benign requests and identify the WAF vendor, or its absence.  
  
        
ffuf / Feroxbuster — Directory and file discovery with recursion into matched folders.  
  
        
openssl s_client / dig — Read the TLS certificate chain and DNS records directly.  
  
      
Vulnerability assessment  
  
        
Nuclei (tech-detect + custom template) — Template-driven technology and exposure checks; write one  
  
                  
template for the X-Powered-By floor.  
  
        
Nuclei PHP2Shell template — Check the interpreter version and the upload-progress surface at scale.  
  
        
Burp Suite / OWASP ZAP — Intercepting proxy for replay, mutation and byte-level request comparison.  
  
      
Verification  
  
        
Browser DevTools / curl from approved egress — Reproduce the request from a normal browser or an  
  
                  
approved proxy to distinguish a scanner block from an application response.  
  
      
(Offensive and post-exploitation entries are omitted; pass --authorized to include them.)  
  
    
NEXT STEPS  
  
      
1. Verify the response from your own egress  
  
         
$ curl -sSI https://www.example.com/  
  
         
why: Confirms the status line and headers without any scanner-specific User-Agent.  
  
      
2. Re-scan after the upgrade  
  
         
$ php2shell-poc2026 https://www.example.com/  
  
         
why: Confirms the exposed host now reports a fixed PHP version.  
  
      
3. Emit a CI-ready SARIF report  
  
         
$ php2shell-poc2026 --format sarif --output php2shell.sarif https://www.example.com/  
  
         
why: Turns the finding into a machine-readable artifact for the security backlog.  
  
      
4. Inventory sibling hosts (authorized)  
  
         
$ nuclei -u https://www.example.com/ -t tech-detect  
  
         
why: Find other hosts on the same branch before closing the finding.  
  
      
5. Inspect the TLS certificate  
  
         
$ openssl s_client -connect www.example.com:443 -servername www.example.com /dev/null |  
  
           
openssl x509 -noout -subject -issuer -dates -ext subjectAltName  
  
         
why: Certificate SANs and issuer are passive origin-discovery signals.  
  
      
6. Resolve DNS  
  
         
$ dig +short www.example.com  
  
         
why: Confirms where the hostname points and whether a CDN/origin split is visible.  
  
      
7. Enforce scope on the next run  
  
         
$ php2shell-poc2026 --scope scope.txt --require-scope https://www.example.com/  
  
         
why: Fails closed (exit 4) if the target is outside the authorized list.  
  
    
HOW TO FIX IT  
  
      
- PHP 7.3.33 (branch 7.3) has no fixed release — plan a migration to a supported branch (8.4 or 8.5).  
  
        
Everything below is compensation, not a fix.  
  
      
- If migration cannot start now: apply the distro/vendor backport for 7.4/8.0/8.1 where it exists, and  
  
        
note the advisory it tracks so the exception has an expiry date.  
  
      
- Compensate at the edge meanwhile — the WAF rules listed in this report, plus origin locking so a leaked  
  
        
direct IP cannot bypass them.  
  
      
- EOL interpreters cluster on forgotten hosts: sweep subdomains, vhosts and staging boxes for the same  
  
        
branch before closing the finding.  
  
    
REQUEST TRACE  
  
      
GET https://www.example.com/  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=912ms  
  
      
GET https://www.example.com/php2shell-fd26384daa5374c2.txt  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=944ms  
  
      
GET https://www.example.com/php2shell-fd26384daa5374c2.php  
  
      
<-- 403 server=LiteSpeed powered-by=PHP/7.3.33 bytes=31013 time=898ms  
  
      
access: blocked — every probe, including a random control path, returned HTTP 403 — from LiteSpeed — the …  
  
      
probes: 3 content, 3 requests, version 7.3.33, verdict eol  
  
    
RUN SUMMARY  
  
    
targets:  
      
1 (exposed 1, patched 0, no PHP 0)  
  
    
unverified: 0  
  
    
errors:  
       
0  
  
    
coverage:  
     
0/3 probes usable (0.0%) · 1/1 targets assessable  
  
    
requests:  
     
3 in 2.83 s  
  
    
Tip: -q for the short version.  
  
         
--format json|markdown|sarif|csv|ndjson · --dry-run · exit codes 0/1/2/3/4.  
  
    
AUTHORIZED PENETRATION TEST  
  
        
· Detected PHP 7.3.33. The lab POC is calibrated for PHP 8.4.26 (bookworm).  
  
    
? This host is exposed. Proceed with the authorized penetration test? [y/N] y  
  
    
AUTHORIZED USE ONLY  
  
    
----------------------------------------------------------------------------------------------------------  
  
    
Do not use this scanner or POC against military, government, critical infrastructure, or any system you do  
  
    
not own or have explicit written permission to test. Unauthorized access is illegal.  
  
    
This POC performs memory exhaustion and worker rotation. It may disrupt the target service. It is  
  
    
calibrated for the bookworm PHP 8.4.26 lab build and may fail against other builds.  
  
    
The authors and contributors are not responsible for any damage, misuse, or legal consequences. By  
  
    
proceeding, you confirm that you have authorization and accept full responsibility for your actions.  
  
    
PENETRATION TEST  
  
        
· Target: https://www.example.com/  
  
        
· Profile: bookworm PHP 8.4.26 lab build  
  
        
· Proof command: id>/out/p1 (the target writes /out/p1; the lab shares it as ./out/p1)  
  
        
· No offsets file found; using the bookworm defaults.  
  
        
· Offsets: system=0x43a870 destructor=0x613c50 carrier=0x690d8  
  
        
· The POC sends large multipart requests and can take several minutes.  
  
        
→ Preparing a clean PHP worker: sending the reset request (about 12 MB)  
  
        
← Response received in 4.0 s.  
  
        
→ Sending the session warm-up request  
  
        
← Response received in 0.7 s.  
  
    
[1/3] Heap layout  
  
        
· Measuring the Zend heap from the session cookie.  
  
        
→ Heap attempt 1/3.  
  
        
→ Sending the heap-shaping warm-up (about 12 MB)  
  
        
← Response received in 4.1 s.  
  
        
→ Sending the heap-leak request  
  
        
← Response received in 1.4 s.  
  
        
✗ The heap leak did not return a usable anchor.  
  
        
→ Recycling before the next heap attempt: sending the reset request (about 12 MB)  
  
        
← Response received in 4.4 s.  
  
        
→ Sending the session warm-up request  
  
        
← Response received in 0.8 s.  
  
        
→ Heap attempt 2/3.  
  
        
→ Sending the heap-shaping warm-up (about 12 MB)  
  
        
← Response received in 4.5 s.  
  
        
→ Sending the heap-leak request  
  
        
← Response received in 1.5 s.  
  
        
✗ The heap leak did not return a usable anchor.  
  
        
→ Recycling before the next heap attempt: sending the reset request (about 12 MB)  
  
        
← Response received in 4.7 s.  
  
        
→ Sending the session warm-up request  
  
        
← Response received in 0.8 s.  
  
        
→ Heap attempt 3/3.  
  
        
→ Sending the heap-shaping warm-up (about 12 MB)  
  
        
← Response received in 4.7 s.  
  
        
→ Sending the heap-leak request  
  
        
← Response received in 1.4 s.  
  
        
✗ The heap leak did not return a usable anchor.  
  
        
→ Recycling before the next heap attempt: sending the reset request (about 12 MB)  
  
        
← Response received in 5.0 s.  
  
        
→ Sending the session warm-up request  
  
        
← Response received in 0.6 s.  
  
        
✗ POC failed: The heap leak did not return a usable anchor.  
  
    
PENETRATION TEST — FAILED  
  
        
✗ RESULT: FAILED — no proof was observed.  
  
        
· Reason: The heap leak did not return a usable anchor.  
  
        
· Attempts: heap 3/3 · library 0/3 · command 0/5  
  
        
· Duration: 42.46 s  
  
        
· Technical details:  
  
        
· Status: failed  
  
        
· Error code: HEAP_LEAK_FAILED  
  
        
· Heap anchor: unavailable  
  
        
· Library base: unavailable  
  
        
· Carrier address: unavailable  
  
        
· system() address: unavailable  
  
        
· Output log:  
  
        
│ Target: https://www.example.com/  
  
        
│ Profile: bookworm PHP 8.4.26 lab build  
  
        
│ Proof command: id>/out/p1 (the target writes /out/p1; the lab shares it as ./out/p1)  
  
        
│ No offsets file found; using the bookworm defaults.  
  
        
│ Offsets: system=0x43a870 destructor=0x613c50 carrier=0x690d8  
  
        
│ The POC sends large multipart requests and can take several minutes.  
  
        
│ The heap leak did not return a usable anchor.  
  
        
│ The heap leak did not return a usable anchor.  
  
        
│ The heap leak did not return a usable anchor.  
  
        
│ POC failed: The heap leak did not return a usable anchor.  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa2BwmyFtcSiaY6bE1QjKDp3cN77TfgCsZJDQmEzLzj3Nyp01WialvkicBhA68oKktx9SmibmicZLcGE5ibH9fqICNia1VX6zIJycgq0xY/640?wx_fmt=png "")  
## 附录 B：命令速查  
### 扫描  
  
# 被动  
  
php2shell  
  
--profile  
 passive https://www.example.com/  
  
# 标准  
  
php2shell  
 https://www.example.com/  
  
# 深度  
  
php2shell  
  
-d  
 https://www.example.com/  
  
# 深度 + 攻击者笔记 + 工具链 + 技术细节  
  
php2shell  
  
-d  
  
-r  
  
-c  
  
-v  
 https://www.example.com/  
  
# 中文输出  
  
php2shell  
  
-zh  
 https://www.example.com/  
  
# 英文输出  
  
php2shell  
  
-en  
 https://www.example.com/  
  
# 先预演，不打开 socket  
  
php2shell  
  
--dry-run  
  
-d  
 https://www.example.com/  
### 报告格式  
  
php2shell  
  
--json  
 https://www.example.com/   
>  
 report.json  
  
php2shell  
  
--format  
 markdown https://www.example.com/   
>  
 report.md  
  
php2shell  
  
--format  
 sarif   
--output  
 php2shell.sarif https://www.example.com/  
  
php2shell  
  
--format  
 csv https://www.example.com/   
>  
 report.csv  
  
php2shell  
  
--format  
 ndjson https://www.example.com/   
>  
 report.ndjson  
### 范围与授权  
  
php2shell  
  
--scope  
 scope.txt   
--require-scope  
 https://www.example.com/  
  
php2shell  
  
--exclude  
 excluded.txt https://www.example.com/  
  
php2shell  
  
--authorized  
  
-d  
 https://www.example.com/  
### 常用选项  
<table><thead><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">选项</span></span></span></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">说明</span></span></span></span></span></p></td></tr></thead><tbody><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-p</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--profile</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">passive</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">standard</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">deep</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-d</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--deep</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--profile deep</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> 的别名</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-r</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--redteam</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">攻击者笔记</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-c</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--toolchain</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">工具链建议</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-v</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--verbose</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">技术细节和请求跟踪</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-q</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">, </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--quiet</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">只保留结论、发现表和运行摘要</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--lang</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">auto</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">en</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">zh</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-en</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">-zh</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">语言短参数</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--fail-on</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">exposed</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">inconclusive</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">any</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">never</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--dry-run</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">打印请求计划，不打开 socket</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--scope</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;mso-fareast-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf=""> / </span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--require-scope</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">范围控制，失败关闭</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--browser-ua</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">浏览器 UA，仅授权时显式开启</span></span></span></span></span></p></td></tr><tr><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style=""><span leaf="">--authorized</span></span></span></span></span><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"></span></span></p></td><td data-colwidth="264" width="264" valign="top" style="border-top:none;border-left:none;border-bottom:none;border-right:none;padding:0pt 5.4pt 0pt 5.4pt;"><p style=""><span style="mso-bookmark:X7c88e18243de5f8fdf696b04f40cd45a47fa3d6;"><span style="mso-bookmark:附录-b命令速查;"><span style="mso-bookmark:常用选项;"><span style="font-family:Aptos;mso-ascii-font-family:Aptos;font-variant:normal;text-transform:none;"><span leaf="">包含攻击/后利用工具条目</span></span></span></span></span></p></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ia7TorzX5Pa3zHkiaRa1GSibpqvvozYXGlUweIiavUL83yYKZODVwda3BShm2sgtU6rAU69e5vpIu0iaB87NS6tUpaGvL7HEuic9wk7DxJ57icG5qw/640?wx_fmt=png "")  
## 附录 C：参考资料  
  
·  
PHP2ShellPOC2026 项目  
  
·  
CVE-2026-109618  
  
·  
CVE-2026-109619  
  
·  
PHP 发布说明  
  
·  
MITRE ATT&CK  
  
·  
项目内文档：  
  
o  
docs/METHODOLOGY.md  
  
o  
docs/POC.md  
  
o  
docs/AUTHORIZED-USE.md  
  
o  
docs/JSON-SCHEMA.md  
  
o  
docs/UML.md  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa0rVynmicgGk6vw5zH77ytUxDxOatwNYTekxBpfBZRaicsKaSUkmOyZzBnV4MPCnfeg0RzxoUblh8u1juh3T5M8XDhP5Gt81Ka0E/640?wx_fmt=png "")  
## 发布信息建议  
  
·  
标题  
：从 403 到 RCE 风险：PHP2Shell 漏洞全面解析与 PHP2ShellPOC2026 3.0 实战  
  
·  
标签  
：PHP 安全、漏洞分析、网络安全、WAF、CVE、Ruby、应急响应、渗透测试  
  
·  
摘要  
：一次真实授权评估中，扫描器在全部 403 的情况下仍判定 PHP 7.3.33 EOL 高风险；POC 因构建不匹配和边缘拦截失败。本文完整拆解报告、POC 三阶段、失败原因、防守检查和修复路径。  
  
·  
封面  
：使用本文顶部已嵌入的   
php2shell-cover.jpg  
（2560×1440，16:9）——左侧终端 403 响应，右侧 PHP 7.3.33 / EOL / 10.0 评分面板，背景为 WAF 六边形墙与 ATT&CK 杀伤链节点，直接上传 CSDN 封面即可。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ia7TorzX5Pa2UmVD71UMEzCDBFNHNWN9rescMacwE3fVjIfHMmZxZQdM3DCnAPWwQK2efuto8vV7aKV7Boj0wNYibGMjvRicMmIPGLTFAIN7vI/640?wx_fmt=png "")  
  
再次提醒  
：本文仅用于授权安全评估、防御建设和漏洞管理。不要对未授权系统发送扫描或 POC 请求。POC 可能中断服务，失败不代表安全，成功也不授权进一步利用。  
  
