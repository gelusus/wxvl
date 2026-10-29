#  别再人工刷漏洞库了 CNNVD 官方 API 这样接 php2shell 当天就进台账  
宝十八
                    宝十八  网络安全老宋   2026-10-29 04:00  
  
**导语：**  
 你好，我是网络安全老宋。  
安全攻防干货准时送达！  
  
  
网络安全  
老宋  
// 实战教程 · 漏洞情报  
  
// 实战教程 · 漏洞情报  
# 别再人工刷漏洞库了CNNVD 官方 API 这样接php2shell 当天就进台账  
  
上周 php2shell（PHP 高危 RCE）多国紧急预警。这篇教你用 CNNVD 官方 OpenAPI，把漏洞情报工程化拉取并接进资产台账，新漏洞一入库，命中的资产当天就弹预警。  
  
CNNVD API  
漏洞情报  
php2shell  
  
●一句话精华  
  
漏洞情报的第一落点应该是你自己的资产台账，而不是公众号——把 CNNVD 官方 OpenAPI 接进来，新漏洞一入库，资产先知道。  
  
目录 · Table of Contents  
  
01  
两个库别搞混：CNNVD 和 CNVD  
  
02  
为什么用官方 API 而不是爬页面  
  
03  
前置：实名认证与 AppID  
  
04  
签名机制：四头加一把 SHA256 锁  
  
05  
可运行代码：从请求到 XML 链接  
  
06  
工程化：把情报搬进资产台账  
  
07  
本周实战与合规收尾  
  
上周有个事挺扎心。10 月 21 日、22 日，国家信息安全漏洞共享平台连发两道公告，PHP 爆出代号「php2shell」的高危远程代码执行漏洞，编号 CVE-2026-109618 和 CVE-2026-109619，对应 CNVD-2026-41674、CNVD-2026-41675。多国机构当天拉响警报，说这漏洞正在被广泛利用，影响 PHP 8.2 到 8.5 一堆版本，底下还拖着 WordPress、Drupal、Laravel 这些大家都在用的框架。  
  
老宋问一句：你单位是哪天知道这个漏洞的？刷新闻看到的？等客户来问才查的？还是根本还没动静？  
  
说真的，漏洞情报这东西，靠人肉刷页面，永远慢半拍。这一篇老宋不教你怎么修 PHP（那个官方公告写得很清楚），而是教你一个更根本的事：怎么用 CNNVD 的官方 API，把漏洞情报自动「搬」进你自己的资产台账  
。这样新漏洞一入库，命中的资产当天就弹预警，不用你天天盯着网页。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flibdIAo7U6JNvSpswG8VxyNraJjiauwXrVdKQYPhQKrHgulXlxynHqhaAYGcnFqMibDa1w5FaQjmGssr5F92bnibtoibnJBJXRicXicyw/640?wx_fmt=png&from=appmsg "")  
  
▲ 国家漏洞库经官方 API 流入你的资产台账，新漏洞一入库，资产先知道  
  
## 01两个库别搞混：CNNVD 和 CNVD  
  
  
动手之前先正本清源，这俩名字太像，搞混了后面全白干。  
  
CNNVD，全称国家信息安全漏洞库，由中国信息安全测评中心运营，是国内最权威的中文漏洞库之一。它的特点是中文描述细、厂商和产品字段全，还提供「增强版 XML」，里面连补丁链接、参考链接都给你备齐了。  
  
CNVD，全称国家信息安全漏洞共享平台，由 CNCERT 运营。本周那个 php2shell，就是 CNVD 先发的通告。  
  
两个库的编号体系不一样：CNNVD 是 CNNVD-2026-xxxxx，CNVD 是 CNVD-2026-xxxxx，别拿混了。不过 CNNVD 也会同步收录同类漏洞。本篇我们用的，是 CNNVD 对外开放的 OpenAPI 接口——选它，主要图三件事：中文语境、字段最全、适合做资产关联  
。  
  
## 02为什么要用官方 API，而不是爬页面  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flibhGdUA0pia5GFrCpOgtE91xg71iceeSPQE3rfEPBsyO3ARIaQj4XHWicOXu7Tib7PiaicYW6708O0JEToyJ8WxPnicLQB8nIOejeyUUs/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93flicO6iaCwtCTMASedHqHx1UBng65XIL8fxN6qC0UOkMyl0sewMhvUUGBZ4oxuujWV8468Yy2yXoicIAEY5EMxIxHLvR1lltLleqss/640?wx_fmt=png&from=appmsg "")  
  
有的兄弟图省事，直接写个脚本爬 CNNVD 网页。老宋劝你别这么干，坑不少。  
  
第一，反爬和登录态。漏洞详情页很多要登录才能看全，爬虫一上去就被限流、被封。  
  
第二，字段不全。网页上展示的字段，和接口返回的增强版 XML 根本不是一个量级，你爬到的可能是阉割版。  
  
第三，合规风险。批量抓取网站，搞不好就踩了人家网站的使用条款，单位用尤其要慎重。  
  
那第三方聚合平台呢？延迟、字段被裁剪、收费、还不可控，真出了事你连数据源都追不回来。  
  
