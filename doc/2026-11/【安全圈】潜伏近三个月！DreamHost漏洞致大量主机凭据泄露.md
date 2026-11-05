#  【安全圈】潜伏近三个月！DreamHost漏洞致大量主机凭据泄露  
 安全圈   2026-11-05 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：2026 年 11 月 5 日，美国知名网站托管与域名服务商 DreamHost 正式确认遭遇严重安全事件。攻击者利用其自研 Web 文件管理器的底层软件漏洞，自 2026 年 8 月起秘密潜伏近 3 个月，直至 10 月 23 日官方察觉异常才被强行切断。期间大量用户的 FTP/SFTP 账户密码、已激活的会话令牌（Session Token）以及部分 Webmail 邮箱凭据遭成批窃取，托管服务器上的源码与数据库面临全量拖库及网页挂马风险  
  
## 🔍 攻击链路：Web 文件管理器沦为黑客跳板  
  
DreamHost 提供的文件管理器是帮助站长在浏览器端管理文件的 Web 工具。根据官方初步调查通报，黑客突破与凭据抓取的路径极为直接：  
- **渗透运行服务器**  
：黑客利用文件管理工具中的未公开代码漏洞，直接拿到了承载该服务的基础设施访问权限  
  
- **截获会话与凭据**  
：在攻击生效的长达 80 多天内，只要站长登录过该网页文件管理器，其输入的 FTP/SFTP 用户名、明文/哈希密码便被静默捕获；黑客同时批量导出了用户的登录会话令牌（Session Token），可直接绕过二次验证伪造登录态  
  
- **蔓延至企业邮箱**  
：除主机文件管理系统外，DreamHost 确认部分网页邮箱（Webmail）系统同样遭到波及，少量客户的邮箱账户密码外泄，邮件往来记录面临被窥探风险  
  
## ⚡ 致命破坏力：直通代码库与供应链污染  
  
获得 FTP/SFTP 凭证等于拿到了受害者站点的绝对读写权，黑客可实施两类高烈度攻击：  
- **一键打包核心资产**  
：通过 SFTP 客户端直接将整套网站源码、wp-config.php  
 或 .env  
 等环境变量文件全盘打包下载，其中包含的数据库账号、密码、支付 API Key 随即二次失窃  
  
- **隐蔽注入供应链恶意代码**  
：利用写权限在已有静态 JS 文件、PHP 模板中植入远程恶意重定向脚本或虚假登录表单，当正常访客浏览这些网站时，无差别触发钓鱼或恶意下载  
  
## 🛡️ 站长应急处置清单：四个必做排查动作  
  
所有托管在 DreamHost 上的站点负责人，应立即执行以下加固步骤：  
- **全量轮换凭证并吊销会话**  
：立即登录 DreamHost 控制台重置主账号密码、所有 FTP/SFTP 用户密码、数据库密码，并点击“注销所有已连接设备与会话”  
  
- **比对生产环境代码指纹**  
：使用本地干净的 Git 版本库与服务器现有文件做 git diff  
 比对，重点检查近 90 天内被修改的 index.php  
、核心 JS 文件以及新增的隐藏脚本  
  
- **审计 SSH 公钥配置**  
：检查服务器家目录下的 ~/.ssh/authorized_keys  
，彻底删除任何非本人添加的未知公钥条目，封堵后门跳板  
  
- **检查邮箱转发与自动过滤规则**  
：登录 Webmail 后台，排查是否存在将收件静默转发至未知外部域名的规则，防止重置凭证邮件被黑客截胡  
  
  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】9.8分Linux内核高危RCE！无需认证隔空打穿](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=1&sn=dfbd72f6698a155131d20cada69f5589&scene=21#wechat_redirect)  
  
  
  
[【安全圈】隔空投送秒越狱？安全团队演示攻破iOS锁定模式](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=2&sn=c3a9e1733d1b652c93ff442f8514bdf2&scene=21#wechat_redirect)  
  
  
  
[【安全圈】零知识架构翻车？知名密码管理器mSecure疑似遭拖库](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079699&idx=3&sn=de8663b781d1914ceb2dc416f348a11a&scene=21#wechat_redirect)  
  
  
  
[【安全圈】面试题下毒pyenv！朝鲜黑客假借招聘攻破币安内网](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079687&idx=1&sn=2698a551a0d839d42d48e0600cd1befd&scene=21#wechat_redirect)  
  
  
  
  
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
  
  
  
