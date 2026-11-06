#  【已复现】OpenSSH musl libc 无密码账户认证绕过漏洞  
源影网安
                    源影网安  源影安全团队   2026-11-06 00:25  
  
OpenSSH 在 musl libc 生态（Alpine、OpenWrt、嵌入式固件）上普遍以无 PAM 方式构建，此时 sshd 的密码验证直接走 libc 的 crypt()：将候选口令哈希后与 shadow 字段做比对，目标账户密码字段过短无法充当盐时（典型即无密码账户的哨兵值 *），sshd 会遍历本地账户借用第一个 $5$/$6$ 哈希的盐。  
  
本次漏洞正是这组机制的碰撞：musl 的 crypt() 对超过 256 字节的密钥返回错误指示符 "*"，恰好与无密码账户的存储字段 "*" 一致——攻击者提交任意 ≥257 字节的口令，sshd 哈希后得到 "*"，与账户字段命中，一次连接即登录成功，无需任何重试。前提是机器得"凑齐"四个条件：Alpine 这类 musl 小众系统、开着密码登录（默认就开）、机器上有人设过密码、目标是个本来就没密码的账号——主流 Ubuntu/Debian 服务器凑不齐，基本免疫；但 Alpine 云主机和嵌入式设备出厂就自带前三条，最危险。官方在 10.6p2 于 sshd 侧规避：存储字段长度不足 4 字符不参与比较。  
  
我们在本地 Docker 靶机（Alpine 3.21.10 源码编译 OpenSSH 10.6p1，10.6p2 作对照）完成了复现：对密码字段为 * 的账户提交 300 字节口令，单次连接登录成功；构造 uid=0 的同类型账户后直接以 root 身份进入；边界测试 256 字节拒绝、257 字节起稳定成功，与 musl crypt() 阈值精确吻合；修复版全长度拒绝。该机制自 2016 年即存在，升级 OpenSSH ≥10.6p2 并重启 sshd，临时缓解为 PasswordAuthentication no。  
  
复现截图：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/gRYowTXTvLHgJoHBzGzKDuDImKrT75QXhBNQFdAOGPHCt8ddyvGeyTcqFcJbKz6wWElYwvM8BCLoDPjPbQNHJOCuoTSUNpcxP7aXDNZ7yJQ/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
  
  
