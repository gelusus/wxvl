#  你的堆正在出卖你：从 Cobalt Strike 配置直扫到 HeapExplorer 堆取证  
S12
                    S12  赛博生存指南   2026-11-23 06:58  
  
> 原文：https://medium.com/@s12deff/your-heap-is-betraying-you-36027e4db132  
  
作者：S12 - 0x12Dark Development  
  
发布时间：2026-11-17  
  
翻译模型：GLM-5.3  
  
质量评分：★★★★☆（4/5） ｜ 标签：检测基础设施 · 红队工具链 · 分析工具与方法论  
  
  
**AI 摘要**  
：implant 作者盯 syscall、盯注入、盯免杀，却很少盯堆——但 std::string  
 超过 15 字符、std::vector  
 / std::map  
 的内部缓冲、std::unique_ptr  
 托管的对象全部落在堆上，一处 malloc  
 不写也会留痕。Cobalt Strike beacon 就是现成案例：运行时配置经 malloc  
 落堆、2048 字节的分配块尺寸成了 BeaconEye 直扫命中的签名，官方对策是 Sleep()  
 前用 HeapWalk()  
 逐块加密。作者配套开源了实时堆分析工具 HeapExplorer（块浏览、分配监控、熵评分、字符串提取、双快照 diff），并用一个带 BCFG / TMPD magic tag 的 PoC 完整演示了堆足迹怎么被逐项翻出来。  
  
堆内存取证这条线，仓库里此前从“内存工件伪装”的角度走过（Moneta 那篇看的是攻击者怎么给恶意内存做遮罩），这篇正好从对面看过来：不遮罩的时候，堆里到底躺着什么、谁来翻、翻到什么程度。S12 的文章一贯红队蓝队都照顾到——红队读者拿它当自查清单，检测工程师拿它当狩猎面。本文还带了一个作者自述“用 AI 快速写出来”的开源工具 HeapExplorer，七个功能点都对着堆取证的实际动作设计，配合文末 PoC 可以整套复跑。  
## 想更深入 Windows offensive development？  
  
作者提供从入门到进阶的视频课程与文字版模块（mini course），持续更新；另有每周更新的技术数据库（100+ 真实技术）以及面向团队的定制 C2 agent 与咨询服务，见其课程站 0x12darkdev.net。  
## 引言  
  
多数 implant 作者关心的是代码**做什么**  
：syscall、注入路径、免杀手法。堆这块，不到出事很少有人看它一眼。  
  
这篇文章讲堆里有什么、为什么这很重要。  
### 什么东西落进了堆  
  
在列举什么落堆之前，先得说清楚是哪个堆。每个 Windows 进程启动时就带一个默认进程堆，经 GetProcessHeap()  
 拿到；在这之上，CRT 会另建一个私有堆来伺候 malloc  
 / new  
。所以你的 implant 大概率同时在用至少两个堆，而这两个你都没显式管过。  
  
规则很简单：**implant 在运行期动态分配的任何东西，最后都落在堆上**  
。实际范围比直觉要大。  
  
显式调用是明摆着的：malloc  
、new  
、HeapAlloc  
、RtlAllocateHeap  
。  
  
但在 C++ 里你很少直接写这些。堆是通过标准库悄悄填满的：  
- std::string  
 超过 15 个字符就在堆上分配缓冲（多数实现用 Small String Optimization，短字符串留在栈上，一超长就溢到堆）  
  
- std::vector  
、std::map  
、std::unordered_map  
 的内部缓冲全部在堆上分配  
  
- std::unique_ptr  
 和 std::shared_ptr  
 把托管对象放在堆上  
  
所以，只要你的 implant 用 std::string  
 拼 C2 URL、用 std::vector<Task>  
 存收到的命令、用 std::unique_ptr<Config>  
 放配置结构体，这些东西就全在堆上——一行 malloc  
 都没写过。  
  
![文章题图：堆内存中的数据块示意](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqWUlrSFicXxUIRzHIcucJjgQwnLia8blBiaL7pWaS7dk52icFm5V51P6e89iaFkxBvzONegzBUeBhzLy3jWUqFpa7DQcibCJ0fGnOsao/640?wx_fmt=png&from=appmsg "")  
### C2 Agent：以 Cobalt Strike 为例  
  
Cobalt Strike 的 beacon 是个很好的参照点，因为它的堆行为已经被逆向清楚，围绕它建的检测工具也是公开的。  
  
beacon 把配置内嵌在二进制里，是一个打包的 TLV blob：每个设置项 type、length、value，整体与单字节密钥 XOR。beacon 启动后最先做的事之一，就是把内嵌配置加载进堆内存，随后把打包的原始配置整体清零。  
  
运行时配置是一个位于堆上的 C/C++ 数组，由 beacon 代码解析内嵌配置后建起来，二进制数据的内存经 malloc  
 动态分配。  
  
