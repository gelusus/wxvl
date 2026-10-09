#  Telegram Desktop：一次点击就能接管账号的 IPC 注入漏洞  
 Ots安全   2026-10-09 10:54  
  
**威胁简报**  
  
  
**恶意软件**  
  
  
**漏洞攻击**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zNsFJyIuL0HBKsTXWKicQwW0CpT3qiburSw5ibsxJowPBavDSVI0SQGcp694kB24a8tKQnY8ksBNJz9ibQDsIl3XDspCOMQZn7Yic7JCA067NxWE/640?wx_fmt=jpeg&from=appmsg "")  
## 一个链接，账号易主  
  
有人把你拉进一个 Telegram 群组。群里出现一条链接。你随手一点，自己的账号就不再只属于你了。  
  
这并非钓鱼，也不是诱导你输入验证码的骗局。安全研究者 beaksec 新近披露的攻击链，从受害者点击一个普通链接开始，到攻击者拿到完整的本地会话文件为止，全程不需要额外交互。整个漏洞由两个原本“不算严重”的小问题叠加而成：一个是 Telegram Desktop 单实例进程间通信（IPC）中记录分隔符未转义，另一个是内部 interpret:  
 方案缺少授权校验。二者相遇，就让“点击链接”这件平常事变成了账号接管的入口。  
## 漏洞概览  
- **影响范围**  
：Telegram Desktop ≤ 7.2.8，已在 Windows 6.9.3 上确认  
  
- **固定版本**  
：7.2.9（commit db3405699f  
）  
  
- **CVE**  
：CVE-2026-107181  
  
- **CWE**  
：CWE-143（记录分隔符未正确中和）  
  
- **影响**  
：远程诱导任意本地文件读取，并可将文件外发到攻击者控制的聊天；可进一步接管账号  
  
- **CVSS 3.1**  
：8.1 High（AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N  
）  
  
- **CVSS 4.0**  
：8.6 High（VulnCheck）  
  
需要强调的是，这里的“任意本地文件读取”不是浏览器沙箱里的那种限制读取，而是攻击者能把你磁盘上的敏感文件直接发到他自己频道里。当目标文件是 Telegram 本地数据时，也就等于把整个账号拱手让人。  
## 缺陷一：单实例 IPC 的记录分隔符注入  
  
Telegram Desktop 会向操作系统注册 tg://  
 协议。当用户点击一个以 tg://  
 开头的链接时，系统会把链接作为命令行参数启动 Telegram。  
  
如果 Telegram 当前没有运行，新进程会直接处理这条链接，事情很简单。但如果已经有实例在运行，操作系统仍然不知道这一点，它照旧启动第二个 Telegram 进程。第二个进程会尝试连接本地 socket；一旦连接成功，就说明“前辈”还在，于是它把链接文本交给运行中的实例，然后自己退出。  
  
这就引入了一个不可避免的环节：把内存中的 URL 对象序列化成文本，通过 socket 发过去，接收端再反序列化回来。Telegram 自己设计了一套简单的格式：  
  
```
OPEN:<URL>;
```  
  
  
关键词是 OPEN:  
，参数是 URL，末尾的分号表示这条指令结束。接收端按分号切割，把每一段当成独立指令处理。相关代码位于 sandbox.cpp  
：  
  
```
// sandbox.cpp:295-297
for (constauto &url : cRefStartUrls()) {
    commands += u"OPEN:"_q + url.toString(QUrl::FullyEncoded) + ';';
}
```  
  
  
问题在于：**URL 参数里如果出现分号，这个分号不会被转义**  
。对新进程来说，分号只是 URL 查询参数里的普通字符；它忠实地把整个字符串写进 socket：  
  
```
OPEN:tg://x?a=1;CMD:quit;
```  
  
  
运行中的实例按分号一切，就得到了两条指令：  
  
```
OPEN:tg://x?a=1
CMD:quit
```  
  
  
这就是 CWE-143 所描述的“记录分隔符未正确中和”。原始链接里藏着的分号，在 IPC 另一头变成了指令边界。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0GZF8Tby9aEfAdVHcPrbNCfq4bsQhjc472NhfYLKlNUe3wzhk7mne4KvciaicIVbg4gLXyxssT1vh1Cft6VZW0iaC6DUF7dNx83VQ/640?wx_fmt=png&from=appmsg "")  
  
研究者尝试了 CMD:  
，但它只接受 show  
 和 quit  
，危害性有限。真正的杀招在于 OPEN:  
 对 URL scheme 没有任何过滤——它不只接受 tg://  
，任何 scheme 都能往里塞。于是第二条指令可以被替换成 interpret:  
。  
## 缺陷二：interpret: 内部的未授权文件读取  
  
interpret:  
 是 Telegram 内部使用的一个 URI 方案，没有向操作系统注册，普通浏览器或系统根本不认识它。但在 Telegram 自己的代码里，它可以被当作 start URL 处理：  
  
