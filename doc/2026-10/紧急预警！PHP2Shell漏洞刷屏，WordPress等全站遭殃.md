#  紧急预警！PHP2Shell漏洞刷屏，WordPress等全站遭殃  
零日手记
                    零日手记  随笔漫记安全路   2026-10-21 10:18  
  
10月20日，PHP核心两个未认证RCE漏洞的细节在协调披露前**提前泄露**  
，PHP官方紧急宣布次日发布安全更新。安全社区给这对漏洞起了个直白的名字：**php2shell**  
——PHP直接变shell。  
- CVE-2026-109618 / CVE-2026-109619  
  
- CVSS v4.0评分 **9.2（Critical）**  
  
- watchTowr已成功复现，Huntress和Florian Roth跟进监测  
  
- PoC仓库已经出现在GitHub（16星，exploit开发中）  
  
- **PHP 5.x / 6 / 7.x / 8.0 / 8.1没有官方补丁**  
——只有受支持的8.2~8.5发布修复  
  
一句话概括危害：**向任何PHP站点发一个普通multipart POST请求，就可能以web服务器身份执行任意命令**  
。WordPress、Drupal、Joomla、Magento、各类自研PHP站点，全在射程内。  
  
**根因：session上传进度机制里的UAF**  
  
漏洞位于ext/session/session.c  
的RFC1867上传进度回调（php_session_rfc1867_callback  
）。  
  
关键缺陷：该回调把两个裸指针zval*  
（post_bytes_processed  
和current_file_bytes_processed  
）直接存进了活跃的HashTable结构。  
  
攻击路径四步：  
1. **触发**  
——发一个multipart POST，带PHP_SESSION_UPLOAD_PROGRESS  
字段（该功能**默认开启**  
），激活进度跟踪  
  
1. **释放（Free）**  
——进度更新周期重读session数据时，zend_hash_update()  
用zval_ptr_dtor  
析构旧数据树，那两个指针指向的内存被释放  
  
1. **重占（Reclaim）**  
——攻击者构造的zend_fcall_info  
和zend_internal_function  
结构占住这块刚释放的内存，悬空指针现在指向攻击者控制的伪造函数结构  
  
1. **执行（Execute）**  
——PHP调用zend_call_function()  
时执行的已经是攻击者的代码，最终落地system()  
任意命令执行  
  
经典的堆UAF三连：Free→Reclaim→Execute。 PHP核心、默认配置、无需登录——三要素齐了。  
  
已确认在标准php-src构建上利用成功（8.4.4和8.2.34），PoC还支持通过--syms  
参数适配不同发行版的二进制布局（Debian Bookworm、Ubuntu Noble/Jammy）。  
  
**受影响版本：好消息和坏消息**  
<table><thead><tr><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">分支</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">状态</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">PHP 8.5</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">&lt; 8.5.12</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">8.5.12</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">PHP 8.4</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">&lt; 8.4.27</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">8.4.27</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">PHP 8.3</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">&lt; 8.3.36</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">8.3.36</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">PHP 8.2</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">&lt; 8.2.35</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">8.2.35</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">PHP 8.1及以下</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">EOL</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">无官方补丁</span></strong></td></tr></tbody></table>  
坏消息有两个：  
1. **PHP 8.1及更早版本永远不会收到官方修复**  
——而这些版本在存量环境里占比极高。国内大量老站点、老面板（宝塔老版PHP）、老框架项目还跑在7.4/8.1上  
  
1. 修复只覆盖8.2~8.5，意味着**升级到受支持分支是唯一的彻底解法**  
，EOL环境只能靠缓解措施续命  
  
**缓解措施：一行配置先止血**  
  
漏洞依赖session.upload_progress.enabled  
（**默认为On**  
）。关闭它即可中和攻击向量，且不影响正常文件上传功能：  
  
php.ini：  
```
session.upload_progress.enabled = Off
```  
  
PHP-FPM池配置（/etc/php/*/fpm/pool.d/www.conf  
）：  
```
php_admin_flag[session.upload_progress.enabled] = off
```  
  
改完重启php-fpm生效。绝大多数业务根本不用这个上传进度条功能（前端进度条基本靠JS自己算），关掉几乎零成本。  
  
**运维侧的检查与加固：**  
1. **立刻盘点全部PHP环境**  
——公网Web站点、内网设备、托管面板（cPanel/Plesk）、Docker镜像里的PHP版本  
  
1. 受支持分支：**升级到8.2.35 / 8.3.36 / 8.4.27 / 8.5.12**  
  
1. EOL版本：关闭upload_progress + WAF规则 + 限制公网暴露，条件允许尽快升级  
  
1. **WAF拦截**  
：拦截multipart请求中的PHP_SESSION_UPLOAD_PROGRESS  
字段即可阻断触发条件  
  
1. **容器注意**  
：docker build --pull  
重建镜像，缓存的基础镜像不带补丁——多个项目踩了这个坑  
  
**入侵排查（如果晚了）：**  
  
利用成功后最典型的特征是**php-fpm/php-cgi/apache worker直接拉起异常子进程**  
：  
- shell类：/bin/sh  
、/bin/bash  
、cmd.exe  
、powershell.exe  
  
- 下载工具：curl  
、wget  
、nc  
、certutil  
  
- 侦察命令：whoami  
、id  
、uname -a  
  
排查建议：检查web根目录和/tmp  
、/var/tmp  
有没有落地webshell；翻PHP-FPM和web服务器日志里的multipart POST；没有EDR的环境可以上Florian Roth的process-watcher观察模式。发现异常立即隔离主机、保留日志。  
  
**时间线与社区反应**  
- **10月20日**  
：漏洞细节泄露，PHP官方发布"Upcoming Critical Security Release"预告  
  
- **10月21日 14:00 UTC**  
：各发行版和PHP官方同步放出补丁包（8.5.12/8.4.27/8.3.36/8.2.35）  
  
- CERT-EU同步发布安全通告（2026-018）  
  
- 补丁发布当天，mailcow、FrankenPHP、各类Docker镜像项目连夜跟进重建，多个项目同时在php.ini里关闭upload_progress做防御性加固  
  
- watchTowr确认复现CVE-2026-109618，PoC仓库在补丁发布当天即上线  
  
这个节奏说明：**细节泄露在先，补丁窗口极窄**  
。10月20日到21日之间，全网已经存在"知道漏洞但没补丁"的窗口期，这段时间内的利用无法事后从日志里假设不存在。从10月20日起暴露在公网的PHP服务，都应该按"可能已被探测"处理。  
  
**结语**  
  
php2shell的组合拳打在了一个尴尬的位置：**默认开启的功能+核心代码内存安全缺陷+EOL版本海量存量**  
。  
  
"PHP已死"喊了十年，但互联网仍有超过一半的网站跑在PHP上，其中相当比例停留在7.x/8.0/8.1。这次漏洞等于对这些环境下了一份最后通牒：要么升级，要么先关配置，要么接受被一个multipart请求拿下的风险。  
  
今晚就把那行配置改了吧——session.upload_progress.enabled = Off  
。  
  
**参考来源**  
：php.net官方公告、CERT-EU 2026-018、watchTowr复现确认、GitHub PoC仓库（eness/php2shell-CVE-2026-109618-CVE-2026-109619-PoC）  
  
