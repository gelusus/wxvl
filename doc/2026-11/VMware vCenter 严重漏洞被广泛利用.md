#  VMware vCenter 严重漏洞被广泛利用  
原创 泼猴看世界
                    泼猴看世界  表哥带我   2026-11-22 11:54  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/UA4ABKCY6OzHQwq5T4GEZcq7YCle2jPsQsKV7HKSXom5vsjicN6TQiaDOSIiclY8jdpX9XS8EKYLyvKv47mvz6Nkf6kwFSp1YkiawreLwobafzw/640?wx_fmt=webp&animated=1&from=appmsg "")  
  
**Broadcom发布了紧急安全更新，以解决VMware vCenter Server中一个关键身份验证绕过漏洞，该漏洞自2026年9月以来一直被威胁者积极利用。**  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/UA4ABKCY6Oy7q2eIiboXrSSTwNVEaGr7Lkb2DEcZ8syvw6tckOBGicic5yvwYQGNhdGExAYefZ5J7Zv8PfYv8oe2RibZV4qMRJOMWbU3bcryplo/640?wx_fmt=webp&from=appmsg "")  
  
**▍ 一、事件速览**  
  
<table></table><table><caption></caption><colgroup><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">项目</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">编号</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">CVE-2026-123890（同批 123891–123894）</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">厂商通告</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">VMSA-2026-0009，2026-11-19</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">产品</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">VMware vCenter Server，含 VCF / VVF / Telco Cloud 内嵌部署</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">组件</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">VMware Directory Service（vmdird）</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">类型</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">认证绕过，CWE-288 使用替代路径或通道</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">评分</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">CVSS 3.1 9.8，AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">前置条件</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">仅需网络可达，无凭据、无交互</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">结果</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">取得 administrator@vsphere.local 已认证会话</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">修复版本</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.1.1.0200 / 9.0.2.0200 / 8.0 U3m / 5.2.5</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">缓解措施</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">厂商明确声明 None</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">在野利用</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">自 2026-09 起作为零日使用</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">KEV</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">2026-11-20 收录</span></section></td></tr></tbody></table>  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/UA4ABKCY6OwRakGXFibI33OjMLl0LGV68qtVA5B4Vt7HjaxgX6wzslRoZDvIwb8BLmV0Wf0eQsx5O36OANEsichpuCww4cRyWGshM8Yp8AZXo/640?wx_fmt=webp&from=appmsg "")  
  
**▍ 二、披露时间线**  
  
这条漏洞的时间结构比多数 CVE 更值得注意：先被使用，后被披露，披露时补丁同步到位。  
- 2026 年 9 月起：在野利用开始，此时无公开编号、无补丁，处于零日窗口；  
  
- 2026-11-19：Broadcom 发布 VMSA-2026-0009，同步给出四个分支的修复版本；  
  
- 2026-11-19 起：公开渠道出现标注为 PoC 的第三方代码，多数未经验证；  
  
- 2026-11-20：CISA 纳入 KEV，要求 2026-11-23 前完成处置。  
  
从 9 月到 11 月，暴露窗口约两个半月。这个长度决定了处置不能只做打补丁，必须按已失陷排查。  
  
**▍ 三、组件与协议背景**  
  
理解这条漏洞需要先明确三个对象。  
  
vmdird 是 vCenter 内置的目录服务，负责身份、组、权限、证书、solution user 等对象的存储与认证。它不依赖外部 LDAP，vCenter 单机即可提供完整目录能力。  
  
data.mdb 是它的数据库文件，LMDB 格式。目录里的所有对象——账号、组成员、权限绑定、机器账号、SAML 相关配置——都在一个文件里。拿到读写能力，等于拿到 vCenter 的身份体系。  
  
SRP（Secure Remote Password）是目录服务使用的认证协议之一，设计目标是不让口令在网络上以任何形式传输：客户端与服务端各自基于盐值、口令派生材料与协商参数计算证据，互验后导出会话密钥。正常情况下即使抓包也无法反推口令。  
  
这条漏洞的落点在 SRP 握手依赖的时间戳参数：其生成方式可被预测。当该值可被攻击者预先推算，会话密钥的推导就不再需要口令参与，协议仍然正常完成，但认证前提已被抽空。  
  
公开披露到此为止，未给出具体字段构造。这一点决定了任何声称发某个固定报文就能进入的代码都不可信。  
  
**▍ 四、攻击链分阶段拆解**  
  
