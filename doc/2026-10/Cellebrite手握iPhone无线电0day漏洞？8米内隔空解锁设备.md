#  Cellebrite手握iPhone无线电0day漏洞？8米内隔空解锁设备  
原创 黑鸟
                    黑鸟  黑鸟   2026-10-21 15:44  
  
有相关人士拿到了一份Cellebrite内部文档，标着CONFIDENTIAL，描述了一种"突破性方法"：  
能对iPhone 11到16全系列机型、在iOS 26至26.4+上实现AFU(After First Unlock)级别的数据提取。  
  
AFU这个概念在移动取证圈子里分量很重，它意味着用户首次解锁设备之后，取证工具可以绕过锁屏密码直接读取设备内的加密数据，是最核心的能力指标之一。  
  
Cellebrite给这项技术取名Advanced Unlocks，以下简称AU。  
  
别跟他们卖了多年的那个"advanced unlocking services"搞混，这是一条全新的产品线，今年早些时候的一份官方新闻稿已经证实AU是2026年的新东西。  
# AU到底是什么  
  
Cellebrite的宣传单页长这样：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibO9kiauylaDpNNTdYWRjrB6rdxVtKOhQhuyYH6JPbmr8l4yqk6Dn38ibOaUMT379MYdwtrHLDZy2aIoe5lvJQJJMwD8ICw2OmaRttCgicmFSbY/640?wx_fmt=png&from=appmsg "")  
  
Cellebrite Advanced Unlocks宣传页  
  
AU产品宣传页，2026年9月15日修订版  
  
按照Cellebrite的说法，AU具备以下特性：  
  
它是一种"下一代AFU提取向量，对Apple操作系统变更具有更强的抗性"。意思就是Apple更新iOS之后传统取证手段往往会失效，但AU号称更耐折腾。  
  
它能在Stolen Device Protection(被盗设备保护)开启的情况下正常工作。这个功能是Apple在iOS 17.3引入的，开启后会收紧USB接口的限制策略，对依赖USB通道的取证工具来说是道硬门槛。  
  
AU由一台专用设备驱动，标称有效距离约8米。  
  
没看错，8米。  
  
一个取证工具，有效距离8米。这不像是传统意义上的USB取证设备，更像是一种基于近距离无线电能力的东西。  
  
这份文档最早写于2026年3月，最后一次修订是9月15日。一个很显眼的缺失：iPhone 17和Air完全不在支持列表里，而这两款机型已经发售差不多一年了(18 Pro还没上市)。  
# 为什么是蓝牙  
  
AU的支持范围覆盖11到16全系列，却唯独缺少17系列。三次修订、跨越六个月，这个名单始终没变，看起来更像是一堵墙而不是一个还没做完的待办事项。  
  
要搞明白这堵墙，得先知道AU走的是哪条通道。  
  
Modem(基带)和NFC都试过，对不上号，Bluetooth(蓝牙)才是最吻合的选项。原因很直接：Apple官方给出的蓝牙有效范围大约10米，和AU标称的8米高度吻合。  
  
有一段时间，所有支持AU的iPhone都搭载Broadcom(博通)的蓝牙芯片，所以一度认定这是一个特定芯片家族的漏洞。后来又倾向于认为这是一条硬件无关的Bluetooth到UWB链路。但精确锁定AU依赖的具体漏洞，在当时看来可行性不大。  
  
后来AU的iOS支持开始掉队，这才有了突破口。  
  
"26.4+"这个写法也许只是"更新的版本也行，信我们"的意思，但AU明确声明支持的最高iOS版本和当时最新的iOS之间的差距一直在拉大：  
文档修订日期AU声称支持的最高iOS当时最新iOS落后版本数3月18日26.3+26.3.1(3月4日发布)05月20日26.4.226.5(5月11日发布)19月15日26.4+26.7/27(9月14日发布)3个版本，加上iOS 					27+  
  
如果某个iOS更新确实干掉了AU，那它最可能是在26.5里修复的。  
# bluetoothd里藏了什么  
  
