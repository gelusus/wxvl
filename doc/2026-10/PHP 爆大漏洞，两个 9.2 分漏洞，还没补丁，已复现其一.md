#  PHP 爆大漏洞，两个 9.2 分漏洞，还没补丁，已复现其一  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-21 05:44  
  
QUOTE  
  
PHP 官方提前一天发布公告，两个严重漏洞将于 10 月 21 日 14:00 UTC 修复。截至目前，  
   
受影响版本区间与具体成因均未公布  
   
。  
  
2026 年 10 月 20 日，PHP 安全团队发布事前公告，宣布将于 10 月 21 日 14:00 UTC 发布安全版本，修复  
   
两个严重漏洞  
   
。官方对两者的定级相同：PHP 核心远程代码执行，CVSS v4.0 评分   
9.2  
   
。  
  
官方公告只写到「两个 critical 漏洞、同一评分、同一发版时间」。受影响版本区间、配置要求、所在函数、CWE 类别，均要等发版时才会公布。  
  
01  
  
OVERVIEW  
### 漏洞简述  
  
  
1.1 两个编号的漏洞差异  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">维度</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">CVE-2026-109618</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">CVE-2026-109619</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">官方确认</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">是</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">是</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">官方定级</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">核心 RCE，CVSS v4.0 9.2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">核心 RCE，CVSS v4.0 9.2</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">受影响版本</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">未公布</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">未公布</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">成因与触发条件</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">未公布</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">未公布</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">第三方复现声明</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">watchTowr 称已复现，细节未公开</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无公开声明</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">公开可用 PoC</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">在野利用</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无确认</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无确认</span></section></td></tr></tbody></table>  
  
  
1.2 当前可验证的攻击面  
  
公开情报把两者的攻击面指向 PHP 会话上传进度处理路径，即   
ext/session/session.c  
   
中的   
php_session_rfc1867_callback()  
   
。该功能自 PHP 5.4 引入，  
   
session.upload_progress.enabled 出厂默认 On  
   
。  
  
它是目前  
   
唯一能在本地完整验证、且与多家厂商描述不冲突  
   
的入口。  
  
源码层面存在两个结构上值得怀疑的位置。  
  
长度与缓冲区解耦  
  
value_len 取自 *data->newlength，指向调用方栈上变量，可被前序回调改写；而 *data->value 指向的缓冲区由 multipart 读取器分配。两者若不保持一致，随后的 ZVAL_STRINGL 与 smart_str_appendl 就是越界写。  
  
生命周期与引用计数  
  
一次请求内 php_session_initialize() 会被多次调用，每次进度更新触发一次，而 progress->data、progress->current_file、progress->sid 都是跨调用存活的裸指针或内联 zval。多次 track_init 与 zend_hash_update 之后，只要有一处引用计数算错，就会出现释放后使用。  
  
  
02  
  
IMPACT  
### 影响范围  
  
  
2.1 补丁覆盖边界  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">分支</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">状态</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">官方修复</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">8.5</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">活跃支持</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">有</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">8.4</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">活跃支持</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">有</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">8.3</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">安全维护</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">有</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">8.2</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">安全维护，2026-12-31 结束</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">有，剩两个多月</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">8.1 及更早</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">已 EOL</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">无</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">5.x / 7.x</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">已 EOL</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">无</span></section></td></tr></tbody></table>  
  
PHP 5.x、7.x、8.0、8.1 的资产从补丁发布那一刻起进入无官方修复状态。  
  
2.2 部署形态  
  
漏洞位于解释器内部，受害面不是使用了某个特定框架的站点，而是  
   
所有由 PHP 处理请求的服务  
   
。  
  
共享主机与多租户面板，WordPress、Drupal、Joomla、MediaWiki、phpBB、WHMCS，Magento 与 Adobe Commerce、Shopware、WooCommerce，Laravel 与 Symfony 等框架应用，Docker 的 php:*-fpm 与 php:*-apache 基础镜像，厂商随产品捆绑的 PHP 运行时。  
  
