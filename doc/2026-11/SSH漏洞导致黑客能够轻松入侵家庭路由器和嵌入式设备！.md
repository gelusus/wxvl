#  SSH漏洞导致黑客能够轻松入侵家庭路由器和嵌入式设备！  
原创 mmc
                    mmc  AGI安全   2026-11-09 01:35  
  
## 漏洞概况  
  
257 个字节以上的任意密码，可以登录没有设置密码的账号  
  
11 月 5 日，OpenSSH 发布 10.6p2。发布说明为 **critical security workaround**  
：在缺少 PAM 支持的 musl 系统上，sshd 会接受任意长度在 257 到 1024 字节之间的密码，登录那些密码字段为   
*  
 的账号。  
  
*  
 在   
/etc/shadow  
 里的含义是「这个账号不接受密码登录」，输入任何密码都无法登陆。但这个漏洞让它接受了长度超过 256 字节的任意输入。  
  
利用条件很少：网络能到达 SSH 端口，知道一个账号名。不需要密钥，不需要知道口令，也不需要反复猜测。一次连接即可成功，没有重试。  
  
漏洞由 Wakefield 的 Özgün Kültekin 报告，把这个漏洞命名为   
*sshole  
，并公开了技术说明、自查脚本与可复现的容器环境。OpenSSH 认为根因在 sshd 之外，但仍可以在 sshd 里规避。截至 11 月 5 日没有分配 CVE 编号，也没有已发生的攻击报告。  
  
OpenSSH 10.6p2 发布说明写明 musl libc 且禁用 PAM 的 Linux 系统受影响，并说明漏洞由 Wakefield 的 Özgün Kültekin 报告OpenSSH 官方发布说明（2026-11-05）禁用 PAM + musl10.6p2 已修复OpenSSH 10.6p2 发布说明中的修复条目OpenSSH 官方发布说明（2026-11-05）  
## 漏洞原理  
  
第 1 步  
没有密码的账号，存储值是一个星号  
  
Unix 用   
/etc/shadow  
 里的   
*  
 表示账号没有可用密码。它不是哈希，没有算法、没有盐、没有长度，所以任何密码都无法与它匹配。这个写法从 1980 年沿用至今。  
  
第 2 步  
sshd 计算hash的过程  
  
不带 PAM 时，sshd 自己调用   
crypt()  
 计算哈希再比对。  
*  
 这个字符太短，不能当作盐，于是 sshd 会从机器上获取真实账号的真实哈希来充当盐。这个行为是 2016 年为修复 CVE-2016-6210 引入的：当时是为了防止攻击者通过响应时间判断哪些用户真实存在，做法是对不存在的账号也做一次等量的哈希计算。  
  
第 3 步  
musl 在失败时返回星号  
  
2012 年 8 月 9 日，musl 把   
crypt()  
 失败时的返回值从 NULL 改成   
*  
，因为很多程序不检查 NULL，会直接崩溃。三周后，musl 又给 SHA-512 加上了 256 字节的输入上限，防止有人用一个超长密码让服务器长时间计算。超过 256 字节时，  
crypt()  
 返回   
*  
。  
  
musl sha512crypt 源码定义 KEY_MAX 256，并在输入长度超过该值时返回 0研究者页面保存的 musl 源码截图上限：256 字节超过上限返回失败musl 源码中的 256 字节上限  
  
上图来自研究者公开页面保存的 musl 源码，能看到   
#define KEY_MAX 256  
 和长度检查。  
  
第 4 步  
两个星号相等  
  
sshd 把返回的   
*  
 与存储的   
*  
 比较，结果相等，判定密码正确。上限 1024 字节来自 sshd 自身对密码长度的限制。  
  
攻击者用 300 字节的密码登录密码字段为 * 的账号：musl 的 crypt() 返回 *，sshd 与存储值比对后判定相等，登录通过，再用 doas 提权到 root登录在一次连接内完成左：客户端的会话　右：sshd 内部的计算$ ssh alice@alpine-boxalice@alpine-box's password:AAAA…（共 300 字节）Welcome to Alpine!alpine-box:~$ iduid=1000(alice) gid=1000(alice)alpine-box:~$ doas sualpine-box:~# iduid=0(root) gid=0(root)sshd 内部的计算pw_password = "*"salt = NULLsalt = pick_salt()hash = crypt(pw, salt)musl: >256 字节 → "*"strcmp("*", "*") == 0return 1musl 的 crypt() 在失败时返回 *，而 * 正好是「该账号没有密码」在 /etc/shadow 里的存储值。两个 * 相等，登录被接受任意 257 到 1024 字节的密码，都会被无密码的账号接受。  
  
