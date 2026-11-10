#  一个搜索框拿管理员：Craft CMS未认证SQL注入漏洞复盘，CVSS 9.8全版本在射程  
零日手记
                    零日手记  随笔漫记安全路   2026-11-10 04:38  
  
2026年11月9日，GitHub Advisory Database上出现一条critical级别记录：**Craft CMS全版本未认证SQL注入，CVSS 9.8，可直接提权管理员**  
。影响4.0以来所有版本，补丁9月中旬就已发布，但大量站点至今未更新。  
  
这个漏洞值得拆开看，因为它踩中了一个经典陷阱：**PDO emulated prepares + sprintf拼接SQL = 堆叠注入**  
。  
## 漏洞档案  
<table><thead><tr><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">项目</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">详情</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">CVE</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">CVE-2026-115934</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">GHSA</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">GHSA-qcvp-ggx3-97r6</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">CVSS</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">9.8（AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H）</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">CWE</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">CWE-89 SQL注入</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">影响版本</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">Craft CMS 4.0.0–4.18.8 / 5.0.0–5.11.1</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">补丁版本</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">4.18.9 / 5.11.2（2026-09-17发布）</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">前置条件</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">前端使用了</span><code><span leaf="">craft.search</span></code><span leaf="">搜索功能 + MySQL/MariaDB后端</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">认证要求</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">无需认证</span></strong></td></tr></tbody></table>## 攻击链：从搜索框到管理员  
  
Craft CMS的前台搜索功能通过craft.search  
处理用户输入，搜索词经Search::normalizeSearchQuery()  
解析后，最终流入Search::_sqlWhere()  
方法拼成SQL WHERE子句。  
  
问题出在_sqlWhere()  
——它对列名$key  
做了quoteColumnName()  
转义，但**值$val直接用sprintf拼进SQL字符串**  
：  
```
// 漏洞代码 (src/services/Search.php)private function _sqlWhere(string $key, string $oper, string|int $val): string{    $key = Craft::$app()->getDb()->quoteColumnName($key);    // ↑ $key做了转义    // ↓ $val直接sprintf拼接，没有quoteValue！    return sprintf("(%s %s '%s')", $key, $oper, $val);}
```  
  
$val  
被单引号包裹后直接拼入SQL。攻击者在搜索参数中闭合单引号，就能注入任意SQL。  
  
**关键放大器：PDO emulated prepares**  
。Craft CMS的MySQL连接默认开启PDO::ATTR_EMULATE_PREPARES  
，这意味着PDO在客户端模拟预处理，不在MySQL服务端执行真正的参数化查询。后果是**堆叠查询（stacked queries）可以执行**  
——攻击者用分号串联第二条SQL语句，实现写操作。  
  
攻击者只需在前台搜索URL中构造[attribute]  
参数：  
```
GET /actions/search/search?q=test&[attribute]=1';UPDATE users SET admin=1 WHERE id=1;--
```  
  
三条路径全部走通：  
<table><thead><tr><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">步骤</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">操作</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">1</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">闭合单引号+分号</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">脱离原查询上下文</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">堆叠</span><code><span leaf="">UPDATE users SET admin=1 WHERE id=1</span></code></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">user ID 1的admin标志位置1</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">3</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">Craft CMS权限检查短路</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><strong><span leaf="">admin列=1即跳过所有权限校验</span></strong><section><span leaf="">，无需改密码</span></section></td></tr></tbody></table>  
同一个写原语还能覆盖任意账户的密码哈希。报告者原话："The write primitive was used to set the admin flag on user ID 1 in the users table, confirmed by reading the value back."  
## 补丁：三处修复  
  
补丁commit 8e3023c55e  
改了三个文件：  
  
**修复1：_sqlWhere()对值做quoteValue()**  
```
// 修复后private function _sqlWhere(string $key, string $oper, string|int $val): string{    $db = Craft::$app()->getDb();    $key = $db->quoteColumnName($key);    $val = $db->quoteValue($val);  // ← 新增：值转义    return "($key $oper $val)";    // ← 不再手动包单引号}
```  
  