捆绑运行时这一类最容易漏掉  
   
。它不出现在依赖清单里，也不出现在 SBOM 里，运维看到的是某个管理系统版本，看不到它内部跑着哪个 PHP。  
  
2.3 容易漏掉的三类资产  
  
容器镜像  
  
FROM php:8.2-fpm 锁定的是 tag，tag 不会自动跟随安全版本更新。多数团队的镜像扫描只看操作系统包，不看 PHP 自身版本。  
  
空壳站点  
  
只做健康检查、只返回静态内容的 index.php，在资产表里通常被标为低价值。但只要它由 PHP 处理、接受 POST，就是一个可用的触发点。  
  
反向代理后的老系统  
  
前面是 nginx，后面是 PHP 5.6 的老业务。nginx 不是 PHP，补不了这个洞。  
  
2.4 两个编号的差异  
  
目前无法拆分。受影响的最低版本、是否影响某个特定 SAPI、是否依赖特定配置、是否与另一个编号连锁，全部未公布。任何给出精确区间的说法目前都没有官方依据。  
  
  
03  
  
PRECONDITIONS  
### 利用条件与场景  
  
  
3.1 四条共同条件  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">条件</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">默认值</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">PHP 版本落在受影响区间</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">视资产，未公布</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">HTTP 暴露（FPM / mod_php / CGI）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">视部署</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">session.upload_progress.enabled = On</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">On</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">请求能被 PHP 解析（任意 .php 入口）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">视站点</span></section></td></tr></tbody></table>  
  
第 3 条是影响面覆盖大量默认配置的原因。管理员不需要做错任何事，这项本来就是开的，官方 php.ini 里它处于注释状态，注释即默认值。  
  
3.2 三个常见误判  
  
误判一：网站没有上传功能  
  
会话上传进度是 PHP 会话扩展的内置功能，不是业务代码。SAPI 解析 multipart 请求体时发现 PHP_SESSION_UPLOAD_PROGRESS 字段，就会把进度写进会话存储，这个动作发生在脚本编译执行之前。触发与有没有上传接口无关。  
  
误判二：需要登录才能访问后台  
  
处理早于路由、早于鉴权、早于任何应用代码。攻击者不需要知道站点上有什么页面。  
  
误判三：有 WAF 就安全  
  
如果触发点在请求体解析，请求在进入 WAF 规则匹配之前已经被 PHP 处理过一轮。签名要等 PoC 公开才写得出来。  
  
3.3 cleanup 变量的影响  
  
session.upload_progress.cleanup 默认 On，请求结束后删除进度数据。cleanup 为 On 时进度数据只在请求生命周期内存在，会话文件在请求结束后被清空为空壳，攻击窗口是请求处理期间，对内存破坏型利用够用，对持久化写入不够。cleanup 为 Off 时数据持久化在会话文件里，通常是运维为了做进度条手动改的，属于少数配置。这个变量决定攻击面能否变成持久化写入原语，但不决定两个编号是否成立。  
  
3.4 典型场景  
  
空壳站点  
   
　一个只返回 hello 的 index.php 就是一个可用的触发器。  
  
共享主机  
   
　会话目录常常全局共享，站点之间只靠文件权限隔离。目录权限宽松时，影响会横向扩散。  
  
容器  
   
　构建一次，跑三年。补丁发布后 pull 到的仍是旧层，除非显式改 tag 或重建镜像。  
  
供应链  
   
　WHMCS、部分 ERP、国产 CMS、网络设备管理面自带 PHP 运行时，在补丁窗口里几乎必然被漏掉。  
  
  
04  
  
REPRODUCTION  
### 复现  
  
  
4.1 复现级别定义  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">级别</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">定义</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">109618</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">109619</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">L0 入口与原语</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">请求能走到目标处理路径并产生可控副作用</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">已复现</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">已复现，属共享入口</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">L1 崩溃</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">触发内存破坏，进程崩溃</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">未复现</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">未复现</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">L2 RCE</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">控制流劫持，执行任意命令</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">未复现</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">未复现</span></section></td></tr></tbody></table>  
  
