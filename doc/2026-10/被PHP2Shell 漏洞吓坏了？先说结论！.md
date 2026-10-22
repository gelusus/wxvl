#  被PHP2Shell 漏洞吓坏了？先说结论！  
安全团队
                    安全团队  墨雪飘影   2026-10-22 03:12  
  
如果你的时间线这两天被「**PHP 史诗级漏洞**  
」「**一个请求PHP接管服务器**  
」刷屏了，先深呼吸。  
  
甚至有公众号拿出的利用条件是「攻击面为可接收 HTTP 请求的 PHP 端点」——按这个标准，全互联网七成的网站都该当场去世，属实难评。  
  
先说结论：**漏洞是真的，严重是真的，两个 CVE 也真的修了。**  
但营销号恐慌话术里掺的水，也是真的。  
  
01  
先看官方怎么说的  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/o5nTnHEDKUw7icRibKB8MRHeKibiaqNjQywyDRcJJxZWMO9Gz4ibUG27AHm4EkW9LLVOjibel56p8LRtXPCb9YGYibic340L5ibzbDuMUdXKAIl4kYyg/640?wx_fmt=other&from=appmsg "")  
  
PHP 官方安全公告（10 月 21 日）原文，两个关键段落：  
  
PHP 8.5.12、8.4.27、8.3.36 和 8.2.35 修复了两个可导致**未认证远程代码执行**  
的关键漏洞。所有 PHP 用户被强烈建议立即更新。  
  
谁处于风险中：**所有**  
对外提供 Web 请求的 PHP 安装，包括 PHP-FPM、Apache mod_php、php-cgi、FrankenPHP 和 LiteSpeed。我们认为 **mod_php 风险最高**  
——已有默认配置下验证的利用，未认证请求可以在任何应用代码运行之前触发攻击。  
  
划重点 ①　"所有 PHP 安装" ——不是只有某个特定 SAPI 才中招的豁免；  
  
划重点 ②　"mod_php 风险最高、已验证" ——这是生产环境最常见的部署方式，不是实验室玩具。  
  
注意，官方说的是风险与已验证的利用，不是一键接管任何PHP服务器。  
  
02  
30 秒说清楚这洞是啥  
  
  
PHP 处理一个请求时，multipart 表单（带文件上传的表单）的解析发生在**所有扩展初始化之前**  
；如果解析中途内存耗尽，请求会提前崩溃——此刻清理工作被跳过，销毁工作却照常执行。  
  
两个 CVE 就藏在这条缝的两端：  
  
**CVE-2026-109618**  
：崩溃时一张哈希表的计数器虚增，一个从没被写入的「幽灵槽位」在请求收尾时被当成正常数据析构；  
  
**CVE-2026-109619**  
：上一个请求留下的会话对象没被清掉，下一个请求在自己的初始化之前就去碰它。  
  
手法很巧妙，机制分析值得细读。但机制成立和打得到你之间，隔着几座山。  
  
03  
从机制成立到打得到你，隔着几座山  
  
  
⛰️ 一：攻击者得先有两张「藏宝图」  
  
整个攻击的核心，是往内存里塞一段**伪造的数据结构**  
：伪造的哈希表、伪造的执行栈、伪造的函数指针。其中好几个字段必须填写**绝对内存地址**  
——你的进程的堆在哪、PHP 二进制加载在哪。这些地址由 ASLR 随机分配，每次重启都不一样。  
  
我们团队的实验，是以本机特权读 /proc/<pid>/maps  
 拿到这两个值的——翻译一下，攻击者坐在机器旁边，翻开进程地图把地址抄下来填了进去。我们在复现中没有实现远程泄露。（技术详情请点击"阅读原文"  
）  
  
但要说明：独立复现方（Datadog Security Labs）已经指出，内存泄露路径是现实存在的，包括 PHPSESSID 相关技术。所以需要泄露是真前提，但它不构成躺平的理由。  
  
⛰️ 二：精度要求苛刻到离谱  
  
我们团队实测，固定请求形状、只调燃烧字段个数，结果是这样：  
<table><tbody><tr><td style="padding: 9px 6px;background: rgb(30, 75, 143);color: rgb(255, 255, 255);font-weight: bold;text-align: center;border: 1px solid rgb(30, 75, 143);"><section><span leaf="">燃烧字段数</span></section></td><td style="padding: 9px 6px;background: rgb(30, 75, 143);color: rgb(255, 255, 255);font-weight: bold;text-align: center;border: 1px solid rgb(30, 75, 143);"><section><span leaf="">崩溃点</span></section></td><td style="padding: 9px 6px;background: rgb(30, 75, 143);color: rgb(255, 255, 255);font-weight: bold;text-align: center;border: 1px solid rgb(30, 75, 143);"><section><span leaf="">打中谁</span></section></td></tr><tr><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">16</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">哈希表插入</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">一张持久化的内部表</span></section></td></tr><tr><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);background: rgb(244, 247, 251);"><section><span leaf="">17</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);background: rgb(244, 247, 251);"><section><span leaf="">同上</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);background: rgb(244, 247, 251);"><section><span leaf="">隔壁的子表</span></section></td></tr><tr><td style="padding: 9px 6px;text-align: center;border: 1.5px solid rgb(232, 89, 12);color: rgb(232, 89, 12);font-weight: bold;"><section><span leaf="">18</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1.5px solid rgb(232, 89, 12);color: rgb(232, 89, 12);font-weight: bold;"><section><span leaf="">同上</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1.5px solid rgb(232, 89, 12);color: rgb(232, 89, 12);font-weight: bold;"><section><span leaf="">目标表 ✅</span></section></td></tr><tr><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">19</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">解析器外层</span></section></td><td style="padding: 9px 6px;text-align: center;border: 1px solid rgb(219, 226, 236);"><section><span leaf="">整个攻击作废</span></section></td></tr></tbody></table>  
多一个字段、少一个字段，崩的地方就换了，攻击直接落空。这不是发个畸形请求就行的攻击，是掐着字节喂请求。  
  
