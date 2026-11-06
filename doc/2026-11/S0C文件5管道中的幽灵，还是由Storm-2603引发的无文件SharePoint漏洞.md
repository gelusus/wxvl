#  S0C文件5:管道中的幽灵，还是由Storm-2603引发的无文件SharePoint漏洞?  
Securelist
                    Securelist  网络与信息安全参考   2026-11-05 23:50  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Oy7lbA08GzaxIvZsb9WMEar9JTVX1yz1eKlvxDfVHkxicuMkicj7KUAt4FbSAl6XUUoiaFoamdiaa5KtI3x43usrxcejBdxqQeh4tJ5v3wV28hU/640?wx_fmt=jpeg&from=appmsg "")  
  
THE SOC FILES / 05  
2026.11.05  
  
管道里的幽灵  
 ·   
Storm-2603 无文件入侵  
  
  
一次针对本地 SharePoint 的入侵复盘：ToolShell 漏洞利用之后，攻击者把访问层整个搬进了 w3wp.exe 的内存里。  
  
  
MEMORY  
  
无文件 · 无落盘 · 可检测  
  
ToolShell  
VPP WEBSHELL  
  
01  
  
PART  
  
执行摘要  
  
一份关于内存驻留入侵的速览  
  
2025 年，多个威胁行为体利用了本地部署 Microsoft SharePoint 中的   
CVE-2025-53770  
 与   
CVE-2025-53771  
（即   
ToolShell  
 漏洞），通过构造的 HTTP 请求实现远程代码执行。其中一个行为体在利用成功后，在目标系统上投放了勒索软件。微软将与勒索软件相关的   
ToolShell  
 活动归因为 Storm-2603，并将其与其他行为体区分开来——后者观察到的活动主要集中在凭据窃取、间谍活动与数据外泄上。  
  
去年 12 月，我们的回溯分析识别出了与 Storm-2603 战术、技术与程序（TTP）一致的活动。该活动表现为：攻击者把 IIS 工作进程当作通往客户基础设施的  
内存桥接  
，将访问与控制层完全维持在   
w3wp.exe  
 内部，不向磁盘写入任何内容。尽管随后的后利用活动产生了磁盘痕迹，但用于维持访问的这座“桥”始终驻留在  
内存中  
。  
  
下面详细拆解这次攻击。  
  
![攻击链：从侦察到基础设施失陷](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzazcbNdTMWSDiblIPYibMdxYmgPwsS5dI6YspgtjRutjsMMukLvxLsVWoJBicdtC6eOlX5qlzKRu8QBAuy0eZgPQP7c3iaa1CqhH0s/640?wx_fmt=png&from=appmsg "")  
  
攻击链：从侦察到基础设施失陷  
  
02  
  
PART  
  
入侵时间线：战术性延迟与漏洞利用  
  
两个阶段，三天沉默  
  
本报告分析的入侵分为两个阶段，  
初始访问  
与后续利用活动之间存在一段时间空窗。攻击时间线如下：  
  
- **12 月 20 日——漏洞指纹识别**  
：单个 HTTP 请求打向与   
ToolPane  
 组件相关的 SharePoint 管理端点，用于判断主机是否存在   
CVE-2025-53770  
 漏洞。  
  
- **12 月 21—23 日——战术性沉寂**  
：初次探测之后是三天的静默期，把  
漏洞指纹识别  
与后续恶意活动隔开，同时可能降低被立即发现的可能性。  
  
- **12 月 24 日——高强度二次接触**  
：平安夜，许多公司的安全值守力量缩减，攻击者通过 IIS 工作进程（  
w3wp.exe  
）在最初探测的同一端点上执行了完整的 RCE 链路。  
  
- **12 月 26 日——事件发现**  
：三天的间隔最初使得 12 月 20 日的探测无法与后续恶意活动关联，事件被标记为低优先级。回溯分析才确认，那次探测正是后续入侵的初始阶段。  
  
  
  
![入侵时间线](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzZWH1jjUqIDvMD6hiank5Yic5s6J96ibewfbmnbF1NmZicLtIZ72Xj1p8quk5NxGIxmuuibzHZIe3Kz1TiaJctlALenA5v7ltvU82FOs/640?wx_fmt=png&from=appmsg "")  
  
入侵时间线  
  
03  
  
PART  
  
初始访问验证阶段  
  
先确认，再动手  
  
FINGERPRINT  
漏洞指纹识别  
一次不带恶意的探针  
  
12 月 20 日的一次  
漏洞指纹识别  
请求，是与目标之间可观察到的第一次交互。该请求来自与 ONYPHE 相关的基础设施——这是一个互联网规模的攻击面发现与管理平台，用于识别暴露资产、漏洞与网络威胁。与 Shodan、Censys 等被广泛使用的工具不同，后者只提供一份互联网暴露资产清单，并与已知漏洞的开源数据做关联；而 ONYPHE 会真正去检测目标系统是否存在漏洞。  
  
![使用 ONYPHE 进行指纹识别](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08Gzbc4ibicpDVCHNjBydpqWiagctic1Dzrd93w2iaWByPsgRryIRbylbfUiaeBdLe8HdhxOJtQJSTGgn9EGkRicIwkK7aJ1vHDszH0ibHtH4/640?wx_fmt=png&from=appmsg "")  
  
使用 ONYPHE 进行指纹识别  
  
