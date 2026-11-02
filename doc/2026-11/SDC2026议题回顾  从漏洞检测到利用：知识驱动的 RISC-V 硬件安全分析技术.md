#  SDC2026议题回顾 | 从漏洞检测到利用：知识驱动的 RISC-V 硬件安全分析技术  
SDC2026
                    SDC2026  看雪学苑   2026-11-02 09:59  
  
**SDC2026**  
  
议题回顾  
  
  
RISC-V 开放指令集降低了处理器设计门槛，也让更多主体能够在更短时间内设计更复杂的芯片。与此同时，处理器安全分析不足可能带来新的硬件漏洞。  
  
  
**在第十届安全开发者峰会（SDC2026）上，中国科学院软件研究所博士生——芮志清，围绕漏洞检测、根因分析与漏洞利用三个环节，介绍团队在 RISC-V 处理器安全研究中的方法，并以 VeeR-EL2 的 PMP 执行权限绕过漏洞为例，展示从差分模糊测试到根因定位、再到 PoC 验证与报告交付的完整过程。**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Cpo2XCpI7K0HQvllwB5nIYZoR4jzJ4eibl5iaiaT0DjKUzg5Evh2FLsJ1nDK53Gic1w6nZR1cwOicafE8oSQou2LLQQQJcBdLalKWWyicVUycZREc/640?wx_fmt=jpeg&from=appmsg "")  
  
**芮志清**  
  
**中国科学院软件研究所博士生**  
  
  
*以下为整理后的速记  
  
  
**01**  
  
**背景简介**  
  
大家好，我是来自中国科学院软件研究所的芮志清，目前是博士五年级在读，导师是吴敬征研究员，预计于 2027 年毕业。我的研究方向是系统安全与软硬件协同安全。今天主要从背景简介、漏洞检测、根因分析、漏洞利用和成果总结五个方面进行分享。  
  
  
计算机领域有许多开放标准，例如网络、操作系统和编译器等。但在处理器指令集领域，长期以来，x86、Arm、MIPS 等架构都由相应厂商掌握，使用或基于这些指令集开展研发，往往需要获得授权。2010 年，伯克利开始开发一套开放、免费的指令集架构 RISC-V。经过发展，RISC-V 已逐步得到广泛应用，在国内的发展也非常快。  
  
  
开放带来了机会，也带来了新的安全问题。由于设计门槛降低，更多主体可以在更短时间内设计出更复杂的芯片；但芯片研发往往更关注性能与流片效率，处理器安全分析可能因此不够充分，进而产生攻击漏洞。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0M3ILFMiasiaUXnXf17tSOhqIFFaqSHKVB0VMhoVDicicmSEWxo5Q974Ez9DGTZV42ndn36YmX8gS9OQnOLkObkgk5YfqOZ6wicOTY/640?wx_fmt=png&from=appmsg "")  
  
图1  
  
开放的 RISC-V 指令集在带来设计机会的同时，也带来了新的安全挑战  
  
  
例如，阿里达摩院玄铁 C910 处理器曾被发现 GhostWrite 漏洞，可导致任意写并实现提权；龙芯 LoongArch 处理器则出现过 LoongLeak 漏洞，可能造成缓存数据泄露。此类硬件级漏洞往往难以单纯依靠上层软件修复，修复也可能带来较大的性能开销，因此需要从硬件层面开展检测与分析。  
  
  
通常，我们会把计算技术栈划分为软件和硬件两个层级，并倾向于把底层硬件视为可信的黑盒。但硬件内部还有多个层次，包括微架构、寄存器传输级（RTL）、门级网表，直至晶体管等物理层级。上层软件所依赖的可靠性，是通过底层逐层封装实现的；这些层次本身并不意味着绝对可靠。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K1XsAoT0srobbsuEq4kdhOibTBibnmKZrFAwrlc81EpiaFuw5z68nicCAMLevQR21xYBMQAKLwax7btBsibERRc66C3LMFibpiaVGBwC4/640?wx_fmt=png&from=appmsg "")  
  
图2  
  
