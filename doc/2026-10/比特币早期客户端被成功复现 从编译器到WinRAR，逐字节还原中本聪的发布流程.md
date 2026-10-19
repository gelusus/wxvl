#  比特币早期客户端被成功复现: 从编译器到WinRAR，逐字节还原中本聪的发布流程  
tuto
                    tuto  杂杂咱谈   2026-10-19 16:06  
  
## 一、项目概述  
  
一个名为 bitcoin-0.1-rebuilt  
 的开源项目，成功从比特币(Bitcoin)最早期的 0.1.0 版本源码出发，重新构建出与中本聪(Satoshi Nakamoto)于 2009 年发布的 bitcoin.exe**逐字节完全一致**  
的可执行文件。  
  
该项目不仅复现了原始程序，还重现了中本聪当年使用 WinRAR 打包发布文件的过程，最终生成与历史发布档案具有相同哈希值的 bitcoin-0.1.0.rar  
。  
  
这一成果的意义不仅在于复现一个历史软件版本，更在于通过二进制文件分析、编译器版本还原、依赖库匹配和构建环境重建，验证原始发布程序与公开源码之间的对应关系，并进一步还原比特币早期开发环境的诸多细节。  
  
项目地址：  
  
https://github.com/t4t5/bitcoin-0.1-rebuilt  
  
中本聪研究所(Satoshi Nakamoto Institute)历史代码档案：  
  
https://satoshi.nakamotoinstitute.org/code/  
## 二、核心成果：原始二进制文件完全匹配  
  
项目通过重建历史编译环境，生成了与原始发布文件完全一致的可执行文件及压缩档案。  
### 1. 原始可执行文件  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">信息</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件名称</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">bitcoin.exe</span></code></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件大小</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">6,440,960 字节</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">SHA-256</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">fbcac071d92e26d82ec917214e334bd43850c0691f113bab1d4741c9bdd30d2d</span></code></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">验证结果</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">与目标历史二进制文件逐字节匹配</span></section></td></tr></tbody></table>### 2. 原始发布压缩包  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">项目</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">信息</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件名称</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">bitcoin-0.1.0.rar</span></code></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件大小</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2,132,686 字节</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">MD5</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">91e2dfa2af043eabbb38964cbf368500</span></code></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">SHA-1</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">ec9ed4ccbc990eceb922ff0c4d71d1ad466990dd</span></code></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">验证结果</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">与历史发布压缩包的哈希值一致</span></section></td></tr></tbody></table>  
哈希值匹配与文件大小一致并不是同一个概念。本项目的关键成果是重新构建的可执行文件与目标文件逐字节一致，而生成的压缩包也通过了原始发布档案的哈希校验。  
## 三、为什么这一项目值得关注？  
### 1. 验证公开源码与历史二进制文件的一致性  
  
对于早期软件而言，即使公开了源代码，也不一定能够直接重新编译出与历史发布版本完全相同的程序。  
  
编译器版本、依赖库、编译参数、文件时间戳以及构建工具等因素，都可能导致生成的二进制文件出现差异。  
  
该项目通过复现历史构建环境，成功生成与原始 bitcoin.exe  
 完全一致的文件。这为研究人员分析早期比特币客户端的发布过程提供了有力的技术证据。  
### 2. 比特币 0.1 版本并非为可复现构建而设计  
  
现代软件工程越来越重视可复现构建，即在满足相同构建条件的情况下，不同环境能够生成完全一致的二进制产物。  
  
但比特币最初发布于 2009 年，当时并没有专门为这一目标设计构建流程。  
  
原始程序是在中本聪自己的 Windows XP 环境中构建的，使用了当时的 MinGW 工具链以及多个特定版本的第三方依赖。  
  
为了实现逐字节复现，项目研究人员需要从 2006—2008 年期间发布的软件包中寻找匹配组件，精确还原编译器、二进制工具链、依赖库，甚至某些文件的换行符格式。  
### 3. 二进制文件成为研究早期开发环境的重要证据  
  
