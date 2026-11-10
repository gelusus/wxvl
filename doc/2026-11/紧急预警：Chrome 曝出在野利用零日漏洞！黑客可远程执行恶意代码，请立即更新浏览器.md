#  紧急预警：Chrome 曝出在野利用零日漏洞！黑客可远程执行恶意代码，请立即更新浏览器  
原创 维度攻防
                        维度攻防  维度攻防   2026-11-10 07:25  
  
各位小伙伴注意！Google 紧急发布安全更新，修复了一枚  
已经被黑客在真实网络攻击中利用的 Chrome 零日漏洞，攻击者只要诱导用户访问恶意网页，就可以远程在设备上执行恶意代码，窃取数据、植入木马风险极高。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/56vojF9ZLWGTxia1ib1Xrb1zfuSppYHq9TPHdp3rRKsYtfsw9fF2awqyUXIEg6sbj29iaOaUNWesGIP3y0sQNbYiaYvmEXzJuexiciaq49LZj4bI4/640?wx_fmt=png&from=appmsg "")  
## 🚨漏洞核心情况：CVE‑2026‑125350  
  
本次高危零日漏洞编号为  
CVE‑2026‑125350，属于  
高危释放后重用（Use‑after‑free）内存缺陷，存在于 Chrome 的 XML 解析组件中。  
> 💡通俗解释什么是释放后重用漏洞：  
  
程序把一块内存释放之后，依旧继续读写这块内存，黑客利用内存错乱，破坏浏览器运行逻辑，实现执行任意恶意代码。  
  
  
攻击方式非常隐蔽：受害者只需要点开钓鱼邮件、恶意广告、被入侵网站中的恶意链接，不需要额外点击、下载文件，访问网页的瞬间就可能遭受攻击。  
  
该漏洞由匿名研究者在 2026‑11‑06 上报，仅仅 3 天谷歌就推送补丁。目前谷歌确认  
野外已经存在可用攻击 EXP，但尚未披露攻击者身份与受影响用户规模。为防止黑客利用未修复设备，漏洞完整技术细节将推迟公开，待绝大多数用户完成更新之后才会解锁详情。  
## 📦本次更新一共修复 9 处安全缺陷  
  
这次 Chrome 安全补丁合计修复 9 个漏洞，其中 7 个评定为  
高危等级，覆盖 XML、V8 引擎、WebGL、GPU、文件系统、合成渲染多个模块。  
  
表格  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">CVE 编号</span></section></th><th style="text-align: left;"><section><span leaf="">组件</span></section></th><th style="text-align: left;"><section><span leaf="">漏洞类型</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125350</span></section></td><td style="text-align: left;"><section><span leaf="">XML</span></section></td><td style="text-align: left;"><section><span leaf="">释放后重用（零日，野外已利用）</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125346</span></section></td><td style="text-align: left;"><section><span leaf="">XML</span></section></td><td style="text-align: left;"><section><span leaf="">释放后重用</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125344</span></section></td><td style="text-align: left;"><section><span leaf="">V8</span></section></td><td style="text-align: left;"><section><span leaf="">类型混淆</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125348</span></section></td><td style="text-align: left;"><section><span leaf="">GPU</span></section></td><td style="text-align: left;"><section><span leaf="">越界写入</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125347</span></section></td><td style="text-align: left;"><section><span leaf="">FileSystem</span></section></td><td style="text-align: left;"><section><span leaf="">授权错误</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125343</span></section></td><td style="text-align: left;"><section><span leaf="">Compositing</span></section></td><td style="text-align: left;"><section><span leaf="">缓冲区溢出</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">CVE‑2026‑125345</span></section></td><td style="text-align: left;"><section><span leaf="">WebGL</span></section></td><td style="text-align: left;"><section><span leaf="">未初始化资源</span></section></td></tr></tbody></table>  
另外还有 2 个中危 GPU 相关漏洞同步完成修复。大部分漏洞来自谷歌内部安全团队挖掘，也有外部安全研究员提交报告贡献漏洞发现工作。  
## ✅你的浏览器应该升级到哪个版本？  
- Windows / Mac：  
157.0.8103.50 / .51  
  
- Linux：  
157.0.8103.50  
  
- 企业延长稳定版：  
156.0.8078.96  
  
> ⚠️划重点：Chrome 后台会自动下载更新，  
但是补丁不会生效，必须手动重启浏览器！  
  
### 🛠️快速更新操作步骤  
1. 点击 Chrome 浏览器右上角「三个点」菜单  
  
1. 选择「帮助」→「关于 Google Chrome」  
  
1. 浏览器自动检测下载新版本，下载完成点击「重新启动 (Relaunch)」  
  
1. Linux 用户请通过系统软件包管理器完成升级  
  
> 重要提醒：Edge、Opera 等所有基于 Chromium 内核的浏览器，共用同一套引擎代码，也存在同源风险，请大家留意对应浏览器厂商推送的安全补丁，及时更新。  
  
## 💻安全建议，个人 & 企业都要看  
  
🔹  
普通用户  
1. 立刻重启 Chrome 完成升级，不要忽略浏览器右上角的重启提示  
  
1. 来历不明邮件、社交软件发来的陌生链接不要随意点开  
  
1. 尽量不访问小众、不知名的网站，降低接触恶意网页的概率  
  
🔹  
企业管理员  
1. 优先推进终端 Chrome 版本升级，关注延长稳定渠道版本更新状态  
  
1. 加强钓鱼邮件告警，警惕针对该零日漏洞的定向钓鱼攻击  
  
1. 持续关注漏洞公开细节，做好后续威胁研判  
  
> 虽然谷歌有 AddressSanitizer、libFuzzer 等大量自动化漏洞挖掘工具，但是依旧会有零日漏洞流入野外攻击。浏览器作为上网入口，及时更新是抵御网页攻击最有效的手段。  
  
  
  