L0 是入口层面的验证，不是归因。在官方拆分之前，这条路径无法单独归给某一个编号。  
  
4.2 复现环境  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">项目</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">值</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">运行时</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">官方镜像 php:8.5.11-cli / php:8.5.11-apache</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">PHP 版本</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">PHP 8.5.11 (cli) NTS / Apache 2.4.69 mod_php</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">会话处理器</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">files</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">关键配置</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">upload_progress.enabled=On，cleanup=Off（观察用）</span></section></td></tr></tbody></table>  
  
用容器而非本地编译，是因为官方镜像就是目标版本，构建方式可复现。本地编译 8.5.11 需要在 --disable-all 之上自行拼装扩展，反而引入变量。  
  
4.3 PoC-1：最小报文  
  
把入口压到最小的 multipart 请求，221 字节。  
  
.  
.  
.  
http  
  
POST /empty.php HTTP/1.1  
  
Host: 127.0.0.1:18899  
  
Cookie: PHPSESSID=poc0001  
  
Content-Type: multipart/form-data; boundary=----poc  
  
Content-Length: 221  
  
  
------poc  
  
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"  
  
  
MARKER_109618_109619  
  
------poc  
  
Content-Disposition: form-data; name="file"; filename="a.txt"  
  
Content-Type: text/plain  
  
  
X  
  
------poc--  
  
四个约束，少一个就不生效。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">约束</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">原因</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">Content-Length 与字节数完全一致</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">解析器按长度读，差一个字节就截断</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">PHPSESSID 只含 [A-Za-z0-9,-]</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">文件 handler 在 mod_files.c:168 校验</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">必须带一个文件 part</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">会话初始化发生在 FILE_START，纯表单字段不会落盘</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">目标脚本存在即可</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">处理发生在脚本执行之前，不需要它调用 session_start()</span></section></td></tr></tbody></table>  
  
PITFALL  
  
第二行这条容易踩坑。用带下划线的 SID（例如 probe_0001）时，PHP 会  
   
静默丢弃会话  
   
，请求照样返回 200，但会话目录里没有文件。排查时很容易误判成漏洞不存在。实际排查中我在这上面绕了一圈，最后用对照实验才定位到字符集校验。  
  
4.4 实测输出  
  
时序取证脚本在脚本第一行读取状态。  
  
.  
.  
.  
php  
  
<?php  
  
header('Content-Type: text/plain');  
  
$sid = $_COOKIE['PHPSESSID'] ?? '(none)';  
  
$file = rtrim(ini_get('session.save_path'), '/') . '/sess_' . $sid;  
  
  
echo "session_status_at_script_start=" . session_status() . "\n";  
  
echo "session_file_exists_before_app_code=" . (is_file($file) ? 'YES' : 'NO') . "\n";  
  
echo "session_file_size=" . (is_file($file) ? filesize($file) : 0) . "\n";  
  
.  
.  
.  
bash  
  
curl -sS --noproxy '*' http://127.0.0.1:18899/precheck.php \  
  
　-H "Cookie: PHPSESSID=demo0001" \  
  
　-F "PHP_SESSION_UPLOAD_PROGRESS=DEMO_MARKER_01" \  
  
　-F "file=@lab/www/plain.txt"  
  
.  
.  
.  
output  
  
sid=demo0001  
  
session_status_at_script_start=1  
  
session_file_exists_before_app_code=YES  
  
session_file_size=364  
  
session_status() 返回 1，即 PHP_SESSION_NONE，说明脚本执行时会话尚未开启；而会话文件此时已存在。落盘内容：  
  
.  
.  
.  
session  
  
