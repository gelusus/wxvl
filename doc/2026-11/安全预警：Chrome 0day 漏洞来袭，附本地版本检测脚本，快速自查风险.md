#  安全预警：Chrome 0day 漏洞来袭，附本地版本检测脚本，快速自查风险  
原创 维度攻防
                        维度攻防  维度攻防   2026-11-13 05:34  
  
近期多次监测到 Chrome 浏览器 0day 漏洞在野外被活跃利用，漏洞多集中在 V8 JS 引擎，属于内存破坏类高危漏洞，CVSS 评分 8.8 分，风险等级严重。  
  
很多小伙伴会问：0day 到底是什么？我的浏览器有没有中招？能不能用脚本批量检测内网终端？  
  
今天这篇文章，把原理、风险、自查方法、检测脚本、源码解析、应急处置方案一次性讲清楚。  
  
⚠️ **重要前置说明**  
0day 漏洞 = 厂商尚未公开细节、但已经被黑客在真实攻击中利用的漏洞。  
> 重点：**不存在一个脚本可以直接检测机器是否已经被 0day 漏洞利用成功**  
。 下面提供的脚本，作用是：**批量读取本机终端 Chrome 版本号，判断是否处于受影响版本区间**  
，属于版本基线检测，不是漏洞利用检测，也不是入侵行为检测。 真正的漏洞利用发生在渲染进程内存内，无简单特征，传统脚本很难直接捕获攻击行为。  
  
## 一、漏洞简述：为什么 Chrome 0day 这么危险？  
  
这类漏洞大多位于 Chrome V8 JavaScript 引擎，常见类型：释放后重用（UAF）、类型混淆、越界访问。  
  
✅ 攻击链路： 黑客制作恶意网页 → 诱导用户访问（钓鱼、邮件、广告跳转）→ 无需用户额外点击、下载文件 → 触发内存漏洞，在渲染沙箱内执行任意代码 → 后续配合沙箱逃逸、注入、下载木马、窃取 Cookie、账号凭证、本地文件等  
  
✅ 受影响范围： Google Chrome 正式版、Chromium 内核浏览器：Edge、360 极速、QQ 浏览器、Brave 等全部基于 Chromium 内核的产品，同步受同源漏洞影响。  
  
✅ 攻击特点：  
- 无文件攻击，很多攻击全程不落地 exe  
  
- 痕迹隐蔽，EDR、杀毒容易漏报  
  
- 只要浏览器版本没更新，访问恶意页面即存在风险  
  
> 谷歌的修复逻辑：收到野外利用情报后，紧急推送版本补丁，更新后封堵漏洞。**最有效的防护手段永远是升级浏览器**  
。  
  
## 二、手动自查方法（个人用户）  
  
普通用户不用脚本，一键查看版本： 在 Chrome 地址栏输入：  
```
chrome://settings/help
```  
  
页面会自动检测版本、自动更新。 ✅ 安全：版本 ≥ 官方修复后的稳定版 ❌ 风险：版本低于修复版本，尽快更新并重启 Chrome  
> 注意：更新后**必须完全关闭所有 Chrome 窗口重启**  
，补丁才会生效，只刷新页面没用。  
  
## 三、Windows 版本检测脚本  
  
适用场景：运维、安全人员对内网 Windows 终端批量巡检，导出清单，快速定位低版本风险主机。  
> 运行环境：Python3，Windows 平台，仅读取注册表，无写入、无修改操作。  
  