该请求的特征与 ONYPHE 的漏洞检测行为一致：后者使用主动但非侵入式的探测，来识别暴露在互联网上、且受特定 CVE 影响的系统。针对   
ToolShell  
，ONYPHE 采用了 Niels Hofmans（hazcod）的思路，即尝试向目标服务器注入一个无害标记，以此判断其是否存在漏洞。  
  
![ONYPHE 官方的 CVE 检测器（ToolShell）](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzaJXpl5BYZ1taZQDQ7TPMibJxaA7xowyD9oLWIWruasShibaBiaAzycRib9ibU4NHnunnwT2hDHUmNupdBx10az90bhZ3M3dLdxRKgE/640?wx_fmt=png&from=appmsg "")  
  
ONYPHE 官方的 CVE 检测器（ToolShell）  
  
这一阶段相当于针对暴露在 HTTP 之上的 SharePoint 执行面做了一次验证：  
  
- 目标通过 ASP.NET 管道处理了结构化请求，且未拒绝。  
  
- 请求路径抵达了能够处理序列化输入的应用逻辑。  
  
- 在应用边界处，没有任何防护措施中断或改变执行流。  
  
  
  
大约 12 分钟后，出现了第二条来自 Pfcloud UG 相关基础设施的入站连接——这是一家以防弹托管（bulletproof hosting，BPH）模式运营的服务商。考虑到时间点以及该 IP 有记录的扫描与滥用历史，这次连接可能是针对新识别目标的一次后续侦察。此阶段并未尝试利用。  
  
![来自 Pfcloud UG（BPH）所属 IP 的连接](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzZhHBUxsl1EGOqkRBIU9IAkmQuRbArdOia6pjtHtfpR0Iicib4wOPGnvLw47k1zdcAfiaHHpvh3HGAhj78V2Spkm19GEepxq2dvCek/640?wx_fmt=png&from=appmsg "")  
  
来自 Pfcloud UG（BPH）所属 IP 的连接  
  
04  
  
PART  
  
主动利用：圣诞老人拿走了数据，还留下一扇后门  
  
平安夜的完整 RCE 链  
  
主要的恶意活动发生在 12 月 24 日。攻击者从 45.91.94.204（45.91.92.0/22，ASN 63473，HostHatch, LLC）向同一台 SharePoint 主机发送了构造请求，导致 IIS 工作进程（  
w3wp.exe  
）内执行了一段 WebShell。载荷没有写入磁盘，而是被直接加载进 ASP.NET 运行时，并在后续请求处理中被使用。该植入体在受影响的应用程序域（AppDomain）生命周期内持续驻留；一旦应用程序池回收或 IIS 重启，这份运行时状态即告丢失。  
  
ORIGIN  
攻击来自一个 SPN 节点  
出口 IP 背后还有一层  
  
IP 地址 45.91.94.204 是一个 Safing Privacy Network（SPN）节点。SPN 将每条连接经由包含多个中继节点的多层 VPN 路径转发，从而掩盖真实来源 IP。  
  
![SPN 节点页面](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzZ8gYMEc2NIibRSfaWQh9Pmica8Kxxjg6VI4gQn7ffpUD8zV0QTEqD5N0HVIfRhYZibkgCA1H1ZGtHg2IaibpuzJicUU0f8wEgrn6sY/640?wx_fmt=png&from=appmsg "")  
  
SPN 节点页面  
  
SPN 对外宣称能为互联网流量提供强隐私保护。其基础设施在很多方面与 Tor 相似，但存在若干差异，具备以下能力：  
  
- **速度**  
：接近 VPN 的性能，可支撑高速数据传输。  
  
- **三层支持**  
：支持 TCP、UDP、ICMP 等多种协议（Tor 仅支持 TCP）。  
  
- **应用路由**  
：无需浏览器即可直接建立隧道。  
  
- **多重身份**  
：可同时使用多个出口节点，以绕过基于 IP 的速率限制。  
  
- 固定长度的数据包与填充机制，使流量分析更困难。  
  
- 它比 Tor 少见，且经常能绕过主流的 Tor 黑名单。  
  
  
  
ARCHITECTURE  
WebShell 架构  
多模块内存框架  
  
攻击者注入到   
w3wp.exe  
 运行时中的 WebShell，是一个多模块的内存框架：执行由主组件（  
G.cs  
）编排，并通过跨多个命名空间的若干功能单元进行扩展。  
  
其实现遵循模块化的 C# 工程结构，每项能力被隔离在独立的编译单元中，并在运行时动态调用。  
  
![WebShell 组件构成](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzYMRds3zZjuxAsTuTLsbXIzfAlrJBAKfPV3PW4iaUo36pjjtESBFaVm5X5KLcBgpLMiaQoEbeQxuBVbOmgUkyKD67RWBKnZ2icKvY/640?wx_fmt=png&from=appmsg "")  
  
WebShell 组件构成  
  
### “G.cs”：管道控制与 C2  
  
  
**管道控制**  
  
WebShell 的主组件是   
G.cs  
。它负责编排植入体的执行上下文、生命周期、C2 与会话管理。  
  
![G.cs 注册虚拟路径提供程序](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzYR06orwph7HvHWQqAUfIaFj0ApKlHfmsticB3iaPSDiachx4X9otHugN0ZPlVWrqC9CWTtmKlSHDmHAPicelkTAQ6iaiazKicib7Kvbzo/640?wx_fmt=png&from=appmsg "")  
  
