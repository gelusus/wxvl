#  开源 Webmail 巨头 Roundcube 爆出在野 漏洞  
原创 播风者
                    播风者  黑白之道   2026-10-05 01:12  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MSx2fqEGslnmU6zK8IQbZxqdPDsBNP21xF455gnEMHJQE0U7a6pz6k5smicMvbDvBCQv5WNSY8oswdNQSw5uQgLwLzaW5jvAUA/640?from=appmsg "")  
> **导语**  
：Roundcube 官方于2026年5月24日悄然发布安全补丁，修复了 CVE-2026-48842（CVSS 8.1）。然而直到9月下旬，加拿大网络安全中心（CCCS）与 SOCRadar 才联合拉响警报——黑客已针对全球仍未打补丁的**50多万台**  
 Roundcube 服务器展开大规模自动化扫荡与实战利用。从补丁发布到在野爆发，中间相隔**整整四个月**  
，大量企业和政府邮件系统在此期间已沦为黑客的"数据提款机"。  
  
## 一、漏洞速览  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">项目</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">详情</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">漏洞编号</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">CVE-2026-48842</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">CVSS 3.1 评分</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">8.1（高危）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">向量</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">漏洞类型</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">SQL 注入（SQL Injection）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">受影响组件</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">virtuser_query 插件</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">攻击前提</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">无需账号，</span><strong><span leaf="">预认证攻击（Pre-Auth）</span></strong></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">危害结果</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">越权读取/篡改数据库用户信息、通讯录等敏感资产</span></section></td></tr></tbody></table>  
**风险定级：🔴 高危**  
——攻击无需凭证，可直接入数据库，影响面广，在野已有利用。  
## 二、技术根因分析  
### 2.1 缺陷位置  
  
漏洞根植于 Roundcube 的 **virtuser_query**  
 插件。该插件负责在用户登录验证**之前**  
，将邮箱地址映射至对应的数据库账号，是 Roundcube 用户认证流程的关键桥梁。  
### 2.2 缺陷机理  
  
问题出在 PHP 代码对用户输入的处理环节——使用了存在缺陷的 preg_replace()  
 正则反斜杠转义逻辑。正常流程中，系统应对用户传入的字符进行转义以防止 SQL 语义被篡改；然而攻击者发现，当输入中包含特定构造的**反斜杠转义序列**  
时，preg_replace 的转义行为会产生"逃逸"效果：  
- 反斜杠在正则替换过程中被二次解析  
  
- 单引号被意外闭合并脱离原 SQL 语句结构  
  
- 攻击者由此注入自定义的 SQL 语句片段  
  
这种绕过方式在 SQL 注入中属于**字符串逃逸**  
类攻击，经典而高效，且极难被传统规则类 WAF 捕获。  
### 2.3 攻击路径  
  
攻击者只需在登录界面或用户查询接口，构造包含特殊反斜杠序列的 Payload，无需任何有效账号，直接发送请求即可触发 virtuser_query 插件的 SQL 查询。Payload 示例结构（以单引号闭包为核心）：  
```
...' OR '1'='1 [构造的反斜杠逃逸序列] --
```  
  
一旦触发成功，攻击者即可：  
- 枚举数据库中所有用户账号信息  
  
- 读取/导出通讯录数据  
  
- 在某些配置下可进一步写入或提升权限  
  
![Roundcube CVE-2026-48842 攻击路径示意图](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6M2SzenKsGqZDkXnEfl0QcabOffx9OJ3z4A0RoN2nu432UkyyFvqKvUtVqV6D24klrduRKINuzQIYj7SkHOlaePo4skV5RdF9c/640?from=appmsg "Roundcube CVE-2026-48842 攻击路径示意图")  
## 三、影响范围  
  
**受影响版本**  
：  
- Roundcube 1.6.x（**1.6.16 之前**  
的所有版本）  
  
- Roundcube 1.7.x（**1.7.1 之前**  
的所有版本）  
  
