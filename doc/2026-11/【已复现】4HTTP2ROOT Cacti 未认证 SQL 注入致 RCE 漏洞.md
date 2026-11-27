#  【已复现】4HTTP2ROOT Cacti 未认证 SQL 注入致 RCE 漏洞  
原创 易富贤
                    易富贤  源影安全团队   2026-11-27 02:18  
  
Cacti 是一套开源的网络监控与性能绘图系统，通常以 LAMP（Linux、Apache、PHP 加 MySQL/MariaDB）方式部署，底层用 RRDtool 存储时序数据。它通过 SNMP、脚本等方式周期性采集路由器、交换机和服务器的流量与性能指标，再把结果绘制成图表供运维集中查看。系统采用“设备—数据源—图形—轮询”的模型，由后台 poller 定时执行采集任务；为方便免登录查看图表，它还提供了 guest（访客）账号机制。  
  
该漏洞被研究者命名为“4HTTP2ROOT”，实质是一条未认证的存储型（二次）SQL 注入进而达成操作系统命令执行的利用链，前提是管理员开启了访客访问。攻击者无需任何凭据，用四个 HTTP 请求即可完成：先访问 auth_profile.php 取得访客会话和 CSRF 令牌，再经 update_data 把一段未经校验的字符串写入 settings_user.preview_graphs_per_page；随后请求 graph_view.php 的 preview 动作时，该值被当作图形数量拼进数据库查询的 LIMIT 子句，借 PDO 多语句执行堆叠一条 UPDATE，篡改 poller_item.arg1。  
  
最后请求 graph_realtime.php 的 init 动作，触发 cmd_realtime.php 通过 popen 执行被改写的命令，并把结果写回可访问的文件，从而完成整条链路。靶场实测该命令以 www-data 身份运行，可读取 /etc/passwd 等敏感文件；只有当后端轮询进程以 root 运行时，危害才会升级为 root，因此名称里的“到 root”是有条件的，默认部署通常止步于 Web 服务用户。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/gRYowTXTvLHOqbGjzqMjoNZiaHibIY1Xdw31OtDpQT8ymv7lbiaq5ricCl8MicCaxu4iaPau06Hnxia181JdJeE8lDVRmzRLZKLvRBBiaxKr3AnYga8/640?wx_fmt=png&from=appmsg "")  
  
  
