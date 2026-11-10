#  ​OpenSSH 高危认证绕过漏洞遭攻击者利用，超长密码即可绕过 SSH 登录验证  
 看雪学苑   2026-11-09 09:59  
  
攻击者已开始针对 OpenSSH 的 SSHole 认证绕过漏洞发起攻击。SANS 互联网风暴中心监测发现，攻击者通过提交异常长度的密码，尝试绕过 SSH 身份验证，在特定配置的 Linux 服务器上获取未经授权的访问权限。  
  
  
近日，SANS Internet Storm Center（互联网风暴中心）发布最新监测报告，指出攻击者已经开始尝试利用 OpenSSH 中的 SSHole 认证绕过漏洞。  
  
  
该漏洞允许远程攻击者在特定条件下，通过提交超长密码绕过身份验证，访问原本禁用了密码登录的 Linux 用户账户。整个过程不需要知道目标账户的真实密码，也不需要持有 SSH 私钥。  
  
  
**一、攻击活动已出现，蜜罐记录持续增长**  
  
  
SANS 研究员 Johannes Ullrich 表示，其部署的 Cowrie SSH 蜜罐于 2026 年 11 月 5 日首次捕获到明确针对 SSHole 漏洞的利用尝试。  
  
  
攻击者使用的方式相当简单：向 SSH 服务提交由大量重复字符“A”组成的超长密码。  
  
  
监测数据显示：  
- 11 月 5 日：记录到 232 次超长密码请求。  
  
- 11 月 6 日：记录到 713 次超长密码请求。  
  
需要注意，这些数据代表可疑请求数量，并不意味着发生了 945 次成功入侵，也不能直接对应攻击者人数。  
  
  
事实上，蜜罐在 11 月 1 日就已经出现超长密码请求，但早期流量更像是异常的暴力破解脚本，而非有针对性的漏洞利用。  
  
  
研究人员还发现，部分脚本甚至将下载命令直接填入密码字段。由于 Cowrie 仅记录密码的前 255 个字符，研究人员无法完整还原所有超长密码请求。  
  
  
目前，监测到的主要扫描来源 IP 包括：  
- 213.86.169.133：48 次  
  
- 152.233.50.185：44 次  
  
- 92.255.57.7：43 次  
  
研究人员怀疑部分蜜罐登录可能已经成功，但尚未确认存在成功建立会话的证据。  
  
  
这意味着，目前可以确认漏洞已遭到主动探测和利用尝试，但实际入侵结果仍有待进一步核实。  
  
  
**二、SSHole 漏洞是如何绕过身份验证的？**  
  
  
SSHole 由研究人员 Özgün Kültekin 披露，主要影响使用 musl libc 的特定 OpenSSH 部署环境。  
  
  
漏洞并非影响所有 OpenSSH 服务器，需要同时满足以下条件：  
1. OpenSSH 运行在使用 musl libc 的环境中。  
  
1. 密码验证未通过 PAM（可插拔认证模块）处理。  
  
1. SSH 服务开启了密码认证。  
  
1. 目标用户的密码字段为 *，即账户处于禁用密码登录的状态。  
  
1. 系统中还存在另一个具有有效 SHA-crypt 密码哈希的本地账户。  
  
其核心问题出现在密码处理过程中。  
  
  
当攻击者提交超过 256 字节的密码时，musl libc 中的 crypt() 函数可能返回 *，用于表示密码处理错误。  
  
  
而 OpenSSH 在后续认证过程中，可能将这个返回值与目标账户存储的 * 密码字段进行比较，并错误地将其视为匹配，从而绕过身份验证。  
  
  
根据研究人员披露的信息，可触发该问题的密码长度范围为 257～1024 字节。  
  
  
需要强调的是，该漏洞并不意味着所有禁用密码登录的账户都存在风险。对于存储有效密码哈希的账户，这种绕过方式并不适用。  
  
  
**三、哪些服务器需要重点关注？**  
  
  
SSHole 的影响范围具有一定的配置限制，尤其需要关注使用 musl libc 的 Linux 发行版及相关 OpenSSH 部署。  
  
  
目前，OpenSSH 已在 10.6p2 版本中修复该问题，Alpine Linux 也已发布相应的修复软件包。  
  
  
对于服务器管理员，建议优先采取以下措施：  
  
**1. 尽快安装安全更新**  
  
升级到发行版提供的修复版本，并在更新后重启 SSH 服务（sshd）。  
  
注意：部分 Linux 发行版可能通过回移补丁的方式修复漏洞，因此不能仅凭 OpenSSH 版本号判断系统是否安全，应进一步检查发行版的软件包修订版本。  
  
  
**2. 暂时关闭密码认证**  
  
如果暂时无法完成补丁升级，可以在 SSH 配置中设置：  
  
PasswordAuthentication no  
  
该配置可以阻断此次漏洞利用所依赖的密码认证路径。  
  
  
**3. 检查 PAM 认证配置**  
  
StepSecurity 的测试也证实，通过 PAM 处理密码验证可以避免该漏洞触发。  
  
对于相关环境，应检查 SSH 服务的认证配置，确保密码验证经过 PAM 处理。  
  
  
**4. 排查异常登录记录**  
  
重点关注原本禁用密码登录的账户是否出现成功的密码认证记录，尤其是同一会话中先出现认证失败，随后又出现认证成功的异常情况。  
  
  
**四、总结**  
  
  
SSHole 漏洞再次提醒我们，身份认证机制中的边界处理问题，同样可能成为远程攻击的突破口。  
  
  
此次漏洞利用不依赖复杂的攻击链，仅通过构造特定长度的密码，就可能在满足条件的服务器上绕过 SSH 身份验证。  
  
  
虽然目前公开的蜜罐数据尚未证实大规模成功入侵，但攻击活动已经出现，相关服务器管理员应尽快核查运行环境、认证配置及软件补丁状态，避免系统暴露在潜在风险之中。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K0TH3JMnERBMXhwU5ibdTv6fHuTiahlxKG8AQ468AG01TkLCTOHictTvfTYHLibMPX4vxARiaTlQiavxbQ6Qqy7vO2JvfJqVOuicytSKM/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K3FBAsRA4Qb5jMMrSteyw7FHYpBdCfbrp0eW3FX45aLx5Q9iaHU4XKpeAlIG0EpBwcOiax99XIBJAtJBAuvdobb1rZiciby0P9T5Zs/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3kDy2ibUFvAgiaDMqB5BCCPAwSVxpgYgEXiadaEp4AGgGpOD2NRB87AVOvKKbabkFRDcgMLNkHMS1WZmjRzTqGBC6iaWH1r28xmZs/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3LvBnUoRJItQyyvcY0iczEAeYO4jgIz6ZCENicLBl6ibXbr2EP9eGngfoD3hlD5QnGnneHkp2UEm58P6BM5C8LjC0NwXZfkw46E0/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
