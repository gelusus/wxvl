#  Broadcom发布VMware安全更新 VMSA-2026-0011: 一次性修复23个漏洞，4个Critical漏洞需紧急处理  
tuto
                    tuto  杂杂咱谈   2026-11-24 16:02  
  
## 一、漏洞概述  
  
2026 年 11 月 24 日，Broadcom 发布 VMware 安全公告 **VMSA-2026-0011**  
，针对 VMware 多款虚拟化产品发布安全更新，一次性修复 **23 个安全漏洞**  
。  
  
本次漏洞涉及 ESXi、vCenter Server、VMware Workstation 和 VMware Fusion 等产品，其中：  
- **4 个漏洞被评定为 Critical（严重）**  
- **5 个漏洞被评定为 High（高危）**  
- 其余漏洞评级为中危或其他级别  
  
本次更新中最值得关注的是 ESXi 宿主机上的两个远程代码执行漏洞，均获得 CVSS 9.8 分。攻击者在满足网络可达条件的情况下，可能利用这些漏洞在 ESXi 宿主机上执行代码，对虚拟化基础设施造成严重威胁。  
  
此外，vCenter Server Directory Service 还存在一个严重的认证绕过漏洞，可能使攻击者在未经授权的情况下访问相关服务。  
## 二、重点漏洞分析  
  
本次安全更新涉及多个组件，以下是需要重点关注的漏洞。  
### 1. ESXi I/O 过滤器服务远程代码执行漏洞  
  
**漏洞等级：Critical（严重）**  
  
**CVSS 评分：9.8**  
  
涉及漏洞：  
- **CVE-2026-123910：堆越界写（Heap Out-of-Bounds Write）**  
- **CVE-2026-123911：释放后使用（Use-After-Free，UAF）**  
两个漏洞均存在于 VMware ESXi 的 I/O 过滤器服务组件中。  
  
漏洞成因主要涉及内存管理问题：  
- CVE-2026-123910：攻击者可以利用堆内存越界写入缺陷，破坏目标进程的内存数据。  
  
- CVE-2026-123911：由于对象释放后仍被继续访问，攻击者可能利用该缺陷造成内存破坏。  
  
**安全影响：**  
  
两个漏洞均被评定为 CVSS 9.8，属于本次更新中风险最高的漏洞。  
  
在满足相应网络访问条件的情况下，攻击者可能利用漏洞在 ESXi 宿主机上执行代码，进而威胁宿主机及其承载的虚拟机。  
  
这类漏洞的危险性在于，攻击目标不再局限于单个虚拟机，而是可能直接影响底层虚拟化平台。  
### 2. vCenter Directory Service 认证绕过漏洞  
  
**漏洞等级：Critical（严重）**  
  
**CVE 编号：CVE-2026-123903**  
  
**CVSS 评分：9.8**  
  
**漏洞类型：认证绕过（Authentication Bypass）**  
  
**影响组件：vCenter Directory Service**  
  
该漏洞存在于 vCenter Server 的 Directory Service 组件中。  
  
网络攻击者可能利用该漏洞绕过正常身份认证机制，获得未经授权的访问权限。  
  
**安全影响：**  
  
vCenter Server 是 VMware 虚拟化环境的核心管理平台，负责管理 ESXi 主机、虚拟机以及相关基础设施资源。  
  
如果攻击者成功利用该漏洞，可能进一步威胁虚拟化环境的管理安全，造成未授权访问及后续攻击风险。  
### 3. vCenter 任意文件写入漏洞  
  
**CVE 编号：CVE-2026-123904**  
  
**漏洞类型：任意文件写入**  
  
**CVSS 评分：7.5**  
  
该漏洞与 vCenter Server 处理特定归档文件时的安全缺陷有关。  
  
攻击者可以利用该问题向目标系统写入任意文件。在特定条件下，恶意文件写入可能进一步导致设备代码执行。  
  
该漏洞虽然没有达到 Critical 级别，但考虑到其可能造成的系统完整性破坏，仍需要及时修复。  
### 4. XHCI USB 控制器越界写入漏洞  
  
本次更新还修复了 VMware XHCI USB 控制器中的越界写入问题。  
  
该漏洞可能被具有虚拟机管理员权限的攻击者利用，在运行 VMware Workstation 或 VMware Fusion 的宿主机上执行代码。  
  
这一问题尤其值得关注桌面虚拟化环境的用户。  
  
与前述 ESXi 宿主机漏洞不同，该漏洞涉及虚拟机管理权限与宿主机安全边界之间的风险。  
## 三、漏洞利用与攻击风险  
  
本次安全公告中最值得警惕的是 ESXi I/O 过滤器服务中的两个内存破坏漏洞。  
  