G.cs 注册虚拟路径提供程序  
  
为确保在运行时中持续存在，  
G.cs  
 执行以下动作：  
  
- 通过 ASP.NET   
HostingEnvironment  
 类中非公开的   
RegisterVirtualPassProviderInternal  
 方法注册一个自定义   
VirtualPathProvider  
，使 ASP.NET 在虚拟路径处理时使用它。该提供程序会被自动加入已注册虚拟路径提供程序链。  
  
- 将该提供程序的 .NET Remoting 生命周期策略委托给链上的前一个提供程序，从而保留既有的生命周期行为。  
  
  
  
此外，为了规避防御，并让   
VirtualPathProvider  
 即便在 WebShell 所在 AppDomain 的应用为预编译（precompiled）状态下也能完成注册，该模块还实施了信任控制颠覆（trust control subversion，  
T1553  
）技术：  
  
- 使用反射检查应用是否为预编译。  
  
- 提取管理应用编译的   
BuildManager  
 类的内部状态字段指针，并在运行时修改它们。  
  
- 再次使用反射，强制完成运行时注入。  
  
  
  
G.cs  
 使用   
GetCacheKey()  
 函数作为虚拟路径处理流程的拦截点。它处理发往 SharePoint 服务器的入站请求，检查请求头，从而把攻击者的请求筛选出来。  
  
![GetCacheKey 中的 User-Agent 判断与线程池调整](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzZPfC2pJ7qG9bKV5J4lxWeMR1YZMBiapxdulZwHicV1viaOAetOjcV2d7pZqPvpxicCA9nrubeNofbu7QtHTibVhTJ87ibhZANc71RvA/640?wx_fmt=png&from=appmsg "")  
  
GetCacheKey 中的 User-Agent 判断与线程池调整  
  
请求头中特定的 User-Agent 会触发 WebShell 执行。若请求不满足该条件，植入体保持休眠，使其流量更难被识别。  
  
![触发条件中硬编码的 User-Agent 字符串](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzZPPx3guibvYCiadowyuUhSB4RTrteHk85dnVxksar9ZzkauiacXhBLe7yX0VZJaoVWzbn9L3h7LYJWBrXIkXvfWAxNCE96caJ2rg/640?wx_fmt=png&from=appmsg "")  
  
触发条件中硬编码的 User-Agent 字符串  
  
**命令与控制层（内存传输）**  
  
同一个核心模块（  
G.cs  
）实现了通信层。  
  
![自定义二进制协议的封包与 XOR 混淆实现](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzbHBNibHctBLKahpTfpiaUEoebwJRiatRLQTnWRibKTdSyrCPacYQF41VcRejGNTsmktKLTdGL8fw0FaEqRV8pHIPOk0rV5WKSUib7w/640?wx_fmt=png&from=appmsg "")  
  
自定义二进制协议的封包与 XOR 混淆实现  
  
它不依赖外部框架或标准信标，而是使用自定义二进制协议，配合基于 XOR 的混淆与 C2 交换数据。这些通信混入正常应用流量之中。该模块利用   
GetCacheKey()  
 钩子拦截标准 HTTP 请求；当请求头中出现特定 User-Agent 时，模块会解析它、提取载荷，并在需要时与目标网络内部的一台主机建立 TCP 连接。  
  
![双向 TCP 隧道初始化](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzYSe8RnIbXrAHekBm3m2dvLBLjpFDRt2UHymDR3x6YRUv2tMgdSzRGaicu7DcDN4R6IccOibOdQWv95vkMNpHCpuibArHkKszrBdg/640?wx_fmt=png&from=appmsg "")  
  
双向 TCP 隧道初始化  
  
G.cs  
 建立一条全双工代理隧道以支撑交互式控制。它在  
内存中  
保存活跃的目标套接字，用于配对并发的 HTTP 读、写通道。  
  
![套接字缓存与连接建立](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzZcgibVX9DcSsuqpRDVgLXozzoib2nCiazHvnhFzxOmnYgeHfDibYmusNLSDHhohTvaickwSk54nWaR1Tca3Kd4fzOX8ia95PXbrwXgU/640?wx_fmt=png&from=appmsg "")  
  
套接字缓存与连接建立  
  
从 C2 服务器到目标网络内任意主机的 TCP 连接均被支持，用于跳板与  
横向移动  
。流量经由 SharePoint 主机代理转发。  
  
![目标到攻击者的数据通道](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzbRWUISgXP5GdfDicHHtcacE2z2SwkMHOKbayyCTAc3sMqIJibbhD01TZJ8eog5XuLroyrI5qibibdR2vZKiadvZL9AkuQEen7fGibQw/640?wx_fmt=png&from=appmsg "")  
  
目标到攻击者的数据通道  
  
![攻击者到目标的数据通道](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzYlicSNlXiaOIjCKDIB5zM6kzDLLdNya37YznmiaBu11g3vHCLb7hzF7fUTMibibrFeM3SueXvia6NFCfpK1x07G77IE3uywuQKZBLGo/640?wx_fmt=png&from=appmsg "")  
  
攻击者到目标的数据通道  
  
从截图中可以看到，攻击者使用 suo5 工具实现流量代理。其作者将该工具宣传为一款稳定的正向代理工具，适用于出站互联网连接被禁止的场景。  
  
