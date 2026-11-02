#  【安全圈】AuthMask高危漏洞曝光：恶意App冒充Teams窃取M365令牌  
 安全圈   2026-11-02 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：网络安全机构 Enclave.ai 正式披露代号为 **AuthMask**  
 的严重安全漏洞（**CVE-2026-80097**  
）。该漏洞根植于 Android 版 **Microsoft Authenticator（微软身份验证器）**  
的 SSO 认证代理组件中。任何无须任何特殊系统权限的普通第三方恶意应用，仅通过向验证器发送特定 IPC 请求，即可伪装成已安装的 Microsoft Teams 或 Outlook，直接诱导验证器吐出高权限 Microsoft Graph 访问令牌与刷新令牌，甚至能静默篡改多因素认证设  
置与 FIDO2 通行密钥（Passkey）  
  
## 🎭 信任盲区：仅凭“自称”完成身份授权的认证代理  
  
Microsoft Authenticator 在移动端不仅仅是一款用于查看 6 位动态验证码的 TOTP 工具，更是微软企业级生态中的核心**认证代理程序（Broker Application）**  
。当用户在手机上使用 Microsoft Teams、OneDrive 或 Outlook 时，底层 MSAL（微软身份验证库）会通过 Android 跨进程通信（IPC）直接调用 Authenticator 集中处理凭据置换，实现免重复输入的单点登录（SSO）  
  
然而，安全研究员奥菲克·莱文（Ofik Levin）团队深入逆向发现，Authenticator 暴露给外部应用调用的认证端点存在致命的身份信任盲区：  
  
**1. 声明式参数全盘采信：**  
调用方发起获取 Token 的 Intent 时，可以在参数中自行填充 client_id  
、package_name  
 和 redirect_uri  
。  
  
**2. 缺失真实的调用方 UID 校验：**  
验证器内部逻辑仅仅根据请求中声称的参数，去核验重定向 URI 是否与微软官方配置一致，却**从未调用底层的系统 API（如 Binder.getCallingUid）来核查实际发起该调用的 Linux 进程到底是谁**  
。  
  
**3. 借壳生蛋：**  
恶意应用只需声明自己是设备上真实存在的 com.microsoft.teams  
，验证器在检查设备中确实安装了合法的 Teams 后便采信了请求，并将企业级授权凭据直接交还给恶意应用。  
```
// AuthMask 恶意 Intent 构造与身份伪造关键参数
Intent intent = 
new
 Intent(
"com.microsoft.identity.client.broker"
);
intent.setComponent(
new
 ComponentName(
"com.azure.authenticator"
,
"com.microsoft.identity.client.BrokerHostActivity"
));
// 虚假声明自身为官方 Teams 应用
intent.putExtra(
"client_id"
, 
"1fec8e78-bce4-4aaf-ab1b-5451cc387264"
); 
// 微软官方 Teams Client ID
intent.putExtra(
"package_name"
, 
"com.microsoft.teams"
);
intent.putExtra(
"redirect_uri"
, 
"msauth://com.microsoft.teams/..."
);
// 验证器未核验真实调用者 UID，直接返回真实 Access Token！
```  
  
## 💥 危害穿透全栈：从窃取内网邮件到操纵通行密钥  
  
利用该漏洞获得的凭据并非局部沙箱数据，而是具备强大破坏力的 **Microsoft Graph 委托令牌（Delegated Access Token）**  
：  
- **窃取全量 M365 数据：**  
以伪装 Microsoft Teams 为例，恶意软件获得的委托权限包括：读取与发送企业内部 Exchange 邮件、遍历 SharePoint 和 OneDrive 文档库、窃取群聊敏感记录以及操纵日历会议  
  
- **长期持久化后门：**  
攻击者同时可获取有效期较长的刷新令牌（Refresh Token），即便用户离开移动设备，攻击者也可在云端持续维持访问通道  
  
- **篡改 FIDO2 通行密钥：**  
更为危急的是，微软 Authenticator 内部包含账户安全设置管理接口。攻击者可调用该特权通道，在受害者不知情的情况下**添加攻击者控制的 Passkey 通行密钥**  
，或删除现有的硬件 MFA 校验，实现云上账户的完全鸠占鹊巢  
  
## 🛡️ 影响范围与企业处置方案  
  
该漏洞影响 Android 端 Microsoft Authenticator **6.0.0 至 6.2606.4246 之间所有存量版本**  
。iOS 版本由于采用苹果封闭的 Keychain 共享机制与不同的进程沙箱，不受此漏洞波及  
- **强制客户端版本基线：**  
微软已在发布版本 6.2606.4246  
 中修复该漏洞，加入了基于签名公钥比对的真实发起方 UID 校验。企业 IT 管理员应在 Microsoft Intune / MDM 策略中设立合规规则，阻断低于此版本的移动设备接入企业资源  
  
- **审计 Azure AD / Entra ID 异常访问：**  
重点排查通过移动设备端 Teams 或 Office 客户端发起的异常 Graph API 调用，尤其是短时间内发生多因素验证方式（MFA Method）变动或 Passkey 增删的审计日志  
  
- **严查第三方侧载软件：**  
普通员工应警惕通过非法渠道侧载（Sideloading）安装的未知 APK，防范搭载 AuthMask 攻击载荷的恶意应用潜伏  
  
  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】24小时内超6万次攻击针对WordPress RCE漏洞，更新过程无需登录就可以利用](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079657&idx=1&sn=0e7f6a51f91ae4c6aadcb6d7ca13c2b3&scene=21#wechat_redirect)  
  
  
  
[【安全圈】日本航空遭遇第三方数据泄露，暴露乘客信息](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079657&idx=2&sn=0312f3c3c447b408d319b55721289b3e&scene=21#wechat_redirect)  
  
  
  
[【安全圈】黑客利用 Cisco SD-WAN 漏洞部署定制的 KeyDrift 恶意软件](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079657&idx=3&sn=62b5b110843a42aca0265bbe72c31840&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Gitee 725 万个公开仓库被扫出 26400 个有效凭证，9473 个是阿里云密钥](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079646&idx=1&sn=b67092a6e295dc9135d4f0304a210caa&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
  
