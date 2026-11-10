#  Tenda AC20（≤ V16.03.08.12）路由器缓冲区溢出漏洞分析  
秋名山上的小柠
                    秋名山上的小柠  蚁景网络安全   2026-11-10 09:39  
  
## 1.前言  
  
在对 Tenda AC20 路由器 进行安全分析时，发现其固件在处理特定输入时存在 缓冲区溢出漏洞。  
  
该漏洞源于程序在拷贝用户输入时缺乏有效的边界检查，攻击者可以通过构造恶意请求触发溢出，从而导致系统崩溃，甚至在某些场景下获得更高权限，进而完全控制设备  
  
经过多次验证与测试，确认该漏洞风险较高，对设备的安全性影响严重。  
  
我整理了一份详细的技术报告，内容包括漏洞成因、复现方法及修复建议，并通过正规渠道提交给厂商及 CVE 分配机构。  
  
该漏洞已被正式收录，编号为   
CVE-2025-8939  
## 2.漏洞概述  
  
Tenda AC20 路由器被发现存在缓冲区溢出漏洞。攻击者可以通过向某些路径发送特制的 HTTP POST 请求来触发此漏洞，从而可能导致拒绝服务 (DoS) 攻击，甚至远程代码执行 (RCE)  
## 3.漏洞细节  
  
AC20 路由器的最新固件可从腾达官网下载：AC20 升级软件 -   
腾达官网  
  
固件可以使用以下在线工具解压：https://zhiwanyuzhou.com/multiple_analyse/firmware/  
  
或者直接使用binwalk解包也是可以的，获得以下文件  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexxCWyP0JibZWB3qJvlw3Eprv13OX5roYvSsiboNK27toMia6l0WXnQtGnA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "null")  
  
  
我们要找到/squashfs-root/bin/httpd 二进制文件  
  
因为httpd就是Tenda 路由器的 Web 管理后台进程，大部分家用路由器，包括 Tenda都提供一个 Web 管理界面，这个界面其实就是由路由器内部的一个小型 Web 服务器httpd提供的  
  
当用户在浏览器里访问路由器后台时，请求会被发送到路由器本地运行的 httpd 服务，这个二进制文件负责接收、解析 HTTP 请求，比如说登录、修改 Wi-Fi 密码、固件升级等，然后调用底层的系统函数或配置接口  
  
由于 httpd 要解析用户提交的数据，如果代码没有做好边界检查，就可能导致缓冲区溢出、命令注入等问题，这也是为什么路由器的很多漏洞，很常见的溢出、注入、未授权访问都集中在 httpd 这个二进制文件里  
  
解包文件里面还有dhttpd二进制文件，它与httpd最大的不同就在于httpd是Tenda 的主后台 Web 管理进程，就是能够对用户可见的路由器后台，而dhttpd则会用来跑一些非核心但需要 Web 接口的功能，像什么诊断、子模块或者是其他特定功能  
  
在Tenda AC 系列里，Web 管理界面并没有用第三方服务器，比如lighttpd，或者是boa之类的，而是厂商自己写的 httpd 二进制文件  
  
那么，所有 HTTP 请求直接由这个 httpd 处理，路由器后台的逻辑，像什么 Wi-Fi 配置、系统管理之类的也都在里面实现，所以漏洞就会出现在 httpd 本身  
  
换句话说，Tenda 的 httpd 本身就是 Web 服务 + 业务逻辑的二合一  
  
回到正题，我们在httpd文件中发现函数fromSetWifiGusetBasic有缓冲区溢出的风险  
  
可以看到这里的函数fromSetWifiGusetBasic，该函数会获取shareSpeed的值，然后将其复制到Var数组中，且没有进行长度检查，从而导致缓冲区溢出漏洞  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfex9GCokQIfnzibkraJ28wT9ricLrPEOGI552hyMVndVqjJG5iarEm5Xgib8g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "null")  
  
  
交叉引用后再往上翻就会发现它其实是有个前提条件的，那就是请求路径必须是WifiGuestSet  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexWsKLP7OIeiaJdL0PJsWIibUdibSEynHIxydFAnwcAuibKob30CbLxIzbQw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "null")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexfA0Wyd6WIW12r759icvmt2HJD2UczDyH9QnGUOxEFdSCP5BqD1q5zfw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3 "null")  
  
  
这行代码的作用，是把 WifiGuestSet 这个字符串与具体的处理函数 fromSetWifiGuestBasic 绑定在一起。换句话说，只要有请求命中 WifiGuestSet，设备就会调用对应的函数去执行  
  
所以我们Poc的请求路径就应该是POST /goform/WifiGusetSet HTTP/1.1  
  
路径前加 /goform/  
 是 Tenda 固件的惯用套路，结合代码逻辑，基本可以确认  
## 4.漏洞验证  
  
确定漏洞点之后，我们先把固件跑起来，模拟一个真实运行环境，更好让我们观察是否因为栈溢出而产生崩溃页面  
  
找到 /bin/httpd 文件。要模拟环境，使用以下命令：  
```
sudo chroot ./ ./qemu-mipsel-static ./bin/httpd
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexvcByMaKdaibgE8ErricSFJhs70I9QkCDDRCgJ6ibB7Lx4PicGtazyxXOmQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "null")  
  
  
等一会之后，直接上浏览器输入对应IP地址 192.168.102.145，就可以进入到AC20路由器页面  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexLwWvD9Hong8DhALMtI1icjKFt9mUiaiaNQdqBe4Ogd0pP2P5GZn4IA39A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5 "null")  
  
  
跑起来之后，我们使用burpsuite进行一个发包测试，发送一堆垃圾数据到sharedSpeed  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexjwia3EeEbY97sn1b6TXLol4gD1ic7DXPwDBTOE2Qmtibc8TdwLdoaa3zw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "null")  
  
  
可以发现192.168.102.145/main.html出现了崩溃信息  
  
而从终端也观察到分段错误，确认发生了栈溢出，也就是shareSpeed这里存在一个缓冲区错误  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwJOPK8icPueknNzqphZhfexWt74Bopu6Sv6biaTl8kibnicO6oL2LyWexedOViaWxR3G0soiclxiaias6nrw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7 "null")  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzkxNTIwNTkyNg==&mid=2247549615&idx=1&sn=5de0fec4a85adc4c45c6864eec2c5c56&scene=21#wechat_redirect)  
  
  