硬件安全漏洞可能存在于微架构、RTL、门级乃至晶体管等不同层次  
  
  
近年来出现的硬件漏洞说明，底层硬件问题可以被软件触发和利用。例如，Meltdown 利用了微架构相关行为窃取敏感信息；RowHammer 则可以通过对内存进行特定访问，影响相邻内存单元中的数据。此外，还有密钥缓存漏洞、时序侧信道漏洞等问题。硬件漏洞并非只能停留在底层，它们可能经由软件执行路径造成实际影响。  
  
  
**02**  
  
**漏洞检测：历史漏洞知识引导的处理器差分模糊测试**  
  
  
  
**1. 处理器模糊测试面临的挑战**  
  
模糊测试通过构造并变异测试用例，在待测目标上执行，以动态方式发现漏洞，已经广泛用于软硬件漏洞挖掘。将模糊测试用于处理器时，输入不再只是较高层的软件用例，而是汇编指令或机器码。指令之间的控制流、数据依赖和执行语义会共同影响处理器状态，因此，单纯随机生成大量指令并不一定能有效触发稀有状态。  
  
  
第一个挑战是有效状态探索困难。测试用例数量不断增加，并不代表有效探索能力会持续提升。若指令顺序不符合预期，程序可能过早退出，无法到达目标状态；随机生成还可能反复探索重复路径，难以覆盖真正关键的边界情况。  
  
  
第二个挑战是真实漏洞判定困难。软件模糊测试可以借助 Sanitizer 或进程崩溃等信号发现问题，但处理器功能错误不一定会导致崩溃，也不一定出现明显的运行时异常。有时只能观察到 PC、寄存器等架构状态发生变化，单凭这些变化很难直接判断处理器是否存在缺陷。  
  
  
  
**2. 方法框架：高质量种子、结构感知变异与差分验证**  
  
针对这两个问题，团队提出了历史漏洞知识引导的处理器差分模糊测试方法。整体思路分为三个步骤：基于历史漏洞构建高质量初始种子；通过结构感知的相似度引导变异；最后将待测处理器与参考实现进行差分比较，以发现处理器缺陷。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0cZO9E53OonBn3IB8VY8azZcvclRC9pAlgqLhdRWARjyXZokLvGEjZ879kvxHqVcUIfdASyB8do7n13yQf1eTWSMWUSantpLs/640?wx_fmt=png&from=appmsg "")  
  
图3  
  
方法框架由历史漏洞种子构建、相似度引导变异和差分测试三个部分组成  
  
  
  
**3. 步骤一：基于历史漏洞构建高质量初始种子**  
  
这一步借鉴了一个经典观察：程序某一部分已经发现的错误越多，该部分还存在其他错误的可能性也越高。处理器虽然实现各不相同，但遵循相同的指令集规范，因此，一个处理器上已经出现的漏洞模式，也可能在其他处理器中出现。  
  
  
团队收集了 Rocket、BOOM、香山、Ibex 等开源处理器仓库中的 Issue、PR、Commit 信息，以及相关 CVE 记录。由于原始信息往往以文字描述漏洞为主，缺少可以直接运行的测试用例，因此需要先整理漏洞描述，再提取或构造对应的测试种子。最终，团队从 37 份历史漏洞报告中整理出 42 个种子。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0F1wRjcmWEgvEzwK7pQxNNTe9WzVUbFvR92Jp8a2zWFZ7q5TmuyVS9erjw59aUSTc0vu0Y9taJOBX5lDvsVnb6ibIdwYpAN6r4/640?wx_fmt=png&from=appmsg "")  
  
图4  
  
从处理器仓库记录与 CVE 信息中筛选历史漏洞，并整理为可用于测试的种子  
  
  
  
**4. 步骤二：结构感知的相似度引导变异**  
  
硬件代码具有并行执行等特点，代码覆盖率在处理器测试中容易趋于饱和，传统覆盖率引导方法的效果有限。为此，团队构建了 RISC-V 二进制语义相似度度量方式，将指令类型、操作码、子语义和字段等信息纳入相似度计算，再将多条指令之间的成对相似性进行合并。  
  
  
在具体变异时，方法按控制转移指令划分基本块，并尽量保持控制流不变，只对基本块内不影响控制流的指令进行变异；随后根据变异后基本块与原始基本块的相似度，保留差异较大的变种。这样既能保留历史漏洞种子的有效结构，又能探索新的指令组合。  
  
  
  