把iOS 26.4(build 23E246)和26.5(build 23F77)的bluetoothd二进制拉出来做了diff。bluetoothd是iOS上管所有蓝牙连接的守护进程，权限很高，持有一大堆entitlements(权限声明)。  
  
26.5版本里这个守护进程冒出了一组安全相关的修改，全部集中在L2CAP(Logical Link Control and Adaptation Protocol，逻辑链路控制与适配协议)层。L2CAP是蓝牙协议栈的多路复用层，卡在蓝牙无线电硬件和上层应用协议中间，负责信令交互、数据分段和重组。  
  
用一张协议栈层次图来理解：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibO9kiauylaDqfG7kaYU0UicDXsVHtBFx9VdkcUExNwOGVbujW0E5bQb3hHMibcTQhMKREZTqsEjSibZk4bZlV7oOyg6822tymeQBXxo4BxibL9fk/640?wx_fmt=png&from=appmsg "")  
  
关键点在于，L2CAP信令PDU(Protocol Data Unit，协议数据单元)的处理发生在设备配对之前。任何处于BLE(Bluetooth Low Energy)范围内的设备都可以发这些数据包，不需要用户交互，不需要配对，不需要点任何弹窗。  
  
这些修改里，一个修复格外显眼：L2CAP channel manager中的UAF(Use-After-Free，释放后重用)漏洞。处理聚合L2CAP数据包的接收循环会遍历所有连接，调用各自的PDU处理函数。26.4版本中，如果某个处理函数在迭代过程中释放了自己对应的连接对象，循环还拿着悬空指针(dangling pointer)继续跑，典型的内存漏洞利用形态，把memory bug变成代码执行能力。  
  
这个漏洞代码至少从iOS 17(2023年9月)就存在了，在iOS 18时期Apple重构了L2CAP连接管理器之后，可能变得更容易被利用。  
  
26.5还收紧了L2CAP的其他几条路径，包括信令解析器的边界检查和传输切换验证，但这些都不像是AU的候选入口。  
  
加固没停在26.5。26.6在蓝牙协议栈其他位置(HCI事件处理)又加了边界检查，iOS 26.7和iOS 27修了[1]，一个没署名的蓝牙RCE(Remote Code Execution，远程代码执行)，恰好也落在同一个聚合数据包接收路径上，位置还更靠前。26.5刚加固过的代码，两个版本后又冒出一个RCE，Apple真的把所有洞都堵上了？  
# 推测的完整利用链  
  
碎片拼起来，AU的理论利用链大概长这样：  
  
第一步：Over the Air(无线注入)  
  
向目标设备发送精心构造的蓝牙L2CAP信令数据包。这一步发生在任何身份认证之前，不需要配对，不需要用户操作。攻击者只需要待在目标设备的蓝牙范围内。  
  
第二步：Into the Daemon(攻入守护进程)  
  
触发UAF漏洞劫持bluetoothd进程，执行攻击者控制的代码。第一和第二步构成了AU的核心机密。在MIE(Memory Integrity Enforcement，内存完整性执行)设备上，这一步大约有94%的概率会被拦截。失败会导致bluetoothd崩溃并重置BLE连接，但在实验室环境下，攻击者可以反复尝试几分钟直到成功。  
  
第三步：Into the Kernel(进入内核)  
  
从守护进程提权到内核控制。bluetoothd持有与多个内核驱动的直接IOKit连接，任何一个都可能是逃出沙盒的路径。这一步虽然重要，但跟前两步比起来技术含量相对常规。MIE同样可能在这一步构成阻碍，而且这里失败的代价更高：kernel panic(内核恐慌)会重启设备，重启后进入BFU(Before First Unlock)状态，AFU期间保存在内存中的加密密钥就全没了。  
  
第四步：Out the Door(数据出门)  
  