核心变化：$val  
从sprintf裸拼接改为quoteValue()  
转义，同时去掉外层手动单引号（quoteValue()  
自带引号包裹）。  
  
**修复2：SearchQuery构造函数白名单defaultTermOptions**  
```
// 修复后：只允许这4个keyprivate const ALLOWED_DEFAULT_TERM_OPTIONS = ['subLeft', 'subRight', 'exclude', 'exact'];$this->_defaultTermOptions = array_intersect_key(    $defaultTermOptions,    array_flip(self::ALLOWED_DEFAULT_TERM_OPTIONS),) + [...defaults...];
```  
  
防止攻击者通过defaultTermOptions  
注入非预期参数。  
  
**修复3：ElementHelper::cleanseQueryCriteria()校验search类型**  
```
// 修复后：search必须是字符串if (isset($criteria['search']) && !is_string($criteria['search'])) {    unset($criteria['search']);}
```  
  
防止数组类型绕过。  
## 时间线  
<table><thead><tr><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">日期</span></section></th><th style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top; font-weight: bold; background-color: rgb(240, 240, 240);"><section><span leaf="">事件</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-09-10</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">研究者通过Hackrate平台提交漏洞</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-09-11</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">Craft团队确认接受，创建GHSA</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-09-15</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">修复commit提交（8e3023c55e）</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-09-17</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">补丁版本4.18.9 / 5.11.2发布</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-10-19</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">安全公告公开</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-11-09</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">严重级别从High升级Critical（CVSS 9.8）</span></section></td></tr><tr><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">2026-11-10</span></section></td><td style="font-size: 0.75em; padding: 9px 12px; line-height: 22px; color: rgb(34, 34, 34); border: 1px solid rgb(216, 216, 216); vertical-align: top;"><section><span leaf="">GitHub Advisory Database收录</span></section></td></tr></tbody></table>## 临时缓解  
  
无法立即升级的站点，在Web服务器层拦截含[attribute]  
或%5Battribute%5D  
的查询字符串：  
```
# Nginxif ($query_string ~* "(%5Battribute%5D|\[attribute\])") {    return 403;}
```  
```
# ApacheRewriteEngine OnRewriteCond %{QUERY_STRING} (.*(\[attribute\]|%5Battribute%5D).*)RewriteRule .* - [F]
```  
  
注意：这只是止血，根本方案是升级到4.18.9+/5.11.2+。  
## 两个教训  
  
**教训一：sprintf拼SQL永远是隐患。**  
 这个函数对列名做了quoteColumnName()  
却漏了值的quoteValue()  
——典型的"修了一半"。PHP生态里sprintf  
+SQL的组合是高危模式，即使你手动包了单引号，一旦值内含单引号就绕过。正确做法：要么全程参数化查询，要么对所有动态部分调用对应的quote方法。  
  
**教训二：PDO emulated prepares是堆叠注入的温床。**  
 当PDO::ATTR_EMULATE_PREPARES=true  
（MySQL默认），PDO在客户端拼接SQL字符串再整体发送给服务端，分号分隔的多条语句可以一起执行。真正的服务端预处理不支持堆叠查询。Craft CMS + MySQL的组合默认就开着这个选项，把一个本应只能读取的注入点升级成了读写原语。  
  
检查你的PHP应用：  
```
// 关闭emulated prepares（推荐）$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```  
  
这会让PDO使用真正的服务端预处理，堆叠查询将被拒绝。注意少数旧代码可能依赖emulated prepares的行为，切换前做回归测试。  
  
参考：  
- GitHub Advisory: GHSA-qcvp-ggx3-97r6 / CVE-2026-115934  
  
- 补丁commit: craftcms/cms@8e3023c55e  
  
- 漏洞报告: Hackrate HCKRT-Y64NY8  
  
- 补丁版本: 4.18.9 / 5.11.2  
  
  
  
