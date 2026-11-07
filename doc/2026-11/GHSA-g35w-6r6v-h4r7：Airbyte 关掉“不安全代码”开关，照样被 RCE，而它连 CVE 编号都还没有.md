#  GHSA-g35w-6r6v-h4r7：Airbyte 关掉“不安全代码”开关，照样被 RCE，而它连 CVE 编号都还没有  
原创 mourenzheng
                    mourenzheng  网络安全民工   2026-11-06 11:01  
  
## 一、漏洞定性：一个“设计开关”形同虚设的反射注入  
  
Airbyte 的 Connector Builder 与 Manifest Server 在 **2.2.0 之前**  
，不限制声明式 manifest 可以引用哪些 Python 类作为自定义组件。提交一份 manifest 触发一次测试读取，服务端就会加载并调用容器内**任意已安装的 Python 对象**  
，在 Connector Builder 或 Manifest Server 容器内以 Airbyte 用户的身份执行任意命令。CWE 归类为**不安全反射（CWE-470）**  
，同时标注 CWE-306（关键功能缺失认证）。  
  
最需要单独指出的一点：**即使 AIRBYTE_ENABLE_UNSAFE_CODE 没有启用，这个利用依然成立**  
。这个本该拦住自定义代码的开关，在 manifest 的类加载路径上失去了作用。漏洞的根源不是内存破坏，而是**组件引用校验的设计缺陷**  
——攻击者不需要写 shellcode，只需要在 manifest 里指向一个不该被加载的类。  
<table><thead><tr><th data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">项</span></span></section></th><th><section><span leaf=""><span textstyle="" style="font-size: 15px;">值</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">漏洞编号</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">GHSA-g35w-6r6v-h4r7</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（</span></span><strong><span leaf=""><span textstyle="" style="font-size: 15px;">尚无 CVE</span></span></strong><span leaf=""><span textstyle="" style="font-size: 15px;">）</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">漏洞类型</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">不安全反射（CWE-470）/ 任意代码执行</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">CVSS 3.1（Airbyte 自评）</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">9.9 Critical</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（</span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">S:C</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;">）</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">CVSS 3.0（JVN 官方口径）</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">8.8</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（</span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;">）</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">CVSS 4.0（JVN 官方口径）</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">8.7</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（</span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">PR:L</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;">，无范围变更）</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">受影响版本</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">全部自托管版本 &lt; 2.2.0</span></span></strong></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">修复版本</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">2.2.0</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">（随附 </span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">airbyte/manifest-server</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;"> 7.23.7、</span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">airbyte-cdk</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;"> 7.23.7）</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">在野利用</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">已确认</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">。日本 JPCERT/CC 于 2026-11-04 以 </span></span><strong><span leaf=""><span textstyle="" style="font-size: 15px;">JVN</span><a class="wx_topic_link" topic-id="mw4uyx0x-fn5tbl" style="color: #576B95 !important;" data-topic="1" data-recommend=""><span textstyle="" style="font-size: 15px;">#12040144</span></a></span></strong><span leaf=""><span textstyle="" style="font-size: 15px;"> 通告，明确写明“已确认在野利用”</span></span></section></td></tr><tr><td data-colwidth="166"><section><span leaf=""><span textstyle="" style="font-size: 15px;">Airbyte Cloud</span></span></section></td><td><strong><span leaf=""><span textstyle="" style="font-size: 15px;">无需动作</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">，云服务不受影响</span></span></section></td></tr></tbody></table>  
**没有 CVE 编号这件事本身就是一个操作性问题**  
：它不会出现在绝大多数扫描器的结果里，也不会进入依赖 CVE 编号的自动化补丁流程。资产清点只能靠**主动识别 Airbyte 部署并核对版本**  
，而不是等扫描器报出来。  
## 二、利用门槛：两档，差别极大  
  
漏洞的 CVSS 向量中 PR:L  
（需要低权限）反映的是 2.x 的默认场景，但 **1.x 的 legacy Helm 部署把门槛降到了零**  
。  
  
**第一档：Airbyte 2.x（< 2.2.0）。**  
 需要 **Workspace Reader 或 Organization Reader 及以上角色**  
。这意味着攻击者必须先拥有一个低权限的 Airbyte 账户。在一个典型的数据团队中，Reader 角色往往被广泛分配——查看连接器、浏览工作区、读取配置——这些权限看起来“只读”，但足以提交一份 manifest 并触发类加载。  
  
**第二档：Airbyte 1.x + legacy Helm chart。**  
 Connector Builder 服务**不继承 global.auth.enabled 设置**  
。任何能通过网络访问到该服务的客户端，**无需任何认证**  
即可提交 manifest 并利用。这是完全未认证的远程代码执行。  
  