官方 API 就不一样：实名单位申请后字段全、可溯源、稳定  
，还支持按时间窗口增量拉取——你今天只拉昨天新增的，不用每次全量轰。前提是，你得先完成单位实名认证  
。个人用户只能查和报送，API 接口是单位实名认证之后才解锁的权益，这点后面细说。  
  
## 03前置条件：实名认证与 AppID  
  
  
先把门槛说清楚，免得你兴冲冲去申请发现没资格。  
  
个人用户注册后，权益包括：获取信息安全资讯、海量漏洞信息查询、报送漏洞拿提交证明，实名认证后还能查「受漏洞影响产品信息」。但漏洞信息获取 API 接口  
，是单位用户完成实名认证之后的专属权益，同时解锁的还有受漏洞影响产品信息、多维度合作资格，以及兼容性、技术支撑、厂商合作等背书。  
  
申请路径不复杂：登录 CNNVD 官网，进「单位中心」，找「账号安全」里的「OpenAPI 密钥管理」，申请生成 API 密钥，你会拿到两个东西——APP_ID  
   
和  
 APP_SECRET  
。  
  
老宋重点提醒一句：APP_SECRET 是签名用的根密钥，千万别硬编码进代码仓库，更别截图发群里  
。建议放环境变量或者你们单位的密钥管理服务，出问题的是密钥泄露，不是代码。  
  
## 04签名机制：四个请求头加一把 SHA256 锁  
  
  
拿到密钥，调用之前得先搞懂它的签名。接口要求你在请求头里带四个东西：  
  
X-Appid 是你的 APP_ID；X-Timestamp 是当前毫秒时间戳；X-Nonce 是一个随机串，专门防重放；X-Sign 是签名本身。  
  
签名的算法很简单，但设计得挺讲究：把「请求方法 + 接口URI + 时间戳 + 随机串 + APP_SECRET」拼成一行字符串，做一次 SHA256，结果就是 X-Sign。  
  
为什么这么设计？方法加 URI，防止有人篡改你要调的接口；时间戳加随机串，配合有效期，防止别人把你的请求拦下来重放一遍。而最关键的 APP_SECRET，永远只参与签名，不从网络里走  
。换句话说，就算有人截获了你这一次请求，没有 secret 他也造不出下一个合法签名。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/yJLbez93flic8Fr0ERvog5ddamnGxOeibohWYcYgk9gehLTBylgymial12OpsYicD1l3Vv3KRnLJZziclo7ktSFm0ia5z6So3OywThZaziaDRu1fAU/640?wx_fmt=png&from=appmsg "")  
  
▲ 3D 等距视角，左侧四张叠放的 HTTP 请求头卡片，右侧一个发光的橙色挂锁代表 SHA256 签名锁，橙色箭头连接，浅色背景，科技感  
  
## 05可运行代码：从请求到拿到 XML 链接  
  
  
光说不练没用，老宋给你一段能直接跑的 Python。核心就是拼签名、组请求头、发 POST、拿返回。  
  
⏰ Shell  
  
# CNNVD 官方 API 拉取漏洞 XML 信息（单位实名认证用户）  
  
# 需自行替换 APP_ID 和 APP_SECRET，建议从环境变量读取  
  
import hashlib, json, random, string, time, requests  
  
  
HOST = "https://www.cnnvd.org.cn"  
  
URI  = "/cnnvdweb/2.0/openapi/vul/xml"  
  
URL  = HOST + URI  
  
  
APP_ID     = "替换为你的APP_ID"  
  
APP_SECRET = "替换为你的APP_SECRET"   # 别写死，用环境变量或密钥管理服务  
  
  
# 1. 构造认证参数  
  
method    = "POST"  
  
timestamp = str(int(time.time() * 1000))  
  
nonce     = "".join(random.choices(string.ascii_letters + string.digits, k=16))  
  
  
# 2. 计算签名：SHA256(method + URI + timestamp + nonce + secret)  
  
sign_str = f"{method}{URI}{timestamp}{nonce}{APP_SECRET}"  
  
x_sign   = hashlib.sha256(sign_str.encode("utf-8")).hexdigest()  
  
  
# 3. 组装请求头与请求体（按发布时间窗口增量拉取）  
  
headers = {"Content-Type": "application/json",  
  
"X-Appid": APP_ID, "X-Timestamp": timestamp,  
  
"X-Nonce": nonce, "X-Sign": x_sign}  
  
payload = {"pageNum": 1, "pageSize": 50,  
  
"publishStartDate": "2026-10-21",  
  
"publishEndDate": "2026-10-23"}  
  
  
try:  
  
r = requests.post(URL, json=payload, headers=headers, timeout=10)  
  
data = r.json()  
  
for rec in data["data"]["records"]:  
  
# xmlContent 是 XML 下载链接，有效期仅 10 分钟，拿到尽快下载  
  
print(rec["vulName"], "->", rec["xmlContent"])  
  
except Exception as e:  
  
print("请求出错:", e)  
  
返回结构里，data.total 是命中总数，data.records 是漏洞列表，每条都有 vulName、cve_id（在 XML 里）、危害等级，以及最关键的  
 xmlContent  
