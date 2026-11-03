#  【安全圈】被控端反杀主控端！TeamViewer爆8.8分堆溢出RCE  
 安全圈   2026-11-03 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：常用于远程 IT 运维的 TeamViewer 客户端曝出高危反向代码执行漏洞（CVE-2026-111585，CVSS 8.8）。当运维管理员主动向恶意的受控主机发起远程协助时，受控机器能利用畸形画面参数反向击穿控制端进程，在管理员电脑上执行任意代码，颠覆了传统远程控制的攻击流向  
## 💥 整数溢出破坏堆内存：受控端伪造超大分辨率  
  
在 TeamViewer 握手建联阶段，受控端负责向上报当前屏幕的显示尺寸（宽、高与色彩深度），主控端据此分配用于渲染远端桌面的帧缓冲区  
  
安全审计发现，15.83 之前的主控端在计算所需内存容量时，使用了固定 32 位整型做乘法运算（width * height * 4  
）。恶意受控端上报经过精心设计的异常分辨率数值后，乘积发生整型回绕（Integer Wrap-around），导致计算出的缓冲区大小被截断为极小的数值  
  
主控端随后以极小的内存块完成堆分配，但在解包后续回传的实际像素数据流时，写入的数据量远超分配边界，直接触发严重的堆越界写入（Heap Out-of-Bounds Write）。借助精确的堆内存布局控制，攻击者可改写函数调用指针，绕过 ASLR/DEP 防护并直接在主控机执行任意 Shellcode  
## 🌐 受影响范围与受害场景  
  
该缺陷覆盖主流桌面和移动控制端平台，影响具体清单如下：  
- **桌面主客户端**  
：TeamViewer Windows、macOS、Linux 客户端版本 < 15.83  
（包含免安装便携版 Portable）  
  
- **移动控制端**  
：Android 平台的 TeamViewer Remote Control 应用程序版本 < 15.83  
  
- **运维工单钓鱼场景**  
：外部攻击者提交虚假技术支持工单，引诱运维人员主动远程连接一台已布设攻击载荷的靶机，主控端在接通画面的瞬间即遭静默接管  
  
- **跳板机反噬风险**  
：运维人员的电脑通常常驻大量生产服务器凭据、VPN 隧道与特权私钥，一旦主控端失守，将直接引发全内网被横向穿透的高危连锁反应  
  
## 🛡️ 官方修复与安全加固建议  
  
TeamViewer 官方已发布正式安全补丁，增加了对远端屏幕几何参数的算术溢出校验与内存安全边界控制。建议相关技术团队执行以下处置：  
- **全面升级版本**  
：核查各平台已安装客户端，统一更新至 15.83  
 或更高版本；重点排查排查运维人员电脑中遗留的绿色解压版（Portable）  
  
- **启用连接白名单**  
：在管理控制台强制开启信任伙伴列表（Allowed Partner List），禁止向未经验证的匿名远程 ID 发起连接会话  
  
- **运维环境物理隔离**  
：严禁在承载堡垒机权限或核心域管凭据的高特权机器上运行通用远控软件；建议将远程协助客户端收敛至独立的无特权受限虚拟机中  
  
  
  
  
  END    
  
  
阅读推荐  
  
  
[【安全圈】10分满分漏洞！LXD与MicroCloud遭伪造证书免密接管](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079668&idx=1&sn=1c6ae764b6dcb5d4403c76892511c741&scene=21#wechat_redirect)  
  
  
  
[【安全圈】安卓0-Click死亡短信曝光：无需点击令手机反复死机崩溃](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079668&idx=2&sn=1390d439ffaecfa8a8e4a48741af4adb&scene=21#wechat_redirect)  
  
  
  
[【安全圈】AuthMask高危漏洞曝光：恶意App冒充Teams窃取M365令牌](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079668&idx=3&sn=1bb527a696a34a38038d7cb851027e9b&scene=21#wechat_redirect)  
  
  
  
[【安全圈】24小时内超6万次攻击针对WordPress RCE漏洞，更新过程无需登录就可以利用](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079657&idx=1&sn=0e7f6a51f91ae4c6aadcb6d7ca13c2b3&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
  