**Airbyte 1.x 官方不再提供修复**  
。唯一的路径是升级到 2.2.0。如果组织中存在遗留的 1.x 部署（尤其是用旧版 Helm chart 安装的），它就是一个**没有认证门槛的 RCE 入口**  
，而它可能连 CVE 编号都没有，因此不会触发任何基于 CVE 的警报。  
## 三、利用后果：凭据集中处的单点突破  
  
Airbyte 容器的价值不在于容器本身，而在于它**握着什么**  
。一个数据集成平台的存在意义就是连接源数据库、SaaS API 和目标数仓。这些连接凭据——数据库密码、API token、云存储密钥——集中存储在 Airbyte 容器可达的范围内。  
  
在 Connector Builder 或 Manifest Server 容器内执行命令，意味着攻击者可以读取这些凭据，并且可以从容器访问到它所能触达的内部服务。**一个低权限的 Workspace Reader 账户，或者一个未认证的 1.x Connector Builder 端点，就可能成为通往多个上下游系统的跳板**  
。  
  
这也是为什么修复建议中必须把 **“升级”和“轮换凭据”当成两件独立的事**  
。升级关闭了执行路径，但已经泄露的凭据不会因为升级而失效。  
## 四、Python 检测脚本（受控验证，仅限授权环境）  
  
由于漏洞利用需要提交一份 manifest 并触发类加载，**外部黑盒无法在不发送利用载荷的情况下可靠判定**  
。以下脚本执行的是**受控的版本指纹与认证面探测**  
，不提交恶意 manifest，不执行命令。适用于授权环境下的资产排查。  
```
#!/usr/bin/env python3
"""
GHSA-g35w-6r6v-h4r7 Airbyte Connector Builder 安全检测（只读）
用法: python airbyte_ghsa_check.py https://airbyte.example.com
仅探测版本端点与认证状态，不提交 manifest，不执行命令。
"""
import sys
import re
import requests
import urllib3
urllib3.disable_warnings()
TIMEOUT = 10
def check(base):
    base = base.rstrip("/")
    headers = {"User-Agent": "Mozilla/5.0 (security-check)"}
    # 1) 确认是 Airbyte
    try:
        r = requests.get(base, headers=headers, timeout=TIMEOUT, verify=False)
    except requests.RequestException as e:
        print(f"[-] 连接失败: {e}")
        return
    if "airbyte" not in r.text.lower() and "airbyte" not in r.headers.get("Server", "").lower():
        print("[-] 未检测到 Airbyte 特征")
        return
    print(f"[*] 目标: {base}")
    print("[*] 确认是 Airbyte 实例")
    # 2) 尝试读取版本信息（路径可能因版本/部署而异）
    version = None
    for path in ["/api/v1/health", "/api/v1/version", "/health"]:
        try:
            rv = requests.get(f"{base}{path}", headers=headers,
                              timeout=TIMEOUT, verify=False)
        except requests.RequestException:
            continue
        if rv.status_code == 200:
            m = re.search(r"(\d+\.\d+\.\d+)", rv.text)
            if m:
                version = m.group(1)
                print(f"[*] 从 {path} 提取到版本: {version}")
                break
    if version:
        parts = tuple(int(x) for x in version.split(".")[:3])
        if parts < (2, 2, 0):
            print(f"[!] 版本 {version} < 2.2.0，受 GHSA-g35w-6r6v-h4r7 影响")
            print("[!] 2.x 需要 Workspace Reader 及以上角色；1.x legacy Helm 可能未认证可利用")
        else:
            print(f"[+] 版本 {version} >= 2.2.0，已修复")
    else:
        print("[*] 无法自动提取版本，请通过 Airbyte UI 或部署清单确认")
    # 3) 探测 Connector Builder 端点是否可达（不提交 manifest）
    cb_paths = ["/api/v1/connector_builder", "/connector_builder", "/api/v1/manifest"]
    for path in cb_paths:
        try:
            rc = requests.get(f"{base}{path}", headers=headers,
                              timeout=TIMEOUT, verify=False)
        except requests.RequestException:
            continue
        if rc.status_code in (200, 401, 403, 405):
            print(f"[*] Connector Builder 端点 {path} 返回 {rc.status_code}")
            if rc.status_code == 200:
                print("[!] 该端点无需认证即可响应，需人工确认是否暴露了 manifest 提交能力")
            break
    print("\n[*] 本脚本不提交任何 manifest，不执行命令。")
    print("[*] 若版本 < 2.2.0，请升级并审计容器可达的凭据。")
if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"用法: {sys.argv[0]} https://airbyte.example.com")
        sys.exit(1)
    check(sys.argv[1])
```  
  
  
**Nuclei 的适用性**  
：该漏洞的利用需要提交结构化的 manifest JSON 并触发类加载，不是单个 HTTP 请求就能验证的形态。Nuclei 更适合做**版本指纹**  
（如果 /api/v1/health  
 或 /api/v1/version  
 暴露版本号），但**不建议**  
