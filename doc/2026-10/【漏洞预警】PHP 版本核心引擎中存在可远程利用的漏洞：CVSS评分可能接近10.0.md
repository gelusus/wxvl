#  【漏洞预警】PHP 版本核心引擎中存在可远程利用的漏洞：CVSS评分可能接近10.0  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-21 08:43  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
2026年10月20日，PHP安全团队发出紧急警报，通知即将发布一个关键安全更新，修复两个CVSS v4.0评分均为**9.2**  
的核心引擎漏洞。然而更令人担忧的是，安全研究人员称这两个漏洞的信息已提前泄露，实际影响可能比官方公布的更为严重，  
CVSS评分可能达到**10.0**  
，属于最高级别的安全威胁。  
  
这一消息立即在全球网络安全界引发高度关注，因为PHP作为最流行的服务器端脚本语言，支撑着全球超过**70%的网站**  
，包括WordPress、Magento、Shopware、WooCommerce等大量流行的网络应用和电子商务平台。  
## 漏洞基本信息：双重大杀器CVE-2026-109618和CVE-2026-109619  
### 官方时间线与评估  
  
根据PHP官方公告，安全更新定于2026年10月21日14:00 UTC发布，解决了两个被PHP安全团队评级为CVSS v4.0 9.2的关键漏洞。PHP团队表示："我们保留在漏洞详情公开或在野利用出现时提前发布的权利。"  
### 泄露信息的更多细节  
  
然而，多个独立安全研究团队的报道揭示了更严峻的情况。安全公司watchTowr在10月20日向BleepingComputer证实，这两个漏洞"确实是真实的"，并称它们被跟踪为CVE-2026-109618和CVE-2026-109619，是远程代码执行(RCE)漏洞，被认为影响PHP 5、6、7和8版本。  
  
据Sansec公司分析，一个名为**PHP2Shell**  
的漏洞利用工具的截图已经与公告一起泄露。该工具横幅声称它对**从PHP 5.4.0开始的每个PHP版本**  
都有效。这些漏洞会影响运行PHP 5.4或更高版本的所有Web服务器，这包括Magento、Shopware和WooCommerce商店。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZjeYJsicm4AeQuKQs0S6ZGBaGnh2krib7IW4aqfHDGyG2Viapl4VNnIPqtrGibndWFedib6Y7D9Uk6ibiarI340wTicA84AsiaApFxHSr4Q/640?wx_fmt=png&from=appmsg "")  
## 技术影响的严重性分析  
### 影响范围的极端广泛性  
  
这两个漏洞的特殊之处在于它们存在于**PHP核心引擎本身**  
，而不是基于PHP构建的应用程序中。这意味着：  
1. **影响范围最大化**  
：所有使用受影响的PHP版本的网站都可能存在风险，而不仅仅是特定应用  
  
1. **修复复杂度高**  
：需要在服务器基础设施层面更新PHP，而不是简单的应用更新  
  
1. **利用门槛降低**  
：一旦漏洞细节公开，攻击者可以利用单一工具攻击大量不同平台  
  
### 核心问题：远程代码执行(RCE)的本质  
  
远程代码执行被认为是网络安全威胁中的"圣杯"，因为它允许攻击者在目标服务器上执行任意代码。在PHP核心引擎中发现的RCE漏洞尤其危险，因为：  
1. **无需身份验证**  
：攻击者无需登录或拥有任何凭据  
  
1. **完全控制**  
：成功利用可能导致完全服务器接管  
  
1. **隐蔽性强**  
：攻击痕迹可能很难检测  
  
## 版本影响：向后兼容性的可怕代价  
### 支持的版本对比实际影响  
  
PHP官方只支持8.2及以后的版本，但根据泄露信息，这个漏洞可能影响**远至PHP 5.4**  
的版本。这意味着数以百万计仍在运行旧版PHP的系统将面临严重风险，而这些系统往往得不到官方补丁支持。  
  
**关键统计**  
：  
- PHP 5.x：已停止支持多年，但仍广泛部署  
  
- PHP 7.x：许多托管服务商仍在使用  
  
- PHP 8.0-8.1：已到生命周期结束，但仍有大量安装  
  
### 实际风险矩阵  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">版本状态</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">估计安装量</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">官方修复</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">实际风险</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">PHP 5.4+</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">数百万</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">无</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">极高</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">PHP 7.x</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">数千万</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">可能无</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">高</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">PHP 8.0-8.1</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">大量</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">可能无</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">高</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">PHP 8.2+</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">多数</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">有</span></section></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">中等</span></section></td></tr></tbody></table>  
## 利用场景与现实威胁预测  
### 攻击者视角的吸引力  
  
Huntress公司的安全分析师指出："当PHP本身出现关键漏洞时，而不是在构建在其上的某个应用中，你不得不去寻找识别易受攻击的PHP版本的地方就会变得非常多，非常快。"  
  
**预测的攻击时间线**  
：Sansec公司预测"我们预期在发布后15分钟内开始大规模攻击。"这种快速的武器化能力反映了这类漏洞的严重性。  
### 检测与发现的挑战  
  
Florian Roth发布的process-watcher工具突出显示了检测这些bug利用的关键特征：**由PHP产生可疑子进程**  
。如Huntress所指出的那样，这表现在PHP工作进程（如php-fpm  
、php-cgi  
、php-cgi.exe  
、httpd  
或apache2  
）启动异常程序：  
- 像sh  
、bash  
、cmd.exe  
或powershell.exe  
这样的shell  
  
- 像python  
或perl  
这样的解释器  
  
- 像curl  
、wget  
、nc  
或certutil.exe  
这样的下载或网络工具  
  
## 企业应急响应与缓解策略  
### 立即行动清单  
  
基于目前可获得的信息，组织应采取以下紧急措施：  
  
**1. 完整的PHP资产清单**  
（最高优先级）  
- 全面盘点所有PHP安装，包括Web服务器、网络设备、控制面板  
  
- 特别注意可能捆绑php.exe  
或php-cgi.exe  
的供应商设备和Windows软件  
  
**2. 补丁准备计划**  
（时间关键）  
- 将美国东部时间10月21日上午10点规划为补丁窗口  
  
- 首先修补面向互联网的系统  
  
- 监控官方PHP公告获取修复版本号  
  
- 检查Linux发行版和托管服务提供商提供的后向移植包  
  
**3. 生命周期结束(PHP EOL)环境识别**  
- 识别运行PHP 5、7或8.0/8.1的系统  
  
- 考虑升级、隔离或部署WAF保护  
  
- 对于无法升级的系统，考虑将其置于VPN或WAF之后  
  
**4. 增强监控准备**  
- 设置对PHP工作进程启动shell、解释器或下载工具的警报  
  
- 在没有端点检测的Linux服务器上，考虑部署process-watcher进行快速可见性  
  
- 保留Web服务器和PHP日志以便事后分析  
  
**参考资料**  
：  
- PHP官方公告：即将发布的关键PHP安全更新  
  
- BleepingComputer：PHP提醒管理员注意两个关键漏洞，紧急补丁即将发布  
  
- Sansec：关键PHP RCE漏洞CVE-2026-109618和CVE-2026-109619  
  
- Huntress：快速响应php2shell，泄露的关键PHP漏洞  
  
- PHP基金会讨论：即将发布的关键PHP安全更新  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
