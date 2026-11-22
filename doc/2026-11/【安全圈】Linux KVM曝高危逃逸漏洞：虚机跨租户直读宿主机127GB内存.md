#  【安全圈】Linux KVM曝高危逃逸漏洞：虚机跨租户直读宿主机127GB内存  
安全圈
                    安全圈  安全圈   2026-11-21 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Linux 内核内置的核心虚拟化组件 KVM（Kernel-based Virtual Machine）披露高危越权与逃逸漏洞 **CVE-2026-99312**  
。该漏洞存在于 AMD 处理器的嵌套虚拟化（Nested Virtualization）代码路径中，Linux 内核 CVE 团队评估得分为 **8.8**  
。攻击者在租用或控制一台普通客户虚拟机的前提下，可穿透虚拟化硬件边界，直接映射并读取宿主机高达 127 GB 的物理物理内存，甚至向宿主机内核写入特制指令，威胁多租户云计算环境的隔离底座。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyHZP2IOlKoibsI3FUw2405ts1yE7SMDSticbrnkDRqqJVlDYFETnVGzHSjQ4qFHtahNnVuIlz7X5ibL6ic4SmrU4z358mbcbuiasPqQ/640?wx_fmt=other&from=appmsg "")  
  
该漏洞由安全机构 Terra Security 的研究员 Oren Yomtov 独立发现，并在 DigitalOcean 公有云环境中得到完整验证。针对多租户共享母机场景，一旦某一租户实例沦陷，攻击者即可窥探同一物理机上所有其他企业租户的内存数据，包括明文密码、私钥与会话凭据。目前上游内核补丁已正式发布，但各大主流 Linux 发行版二进制更新尚未全面就位，管理员需立即排查配置。  
## 🔍 漏洞底层机理：悬垂页表指针与“补丁引入缺陷”  
  
KVM 依靠硬件级嵌套页表（Nested Page Table, NPT）实现客户机地址空间与物理内存的映射，CPU 内部通过每个虚拟 CPU（vCPU）维护的指针来检索页表基地址。本次漏洞触发流程呈现高度硬核的内存管理破绽：  
- **INIT 信号重置疏漏**  
：当宿主机允许虚拟机再运行下一层虚拟实例（嵌套虚拟化），内层虚机在停止时会向硬件发送 INIT 重置信号。KVM 内核代码在处理该信号时，疏漏了恢复外层虚机页表指针的动作，导致指针依然停留在已经由内核释放的内存页地址上，形成 Use-After-Free 悬垂指针。  
  
- **恶意地址空间锻造**  
：一旦宿主机内核重新分配并复用了该释放页，恶意虚机即可在客户机内部向对应内存页写入自定义页表项。AMD 处理器将其误认作合法的 NPT 页表继续解析，使虚机获得了指向宿主机任意物理地址的访问通道。  
  
- **历史补丁引发的历史暗坑**  
：溯源审计显示，该缺陷并非原生代码残留，而是 2022 年内核团队修复另一嵌套虚拟化漏洞 CVE-2022-3344 时引入的二次缺陷。当年的修补方案切换了控制块路径，却遗漏了页表指针重置，导致该严重漏洞在 Linux 内核稳定分支中悄无声息潜伏了将近整整 4 年。  
  
## 💥 实操攻防验证：127GB 内存通读与宿主机内核覆写  
  
根据 Terra Security 与云计算厂商公布的技术实测报告，攻击者在受害系统上展现了完全突破边界的破坏力：  
- **公有云读权限实测**  
：研究人员在 DigitalOcean 租用的普通 AMD Droplet 云服务器中启动测试，利用该漏洞首先读取到了底层宿主机的 Linux 内核版本 Banner，紧接着将母机整整 **127 GB**  
 物理 RAM 全部映射为只读。多租户环境下，这片物理空间混合着同服务器上其他客户实例的实时计算内容。  
  