用 Nuclei 发送 manifest 载荷——那本身就是利用行为，而非检测。实际的资产排查应依赖版本核对（上文本地脚本或部署清单）和认证面审计。  
## 五、修复与处置：升级、收口、轮换凭据  
### 5.1 升级（唯一根治）  
  
**升级至 Airbyte 2.2.0 或更高版本**  
。修复版本随附 airbyte/manifest-server  
 7.23.7 和 airbyte-cdk  
 7.23.7。修复后，通过 Connector Builder 提交的 manifest 中的自定义组件**只有在显式设置 AIRBYTE_ENABLE_UNSAFE_CODE=true 时才会被接受**  
。  
  
**Airbyte 1.x 官方不再提供修复**  
。必须升级到 2.2.0，没有向后移植的补丁。  
### 5.2 无法立即升级时的临时措施  
<table><thead><tr><th data-colwidth="256"><section><span leaf=""><span textstyle="" style="font-size: 15px;">措施</span></span></section></th><th><section><span leaf=""><span textstyle="" style="font-size: 15px;">效果</span></span></section></th><th><section><span leaf=""><span textstyle="" style="font-size: 15px;">局限</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="256"><section><span leaf=""><span textstyle="" style="font-size: 15px;">限制 Connector Builder / Manifest Server 网络访问，仅允许 Airbyte 服务器可达</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">阻断外部直接提交 manifest</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">不修复漏洞本身，内部恶意用户仍可利用</span></span></section></td></tr><tr><td data-colwidth="256"><section><span leaf=""><span textstyle="" style="font-size: 15px;">将 Workspace Reader / Organization Reader 角色仅授予可信用户</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">降低 2.x 的 </span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">PR:L</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;"> 暴露面</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">不解决 1.x 的未认证路径</span></span></section></td></tr><tr><td data-colwidth="256"><strong><span leaf=""><span textstyle="" style="font-size: 15px;">特别检查 1.x legacy Helm 部署</span></span></strong><section><span leaf=""><span textstyle="" style="font-size: 15px;">：确认 Connector Builder 未暴露在办公网或公网</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">移除零认证入口</span></span></section></td><td><section><span leaf=""><span textstyle="" style="font-size: 15px;">需要主动清点，扫描器不会告警</span></span></section></td></tr></tbody></table>### 5.3 升级之后的必做项：轮换凭据  
  
JPCERT/CC 的通告没有提供入侵指标、受害数量或攻击者后续动作的细节。这意味着**无法仅从漏洞通告判断一个 Airbyte 实例是否已经被利用**  
。对于任何在 2.2.0 之前运行、且 Connector Builder 对不可信用户可达的实例，应视为**可能已经暴露**  
。  
  
**处置顺序**  
：  
1. 升级到 2.2.0+。  
  
1. **轮换 Airbyte 容器可达的所有凭据**  
：源数据库密码、SaaS API token、目标数仓密钥、云存储访问密钥。  
  
1. 检查容器或 Pod 的异常出站连接、异常进程、以及 Airbyte 日志中非预期的 manifest 提交记录。  
  
1. 如果无法确认历史暴露情况，轮换是唯一确定的止损动作。  
  
## 六、结语：没有 CVE 的漏洞，不会等你“按流程”处理  
  
GHSA-g35w-6r6v-h4r7 的 CVSS 评分在 8.7 到 9.9 之间浮动，取决于谁在打分——JVN 给 8.8（3.0）/ 8.7（4.0），Airbyte 自评 9.9。但评分差异不是重点。重点是三件事同时成立：**在野利用已被日本 CERT 确认；AIRBYTE_ENABLE_UNSAFE_CODE 开关挡不住它；它没有 CVE 编号，不会出现在扫描器报告和自动化补丁流程里**  
。  
  
对于自托管 Airbyte 的团队，行动不需要等待 CVE 分配：**核对版本（< 2.2.0 即受影响）、升级、收口 Connector Builder 的网络可达性、轮换容器握着的凭据**  
。1.x legacy Helm 部署要单独查——那个路径不需要任何认证。  
  
Airbyte 容器里的凭据价值，远远超过容器本身。一个“关掉了自定义代码”却依然能加载任意 Python 类的反射缺陷，在一个持有上下游系统密钥的容器里，就是一个**通往数据基础设施的跳板**  
。  
  
  
