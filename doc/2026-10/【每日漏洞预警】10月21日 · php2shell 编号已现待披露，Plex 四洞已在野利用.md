#  【每日漏洞预警】10月21日 · php2shell 编号已现待披露，Plex 四洞已在野利用  
原创 chicken
                    chicken  爱坤sec   2026-10-21 02:06  
  
每日漏洞预警  
  
10 月 21 日  
  
9 个高危漏洞 · 4 个未授权可打 · 滚动 24 小时  
  
9  
  
高危漏洞  
  
4  
  
未授权可打  
  
24h  
  
统计窗口  
  
今日概览  
  
过去 24 小时（10-20 08:27 → 10-21 08:27）新增 CVE 多达 1931 个  
、其中 ≥9.0 的 128 个  
。今天有两条必须分开看：① php2shell  
——PoC 仓库和测绘 dork 已经出现，但 CVE 详情还没发布，能否复现、影响哪些版本都暂无公开信息  
，而它的测绘面覆盖了几乎所有主流 PHP 应用，所以放在第一位盯；② Plex Media Server 四个洞被 CISA 列入 KEV  
——分数只有 4.3~7.2，但已经在野利用  
，且四洞可串联。另外 Oracle 一次性出了 7 个 10.0  
。  
  
   
  
01  
CVE-2026-109618 / CVE-2026-109619（编号已占位，详情未公布）  
  
php2shell · PHP 生态（不是单个插件，是底座）  
  
PHP → Shell 利用链（技术细节未公开，PoC 与测绘仓库已先行）  
  
CVSS 未公布 · 待确认  
  
   
  
php2shell  
 是今天最需要盯、但公开信息最少的一条。这一对编号目前的状态很特殊：NVD 查不到、GitHub Advisory 查不到、MITRE 的 CVE 记录也是未发布  
，但嗅探痕迹已经出来了——几个刚建的仓库同时在动：php2shell-poc  
、php2shell-check  
，以及一个专门放 Censys dork  
 的仓库。那个 dork 很有信息量，它同时匹配 MediaWiki、WordPress、phpBB、Magento、Laravel、Drupal、Symfony、cPanel WHM  
——这些产品的共同点只有一个：它们都跑在 PHP 上  
。换句话说，这大概率不是某个插件的洞，而是 PHP 运行时/框架层面  
的问题，一旦细节公布，受影响面是「所有 PHP 站点」，而不是「装了某个插件的人」。在细节公开前，任何声称能复现的说法都应视为未经验证；本文只记录「编号已存在、PoC 仓库已出现」这一事实。  
  
  
📦 影响版本  
  
受影响产品/版本尚未公布  
（公开 dork 覆盖 WordPress / Drupal / Laravel / Symfony / Magento / MediaWiki / phpBB / cPanel WHM）  
  
✅ 修复版本  
  
官方修复版本未公布；先把 PHP 升级到当前最新稳定版，并盯住两个 CVE 的正式公告  
  
  
🎯 利用状态：  
 ⚠️ 编号未发布 + 已有 PoC/测绘仓库，技术细节尚未公开  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：499,359 个（fofa 全网测绘，2026-10-21）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
02  
CVE-2026-96652 / CVE-2026-96654 / CVE-2026-96655 / CVE-2026-96656  
  
Plex Media Server（NAS / 家庭影音服务器，公网常见）  
  
SSRF + 编码 URI 处理缺陷 + 任意文件写（四洞可串联）· 🔥 已列入 CISA KEV  
  
CVSS 7.2 · HIGH  
  
   
  
这一组四个洞分数都不高（4.3 ~ 7.2）  
，但 10-20 被 CISA 一次性全部列入 KEV  
，说明已经在野利用——今天最该处理的就是它。CVE-2026-96652（4.3）  
：只要带任意 X-Plex-Token  
，就能在 protocol  
 参数里塞完整 URL，让 Plex 服务端向攻击者指定地址发 POST（SSRF）；CVE-2026-96655（4.3）  