**5. 步骤三：基于参考模拟器的差分模糊测试**  
  
对于处理器功能错误，参考实现可以帮助判断待测目标的行为是否符合规范。团队将相同测试输入分别交给待测处理器和指令集参考模拟器（Spike），比较两端的执行结果，并检查 PC、寄存器等可观测架构状态。如果参考模型与待测目标的行为不一致，就可以进一步分析潜在缺陷。该方法不依赖目标是否发生显式崩溃，因此能够发现静默的功能错误。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K27ZSaiastuYeibmFl4AMPvH8YYQ5u9WDxm144o5ibicWnXUac2FOoQYczzLIVDJwuH8B0r2Pj6JdOIgFXCReW4ia7Z3ibKH0LDj0r5Q/640?wx_fmt=png&from=appmsg "")  
  
图5  
  
实验结果显示，该方法能够更快达到较高覆盖率；新发现漏洞也能与历史漏洞种子建立关联  
  
  
实验结果表明，该方法可以在较短时间内达到更高的覆盖率。对新发现漏洞的分析也显示，部分新漏洞能够追溯到已有的历史种子，说明利用历史漏洞知识引导测试具有实际效果。  
  
  
  
**6. 案例：PMP 执行权限绕过漏洞**  
  
接下来以 VeeR-EL2 的 IFU（取指单元）PMP 执行权限绕过漏洞为例，说明上述方法如何发现问题。PMP（Physical Memory Protection，物理内存保护）机制通过一组配置寄存器，限制处理器对不同物理地址区域的访问权限，其中包括执行权限。  
  
  
假设内存区域 A 允许执行，而相邻区域 B 禁止执行。一条 32 位指令从边界前两个字节处开始，低 16 位落在允许执行的区域，高 16 位落在禁止执行的区域。按照权限检查要求，这条跨越边界的指令应触发指令访问异常；但 VeeR-EL2 的 IFU 没有独立检查高半字的执行权限，导致本应被拒绝的指令仍然执行。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K1FlfVzWpoml3C6IuJ3OlwT88Hia2qWSukV4ZYZZsABn3R88ziaI1eK9f70L8orx2X7k0M8IdeOppronKQSdTQicDYRtibmQicmwPnc/640?wx_fmt=png&from=appmsg "")  
  
图6  
  
32 位指令跨越 PMP 权限边界时，指令两部分分别落入可执行与不可执行区域  
  
  
测试种子来自 Ibex #902，该历史问题涉及 NA4/NAPOT PMP 地址匹配逻辑中的边界判断。团队保留了 PMP 漏洞种子的典型结构：先配置 PMP，再引导控制流跳转到目标边界附近，并在此基础上变异指令。  
  
  
差分模糊测试时，参考模拟器认为目标位置应触发同步指令访问异常，但待测处理器没有报告预期异常。若只观察待测目标自身，这种情况不一定会表现为崩溃；通过与参考模拟器进行差分比较，才发现了潜在漏洞。至此，测试已经确认了行为不一致，但还需要继续追溯 RTL 代码，才能回答究竟是哪一处实现导致了异常。  
  
  
**03**  
  
**根因分析：让大模型依据波形证据逐步定位**  
  
  
  
**1. 从表层异常追溯到代码根因**  
  
处理器测试发现的通常是表层异常，而定位深层根因往往需要大量人工工作。分析人员需要穿过程序、事务、指令和信号等多个层级，处理规模庞大的仿真波形。符号化算法难以完整表征软硬件交互，若缺乏语义信息，就容易陷入大量细节中。  
  
  
为解决这一问题，团队将根因分析建模为由大模型驱动的决策过程，让模型能够直接观测波形等细粒度实际行为，并根据证据不断提出、验证假设。整体架构由 Tracer 波形文本化工具和 Chaser 根因分析工作流组成。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3ibA2oejPxlpo1fiacQtWkMyibOibpUjzqEf27Eok2bNaQCtoe6iaOY4GLdWYUh7uhuX7hPn1NqmicCKgaBtMQ6rPnBDAP1OlxGor9o/640?wx_fmt=png&from=appmsg "")  
  