拿到内核控制权之后，大部分文件内容和一部分加密密钥就暴露了。USB Restricted Mode(USB限制模式)可以被解除，设备通过USB接口完成镜像提取。不需要破解锁屏密码，到了这一步，密码已经不太重要了。  
# 硬件代差：为什么iPhone 17不行  
  
AU支持的机型列表(11到16)连续三次修订都不包含17系列，这说明问题出在硬件层面而非软件适配进度。  
  
对比一下iPhone 16之后发生了什么变化：  
变化项17/17 Pro/Air17e能解释AU的缺失吗蓝牙芯片Apple 	N1取代Broadcom仍然是Broadcom不能，17e沿用了BroadcomMIE(内存完整性执行)支持支持能，所有缺失机型都支持  
   
  
如果AU依赖的是Broadcom芯片的特定组件，17e应该被支持才对，因为它没有换掉蓝牙芯片。  
  
MIE是唯一一个覆盖所有缺失机型的变量。它让内存损坏类漏洞变得更难利用，覆盖范围包括内核和多个用户态进程，bluetoothd正在其中。MIE设备和非MIE设备上运行的bluetoothd二进制文件在同一iOS版本下逐字节相同，拦截发生在硬件层面而非代码层面。  
  
MIE使用4位内存标签(memory tag)。攻击者盲猜标签成功的概率是1/16，也就是大约6.25%。换算一下，每次尝试被拦截的概率约94%。这不是一个绝对的防御，在实验室里多试几次总能碰上好运，但它确实大幅提高了利用门槛。  
  
另一份2026年3月的Cellebrite文档(此处不予公开)把"iPhone 11到16运行iOS 26+"明确归因于AU能力，而把iPhone 17列在了Inseyets通用能力(同意提取等常规取证手段)之下。17系列不是Cellebrite碰不了，而是AU这种方法在它身上不管用。  
  
一个全新的硬件级防御恰好撞上了号称"不怕软件补丁"的AU，这个时间巧合挺有意思。  
# Apple的沉默  
  
iOS 26.5悄悄修复了一个可以通过无线电远程利用的严重漏洞，但Apple在安全公告中对此只字未提。  
  
有时候Apple会事后补充安全公告，或者提到某个补丁但不致谢发现者，通常意味着漏洞是内部发现的。这次两样都没做。26.5安全公告里倒是有条模糊的CoreBluetooth致谢，没有CVE编号，但那修的是framework(框架)层，位置更靠上，跟守护进程不是一回事。而且这条致谢还是事后补上去的。  
  
Apple也不是头一回这么干了。一个严重漏洞被悄悄修掉，没有任何公开讨论，你很难不想：还有多少类似的修复是我们不知道的？  
# Cellebrite为什么要把事情说破  
  
Apple通常在安全公告里披露漏洞修复，Cellebrite通常把技术细节藏在闭门会议里，这份文档自己就写着"完整信息仅在会议中分享"。但这次角色反转了。  
  
Cellebrite的处境其实不难理解：iPhone平均安全水位不断升高，含含糊糊说"我们能解锁iPhone"已经不够有说服力了，checkm8这类曾经的主力利用手段在2026年早就过了保质期。Apple的安全加固正在拿到回报，USB接口这个取证主战场越来越难打，攻击者被逼着往更不好走的路径上挤。  
  
Cellebrite需要在市场上证明技术实力，不能光靠一句"信我"。但他们宣传AU"对Apple操作系统变更具有更强抗性"，结果iOS 26.5发布几个月后AU就哑火了，这打脸来得挺快。  
  
Apple这边也有自己的尴尬。蓝牙这条前线是不是也守不住了？下一场攻防战会在哪里打响？这些问题，大概没有谁愿意公开回答。  
  
相关阅读：[iOS 27.0.1完整攻击链：从消息一键RCE到Chollima持久化越狱](https://mp.weixin.qq.com/s?__biz=MzAxOTM1MDQ1NA==&mid=2451189367&idx=1&sn=44342bec2538bd6384c4b87bf94b62f7&scene=21#wechat_redirect)  
  
  