：已认证用户可通过 /video/:/transcode/universal  
 的 path 参数请求任意内网/外网地址（另一个 SSRF）；CVE-2026-96654（6.5）  
：对网页里编码后的 URI scheme 处理不当，可调用其它插件的函数并自带参数；CVE-2026-96656（7.2）  
：文件名/路径可被外部控制，管理员可写入任意文件，且在加载时执行。四条按官方描述可以互相当跳板  
：SSRF 打内网 → 插件函数调用 → 落文件执行，Plex 常年跑在 NAS 上、以 root/高权限运行、还挂着一堆媒体库与云盘凭据，被打穿后损失远大于分数。  
  
  
📦 影响版本  
  
Plex Media Server（版本以 plex.tv/security 公告为准，四条均涉及）  
  
✅ 修复版本  
  
升级到 Plex 官方安全公告中的最新版本  
  
  
🎯 利用状态：  
 🔥 已列入 CISA KEV（2026-10-20 加入），已在野利用  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：7,045 个（fofa 全网测绘，2026-10-21）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
03  
CVE-2026-101809 / CVE-2026-87337 / CVE-2026-91216 / CVE-2026-91221 / CVE-2026-91231 / CVE-2026-91299 / CVE-2026-101446  
  
Oracle 2026 年 10 月 CPU（EBS / Access Manager / SOA Suite / JD Edwards）  
  
未授权 HTTP / T3 / IIOP → 直接接管，范围溢出到其它产品  
  
CVSS 10.0 · CRITICAL  
  
   
  
这次 Oracle 关键补丁更新一口气出了多个 10.0  
，共同点非常危险：无需认证  
、网络可达即可打、而且 scope change  
（影响溢出组件本身，波及同域其它产品）。具体包括：CVE-2026-101809（10.0）  
 E-Business Suite Workflow Notification Mailer  
（12.2.3 ~ 12.2.15）；CVE-2026-87337（10.0）Access Manager  
 认证引擎（12.2.1.4.0 / 14.1.2.1.0），可未授权增删改身份数据；CVE-2026-91216 / 91221（10.0）  
 SOA Suite Fabric Layer  
（走 T3、IIOP  
，12.2.1.4.0 / 14.1.2.0.0）；CVE-2026-91231（10.0）  
 SOA Suite Adapters  
（走 HTTP）；CVE-2026-91299（10.0）  
 JD Edwards Orchestrator（9.2.0.0 ~ 9.2.26.5）。Oracle 的 T3/IIOP 通道历来是反序列化与绕过的主战场，这类 10.0 基本等于「前端一暴露就有机会」。  
  
  
📦 影响版本  
  
EBS 12.2.3~12.2.15、Access Manager 12.2.1.4.0/14.1.2.1.0、SOA Suite 12.2.1.4.0/14.1.2.0.0、JDE 9.2.0.0~9.2.26.5  
  
✅ 修复版本  
  
按 Oracle CPU Oct 2026（cpuoct2026）打对应补丁  
  
  
🎯 利用状态：  
 ⚠️ 厂商公告与评分已公开，暂无公开 PoC  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
04  
CVE-2026-81212  
  
IBM DataStage on Cloud Pak for Data 5.4.0.0  
  
未授权写入共享 Redis（跨租户 key 污染）  
  
CVSS 10.0 · CRITICAL  
  
   
  
未授权用户可以对共享 Redis 里任意租户的 key 执行任意写  
。这一下直接打穿了平台用来隔离租户凭据的加密边界  
：攻击者可以往别的租户的缓存里塞自己的 flow 参数（或覆盖对方的），等对方的数据管道跑起来时，实际执行的是被污染的配置——典型后果是凭据被引到攻击者控制的端点  
，或者把数据写到攻击者指定的位置。multi-tenant 的数据集成平台出这种洞，影响面按「平台上有几个租户」来算，不是按实例算。  
  
  
📦 影响版本  
  
