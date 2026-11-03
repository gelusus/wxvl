#  TightVNC 曝出 RCE 漏洞，可致 Windows 系统远程代码执行  
 网安百色   2026-11-03 10:50  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJns6q8TbibF6icrj3yHvaQibjwulLRXC6UrFWicldJPo5D5J8dDib48gHeTgCSNd15ibImGq4B1nqb2nP9w8nueYvIEII7fCZa7CkLTPc/640?wx_fmt=png&from=appmsg "")  
  
TightVNC Viewer 中存在一处漏洞，允许攻击者在密码验证提示出现之前，在 Windows 系统上执行任意代码。韩国 SSD Labs 的研究员 SeongJoon Cho 对此进行了披露，并提供了针对 TightVNC 2.8.88 x64 版本的概念验证（PoC）利用代码。该攻击通过恶意的 VNC 服务端发送特制的屏幕数据，从而破坏查看端（Viewer）的内存。  
  
受影响的组件是“查看端（Viewer）”，而不仅仅是运行 TightVNC 服务端的 Windows 机器。攻击者需要满足以下条件之一：能够通过网络访问处于“监听模式（Listening Mode）”的查看端，或者诱使用户将查看端连接到攻击者控制的服务器。目前的披露信息并未表明该漏洞已在野外被主动利用（in-the-wild exploitation）。  
  
该漏洞的触发始于服务端通过远程帧缓冲协议（RFB, Remote Framebuffer protocol）宣告其屏幕尺寸。TightVNC 会计算容纳该屏幕所需的内存大小，但在处理时将结果转换为了 32 位整数。当屏幕尺寸过大时，该数值会发生整数溢出回绕（wrap around），导致实际分配的内存远小于预期。  
  
在报告的测试用例中，服务端宣告了一个 32,768 x 32,769 分辨率、每像素 32 位的屏幕。查看端仅分配了 128 KB 的内存，却保留了代表超过 4 GB 像素数据的尺寸参数。在后续的屏幕更新中，由于指针计算缺乏对实际分配大小的边界检查，数据会被写入该小缓冲区之外，导致越界写入。  
### TightVNC 认证前远程代码执行（Pre-Auth RCE）漏洞  
  
该 PoC 成功将这种内存破坏转化为代码执行。它首先通过修改传出缓冲区的长度来泄露相邻内存（信息泄露），从而获取绕过地址空间布局随机化（ASLR）所需的关键地址。  
  
随后，它重定向函数调用，并利用返回导向编程（ROP, Return-Oriented Programming）技术复用进程内现有的代码片段，最终通过 Windows 的 CreateProcessW  
 函数启动恶意进程。  
  
“监听模式”会将查看端暴露在 TCP 5500 端口上。根据披露信息，该模式会在无密码验证或 IP 限制的情况下，直接接受来自服务端的入站连接。  
  
能够访问该端口的攻击者可以直接瞄准该查看端发起攻击。此外，如果用户主动向外连接到恶意的技术支持服务器，也会形成另一条攻击路径，且漏洞利用过程发生在任何身份验证之前。  
  
这种攻击条件与依赖弱口令的 VNC 连接劫持有本质区别。它也不同于那些伪装成安装程序的远控工具（用户受欺骗而安装了合法软件）。在此场景中，攻击直接针对查看端程序内部的内存处理逻辑。  
  
研究人员表示，厂商此前未作回应，并于 2026 年 10 月 3 日修复了该问题。然而，TightVNC 的官方发布说明中提及的是 10 月 2 日和 10 月 5 日的安全更新，因此确切的修复日期尚不明确。厂商指出，2.8.89 和 2.8.90 版本已解决查看端的安全问题，并建议该分支的用户升级至 2.8.90 版本。  
  
应尽快更新受影响的查看端程序，关闭不必要的“监听模式”，将 5500 端口的访问限制为仅受信任的系统，并避免连接不受信任的 VNC 服务端。网络层面的访问控制可以降低暴露面，但绝不能替代对存在漏洞的查看端程序进行补丁修复。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