```
// application.cpp:1162-1164
if (url.scheme() == u"interpret"_q) {
    interprets.append(url.path());
returnfalse;
}
```  
  
  
interpret:  
 原本是 Telegram 发布新版本时用的内部工具。构建脚本会写一个小型指令文件，说明要发送到哪个频道、发送什么文件、配什么文字说明，然后启动 Telegram 并传给它：  
  
```
# Telegram/build/updates.py:206
subprocess.call(...
'Telegram -sendpath interpret://' + scriptPath + '/.../command.txt',
shell=True
)
```  
  
  
指令文件长这样：  
  
```
from: 1234567890
channel: 1987654321
file: out/Release/deploy/6.9.3/tsetup.6.9.3.exe
caption: TDesktop at 12.06.26: ...
```  
  
  
from:  
 用于和当前登录账号 ID 比对，防止用错账号发布；如果直接省略这一行，校验就被跳过。channel:  
 指定目的地，可以是频道或超级群组。实际干活的是 InterpretSendPath  
：  
  
```
// support_helper.cpp:673-680
QStringInterpretSendPath(not_null<Window::SessionController*> window,
const QString &path)
{
    QFile f(path);
if (!f.open(QIODevice::ReadOnly)) {
return"App Error: Could not open interpret file: " + path;
    }
constauto content = QString::fromUtf8(f.readAll());
    ...
}
```  
  
  
这个函数打开 path  
 指定的文件，读取内容，然后发到 channel  
 指定的聊天。**整个过程没有弹窗确认，也没有检查是谁发起的请求**  
。当它是从命令行被构建脚本调用时，这不算漏洞——攻击者已经能操作电脑了。但当它可以通过 socket、通过点击链接触发时，它就成了一把能远程启动的钥匙。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0E4iacAVkJSNRpUFjmeuFrYeYWh0rIgMtmB7aq4lpqT6oxAzuo76e1svksBeuticBUBIupzrXBymR0wqCdRB1kgyBlZkiaTe3C7Io/640?wx_fmt=png&from=appmsg "")  
  
于是攻击者可以构造这样一条链接：  
  
```
tg://x?a=1;OPEN:interpret:../../../Downloads/Telegram%20Desktop/instructions.txt
```  
  
  
其中 interpret:  
 指向一个已经落在磁盘上的指令文件。只要文件存在，内容就会按攻击者的意图被外发。  
## 从文件读取到账号接管：关键在 tdata  
  
到目前为止，攻击者能做的是“把受害者磁盘上的某个文件发到自己的频道”。要演变成账号接管，还需要读对文件。  
  
Telegram Desktop 把本地数据存放在 tdata  
 目录下，并且默认加密。但它采用的是**密钥封装**  
方案，涉及两把密钥：  
- **DEK（数据加密密钥）**  
：长度足够、熵值高，负责加密用户数据。  
  
- **KEK（密钥加密密钥）**  
：不直接是密码，而是由密码通过 KDF（密钥派生函数）结合 salt 计算而来，只负责加密 DEK。  
  
用伪代码表示开锁过程：  
  
```
salt, encrypted_DEK = read("tdata/key_datas")
passcode = user_passcode()          # 若未设置，则为空字符串
KEK = KDF(passcode, salt)
DEK = decrypt(encrypted_DEK, KEK)
session = decrypt(authorization_file, DEK)
```  
  
  
默认情况下，Telegram Desktop 不会要求用户设置本地密码。此时 passcode  
 是空字符串，KEK 完全由空密码和 salt 派生而来，而 **salt 就明文存在 tdata/key_datas 里**  
。只要拿到 key_datas  
，攻击者就能重算 KEK，解出 DEK，进而解密所有会话数据。  
  
真正需要窃取的是三个文件：  
  
```
tdata/
├── key_datas              # salt + 被 KEK 加密的 DEK
├── D877F783D5D3EF8Cs      # MTProto 授权信息，被 DEK 加密
└── D877F783D5D3EF8C/
    └── maps               # 数据索引，无敏感内容但加载授权时需要
```  
  
  
目录名 D877F783D5D3EF8C  
 并非随机，而是从默认数据名 data  
 派生，每台安装都一样。攻击者把这三个文件放进自己机器上的干净 tdata  
 目录，启动 Telegram，就能以受害者身份登录。  
## 完整攻击链：只需要一次点击  
  
把两条缺陷串起来，攻击链如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/zNsFJyIuL0GtC84FnPn4FI3icE2Dr5pRfia6MY6zPumh89WwFImFJwE6sdsXV80j6YW3Fuc7j2t8oGyQSXv1D55fqfnDGTQ2UkJHELFDdFWzk/640?wx_fmt=png&from=appmsg "")  
1. **建群拉人**  
：攻击者创建一个超级群组，把受害者拉进去。Telegram 默认隐私设置允许任何人把你拉进群，受害者不会收到确认请求。  
  
