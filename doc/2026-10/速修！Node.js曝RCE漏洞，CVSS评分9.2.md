#  速修！Node.js曝RCE漏洞，CVSS评分9.2  
原创 微步情报局
                    微步情报局  微步在线研究响应中心   2026-10-23 13:07  
  
1. 结论  
  
  
**近日，Node.js官方发布了10月安全更新，本次更新修复了 10 个漏洞，包含 4 个高危漏洞。**  
其中最需要重点关注的是**CVE-2026-111497（Brotli 解码器RCE）。**  
  
另外三个高危漏洞为 CVE-2026-105482（dns.resolve*  
 空字节截断）、CVE-2026-105483（TLS 主机名校验绕过，仅影响 26.x / 24.x）和CVE-2026-106069（TLS 主机名校验绕过），其余为中危、低危漏洞。  
<table><tbody><tr><th style="border:1px solid #ddd;padding:12px;text-align:left;background:#f1f5f9;font-weight:bold;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;background-color:#f8f9fa;"><section><span leaf="">CVE 编号</span></section></th><th style="border:1px solid #ddd;padding:12px;text-align:left;background:#f1f5f9;font-weight:bold;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;background-color:#f8f9fa;"><section><span leaf="">严重程度</span></section></th><th style="border:1px solid #ddd;padding:12px;text-align:left;background:#f1f5f9;font-weight:bold;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;background-color:#f8f9fa;"><section><span leaf="">漏洞名称</span></section></th><th style="border:1px solid #ddd;padding:12px;text-align:left;background:#f1f5f9;font-weight:bold;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;background-color:#f8f9fa;"><section><span leaf="">受影响版本</span></section></th></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">CVE-2026-111497</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">高危</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">Brotli 解码器RCE</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">26.x、24.x、22.x</span></strong></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-105482</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">高危</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><code style="font-family:Consolas,monospace;font-size:0.92em;"><span leaf="">dns.resolve*</span></code><section><span leaf=""> 空字节截断</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-105483</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">高危</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">TLS 主机名校验绕过</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106069</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><strong style="font-weight:bold;color:#000000;"><span leaf="">高危</span></strong></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">TLS 主机名校验绕过</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-105481</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">中危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">HTTP 请求与响应顺序错误</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106072</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">中危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">TLS 证书主机名校验不当</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106074</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">中危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">密码学随机值重复</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106070</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">低危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">ZIP 校验失败后暴露不可信数据</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106071</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">低危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">权限模型文件系统访问绕过</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x</span></section></td></tr><tr><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">CVE-2026-106073</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">低危</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><code style="font-family:Consolas,monospace;font-size:0.92em;"><span leaf="">fs.realpathSync</span></code><section><span leaf=""> 权限绕过</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;font-size:14px;line-height:20px;text-indent:0;overflow-wrap:anywhere;"><section><span leaf="">26.x、24.x、22.x</span></section></td></tr></tbody></table>  
**建议：升级到 22.23.4、24.22.1 或 26.12.1，并重启进程。**  
 会解压不可信 Brotli（br  
）数据的进程优先完成升级与重启；优先处理面向互联网的服务。**20.x、21.x、23.x、25.x 已停止支持，官方本次不提供修复，需要迁到仍受支持的版本线。**  
  
2. 重点说明  
  
### 2.1 CVE-2026-111497　Brotli 解码器越界写  
  
**漏洞原理：**  
Brotli 流在声明只有一种数据块类型后再做块类型切换，属于格式错误。旧代码对该错误返回与「还需要更多输入」相同的值，调用方未检查返回值，解码器带着错误状态继续运行。随后对内部 8 字节残余比特缓存的写入可以越过该缓存，覆盖解码器状态中的位置、距离和环形缓冲区相关字段，形成越界写。补丁将该情况改为立即返回格式错误 BROTLI_DECODER_ERROR_FORMAT_BLOCK_SWITCH  
（值为 -17）。  
  
**受影响场景：**  
本漏洞在进程对不可信数据执行 Brotli 解压时触发。以下场景会执行该解压：  
  
1. 应用作为 HTTP **客户端**  
 访问外部资源。内置 fetch()  
（由 undici 实现）在响应头为 Content-Encoding: br  
 时调用 zlib.createBrotliDecompress  
。axios、got 默认同样解压，可配置关闭。按用户给定 URL 抓取外部资源（URL 预览、图片代理、webhook 回调、构建产物或软件包下载）且使用上述客户端时，可能形成该漏洞的攻击路径；实际触发需要上游返回特制的恶意 Brotli 数据。  
  
2. 应用作为 HTTP **服务端**  
，使用 Express 5 / body-parser 2 的 json()  
、raw()  
。请求头为 Content-Encoding: br  
 时解压请求体（inflate  
 选项默认为 true）。  
  
