#  php2shell RCE漏洞相关细节：AI助力发现十年历史漏洞，全球70%互联网基础设施受影响  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-22 00:51  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
# 引言：互联网安全历史上的转折点事件  
  
2026年10月21日，全球网络安全界见证了可能是历史上影响最为广泛的安全事件：两个**CVSS评分9.2**  
的关键PHP远程代码执行漏洞被披露和修补。被统称为"php2shell"的这两个漏洞（CVE-2026-109618和CVE-2026-109619）已经存在**超过十年**  
，影响**70%的互联网网站**  
，而它们的发现和武器化则标志着AI在网络安全领域应用的转折点。  
  
更令人担忧的是，早在官方补丁发布的数小时前，Hadrian安全团队已经报告"php2shell在野外大规模利用"，大多数攻击利用的是**PHP的会话上传进度路径**  
。他们给出了紧急建议："如果你运行互联网面向的PHP且无法立即缓解，请将这些系统下线直到补丁发布并安装完成。关闭服务几个小时的成本远低于服务器被入侵。"  
## 漏洞统计：震惊互联网的现实数据  
### Wiz研究团队的惊人发现  
  
Wiz研究团队在长达数月的研究中发现了四个可被利用的漏洞被归入两个CVE中：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">跟踪ID</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">漏洞类型</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">攻击者可实现的后果</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">我们演示的内容</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">CVE-2026-109618</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">上传进度反序列化UAF</span><span leaf=""><br/></span><span leaf="">远程进程内存泄漏</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">破坏进程内存并执行代码</span><span leaf=""><br/></span><span leaf="">未经身份验证的PHP工作进程内存泄漏</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">WordPress、Drupal和Joomla上的预认证RCE</span><span leaf=""><br/></span><span leaf="">应用程序无关的远程内存泄漏</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">CVE-2026-109619</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">致命错误回溯UAF</span><span leaf=""><br/></span><span leaf="">OPcache类型混淆</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">破坏进程内存并执行代码</span><span leaf=""><br/></span><span leaf="">破坏共享PHP状态并执行代码</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">Drupal上的预认证RCE</span><span leaf=""><br/></span><span leaf="">WordPress上的预认证RCE</span></section></td></tr></tbody></table>  
  
**重要提示**  
：虽然演示中选择了三种流行的内容管理系统作为攻击目标，但漏洞存在于**PHP本身**  
。  
即使是一个简单的"Hello World"应用，只要运行在受影响的PHP版本上，就可能受到攻击。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZga7VcdyrgE5Jh9MrukzxM3BxorTFIbUeOk70iaft5M3vlcGT9ZX7LaDygNhTP1nicVyTiaPwyBQAR6v3UBteFmjYyBGaVDMQtmIk/640?wx_fmt=png&from=appmsg "")  
### 版本影响的惊人数据  
  
Hadrian在10月21日的更新中提供了令人震惊的统计数据，基于W3Techs的数据：  
  
**PHP版本分布与修复期望**  
- PHP 4：0.1%，无官方修复  
  
- PHP 5：7.7%，无官方修复  
  
- PHP 7：27.0%，无官方修复  
  
- PHP 8.0：~3.8%，无官方修复  
  
- PHP 8.1：~9.9%，无官方修复  
  
- **总计无官方修复：~48.5%**  
  
有官方修复的版本：  
- PHP 8.2：~18.1%，有  
  
- PHP 8.3：~22.7%，有  
  
- PHP 8.4：~7.9%，有  
  
- PHP 8.5：~2.9%，有  
  
**合计来看，这约占所有可检测服务器端语言网站的1/3。**  
 Hadrian团队指出："维护良好的网站通常会隐藏其版本，一些Linux发行版还会向后移植他们分发的PHP包的修复。"  
## 技术细节深度剖析  
### 漏洞类别：内存安全问题  
  
所有这些漏洞都是内存安全问题，源于PHP在请求处理过程中处理致命错误的方式，特别是**内存不足错误**  
。这些错误可能导致PHP内部状态在请求意外结束后被破坏。后续请求或工作进程可以重用这个陈旧状态，导致内存泄漏或破坏。  
### CVE-2026-109618：上传进度反序列化UAF  
  
**根因**  
：PHP的Session模块可以在同一预初始化窗口期间解码upload-progress处理期间的会话数据。解码器将请求拥有的簿记存储在BG(unserialize)  
中。致命分配失败可以中断此路径，销毁请求堆并留下反序列化器的全局状态。  
  
**利用机理**  
：后来由同一工作进程处理的请求可以消耗攻击者重新获取的内存作为活动反序列化器簿记。陈旧状态变成强大的内存破坏原语。Wiz团队将其与内存泄漏相结合以消除地址不确定性，并在WordPress、Drupal和Joomla上实现了未经身份验证的远程代码执行。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZh1Iica2bEbjZjsdIRH5K0p3HuhvbQWlJOEEOB47wJ8m1e92OJeq0fHtn7yeMNhRbLbnhKaZy8HybUHheMenCEHM8AVwMSWn6W4/640?wx_fmt=png&from=appmsg "")  
### CVE-2026-109619：OPcache类型混淆  
  