图7  
  
根因分析架构由 Tracer 波形文本化工具与 Chaser 迭代分析工作流组成  
  
  
  
**2. Tracer：将原始波形转化为可理解的时序证据**  
  
原始波形记录了不同信号在不同时间点的取值，可以理解为一份规模很大的时序数据。但数据本身通常比较零散，直接交给大模型，难以让模型快速把握关键事件。Tracer 首先结合 RTL 代码，筛选需要关注的关键信号，再根据时间顺序组织软件动作与硬件事件，并对波形内容进行压缩，输出结构化文本日志。  
  
  
在 PMP 案例中，分析会关注 rvfi_pc、rvfi_insn、ifc_fetch_addr_bf、ifu_pmp_addr、pmp_chan_addr[0]、pmp_chan_err[0]、ic_access_fault_f[1:0]、mcause 和 mepc 等信号。工具把这些信号与关键事件对应起来，例如 PMP 配置完成、跳转到边界前两个字节、IFU 发起取指、PMP 进行权限判断、异常信号形成以及架构状态更新。这样，大模型能够围绕与漏洞相关的时序证据进行分析，而不是直接面对全部原始波形。  
  
  
  
**3. Chaser：波形验证与代码归因交替进行**  
  
Tracer 输出的文本提供了初始线索，但线索仍需要验证。Chaser 工作流在波形验证和代码归因之间反复迭代：先确认某个事件是否真实发生，再回溯相关代码，提出下一轮需要验证的问题，然后再次读取波形。通过这种方式，模型逐步排除不成立的假设，直到形成可以验证的因果链。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3R8DEw9n43SZlqhKuMJ6A7OWoNYs1056ibaic32ibyR3rnPJq5XNxX9ETKy39Jice0bJzBlVOUY9TXnMXamJqOY91KV6TRU32o6Dk/640?wx_fmt=png&from=appmsg "")  
  
图8  
  
根因链路表明，问题出在 IFU 的权限检查粒度与 32 位指令实际覆盖范围不一致  
  
  
在本案例中，参考模型显示 PC 位于边界前两个字节时应触发异常，而待测处理器却执行了跨界指令。迭代分析逐步回溯到相关信号和 RTL 逻辑，最终确认：IFU 只对取指基址进行 PMP 执行权限检查，没有对 PC+2 对应的高半字地址进行独立检查。与之对照，LSU 的访问检查会考虑起始和结束地址，而 IFU 在这一场景下只检查基址。根因因此可以明确为：IFU 的权限判定粒度与 32 位指令实际覆盖的地址范围不一致。  
  
  
这种分析方式并不是让大模型仅凭代码猜测原因，而是先用波形确认实际行为，再通过代码归因解释行为，随后回到波形进行交叉验证。最终输出的不只是一个可疑代码位置，而是从输入指令、关键信号到权限检查结果的完整根因链路。  
  
  
**04**  
  
**漏洞利用：知识驱动的 PoC 智能体**  
  
  
  
**1. 从发现缺陷到证明实际影响**  
  
完成漏洞检测和根因分析后，还需要回答漏洞是否能够被实际利用。随着大模型辅助发现的漏洞数量增加，PoC 可以用于证明漏洞的实际价值。但从缺陷到可复现利用，仍然存在三个挑战：一是从 RTL 行为中推导触发条件、状态传播与安全影响并不容易；二是攻击构造和仿真调试依赖对具体 IP 接口、内部状态和验证环境的了解；三是即使攻击代码能够运行，也需要区分一般功能异常与真正的安全机制绕过，并形成稳定、可审计的证据。  
  
  
  
**2. 构建知识驱动的漏洞验证智能体**  
  
