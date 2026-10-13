#  Web漏洞挖掘实战宝典：40 个必知技巧，全是看得到赏金的漏洞实操思路！  
 沧海讲安全   2026-10-13 09:08  
  
从事网络安全工作，本质上是在与各类潜在风险持续博弈，稍有不慎就可能引发安全事件。  
  
Web漏洞的产生，根源往往是系统对输入数据、权限边界和交互流程的信任校验不足。  
  
开发过程中，如果对用户输入、权限控制和系统交互的校验不够严格，就很容易留下安全隐患。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9SGuSiaPo8E4ibvZeScVpZF1V8JdMtTBd6km8CkibX15iaq14uUdVTX9Z23WENpVKBBA71DfIuSfqOl9GYlxLNoVy3FqVOVzjhuibPo/640?wx_fmt=jpeg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
常见的风险主要集中在以下三类：  
  
**大门敞开，谁都能逛**  
：  
用户输入没做好过滤，SQL注入、XSS啥的，直接把恶意代码当成正常指令执行了，这不是搞笑吗？  
  
**自家院子变成公共广场：身份验证和会话管理乱成一团，越权访问、JWT篡改，黑客直接大摇大摆进你家院子闲逛，想干啥干啥！**  
  
**逻辑鬼才写的代码**  
：业务规则校验形同虚设，订单金额随便改，短信接口被刷到爆，这不就是等着被人薅羊毛吗？  
  
那挖洞的正确打开方式到底是啥呢？  
  
默认全员“坏人”，一个都别信  
：  
把每个输入都当成来搞破坏的，盯着系统怎么接招（比如随手丢个 <  
 进去，看会不会直接触发 XSS）。  
  
**顺藤摸瓜，盯死数据全程**  
：  
数据跑过参数、Cookie、Headers 这些不同位置时，处理逻辑往往不一样，得挨个扒清楚，别漏过任何一处。  
  
**专挑系统难受的招使**  
：  
上高并发刷请求、搞中间人劫持这类非常规操作，专门测系统的抗压底线，竞争条件、协议级漏洞这些幺蛾子，全靠这么逼出来。  
  
下面是精心炮制的  
40个Web常见漏洞挖掘技巧  
，拿走不谢！  
  
01  
  
注入攻击，无孔不入（5个）  
  
1. SQL注入：操控数据库查询  
  
**原理**  
：输入未过滤直接拼接入SQL语句，可任意操控数据库。  
**检测**  
：输入  
' OR 1=1#  
 看是否返回异常数据，或用SQLMap自动扫描。  
**技巧**  
：除GET/POST参数外，Cookie、HTTP头也可能是注入点，可用联合查询获取库名、表名。  
### 2. NoSQL注入：JSON查询漏洞  
  
**原理**  
：针对MongoDB等JSON查询数据库，利用参数解析缺陷绕过校验。  
**检测**  
：提交   
username [$ne] = 1&password [$ne] =1  
，测试能否免密登录。  
**技巧**  
：检查JSON参数是否未经校验直接作为查询条件。  
### 3. 命令注入：操控服务器指令  
  
**原理**  
：输入直接拼接入系统命令，可让服务器执行任意操作。  
**检测**  
：输入  
; ls  
或  
| dir  
，查看是否返回目录列表。  
**技巧**  
：可用分号、管道符等特殊字符尝试绕过过滤。  
### 4. XXE：XML外部实体漏洞  
  
**原理**  
：解析XML时加载恶意外部实体，导致敏感文件泄露。  
**检测**  
：上传含   
<!ENTITY xxe SYSTEM "file:///etc/passwd">  
 的XML，测试能否读取服务器文件。  
**技巧**  
：检查文件上传、API接口是否支持XML且未禁用外部实体解析。  
### 5. LDAP注入：认证查询漏洞  
  