**根因**  
：OPcache在共享内存中持久化已编译的PHP结构，并分配索引到每个工作进程运行时指针映射中。工作进程通过共享边界协调这些索引。在zend_accel_script_persist()  
期间，PHP推进进程本地CG(map_ptr_last)  
边界。只有在持久化完成后，它才发布相应的共享边界。这两个事件之间的分配失败会使进程本地推进到位，并让部分持久化对象引用共享状态仍认为可用的槽位。  
  
**利用机理**  
：当另一个工作进程稍后为不同的运行时对象重用这些槽位之一时，陈旧的共享元数据解析为错误类型的指针。这种不匹配会创建跨工作进程的类型混淆并破坏共享OPcache状态。Wiz团队使用此原语构建了针对WordPress的未经身份验证的RCE链。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZgnmicYBjLj2AfvIO6tRSJz5Gt9LjPvatZyPEaOm4C2A4Td8f1GulEMZ9uwaM9hSicic1tAdtPM42yiaNsOrsykXEliaOTmfffN9MHY/640?wx_fmt=png&from=appmsg "")  
### 攻击者获得的能力  
  
通过这些漏洞，攻击者可以实现两种关键能力：  
1. **读取敏感数据**  
：从PHP进程内存中读取密码、令牌、API密钥等敏感数据，**无需实现远程代码执行**  
  
1. **完全控制**  
：实现**未经身份验证的远程代码执行(RCE)**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/TKdPSwEibsZjS5yKhx5f1ayib6ptrBTnhenmuHpJyIXDodSzYqafZMx7GFibO8IkNwhwbxB4MUFXtKxibtmJ0bcymqF2eiala7T3E96FaUTmUol4/640?wx_fmt=png&from=appmsg "")  
## 从理论漏洞到实际武器  
### AI在漏洞发现中的革命性作用  
  
Wiz研究团队强调："这些漏洞在代码库中存在超过十年。将这些缺陷转化为工作利用需要链接多个错误并考虑PHP如何在请求之间分配和重用内存。内存错误是出了名的复杂；PHP版本、配置和应用程序行为的差异可能会破坏利用所依赖的假设。"  
  
**AI时代的改变**  
：有了AI，这不再是一个障碍。大型语言模型可以帮助使利用适应不同的PHP版本、配置和内存布局。通过测试和调试工具，模型可以分析失败并迭代调整，减少每个目标所需的专家工作量。这**显著降低了跨不同部署扩展攻击的障碍**  
。  
### AI工具链  
  
Wiz研究团队在整个研究过程中使用了以下AI工具：  
- **Wiz Atlas**  
：该公司自研的AI漏洞研究平台  
  
- **Gemini 4 Argon**  
：谷歌的先进语言模型  
  
- **GPT-5.6 Sol**  
：OpenAI的模型  
  
特别值得注意的是，**Gemini 4 Argon**  
在漏洞发现、利用开发、攻击检测和补丁验证方面做出了极高的贡献。  
### 影响深远的启示  
  
谷歌DeepMind安全主管Raluca Ada Popa评论道："使用Gemini 4 Argon，团队在Wiz展示了为什么前沿AI是网络防御的游戏规则改变者。十多年来，这些关键漏洞一直未被处理，因为漏洞发现和利用极其复杂。我们很高兴看到Gemini 4 Argon帮助防御者弥合差距，找到漏洞并帮助修补它们。"  
## 野外的攻击情况  
### 攻击时间线  
  
Hadrian团队提供了详细的攻击时间线更新：  
  
**10月21日09:50 UTC（补丁落地前）**  
：  
- 建议设置session.upload_progress.enabled = 0  
并重新加载PHP-FPM或Apache  
  
- 强调这仍不是完整修复  
  
**10月21日11:35 UTC**  
：  
- "我们现在看到php2shell在野外大规模利用"  
  
- "我们观察到的大多数攻击滥用PHP的会话上传进度路径"  
  
**10月21日14:35 UTC**  
：  
- "利用仍在大规模进行，所以今天修补是优先事项"  
  
- 提供了明确的修复版本：8.5.12、8.4.27、8.3.36和8.2.35  
  
### 实际攻击特征  
  
攻击通常表现出以下特征：  
1. **大量异常请求**  
：后跟内存限制错误或重复的HTTP 500响应  
  
1. **异常大或格式错误的PHPSESSID响应cookie**  
  
1. **无法解释的PHP工作进程变动**  
  
1. PHP工作进程(php-fpm、php-cgi、httpd、apache2)启动shell或下载工具  
  
## 立即缓解措施  
### 紧急缓解方案  
  
根据Hadrian团队的即时反馈，以下是可以立即实施的策略：  
  
**临时缓解措施**  
：  
1. **禁用session.upload_progress**  
：设置session.upload_progress.enabled = 0  
并重新加载PHP-FPM或Apache  
  
1. 重要：在php.ini或FPM池中设置，而不是通过ini_set()或.user.ini  
  
