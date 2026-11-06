#  速查！SSH认证绕过漏洞，无需密码直接ssh登录服务器  
网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-11-06 02:35  
  
2026 年 11 月 5 日，OpenSSH 项目组发布   
10.6p2 安全更新  
   
，修复了一个   
潜伏近十年的密码认证绕过漏洞  
   
（公开代号 sshhole）。在采用   
musl libc 且未启用 PAM  
   
的 Linux 系统中，攻击者可在无需掌握任何凭据的情况下，发送特定长度的密码请求，直接登录密码字段为   
*  
   
的本地账户。截至发稿，互联网上已出现   
3 个公开验证代码仓库  
   
，  
   
暂未监测到在野攻击事件  
   
。  
  
  
📌 本文看点  
  
01  
  
受影响系统与版本清单  
  
02  
  
公开 PoC 与在野利用现状  
  
03  
  
排查修复与应急处置  
  
01  
REPORT  
### 漏洞事件通报  
  
  
当地时间 2026 年 11 月 5 日，OpenSSH 官方发布   
10.6p2 紧急更新  
   
，披露并修复了一处针对 Linux 平台的高危密码认证绕过缺陷。该漏洞由安全公司 Wakefield 的研究人员 Özgün Kültekin 于 11 月 4 日上报，公开代号为 **sshhole**  
   
（亦作   
*sshole  
   
）。截至 11 月 6 日发稿时，  
   
MITRE 尚未为其分配正式 CVE 编号  
   
；国内安恒信息 CERT 将其收录为   
WM-202611-000001  
   
，CVSS 3.1 评分为 **9.8 分**  
   
（严重级别）。  
  
根据官方更新说明，当 OpenSSH 服务端（  
   
sshd  
   
）运行在使用   
musl libc C 标准库  
   
、且未启用可插拔认证模块（PAM）的 Linux 系统上时，  
   
远程攻击者无需知道目标账户的密码或私钥  
   
，只需在单次 SSH 连接中提交特定长度的字符串，即可绕过密码校验，直接登录   
/etc/shadow  
   
中密码字段为单个星号（  
   
*  
   
）的系统账户。此类账户在生产环境中通常被用于  
   
仅允许 SSH 公钥登录的运维账号、自动化部署账号以及云主机默认管理账号  
   
。  
  
OpenSSH 维护者 Damien Miller 在发布声明中提醒相关系统管理员，若当前运行环境属于使用 musl libc 且未启用 PAM 的 Linux 发行版（例如 Alpine Linux），应立即在配置中设置   
PasswordAuthentication no  
   
或临时关闭 SSH 服务，再执行版本升级。从漏洞上报到官方发布补丁，  
   
整个响应周期不足 48 小时  
   
。  
  
9.8  
  
CVSS 3.1 评分  
  
48 小时  
  
上报至补丁发布用时  
  
3 个  
  
公开 PoC 仓库数量  
  
  
02  
ANALYSIS  
### 漏洞技术原理  
  
  
源码审计结果表明，该漏洞  
   
并非缓冲区溢出或加密算法破解  
   
，而是由 Unix 影子文件规范、musl libc 错误返回值设计与 OpenSSH 防枚举机制三处历史代码交互引发的  
   
返回值状态混淆  
   
。  
  
在类 Unix 系统的   
/etc/shadow  
   
文件规范中，密码哈希字段置为单个   
*  
   
代表该账户禁止使用密码登录，正常情况下任何输入口令计算出的哈希值都不可能与单字符   
*  
   
相同。然而，2012 年 8 月 musl libc 为防止旧程序因未检查   
NULL  
   
指针而崩溃，在提交   
b3c4cc12  
   
中将   
crypt()  
   
函数的计算失败返回值统一设定为字符串   
*  
   
；同月，为防范超长口令引发拒绝服务（DoS），又在提交   
88bf5a8a  
   
中为 SHA-256 与 SHA-512 哈希算法设置了   
256 字节的口令长度上限  
   
（  
   
KEY_MAX 256  
   
）。  
  
