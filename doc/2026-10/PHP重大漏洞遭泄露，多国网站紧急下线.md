#  PHP重大漏洞遭泄露，多国网站紧急下线  
 金天的网络安全   2026-10-22 04:01  
  
2026年10月21日，PHP官方披露两项严重漏洞并发布安全更新公告：CVE-2026-109618和CVE-2026-109619。建议运行PHP Web服务的用户尽快升级，并依据实际SAPI、应用操作及配置评估风险。  
<table><tbody><tr><td data-colwidth="576" style="border-color:#ff0000;"><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">漏洞名称:</span>PHP远程代码执行漏洞</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">官方评级：</span>严重（Critical，PHP 官方评级）</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">CVSS4.0评分:</span>两项均为9.2</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">CVE编号:</span>CVE-2026-109618/CVE-2026-109619</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">危害描述:</span>未经身份验证的攻击者可通过构造HTTP 请求触发 PHP 请求处理过程中的异常内存耗尽，使未正确初始化或已释放的内部状态被继续使用，进而造成 Web 工作进程崩溃、进程内存信息泄露，甚至以 Web 服务进程权限执行任意代码。</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">影响范围影响版本：</span>PHP 5.4.0–8.1.x；PHP 8.2.x &lt; 8.2.35；PHP 8.3.x &lt; 8.3.36；PHP 8.4.x &lt; 8.4.27；PHP 8.5.x &lt; 8.5.12。</span></section><section style="margin-bottom: 8px;"><span leaf=""><span textstyle="" style="font-weight: bold;">安全版本：</span>PHP 8.2.35、8.3.36、8.4.27、8.5.12，或对应分支后续安全版本。</span></section></td></tr></tbody></table>  
  官 方 修 复 方 案    
  
• 尽快升级至PHP 8.2.35、8.3.36、8.4.27、8.5.12或对应分支后续安全版本。这些版本同时修复上述两项漏洞；升级至包含完整补丁的版本是完整修复方案。  
  
• 从PHP官方或操作系统供应商获取更新。使用发行版回补补丁时，依据供应商安全公告确认修复状态，不能仅比较上游版本字符串。  
  
• 完成更新后重启全部PHP工作进程和相关服务，核对实际Web SAPI的版本及生效配置，避免仅检查CLI版本。  
  
  多 国 网 站 下 线 自 保    
  
PHP是一款网站开发编程语言，全球大约七成网站都在采用。本次爆出问题的相关代码最早在2012年推出的PHP 5.4.0版本中就已存在，该风险在默认配置下就会生效，也就是说这个漏洞已经潜藏超过十年。  
  
该漏洞在今年8月，由网络安全企业Calif与Wiz分别依靠AI技术发现。之后多方在保密机制下协同开展修复工作，历时近两个月。但在补丁正式发布前一日，即当地时间20日，漏洞详情被外泄。PHP安全团队只能对外确认补丁发布计划，黑客抓住这个窗口期，在补丁正式上线的数小时之前就启动攻击行动。  
  
据媒体消息，不少国家为抵御入侵风险，在补丁发布前临时关停相关站点。荷兰国家网络安全中心首先将官网下线，荷兰议会、鹿特丹港口以及多家医院网站陆续关停；芬兰暂停所有面向民众的政府线上业务，法国内政部临时关闭地方政务网站。德国数字部门相关发言人称，联邦IT服务机构ITZBund关停多处网站；日本多家大型互联网平台启动紧急运维。  
  
美国白宫相关人员称，国家网络总监办公室联合FBI等机构统筹处置，督促联邦单位与关键基础设施落实漏洞修复。  
  
PHP于21日放出8.5.12、8.4.27、8.3.36、8.2.35四个修复版本。开发团队提示，  
攻击者可借助该漏洞在无需账号密码的前提下远程接管服务器，升级版本是唯一可靠的防护手段。  
  
<table><caption><section style="text-align: left;"><span leaf=""><span textstyle="" style="color: rgb(172, 57, 255);font-weight: bold;">阅读延伸&gt;&gt;</span></span></section></caption><tbody><tr><td data-colwidth="576" style="border-color:#ffa900;"><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247484573&amp;idx=1&amp;sn=baaa0d75c7c97cdb212ae80a719e81ab&amp;scene=21#wechat_redirect" textvalue="“银狐”病毒进程注入排查" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">“银狐”病毒进程注入排查</span></a></span></section><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247484448&amp;idx=1&amp;sn=2c8b399541d4595a191c943b40636bfb&amp;scene=21#wechat_redirect" textvalue="“银狐”病毒：变种快、隐蔽强" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">“银狐”病毒：变种快、隐蔽强</span></a></span></section><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247484827&amp;idx=1&amp;sn=f6447f5d6157426eed947ae474888679&amp;scene=21#wechat_redirect" textvalue="近期多起供应链投毒事件安全风险分析" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">近期多起供应链投毒事件安全风险分析</span></a></span></section><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247484470&amp;idx=1&amp;sn=7a2b5a0969318e5c0e2d0bfa9d68d00f&amp;scene=21#wechat_redirect" textvalue="“银狐”病毒防御：构建多维立体安全防线" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">“银狐”病毒防御：构建多维立体安全防线</span></a></span></section><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247485471&amp;idx=1&amp;sn=fb46b50671758e19d830fd0a318d6f06&amp;scene=21#wechat_redirect" textvalue="AI合规要求盘点：大模型厂商 &amp; AI应用集成商" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">AI合规要求盘点：大模型厂商 &amp; AI应用集成商</span></a></span></section><section style="margin-bottom: 8px;"><span leaf=""><a class="normal_text_link mp_article_text_link" target="_blank" style="color: rgb(0, 128, 255);" href="https://mp.weixin.qq.com/s?__biz=MzkxMTczODAzNg==&amp;mid=2247485415&amp;idx=1&amp;sn=1b8fa48711ca6aa572ee412e76520703&amp;scene=21#wechat_redirect" textvalue="落地数据安全体系，还在只靠重金堆产品？" data-itemshowtype="0" linktype="text" data-linktype="2"><span textstyle="" style="color: rgb(0, 128, 255);text-decoration: underline;">落地数据安全体系，还在只靠重金堆产品？</span></a></span></section></td></tr></tbody></table>  
![](https://mmbiz.qpic.cn/mmbiz_png/11FgJZDETciajiaz8IQapvibxSyHsMic52Kv02XcmeXlE9hWWtRF3gibIS7JRgPCd64It5DvAuhObAPWVRt6MlNC7mxHheCaJ97SCRAVkzX8P5FM/640?wx_fmt=png&from=appmsg "")  
  
  