```
import winreg
import re
import sys
def get_chrome_version():
    """读取注册表获取Chrome版本号"""
    reg_paths = [
        r"SOFTWARE\Google\Chrome\BLBeacon",
        r"SOFTWARE\Wow6432Node\Google\Chrome\BLBeacon"
    ]
    for path in reg_paths:
        try:
            key = winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, path)
            version, _ = winreg.QueryValueEx(key, "version")
            return version.strip()
        except FileNotFoundError:
            continue
        except Exception as e:
            print(f"读取异常：{e}")
    return None
def parse_version(ver_str):
    """把版本字符串转为数字元组，用于对比"""
    match = re.search(r"(\d+)\.(\d+)\.(\d+)\.(\d+)", ver_str)
    if match:
        return tuple(map(int, match.groups()))
    return None
def check_vulnerable(ver_tuple):
    # 示例阈值：可根据最新CVE官方修复版本修改
    # 例：假设安全基线：主版本 >= 157
    if not ver_tuple:
        return False, "无法识别版本"
    major, minor, build, patch = ver_tuple
    if major < 157:
        return True, "【高危】版本低于安全基线，存在0day风险，请立即更新"
    else:
        return False, "✅ 当前版本安全，已修复对应漏洞"
if __name__ == "__main__":
    print("===== Chrome版本安全检测工具 =====")
    ver = get_chrome_version()
    if not ver:
        print("❌ 未检测到本机安装Chrome浏览器")
        sys.exit(0)
    print(f"检测到Chrome版本：{ver}")
    vt = parse_version(ver)
    risk, msg = check_vulnerable(vt)
    print(msg)
```  
  
📌 使用说明：  
1. 管理员权限运行（读取注册表权限更稳定）  
  
1. 自行修改 check_vulnerable  
 里面的版本基线，匹配最新 CVE 修复版本  
  
1. 企业内网可以搭配 WMIC、域脚本、SaltStack 批量下发采集，输出 CSV 报表  
  
## 四、macOS 专用检测脚本（暗影安全）  
> 适配 macOS 平台，Python3.8+，纯本地检测，**无任何网络请求**  
脚本特点：内置轻量混淆，优先读取 Plist 获取版本，兼容 Mac 上两种 Chrome 应用名  
  
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
""" 版权：暗影安全
本脚本仅对【本地】做安全检查，不产生任何网络行为。
"""
import base64
import os
import platform
import re
import struct
import subprocess
import sys
def _k(seed: int) -> bytes:
    out = bytearray()
    s = (seed * 0x9E3779B1) & 0xFFFFFFFF
    for _ in range(32):
        s = (s * 1103515245 + 12345) & 0x7FFFFFFF
        out.append(s & 0xFF)
    return bytes(out)
def _d(blob: str, seed: int) -> str:
    raw = bytearray(base64.b64decode(blob))
    ks = _k(seed)
    return bytes(b ^ ks[i % len(ks)] for i, b in enumerate(raw)).decode("utf-8")
_C = {
    "os":    "oMyQBFnH",
    "edge":  "37orC8qFMJD2tJroYw==",
    "probe": "BlACCwEWQAgBFEII",
    "tag":   "zTZZzcc71AzAlXse6cMmm6m7rrhDHkThsxpDgs+mCXQFmuNkqlXsRYuQJkLAlA==",
}
_S = {"os": 7, "edge": 1337, "probe": 40961, "tag": 65537}
def C(key: str) -> str:
    return _d(_C[key], _S[key])
class Fingerprinter:
    _PLIST_TMPL = ("/Applications/{name}.app/Contents/Info.plist",)
    _BIN_TMPL = ("/Applications/{name}.app/Contents/MacOS/{name}",)
    _BROWSERS = (
        [0x47, 0x6F, 0x6F, 0x67, 0x6C, 0x65, 0x20, 0x43, 0x68, 0x72, 0x6F, 0x6D, 0x65],
        [0x43, 0x68, 0x72, 0x6F, 0x6D, 0x65],
    )
    def __init__(self) -> None:
        self._facts: dict = {}
    def collect_platform(self) -> "Fingerprinter":
        fam = platform.system().lower()
        self._facts["family"] = fam
        if fam == "darwin":
            mac_ver = platform.mac_ver()[0] or "0"
            try:
                kern = platform.release()
            except Exception:
                kern = ""
            self._facts["os_ver"] = mac_ver
            self._facts["kern"] = kern
            try:
                self._facts["os_major"] = int(kern.split(".")[0]) - 9
            except Exception:
                self._facts["os_major"] = 0
        return self
    def collect_browsers(self) -> "Fingerprinter":
        found = []
        for encoded in self._BROWSERS:
            name = bytes(encoded).decode("utf-8")
            ver = self._version_from_plist(name) or self._version_from_binary(name)
            if ver:
                found.append((name, ver))
        self._facts["browsers"] = found
        return self
    def _version_from_plist(self, name: str):
        for path in self._PLIST_TMPL:
            p = path.format(name=name)
            if not os.path.exists(p):
                continue
            try:
                out = subprocess.run(
                    ["defaults", "read", p, "CFBundleShortVersionString"],
                    capture_output=True, text=True, timeout=5,
                )
                v = out.stdout.strip()
                if re.fullmatch(r"\d+(\.\d+){1,3}", v):
                    return v
            except Exception:
                pass
        return None
    def _version_from_binary(self, name: str):
        for path in self._BIN_TMPL:
            p = path.format(name=name)
            if not os.path.exists(p):
                continue
            try:
                out = subprocess.run(
                    [p, "--version"], capture_output=True, text=True, timeout=5
                )
                m = re.search(r"(\d+)\.(\d+)\.(\d+)\.(\d+)", out.stdout)
                if m:
                    return ".".join(m.groups())
            except Exception:
                pass
        return None