于是运行期的内存里，你有这么几样东西：  
- 运行时配置数组：malloc  
 在堆上，每个配置项一个条目  
  
- 二进制值（比如 RSA 公钥）：单独 malloc  
，指针存在数组里  
  
- 指向该数组的指针，存在 .data  
 里  
  
BeaconEye 就只扫堆内存，找 Cobalt Strike 在初始化时动态分配的配置结构。分配块大小是 2048 字节——一个已知签名，走堆条目时定位配置块相当直接。  
  
Cobalt Strike 对此的应对是加了一个新思路：Sleep()  
 之前把堆加密，醒来后再解密。用 HeapWalk()  
 可以遍历堆里每一个分配块逐块加密，加密前挂起全部线程，加密完成后再恢复。  
## HeapExplorer  
  
工具仓库：https://github.com/S12cybersecurity/HeapExplorer  
  
出于这个目的，我直接用 AI 快速做了个工具：一个实时堆分析工具，专门服务本文这类研究。  
### 功能  
- 浏览目标进程的所有活动堆块：**地址、大小、内容预览**  
  
- 实时观察每一次分配与释放  
  
- 用熵评分揪出加密或压缩过的缓冲  
  
- 直接从堆内存提取字符串：C2 域名、管道名、路径（std::string  
）  
  
- 对比两个快照，看清两次之间到底变了什么  
  
- 标记可执行堆区与没有落盘模块背书的分配  
  
- 附加到运行中的进程，或直接从工具里启动一个  
  
## PoC  
  
先来分析这段代码：  
```
#include <windows.h>  #include <wincrypt.h>  #include <iostream>  #include <string>  #include <vector>  #include <cstring>  #pragma comment(lib, "advapi32.lib")  // Magic tags, first 4 bytes of each block, visible in HeapExplorer's hex preview  #define TAG_CONFIG  0x47464342  // "BCFG"  #define TAG_TEMPBUF 0x44504D54  // "TMPD"  struct BeaconConfig {      uint32_t magic;      uint8_t  aes_key[32];      uint8_t  c2_encrypted[64];      uint32_t sleep_ms;      uint32_t jitter;      uint32_t version;  };  static void XorBuffer(uint8_t* buf, size_t len, uint8_t key) {      for (size_t i = 0; i < len; i++) buf[i] ^= key;  }  static void FillHighEntropy(uint8_t* buf, size_t len) {      HCRYPTPROV hProv = 0;      if (CryptAcquireContextA(&hProv, nullptr, nullptr, PROV_RSA_FULL, CRYPT_VERIFYCONTEXT)) {          CryptGenRandom(hProv, (DWORD)len, buf);          CryptReleaseContext(hProv, 0);      }  }  int main() {      HANDLE hSecureHeap = HeapCreate(HEAP_NO_SERIALIZE, 0, 0);      if (!hSecureHeap) {          std::cerr << "HeapCreate failed\n";          return 1;      }      BeaconConfig* config = (BeaconConfig*)HeapAlloc(hSecureHeap, HEAP_ZERO_MEMORY, sizeof(BeaconConfig));      if (!config) {          HeapDestroy(hSecureHeap);          return 1;      }      config->magic = TAG_CONFIG;      config->sleep_ms = 15000;      config->jitter = 500;      config->version = 1;      FillHighEntropy(config->aes_key, sizeof(config->aes_key));      FillHighEntropy(config->c2_encrypted, sizeof(config->c2_encrypted));      // These exceed SSO threshold — heap-allocated in the default process heap      std::string c2_domain = "192.168.1.100:4444";      std::string user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64)";      std::string pipe_name = "\\\\.\\pipe\\svchost_comms";      std::cout << "Beacon initialized. C2: " << c2_domain << "\n";      std::cout << "Take initial snapshot in HeapExplorer now...\n";      Sleep(3000);      for (int i = 0; i < 5; i++) {          std::cout << "Cycle " << i + 1 << " — rotating AES key...\n";          // Short-lived decryption buffer — TAG_TEMPBUF in first 4 bytes          size_t alloc_size = sizeof(uint32_t) + sizeof(config->c2_encrypted);          uint8_t* temp_buf = (uint8_t*)HeapAlloc(hSecureHeap, 0, alloc_size);          if (temp_buf) {              *(uint32_t*)temp_buf = TAG_TEMPBUF;              uint8_t* payload = temp_buf + sizeof(uint32_t);              memcpy(payload, config->c2_encrypted, sizeof(config->c2_encrypted));              XorBuffer(payload, sizeof(config->c2_encrypted), 0xAB);              HeapFree(hSecureHeap, 0, temp_buf);          }          // Rotate key: same block address, different content — Snapshot Diff shows Modified          FillHighEntropy(config->aes_key, sizeof(config->aes_key));          // C2 response: vector buffer on the default process heap, high entropy          std::vector<uint8_t> c2_response(256);          FillHighEntropy(c2_response.data(), c2_response.size());          std::cout << "C2 response received (" << c2_response.size() << " bytes)\n";          std::cout << "Sleep " << config->sleep_ms << "ms...\n";          Sleep(config->sleep_ms);      }      SecureZeroMemory(config, sizeof(BeaconConfig));      HeapFree(hSecureHeap, 0, config);      HeapDestroy(hSecureHeap);      std::cout << "Beacon done. Custom heap destroyed.\n";      return 0;  }
```  
  