可执行文件中不仅包含程序逻辑，还可能保留编译器路径、调试字符串、资源时间戳、PE 文件头信息、导入表及节区布局等线索。  
  
通过将这些特征与不同版本的构建工具进行比对，研究人员能够进一步推断原始软件的构建方式。  
  
本项目正是利用这些线索，逐步还原了中本聪当年的开发环境和软件打包流程。  
  
这些结果能够支持对特定构建环境的技术推断，但不能仅凭构建环境就确认开发者的现实身份或具体物理设备。  
## 四、还原中本聪的历史构建环境  
  
根据项目分析，原始程序的构建过程可以分为三个阶段。  
### 第一阶段：构建基础依赖库  
  
约在 2008 年 11 月 28 日，中本聪在 Windows XP 环境中构建了以下依赖组件：  
- wxWidgets 2.8.9  
  
- Berkeley DB 4.7.25  
  
- MinGW GCC 3.4.5  
  
- 当时的 GNU binutils 工具链  
  
之后，构建环境中的 binutils 又升级到了 2.18.50-20080109  
 快照版本。  
### 第二阶段：构建 Bitcoin 客户端  
  
2009 年 1 月 10 日，中本聪使用 Bitcoin 自带的 Makefile 构建客户端程序。  
  
当时使用的是 Windows 原生命令提示符 cmd  
，而不是 MSYS 环境。  
  
这一细节可以通过可执行文件中保留的路径信息进行交叉验证。MSYS 可能会转换部分路径，而原始程序保留的路径特征与直接在 cmd  
 中构建的结果相符。  
  
原始可执行文件的完成时间约为 2009 年 1 月 10 日 23:16 UTC。  
### 第三阶段：还原原始发布压缩包  
  
生成可执行文件之后，项目还使用历史版本的 WinRAR，在 Wine 环境中重现原始压缩包。  
  
通过匹配压缩算法、文件顺序、文件时间戳及文件属性，最终成功生成与历史发布档案哈希值一致的 RAR 文件。  
## 五、关键构建组件及证据  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">组件</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">还原结果</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">主要证据</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">操作系统</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Windows XP</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Berkeley DB 配置检查与 XP 运行库行为相符</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">编译器</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">MinGW GCC 3.4.5</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">可执行文件中保留的 libgcc 源码路径</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">MinGW 运行库</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">mingw-runtime 3.13</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">mingwm10.dll</span></code><section><span leaf=""> 与对应历史版本完全一致</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">w32api</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">3.10 或 3.11</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">导入信息与这两个版本相符</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">依赖库使用的 binutils</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2.17.50 或更早的相近版本</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">wxWidgets 与 Berkeley DB 的指令编码、填充布局特征</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Bitcoin 使用的 binutils</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2.18.50-20080109</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">目标文件、导入跳板及节区标志等特征</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">wxWidgets</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">2.8.9</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">版本字符串、嵌入式清单及 CRLF 换行格式</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Berkeley DB</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">4.7.25</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">程序中的版本字符串</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">OpenSSL</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">0.9.8h</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">使用随程序发布的 </span><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">libeay32.dll</span></code></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Boost</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">1.34.1</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">头文件及其对应代码特征</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">构建配置</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">默认 Debug 构建，随后执行剥离操作</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">调试字符串仍然存在，但符号表已被移除</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">命令解释器</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">Windows </span><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">cmd</span></code></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">原始路径信息与 MSYS 路径转换行为不符</span></section></td></tr></tbody></table>  
其中，部分组件的版本可以较为精确地确定，而另一些组件只能缩小到一个可能的版本范围。项目依据的是多个二进制特征的综合匹配，而不是单一字符串判断。  
## 六、如何复现这一历史构建过程？  
  
该项目提供了自动化构建脚本，可用于在现代 Linux 环境中重现历史版本。  
### 1. 环境要求  
- x86-64 Linux 主机  
  
- Docker  
  
- curl  
- 约 2 GB 可用磁盘空间  
  
