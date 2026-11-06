#  OpenSSH 密码认证绕过漏洞，来看看你会被插进去吗  
原创 喵星不安全
                    喵星不安全  喵星安全研究所   2026-11-06 03:31  
  
先说总结再看原理，这个洞最骚的地方不是技术多花，而是它专挑你以为已经锁好的门下手，257 字节乱码一塞，直接插进去  
### 一、一句话总结  
  
**glibc / PAM / 已关密码登录 / 不跑 sshd 的，可以安睡**  
**musl + 无 PAM + 密码认证开着 + 有 * 账户的，爬起来修**  
  
二、原理到底是啥  
  
对于密码字段是 * 的无密码账户，sshd 会拿用户输入的密码去算哈希；而musl libc的crypt()遇到超过256字节的密码时，会返回 * 表示失败。这个 * 恰好和账户里存的 * 完全一样，sshd 就误判“密码正确”，放行登录，懂了吗，臭弟弟  
### 三、它到底在干什么  
  
简单说：  
  
对那些密码字段是 *  
 的无密码账户，sshd 会拿用户输入去算哈希。  
  
musl 的 crypt()  
 碰到 **257 字节以上**  
的密码，会直接返回一个 *  
。  
  
巧的是，这个 *  
 正好和账户里存的 *  
 一模一样。  
  
于是 sshd 就误以为：“嗯，对上了，进来吧。”  
  
不是暴力破解，也不是内存溢出。  
  
更像是几层“合理设计”叠在一起，最后把门缝挤开了。  
### 四、谁能被它插进去  
```
必须同时满足这几条：

1）没开 PAM，或者配了 UsePAM no
2）PasswordAuthentication yes
3）存在密码字段为*的无密码账户；
4）系统里至少有一个 $5$ 或 $6$ 的真实密码哈希。
```  
  
Alpine 最容易中招，尤其是 tiny-cloud 镜像里的alpine用户；OpenWrt、Buildroot、Yocto 这些 musl 环境也要查；Debian、Ubuntu、RHEL 这类 glibc 系统，以及用 PAM 校验密码的，基本不受影响。  
### 五、怎么防止插进来  
  
最稳的是升级  
```
apk upgrade -U -a
rc-service sshd restart
```  
  
来不及升级，就先关密码登录  
```
PasswordAuthentication no
```  
  
改完用sshd -t 检查检查，再重启，别急着断当前会话，小心把自己锁外面![](https://mmbiz.qpic.cn/sz_mmbiz_png/YstVCVhPEsAGy6WfktEiaOsWw1vgbK2Zfcvs1hN8g2onWEwUGYZeicqRGXiaBe3DD6BsfowsicReicUiaucjmqkIwBgyxQMmicWNialAFW17edjmVicQ/640?wx_fmt=png&from=appmsg "")  
  
### 六、怎么判断是不是第一次  
  
去看看 /var/log/messages  
，重点找  
```
Accepted password for
```  
  
先看看root  
、alpine  
、部署账户、仅密钥账户，再顺手摸一摸doas记录、新建用户、文件变动、授权密钥。如果有痕迹，就按开口  
处理。  
### 七、送给运营一句话  
  
一是分清 *  
 账户和 !  
 账户；!  
 不会触发这个洞。  
  
二是把“无 PAM + 密码认证开启”单独列为风险项，不能只看有没有开密码登录。先把手里 Alpine 和 musl 环境的 sshd 过一遍。  
  
别等别人拿着 257 字节法棍，“轻轻撬开你的门”  
  
