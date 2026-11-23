#  在 iTerm2 里 cat 一个文件，命令就执行了：一次终端转义注入的漏洞复盘  
 Ots安全   2026-11-23 05:24  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0G4DBXgnQH3E0FXU2QgHX0eMwGpO0N85lU9ml29iamM9WJUgicicUZRL4keNfLfXZsMyv7DAfZDSR3Mj3p79rjribJx5Asm50MicG3o/640?wx_fmt=png&from=appmsg "")  
  
2026 年 11 月 21 日，安全研究者 ii4gsp 公开了一篇针对 macOS 终端 iTerm2 的研究，标题很直白：通过 Kitty 图像协议的 APC 响应注入实现任意命令执行。它的触发条件简单到有点反直觉：**在 iTerm2 里用 cat 打开一个被构造过的文件，文件里藏着的命令就可能被 shell 执行**  
。不需要点击，不需要额外交互，也不需要修改任何默认设置。  
  
这篇文章之所以值得单独复盘，不是因为它用了什么新颖的内存破坏技巧，恰恰相反，它是一个纯粹的**逻辑缺陷**  
：没有越界写、没有竞争条件，三个各自看似无害的设计决定拼在一起，就成了一条完整的执行链。本文按根因、链条、绕过、补丁的顺序拆开讲，并把能说明问题的源码片段照录出来。  
  
## 一、先看问题  
  
先把能核实的事实和不能证实的部分分开，后面所有判断都基于这张表。  
<table><thead><tr><th data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;background:#f2f2f2;font-weight:600;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">项目</span></span></section></th><th style="border:1px solid #c9c9c9;padding:6px 12px;background:#f2f2f2;font-weight:600;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">内容</span></span></section></th><th style="border:1px solid #c9c9c9;padding:6px 12px;background:#f2f2f2;font-weight:600;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">来源与口径</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">漏洞性质</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">Kitty 图像协议文件传输错误响应中的命令注入，逻辑缺陷</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">研究者原文 / iTerm2 官方更新说明</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">CVE 编号</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">未分配 CVE</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">多家安全聚合站明确记录，本文不编造编号</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">影响版本</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">3.6.1 至 3.7.8</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">研究者原文</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">修复版本</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">3.7.9，2026-11-20 构建</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">iTerm2 官方更新说明</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">发现者</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">Minseong Kim，NSHC Red Alert Labs</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">iTerm2 官方更新说明致谢（原文未提）</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">触发方式</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">在终端显示被构造的文件，例如 </span></span><code><span leaf=""><span textstyle="" style="font-size: 15px;">cat</span></span></code><span leaf=""><span textstyle="" style="font-size: 15px;">，APC 转义序列被 VT100 解析器消费</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">研究者原文</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">适用 shell</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">bash、zsh、fish 等全部 shell，与 shell 类型无关</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">研究者原文</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">在野利用</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">暂无任何在野利用报告</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">多家聚合站核查，官方未提及</span></span></section></td></tr><tr><td data-colwidth="130" style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">PoC 状态</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">2026-11-21 公开</span></span></section></td><td style="border:1px solid #c9c9c9;padding:6px 12px;"><section><span leaf=""><span textstyle="" style="font-size: 15px;">研究者原文与聚合站一致</span></span></section></td></tr></tbody></table>  
有四处需要在开头就厘清。  
  
其一是**编号问题**  
。这不是一条有 CVE 的漏洞。截至公开时它没有被分配 CVE 编号，多家安全聚合站都明确写了这一点。公众号行文里容易顺手给个"CVE-XXXX"让文章显得正式，但本文不会这么做：没有就是没有，这是事实核查的底线。  
  
其二是**官方与研究者口径高度一致**  
。iTerm2 官方在 3.7.9 的更新说明里写明：修正了一个安全问题，被构造的文件在终端显示时（例如用 cat  
）可能滥用 Kitty 图像协议与拖放协议执行任意命令；回复程序的消息不再包含可被注入到 shell 的控制字符，标题上报也做了同样加固。这与研究者的技术分析对得上，属于少见的"厂商公告即确认"情形。  
  
其三是**"逻辑缺陷"这个定性的分量**。研究者特意强调这是逻辑 bug，不需要内存破坏，且在默认安装上 100% 可复现。这意味着传统的"堆溢出防护""地址随机化"在这里完全不起作用，因为根本没有内存越界，问题出在数据流向与信任边界的设计上。  
  
