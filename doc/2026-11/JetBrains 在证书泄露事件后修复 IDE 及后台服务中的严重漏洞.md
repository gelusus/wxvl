#  JetBrains 在证书泄露事件后修复 IDE 及后台服务中的严重漏洞  
sec随谈
                    sec随谈  sec随谈   2026-11-02 01:08  
  
**内容摘要**  
  
JetBrains 于 2026 年 10 月 30 日发布紧急安全更新，修复了因数字证书泄露导致的严重 IDE 漏洞。这些漏洞源于受信任的中间证书颁发机构（CA），攻击者可借此对开发者发起中间人攻击。软件工程师必须立即更新自己的 IDE、JetBrains Runtime 和后台服务，以防流量被拦截。  
  
**为何值得关注**  
  
据 JetBrains 估计，其开发软件服务于全球超过 1600 万名开发者。软件工程师用这些集成开发环境（IDE）来构建、测试和部署关键的企业应用。因此，拦截开发者流量会构成严重的软件供应链威胁。  
  
2026 年 10 月，证书颁发机构 eMudhra 遭遇重大安全泄露。根据   
JetBrains 官方安全更新  
公告，"eMudhra 的 8 个中间证书的私钥被泄露"。持有这些私钥的攻击者可以为互联网上的任意域名伪造有效的 TLS 证书。不修复这些漏洞，攻击者就能对开发者的工作站发起中间人攻击。不过，JetBrains 在公告中确认："截至发布时，我们尚未发现任何针对 JetBrains 用户利用该问题的情况。"此外，研究人员也未公开针对这些问题的 PoC 利用代码。  
  
**攻击原理**  
  
这些漏洞使网络攻击者能够冒充安全端点，而不会触发操作系统的标准告警。  
  
**被泄露的中间证书与 JBR 信任库**  
  
在正常的开发流程中，IDE 会连接云端代码仓库、插件市场和授权服务器。公告警告称，"网络攻击者可以利用这些证书冒充 JetBrains 工具所连接的网站和服务"。一旦攻击者拦截到网络流量，就能出示一张由被泄露的 eMudhra 中间证书签发的伪造证书。  
  
操作系统常规的补丁管理无法在开发者工作站上解决这个问题。厂商解释说："JetBrains IDE 是根据其内置运行时 JetBrains Runtime（JBR）的信任库来校验证书的，因此更新操作系统并不能防护该问题。"为了阻断伪造连接，JetBrains 把 Oracle 针对   
CVE-2026-91403  
 和   
CVE-2026-114820  
 的修复集成进了 JBR。该更新移除了对四个根证书的信任：emSign ECC Root CA – G3、emSign Root CA – G1、emSign Root CA – G2 以及 AAA Certificate Services。  
  
**内嵌浏览器与后台守护进程漏洞**  
  
除运行时信任库外，还有两个具体漏洞削弱了桌面环境的安全性。其一是   
CVE-2026-118696  
（CVSS 评分 8.0）：基于 IntelliJ 的 IDE 内嵌的 Chromium 浏览器（JCEF）信任了已被吊销的中间证书。当开发者在 IDE 内查看文档或网页预览时，未打补丁的浏览器会一直接受这些已吊销的证书，直到 Google 的更新机制刷新吊销列表为止。  
  
其二是   
CVE-2026-118697  
，这是 jetbrainsd 后台服务中的一个严重漏洞（CVSS 评分 9.0）。该守护进程与 Toolbox App 和桌面版 IDE 一同运行，负责处理本地后台任务。由于 jetbrainsd 维护着一份独立于 JBR 的内部受信证书列表，它仍然信任那些被泄露的密钥。拦截该守护进程通信的攻击者，可以冒充后端服务或篡改软件安装过程。  
  
**受影响版本**  
  
这些漏洞影响整个桌面版 IDE 系列，包括 IntelliJ IDEA、PyCharm、WebStorm、PhpStorm、CLion、Rider、GoLand、RubyMine、DataGrip、RustRover 和 DataSpell。具体而言，2026.3、2026.2.4、2026.1.5.1 和 2025.3.6.2 之前的版本含有存在漏洞的信任库。此外，3.9.1 和 3.8.2 之前的 Toolbox App、JetBrains Gateway 以及 ReSharper 也受影响。2025.1 及更早的旧版 IDE 将不会获得补丁。  
  
**补丁与缓解措施**  
  
开发者必须立即升级到   
JetBrains 已修复问题门户  
中列出的最新修复版本。已修复的版本包括 IntelliJ IDEA 2026.2.4、Toolbox App 3.9.1 和 jetbrainsd 0.9.9728。更新 Toolbox App 会自动为所有已安装的 IDE 更新 jetbrainsd 守护进程。  
  
如果开发者无法立即升级，应在 IDE 设置中禁用 JCEF，并手动更新 jetbrainsd 守护进程。此外，各组织还应检查自定义的项目 JDK 和外部工具运行时（如 Node.js 或 Python），确保它们的证书库不信任已被吊销的 eMudhra 根证书。  
  
参考链接：  
  
https://www.jetbrains.com/privacy-security/issues-fixed/  
  
  