为提升性能，  
G.cs  
 对   
ThreadPool  
 进行了操纵：抬高并发操作线程数上限与最大超时。  
  
![线程池操纵](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzYBRO6A3WARV51mfj348Q4cweMqtQH4gPqzB0w3I6bdT90m7WLib1VzA4gV5X43YzSZj8B5IW34hXWgvJ57HGjTYTR6CFwkmtjY/640?wx_fmt=png&from=appmsg "")  
  
线程池操纵  
  
### “Prog.cs”：执行入口与任务调度器  
  
  
Prog.cs  
 是入口点，负责接收命令行参数并将其传递给 uk/  
London.cs  
。后者充当编排者，处理命令解析与权限提升（通过命名管道模拟）。我们稍后会详细分析它。若传入的是 “suo5” 命令，  
London.cs  
 会触发   
G.cs  
 以内存 Shell 的形式挂入 ASP.NET 运行时。一旦激活，  
G.cs  
 就在 Web 服务器内存内部独立处理入站 Web 流量。  
  
![Prog.cs 的命令分发逻辑](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzYIKWWnP8WHuwkibrxnsmJk8emSKEddOQFIEsiab5DrlY0QCnko9Wok7kBJAbibmibeMAz0sLzZ8BzmzLaoOvWMfoYrISBUibTp3GLc/640?wx_fmt=png&from=appmsg "")  
  
Prog.cs 的命令分发逻辑  
  
### 主机交互模块  
  
  
以下模块提供直接的系统交互能力，各自实现为一个独立单元。当   
London.cs  
 收到来自   
Prog.cs  
 的相应命令时，会触发这些模块。  
  
**"FileDownloader.cs"**  
  
![FileDownloader.cs 实现](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzbcKCR5eicXNkH2gibHesH4nlicIdRk2aD4h0taNzH3ibMBWibGoeQByOkRuOs6fRsh8ibpa335FTibq3pPVnOSAM20spmG6b1SZWzsJ4/640?wx_fmt=png&from=appmsg "")  
  
FileDownloader.cs 实现  
  
FileDownloader.cs 模块从 C2 提供的外部 URL 下载任意文件。它在无需任何用户交互的情况下，把下载的文件存放在 Program Data 目录中。  
  
**"LoggedOnUsers.cs"**  
  
![LoggedOnUsers.cs 实现](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzYicanwOevJZ2aZFiamK7qV2m3Dm0UA8E0lOPaYUE3kfHTUbX0wZOSw7vr3UpOkntr7gkp5uC6Lyc1ZmzXeHTI47fcSCEMQAzIxE/640?wx_fmt=png&from=appmsg "")  
  
LoggedOnUsers.cs 实现  
  
该模块枚举已登录用户，并识别其会话类型（交互式、服务、远程）。结果保存在  
内存中  
。攻击者可通过   
G.cs  
 提供的代理与隧道能力将其取回。  
  
**"ProcessList.cs"**  
  
![ProcessList.cs 实现](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzZEggib4wn2zEziaaib7vicMCO5WfFgz4FSEypSaWBiaIDcVhF1f9dnEiaAynfJLj1pIVTEX98Po0Ds5dHiccA1UnPv3Pgss7ISHIP1jI/640?wx_fmt=png&from=appmsg "")  
  
ProcessList.cs 实现  
  
该模块列出正在运行的进程，并给出对应可执行文件在磁盘上的完整路径。结果保存在  
内存中  
，攻击者可经   
G.cs  
 取回。  
  
**"NetworkInfo.cs"**  
  
![NetworkInfo.cs 实现](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzbrVAxF4P0CafUmvL0CziakmDxnWqc7vEwa5nmV02IRU7M9VsPnv29hDPskjGUI4jkPDMGAJ0hvalPU2MDKqJxgZAqiaYBDMKYKY/640?wx_fmt=png&from=appmsg "")  
  
NetworkInfo.cs 实现  
  
该模块提取现有网络连接的信息，包括协议、本地与远端 IP 及端口、连接状态。它还会枚举网络接口与请求头。结果保存在  
内存中  
，攻击者可经   
G.cs  
 取回。  
  
**"MSI.cs"**  
  
![MSI.cs 实现](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzZSTFUDNGrtvQ5HXwhHn8r8r0XLibh0DM6muB5XlzekJwcZ6C8b95yqiahaibv6HNtYJa3UCgU4pSuf3Iq0cEbyPO1ibDTqTuPvhdg/640?wx_fmt=png&from=appmsg "")  
  
MSI.cs 实现  
  
该模块静默安装并执行此前由 FileDownloader.cs 下载的 MSI 包。  
  
**"User.cs"**  
  
![User.cs 通过 SAMRPC 创建账户](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzYJibnnesricliaptn5CnZ4zbCcCTwm2ibuOpEa6rFV04IXaFDlxUffRIR8osP5Axfibuibszky1tQwzoctkwKU21lwSclGuXxFNXsA0/640?wx_fmt=png&from=appmsg "")  
  
User.cs 通过 SAMRPC 创建账户  
  
该模块使用原生 SAM RPC 子系统（  
SAMRPCNative  
 命名空间）创建管理员账户，而不是使用标准的命令行工具。这可能表明攻击者在试图规避检测，并为自身提供一条特权、隐蔽的访问通道——即便 WebShell 完成其使命之后，该通道仍能在目标系统上存续。  
  