团队构建了知识驱动的漏洞 PoC 智能体框架，将漏洞理解、利用代码构造、仿真调试、攻击验证和交付打包串成一套流程。框架接入本地知识库，包括 MITRE CWE 漏洞类型参考、RISC-V ISA／特权架构／调试规范，以及 Caliptra RTL 手册、调试笔记和常见陷阱等领域资料。相关知识按需加载，以弥补通用模型对特定 IP、接口、观测点和调试流程了解不足的问题。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0MwicqAva10EoQ2N9Ony56l428CcNYEjzThVIktfibHfibrmwfUSpJJPWuVxdicaaqqicm3iaXoIOJCDetYOhHq8K2cSm0srMtISLWg/640?wx_fmt=png&from=appmsg "")  
  
图9  
  
知识驱动的 PoC 智能体将漏洞分析、构造 PoC、仿真调试、攻击验证与交付打包串联起来  
  
  
框架的执行过程包括五个阶段：首先阅读 RTL、映射 CWE、形成漏洞假设并推导触发条件；其次生成利用代码和输入／指令序列，准备初步测试；然后运行仿真并观测 UART、Trace 或 FST 等信息，通过迭代调试实现稳定复现；接着重新执行测试，确认安全绕过并记录证据；最后整理 PoC 源码、运行日志、演示记录和漏洞报告。  
  
  
框架还会把调试中的成功经验和失败尝试沉淀回知识库，例如成功的利用步骤、关键观测信号、环境配置、常见陷阱和无效路径。通过持续回写与更新，后续分析可以复用已有经验，减少重复试错。  
  
  
  
**3. 案例：在 Caliptra SoC 仿真中完成端到端验证**  
  
在 PMP 执行权限绕过案例中，团队使用 ClaudeCode 与 GLM-5.3 执行漏洞分析和利用任务，并结合历史漏洞、CWE、RISC-V 规范和 Caliptra 调试知识推进分析。任务要求智能体确认漏洞真实性、触发条件与安全影响，构造可复现的 Exploit，运行仿真并调试，最终录制利用过程、整理漏洞报告。  
  
  
第一次运行 PoC 时，仿真出现异常循环。通过检查日志和反汇编，团队发现程序在恢复执行状态设置之前发生了早期 trap，随后返回恢复地址并导致固件重启；PMP 已锁定后，相关写操作又触发 store fault，形成循环。智能体据此继续检查并修改 PoC，直到测试通过。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K16SYiaV6TtlyaBfz1WBmUJNf1wHW3Ut8sKkEVVmILMMbXG5XrjAP8OdkIZBicbw9Cu7icT4CWnkaI77Dz88PGZicPiaoSeaSKia0tNs/640?wx_fmt=png&from=appmsg "")  
  
图10  
  
PoC 调试通过后，测试验证了对齐取指、常规读取受限，以及跨界取指绕过执行权限检查等行为  
  
  
最终，端到端利用在仿真中成功。测试结果包括：对齐取指到 0x40020004 时触发 mcause=1，哨兵寄存器值保持不变，说明权限限制生效；读取保险库时触发 mcause=5，说明常规读取受到限制；当通过 jalr 跳转到 0x40020002，触发跨界取指时，来自锁定、禁止执行区域的高半字参与组成指令并被执行，a0 变为 0x5EED0000。预期的异常直到下一次取指、PC 为 0x40020006 时才出现。  
  
  
在这一过程中，智能体不仅需要生成代码，还需要结合仿真知识反复调试，并把成功的运行日志、PoC 源码、漏洞报告和演示记录整理为可复现的交付材料。最终输出可以供人工进一步审核与提交。  
  
  
**05**  
  
**成果总结**  
  
团队持续在香山处理器与 NEMU 模拟器上开展漏洞挖掘与分析，共发现 18 个缺陷，并申请了 7 个 CVE 编号和 1 个 CNNVD 编号。缺陷类型包括访问控制错误、浮点单元错误、译码单元错误、地址对齐检查错误、异常处理错误、寄存器初始化或配置问题等。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3FKk49DxNM6DMkVBkxDibLNkibksZMNic0xfqIYrX4EUAtmouRHcicmLBLm8TvdvOb1jXs4w8PhAyyFjXkxicsh7nZ52nsGt7OZH7w/640?wx_fmt=png&from=appmsg "")  
  