IBM DataStage on Cloud Pak for Data 5.4.0.0  
  
✅ 修复版本  
  
按 IBM 安全公告 node/7289603 升级  
  
  
🎯 利用状态：  
 ⚠️ 厂商公告已公开，暂无公开 PoC  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
05  
CVE-2026-113672 / CVE-2026-113708 / CVE-2026-113709  
  
Argo CD（Kubernetes GitOps 持续交付）  
  
jsonnet importer 越出仓库根目录 → 任意文件读取（SA token / 私钥 / 仓库凭据）  
  
CVSS 9.9 · CRITICAL  
  
   
  
repo-server  
 在评估 .jsonnet  
 文件时，importer 没有把 import / importstr / importbin  
 限制在仓库根目录内：绝对路径、父目录穿越（../）、仓库内的符号链接  
都能越界，读取 repo-server 进程能读到的任意文件  
——包括 /proc/self/environ  
、投射进去的 ServiceAccount token  
、挂载的私钥、以及仓库凭据。Argo CD 本身就握有集群部署权限，repo-server 拿到的 SA token 往往能直接改工作负载，所以这条的落点是「从能提 PR 的人，变成能掌控集群的人」。三条 CVE 是同一根因在不同版本分支上的编号（0.9.0 ~ 3.3.15 等）。  
  
  
📦 影响版本  
  
Argo CD 0.9.0 ~ 3.3.15、3.4.x < 3.4.10、3.5.x < 3.5.4、3.6.0-rc2 之前  
  
✅ 修复版本  
  
Argo CD ≥ 3.3.15 / 3.4.10 / 3.5.4 / 3.6.0-rc2  
  
  
🎯 利用状态：  
 ⚠️ 官方修复 commit 已公开，暂无公开 PoC  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：82,586 个（fofa 全网测绘，2026-10-21）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
06  
CVE-2026-113389 / CVE-2026-113390 / CVE-2026-113391  
  
DrayTek Vigor2960 路由器（1.5.1.6 Rev 8218）  
  
未授权命令执行（Web 管理接口多函数）  
  
CVSS 9.9 · CRITICAL  
  
   
  
同一固件版本上一次性曝出三条 9.9  
，都落在 Vigor2960 的 Web 管理接口处理函数上（其中 113391 与 upload 相关），可远程触发命令执行。Vigor 系列是中小企业与分支机构的常见出口设备，一旦被打，攻击者拿到的是整条出口链路  
——可以做流量劫持、内网横向、把设备当跳板。这类设备的固件更新率极低，实际暴露面通常比 FOFA 显示的还大。  
  
  
📦 影响版本  
  
DrayTek Vigor2960 1.5.1.6 Revision 8218  
  
✅ 修复版本  
  
升级 DrayTek 官方修复固件，并关闭 WAN 侧管理  
  
  
🎯 利用状态：  
 ⚠️ VulDB 已公开评分与影响版本，暂无公开 PoC  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：44 个（fofa 全网测绘，2026-10-21）  
  
   
  
   
  
07  
CVE-2026-112371 / CVE-2026-112370  
  
H3C Magic NX15 无线路由器（★仅 GHSA 收录）  
  
未授权代码注入 / OS 命令注入（利用已公开）  
  
CVSS 9.9 · CRITICAL  
  
   
  
CVE-2026-112371（9.9）  
：/api/esps  
 的 write  
 处理函数对 path  
 参数不做校验，可造成代码注入  
；CVE-2026-112370（9.9）  
：/usr/bin/usrlist  
 的 sub_416874  
 存在 OS 命令注入  
。两条都能远程触发  
，而且披露方明确写了利用代码已公开、可被直接使用  
。这两条目前只有 GitHub Advisory 收录、NVD 尚未入库  
——只看 NVD 会整组漏掉。H3C Magic 系列在家用/小微场景装机量大，管理界面被映射到公网的不少。  
  
  
📦 影响版本  
  
