#  iTerm2严重RCE漏洞曝光: 利用Kitty图像协议注入，查看恶意文件即可执行命令  
tuto
                    tuto  杂杂咱谈   2026-11-21 16:06  
  
## 一、漏洞概述  
  
研究人员披露了一个影响 macOS 终端模拟器 **iTerm2**  
 的远程代码执行漏洞。该漏洞存在于 iTerm2 对 Kitty Graphics Protocol 的处理逻辑中，攻击者可以通过构造包含恶意 APC 转义字符的普通文件，在受害者使  
用 cat 命令  
查看文件时触发命令注入，最终在受害者本地终端执行任意命令。  
  
该漏洞并非传统的内存破坏问题，而是由多个处理逻辑缺陷组合形成的命令注入漏洞。  
  
**核心特点：**  
- **漏洞类型：远程代码执行 / 命令注入。**  
- **影响组件：iTerm2 Kitty 图像协议处理模块。**  
- **攻击条件：受害者使用 iTerm2 输出攻击者构造的恶意文件。**  
- **认证要求：无需额外身份认证。**  
- **用户交互：需要受害者主动使用终端查看恶意文件，例如执行 cat README.md。**  
- **影响范围：支持 Kitty Image Protocol 的受影响版本。**  
- **修复版本：iTerm2 3.7.9。**  
该漏洞在默认安装配置下即可触发，不需要修改特殊设置，并且研究人员表示其在 Bash、Zsh 和 Fish 等常见 Shell 中均可复现。  
## 二、受影响版本  
  
根据研究人员披露的信息：  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">说明</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">受影响产品</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">iTerm2</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">受影响组件</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Kitty Image Protocol</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">受影响版本</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">支持 Kitty Image Protocol 的相关版本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">修复版本</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">iTerm2 3.7.9</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">默认配置</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">存在风险</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">操作系统</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">macOS</span></section></td></tr></tbody></table>  
**安全建议：**  
 使用 iTerm2 的用户应检查当前安装版本，优先升级至 3.7.9 或更高版本。  
## 三、漏洞技术原理  
  
该漏洞由三个相互关联的逻辑缺陷共同导致。  
### 3.1 APC 处理缺少可信终端检查  
  
iTerm2 使用 VT100 解析器处理终端控制序列。当解析器接收到 APC 序列时，  
会调用 executeAPC: 函数  
。  
  
正常情况下，部分 OSC 控制序列在执行前会检查终端是否可信，以限制不可信输入触发敏感操作。  
  
但研究人员发现，APC 处理路径缺少相应的可信状态检查。  
  
当 APC 序列满足 Kitty 图像协议的格式要求时，程序便会创建图像命令对象，并将其传递给 Kitty 图像控制器执行。  
  
这意味着：  
- APC 序列可以直接进入处理流程。  
  
- 不会执行与 OSC 相同的可信终端检查。  
  
- 恶意文件中的 APC 字节能够被终端解析器处理。  
  
因此，攻击者可以利用普通文件中的隐藏控制序列触发后续图像协议处理逻辑。  
### 3.2 Kitty 文件传输错误处理保留换行符  
  
漏洞的第二个关键问题位于 Kitty 图像协议的文件传输功能。  
  
当 iTerm2  
 收到 t=f (File Transmit)  
类型的图像命令时，会对 Base64 解码后的数据进行处理，并将其作为文件路径。  
  
相关代码首先通过 URL.resourceValues() 检查目标文件是否存在  
。  
  
如果文件不存在，系统会抛出异常，随后进入错误处理流程：  
```
catch {    return "EBADF:\(error.localizedDescription)"}
```  
  
问题在于，macOS 的 NSError.localizedDescription 会保留文件名中的实际换行符 (0x0A)  
，而不会对这些字符进行转义。  
  
此外，  
代码虽然存在 filenameIsSafe() 文件名安全检查  
，但该检查位于文件资源属性读取成功之后。  
  
当攻击者构造的文件路径指向一个不存在的文件时，程序会在执行安全检查之前进入异常处理流程，导致该安全检查无法发挥作用。  
  
攻击者由此能够将换行符嵌入错误消息中，为后续命令注入创造条件。  
### 3.3 错误响应被写入 PTY 标准输入  
  
第三个问题是错误响应的输出位置。  
  
在 Kitty 图像协议的响应处理函数 respondToTransmit() 中，默认 verbosity 设置为 .normal  
。  
  
此模式下，即使处理过程中发生错误，iTerm2 仍然会生成协议响应，并将其发送给终端。  
  
随后，响应经过委托调用链，最终进入：  
```
[delegate screenSendReportData:    [string dataUsingEncoding:encoding]];
```  
  
这里的关键问题是：screenSendReportData: 使用与键盘输入相同的 PTY 数据通道  
。  
  
