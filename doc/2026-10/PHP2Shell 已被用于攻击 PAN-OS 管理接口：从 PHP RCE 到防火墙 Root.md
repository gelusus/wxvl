#  PHP2Shell 已被用于攻击 PAN-OS 管理接口：从 PHP RCE 到防火墙 Root  
原创 小智
                    小智  智榜样网络安全学习中心   2026-10-31 14:26  
  
防火墙通常被放在网络边界，负责过滤流量、控制访问。但如果防火墙自己的**管理界面暴露在公网**  
，而且管理服务里还存在一个**未认证远程代码执行漏洞**  
，情况就完全不一样了。  
  
10 月 29 日，Palo Alto Networks Unit 42 更新了 PHP2Shell 威胁简报，确认攻击者正在利用 **CVE-2026-109618 和 CVE-2026-109619**  
 攻击暴露在互联网的 PAN-OS 管理接口。Dream Research 此前披露，一组攻击活动把 PHP2Shell 与 Linux 内核提权漏洞 Fragnesia（CVE-2026-46300）串联，**最终取得设备 root 权限**  
。Unit 42 表示已独立验证相关攻击工具和研究结果。  
  
这次事件比较值得注意的地方，是**攻击者并没有停留在防火墙本身**  
。拿到边界设备之后，他们继续建立通往内网的通道，访问内部防火墙管理服务器，**并开始枚举 Active Directory**  
。  
  
