#  紧急！PHP2Shell高危漏洞来袭，全球超七成网站面临未授权RCE威胁  
 聚铭网络   2026-10-22 09:30  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/uaicMoGl6iaSz34icEtlMpNrZxACe3gTguJiahs3ias2D2kS6ozGicAjicWicluQ1LHzMVaFmAuGP4ibdB4nJY8Y1Dmxgcg/640?wx_fmt=gif "")  
  
  
  
******安全通告**  
  
  
2026年10月21日，PHP官方紧急披露  
远程代码执行漏洞  
“PHP2Shell”，涉及  
CVE-2026-109618与  
CVE-2026-109619，CVSS v4.0评分高达  
9.2分。未经身份验证的攻击者只需发送特制请求，即可远程执行代码、获取服务器控制权。  
PHP 是广泛使用的服务器端脚本语言，支撑全球约七成网站业务。  
漏洞披露后，  
荷兰、芬兰、法国等国  
政府相关网站及公共服务站点相继采取临时下线或紧急维护措施。  
  
  
**漏洞概况**  
  
  
本次披露的漏洞位于PHP核心引擎的内存管理与错误处理逻辑中，相关内存安全问题已存在十余年，此前因利用链复杂被认为难以实际利用，本次借助AI模型辅助分析  
才得以还原利用路径  
。  
  
其影响范围覆盖PHP-FPM、Apache mod_php、php-cgi、FrankenPHP及LiteSpeed等主流部署形态。  
其中Apache mod_php环境风险最高——  
研究人员已完成免登录RCE利用验证，并在最新WordPress、Drupal、Joomla环境复现攻击。  
- CVE-2026-109618（OOM处理缺陷）：攻击者构造特殊multipart POST请求，在不突破post_max_size的前提下触发内存超限；PHP在OOM中断时请求内存未完成初始化，后续清理流程处理无效内存状态，造成内存损坏并最终触发远程代码执行。  
  
- CVE-2026-109619（跨请求残留全局状态）：工作进程处理致命错误后，全局指针仍指向已释放内存；同一进程处理后续请求时复用过期状态，触发use-after-free，可造成内存信息泄露或远程代码执行，泄露密钥、Cookie、接口凭证等敏感数据。  
  
**影响范围与修复版本**  
  
  
受影响版本为PHP 8.2.x、8.3.x、8.4.x、8.5.x全部未修复版本。  
  
官方已发布安全补丁，请及时更新至最新版本：  
  
- PHP 8.5.12  
  
- PHP 8.4.27  
  
- PHP 8.3.36  
  
- PHP 8.2.35  
  
下载地址：https://www.php.net/downloads.php  
  
  
  
对于PHP 8.1及更早停服版本（7.4、8.0等），PHP团队仅提供源码回移补丁，不提供官方二进制包。相关用户可向操作系统发行版或LTS厂商获取补丁，或尽快升级至受支持版本。  
  
  
  
**聚铭网络安全建议**  
  
  
1、全面清点PHP资产  
  
PHP的部署形态较为分散，除常规Web应用外，容器镜像、cPanel/Plesk等控制面板、厂商设备内置运行时、Windows环境捆绑的php-cgi.exe均可能受影响，建议优先排查面向互联网的mod_php与FrankenPHP实例。  
  
2、打补丁后务必重启服务  
  
仅安装修复包不会把补丁加载进运行中的进程，必须重启Apache、PHP-FPM及所有长期运行的PHP worker，补丁才能真正生效。  
  
3、升级后复核内存参数  
  
补丁引入了硬性内存限制，达到阈值即终止进程。升级后应复核memory_limit与post_max_size配置，避免业务因内存限制异常中断。  
  
4、  
无法升级时的临时防护  
  
对无法升级至PHP 8.x的老旧系统，建议部署已覆盖CVE-2026-109618/109619专项虚拟补丁、支持持续规则更新与托管运营的商业/云WAF 作为边界补偿控制。  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/uaicMoGl6iaSyDDwiaZW3NekPKKmKWzqF2oSiakCV4ETINEOMlxibleGPORgbfXO4GiaNpGyqmdqRY3Yaf84WmeP12Kg/640?wx_fmt=gif&from=appmsg "")  
  
**结 语**  
  
本次PHP2Shell属内核级高危漏洞，攻击链简单，PoC存在公开扩散风险  
。聚铭网络提醒用户尽快开展资产排查，优先完成PHP版本升级，防范攻击者利用漏洞  
入侵、植入木马、窃取核心业务数据。  
  
针对本次漏洞，  
聚铭网络全线安全产品已完成检测与防护能力适配，覆盖  
聚铭下一代智慧安全运营中心（AISOC）、  
聚铭网络流量智能分析审计系统（iNFA）、  
聚铭网络脆弱性扫描系统（CSV）以及  
聚铭Web应用防护系统（WAF）  
等，助力用户及时发现并拦截相关攻击。  
  
聚铭网络将持续关注漏洞后续利用与检测规则更新。排查或修复过程中如需技术支持，可拨打官方咨询热线400-1158-400。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/uaicMoGl6iaSwibWPeKNIiazrDQODQn9gRia7g8a1AicicnMffTicEUd3wC5zsR6VgiaJhGj9l7ViaGPTac6fF4xUVtiaITcA/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aKicBJwHBSq7DvexQVaXvK6JbrU8ia8StpjOgQbxUVvcCCP9RpejwSSRhibENxt9KusmRRToSzgCq32h8XWCpLqLzyFJrfcCicI7SyamPmOFFiao/640?wx_fmt=gif&from=appmsg "")  
  
  
**★**  
  
  
  
**往期回顾**  
  
  
  
**★**  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzMDQwMjg5NA==&mid=2247492709&idx=1&sn=d13d438443bd9246e1020f605120d860&chksm=e8b15a44dfc6d3525bd1ff3c187ac55be18b1f9ab41c00916b75d128ae158dc2502d7956e794&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzMDQwMjg5NA==&mid=2247508361&idx=1&sn=4ac35feac3090b51dbee15d6bf30f9a0&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzIzMDQwMjg5NA==&mid=2247508847&idx=1&sn=d9b1d6392e2c912e34546929e0ed905c&scene=21#wechat_redirect)  
  
  
  