**原理**  
：输入拼接入LDAP查询语句，可绕过身份认证。  
**检测**  
：输入  
*)(uid=*))(|(uid=*  
，测试能否直接登录。  
**技巧**  
：重点关注登录框、搜索功能的特殊字符过滤是否到位。  
  
02  
  
身份验证：门神睡着了（6个）  
  
6. 弱口令：易猜测的密码风险  
  
**检测**  
：使用字典爆破，测试是否存在 admin/123456  
 这类常见弱密码。  
**技巧**  
：结合社会工程学构建专属字典（如姓名+生日组合），提升爆破成功率。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9QRuJz9fekyicb946A93KpEfDnEE0z0pyqvr1chSgvKdAQWlKXbFfDDCzkO8eboNb5Z1OD5S536kEBgbRhLR0J6nHicUnQ7R9Its/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "")  
### 7. JWT：令牌校验漏洞  
  
**原理**  
：签名算法未严格校验，或密钥硬编码在代码中。  
**检测**  
：用   
jwt_tool  
 伪造令牌，测试是否支持   
none  
 算法（即无需签名即可通过校验）。  
**技巧**  
：将Header中   
alg  
 字段改为   
none  
，尝试绕过签名验证。  
### 8. OAuth 2.0：授权流程风险  
  
**高危点**  
：  
redirect_uri  
未严格校验，  
state  
 参数缺失。  
**风险**  
：攻击者可构造钓鱼链接，骗取用户授权码。  
**技巧**  
：检查回调域名白名单是否过于宽松，是否存在可绕过的情况。  
### 9. 会话固定：Session ID被预设  
  
**原理**  
：攻击者预先指定Session ID，用户登录后沿用该ID，导致会话被劫持。  
**检测**  
：对比登录前后Session ID是否变更，未变更则存在风险。  
**技巧**  
：测试是否可通过URL参数传递Session ID（如   
?PHPSESSID=123  
）。  
### 10. 短信验证码：接口滥用风险  
  
**检测**  
：抓包重放验证码请求，测试是否存在无限制发送短信的漏洞。  
**技巧**  
：用Burp Intruder自动化测试，验证验证码是否为短位数字（如4位），是否存在爆破风险。  
### 11. 密码重置：逻辑绕过风险  
  
**常见问题**  
：验证码直接回显在页面、重置Token未失效、用户ID可随意修改。  
**案例**  
：修改响应包中   
is_valid  
 字段，即可绕过短信验证步骤。  
  
**03**  
  
客户端：前端代码不靠谱（6个）  
  
12. XSS：前端脚本注入  
  
**类型**  
：反射型、存储型、DOM型。  
**检测**  
：输入   
<script>alert(1)</script>  
，查看输出是否被转义。  
**技巧**  
：尝试用   
onmouseover  
 等事件处理函数绕过过滤。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9S6zSLTW5qhvbrQ7dZGYDhTf1cs3dKFosjFI2BcxZRVvl4RlE31kumiaUESiaYGKribaTth4iaC842LdPuUMWaWItLr5qtOibZSawZM/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "")  
### 13. CSRF：跨站请求伪造  
  
**检测**  
：检查关键请求是否缺少Token或Referer校验。  
**技巧**  
：构造恶意页面自动提交表单（如转账请求），冒充用户操作。  
### 14. 点击劫持：诱导误触  
  
**原理**  
：利用透明iframe覆盖诱导用户点击。  
**检测**  
：检查是否配置  
X-Frame-Options  
 响应头。  
**技巧**  
：通过CSS   
opacity:0  
 隐藏恶意元素实现诱导。  
### 15. CORS：跨域配置风险  
  
**高危配置**  
：  
Access-Control-Allow-Origin: *  
 允许任意域名跨域访问。  
**检测**  
：发送带   
Origin:恶意域名  
 的请求，检查响应头是否放行。  
**风险**  
：配置不当可导致用户敏感数据被窃取。  
### 16. 不安全重定向：跳转至恶意站点  
  
**检测**  
：将   
redirect_url  
 参数修改为外部域名测试。  
**技巧**  
：尝试用URL编码（如   
%2e%2e%2f  
 代替  
../  
）绕过过滤。  
### 17. DOM型漏洞：前端逻辑缺陷  
  
**原理**  
：客户端脚本未过滤输入直接操作DOM。  
**检测**  
：在URL片段中输入  
#<img src=x onerror=alert(1)>  
 测试。  
  
**04**  
  
服务端：后端的秘密（7个）  
  
18. 文件上传：恶意文件上传  
  
**绕过方式**  
：修改  
Content-Type  
，使用双扩展名（如  
shell.php.jpg  
）绕过检测。  
**技巧**  
：利用Web服务器解析漏洞（如  
test.php.xxx  
）执行上传文件。  
### 19. 任意文件读取：服务器文件泄露  
  
**检测**  
：尝试读取  
/etc/passwd  
或项目配置文件。  
**技巧**  
：利用  
../  
目录遍历读取敏感文件（如  
file=../../etc/passwd  
）。  
### 20. SSRF：服务器端请求伪造  
  
**原理**  
：利用服务端向内部网络发起请求。  
**检测**  
：访问云元数据地址（如  
http://169.254.169.254  
）测试。  
**技巧**  
：通过DNS Rebinding绕过IP白名单限制。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9SgP4w7mEyz12vicUTeRJCgDpeF7ozkVRhia7iaibibuK5krEsibTy3DbsmwatYGgpu6OmEKoFicmnK5usAtx7sCT6skfib5XQic4xZ4usY/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3 "")  
### 21. 反序列化：代码执行风险  
  
**常见场景**  
：Java、PHP、Python的反序列化函数。  
**检测**  
：提交恶意序列化数据触发远程代码执行。  
**技巧**  
：借助ysoserial等工具生成利用Payload。  
### 22. 未授权访问：免登录直入后台  
  
