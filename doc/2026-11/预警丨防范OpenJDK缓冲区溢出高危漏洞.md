#  预警丨防范OpenJDK缓冲区溢出高危漏洞  
 网络安全和信息化   2026-11-23 09:10  
  
近日，工业和信息化部网络安全威胁和漏洞信息共享平台（  
NVDB  
）监测发现，  
OpenJDK  
存在缓冲区溢出高危漏洞，危害严重。  
  
OpenJDK  
是  
Java  
平台标准版（  
Java SE  
）的开源实现，由甲骨文（  
Oracle  
）公司主导维护，  
Oracle Java SE  
、  
Oracle GraalVM  
等产品均基于其源代码构建。由于其  
Base64  
解码器存在堆缓冲区溢出缺陷，攻击者可通过发送恶意数据损坏内存，造成内存敏感信息泄露或远程执行任意代码。受影响产品包括  
OpenJDK  
及  
Oracle Java SE  
、  
Oracle GraalVM for JDK  
、  
Oracle GraalVM Enterprise Edition  
、  
Red Hat OpenJDK  
、  
Amazon Corretto  
等基于其源代码构建的发行版。  
  
目前，  
OpenJDK  
社区及部分发行版厂商已修复漏洞并发布安全公告，参考链接如下：  
  
OpenJDK  
：  
https://openjdk.org/groups/vulnerability/advisories/2026-11-17  
；  
  
Oracle  
相关产品：  
https://www.oracle.com/security-alerts/cspunov2026.html  
；  
  
Red Hat OpenJDK  
：  
https://access.redhat.com/security/vulnerabilities/RHSB-2026-018  
；  
  
Amazon Corretto  
：  
  
1  
）  
https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/what-is-corretto-17.html  
；  
  
2  
）  
https://docs.aws.amazon.com/corretto/latest/corretto-21-ug/what-is-corretto-21.html  
；  
  
3  
）  
https://docs.aws.amazon.com/corretto/latest/corretto-25-ug/what-is-corretto-25.html  
；  
  
4  
）  
https://docs.aws.amazon.com/corretto/latest/corretto-27-ug/what-is-corretto-27.html  
。  
  
建议相关单位和用户立即开展全面排查，及时升级至最新安全版本，或采取关闭  
Base64  
解码加速路径、限制虚拟机  
AVX-512  
指令集使用、收敛互联网暴露面、加强网络访问控制等临时防护措施，消除网络安全隐患。  
  
来源：工业和信息化部网络安全威胁和漏洞信息共享平台（  
NVDB  
）  
  
  
-  
END  
-  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/co91jb4rYkM8O2Vk1NMvSibBxaP5NSgRL1CpLoy06mLBVffNPZvoS2J6QNQnu0ybyprZ4UicZ4X7icgo5M3YWndwg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=1 "")  
  
  
**欢迎关注我们~**  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/co91jb4rYkOSU1xg4xqGTRzKia2u7n9vpBNvN4tu4xdZsDQLJ6ysIANeNPwWOCANicztN09SZt9JicD9icgiaf2ZV1Q/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=2 "")  
  