其四是**证据边界**  
。研究者只做了概念验证，演示的是启动计算器这类无害命令；他明确写了没有隐藏执行输出：注入命令执行时终端会出现可见的乱码报错，命令本身在乱码之间也是可见的，隐藏这段输出不在本研究范围内。把这条漏洞写成"无痕静默执行"会超出证据能支撑的范围。另有几条开放问题公开信息并未说明：官方公告未给出技术细节、未说明是否有更广的攻击面、也未披露发现时间线，这些保持开放。  
## 二、攻击面：一次 cat 就够  
  
攻击载体是被嵌入文件里的 APC 转义序列。APC 是 ANSI 转义序列的一种，以 ESC _  
 开头、ESC \  
 结尾。攻击者把它藏进任意文件，README、日志、配置、源码都可以，前提只有一个：文件内容会被输出到终端。  
  
当文件内容经 cat  
 冲到屏幕上，原始字节被交给 iTerm2 的 VT100 解析器。这里出现了一个反直觉之处：**APC 序列会被解析器消费掉，不会出现在 cat 的输出里**  
。用户看到的是一份看起来完全正常的文档，看不到藏在中间的转义字节。换句话说，"看不见"是这条链天然的伪装，但它伪装的只是载荷本身，不是执行结果。  
  
触发之后会发生什么：iTerm2 把一段错误响应写回伪终端的标准输入，shell 把这段数据当成用户敲进去的命令来读，于是执行了注入的内容。由于响应里带着换行符，shell 的行规整会把数据切成多行处理。研究者给出了一个直观的分解：  
```
/tmp/x\nopen -a Calculator\nz
       ^^^^^^^^^^^^^^^^^^^^
       injected command (between newlines)
```  
  
文件路径 /tmp/x  
 被刻意构造成包含一个换行、一条命令、再一个换行。命令被夹在两个换行之间，正好被 shell 当作独立的一行来执行。  
  
关于适用 shell，这一点需要说清：bash  
、zsh  
、fish  
 全部受影响，与用哪个 shell 无关。原因在链条的落点是伪终端的输入行规整，那是终端层与内核行规整的行为，还没到 shell 解析命令行那一步，所以换 shell 躲不开。真正决定成败的是几个与终端实现相关的细节，下一节展开。  
## 三、根因：三个缺陷拼成一条链  
  
研究者的分析把问题拆成三个独立缺陷。单独看每一个都不致命，串起来才构成了从"显示文件"到"执行命令"的完整通路。  
### 3.1 缺陷一：APC 处理没有信任检查  
  
iTerm2 的 VT100 解析器收到 APC 序列时调用 executeAPC:  
。研究者照录的反编译逻辑如下：  
```
// sources/VT100/VT100Terminal.m — executeAPC:

- (void)executeAPC:(VT100Token *)token {
    if (_tmuxMode) {                                                // [A]
        [_delegate terminalSetWindowTitle:...];
        return;
    }
    if ([token.string hasPrefix:@"G"]) {                            // [B]
        iTermKittyImageCommand *command =
            [[iTermKittyImageCommand alloc]
                initWithAPCString:[token.string substringFromIndex:1]];
        if (!command) {
            return;
        }
        [_delegate terminalDidReceiveKittyImageCommand:command];    // [C]
    }
}
```  
  
看它检查了什么：在 [A] 处检查 tmux 模式，在 [B] 处只检查字符串是否以 G  
 开头，在 [C] 处就构造 Kitty 图像命令交给代理。对照 OSC 序列的处理会发现，OSC 处理里存在的 terminalIsTrusted  
 检查在这里**完全缺席**  
。任何到达 VT100 解析器的 APC 都会被无条件执行，不关心来源是否可信。  
  
委托链上也没有补救。命令被直接转给图像控制器：  
```
// sources/VT100Screen/VT100ScreenMutableState+TerminalDelegate.m

- (void)terminalDidReceiveKittyImageCommand:(iTermKittyImageCommand *)command {
    [_kittyImageController execute:command];                        // [A]
}
```  
### 3.2 缺陷二：错误信息保留了换行  
  
