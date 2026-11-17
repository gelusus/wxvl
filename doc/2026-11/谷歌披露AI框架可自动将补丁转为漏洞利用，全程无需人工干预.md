#  谷歌披露AI框架可自动将补丁转为漏洞利用，全程无需人工干预  
 FreeBuf   2026-11-17 10:00  
  
![FreeBuf](https://mmbiz.qpic.cn/mmbiz_gif/icBE3OpK1IX1UyZu9ic96ufRXia8fE56q0LKgUEHDDYFxv7bvsjAJRLRHrmtQw6k0TtYETic0lJR8sAnAvUx5B6VMGpamkGqDP0iaC7ibibANAPHbI/640?wx_fmt=gif "")  
  
  
![全自主AI攻击框架](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX1BMsaSsLiamgcmI2QwTCXSfonNzzd66olFPDzZ6nhbROXEOTAAPwDGoZT369yufeAexDmLD358hppRJoxKibpuLTDISnlXicDezI/640?wx_fmt=png "")  
  
  
Part  
01  
  
首次发现全自主AI攻击框架  
  
  
谷歌威胁情报小组（GTIG）今日发布最新报告，首次发现攻击者使用全程无需人工干预的AI框架，可直接将安全补丁转化为漏洞利用程序。  
  
  
这个名为exploit_agent的框架发现于一台公网暴露的攻击前置服务器上，已接入多个前沿大模型及本地部署的大语言模型。GTIG指出，该框架会持续比对热门Web应用每次补丁发布前后的代码差异。一旦识别出潜在漏洞，就会自动编写漏洞利用程序并执行。  
  
  
同一台服务器上还存放着从受害者处窃取的数据，同时存放有多类脚本，可用于实施暴力破解、安装webshell，甚至清除其他攻击团伙留下的webshell。受害者侧遥测数据显示，框架运营者主要瞄准法国媒体、娱乐、教育领域的网站。框架配置文件中有一条俄语注释，提醒部署者将密钥"填在这里，不要填在聊天窗口里"。  
  
  
两个月前发布的上一季度跟踪报告中，GTIG还表示尚未在野观测到针对目标部署的全自主攻击链路。这份题为《Agentic威胁活动跨过重要门槛》的最新报告显示，过去一个季度情况已经发生变化。由于已修复漏洞的源码参考范围有限，GTIG认为，攻击者让LLM针对n-day漏洞生成利用代码，比挖掘0Day漏洞效率更高。  
  
  
Part  
02  
  
漏洞利用生成周期大幅缩短  
  
  
漏洞披露到可用利用程序生成的时间差正在大幅缩短，PaperCut旗下打印管理服务器的两个漏洞就是典型案例：今年8月底，攻击者将这两个漏洞组合利用，作为0Day发起攻击。安全研究人员与Huntress Labs在漏洞披露后不到一天，就用大语言模型生成了可用的漏洞利用程序。  
  
  
漏洞披露4天后，GreyNoise Intelligence监测到一起攻击活动。据信攻击者提前借助LLM生成了对应的漏洞利用程序，才得以入侵数百家机构。GreyNoise发现，攻击者在其中一家机构获得初始访问权限后，仅用几分钟就拿到了域管理员权限。  
  
  
除此之外，威胁行为者已开始使用开源agentic渗透测试框架，将实战入侵中的分散步骤自动化。在一起针对美国航空航天领域的入侵事件中，研究人员观测到威胁行为者使用了multi-agent框架CAIRN。  
  
  
另有威胁行为者在8月底至10月初期间，运行基于PentAGI搭建的定制化平台，对亚洲地区大量机构开展漏洞扫描。从当前情况看，无限制的自主执行会留下大量清晰的取证痕迹，这类框架反而更容易被防御方检测到。  
  
  
Part  
03  
  
AI技术遭滥用于多类恶意场景  
  
谷歌已采取相关处置措施  
  
  
GTIG表示，如今影响行动中对模型的使用早已不止撰写帖子这么简单。有商业承包商借助Gemini API搭建了一套交钥匙平台，利用云手机农场可运营超过10万个社交媒体账号。另有一个平台专门针对欧洲和中东地区的政治舆论开展操作，为了让生成的照片看起来像智能手机实拍，其使用的提示词特意要求模拟相机成像缺陷。  
  
  
在政府背景威胁行为者的活动中，被编号为UNC6836的疑似朝鲜组织曾用虚假招聘账号接触北美和欧洲的加密货币机构。研究人员发现这些账号使用的头像图片带有谷歌SynthID水印。该组织之后还篡改内部代码仓库中的.claude/settings.json文件，让用户每次启动Anthropic的Claude Code时，都会自动下载其投放的SOMBERMEME后门。  
  
  
谷歌表示，已封禁所有排查到的、涉及滥用Gemini的账号和相关资产。Google DeepMind也已根据这些发现，强化了旗下分类器的检测能力。  
  
  
参考来源：  
  
Google says an AI framework turned security patches into exploits with no human in the loop  
  
https://siliconangle.com/2026/11/16/google-says-an-ai-framework-turned-security-patches-into-exploits-with-no-human-in-the-loop/  
  
### 推荐阅读电报讨论  
  
  