因此，原本应该作为终端协议响应处理的数据，会被写入伪终端的输入端，最终被 Shell 当作用户输入处理。  
  
由于错误消息保留了攻击者控制的换行符，Shell 便可能将其识别为多个独立命令。  
  
三个缺陷至此形成完整的命令注入链。  
## 四、完整漏洞利用链  
### 4.1 攻击者构造恶意文件  
  
攻击者首先制作一个普通文本文件，  
例如README.md、.txt 或.log文件  
。  
  
文件中除了正常文本内容外，还嵌入一段特殊的 APC 转义序列：  
```
ESC _ G ... ESC \
```  
  
该序列符合 Kitty 图像协议的命令格式，其中包含经过 Base64 编码的恶意文件路径。  
  
攻击者通过文件路径中的换行符，将预期执行的 Shell 命令夹在两段路径文本之间。  
### 4.2 恶意文件传播  
  
攻击者可以通过多种方式向目标用户提供恶意文件，例如：  
- 在 Git 仓库中提交经过伪装的 README.md。  
  
- 通过共享文件夹分发恶意文件。  
  
- 将恶意文件打包进 ZIP 等压缩包。  
  
- 通过电子邮件附件发送文件。  
  
- 将恶意文件伪装成项目说明、安装指南或配置文件。  
  
由于 APC 控制序列嵌入在普通文件内容中，用户仅通过常规文本阅读可能无法发现异常。  
### 4.3 受害者使用 cat 命令触发漏洞  
  
当用户在 iTerm2 中执行：  
```
cat README.md
```  
  
终端会接收到文件中的全部字节。  
  
其中，普通文本正常显示，而嵌入的 APC 控制序列会被 VT100 解析器识别并消费，因此不会以普通文本形式显示出来。  
  
随后，iTerm2 执行 Kitty 图像协议中的文件传输处理逻辑。  
  
由于恶意文件指定的目标路径不存在，程序进入异常处理流程，并将包含换行符的错误消息构造成协议响应。  
### 4.4 错误响应注入 Shell  
  
错误响应最终被写入 PTY 输入通道。  
  
Shell 接收到包含换行符的内容后，会将其拆分成多行处理。  
  
其中，攻击者预先嵌入的命令可能被当作独立 Shell 命令执行。  
  
研究人员使用以下命令作为演示：  
```
open -a Calculator
```  
  
最终，macOS 的计算器应用程序被启动，以此证明攻击者能够通过文件展示行为触发本地命令执行。  
### 4.5 攻击流程图  
```
攻击者  |  v构造恶意 README.md  |  +-- 正常文本内容  |  +-- 隐藏 APC 转义序列         |         v    Base64 编码恶意文件路径         |         v通过 Git 仓库 / 邮件 / 压缩包传播         |         v受害者使用 iTerm2 执行 cat README.md         |         vVT100 解析器识别 APC 序列         |         vKitty Image Protocol 文件传输处理         |         v访问不存在的恶意文件路径         |         vNSError.localizedDescription保留攻击者控制的换行符         |         v构造 Kitty 错误响应         |         v错误响应写入 PTY 标准输入         |         vShell 将换行符解析为命令分隔符         |         v<命令执行>
```  
  
  
**关键问题：**  
 原本用于向终端报告图像处理结果的协议响应，最终进入了 Shell 的输入通道。攻击者正是利用这一信任边界问题，将协议错误消息转化为可执行命令。  
## 五、漏洞利用特点与安全控制绕过  
  
研究人员指出，该漏洞能够绕过或避开多个原有安全控制。  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">安全机制</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">状态</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">原因</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">filenameIsSafe()</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">被绕过</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件不存在时，程序先抛出异常，无法执行后续文件名检查</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">terminalIsTrusted</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">缺失</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">APC 处理路径没有执行相应可信终端检查</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">disablePotentiallyInsecureEscapeSequences</span></code></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">无法防护</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">该设置并未覆盖此次利用涉及的 APC 处理路径</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Kitty 默认响应模式</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">有利于攻击者</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">.normal</span></code><section><span leaf=""> 模式会发送错误响应</span></section></td></tr></tbody></table>  
另外，该漏洞有几个值得关注的技术特点：  
1. **无需内存破坏：不依赖缓冲区溢出、堆损坏或竞争条件。**  
1. **无需特殊配置：研究人员表示，默认 iTerm2 安装配置即可触发。**  
1. **不依赖特定 Shell：Bash、Zsh 和 Fish 均受到该利用方式影响。**  
1. **文件类型限制较少：只要文件内容能够被输出到终端，就可能成为攻击载体。**  
1. **存在用户交互要求：需要受害者主动使用终端输出恶意文件，而不是仅仅下载文件便自动执行。**  
研究人员展示的 PoC 会在终端输出中留下异常字符和 Shell 错误信息，命令执行过程并非完全隐蔽。  
## 六、修复方案与补丁分析  
  