**案例**  
：直接访问管理接口（如  
/admin  
）无需认证。  
**技巧**  
：扫描常见后台路径（如  
phpMyAdmin  
）探测暴露面。  
### 23. 越权访问：权限越界操作  
  
**类型**  
：水平越权（同权限用户间互访）、垂直越权（低权限访问高权限功能）。  
**检测**  
：修改请求中的用户ID参数（如user_id=123  
→user_id=456  
）测试。  
### 24. 敏感信息泄露：敏感数据暴露  
  
**来源**  
：错误页面堆栈信息外泄、备份文件（如  
.bak  
）未清理。  
**技巧**  
：利用目录扫描工具探测敏感文件与配置文件。  
  
**05**  
  
配置与协议：细节决定成败（6个）  
  
25. 不安全HTTP方法：多余接口风险  
  
**检测**  
：发送OPTIONS请求，查看服务器支持的HTTP方法。  
**修复**  
：禁用WebDAV及非必要的HTTP方法，减少攻击面。  
### 26. 目录遍历与浏览：目录结构暴露  
  
**检测**  
：访问  
/static/../  
等路径，测试是否返回目录列表。  
**技巧**  
：利用中间件配置缺陷（如Apache默认开启目录浏览）获取文件信息。  
### 27. HTTP响应头注入：篡改响应内容  
  
**原理**  
：用户输入未过滤直接写入HTTP响应头。  
**检测**  
：输入  
Set-Cookie:恶意内容  
，测试能否篡改响应头信息。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9TDiaYyrNqFfrATVHtGXibFBBhTicHDgcVhUvPgnDVvyiclkm5yEebETuLSj8aYnrcBKpB3DaH0sNfG31XtAazjib3SXmEUeRjZLjDQ/640?wx_fmt=jpeg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "")  
### 28. 过时SSL/TLS协议：加密配置缺陷  
  
**检测**  
：使用SSL Labs等工具检测服务器支持的协议版本与加密套件。  
**风险**  
：启用SSLv3或弱加密套件，易遭受中间人攻击。  
### 29. 主机头注入：Host头伪造风险  
  
**原理**  
：伪造Host头访问内部服务或绕过访问控制。  
**检测**  
：修改Host为  
localhost  
或内部IP，测试访问权限。  
**技巧**  
：利用域名绑定规则绕过原有访问限制。  
### 30. 缓存投毒：缓存内容污染  
  
**原理**  
：操纵缓存服务器存储恶意响应内容。  
**检测**  
：注入  
X-Forwarded-Host  
等恶意请求头，测试缓存是否被污染。  
**技巧**  
：利用缓存键与响应内容分离的设计缺陷实现投毒。  
  
**06**  
  
业务逻辑：程序员的脑袋（5个）  
  
31. 短信轰炸：接口滥用骚扰  
  
**检测**  
：重复请求短信接口，测试是否触发无限制发送。  
**技巧**  
：更换IP或修改手机号末位，绕过频率限制。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CsKJlMFPH9SNHWOURG9oySpORvMTG1f4BOh1WibGjeBjLal3rqTRmHtgs9x5fwjaxIS7T6Ztjc8box15em6h6ib6taQ6R7AC8XOSbUUrVRTeA/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5 "")  
### 32. 订单金额篡改：价格校验缺失  
  
**原理**  
：仅前端校验价格，后端未做二次验证。  
**检测**  
：修改POST请求中的  
price  
字段，测试能否以低价下单。  
**技巧**  
：尝试提交负数或极低金额。  
### 33. 验证码：校验机制失效  
  
**类型**  
：验证码复用、前端生成、未与会话绑定。  
**案例**  
：响应包直接返回验证码明文，导致校验失效。  
### 34. 接口参数污染：多值参数冲突  
  
**原理**  
：重复参数覆盖业务逻辑（如  
user_id=1&user_id=2  
）。  
**技巧**  
：测试后端对多值参数的处理逻辑，寻找逻辑漏洞。  
### 35. 时间竞争：并发逻辑缺陷  
  
**案例**  
：并发请求兑换积分，导致超额领取。  
**检测**  
：使用多线程工具模拟高并发操作，测试是否存在竞态条件。  
  
**07**  
  
其他高危漏洞：防不胜防（5个）  
  
36. WebSocket：跨站劫持风险  
  
**风险**  
：未校验Origin头，可导致跨站WebSocket劫持。  
**检测**  
：伪造Origin头发起恶意连接测试。  
### 37. 服务端模板注入（SSTI）  
  
**常见框架**  
：Jinja2、Freemarker。  
**检测**  
：输入  
{  
{7*7}}  
，观察是否返回  
49  
。  
**技巧**  
：利用Payload尝试执行系统命令。  
### 38. HTTP请求走私：前后端解析差异  
  