项目的构建流程需要运行 32 位 Wine 环境，因此不能直接使用 ARM 主机完成该构建流程。  
### 2. 执行构建  
  
克隆项目仓库：  
```
git clone https://github.com/t4t5/bitcoin-0.1-rebuilt.gitcd bitcoin-0.1-rebuilt
```  
  
运行构建脚本：  
```
./build.sh
```  
  
首次构建在 12 核机器上大约需要 6 分钟，具体时间取决于硬件性能、网络状况和依赖下载速度。  
### 3. 验证构建结果  
  
成功复现时，脚本会输出类似以下信息：  
```
[build] MATCH: build/bitcoin/bitcoin.exe has SHA-256 fbcac071d92e26d82ec917214e334bd43850c0691f113bab1d4741c9bdd30d2d[build] rar[build] MATCH: build/rar/bitcoin-0.1.0.rar has MD5 91e2dfa2af043eabbb38964cbf368500 and SHA-1 ec9ed4ccbc990eceb922ff0c4d71d1ad466990dd
```  
  
其中，MATCH  
 表示对应文件通过了脚本设置的哈希校验。  
  
如果某个依赖包的原始下载地址已经失效，可以在 downloads/MANIFEST  
 中为对应条目添加备用 URL，也可以通过 DOWNLOAD_MIRROR  
 指定包含相同文件名的本地镜像目录。  
  
建议在隔离的测试环境中执行历史软件构建流程，不要将早期客户端直接用于实际资金管理或生产环境。  
## 七、WinRAR 打包过程的复现  
  
该项目不仅复现了程序本身，还对历史发布压缩包进行了还原。  
  
研究结果显示，中本聪使用的 WinRAR 版本范围为 3.40—3.51，对应时间大致为 2004 年 9 月至 2005 年 10 月。  
  
项目使用 WinRAR 3.51 的命令行程序 Rar.exe  
，并在 Wine 中执行打包操作。  
  
主要还原参数如下：  
<table><thead><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">设置</span></section></th><th style="padding: 6px 13px;font-weight: 600;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">还原结果</span></section></th></tr></thead><tbody><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">压缩工具</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">WinRAR 3.40—3.51 范围内的版本</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">压缩级别</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">最佳压缩，</span><code style="font-family: ui-monospace, SFMono-Regular, &#34;SF Mono&#34;, Menlo, Consolas, &#34;Liberation Mono&#34;, monospace;font-size: 13.6px;padding: 0.2em 0.4em;margin: 0px;white-space: break-spaces;background-color: rgba(129, 139, 152, 0.12);border-radius: 6px;"><span leaf="">-m5</span></code></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">字典大小</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">4 MB</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">压缩模式</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">固实压缩</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件排序</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">按扩展名分组排序</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件时间</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">按原始压缩包记录恢复</span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件夹时间</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">按原始压缩包记录恢复</span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top: 1px solid rgba(209, 217, 224, 0.7);"><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">文件属性</span></section></td><td style="padding: 6px 13px;border: 1px solid rgb(209, 217, 224);"><section><span leaf="">包含 Archive 和 NTFS Compressed 属性</span></section></td></tr></tbody></table>  
项目发现，文件排序顺序、时间戳以及 NTFS 压缩属性等细节，都会影响最终压缩包是否能够与原始档案完全一致。  
  
由于 Wine 无法直接设置 NTFS Compressed 属性，项目还使用了额外脚本修改 RAR 文件头中的相应属性位，并重新计算相关校验值。  
  
这说明，即使压缩包中的文件内容和压缩数据已经一致，元数据差异仍可能导致最终文件的哈希值不同。  
## 八、项目存在的限制与注意事项  
### 1. 文件名中的 0.1.0 不完全代表首发当天的版本  
  
历史档案中名为 bitcoin-0.1.0  
 的文件，实际对应的是 2009 年 1 月 10 日更新后的版本。  
  
其中，serialize.h  
 文件包含：  
```
VERSION = 101
```  
  