图11  
  
处理器与模拟器缺陷统计，包含缺陷原因、修复情况及相关编号  
  
  
团队也持续参加国内外硬件安全竞赛。2025 年 4 月，团队获得 HACK@DATE 2025 硬件漏洞挖掘比赛全球总冠军；2025 年 9 月，获得 HACK@CHES 2025 全球总冠军；2025 年 12 月，获得首届“万众一芯”黑客马拉松冠军；2026 年 7 月，再获 HACK@DAC 2026 全球总冠军。HACK@DAC 2026 决赛阶段，团队实现了 75 个漏洞的全自动化利用。  
  
  
开放的 RISC-V 指令集架构开启了处理器设计的新时代，也带来了新的安全风险。围绕这些风险，团队持续探索硬件模糊测试、根因分析和自动化漏洞验证生成等技术，并将相关方法用于国际硬件安全比赛和真实场景下的漏洞挖掘。  
  
  
以上是本次分享的主要内容。欢迎大家就 RISC-V 硬件安全、漏洞挖掘与自动化验证开展合作交流。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3kuFjlgwLTicAm9eYW3dFs55Dj1HrckczSMUhnTtR5Td99IYol5CicjLgLJLy5DH9VBnXx5bvtW9o9modhmABwskcRCE0vAQHTo/640?wx_fmt=gif&from=appmsg "")  
  
往期议题回顾  
  
[SDC2026议题回顾 | 从 LLM 到 Agent：AI 安全攻击面的过去、现在与未来](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623168&idx=1&sn=c067079a13953815453d84054be125f5&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | 一次渗透 Agent 的架构减法实践：大模型、工具和Harness谁更重要？](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623198&idx=1&sn=a2f7444454ef6dd0e7fd9774e731b005&scene=21#wechat_redirect)  
  
  
[SDC2026议题回顾 | Storm: AI审计时代下，Fuzzing还有用吗？](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458623398&idx=1&sn=5313300d03566b0a72a9824d15c96d4e&scene=21#wechat_redirect)  
  
  
  
  
**PPT下载及视频回放**  
  
  
峰会议题PPT及回放视频已上传至  
看雪课程-【看雪2026 SDC】：  
  
https://www.kanxue.com/book-section_list-107.htm  
  
  
【已购票的参会人员可免费获取】：  
我方已通过手机短信发送“兑换码”  
  
  
免费获取方式：  
  
1. 登录看雪账号，进入上述  
课程页面。  
  
2. 点击“立即购买”  
  
3. 输入大会于 2026年10月27日 通过手机短信发送的  
兑换码。  
  
4. 提交订单后，即可免费开通课程查看权限，获取大会演讲PPT及视频。  
  
  
*温馨提示：使用兑换码无需支付任何费用。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K2A2TSG8ic4bpplTlt9nEVbl0utGEibULZdEBAlZziauibzURMaUmJVyibv5giaSlzpvgBEUdLpX6o3n9q1nuZW0MrdjLFQ7q7trsgbI/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3HztvhOjpFy4zNujeW5Rn5WVmOu0XWWwvEdiaWcwulDneL9IzMlq0xUzQJ7KaJHg2bClAB0Wg0KKicmItBZDPIJcyicg2kiahjGek/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K2X7uMSW7Qnl6hiaADHZHXYwiaQYxVCdTwD2P4mYjyaa1rATbuVj7BLnU2C4Nh5t17U1DNx2wthpLFIibG9KRGarY6qGbSFleQ4ms/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K0VHw5ykGAgMFxoTcoqsqicbDL1JyJXRfwxJHsRgHh2L9LsVmHHQU4uAvWReXhgeyuWY1Pbm8ocibyau5eI4FcdusuicjuYDpo6cs/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K1Oa7EjoKOicnwmVayUVJZMGl7aJorNExibjYY2wLA8n0G9AE0HALkI6iaRdX5kUeEIbk8t2Q64NvKhSU0ALvJ945rdbj6wyFIxME/640?wx_fmt=gif&from=appmsg "")  
  
点击阅读原文查看更多  
  
