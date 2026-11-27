#  【安全圈】Cacti在野0-day遭公开：仅需4次HTTP请求即可远程RCE  
安全圈
                    安全圈  安全圈   2026-11-27 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：开源网络监控中枢 Cacti 突发严重在野 0-day 漏洞，PoC 与完整利用链已在开源社区彻底公开，研究社区代号追踪为“4HTTP2SHELL”及“4HTTP2ROOT”。攻击者无需账号登录、无需有效 API 凭据、无需任何钓鱼交互，仅凭默认开放的访客（Guest）权限，向目标服务器发出 4 次连续 HTTP 请求，即可在最新版本 Cacti Core 1.2.31 上直接实现远程代码执行（RCE）。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyEia5W1PJXlJiawagA5DjryvvLeWVCJ89xrDTia8ibhOxDsZaP1t424rDibkQHh9QPMvpYM1vZibNyQEZNNWia8U6piaduhrFSmKR8oU5E/640?wx_fmt=other&from=appmsg "")  
  
漏洞由独立安全研究员“ek0ms savi0r”公开披露。由于大量企业和数据中心将 Cacti 部署在管理网络核心，并配置了针对路由器、交换机、核心服务器的 SNMP 凭据与巡检权限，该漏洞一旦遭遇实网撞库，将成为打穿企业内网纵深的关键跳板。  
## 🔍 4次请求漏洞链深度拆解：从配置写入到 popen() 触发  
  
分析显示，整个利用链极为精巧，四次 HTTP 往返环环相扣，依次击穿了四道安全防御假设：  
- **第 1 次请求（篡改数字配置）**  
：以 Guest 身份请求 auth_profile.php?action=update_data  
。Cacti 允许访客更新用户配置偏好，其中名为 preview_graphs_per_page  
 的参数控制预览图表分页数，本应为严格整数，但后台缺失数值型强校验，允许将任意字符串原样写入数据库。  
  
- **第 2 次请求（触发二阶 SQL 注入）**  
：请求图表预览页面 graph_view.php  
。系统取出上一步持久化的恶意字符串拼入 SQL 查询中的 LIMIT 语句。  
  
- **第 3 次请求（PDO 堆叠查询篡改 poller 表）**  
：Cacti 底层使用的 PHP PDO 数据库驱动支持堆叠查询（Stacked statements）。攻击者借此执行 UPDATE 语句，直接篡改 Cacti 内部负责调度命令采集的核心数据表 poller  
，在其中写入后门命令。  
  
- **第 4 次请求（调用实时轮询端点触发 RCE）**  
：访问实时轮询端点 graph_realtime.php  
。该脚本从 poller 表中取出被篡改的记录，未经过任何白名单过滤或清洗直接传入 PHP 的 popen()  
 函数执行，攻击者瞬间获取 Web 服务器用户执行权限。  
  
## ⚡ 致命破坏力：网络中枢失守与凭据大范围失窃  
  
知名威胁监测机构 watchTowr 在专项预警中指出，Cacti 系统的失窃风险不仅限于单台服务器：  
  
**网络凭据高价值汇聚点**  
：Cacti 作为集中监控平台，通常统一纳管内网核心网关、防火墙、负载均衡以及各业务主机，数据库中保存有大量明文或可逆加密的 SNMP Community 字符串、SSH 登录凭据、API Token 以及网络拓扑架构图。黑客通过该漏洞不仅能压制告警，还能以此为枢纽直接纵向横移突破整座内网。  
## 🛡️ 官方补丁缺失期间的紧急缓解方案  
  
截至发稿，Cacti 官方尚未正式发布修复新版本。针对正在全球范围内被扫描的生产环境，运维与安全团队应立即采取以下防御措施：  
- **立即禁用 Guest（访客）账户**  
：进入 Cacti 管理控制台 ➔ Console -> Configuration -> Users  
，将 Guest 账号状态修改为禁用（Disabled）或彻底删除。此举可直接斩断 4 次 HTTP 链条的起点。  
  
- **收缩网络暴露面**  
：绝不能将 Cacti Web 接口直接映射到公网；通过反向代理（如 Nginx）限制访问源 IP，或强制要求先接入企业内网 VPN 后方可访问。  
  
- **排查 poller 数据表存量后门**  
：直接登录底层 MySQL/MariaDB 数据库，执行 SQL 审查指令：SELECT * FROM poller WHERE command != '';  
，核查是否存在非官方预置的外部系统命令调用记录。  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】1Password客户数据被盗](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=1&sn=c793b7ec19968c2a4193647ef032b172&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Drupal紧急修复高危在野漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=2&sn=d0a76361f98bec89bc0c6880d05cc226&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】假面试题暗藏恶意NuGet包](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080067&idx=3&sn=e2707d02974793f7ea6a669aaa8a65c5&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】美中情局确认遭网络入侵：开源情报分析系统失守](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=1&sn=51b9ba7239b6c1d7b4895d8f712c93c9&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Mware曝9.8分高危越权：vCenter目录服务再遭突破](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080056&idx=2&sn=3241348bb1eaf2254cc0a28b11f7c3a1&scene=21#wechat_redirect)  
  
  
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
  
  
