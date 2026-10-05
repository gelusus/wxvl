#  cPanel & WHM曝3个严重漏洞: 最高CVSS 9.9，可导致Root代码执行与管理员会话劫持  
原创 tuto
                    tuto  杂杂咱谈   2026-10-04 02:39  
  
cPanel于2026年9月29日发布安全更新，修复了  
3个影响cPanel&WHM的安全漏洞  
，分别为  
CVE-2026-93698、CVE-2026-93029和CVE-2026-93697  
。  
  
其中，CVE-2026-93698漏洞存在于  
Multilang adminbin  
组件中，可导致攻击者执行任意命令并获得  
Root权限  
；另外两个漏洞则属于存储型XSS，攻击者可利用低权限账户向WHM管理界面植入恶意脚本，在管理员访问相关页面后，以管理员会话上下文执行操作。  
## 一、漏洞概述  
  
本次cPanel安全更新涉及3个漏洞：  
<table><thead><tr><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">CVE编号</span></span></section></th><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">漏洞类型</span></span></section></th><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">影响组件</span></span></section></th><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">严重程度</span></span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93698</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">命令执行</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Multilang adminbin</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVSS 9.9 / Critical</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93029</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">存储型XSS</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">WHM Manage SSL Hosts</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVSS 9.0 / Critical</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93697</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">存储型XSS</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">WHM Mass Modify Accounts</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVSS 9.0 / Critical</span></span></section></td></tr></tbody></table>  
CVE-2026-93698的CVSS v3评分为9.9，攻击向量为网络、攻击复杂度低、需要低权限，并可能造成机密性、完整性和可用性全面影响。  
  
另外两个XSS漏洞目前公开的CVE记录均为  
CVSS 9.0 Critical  
。  
# 二、技术细节  
## 1. CVE-2026-93698：Multilang Adminbin命令执行  
  
该漏洞源于  
Multilang adminbin对输入数据验证不足  
，攻击者可以通过该组件构造恶意输入，从而实现任意命令执行。  
  
cPanel官方确认，成功利用该漏洞后，攻击者能够以  
Root用户身份执行代码  
，从而获得服务器最高权限。  
### 漏洞利用链  
```
攻击者      
   ↓      
构造恶意输入     
   ↓      
Multilang adminbin     
   ↓     
输入验证不足     
   ↓     
任意命令执行     
   ↓     
Root权限    
   ↓    
完全控制cPanel服务器  
```  
  
由于Root权限可以访问服务器上的网站、数据库、邮件、配置文件以及其他托管账户，因此一旦成功利用，影响范围可能覆盖整个cPanel主机。  
## 2. CVE-2026-93029：WHM Manage SSL Hosts存储型XSS  
  
CVE-2026-93029存在于WHM的  
Manage SSL Hosts  
管理界面。  
  
漏洞允许低权限账户向相关页面写入恶意脚本。当拥有更高权限的WHM管理员访问受影响页面时，恶意脚本会在管理员的会话上下文中执行。  
### 攻击流程  
```
低权限cPanel账户      
        ↓     
写入恶意脚本    
        ↓     
Manage SSL Hosts   
        ↓    
恶意内容持久化    
        ↓  
WHM管理员访问页面     
        ↓    
XSS脚本执行     
        ↓    
继承管理员会话权限    
        ↓    
执行管理员可执行的操作
```  
  
cPanel官方指出，该漏洞可使非特权账户在WHM管理员会话上下文中执行脚本，并进一步执行管理员权限范围内的管理操作。  
## 3. CVE-2026-93697：WHM Mass Modify Accounts存储型XSS  
  
CVE-2026-93697漏洞则是位于WHM Mass Modify Accounts功能，同样属于存储型XSS漏洞。  
  
攻击者可以利用低权限账户植入恶意脚本。当管理员访问受影响页面后，脚本将在管理员的WHM会话上下文中执行。  
  
攻击者理论上可以借助管理员权限进一步执行账户管理等操作，因此在共享主机、多租户服务器环境中风险较高。  
# 三、漏洞影响  
  
三个漏洞形成了不同的攻击路径：  
### CVE-2026-93698  
  
低权限/符合漏洞利用条件 → 任意命令执行 → Root  
  
成功利用后可能导致：  
- 获取服务器Root权限  
  
- 控制网站及Web应用  
  
- 访问托管账户  
  
- 修改服务器配置  
  
- 访问数据库  
  
- 访问邮件数据  
  