图像控制器收到 t=f  
（文件传输）命令时，会解码 base64 载荷并把它当作文件路径。问题出在路径不存在时的错误处理：  
```
// sources/InlineImages/KittyImageController.swift — handle(command:fileURL:query:)

private func handle(command: KittyImageCommand.ImageTransmission,
                    fileURL: URL,
                    query: Bool) -> String? {
    do {
        let values = try fileURL.resourceValues(forKeys: [.isRegularFileKey])  // [A]
        guard values.isRegularFile == true else {
            return "EBADF:Not a file"
        }
        guard filenameIsSafe(fileURL: fileURL) else {                           // [B]
            return "EBADF:Invalid path"
        }
        let content = try Data(contentsOf: fileURL)
        return handle(command: command, data: content, query: query)
    } catch {
        return "EBADF:\(error.localizedDescription)"                            // [C]
    }
}
```  
  
这里藏着一个很精巧的顺序问题。在 [A] 处 resourceValues()  
 检查文件是否存在，文件不存在就抛异常；在 [B] 处有一个 filenameIsSafe()  
 的黑名单检查，但这段代码只有在 resourceValues()  
 成功之后才会走到，对不存在的文件，它永远不会执行。于是在 [C] 处，error.localizedDescription  
 被直接拼进错误信息。而 macOS 的 NSError.localizedDescription  
 会**原样保留文件路径里的换行符（0x0A）**  
，不做转义。  
  
这就是换行进入错误响应的入口。攻击者要做的是让路径不存在，好让流程掉进 catch 分支。  
### 3.3 缺陷三：响应被无条件写回伪终端  
  
错误信息接下来被包装成 APC 格式的响应发出去：  
```
// sources/InlineImages/KittyImageController.swift — respondToTransmit()

func respondToTransmit(_ imageTransmission: KittyImageCommand.ImageTransmission,
                        display: KittyImageCommand.ImageDisplay?,
                        error: String?) {
    if imageTransmission.more == .expectMore {
        return
    }
    var args = [String]()
    args.append("i=\(imageTransmission.identifier)")
    // ...
    let payload = args.joined(separator: ",") + semi + (error ?? "OK")
    let message = "\u{1B}_G\(payload)\u{1B}\\"

    switch imageTransmission.verbosity {
    case .normal:                                                               // [A]
        delegate?.kittyImageControllerReport(message: message)                  // [B]
    case .query:
        if error != nil {
            delegate?.kittyImageControllerReport(message: message)
        }
        return
    case .quiet:
        return
    }
}
```  
  
默认的详细级别 .normal  
 在 [A] 处**总会**  
发送响应，不管这是不是一条错误信息。在 [B] 处响应被包装成 ESC_G...ESC\  
 的 APC 格式交给代理。  
  
委托链的末端是问题落地的时刻：  
```
// sources/VT100Screen/VT100ScreenMutableState.m — kittyImageControllerReportWithMessage:

- (void)kittyImageControllerReportWithMessage:(NSString *)string {
    [self willSendReport];
    __weak __typeof(self) weakSelf = self;
    const NSStringEncoding encoding = self.terminal.encoding;
    [self addSideEffect:^(id<VT100ScreenDelegate> delegate) {
        [delegate screenSendReportData:                                         // [A]
            [string dataUsingEncoding:encoding]];
        [weakSelf didSendReport:delegate];
    } name:@"kitty image report"];
}
```  
  
在 [A] 处，screenSendReportData:  
 把数据写入伪终端。这个函数发送数据的路径和键盘输入是同一条，所以 shell 会把这些数据当成用户亲手敲进去的内容。到这里，链条闭合：文件里的字节 → 解析器消费 → 错误信息保留换行 → 响应写回 stdin → shell 按行执行。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0GkRkkSLCpcTReJkWenoRVJAVAs1joiaHXiah3Cj3lAOjKN5MgyhPtMlzicbRR3YD9WcYRPfso7g1XXAGibMDNgEa1YdAlt0lbMutY/640?wx_fmt=png&from=appmsg "")  
## 四、为什么三道现有防护都没拦住  
  
