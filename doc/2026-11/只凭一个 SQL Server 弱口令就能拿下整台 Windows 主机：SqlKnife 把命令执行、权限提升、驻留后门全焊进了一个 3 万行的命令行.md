#  只凭一个 SQL Server 弱口令就能拿下整台 Windows 主机：SqlKnife 把命令执行、权限提升、驻留后门全焊进了一个 3 万行的命令行  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-11-28 02:49  
  
重点导读免责声明
本文内容仅用于安全学习与技术交流，涉及的所有命令请仅在获得授权的测试环境中执行。请遵守所在国家或地区的法律法规与所在组织的渗透测试授权流程，禁止用于任何非法用途。
重点导读它是什么
SqlKnife 是一个面向内网 SQL Server 的安全检测工具，主体为 C++ 与 C# 实现，编译后为单个命令行程序。它把 SQL Server 侧的多种能力封装成参数开关：扩展存储过程调用、OLE 自动化调用、CLR 存储过程加载、基于 NT 服务的本地权限提升、注册表持久化，以及防火墙与远程桌面策略调整。
工具面向的典型场景是：内网资产测绘阶段拿到一个 SQL Server 的高权限账号，需要快速判断该账号能否触达宿主机、能否获取操作系统层权限、能否留下可被审计发现的痕迹。这些判断在传统流程中需要分别使用多个工具、多套环境变量和不同的命令拼接方式，SqlKnife 把它们统一为一个可执行文件与一组固定的参数。
重点导读功能全景

xp_cmdshell 的启用、命令执行、禁用
Ole Automation Procedures 的恢复与程序启动
CLR 存储过程加载与命令执行
远程下载可执行文件并运行
Sweet Potato 类 NT 身份劫持提权
EfsPotato 类 EFS RPC 提权
远程桌面服务开启与端口回读
防火墙与防病毒策略关闭
Shift 键劫持后门
CLR 版本适配（.NET 3.5 与 .NET 4.0）

重点导读命令速查



参数组合
作用




-H <host> -P <port> -u <user> -p <pass>
连接参数


-D <dbname>
指定数据库，默认 master


--xpcmd --fix / --xpcmd --remove
启用 / 禁用 xp_cmdshell


--xpcmd -c <cmd>
通过 xp_cmdshell 执行命令


--oacreate --fix / --oacreate -c <cmd>
恢复 OLE 自动化并执行程序


--clrcmd --fix / --clrcmd -c <cmd>
安装并调用 CLR 命令执行存储过程


--clrdexec --fix / --clrdexec -c <url>
安装并调用下载执行存储过程


--dbup --fix / --dbup -c <cmd>
安装并调用 Sweet Potato 提权存储过程


--dbup2 --fix / --dbup2 -c <cmd>
安装并调用 EfsPotato 提权存储过程


--openrdp
开启远程桌面并放行端口


--disfw
关闭防火墙与防病毒


--shift
写入 Shift 键劫持项


--3 / --4
选择 CLR 载荷版本，默认 .NET 4.0



重点导读xp_cmdshell
最基础的一条路径。启用阶段只是常规的 sp_configure 加 RECONFIGURE WITH OVERRIDE，然后置位 xp_cmdshell。
恢复 xp_cmdshell
xp_cmdshell 参数配置
执行命令时，工具拼接 Exec Master..xp_cmdshell '<cmd>' 并执行，随后读取结果集第二列作为回显。
执行命令
清理阶段把 xp_cmdshell 置回 0，并把高级选项恢复。
禁用 xp_cmdshell
重点导读OLE 自动化
声明一个整型变量接收 wsScript.shell 对象地址，随后调用 SP_OAMethod 执行 run 方法。该路径没有标准输出回显，工具自身在文档中也标注了这一点。
恢复 Ole Automation Procedures
执行程序
重点导读CLR 存储过程
工具将 .NET 程序集以十六进制形式写入 CREATE ASSEMBLY 语句，配合 WITH PERMISSION_SET = UNSAFE 与 TRUSTWORTHY 数据库选项，再以 EXTERNAL NAME 绑定为存储过程。命令行侧对应 --clrcmd 与 --clrdexec 两个模式。
命令执行存储过程内部以 sqlps.exe 承载被执行的命令，并通过 SqlDataRecord 把标准输出回传到客户端。
下载执行存储过程接收 URL，可选接收第二个参数作为落地文件名，未指定时落到 C:\windows\temp 目录下的随机四位数字命名的 exe 文件，下载完成后直接运行。
重点导读甜土豆
Sweet Potato 的载荷被编译为 C# 程序集，安装为存储过程 PotatoInSQL 后调用。载荷内部完成特权检查、NTLM 协商与令牌获取。
载荷通过 6666 端口与本机监听的 COM 服务建立连接，捕获 NTLM 认证报文并构造 Type 3 消息完成认证，进而获取到高权限令牌；随后以该令牌创建进程执行命令。程序集通过 SQL 查询管道把执行过程中的输出逐行发送回数据库客户端，因此在 ADO 侧可以看到标准输出。
安装 .NET 3.5 版本载荷：
安装 PotatoInSQL
CLR 版本选择
利用提权执行命令：
提权执行命令
输出回显
重点导读EfsPotato
EfsPotato 的原理是 MS-EFSR 的 EfsRpcOpenFileRaw 接口配合 SeImpersonatePrivilege 完成本地权限提升。载荷中同时启动了命名管道线程与 RPC 线程：管道线程等待客户端连接，RPC 线程向 \\localhost\PIPE\<random> 发起调用，服务端在处理请求时通过 ImpersonateNamedPipeClient 拿到客户端身份令牌，最后以该令牌调用 CreateProcessAsUser 执行命令。
EfsPotato 安装与执行
重点导读RDP 与防火墙
开启远程桌面通过写入 Terminal Server 相关注册表项完成，fDenyTSConnections 与 UserAuthentication 双双置零，同时把 UserAuthentication 限制解除。端口号从 RDP-Tcp 子键读出后，写入防火墙规则字符串中的 LPort 字段，实现端口放行与开启的联动。
开启 RDP
写入防火墙规则
关闭防火墙的写法是分别改写 DomainProfile、PublicProfile、StandardProfile 三个配置分支下的 EnableFirewall 值，并同时把 Windows Defender 的 DisableAntiSpyware 置为 1。工具在说明中标注了这条路径可能因系统版本差异而失效。
关闭防火墙
重点导读Shift 劫持
在 Image File Execution Options 分支下为 sethc.exe 设置 debugger 项，指向 cmd.exe，之后每次按下 Shift 键都会拉起命令提示符。
Shift 劫持
重点导读清理痕迹
与 --fix 对应的 --remove 会删除对应的存储过程与程序集，把 clr enabled 置回 0，把数据库的 TRUSTWORTHY 属性还原为 OFF，最后把高级选项恢复为默认。
启用与还原配置
需要注意的边界是：--fix 与 --remove 参数只作用于 xpcmd、oacreate、clrcmd、clrdexec、dbup 五类模式，注册表类与端口类的操作没有对应的回滚命令。
重点导读源码结构
仓库内包含一个解决方案与多个子工程：

