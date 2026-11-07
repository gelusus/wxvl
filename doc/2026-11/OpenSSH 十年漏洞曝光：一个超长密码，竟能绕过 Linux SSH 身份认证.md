#  OpenSSH 十年漏洞曝光：一个超长密码，竟能绕过 Linux SSH 身份认证  
原创 🅰🅸
                    🅰🅸  独眼情报   2026-11-07 03:57  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cBGhzWwhSAjm12QUaLkTgypdgS4YHvYp6qtAXjAhV8ACIk8hDTdhWr0EDVnBibDGHkYictV2IcC3iasHeW0gaLiaD4wcygKbLBKgIfT9URFFPMI/640?wx_fmt=webp&from=appmsg "")  
  
2026年11月5日，OpenSSH 紧急发布 10.6p2 版本，修复一个潜伏近十年的身份认证绕过漏洞，编号为 CVE-2026-123158，漏洞名称为「sshole」。  
  
  
  
该漏洞主要影响使用 musl libc、未启用 PAM 的 Linux 系统，尤其是 Alpine Linux。攻击者只需要向 SSH 服务提交一个超过256字节的密码，就可能绕过原本禁止密码登录的账户认证，在特定配置下甚至能够直接获得服务器 root 权限。  
  
这不是传统意义上的密码破解，而是认证程序将「密码计算失败」错误地当成了「密码验证成功」。  
## 一、漏洞是如何产生的？  
  
此次漏洞涉及三个关键组件：musl libc、OpenSSH 和 Linux 用户账户配置。  
  
首先，musl libc 是一种轻量级 C 标准库，被 Alpine Linux 等系统广泛采用。Linux 通常通过 crypt()  
 函数计算用户输入密码的哈希值，再与 /etc/shadow  
 中保存的密码哈希进行比较。  
  
musl 对密码长度设置了256字节的限制。当输入密码超过这一长度时，crypt()  
 会拒绝计算，并返回特殊字符串：  
```
*

```  
  
问题在于，Linux 的 /etc/shadow  
 中，*  
 本身也被用作无有效密码哈希的标记，通常用于禁止账户通过密码认证。  
  
也就是说，musl 返回的错误标记，恰好与部分 Linux 账户保存的密码字段相同。  
  
第二个问题来自 OpenSSH。  
  
2016年，OpenSSH 为修复 CVE-2016-6210 用户名枚举漏洞，调整了密码验证逻辑。对于没有有效密码哈希的账户，OpenSSH 会借用其他账户的密码哈希盐值执行计算，以减少不同账户之间的认证时间差异。  
  
这一机制原本是为了防止账户信息泄露，却意外为后续的认证绕过创造了条件。  
  
当攻击者提交超过256字节的密码时，musl 返回 *  
，而目标账户的密码字段同样是 *  
。在特定认证路径下，两者发生匹配，最终导致身份认证绕过。  
## 二、攻击者如何获得服务器权限？  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cBGhzWwhSAjznd3h8x7e4DbPVUOEueic3CyLT9Dh6HmVbH6iaSYch6kI8hGH4SlPLjInyWOHmIYBQnk0bXgCwWOSZ0VYOxuAWcCiaTcrL6kpr8/640?wx_fmt=webp&from=appmsg "")  
  
整个攻击过程不需要破解密码，也不需要复杂的漏洞利用链。  
  
攻击者只需要能够访问目标 SSH 服务，并知道一个符合条件的账户名称。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/cBGhzWwhSAgib90mDicxYBA3WQlF0oSefVjTQ2gC6iaWBicDXsTXpTtUB9XdPYu3CpYic6GV6u0OL4WWxNjsx1cajulyK9d349lBWrGAsbfWxwK0/640?wx_fmt=webp&from=appmsg "")  
  
攻击流程如下：  
1. 攻击者连接目标 Linux 服务器的 SSH 端口。  
  
1. 选择一个密码字段为 *  
 的账户进行登录。  
  
1. 提交超过256字节的密码。  
  
1. musl libc 的 crypt()  
 拒绝计算，并返回 *  
。  
  
1. OpenSSH 在特定认证逻辑下，将返回值与账户密码字段进行比较。  
  
1. 两者匹配，SSH 认证被错误地判定为成功。  
  
攻击者由此获得目标账户的 SSH 会话。  
  
值得注意的是，攻击者不需要知道账户真实密码，也不需要反复尝试。满足条件时，一次连接就可能完成认证绕过。  
## 三、为什么 Alpine Linux 风险更高？  
  
