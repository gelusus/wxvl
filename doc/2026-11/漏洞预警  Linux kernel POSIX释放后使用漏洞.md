#  漏洞预警 | Linux kernel POSIX释放后使用漏洞  
浅安
                    浅安  浅安安全   2026-11-01 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-98260  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Linux kernel是美国Linux基金会开源的操作系统Linux所使用的内核，  
POSIX定时器是内核提供给应用层的标准定时器接口，支持按线程CPU时间触发定时事件，供多线程业务程序使用。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQlfTO30MhxQXbwG1WSjsS1TMvUY3aIsCJ1ic2kxclUiaYic5gV2p9dmfrcTERVZZ5bEOnWFIvWxTRHhnpSrVwYdscfiaPjyuYZZadMxgIIH8Gg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-98260**  
  
**漏洞类型：释放后重用**  
  
**影响：**  
获取root权限  
  
**简述：**  
Linux Kernel POSIX存在释放后重用漏洞，该漏洞源于非leader线程在调用execve()时，de_thread()交换TID并释放原leader任务后，POSIX CPU定时器的清理操作被放置在多个可能失败的后续步骤之后。若这些步骤失败导致任务进入do_exit()路径，exit_itimers()会释放k_itimer结构，但其timerqueue节点仍挂在存活任务的posix_cputimers队列中。后续release_task()或timerqueue_del()操作会解引用已释放节点，造成使用后释放。攻击者可利用该漏洞触发内核内存破坏，导致系统崩溃或实现本地权限提升至root，甚至容器逃逸。  
  
**0x04 影响版本**  
- 5.7 <= Linux Kernel < 6.6.158  
  
- Linux Kernel 6.12.* < 6.12.112  
  
- Linux Kernel 6.18.* < 6.18.54  
  
- Linux Kernel 7.2.* < 7.2.8  
  
- Linux Kernel 7.3.* < 7.3-rc4  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.kernel.org/  
  
  
  
  