以下为   
musl  
   
标准库   
src/crypt/crypt_sha512.c  
   
中的关键实现逻辑：  
  
.  
.  
.  
c  
  
[#define]()  
 KEY_MAX 256  
  
/* 拒绝超过 256 字节的超长口令 */  
  
for (i = 0; i <= KEY_MAX && key[i]; i++);  
  
　　if (i > KEY_MAX)  
  
　　　　return 0;  
  
/* 外层包装函数将计算失败统一折叠为字符串 "*" */  
  
p = sha512crypt(key, setting, output);  
  
q = sha512crypt(testkey, testsetting, testbuf);  
  
if (!p || q != testbuf || memcmp(testbuf, testhash, sizeof testhash))  
  
　　return "*";  
  
return p;  
  
2016 年，OpenSSH 为修复用户名枚举侧信道漏洞（  
   
CVE-2016-6210  
   
）引入了   
pick_salt()  
   
函数。当目标账户  
   
没有合法密码哈希  
   
（例如密码字段为   
*  
   
）时，  
   
sshd  
   
会从系统中其他拥有真实   
$5$  
   
或   
$6$  
   
哈希的本地账户处  
   
借用盐值  
   
（salt），调用   
xcrypt(password, salt)  
   
执行一次完整的哈希运算以拉平响应耗时。  
  
由于 OpenSSH 在   
auth-passwd.c  
   
中允许的最大口令长度为 1024 字节（  
   
MAX_PASSWORD_LEN 1024  
   
），当客户端提交长度为   
257 至 1024 字节  
   
的任意密码时，musl 的   
crypt()  
   
函数因触发超长限制而返回错误标记   
*  
   
。随后，  
   
sshd  
   
将该返回值与目标账户在   
/etc/shadow  
   
中存储的   
*  
   
执行字符串比对，判定密码匹配并放行登录。  
  
  
03  
SCOPE  
### 受影响与不受影响系统范围  
  
  
该漏洞对  
   
底层 C 标准库、认证框架及软件版本  
   
有严格的触发边界，并非所有 Linux 服务器均受波及。具体受影响与不受影响的系统及版本划分如下。  
  
一、受影响的系统类型与版本  
  
<table><thead><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">系统 / 组件类别</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">受影响的具体版本范围</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">受影响前提与场景说明</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">修复版本</span></section></th></tr></thead><tbody><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">OpenSSH 服务端</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">7.3p1 至 10.6p1（2016 年 7 月以来各版本）</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">编译时未启用 PAM（--without-pam）或配置为 UsePAM no，且底层链接 musl libc</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">10.6p2 及以上</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">Alpine Linux 发行版</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">3.16、3.17、3.18、3.19、3.20、3.21、3.22、3.23、3.24 及 edge 分支</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">官方 openssh-server 软件包默认不启用 PAM；自 3.16 版起默认 root 密码字段改为 *</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">官方已同步推送安全更新包（如 3.21.10、3.22.8、3.23.8、3.24.4 对应的 openssh 更新）</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">Alpine 官方云主机镜像</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">AWS、Azure 等云平台基于 Alpine 3.16 以上版本构建的官方镜像</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">镜像内置默认用户 alpine 的密码字段为 *，且默认配置免密 doas 权限，登录后可直接提权至 root</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">升级镜像内 openssh 包至 10.6p2-r0 及以上，或禁用密码认证</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">Alpine 基础容器镜像</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">基于 alpine:3.16 至 alpine:3.24 且自行安装运行 openssh-server 的 Docker / OCI 容器</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">官方基础镜像默认不含 sshd，仅在用户手动安装并启动 openssh-server 时受影响</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">重新拉取最新软件源执行 apk upgrade openssh 重建镜像</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">嵌入式与定制 musl 发行版</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">OpenWrt（各稳定版分支）、Buildroot、Yocto Project、Void Linux（musl 版）、Chimera Linux、Adélie Linux</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">系统未默认安装 dropbear 而是额外部署了未启用 PAM 的 openssh-server（低于 10.6p2），且存在 * 标记账户</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">升级 openssh 至 10.6p2 或打入上游 MIN_PASSWORD_HASH_LEN 补丁</span></section></td></tr></tbody></table>  
  
补充说明：在   
Alpine Linux 3.15  
   
及更早版本中，系统初始化默认将 root 账户的密码字段设置为感叹号   
!  
   
而非   
*  
   
，因此默认状态下的 root 账户无法直接触发该漏洞；但若管理员在该版本系统中创建了密码字段为   
*  
   
的普通运维账户或服务账户，该账户依然受影响。  
  
二、不受影响的系统类型与版本  
  
<table><thead><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">系统 / 组件类别</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">具体发行版与版本范围</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 6px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">不受影响的技术原因</span></section></th></tr></thead><tbody><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">基于 glibc / libxcrypt 的主流 Linux 发行版</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">Ubuntu（全版本，如 20.04 / 22.04 / 24.04）、Debian（10 / 11 / 12 等全版本）、RHEL / CentOS / Rocky Linux / AlmaLinux（7 / 8 / 9 等全版本）、Fedora、openSUSE / SLES、Arch Linux、国产龙蜥（Anolis OS）/ 欧拉（openEuler）等默认环境</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">glibc 与 libxcrypt 在 crypt() 计算失败时返回“*0”、“*1”或 NULL，与 /etc/shadow 中的单个“*”字符串不相等，无法通过 strcmp 校验</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">启用 PAM 认证的系统</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">任何将 sshd_config 配置为 UsePAM yes（且编译支持 PAM）的 Linux 系统，包括安装了 openssh-server-pam 并开启 PAM 的 Alpine 环境</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">开启 PAM 后，密码验证交由 PAM 模块（如 pam_unix.so）处理，不再走 OpenSSH 自身的 sys_auth_passwd() 与 xcrypt() 逻辑</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">非 Linux 操作系统</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">OpenBSD（全版本）、FreeBSD、NetBSD、macOS、Windows（含官方内置 OpenSSH Server）</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">底层未使用 musl libc；OpenBSD 原生系统采用 bcrypt 及专用认证机制，不存在“*”错误折叠行为</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">非 OpenSSH 服务端软件</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">OpenWrt 默认搭载的 Dropbear SSH、Golang x/crypto/ssh 自研服务端、libssh、tinyssh 等</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">未包含 OpenSSH 针对 CVE-2016-6210 引入的 pick_salt() 借盐逻辑</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">密码字段为 !、!! 或正常哈希的账户</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">使用 usermod -L 或 passwd -l 锁定的账户（前缀为 !）、已设置正常强密码的账户、系统中不存在的用户名</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">crypt() 返回的“*”无法与“!”、“!!”或长度超过数十字符的正常哈希值匹配；不存在的用户无真实“*”条目可供匹配</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><strong style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">已禁用密码认证的 OpenSSH</span></strong></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">OpenSSH 低于 10.6p2 但显式配置了 PasswordAuthentication no 的主机，或已升级至 OpenSSH 10.6p2 及以上版本的所有主机</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 6px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">服务端在进入密码校验函数前即拒绝密码认证请求，或已通过 10.6p2 新增的长度校验拦截</span></section></td></tr></tbody></table>  
  
  
04  
INTEL  
### 公开 PoC/EXP 与在野利用  
  
  
针对安全团队普遍关注的武器化进展，综合官方渠道、代码托管平台及主流漏洞库的检索结果，  
   
截至 2026 年 11 月 6 日  
   
情况如下：  
  
OpenSSH 官方更新日志  
   
仅披露了缺陷成因与补丁代码，未附带验证脚本  
   
。但在补丁发布当日（11 月 5 日），GitHub 平台已陆续出现 3 个第三方公开仓库：  
  
predkambrij/sshhole-repro  
  
自包含的 Bash 自动化复现工程。脚本通过 Docker 构建 Alpine 环境，利用   
SSH_ASKPASS  
   
配合   
ssh -o NumberOfPasswordPrompts=2  
   
自动完成两次密码提交并执行 id 命令验证权限。  
  
SkrudjReal/sshhole-exploit  
  
基于 Python   
paramiko  
   
库编写的  
   
远程验证与利用工具  
   
（PoC/EXP），支持漏洞状态检测（  
   
--check  
   
）、远程命令执行（  
   
--cmd  
   
）、交互式会话获取（  
   
--shell  
   
）及多用户名枚举尝试。  
  
leonjza/sshhole-lab  
  
提供用于本地复现测试的  
   
脆弱容器靶场环境  
   
，仓库本身不含攻击载荷。  
  
与此同时，Exploit-DB、ProjectDiscovery Nuclei 官方模板库以及 Metasploit 框架  
   
暂未收录针对该漏洞的检测插件或利用模块  
   
。在野利用方面，安恒信息 CERT 通告及海外安全社区跟踪数据均显示，目前暂未发现针对该漏洞的公开在野攻击或大规模扫描报告。不过，鉴于公开 EXP 已经具备  
   
完整的远程命令执行能力且触发门槛较低  
   
，未修复的公网暴露资产面临较高的潜在扫描风险。  
  
  
05  
EXPLOIT  
### 漏洞利用条件与触发细节  
  
  
在实际网络环境中，攻击者要成功触发该漏洞，  
   
目标主机必须同时满足以下条件  
   
：  
  
1  
操作系统为 Linux，动态链接库为   
musl libc  
，且 OpenSSH 版本低于   
10.6p2  
。  
  
2  
sshd 未启用 PAM（  
UsePAM no  
），且开启了密码认证（  
PasswordAuthentication yes  
，OpenSSH 默认开启）。  
  
3  
目标用户名在系统中真实存在，其在   
/etc/shadow  
 中的密码字段  
严格为单个星号  
，且未被   
DenyUsers  
、  
AllowUsers  
 等访问控制策略拦截（若目标为   
root  
，还需服务端配置   
PermitRootLogin yes  
，因为默认的   
prohibit-password  
 会直接拦截 root 的密码登录请求）。  
  
4  
目标系统的   
/etc/shadow  
 中  
至少存在另一个带有真实密码哈希的本地账户  
（  
$5$  
 表示 SHA-256、  
$6$  
 表示 SHA-512），以便   
pick_salt()  
 借用盐值触发 musl 的超长口令分支。  
  
在利用技术细节上，公开源码审计与插桩调试记录证实了两个关键约束。第一是  
   
口令长度窗口限制在 257 至 1024 字节之间  
   
：低于 257 字节无法触发 musl 的   
KEY_MAX  
   
报错返回   
*  
   
，超过 1024 字节则会被 OpenSSH 的   
MAX_PASSWORD_LEN  
   
提前丢弃，公开脚本多采用 300 字节填充字符。  
  
257 字节  
  
触发 KEY_MAX 的口令下限  
  
1024 字节  
  
MAX_PASSWORD_LEN 上限  
  
第二是  
   
必须在同一条 SSH 连接内发起第 2 次密码认证才能成功  
   
。原因是 OpenSSH 的   
shadow_pw()  
   
函数返回的是指向 musl 内部   
getspnam()  
   
静态缓冲区的指针；同一连接的第 1 次认证调用   
pick_salt()  
   
扫描账户表借用盐值时，  
   
会覆盖该静态缓冲区导致首次比对失败  
   
；第 2 次认证时   
pick_salt()  
   
直接读取已缓存的盐值而不再扫描文件，  
   
pw_password  
   
保持为   
*  
   
，比对得以通过（在标准 OpenSSH 客户端机制中对应同一连接内允许至少两次密码提示，即   
NumberOfPasswordPrompts=2  
   
）。  
  
！踩坑提示 🕳  
  
这也意味着，如果在安全评估中仅在单次连接内测试一次密码响应，将会得出不受影响的误判结果；防御方在配置检测规则或审计认证日志时，也往往能观察到同一会话内先出现一次密码认证失败、紧接着出现一次密码认证成功的典型序列特征。  
  
  
06  
FIX  
### 排查与修复建议  
  
  
一、主机受影响状态自查  
  
运维人员可在 Linux 主机终端依次执行以下命令，  
   
核查当前环境是否落入受影响范围  
   
：  
  
bash  
  
ldd --version 2>&1 | head -1  
  
sshd -V 2>&1 | head -1  
  
sshd -T 2>/dev/null | grep -Ei '^(usepam|passwordauthentication|permitrootlogin)'  
  
awk -F: '$2 == "*" {print "star_account:" $1}' /etc/shadow  
  
awk -F: '$2 ~ /^\$[56]\$/ {print "salt_donor:" $1}' /etc/shadow  
  
💡 判定标准：ldd 输出包含 musl、OpenSSH 版本低于 10.6p2、usepam 为 no、passwordauthentication 为 yes，且后两条命令同时输出了星号账户与 $5$ / $6$ 账户，四项同时成立即判定该主机受影响，需立即处置。  
  
二、入侵痕迹日志排查  
  
建议同步检索系统认证日志，  
   
排查是否存在密码字段为星号的账户通过密码方式成功登录的异常记录  
   
：  
  
bash  
  
grep -Ei 'Accepted password' /var/log/messages /var/log/auth.log 2>/dev/null  
  
按照 Alpine 安全团队给出的应急口径，如果  
   
原本仅配置密钥登录的无密码账户  
   
（如   
alpine  
   
、  
   
deploy  
   
或   
root  
   
）在日志中出现   
Accepted password  
   
记录，应按  
   
主机已被未授权访问启动应急响应  
   
，全面核查新增系统账户、  
   
~/.ssh/authorized_keys  
   
变更、  
   
doas  
   
或   
sudo  
   
提权记录及异常外联进程。  
  
三、官方补丁升级与缓解措施  
  
OpenSSH 10.6p2 在   
auth-passwd.c  
   
中新增了  
   
最小密码哈希长度宏  
   
MIN_PASSWORD_HASH_LEN 4  
   
，在执行   
strcmp  
   
前强制校验   
strlen(pw_password) >= 4  
   
，使长度为 1 的   
*  
   
在比对前即被拒绝：  
  
.  
.  
.  
c  
  
[#define]()  
 MIN_PASSWORD_HASH_LEN　4  
  
[#define]()  
 MAX_PASSWORD_LEN　1024  
  
ret = encrypted_password != NULL &&  
  
　　strlen(pw_password) >= MIN_PASSWORD_HASH_LEN &&  
  
　　strcmp(encrypted_password, pw_password) == 0;  
  
建议受影响用户优先采取以下修复措施：  
  
STEP 01  
升级软件包（推荐）  
  
Alpine Linux 用户可直接通过官方源升级并重启服务：  
  
bash  
  
apk update && apk upgrade openssh  
  
rc-service sshd restart  
  
采用源码编译部署的环境，应下载 OpenSSH 10.6p2 官方源码包，保留原有的   
--with-shadow  
   
等构建参数重新编译安装。  
  
STEP 02  
临时缓解措施  
  
对于暂不具备升级条件的业务节点，应在  
   
确认 SSH 公钥登录正常可用且留有控制台带外管理通道  
   
的前提下，显式关闭密码认证：  
  
bash  
  
printf 'PasswordAuthentication no\nKbdInteractiveAuthentication no\nPermitRootLogin prohibit-password\n' > /etc/ssh/sshd_config.d/00-sshhole-mitigation.conf  
  
sshd -t && rc-service sshd restart  
  
STEP 03  
操作注意事项  
  
在未启用 PAM 的 Alpine Linux 环境中，  
 **切勿通过将 /etc/shadow 中的星号手工修改为感叹号来规避本漏洞**  
   
。在无 PAM 模式下，OpenSSH 会将密码字段前缀为感叹号的账户判定为全局锁定，从而导致正常的 SSH 公钥登录一并失效。正确的处置方式始终是升级至 10.6p2 或关闭密码认证。  
  
END  
  
我是 网络安全透视镜  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