这次漏洞并不是所有 Linux 系统都受到影响，是否存在风险取决于多个配置条件。  
  
主要需要同时满足：  
- 系统使用 musl libc，且 SSH 密码认证没有通过 PAM 处理。  
  
- OpenSSH 开启 PasswordAuthentication  
。  
  
- 至少有一个账户的 /etc/shadow  
 密码字段为 *  
。  
  
- 系统中至少还有一个账户使用 $5$  
 或 $6$  
 开头的密码哈希。  
  
其中，Alpine Linux 是最值得关注的目标。  
  
Alpine 被广泛用于 Docker 基础镜像、云服务器、嵌入式设备和轻量级 Linux 环境。其官方云镜像还存在一个值得警惕的账户配置：部分 tiny-cloud 镜像包含默认 alpine  
 账户，该账户密码字段为 *  
，同时具有免密 doas  
 权限。  
  
当管理员设置控制台密码，或者通过用户数据创建带密码哈希的新账户后，就可能满足漏洞利用条件。  
  
这意味着攻击者可以先利用漏洞登录 alpine  
 账户，再通过免密 doas  
 提升至 root。  
  
攻击链就变成了：  
  
SSH 认证绕过 → 登录 alpine 账户 → 免密 doas → 获取 root 权限。  
  
需要注意，Alpine 镜像并非首次启动就必然存在漏洞，是否可利用取决于后续账户配置。  
  
此外，OpenWrt 默认使用 Dropbear 提供 SSH 服务，并不直接受到此次 OpenSSH 漏洞影响。只有额外安装 OpenSSH 且满足相关条件的设备才需要排查。  
## 四、漏洞为何潜伏近十年？  
  
此次事件最值得关注的是漏洞的形成时间。  
  
2012年，musl 引入了相关密码长度限制和错误返回行为；2016年，OpenSSH 为修复用户名枚举问题调整认证逻辑；直到2026年11月，研究人员才发现这些机制组合后可能造成身份认证绕过。  
  
这说明漏洞并非简单来自某个组件的单独缺陷，而是多个软件在不同时间做出的设计决策发生了意外冲突。  
  
musl 的错误返回值本身有明确用途，OpenSSH 的历史修改也有安全目的，但两个组件之间缺少对异常状态的充分验证。  
  
从软件供应链安全角度看，这类跨组件漏洞尤其难以发现。单独检查 OpenSSH、musl libc 或 Linux 账户配置，都未必能够发现问题。  
## 五、修复方案与安全建议  
  
OpenSSH 已于2026年11月5日发布 10.6p2，修复这一认证绕过问题。Alpine Linux 也同步发布了多个分支的安全更新。  
  
Alpine 管理员可以执行：  
```
apk upgrade -U -a

```  
  
更新完成后重启 SSH 服务：  
```
rc-service sshd restart

```  
  
如果暂时无法更新，可以先关闭密码认证：  
```
PasswordAuthentication no

```  
  
随后检查配置并重启 SSH 服务。  
  
安全团队还应重点检查以下内容：  
1. 排查所有使用 Alpine Linux 的云服务器和容器。  
  
1. 检查 SSH 是否启用了密码认证。  
  
1. 检查 /etc/shadow  
 中是否存在 *  
 密码字段。  
  
1. 检查是否存在异常 SSH 登录记录，特别是原本只允许公钥登录的账户。  
  
1. 排查 doas  
、sudo 权限以及近期新增账户。  
  
1. 如果无法排除攻击者已经成功登录，应进一步检查主机是否失陷，并评估凭据轮换需求。  
  
## 六、事件研判  
  
sshole 的危险之处，在于攻击门槛低、利用过程简单，而且可能直接导致服务器完全失陷。  
  
对于企业而言，最值得优先排查的不是所有 Linux 服务器，而是使用 musl libc、运行 OpenSSH Server、开启密码认证的 Alpine 云实例和嵌入式设备。  
  
目前公开披露的信息尚未证明该漏洞已经遭到大规模实际利用，但其利用条件明确，自动化检测和攻击实现成本较低，公开披露后应重点关注相关 SSH 服务的异常登录活动。  
  
这起事件再次说明，身份认证系统不仅需要验证密码是否正确，更必须严格区分验证失败、计算异常和认证成功。  
  
一个看似无害的错误返回值，在跨组件交互中可能变成绕过整个身份认证体系的入口。  
  
  