这条漏洞最值得防守方琢磨的，不是某个函数的 bug，而是几道本来应该起作用的安全控制为什么集体失效。研究者整理了一张对照表：  
<table><thead><tr style="box-sizing: border-box;"><th data-colwidth="198" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">安全控制</span></section></th><th data-colwidth="102" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">状态</span></section></th><th data-colwidth="264" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">失效原因</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td data-colwidth="198" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">filenameIsSafe()</span></code></td><td data-colwidth="102" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">被绕过</span></section></td><td data-colwidth="264" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">位于 </span><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">resourceValues()</span></code><span leaf=""> 抛异常之后，对不存在的文件不可达</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="198" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">terminalIsTrusted</span></code></td><td data-colwidth="102" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">缺席</span></section></td><td data-colwidth="264" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">executeAPC:</span></code><section><span leaf=""> 里没有这项检查（OSC 里有）</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="198" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">disablePotentiallyInsecureEscapeSequences</span></code></td><td data-colwidth="102" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">不适用</span></section></td><td data-colwidth="264" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">不覆盖 APC 序列</span></section></td></tr><tr style="box-sizing: border-box;"><td data-colwidth="198" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">默认详细级别</span></section></td><td data-colwidth="102" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><section><span leaf="">反而不利</span></section></td><td data-colwidth="264" style="box-sizing: border-box;border: 1.11111px solid rgb(233, 233, 231);padding: 6px 12px;"><code style="box-sizing: border-box;font-family: &#34;JetBrains Mono&#34;, &#34;SF Mono&#34;, &#34;Cascadia Code&#34;, Menlo, Consolas, &#34;Fira Code&#34;, &#34;Source Code Pro&#34;, &#34;DejaVu Sans Mono&#34;, &#34;Liberation Mono&#34;, &#34;Ubuntu Mono&#34;, &#34;Noto Sans Mono&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, monospace, &#34;SoloMD CJK&#34;, &#34;Apple Color Emoji&#34;, &#34;Segoe UI Emoji&#34;;font-size: 0.9em;background: none 0% 0% / auto repeat scroll padding-box border-box rgb(240, 238, 232);padding: 0.15em 0.4em;border-radius: 4px;"><span leaf="">.normal</span></code><section><span leaf=""> 是默认值，错误响应总是被发送</span></section></td></tr></tbody></table>  
四行里有三行是"根本没走到"，一行是"默认设置帮了攻击者"。这种组合在代码审计里很难靠单点检查发现，因为每一处单看都符合某种设计逻辑：黑名单检查放在文件存在性检查之后，是因为正常流程下文件都存在；.normal  
 默认发送响应，是为了让程序能收到图像传输的回执；APC 不加信任检查，是因为它被认为是内部协议。问题恰恰出在"正常流程"之外的那条路径上。  
  
还有一个容易被忽略的细节，关于**为什么不能用 ASCII 双引号**  
。研究者解释，如果文件路径里包含 ASCII 双引号 "  
，它会原样出现在错误信息里，被送到 shell 后，bash 会在遇到引号时进入引号模式，换行符就不再充当命令分隔符，注入失效。macOS 的 NSError.localizedDescription  
 用的是 Unicode 弯引号 “ ”  
，在 bash 里不是特殊字符，不会触发引号模式，于是换行继续充当分隔符。这是一个很典型的"字符编码的偶然差异改变了安全语义"的例子。  
## 五、补丁：为什么加信任检查没用  
  
iTerm2 开发者确认后用两次提交修复了这个问题。  
  
有意思的是，研究者特意分析了一个看似合理但其实无效的方案：**给 APC 处理器加上 terminalIsTrusted 检查并不能保护默认安装**  
。原因是 terminalIsTrusted  
 只是高级设置"禁用潜在不安全的转义序列"的反面，而那个设置在默认状态下是**关闭**  
的，也就是说终端默认就是被信任的。加这项检查，对绝大多数默认配置的用户等于什么都没做。真正堵住洞的是响应净化。  
  
头一次提交新增了 sanitizedResponseMessage(_:)  
，在响应被封装发送之前，把所有 C0 控制字符（0x00–0x1F，含 ESC、CR、LF）和 DEL（0x7F）替换成空格：  
```
+    private static func sanitizedResponseMessage(_ string: String) -> String {
+        var result = ""
+        result.unicodeScalars.reserveCapacity(string.unicodeScalars.count)
+        for scalar in string.unicodeScalars {
+            if scalar.value < 0x20 || scalar.value == 0x7f {
+                result.unicodeScalars.append(" ")
+            } else {
+                result.unicodeScalars.append(scalar)
+            }
+        }
+        return result
+    }
+
-        let payload = args.joined(separator: ",") + semi + (error ?? "OK")
+        let payload = args.joined(separator: ",") + semi + (error.map { Self.sanitizedResponseMessage($0) } ?? "OK")
```  
  