### SAMRPCNative  
  
  
SAMRPCNative  
 命名空间为原生 Windows   
samlib.dll  
 库实现了一层低层 C# P/Invoke 封装，由三个类型组成：SAMStructs.cs、SAMMethods.cs 与 SAMEnums.cs。  
  
**"SAMEnums.cs"**  
  
![SAMEnums.cs 中的访问掩码枚举](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzbAvPt2fMZhaj0yeou1FIK4m2cbhtcIsrhTuqbLBPpx9Rf3iboAe74INZ3uxjgA1Op5ibFsoibpmsuwibMlziaaRibtB1oOFzsamYMJw/640?wx_fmt=png&from=appmsg "")  
  
SAMEnums.cs 中的访问掩码枚举  
  
实现结构化的位标志列表与常量标志，把数值编码映射为 SAM 中的安全含义，并转换系统状态返回码。  
  
**"SAMStructs.cs"**  
  
![SAMStructs.cs 中的原生结构体定义](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzY1F9OrpnNZGRl0raPD1ylECQqM6icjPth6ibqH863B7UMDg59KYFZoib02qdnkFgnJJll2sX21aXpJ3bY4qohlv2pMWeuKSMxl2Q/640?wx_fmt=png&from=appmsg "")  
  
SAMStructs.cs 中的原生结构体定义  
  
定义 P/Invoke 内存对齐与封送（marshalling）所需的原生数据结构。它对数据结构进行安全格式化与对齐。  
  
**"SAMMethods.cs"**  
  
![SAMMethods.cs 中的 samlib.dll 导入声明](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzafZGpNdGfvbOnydWwKrKCOUn6l1pEAEgRTQxSibwZXm9VWMB0L5cnkyqsvPdLjCckEW60RK8KxqzkKmnDdGbwQEGrvsTOvOXia8/640?wx_fmt=png&from=appmsg "")  
  
SAMMethods.cs 中的 samlib.dll 导入声明  
  
包含从   
samlib.dll  
 导入的外部 API 声明（P/Invoke），用于执行连接 SAM 服务器、创建用户、打开域等动作。  
  
这一层对攻击者至关重要，因为它使账户操作成为可能。  
  
### 权限提升  
  
  
uk 命名空间包含用于  
权限提升  
的攻击原语。为将权限提升至 SYSTEM，它使用了一种本地令牌模拟技术，通过从   
EfsPotato  
 工具借鉴的加密文件系统远程协议（  
EFSRPC  
）接口实现。  
  
![EfsRpcEncryptFileSrv 调用](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzbJsEEjQnuE8icicoKniblFkv3D2DgBYEvbcLV6NOv3MLsVUdx0NoVDlmun8pcyDA1QocdRib0qNsqoOcclMiaNYx9HUDaZX2uEEc8E/640?wx_fmt=png&from=appmsg "")  
  
EfsRpcEncryptFileSrv 调用  
  
该功能实现在 uk/London 类中。它先创建一个恶意命名管道，同时由 EfsrTiny 类（  
EfsPotato  
 工具的精简版本）连接   
LSASS.exe  
 的 RPC 服务器。随后，  
London.RpcThread  
 函数以构造的   
\\localhost  
 路径调用 EfsrTiny 中的   
EfsRpcEncryptFileSrv  
 函数，诱使   
LSASS.exe  
 反向连接攻击者的命名管道。最后，  
London.Exp  
 函数使用   
ImpersonateNamedPipeClient  
 强制并模拟   
LSASS.exe  
 的令牌，再通过   
CreateProcessAsUser  
 以 SYSTEM 权限派生新进程。由于 WebShell 驻留在 IIS 工作进程中，它天然具备执行该技术所需的   
SeImpersonatePrivilege  
 权限。  
  
![命名管道与令牌模拟流程](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08Gza3C1xjZVLTib18Aakicxp92Tr9KbEHYeJwiakcI5UlRH5Dq6t8MPKuop8woqfuKialpXo6MpV6jRsC4HQxniaOLIH3AfQrH983Mdfs/640?wx_fmt=png&from=appmsg "")  
  
命名管道与令牌模拟流程  
  
uk 命名空间还实现了一个最小化的 MSRPC 客户端层：通过显式定义   
RPC_CLIENT_INTERFACE  
、  
MIDL_STUB_DESC  
 等结构，伪造接口存根（stub）。这使得与 Windows RPC 运行时（  
Rpcrt4.dll  
）的直接交互成为可能，而不必依赖 MIDL 生成的存根或更高层的 RPC 抽象。由此获得的绑定与封送控制能力，超出了标准客户端 API 的限制。  
  
05  
  
PART  
  
后利用活动  
  
把 Velociraptor 当成执行框架  
  
WebShell 注入后不久，被感染的工作进程派生了   
msiexec.exe  
，用于安装托管在   
Supabase  
 云存储上的远程 MSI 载荷。该载荷是   
Velociraptor  
——一款开源的数字取证与事件响应工具，具体版本为 0.73.4.0，受   
CVE-2025-6264  
 影响——攻击者滥用它在目标网络中执行命令。  
  
2025 年 10 月，Cisco Talos 曾报告  
威胁行为体  
将   
Velociraptor  
 用作失陷后执行框架，并将该活动归因于 Storm-2603。该工具支持任意命令执行，可用于多种目的。在本案例中，它被用于  
