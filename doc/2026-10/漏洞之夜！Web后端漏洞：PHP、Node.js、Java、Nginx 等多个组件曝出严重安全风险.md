#  漏洞之夜！Web后端漏洞：PHP、Node.js、Java、Nginx 等多个组件曝出严重安全风险  
点击关注 👉
                    点击关注 👉  马哥网络安全   2026-10-22 09:00  
  
完整报告PPT下载：  
  
https://deerc.cloud/api/v1/public/share/debab3b2-c80a-4ebe-b75e-ba3130473e6c.pptx  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRAV1M6ibE7VLwLZNtAlZB6gnTUicXoe7DOAbJZseokMaGyeNnWBvcaic80BfXtjMqhfEIkDUxCgnx0wuZ1KvoSnhxmv1zYaIyQ10g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0 "")  
  
近期，多个主流 Web 后端运行时、开发框架及基础设施组件接连披露严重安全漏洞，涉及 PHP、Node.js、Java、ASP.NET Core、Ruby on Rails、Nginx、Apache HTTP Server 及数据库相关组件。  
  
其中，部分漏洞允许未经身份验证的远程攻击者执行任意代码，部分可能绕过身份认证或访问控制，还有一些已经出现公开的在野利用证据。  
  
对于仍在运行旧版本组件的互联网应用、企业内部系统以及云端服务而言，这些漏洞值得重点关注。  
  
本文基于截至 **2026年10月21日**  
 的公开安全公告，对近期披露的高危漏洞及仍有修复价值的历史严重漏洞进行整理，重点介绍漏洞影响、触发条件与修复建议。  
## 一、PHP：两项严重漏洞可能导致未认证远程代码执行  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRCict8OopSUx5fTFyZwiciaxCQu8xD2RjCqAKNQtv5mmRvDbStV6x2cnibmYwu0xdqFuw7w6bkHGUgicDMcyTUdxl1R4QvVRjAMFF2c/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1 "")  
## 二、Node.js：Brotli 内存越界写入可能引发 RCE  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRBHib7s6Q3nY44mY7FJDfEkbyxa7NUib9IViamOqWtjcDeYOIou4oIezSW92okjqzZBywUpRvVPcuenacib3CuQXzpf0Ttwz0mre8Q/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2 "")  
## 三、Java：Jetty、Tomcat、Spring 和 Apache MINA 均存在严重风险  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRAtJ5typsDl6e4y8sl6s7CSNHCSlKuapp3xFESOwaDARuhXK4mt0XEkUMkD982iaAaK6681tZWiaEricaRgAAOCk6UV4syibF0vYUc/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRDD3m1EumvjSbx6AnvOfh8ou0jQUlT9Z9J4LYuaCu5Ffgp47AmCHd8GOPicIexX4nwfEhnl2POoEboIIb7FtIVj0yAplzGfRwF8/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRAliawrTnibxAqR8XjsRYtdMSOqicPSjic9UNB9gmXrKzyWyeoWyKZ38y2urD4XX44weDON8OLKL0Z2wOl2iawg7DVczQDEwWTzCR3Q/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRCZUdT2w94jaicxHLibfqJ46fxtyDFeJmz2jQvEdodExwOZte3hS0XRoheZqA9ufwxP4fFR5SNhszyboFx47WlYd3jIf9SrdpI6g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6 "")  
## 四、ASP.NET Core：Kestrel 请求走私与安全机制绕过  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRAn5VUXLic0jIxjhIQYyrjGv5OfH5NLDPJShWjaBXzEhiaESA1HukKDM7kQoiaryicLqAgSfHCy4IoQwwXqY6yCQKk07ia020YT7P4w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=7 "")  
## 五、Nginx & Apache HTTP Server：多个内存安全漏洞 & HTTP/2 Double-Free 漏洞值得重点关注  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRClRlkogDNvmiaeqWHict2bH5MBOeOeeLzneLr9vHVcz4tfv8VUBDam4b3p9bXmK9icRoHt7DD5m4sic6z68YLN8ibPbca8ibwNGcOicg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=8 "")  
## 六、：Ruby on Rails：图片处理功能可能泄露服务器敏感文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRBvkibv4XIZAiaYmg9cvfmtSCicwIoqmqjLDfzfxNDeibNk24iaUoRV6nicuPX6bjd1HibSaia36AibO6Ie38qT5icXibDbxRdEwrib7vmlnCQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=9 "")  
## 七、Python：Starlette 特定场景存在授权绕过风险  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRAUrdQpicXib1rg5fnzvE55HcCibwn82mqP3IKQS5ZG6C6TeUUer9sK8bpibS9TFEVoAwaut2urXtBkmJbyJgnTCicYxIUw1qYeleb8/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=10 "")  
## 八、Oracle WebLogic 前置代理：访问控制绕过已进入 KEV  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRDXEtkmhE1JEwNYBDsb8M1ZCN4ZMibeqYu6TgpVvA9KV1mzwricTXheLVq2kPwqGd9wEgDrSSJQGhibJTpic0FwKGMdjZsDiaUhGfhU/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11 "")  
## 九、企业应该优先修复哪些漏洞？  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/fnnfOjwAzRAGvIKYiaPQChSsZ1ianlb2Hryj4cR7PahGibg81XMUwcux5Ca5xgXtr02icrRPn91k8VCXYO08Hhw2qxficdHO2tUg73fMxh2meRJc/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=12 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fnnfOjwAzRBoVx1P72vR4QgbytxHFV0TeuLJlp7icSJqibwr90XD03fmia2pWDrfibGXCGPu9KoCpxVnfqofibpgrEwzicIAzqWgfJHh6RkUicIOfI/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=13 "")  
  