在 HeapExplorer 里最先看到的是 IOC 字符串：  
  
![HeapExplorer 字符串面板：C2 地址与管道名被标为 IOC](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqXenPxlktT6mz0crfS8k7HYFAVUyicqJPzYAhymFeybrgGuuLINZOoCiaolNeZ8YvVia8zll6libtVtv59DdkcsodypgxLedYIDF5I/640?wx_fmt=png&from=appmsg "")  
  
接着找另外两个为配置分配的内存缓冲。第一个就是 BeaconConfig：  
```
struct BeaconConfig {      uint32_t magic;      uint8_t  aes_key[32];      uint8_t  c2_encrypted[64];      uint32_t sleep_ms;      uint32_t jitter;      uint32_t version;  };
```  
  
**BCFG**  
 前缀标记出我们的配置缓冲。下面这张快照处于初始化阶段、第一轮密钥轮换之后：  
  
![HeapExplorer 块详情：BCFG 开头的配置块十六进制视图](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqV5VGicVBaDmsFjKCnlEzEhVVEibeGvSOckOIz7GcmqjhA5QCHT0EAeOpjlnWhSTvWuR1WgHqpOSLibYrmaA4EtdELvl3Cq68rmvk/640?wx_fmt=png&from=appmsg "")  
  
对比字节 4–35，能看出只有它们变了——正是密钥轮换操作。  
  
数据是加密的，所以不逆向就没法读出明文。  
  
![同一 BCFG 配置块在密钥轮换后的块详情视图](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqU4nAYfF251RWxia3F5OMuCtiaN6pXibNlNw1Rk5tJWhic3NZps0f0icjDIuEL2SINa1TJ1T2MZCVmMP8aksulkChMS86MVKpknuZFI/640?wx_fmt=png&from=appmsg "")  
## 结论  
  
堆里躺着的东西，比你显式放进去的多。标准库类型、运行时配置结构体、临时解密缓冲，全都会留下工件，直到显式释放——有些时候留得更久。  
  
关注作者：X（@Salsa12__）｜Discord｜Instagram｜YouTube。  
## 译者补充：几点解读与延伸  
  
**一、SSO 阈值不是通用常数。**  
 原文说 std::string  
 超 15 字符落堆，这是 MSVC 和 libstdc++ 的口径（内部缓冲 16 字节，留 1 字节给 \0  
）；libc++（LLVM）的阈值是 22 字符。写跨平台 implant 或做扫描规则时，按 15 当硬边界会把 libc++ 构建下的短字符串漏判成「必然在栈上」。  
  
**二、BeaconEye 靠什么命中，天花板在哪。**  
 BeaconEye 用 YARA 规则扫进程内存里的 beacon 配置结构，2048 字节分配块尺寸是它的定位锚点之一。这篇讲清了它为什么只扫堆：配置解析后必然经 malloc  
 落堆，明文窗口在初始化之后就一直开着，直到 sleep 加密生效。反过来说，一旦 beacon 采用了 sleep 期堆加密（CS 官方 2021 年加的 sleep mask 就是这个思路），直扫就只剩唤醒后的短暂窗口——这正是攻防两边在“时间窗”上做文章的地方，和仓库已译的《9 秒窗口 EDR 开机竞态》是同一类博弈。  
  
**三、堆加密的代价。**HeapWalk()  
 逐块加密要挂起全部线程，对时延敏感的任务链有副作用；而且只护住 sleep 期，运行中的配置读取、任务解析仍在堆上留瞬时明文。检测侧真正的机会在“分配行为”而非“内容”：固定尺寸的配置块、周期性的高熵整体改写（本文 PoC 的密钥轮换就是最典型信号）、无落盘模块背书的可执行堆分配——HeapExplorer 的快照 diff 和熵评分就是照这个思路做的。  
  
**四、对工具本身的合理预期。**  
 作者自述 HeapExplorer 是“用 AI 快速做的”，定位是研究辅助而非工程化产品；它的价值在七个功能点对应的检查动作，而不是稳定性。仓库读者若要上手，建议先在 PoC 样本上复跑本文流程，再考虑接到自己的分析流里。  
## 参考  
- 原文：https://medium.com/@s12deff/your-heap-is-betraying-you-36027e4db132  
  
- HeapExplorer：https://github.com/S12cybersecurity/HeapExplorer  
  
- 作者课程站：https://0x12darkdev.net/  
  
