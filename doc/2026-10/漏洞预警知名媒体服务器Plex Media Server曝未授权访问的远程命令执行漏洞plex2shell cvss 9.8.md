#  漏洞预警|知名媒体服务器Plex Media Server曝未授权访问的远程命令执行漏洞plex2shell cvss 9.8  
原创 小白哥
                    小白哥  小白安全成长记   2026-10-20 09:25  
  
最近真是哪哪都是RCE，大家用的漏洞情报是自己鼓捣的，还是用的哪家的，求推荐！！！这不又一个rce，关键还有poc。。。今天的主角是**Plex Media Server。**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Ueoom4Tb3S9UUdLvsEBAIkINxxe0hF9h5tMo6FPND2UJc8gicicGyjol26mwVxFicrJnq6eic9hmq0xncehOOfggg9aPVXvYOUax1KbWtBMd0V0/640?wx_fmt=png&from=appmsg "")  
  
**Plex Media Server（PMS）**  
 是一款功能强大、极受欢迎的**个人媒体服务器软件**  
。它可以让你将自己电脑或 NAS（网络附加存储）上的电影、电视剧、音乐、照片等各种媒体文件集中管理，并流式传输到家里的电视、手机、平板、游戏机甚至外网的各种设备上。  
  
Plex 采用经典的 **C/S（客户端/服务器）架构**  
：  
- **服务端（Plex Media Server）**  
：运行在你的主力存储设备或服务器上，负责索引文件、下载元数据、转码和处理流传输（默认监听 32400  
 端口）。  
  
- **客户端（Plex App / Client）**  
：负责展示界面、发送播放指令并解码渲染视频。  
  
在低于 1.43.3.10828 的版本存在**未授权远程代码执行（Unauthenticated RCE）漏洞利用链工具**  
（PoC）。目前已发现在野利用，有折腾这个应用的老铁注意了。  
### 1. 漏洞说明  
- **目标软件**  
：Plex Media Server < 1.43.3.10828（官方已在 1.43.3.10861 及更高版本中修复）。  
  
- **危害程度**  
：严重（CVSS 评分为 9.8 / 10.0）。攻击者无需任何前置权限（Unauthenticated），即可通过互联网直接对目标 Plex 服务器执行任意代码（RCE）。  
  
- **主要功能**  
：该 Python 脚本（plex2shell.py  
）串联了多个漏洞，能够自动完成从信息窃取到最终反弹 Shell（Reverse Shell）的完整攻击流程。  
  
### 2. 漏洞攻击链（CVE 组合拳）  
  
该工具能够成功利用，是因为它组合串联了四个不同的漏洞环节：  
1. **CVE-2026-96652（未授权 SSRF）**  
：  
  
1. 通过 /player/timeline  
 接口中的 protocol  
 参数，攻击者可以注入任意 URL 触发服务端请求伪造（SSRF），且该接口对 X-Plex-Token  
 的校验存在缺陷（只检查字段是否存在，不验证有效性）。  
  
1. **CVE-2026-96654（本地回环跳转与输入注入）**  
：  
  
1. 利用服务对 127.0.0.1  
 本地回环（Loopback）高度信任的机制，配合参数注入（如 identifier  
 字段），迫使 Plex 服务器向攻击者指定的外部服务器发送带有管理员凭据的请求，从而**窃取管理员的 Owner Token**  
。  
  
1. **CVE-2026-96655（已授权的转码器 SSRF 与任意文件写入）**  
：  
  
1. 利用获取到的管理员 Token 修改 Plex 转码器配置（TranscoderH264Options=dump-yuv  
），使得服务器在后续处理视频转码时，可以将无损解码的帧（攻击者可控的恶意共享对象 .so  
 或动态库文件）写入到指定的插件/编解码器目录中。  
  
1. **CVE-2026-96656（通过未签名动态库加载实现代码执行）**  
：  
  
1. 触发第二次转码，促使 Plex 重新加载和动态链接（dlopen  
）指定目录下的 .so  
 文件，从而以 plex  
 进程权限执行自定义代码，最终成功弹出反弹 Shell。  
  
### 3. 利用说明  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ueoom4Tb3SibSIdJkEiaweDeH1UVhTmj28Gk3L1jB5B7Q8X2jxoyT6wudyRPtQ4yhEgCVoHu5mGEAEjKibFUSjooy4CKgPDoswMr4w2TnHzLo4/640?wx_fmt=png&from=appmsg "")  
- **plex2shell.py**  
：核心攻击脚本，支持 Linux（原生/Docker）和 Windows 等目标系统环境。  
  
- **payload_linux.c / payload_windows.c**  
：用于植入目标系统的底层二进制载荷源码。  
  
- **detection/**  
：包含用于防御和检测的规则（如 YARA 规则等）。  
  
- poc链接:https://github.com/code-zm/plex2shell/  
  
### 4. 安全建议  
  
由于该漏洞链具有较高的自动化危害且已被公开利用，请勿用于非法用途，建议所有使用 Plex Media Server 的用户：  
- **立即升级**  
：将 Plex Media Server 升级到 1.43.3.10861  
 或更高版本。  
  
- **避免直接暴露公网**  
：若无必要，不要将 Plex 服务的 32400  
 端口直接裸露在互联网上，建议通过 VPN 或受密码保护的反向代理（如Nginx 配合认证）进行访问。  
  
- END -  
  
  
