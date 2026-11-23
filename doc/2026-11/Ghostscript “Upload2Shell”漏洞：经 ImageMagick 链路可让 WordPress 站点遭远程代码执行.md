#  Ghostscript “Upload2Shell”漏洞：经 ImageMagick 链路可让 WordPress 站点遭远程代码执行  
原创 网络安全9527
                    网络安全9527  安全圈的那点事儿   2026-11-23 06:44  
  
一处被命名为 Upload2Shell 的 Ghostscript 漏洞，可把常规的 PDF 预览生成流程，变成使用 Imagick 且安装了有漏洞 Ghostscript 的 WordPress 站点上的远程代码执行入口。  
  
   
  
安全研究员 Paulos Yibelo 与 shturl 于 2026年11月21日披露该漏洞，报告称命令会以渲染进程的权限运行。官方 CVE 记录 CVE‑2026‑125396 认定 10.08.0 之前的 Ghostscript 版本受影响，CVSS 4.0 评分为 8.5。  
  
   
  
Ghostscript Upload2Shell 漏洞详情  
  
   
  
WordPress 本身并不需要直接调用 Ghostscript。当站点生成 PDF 预览时，它可能调用 PHP 的 ImageMagick 扩展 Imagick；随后 ImageMagick 再把文档交给 Ghostscript 渲染。  
  
   
  
此前另一起涉及 JPEG 2000 处理的 Ghostscript 漏洞也曾出现过同样的依赖链，但 Upload2Shell 的根因出在解释器的 Type 1 字体加载器中。仅仅把上传文件存下来并不会触发漏洞，文档必须到达那个有漏洞的渲染器才行。  
  
   
  
字体加载器中存在一处释放后重用（use‑after‑free）：内存已被释放后，软件仍继续使用对应的引用。研究人员证明，由此造成的内存破坏可以绕过 Ghostscript 的  -dSAFER  安全限制——该限制原本用来约束不受信任文档能做的操作。  
  
   
  
漏洞利用链示意（来源：PWN AI / Ghostescape）  
  
   
  
1. 类型错误（Type 1 路径）：在校验之前就发生拷贝，提交错误可被分类为可捕获错误；  
  
​  
  
2. 续接执行：受保护字典暴露，错误处理程序进入特权续接流程；  
  
​  
  
3. 恢复：本地对象被释放，但受保护字典仍持有该引用；  
  
​  
  
4. 重用：悬空引用被解引用，被释放的内存被复用，原生读写随之发生。  
  
   
  
对象生命周期：分配的本地对象在 restore 与内存复用后，字典里留下的安全引用指向了内容完全不同的复用内存。最终在 Ghostscript 内部：原生内存控制 → 路径控制状态被篡改 → -dSAFER 绕过。  
  
   
  
WordPress 侧的独立漏洞  
  
   
  
相关的 WordPress 核心公告把应用层暴露单独编号为 CVE‑2026‑65640。WordPress 在 7.0.4 版本修复了该应用层问题，但底层 Ghostscript 缺陷仍需另行修补。  
  
   
  
WordPress 核心这条路径需要媒体上传权限，默认情况下作者（Author）及以上角色都具备。研究人员还报告，某些 Forminator 与 Essential Addons for Elementor 插件配置，会把文档处理能力直接暴露给未登录访客。  
  
   
  
这与此前的 Forminator 文件删除漏洞不是一回事。这里的关键问题是：通过公开表单提交的文件，是否被送进了 WordPress 媒体库所用的同一套渲染栈。  
  
   
  
研究中还报告了另外两处命令执行场景：  
  
   
  
- Roundcube 服务端附件预览；  
  
​  
  
- Debian 13 上的共享 CUPS 打印队列。  
  
   
  
其中 Roundcube 的发现与此前提取反序列化漏洞相互独立。  
  
   
  
为什么 CVSS 评本地向量，却仍能远程打？  
  
   
  
这些演示可以解释一个常见疑问：为什么 Ghostscript CVE 标注的是本地攻击向量，却不能排除远程暴露？  
  
   
  
- 评分描述的是解释器本身的缺陷；  
  
​  
  
- 而一个面向网络的应用，可以代表远端用户，把不受信任的文档送进那个解释器。  
  
   
  
处置建议  
  
   
  
1. 更新 WordPress 核心及相关插件；  
  
​  
  
2. 随后必须核实操作系统或容器实际运行的 Ghostscript 是否为 10.08.0，或已回移植补丁的厂商软件包；  
  
​  
  
3. ⚠️ 仅更新 WordPress 并不会修复主机上供其他应用使用的 Ghostscript。  
  
   
  
如果暂时无法打补丁：  
  
   
  
- 条件允许时，暂停对不受信任文档的预览生成；  
  
​  
  
- 排查允许上传到媒体库的公开表单；  
  
​  
  
- 若预览功能必须保留，把渲染进程隔离到低权限工作进程中，限制文件系统访问，不给不必要的网络访问；  
  
​  
  
- 顺带检查附件预览、打印服务以及 Web 上传入口，共享打印队列要求认证。  
  
   
  
更新完成后，要核实部署的工作进程实际使用的是哪个 Ghostscript 构建版本。只检查独立可执行文件，可能会漏掉把 Ghostscript 当库链接使用的部署。监控侧同理：看不到独立的 Ghostscript 进程，并不能证明某个服务从未渲染过这类文件。  
  