——那是 XML 详情文件的下载链接。注意，这个链接有效期只有 10 分钟  
，你拿到就得赶紧下，别等下午再想起来。  
  
## 06工程化：把情报搬进资产台账  
  
  
到这一步，你只是「能拉」了。真正的价值在「拉完之后」。老宋给你一条最小可用的工程化链路：  
  
第一步，增量拉取。别每次全量，用 publishStartDate 和 publishEndDate 每天拉昨天的增量，cron 定时跑，稳。  
  
第二步，XML 解析入库。把每条漏洞的 CNNVD 编号、CVE 编号、厂商、产品、漏洞描述、技术危害等级、综合危害等级、补丁链接、参考链接，落进你们自己的漏洞库表（MySQL 就行）。  
  
第三步，也是最值钱的一步——资产关联匹配  
。把你解析出来的厂商和产品，和 CMDB 里的资产清单做比对。比如 XML 里出现「Drupal / SAML SSO」，你 CMDB 里刚好有十台跑 Drupal 的 Web 服务器，那就自动命中，生成一张预警工单。  
  
第四步，接进 SOC 或者你们的漏洞管理平台。和一键处置、告警系统打通，形成「情报进来 → 资产命中 → 工单生成 → 处置闭环」的完整回路。到这，漏洞情报才真正从「看新闻」变成了「管资产」。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/yJLbez93flibTfAcFbVKmdl3FTgwfJ5rkMECRdyRsAvmh7GYjN02JZmHNLSZgxYR70NXuoGQe6Zay3h0culrLfbnGOTFyKRDzBlBD3tSV664/640?wx_fmt=png&from=appmsg "")  
  
▲ 深蓝技术线框蓝图风格，漏洞情报工程化链路从左到右：定时拉取齿轮、XML 解析节点、漏洞入库圆柱、资产匹配节点、SOC 预警铃铛，青白线条连接，深蓝背景  
  
## 07本周实战与合规收尾  
  
  
回到开头那个 php2shell。用上面这套流程，你在 10 月 23 日早上拉一下 10 月 21 日到 23 日的窗口，CNNVD 同步收录的那几条 PHP 漏洞就进你库了。再一匹配 CMDB，凡是跑了 PHP 8.3 低于 8.3.36、或者 WordPress、Drupal、Laravel 的资产，当天就能弹出预警：「贵单位 X 台 Web 服务器命中 CNNVD 新收 PHP 高危 RCE，请 48 小时内升级至 PHP 8.5.12、8.4.27、8.3.36 或 8.2.35」。  
  
这比人工刷页面快在哪？自动化当天入库，人肉可能一两天天后才刷到，而这  
一两  
天，漏洞的 PoC 已经在网上传开了。处置建议直接引用官方：升级到修复版本；暂不能升的，用 WAF 拦异常 multipart 请求、调 pm.max_requests 这些临时缓解。  
  
最后几点合规提醒，老宋一并交代：xmlContent 链接 10 分钟有效期，下载要快  
；接口有频率限制，按官方节奏走增量，别狂刷；密钥保管好，别进 git；数据只用于本单位合规预警和资产处置，守好 CNNVD 的使用边界。也别把 CNNVD 当唯一源，和 CISA KEV、CNVD、NVD 交叉比对，覆盖才全。  
  
// 老宋说  
  
把漏洞情报从「看新闻」升级成「拉接口」，听起来是个技术动作，本质是安全运营从被动到主动的一小步。本周 php2shell 这种「PoC 已广泛传播」的高危漏洞，留给甲方的反应时间是以小时计的。你单位要是还在靠人肉刷页面，等看到的时候资产可能已经被扫了。实名认证、申请 API、接进台账——这三步今晚就能立项，比等任何一条提醒都实在。  
  
## 读推荐阅读  
  
1. 本次分析的原文章：CNNVD 通过官方 API 接口获取漏洞信息（baby神 @ 51CTO）  
  
https://blog.51cto.com/babyshen/15036775  
  
2. CNNVD 漏洞 XML 信息获取接口文档（官方，2026-06-24 更新）  
  
https://www.cnnvd.org.cn/frontend/specDocDownload  
  
3. CNNVD 官网：漏洞通报、热点漏洞与文档规范下载  
  
https://www.cnnvd.org.cn/  
  
4. 国家信息安全漏洞共享平台（CNVD）关于 PHP 远程代码执行漏洞的安全公告（php2shell）  
  
https://www.cert.org.cn/publish/main/9/2026/20261022162132536309183/20261022162132536309183_.html  
  
5. 界面新闻：PHP 高危漏洞或冲击全球七成网站，多国机构紧急预警  
  
https://new.qq.com/rain/a/20261022A0DMWH00  
网络安全老宋 · 转载请注明出处  
防御，不是在演练期间发现攻击，而是在演练开始前就把攻击面收敛到最小。  
  
end  
  
  
  
不想错过文章内容？读完请点一下**“在看**  
**”**  
，加个**“****关注”**  
，您的支持  
是我创作的动力  
  
期待您的一键三连支持（点赞、在看、分享~）  
  