把公开信息串起来，完整链路分为六段。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/UA4ABKCY6OxnTwy4qg823qNeRM4Jb8MHaKiakPsiaEEJdHfeJZBWdTIKz8uvt5IdsTJnvXb9f5ScS2TNhQCkiaZiaylYrQNSL2naXnvwdFNz968/640?wx_fmt=webp&from=appmsg "")  
  
**4.1 侦察**  
  
确认 443（vSphere Client / API）可达，并确认 2012（vmdird）是否暴露。多数失陷案例的前提是管理面能被外部触达。  
  
**4.2 建立管理员会话**  
  
利用 SRP 时间戳可预测的缺陷，在不知道任何口令的情况下完成认证协商，取得 administrator@vsphere.local 的会话。这一步不产生口令错误、不触发账号锁定，日志上表现为一次成功的目录服务认证。  
  
**4.3 目录服务操作**  
  
以管理员会话为跳板，通过目录服务执行命令、读取文件，其中包括 data.mdb。到这一步，攻击者已能枚举全部身份对象。  
  
**4.4 窃取信任根**  
  
从 vCenter 的 SAML 令牌服务中提取签名密钥。这是整条链最关键的一步：SAML 令牌是 vCenter 各组件之间、以及 vCenter 与外部身份源之间互相信任的凭证，签名密钥即信任根。  
  
**4.5 持久化**  
  
公开报告记录了两件工具化操作：创建后门机器账号；用窃取的密钥伪造管理员级 SAML 令牌，并把 Global Administrator 权限授予 vCenter 根目录对象。此时攻击者不再依赖那个被绕过的会话。  
  
**4.6 横向与冒充**  
  
第二阶段工具使用同一把密钥伪造任意用户的令牌，绕过 MFA、冒充任意身份。配合 vCenter 对 ESXi 的管理关系，控制面向上延伸到主机，向下覆盖全部虚拟机。  
  
这条链的性质不是一次越权，而是接管信任体系。  
  
**▍ 五、受影响范围与修复版本**  
  
<table><caption></caption><colgroup><col/><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">分支</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">受影响</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">vCenter 9.1.x</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">&lt; 9.1.1.0200</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.1.1.0200</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">vCenter 9.0.x</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">&lt; 9.0.2.0200</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.0.2.0200</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">vCenter 8.0</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">&lt; 8.0 U3m</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">8.0 U3m</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">vCenter 5.x</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">&lt; 5.2.5</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">5.2.5</span></section></td></tr></tbody></table>  
  
部署形态上以下全部在内：独立 vCenter、SDDC Manager、VMware Cloud Foundation、VMware vSphere Foundation、VMware Telco Cloud Platform、VMware Telco Cloud Infrastructure。判断口径不是有没有装 vCenter，而是这套平台里是否包含 vCenter。  
  
Telco Cloud 的对应关系见 KB463116：TCP 3.0、4.x、5.0.x、5.1.x、5.2 与 TCI 3.0 的 vCenter 组件统一以 8.0 U3m 为修复版本；未列出的版本或处于扩展支持期的实例需走厂商支持渠道。  
  
vSphere 6.5 与 6.7 已过通用支持期，厂商不再评估，公开分析建议按受影响处理。  
  
两个操作约束必须在动手前确认：  
  
第一，没有 workaround。限制网络访问只是补偿控制，厂商原文如此定性，不能替代补丁。  
  
第二，这批更新不适用于 vCenter Quick Patch，必须走标准 vSphere 或 VCF 更新流程。并且安装 8.0 U3m 或 9.0.2.0200 会触发 Back-in-Time 限制，阻断后续升级到 VCF 9.1.1。补丁路径要和升级规划一起定。  
  
**▍ 六、为什么它的实际危害高于 9.8 这个数字**  
  
CVSS 描述单次利用的技术后果，不描述善后难度。这条漏洞有三点超出评分范围的影响。  
  
持久性不随口令轮换消失。常规认证绕过在补丁加改密码之后即失效；这里攻击者持有的是 SAML 签名密钥，改管理员口令不会让伪造令牌失效。只打补丁不轮换密钥，等于把后门留在原地。  
  
日志可见性差。攻击者使用的是目录服务的正常协议与正常端口，认证结果是成功。基于失败次数、异常状态码的检测规则在此基本无效，只能靠来源、时段、对象变更这类行为特征。  
  
控制面放大。vCenter 一次失守，向下覆盖全部虚拟机，向上覆盖全部 ESXi 主机，横向覆盖所有接入同一 SSO 域的组件。  
  
**▍ 七、自查**  
  
**7.1 版本核对**  
  