3. 应用自行调用 zlib.brotliDecompress  
、zlib.brotliDecompressSync  
、zlib.createBrotliDecompress  
（文件或构建产物解压、持续集成缓存等）。  
  
仅使用 http.createServer  
、未再接会解压的中间件时，入站请求体不会自动解压。  
  
**漏洞危害：**  
 Node.js 官方评级为 High，漏洞标题为越界写可导致远程代码执行。Google 给出 CVSS 4.0 评分 9.2（Critical）。具体影响分三层讨论：  
- 拒绝服务：构造流可让解码器自旋或挂起，Node 异步 zlib 默认只有 4 个 libuv 线程，几个请求就能占满并拖垮同进程的 fs/crypto/DNS，同步 API 直接卡事件循环。  
  
- 内存泄露：越界读可能把堆内容带进解压输出或错误路径，泄露密钥、指针等。  
  
- 远程代码执行：越界写可污染 pos  
、max_distance  
、ringbuffer  
 等字段，进而做任意读写并劫持控制流；Vercel 的安全公告也确认研究者在未修复的 22/24/26 上演示了进程内存泄露和命令执行。  
  
**可利用性需客观看待。**  
公开 PoC 在披露后 48 小时内已出现多个，Imperva 等团队在特定 Node 构建上完成了无需调试器的 RCE；**但成功往往依赖堆布局、分配器行为、ASLR 和多次尝试**  
，不是对任意 Node 服务都能稳定一击必中。  
  
**处置建议：**  
  优先升级并重启会解压不可信 Brotli 的进程。容器镜像、CI 运行器、Electron 等内嵌 Node 或同一套 Brotli C 库的程序一并升级。  
  
部分 Linux 发行版把 Node 动态链接到系统 libbrotlidec  
。这种情况除升级 Node 外，还需将系统 Brotli 库升到 1.0.10、1.1.1 或 1.2.1（或发行版已 backport 的对应版本）。  
### 2.2 CVE-2026-105483 / 106069 / 106072　TLS 主机名校验  
  
这三条漏洞针对 TLS 客户端主机名校验。CVE-2026-106069 与 CVE-2026-106072 为同一处代码变更。CVE-2026-105483 为 24.x / 26.x 上的回归，修复后与 106069 一同表现为拒绝非规范 IP 主机名。  
  
**漏洞原理：**  
 Node 作为 TLS 客户端校验对端证书时，部分主机名会被操作系统解析为 IP 地址（八进制、十六进制、短写、全角数字、表意句号 。  
 等）。此时连接按 IP 建立，证书校验却按 DNS 名称或错误的 IP 进行，证书与目标地址的对应关系校验失败。105483 只影响 24.x / 26.x。106069 与 106072 覆盖 22/24/26：原始主机名不是 IP、经国际化域名转换（IDNA）之后变成 IP 的，直接拒绝（错误信息为 Host is a non-canonical IP address  
）。  
  
**受影响场景：**  
 用 Node 向外发起 TLS 连接，并且主机名来自用户输入，或业务使用上述非规范 IP 写法。证书的 DNS 主体备用名称采用 IP 地址形式时，风险更高。  
  
**处置建议：**  
 随本次更新一优先升级并重启优先升级并重启并升级到 22.23.4、24.22.1 或 26.12.1。重点检查反向代理、出站网关、以及按用户给定 URL 发起请求的服务。  
### 2.3 CVE-2026-105482　dns.resolve* 在空字节处截断  
  
**漏洞原理：**dns.resolve*  
 把域名交给 c-ares 时按 C 字符串处理，查询停在第一个空字节。JavaScript 侧看到的仍是完整字符串。若应用用 endsWith('.trusted.corp')  
 这类后缀白名单判断 JavaScript 字符串，同时用 dns.resolve*  
 做真实查询，两侧看到的不是同一个域名。dns.lookup  
 / getaddrinfo  
 不受本条漏洞影响。  
  
**处置建议：**  
 随本次更新一并升级到 22.23.4、24.22.1 或 26.12.1。应用若调用 dns.resolve*  
，升级后主机名含空字节会抛出 ERR_INVALID_ARG_VALUE  
，需处理该异常。  
### 2.4 其余中危与低危漏洞  
  
**CVE-2026-105481：**  
 Node 作为 HTTP 客户端、启用 keep-alive 时，空闲套接字复用前未检查内核里是否已有未读数据。对端可以在两次请求之间写入一段伪造响应，使后续请求读到这段数据。补丁在复用前用 MSG_PEEK  
 检查，发现未读字节则丢弃该套接字并新建连接。  
  
**CVE-2026-106074：**crypto.randomUUID()  
 / randomInt()  
 的随机数缓存会写入用户态启动快照。从同一快照启动的多个进程会得到同一批随机值。普通 node app.js  
 不经过启动快照或单文件可执行应用（SEA）的，不受本条漏洞影响。  
  