## 十、真正的风险往往隐藏在底层组件中  
  
从本轮漏洞可以看到，Web 后端的安全风险并不只存在于业务代码。  
  
运行时的内存管理、Web 容器的安全约束、反向代理的协议解析、框架的反序列化机制，以及第三方依赖，都可能成为攻击入口。  
  
对于企业而言，安全治理不能只关注业务开发阶段的代码审计。  
  
更重要的是建立持续性的资产识别、依赖管理、漏洞监测和修复闭环。  
  
**一个业务系统即使没有明显的代码漏洞，也可能因为底层运行环境长期未更新，而暴露在严重安全风险之中。**  
  
对于仍在运行旧版本 PHP、Node.js、Java Web 容器或 Nginx 的服务，建议尽快开展一次版本和配置排查。  
  
安全漏洞并不可怕，真正需要警惕的是：漏洞已经公开，生产环境却仍然不知道自己是否受到影响。  
##   
## 参考资料  
  
本文基于截至2026年10月21日的公开安全公告整理。由于漏洞公告和修复版本可能持续更新，具体影响范围请以供应商官方记录为准。  
  
PHP 官方：https://www.php.net/  
  
Node.js 安全公告：https://nodejs.org/en/blog/vulnerability/  
  
Eclipse Jetty：https://jetty.org/security.html  
  
Apache Tomcat：https://tomcat.apache.org/security-10.html  
  
Apache MINA：https://mina.apache.org/  
  
Nginx：https://nginx.org/en/security_advisories.html  
  
Apache HTTP Server：https://httpd.apache.org/security/vulnerabilities_24.html  
  
Oracle Critical Patch Update：https://www.oracle.com/security-alerts/cpuoct2026.html  
  
CVE 官方数据库：https://www.cve.org/  
  
NVD 漏洞数据库：https://nvd.nist.gov/  
  
CISA 已知遭利用漏洞目录：https://www.cisa.gov/known-exploited-vulnerabilities-catalog  
  
**说明：**  
 本文并非全球 Web 后端历史 CVE 的完整清单。未列出某个漏洞不代表该组件安全，也不代表所有列出的漏洞都已遭利用。漏洞等级、利用条件和修复状态应以对应官方公告为准。  
  
内容转自Deer Agent，侵删  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/2I159AwKj57Igp6433XwH04fh0vYWiaftlzqNnyxkcIpMeib6q1oJ7EMBBm1GaAUgmYLR3UaTzp1tbfcbc2u7tutFL4ySxHiaAt07FAUsaRG7A/640?wx_fmt=jpeg&from=appmsg "")  
  
  
