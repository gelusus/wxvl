#  Windows远程访问API存在漏洞，攻击者可获取PC完整SYSTEM控制权  
 FreeBuf   2026-11-25 10:26  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX3HckWXWuiaExSw8kdJ4Q0mhtwTRmEOzEDx5lKjhAauYsIpYw7qbMEyuL6LQjC99p0vPmMEBmNKkCuibKEIe9bnsESe3iadFAowYs/640?wx_fmt=gif "")  
  
  
![Windows远程访问API漏洞](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX2eMxerA1T6Hn3mOqgSBcOXaLXDV13WXaFJNWeTTS0fL7ZQn9ODydCk4bVEVAfIVNGKP7KLwakS0DkCYkuWBCdibovlHqSZbzQ0/640?wx_fmt=png "")  
  
  
Windows远程访问API存在一处漏洞，仅拥有计算机有限访问权限的攻击者可利用该漏洞以SYSTEM权限运行代码。微软在2026年11月10日发布的补丁星期二更新中修复了该漏洞，编号为CVE-2026-119853。  
  
  
微软将该漏洞评级为“重要”，CVSS基础评分为7.8，更新覆盖Windows 10、Windows 11、Windows Server的所有受影响版本。  
  
  
尽管该组件名称中带有“远程”字样，但攻击者无法通过互联网直接入侵Windows设备。攻击者必须预先获得受影响设备的代码执行权限，无需管理员权限，也不需要其他用户配合。  
  
  
![PoC DLL被svchost.exe以SYSTEM权限加载](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3Ofs1ic5TELYkiaPOEibl9St19poibQ719crIicz6NvjLJSN9u3oWbSVNVVy6mHIyPIdiaEediacxbnC4tvA7sKcRPVwYhics3OKeoWV8/640?wx_fmt=png "")  
  
  
攻击者获得初始立足点后，该漏洞的危害便会显现：以SYSTEM权限运行的代码可以访问受保护文件，修改标准用户无权改动的系统设置。  
  
  
Part  
01  
  
远程访问服务存在路径漏洞  
  
  
Windows远程访问服务（RAS）主要负责管理VPN等网络连接，会将连接设置存储在配置文件中，并支持通过软件库自定义连接处理逻辑。  
  
  
微软将该漏洞归类为不受信任的搜索路径问题，即具备高权限的Windows组件可能从攻击者可控的路径加载代码。该漏洞的核心风险并非用户可以修改自身的连接设置，而是Windows最终会以RAS服务的权限运行用户控制的代码。  
  
  
![PoC以标准用户权限启动，最终打开NT AUTHORITY SYSTEM权限的控制台](https://mmbiz.qpic.cn/sz_mmbiz_png/icBE3OpK1IX06JOOK6pia6N8HicyVicU8Jk5ibKiahXf9F7TPWazuQslIbfWsbOfPS9ib1EQnQgGrvibzLGJLtd09lobibaxR0obGevcCxQ23nUUeRRU/640?wx_fmt=png "")  
  
  
Part  
02  
  
研究人员完成漏洞复现  
  
  
Core Security在11月23日发布的分析报告中指出，研究人员观测到Windows服务主机进程在SYSTEM账户下加载了研究者控制的动态链接库（DLL）。研究人员在Windows 11 Pro 25H2（内部版本26200.9711）环境下完成验证，测试过程无需连接VPN或互联网。  
  
  
这一测试结果进一步明确了该漏洞的利用边界：攻击者必须先获得本地代码执行权限，无法直接通过远程触发。作为对比，PhantomRPC是另一个独立的Windows提权漏洞，成因是服务边界处存在不同类型的安全缺陷。  
  
  
微软首次发布安全公告时表示，尚未观测到该漏洞被在野利用，但将利用可能性评级为“更可能发生”。目前已有研究演示证明该漏洞可在测试环境下被利用，但没有证据表明攻击者已在真实系统中使用该漏洞。这一情况与此前Windows SignDrop漏洞披露时的相关争议存在明显区别。  
  
  
![Process Explorer显示cmd.exe以SYSTEM权限运行](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX3liaV46KWTj5X18ywbhYSgFPaooYBw6EZ8CxiavplHgNsYJtRK22rU60pqvSDHljRSbfLTXD6ibnlcD9shyZu7Wf97LfOEG7K5Uo/640?wx_fmt=png "")  
  
  
Part  
03  
  
用户需及时安装补丁  
  
  
管理员应尽快安装对应版本的11月安全更新，或包含该修复的后续累积更新。微软安全公告已按Windows版本列出了对应的更新包和内部版本号。  
  
  
例如，Windows 10 22H2对应的补丁为KB5131096，安装后内部版本号升至19045.7846；Windows Server 2022对应的补丁为KB5131090，安装后内部版本号升至20348.5870。用户可通过Windows版本健康页面，对照版本更新历史核查当前系统的内部版本号。注意，修改VPN配置文件无法替代补丁修复。  
  
  
在补丁推送完成前，安全团队可筛查端点日志，排查Windows服务主机进程从普通用户可写入路径加载异常DLL的行为。发现此类活动时，应先核查对应DLL的路径和数字签名，再检查同一时间点服务主机启动的所有进程。  
  
  
参考来源：  
  
Windows Remote Access API Flaw Could Let Hackers Gain Full SYSTEM Control of PCs  
  
https://cybersecuritynews.com/windows-remote-access-api-flaw/  
  
###   
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651351016&idx=3&sn=d224081e5f21f76580f08cd93c037d34&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
###   
  
  
  