class Baseline:
    _SEG = ((0x9C + 0x1), 0x0, (0x1FAC << 2) | 0x7, (0x30 + 0x3))
    @staticmethod
    def _pack(t: tuple) -> int:
        return struct.unpack(">Q", struct.pack(">HHHH", *t))[0]
    def __init__(self) -> None:
        self._hi = self._pack(self._SEG)
    @staticmethod
    def parse(s: str) -> tuple:
        parts = [int(x) for x in re.findall(r"\d+", s)[:4]]
        while len(parts) < 4:
            parts.append(0)
        return tuple(parts)
    def within(self, ver: str) -> bool:
        return self._pack(self.parse(ver)) <= self._hi
class Pipeline:
    def __init__(self, fp: Fingerprinter, bl: Baseline):
        self._fp = fp
        self._bl = bl
    def run(self) -> int:
        facts = self._fp.collect_platform().collect_browsers()._facts
        gate_os = facts.get("family") == C("os")
        hits = []
        for name, ver in facts.get("browsers", []):
            if self._bl.within(ver):
                hits.append((name, ver))
        self._report(facts, gate_os, hits)
        return (1 if gate_os else 0) | (2 if hits else 0)
    def _report(self, facts, gate_os, hits) -> None:
        bar = "=" * 62
        print(bar)
        print("  本地安全检查   ")
        print(bar)
        print(f"  [*] 平台类别     : {facts.get('family', '未知')}")
        if facts.get("os_ver"):
            print(f"  [*] 操作系统     : macOS {facts['os_ver']} "
                  f"(内核 {facts.get('kern', '未知')})")
        browsers = facts.get("browsers") or []
        if browsers:
            for name, ver in browsers:
                print(f"  [*] 检测到浏览器   : {name} {ver}")
        else:
            print("  [*] 检测到浏览器   : (无)")
        if gate_os and hits:
            print("-" * 62)
            print(C("tag"))
            for name, ver in hits:
                print(f"      -> {name} {ver}")
            print(f"      受影响版本小于等于   : {C('edge')}")
            print("      影响说明     : 客户端可触发")
            print(bar)
            print("  [!] 处置建议：请立即升级受影响的浏览器版本。")
            print(bar)
        else:
            print("-" * 62)
            reason = []
            if not gate_os:
                reason.append("平台条件不满足")
            if not hits:
                reason.append("无浏览器落在基线区间内")
            print("  [+] 判定结论：未命中（" + "；".join(reason) + "）")
            print(bar)
def main() -> int:
    try:
        assert sys.version_info >= (3, 8)
        assert hasattr(platform, "mac_ver")
    except AssertionError:
        print("[-] 运行环境不支持")
        return 1
    pipeline = Pipeline(Fingerprinter(), Baseline())
    return pipeline.run()
if __name__ == "__main__":
    sys.exit(main())
```  
  
📌 使用说明：  
1. 运行要求：macOS + Python3.8+，普通用户权限即可  
  
1. 检测范围：仅 /Applications  
 目录下的 Chrome，放在其他目录的 Chrome 无法识别  
  
1. 基线更新：漏洞版本变更时，需要同步修改代码内基线参数  
  
1. 返回码采用位掩码，方便自动化批量巡检脚本调用  
  
