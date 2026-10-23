#  满分10分！GitLab又双叒叕出新漏洞了……  
原创 0x00
                        0x00  随笔漫记安全路   2026-10-23 09:11  
  
GitLab发了一次罕见的**Critical Patch Release**  
：10月21日，一次放出7个补丁版本，核心是CVE-2026-110264——**CVSS 3.1满分10.0**  
。  
  
**未经身份认证**  
的攻击者，通过编译一个特制的正则表达式，就能在GitLab服务器上**执行任意代码**  
。影响范围横跨十余年：**从7.0开始的所有GitLab CE/EE版本**  
都中招。  
  
自己搭GitLab的团队，这篇文章看完请立刻安排升级。  
  
**漏洞核心信息**  
- CVE编号：CVE-2026-110264  
  
- 类型：整数溢出 → 远程代码执行  
  
- CVSS 3.1：**10.0**  
（AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H）  
  
- 攻击向量：网络 / 复杂度低 / **无需权限**  
 / 无需用户交互  
  
- 影响范围：GitLab CE/EE 7.0 ≤ 版本 < 18.11.13（以及19.0~19.5各分支）  
  
- 报告者：安全研究员joaxcar，通过HackerOne赏金计划提交  
  
- PoC/EXP：暂未公开；**启明星辰VSRC通报标注"在野利用：已发现"**  
  
CVSS四项关键指标全是最高档：网络可达、低复杂度、零权限、零交互——这是教科书级的"谁都能打"漏洞组合。  
  
**技术原理：正则编译器的整数溢出**  
  
漏洞出在GitLab底层Ruby所用的正则表达式引擎**Onigmo**  
（Oniguruma-mod）的编译过程中。  
  
攻击链路：  
1. 攻击者向GitLab提交一个**特制的正则表达式**  
（GitLab多处功能支持用户输入正则——如CI规则、issue关键字匹配、分支保护规则等）  
  
1. Onigmo编译该正则时发生**整数溢出**  
，分配的缓冲区小于实际需要  
  
1. 在特定条件下，溢出内存被可控数据覆盖  
  
1. 最终在GitLab服务器上**执行任意代码**  
——以GitLab服务进程的权限  
  
"特定条件"这个措辞说明利用有门槛，但CVSS把攻击复杂度评为"低"，GitLab官方也直接定性为Critical并连夜发版——结合通报里"在野利用已发现"，实际威胁不低。  
  
**GitLab官方同步放出了两条检测规则**  
，供自管实例排查利用痕迹：  
- **Onigmo native worker crash**  
——正则引擎原生进程崩溃  
  
- **Org mode render error burst**  
——Org模式渲染错误爆发  
  
自己的实例如果日志里出现过这两类异常，按"可能已被利用"处理。  
  
**影响范围与修复版本**  
<table><thead><tr><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">当前版本</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">升级到</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">18.11.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">18.11.13</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.0.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.0.10</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.1.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.1.9</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.2.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.2.9</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.3.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.3.6</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.4.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.4.3</span></strong></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">19.5.x</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">19.5.1</span></strong></td></tr></tbody></table>  
GitLab.com和Dedicated已打补丁，**自建实例必须自己动手**  
。这波历史跨度罕见的补丁（18.11一路backport到7.0时代以来的所有在用分支）也说明官方认为威胁等级足够高。  
  
注意：**暂无临时缓解措施**  
——启明星辰通报明确写了"临时措施：暂无"。唯一解就是升级。在升级完成前，能做的只有收紧暴露面：GitLab不要裸奔公网、限制到VPN/内网访问、WAF拦截含正则输入的请求。  
  
**同批修复的其他漏洞**  
  
这次Critical Patch一共修了13个问题，除满分漏洞外还有几个值得注意：  
- **CVE-2026-15723（High）**  
：OmniAuth SAML/LDAP自动关联的邮箱查询认证缺陷——配合自建SSO的企业注意  
  
- **OAuth/OIDC发现端点缓存投毒（High）**  
——影响认证流程完整性  
  
- GraphQL work item邮箱字段访问控制缺陷（Medium）  
  
- EE版SAML/LDAP成员锁强制不完整（Medium）——可被未授权加成员  
  
- 内部凭据暴露在响应头中（Medium）  
  
一次补丁修掉认证、授权、缓存投毒、凭据暴露四类问题，建议整体升级不要只盯着满分那个。  
  
**应急行动清单**  
1. **确认版本**  
：管理后台或gitlab-rake gitlab:env:info  
看当前版本  
  
1. **对照上表升级**  
：生产环境建议先备份（gitlab-backup create  
）再升  
  
1. **回查日志**  
：检索Onigmo worker crash和Org mode渲染错误爆发两类异常  
  
1. **排查暴露面**  
：公网直连的GitLab评估是否需要收敛到VPN/内网  
  
1. **升级后观察**  
：关注GitLab官方issue页（gitlab.com/gitlab-org/gitlab/-/work_items/633633）的后续分析  
  
**结语**  
  
GitLab是很多公司的代码核心资产库——源代码、CI/CD流水线、部署密钥全在上面。一个满分RCE落在它身上，等于核心研发基础设施的钥匙孔暴露在了公网。  
  
PoC未公开+官方连夜发版，现在是**补丁窗口期**  
——PoC公开前的每一天都是升级的黄金时间。别等武器化exploit满天飞了再动手。  
  
**参考来源**  
：GitLab官方补丁公告（docs.gitlab.com）、CVE-2026-110264记录（mitre.org）  
  