第二次提交把这个净化能力抽成 NSString  
 上的共享方法 stringByReplacingControlCharactersWithSpace  
，把覆盖范围从 C0 扩展到 C1 字符（0x80–0x9F），同时保留 ZWJ、双向标记这类 Unicode 格式字符（Cf），避免破坏正常的国际化文本。它替换掉了每个文件各自的净化函数，并应用到三条上报路径：Kitty 图像、Kitty 拖放（OSC 72）以及标题上报。拖放那条路径原本只剥离 CR 和 LF，现在改成剥离全部控制字符：  
```
    static func sanitize(_ value: String) -> String {
        return (value as NSString).replacingControlCharactersWithSpace()
    }
```  
  
这个补丁思路很值得记住：它没有去追着某一个攻击载荷做特征匹配，而是在"终端向程序回复数据"这个统一出口上做净化。所有可能被写回 stdin 的响应，无论来自图像协议、拖放还是标题，都先过一遍控制字符过滤。这类"在汇聚点做净化"的修法，比"在每个调用点做判断"更不容易遗漏。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zNsFJyIuL0HDZVCaBsTP2lrRricFQBgnfrfOCueN9WsdPbq3Is4ibj2l76CE6MTVLfeLANcnAfo8eMAwqUesFibqFH5PlctC7XL1hQUtLtFsNA/640?wx_fmt=png&from=appmsg "")  
## 六、自查与处置  
  
**先确认版本。**  
 在 iTerm2 里查看版本，凡是 3.6.1 至 3.7.8 之间的都在影响范围内。升级到 3.7.9 或更高是根本处置，这个版本已经把三条上报路径的控制字符全部净化。  
  
**升级前先收敛行为。**  
 如果暂时无法升级，避免在自己常用的终端里 cat  
 来源不明的文本文件，尤其是从邮件附件、共享目录、代码仓库里拿到的 README、日志、配置文件。这不是根本解法，只是降低触发概率，因为载荷的藏匿没有任何肉眼可见的特征。  
  
**理解防护选项的边界。**  
 打开"禁用潜在不安全的转义序列"这一高级设置，看起来是个稳妥的选择，但注意它并不覆盖 APC 序列，对这条漏洞无效。同样，换用别的 shell 也躲不开，因为问题出在终端写入 stdin 的环节。真正有效的只有升级。  
  
**区分无 CVE 与无风险。**  
 这条漏洞没有 CVE 编号，也没有在野利用报告，但官方已确认并以安全更新形式修复。资产清点时不要因为"查不到 CVE"就把它从队列里划掉，按版本号和厂商公告判断更可靠。  
  
**给团队的提醒。**  
 如果你的团队把终端安全等同于"别乱点链接"，这条漏洞是个很好的反例：显示一个文本文件就够了。可以把这次的教训纳入开发者的安全培训：终端不是被动的显示器，它会解析收到的数据，并把某些数据写回输入流。  
## 七、结束语  
  
把这次事件浓缩成一句话：**终端收到的字节，和它写回 shell 的字节，必须用同一套信任标准来审视**  
。iTerm2 这条漏洞的根源，在于把"程序发给终端的响应"默认当成了可信数据，直接写进了 stdin，却忘了这段响应里的一部分内容其实来自外部文件。三次看似合理的设计决定，APC 不加信任检查、错误信息保留原始字符、响应无条件回写，叠加起来，就跨过了从数据到代码的边界。  
  
它同时提醒我们，逻辑缺陷的审计不能只盯单点。真正危险的不是某行代码写错了，而是几段各自"符合设计意图"的代码在数据流的某个拐点接在了一起。补丁选在汇聚点统一净化，正是对这种结构性问题的正确回应。对普通用户而言，结论更简单：把 iTerm2 升到 3.7.9，别在老版本里 cat  
 来路不明的文件。  
  