- 部署恶意程序或WebShell  
  
- 对服务器进行进一步持久化  
  
cPanel官方明确指出，该漏洞成功利用后可获得Root级代码执行，并控制服务器上的账户、网站和数据库。  
### CVE-2026-93029/CVE-2026-93697  
  
攻击路径则主要表现为：  
  
低权限账户 → 存储恶意XSS → 管理员访问 → 管理员会话上下文执行 → 管理操作  
  
因此，这两个漏洞并非普通的客户端XSS，其核心风险在于  
攻击者可以借助高权限WHM管理员的会话执行管理操作  
。  
# 四、受影响版本  
  
cPanel官方表示，三个漏洞均影响  
所有受支持的cPanel/WHM版本  
，修复版本如下：  
<table><thead><tr><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">产品分支</span></span></section></th><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">修复版本</span></span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">cPanel &amp; WHM 11.110</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">11.110.0.148+</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">cPanel &amp; WHM 11.134</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">11.134.0.61+</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">cPanel &amp; WHM 11.136</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">11.136.0.45+</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">cPanel &amp; WHM 11.138</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">11.138.0.11+</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">WP Squared</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">11.138.1.13+</span></span></section></td></tr></tbody></table>  
cPanel的138分支变更日志也显示，  
11.138.0.11于2026.9.29作为Targeted Security Release发布  
。  
# 五、漏洞信息  
<table><thead><tr><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">项目</span></span></section></th><th style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">信息</span></span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">产品</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">cPanel &amp; WHM</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">漏洞数量</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">3个</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">CVE-2026-93698 / CVE-2026-93029 / CVE-2026-93697</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">最高严重等级</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Critical</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">最高CVSS</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">9.9</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93698</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Multilang adminbin命令执行</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93029</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Manage SSL Hosts存储型XSS</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CVE-2026-93697</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Mass Modify Accounts存储型XSS</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">最高影响</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">Root代码执行 / 管理员会话权限滥用</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">修复时间</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">2026年9月29日</span></span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px; font-weight: 500">CWE</span></span></section></td><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-size: 16px">CWE-79(XSS)等</span></span></section></td></tr></tbody></table>  
目前公开信息显示，CVE-2026-93029和CVE-2026-93697尚未被列入CISA KEV，相关情报页面也显示当前没有已知的公开利用活动。  
# 六、修复与建议  
### 1. 立即升级cPanel/WHM  
  
建议管理员不要仅升级到最低修复版本，而是直接升级至当前可用的  
最新稳定版本  
。  
  
cPanel官方明确建议用户更新至最新修复版本。  
  
可以通过cPanel/WHM自身的更新机制进行升级；对于无法正常自动更新的环境，应检查/scripts/upcp更新流程。  
### 2. 检查WHM暴露面  
- 限制WHM管理端口的公网访问；  
  
- 仅允许可信IP访问WHM；  
  
- 开启并强制使用MFA；  
  
- 减少拥有WHM高权限的账户数量；  
  
- 避免将WHM管理接口直接暴露到互联网。  
  
### 3. 检查潜在入侵痕迹  
  
对于尚未及时更新的服务器检查：  
```
WHM登录日志      
管理员会话活动     
账户创建/修改记录     
SSL配置变化    
异常命令执行     
异常Cron任务     
WebShell及后门文件      
异常出站连接
```  
  
尤其需要关注管理员账户是否出现异常登录，以及SSL主机、账户配置是否发生未经授权的变化。  
### 4. 已遭受攻击的服务器  
  
如果服务器在升级前存在可疑活动，不建议简单地升级后结束。  
  
需进一步：  
  
隔离服务器 → 保留日志 → 排查Root级持久化 → 检查账户 → 轮换密码/API Token → 检查网站及数据库 → 恢复可信环境  
# 七、总结  
  
此次cPanel安全更新并非单一漏洞，而是同时修复了  
1个可导致Root代码执行的严重漏洞 + 2个可劫持高权限WHM管理会话的存储型XSS漏洞  
。  
  
其中  
CVE-2026-93698风险最高，CVSS 9.9  
，cPanel官方确认成功利用后可以Root身份执行代码；而CVE-2026-93029和CVE-2026-93697虽然属于XSS，但由于攻击目标直接涉及WHM管理员会话，同样可能导致高权限管理操作。  
  
  
[#cPanel]()  
&WHM [#CVE]()  
-2026-93698 [#RCE]()  
  
  
