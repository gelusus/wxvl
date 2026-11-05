#  绕过 Carbonara 补丁：拆 MediaTek DA2 堆溢出 heapb8 到任意代码执行  
原创 黑卷
                    黑卷  黑卷的IoT攻防日记   2026-11-05 06:53  
  
2025 年 9 月，GSM 工具 Chimera 悄悄宣称「世界首发」支持 Dimensity 9400 / 8400——而这些 SoC 上的 Download Agent，早在 Carbonara 补丁之后编译。  
  
要么它绕过了补丁，要么手里捏着新洞。R0rt1z2 和 shomy 决定把这条路摸清楚。  
  
![GitHub 评论：有人丢出 Chimera 的 USB 抓包](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84WaH6opbkEKfGeSxKgXlkZJrxCjHx9NFqhZsObbND00GqQkWlKg0s65AKbXjhMA8sgLTegJVibYkKaDuRmhQmu0mhYVqU7HGnGk/640?from=appmsg "")  
  
几个月的 USB 抓包、连夜逆向、无数次回退之后，他们挖出了 **heapb8**  
（读作 heap-bait，「堆饵」）：DA2 USB 文件下载路径上的堆溢出，能在已打 Carbonara 的 V6 设备上拿到任意代码执行。  
  
![攻击链示意：USB 多发 → 覆写块头 → DPC.cb → shellcode](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84XjaRWiczGdMuGbKdB2CiaEUXpKkHKED04F3ckCkXj2vCllnKMmQicY6TThrsBgorLbdrnmt1JbKxSzyENSEqXPnXBE82H7pUTg7c/640?from=appmsg "")  
## 一、战场从 BROM 挪到了 DA2  
  
联发科设备有两套 USB 下载面：BootROM 和 Preloader。前几年 MTKClient 一类工具，主打 BROM USB 栈漏洞。  
  
新 SoC 上 BROM 洞基本被补完，不少 OEM 干脆关掉 BootROM USBDL，只剩 Preloader 这一条路。要刷机、要解锁，就得跟 Download Agent（DA）打交道。  
  
![MediaTek 启动链示意：BROM / Preloader / DA1 / DA2](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84VRhMib5kOV9kyOdhKsiaQgOACdWJomFKSrudKpEvST9N01gtgNfsKobjZvCXHuGEBImrCzHln3NXAqibr1hNjtWHnL8kV2RibR0TU/640?from=appmsg "")  
  
DA 是跑在设备上的小程序，负责 USB 通信和刷写。协议分三代：Legacy V3、XFlash V5、**XML V6**  
（codename chimaera）。heapb8 盯的就是 V6 的 **DA2**  
。  
  
上传 DA 时还要过 SBC / SLA / DAA 三道关。BROM 验完跳 DA1，DA1 再验完跳 DA2——真正干活的下载处理，就在 DA2 里。  
## 二、抓 Chimera：Beagle + UART  
  
目标机选 Nothing Phone 2A（Dimensity 7200）。Chimera 有反分析，Wireshark 直接挂容易翻车，于是上了 Total Phase Beagle USB 480 做硬件旁路抓包。  
  
![Beagle USB 480 硬件抓包器](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84VIjAw4vh9AKR1qW2CDq9k6B0fZa0bo7WNX6AiaO5Ws3KWBwzunibjosDa5ug8G5zxZeiaibz3J6BG6RLFCvFYTsAI3P8mNibI3tYoM/640?from=appmsg "")  
  
光看 USB 不够，还要看设备侧日志。板子背面两颗小焊盘就是 UART：BROM 115200 8N1，后续阶段 921600。  
  
![Nothing Phone 2A 主板（示意）](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84W5KASgHol33GfekUgqFPUKYRR7IO17Tzk1Q4K7z0OgC0icHRd2ndZJzcWnaHqah5As9e0D6wJFSDwxDO2CxoWlkGGYvOEORJVw/640?from=appmsg "")  
  
![UART 焊盘标注（示意）](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84Uxg7icLWAOjibQFic6HKGkicBqEu5QG8QJXjFILk5Frj1ANJWXicaJYTZq7XMWObC9rKUC9JIPgMiaWzDNLFiaj7xdgJmicFIWqdAMKyI/640?from=appmsg "")  
  
一个坑：DA 默认可能把 g_log_switch  
 关掉，outchar()  
 直接不往串口吐。Chimera 抓包里还能看到它主动关日志——防你对照。  
  
![Data Center 打开 Beagle 专有抓包格式](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84UPT9yoOHwibxqnJuBsobvNY73VAx3Vhbob871ZF7WjiaUJrp2jnvwum30lYUTibIORGp85xm8m9Ewh4FDK3d5icicwIiboe5ibQp1sbA/640?from=appmsg "")  
## 三、假洞与真洞：XML 烟雾弹  
  