**原理**  
：利用前后端对请求解析的差异构造恶意请求。  
**检测**  
：混淆Content-Length  
与Transfer-Encoding  
字段发送测试请求。  
### 39. 子域名接管：闲置域名风险  
  
**场景**  
：过期域名未解除DNS解析，仍指向第三方服务。  
**技巧**  
：扫描CNAME记录指向已失效的云服务（如GitHub Pages）进行接管。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/CsKJlMFPH9RGVAeXNGp38efCsDpbaYSTOv5xRPMRyYpOz26AJ9XFwqeo0u4cpYLOIHSf1TAGh1lXyAEvWG6ia1wQQzEapickjBSwjfDEoa3wk/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "")  
  
子域名接管完整流程示意图  
### 40. Web缓存欺骗：缓存内容篡改  
  
**原理**  
：诱导服务器缓存恶意内容，污染正常响应。  
**检测**  
：构造类似  
http://target.com/profile.php/non-existent.css  
的路径测试缓存行为。  
**技巧**  
：利用静态资源缓存规则差异实现欺骗。  
  
记住，实际操作的时候，自动化工具（比如Burp   
Suite  
、SQLMap）和手动测试都要用上，而且一定要**遵守法律法规**  
，拿到授权才能搞！别把自己搞进去了！  
  
「最后」如果你真的想学好一门本事，首先就要考虑自己对这门技术的兴趣，没有天赋还能靠时间和努力去弥补，但如果没有兴趣加持，就很难坚持到最后。你要是正打算尝试网安或者想努力一次，我把这些年用过的视频教程和学习笔记都梳理出来了，现在都无偿分享给大家，需要的找我拿就行（文末自取）。现在哪个行业都不好走，如果没有学历也没有天赋，那就只有努力和坚持了，请相信相信的力量，共勉！  
  
**沧海专属黑客/网络攻防技术资料**  
  
  
  
@沧海讲安全：  
在安全圈待了十多年，已经积累了很多的技术教程，在计算机这个行业，如果不会主动学习，手里没点学习资料，注定是走不远的。我整理的这些资料包含了市场上主流的攻防技术，不说让你成为黑客大佬，帮助你从0到进阶网络安全技术问题不大。  
  
  
**平台铭感，拿资料、学技术看⬇（无偿共享）**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCW28TYVbicW1icR88lb2fLYfLS6ib2Mfic96c3gX0VBFarDLjM2sjicYFE6SVtcyF5DHLPwyUgE4lyzDxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
**部分技术资料预览**  
  
  
**01**  
  
**视频教程**  
  
  
从0到进阶主流攻防技术视频教程（包含红蓝对抗、CTF、HW等技术点）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcYaRKqWc1cxP8sBrX6KZasFTJEVibWmdyoGAuRO4AbzaVjUJ8guoWAzQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcCP8oaOCQm8Cp2qhpCxWiaOjzYrOoA1iac5eSafBicPxSQcpYtchyfVvxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****2**  
  
**书籍Pdf**  
  
入门必看攻防技术书籍pdf（书面上的技术书籍确实太多了，这些是我精选出来的）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcUumOTUmUznuo7MzKl1JiaEQIeSh4ibkO6jxY68zVZz7iayrwGRtGu2bHw/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****3**  
  
安装包/源码  
  
  
主要攻防会涉及到的工具安装包和项目源码（防止你看到这连基础的工具都还没有）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6KqcvsT9h4B1hS9VEPengMcOtNL24949kb4cibKLS9HkIb1k2htW8GYqzMQ/640?wx_fmt=jpeg&from=appmsg "")  
  
  
**0****4**  
  
**面试试题/经验**  
  
  
网络安全岗位面试经验总结（谁学技术不是为了赚$呢，找个好的岗位很重要）  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCVGRW5LEnoxPqDicoM3g6Kqcm6J0eAql29R6DIM8bJW4rweVBicM8ibGMOmLNFTpdcQ0gFvefMTOg9dA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
**平台铭感，拿资料、学技术看⬇（无偿共享）**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/kWXbooRKsCW28TYVbicW1icR88lb2fLYfLS6ib2Mfic96c3gX0VBFarDLjM2sjicYFE6SVtcyF5DHLPwyUgE4lyzDxA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
@沧海讲安全：  
只要你是真心想学黑客/网络安全技术，我这份资料就可以无偿共享给你学习，但是想学技术去乱搞的人别来找我，目前全球网络环境日益紧张，我国在这方面的相关人才比较紧缺，网络安全行业确实也需要更多的有志之士加入进来，我也真心希望帮助大家学好这门技术，如果日后有啥学习上的问题，欢迎找我交流。  
  