**不受影响版本**  
：  
- Roundcube 1.6.16 及以上  
  
- Roundcube 1.7.1 及以上  
  
## 四、修复与应急处置  
### 4.1 首选方案：升级  
  
立即升级至以下安全版本：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">当前分支</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">目标版本</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">下载地址</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">1.6.x</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">1.6.16</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Roundcube 官方仓库</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">1.7.x</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><strong><span leaf="">1.7.1</span></strong></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Roundcube 官方仓库</span></section></td></tr></tbody></table>> ⚠️ **补丁风险提示**  
：本次为安全补丁，升级过程通常无需额外停机时间，但建议在非业务高峰时段执行，并在升级前**完整备份数据库及配置文件**  
。若生产环境使用virtuser_query插件，升级后需验证用户映射逻辑是否正常。  
  
### 4.2 临时规避措施  
  
若短期内无法完成升级，可通过以下任一方式降低风险：  
  
**方式一：禁用 virtuser_query 插件**  
  
编辑 config/config.inc.php  
，定位到插件配置段落，将 virtuser_query 从加载列表中移除：  
```
$config['plugins'] = array(    // 'virtuser_query',  // 注释或删除此行    '其他插件...',);
```  
> ⚠️ 注意：禁用该插件可能导致依赖邮箱地址→数据库账号映射的认证流程失效，请先在测试环境验证对登录功能的影响。  
  
  
**方式二：WAF / IPS 规则拦截**  
  
在 Web 应用防火墙或入侵防御系统中添加规则，拦截包含以下特征的请求（仅供参考，攻击者可变换绕过）：  
- 包含连续反斜杠序列（如 \\\\  
）的请求参数  
  
- 与 virtuser_query 插件端点相关的异常 SQL 片段  
  
> ⚠️ 正则绕过方式多变，WAF 规则仅作辅助手段，**不能替代升级**  
。  
  
## 五、在野利用现状  
  
**一个被沉默了四个月的定时炸弹。**  
- **2026年5月24日**  
：Roundcube 官方发布 1.6.16 与 1.7.1，悄然修复该漏洞，未对外公开披露漏洞细节（属于典型的"静默补丁"策略）。  
  
- **2026年9月下旬**  
：加拿大网络安全中心（CCCS）联合 SOCRadar 发布紧急警报，确认黑客组织正对全球 **50多万台**  
仍未打补丁的 Roundcube 服务器发起大规模自动化扫荡与定向攻击。  
  
- **低门槛，高收益**  
：由于该漏洞为预认证（Pre-Auth）型 SQL 注入，攻击者无需任何账号密码，只要服务器暴露在公网即可直接注入，这让黑客的自动化武器化成本极低——一个 Python 脚本 + IP 段列表，即可批量收割。  
  
- **受害者覆盖**  
：企业邮件系统、政府机构、教育机构均在攻击范围内，通讯录和往来邮件数据已成为主要窃取目标。  
  
## 六、总结与行动建议  
  
CVE-2026-48842 是 Roundcube 近年来风险最高的漏洞之一：预认证、SQL注入、在野利用三重高危属性叠加，无需任何凭证即可直捣数据库。建议所有 Roundcube 管理员：  
1. **立即**  
检查当前运行版本，确认是否在受影响范围内。  
  
1. **优先升级**  
至 1.6.16 / 1.7.1；若暂无法升级，先禁用 virtuser_query 插件。  
  
1. 检查数据库访问日志，排查是否存在异常的 virtuser_query SQL 查询。  
  
1. 如生产环境配置不允许直接升级，需制定分阶段修复计划并缩短观察周期。  
  
**漏洞修复窗口已开启，请勿继续等待。**  
  
**版权声明**  
：本文由华盟网原创发布，保留所有权利。配图由华盟网授权使用。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6OhjxQzQweFNUKUpiaj3ZgQVia6mHk0tJt558k2gs2lbg2GYJc40oxejjhZvrDofMuBy0tib4GG9m3VtxXmaSujh6BcCm6hGPCMu8/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
