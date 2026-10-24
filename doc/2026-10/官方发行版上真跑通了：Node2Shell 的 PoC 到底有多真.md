#  官方发行版上真跑通了：Node2Shell 的 PoC 到底有多真  
原创 Red Hunter
                    Red Hunter  黑白之道   2026-10-24 01:29  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/nGzNudUIJ6N1qc5xWWW7ulvchz0xOYhPaMN8cib9fHoTOxyq1VMVic9N9als7sCMVy2dgAlJpeGfc4kkpwnuhJPSq5w4m1dQqx0AoQhELCoBc/640?from=appmsg "")  
> **导语**  
：10 月 21 日，Node.js 给 Brotli 解码器那个越界写打了补丁。10 月 23 日，Imperva 把 PoC 的实拍放了出来——在官方 24.22.0 镜像上，一段恶意 Brotli 数据真的把代码跑起来了。技术上这叫"独立验证"，实操上这叫"从公告变成了事故预告片"。  
  
## 一、这次和之前那个 PoC 差在哪  
  
之前流传的那份 PoC，只在**故意削弱过的测试构建**  
上拿到过代码执行。在标准构建上，它的作者报出来的是崩溃加有限的进程内存读，并且明说没有实现对正常构建的攻击。  
  
Imperva 红队补上了这一环。他们的独立 PoC 在**官方 Node.js 24.22.0 的 Linux arm64 镜像**  
上跑通了，两条路径都能触发：一条是应用内置 fetch() 去消费外部服务器的 Brotli 压缩响应，另一条是请求体侧的 Express 5 和 body-parser 2 解压进来的 Brotli 请求体。不用调试器，不用插桩二进制，新起的干净容器里直接复现。  
  
验证手法很干净：测试程序里**根本没有创建标记文件的代码**  
，收到那段 Brotli 数据之后，文件自己冒出来了。载荷里的代码确实在目标进程里执行过。  
## 二、根因一句话讲清  
  
Brotli 流在解码推进过程中会声明接下来用哪种块类型。构造数据走到一个"按流自身的结构根本不可能发生"的块切换点，有漏洞的解码器把这个非法状态读成了"还需要更多输入"，于是带着一个已经不对的内部状态继续往下跑。内存消毒器测试显示，从这个非法状态继续处理就会写出预期缓冲区之外。修复方式是在这个位置直接返回格式错误并交给调用方——同样那批输入，在打了补丁的 24.22.1 上全部被拒，标记文件也没出现。  
  
![Imperva 的两条触发路径与修复后行为](https://mmbiz.qpic.cn/mmbiz_png/nGzNudUIJ6P9F8zRGicKyXL3jUfJRNCE3vYtib1l9U5ibgjGlGGUj7R0AOEHJUjNHGgF7HMJw2yOiaEZagH0nXicmfHVPZ1wbPF87JoT2n295EmE/640?from=appmsg "Imperva 的两条触发路径与修复后行为")  
  
图：Imperva 威胁研究团队的示意图，两条路径汇入同一个解码器，来源 imperva.com  
## 三、别急着恐慌，可达边界比想象中窄  
  
**不是每次都成。**  
 Imperva 自己写得很清楚：PoC 并没有在每一次尝试中都成功，他们的执行"需要多次尝试"。概率性意味着它不好做成稳定的武器，也意味着你不能指望靠一次扫描发现入侵。  
  
**覆盖面也没铺满。**  
 只测了 Linux arm64 这一个构建和一套应用配置；22.x 和 26.x 两条线没测。中间件侧测的是 Express 5.0.0 到 5.3.0、body-parser 2.0.0 到 2.3.0 的固定版本，不是逐个版本测过来。  
  
**触发面只有两条。**  
 Express 5 和 body-parser 2 是被配置成会解压 Brotli 请求体的入口，它们只是路，不是洞本身；洞在 Node.js 自带的原生解码器里。另外提一句：裸的 Node.js HTTP 服务器收到 Brotli 请求体并不会自动解压，得有中间件或者显式调用。  
## 四、今天就能做的四件事  
  
**升级，而且要升级到镜像里。**  
 修复版本是 22.23.4、24.22.1、26.12.1。要命的是包清单会骗你——容器镜像里捆绑的那份旧 Node 二进制，SCA 扫包很可能根本看不到。逐个进容器里查版本。  
  
**已经 EOL 的线没有补丁。**  
 20.x 及更早拿不到修复，只能迁到受支持的分支。  
  
**临时缓解。**  
 在更新的窗口期内，尽量限制受影响路径上的不可信 Brotli 流量，Imperva 自己也只给到这个建议——官方公告里没有任何运行时层面的绕过方案。  
  
**WAF 只能挡一半。**  
 Imperva 已经上线了针对性规则并用复现的攻击验证过，但规则管不到应用主动 fetch 回来的 Brotli 响应，那条出站路径只能靠升级解决。  
  
至于排查线索，解压期间进程异常退出、解码器报错可以拿来排优先级，但都不是可靠的被利用信号——得结合运行时版本、应用日志和压缩数据来源一起看。截至 10 月 22 日，还没有野外利用的报告。  
  
最后是 Imperva 自己的判断：AI 正在同时压缩漏洞发现和武器化的时间，防守方在"披露"和"修复"之间的缓冲空间只会越来越小。这话听着像软文，但放在这个洞上很具体——从补丁发布到 PoC 在官方发行版上跑通，只隔了两天。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nGzNudUIJ6PLp8zkzEQLrJia0rAjKhhb4XsfymxKmYSASBcbibicyqNkZe6dAwMLnwXnM9fDB8qicrwxP1AFIz6IAG9crjvcIibpic69zPTXBW8qE/640?from=appmsg "")  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621549&idx=1&sn=21c4b072726d2387d562109ada6b9bbb&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650621944&idx=1&sn=3cc6dc9876a20466d4ec3634deb80220&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650622065&idx=1&sn=09f8ae84c06d4331e71c277b177ae701&scene=21#wechat_redirect)  
> 👇 点击**阅读原文**  
，访问我的网站  
  
  