持久化  
、发现、数据收集与载荷执行。  
  
威胁行为体  
利用   
Velociraptor  
 部署了多套相互独立的隧道机制，包括用于未授权远程访问的 Zoho Assist-Unattended Support、VS Code CLI 隧道、Cloudflare Tunnel，以及一个轻量级反向代理工具。通过这种方式，即便部分工具被检测并清除，他们仍试图维持访问。到了这一阶段，他们也不再依赖 IIS 工作进程的生命周期或   
ToolShell  
 漏洞。  
  
06  
  
PART  
  
杀 EDR 的尝试  
  
易受攻击的驱动 + 侧加载  
  
一旦威胁行为体取得目标系统的访问权限，他们便能上传一个被众多攻击者用来终止 EDR 方案的易受攻击驱动。我们在   
  
C:\programdata\rsndispot.sys  
（054a32d6033b1744dca7f49b2e466ea2）  
  
处检测到了该主驱动。此外，对手还使用了一个恶意 DLL（2d9ebf9c042f596f435865b40edad928）实施侧加载（sideloading）攻击。该 DLL 被识别为 Warlock 勒索软件的 AV 杀手（AV killer），其意图可能是借助存在漏洞的   
rsndispot.sys  
 驱动来终止安全解决方案。  
  
07  
  
PART  
  
勒索软件投放尝试  
  
被主动防御拦下的最后一步  
  
攻击者曾多次尝试在失陷系统上启动  
勒索软件  
。然而，Kaspersky Endpoint Security 的主动防御模块（Proactive Defense Module）拦截了勒索软件进程，有效阻止了该活动。我们获取到的恶意 DLL 被识别为 Warlock 勒索软件。  
  
08  
  
PART  
  
防守方真正该盯的是什么  
  
从文件狩猎转向运行时行为  
  
ARTIFACTS  
ToolPane 反序列化与 CLR 痕迹  
两个程序集同框出现  
  
当攻击者尝试访问   
ToolPane  
 时，动态编译生成的   
App_Web_toolpane.aspx  
 程序集与   
Microsoft.GeneratedCode  
 辅助程序集同时存在于同一个应用程序域中。  
  
这两个程序集本身都用于合法场景。  
App_Web_*  
 程序集在正常的 ASP.NET 页面编译过程中生成，而   
Microsoft.GeneratedCode  
 通常由   
XmlSerializer  
 在合法的序列化工作流中产生。然而，一次发往   
ToolPane  
 端点的请求，如果在同一执行窗口内立即加载了   
Microsoft.GeneratedCode  
，就说明运行时正在处理通过该请求提供的序列化数据。  
  
DETECTION  
检测反序列化利用  
盯住对象图的构造  
  
首先，强烈建议阅读这篇文章（俄文），了解应如何检测 CLR 中的  
反序列化  
利用尝试，其中包含 YARA 规则示例与监控建议。下面概述其检测逻辑的核心思路。  
  
反序列化  
漏洞应被视为一类独立的利用事件。在 CLR 中，反序列化不只是把字节转换成值——它是从输入流中重建一幅对象图（object graph）。序列化与反序列化由格式化器（formatter）执行。使用最广泛的格式化器之一是   
BinaryFormatter  
，它采用 NRBF 格式，该格式把流定义为一系列记录，用以描述类、数组、基元以及对象之间的引用。最终生成的图形态由提供给  
反序列化  
器的数据决定。这意味着输入流不仅定义了值，还定义了运行时必须具体化的具体类型、关系以及生命周期节点。  
  
微软明确指出   
BinaryFormatter  
 不安全，且无法被改造为安全；从 .NET 9 开始，其内置实现实际上已从常规使用中移除。但微软仍在其产品（包括 SharePoint）中继续使用它，因为它强大且易用。要修复与   
BinaryFormatter  
 相关的问题，需要架构层面的改动。类似的警告同样适用于相关的不安全机制，包括   
LosFormatter  
（即存在漏洞的那个页面所使用者）。  
  
不安全反序列化的首要风险在于：CLR 可能解析出意料之外的类型、在正常构造函数不变式之外实例化对象，并调用   
ISerializable  
、  
IObjectReference  
、  
IDeserializationCallback  
 等特殊生命周期机制，以及标记了   
[OnDeserialized]  
 的方法。换言之，攻击者控制的数据可以迫使运行时构造出一幅包含“带可执行副作用类型”的对象图。  
  
因此，一种实用的检测策略应当聚焦于图构造（graph-construction）产生的痕迹。CLR 加载器事件可用来识别相关程序集与应用程序域；随后通过内存与元数据检查，可以揭示与可疑利用链（gadget chain）相关的类型名以及生成的序列化器结构。在类   
ToolShell  
 的场景中，不安全反序列化通常会产生辅助程序集（其证据体现在加载器事件中），其元数据包含完全限定类型名、gadget 类名，以及所构造泛型类型的字符串表示（直接用 YARA 扫即可）。在本案例中，需要查找的痕迹包括   
Microsoft.GeneratedCode  
、  
App_Web_toolpane.aspx*  
（加载器事件中出现即证明发生了反序列化），以及标识   
ObjectDataProvider  
、  
ExpandedWrapper  
、  
LosFormatter  
 的字符串（说明输入流已被反序列化为危险对象）。需要注意的是，这些指标单独看并不显著；它们的价值在于表明运行时已经接收了需要生成辅助代码、以重建危险对象图的数据。  
  
