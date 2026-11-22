#  VMware vCenter 零日认证绕过漏洞，可直接接管虚拟化平台  
原创 维度攻防
                        维度攻防  维度攻防   2026-11-22 04:47  
  
> 公告编号：VMSA-2026-0009  
  
漏洞等级：严重（CVSS:3.1 9.8）  
  
漏洞状态：  
已在野利用，存在公开攻击工具  
  
发布时间：2026-11-22  
  
  
各位运维、安全同行注意！博通于 11 月 19 日发布紧急安全公告 VMSA-2026-0009，修复包含  
CVE-2026-123890在内的一批高危漏洞。其中 CVE-2026-123890 为 VMware vCenter Directory Service（vmdir）认证绕过零日漏洞，威胁情报机构证实该漏洞早在今年 9 月初就已经被 APT 组织 ARMORY CASTLE（UNC5221）实战利用，属于  
已经发生入侵事件的在野 0day。  
## 🔍 漏洞原理  
  
CVE-2026-123890 属于  
CWE-288：备用路径或通道的认证绕过，漏洞存在于 vmdir 服务，对应 TCP 2012 端口的rpc_srp_verifier接口。该服务在 SRP 加密握手流程中存在授权与密码逻辑缺陷，攻击者只要能够访问 2012 端口，  
不需要账号密码，即可构造恶意 RPC 请求，直接拿到 administrator@vsphere.local 的管理员会话权限。  
  
拿到 vCenter 最高权限意味着：✅ 查看、导出全部虚拟机磁盘✅ 篡改 ESXi 集群配置、网络策略✅ 导出 LMDB 身份数据库、窃取 SAML 签名密钥✅ 新增 SSO 管理员账号、持久化后门✅ 横向渗透内网、部署勒索、窃取核心业务数据  
> ⚠️ 重点区分：该漏洞和此前 CVE-2026-59309 不是同一个漏洞，打过 7 月补丁的环境依然受本次漏洞影响，不要误以为已经安全。  
  
  
攻击者配套工具：  
BONESAW攻击链路：访问 2012 端口 → 利用漏洞绕过认证 → 获取管理员会话 → 导出 LMDB 库、提取密钥 → 创建 SSO 管理员账号，完成长期驻留。  
## 📌 受影响版本 & 修复版本  
  
表格  
<table><thead><tr><th style="text-align: left;"><section><span leaf="">产品</span></section></th><th style="text-align: left;"><section><span leaf="">受影响版本</span></section></th><th style="text-align: left;"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr><td style="text-align: left;"><section><span leaf="">VMware vCenter / vSphere Foundation</span></section></td><td style="text-align: left;"><section><span leaf="">9.1.x</span></section></td><td style="text-align: left;"><section><span leaf="">9.1.1.0200</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">VMware vCenter / vSphere Foundation</span></section></td><td style="text-align: left;"><section><span leaf="">9.0.x</span></section></td><td style="text-align: left;"><section><span leaf="">9.0.2.0200</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">VMware vCenter</span></section></td><td style="text-align: left;"><section><span leaf="">8.0</span></section></td><td style="text-align: left;"><section><span leaf="">8.0 U3m</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">VMware Cloud Foundation</span></section></td><td style="text-align: left;"><section><span leaf="">5.x</span></section></td><td style="text-align: left;"><section><span leaf="">5.2.5</span></section></td></tr><tr><td style="text-align: left;"><section><span leaf="">VMware Telco Cloud Platform</span></section></td><td style="text-align: left;"><section><span leaf="">3.0 / 4.x /5.0.x~5.2.x</span></section></td><td style="text-align: left;"><section><span leaf="">升级至 vCenter 8.0 U3m</span></section></td></tr></tbody></table>> 7.0 版本：官方未提供通用补丁，有延保合同的客户需要联系博通支持获取专项修复方案。  
  
## 🚨 攻击面判断  
1. 端口：TCP 2012（vmdir SRP 服务端口）  
  
1. 攻击条件：网络可达该端口，无需账号、无需用户交互  
  
1. 风险放大场景：  
  
   - vCenter 放在公网、DMZ 区，2012 端口未做 ACL 限制  
  
   - 内网横向突破后，对内网 vCenter 集群扫描 2012 端口  
  
1. 现状：目前暂无公开 POC，但是 APT 组织已经掌握稳定利用工具，内网一旦被突破，虚拟化平台极易失守。  
  
## ✅ 应急处置方案（按优先级执行）  
### 1、优先打补丁（唯一根治方案）  
  
尽快将 vCenter 升级到上表对应的修复版本，参考博通官方公告 VMSA-2026-0009、KB463116。  
### 2、临时防护缓解手段（补丁窗口期间）  
- 防火墙、安全组策略：  
严格限制 TCP 2012 端口访问源，仅允许可信运维网段访问，禁止外网、业务服务器直接访问；  
  
- 边界设备、EDR 告警：新增 TCP 2012 端口访问审计，监控陌生 IP 访问该端口；  
  
- 不要直接把 vCenter 管理端口暴露公网。  
  
### 3、入侵排查（重点！怀疑被拿下的环境）  
  
如果你的 vCenter 版本在受影响范围，且内网存在失陷风险，立刻做下面排查：  
1. 核查 SSO 用户列表，检查是否存在陌生管理员账号；  
  
1. 审计 vmdir、vCenter 登录日志，核查异常 administrator 登录；  
  
1. 检索服务器是否存在 BONESAW、SAWSNAKE 相关工具、可疑二进制；  
  
1. 检查 data.mdb（LMDB 数据库）是否存在异常读取、导出行为；  
  
1. 核查 SAML 证书、签名密钥是否被篡改、导出；  
  
1. 核查所有虚拟机快照、克隆、下载记录。  
  
> 提示：攻击者拿到权限后会创建 SSO 账号做持久化，就算后续打了补丁，后门账号依然存在，  
补丁不等于清除入侵。  
  
## 📋 安全运营建议  
1. 虚拟化控制平面（vCenter）是企业核心控制点，安全等级等同于域控，严禁开放不必要端口；  
  
1. 建立 vCenter 资产台账，定期跟踪博通 VMSA 安全公告；  
  
1. 内网红队 / 渗透测试中，增加 2012 端口扫描、vmdir 服务检测项；  
  
1. 重点行业：运营商、政企、制造业、IDC 云平台，vCenter 集群量大，优先排查。  
  
虚拟化平台是现代数据中心的 “大脑”，一旦 vCenter 被拿下，整个虚拟化集群都相当于拱手送人。 这个漏洞属于典型的**低门槛、高危害、已经在野**  
的零日漏洞，留给大家的补丁窗口很短，建议运维、安全团队今晚就启动资产清点与端口收敛。  
  