因此，该二进制文件更接近通常所说的 0.1.1 时代版本，而不是 2009 年 1 月 8 日的最初发布版本。  
  
目前，项目没有找到经过验证、能够确认仍然存在的 1 月 8 日原始发布包副本。  
### 2. 部分时间戳需要作为构建输入  
  
为了实现完全一致，构建脚本需要恢复原始文件中记录的特定时间。  
  
项目指出，有三个构建步骤会将时间信息嵌入最终文件。如果不设置这些时间，生成的可执行文件会在恰好 335 个字节的位置出现差异。  
  
因此，这里的逐字节复现并不是简单地使用默认参数重新编译，而是需要还原原始构建条件。  
### 3. 压缩包的元数据也必须匹配  
  
RAR 文件中的文件时间、文件夹时间和属性同样是复现过程的一部分。  
  
即使压缩数据本身已经完全匹配，如果文件头中的属性位或校验值不同，最终压缩包仍然无法达到相同的哈希结果。  
### 4. 两个 DLL 并非从源码重新构建  
  
项目没有重新编译所有依赖组件。  
  
其中：  
- libeay32.dll  
 使用原始 DLL 的导出表生成相应导入库。  
  
- mingwm10.dll  
 使用 mingw-runtime 3.13 中的对应文件。  
  
因此，该项目它实现了目标 Bitcoin 可执行文件和发布档案的逐字节复现，但并不意味着整个依赖软件栈的每一个组件都被独立地从源码重新构建。  
### 5. 构建运行于 Wine，而非原生 Windows XP  
  
现代环境中的构建过程使用 Wine 运行历史 Windows 工具。  
  
为了尽可能模拟 Windows XP 的行为，项目在 scripts/  
 目录中加入了一些兼容性处理。  
  
这与直接在原始 Windows XP 计算机上完成构建并不是同一种实验条件，但最终生成的目标文件仍然能够通过逐字节比对验证。  
## 九、原始发布档案的交叉验证  
  
项目的 original/archives/  
 目录保存了来自中本聪研究所历史代码档案的 0.1.0 版本发布包。  
  
构建流程会先校验原始 .tgz  
 档案，再从中提取除可执行文件以外的相关文件，随后重新构建客户端并生成 RAR 压缩包。  
  
最后，项目会将生成的 RAR 文件与历史发布档案的哈希值进行比较。  
  
而原始 .rar  
 与 .tgz  
 包含相同的 48 个文件，文件内容逐字节一致。其中，.tgz  
 是对 .rar  
 内容重新打包生成的档案，相关条目的所有者信息为 hal  
。  
  
这一流程通过原始档案、重建后的二进制文件以及重新打包的压缩档案形成了相互验证的证据链。  
## 十、总结  
  
bitcoin-0.1-rebuilt  
 项目展示了软件考古、二进制逆向分析与可复现构建技术的结合。  
  
它不仅重新生成了比特币早期客户端的历史可执行文件，还通过哈希校验、编译器特征分析、依赖库版本识别、时间戳还原和压缩包元数据匹配，尽可能完整地复现了中本聪在 2009 年发布软件时使用的构建流程。  
  
这一成果具有多方面的技术价值：  
- **软件供应链验证：为公开源码与历史二进制文件之间的对应关系提供了强有力的验证证据。**  
- **数字取证研究：展示了如何利用二进制文件中的编译器、路径、时间戳和节区特征还原历史构建环境。**  
- **可复现构建研究：说明了依赖版本、工具链和元数据对于精确复现软件产物的重要性。**  
- **历史软件保存：为研究比特币早期客户端及其发布过程提供了可重复执行的技术方案。**  
值得注意的是，逐字节匹配证明的是重建产物与目标文件一致，并不自动证明该程序没有漏洞，也不代表它适合现代生产环境。  
  
对于安全研究人员和软件供应链分析人员而言，这一项目提供了一个极具参考价值的案例：**通过重建历史工具链与构建条件，可以将一个看似无法复现的旧版二进制文件，转化为能够验证、分析和重复生成的技术对象。**  
  
  