利用   
ToolShell  
 一类漏洞的 gadget 对象，在   
ysoserial.net/YSoNet  
 这类语料库式工具中有便利的汇总，它们收集了针对不安全 .NET   
反序列化  
的已知 gadget 链。从防御角度看，这些项目的价值不在于其攻击能力，而在于那份类型与结构目录——可用于在流、内存与辅助程序集中搜索。它们中的任何一个出现在内存或辅助程序集里，都应被视为利用行为的有力指标。  
  
为简化  
反序列化  
利用检测，我们开发了一款名为   
CLRDeserializationExploitFinder  
 的工具。它能够检测   
ToolShell  
 的利用尝试，以及 ysoserial.net 覆盖的其他 gadget 链。  
  
HUNTING  
检测基于 VPP 的 WebShell / 运行时植入体  
ETW 事件 25 与 47  
  
基于   
VirtualPathProvider  
 的机制应被视为一类独立的运行时植入体（全程不涉及文件）。在 ASP.NET 中，  
HostingEnvironment.RegisterVirtualPathProvider  
 会在应用的编译系统中注册一个新的虚拟路径提供程序（VPP）。一旦注册完成，应用在动态编译期间就能把虚拟路径当作磁盘上的普通页面来提供。动态编译不要求载荷以文件形式存在于文件系统中，整个过程完全运行在应用自身的信任边界之内。编译过程中可能会生成临时文件，但不能把它们当作可靠的取证痕迹。由于 VPP 同样可用于合法用途，防御模型应当从清点合法的   
VirtualPathProvider  
 实现开始；除此之外，任何新 VPP 的注册或使用都应视为异常。  
  
想了解使用 ASP.NET、IIS 与 CLR 动态生成虚拟页面的运作方式，以及如何检测此类活动，可以使用我们的   
VirtualPathProvider  
 Lab 项目。  
  
![VirtualPathProvider Lab 界面](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08GzZqzkJaunxCjVVyZ6L3BaUU550IBia8Qosic85Zv0RVIx4KJwQmww1MgDFia4bE0dic2ibH7KiaHkicrQoWUdGozqkpBUbmpviah96do3k/640?wx_fmt=png&from=appmsg "")  
  
VirtualPathProvider Lab 界面  
  
针对该技术最有用的遥测来源是   
Microsoft-Windows-IIS-Configuration  
 ETW 提供程序。要检测虚拟路径，事件 25 与事件 47 最具价值。事件 25 记录   
ConfigPath  
 到   
PhysicalPath  
 的映射，事件 47 记录配置路径到目录的映射。对于由 VPP 支撑的路径，这些事件会产生不一致：IIS 与 ASP.NET 表现得仿佛该路径有物理实体，而对应文件在磁盘上并不存在。在   
VirtualPathProvider  
 Lab 给出的示例中，这表现为虚拟资源   
googlecheck.aspx  
 被映射到一个并不存在真实文件的物理路径。配置解析与文件系统实际状况之间的这种偏差，是虚拟内容的首要指标。  
  
![事件 25 与 47 记录虚拟路径映射](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzblkYZtNpAVBBaZ2nbdOM0oT5IMfhmib2MG9aXib5g3MTTYwMRVkK8QLAzTjBDqgDU3BXMlP61jd0S8fWs8wksohsMWfTkhcbUw4/640?wx_fmt=png&from=appmsg "")  
  
事件 25 与 47 记录虚拟路径映射  
  
第二个指标是“该有的没有”，而非“不该有的有了”。对于磁盘上的文件，IIS 通常会创建   
FileChangeNotificationMonitor  
 链，体现在事件 17、18、19 与 21 中。而对于由自定义   
VirtualPathProvider  
 提供的虚拟路径，这条链缺失，或与普通文件的正常画像不符，因为并不存在物理对象或相应的磁盘变更生命周期。因此，“事件 25 与 47 出现”加上“同一路径缺少预期的事件 17、18、19 与 21”，就构成了一种可靠的检测模式：该路径对 IIS 而言是配置解析出的资源，却既无文件系统支撑，也没有伴随正常的文件变更监控。  
  
![事件 17 等文件变更监控链](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy7lbA08Gzay7t1QRDAjNkGqZJTT9IXjhxbfULkI2Wt8ep8uD4cmbP8KjD6YA7kgLG3IDbkyvwicxzq995cXoAz5FQIg0XJ9IYOnU69Mm3Ow/640?wx_fmt=png&from=appmsg "")  
  
事件 17 等文件变更监控链  
  
总结来说，这里的狩猎目标是 IIS/ASP.NET 解析资源的方式，与磁盘上实际存在的内容以及变更监控链之间的偏差。对于已知应用，可通过一份合法   
VirtualPathProvider  
 实现的白名单来过滤此类情况；其余一切情况都应视为潜在的运行时植入体，并用 ETW、文件系统核验以及对   
w3wp.exe  
 进程的内存检查来审查。  
  
09  
  
PART  
  
归因  
  
中等置信度  
  
微软将 Storm-2603 作为一个独立  
威胁行为体  
跟踪，未发现其与其他组织有关联。虽然微软在评估该行为体的动机时保持谨慎，但提到该行为体此前曾在积极利用本地部署 SharePoint 漏洞之后投放 Warlock 与 LockBit 勒索软件。在本次分析的入侵中，我们观察到了类似的 SharePoint 利用活动，以及与 Warlock 勒索软件关联的 EDR 杀手和未成功的 Warlock 启动尝试。  
  
