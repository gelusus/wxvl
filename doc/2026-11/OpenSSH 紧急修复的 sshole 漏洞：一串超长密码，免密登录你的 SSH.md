#  OpenSSH 紧急修复的 *sshole 漏洞：一串超长密码，免密登录你的 SSH  
原创 APT-101
                    APT-101  APT-101   2026-11-06 04:00  
  
[#OpenSSH]()  
[#CVE]()  
-2026-123158　[#漏洞通告]()  
[#Alpine]()  
[#安全运维]()  
  
## 01 先看这条公告  
  
2026 年 11 月 5 日，OpenSSH 紧急发布了 10.6p2，修掉一个潜伏多年的认证绕过漏洞，社区给它起了个名字叫 ***sshole**  
（CVE-2026-123158，评分 8.1）。  
  
一句话说清它有多邪门：**攻击者不需要知道你的密码，也不需要 SSH 私钥，只要发一串超过 256 字节的超长密码，就能直接远程登录特定 Linux 系统上的"无密码锁定账户"。**  
 你以为是锁死的账户，它一串字母 A 就能打开。  
## 02 三个条件，缺一不可  
  
先说清楚：不是"随便哪台机子，输入长密码就能秒破"。要同时满足三个条件：  
1. **系统用的是 musl libc**  
，而不是绝大多数发行版（Ubuntu、Debian、CentOS）的 glibc。musl 常见于 Alpine 这类轻量发行版、容器和嵌入式系统。  
  
1. **sshd 没走 PAM**  
。也就是 SSH 服务的密码认证没有经过 PAM 模块（编译时未启用，或配置里 UsePAM no  
）。  
  
1. **存在星号锁定的账户，而且系统里有能"借盐"的活账户**  
。/etc/shadow  
 里某些无密码账户（比如只允许密钥登录的 root、alpine）的密码字段被设成了单个 *  
；同时系统里至少有一个用 SHA-256/SHA-512 哈希的活跃账户，sshd 得从它那里借盐（Salt）。  
  
Alpine 默认环境恰好三条全占，这就是为什么重灾区集中在它身上。  
## 03 根源：一场历史设计的巧合碰撞  
  
这不是哪个环节写错了，是三个"历史正确"碰在了一起：  
- **1980 年代的约定**  
：Unix 传统上把无密码、仅密钥登录的账户，在 shadow 里记为 *  
，意思是"密码不可用"；  
  
- **2012 年的改动**  
：musl libc 的 crypt()  
 在收到超过 256 字节的密码时，为了不崩溃、不超时，直接返回一个特殊字符串 *  
 表示出错；  
  
- **碰撞通关**  
：攻击者发一个超长密码，未启用 PAM 的 sshd 拿它去调用 musl 的 crypt()  
 做比对，musl 返回 *  
（报错），而 sshd 把 *  
 当成正常哈希结果，拿去和 shadow 里该账户的 *  
 一比，完美相等，放行登录。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PIWj1VguNovJMLZxh1B8hwdiaic5joZFSnKvml25Aiam8pKeo6x4t3eSlsJjBQmWxRbzjAjHrEULiaABh2lNWibdJibzeEXPAwScGJvITl4Mu00Hw/640?wx_fmt=png&from=appmsg "")  
  
图1 哨兵撞车：musl 的 crypt 失败返回 *  
，shadow 的 *  
 是锁定标记，OpenSSH 的时序均衡让两者相遇——单看都对，合在一起就是免密登录   
  
![](https://mmbiz.qpic.cn/mmbiz_png/PIWj1VguNouGXwWqHjpzmqRicniaGTFOWkfZTG4iaqUU1j6xHicAiclk00IG2ibdhMHohZLuynzfhUKWKTXZIkIngcjFX12pxcXMRNAnqQOzU8Ovw/640?wx_fmt=png&from=appmsg "")  
  
图2 可利用窗口 257–1024 字节：musl 的 256 字节上限与 OpenSSH 的 1024 字节上限，共同划出这个洞  
## 04 谁中招，谁安全  
  
**高危**  
：  
- **Alpine Linux**  
：默认构建完全符合上面三个条件。尤其要注意，Alpine 官方给 AWS、Azure 的云镜像（Tiny-Cloud）默认带一个**无密码的 alpine 账户**  
，这个账户还能不用密码直接 doas  
 提权，被利用就是直接拿 root；  
  
- 基于 musl 构建、关掉 PAM 的 **OpenWrt、Buildroot、Yocto**  
 自制系统，以及大量 **Alpine Docker 镜像**  
。OpenWrt 默认用的是 Dropbear，只有手动替换成 OpenSSH 才受影响。  
  
**安全**  
：  
- 使用 glibc 的主流系统（Ubuntu、Debian、Fedora、RHEL）完全不受影响；  
  
- 启用 PAM 的系统、用 Dropbear 的系统、OpenBSD 不受影响。  
  
## 05 先自查：三步确认  
  
拿不准就花一分钟自查。三条命令，对应三个条件：  
```
# ① libc 是不是 musl？（glibc 直接排除）
ldd /usr/sbin/sshd | grep -i musl
# ② PAM 是否参与密码认证？（usepam yes 直接排除）
sshd -T 2>/dev/null | grep -i usepam
# ③ shadow 里有没有 * 锁定账户？
awk -F: '$2=="*"{print $1}' /etc/shadow
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNou0VQfQnlvibpAoWMBcXk0qAY9t5dhEGbVV7OTNrNqoxK5wa4FVYJDCWn7hibRQ9UPcib8aUMI8Ft4tuTWfdSy4h4hicZPm1KFDYbw/640?wx_fmt=png&from=appmsg "")  
  
图3 五步判定决策树：版本 → libc → PAM → 认证配置 → 锁定账户，任一步不满足即安全  
  
日志侧再补一眼：如果本应锁定、只允许密钥登录的账户（root、alpine）出现了 Accepted password  
 的成功记录，别犹豫，把这台机器当**已被入侵**  
处理，复核 doas/sudo 权限和有没有异常新建的账号。安全社区也放出了 musl-ssh-bypass-scan 这类开源扫描工具，可用于授权范围内的检测。  
## 06 修复与处置  
  
**第一步，升级**  
。Alpine 3.16 到 3.24 均已发布修复包（3.21.10、3.22.8、3.23.8、3.24.4 等）：  
```
apk upgrade -U -a
rc-service sshd restart
```  
  
**第二步，等不了就关密码认证**  
（最有效的避险手段）：  
```
# /etc/ssh/sshd_config
PasswordAuthentication no
```  
  
改完重启 sshd。暂时全走密钥登录，比什么都稳。  
  
如果已经在日志里看到"被锁账户 + Accepted password"的组合，那就不是补丁的事了：走失陷处置流程，验证它这段时间都做了什么。  
  
