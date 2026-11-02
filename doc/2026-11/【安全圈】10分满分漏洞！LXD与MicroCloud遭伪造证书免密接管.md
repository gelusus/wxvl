#  【安全圈】10分满分漏洞！LXD与MicroCloud遭伪造证书免密接管  
 安全圈   2026-11-02 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Ubuntu 母公司 Canonical 发布最高级别安全警报：旗下核心容器管理引擎 **LXD**  
 与分布式私有云平台 **MicroCloud**  
 爆出 CVSS 10.0 满分严重漏洞。远程管理 API 在处理双向 TLS（mTLS）客户端认证时存在验证逻辑倒置，服务端直接比对客户端递交的整条证书链，而未约束私钥所属的叶证书。攻击者无需获取管理员私钥，仅凭公开的受信任公钥证书拼接至证书链中，即可直接冒充合法管理员免密调用远程 API，全面夺取底层宿主机 root 权限  
  
## 🔍 致命的证书链比对：mTLS认证逻辑倒置  
  
LXD 与 MicroCloud 广泛采用基于 mTLS（Mutual TLS，双向身份验证）的 REST API 实现节点集群通信与远程运维管控。在标准安全设计中，客户端接入管理端口（默认 TCP 8443）时，必须出示客户端证书，并在 TLS 握手阶段通过 CertificateVerify  
 数据包证明自身拥有该证书对应的私钥  
  
然而，Canonical 官方披露的代码审计结果表明，受影响版本的身份校验逻辑存在根本性设计缺陷：  
  
🧩 标准 mTLS 与缺陷实现对比：  
  
**1. 密码学握手层：**  
TLS 协议规范仅强制握手发起方证明其持有“叶证书”（Leaf Certificate，即证书链末端）的私钥。对证书链中附带的上级 CA 或附加证书，协议本身不验证其私钥。  
  
**2. 缺陷权限判定层：**  
LXD 服务端在匹配信任列表（Trust Store）时，未局限于校验通过私钥认证的叶证书，而是**遍历了客户端上传的整条证书链**  
。只要链中任意一个证书存在于受信任指纹库中，系统便直接判定该连接具备合法管理员身份。  
  
这意味着防御边界彻底坍塌：由于 X.509 公钥证书本身属于明文公开展现的凭据（在网络握手、公共存储、日志中均可直接捕获），攻击者只需在本地生成一个由自己控制私钥的临时叶证书，然后将受害服务器信任的管理员公开证书作为“父级链”拼接在握手包中  
  
TLS 握手在叶证书处顺利通过密码学校验，随后 LXD 服务端在检查证书链时发现受信任公钥，直接向未授权请求开放全部管理权限。整个利用过程**完全不需要目标管理员的私钥**  
```
// LXD / MicroCloud 缺陷客户端认证逻辑还原
func
isTrustedClient
(tlsState *tls.ConnectionState, trustStore *TrustStore) 
bool
 {
// 错误实现：遍历客户端提交的所有 peer 证书（包含无私钥证明的链上证书）
for
 _, cert := 
range
 tlsState.PeerCertificates {
fingerprint := sha256Hex(cert.Raw)
if
 trustStore.Has(fingerprint) {
return true 
// 只要链中包含已知公钥即放行，私钥持有校验落空！
}
}
return false
}
```  
## 💥 从API绕过到宿主机沦陷：10分危害的实操路径  
  
LXD 不仅负责管理轻量级系统容器（LXC），还直接接管底层 Linux 内核虚拟化（KVM）虚拟机实例与物理存储池。一旦远程 API 认证被绕过，攻击者便具备与集群超级管理员完全等同的操作能力：  
- **秒级容器逃逸：**  
调用 REST 接口创建具备 security.privileged=true  
 配置的特权容器，将宿主机根目录 /  
 挂载进容器内部，直接读取物理机 /etc/shadow  
 或覆写 cron 任务  
  
- **集群全面横向渗透：**  
在 MicroCloud 架构中，一个计算节点失陷即可通过内部 raft 复制协议同步恶意载荷，污染整套虚拟化云网络  
  
- **公网扫描与批量打点：**  
Shodan 与 FOFA 测绘数据显示，全球有数万个 LXD 8443 端口直面公网开放。公网攻击者只要截获过一次历史 TLS 握手包，即可提取其管理员公钥并发起全自动批量利用  
  
## 🛡️ 受影响版本与应急加固指南  
  
Canonical 目前已通过 Snap 候选与稳定通道紧急推送安全补丁。受影响的全部系统版本与对应修复版本如下：  
- **LXD 6.x（最新特性分支）：**  
升级到 6.10.1  
 或更高版本  
  
- **LXD 5.21（LTS 稳定分支）：**  
升级到 5.21.9  
 或更高版本  
  
- **LXD 5.0（长期支持分支）：**  
升级到 5.0.11  
 或更高版本  
  
- **LXD 4.0（存量长期维护分支）：**  
升级到 4.0.15  
 或更高版本  
  
- **MicroCloud 2.x / 3.x：**  
升级到 2.1.5 / 3.4.1  
 或更高版本  
  
除立即执行 snap refresh lxd  
 更新外，运维与安全团队应立刻执行以下排查处置：  
- **切断公网暴露：**  
检查所有宿主机监听地址，严禁将 core.https_address  
 绑定在 0.0.0.0，必须限制在内网管理 VPN 或专用运维 VLAN 内部  
  
- **审计信任列表与历史日志：**  
执行 lxc config trust list  
，清理所有过期或未知来源的客户端证书指纹；审查近期的 API 访问日志，重点排查是否存在异常 IP 成功调用管理接口的记录  
  
  
  
  
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
  
  
  