upload_progress_DEMO_MARKER_01|a:5:{s:10:"start_time";  
  
i:1792558975;s:14:"content_length";i:364;  
  
s:15:"bytes_processed";i:364;s:4:"done";b:1;  
  
s:5:"files";a:1:{i:0;a:7:{s:10:"field_name";s:4:"file";  
  
s:4:"name";s:9:"plain.txt";s:8:"tmp_name";  
  
s:27:"/tmp/php5m67ppuov16gcTMgxqP";s:5:"error";i:0;}}}  
  
4.5 完整验证矩阵  
  
12 项用例，全部通过。  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">用例</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">A 对照：普通 multipart，不带进度字段</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">200，不生成会话文件</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">B1 带 PHP_SESSION_UPLOAD_PROGRESS</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">200，生成 sess_&lt;SID&gt;</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">B2 写入是否早于应用代码</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">脚本首行观测到文件已存在</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">B3 脚本是否开启会话</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">session_status()=1，未开启</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">B4 数据是否原样落盘</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">标记串完整出现</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">C 文件名是否由 Cookie 决定</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">sess_&lt;SID&gt; 按请求生成</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">D 字段长度是否受业务限制</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">1 MiB，落盘 1,048,929 字节</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">E 对照：SID 含下划线</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">不生成，字符集校验拦截</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">F 对照：SID 路径穿越</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">/tmp/poc_traversal 未创建</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">G 字段值含 NUL</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">原样写入，未截断</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">H 无 Cookie，用 POST PHPSESSID 指定 SID</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">生成 sess_&lt;SID&gt;</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">I 对照：PHPSESSID 排在文件字段之后</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">不生成，FILE_START 时 SID 未定</span></section></td></tr></tbody></table>  
  
D 项说明攻击者能控制多大的写入，唯一边界是 post_max_size 与 upload_max_filesize。对内存破坏型利用来说，可控写入长度直接影响堆布局的可操作性。E 与 F 项说明校验点位置：SID 字符集校验在 ext/session/mod_files.c:168，不在 session.c，换用 Redis 或数据库 handler 时这一层是否等价取决于实现。H 与 I 项说明顺序约束：无 Cookie 时可以用 POST 字段指定 SID，但必须排在文件字段之前，顺序错误会得到静默不生效的假阴性。  
  
4.6 三组对照实验  
  
.  
.  
.  
output  
  
# cleanup = On（出厂默认）  
  
HTTP=200  
  
file=lab/www/sessions/sess_cleanupdemo02 size=0  
  
  
# use_strict_mode = 1  
  
session_file_exists_before_app_code=NO  
  
sess_strictdemo03: ABSENT  
  
  
# upload_progress.enabled = Off  
  
HTTP=200  
  
sess_mitchk01=ABSENT  
  
cleanup 为 On 时文件被创建但请求结束后被清空，默认配置下持久化写入不成立、请求期间存在攻击者可控数据成立。strict_mode 打开后攻击者指定的 SID 未被使用，PHP 改用服务端生成的随机 SID，这把攻击者决定文件名降级为文件名不可预测，但不是银弹。upload_progress 关闭后请求正常，会话文件不再生成，这条缓解实测有效。  
  
4.7 畸形输入阴性结果  
  
.  
.  
.  
probe  
  
[A 超长 boundary (6000B)] -> 200 OK | 存活=YES  
  
[B boundary 上限 5116] -> 200 OK | 存活=YES  
  
[C 进度字段 1 MiB] -> 200 OK | 存活=YES  
  
[D 截断的 part 头] -> 200 OK | 存活=YES  
  
[E 100KB filename] -> 200 OK | 存活=YES  
  
[F 嵌套数组 name[x][y]] -> 200 OK | 存活=YES  
  
[G 5000 个字段] -> 200 OK | 存活=YES  
  
[H 重复进度字段] -> 200 OK | 存活=YES  
  
[I 无结束 boundary] -> 200 OK | 存活=YES  
  
存活检查: OK  
  
随手构造的畸形输入不足以打崩 8.5.11。这组数据同时给补丁后的回归测试留了基线。  
  