另一个对照实验：把伪造结构在内存里整体**平移 8 个字节**  
，其余内容、地址、请求一字节不动——服务端只返回 200，无副作用，不崩溃。  
  
差 8 字节，一切如常。这不是 fuzz 出来的洞，是为单个实验环境**手工雕刻**  
出来的艺术品。手工雕刻意味着：**环境变一点，全部重刻。**  
  
⛰️ 三：利用代价不稳定  
  
在我们的复现环境里，命令执行后 worker 必崩——观测事实，一个请求换一条命令，代价是进程退出。  
  
但 Datadog Security Labs 独立复现后写得很明确：成功的命令执行**不一定**  
导致 worker 崩溃。  
  
04  
别被两极带跑  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/o5nTnHEDKUyezHCXVQnmFJoP2BMgJwaia625aGztSTrvqNGETicM9lo5VVSczWjjnA3mmZaxgwdKhWUbk46g2OhdY3iajBpKzTErXCb6eoLkicQ/640?wx_fmt=png&from=appmsg "")  
  
这是**真实、严重、需要立即处理**  
的漏洞；但它的武器化**不是即插即用**  
，每个目标环境都要重新标定布局。  
  
前者是你今晚升级的理由，后者是 恐慌到连夜关站 没必要的原因。  
  
05  
受影响的不只是你的网站  
  
  
![preview](https://mmbiz.qpic.cn/mmbiz_png/o5nTnHEDKUyyoBBRokxFzClhgR4O5290k9cZ5cGaibgRNWzbkISnicPSUGBDMxGxYwrNx0Mp7mNLRd3httNpbWxibVWKmkTeL1YOfE6sAzyCCU/640?wx_fmt=png&from=appmsg "")  
  
这波冲击已经穿过应用层，抵达网络设备层：  
  
**Palo Alto Networks**  
 官方 PSIRT 已发布产品影响公告（Severity: HIGH）：**PAN-OS 与 Panorama 全部版本在受影响之列**  
，官方修复仍在路上，安全社区的建议是：公网暴露的管理面、VPN 门户，能关先关。  
  
同一公司的 Cortex Xpanse 用遥测数了一遍全球公网：**超过 2,260 万个**  
可能受影响的 PHP 实例暴露在外——注意，这是 潜在暴露面 估算，不是 已被攻陷 名单。  
  
边界防御方在连夜上弹药：**Cloudflare**  
 已对全部 WAF 用户自动部署 php2shell 防护；**PANW**  
 发布 Threat Prevention 签名 543004；**Akamai**  
 上线检测规则 3000991——但注意，这些全是虚拟补丁，**挡不住直连源站的流量，更替代不了 PHP 本体升级**  
。  
  
CMS 生态也有官方口径：**WordPress 官方明确本体无需更新，但必须升级 PHP 运行环境**  
——你的站点代码再干净，底下的解释器有洞照样中招。  
  
06  
正确的应对姿势  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/o5nTnHEDKUwS11tzowymyq9yIBGkww6azibEZbLKFgK24r7XuSNWxic5iadYhicEzkEib9d83WeR19HxncWId8piaT2c7IiacXFt9OjDxOZ039EYc8/640?wx_fmt=png&from=appmsg "")  
  
恐慌不用，动作要有，按优先级排：  
  
**① 升级。**  
8.5.12 / 8.4.27 / 8.3.36 / 8.2.35，官方一次把两个方向都堵了，这是**唯一完整的修复**  
。注意 8.1 及更早版本已 EOL 无常规补丁（7.4/8.0/8.1 有社区最小源码回移），要么升大版本，要么找发行版要回移补丁；  
  
**② 暂时升不了的，先做缓解。**session.upload_progress.enabled=0  
 拆掉已确认链路之一的触发点，fatal_error_backtraces=0  
 关掉一个现实的信息泄露入口——但官方口径很明确：缓解只阻断已确认路径，**不是完整方案**  
；  
  
**③ 设备自查。**  
如果你家网络里有 Palo Alto 设备，立刻核对 PSIRT 公告，公网暴露面该收就收；  
  
**④ 重启 worker、检查边界。**  
开了 Cloudflare 等 WAF 的确认拦截规则已生效；排查「未认证大 multipart 请求、worker 异常重启  
、  
web 进程拉起可疑子进程」这条特征，Elastic 和 Datadog 都给了检测思路。  
  
  
真正危险的不是漏洞本身，  
  
是你在"末日"和"没事"之间来回横跳时，  
  
忘了做那件唯一正确的事。  
  
  
与其转发恐慌，不如花十分钟升级。🛠️  
  
* 本次漏洞复现、文章梳理均由墨雪飘影安全团队依托小何AI平台免费提供的 Deepseek-v4.1-Flash-1017 模型协同梳理完成。  
  
ABOUT US  
  
墨雪飘影漏洞收集平台  
  
🌐 v.mxpy.cn  
  
一家专注于事件型、通用型漏洞及安全情报收录的专业平台，首批通过工信部备案的网络产品安全漏洞收集平台。我们致力于建立一套主动与被动相结合、标准化与流程化的漏洞管理机制，以自动化与智能化技术提升漏洞处理的效率与准确性——从及时发现、评估、跟踪到发布通报，形成完整闭环，为提升我国整体网络安全水位尽一份力。  
  
