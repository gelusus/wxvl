#  【漏洞通告】OpenSSH musl 密码认证绕过漏洞  
深瞳漏洞实验室
                    深瞳漏洞实验室  深信服千里目安全技术中心   2026-11-06 05:06  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/APc6NwjLsxQDtibtunUw4ia0ozfIMiaS0WPMrTAoczVYqh4PV2XDiap8eGictZJZagq9VFd4qPudDqeXIFEOedtM172vdOfwqMFCLLVCKb3GWt80/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞名称：**  
  
OpenSSH musl 密码认证绕过漏洞(SF_2026_46117)  
  
**组件名称：**  
  
OpenSSH  
  
**影响范围：**  
  
7.3p1 <= OpenSSH < 10.6p2  
  
**漏洞类型：**  
  
绕过认证  
  
**利用条件：**  
  
1、用户认证：否  
  
2、前置条件：  
  
1）PasswordAuthentication 开启  
  
2）PAM不启用且sshd 运行于 musl libc  
  
3）存在 shadow 密码字段为 * 的账号且系统内至少还有一个设真实密码的账号  
  
3、触发方式：远程  
  
**综合评价：**  
  
<综合评定利用难度>：容易。  
  
<综合评定威胁等级>：高危，能造成认证绕过，获取服务器权限。  
  
**官方解决方案：**  
  
已发布  
  
  
  
  
**漏洞分析**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/APc6NwjLsxQNNwUz58OpvgbSw7PYNZtXN1yicXY7l8LsbdJ8D1fSibdS01X0d1FsdA095bWOC0mzFVOMHuIYo8ESok2Ss362xfTiaEtEBkIjlQ/640?wx_fmt=gif&from=appmsg "")  
  
组件介绍  
  
OpenSSH是使用SSH透过计算机网络加密通信的实现。它是取代由SSH Communications Security所提供商用版本的开放源代码方案。目前OpenSSH是OpenBSD的子项目。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/APc6NwjLsxQAnjxibN5kFTELsP1TPkqDUdPVE9THa9PibUW2wNZLiasyLtW0bHAl5B2wncIVdlB4h8SBiaiagcMuicFtsTU3ibJ4k92wegMekp86MY/640?wx_fmt=gif&from=appmsg "")  
  
**漏洞简介**  
  
  
2026年11月6日，深瞳漏洞实验室监测到一则OpenSSH组件存在绕过认证漏洞的信息，漏洞威胁等级：严重。  
  
  
sshd 在未启用 PAM 时，以 crypt() 的计算结果与 /etc/shadow 存储的口令字段直接比对来验证密码；当账户口令字段为 *（锁定或仅限密钥登录）时，sshd 会借用系统内其他真实哈希的盐值完成计算，而 musl libc 的 crypt() 对超过 256 字节的口令计算失败并返回字符串 * 而非 NULL，**导致攻击者提交任意 257–1024 字节的超长口令后，计算结果恰好与存储的 * 比对相等、验证通过。因此运行于 musl libc 且未启用 PAM 的 OpenSSH 服务（如 Alpine、OpenWrt）可被攻击者在无任何凭据的情况下登录这类本不接受密码登录的账户，并在特定场景下进一步获得 root 权限。**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/APc6NwjLsxR0w5aP9MBQIm3Ee02icia8cfrffK8b4Zy77wOug0KesPIutHFZoOUnU1GphPFicpUen9M85CaX2EdU420ibx3NTa4oqota8UaJRI4/640?wx_fmt=gif&from=appmsg "")  
  
漏洞验证  
  
搭建受影响的Alpine Linux，OpenSSH 9.7p1复现该漏洞，效果如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/APc6NwjLsxRZtYRCpoG1YUSA6YuYunJSK2Y10SQ39YQrICibXNiaJib8ALPbUzdjcklkGVBicbXzqwgF9WicKoYFhWbRsZc7hdPgZzdveKGhTyZE/640?wx_fmt=png&from=appmsg "")  
  
  
  
**影响范围**  
  
目前受影响的OpenSSH版本：  
  
7.3p1 <= OpenSSH < 10.6p2  
  
  
  
**解决方案**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/APc6NwjLsxTlEzIlu1Jsibqt5c10NxIZAlVgM5RzGvsyIR2HkhGU33x0bib8GWnAAsUmJ5GGIvicFRlEHeV5IcHb5eamqB8Mp4NbV726S6ibINM/640?wx_fmt=gif&from=appmsg "")  
  
**官方修复建议**  
  
  
升级OpenSSH 至最新版本：  
  
OpenSSH >= 10.6p2  
  
下载链接：https://www.openssh.org/txt/release-10.6p2  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/APc6NwjLsxQAhY6VlSgNzYZGlTSTcWHj6PAc7pOdSmtNricqYVa6qcbAlTu09hOv48blmy8kF7yic7DzO6vG3yAonlbz5cFcm8ftFvNbrgww0/640?wx_fmt=gif&from=appmsg "")  
  
**临时修复建议**  
  
  
检查sshd版本是否位于漏洞版本区间， 若无法立即升级，可在 /etc/ssh/sshd_config 中设置 PasswordAuthentication no改用公钥认证并重启 sshd。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/APc6NwjLsxQkUEYm6FLQb91327kmNiauOO16r8BlibKIpxtTH1JF1pDWT85yVqm0bT8C65KBgLAIIpxMVmtJfegiaNFHSfDcO7IibsmsltIsK1o/640?wx_fmt=gif&from=appmsg "")  
  
**深信服解决方案**  
  
  
**风险资产发现**  
  
支持对OpenSSH的主动检测，**可批量检出业务场景中该事件的受影响资产情况，**  
相关产品如下：  
  
**【深信服统一端点安全管理系统aES】**  
已发布资产检测方案，指纹ID:0000336。  
  
**【深信服云镜YJ】**  
已发布资产检测方案，指纹ID:0000336。  
  
**【深信服漏洞评估工具TSS】**  
已发布资产检测方案，指纹ID:0000336。  
  
  
  
参考链接  
  
  
https://sshhole.com/  
  
  
https://www.openssh.org/txt/release-10.6p2  
  
  
https://git.musl-libc.org/cgit/musl/commit/?id=b3c4cc121f70faea45389fe7ddc1127ed5cbd8bb  
  
  
  
时间轴  
  
  
  
**2026/11/06**  
  
深瞳漏洞实验室监测到OpenSSH musl 密码认证绕过漏洞信息。  
  
  
**2026/11/06**  
  
深瞳漏洞实验室发布漏洞通告。  
  
  
点击**阅读原文**  
，及时关注并登录深信服**智安全平台**  
，可轻松查询漏洞相关解决方案。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/APc6NwjLsxSLkoQgdSrh1RdEQYnHQuG09nKeb0TejuOBgCBphbrC15uIia0okJxcdRzwGnL5k6Mf2ia3t8eAJrTkJTiaNsj1Jql69DO6KKzv3o/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/w8NHw6tcQ5zvcIHbwGGYKbqDVYsVKzNNia1jYtHf49C7133AlDXAgex2W4lFvpia56tjQQDkiauNBrl08YbxqG01A/640?wx_fmt=jpeg&from=appmsg "")  
  
  