4.8 第三方 PoC 的核验结果  
  
社区流传的第三方武器化 PoC 包声称 618 是内层指针的释放后使用，给出 62,935 字节的请求。核验分三步。  
  
第一步，静态阅读，不执行  
  
.  
.  
.  
text  
  
VALIDATION.md §7 :  
  
"No ASAN heap-use-after-free was observed"  
  
  
REPORT_VALIDATION.md §7 :  
  
"NOT PROVEN: Live RCE achieved with the correct 8.2.34 lsphp delta"  
  
Runbook 要求用 ps、gdb -p、readelf 登录 origin 主机取地址，那是先拿到主机权限，不属于远程利用。  
  
第二步，源码复核  
  
它声称缓存的 zval 指针会悬空，但：  
  
.  
.  
.  
c  
  
Z_TRY_ADDREF(progress->data); // :3160  
  
zend_hash_update(Z_ARRVAL_P(sess_var), progress->key.s, &progress->data); // :3161  
  
每次插入前先加引用，zend_hash_update 是 ZVAL_COPY_VALUE 所有权转移，因此调用方必须先加引用，否则 bucket 与 progress 共享同一份引用，析构一次就悬空。第二次 zend_hash_update 析构的是新反序列化出来的树，不是被缓存的那棵。  
  
第三步，精确复刻实打  
  
在 PHP 8.5.11 加 Apache 2.4.69 mod_php 下：  
  
<table><thead><tr><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">运行</span></section></th><th style="background: rgb(39, 39, 42);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 12px;text-align: left;"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">run 1（新 SID）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">HTTP 200，无崩溃，command_in_body=False</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">run 2（新 SID）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">HTTP 200，无崩溃</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">run 3（新 SID）</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);"><section><span leaf="">HTTP 200，无崩溃</span></section></td></tr><tr><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">同一 SID 连续 3 次</span></section></td><td style="padding: 8px 12px;border-bottom: 1px solid rgb(228, 228, 231);color: rgb(82, 82, 91);background: rgb(250, 250, 250);"><section><span leaf="">全部 200，无崩溃</span></section></td></tr></tbody></table>  
  
会话文件证明请求确实走到了目标路径，bytes_processed 记录为 62935，files 数组中第一个元素的 bytes_processed 为 10370，与它自己报告的数字一致。请求是对的，效果是零。  
  
结论是这份包不能证明 618，也不能证明 619，它更接近一份未完成的研究草稿。  
  
4.9 复现结论与补丁后动作  
  
L0 达成，两个编号共享的攻击面与写入原语，12 项验证全部通过，可稳定复现。L1 未达成，618 的第三方崩溃 PoC 三次全败，619 没有触发条件可试。L2 未达成，没有公开可用的利用链。  
  
.  
.  
.  
bash  
  
# 1) 官方 changelog  
  
# https://www.php.net/ChangeLog-8.php  
  
  
# 2) 源码 diff  
  
git clone --depth 200 https://github.com/php/php-src.git  
  
cd php-src  
  
git log --oneline --all --grep='CVE-2026-1096' -i  
  
git diff php-8.5.11..php-8.5.12 -- ext/session main/rfc1867.c  
  
  
# 3) 按 diff 性质定性  
  
# 加上界检查 -> 越界写/读  
  
# 加引用计数/延迟释放 -> 释放后使用  
  
# 加类型或取值校验 -> 代码注入  
  
第三步之后，才谈得上哪个编号对应哪段代码。  
  
  
05  
  
REMEDIATION  
### 修复建议  
  
  
5.1 今天：资产盘点  
  
.  
.  
.  
bash  
  
# Linux  
  
php -v; php-fpm -v  
  
ls /usr/sbin/php-fpm* /usr/local/bin/php* 2>/dev/null  
  
  
# 发行版包  
  
dpkg -l | grep -E 'php[0-9.]+-(fpm|cli|cgi)' || rpm -qa | grep php  
  
  
# 容器  
  
