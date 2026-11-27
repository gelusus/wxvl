#  一款专为若依（RuoYi）打造的漏洞扫描器  
xiabai2008
                    xiabai2008  鹏组安全   2026-11-27 00:30  
  
> 通用扫描器告诉你“这里可能有东西”，Ruoyi-Scan告诉你“这个漏洞确实存在/确实不存在/无法判定”。  
  
  
如果你做过政企或外包项目的安全测试，大概率遇到过若依（RuoYi）——国内应用最广的开源Java后台框架之一，二次开发项目遍地都是。但通用扫描器在它面前往往“水土不服”：指纹粗糙、变体不辨、版本无感，误报和漏报同时发生。  
  
一、它和nuclei/xray有什么不同？  
  
不竞争，而是互补。通用工具负责广度，Ruoyi-Scan 负责若依生态的深度与判定纪律。  
<table><tbody><tr><td data-colwidth="191"><p style="text-align: center;"><span leaf="">维度</span></p></td><td data-colwidth="191"><p style="text-align: center;"><span leaf="">通用扫描器</span></p></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">Ruoyi-Scan</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">定位</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">通用POC/Web扫描</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">若依/国产Java框架专项</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">变体识别</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">—</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">5变体（Vue3/App/Plus/Cloud-Plus/Magic）</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">版本感知</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">—</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">4.6.2/4.7.8/4.8.0/4.8.2/4.8.3逐版本实测矩阵</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">未确认项处理</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">无此概念</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">三态判定：CONFIRMED/SAFE /UNKNOWN</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">网络异常</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">报错或跳过</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">强制 UNKNOWN，绝不冒充SAFE</span></section></td></tr><tr><td data-colwidth="191"><section><p style="text-align: center;"><span leaf="">合规交付</span></p></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">部分</span></section></td><td data-colwidth="191"><section style="text-align: center;"><span leaf="">等保2.0/OWASP报告级章节+7 种格式</span></section></td></tr></tbody></table>  
二、核心能力速览  
- 63 个专项 POC：若依 23 个（含 2 个 Plus 变体专属）、Spring Boot 14 个、JeecgBoot 8 个、芋道 yudao 2 个、通用 16 个，覆盖文件读取、SQL 注入、RCE、SSTI、未授权等。  
  
- 三态判定：CONFIRMED（确认存在）/ SAFE（确认不存在）/ UNKNOWN（无法判定）。  
  
- WAF 绕过：11 种策略 + 对抗型传输层（Scrapling 指纹伪装 / 隐身浏览器解 CF 挑战）。  
  
- 漏洞利用链：DAG 拓扑编排 + 条件分支，内置 3 条链。  
  
- nuclei 模板兼容：可直接执行 nuclei-templates 的 YAML 模板（http 协议子集 + 安全白名单）。  
  
- AI POC 生成：--ai 自然语言生成插件，LLM 自验证回灌，无 Key 降级规则模板。  
  
- 企业级特性：批量扫描、分布式（Redis Master-Worker）、Web API（FastAPI + WebSocket）、SIEM 集成、CI/CD 门禁、7 种报告格式 + docx 合规模板 + 整改复测。  
  
三、桌面端：单个 exe，双击即用  
  
对没有 Python 环境的机器，这是最省事的用法。引擎、63 个 POC 和全部报告模板都在编译期嵌入同一个文件，无需 Python / Docker / 任何环境配置。  
- ruoyi-scan.exe：终端版，约 38 MB，命令行使用；  
  
- ruoyi-scan-desktop.exe：单文件版，约 49 MB，免安装双击即用；  
  
- pip install ruoyi-scan：适合 CI / 服务器环境。  
  
工程细节也做得扎实：引擎子进程挂 Windows JobObject，壳被强杀时引擎树自动回收；NSIS 卸载会清理运行时缓存；桌面端支持 Ed25519 验签的在线更新。  
  
四、快速上手  
  
命令行版  
```
pip install ruoyi-scan
ruoyi-scan -p http://target:8080/
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/gL9yql6ibrhL6CWuU8e53vo3k44xrG7zIibHJvH5e4ob6OS1AiaqCF0L4mHKvClwUfqSxttklsaw32IFvsL4BwJo7aicCTyRgib9JMXqPVH94v2M/640?wx_fmt=png&from=appmsg "")  
  
综合扫描（目录 + 漏洞 + 爆破）：  
```
python main.py -u http://target:8080/
```  
  
生成全格式报告：  
```
python main.py -p http://target:8080/ --report ./reports --report-format all
```  
  
GUI版  
  
效果如下-适合小白简单上手  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/gL9yql6ibrhLRXma8IgwNjEZfkJPSTwUFh8t9O6kTCc6E8TV5l3H5LIPFkbkzSkzrFjpGYPBM7T6KDVKCrpfJInpJ9I5UFXTMaFtYeR1XibCI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/gL9yql6ibrhJiaMum5WzIRw5FS5lRThHtZZoHia0IgbFNPeM1yf5J2GN7QIohobAQI303qTk7udMNM4UnQia8P5YGmXyWDHDiaBNTJibsibOZmqa1o/640?wx_fmt=png&from=appmsg "")  
  
github:https://github.com/xiabai2008/ruoyi-scan/releases  
  
[鹏组安全社区站：您身边的安全专家-情报 | 攻防 | 渗透 | 线索 | 资源社区](https://mp.weixin.qq.com/s?__biz=Mzg5NDU3NDA3OQ==&mid=2247491205&idx=1&sn=b212739965f6617c84c89726cc85d50c&scene=21#wechat_redirect)  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0YvAy5BgkyNHF1CWPJ9XSApBFhIGwF5Jh0zD2ySOcHvBkYgicU4xZsqvR3XEjUEnfGKH7ya8TgqCibHpYZKcibDBQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=28 "")  
  
**扫码关注**  
  
**社区**  
  
鹏组安全社区：  
comm.pgpsec.cn  
  
  
专注网络技术与骇客的一个综合性技术性交流与资源分享社区  
  
  
老用户续费88折扣  
![图片](https://mmbiz.qpic.cn/mmbiz_png/gL9yql6ibrhKsNuLia8k78ene1GicuBHwpLfA3LtQ591eDic3aIbcg76HGbhYtXiawES5GRyibgACRhWglt23qty51tIga2nayicfNEv9zMDYibceck/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=3 "")  
  
  
社区首页  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0YvAy5BgkyN92OtiagxgUpDAeq8RbcPacH8L82CwLzHtvucDrP1RrgfzeUYY8cS4WHk8niap3jKZzys9wK5oHB9w/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=29 "")  
  
互助中  
心  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/gL9yql6ibrhJFyJkLWKAxhw3ibLIuEwITd3CmkY1U3NHMQX6Ub6nEahq4iaumsEuVchXG4LN4rMG7s2kibVtkd3Z5Ao2qd9Lxr8HdZJOB9e1MC4/640?wx_fmt=png&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=18 "")  
  
免责声明  
  
由于传播、利用本公众号鹏组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号鹏组安全及作者不为  
此  
承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！  
  
好文分享收藏赞一下最美点在看哦  
  