iTerm2 开发者确认了该问题，并通过两个提交修复相关安全缺陷。  
### 6.1 第一阶段：清理 Kitty 图像协议响应  
  
Commit：a53b1bc  
  
官方首先在 KittyImageController.swift 中增加：  
```
sanitizedResponseMessage(_:)
```  
  
该函数对错误响应中的控制字符进行清理，将以下字符替换为空格：  
- C0 控制字符 (0x00–0x1F)。  
  
- DEL 控制字符 (0x7F)。  
  
- 包括 ESC、CR、LF 等关键字符。  
  
随后，在 respondToTransmit() 和 respondToDisplay() 中对错误消息执行清理，再将其封装为 Kitty 协议响应  
。  
  
这样，即使攻击者在恶意文件路径中嵌入换行符，也无法再通过错误响应将换行符传递给 Shell。  
  
官方同时增加了回归测试，专门验证恶意文件路径中的换行符和其他控制字符不会进入协议响应。  
### 6.2 第二阶段：扩展终端响应的控制字符过滤  
  
Commit：2a63835  
  
在第一阶段修复的基础上，开发者进一步发现，Kitty 图像响应并不是唯一存在类似问题的路径。  
  
其他可能将外部可控数据写入 PTY 的位置也需要加强处理。  
  
因此，第二次提交引入统一的控制字符清理函数：  
```
stringByReplacingControlCharactersWithSpace
```  
  
该函数进一步覆盖：  
- C0 控制字符。  
  
- DEL 字符。  
  
- C1 控制字符 (0x80–0x9F)。  
  
并将其应用于三个终端报告路径：  
1. Kitty 图像协议响应。  
  
1. Kitty 拖放功能。  
  
1. 终端标题报告。  
  
这一修复不仅解决了本次 Kitty 图像协议注入问题，也减少了其他终端报告功能中类似控制字符注入风险。  
### 6.3 修复版本  
  
根据 iTerm2 官方仓库中的修复提交及版本更新记录，安全修复进入 iTerm2 3.7.9。  
  
官方更新说明明确指出，攻击者可以通过构造恶意文件，在用户  
使用 cat 等  
命令显示文件时执行任意命令；修复措施是阻止错误响应携带可能注入 Shell 的控制字符。  
  
建议用户：  
- 将 iTerm2 更新至 3.7.9 或更高版本。  
  
- 不要使用终端直接输出来源不明的文件。  
  
- 对第三方 Git 仓库中的 README、日志和配置文件保持警惕。  
  
- 对来源不明的压缩包和项目文件进行安全检查。  
  
- 在升级前，避免在 iTerm2 中执行不可信项目提供的文件查看命令。  
  
## 七、安全影响与风险分析  
  
该漏洞反映出终端模拟器在处理控制序列时可能存在的一个重要安全问题：**终端输出并不总是纯粹的文本，恶意文件中的控制字符可能触发终端内部的协议处理逻辑。**  
  
在实际攻击场景中，攻击者可以将恶意 APC 序列隐藏在正常的项目文档中，再借助 Git 仓库、邮件附件或共享文件夹进行传播。  
  
如果开发人员或运维人员习惯  
使用 cat 查看项目文件，就可能在不知情的情况下触发恶意协议处理流程。  
  
潜在风险包括：  
- **开发环境入侵：通过恶意 Git 仓库中的 README 或配置文件触发命令执行。**  
- **运维终端失陷：针对习惯使用命令行查看日志和配置文件的管理员。**  
- **供应链攻击：将恶意控制序列植入开源项目或第三方交付文件。**  
- **本地权限滥用：命令以当前运行 iTerm2 的用户权限执行，可能进一步访问该用户有权访问的文件、凭据和其他资源。**  
该漏洞本身并不意味着攻击者可以直接获得 macOS root 权限。实际权限取决于运行 iTerm2 的用户账户及其系统权限。  
## 八、总结  
  
iTerm2 Kitty 图像协议 RCE 漏洞展示了一种较为特殊的终端命令注入攻击方式。  
  
攻击者无需利用传统内存破坏漏洞，只需将恶意 APC 控制序列嵌入普通文件，通过 Kitty 图像协议的错误响应处理逻辑，将攻击者控制的换行符注入 PTY 输入通道，最终实现 Shell 命令执行。  
  
此次漏洞的核心问题并不在于 Kitty 图像协议本身，而在于终端模拟器错误地将未经清理的协议响应写入 Shell 输入通道，导致数据处理边界发生混淆。  
  
**对于 macOS 开发人员、系统管理员及安全研究人员而言，应尽快升级 iTerm2 至 3.7.9 或更高版本，同时提高对第三方项目文件和终端控制序列的安全意识**  
  
****  
  
引用来源:   
ii4gsp的github  
  