抓包里 Chimera 会发两次 CMD:SECURITY-SET-ALLINONE-SIGNATURE  
。第一次文件内容居然像 ARM64 指令序言；第二次文件名塞满 &  
 一类特殊字符，然后**中途 abort**  
。  
  
AIO 命令会 mxmlLoadString  
 解析 XML，再 fp_read_host_file  
 拉文件。拉之前对文件名做 mxml_escape  
——固定 **512 字节**  
缓冲，特殊字符展开却不设界。一百多个 &  
 扩成 &amp;  
，就能越出约 **172 字节**  
。  
  
![关键数字示意：V6 / 172B / 0x10 / 两条 CVE](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84UO8dZeno4dyqMqFUgQV1fH7A58NHMTWkowN6no5J9vbglQr7FwbGdDhMj5ccQTkG2n8ia5UA0p9V1HrP6mHHxMk98wPlpd8fBU/640?from=appmsg "")  
  
一开始所有人都以为 XML 展开就是 exploit。可文件名是 XML 字符串：**空字节进不去**  
，假堆块头拼不出来。白忙一场。  
  
![XML 越界覆写块头示意（字节受限）](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84VAiaKweyBzjCw7ibaQ7F7jjSOk4FqlQZT7lcPuIBlrt71NOSEDMo8XPzBuK0GLIqmQXX3QnkYYkBpPkgU7sxZgf9wHlcyQgI8qc/640?from=appmsg "")  
  
![真假洞示意：XML 死胡同 vs USB 多发](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84WJFOrUDLTqF2ErQiarWqMyyKeHuJJhkqVSuXegb4v9Wqib1YCGtbaZgzpYuIZdibkibFmguHtZjJUO4kqzkHwwVnWoaOjDLavCavk/640?from=appmsg "")  
## 四、真杀伤：USB「少报多发」  
  
回头看抓包：协议要求主机先声明文件长度。Chimera 声明只要 **0x13FC**  
 字节，DA 按宣称长度 +4 分配缓冲；实际却多塞数据，读循环按 **packet_size（0x20000）**  
 收包，只在 bytes_received == advertised_size  
 时停。  
  
结果：缓冲按「小尺寸」开，写入按「大包」走——直接写穿下一块的 header。而且这是**原始 USB 载荷**  
，每个字节（含空字节）都可控。  
  
![堆布局示意：AIO 缓冲与相邻块](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84UGW0iaNFPmzjqcpKkmfGiaoYxx5oWicZWJlRQTjkVeuicef7BWcUsWHticzttoicSdqnvZ8mR9ib4Rd2zJQUZ5s258olaDiccUWI1SjS8/640?from=appmsg "")  
  
ARM64 上，free()  
 会把 alloc.size  
 写到 alloc.ptr + 0x10  
。溢出改掉下一块的 ptr / size，就变成**任意写**  
。  
  
写谁？DA2 命令循环末尾有个 **DPC（Deferred Procedure Call）**  
 结构，.bss  
 固定地址，cb  
 非空就会被调用。把 dpc->cb  
 改成 shellcode 地址，命令一结束就跳进去。  
  
![堆转储：AIO 签名特征](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UQkicnFkwFnGfNeQESqiafpE8zsN7YssqeNhmDGq0JI98RjMicHTXFy0UMEytnicmEMlhcGdIBvibU7zzEFsluNibA9Fl5llO0VwAqs/640?from=appmsg "")  
## 五、落地：NOP 雪橇 + hakujoudai  
  
堆基址随机型/DA 版本变，AIO 缓冲位置也不稳。他们的懒办法：塞大约堆大小 10% 的 **NOP 雪橇**  
（量级 ~80MB），shellcode 放末尾，dpc->cb  
 指向雪橇 95% 处——滑到哪都进 payload。  
  
![shellcode 块与 NOP 雪橇示意](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84URcaeiaic7NrVK6HrTdgFfibteJJjsHq0GBl0DkBBq9kjFeQAavsibic3fogdZKf4qCZFyVicfjHVrqA8duyPsDj6v4fXG4sZUxcuia8/640?from=appmsg "")  
  
简化版甚至不用 Chimera 那套「花文件名」：一串 A  
 把 XML 节点分配到 AIO2 缓冲后面就够；特殊字符多半是故意混淆。  
  
![不同利用路径对比示意](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84UqwJ9A8TPrCs5KzTWcsMBYsGSXCJHoZcVQFHySFOnAwhooTVgHdcTVibDDias03r3GiaAoQ6BNQ9fLINAR2JuIdVVZdMxjyYctbo/640?from=appmsg "")  
  
