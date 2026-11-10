#  CISA 紧急预警：4 个高危漏洞已遭在野利用，含 OpenSSH “SSHole” 认证绕过漏洞  
原创 维度攻防
                        维度攻防  维度攻防   2026-11-10 08:28  
  
> 导语：美国 CISA 将 4 个高危漏洞录入 KEV 已知被利用漏洞目录，其中 OpenSSH “SSHole” 漏洞已观测到真实攻击，攻击者可实现认证绕过、植入持久后门、下发挖矿木马，企业请尽快排查升级。  
  
  
近日，美国网络安全和基础设施安全局 CISA，将 4 个高危安全漏洞正式加入  
KEV（已知被利用漏洞）目录，确认上述漏洞已经出现真实在野攻击行为，风险等级极高，提醒各机构尽快完成漏洞修复工作，美国联邦民用机构被要求在  
2026‑11‑12 前完成全部修复处置。  
> 💡小科普：只要漏洞被录入 CISA KEV 目录，就代表已经捕获到互联网真实攻击样本，是安全运营人员的高危修复优先级清单，政企单位建议参考该时间窗口完成整改。  
  
## 📋4 个高危漏洞详情  
  
表格  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">CVE 编号</span></section></th><th style="text-align: left;"><section><span leaf="">漏洞名称</span></section></th><th style="text-align: left;"><section><span leaf="">CVSS</span></section></th><th style="text-align: left;"><section><span leaf="">漏洞简述</span></section></th><th style="text-align: left;"><section><span leaf="">补丁时间</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-weight: 500;">CVE‑2026‑123158</span></span></section></td><td style="text-align: left;"><section><span leaf="">SSHole（OpenSSH）</span></section></td><td style="text-align: left;"><section><span leaf="">8.1</span></section></td><td style="text-align: left;"><section><span leaf="">OpenBSD OpenSSH 异常返回值漏洞，<span textstyle="" style="font-weight: 500;">可实现认证绕过</span>，攻击者无需合法账号即可突破 SSH 身份校验</span></section></td><td style="text-align: left;"><section><span leaf="">2026‑11</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-weight: 500;">CVE‑2026‑102895</span></span></section></td><td style="text-align: left;"><section><span leaf="">GPUStack</span></section></td><td style="text-align: left;"><section><span leaf="">9.8</span></section></td><td style="text-align: left;"><section><span leaf="">身份认证缺陷，<span textstyle="" style="font-weight: 500;">未授权获取管理员权限</span>，可篡改模型配置、窃取密钥、执行任意代码</span></section></td><td style="text-align: left;"><section><span leaf="">2026‑07</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-weight: 500;">CVE‑2026‑116779</span></span></section></td><td style="text-align: left;"><section><span leaf="">SignalWire FreeSWITCH</span></section></td><td style="text-align: left;"><section><span leaf="">9.8</span></section></td><td style="text-align: left;"><section><span leaf="">代码注入漏洞，泄露配置与账号凭证、接管呼叫控制、远程执行命令</span></section></td><td style="text-align: left;"><section><span leaf="">2026‑10</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf=""><span textstyle="" style="font-weight: 500;">CVE‑2026‑123218</span></span></section></td><td style="text-align: left;"><section><span leaf="">IBM Aspera Faspex</span></section></td><td style="text-align: left;"><section><span leaf="">9.8</span></section></td><td style="text-align: left;"><section><span leaf="">授权绕过，已认证用户可构造特殊报文提权，实现权限提升</span></section></td><td style="text-align: left;"><section><span leaf="">2026‑11</span></section></td></tr></tbody></table>## 🚨重点关注：OpenSSH SSHole（CVE‑2026‑123158）攻击现状  
  
这个被命名为  
SSHole的 OpenSSH 漏洞风险尤其值得运维人员重点关注，从 2026 年 11 月 6 日起，安全厂商已经监测到互联网上出现大量针对该漏洞的攻击尝试。  
1. Previdian 安全厂商统计，已经记录来自 14 个独立 IP 合计 22 次漏洞利用尝试；  
  
1. GreyNoise 蜜罐数据显示：首次攻击发生于 UTC 时间 11 月 7 日 00:46，6 个攻击源 IP 共计产生 18 套完整攻击链；  
  
1. 攻击行为包含资产侦查、读取密码哈希、窃取 root SSH 私钥、修改authorized_keys  
写入 SSH 公钥建立持久后门；  
  
1. 威胁团伙成功入侵后，会部署 TAC Agent 远控木马，实现命令执行、系统探测、文件上传下载；部分黑客利用该漏洞植入  
挖矿木马、构建僵尸网络；  
  
1. 多家厂商表示：当前攻击规模尚未大规模爆发，但漏洞利用代码已经流出，预计攻击数量会快速上涨。  
  
> 其他 3 个高危漏洞，目前公开情报尚未披露详细攻击手法、攻击组织与攻击规模，但 CISA 确认存在真实利用，不可掉以轻心。  
  
## ✅企业应急处置建议  
1. 优先升级补丁  
  
   - OpenSSH：尽快升级至 2026‑11 发布的安全修复版本，修复 SSHole 认证绕过漏洞；  
  
   - GPUStack：升级至 7 月发布的修复版本，该漏洞允许无授权获取管理员权限，暴露在公网的实例风险极大；  
  
   - FreeSWITCH、IBM Aspera Faspex：升级对应官方安全版本。  
  
1. 入侵痕迹排查（非常重要，仅打补丁无法清除已经入侵的攻击者）  
  
   - 检查 SSH 的authorized_keys  
文件，确认是否存在陌生公钥；  
  
   - 审计系统账号、crontab 定时任务，排查可疑计划任务；  
  
   - 检查系统进程，查找挖矿程序、TAC Agent 等可疑远控程序；  
  
   - 核查系统日志、SSH 登录日志，关注陌生 IP 登录行为。  
  
1. 网络边界防护  
  
   - 公网 SSH 服务尽量关闭，如业务必须开放，启用 IP 白名单访问；  
  
   - 对公网暴露 GPUStack、FreeSWITCH、IBM Aspera Faspex 服务做访问控制，限制访问源地址。  
  
1. 持续威胁监测  
  
   - 安全设备增加对 SSHole、GPUStack 漏洞利用行为的告警监测，持续关注安全厂商威胁情报更新。  
  