从攻击影响来看，主要存在以下几类风险：  
1. **ESXi 宿主机代码执行：利用堆越界写或释放后使用漏洞，攻击者可能在 ESXi 宿主机上执行代码。**  
1. **vCenter 未授权访问：通过 Directory Service 认证绕过漏洞，攻击者可能绕过正常认证机制访问管理服务。**  
1. **任意文件写入：利用特定归档文件处理缺陷，攻击者可能向目标设备写入恶意文件。**  
1. **虚拟机与宿主机边界风险：XHCI USB 控制器漏洞可能使具有虚拟机管理权限的攻击者影响运行 Workstation 或 Fusion 的宿主机。**  
截止目前为止Broadcom 目前尚未确认上述漏洞已在真实攻击活动中遭到利用。  
## 四、漏洞披露与研究人员  
  
根据 Broadcom 公告，本次两个 ESXi I/O 过滤器服务漏洞分别由以下安全研究人员报告：  
- **CrowdStrike：参与报告其中一个 ESXi 远程代码执行漏洞。**  
- **DEVCORE：参与报告另一个 ESXi 远程代码执行漏洞。**  
这两个漏洞均被评定为 CVSS 9.8，说明 ESXi 宿主机的相关组件存在较高安全风险。  
## 五、受影响产品及修复版本  
  
Broadcom 本次安全更新覆盖 vCenter Server 和 ESXi 的多个版本。  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">产品</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">修复版本</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">vCenter Server</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">9.1.2</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">vCenter Server</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">9.0.2.0300</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">vCenter Server</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">8.0 U3m</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">ESXi</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">9.1.2</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">ESXi</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">8.0 U3m</span></section></td></tr></tbody></table>  
**特别说明：**  
  
ESXi 9.0 对应的修复补丁目前仍在准备中，相关用户需要持续关注 Broadcom 后续发布的安全更新。  
  
建议管理员根据实际部署版本，核对官方公告中的受影响版本及对应补丁，不要仅凭产品主版本号判断系统是否安全。  
## 六、安全修复与防护建议  
  
鉴于本次漏洞涉及 ESXi 宿主机远程代码执行和 vCenter 身份认证绕过，建议 VMware 管理员尽快采取以下措施：  
### 1. 尽快安装官方安全补丁  
  
优先升级以下组件：  
- VMware ESXi 宿主机  
  
- VMware vCenter Server  
  
- VMware Workstation  
  
- VMware Fusion  
  
对于尚未获得对应修复版本的 ESXi 9.0 环境，关注 Broadcom 官方安全公告。  
### 2. 限制管理接口网络暴露  
- 不要将 ESXi 管理接口直接暴露到公网。  
  
- 限制 vCenter Server 的访问来源。  
  
- 通过专用管理网络或 VPN 访问虚拟化管理平台。  
  
- 使用防火墙限制非必要的网络连接。  
  
- 避免将管理服务直接开放给不可信网络。  
  
### 3. 加强虚拟化平台访问控制  
- 定期审查 ESXi 和 vCenter 管理员账户。  
  
- 删除不必要的高权限账户。  
  
- 遵循最小权限原则。  
  
- 启用多因素认证，并加强管理账户保护。  
  
- 定期检查异常登录和权限变更记录。  
  
### 4. 排查潜在异常活动  
- ESXi 宿主机上的异常进程和命令执行行为。  
  
- vCenter Directory Service 的异常访问记录。  
  
- ESXi 管理接口的非预期网络连接。  
  
- 虚拟机配置及权限的异常变更。  
  
- 宿主机文件系统中的可疑文件写入行为。  
  
对于尚未完成升级的环境，应提高安全监控级别。  
## 七、总结  
  
Broadcom 发布的 VMSA-2026-0011 是一次涉及 VMware 虚拟化基础设施的重要安全更新，共修复 23 个漏洞，其中 4 个达到 Critical 级别，5 个达到 High 级别。  
  
其中，**CVE-2026-123910 和 CVE-2026-123911 两个 CVSS 9.8 的 ESXi 远程代码执行漏洞，以及 CVE-2026-123903 vCenter 认证绕过漏洞，是本次更新最需要优先关注的问题。**  
  
由于 ESXi 和 vCenter Server 通常处于企业虚拟化基础设施的核心位置，一旦相关漏洞被成功利用，可能对宿主机、虚拟机及整个虚拟化管理环境造成连锁影响。  
  
建议企业安全团队立即梳理 VMware 资产清单，确认受影响版本，优先修复 ESXi 和 vCenter Server，并严格限制管理面的网络暴露。  
  
  
     
引用来源:   
Broadcom 官方安全公告  
  