拿到执行后，payload 名叫 **hakujoudai（白杖代）**  
：先把被搞烂的堆修回去，再注册自定义命令，回到命令循环。penumbra 集成整条链，之后可读写分区、刷镜像，甚至在原厂 DA 会挡的机型上解锁 bootloader。  
  
![penumbra 跑 HeapB8](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84WGDmb61W2RM5hnOD8iaxtck0B2o3Picyb1gWmflTSiaQdZ9lGlesLaaOx0bu6FVYia6SV9YMsuXWomgWkoo1VB1pD4QWTol4zNtBY/640?from=appmsg "")  
  
![hakujoudai 修堆并注册命令（UART）](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84UR5utJic9JUyiaDibCPekLh7Iicnj9d9tXNricNXqBvgicxVmx41YpZWYP46lAC6ibxQ4nmkjw3Q49zrdFC7pyG5JTe1NYYMaibvyE9cM/640?from=appmsg "")  
  
作者对照补丁，怀疑对应 **CVE-2025-20658**  
（逻辑问题）与 **CVE-2025-20656**  
（缺边界检查）——以官方描述为准，编号是推断。  
## 黑卷点评  
  
BROM 补完、OEM 关 USBDL 之后，真正还在干活的，是 **Preloader + DA2**  
。Carbonara 不是终点；GSM 工具「世界首发」新 SoC，往往说明下载面又被撕开一条缝。  
- **声明长度 ≠ 实际写入**  
。fp_read_host_file  
 按 advertised 分配、按 packet 读取——少报多发就是洞。协议状态机要对齐看，不能只信 XML。  
  
- **混淆是给逆向准备的烟雾**  
。花文件名、垃圾 padding、主动关 UART，都会把你拖进 XML 死胡同；可控空字节在 USB 层。  
  
- **物理 USB + 未鉴权 DA 面**  
仍是量产与售后刷机的高频入口。升带 ALPS09474894 / ALPS09625423 的 DA；产线夹具别把「能进下载模式」当成默认可信。  
  
补丁之后，战场在 DA2——堆饵已经抛下去了。  
  
![黑卷点评示意：补丁之后战场在 DA2](https://mmbiz.qpic.cn/mmbiz_png/JdicK53hX84Xle4HJ4fKZQtrP7toYFTwzB54fX550NrxWWyryCXfkv8W3l5SqvB1l8xcOmayiaWrREjS2u8QCxOr1m8APFCfwKc63xFoRL4r0/640?from=appmsg "")  
  
 ⚠️ 免责声明  
  
 本人所有文章均为技术分享，均用于防御为目的的记录，请勿用于其他用途，否则后果自负。   
  
🔧 更多内容在星球「黑卷的IoT攻防日记」  
  
这里长期更新，专注 IoT / 车联网 / 机器人 / AI 安全实战：  
  
· 每天一篇一线漏洞拆解与复现思路，紧跟最新 CVE  
  
· 累计整车实测 20+、IoT 组件 100+ 的经验沉淀与踩坑笔记  
  
· 车联网 / V2X / 固件逆向 的资料、工具与字典  
  
· 星球里提问，我 24 小时内必答，一起挖洞、上分、接项目  
  
![星球「黑卷的IoT攻防日记」](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UA0okjC5RwUm10kAPD5XbiaW2J8ugzRHQ9JxThVM9o6scwMfcc2qAQ1wObWH6NOe5WYAZic0MWp8ybbBQhtrZhPU2purdBtUjT0/640?from=appmsg "")  
  
👇 扫码进星球「黑卷的IoT攻防日记」· 🎟 早鸟价 ¥98/年  
  
![知识星球付费二维码](https://mmbiz.qpic.cn/sz_mmbiz_png/JdicK53hX84UdZD4bFkwE80Mymmkk8icvvXS3nA1SDYFxk9pmiaxicXHVZLx4VJiawbibevJ5X1sUcVicPynHDaoTWC8lianB37P7ibEpu1BIjVjj5Po/640?from=appmsg "")  
  
**往期推荐**  
- [从调试口抽到改固件：Sonoff ZigBee 网关的芯片到云端两条 CVE](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247486121&idx=1&sn=fa00d6550883f102a66c5308fc9197e8&scene=21#wechat_redirect)  
  
  
- [EOL 路由照样打穿：Netgear WGR614v9 UART + Bitdefender Box SPI 降级 RCE](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247485429&idx=1&sn=5a4cdfd6982657765522258b8310892d&scene=21#wechat_redirect)  
  
  
- [从 UART 焊到未认证后门：ANJIA PTZ 摄像头（CVE-2026-31077）](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247486026&idx=1&sn=3786d4d0ca929f9d35b3913c6a6d6c1a&scene=21#wechat_redirect)  
  
  