SqlKnife：C++ 主程序，命令行解析与全部 T-SQL 拼接
PotatoInSQL：C# 项目，Sweet Potato 载荷及 NTLM 协商实现
EfsPotato：C# 项目，EFS RPC 提权载荷
SqlCmdExec：C# 项目，CLR 命令执行存储过程
DownLoadExec：C# 项目，下载执行存储过程
sqltool.py：Python 脚本，程序集十六进制文本的分段输出工具

C++ 侧由三个文件构成主链路。SqlKnife.cpp 负责参数解析，使用 getopt_long_only 处理长选项，将方法枚举设为 XPCMD、CLRCMD、OACREATE、CLRDEXEC、DBUP、DBUP2 之一；MsSqlExploit.cpp 按方法分派到 dofix、remove 与 execcmd；DataSourceHandle.cpp 封装 ADO 连接与查询，区分 basequery 与 justquery 两类调用，前者读取结果集回显，后者只执行不取回。
DataSourceHandle 通过 #import 引入 msado15.dll，用 SQLOLEDB 提供者拼接连接串，命令超时设置为 8 秒。启用远程桌面时，读取到的端口号会被拼接进 xp_regwrite 的规则字符串中，这是代码中唯一一处跨模块的数据回填。
载荷的组织方式有两种。.NET 3.5 与 .NET 4.0 各编译一份程序集，分别以内嵌十六进制字符串的形式存放于 C++ 源码中，避免运行时依赖外部文件。sqltool.py 则用于在本地生成这些十六进制文本，它把完整 SQL 文本按长度均分为十三段输出，避免单条语句过长。
重点导读核心技术架构

通信层：Windows ADO，SQLOLEDB 提供者
入口层：C++ 命令行，getopt_long_only
执行层：扩展存储过程、OLE 自动化、CLR 外部存储过程
载荷层：C# 程序集，.NET 3.5 与 .NET 4.0 双版本
提权层：NTLM 认证劫持、EFS RPC 接口滥用、命名管道客户端模拟
持久化层：注册表 IFEO 项、扩展存储过程开关、CLR 程序集与存储过程
策略层：Terminal Server 注册表项、FirewallPolicy 分支、Defender 策略项
打包层：Python 十六进制文本分段生成

重点导读使用准备
编译主程序需要 Visual Studio 环境以及 msado15.dll 的导入库；.NET 载荷子工程需要 SQL Server 的 CLR 类型引用。程序本身不依赖额外的第三方运行时，目标机器上需要启用 CLR 的 .NET Framework。
命令格式保持一致：连接参数固定为 -H、-P、-u、-p，可选 -D 指定数据库，-c 传入要执行的命令，模式开关决定行为，--fix 与 --remove 决定是准备还是清理。
重点导读实战流程
一个典型的检测流程是先以 --xpcmd --fix 判断该账号是否具备开启扩展存储过程的权限，再用 --xpcmd -c whoami 验证宿主机上下文；如果系统层面权限不足，转向 --dbup 或 --dbup2 走本地提权路径，再以 --openrdp 打通横向通道。每一类操作结束后立即执行对应的 --remove，最后用注册表类操作的核对清单确认痕迹清理完毕。
重点导读风险提示
该工具的操作全部依赖 SQL Server 服务账号本身的权限，工具本身不提供任何越权之外的隐蔽性保障。xp_cmdshell 开关、CLR 程序集、TRUSTWORTHY 属性、注册表 IFEO 项都属于可被数据库管理员直接审计的对象。在生产环境使用前应确认已获得书面授权，并在测试环境中先行验证。
重点导读项目地址
本文介绍的项目开源地址如下：
本公众号非项目作者，仅做技术分享。
https://github.com/0x727/SqlKnife_0x727

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