10.6p2 的改动是：存储字段短于 4 个字符时，直接拒绝匹配。OpenSSH 称这是 workaround，因为   
crypt()  
 的返回值语义不由 sshd 决定；Alpine 的稳定版软件包则直接打了同一个补丁。  
## 漏洞利用条件  
  
研究者公开的自查脚本按五条逐项检查，全部成立时才会被利用：  
  
五个条件：musl 系统、sshd 无 PAM、PasswordAuthentication yes、存在密码字段为 * 的账号、另一账号有 $5$ 或 $6$ 哈希五个条件全部成立时才会被利用1系统使用 musl libcAlpine、Buildroot、Yocto，以及 OpenWrt 上另装的 OpenSSH2sshd 未编译 PAM 支持Alpine 的 openssh-server 包默认如此；openssh-server-pam 走 PAM 则安全3PasswordAuthentication 为 yes这是 OpenSSH 的默认值4存在密码字段为 * 的账号key-only 用户、LDAP 用户、Alpine 的 root 与云镜像里的 alpine 用户5另有账号存着 $5$ 或 $6$ 哈希sshd 要借用它的盐来计算哈希五条全满足：任意 257–1024 字节的密码都能登录五个条件同时成立才会被利用musl、无 PAM、口令登录开着、一个 * 账号、另一个账号有哈希。  
  
PermitRootLogin prohibit-password  
 默认挡住用密码直接登录 root 的情况。但如果攻击者能登进一个没有密码的普通账号，再用   
doas  
 提权，这层保护就失效了。  
  
Alpine 3.16 起，基础镜像里的 root 是   
root:*  
。3.18.11 到 3.24.3 的 tiny-cloud 云镜像里，  
alpine  
 用户拥有免密   
doas  
。这类镜像首次启动时是安全的，一旦机器上出现   
$6$  
 哈希（例如管理员设置了一个控制台密码），就可以用长密码登录   
alpine  
 再提权到 root。  
  
Alpine 2021 年提交把 genrootfs.sh 中 root 的密码字段由感叹号改为星号，以允许禁用 PAM 时使用 SSH 密钥登录研究者页面保存的 Alpine 提交截图修改前：root:!修改后：root:*Alpine 将 root 的密码字段改为 *研究者页面保存的 Alpine 提交截图  
  
上图是研究者页面保存的 Alpine 提交记录。提交说明写明，改成   
*  
 是为了在禁用 PAM 时仍允许 SSH 密钥登录；截图里的 diff 显示   
root:!  
 被改为   
root:*  
。  
  
被   
!  
 锁定的账号不受影响：  
!  
 不等于   
*  
。整台机器上没有任何   
$  
 开头的哈希时也不受影响，因为没有盐可以借。  
## 漏洞影响范围  
  
受影响：Alpine 3.16 以上、装了 OpenSSH 的 OpenWrt、Buildroot/Yocto 与 musl 容器；不受影响：OpenWrt 默认的 Dropbear、glibc 发行版、走 PAM 的 sshd、OpenBSD/FreeBSD/macOS受影响的范围判断标准只有一条：musl + 无 PAM 的 sshd受影响Alpine 3.16 及以上的镜像alpine 云镜像的 alpine 用户装了 OpenSSH 的 OpenWrtBuildroot / Yocto 系统大量已发布的 musl 容器不受影响OpenWrt 默认的 DropbearDebian / Ubuntu / Fedora走 PAM 的 sshdOpenBSD / FreeBSD / macOS密码字段为 ! 的锁定账号标题中的「家庭路由器」需要加限定：OpenWrt 默认用 Dropbear，不受影响；只有另行安装了 OpenSSH 服务器的路由器才在范围内。路由器 SSH安装了 OpenSSH的路由器和嵌入式设备受影响  
  