![PHP2Shell 攻击活动时间线](https://mmbiz.qpic.cn/mmbiz_jpg/ibzm8nWOdauOGMMl6LMLtW3EstK3l7Jg1OjBx3w9CeaavF4HwicaHhXboy1HXWgsAlNec7Pa4h3VfWJfpL9yIp4rq9wyubVofrpK328ViadnR8/640?wx_fmt=jpeg&from=appmsg "")  
## 先说清楚：这次受影响的是管理界面，不是所有 VPN 入口  
  
PHP2Shell 指的是 PHP 中**两个内存安全问题**  
：CVE-2026-109618 和 CVE-2026-109619。两项漏洞的 **CVSS 4.0 评分均为 9.2**  
。  
  
简单理解，问题出在 PHP 处理内存耗尽等异常时，**错误处理流程可能让内部状态没有被正确恢复**  
。longjmp()  
 跳转之后，请求数据或全局状态可能处于未初始化、已经释放却仍被引用的状态。后续处理再使用这些状态时，就可能造成**内存破坏，最终执行攻击者控制的代码**  
。  
  
这不是某个 WordPress 插件或 PHP 网站框架独有的问题，而是 **PHP 运行时层面的漏洞**  
。PHP 项目在 10 月 21 日发布了安全版本，修复版本包括 **PHP 8.5.12、8.4.27、8.3.36 和 8.2.35**  
；更早的 PHP 版本中也有受影响版本，但已经停止维护的分支不会再获得常规安全修复。  
  
不过，本文重点是 PAN-OS。Dream Research 指出，**相关 PAN-OS 版本的管理 Web 服务使用 PHP 8.0.30**  
。设备用户不能把它当作一台普通 Linux 服务器，直接单独替换系统里的 PHP；**应当按照 Palo Alto Networks 发布的 PAN-OS 热修复版本升级**  
。  
  
这里还有一个容易被误解的地方：**受影响的是 PAN-OS 管理 Web 界面，包括 HTTPS Web UI 和 XML API。**  
 Palo Alto Networks 明确说明，GlobalProtect Portal、Gateway 和 Clientless VPN 本身不受这两个 PHP 漏洞影响。若数据平面接口绑定了管理配置文件，则需要根据厂商公告进一步确认暴露情况。  
  
所以，不能看到设备开放了 GlobalProtect 就直接认定存在 PHP2Shell 风险。**真正需要优先排查的是管理服务是否被放到了互联网侧**  
。  
## 从 PHP 代码执行到 root，中间还差一次提权  
  
**这次攻击不是拿到 PHP 代码执行就结束了。**  
  
根据 Dream Research 的分析，攻击者使用的是**针对 PAN-OS 定制的 PHP2Shell 利用实现，并非简单复制公开 PoC**  
。漏洞触发后，最初获得的是 Web 服务运行上下文中的**低权限命令执行**  
。对于边界防火墙来说，这已经非常危险，但还没有达到 root。  
  
攻击者随后把 PHP2Shell 与 **Fragnesia（CVE-2026-46300）这个 Linux 内核本地提权漏洞串联**  
。前者提供初始代码执行，后者把权限从 Web 服务用户提升到 root。两个漏洞配合后，攻击者可以在受影响设备上获得更高权限，并**留下可从 Web 访问的 PHP WebShell**  
。  
  
这也是这起事件真正麻烦的地方：如果只看 PHP 漏洞，可能会以为攻击者最多拿到一个低权限 Web 进程；如果只看内核提权，又可能忽略攻击者是怎样从互联网进入设备的。**两段漏洞链组合起来，才形成了从未认证 HTTPS 请求到设备 root 权限的完整路径**  
。  
  
![从公网入口到内网管理服务器及域控制器的攻击路径](https://mmbiz.qpic.cn/mmbiz_png/ibzm8nWOdauNicFA8libCbGkcQNwT6nx5RfhYymicZ7CIBLS3KLjU27PIegzMDeJS7VK0zibWlcvjVoCwiaicicoNyy8N7xkt0Fuicuoiaia9ar5aQzCzc/640?wx_fmt=png&from=appmsg "")  
## 19,000 个目标，最后有多少设备被攻陷？  
  
Dream Research 在调查中梳理了攻击者的资产筛选和利用过程。报告称，攻击者先通过 FOFA 找到**约 19,000 台**  
管理界面暴露在公网的 PAN-OS 设备；之后进行存活和版本识别，**约 5,000 台**  
返回了可识别的信息。再按照地理位置和可利用版本筛选，目标范围缩小到**约 240 台**  
。  
  
后续攻击者对这些目标进行自动化尝试。Dream 报告称，**第一轮有 24 台防火墙被成功取得 root 权限**  
。其中特别值得注意的是，**22 台设备在植入 WebShell 后暂时没有进一步活动**  
；另外两台与同一个美国政府关联环境有关，攻击者从中挑选目标继续向内网推进。  
  
其中一台设备提取出的配置文件包含内部网络拓扑、域控制器、中央防火墙管理服务器、Keycloak SSO、GitLab 和 VPN 基础设施等信息。攻击者随后通过**反向 SSH 隧道**  
连接内网，并进一步攻陷一台**只能从内网访问的中央防火墙管理服务器**  
。Dream 将这台服务器计入后，报告中的被攻陷服务器总数达到 **25 台**  
。  
  
**这里要区分“被攻陷”和“后续攻击成功”。研究人员确认攻击者枚举了 AD 用户和组、解密了 LDAP 服务账号凭据，并尝试进行密码喷洒、创建后门账号和提取 SAML 签名密钥；但这些后续动作是否成功，Dream 表示从恢复到的证据中无法确定**  
。  
## 真正危险的是边界设备变成了内网跳板  
  
防火墙的配置自身就包含着很多敏感的信息。当设备的root权限被获取到的时候，攻击者或许会去接触网络拓扑，接口地址，VPN配置，证书私钥，设备认证的密钥还有服务账号的相关内容。  
  
Dream报告所阐述的后续操作包含：在设备之上构建**能够重启之后自动复原的反向的SSH隧道**  
通过隧道去访问域的控制器，内部的防火墙管控服务器还有VPN的基础设施，之后**借助中央管理器已经具备的LDAP连接能力去枚举域的用户以及组**  
。  
  
蓝队需要特别留意这条路径。许多企业会将防火墙的管理面与普通的业务网分隔开来，但是**若边界防护墙自身被掌控**  
，攻击者极有可能从原先被信任网络节点朝内侧转移。**此刻仅仅去查看Web服务器以及业务主机的日志，是不够的。**  
## 蓝队应该从哪里开始排查？  
  
第一步就是确定管理面是否暴露。查看设备管理的接口公网能够达到，管理配置的文件，上游ACL还有可以访问的源的地址。厂商提议将管理界面公网访问予以移除，**仅允许可信的内部地址以及专用的跳板机进行访问**  
。因运维方便的管理面，不应直接被开放给全部互联网使用。  
  
第二步就是去核对PAN和OS的版本情况。PaloaltoNetworks已经在10月27号到29号这个时间段里陆续发布了好几个分支修复的补丁。不同minor分支相应修复版本存在差异，**不能仅依据“大版本数量”来判定是否具备安全性**  
。下面列举厂商公告当中的修复难度，实际进行升级之前还得按照设备当前的分支去核对公告情况：  
<table><thead><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><th style="color:rgb(26, 26, 26);background-color:rgb(247, 248, 250);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:700;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(26, 26, 26);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">PAN-OS 分支</span></section></th><th style="color:rgb(26, 26, 26);background-color:rgb(247, 248, 250);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:700;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(26, 26, 26);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">公告列出的修复版本示例</span></section></th></tr></thead><tbody><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">12.2</span></section></td><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">12.2.4-h1</span></section></td></tr><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><td style="color:rgb(63, 63, 63);background-color:rgb(251, 252, 253);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">12.1</span></section></td><td style="color:rgb(63, 63, 63);background-color:rgb(251, 252, 253);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">12.1.4-h13、12.1.7-h11、12.1.10-h3</span></section></td></tr><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">11.2</span></section></td><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">11.2.4-h23、11.2.7-h23、11.2.10-h16、11.2.13-h4、11.2.16-h1</span></section></td></tr><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><td style="color:rgb(63, 63, 63);background-color:rgb(251, 252, 253);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">11.1</span></section></td><td style="color:rgb(63, 63, 63);background-color:rgb(251, 252, 253);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">11.1.4-h38、11.1.6-h40、11.1.7-h12、11.1.10-h36、11.1.13-h16、11.1.16-h4</span></section></td></tr><tr style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:26.1px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:0px;padding-right:0px;padding-bottom:0px;padding-left:0px;display:table-row;border-collapse:collapse;"><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">10.2</span></section></td><td style="color:rgb(63, 63, 63);font-family:-apple-system, BlinkMacSystemFont, &#34;PingFang SC&#34;, &#34;Helvetica Neue&#34;, &#34;Microsoft YaHei&#34;, sans-serif;font-size:14.5px;font-weight:400;line-height:24.65px;letter-spacing:0.6px;text-align:left;text-decoration-color:rgb(63, 63, 63);text-decoration-style:solid;vertical-align:middle;word-break:break-word;overflow-wrap:break-word;margin-top:0px;margin-right:0px;margin-bottom:0px;margin-left:0px;padding-top:9px;padding-right:12px;padding-bottom:9px;padding-left:12px;border-top-width:0.571429px;border-right-width:0.571429px;border-bottom-width:0.571429px;border-left-width:0.571429px;border-top-style:solid;border-right-style:solid;border-bottom-style:solid;border-left-style:solid;border-top-color:rgb(235, 237, 240);border-right-color:rgb(235, 237, 240);border-bottom-color:rgb(235, 237, 240);border-left-color:rgb(235, 237, 240);display:table-cell;border-collapse:collapse;"><section><span leaf="">10.2.7-h39、10.2.10-h42、10.2.13-h27、10.2.16-h12、10.2.18-h13</span></section></td></tr></tbody></table>  
第三步是检查设备是否已经被利用。Dream 披露的一个重要文件线索是：  
```
/unauth/php/change_password.php
```  
  
报告显示，攻击者于已被攻陷的设备之上放置一个体积较小的PHPWebShell。**若此路径出现于设备文件体系中，需当作具有高优先级的事件进行调查**  
，**不可简单删除之后就结束进行处置**  
。  
  
另外还需要核对：  
- 是否存在不认识的 systemd 服务或 rc.local  
 启动项；  
  
- /usr/local/lib/.panops/  
 下是否出现异常脚本或文件；  
  
- 管理面是否出现来源异常的 HTTPS/XML API 请求、异常的大体积 multipart 请求或请求后服务进程异常；  
  
- 是否有防火墙主动向陌生地址建立 SSH 或其他长期出站连接；  
  
- 配置、证书、HA 复制密钥和设备认证密钥是否被读取或需要轮换；  
  
- 是否出现来自防火墙或管理服务器的异常 LDAP 查询、账号枚举、密码喷洒和新账号创建记录。  
  
此类线索需结合设备的日志，配置的变更记录，网络的流量以及厂商的支持包进行交叉的核对。**并非所有异常的请求都表明PHP2Shell已经成功运用，单个的指标也不可以单独作为失陷的结论。**  
  
要是已经察觉到WebShell，没有启动项或者反向隧道的话，**建议按照设备失陷来进行处理**  
：先把管理面隔开并且保存日志以及配置的证据，之后联系厂商来开展取证工作；随后按照可信镜像进行重建或者按照厂商的指导来让设备恢复，换上设备证书，服务账号还有相关的密钥，并且检查被他人信任或者管理的别的系统。**仅仅升级补丁不能够自动把已经留存下来的持久化清除掉。**  
## 这次事件还有一个 AI 相关细节  
  
Dream在对基础设施进行攻击的时候发现**ClaudeCode**  
，并且配置DeepSeek，Kimi模型的人工智能安全工具aiscan，同时发现ofA的APIKey以及**大约40个后进行渗透的脚本**  
。  
  
但是研究报告同样明确表明：**当下没有直接的证据能够证实AI模型所编写的PHP2Shell是运用代码或者是后渗透的脚本**  
。科研人员依据代码的结构，注释的风格还有脚本的一致性，觉得存在AI帮助开发的可能性；**这并不等同于一次全然自主的人工智能攻击**  
。  
  
更为精准的认知是，人工智能工具或许参与攻击行为的辅助性操作，但是漏洞的探究，版本的适配以及利用链的校准仍然显示出许多人工技术的投入。**简单地将这起事件书写成AI自动攻克防火墙，反而会遮蔽真正值得解析的技术方面的问题。**  
## 最后  
  
这一回PHP2Shell进行攻击所得到的教训不是很复杂的：**管理的界面不能够出现在公网当中**  
，边界的设备也不可以因为处在网络的入口而被默认当作是安全的。攻击者先从一个没有被认证的管控请求起始，先借助PHP内存方面的问题来执行代码，之后通过内核的漏洞提升成为root，**接着把防火墙转变成为进入内网跳板**  
。  
  
对于蓝队而言，**对CVE进行修补仅仅是首要的步骤**  
。需确认管理方面以往有无暴露，有无被利用，配置以及凭据有无泄露，还要确认**攻击者能否通过中央管理的平台持续进入AD以及其他网络的区域**  
。  
  
**如果设备曾经暴露在公网且运行受影响版本，应该按“可能已经被访问”来做排查，而不是只确认今天的版本号是否升级。**  
  
  