1. **Cloudflare保护**  
：保持通用规则-异常执行规则开启  
  
1. Cloudflare CTO已经表示他们正在看到早期攻击并更新缓解措施  
  
1. **隔离旧版本主机**  
：找到被遗忘的PHP主机（EOL 8.1及更早版本、CMS堆栈、设备），标记所有者并安排更换窗口  
  
**云服务商的响应**  
：  
- **Cloudflare**  
：发布了针对性的WAF规则  
  
- **Fastly**  
：实现了主动保护  
  
- **Google Cloud**  
：发布了响应指南  
  
- **AWS和Azure**  
：与Wiz合作制定客户指南  
  
### 检测与监控  
  
**关键监控点**  
：  
1. **进程监控**  
：PHP工作进程启动shell、解释器或下载工具  
  
1. **日志分析**  
：查找大而异常的PHPSESSID cookie  
  
1. **资源行为**  
：注意无法解释的PHP工作进程重启  
  
**Huntress建议的检测方法**  
：  
- 在Linux服务器上使用Florian Roth的process-watcher  
  
- 监控命令行中出现反向shell模式：/dev/tcp/  
、fsockopen  
、mkfifo  
  
- 警惕运行简单侦察命令的Web服务器：whoami  
、id  
、uname -a  
、ipconfig  
  
## 永久解决方案  
### 官方补丁版本  
  
PHP已经发布了以下固定版本：  
- PHP 8.5.12  
  
- PHP 8.4.27  
  
- PHP 8.3.36  
  
- PHP 8.2.35  
  
**重要提醒**  
：Linux发行版和LTS供应商已提前收到补丁，所以检查您的供应商的更新包。  
### 生命周期结束版本的处理  
  
**无官方修复的版本应对策略**  
：  
1. **升级到受支持版本**  
：最佳长期解决方案  
  
1. **检查分销商后向移植**  
：一些Linux发行版提供后向移植修复  
  
1. **源代码修复**  
：PHP已为7.4、8.0和8.1发布了最小源代码后向移植，适用于从源代码构建的团队  
  
1. **立即隔离**  
：如果无法升级，则将其置于VPN或WAF之后  
  
### 更新后的重启要求  
  
**关键执行步骤**  
：  
1. **更新PHP**  
：安装官方修复版本  
  
1. **重启工作进程**  
：必须重启PHP-FPM、Apache或其他运行PHP工作进程的服务  
  
1. **验证运行版本**  
：在将系统重新上线之前验证运行的版本  
  
1. **逐步上线**  
：首先修补最高风险主机  
  
## 历史比较与行业影响  
### 网络安全历史上的三个重大转折点  
  
Wiz团队将php2shell事件与历史上的重大安全事件进行比较：  
1. **2014年 Heartbleed**  
：暴露了保护互联网的共享基础设施的脆弱性  
  
1. **2021年 Log4Shell**  
：揭示了软件依赖性如何创建广泛暴露  
  
1. **2026年 php2shell**  
：标志着AI时代通过改变长期存在漏洞的实际可利用性来改变安全局面的转折点  
  
### 代码遗产与安全债务  
  
php2shell事件暴露了代码遗产的安全债务问题：  
1. **架构决策的长尾效应**  
：十多年前的决定可能带来数十年的安全风险  
  
1. **向后兼容性的双刃剑**  
：在支持旧功能的同时也可能延续安全隐患  
  
1. **技术债务的累积**  
：未解决的安全问题会随时间积累成系统性风险  
  
## 未来展望与应对策略  
### AI在网络安全中的新范式  
  
Wiz团队指出："这改变了发现和经济性。随着AI减少开发和适应复杂利用所需的努力，曾经似乎不切实际利用的漏洞成为可行目标。"  
  
**防御机会**  
：同样的能力也给了防御者机会去发现和修复那些经受多年审查的缺陷。  
  
**实际应用**  
：使这项工作变得常规：系统地在现有软件中找到缺陷，并在它们进入生产环境之前捕获新的缺陷。  
### 长期安全策略建议  
  
**供应链安全重建**  
：  
1. **深度依赖分析**  
：理解关键组件的依赖关系  
  
1. **持续更新策略**  
：制定严格的版本管理政策  
  
1. **替代方案规划**  
：为关键组件准备替代解决方案  
  
**AI辅助安全开发**  
：  
1. **自动漏洞检测**  
：集成AI工具到开发流程  
  
1. **代码审计自动化**  
：使用AI加速安全审查  
  
1. **威胁建模辅助**  
：利用AI支持安全设计决策  
  
## 参考资料  
- Hadrian.io：php2shell泄露的关键PHP RCE漏洞技术细节  
  
- Wiz.io：php2shell：AI揭示影响70%互联网的关键RCE漏洞   
  
- PHP官方：即将发布的关键PHP安全更新  
  
- BleepingComputer：PHP警告管理员两个关键漏洞，明日发布紧急补丁  
  
- Huntress：快速响应php2shell泄露的关键PHP漏洞  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