- **裸机宿主机写权限实测**  
：在独占的物理 AMD 测试服务器上，研究人员进一步验证了物理内存写入能力。虚机成功向宿主机运行中的活动内核内存页精确注入 32 字节自定义数据，加载在宿主机内的见证内核模块当场捕获并确认了这笔非法改写，证明该漏洞理论上具备构造完整宿主机 RCE 逃逸的条件。  
  
- **前提条件门槛**  
：攻击者必须拥有目标虚拟机内部的 Root 权限以调用特权指令配置嵌套虚拟化环境；外部未经认证的纯网络攻击无法直接远程触发该漏洞。  
  
## 📦 受影响范围与内核修补进度  
  
该漏洞的影响面完全受限于硬件平台与内核模块配置：  
- **芯片限制**  
：仅影响搭载 **AMD 处理器**  
 且开启嵌套虚拟化支持的 Linux 宿主机。Intel 处理器因采用不同的 VMX 硬件虚拟化指令集与独立代码树，完全不受此漏洞影响。  
  
- **内核分支**  
：缺陷代码已随长周期维护合入 6.1 系列，并反向移植到 5.15.81 与 6.0.11 等早期版本。目前 Linux 官方上游已发布修复内核，修复版本分别为：**Linux 7.3.5**  
、**Linux 6.18.64**  
 以及 **Linux 6.12.118**  
。  
  
- **发行版适配现状**  
：截至目前，Debian 官方跟踪器显示其全部发布版本均受影响，仅 Unstable 分支有补丁；Ubuntu 跟踪器将风险定为 High 且暂未推送编译好的安装包；Red Hat 评估 RHEL 9 与 RHEL 10 受到波及，修复包延后发布，RHEL 8 则不受影响。  
  
## ⚡ 应急处置：停用 AMD 嵌套虚拟化自保指南  
  
特别强调：在客户虚拟机内部升级操作系统无济于事，防线完全取决于物理宿主机内核。在各大 Linux 发行版推出正式内核补丁并重启生效前，唯一能够百分之百封堵该攻击路径的措施，是在宿主机上彻底禁用 AMD 嵌套虚拟化。  
  
**第一步：检查当前宿主机是否处于危险开启状态**  
  
在物理宿主机终端运行以下命令：  
cat /sys/module/kvm_amd/parameters/nested  
  
若输出为 **1**  
 或 **Y**  
，表示内核已开启嵌套虚拟化，当前处于易受攻击暴露面；若输出为 **0**  
 或 **N**  
，则表示已禁用，天然免疫该漏洞。  
  
**第二步：固化模块配置文件并禁用**  
  
在宿主机系统配置目录中添加持久化规则（需 root 权限）：  
echo "options kvm_amd nested=0" > /etc/modprobe.d/kvm-amd-security.conf  
  
**第三步：重启物理宿主机使配置生效**  
  
⚠️ 注意：Linux 内核无法在虚机运行状态下动态卸载重载正在使用的 kvm_amd 模块。必须规划业务维护窗口重启物理宿主机。重启后再次读取 /sys/module/kvm_amd/parameters/nested  
，确认返回值变为 0。停用后虚机将无法在内部运行 Docker 嵌套虚机或下层 Hypervisor，云平台需提前知会相关客户。  
  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】群晖DSM曝9.8分严重漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079944&idx=1&sn=5b3509c80b236854f9a85bd7dcd0cd3e&scene=21#wechat_redirect)  
  
  
  
[【安全圈】VMware曝9.8分零日漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079944&idx=2&sn=169d2011f57187ae08f15408372fdc78&scene=21#wechat_redirect)  
  
  
  
[【安全圈】Windows锁屏曝零日漏洞：无需密码绕过屏幕锁运行程序](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079944&idx=3&sn=2c2d5cac849a0463046767c1a08c8fb2&scene=21#wechat_redirect)  
  
  
  
[【安全圈】NGINX潜伏十年漏洞曝光：无需认证直接远程执行代码](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079933&idx=1&sn=c777c98c39cf538af3ea38c136824b5d&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
  