Alpine 的 Docker 镜像每周下载量超过 3000 万次，这是风险面最大的部分：容器里通常只有 root 和几个服务账号，其中很多没有设置密码。  
  
安装了OpenSSH的路由器和嵌入式设备受到很大影响！  
## 修复方案  
  
第 1 步  
升级并重启 sshd  
  
处理三步：apk upgrade -U -a 后重启 sshd；无法升级时写入 drop-in 关闭 PasswordAuthentication；再查日志里的 Accepted password 与 doas 记录升级、关闭、排查只升级软件包不会替换正在运行的 sshd$ apk upgrade -U -a$ rc-service sshd restart# 只升级软件包时，正在运行的 sshd 不会被替换$ echo "PasswordAuthentication no" > /etc/ssh/sshd_config.d/00-no-passwords.conf$ sshd -t && rc-service sshd restartpasswordauthentication no升级时要注意先升级，再重启 sshdAlpine 点版本已带修复包自建源码用 10.6p2排查是否已被登录grep "Accepted password" 日志查 doas 记录、新用户、新公钥无法排除就按已失陷处理OpenSSH 10.6p2 · Alpine 3.21.10 / 3.22.8 / 3.23.8 / 3.24.4升级后必须重启 sshd升级软件包不会替换运行中的进程，重启之后修复才生效。  
  
只执行   
apk upgrade  
 不会替换正在运行的 sshd 进程，必须重启服务。OpenSSH 10.6p2 已包含修复；Alpine 为 3.21.10、3.22.8、3.23.8、3.24.4 发布了固定包，已过支持期的 3.16 到 3.21 分支也一并发了包。  
  
第 2 步  
不能立刻升级时，先关闭密码登录  
  
关闭之前先确认密钥登录可用，并保持当前会话不退出，直到用新会话登录成功。写入 drop-in 文件后，用   
sshd -t  
 检查配置，再重启 sshd。Alpine 从 3.19 起默认读取   
/etc/ssh/sshd_config.d/  
 目录。  
  
如果自己构建 OpenSSH，10.6p2 的补丁可以应用到 7.3p1 及之后的版本。  
  
第 3 步  
排查是否已经被人登过  
  
在   
/var/log/messages  
 里查找   
Accepted password  
，并确认对应的账号本不该有密码。同时检查   
doas  
 使用记录、最近新建的账号和文件、以及不认识的   
authorized_keys  
。Alpine 公告的原话是：如果无法排除发生过登录，就把这台机器当作已经失陷处理。  
## 漏洞总结  
  
2012 年 musl 让 crypt() 失败时返回 * 并加上 256 字节上限，2016 年 sshd 开始借用其他账号的盐，2022 年 Alpine 把 root 改为 *，2026 年 11 月 5 日 10.6p2 修复事件时间线问题的两半分别来自 2012 年和 2016 年2012-08-09musl 改返回 *2012-08-29加 256 字节上限2016-07-15sshd 借用盐2022Alpine root 改为 *2026-11-0510.6p2 修复测试规模12 个 sshd 构建约 17,600 次异常登录为什么走 PAM 不触发PAM 路径不拿 * 去比对最早验证的版本openssh 7.5p1配 musl 1.1.1610 月 30 日，OpenSSH 修了同一函数里的 pick_salt()，问题就在它下面四行。两处改动分别来自 2012 年和 2016 年2012 到 2026：两次改动叠加2012 年的返回值与 2016 年的借用盐，在 2026 年被发现可以互相抵消。  
  
问题分别来自 2012 年和 2016 年的两次改动，之后十年没有人把两部分问题放在一起看。2026 年 10 月 30 日，OpenSSH 的一次安全审查修了同一函数里的   
pick_salt()  
。  
  
OpenSSH 团队在发布说明里写了一段关于 AI 的说明：近期收到大量由 AI 模型发现或辅助生成的报告，其中一些 bug 后来又被另一位研究者独立发现。这说明不向开源项目报告问题的一方同样能发现这些 bug。因此团队决定更频繁地发布小版本，而不是把修复攒到下一个规划版本。  
  
如果你觉得文章对你有用，期望点赞，分享，转发给更多人！  
  
我们是mmc,我们下期再见！  
  