其他已记录的入侵事件显示，Storm-2603 会利用边界邮件网关中的已知缺陷取得  
初始访问  
，随后以提升后的权限部署   
Velociraptor  
 框架，并通过商业远程管理软件建立持久加密隧道，以统筹二次  
横向移动  
。这与我们在本案例中观察到的后利用行为一致。结合   
ToolShell  
 利用场景，我们以中等置信度（moderate confidence）将本案例归因于 Storm-2603。  
  
10  
  
PART  
  
结论  
  
检测思路必须换一层  
  
从本文分析的这起事件中，最重要的防御启示是：利用一台暴露在互联网上的 SharePoint 服务器，足以在应用  
信任边界  
内部直接实现无文件代码执行。Storm-2603 并不需要传统的投放器、持久化服务或暴露的 C2 端点；它在   
w3wp.exe  
 内部部署了一个 WebShell，并由此统筹隧道、  
权限提升  
、凭据访问与横向移动。因此，检测思路必须从基于文件的 WebShell 狩猎，转向运行时行为——例如   
w3wp.exe  
 的异常活动、异常的出站连接、RPC 强制认证、令牌模拟，以及应用失陷后引入的远程管理或 DFIR 工具。  
  
要正确归因恶意活动并制定恰当的响应措施，防守方应重建跨应用层、主机层与身份层的执行链及时间关系。同时应保留内存与 IIS 遥测数据，把低信号的侦察行为与后续特权活动关联起来，而不是孤立地评估单个事件。一旦确认 SharePoint 漏洞被利用且发生了代码执行，就应触发主机级遏制、  
持久化  
狩猎、凭据与已暴露 MachineKey 的轮换，以及由 TTP 驱动的调查，而不是仅仅针对初始漏洞或已知 IOC 做补救。  
  
卡巴斯基产品将本文提到的威胁检测为：  
  
- Trojan.MSIL.Agent.gen  
  
- Trojan.Win64.DLLhijack.cdp  
  
  
  
11  
  
PART  
  
失陷指标  
  
可直接入库的指标清单  
  
指标  
  
说明  
  
45.91.94[.]204  
  
SPN 节点  
  
0xFD91321BF7FF7245DDA9B7B5D791ACE8  
  
vskp2 —— WebShell 植入体  
  
hxxps://vdfccjpnedujhrzscjtq[.]supabase[.]co/storage/v1/object/public/image/v4[.]msi  
  
存在漏洞的 Velociraptor 安装包  
  
0x3A55D8F8F29716C694671834D8C6CB29  
  
C:\ProgramFiles\Velociraptor\Velociraptor.exe  
  
hxxps://auth[.]qgtxtebl[.]workers[.]dev/  
  
Velociraptor 服务器 URL  
  
199.232.210[.]172  
  
C2 流量中继  
  
0x991EC6E04413F2D4D29341C17A8A54D5  
  
被滥用的合法 VS Code  
  
0x3D553B9061C03F891CE97D7D2E64DDEE  
  
用于隧道的 VS Code Insiders  
  
91.224.92[.]121  
  
防弹托管 C2 服务器  
  
49.51.132[.]100  
  
SPN 节点  
  
hxxps://github[.]com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64[.]msi  
  
被滥用的合法隧道工具  
  
0x2C32E44A013CBFC5A323EFB7694701D2  
  
被滥用的隧道工具可执行文件  
  
hxxps://files[.]catbox[.]moe/8f7vex[.]msi  
  
被滥用的 Zoho Assist Unattended Agent 下载地址  
  
0x112A86ECA3CB5A0B5E20AB744CC5AF93  
  
为侧加载而改名的 7z.exe  
  
0xA930F81E3FB0A226EC5904D9043A17B8  
  
C:\ProgramFiles(x86)\ZohoMeeting\UnAttended\ZohoMeeting.7z  
  
204.76.203[.]219  
  
防弹托管 C2 服务器  
  
0x054a32d6033b1744dca7f49b2e466ea2  
  
存在漏洞的驱动  
  
SOURCE / 原文信息  
  
Securelist  
  
原标题：The SOC Files 5: Ghost in the pipeline, or a fileless SharePoint breach by Storm-2603  
  
作者：Alexander Rodchenko、Rodrigo Wanderson（Kaspersky GReAT）  
  
来源：Securelist（卡巴斯基官方博客），2026 年 11 月 5 日  
  
原文链接：https://securelist.com/storm-2603-fileless-toolshell-attack/121467/  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Oy7lbA08GzYvpBib3UUh1oXNdfmBk6CdJWa7j3mTo0hQJO49rysH6UlmsLhaSeJze3fjcPIVQRbMHPKI6ZxlmfToM65KeuuJssTFf7JrtPmw/640?wx_fmt=png&from=appmsg "")  
  
更多   
山东区域  卡巴斯基 反针对性攻击 解决方案  
 详情，请联系：  
  
高进 - 186 6376 1060  
  
如果你觉得今天这篇有收获，欢迎点赞、在看、转发三连，我们下篇见。  
  
赞  
  
  
在看  
  
  
收藏  
  
THANKS FOR READING  
  
  