**CVE-2026-106070（仅影响 26.x）：**zlib.ZipFile  
 在本地文件头校验失败后仍保留未完成的文件头，再次读取可能读出文件起始处在校验失败前已写入的数据。仅使用该 API 的 26.x 进程相关。  
  
**CVE-2026-106071、CVE-2026-106073：**  
 均要求进程启用 --permission  
。106073：realpathSync  
 在权限检查之前遍历路径，可用于路径探测。106071（仅影响 26.x）：通配符规则按路径长度匹配，更短前缀会获得超出预期的权限。未启用 --permission  
 的进程不受这两条漏洞影响。  
  
3. 自查方法  
  
### 确认版本与优先升级范围  
  
**1. 运行时版本**  
```
node -v
node -p "require('zlib').constants.BROTLI_DECODER_ERROR_FORMAT_BLOCK_SWITCH"
```  
  
**建议：**  
 使用 Node.js 22.x、24.x、26.x 的，分别升级至不低于 22.23.4、24.22.1、26.12.1 的版本，并重启相关进程。20.x、21.x、23.x、25.x 无官方修复，需要迁到仍受支持的版本线。  
  
**2. 该业务系统或服务是否解压不可信 Brotli**  
  
- 作为客户端：是否使用内置 fetch()  
，或 axios / got 等会按 Content-Encoding  
 自动解压响应的库，且会访问不可信的上游（含 URL 预览、图片代理、webhook、构建产物或软件包下载）  
  
- 作为服务端：是否使用 Express 5 / body-parser 2 等会按请求 Content-Encoding  
 解压请求体的中间件  
  
- 是否调用 zlib.brotliDecompress  
、zlib.brotliDecompressSync  
、zlib.createBrotliDecompress  
 解压来源不可信的数据  
  
**建议：**  
 优先升级并重启会解压不可信 Brotli 的业务系统或服务。  
### 确认运行时副本与系统库  
  
**1. 是否动态链接系统 Brotli**  
  
解码器可能编入 node  
 二进制，也可能位于系统的 libbrotlidec  
。官方二进制通常静态编入 Brotli。  
```
ldd "$(command -v node)" | grep -i brotli
```  
  
**建议：**  
 输出中出现 libbrotlidec  
 的，升级 Node 的同时将系统 libbrotli  
 升到 1.0.10、1.1.1 或 1.2.1（发行版包名与回移植版本以仓库为准）。未出现的，升级 Node 即可。  
  
**2. 容器、持续集成与仍在运行的旧二进制**  
  
升级宿主机 node  
 不会改变容器内的运行时。持续集成（CI）环境可能使用独立安装的运行时，Electron 应用通常内嵌运行时，均需分别核实。/proc/<pid>/exe  
 显示 (deleted)  
 表示该进程仍在使用已替换的旧文件。  
  
**建议：**  
 容器需更换基础镜像标签并重新构建，再分批更换运行中的实例。持续集成运行器、Electron 内嵌运行时需单独升级。对显示 (deleted)  
 的进程核实实际运行版本；若尚未运行修复版本，应在更新后重启。  
  
4. 修复方案  
  
  
**升级到修复版本（22.23.4、24.22.1 或 26.12.1）并重启。**  
容器更换镜像标签后重建、滚动发布。CI 运行器、Electron 应用同样升级其内嵌运行时。  
  
5. 临时缓解措施  
  
  
**CVE-2026-111497（Brotli 解码器越界写）**  
  
- 防护类设备或反向代理若能检查 HTTP 头部，可拒绝 Content-Encoding  
 中含 br  
 的入站请求（按编码令牌比较，忽略大小写）。  
  
- 应用作为客户端访问不可信上游时：axios、got 可关闭自动解压。  
  
**CVE-2026-105482（`dns.resolve*`域名空字节截断）**  
  
- 在调用 dns.resolve*  
 前检查域名：若包含空字节（'\0'  
），拒绝本次查询。  
  
**以上临时缓解不能替代升级，建议尽快升级到修复版本。**  
  
参考  
  
  
- Node.js 安全通告：https://nodejs.org/en/blog/vulnerability/october-2026-security-releases  
  
- 22.23.4：https://nodejs.org/en/blog/release/v22.23.4/  
  
- 24.22.1：https://nodejs.org/en/blog/release/v24.22.1/  
  
- 26.12.1：https://nodejs.org/en/blog/release/v26.12.1/  
  
- CVE-2026-111497：https://www.cve.org/CVERecord?id=CVE-2026-111497  
  
- undici fetch 解压：https://github.com/nodejs/undici/blob/main/lib/web/fetch/index.js  
  
- body-parser：https://github.com/expressjs/body-parser  
  
- Brotli C 库修复提交：https://github.com/google/brotli/commit/0ed4268a  
  
- CISA KEV：https://www.cisa.gov/known-exploited-vulnerabilities-catalog  
  