1. **落地指令文件**  
：攻击者在群里上传三个普通文本文件，分别指向 key_datas  
、授权文件和 maps  
。Telegram Desktop 默认会自动下载群聊中不超过 8 MiB 的文件，文件会被保存到 C:\Users\<user>\Downloads\Telegram Desktop\<文件名>  
。  
  
1. **发送 https 链接**  
：攻击者再发一条看起来无害的 https://example.com/rules  
 链接。  
  
1. **302 跳转**  
：受害者点击后，浏览器访问攻击者服务器，服务器返回 302，把浏览器导向精心构造的 tg://  
 注入链接：  
  
```
tg://x?a=1
;OPEN:interpret:../../../Downloads/Telegram%20Desktop/instructions1.txt
;OPEN:interpret:../../../Downloads/Telegram%20Desktop/instructions2.txt
;OPEN:interpret:../../../Downloads/Telegram%20Desktop/instructions3.txt
```  
  
  
**5.注入触发**  
：  
系统启动第二个 Telegram 进程，它把链接交给运行中的实例；实例按分号拆分，三条 interpret:  
 命令依次执行。  
  
**6.接管账号**  
：  
三个目标文件被上传到攻击者控制的频道。攻击者把它们拼成 tdata  
，即可在本地恢复受害者会话。  
  
注意一个关键细节：**在 Telegram 聊天窗口内点击 tg:// 链接不会被注入**  
。因为应用内部直接处理这类链接，不走 socket。所以攻击者必须把受害者引向外部浏览器，再用 302 跳转回来。  
## 修复与缓解  
  
Telegram 在 2026-09-16 提交了修复 db3405699f  
，并在次日发布 7.2.9。该提交做了四件事：  
1. 完全移除 interpret://  
 方案与 Support::InterpretSendPath  
；  
  
1. 对单实例 socket 的分隔符做转义（写入前 percent-hex 编码，拆分后解码），让数据中的分号无法再变成边界；  
  
1. 同一条连接如果包含 OPEN:  
，则跳过 CMD:  
 和 CTRL:  
 记录；  
  
1. 一旦连接上出现非本地 URL，后续本地文件路径会被丢弃。  
  
研究者特别指出，这次修复发布得非常低调：7.2.9 的更新日志只提到“渲染修复”，提交标题是“Remove legacy interpret path helper”，没有任何配套安全公告。直到 2026-10-07，CVE-2026-107181 才正式分配。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0GsYQLcUJTDvA7Nrw5CbTknHYuKzK8GCPHzJtQ5eJNTE7oN4WdibD6MeZQDQankzfhh6eSrhIEwGTmstTVw7VsNwL2XK0zRH5Ns/640?wx_fmt=png&from=appmsg "")  
  
在升级到 7.2.9 之前，用户还可以采取以下缓解措施：  
- **开启“询问保存每个文件的位置”**  
：关闭自动下载，指令文件就无法落到磁盘。  
  
- **限制谁能把你加入群组**  
：改为“仅联系人”或更小范围，阻断外发目的地。  
  
- **设置本地密码**  
：像设置真正的密码一样设置 Telegram 本地密码。它不能阻止文件被读取，但能让窃走的会话数据无法解密。  
  
## 披露时间线  
- **2026-06-25**  
：通过 ZDI 向厂商报告  
  
- **2026-09-16**  
：Telegram 独立修复并提交 db3405699f  
  
- **2026-09-17**  
：Telegram Desktop 7.2.9 发布  
  
- **2026-09-30**  
：ZDI 以“已修复”结案，披露权交还研究者  
  
- **2026-10-03**  
：beaksec 发布本文  
  
- **2026-10-07**  
：CVE-2026-107181 分配  
  
## 结语  
  
这个案例再次说明，真正危险的漏洞往往不是某个孤立的缓冲区溢出，而是两个“低风险”设计的组合：一条未转义的分隔符，加上一个缺少授权的内部接口。单独看都很普通，合在一起就能让一次普通点击变成账号接管。  
  
对普通用户来说，比较稳妥的防护仍是及时更新；对安全研究者来说，它提醒我们：IPC 边界、URI scheme 处理以及内部调试/发布接口，都是值得反复审视的攻击面。  
  
**END**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zNsFJyIuL0Gb0PtVTib29Sfzs03UBUnAA3qlUj2PMOD7qv6uicQJcAOR7LGIDyw4tExknq11IzVzO5YtLZzibnibQ86f6lVJ7CfYyIjAlq2wcag/640?wx_fmt=jpeg&from=appmsg "")  
  
  
公众号内容都来自国外等平台- 搜索的内容通过结合编写 -   
  
公众号 |   
AnQuan7 (Ots安全)  
  