docker ps --format '{{.Image}}' | grep -i php  
  
docker images --format '{{.Repository}}:{{.Tag}}' | grep -iE 'php|wordpress|magento'  
  
产出一张表：主机、PHP 版本、SAPI、是否公网可达、是否有业务、能否在两小时内升级。排序规则是公网、有业务、拿不到补丁，三条同时命中的今天就要给出处置决策。  
  
5.2 补丁发布后 0 到 2 小时  
  
以官方 release notes 为准，核对确切的受影响区间与修复版本号，不引用传闻版本。公网层优先升级。必须重启 PHP-FPM 或 Web 服务器，OPcache 与 worker 都是进程级的，只换文件不重启等于没修。验证实际生效的版本：  
  
.  
.  
.  
bash  
  
php -v  
  
curl -s http://127.0.0.1/_t.php | grep -E 'PHP Version|Server API'  
  
5.3 补丁不可及时：缓解  
  
官方事前公告称没有 workaround。以下是基于攻击面的缓解，属于降低可达性，不是修复漏洞。  
  
.  
.  
.  
php.ini  
  
session.upload_progress.enabled = 0  
  
session.use_strict_mode = 1  
  
PHP-FPM 必须在 pool 配置里写：  
  
.  
.  
.  
pool.d/www.conf  
  
php_admin_flag[session.upload_progress.enabled] = off  
  
php_admin_flag[session.use_strict_mode] = on  
  
  
systemctl reload php8.3-fpm # 换成你的版本  
  
验证必须从 HTTP 侧做。  
  
.  
.  
.  
bash  
  
echo '<?php phpinfo();' > /var/www/html/_t.php  
  
curl -s http://127.0.0.1/_t.php | grep -iE 'upload_progress.enabled|use_strict_mode'  
  
rm -f /var/www/html/_t.php  
  
副作用方面，session.upload_progress 只服务 PHP 原生进度条，WordPress、Laravel、Magento、Shopware 都不用，关掉通常无感知。session.use_strict_mode 是官方推荐的整体加固项，但 Redis 或数据库自定义 handler 若未实现 strict mode 校验，这一项对它无效。不要改 session.serialize_handler，会破坏现有会话；也不要改 session.upload_progress.prefix，与本漏洞无关。  
  
访问控制层可以再兜一层，仅在确认不使用进度条时：  
  
.  
.  
.  
nginx  
  
if ($request_body ~* "PHP_SESSION_UPLOAD_PROGRESS") { return 403; }  
  
只能当辅助手段。  
  
5.4 补丁后 2 到 24 小时：按行为狩猎  
  
diff 出来之前写不出精确特征，按行为找：  
  
异常子进程  
   
　php-fpm、apache、php-cgi 派生 sh、bash、curl、wget、nc、python  
  
命令行特征  
   
　/dev/tcp/、mkfifo、base64 -d、certutil  
  
新增文件  
   
　Web 根目录新增单文件 PHP，文件名随机、内容混淆  
  
异常外联  
   
　请求突增后 Web 主机立即向陌生地址发起出站连接  
  
.  
.  
.  
bash  
  
ps -eo ppid,pid,user,cmd --sort=ppid | grep -A2 -E 'php-fpm|apache2|httpd'  
  
find /var/www /usr/share/nginx /home/*/public_html -name '*.php' -mtime -1 -ls 2>/dev/null  
  
确认命中则隔离主机、保留 Web 与 PHP 日志、排查 webshell 与新增计划任务。  
  
5.5 长期：把运行时纳入资产管理  
  
这次暴露的是资产管理问题。多数团队能列出用了哪些框架，列不出跑了哪些 PHP 运行时。把 PHP 版本纳入 CMDB 与 SBOM，与主机、镜像、厂商产品绑定。给 EOL 运行时设定强制退出时间。建立起安全版本发布后数小时内完成公网层升级的演练机制。  
  
  
  
END  
  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
  
