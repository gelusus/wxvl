#  安天发布PHP2Shell漏洞专项系列加固工具  
 安天集团   2026-10-25 06:25  
  
点击上方"蓝字"  
  
关注我们吧！  
  
  
##   
  
PHP2Shell高危漏洞涉及CVE-2026-109618、CVE-2026-109619，CVSS 4.0评分达9.2，属于严重级别，影响PHP 8.1至8.5多个版本。攻击者无需认证即可尝试远程利用，成功后可能执行任意命令、植入WebShell、控制服务器，并进一步实施数据窃取、业务篡改和内网横向渗透。  
  
升级官方修复版本仍是根本处置手段，  
但目前官方补丁有效性遭  
到  
了社区质疑，也存在  
第三方定制PHP等环境中  
补丁无法实施等情况  
。  
因此，运营者同样需要重视环境配置加固这个基本安全能力。通过配置加固可以  
降低漏洞利用成功率，并限制漏洞利用后的命令执行、恶意文件写入、持久化和横向移动等行为。  
由于安全配置点众多，而且有复杂的业务关联影响，有些可以作为常态化的安全策略，有的则适合在缺乏补丁或者补丁无效情况下作为应急手段。  
  
针对本次两个重大漏洞，  
安天  
紧急研发了  
PHP2Shell漏洞专项系列加固工具，面向Linux和Windows Server两个主要载体  
，并进一步适配统信UOS、银河麒麟等x86_64国产化操作系统  
。依托安天AVL Code智能体协同分析，  
安天工程师  
分别梳理了Linux平台212项、Windows平台195项相关配置规则  
，逐项明确默认值、建议值、防护作用、适用条件，以及可能影响业务连续性的副作用和配置代价。  
  
基于安天新版配置框架，上述知识被转化为可扫描、可加固、可验证、可回滚  
的专项工具。每项规则均配套变更影响、效果校验和回滚方案，执行过程遵循预检预览、快照留存、不强制覆盖原有配置的原则，帮助运维与安全人员清楚掌握“改什么、为什么改、付出什么代价、如何验证、如何恢复”。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XBFaicYdOHk9efyu5HOnVhq5colibOBno4nKCVicRkibhMQajEYzWeWHZh9AUzYUd1EpUMzAEYx7MAGB2zxspFt5ZbmSjYjrtyw7KdOy4VBmgzc/640?wx_fmt=png&from=appmsg "")  
  
图1 Windows版PHP2Shell漏洞专项加固工具界面   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XBFaicYdOHk83spLoWOwVjfj1luHj00ur47NiaqFtISa6NNRTUOzfvYVHdYnwCK9RC2JibMFvE8uAD6GpyrZAzTKc4o6C0LJAoGVhTOOibkVps8/640?wx_fmt=png&from=appmsg "")  
  
  
图2 Linux版phphard命令行工具运行界面  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHkibWicicLibOxBATib2L6IiapnXyEAUr0DwCMBIsGhicsx4Br5iadicPuS1tIDFibaMCiaicqt1DHj7KFyqPQREYbHEfal1MUCPY4BOEWN2JMI/640?wx_fmt=png&from=appmsg "")  
  
图3 信创版PHP2Shell漏洞专项加固工具运行界面  
  
10月24日起，系列工具先后在安天垂直响应平台（vs.antiy.cn）发布Windows版本、Linux版本（Ubuntu/Debian）和信创版本（麒麟/统信），并在“安天垂直响应”服务号公开。  
  
该系列工具用于在补丁升级完成前快速收敛风险暴露面，并作为补丁修复后的叠加防护，不能替代官方补丁，配置加固成功也不等于漏洞就一定无法利用  
。  
配置  
工具均为纯本地绿色工具，无任何联网行为。  
  
本批次配置工具安天开始启用了新版的系统配置框架，当前Windows版本已经基于新框架制作，其他平台版本后续更新启用。  
此前配置框架中，为了让网络管理者了解配置项，配置项知识文件是作为明文内置的在配置工具中的。但由于少数友商，未将安天签名加入白名单，同时其字符串加权的误报机制会误报安天的工具，因此从本版本开始安天配置文件以密文方式存放。  
  
附录：配置点分类表  
  
表 1 Linux配置点分类表  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHk9Ib7Hmt1ZLc8Wg3bQF9ZSLzJTjO9K6iaC017q7QbGoQt8BicVictea2ZapmIebjd6zUu470ozjmQJZNHo0ML1rTRPsa5x71WxfHg/640?wx_fmt=png&from=appmsg "")  
  
表 2  Windows配置点分类表  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XBFaicYdOHkib0TsBn6X8p6hl5ibEfrE0WZhrV118oMySia8EgVNuzySibSbFDPewKdTTSyPz6NQ9YXjh626ibQ7SQSMRtUGN10BtIjCLj9OCMr48/640?wx_fmt=png&from=appmsg "")  
  
  
**往期推荐:**  
  
  
  
  
  
# PHP2Shell漏洞处置与响应指引  
  
  
  
[CVE-2026-109618、CVE-2026-109619（PHP2Shell）漏洞FAQ](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215632&idx=1&sn=58bb9d14311e8757f28334bddf6394be&scene=21#wechat_redirect)  
  
  
  
[关于PHP核心远程代码执行漏洞的风险提示及研判思考](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215619&idx=1&sn=f05d1b820db4feca2afaa773b09fe016&scene=21#wechat_redirect)  
  
  
  
[2026年10月微软补丁日风险提示](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215613&idx=1&sn=3e5649d5630933cccfa8626987b1efd4&scene=21#wechat_redirect)  
  
  
  
[2026中国网络安全创新创业大赛 | 安天AVL Code获奖](https://mp.weixin.qq.com/s?__biz=MjM5MTA3Nzk4MQ==&mid=2650215515&idx=1&sn=1dea52c669e041d35f049f0fdce5bb1e&scene=21#wechat_redirect)  
  
  