H3C Magic NX15（NX15V100R017 / 100R017）  
  
✅ 修复版本  
  
升级 H3C 官方修复固件；无补丁则隔离到访客网络  
  
  
🎯 利用状态：  
 ✅ 利用代码已公开  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：21 个（fofa 全网测绘，2026-10-21）  
  
参考 / PoC  
```
```  
  
  
   
  
   
  
08  
CVE-2026-113678  
  
Froxlor 主机管理面板  
  
符号链接跟随 → 租户提权到 root  
  
CVSS 9.9 · CRITICAL  
  
   
  
Statistics::makeChownWithNewStats  
 存在符号链接跟随缺陷：已认证的租户可以把自己文档根目录下的 goaccess 统计目录替换成符号链接  
，随后 以 root 身份运行的流量统计 cron  
 会执行 chown -R  
，把符号链接指向的、原本属于 root 的目录树整体交到攻击者手里  
——接着往里面写东西就能提权到 root。这类「面板 + root cron + 符号链接」的组合在虚拟主机面板里反复出现，凡是给客户开 shell/文件管理权限的面板都要特别小心。  
  
  
📦 影响版本  
  
Froxlor < 2.3.15  
  
✅ 修复版本  
  
Froxlor ≥ 2.3.15  
  
  
🎯 利用状态：  
 ⚠️ 修复信息已公开，暂无公开 PoC  
  
   
  
   
  
09  
CVE-2026-113506  
  
aaPanel 宝塔国际版面板  
  
ExecShell 处理函数缺陷（9.9 级，细节未完全公开）  
  
CVSS 9.9 · CRITICAL  
  
   
  
aaPanel ≤ 8.25.0  
 的 ExecShell  
 处理函数存在缺陷，评级直接给到 9.9  
。面板类产品里叫 ExecShell 的函数通常就是「把拼接好的字符串交给 shell 执行」，一旦参数可控即是命令执行——但截至目前公开信息只有函数名与评分  
，没有更细的触发路径与 PoC，所以这里只按厂商描述与评分记录，不臆测利用细节。aaPanel 面板被大量用于中小企业与个人服务器，且不少人图省事把面板端口直接开在公网。  
  
  
📦 影响版本  
  
aaPanel ≤ 8.25.0  
  
✅ 修复版本  
  
升级 aaPanel 最新版，面板端口不要暴露公网  
  
  
🎯 利用状态：  
 ⚠️ 仅厂商描述与评分公开，暂无 PoC  
  
搜索语法 / FOFA  
```
```  
  
🔍 影响资产数量：1,548 个（fofa 全网测绘，2026-10-21）  
  
   
  
处置建议  
  
1  
Plex Media Server 立即升级到官方最新版（plex.tv/security 公告），并把 32400 从公网摘掉或加反代鉴权  
  
2  
Oracle 用户按 cpuoct2026 打补丁：EBS Workflow、Access Manager、SOA Suite、JD Edwards Orchestrator 全在列  
  
3  
IBM Cloud Pak for Data 5.4.0.0 的 DataStage 组件按 IBM 公告 node/7289603 升级，并检查共享 Redis 是否被写入陌生 key  
  
4  
Argo CD 升级 3.3.15 / 3.4.10 / 3.5.4（或 3.6.0-rc2 之后），并限制 repo-server 对宿主路径与 SA token 的读取  
  
5  
DrayTek Vigor2960 升级固件，关闭 WAN 侧管理界面；H3C Magic NX15 建议直接换机或隔离到访客 VLAN  
  
6  
Froxlor 升级 2.3.15；aaPanel 升级到最新版并核查面板是否暴露公网  
  
⚠️ 安全提示  
  
本文内容仅用于安全研究与防御加固学习，所有信息来自厂商公告与公开漏洞库。请勿对未授权目标实施攻击，否则后果自负。  
  
— EOF —  
  
