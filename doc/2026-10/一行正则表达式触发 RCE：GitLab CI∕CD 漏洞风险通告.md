#  一行正则表达式触发 RCE：GitLab CI/CD 漏洞风险通告  
 云鼎实验室   2026-10-14 10:13  
  
## 概述  
  
GitLab 是广泛应用于代码仓库管理的开源平台，基于 Git 构建，提供完整的 Web 代码协作服务，被众多企业和开发团队用于 CI/CD 流水线管理与代码托管。  
  
腾讯云安全中心监测到 GitLab CE/EE 被披露存在整数溢出漏洞，漏洞编号 **CVE-2026-97229**  
。攻击者可通过在 CI/CD 配置中构造特制的正则表达式，触发正则表达式编译器中的整数溢出，进而以 git 用户身份在服务器上执行任意操作系统命令。目前漏洞细节与 PoC 代码已在第三方渠道公开，利用门槛进一步降低，建议受影响用户立即开展安全自查并升级修复。  
  
## 漏洞详情  
  
GitLab CE/EE 的 CI/CD 正则表达式编译器基于 Ruby 的 Onigmo 正则引擎。在存在漏洞的版本中，由于程序大小计算存在整数溢出，当攻击者提交的正则表达式最初以紧凑的 US-ASCII 格式编译并通过 GitLab 的正则表达式验证后，若该表达式被用于匹配包含 UTF-8 字符的仓库路径，Onigmo 引擎会在验证完成后针对新编码重新编译该正则。在重新编译过程中，整数溢出导致生成无效的正则表达式指令，破坏控制流完整性，使攻击者得以在 Ruby 进程中执行受控的 Onigmo 虚拟机操作，最终以 GitLab 的 git 用户身份执行任意操作系统命令。  
  
**注：**  
要成功利用该漏洞，攻击者必须具有将恶意 CI/CD 配置文件 .gitlab-ci.yml 提交至公共项目分支的权限。若 GitLab 实例允许新用户自行注册并创建公共项目，则攻击者仅需一个普通账户即可满足利用条件。  
  
<table><thead><tr style="background: rgb(246, 248, 250);"><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">字段</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">漏洞名称</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">GitLab CE/EE 整数溢出远程代码执行漏洞</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">漏洞编号</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><strong style="color: rgb(0, 82, 255);"><span leaf="">CVE-2026-97229</span></strong></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">危害等级</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><span style="display: inline-block;background: rgb(215, 58, 73);color: rgb(255, 255, 255);font-size: 12px;font-weight: 600;padding: 2px 8px;border-radius: 3px;"><span leaf="">高危</span></span></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">漏洞类型</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">应用漏洞 / 整数溢出 / 远程代码执行（RCE）</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">影响范围</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">GitLab CE/EE 19.2.x / 19.3.x / 19.4.x</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);font-weight: 600;"><section><span leaf="">参考链接</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">https://cloud.tencent.com/announce/detail/2508</span></section></td></tr></tbody></table>  
  
## 影响版本  
  
受影响版本：  
- 19.2 <= GitLab CE/EE < 19.2.8  
  
- 19.3 <= GitLab CE/EE < 19.3.4  
  
- 19.4 <= GitLab CE/EE < 19.4.2  
  
官方修复后的安全版本：  
- GitLab CE/EE >= 19.2.8  
  
- GitLab CE/EE >= 19.3.4  
  
- GitLab CE/EE >= 19.4.2  
  
  
  
## 处置建议  
### 1、升级至安全版本  
  
官方已发布漏洞补丁及修复版本，建议评估业务受影响情况后参照下表升级。建议在升级前做好数据备份工作，避免出现意外。  
  
<table><thead><tr style="background: rgb(246, 248, 250);"><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">当前版本范围</span></section></th><th style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;text-align: left;font-weight: 600;color: rgb(26, 26, 26);"><section><span leaf="">建议升级至</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">19.2.x（&lt; 19.2.8）</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">GitLab CE/EE 19.2.8</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">19.3.x（&lt; 19.3.4）</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">GitLab CE/EE 19.3.4</span></section></td></tr><tr><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">19.4.x（&lt; 19.4.2）</span></section></td><td style="border: 1px solid rgb(234, 234, 234);padding: 8px 12px;color: rgb(63, 63, 63);"><section><span leaf="">GitLab CE/EE 19.4.2</span></section></td></tr></tbody></table>  
### 2、临时缓解措施（升级前可先执行）  
- 限制对公共项目的匿名访问，并阻止不受信任的用户创建或写入公共项目，以降低被利用的风险。  
  
- 监控异常的 CI/CD 管道执行行为和 git 用户下的可疑进程，及时发现潜在的攻击活动。  
  
  
  
## 参考链接  
- 腾讯云安全公告：https://cloud.tencent.com/announce/detail/2508  
  
**-END-**  
  
**——关注云鼎实验室，获取更多安全情报——**  
  
  