以 VAMI（5480）或 vCenter 管理界面显示的完整版本为准，逐字对照修复版本。关键在 build 号，不在主版本号——9.1.1.0200 与 9.1.0.xxxx 属同一主版本，但一个是修复后，一个不是。  
  
**7.2 端口暴露**  
  
除 443 外，重点确认 2012 是否对管理网段以外开放。vmdird 不应出现在非管理网络。同时检查 2010、2020、389、636、5480、9443 的可达范围。  
  
**7.3 只读自查脚本**  
  
端口连通性判断与公开端点版本指纹。  
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""CVE-2026-123890 只读自查：端口暴露 + 版本指纹。不做认证尝试。"""

import argparse
import socket
import ssl
import re
from urllib.request import Request, urlopen

PORTS = [
    (443, "vSphere Client / API"),
    (2012, "VMware Directory Service"),
    (2010, "vmdird alt"),
    (2020, "vmdird alt"),
    (389, "LDAP"),
    (636, "LDAPS"),
    (5480, "VAMI"),
    (9443, "VAMI mgmt"),
]

FIXED = [("9.1.x", "9.1.1.0200"), ("9.0.x", "9.0.2.0200"),
         ("8.0", "8.0 U3m"), ("5.x", "5.2.5")]

def probe(host, port, timeout):
    try:
        s = socket.create_connection((host, port), timeout=timeout)
    except Exception:
        return False
    s.close()
    return True

def fingerprint(host, port, timeout):
    ctx = ssl._create_unverified_context()
    for path in ("/sdk/vimServiceVersions.xml", "/sdk/", "/ui/"):
        url = "https://%s:%d%s" % (host, port, path)
        try:
            req = Request(url, headers={"User-Agent": "selfcheck/1.0"})
            with urlopen(req, timeout=timeout, context=ctx) as r:
                body = r.read(8192).decode("utf-8", "replace")
        except Exception:
            continue
        m = re.search(r"<version>\s*([0-9][^<\s]*)\s*</version>", body)
        if m:
            return m.group(1)
        m = re.search(r"(?:version|build)\s*[:=]\s*([0-9][0-9A-Za-z._-]*)", body, re.I)
        if m:
            return m.group(1)
    return None

def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("-H", "--host", required=True)
    ap.add_argument("-t", "--timeout", type=float, default=4.0)
    a = ap.parse_args()

    print("=" * 60)
    print("目标:", a.host)

    open_ports = []
    for port, desc in PORTS:
        ok = probe(a.host, port, a.timeout)
        print("  %-5d %-6s %s" % (port, "OPEN" if ok else "-", desc))
        if ok:
            open_ports.append(port)

    print("-" * 60)

    if 443 in open_ports:
        ver = fingerprint(a.host, 443, a.timeout)
        print("版本指纹:", ver or "未获取（需人工核对 VAMI）")
        print("对照修复版本:")
        for branch, fixed in FIXED:
            print("   %-8s -> %s" % (branch, fixed))
    else:
        print("443 不可达：若为内网实例，请从管理网段重测。")

    risky = [p for p in open_ports if p in (2012, 2010, 2020, 389, 636)]
    if risky:
        print("-" * 60)
        print("目录服务端口对外开放:", ",".join(str(p) for p in risky))

    print("=" * 60)

if __name__ == "__main__":
    main()

```  
  
用法：  
```
python3 check_cve_2026_123890.py -H vc.example.com

```  
  
接下来这个适合已经部署VMware PowerCLI工具的Windows运维环境，直接对接vCenter获取版本信息。  
```
<#
vCenter CVE-2026-123890 受影响版本检测脚本
依赖：VMware PowerCLI 模块
安装命令：Install-Module -Name VMware.PowerCLI -Scope CurrentUser
运行前先执行：Connect-VIServer -Server <vCenter地址> -User <账号> -Password <密码>
#>

# 忽略证书警告（生产环境建议配置可信证书）
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false | Out-Null

if (-not $global:DefaultVIServers) {
    Write-Host "请先使用 Connect-VIServer 连接目标vCenter" -ForegroundColor Red
    exit
}

# 获取核心版本信息
$vcServer = $global:DefaultVIServers[0]
$aboutInfo = Get-View ServiceInstance
$versionFull = $aboutInfo.Content.About.Version
$buildNumber = $aboutInfo.Content.About.Build
$productName = $aboutInfo.Content.About.Name

Write-Host "========================================"
Write-Host "目标vCenter: $($vcServer.Name)"
Write-Host "产品名称: $productName"
Write-Host "完整版本: $versionFull"
Write-Host "Build编号: $buildNumber"
Write-Host "========================================"

# 受影响主版本判定
$majorVersion = [version]$versionFull
$affected = $false

if ($majorVersion -ge [version]"8.0.0" -and $majorVersion -lt [version]"8.0.3") {
    $affected = $true
    $fixVersion = "8.0 U3m (Build 24453095)"
}
elseif ($majorVersion -ge [version]"9.0.0" -and $majorVersion -lt [version]"9.0.2.0200") {
    $affected = $true
    $fixVersion = "9.0.2.0200 (Build 24518301)"
}
elseif ($majorVersion -ge [version]"9.1.0" -and $majorVersion -lt [version]"9.1.1.0100") {
    $affected = $true
    $fixVersion = "9.1.1 对应累积补丁 (Build 24537668)"
}
elseif ($majorVersion -lt [version]"8.0.0") {
    $affected = $true
    $fixVersion = "无官方补丁，建议升级或严格隔离"
}

if ($affected) {
    Write-Host "`n[风险提示] 当前版本存在CVE-2026-123890漏洞风险" -ForegroundColor Yellow
    Write-Host "建议修复版本: $fixVersion"
    Write-Host "注意：本次补丁为累积更新包，不支持Quick Patch快速安装"
}
else {
    Write-Host "`n[状态正常] 当前版本不在已知受影响范围内" -ForegroundColor Green
}

# 增强链接模式状态查询（仅作信息补充，不影响风险判定）
try {
    $elmSetting = Get-AdvancedSetting -Entity $vcServer -Name "config.vpxd.sso.elm.enabled" -ErrorAction Stop
    $elmStatus = if ($elmSetting.Value -eq 'true') { "已启用" } else { "未启用" }
    Write-Host "`n增强链接模式状态: $elmStatus"
    Write-Host "注：无论是否启用增强链接模式，均不改变漏洞存在性"
}
catch {
    Write-Host "`n增强链接模式状态查询失败，不影响核心检测结果"
}

Write-Host "`n检测完成。最终判定请以博通官方安全公告为准。"


```  
  
**7.4 日志与痕迹**  
  
补丁只管未来。暴露两个多月的实例，按已失陷排查。以下路径随 vCenter 版本不同，先确认再查：  
```
ls -la /var/log/vmware/vmdird/
ls -la /var/log/vmware/vpxd/
ls -la /var/log/vmware/sso/

```  
  
**排查痕迹项**  
  
<table><caption></caption><colgroup><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">痕迹项</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">新增本地用户</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">非变更窗口内出现的账号</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">新增 solution user</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">目录服务里非预期的解决方案身份</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">新增机器账号</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">公开报告记录了后门机器账号的创建</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">角色/权限组变更</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">尤其对根目录对象授予 Global Administrator</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">data.mdb 异常读取</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">目录数据库被访问的时间与来源</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">SAML 签名密钥访问</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">密钥文件被读取的审计记录</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">/tmp/123</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">公开报告记录的落地路径</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">管理员异常认证</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">来源 IP 与既有管理习惯不符的成功登录</span></section></td></tr></tbody></table>  
  
其他推荐阅读  
  
[今天你被xxxx2shell漏洞了吗](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488753&idx=1&sn=83dbb5132522cb31d72a765326eb7ea9&scene=21#wechat_redirect)  
  
  
[网传某光猫被曝隐藏插件可识别手机号、扫描内网](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488735&idx=1&sn=47c09ac5583c773401e663560d4526a0&scene=21#wechat_redirect)  
  
  
[Kimi高级研发穿女仆装出席云桥大会](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488728&idx=1&sn=caf9bec0e3ff0ddfabe50efb544cba7a&scene=21#wechat_redirect)  
  
  
[PHP爆 远程命令执行等 多个严重漏洞](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488683&idx=1&sn=167ca43bc9b755881a847642b7242ded&scene=21#wechat_redirect)  
  
  
[Oracle MySQL 再现严重漏洞](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488673&idx=1&sn=13bb55948b944639f0e8747a70802495&scene=21#wechat_redirect)  
  
  
[【吃瓜】AI挖洞别把自己挖进去了](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488590&idx=1&sn=732ebddd6338573513adf78820554a81&scene=21#wechat_redirect)  
  
  
[【吃瓜】麦当劳APP疑被AI渗透攻击](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488601&idx=1&sn=025804ddd8f12c827f989b9b772c06c0&scene=21#wechat_redirect)  
  
  
[360集团副总裁卜思南被警方带走调查](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488612&idx=1&sn=ed991ce45ad2168b2e02adbb00cd6185&scene=21#wechat_redirect)  
  
  
  
点一下推荐吧  
，喜欢的转发一下  
  
