#  WAF 拦截 + Java 11 强防御？Log4j2 绝境下的 Bypass 与 RCE  
原创 暗夜铭少
                    暗夜铭少  黑帽渗透技术   2026-10-31 23:41  
  
Log4j2  
JNDI注入  
WAF Bypass  
内存马  
提权  
付费合集 · 第 10 篇  
  
如果要在近五年的安全圈选一个“核弹级”漏洞，Log4j2（CVE-2021-44228）绝对榜上有名。  
  
网上关于它的教程铺天盖地，但大多是“搭个靶场，发个 ${jndi:ldap://...}  
，弹个计算器”就结束了。  
  
如果你把这种思路带到真实的生产环境中，大概率会撞得头破血流。今天这个靶场，我们直接把难度拉满：**Ubuntu 20.04 + Tomcat 9 + Log4j2 2.14.1，外加一层拦截 ${jndi: 特征的 WAF，目标机器还有出网限制（只允许 80/443），且运行在 Java 11 高版本下（默认禁止远程加载恶意类）。**  
  
在这种“地狱难度”下，如何打进内网，并最终拿下 Root？  
  
![](https://mmbiz.qpic.cn/mmbiz_png/VqJrJwib7Fo0KlMMhUkrtO2ibZ6SwA0oG081qOXcQGmZFKTV3vw8Pd85oZk4wiajmib0KrS7lJcgppRXG0ia0tWgsNfq8lHXauMPfHmv33qw3xZ8/640?wx_fmt=png&from=appmsg "")  
  
