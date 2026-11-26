#  【安全圈】Drupal紧急修复高危在野漏洞  
安全圈
                    安全圈  安全圈   2026-11-26 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Drupal 官方安全团队正式发布高危安全公告（SA-CORE-2026-019），披露一个编号为 CVE-2026-136716 的核心层高危漏洞，并确认该漏洞已遭遇在野实网利用。攻击者可滥用网站异常或未完成的安装状态，无需任何鉴权凭据即可直接接管站点管理员（User 1）控制权。统计显示全球仍有超 3 万个公网暴露节点处于受影响分支。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyGQjWDiam9HIwvTV4dicf9xdqjEl82B6ZA0HQfTOoBRSIM5WrIZLpq4WYRaXiaiblnGf65oqelC6PKhxktj4B5hpNyQsuUic6nO4xzQ/640?wx_fmt=other&from=appmsg "")  
  
此项漏洞与近期披露的 CVE-2026-132165（代号 Fail2Install）紧密交织。虽然此前补丁尝试修补核心安装脚本（install.php）中的鉴权缺陷，但分析证实部分特定配置与残留安装态的生产站点仍能被黑客绕过验证，直接重入安装流程实现提权。  
## 🔍 漏洞机理分析：隐蔽的安装状态条件检查缺陷  
  
CVE-2026-136716 归属于 CWE-754（异常或例外条件检查不当），其技术危险性主要集中在以下三个维度：  
- **安装态逻辑盲区**  
：当生产站点处于某种异常部署分支、未彻底完结初始配置或在上一轮补丁更新后未清理初始化标记时，Drupal 内核未能准确校验当前系统的运行态，误将其判定为待配置环境。  
  
- **前端无任何感知识别**  
：Drupal 官方特别警告，系统前端 Web UI 不会显示任何受损或脆弱状态提示；更危险的是，已被攻击者成功入侵的站点，其安装状态往往已被攻击脚本改写抹平，管理员无法单纯依赖界面状态排查。  
  
- **直接夺取最高权限**  
：通过触碰开放的 install.php 并提交特制环境参数，攻击者可直接重置系统 User 1（超级管理员）账号的邮箱或重绑管理员凭证，实现全站所有代码、数据库及附件的完全掌控。  
  
## 🌐 全球敞口测绘：3.1 万个公网站点面临威胁  
  
根据网络安全研究机构 StepSecurity 发布的全球测绘遥测数据，该漏洞引发的网络安全态势严峻：  
- **公网基数庞大**  
：截至 2026 年 11 月 26 日，全球公网可直接访问的 Drupal 站点约为 55,800 个。  
  
- **超半数版本脆弱**  
：在公网节点中，约有 31,500 个站点正运行着包含安装器脆弱逻辑的版本分支。  
  
- **废弃版本暴露**  
：其中更有近 15,000 个暴露节点仍停留在已彻底停服（EOL）的 Drupal 8 与 Drupal 9 分支上，官方已不再为其提供任何安全更新，成为黑客自动化扫描脚本的活靶子。  
  
## 🛡️ 处置矩阵与官方排查要求  
  
Drupal 官方呼吁所有站长立即对照版本矩阵执行升级并完成侵害排查：  
  
**版本修复升级矩阵**  
：  
  
• Drupal 11.4.x 用户 ➔ 升级至 **11.4.14**  
  
• Drupal 11.3.x 用户 ➔ 升级至 **11.3.23**  
  
• Drupal 10.6.x 用户 ➔ 升级至 **10.6.23**  
  
• Drupal 11.0.x ~ 11.2.x 及 10.5.x 以下版本均已 EOL，官方无修复补丁，必须迁移升级至受支持主干版本。  
  
完成版本迭代后，管理员应按以下规范展开现场加固：  
- **执行 update.php**  
：在最新构建下运行 update.php 规范闭合残留状态。  
  
- **彻底阻断 install.php 外网访问**  
：在 NGINX 或 Apache 接入层配置禁止外部访问 install.php 脚本，或将其文件权限严格收缩。已按官方指引封禁安装器访问的站点不受本次漏洞威胁。  
  
- **核验管理员审计日志**  
：检查系统日志中关于 User 1 账户的最近变更记录，核对绑定的管理员邮箱，并建议主动重置一次超级管理员口令。  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】美中情局确认遭网络入侵：开源情报分析系统失守](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=1&sn=51b9ba7239b6c1d7b4895d8f712c93c9&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Mware曝9.8分高危越权：vCenter目录服务再遭突破](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=2&sn=3241348bb1eaf2254cc0a28b11f7c3a1&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】单比特修改击穿代码完整性：谷歌披露.NET内核漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=3&sn=3bb92f2f5718e148e60b31509a53ae07&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】广联智通（GL.iNet）路由器与KVM遭黑客批量入侵](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080043&idx=1&sn=3aebb0248c3cc0f96c83f4038e02fa45&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】微软Azure APIM经典版遭利用](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080043&idx=2&sn=d8c61ce5bae76c1db49db4a42a7af82c&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
